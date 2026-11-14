# ADR 0015: Llamadas de Voz Privadas 1 a 1

## Estado
Aceptado

## Contexto
La historia #50 pide llamar por voz a otro usuario desde una conversación directa: suena del otro lado, se puede aceptar o rechazar, cualquiera de los dos corta y quien llama se entera de cómo terminó (rechazada, no respondió, ocupado, finalizada). Depende de #42 (DMs) y #43 (bloqueo), ver [ADR 0011](0011-mensajes-directos.md).

Lo que había:

- **El join de Voice es solo para canales de servidor.** `POST /rooms/:channelId/join` le pregunta a Server-Roles (`can-connect`), y la presencia, los webhooks y la moderación usan claves por canal y por servidor.
- **La regla "¿puede contactarlo?" vive en Chat.** `ConversationService.assertCanSend` revisa participante, conversación `pending`, bloqueo en los dos sentidos y suspensión, y Chat ya tiene el cache del id canónico.
- **El cliente ya escucha un inbox por usuario** (`/inbox/ws` de Chat, `inbox:<usuario>` en `chat-redis`), siempre abierto. `/ws/voice` solo está abierto mientras se mira un servidor.
- **Voice y Chat corren en el nodo realtime**, en la misma red de Docker. Voice tiene una sola instancia.
- **`X-User-Id` puede ser un alias** (login federado).

## Decisión
1. **Una sala de LiveKit de 2 personas por llamada, en Voice.** Módulo nuevo `calls` en Voice-Signaling, separado del join de canales. La sala se llama `call-<uuid>` y se crea con `maxParticipants: 2` antes de dar el token. El token (10 min, identidad = `X-User-Id` crudo) deja publicar el micrófono y la cámara (video de la historia #54, sin roles que lo limiten), no la pantalla. Quien llama entra al iniciar; quien recibe, al aceptar. El cliente usa la misma integración con LiveKit que los canales.
2. **Autorización delegada a Chat.** Voice pregunta `GET /internal/conversations/:id/call-access?user_id=…` al iniciar y otra vez al aceptar (el bloqueo pudo aparecer mientras sonaba; ahí un `DM_PENDING` no corta, porque para quien recibe la conversación puede seguir sin aceptar cuando llama quien la rechazó). Chat aplica las mismas reglas que a un DM, sin mandar nada, y devuelve los ids canónicos de los dos. Los errores son los de los DM: `403 DM_NOT_DELIVERABLE` genérico (nunca "te bloquearon") y `403 DM_PENDING`. Para identificar a quien acepta, rechaza o corta, Voice usa la identidad guardada en la llamada o `GET /internal/users/:id/canonical` de Chat.
3. **Aviso por el inbox de Chat.** Voice llama a `POST /internal/inbox/events` y Chat publica `CALL_RINGING`, `CALL_ACCEPTED` y `CALL_ENDED` (motivo en `call.endReason`: `declined`, `missed`, `canceled`, `ended`, `unavailable`) en `inbox:<usuario>` de los dos, en todos sus dispositivos. Chat solo acepta tipos que empiecen con `CALL_`. Si el aviso falla al iniciar, la llamada no se crea (`503`); al terminar, el fallo se loguea y no frena el corte.
4. **Estado en `voice-redis` y timer en el proceso.** `call:<id>` (estado y fechas), `call:user:<canónico>` (ocupado) y `call:identity:<crudo>`. Reservar el ocupado de los dos y crear la llamada es un script Lua, y cada transición es un compare-and-set: aceptar y que venza el timbre a la vez no ganan los dos. El timbre dura 30 s con un `setTimeout` por llamada; TTLs de red de seguridad de 45 s sonando, 12 h activa y 5 min terminada. Ocupado (sonando o activa) responde `409 CALL_BUSY`.
5. **Fin de la llamada.** Cortar (cualquiera de los dos), rechazar, el timbre vencido, alguien que se va de la sala sin cortar (webhook de LiveKit, separado de los canales por el prefijo `call-`), un bloqueo detectado al aceptar y `user.suspended` terminan la llamada, borran la sala y el ocupado y avisan a los dos.
6. **Sin cambios de red ni de Kong.** Voice llega a Chat por `http://chat-engine-service:3000` (`CHAT_SERVICE_URL`, mismo nodo). Las rutas públicas cuelgan de `/api/v1/voice`, que Kong ya rutea. Las `/internal/*` nuevas de Chat responden 404 si llegan por Kong (traen `X-User-Id` o `X-Forwarded-Prefix`), igual que `/internal/metrics/*`.

El detalle está en el ADR-04 de Voice-Signaling-Session-Service.

**Se aparta de [ADR 0011](0011-mensajes-directos.md)**, que sugería que Voice consultara `/internal/relations` a Auth. La relación sola no alcanza: también cuentan la conversación (que exista, que participe, que no esté `pending`) y la suspensión, y esa regla ya está en Chat. Consultar a Auth la duplicaría en Voice, que divergiría con el primer cambio, y sumaría una dependencia entre nodos (realtime → edge) donde Chat está en el mismo nodo. Por lo mismo, Voice no necesita `AUTH_SERVICE_URL` ni la regla del Security Group a edge:8001.

## Alternativas Descartadas
1. **WebRTC P2P, sin SFU:**
   - *Descartada:* Necesita señalización propia (ofertas, candidatos ICE) y TURN para NAT simétricos. LiveKit ya resuelve todo eso, está desplegado y el cliente ya lo usa.
2. **Que Voice consulte a Auth o guarde su propia lista de bloqueos** (lo sugerido en ADR 0011):
   - *Descartada:* Ver arriba. Repite la regla de Chat y agrega una dependencia entre nodos.
3. **WebSocket propio de Voice para avisar:**
   - *Descartada:* `/ws/voice` no está abierto siempre. Un socket más por usuario duplica reconexión, autenticación en Kong y multi-dispositivo, que el inbox ya resuelve.
4. **Avisar por RabbitMQ (Voice publica, Chat reenvía al inbox):**
   - *Pros:* Voice no depende de que Chat responda para avisar.
   - *Contras:* Es asincrónico: al iniciar no se sabe si el aviso salió, y sin aviso no tiene que haber llamada. Suma un consumidor en Chat y depende del nodo infra para algo que pasa entero en realtime.
5. **Voice escribe directo en `chat-redis`:**
   - *Descartada:* Acopla Voice al formato interno y a la base de otro servicio; un cambio del inbox lo rompería en silencio.
6. **Timer distribuido (cola con retardo o notificaciones de expiración de Redis):**
   - *Descartada:* Con una instancia no hace falta. Las notificaciones de expiración de Redis no son puntuales y no traen el valor.

## Consecuencias
- **Positivas:**
  - Una sola regla de "¿puede contactarlo?" para DMs y llamadas: un cambio en Chat vale para las dos.
  - Sin puertos, reglas de Security Group ni rutas de Kong nuevas.
  - El aviso llega a todos los dispositivos de quien recibe, y al atender o rechazar en uno deja de sonar en los otros.
  - Con Chat caído, cortar una llamada sigue funcionando (la identidad está guardada en la llamada).
- **Negativas:**
  - **Iniciar y aceptar dependen de Chat:** sin Chat responden `503`. Es una dependencia síncrona nueva (Voice → Chat), dentro del nodo.
  - **Timer en memoria:** con más de una instancia de Voice, o si el proceso se reinicia mientras suena, nadie publica `missed`. El TTL de 45 s libera el ocupado y el cliente da la llamada por perdida a los `expiresAt + 10 s`. Para escalar Voice hay que mover el timer (por ejemplo, un barrido de llamadas vencidas con el mismo compare-and-set).
  - **Un bloqueo no corta una llamada activa:** se revisa al iniciar y al aceptar.
  - **Sin push con la app cerrada** (M3 de `docs/DEUDA-TECNICA.md` del orquestador): solo suena si el inbox está conectado.
  - **No queda constancia en la conversación:** ni "llamada perdida" ni la duración.
  - **`POST /internal/inbox/events` de Chat lo protege solo la red:** quien llegue al 3000 de realtime puede publicar un evento `CALL_*` en el inbox de cualquiera. Igual que el resto de `/internal`, el Security Group solo abre ese puerto a edge (Kong), y por Kong la ruta responde 404.
