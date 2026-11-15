# Bitácora del proyecto

> Evolución semanal del backlog y de lo que se entregó. Se arma con los issues de los diez repos de
> `discordia-g12` (el backlog), los PRs mergeados y los commits. Semanas de lunes a domingo.
> Última actualización: 14/11/2026.

## Cómo se lee

- **Backlog:** issues creados (+) y cerrados (−) en la semana, y cuántos quedaban abiertos el domingo.
- **Entregado:** lo que llegó a `develop` o a `main` (PRs mergeados).
- **Decisiones:** cambios de rumbo, con el documento donde quedaron.
- Las historias se nombran con su número de la consigna: **[O#N]** obligatoria, **[Op#N]** optativa.

## Resumen

| Semana | Hito | Issues + / − | Abiertos al cierre | PRs | Commits |
|---|---|---:|---:|---:|---:|
| 31/08 – 06/09 | Checkpoint 0 (04/09) | 43 / 0 | 43 | 0 | 10 |
| 07/09 – 13/09 | | 22 / 24 | 41 | 11 | 50 |
| 14/09 – 20/09 | | 6 / 15 | 32 | 6 | 43 |
| 21/09 – 27/09 | Checkpoint 1 (25/09) | 10 / 15 | 27 | 26 | 90 |
| 28/09 – 04/10 | | 4 / 2 | 29 | 7 | 34 |
| 05/10 – 11/10 | | 3 / 7 | 25 | 6 | 17 |
| 12/10 – 18/10 | | 2 / 2 | 25 | 15 | 37 |
| 19/10 – 25/10 | Checkpoint 2 (28/10) | 4 / 7 | 22 | 9 | 47 |
| 26/10 – 01/11 | | 2 / 10 | 14 | 51 | 107 |
| 02/11 – 08/11 | | 4 / 6 | 12 | 24 | 32 |
| 09/11 – 15/11 | Checkpoint 3 (13/11, en curso) | 2 / 2 | 10 | 14 | 28 |

Commits sin merges ni los de Dependabot, de todas las ramas de los diez repos. Los PRs incluyen los de
release (`develop → main`).

---

## Semana 1: 31/08 – 06/09 (Checkpoint 0)

**Backlog:** +43. Se cargó entero el 06/09:
- las historias obligatorias, cada una en el repo del servicio que la resuelve;
- las tareas de base: estructura de directorios, Docker Compose, CI/CD, migraciones y prototipo.

**Entregado:** la organización, los repos de cada servicio y el orquestador con submódulos. Arquitectura
inicial y ADRs 0001 a 0009 en `org-docs` (microservicios, stack, una base por servicio, outbox, gateway,
WebSockets, topología de voz, sagas, frontend y mobile).

**Decisiones:** se separa el sistema en siete servicios, con lenguajes distintos (Python y Node) y dos tipos
de base (Postgres y MongoDB). Ver los ADRs 0001 a 0004.

## Semana 2: 07/09 – 13/09

**Backlog:** +22 / −24, quedan 41 abiertos.
- Se suman tareas por servicio: Dockerfile, CI y Alembic.
- Se suman los prototipos de web, mobile y backoffice.
- Se suma Redis Pub/Sub para Chat (Chat #7).

**Entregado:**
- Docker Compose de todo el sistema (orquestador #6), y Dockerfile y CI en cada servicio.
- Alembic en Auth y Server-Roles.
- Server-Roles: crear servidor [O#6]; crear, editar y eliminar canal [O#11–13]; crear y eliminar rol
  [O#21, Op#55].
- Chat: enviar mensaje por WebSocket [O#15] e historial con recuperación de mensajes perdidos [O#16].
- Prototipos de web y mobile en Figma.

## Semana 3: 14/09 – 20/09

**Backlog:** +6 / −15, quedan 32 abiertos.

**Entregado:**
- Server-Roles: editar permisos de un rol [O#22], asignar rol [O#23], ver miembros por rol [Op#58] y rol por
  defecto [O#24]. CI con ruff, bandit (SAST) y umbral de cobertura (#41).
- Integración con Auth0 y endpoint interno de permisos, que Chat usa para validar el envío.
- Registro [O#1] y perfil propio [O#5] en Auth.
- WebApp: el prototipo de Figma implementado, y el registro y el login en web y mobile.

## Semana 4: 21/09 – 27/09 (Checkpoint 1)

**Backlog:** +10 / −15, quedan 27 abiertos. Es la semana de más actividad (90 commits).

**Entregado:**
- Gateway (Kong): rutas, validación del token y rate limiting en el recupero de contraseña [O#3].
- Auth: login [O#2], edición de perfil [O#4], verificación de email y recupero de contraseña.
- Server-Roles: generar invitación [O#7], vista previa de la invitación, unirse por invitación [O#8],
  abandonar servidor [O#9], listar roles y miembros y editar nombre y color de un rol. Endpoint de permisos
  para la voz (`can-connect`).
- WebApp: roles en web y mobile (crear, borrar, permisos, asignar, rol por defecto), abandonar servidor y
  unirse por invitación.
- LiveKit configurado en el orquestador y en Voice.

## Semana 5: 28/09 – 04/10

**Backlog:** +4 / −2, quedan 29 abiertos. Se suman optativas de voz: video [Op#54], indicador de quién habla
[Op#51] y silenciar a otro localmente [Op#52].

**Entregado:**
- Voz de punta a punta en la web: unirse con LiveKit [O#17], ver participantes [O#20] y silenciar a otro
  participante [Op#52]. Voice suma webhooks de LiveKit, presencia por WebSocket, y timeouts y reintentos en
  `can-connect`.
- Las bases relacionales pasan de contenedores a **Amazon RDS** (orquestador #12).
- Primer release `develop → main` de Gateway, Auth, Chat y Voice.

## Semana 6: 05/10 – 11/10

**Backlog:** +3 / −7, quedan 25 abiertos.
- Se cierran las historias de voz [O#17–20] y el indicador de quién habla [Op#51].
- Se suman la distribución en tres EC2 con Terraform y la observabilidad.

**Entregado:**
- Voz en mobile: unirse, participantes y silenciar con una pulsación larga.
- Server-Roles: categorías [O#14].

## Semana 7: 12/10 – 18/10

**Backlog:** +2 / −2, quedan 25 abiertos. Se cierran reordenar canales [Op#41] y la integración de Chat con
RabbitMQ.

**Entregado:**
- **Arquitectura de tres nodos** (orquestador #15): edge, realtime e infra, con un compose por nodo,
  Terraform y observabilidad (Prometheus, Grafana, Node Exporter y cAdvisor).
- Kong con upstreams por nodo (Gateway #9).
- Cada servicio publica su imagen en GHCR desde el CI.
- Server-Roles: banear con chequeo de jerarquía [O#25] y transferir la propiedad (backend) [O#10].
- Chat: heartbeat del WebSocket y consumidor de RabbitMQ resiliente. Voice: heartbeat.
- WebApp: reconexión del WebSocket del chat, y primer release a Cloudflare Pages (#30).

**Decisiones:** una sola VPC con un Security Group por nodo, y todas las bases en el RDS (16/10). Ver
`docs/ARQUITECTURA.md`.

## Semana 8: 19/10 – 25/10 (Checkpoint 2)

**Backlog:** +4 / −7, quedan 22 abiertos.
- Se cierran la migración a tres EC2, banear y revocar baneo [O#25, O#26], y la observabilidad.
- Se cierran también el rediseño del frontend y el CI de la WebApp.
- Se abre "Automatizar CD" (orquestador #18).

**Entregado:**
- Imágenes en **S3**: avatares e íconos de servidor (Auth #20, Server-Roles #70, orquestador #19).
- Revocar baneo [O#26], en el backend y la web.
- **Rediseño del frontend** (WebApp #35): web y mobile con la misma lógica (`@discordia/core`) y el mismo
  sistema de diseño. Expo SDK 58.
- CI de la WebApp: typecheck, lint, tests y builds de web, mobile y Docker, con Node 22.
- Orquestador: el submódulo `.github` pasa a `org-docs/` y `main` se alinea con `develop` (#20 y #21).

**Decisiones:**
- El deploy automático va solo desde `main`, por SSM y OIDC. El workflow reutilizable vive en el orquestador.
  Ver `docs/ARQUITECTURA.md`, sección 11.
- La deuda técnica se lleva en `docs/DEUDA-TECNICA.md`.

## Semana 9: 26/10 – 01/11

**Backlog:** +2 / −10, quedan 14 abiertos.
- Se cierran "Automatizar CD", transferir propiedad [O#10], video en llamada [Op#54], el CI de Payments, S3,
  el prototipo del backoffice y la documentación (orquestador #25).
- Se abre y se cierra menciones [Op#46] (Chat #33), y se cierran los no leídos [O#27, O#28] (Notification
  #1 y #2, resueltos en Chat).

**Entregado:**
- **Despliegue continuo** (orquestador #22 a #24):
  - Infraestructura: proveedor OIDC de GitHub y rol `discordia-cd` en Terraform.
  - Scripts del nodo: `deploy/deploy-service.sh`, con healthcheck y rollback, y
    `deploy/deploy-config.sh`.
  - Workflows `deploy.yml` (reutilizable) y `deploy-config.yml`.
  - Job `deploy` en el CI de los siete servicios.
- Primer deploy automático el 26/10 (Payments, en infra). El 27/10, releases a `main` de Gateway, Auth,
  Server-Roles, Chat, Notification y Voice, todos desplegados solos y sanos. El primer deploy de la
  configuración también salió bien.
- Payments: job de tests en el CI y dependencias fijadas. Dependabot en los repos; `npm audit` en Voice y
  bandit en Payments.
- Server-Roles: cambiar y quitar el ícono del servidor; permisos de voz por rol (VIDEO, SCREEN_SHARE) y los
  permisos del usuario en la respuesta del servidor.
- Voice: el token de LiveKit limita qué puede publicar cada usuario; se saca de la llamada a quien banean o
  expulsan.
- Chat: el permiso de escritura se actualiza en vivo al cambiar los roles; el silenciado puede leer sin
  escribir.
- WebApp:
  - cámara en llamadas [Op#54];
  - subida de avatar e ícono;
  - avisos de silencio, baneo y expulsión;
  - paginación del historial ("cargar mensajes anteriores") [O#16] (#43);
  - transferir la propiedad del servidor [O#10] (#44).
- Documentación: `docs/ARQUITECTURA.md` (sección 11, Despliegue) y `deploy/README.md` actualizados al CD
  (28/10).
- **Mensajes no leídos** por canal y servidor [O#27, O#28] (31/10): Chat guarda el estado de lectura
  (`read_states`, ADR-02 de Chat, #32), Server-Roles expone los canales de texto que ve cada usuario (#82) y
  la WebApp muestra los contadores y los marca como leídos al abrir el canal, en web y mobile (#60).
- **Menciones** [Op#46] (31/10): de usuario, de rol y `@everyone`, con autocompletado y resaltado. Chat las
  detecta y las cuenta en los no leídos (#34, ADR-03 de Chat); Server-Roles le pasa los permisos de mención y
  los roles de cada miembro (#83); WebApp #61.
- WebApp: primer MVP del backoffice (#46) y suite E2E con Playwright contra Auth0, Kong y Postgres, en el CI
  (#59).

## Semana 10: 02/11 – 08/11 (en curso)

**Backlog:** +4 / −6, quedan 12 abiertos.
- Se cierran login con proveedor federado [Op#33], reordenar jerarquía de roles [Op#56], 2FA [Op#35], el
  pub/sub del chat (Chat #7), la adaptación de la observabilidad a la nueva infraestructura
  (observability #4) y el prototipo (orquestador #1).
- Se abre adjuntar archivos en mensajes [Op#47] (Chat #38).

**Entregado:**
- **Backoffice, etapa 0:** app aparte en el monorepo (`apps/backoffice`, WebApp #62), con su login de Auth0,
  su dominio y su proyecto de Cloudflare Pages (orquestador #27). Todas las pantallas tienen su forma final y
  quedan deshabilitadas hasta que exista su endpoint de staff. Gateway acepta el origen del backoffice y valida
  el rol `staff` (#21).
- **Login con Google** [Op#33]: en web y mobile, con onboarding del perfil. Auth vincula la identidad federada
  a la cuenta existente si el email está verificado (Auth #34, WebApp #66).
- **Reordenar la jerarquía de roles** [Op#56] (Server-Roles #85, WebApp #65).
- **2FA** [Op#35]: TOTP con código QR, códigos de recuperación de un solo uso y desactivación, en web y
  mobile (Auth #36, WebApp #69).
- **Pub/sub del chat** (Chat #36): los mensajes se reparten entre instancias por Redis, como pide el ADR 0006.
  Cierra el RNF que quedaba abierto desde la semana 2.
- **Estado de la plataforma en el backoffice:** Notification-Audit-Metrics consulta los `/readyz` de cada
  servicio y expone `GET /api/v1/status` y `/api/v1/admin/status` (#23, con su ADR-02); Kong publica los
  `/readyz` internos y métricas de Prometheus (Gateway #23); el backoffice muestra el estado en vivo
  (WebApp #67). La observabilidad se adapta a los tres nodos (observability #5, orquestador #29). Es la base
  de [Op#74]; faltan los incidentes.
- Mobile: los builds de release conservan las clases de WebRTC (reglas de R8).
- **Adjuntos en mensajes** [Op#47]: subida y envío de imágenes, audio, video y documentos hasta 10 MB. Chat guarda el archivo (en disco local con volumen persistente o S3) y emite el mensaje con adjuntos (Chat #39); Kong expone la descarga pública por `/api/v1/chat/uploads` (Gateway #24); web y mobile permiten adjuntar con vista previa y renderizan visor de fotos, reproductores de audio/video y tarjetas de descarga (WebApp #70).
- **Resolución de alias federados en perfiles:** Auth vincula los IDs federados de Google a la cuenta unificada para que el autor no aparezca con alias genéricos tipo "User 5321" en el chat y lista de miembros (Auth #38).
- **WebApp CI y Mobile:** deshabilitación temporal de E2E en CI por aislamiento de base de datos compartida; correcciones de memoización en React Compiler y refs en mobile.
- Releases a `main` el 02/11 de Chat, WebApp, Gateway, Server-Roles y el orquestador, y el 03/11 de WebApp,
  Gateway, Notification, observability y el orquestador.

**Decisiones:**
- El rol `staff` sale de `app_metadata.role` de Auth0, que solo se cambia desde el Dashboard o la Management
  API, y lo valida el backend en cada ruta de staff. Ver `Gateway/docs/ADR-002-Rol-Staff-Backoffice.md`.
- Alcance del Checkpoint 3: las cuatro obligatorias que faltan y 18 optativas (74 pts), para cerrar con 221
  pts sobre un mínimo de 216 (02/11). El 03/11 se suman adjuntos [Op#47] y llamada de voz privada [Op#50],
  para completar el bloque de mensajes directos junto con DM [Op#42] y bloqueo [Op#43]: el cierre previsto
  pasa a 239 pts. Ver `PLAN-CHECKPOINT-3.md`.

---

## Semana 11: 09/11 – 15/11 (Checkpoint 3, en curso)

**Entregado:**
- **Backoffice de usuarios [O#29, O#30, O#31]**: listado de usuarios con búsqueda por email/nombre y filtro por estado (Auth-User PR #44, Gateway PR #26), suspensión y reactivación a nivel plataforma (`ADR-002-Suspension-Usuarios`) con desconexión de sesiones activas en Chat (PR #45) y Voice (PR #39), y habilitación de `users.list`, `users.suspend` y `users.reactivate` en `apps/backoffice` (WebApp PR #76). Con esto se completan las 32 historias obligatorias (104 pts).
- **Backoffice de servidores [O#32, Op#68, Op#69]**: listado de servidores con filtros, suspensión y levantamiento (`ADR-002-Suspension-de-Servidores` en Server-Roles PR #98, Chat PR #51, Voice PR #40, Gateway PR #29, WebApp PR #84) y eliminación permanente desde el backoffice con cierre y redirección en vivo en web y mobile (Server-Roles PR #102, Chat PR #53, Voice PR #42, Gateway PR #31, WebApp PR #86).
- **Edición, borrado y reacciones en mensajes [Op#44, Op#45, Op#48]**: edición de mensajes en canales y DMs con sync al inbox, borrado lógico (tombstone) respetando `MANAGE_MESSAGES` y reacciones con emoji por categorías (`can_add_reactions`, `ADR-04-Reacciones-y-Catalogo-Emojis`) en Chat (PR #46), Server-Roles y WebApp (PR #77, web y mobile).
- **Configuración del servidor en vivo [Op#39, parcial]**: edición de nombre, descripción, ícono y banner (`PATCH /servers/{id}` en Server-Roles PR #96), sincronización en tiempo real por el inbox (`SERVER_UPDATED` en Chat, [ADR 0013](docs/architecture/decisions/0013-cambios-de-servidor-en-vivo.md)) e interfaz en web (WebApp PR #80). Resta habilitar la edición de nombre, descripción y banner en mobile.
- **Registro de auditoría de moderación [Op#61]**: permiso `VIEW_AUDIT_LOG` (bit 23) y eventos enriquecidos en Server-Roles (PR #97), evento `message.deleted_by_moderator` en Chat, consumidor y endpoint en Notification (PR #26, `ADR-04`), ruta en Kong (Gateway PR #28) y visor con filtros y diff en web y mobile (WebApp PR #82).
- **Estado de la plataforma e incidentes [Op#74, parcial]**: publicación y resolución de incidentes con migraciones y sugerencia automática desde `/readyz` en Notification (PR #25, `ADR-03`), Gateway (PR #27), backoffice (`status.incidents`) y banner/modal en web (WebApp PR #81). Resta mostrar el aviso en mobile.
- **Eliminar servidor [Op#40]**: `DELETE /servers/{id}` en Server-Roles (PR #94) con borrado en cascada en Postgres, borrado de icono en S3 y evento en RabbitMQ para Chat. Modal de confirmación y redirección en web y mobile (WebApp PR #78).
- **Gráfico de evolución de miembros [Op#72]**: tabla append-only `ServerMemberEvent` y endpoint `GET /servers/{id}/metrics/member-growth` en Server-Roles (PR #95), y componente `MemberGrowthChart` en web y mobile (WebApp PR #79). Ver [ADR 0012](docs/architecture/decisions/0012-metricas-crecimiento-miembros.md).
- **Métricas globales de la plataforma [Op#71]**: `GET /api/v1/admin/metrics?from&to` (staff) en Notification (PR #27), consultando a Chat (`/internal/metrics/messages`, PR #52) y Server-Roles (`/internal/metrics/servers`, PR #101), ruta `admin-metrics` en Kong (PR #30) y pantalla en el backoffice (WebApp PR #85). Ver [ADR 0014](docs/architecture/decisions/0014-metricas-globales-de-la-plataforma.md).
- **Métricas de actividad de un servidor [Op#70]**: `GET /api/v1/servers/:serverId/metrics` en Notification (PR #28) con endpoints internos en Chat (PR #55) y Server-Roles (PR #103), ruta en Kong (PR #32), variables en `docker-compose.yml` (orquestador PR #36) y gráficos/tarjetas en ajustes del servidor en web y mobile (WebApp PR #89).
- **Llamadas de voz privadas [Op#50, Op#54]**: llamadas 1 a 1 sobre LiveKit (`call-<uuid>`), autorización delegada a Chat (`/internal/conversations/:id/call-access`) y aviso por el inbox en Voice (PR #44), Chat y WebApp (PR #87, audio y cámara en web y mobile). Ver [ADR 0015](docs/architecture/decisions/0015-llamadas-privadas.md).

**Decisiones:**
- Arquitectura de métricas de crecimiento basada en tabla append-only `server_member_events` calculada bajo demanda, sin requerir cron jobs pesados. Ver [ADR 0012](docs/architecture/decisions/0012-metricas-crecimiento-miembros.md).
- Cambios de configuración del servidor propagados por el inbox de Chat (`SERVER_UPDATED`) para invalidar `['servers']` en vivo sin polling. Ver [ADR 0013](docs/architecture/decisions/0013-cambios-de-servidor-en-vivo.md).
- Métricas globales consultadas en el momento a Chat y Server-Roles, en vez de eventos y contadores en Notification. Ver [ADR 0014](docs/architecture/decisions/0014-metricas-globales-de-la-plataforma.md).
- Llamadas privadas 1 a 1 en una sala de LiveKit separada, delegando la autorización en Chat y reutilizando el inbox del cliente. Ver [ADR 0015](docs/architecture/decisions/0015-llamadas-privadas.md).

---

## Estado del backlog al 14/11

**Obligatorias:** **32 / 32 hechas (104 pts)**. Se completaron las cuatro del backoffice ([O#29–O#32]).

**RNF que faltan:**
- S3 para los adjuntos de Chat en producción (`deploy/docker-compose.yml` y Terraform);
- pruebas de carga con k6 y su documentación;
- contratos OpenAPI verificados para los servicios en Node;
- cobertura ≥ 70 % medida por servicio;
- proteger `main` (limitado por el plan Free de GitHub en repos privados).

**Optativas hechas (✅):** login con proveedor federado [Op#33], 2FA [Op#35], perfil público [Op#36],
estado personalizado [Op#38], configuración del servidor [Op#39], eliminar servidor [Op#40], reordenar canales
[Op#41], mensajes directos (DM) [Op#42], bloqueo y desbloqueo [Op#43], editar mensaje [Op#44], eliminar mensaje
[Op#45], menciones [Op#46], adjuntar archivos en mensajes [Op#47], reacciones con emoji [Op#48], llamadas de
voz privadas [Op#50], indicador de quién habla [Op#51], silenciar a otro localmente [Op#52], video en llamada
[Op#54], eliminar rol [Op#55], reordenar jerarquía de roles [Op#56], ver miembros por rol [Op#58], expulsar miembro
[Op#59], silenciar temporalmente [Op#60], registro de auditoría de moderación [Op#61], eliminar servidor desde el
backoffice [Op#68], suspender servidor completo [Op#69], métricas de actividad de un servidor [Op#70], métricas
globales [Op#71], gráfico de crecimiento de miembros [Op#72] y estado de la plataforma e incidentes [Op#74].
Suman **127 pts**. Con las 32 obligatorias (104 pts), el total confirmado es **231 pts** (superando el mínimo
exigido de 216 pts por 15 pts de margen).

**Optativas abiertas en el backlog:** permisos por canal [Op#57] (5 pts, issue abierto Server-Roles #18) y exportar
métricas a CSV [Op#73] (3 pts, issue abierto observabilidad #10).
