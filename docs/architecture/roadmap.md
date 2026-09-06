# Roadmap de Desarrollo y Planificación - Proyecto Discordia (6 Integrantes)

## 1. Distribución de Puntos y Alcance

Para un equipo de **6 integrantes**, los requisitos de entrega de la cátedra son:
- **Historias Obligatorias:** 32 historias = **104 puntos** (100% obligatorio).
- **Historias Optativas Mínimas:** **112 puntos** (48.9% del pool optativo).
- **Puntaje Total Mínimo a Entregar:** **216 puntos**.
- **Superficies Obligatorias:** Aplicación Mobile (React Native), Cliente Web (React) y Backoffice de Staff (React).

---

## 2. Selección de Historias Optativas (112 Puntos Seleccionados)

Se seleccionó un conjunto sinérgico y coherente con la arquitectura diseñada, priorizando valor de usuario, consistencia técnica y completitud de las épicas:

| ID | Historia Optativa | Épica | Puntos | Justificación Técnica y Dependencias |
| :--- | :--- | :--- | :---: | :--- |
| **HU 33** | Login con proveedor federado (Google OAuth) | Usuarios | 5 | Estándar de la industria, reutiliza JWT del Auth Service. |
| **HU 35** | Autenticación de dos factores (2FA / TOTP) | Usuarios | 5 | Refuerzo de seguridad exigido en RNF. |
| **HU 36** | Visualización de perfil público | Perfil | 2 | Complemento natural del perfil propio. |
| **HU 37** | Estado de actividad | Perfil | 5 | Apalanca el bus de presencia en Redis. |
| **HU 39** | Administrar configuración del servidor | Servidores | 3 | Edición de nombre, icono, descripción. |
| **HU 40** | Eliminar servidor | Servidores | 5 | Cascada distribuida de eliminación mediante eventos. |
| **HU 42** | Enviar mensaje directo (DM) | Mensajería | 8 | Reutiliza el motor de WebSockets y colecciones MongoDB. |
| **HU 43** | Bloquear y desbloquear usuarios | Mensajería | 3 | Regla de autorización para DMs y visibilidad. |
| **HU 44** | Editar mensaje | Mensajería | 3 | Evento WebSocket de actualización en vivo. |
| **HU 45** | Eliminar mensaje | Mensajería | 3 | Evento WebSocket de borrado en vivo y consistencia. |
| **HU 46** | Menciones (@usuario, @rol, @everyone) | Mensajería | 5 | Dispara eventos de notificación dirigida. |
| **HU 47** | Adjuntar archivos en mensajes | Mensajería | 5 | Integración con Object Storage (S3/Cloudinary). |
| **HU 48** | Reacciones con emoji | Mensajería | 3 | Evento en tiempo real de bajo payload. |
| **HU 49** | Buscar mensajes | Mensajería | 5 | Índices de texto en MongoDB por canal/DM. |
| **HU 51** | Indicador de quién está hablando | Canal de Voz | 3 | Evento de audio level desde la SFU / WebSockets. |
| **HU 52** | Silenciar a otro participante localmente | Canal de Voz | 2 | Control de audio en cliente (WebRTC track mute). |
| **HU 59** | Expulsar miembro del servidor | Moderación | 3 | Acción de moderación sin baneo permanente. |
| **HU 60** | Silenciar temporalmente a un miembro | Moderación | 5 | Restricción temporal de envío de mensajes y voz. |
| **HU 61** | Ver registro de auditoría de moderación | Moderación | 3 | Persistencia en Notification & Audit Service. |
| **HU 63** | Filtro automático de palabras prohibidas | Moderación | 3 | Interceptor en Chat Engine antes de persistir/emitir. |
| **HU 64** | Silenciar notificaciones de canal/servidor | Notificaciones| 3 | Matriz de preferencias por usuario en BD. |
| **HU 65** | Configurar preferencias de notificación | Notificaciones| 3 | Ajustes de alertas push e in-app. |
| **HU 66** | Notificaciones push (Firebase FCM) | Notificaciones| 5 | Integración móvil nativa en React Native. |
| **HU 70** | Ver métricas de actividad de un servidor | Métricas | 5 | Visualización gráfica de actividad en servidor. |
| **HU 71** | Ver métricas globales de la plataforma | Métricas | 5 | Dashboard en Backoffice para staff. |
| **HU 74** | Estado de la plataforma e incidentes | Métricas | 5 | Integración del Backoffice con health checks /readyz. |
| **HU 76** | Suscribirse a Plan Premium | Monetización | 13 | Saga Orquestada con pasarela de pago (Stripe/MP). |
| **HU 77** | Cancelar suscripción Premium | Monetización | 3 | Gestión de ciclo de vida de suscripciones recurrentes. |
| **HU 78** | Comprar puntos | Monetización | 8 | Acreditación atómica de saldo mediante Webhooks y Saga. |
| **HU 79** | Usar puntos para boostear servidor | Monetización | 13 | Descuento de saldo y escalado de nivel de servidor. |
| **HU 80** | Ver beneficios de servidor boosteado | Monetización | 3 | Activación de badges y límites ampliados. |
| **TOTAL** | **31 Historias Optativas Seleccionadas** | | **112 pts** | **Cumple exactamente el 100% de la cuota de 6 integrantes.** |

---

## 3. Plan de Checkpoints y Asignación de Roles en el Equipo

### 3.1. Roles y Responsabilidades Iniciales (Rotativos entre Checkpoints)
Para cumplir la directiva de rotación y evitar silos de conocimiento (*"que nadie sea el único que puede explicar su servicio"*):
- **Pareja 1 (Integrantes A y B):** Auth & User Service + Server & Roles Core Service + Seguridad (JWT, 2FA, RBAC).
- **Pareja 2 (Integrantes C y D):** Chat Engine Service (WebSockets, MongoDB, Redis) + Notification & Audit Service + Firebase FCM.
- **Pareja 3 (Integrantes E y F):** Voice Signaling Service (WebRTC, LiveKit SFU) + Payments & Monetization (Sagas, Stripe/MP) + DevOps / API Gateway.
- **Cross-functional:** Cada pareja es responsable del frontend correspondiente en **Web**, **Mobile** y **Backoffice** para sus historias, asegurando la entrega vertical *end-to-end*.

---

### 3.2. Cronograma de Checkpoints

```
[CP 0: 04/09] ───► [CP 1: 25/09] ───► [CP 2: 29/10] ───► [CP 3: 13/11] ───► [ENTREGA FINAL: 04/12]
Bootstrap E2E      Core Features      Moderación & Staff   Voz & Monetización  Polish, Resiliencia
Infra & Compose    UX Prototypes      Notificaciones       Pruebas Carga k6    Auditoría y Docs
1er ADR (Stack)    70% Test Cov       Integración          Mutation Testing    Defensa Grupal
```

#### Checkpoint 0: Bootstrap y Verificación de Entorno (Viernes 04/09)
- **Objetivo:** Poner en marcha el esqueleto técnico, CI/CD, entorno local reproducible con Docker Compose y verificar el flujo end-to-end mínimo.
- **Entregables:**
  - Repositorios configurados con Dockerfiles, linter y pipelines de GitHub Actions.
  - API Gateway configurado enrutando a los servicios iniciales.
  - Endpoints `/livez` y `/readyz` funcionando en cada servicio.
  - Primeros ADRs (ADR 01 a ADR 05) formalizados.
  - Flujo E2E mínimo funcionando: Registro/Login de usuario -> Creación de Servidor -> Creación de Canal de Texto -> Envío y recepción de un mensaje de texto.

#### Checkpoint 1: Core de Comunidad y Mensajería (Viernes 25/09)
- **Objetivo:** Consolidar la experiencia de usuario y el núcleo funcional de la plataforma.
- **Alcance Funcional (Historias Obligatorias y Optativas Base):**
  - Prototipos UX completos en Figma (Mobile, Web y Backoffice) con Sistema de Diseño unificado.
  - Épicas: Usuarios (HU 01 a HU 04, HU 33 OAuth, HU 35 2FA), Perfil (HU 05 a HU 07, HU 36, HU 37), Servidores (HU 08 a HU 11, HU 39), Canales (HU 12 a HU 14), Mensajería Obligatoria (HU 15 WebSocket en tiempo real, HU 16 Historial y reconexión), Roles básicos (HU 21 a HU 24).
  - OpenAPI contracts documentados y tests de contrato.
  - Cobertura de tests unitarios >= 70%.

#### Checkpoint 2: Moderación, Notificaciones y Backoffice de Staff (Jueves 29/10)
- **Objetivo:** Incorporar herramientas de administración global, moderación reactiva y notificaciones.
- **Alcance Funcional:**
  - Épica Moderación: HU 25 (Banear), HU 26 (Revocar baneo), HU 59 (Expulsar), HU 60 (Silenciar), HU 61 (Auditoría), HU 63 (Filtro de palabras).
  - Épica Notificaciones: HU 27, HU 28 (Notificaciones in-app obligatorias), HU 64, HU 65 (Preferencias), HU 66 (Push FCM).
  - Épica Mensajería Optativas: HU 42 (DMs), HU 43 (Bloquear), HU 44 (Editar), HU 45 (Eliminar), HU 46 (Menciones), HU 47 (Adjuntos S3), HU 48 (Reacciones), HU 49 (Búsqueda).
  - Épica Administración (Backoffice): HU 29 (Suspender usuario), HU 30 (Reactivar), HU 31 (Listar usuarios con filtros), HU 32 (Listar servidores), HU 74 (Estado e incidentes).
  - Pruebas de integración automatizadas inter-servicios.

#### Checkpoint 3: Canal de Voz, Monetización y Pruebas de Carga (Viernes 13/11)
- **Objetivo:** Integrar las piezas de mayor riesgo técnico (Voz WebRTC SFU y Monetización con pasarelas de pago) y validar el rendimiento bajo estrés.
- **Alcance Funcional y Técnico:**
  - Épica Canal de Voz: HU 17 (Unirse a canal de voz), HU 18 (Salir), HU 19 (Silenciar mic), HU 20 (Participantes conectados), HU 51 (Indicador de voz), HU 52 (Mute local).
  - Épica Monetización: HU 76 (Plan Premium), HU 77 (Cancelar), HU 78 (Comprar puntos), HU 79 (Boost de servidor), HU 80 (Beneficios boost).
  - Épica Métricas: HU 70 (Métricas de servidor), HU 71 (Métricas globales).
  - Pruebas de carga y estrés con **k6 / Artillery** simulando concurrencia masiva en HTTP y WebSockets.
  - Análisis de **Mutation Testing** (mutmut / Stryker) sobre servicios críticos.

#### Entrega Final: Estabilidad, Resiliencia y Cierre (Viernes 04/12)
- **Objetivo:** Pulido integral, verificación de todos los RNF, documentación final y preparación de la defensa.
- **Entregables:**
  - 100% de historias obligatorias (104 pts) + 112 pts de optativas = **216 pts totales validados**.
  - Documentación completa: READMEs con setup local, Diagramas C4 actualizados, catálogo completo de ADRs.
  - Guías de usuario: Aplicación Mobile y Backoffice de Staff.
  - Bitácora de evolución semanal del backlog y Retrospectiva final de ingeniería del equipo.
