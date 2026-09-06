# ADR 0006: Arquitectura de Mensajería en Tiempo Real y WebSockets Multi-Instancia

## Estado
Aceptado

## Contexto
La mensajería en canales y mensajes directos (DMs) es el núcleo de Discordia. Requiere entrega en tiempo real (< 100ms), soporte para múltiples réplicas del servicio en paralelo, persistencia de historial y sincronización de mensajes perdidos ante desconexiones transitorias de red en dispositivos móviles.

## Decisión
1. **Protocolo:** Se utiliza **WebSockets** bidireccional sobre Node.js/TypeScript.
2. **Escalabilidad Horizontal con Redis Pub/Sub:**
   - Cada réplica del `Chat Engine Service` gestiona un conjunto de conexiones de clientes.
   - Cuando un cliente conectado al *Pod A* envía un mensaje al canal `#general`, el *Pod A* valida permisos, almacena el mensaje en MongoDB y publica el payload en Redis Pub/Sub (`channel:{channel_id}`).
   - Todos los pods con usuarios conectados a dicho canal reciben el evento de Redis y lo envían a los clientes correspondientes.
3. **Manejo de Desconexiones y Recuperación:**
   - Cada mensaje posee un identificador secuencial / timestamp.
   - El cliente localmente almacena el `last_message_id_seen`. Al reconectarse, envía un handshake solicitando los mensajes ocurridos desde ese ID, los cuales el servidor recupera de MongoDB.

## Alternativas Descartadas
1. **HTTP Short/Long Polling:**
   - *Descartada:* Sobrecarga masiva de peticiones HTTP, alta latencia e ineficiencia en consumo de batería móvil.
2. **Server-Sent Events (SSE):**
   - *Pros:* Unidireccional simple desde el servidor.
   - *Contras:* No proporciona canal bidireccional integrado para eventos de escritura (typing indicator, confirmaciones).

## Consecuencias
- **Positivas:**
  - Entrega instantánea y escalabilidad horizontal transparente.
  - Sincronización robusta ante cortes de conectividad móvil.
- **Negativas:**
  - Mayor consumo de descriptores de socket y memoria en servidor, requiriendo pruebas de carga específicas con k6.
