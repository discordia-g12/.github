# Estado de Historias de Usuario — Discordia

> Estado de cada historia de la consigna contra lo que está en `develop` de cada repo. Última revisión:
> 14/11/2026. El alcance acordado para el Checkpoint 3 está en
> [`PLAN-CHECKPOINT-3.md`](https://github.com/discordia-g12/discordia-orchestrator/blob/develop/org-docs/PLAN-CHECKPOINT-3.md)
> y la evolución semanal, en la
> [bitácora](https://github.com/discordia-g12/discordia-orchestrator/blob/develop/org-docs/BITACORA.md).

**Estado:** ✅ hecha · 🟡 parcial (no cumple todos los CA) · ⬜ pendiente. Solo suman puntos las ✅.
**CP3:** ● comprometida para el Checkpoint 3 (13/11).

## Resumen

| | Hechas | Puntos hechos | Comprometidas CP3 | Puntos al cierre del CP3 | Total |
|---|---:|---:|---:|---:|---:|
| **Obligatorias** | 32 / 32 | **104** | — | **104** | 104 |
| **Optativas** | 30 / 49 | **127** | 2 (8 pts) | **135** | 229 |
| **Total** | 62 / 81 | **231** | 2 (8 pts) | **239** | 333 |

Mínimo de la cátedra para 6 integrantes: 104 obligatorios + 112 optativos = **216 pts**. Con lo mergeado en `develop` y cerrado en GitHub Issues / Projects, **Discordia ya supera el mínimo exigido (+15 pts de margen)**. Restan únicamente 8 pts de las historias comprometidas para CP3 que siguen abiertas en el backlog (#57 y #73).

## 1. Historias obligatorias (104 / 104)

| # | Estado | CP3 | Historia | Épica | Pts | Evidencia / notas |
| -: | :-: | :-: | --- | --- | -: | --- |
| 1 | ✅ | | Registro de usuarios | Usuarios | 3 | Auth0 + `Auth-User-Service`, WebApp #2, #4. E2E `auth.spec.ts` |
| 2 | ✅ | | Login con email y contraseña | Usuarios | 2 | Kong (`auth0_validator.lua`), login web y mobile |
| 3 | ✅ | | Recupero de contraseña | Usuarios | 3 | `Auth-User-Service` `/password-reset`, rate limiting en Kong, WebApp #20 |
| 4 | ✅ | | Edición de perfil | Perfil | 2 | `PATCH /profile` y `/profile/avatar` (S3), WebApp #16, #39 |
| 5 | ✅ | | Visualización de perfil propio | Perfil | 1 | `GET /users/me`, WebApp #15 |
| 6 | ✅ | | Crear servidor | Servidores | 3 | `POST /servers`, ícono en S3. E2E `fullstack.spec.ts` |
| 7 | ✅ | | Generar invitación | Servidores | 5 | `POST /servers/{id}/invites`, WebApp #19. E2E `invites.spec.ts` |
| 8 | ✅ | | Unirse a servidor vía invitación | Servidores | 3 | Vista previa (Server-Roles #54) y `POST /invites/{code}/join`, WebApp #19 |
| 9 | ✅ | | Abandonar servidor | Servidores | 2 | Server-Roles #6, WebApp #18 |
| 10 | ✅ | | Transferir propiedad del servidor | Servidores | 3 | Server-Roles #66, WebApp #44 |
| 11 | ✅ | | Crear canal | Canales | 3 | `POST /servers/{id}/channels`. E2E `channels.spec.ts` |
| 12 | ✅ | | Editar canal | Canales | 2 | Server-Roles #33 |
| 13 | ✅ | | Eliminar canal | Canales | 2 | Server-Roles #30 |
| 14 | ✅ | | Organizar canales en categorías | Canales | 3 | `POST /servers/{id}/categories`. E2E `channels.spec.ts` |
| 15 | ✅ | | Enviar mensaje en un canal | Mensajería | 8 | Chat (WebSocket + MongoDB + RabbitMQ). E2E `chat.spec.ts`. Pub/sub entre instancias con Redis (Chat #36) |
| 16 | ✅ | | Historial y reconexión | Mensajería | 13 | `GET /channels/{id}/messages`, WebApp #29 y #43 (paginación) |
| 17 | ✅ | | Unirse a canal de voz de servidor | Canal de Voz | 11 | LiveKit + Voice (`/join-token`), WebApp #21, #26 |
| 18 | ✅ | | Salir de canal de voz | Canal de Voz | 1 | Voice `/leave` y webhooks de LiveKit (#18) |
| 19 | ✅ | | Silenciar micrófono propio | Canal de Voz | 1 | Web y mobile |
| 20 | ✅ | | Ver participantes conectados | Canal de Voz | 3 | Voice `/ws/voice`, WebApp #23, #27 |
| 21 | ✅ | | Crear rol | Roles y Permisos | 3 | Server-Roles #12, WebApp #8, #9 |
| 22 | ✅ | | Editar permisos de un rol | Roles y Permisos | 5 | Server-Roles #13, WebApp #10, #11, #12 |
| 23 | ✅ | | Asignar rol a un miembro | Roles y Permisos | 3 | Server-Roles #14, WebApp #16 |
| 24 | ✅ | | Definir rol por defecto para nuevos miembros | Roles y Permisos | 2 | Server-Roles #37; se asigna al unirse por invitación |
| 25 | ✅ | | Banear miembro del servidor | Moderación | 3 | Server-Roles #10, WebApp #31, #42 |
| 26 | ✅ | | Revocar baneo | Moderación | 2 | Server-Roles #20, WebApp #31 |
| 27 | ✅ | | Ver notificaciones no leídas | Notificaciones | 3 | Chat #32 (`read_states`, ADR-02), Server-Roles #82, WebApp #60 |
| 28 | ✅ | | Marcar notificaciones como leídas | Notificaciones | 1 | Ídem |
| 29 | ✅ | | Suspender usuario a nivel plataforma | Administración | 3 | Auth-User PR #44 (`ADR-002-Suspension-Usuarios`), Gateway PR #26, desconexión en Chat PR #45 y Voice PR #39, WebApp PR #76 (`users.suspend` en `ready`) |
| 30 | ✅ | | Reactivar usuario suspendido | Administración | 1 | Auth-User PR #44 (`DELETE /suspension`), Gateway PR #26, WebApp PR #76 (`users.reactivate` en `ready`) |
| 31 | ✅ | | Ver listado de usuarios con filtros | Administración | 2 | Auth-User PR #44, Gateway PR #26, WebApp PR #76 (`users.list` en `ready`) |
| 32 | ✅ | | Ver listado de servidores de la plataforma | Administración | 2 | Server-Roles PR #98 (`GET /api/v1/admin/servers`), Gateway PR #29, WebApp PR #84 (`servers.list` en `ready`) |

Todas las historias obligatorias están completas en `develop`, incluidas las cuatro del backoffice
(`Discordia-WebApp/apps/backoffice`, habilitadas en `src/features.ts`).

## 2. Historias optativas (127 / 229)

| # | Estado | CP3 | Historia | Épica | Pts | Evidencia / notas |
| -: | :-: | :-: | --- | --- | -: | --- |
| 33 | ✅ | | Login con proveedor federado | Usuarios | 5 | Google en web y mobile, vinculación por email verificado. Auth #34, WebApp #66 |
| 34 | ⬜ | | Registro con PIN | Usuarios | 2 | |
| 35 | ✅ | | Autenticación de dos factores (2FA) | Usuarios | 5 | TOTP con QR y códigos de recuperación, en web y mobile. Auth #36, WebApp #69 |
| 36 | ✅ | | Visualización de perfil público | Perfil | 2 | `GET /users/{id}/profile` en Auth-User (PR #44), soporte en WebApp (web y mobile, issue Auth-User #43 cerrado) |
| 37 | ⬜ | | Estado de actividad | Perfil | 5 | |
| 38 | ✅ | | Estado personalizado | Perfil | 2 | `custom_status` (64 caracteres) en Auth |
| 39 | ✅ | | Administrar configuración del servidor | Servidores | 3 | `PATCH /servers/{id}` en Server-Roles (PR #96), evento `SERVER_UPDATED` en Chat y WebApp (issue Server-Roles #88 cerrado) |
| 40 | ✅ | | Eliminar servidor | Servidores | 5 | `DELETE /servers/{id}` en Server-Roles (PR #94), cascada en Postgres y S3, evento en RabbitMQ para Chat (limpieza de canales y mensajes). Modal de confirmación en web y mobile con redirección (WebApp PR #78) |
| 41 | ✅ | | Reordenar canales | Canales | 2 | `PUT /servers/{id}/channels/order` (Server-Roles #9, PR #40) |
| 42 | ✅ | | Enviar mensaje directo (DM) | Mensajería | 8 | Chat (WebSockets, MongoDB, inbox en Redis, issue Chat #41 cerrado) y WebApp (web y mobile) |
| 43 | ✅ | | Bloquear y desbloquear usuarios | Mensajería | 3 | Auth-User (`/blocks`), Chat y Server-Roles (issue Server-Roles #87 cerrado) |
| 44 | ✅ | | Editar mensaje | Mensajería | 3 | Chat PR #46 (canales y DMs, sync al inbox) y WebApp PR #77 (web y mobile) |
| 45 | ✅ | | Eliminar mensaje | Mensajería | 3 | Chat PR #46 (tombstone y chequeo de permisos, `can_manage_messages` en Server-Roles) y WebApp PR #77 (confirmación y tombstone en web y mobile) |
| 46 | ✅ | | Menciones | Mensajería | 5 | Usuario, rol y `@everyone`; cuenta en no leídos. Chat #34 (ADR-03), Server-Roles #83, WebApp #61 |
| 47 | ✅ | | Adjuntar archivos en mensajes | Mensajería | 5 | Subida multipart local/S3, ruta pública Kong, preview y renderizado de imágenes, audio, video y docs en web y mobile. Chat #39, Gateway #24, WebApp #70 |
| 48 | ✅ | | Reacciones con emoji | Mensajería | 3 | Chat PR #46 (toggle, permisos y broadcast en tiempo real), `can_add_reactions` en Server-Roles y WebApp PR #77 (web y mobile, `ADR-04-Reacciones-y-Catalogo-Emojis`) |
| 49 | ⬜ | | Buscar mensajes | Mensajería | 5 | |
| 50 | ✅ | | Iniciar y recibir llamada de voz privada | Canal de Voz | 13 | Voice PR #44 (sala 1 a 1 en LiveKit, ADR 0015, issue Voice #41 cerrado), Chat (autorización y eventos inbox) y WebApp PR #87 (audio y cámara) |
| 51 | ✅ | | Indicador de quién está hablando | Canal de Voz | 3 | Eventos de audio de LiveKit, WebApp #22 |
| 52 | ✅ | | Silenciar a otro participante localmente | Canal de Voz | 2 | WebApp #24 (web) y #28 (mobile) |
| 53 | ⬜ | | Compartir pantalla | Canal de Voz | 8 | Ya existe el permiso `SCREEN_SHARE` en Server-Roles y `TrackSource.SCREEN_SHARE` en el token de LiveKit en Voice; falta implementación en WebApp |
| 54 | ✅ | | Video en llamada | Canal de Voz | 8 | Voice #17, WebApp #38 (web y mobile). Llamadas privadas (#50): el token de la llamada deja publicar la cámara y el panel de la llamada muestra el video (Voice PR #44, WebApp PR #87) |
| 55 | ✅ | | Eliminar rol | Roles y Permisos | 3 | Server-Roles PR #32, WebApp #8, #9 |
| 56 | ✅ | | Reordenar jerarquía de roles | Roles y Permisos | 3 | `PUT /servers/{id}/roles/order`, Server-Roles #85, WebApp #65 |
| 57 | 🟡 | ● | Sobrescribir permisos de un rol en un canal específico | Roles y Permisos | 5 | Cálculo interno listo en Server-Roles (`/internal/channels/{id}/access`), issue Server-Roles #18 sigue abierto |
| 58 | ✅ | | Ver miembros por rol | Roles y Permisos | 2 | Server-Roles PR #39, WebApp #14 |
| 59 | ✅ | | Expulsar miembro del servidor | Moderación | 3 | Server-Roles `DELETE /members/{id}`, Voice #24, WebApp #42 |
| 60 | ✅ | | Silenciar temporalmente a un miembro | Moderación | 5 | Server-Roles `/mutes`, chequeo en Chat (#17, #19) y Voice, WebApp #42 |
| 61 | ✅ | | Ver registro de auditoría de moderación | Moderación | 3 | Server-Roles PR #97 (`VIEW_AUDIT_LOG` bit 23 y outbox enriquecido), Chat (`message.deleted_by_moderator`), Notification PR #26 (`ADR-04`), Gateway PR #28 y WebApp PR #82 (visor con filtro y diff en web y mobile) |
| 62 | ⬜ | | Reportar mensaje o usuario | Moderación | 5 | |
| 63 | ⬜ | | Filtro automático de palabras prohibidas | Moderación | 3 | |
| 64 | ⬜ | | Silenciar notificaciones de un canal o servidor | Notificaciones | 3 | |
| 65 | ⬜ | | Configurar preferencias de notificación | Notificaciones | 3 | |
| 66 | ⬜ | | Notificaciones push | Notificaciones | 5 | Depende de #65 |
| 67 | ⬜ | | Centro de notificaciones con historial | Notificaciones | 5 | |
| 68 | ✅ | | Eliminar servidor desde el backoffice | Administración | 5 | Server-Roles PR #102, Chat PR #53, Voice PR #42, Gateway PR #31 y WebApp PR #86 (`servers.delete` en `ready` + cierre/redirección en vivo en web y mobile) |
| 69 | ✅ | | Suspender servidor completo | Administración | 5 | Server-Roles PR #98 (`ADR-002-Suspension-de-Servidores`), Chat PR #51, Voice PR #40, Gateway PR #29 y WebApp PR #84 (`servers.suspend` en `ready` + bloqueo en web y mobile) |
| 70 | ✅ | | Ver métricas de actividad de un servidor | Métricas | 5 | `GET /api/v1/servers/:serverId/metrics` en Notification PR #28 (con `/internal/metrics/messages` en Chat PR #55 e `/internal/metrics/servers/{id}` en Server-Roles PR #103), Gateway PR #32, orquestador PR #36 y WebApp PR #89 (web y mobile) |
| 71 | ✅ | | Ver métricas globales de la plataforma | Métricas | 5 | `GET /api/v1/admin/metrics?from&to` (staff) en Notification (PR #27), que junta los conteos de Chat (PR #52) y de Server-Roles (PR #101); ruta `admin-metrics` en Kong (PR #30); pantalla en el backoffice (WebApp PR #85). [ADR 0014](docs/architecture/decisions/0014-metricas-globales-de-la-plataforma.md) |
| 72 | ✅ | | Gráfico de crecimiento de miembros | Métricas | 3 | Server-Roles: `ServerMemberEvent`, migración Alembic y endpoint `GET /servers/{id}/metrics/member-growth` con permisos de admin. Web y mobile: tarjetas de métricas, selector 7d/14d/30d, gráfico SVG y tooltip día a día (WebApp PR #79). [ADR 0012](docs/architecture/decisions/0012-metricas-crecimiento-miembros.md) |
| 73 | ⬜ | ● | Exportar métricas a CSV | Métricas | 3 | Depende de #70 o #71 |
| 74 | ✅ | | Estado de la plataforma e incidentes | Métricas | 5 | Publicación, resolución y sugerencia automática de incidentes en Notification (PR #25, ADR-03), Gateway (PR #27), backoffice y banner/modal web (WebApp PR #81, issue observabilidad #11 cerrado) |
| 75 | ⬜ | | Gestionar emojis personalizados del servidor | Monetización | 5 | |
| 76 | ⬜ | | Suscribirse a Plan Premium | Monetización | 13 | Payments es solo el esqueleto, con CI/CD |
| 77 | ⬜ | | Cancelar suscripción Premium | Monetización | 3 | Depende de #76 |
| 78 | ⬜ | | Comprar puntos | Monetización | 8 | |
| 79 | ⬜ | | Usar puntos para boostear un servidor | Monetización | 13 | Depende de #78 |
| 80 | ⬜ | | Ver beneficios de servidor boosteado | Monetización | 3 | Depende de #79 y #75 |
| 81 | ⬜ | | Ver expediente de investigación de usuario o servidor | Administración | 5 | Depende de #62 (`cases.user` y `cases.server` siguen en `soon`) |

### Optativas por épica

| Épica | Hechas | Puntos hechos / disponibles |
|---|---:|---:|
| Usuarios | 2 / 3 | 10 / 12 |
| Perfil | 2 / 3 | 4 / 9 |
| Servidores | 2 / 2 | 8 / 8 |
| Canales | 1 / 1 | 2 / 2 |
| Mensajería | 7 / 8 | 30 / 35 |
| Canal de Voz | 4 / 5 | 26 / 34 |
| Roles y Permisos | 3 / 4 | 8 / 13 |
| Moderación | 3 / 5 | 11 / 19 |
| Notificaciones | 0 / 4 | 0 / 16 |
| Administración | 2 / 3 | 10 / 15 |
| Métricas | 4 / 5 | 18 / 21 |
| Monetización | 0 / 6 | 0 / 45 |
