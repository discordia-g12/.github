# ADR 0001: Adopción de Arquitectura de Microservicios y Bounded Contexts

## Estado
Aceptado

## Contexto
El proyecto Discordia requiere modelar una plataforma de comunicación compleja con diversos dominios funcionales: gestión de identidad y usuarios, administración de comunidades (servidores, canales y roles), mensajería en tiempo real de alta concurrencia, canales de voz interactivos con WebRTC, monetización con pagos externos y herramientas de moderación y auditoría para staff.
El equipo está compuesto por 6 integrantes y el enunciado exige el desarrollo de servicios desacoplados con al menos dos lenguajes de programación y bases de datos independientes por servicio, evitando la construcción de un monolito.

## Decisión
Se decide adoptar una **Arquitectura de Microservicios** particionada según los **Bounded Contexts de Domain-Driven Design (DDD)** en 6 servicios autónomos:
1. `Auth & User Service`: Identidad, credenciales, perfiles y estados de usuario.
2. `Server & Roles Core Service`: Servidores, canales, jerarquías de roles y moderación de servidor.
3. `Chat Engine Service`: Mensajería en tiempo real, canales de texto, DMs, historial y WebSockets.
4. `Voice Signaling Service`: Señalización de sesiones de voz y coordinación con la SFU WebRTC.
5. `Payments & Monetization Service`: Suscripciones Premium, paquetes de puntos y boosts.
6. `Notification & Audit Service`: Notificaciones push/in-app, métricas y registro de auditoría global.

Cada servicio posee su propia base de datos (*Database-per-Service*), su ciclo de vida de despliegue independiente (Dockerfile) y se comunica con los demás a través de contratos OpenAPI y eventos asíncronos en RabbitMQ.

## Alternativas Descartadas
1. **Monolito Modular:**
   - *Pros:* Simplicidad de despliegue inicial y transacciones ACID locales.
   - *Contras:* Viola los requisitos explícitos del enunciado (que exige múltiples lenguajes y bases de datos independientes sin monolito), genera acoplamiento temporal y dificulta el escalado independiente del motor de WebSockets frente a servicios CRUD.
2. **Microservicios Hiper-Granulares (Nano-servicios, ej. 12+ servicios):**
   - *Pros:* Aislamiento extremo de funciones individuales.
   - *Contras:* Sobrecarga operativa inmanejable para un equipo de 6 integrantes, latencia excesiva de red y complejidad desmedida en pruebas locales y orquestación.

## Consecuencias
- **Positivas:**
  - Alto desacoplamiento que permite a los pares del equipo trabajar en paralelo sin colisiones de código.
  - Escalado independiente de servicios de alta carga (ej. Chat y Voz) respecto a servicios transaccionales.
  - Cumplimiento estricto de las restricciones del curso.
- **Negativas:**
  - Necesidad de gestionar consistencia eventual y transacciones distribuidas (Sagas).
  - Mayor complejidad de observabilidad y despliegue local (mitigada mediante Docker Compose y tracing distribuido).
