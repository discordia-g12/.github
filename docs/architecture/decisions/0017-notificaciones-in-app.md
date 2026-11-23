# ADR 0017: Notificaciones in-app entre Chat y Notification

## Estado
Aceptado

## Contexto
La historia "Responder en un hilo" pide, en su CA5, avisarle a quien participa de un hilo cuando otro responde,
"según sus preferencias de notificación". Detrás viene una épica de notificaciones: silenciar un canal o un
servidor, elegir el nivel de aviso por canal o servidor, notificaciones push y un centro de notificaciones con
historial.

Hasta ahora no había notificaciones:

- Notification-Audit-Metrics-Service tenía auditoría, métricas y estado, pero ni bandeja ni preferencias. Kong
  ya le enrutaba `/api/v1/notifications` y respondía `404`.
- Chat solo publicaba `message.deleted_by_moderator`. El `chat.message.created` que planteaba el ADR 0004 para
  las menciones nunca se hizo.
- El tiempo real vive en Chat: cada cliente tiene abierto el socket `inbox:<usuario>` (ADR 0006 y ADR 0011).

Lo que hay que decidir es qué servicio sabe qué, cómo viaja el aviso y cómo se garantiza que no se pierda, de
forma que la épica sume casos sin cambiar el contrato.

## Decisión

### Quién decide qué
- **Chat decide quién participa.** Es un dato del chat: el autor del mensaje original, quienes respondieron y
  los mencionados (con un tope de 100 por hilo). Chat saca al autor de la respuesta y, en un servidor, a quienes
  ya no son miembros (`/internal/servers/{id}/member-ids` de Server-Roles).
- **Notification decide a quién avisar.** Es el dueño de las preferencias y de la bandeja. Chat nunca lee las
  preferencias.

### El evento `chat.message.created`
- Va al exchange `server.events`, como el resto de los eventos de dominio. Hoy solo sale cuando el mensaje es
  una respuesta de hilo con alguien a quien avisar.
- El contrato es general, para que las historias de la épica lo reutilicen sin versionarlo:

  ```
  event_id, event_type, server_id (null en un DM), channel_id, thread_id (o null), message_id, author_id,
  thread_participant_ids, mention_user_ids, mention_role_ids, mention_everyone, is_dm, preview (140), created_at
  ```

- **No se pierde (outbox de Chat).** Lo pendiente se guarda en la propia respuesta (`outbox`), en la misma
  escritura del documento, que en Mongo es atómica. Se publica con *publisher confirms*. Lo que no se confirmó lo
  reintenta un relay con backoff, con el mismo `event_id` (ADR-07 de Chat). Mismo objetivo que el outbox de
  Server-Roles (ADR 0004), con la forma que permite Mongo sin transacciones.
- **Se puede repetir; Notification lo absorbe.** Cada aviso es único por `(event_id, user_id)`: un reintento o
  dos instancias de Chat publicando lo mismo no duplican nada.

### Notification
- Cola propia `notification_message_events`, separada de la de auditoría, con DLQ (`<cola>.dlq`) y
  `prefetch(10)`. Un evento inválido o un error de base va a la DLQ, sin reencolar y sin perderse. La auditoría
  pasó al mismo esquema (`notification_audit_events.v2`).
- Tres pasos separados: candidatos (quién y por qué), resolución de preferencias (función pura) y entrega.
- **Preferencias** por `server`, `channel` o `thread`. El nivel (`all`, `mentions`, `none`; `null` hereda) y el
  silencio (`muted`, `muted_until`) son campos distintos: silenciar y reactivar no pisan el nivel elegido. Se
  aplica la más específica (hilo → canal → servidor → default: `mentions` en un servidor, `all` en un DM). Un
  silencio vigente en cualquier nivel silencia todo lo de abajo.
- La bandeja en Postgres es la fuente de verdad. La retención es de 30 días para las leídas y 90 para el resto.

### El aviso en vivo
- Notification llama a `POST /internal/inbox/events` de Chat (mismo nodo, `CHAT_ENGINE_URL`) con
  `NOTIFICATION_CREATED`, en lotes de 10 usuarios, que es lo que acepta Chat por llamada.
- La ruta exige `X-Internal-Token` (`INTERNAL_API_TOKEN`, compartido entre Chat, Voice y Notification). Antes
  la protegía solo la red (deuda V9) y, al aceptar avisos, cualquiera que llegara al 3000 de realtime podía
  mostrarle una notificación falsa a cualquier usuario.
- El evento no lleva el id del aviso ni el usuario, así un mismo mensaje viaja igual para todos. El cliente
  vuelve a pedir la bandeja. Si el aviso en vivo falla, la bandeja lo tiene igual: los clientes la piden cada
  60 s y al volver el foco.

## Alternativas descartadas

| Alternativa | Por qué se descartó |
|---|---|
| Todo en Chat (seguir el hilo, contador de no leídos de hilos) | Las preferencias y la bandeja son de Notification (ADR 0001). La épica pide push e historial: Chat crecería con algo que no es del chat. |
| Que Chat lea las preferencias y mande solo a quienes avisar | Chat consultaría a Notification en cada mensaje, en el camino de escribir. |
| WebSocket propio en Notification | Duplica Redis, reconexión y una ruta más en Kong. El inbox de Chat ya está abierto en todos los clientes. |
| Mandar a todos los miembros del servidor en el evento | No escala con servidores grandes ni con `@everyone`. Para el nivel `all`, Notification busca quién lo pidió con un índice sobre sus preferencias. |
| Publicar sin outbox | Una caída entre guardar y publicar, o RabbitMQ caído, pierde el aviso: un usuario que no se entera de que le respondieron. |
| Un nivel `muted` en lugar de un campo aparte | Reactivar no sabría a qué nivel volver. |

## Consecuencias
- Notification depende de Chat solo para el aviso en vivo, no para guardar el aviso: con Chat caído, los avisos
  se guardan igual y se ven al refrescar.
- `INTERNAL_API_TOKEN` es un secreto más en el `nodes.env` de realtime. Mientras esté vacío, Chat no lo exige
  (modo de transición, ver `deploy/README.md` del orquestador). Hay que cargarlo y después volverlo obligatorio.
- Las DLQs no se vacían solas: hay que revisarlas desde el panel de RabbitMQ.
- Puntos de extensión para la épica, sin cambiar el contrato:
  - **Menciones y DMs:** Chat publica `chat.message.created` también para esos mensajes, y Notification suma
    candidatos con `reason` `mention` y `dm` (el esquema ya los acepta). `@everyone` y las menciones de rol no
    pueden viajar como lista: Notification tendrá que pedir los miembros a Server-Roles.
  - **Silenciar canal o servidor y preferencias por canal:** el modelo y la resolución ya los cubren. Falta la
    interfaz.
  - **Push:** un paso más en la entrega de Notification, con tokens de dispositivo y `push_enabled` por usuario.
  - **Centro de notificaciones:** es la misma bandeja, que ya tiene historial y estado vacío.
- El nivel `all` en canales activos multiplica los eventos en RabbitMQ (nodo infra, `t3.micro`): hay que medirlo
  antes de activarlo.
