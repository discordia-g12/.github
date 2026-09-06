# ADR 0004: Comunicación Asíncrona con RabbitMQ, Topología de Exchanges y Patrones Distribuidos

## Estado
Aceptado

## Contexto
En arquitecturas distribuidas orientadas a microservicios, existen dos paradigmas fundamentales para la interacción entre servicios:

### 1. El Problema del Modelo Sincrónico Encadenado (HTTP Request/Response)
Si ante una acción del usuario (ejemplo: un moderador banea a un usuario de un servidor mediante `POST /servers/{id}/bans`), el servicio emisor intenta coordinar todo de forma síncrona:
1. Registra el baneo en su base de datos.
2. Llama vía HTTP a `Chat Engine Service` para cerrar sus WebSockets activos.
3. Llama vía HTTP a `Voice Signaling Service` para desconectarlo de la sala de voz.
4. Llama vía HTTP a `Notification Service` para enviarle un correo/push informando la sanción.
5. Llama vía HTTP a `Audit Service` para registrar la acción del moderador.

**Consecuencias críticas del modelo síncrono:**
- **Amplificación de Latencia:** La respuesta al usuario tarda la suma del tiempo de los 5 servicios ($T = t_1 + t_2 + t_3 + t_4 + t_5$).
- **Acoplamiento Temporal:** El cliente y **todos** los servicios involucrados deben estar vivos, saludables y disponibles al mismo milisegundo.
- **Fallos en Cascada:** Si el servicio de Notificaciones o Auditoría se cae o sufre lentitud, la petición del moderador falla o se bloquea, impidiendo el baneo.

### 2. La Solución: Desacople en el Tiempo mediante Asincronismo
En el modelo asíncrono, el servicio que procesa la petición del usuario realiza su cambio de estado local de inmediato, emite un **Evento** a un intermediario (**Message Broker**) y responde al cliente con éxito en milisegundos.
El intermediario almacena el evento en colas persistentes (**FIFO**), amortigua picos de tráfico (*buffering*) y garantiza que los servicios interesados consuman el mensaje a su propio ritmo, incluso si estuvieron momentáneamente caídos.

---

## Decisión

Se decide adoptar **RabbitMQ** como Message Broker central, utilizando una arquitectura basada en **Topic Exchanges**, complementada con la combinación de **Pub-Sub (entre servicios)** y **Work Queues (entre réplicas de un mismo servicio)**, y asegurada mediante el **Transactional Outbox Pattern** e **Idempotencia**.

```
                               ┌────────────────────────────────────────────────────────┐
                               │                 RABBITMQ BROKER                        │
                               │                                                        │
                               │               ┌───────────────────────┐                │
                               │               │     Topic Exchange    │                │
                               │               │   "discordia.events"  │                │
                               │               └───────────┬───────────┘                │
                               │                           │                            │
                               │         Routing Key:      │                            │
                               │    "server.member.banned" │                            │
                               │                           │                            │
                               │        ┌──────────────────┼──────────────────┐         │
                               │        │                  │                  │         │
                               │        ▼ (Binding)        ▼ (Binding)        ▼         │
                               │   ┌─────────┐        ┌─────────┐        ┌─────────┐    │
                               │   │  Cola   │        │  Cola   │        │  Cola   │    │
                               │   │  Chat   │        │   Voz   │        │ Notif   │    │
                               │   └────┬────┘        └────┬────┘        └────┬────┘    │
                               └────────┼──────────────────┼──────────────────┼─────────┘
                                        │                  │                  │
               ┌────────────────────────┘                  │                  └────────────────────────┐
               ▼ (Work Queue)                              ▼ (Work Queue)                              ▼ (Work Queue)
       ┌───────────────┐                           ┌───────────────┐                           ┌───────────────┐
       │  Chat Worker  │                           │  Voice Worker │                           │ Notif Worker  │
       │   (Pod 1)     │                           │   (Pod 1)     │                           │   (Pod 1)     │
       └───────────────┘                           └───────────────┘                           └───────────────┘
               ▲ (Compite por la cola)                     ▲ (Compite por la cola)                     ▲ (Compite por la cola)
       ┌───────────────┐                           ┌───────────────┐                           ┌───────────────┐
       │  Chat Worker  │                           │  Voice Worker │                           │ Notif Worker  │
       │   (Pod 2)     │                           │   (Pod 2)     │                           │   (Pod 2)     │
       └───────────────┘                           └───────────────┘                           └───────────────┘
```

---

## 1. Topología de RabbitMQ: Exchanges, Colas y Enrutamiento

### A. Topic Exchange (`discordia.events`)
En lugar de publicar directamente a una cola, los productores publican a un único **Topic Exchange** centralizado. El exchange enruta los mensajes hacia las colas evaluando la **Routing Key** con comodines:
- `*` (asterisco): Coincide exactamente con una palabra.
- `#` (numeral): Coincide con cero o más palabras.

**Ejemplo de Nomenclatura de Routing Keys:**
- `server.member.banned`
- `server.member.muted`
- `server.channel.deleted`
- `chat.message.created`
- `user.profile.updated`
- `payment.boost.applied`

### B. Unión de Patrones: Pub-Sub (Notificar) + Work Queue (Repartir)
El diseño aplica la sinergia de ambos patrones:
1. **Pub-Sub (Nivel Inter-Servicio):**
   - Cada microservicio interesado crea su **propia cola duradera y persistente** y la enlaza (*bind*) al Exchange con los patrones que le interesan:
     - `chat_events_queue` $\rightarrow$ Enlazada a `server.member.*`, `server.channel.deleted`.
     - `voice_events_queue` $\rightarrow$ Enlazada a `server.member.banned`, `server.member.muted`.
     - `notification_events_queue` $\rightarrow$ Enlazada a `*.created`, `*.banned`, `payment.*`.
   - Cuando el Exchange recibe `server.member.banned`, **duplica y entrega una copia del mensaje a cada una de las 3 colas**.
2. **Work Queue (Nivel Intra-Servicio / Réplicas):**
   - Si el `Chat Engine Service` corre con 3 instancias (Pod 1, Pod 2, Pod 3), **las 3 instancias consumen de la misma cola `chat_events_queue`**.
   - RabbitMQ distribuye los mensajes de esa cola uno a uno (*Round-Robin* con `prefetch_count=1`), garantizando que **cada mensaje sea procesado por un solo worker de ese servicio**.

---

## 2. Casos de Uso Concretos en Discordia

| Evento de Negocio | Productor | Routing Key | Servicios Consumidores | Efecto en los Consumidores |
| :--- | :--- | :--- | :--- | :--- |
| **Baneo de Miembro** | `Server Service` | `server.member.banned` | `Chat`, `Voice`, `Notification`, `Audit` | • `Chat`: Cierra WebSockets del usuario.<br>• `Voice`: Expulsa de canales de voz.<br>• `Notif`: Envía push notificando sanción.<br>• `Audit`: Registra entrada inmutable. |
| **Mensaje Enviado** | `Chat Service` | `chat.message.created` | `Notification & Audit` | • Detecta `@menciones` y genera notificaciones push para usuarios desconectados.<br>• Actualiza métricas de actividad. |
| **Boost Aplicado** | `Payments Service`| `payment.boost.applied` | `Server`, `Notification` | • `Server`: Sube nivel del servidor y desbloquea límites de audio/emojis.<br>• `Notif`: Alerta a los administradores del servidor. |
| **Usuario Suspendido** | `Auth Service` (Staff) | `user.account.suspended` | `Chat`, `Voice`, `Server` | • Revoca tokens y desconecta al usuario de toda la plataforma. |

---

## 3. Patrones Complementarios de Resiliencia y Consistencia

### 3.1. Transactional Outbox Pattern (Eliminación del *Dual-Write*)
**Problema:** Si el `Server Service` guarda el baneo en PostgreSQL y luego intenta publicar a RabbitMQ pero el broker está temporalmente inaccesible o la red falla, el usuario queda baneado en la BD pero nunca se le cierran los sockets en Chat ni en Voz.
**Solución:**
1. Dentro de la **misma transacción SQL**, se hace:
   ```sql
   INSERT INTO bans (user_id, server_id, reason) VALUES (...);
   INSERT INTO outbox_events (event_id, routing_key, payload, status)
   VALUES ('evt-123', 'server.member.banned', '{"user_id": ...}', 'PENDING');
   ```
2. Un worker en segundo plano (*Outbox Publisher*) lee los registros `PENDING`, los publica a RabbitMQ y marca el registro como `PUBLISHED` solo tras recibir la confirmación del broker (*Publisher Confirms*).

### 3.2. Idempotencia y Deduplicación en Consumidores
Dado que los brokers garantizan entrega *al menos una vez* (*At-least-once delivery*), un mensaje puede reentregarse ante un timeout de red.
- **Estrategia:** Cada mensaje incluye un identificador único global `event_id` (UUID v4) y un `timestamp`.
- Los consumidores almacenan en Redis o en su base de datos local los `event_id` procesados en las últimas 24 horas. Si reciben un evento repetido, envían el `ACK` a RabbitMQ inmediatamente sin repetir los efectos secundarios.

### 3.3. Manejo de Errores: Dead Letter Exchange (DLX) y Reintentos
- Si un worker falla al procesar un mensaje debido a un error transitorio, envía un `NACK` con reencolado diferido (*retry delay*).
- Si el mensaje falla reiteradamente (más de 3 reintentos) por un error permanente (payload corrupto o regla de negocio inválida), el broker lo deriva a una **Dead Letter Queue (DLQ)** para análisis del equipo sin bloquear la cola principal.

---

## 4. Excepciones: ¿Cuándo SÍ usamos llamadas síncronas (HTTP REST)?

La comunicación inter-servicio es asíncrona por defecto. Solo se admiten llamadas síncronas directas cuando el cliente **necesita el resultado de forma imperativa para continuar su flujo**:
1. **Login y Validación de Credenciales en API Gateway $
ightarrow$ Auth Service:** El cliente móvil/web espera el token JWT en la respuesta para renderizar la pantalla principal.
2. **Generación de Token de Sala en API Gateway $
ightarrow$ Voice Signaling Service:** El cliente requiere la clave criptográfica de la sala en tiempo real para iniciar la conexión WebRTC.

Cualquier otra interacción (efectos secundarios, auditoría, notificaciones, propagación de estados) es **estrictamente asíncrona**.

---

## Alternativas Descartadas

1. **Llamadas REST Sincrónicas Encadenadas:**
   - *Descartadas:* Acoplamiento temporal extremo, aumento exponencial de latencia y fragilidad ante caídas de servicios secundarios.
2. **Apache Kafka:**
   - *Pros:* Log distribuido particionado de alta retención (*Event Streaming*).
   - *Contras:* Complejidad de configuración (particiones, offset management, zookeeper/kraft) desmedida para las necesidades de mensajería y colas de Discordia, donde RabbitMQ cubre con creces el rendimiento con menor sobrecarga operativa.
3. **Redis Pub/Sub como Broker Global:**
   - *Pros:* Extremadamente rápido.
   - *Contras:* Redis Pub/Sub es de entrega efímera (*fire-and-forget*): si el servicio de notificaciones está caído cuando ocurre el evento, el mensaje se pierde para siempre (no tiene colas duraderas ni confirmaciones persistentes). Redis se reserva solo para WebSockets locales; RabbitMQ maneja la mensajería durable del backend.

---

## Consecuencias

### Positivas
- **Tolerancia a Fallos:** Si el servicio de Notificaciones o Auditoría se reinicia, los baneos, mensajes y pagos siguen funcionando sin interrupción.
- **Absorción de Picos:** Los picos masivos de mensajes o registros se amortiguan en las colas sin saturar la CPU de los consumidores.
- **Escalabilidad Horizontal Simple:** Basta con levantar réplicas de los contenedores para distribuir la carga de trabajo automáticamente (*Work Queue*).

### Negativas y Mitigaciones
- **Consistencia Eventual:** Existe una ventana de unos pocos milisegundos entre la emisión del evento y su procesamiento final; se mitiga con respuestas optimistas en el frontend y WebSocket push.
- **Infraestructura Adicional:** Requiere operar y monitorear el contenedor de RabbitMQ; se mitiga con la interfaz web de administración de RabbitMQ (`rabbitmq:3-management`) y métricas en Prometheus.
