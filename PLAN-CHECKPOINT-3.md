# Plan del Checkpoint 3 (13/11)

> Última revisión: 14/11/2026. El estado detallado de cada historia está en `progress.md` y la evolución en `BITACORA.md`. El tablero general del equipo se gestiona en [GitHub Projects #4 (Discordia Kanban)](https://github.com/orgs/discordia-g12/projects/4).

## Puntos

| | Hechos al 14/11 | En curso / pendientes | Al cierre | Mínimo |
|---|---:|---:|---:|---:|
| Obligatorias | **104** | 0 | **104** | 104 |
| Optativas | **127** | 8 | **135** | 112 |
| **Total** | **231** | 8 | **239** | 216 |

Las 32 historias obligatorias (104 pts) y 30 historias optativas (127 pts) están 100% completas en `develop` y cerradas en el proyecto. **Discordia ya supera el mínimo exigido de la cátedra de 216 pts (+15 pts de margen)**. Restan únicamente 8 pts de las historias comprometidas para CP3 que siguen abiertas en el backlog (#57 y #73).

## Historias

### Obligatorias (104 / 104 pts — 100% completas)

| # | Historia | Épica | Pts | Estado |
| -: | --- | --- | -: | --- |
| 29 | Suspender usuario a nivel plataforma | Administración | 3 | ✅ Hecha (Auth-User PR #44, Gateway PR #26, WebApp PR #76) |
| 30 | Reactivar usuario suspendido | Administración | 1 | ✅ Hecha (Auth-User PR #44, Gateway PR #26, WebApp PR #76) |
| 31 | Ver listado de usuarios con filtros | Administración | 2 | ✅ Hecha (Auth-User PR #44, Gateway PR #26, WebApp PR #76) |
| 32 | Ver listado de servidores de la plataforma | Administración | 2 | ✅ Hecha (Server-Roles PR #98, Gateway PR #29, WebApp PR #84) |
| | | **Subtotal** | **8** | *(Todas las obligatorias completadas)* |

### Optativas comprometidas para CP3 (127 pts hechos / 8 pts pendientes)

| # | Historia | Épica | Pts | Estado |
| -: | --- | --- | -: | --- |
| 35 | Autenticación de dos factores (2FA) | Usuarios | 5 | ✅ Hecha (Auth #36, WebApp #69) |
| 36 | Visualización de perfil público | Perfil | 2 | ✅ Hecha (Auth-User PR #44, WebApp) |
| 39 | Administrar configuración del servidor | Servidores | 3 | ✅ Hecha (Server-Roles PR #96, Chat, WebApp PR #80) |
| 40 | Eliminar servidor | Servidores | 5 | ✅ Hecha (Server-Roles PR #94, Chat, WebApp PR #78) |
| 42 | Enviar mensaje directo (DM) | Mensajería | 8 | ✅ Hecha (Chat, WebSockets, MongoDB, inbox Redis, WebApp) |
| 43 | Bloquear y desbloquear usuarios | Mensajería | 3 | ✅ Hecha (Auth-User, Chat, Server-Roles, WebApp) |
| 44 | Editar mensaje | Mensajería | 3 | ✅ Hecha (Chat PR #46, WebApp PR #77) |
| 45 | Eliminar mensaje | Mensajería | 3 | ✅ Hecha (Chat PR #46, WebApp PR #77) |
| 47 | Adjuntar archivos en mensajes | Mensajería | 5 | ✅ Hecha (Chat #39, Gateway #24, WebApp #70) |
| 48 | Reacciones con emoji | Mensajería | 3 | ✅ Hecha (Chat PR #46, Server-Roles, WebApp PR #77) |
| 50 | Iniciar y recibir llamada de voz privada | Canal de Voz | 13 | ✅ Hecha (Voice PR #44, Chat, WebApp PR #87, ADR 0015) |
| 57 | Sobrescribir permisos de un rol en un canal específico | Roles y Permisos | 5 | 🟡 Parcial (cálculo interno listo en Server-Roles #18; falta API y UI) |
| 61 | Ver registro de auditoría de moderación | Moderación | 3 | ✅ Hecha (Server-Roles PR #97, Notification PR #26, WebApp PR #82) |
| 68 | Eliminar servidor desde el backoffice | Administración | 5 | ✅ Hecha (Server-Roles PR #102, Chat PR #53, WebApp PR #86) |
| 69 | Suspender servidor completo | Administración | 5 | ✅ Hecha (Server-Roles PR #98, Chat PR #51, WebApp PR #84) |
| 70 | Ver métricas de actividad de un servidor | Métricas | 5 | ✅ Hecha (Notification PR #28, Chat PR #55, Server-Roles PR #103, WebApp PR #89) |
| 71 | Ver métricas globales de la plataforma | Métricas | 5 | ✅ Hecha (Notification PR #27, Chat PR #52, Server-Roles PR #101, WebApp PR #85) |
| 72 | Gráfico de crecimiento de miembros | Métricas | 3 | ✅ Hecha (Server-Roles PR #95, WebApp PR #79) |
| 73 | Exportar métricas a CSV | Métricas | 3 | ⬜ Pendiente (base lista en Notification) |
| 74 | Estado de la plataforma e incidentes | Métricas | 5 | ✅ Hecha (Notification PR #25, Gateway PR #27, WebApp PR #81) |

## Infraestructura y requisitos no funcionales

| Tema | Qué falta |
|---|---|
| Adjuntos en producción | Chat no recibe `S3_BUCKET` ni credenciales en `deploy/docker-compose.yml`, y Terraform no le da usuario IAM ni prefijo en el bucket (`local.media_writers`). Sin eso guarda los archivos en el disco del contenedor y se pierden en cada deploy. La ruta pública de Kong (Gateway #24) tiene que salir junto con Chat #39 |
| Rol `staff` | Mecanismo para otorgarlo por fuera de la interfaz (seed o flag en la base), documentado para que el corrector entre al backoffice |
| Eventos hacia métricas y auditoría | Chat publica eventos de mensajes y Notification-Audit-Metrics consume los de mensajería, membresía y moderación (base de #61 y #70–#73) |
| Cascada al eliminar o suspender un servidor | Por eventos: Chat borra o bloquea los mensajes y Voice corta las llamadas activas, con consumidores idempotentes |
| Prueba de carga | Un escenario con k6 sobre los endpoints críticos y la entrega de mensajes en tiempo real |
| Cobertura de tests | Al menos 70 % medido en cada servicio backend |
| CI que bloquee el merge | Con el plan Free de GitHub no se pueden proteger ramas en repos privados: hacer públicos los repos o documentar la alternativa |
| Estados de carga y error | En todas las pantallas nuevas, incluido el backoffice |

## Documentación

| Documento | Qué falta |
|---|---|
| Contratos OpenAPI | Chat, Voice y Notification-Audit-Metrics (Node) no generan OpenAPI. Los servicios en Python lo generan con FastAPI |
| ADR: CD por SSM y OIDC | El despliegue automático desde `main` |
| ADR: eliminación de servidor | Estrategia de consistencia de la cascada entre servicios (#40, #68 y #69) |
| ADR: mensajes directos | Modelo de las conversaciones, autorización de acceso, bloqueo y llamada privada (#42, #43 y #50) |
| ADR: adjuntos | Dónde se guardan, formatos y tamaño máximo, y cómo se sirven (#47) |
| ADR: métricas | Qué eventos se agregan y cómo se guardan (#70–#73) |
| ADRs de Notification-Audit-Metrics y Payments | Notification tiene el de CI y el del estado de la plataforma; Payments no tiene ninguno |
| Diagrama C4 de componentes | Están los niveles de contexto y contenedores; falta el de componentes |
| Resultados de la prueba de carga | Cuellos de botella encontrados y qué se cambió |
| Guía de usuario del backoffice | Opcional, se valora |
