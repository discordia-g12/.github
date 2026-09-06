# ADR 0008: Consistencia Distribuida, Idempotencia y Saga Pattern en Pagos y Monetización

## Estado
Aceptado

## Contexto
Flujos como la compra de paquetes de puntos, la suscripción a planes Premium y la aplicación de Boosts a servidores involucran múltiples microservicios (`Payments Service`, `Server Service`, `Notification Service`) e integraciones externas (Stripe / MercadoPago Sandbox). Ante fallos parciales o caídas de red, no pueden producirse estados inconsistentes (ej. cobrar sin otorgar el boost).

## Decisión
1. **Saga Orquestada para Monetización:**
   - `Payments & Monetization Service` actúa como orquestador de la saga.
   - Si la aplicación de un Boost en `Server Service` falla permanentemente, el orquestador dispara la transacción compensatoria (reversión de saldo de puntos y notificación de error al usuario).
2. **Idempotencia en APIs y Webhooks:**
   - Los endpoints de pago y recepción de webhooks exigen un header `Idempotency-Key` (o ID único de evento del proveedor de pagos).
   - Se almacena la clave en PostgreSQL/Redis durante 24 horas; si se recibe una solicitud repetida, se devuelve la respuesta previa sin re-ejecutar cobros ni créditos.
3. **Clasificación de Errores:**
   - *Transitorios (Timeout de red, HTTP 503):* Se aplican reintentos automáticos con Exponential Backoff y Jitter.
   - *Permanentes (Tarjeta rechazada, servidor no encontrado, saldo insuficiente):* Se cancela la saga y se compensan las acciones previas.

## Alternativas Descartadas
1. **Transacciones Distribuidas 2PC (Two-Phase Commit):**
   - *Descartada:* Bloqueos síncronos prolongados, acoplamiento extremo y falta de soporte nativo entre microservicios heterogéneos y pasarelas externas.
2. **Saga Coreografiada:**
   - *Pros:* Sin coordinador central.
   - *Contras:* Difícil de rastrear y depurar el estado del flujo financiero a medida que crecen los participantes.

## Consecuencias
- **Positivas:**
  - Consistencia garantizada en operaciones monetarias críticas.
  - Resiliencia ante fallos de proveedores de pago externos.
- **Negativas:**
  - Mayor cantidad de estados intermedios que deben modelarse en la interfaz de usuario (`PENDING`, `COMPLETED`, `FAILED`, `REFUNDED`).
