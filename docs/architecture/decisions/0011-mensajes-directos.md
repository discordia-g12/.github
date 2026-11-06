# ADR 0011: Mensajes Directos: Conversación, Solicitud de Mensaje, Inbox y Bloqueo

## Estado
Aceptado

## Contexto
Las historias #42 (mensajes directos), #43 (bloquear y desbloquear) y #36 (perfil público) agregan comunicación entre dos usuarios que no depende de un servidor. El sistema ya tenía mensajes, historial, estado de lectura y adjuntos, pero todo colgado de un canal:

- **El WebSocket de Chat es por canal.** Un DM no le llegaría a quien no tiene esa conversación abierta.
- **Los permisos los decide Core.** `channelAccessGuard` y `ChatService.sendMessage` consultan a Server-Roles, que no conoce los DMs.
- **El socket cachea `canSend` al conectar.** Un bloqueo tiene que regir desde el mensaje siguiente, no desde la próxima conexión.
- **Las relaciones entre usuarios no existían.** Hacía falta guardar amistades, solicitudes y bloqueos, y Chat tiene que consultarlos, pero Auth no publicaba ningún puerto hacia realtime.
- **`X-User-Id` puede ser un alias.** Es el `sub` de Auth0: con login federado una misma persona puede llegar con dos ids.

## Decisión
1. **Una conversación es un canal con id `dm-<ObjectId>`.** Chat guarda `Conversation` en Mongo con `participantKey` único (ids ordenados), que hace idempotente la creación ante dos primeros mensajes simultáneos. `Message` y `ReadState` no cambian: historial, lectura, adjuntos y `/channels/:id/ws` funcionan igual. `channelAccessGuard` tiene una rama para `dm-`: exige ser participante y no consulta a Core.
2. **La conversación nace con el primer mensaje.** Abrir un chat no crea nada (`PUT /conversations/with/:userId`). Ese primer mensaje es solo texto, porque subir un adjunto exige un id `dm-…` que todavía no existe.
3. **Solicitud de mensaje.** Entre amigos la conversación nace `accepted`. Entre no amigos nace `pending`: quien inició ve "esperando respuesta" y no puede mandar otro, y quien recibe no puede responder sin aceptar. Aceptar libera la conversación y no los hace amigos. Rechazar la saca de la lista de quien rechazó; quien inició no se entera, y si quien rechazó escribe después pasa a `accepted`. Hacerse amigos pasa a `accepted` cualquier conversación entre ambos.
4. **Las relaciones viven en Auth, los mensajes en Chat.** Auth guarda `friend_requests`, `friendships` y `user_blocks`, siempre con el id canónico, y expone un API interno (`/internal/users/{id}/canonical` y `/internal/relations`) que no pasa por Kong.
5. **Autorización por envío y fail closed.** En cada DM, por WebSocket o REST, Chat consulta la relación a Auth con un timeout de 2 s, sin usar el `canSend` del socket. Si Auth no responde, 503 y no se entrega nada. El id canónico se cachea 60 s, porque solo cambia al vincular una cuenta; la relación no se cachea.
6. **Bloqueo.** Corta en los dos sentidos hasta desbloquear y conserva el historial. Bloquear borra la amistad y las solicitudes pendientes en una transacción. Quien está bloqueado recibe un error genérico (`403 DM_NOT_DELIVERABLE`, el mismo que para una cuenta suspendida o inexistente), y `relationship` nunca dice que te bloquearon.
7. **Inbox por usuario.** `GET /inbox/ws` es un socket solo de recepción. Cada DM se publica por Redis en `inbox:<usuario>` de ambos participantes y en el canal `dm-…`, así funciona con varias instancias. Un cliente que reconecta vuelve a pedir `GET /conversations`: el inbox no reenvía lo perdido.
8. **Presencia mínima.** Una clave en Redis con TTL de 60 s que renueva el heartbeat del inbox, consultada con `POST /presence`. Solo responde `online` u `offline`; ausente y no molestar son #37.
9. **Perfil público en dos servicios.** Auth responde los datos y `relationship`; un perfil suspendido o inexistente es `404`. Core responde `GET /api/v1/servers/mutual/{user_id}`. El estado de actividad sale de la presencia de Chat. Ninguno devuelve email ni nada de DMs.
10. **Comunicación entre nodos.** Auth publica `8001:8000` y el Security Group de edge lo abre solo al grupo realtime. Kong no cambia: todo cuelga de prefijos que ya ruteaba.

## Cómo se engancha #50 (llamadas privadas)
La llamada se inicia desde una conversación y reutiliza sus decisiones: Voice consulta `/internal/relations` igual que Chat, antes de crear la sala de LiveKit, y rechaza con el mismo error genérico si hay bloqueo (cubre la parte de llamadas de los CA de #43). El aviso de llamada entrante puede viajar por `inbox:<usuario>`. Voice necesita `AUTH_SERVICE_URL` y una regla de Security Group: edge:8001 ya está abierto al grupo realtime, donde corre Voice.

## Alternativas Descartadas
1. **Canal de Core con dos miembros:**
   - *Descartada:* Mezcla DMs con servidores, roles y bans, y haría pasar cada mensaje por Core, que no tiene que saber de relaciones personales.
2. **Rutas `/dm/...` separadas en Chat:**
   - *Descartada:* Duplica historial, lectura, adjuntos y WebSocket, que ya funcionan por id de canal.
3. **Guardar relaciones y bloqueos en Chat:**
   - *Descartada:* Las necesitan el perfil (Auth) y las llamadas (Voice). Quedarían fuera del servicio dueño de la identidad.
4. **Copiar los bloqueos a Chat por eventos:**
   - *Pros:* Sin dependencia síncrona ni puerto nuevo.
   - *Contras:* Con la consistencia eventual, un bloqueo no rige en el mensaje siguiente, que es lo que promete el CA 1 de #43. Se puede revisar junto con la foto de permisos (P1 de `docs/DEUDA-TECNICA.md`).
5. **Notificar por el socket de cada canal abierto:**
   - *Descartada:* Exigiría tener abierto un socket por conversación, y una solicitud nueva no tiene canal abierto todavía.
6. **Decirle a quien fue bloqueado que lo bloquearon:**
   - *Descartada:* Invita a insistir por otra vía y expone la decisión de quien bloqueó.

## Consecuencias
- **Positivas:**
  - Historial, lectura y adjuntos se reutilizan sin cambios de modelo.
  - Un bloqueo rige desde el mensaje siguiente, también en un socket que ya estaba abierto.
  - Si Auth cae, ningún DM se entrega sin autorización, y los canales siguen funcionando.
- **Negativas:**
  - **Dependencia síncrona nueva:** cada DM agrega hasta dos llamadas a Auth, entre nodos, y un puerto abierto entre edge y realtime.
  - **`/internal` sin autenticación:** igual que en Server-Roles, lo protege solo el Security Group. El 8001 expone la API completa de Auth, que confía en `X-User-Id`, a cualquiera que llegue desde realtime.
  - **Una solicitud de amistad bloqueada se puede inferir a nivel API:** responde `202` en vez de `201` y no aparece en `outgoing`. En la interfaz se ve igual que una exitosa.
  - **El estado de amistad no es en tiempo real:** las solicitudes se piden cada 30 s y al volver a la app, y a quien lo desbloquean el composer le vuelve con ese mismo polling, no con un evento.
  - **Sin push a mobile:** un destinatario sin la app abierta ve el mensaje al volver.
