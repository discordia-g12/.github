# ADR 0009: Arquitectura de Clientes Frontend y Mobile

## Estado
Aceptado

## Contexto
El sistema cuenta con tres clientes: Aplicación Mobile, Cliente Web y Backoffice. Se requiere compartir conceptos de diseño, garantizar una excelente experiencia de usuario con manejo robusto de estados de red (carga, error, reintento) y permitir un desarrollo ágil entre los 6 integrantes.

## Decisión
1. **Cliente Web y Backoffice:**
   - Desarrollados en **React 18 / 19 + TypeScript + Vite**.
   - Gestión de estado asíncrono y caché de servidor con **TanStack Query (React Query)** (proporcionando reintentos automáticos, invalidación de caché y estados `isLoading`/`isError`).
   - Gestión de estado de interfaz global (sesión, canal activo, modales) con **Zustand**.
   - Estilizado con **Tailwind CSS** para un sistema de diseño consistente.
2. **Aplicación Mobile:**
   - Desarrollada en **React Native con Expo (TypeScript)**.
   - Navegación nativa con **React Navigation**.
   - Manejo de conectividad con `@react-native-community/netinfo` para feedback visual ante pérdida de conexión.
3. **Sistema de Diseño Unificado:**
   - Se define una paleta de colores, tipografía y componentes compartidos (Tokens de diseño) documentados en Figma para presentar en el Checkpoint 1.

## Alternativas Descartadas
1. **Redux Clásico:**
   - *Descartado:* Exceso de boilerplate innecesario; TanStack Query + Zustand resuelven el 100% de las necesidades con código mucho más conciso.
2. **Flutter para Mobile:**
   - *Descartado:* Implica aprender Dart; React Native permite compartir conocimiento de TypeScript y React entre frontend web y mobile en el equipo.

## Consecuencias
- **Positivas:**
  - Curva de aprendizaje unificada (todos los integrantes dominan TypeScript y React).
  - Manejo de estados de red resiliente exigido en los RNF.
- **Negativas:**
  - Ciertas librerías nativas en React Native (ej. WebRTC) requieren configuración de builds específicos de Expo Development Client.
