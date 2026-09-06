# ADR 0003: Estrategia de Bases de Datos y Persistencia Políglota (Database-per-Service)

## Estado
Aceptado

## Contexto
En el desarrollo de la plataforma Discordia coexisten dominios con naturalezas de datos, requerimientos de consistencia y patrones de acceso radicalmente dispares:
1. **Datos altamente estructurados, relacionales y transaccionales:** Autenticación, perfiles, membresías, jerarquías de roles, permisos granulares y balances financieros/compras (puntos, suscripciones, boosts). En estos flujos, la pérdida de datos o inconsistencias parciales (ej. cobro sin acreditación de puntos) son inaceptables.
2. **Datos semiestructurados de alta concurrencia y volumen masivo:** Mensajería en tiempo real en canales y DMs, donde los mensajes poseen estructuras variables (texto plano, menciones, embeds, arrays de reacciones y adjuntos multimedia) y requieren escrituras y lecturas paginadas de bajísima latencia.
3. **Datos efímeros y en memoria:** Estados de presencia ("usuario escribiendo...", estado de micrófono/cámara en salas de voz) y canales de publicación/suscripción (*Pub/Sub*) para distribuir eventos en tiempo real entre múltiples réplicas de WebSockets.

Adicionalmente, las restricciones de la cátedra establecen expresamente:
- **Prohibición de monolitos de datos:** No se permite que múltiples servicios compartan una única base de datos centralizada.
- **Persistencia políglota obligatoria:** Se debe utilizar al menos dos tecnologías de bases de datos distintas, garantizando como mínimo una SQL y una NoSQL.

---

## Decisión

Se adopta el patrón **Database-per-Service** complementado con una estrategia de **Persistencia Políglota**, seleccionando **PostgreSQL 16 (SQL Relacional)**, **MongoDB 7 (NoSQL Documental)** y **Redis 7 (NoSQL In-Memory / Key-Value)**.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                               PATRÓN DATABASE-PER-SERVICE                              │
├───────────────────────────────┬───────────────────────────────┬────────────────────────┤
│     PostgreSQL 16 (SQL)       │     MongoDB 7 (Document)      │   Redis 7 (In-Memory)  │
├───────────────────────────────┼───────────────────────────────┼────────────────────────┤
│ • Auth & User Service         │ • Chat Engine Service         │ • Chat Engine Service  │
│ • Server & Roles Core Service │   (Historial de mensajes,     │   (Pub/Sub réplicas)   │
│ • Payments & Monetization     │    adjuntos y reacciones)     │ • Voice Signaling      │
│ • Notification & Audit        │                               │   (Sesiones efímeras)  │
└───────────────────────────────┴───────────────────────────────┴────────────────────────┘
```

### 1. Principio de Autonomía de Datos (Database-per-Service)
- Cada microservicio es el **único dueño de su almacén de datos**.
- Queda terminantemente prohibido que un microservicio acceda de forma directa (vía conexión TCP/driver) a la base de datos de otro servicio. Toda interacción inter-dominio se realiza exclusivamente a través de APIs públicas (REST) o eventos asíncronos en RabbitMQ.
- Esto elimina el acoplamiento a nivel de esquema (*schema coupling*), permite evolucionar y migrar bases de datos de forma independiente y aisla fallos de infraestructura.

---

### 2. Fundamentación y Mapeo Detallado por Servicio

#### A. PostgreSQL 16 (Bases de Datos Relacionales con Garantías ACID)
PostgreSQL fue elegido para los servicios que requieren transaccionalidad estricta (**ACID: Atomicidad, Consistencia, Aislamiento y Durabilidad**) e integridad referencial compleja:

1. **`Auth & User Service` (`auth_db`):**
   - **Esquema:** Tablas `users`, `user_profiles`, `federated_identities`, `two_factor_secrets`, `password_reset_tokens`.
   - **Justificación:** Requiere restricciones de unicidad estrictas (`UNIQUE` en emails y usernames) y transacciones atómicas al registrar usuarios (creación de cuenta + perfil inicial + asignación de rol inicial en un único bloque atómico).
2. **`Server & Roles Core Service` (`server_db`):**
   - **Esquema:** Tablas `servers`, `channels`, `categories`, `server_members`, `roles`, `permissions`, `role_assignments`, `channel_permission_overrides`, `server_bans`, `server_mutes`.
   - **Justificación:** Modelo relacional densamente conectado. Aprovecha `Foreign Keys` con políticas de eliminación (`ON DELETE CASCADE`) para garantizar que al eliminar un servidor o canal no queden permisos o roles huérfanos.
3. **`Payments & Monetization Service` (`payments_db`):**
   - **Esquema:** Tablas `wallets`, `point_transactions`, `subscriptions`, `server_boosts`, `idempotency_records`.
   - **Justificación:** Cero tolerancia a la inconsistencia o a condiciones de carrera (*race conditions*). Las operaciones de descuento de puntos y aplicación de beneficios se ejecutan bajo aislamiento transaccional (`SERIALIZABLE` / `READ COMMITTED` con `SELECT ... FOR UPDATE`), garantizando que un usuario no pueda gastar el mismo saldo dos veces.
4. **`Notification & Audit Service` (`notification_db`):**
   - **Esquema:** Tablas `notifications`, `notification_preferences`, `audit_logs`.
   - **Justificación:** Permite consultas estructuradas complejas para el staff del Backoffice (ej. filtrar logs de auditoría por rango temporal, tipo de acción, moderador y servidor afectado).

---

#### B. MongoDB 7 (NoSQL Documental Orientado a Mensajería)
Utilizado exclusivamente como almacén persistente de **`Chat Engine Service` (`chat_db`)**:

- **Estructura Documental del Mensaje:**
  ```json
  {
    "_id": ObjectId("66d5b1f8e4b0a1a2b3c4d5e6"),
    "channel_id": "srv1-channel-general",
    "author": {
      "id": "usr-123",
      "username": "martin",
      "avatar_url": "https://s3.../avatar.png"
    },
    "content": "¡Hola a todos!",
    "attachments": [
      { "url": "https://s3.../diagram.png", "type": "image/png", "size_bytes": 1048576 }
    ],
    "reactions": [
      { "emoji": "👍", "count": 3, "users": ["usr-123", "usr-456", "usr-789"] }
    ],
    "reply_to_message_id": null,
    "edited_at": null,
    "created_at": ISODate("2026-09-02T10:00:00Z")
  }
  ```
- **Justificación:**
  1. **Documentos Auto-Contenidos vs. Múltiples JOINs:** En un modelo relacional (SQL), renderizar un mensaje con adjuntos y reacciones exigiría cruzar al menos 4 tablas (`messages`, `attachments`, `reactions`, `reaction_users`), degradando el rendimiento bajo alto volumen. En MongoDB, el mensaje se persiste y recupera como una única entidad desnormalizada y completa.
  2. **Alto Throughput de Escritura:** Optimizado para inserciones masivas concurrentes en tiempo real.
  3. **Paginación Eficiente:** Indexación compuesta `{ channel_id: 1, created_at: -1 }` que permite recuperar páginas de los últimos 50 mensajes en submilisegundos mediante *cursor-based pagination*.

---

#### C. Redis 7 (NoSQL In-Memory y Mensajería Efímera)
Utilizado como almacén de alta velocidad en memoria RAM (latencia < 1ms) en dos componentes:

1. **`Chat Engine Service` (Bus Pub/Sub Inter-Instancia):**
   - Cuando el servicio se escala horizontalmente (ej. Réplicas A, B y C), un mensaje recibido por la Réplica A se publica en el tópico Redis `channel:{channel_id}`. Las réplicas B y C reciben el payload inmediatamente y lo emiten a sus respectivos clientes conectados por WebSocket.
2. **`Voice Signaling Service` (Estado Efímero de Sesiones de Voz):**
   - Mantiene la lista de participantes activos en salas de voz y sus estados de audio/video (`muted`, `deafened`, `screensharing`).
   - *Razón técnica:* Si un servidor de voz se reinicia, las llamadas WebRTC activas se interrumpen; persistir estos estados en disco sobre SQL provocaría escrituras innecesarias e I/O contention. Redis maneja estos datos 100% en RAM con expiración automática (*TTL*).

---

## 3. Patrones de Integración y Manejo de Datos Distribuidos

Al no contar con JOINs inter-base de datos, se aplican los siguientes patrones estándar de microservicios:

### 3.1. Desnormalización Controlada (*Snapshotting*)
Para evitar llamadas sincrónicas masivas al `Auth Service` cada vez que se leen mensajes de chat, el `Chat Engine` guarda una instantánea inmutable del autor (`username`, `avatar_url`) dentro del documento del mensaje al momento de crearlo.
- Ante actualizaciones de perfil, el `Auth Service` emite el evento asíncrono `UserProfileUpdated` a RabbitMQ para invalidar cachés o sincronizar estados según sea necesario.

### 3.2. Transactional Outbox Pattern (Garantía de Entrega de Eventos)
Para evitar el problema del *dual-write* (escribir en la base de datos pero fallar al publicar en el Message Broker):
- En PostgreSQL, los servicios escriben la entidad de negocio y el evento en la tabla local `outbox_events` dentro de la **misma transacción ACID**.
- Un worker asíncrono lee los eventos pendientes, los publica en RabbitMQ con *Publisher Confirms* y los marca como despachados.

---

## Alternativas Descartadas

1. **Base de Datos Única Compartida (Shared Monolithic Database):**
   - *Descartada:* Viola los principios de microservicios, crea un punto único de falla (*SPOF*), acopla los despliegues de todos los equipos y contradice explícitamente el enunciado de la materia.
2. **Persistencia 100% Relacional (Solo PostgreSQL para todo):**
   - *Descartada:* Obligaría a normalizar el historial de chat en múltiples tablas relacionales con alto costo de `JOINs` en lectura, o almacenar JSONs opacos en columnas `JSONB` perdiendo las optimizaciones de indexación y sharding nativas de un motor documental como MongoDB, además de no cumplir el requerimiento de emplear al menos una tecnología NoSQL.
3. **Apache Cassandra / ScyllaDB para el Chat:**
   - *Pros:* Escalabilidad lineal extrema para escritura de mensajes.
   - *Contras:* Complejidad operacional desmedida para el alcance del TP (configuración de nodos, particionamiento y tombstone management). MongoDB y Redis satisfacen con creces los escenarios de carga y pruebas con k6 con menor fricción operativa.

---

## Consecuencias

### Positivas
- **Autonomía y Escalabilidad:** Cada base de datos puede escalarse, respaldarse y optimizarse de acuerdo a su perfil de carga particular.
- **Rendimiento Óptimo:** Consultas de chat ultra rápidas en MongoDB y transacciones financieras infalibles en PostgreSQL.
- **Cumplimiento Normativo:** Cumple rigurosamente con los requisitos de la cátedra (base de datos por servicio, uso de SQL y NoSQL).

### Negativas y Mitigaciones
- **Consistencia Eventual:** La información compartida entre servicios no es inmediatamente consistente; se mitiga mediante diseño orientado a eventos (RabbitMQ) y desnormalización controlada.
- **Sobrecarga de Operación Local:** Requiere levantar múltiples contenedores en el entorno de desarrollo; se resuelve centralizando la inicialización mediante `docker-compose` y scripts de seeding automatizados.
