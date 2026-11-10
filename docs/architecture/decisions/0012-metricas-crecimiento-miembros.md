# ADR 0012: Métricas de Crecimiento de Miembros del Servidor

## Estado
Aceptado

## Contexto
La historia #72 (gráfico de evolución de miembros) exige que un miembro con permisos de administración pueda consultar la evolución temporal de miembros de su servidor día a día dentro de un período seleccionado, para identificar tendencias de crecimiento o caída.

Requerimientos y restricciones:
- **Historial temporal granular:** La tabla existente `server_members` solo guardaba el estado actual de pertenencia, sin registrar fecha ni causa de bajas (abandonos, expulsiones, baneos).
- **Consistencia y rendimiento:** Realizar snapshots diarios programados vía cron genera sobrecarga operativa y problemas de sincronización en desarrollo y despliegues dinámicos.
- **Seguridad:** La información de métricas debe restringirse estrictamente a administradores (`MANAGE_SERVER` o `ADMINISTRATOR`) y al propietario del servidor.
- **Multiplataforma:** La visualización debe funcionar de forma consistente en la Web y en la app Mobile (React Native).

## Decisión
1. **Modelo de eventos `ServerMemberEvent` en `Server-Roles-Core-Service`:**
   Se añade una tabla append-only `server_member_events` en PostgreSQL con los campos:
   - `id`: UUID (clave primaria).
   - `server_id`: UUID (clave foránea con índice y borrado en cascada).
   - `user_id`: String (identificador canónico del usuario).
   - `event_type`: String (`joined`, `left`, `kicked`, `banned`).
   - `created_at`: DateTime con zona horaria UTC e índice combinado con `server_id`.

2. **Registro atómico de eventos en transacciones existentes:**
   Se registran eventos en el mismo ciclo transaccional de cada operación:
   - Creación de servidor: evento `joined` para el propietario inicial.
   - Aceptación de invitación (`create_server_member`): evento `joined`.
   - Abandono de servidor (`remove_member`): evento `left`.
   - Expulsión de moderación (`kick_member`): evento `kicked`.
   - Baneo de moderación (`ban_member`): evento `banned`.

3. **Migración Alembic con backfill:**
   La migración crea la tabla, índices y genera automáticamente un evento `joined` retroactivo para cada miembro existente en `server_members`, preservando la consistencia histórica.

4. **Cálculo dinámico día a día (`get_member_growth`):**
   El cálculo de la serie temporal se realiza bajo demanda:
   - Se agrupan los eventos anteriores al `start_date` para computar el conteo base (*baseline*). Si no hubiese eventos previos registrados, se infiere del conteo actual en `server_members`.
   - Se itera secuencialmente por cada día del período proyectando el conteo neto acumulado, junto con la suma de altas (`joined`) y bajas (`left` + `kicked` + `banned`).
   - Soporta ventana por días (`?days=7`, `?days=14`, `?days=30`, por defecto 30) o rango personalizado `start_date` / `end_date`.

5. **Control de acceso y autorización:**
   El servicio valida pertenencia y permisos mediante `_get_manageable_server`:
   - Permitido: Propietario (`owner_id`), o miembros con permiso `MANAGE_SERVER` o `ADMINISTRATOR`.
   - Denegado: Miembros sin permisos suficientes o no miembros reciben `403 Forbidden`. Servidores inexistentes devuelven `404 Not Found`.

6. **Endpoints expuestos:**
   - `GET /api/v1/servers/{server_id}/metrics/member-growth` (y alias `/api/v1/servers/{server_id}/member-growth`).

7. **Consumo en Frontend y Visualización:**
   - `@discordia/core`: Tipos TypeScript y hook React Query `useMemberGrowthQuery`.
   - Web: Componente `MemberGrowthChart` con gráficos vectoriales SVG (`Polyline` / `Polygon`), tarjetas de resumen (miembros actuales, variación neta, altas y bajas), selector de período y tooltips interactivos con hover.
   - Mobile: Componente nativo con `react-native-svg` y selector táctil, integrado en la pantalla de Ajustes del Servidor.

## Consecuencias
- **Positivas:**
  - No requiere procesos batch ni workers en segundo plano para generar snapshots diarios.
  - Trazabilidad y auditoría exacta de cuándo y por qué un usuario ingresó o dejó el servidor.
  - Tiempos de respuesta sub-10 ms gracias a índices compuestos en `(server_id, created_at)`.
  - Experiencia de usuario coherente y fluida en Web y Mobile.
- **Negativas:**
  - El tamaño de la tabla `server_member_events` crece de manera proporcional a la rotación de miembros (mitigado con índices compactos).
