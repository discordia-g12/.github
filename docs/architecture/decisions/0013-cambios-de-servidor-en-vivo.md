# ADR 0013: Cambios de Configuración del Servidor en Vivo

## Estado
Aceptado

## Contexto
La historia "Administrar configuración del servidor" pide que, al guardar el nombre, el ícono o el banner, el cambio "se refleje para todos los miembros". Lo que había:

- **Server-Roles ya emitía `server.updated`** (Outbox, ADR 0004) al cambiar el ícono, con solo `{ server_id }`, pero nadie lo consumía.
- **La WebApp pide los servidores una sola vez** (`GET /api/v1/servers`, React Query) y no tiene polling. Otro miembro veía el cambio recién al recargar.
- **El WebSocket de canal no alcanza.** Solo lo tiene abierto quien está mirando un canal de ese servidor. La barra de servidores muestra el nombre, el ícono y el banner de todos, así que el aviso le importa a cualquier miembro conectado, esté donde esté.
- **Ya existe un socket por usuario:** el inbox de Chat (`GET /inbox/ws`, ADR 0011), publicado por Redis en `inbox:<usuario>`, que funciona con varias instancias. Chat no sabe quién es miembro de qué servidor: eso es de Server-Roles.

## Decisión
1. **Chat consume `server.updated`** (binding nuevo en `chat_engine_events`). Server-Roles lo emite al cambiar nombre, descripción, ícono o banner, en la misma transacción que el cambio. Si nada cambió, no lo emite.
2. **Chat le pregunta a Server-Roles a quién avisar:** `GET /internal/servers/{id}/member-ids`, en el mismo puerto interno que `/internal/channels/{id}/access` (no pasa por Kong).
3. **Chat publica `{ type: 'SERVER_UPDATED', serverId }` en el inbox de cada miembro.** Quien no está conectado no recibe nada y ve el cambio la próxima vez que pide sus servidores.
4. **El cliente vuelve a pedir los datos.** Al recibir `SERVER_UPDATED`, `useInbox` (core, compartido por web y mobile) invalida `['servers']`. El evento lleva solo el id, como dice ADR 0004: los datos y los permisos de cada uno salen de la API.
5. **Fallas.** Si el servidor ya no existe (404), no se avisa. Si Server-Roles no responde, se registra y se descarta el aviso, sin reencolar: es una notificación, el dato ya está guardado.

## Alternativas Descartadas
1. **Avisar por el WebSocket de cada canal del servidor** (`SessionManager.getServerSessions`):
   - *Pros:* No necesita endpoint nuevo ni consultar a Server-Roles.
   - *Contras:* No le llega a quien está en otro servidor o en los DMs, que igual ve la barra. Las sesiones son locales a cada instancia de Chat.
2. **Polling o `refetchOnWindowFocus` de `['servers']`:**
   - *Pros:* Sin cambios en el backend.
   - *Contras:* No es en vivo. Cada cliente pide la lista completa (con canales y categorías) cada N segundos, cambie algo o no.
3. **Publicar `SERVER_UPDATED` en el inbox de todos los conectados y que el cliente filtre:**
   - *Descartada:* Le cuenta a quien no es miembro qué servidores existen y cuándo cambian.
4. **Mandar los ids de los miembros dentro del evento:**
   - *Descartada:* El mensaje crece con el servidor, y la lista puede quedar vieja si alguien entra o sale entre el commit y la publicación. Rompe además la regla de ADR 0004 (el evento avisa, no transporta el estado).
5. **Mandar el servidor completo por el inbox:**
   - *Descartada:* `my_permissions` es distinto para cada miembro, así que no hay un payload único, y duplicaría la serialización de la API.

## Consecuencias
- **Positivas:**
  - Todos los miembros conectados ven el cambio en segundos, en web y en mobile, sin recargar.
  - Sirve igual para cualquier cambio futuro de la identidad del servidor: basta con emitir `server.updated`.
  - No hay polling.
- **Negativas:**
  - **Una llamada síncrona más de Chat a Server-Roles** por cada cambio, y un endpoint `/internal` sin autenticación (lo protege el Security Group, igual que los otros).
  - **N publicaciones en Redis por cambio**, una por miembro. Con los tamaños del TP no importa; un servidor muy grande pediría un canal Redis por servidor.
  - **Cada aviso dispara un `GET /api/v1/servers` por miembro conectado**, con todos sus servidores.
  - **Se puede perder un aviso** si Server-Roles no responde en ese momento. El dato está guardado y se ve al volver a pedir la lista.
