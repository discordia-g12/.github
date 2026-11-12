# ADR 0014: Métricas Globales de la Plataforma

## Estado
Aceptado

## Contexto
La historia #71 pide que el staff vea, para un período que elige, la cantidad de usuarios activos, servidores creados y mensajes enviados, sumando todos los servidores. Un usuario sin rol staff no puede verlas.

Lo que había:

- **El rol staff ya existe** (ADR-002 del Gateway): `app_metadata.role` de Auth0, claim `https://discordia.app/role`, `validate("staff", false)` en Kong y un segundo chequeo de `X-User-Role` en el servicio.
- **El backoffice ya tiene la función** `metrics.global` en "próximamente", asignada a Notification-Audit-Metrics.
- **Los datos están repartidos y ya guardados:** los mensajes en MongoDB de Chat (`messages.createdAt`, `authorId`) y los servidores en `server_roles_db` (`servers.created_at`).
- **No hay eventos para esto:** Chat no publica a RabbitMQ (solo consume) y Server-Roles no emite `server.created`.
- **Ningún servicio guarda el último login ni la última actividad** de un usuario.
- **Grafana no sirve:** mide pedidos HTTP y recursos, no usuarios ni mensajes, y no se publica al staff (ADR-002 del Gateway).

## Decisión
1. **Notification agrega, no guarda.** `GET /api/v1/admin/metrics?from&to` pide los conteos en el momento, en paralelo y con timeout de 2 s, a dos endpoints internos nuevos:
   - Chat: `GET /internal/metrics/messages?from&to` devuelve `messages` y `activeUsers`.
   - Server-Roles: `GET /internal/metrics/servers?from&to` devuelve `serversCreated`.

   Son endpoints `/internal/*` como los que ya usan Chat y Voice: no pasan por Kong, los protege la red. Chat está en el mismo nodo que Notification. Server-Roles se alcanza en el 8002 de edge, por la regla del Security Group que ya existe desde realtime.
2. **Usuario activo = envió al menos un mensaje en el período**, en un canal o por DM. Es lo único que se puede contar hoy sin agregar escrituras en otra ruta caliente (login o cada pedido).
3. **Período:** rango semiabierto `[from, to)` en UTC. Por defecto, los últimos 30 días. Máximo 366 días, y 400 si el rango es inválido. Para que los conteos no recorran toda la colección, Chat agrega un índice `{ createdAt: 1, authorId: 1 }` y Server-Roles un índice sobre `servers.created_at`.
4. **Respuesta parcial:** si Chat o Server-Roles no responden, Notification devuelve 200 con ese dato en `null` y `partial: true`, y el backoffice lo avisa. Una caída de Server-Roles no oculta los mensajes, y al revés.
5. **Acceso (CA 2):** ruta propia `admin-metrics` en Kong (`/api/v1/admin/metrics`, GET y OPTIONS para el CORS del backoffice) con `validate("staff", false)`, y Notification vuelve a chequear el rol. Sin token, 401; con rol `user`, 403. Se siguen las reglas de la §4 del ADR-002 del Gateway.
6. **Backoffice:** el hook `usePlatformMetricsQuery` en `@discordia/core` y una pantalla con selector de período (7, 30 o 90 días, o un rango a elección) y tres tarjetas. La función `metrics.global` pasa a `ready`.

El detalle de cada servicio está en el ADR-05 de Notification-Audit-Metrics-Service.

## Alternativas Descartadas
1. **Eventos y contadores en Notification** (lo que se había planeado en el checkpoint 3: Chat publica eventos de mensajes y Notification los cuenta):
   - *Pros:* Notification no depende de que Chat y Server-Roles respondan. Los contadores diarios salen sin consultar nada.
   - *Contras:* Un evento por mensaje es el volumen más alto del sistema, y Chat no tiene outbox (Mongo, sin transacción con RabbitMQ): habría que construirlo, o aceptar perder conteos. Hace falta cargar los datos viejos para los períodos anteriores al cambio. Además, `server.created` no existe. Queda como evolución si el conteo en el momento se vuelve lento.
2. **Que el backoffice llame a cada servicio:**
   - *Descartada:* Habría tres rutas de staff en Kong en vez de una y tres chequeos de rol en vez de uno, y el front decidiría cómo combinar los datos.
3. **Usuario activo = inició sesión en el período:**
   - *Descartada por ahora:* Auth no guarda el último login, y Auth0 lo sabe pero consultarlo por usuario es caro y tiene límite de pedidos. Habría que escribir en Auth en cada login.
4. **Grafana / Prometheus:**
   - *Descartada:* No tiene los datos de negocio y no se expone al staff.

## Consecuencias
- **Positivas:**
  - Funciona con los datos históricos desde el primer día, sin backfill ni workers.
  - Cada servicio cuenta solo lo suyo, y Notification es el único lugar con reglas de staff y de período.
  - El mismo endpoint sirve de base para exportar a CSV (#73).
- **Negativas:**
  - Cada consulta recorre el índice de mensajes del período. Con un año de datos y mucho tráfico puede tardar más que el timeout: por eso el máximo de 366 días y el cache corto en Notification.
  - Un servidor borrado físicamente no se cuenta en "servidores creados".
  - "Usuario activo" no incluye a quien solo leyó o solo estuvo en voz.
  - Suma dos dependencias síncronas a Notification (§9 de `ARQUITECTURA.md` del orquestador), aunque solo para esta pantalla.
