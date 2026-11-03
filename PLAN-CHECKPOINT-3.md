# Plan del Checkpoint 3 (13/11)

> Última revisión: 03/11/2026. El estado de cada historia está en `progress.md`.

## Puntos

| | Hechos al 03/11 | Faltan en este checkpoint | Al cierre | Mínimo |
|---|---:|---:|---:|---:|
| Obligatorias | 96 | 8 | **104** | 104 |
| Optativas | 48 | 87 | **135** | 112 |
| **Total** | 144 | 95 | **239** | 216 |

El alcance deja 23 pts de margen sobre el mínimo: si algo no llega, se cae primero lo que no es de mensajes
directos ni del backoffice.

## Historias

### Obligatorias

| # | Historia | Épica | Pts |
| -: | --- | --- | -: |
| 29 | Suspender usuario a nivel plataforma | Administración | 3 |
| 30 | Reactivar usuario suspendido | Administración | 1 |
| 31 | Ver listado de usuarios con filtros | Administración | 2 |
| 32 | Ver listado de servidores de la plataforma | Administración | 2 |
| | | **Subtotal** | **8** |

### Optativas

| # | Historia | Épica | Pts | Estado |
| -: | --- | --- | -: | --- |
| 35 | Autenticación de dos factores (2FA) | Usuarios | 5 | ✅ Hecha (Auth #36, WebApp #69) |
| 36 | Visualización de perfil público | Perfil | 2 | |
| 39 | Administrar configuración del servidor | Servidores | 3 | 🟡 Falta editar el nombre |
| 40 | Eliminar servidor | Servidores | 5 | |
| 42 | Enviar mensaje directo (DM) | Mensajería | 8 | |
| 43 | Bloquear y desbloquear usuarios | Mensajería | 3 | |
| 44 | Editar mensaje | Mensajería | 3 | |
| 45 | Eliminar mensaje | Mensajería | 3 | |
| 47 | Adjuntar archivos en mensajes | Mensajería | 5 | 🟡 En revisión (Chat #39, Gateway #24, WebApp #70) |
| 48 | Reacciones con emoji | Mensajería | 3 | |
| 50 | Iniciar y recibir llamada de voz privada | Canal de Voz | 13 | |
| 57 | Sobrescribir permisos de un rol en un canal específico | Roles y Permisos | 5 | 🟡 Falta la API y la UI |
| 61 | Ver registro de auditoría de moderación | Moderación | 3 | |
| 68 | Eliminar servidor desde el backoffice | Administración | 5 | |
| 69 | Suspender servidor completo | Administración | 5 | |
| 70 | Ver métricas de actividad de un servidor | Métricas | 5 | |
| 71 | Ver métricas globales de la plataforma | Métricas | 5 | |
| 72 | Gráfico de crecimiento de miembros | Métricas | 3 | |
| 73 | Exportar métricas a CSV | Métricas | 3 | |
| 74 | Estado de la plataforma e incidentes | Métricas | 5 | 🟡 Falta la gestión de incidentes |
| | | **Subtotal pendiente** | **87** | |

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
