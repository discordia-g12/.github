# Estado de Historias de Usuario — Discordia

## 1. Historias Obligatorias

| # | Historia de Usuario | Épica | Estado | Descripción | Evidencia / PRs |
|:---:|---|:---:|:---:|---|---|
| **01** | **Registro de usuarios** | Usuarios | 🟢 Completada | Registro con email y contraseña con validaciones. | Auth0 + `Auth-User-Service`, WebApp PR #2, #4. E2E `auth.spec.ts`. |
| **02** | **Login con email y contraseña** | Usuarios | 🟢 Completada | Inicio de sesión y generación de JWT. | Gateway Kong (`auth0_validator.lua`), WebApp y Mobile login. |
| **03** | **Recupero de contraseña** | Usuarios | 🟢 Completada | Envío de correo para restablecer contraseña. | `Auth-User-Service` `/password-reset`, Discordia-WebApp PR #20. |
| **04** | **Edición de perfil** | Perfil | 🟢 Completada | Modificar nombre, bio y avatar en S3. | `Auth-User-Service` PATCH `/profile` y `/profile/avatar`, WebApp PR #16, #39. |
| **05** | **Visualización de perfil propio** | Perfil | 🟢 Completada | Panel con datos del usuario autenticado. | `Auth-User-Service` GET `/users/me`, WebApp PR #15. |
| **06** | **Crear servidor** | Servidores | 🟢 Completada | Creación de servidor con nombre e icono en S3. | `Server-Roles` POST `/servers`, PUT `/servers/{id}/icon`, E2E `fullstack.spec.ts`. |
| **07** | **Generar invitación** | Servidores | 🟢 Completada | Crear enlaces con código para invitar miembros. | `Server-Roles` POST `/servers/{id}/invites`, WebApp PR #19, E2E `invites.spec.ts`. |
| **08** | **Unirse a servidor vía invitación** | Servidores | 🟢 Completada | Vista previa y alta de miembro usando código de invitación. | `Server-Roles` GET preview #54, POST `/invites/{code}/join`, WebApp PR #19. |
| **09** | **Abandonar servidor** | Servidores | 🟢 Completada | Salida de miembros no dueños del servidor. | `Server-Roles` POST `/servers/{id}/leave` (#6), WebApp PR #18. |
| **10** | **Transferir propiedad del servidor** | Servidores | 🟢 Completada | Traspaso del rol de Owner a otro miembro. | `Server-Roles` POST `/servers/{id}/transfer-ownership` (#11), WebApp PR #44. |
| **11** | **Crear canal** | Canales | 🟢 Completada | Crear canales de texto y voz en un servidor o categoría. | `Server-Roles` POST `/servers/{id}/channels`, E2E `channels.spec.ts`. |
| **12** | **Editar canal** | Canales | 🟢 Completada | Modificar nombre y tópico de un canal. | `Server-Roles` PATCH `/channels/{id}` (#5, PR #33), WebApp UI. |
| **13** | **Eliminar canal** | Canales | 🟢 Completada | Eliminación definitiva de un canal. | `Server-Roles` DELETE `/channels/{id}` (#7, PR #30), WebApp UI. |
| **14** | **Organizar canales en categorías** | Canales | 🟢 Completada | Agrupación de canales en carpetas colapsables. | `Server-Roles` POST `/servers/{id}/categories` (#8), E2E `channels.spec.ts`. |
| **15** | **Enviar mensaje en un canal** | Mensajería | 🟢 Completada | Envío y recepción de mensajes vía WebSockets. | `Chat-Engine` (Fastify + WebSockets + MongoDB + RabbitMQ), E2E `chat.spec.ts`. |
| **16** | **Historial y reconexión** | Mensajería | 🟢 Completada | Paginación de mensajes anteriores y reconexión de WebSocket. | `Chat-Engine` GET `/channels/{id}/messages`, WebApp PR #29, PR #43. |
| **17** | **Unirse a canal de voz** | Canal de Voz | 🟢 Completada | Conexión a sala de voz con LiveKit SFU (WebRTC). | LiveKit + `Voice-Signaling` (`/join-token`), Web & Mobile PR #21, #26. |
| **18** | **Salir de canal de voz** | Canal de Voz | 🟢 Completada | Desconexión y liberación de sesión. | `Voice-Signaling` POST `/leave`, webhooks LiveKit PR #18. |
| **19** | **Silenciar micrófono propio** | Canal de Voz | 🟢 Completada | Mute y unmute de micrófono local. | Controles en Web y Mobile UI (WebRTC audio track mute). |
| **20** | **Ver participantes conectados** | Canal de Voz | 🟢 Completada | Lista en vivo de usuarios en canales de voz. | `Voice-Signaling` `/ws/voice` y REST, WebApp PR #23, #27. |
| **21** | **Crear rol** | Roles | 🟢 Completada | Crear roles con nombre, color y permisos base. | `Server-Roles` POST `/servers/{id}/roles` (#12), WebApp PR #8, #9. |
| **22** | **Editar permisos de un rol** | Roles | 🟢 Completada | Modificar permisos del rol. | `Server-Roles` PATCH `/roles/{id}/permissions` (#13), WebApp PR #10, #11, #12. |
| **23** | **Asignar rol a un miembro** | Roles | 🟢 Completada | Asignar y desasignar roles a miembros. | `Server-Roles` POST/DELETE `/servers/{id}/members/{user_id}/roles` (#14), WebApp PR #16. |
| **24** | **Definir rol por defecto** | Roles | 🟡 En Progreso | Rol asignado automáticamente al unirse miembros. | Endpoint base listo (`Server-Roles` #37, PR #13); pendiente asignar en join (#15, #38). |
| **25** | **Banear miembro del servidor** | Moderación | 🟢 Completada | Expulsar y bloquear el reingreso de un usuario. | `Server-Roles` POST `/servers/{id}/bans` (#10), WebApp PR #31, #42. |
| **26** | **Revocar baneo** | Moderación | 🟢 Completada | Levantar el baneo a un usuario. | `Server-Roles` DELETE `/servers/{id}/bans/{user_id}` (#20), WebApp PR #31. |
| **27** | **Ver notificaciones no leídas** | Notificaciones | 🟢 Completada | Indicadores visuales de mensajes y menciones sin leer. | `Notification-Audit-Metrics` #1, WebApp PR #60 (`Feat/unread core`). |
| **28** | **Marcar como leídas** | Notificaciones | 🟢 Completada | Limpiar el estado de no leído al abrir el canal. | `Notification-Audit-Metrics` #2, WebApp PR #60. |
| **29** | **Suspender usuario (Plataforma)** | Administración | 🟡 En Progreso | Bloquear acceso global a un usuario desde el Backoffice. | Issue `Auth-User-Service` #6, Backoffice WebApp PR #62. |
| **30** | **Reactivar usuario suspendido** | Administración | 🟡 En Progreso | Restablecer acceso a un usuario suspendido. | Issue `Auth-User-Service` #7, Backoffice WebApp PR #62. |
| **31** | **Ver listado de servidores** | Administración | 🟢 Completada | Tabla de servidores en el Backoffice. | WebApp PR #46, #62. |
| **32** | **Ver listado de usuarios con filtros** | Administración | 🟡 En Progreso | Buscar y filtrar usuarios en el Backoffice. | Issue `Server-Roles` #21, Backoffice WebApp PR #62. |

---

## 2. Historias Optativas (Seleccionadas en Roadmap)

| ID | Historia Optativa | Épica | Pts | Estado | Descripción | Evidencia / PRs |
|:---:|---|:---:|:---:|:---:|---|---|
| **HU 33** | **Login con proveedor federado** | Usuarios | 5 | ⚪ Pendiente | Login con Google OAuth. | Pendiente de UI en Auth0. |
| **HU 35** | **Autenticación 2FA** | Usuarios | 5 | ⚪ Pendiente | Segundo factor de autenticación con TOTP. | Backlog. |
| **HU 36** | **Visualización de perfil público** | Perfil | 2 | ⚪ Pendiente | Modal con perfil de otros usuarios al hacer click. | Backlog. |
| **HU 37** | **Estado de actividad** | Perfil | 5 | ⚪ Pendiente | Presencia en tiempo real (online, ausente, offline). | Backlog. |
| **HU 39** | **Administrar configuración de servidor** | Servidores | 3 | 🟢 Completada | Editar nombre e icono del servidor en S3. | `Server-Roles` PATCH `/servers/{id}`, WebApp PR #40. |
| **HU 40** | **Eliminar servidor** | Servidores | 5 | 🟢 Completada | Borrar servidor por el propietario. | `Server-Roles` DELETE `/servers/{id}`. |
| **HU 42** | **Enviar mensaje directo (DM)** | Mensajería | 8 | ⚪ Pendiente | Chat privado 1 a 1 entre usuarios. | Backlog. |
| **HU 43** | **Bloquear y desbloquear usuarios** | Mensajería | 3 | ⚪ Pendiente | Bloqueo de usuarios para no recibir mensajes. | Backlog. |
| **HU 44** | **Editar mensaje** | Mensajería | 3 | ⚪ Pendiente | Modificar mensaje enviado con marca de editado. | Backlog. |
| **HU 45** | **Eliminar mensaje** | Mensajería | 3 | ⚪ Pendiente | Borrar mensajes enviados. | Backlog. |
| **HU 46** | **Menciones (@usuario, @rol)** | Mensajería | 5 | 🟢 Completada | Menciones con autocompletado y resaltado visual. | `Chat-Engine` #33, WebApp PR #61. |
| **HU 47** | **Adjuntar archivos en mensajes** | Mensajería | 5 | ⚪ Pendiente | Subir y enviar archivos/imágenes en el chat. | Backlog. |
| **HU 48** | **Reacciones con emoji** | Mensajería | 3 | ⚪ Pendiente | Reaccionar con emojis a mensajes. | Backlog. |
| **HU 49** | **Buscar mensajes** | Mensajería | 5 | ⚪ Pendiente | Búsqueda por texto y fecha en el canal. | Backlog. |
| **HU 51** | **Indicador de quién está hablando** | Canal de Voz | 3 | 🟢 Completada | Resaltado visual en el participante que habla. | Eventos de audio de LiveKit, WebApp #22. |
| **HU 52** | **Silenciar a otro localmente** | Canal de Voz | 2 | 🟢 Completada | Mutear a otro participante solo para uno mismo. | WebApp PR #24 (web) y PR #28 (mobile). |
| **HU 59** | **Expulsar miembro del servidor** | Moderación | 3 | 🟢 Completada | Expulsar miembro y desconectarlo de voz. | `Server-Roles` DELETE miembro, Voice PR #24, WebApp PR #42. |
| **HU 60** | **Silenciar temporalmente (Mute)** | Moderación | 5 | 🟢 Completada | Bloqueo temporal para hablar en voz y texto. | `Server-Roles` POST `/mutes`, checks en Chat y Voice (PR #42). |
| **HU 61** | **Registro de auditoría de moderación** | Moderación | 3 | ⚪ Pendiente | Historial de bans, kicks y mutes. | Backlog. |
| **HU 63** | **Filtro de palabras prohibidas** | Moderación | 3 | ⚪ Pendiente | Bloqueo automático de mensajes con palabras prohibidas. | Backlog. |
| **HU 64** | **Silenciar notificaciones** | Notificaciones | 3 | ⚪ Pendiente | Silenciar alertas de canales o servidores específicos. | Backlog. |
| **HU 65** | **Configurar preferencias de notificación** | Notificaciones | 3 | ⚪ Pendiente | Elegir qué eventos generan notificación. | Backlog. |
| **HU 66** | **Notificaciones push** | Notificaciones | 5 | ⚪ Pendiente | Notificaciones push en la app mobile vía FCM. | Backlog. |
| **HU 70** | **Métricas de actividad de un servidor** | Métricas | 5 | ⚪ Pendiente | Gráficos de actividad de mensajes y miembros. | Backlog. |
| **HU 71** | **Métricas globales de la plataforma** | Métricas | 5 | 🟡 En Progreso | Métricas de infraestructura y tráfico en producción. | Prometheus + Grafana (`observability` PR #2). |
| **HU 74** | **Estado de plataforma e incidentes** | Métricas | 5 | 🟡 En Progreso | Healthchecks `/livez` y `/readyz` en microservicios. | Endpoints implementados en los 7 servicios. |
| **HU 76** | **Suscribirse a Plan Premium** | Monetización | 13 | ⚪ Pendiente | Compra de suscripción Premium. | `Payments-Monetization-Service` scaffold listo con CI/CD. |
| **HU 77** | **Cancelar suscripción Premium** | Monetización | 3 | ⚪ Pendiente | Baja de suscripción recurrente. | Backlog. |
| **HU 78** | **Comprar puntos** | Monetización | 8 | ⚪ Pendiente | Compra de puntos con pasarela de pago. | Backlog. |
| **HU 79** | **Usar puntos para boostear servidor** | Monetización | 13 | ⚪ Pendiente | Descontar puntos para subir nivel de servidor. | Backlog. |
| **HU 80** | **Ver beneficios de servidor boosteado** | Monetización | 3 | ⚪ Pendiente | Visualización de mejoras activas por boost. | Backlog. |

---

## 3. Historias Bonus (Optativas Extra)

| Historia Bonus | Épica | Estado | Descripción | Evidencia / PRs |
|---|:---:|:---:|---|---|
| **Video en llamada** | Canal de Voz | 🟢 Completada | Video por cámara en vivo con LiveKit SFU. | `Voice-Signaling` #17, WebApp PR #38 (Web y Mobile). |
| **Reordenar canales** | Canales | 🟢 Completada | Cambiar orden de canales en un servidor. | `Server-Roles` PATCH `/channels/reorder` (#9, PR #40). |
| **Eliminar rol** | Roles | 🟢 Completada | Borrar roles y desasignarlos de miembros. | `Server-Roles` DELETE `/roles/{id}` (PR #32), WebApp PR #8, #9. |
| **Ver miembros por rol** | Roles | 🟢 Completada | Listar usuarios que tienen un rol asignado. | `Server-Roles` GET `/roles/{id}/members` (PR #39), WebApp PR #14. |
| **Sobrescribir permisos por canal** | Roles | 🟡 En Progreso | Permisos específicos por canal. | Validación interna lista (`/can-send`, `/can-connect`, Issue #18). |
| **Reordenar jerarquía de roles** | Roles | 🟡 En Progreso | Ordenar precedencia de roles. | `Server-Roles` Issue #17. |
| **Suspender servidor completo** | Administración | 🟡 En Progreso | Congelar servidor desde el Backoffice. | `Server-Roles` Issue #22. |
| **Eliminar servidor desde Backoffice** | Administración | 🟡 En Progreso | Borrar servidor desde el Backoffice. | Backoffice MVP (WebApp PR #46, #62). |

---

## 4. Resumen

### Cantidad de Historias por Sección

| Sección | Total | 🟢 Completadas | 🟡 En Progreso | ⚪ Pendientes | % Hecho | % En curso o hecho |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| **1. Historias Obligatorias** | **32** | **28** | **4** | **0** | **87.5%** | **100.0%** |
| **2. Optativas Roadmap** | **31** | **7** | **2** | **22** | **22.6%** | **29.0%** |
| **3. Historias Bonus** | **8** | **4** | **4** | **0** | **50.0%** | **100.0%** |
| **TOTAL** | **71** | **39** | **10** | **22** | **54.9%** | **69.0%** |

### Puntos (Meta de Cátedra: 216 pts)

* **Completados**: **121 pts** (82 pts obligatorias + 26 pts optativas roadmap + 13 pts bonus).
* **En Progreso**: **46 pts** (22 pts obligatorias + 10 pts optativas roadmap + 14 pts bonus).
* **Total Activo (Hecho + En Progreso)**: **167 pts** (77.3% de la meta).
* **Historias obligatorias sin iniciar**: **0**.
