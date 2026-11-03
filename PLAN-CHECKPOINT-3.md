# Plan del Checkpoint 3 (13/11)

## Puntos

| | Antes del Checkpoint 3 | En este checkpoint | Al cierre | Mínimo |
|---|---:|---:|---:|---:|
| Obligatorias | 96 | 8 | **104** | 104 |
| Optativas | 43 | 74 | **117** | 112 |
| **Total** | 139 | 82 | **221** | 216 |

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

| # | Historia | Épica | Pts |
| -: | --- | --- | -: |
| 35 | Autenticación de dos factores (2FA) | Usuarios | 5 |
| 36 | Visualización de perfil público | Perfil | 2 |
| 39 | Administrar configuración del servidor | Servidores | 3 |
| 40 | Eliminar servidor | Servidores | 5 |
| 42 | Enviar mensaje directo (DM) | Mensajería | 8 |
| 43 | Bloquear y desbloquear usuarios | Mensajería | 3 |
| 44 | Editar mensaje | Mensajería | 3 |
| 45 | Eliminar mensaje | Mensajería | 3 |
| 48 | Reacciones con emoji | Mensajería | 3 |
| 57 | Sobrescribir permisos de un rol en un canal específico | Roles y Permisos | 5 |
| 61 | Ver registro de auditoría de moderación | Moderación | 3 |
| 68 | Eliminar servidor desde el backoffice | Administración | 5 |
| 69 | Suspender servidor completo | Administración | 5 |
| 70 | Ver métricas de actividad de un servidor | Métricas | 5 |
| 71 | Ver métricas globales de la plataforma | Métricas | 5 |
| 72 | Gráfico de crecimiento de miembros | Métricas | 3 |
| 73 | Exportar métricas a CSV | Métricas | 3 |
| 74 | Estado de la plataforma e incidentes | Métricas | 5 |
| | | **Subtotal** | **74** |

## Infraestructura y requisitos no funcionales

| Tema | Qué falta |
|---|---|
| Pub/sub entre instancias del chat | Repartir los mensajes entre instancias con Redis Pub/Sub, como dice el ADR 0006, y una cola de RabbitMQ por instancia. Hoy corre una sola instancia y el broadcast es en memoria. Aplica también a los DM (#42) |
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
| ADR: mensajes directos | Modelo de las conversaciones y autorización de acceso (#42 y #43) |
| ADR: métricas | Qué eventos se agregan y cómo se guardan (#70–#73) |
| ADRs de Notification-Audit-Metrics y Payments | Notification solo tiene el de CI; Payments no tiene ninguno |
| Diagrama C4 de componentes | Están los niveles de contexto y contenedores; falta el de componentes |
| Resultados de la prueba de carga | Cuellos de botella encontrados y qué se cambió |
| Guía de usuario del backoffice | Opcional, se valora |
