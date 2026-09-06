# ADR 0002: Selección del Stack Tecnológico Backend Políglota

## Estado
Aceptado

## Contexto
El enunciado del trabajo práctico exige que los servicios backend utilicen al menos dos lenguajes de programación distintos. El sistema presenta dos perfiles de carga contrastantes:
1. Servicios transaccionales y de lógica de negocio rica (Auth, Servidores, Permisos, Pagos).
2. Servicios orientados a concurrencia I/O intensiva en tiempo real (Chat WebSockets, Señalización WebRTC, Notificaciones).

## Decisión
Se decide utilizar un stack políglota compuesto por **Python 3.12 (FastAPI)** y **Node.js 20 / TypeScript (Fastify)**:

1. **Python (FastAPI + Pydantic + SQLAlchemy Async):**
   - Utilizado en: `Auth & User Service`, `Server & Roles Core Service` y `Payments & Monetization Service`.
   - *Motivación:* Sintaxis clara, tipado estricto con Pydantic, generación automática de contratos OpenAPI/Swagger, robustez de SQLAlchemy para modelos relacionales complejos y excelente soporte para pruebas automatizadas con Pytest y Schemathesis.
2. **Node.js / TypeScript (Fastify + ws / Socket.IO):**
   - Utilizado en: `Chat Engine Service`, `Voice Signaling Service` y `Notification & Audit Service`.
   - *Motivación:* Event loop no bloqueante ideal para mantener decenas de miles de conexiones WebSockets concurrentes con baja latencia, compatibilidad nativa con SDKs de WebRTC (LiveKit) y velocidad de serialización JSON con Fastify.

## Alternativas Descartadas
1. **Go (Golang):**
   - *Pros:* Alto rendimiento y bajo consumo de memoria.
   - *Contras:* Mayor verbosidad en lógica de negocio compleja, curva de aprendizaje en el equipo y menor velocidad para prototipado rápido en comparación con TypeScript y FastAPI.
2. **Java / Kotlin (Spring Boot):**
   - *Pros:* Ecosistema empresarial maduro.
   - *Contras:* Mayor consumo de memoria en contenedores Docker y arranque más lento para entornos de desarrollo local y PaaS gratuitos.

## Consecuencias
- **Positivas:**
  - Cumplimiento de la restricción de 2 lenguajes backend.
  - Optimización de tecnologías según el patrón de carga de cada servicio.
  - Generación automatizada de OpenAPI 3.0 tanto en FastAPI como en Fastify (Swagger plugin).
- **Negativas:**
  - El equipo debe mantener tooling y linters para dos ecosistemas (Ruff/Bandit en Python, ESLint/Prettier en TypeScript).
