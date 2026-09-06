# ADR 0007: Selección de Tecnología y Topología para Canal de Voz (WebRTC SFU)

## Estado
Aceptado

## Contexto
El canal de voz es la funcionalidad de mayor riesgo técnico del proyecto. Debe permitir que múltiples usuarios en un servidor o llamada privada hablen en vivo, silencien su micrófono, vean quién está hablando y compartan pantalla/video.

## Decisión
Se decide adoptar una **Topología SFU (Selective Forwarding Unit)** utilizando **LiveKit** (gestionado / contenedor Docker):

1. **Topología SFU vs Mesh P2P:**
   - La SFU recibe un único stream de audio/video por participante y lo redistribuye a los demás sin recodificar.
   - En dispositivos móviles, esto reduce el ancho de banda de subida de $O(N)$ a $O(1)$, haciéndolo viable para salas de 10+ participantes.
2. **Señalización y Seguridad:**
   - `Voice Signaling Service` valida los permisos de rol del usuario y genera un **LiveKit Room Token** (JWT con claims de sala y permisos de publicación de tracks).
   - El cliente móvil (con `react-native-webrtc` / LiveKit React Native SDK) y el cliente web (con LiveKit JS SDK) se conectan directamente a la SFU utilizando dicho token.
3. **Presencia y Eventos:**
   - El estado de "quién está hablando" (*speaking indicator*) y muteos se gestiona mediante eventos de audio-level provistos nativamente por el SDK de la SFU.

## Alternativas Descartadas
1. **Malla P2P (Peer-to-Peer Mesh):**
   - *Pros:* Sin costo de servidor de medios.
   - *Contras:* Escala $N \cdot (N-1) / 2$ conexiones. Para más de 3 participantes en mobile, colapsa el CPU y la batería del dispositivo.
2. **MCU (Multipoint Control Unit):**
   - *Pros:* Mezcla todo el audio en 1 solo stream saliente.
   - *Contras:* Requiere transcodificación pesada en servidor, con altísimo consumo de CPU y latencia agregada.
3. **Mediasoup Auto-hospedado desde Cero:**
   - *Pros:* Máximo control de bajo nivel en Node.js/C++.
   - *Contras:* Exige configurar manualmente puertos UDP, servidores STUN/TURN (coturn) y orquestación de sockets nativos, aumentando drásticamente el riesgo de retraso en los checkpoints.

## Consecuencias
- **Positivas:**
  - Reducción drástica del riesgo técnico manteniendo una arquitectura moderna de nivel industrial.
  - Excelente soporte multiplataforma (Web y React Native).
- **Negativas:**
  - Dependencia de un componente especializado de infraestructura que debe desplegarse en el entorno de producción o consumirse en cloud tier.
