# ADR 0005: Selección y Rol del API Gateway

## Estado
Aceptado

## Contexto
La plataforma cuenta con 3 clientes (Mobile, Web, Backoffice) que deben interactuar de forma segura y uniforme con múltiples microservicios backend. Se requiere un punto de entrada único que centralice la autenticación, la terminación TLS, el enrutamiento y el rate limiting.

## Decisión
Se selecciona **Kong Gateway (Open Source)** (o alternativamente **NGINX Reverse Proxy**) como API Gateway perimetral:

1. **Enrutamiento:** Unifica las APIs bajo una única URL base (`/api/v1/auth`, `/api/v1/servers`, `/api/v1/chat`, `/api/v1/payments`, `/api/v1/notifications`).
2. **Autenticación Centralizada:** Valida la firma del token JWT en el perímetro antes de reenviar el tráfico a los servicios internos. Inyecta encabezados HTTP seguros (`X-User-Id`, `X-User-Role`, `X-Correlation-Id`).
3. **Rate Limiting:** Aplica cuotas de peticiones por IP para endpoints públicos (ej. 5 intentos de login por minuto) y por `X-User-Id` para usuarios autenticados, previniendo ataques de denegación de servicio y fuerza bruta.
4. **Soporte WebSocket (WSS):** Realiza el upgrade de conexiones HTTP a WebSocket hacia el `Chat Engine Service` y `Voice Signaling Service`.

## Alternativas Descartadas
1. **Comunicación Directa Cliente-Microservicio (Direct Client-to-Microservice):**
   - *Descartada:* Expone endpoints internos a internet, multiplica la complejidad de autenticación en cada servicio y genera acoplamiento con la topología interna.
2. **Backend for Frontend (BFF) Dedicado por Plataforma:**
   - *Pros:* Personalización de payloads por tipo de dispositivo.
   - *Contras:* Duplicación de lógica y mantenimiento de 3 gateways independientes para un equipo de 6 personas.

## Consecuencias
- **Positivas:**
  - Los microservicios internos no necesitan re-validar la firma JWT en cada petición, confiando en los headers inyectados por el Gateway en la red privada.
  - Protección perimetral consistente contra abusos.
- **Negativas:**
  - El Gateway representa un punto único de falla (*Single Point of Failure*); en producción se despliega con alta disponibilidad (múltiples réplicas balanceadas).
