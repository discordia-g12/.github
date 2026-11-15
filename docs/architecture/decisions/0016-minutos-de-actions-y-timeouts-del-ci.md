# ADR 0016: Minutos de GitHub Actions y Timeouts en el CI

## Estado
Aceptado

## Contexto
El CI y el CD de todos los repos corren en GitHub Actions. Cada servicio tiene un `ci.yml` con tres jobs encadenados: `test`, `docker` (construye la imagen y, en un push a `develop` o `main`, la publica en GHCR) y `deploy` (solo en push a `main`, llama al workflow reutilizable del orquestador). Desplegar depende de que Actions funcione (ADR 0010).

La organización `discordia-g12` está en el plan Free y los repos son privados. Con eso tiene **2.000 minutos de Actions por mes, compartidos entre todos los repos**, y el presupuesto de gasto en $0: cuando se terminan, GitHub no cobra, sino que deja de arrancar jobs hasta el día 1 del mes siguiente.

### Qué pasó
El 15/11/2026 llegó el aviso de que se había usado el 100% de los minutos de noviembre. El push siguiente a `develop` de Notification falló en 1 s, sin correr nada, con *"The job was not started because recent account payments have failed or your spending limit needs to be increased"*. Desde ese momento, en ningún repo:

- los PRs se prueban;
- un push a `develop` publica la imagen;
- un merge a `main` despliega.

Los checks no son obligatorios (sin protección de ramas en el plan Free), así que los merges no se bloquean: un merge a `main` deja `main` distinto de lo que corre en producción, sin aviso.

## Análisis
Consumo de noviembre por repo, hasta el corte (minutos de runners Linux):

| Repo | Minutos |
|---|---:|
| Notification-Audit-Metrics-Service | 746 |
| Discordia-WebApp | 598 |
| Chat-Engine-Service | 251 |
| Voice-Signaling-Session-Service | 167 |
| Server-Roles-Core-Service | 128 |
| Auth-User-Service | 108 |
| Gateway | 96 |
| Resto (orquestador, Payments, observability) | 45 |

**Causa principal: un test colgado sin límite de tiempo.** El PR de Dependabot que sube Notification a Fastify 5 (Notification #16) deja colgado un test: el monitor de `status-routes` hace una ronda de más en `onReady` y el proceso nunca termina. Ningún job tenía `timeout-minutes`, así que cada corrida duró el máximo que pone GitHub por defecto, **6 horas**. Hubo tres corridas así (una por cada rebase del PR): **1.080 minutos, más de la mitad del mes**, en un PR que nunca se iba a mergear.

**El resto: la limpieza de Dependabot de esa semana.** Se estaban resolviendo decenas de PRs de Dependabot en los ocho repos. Cada `@dependabot rebase` o `recreate`, y cada merge que deja a los demás PRs del repo desactualizados y obliga a rebasearlos, corre el CI completo, con build de Docker incluido. Además, cada cambio corre el CI dos veces: en el PR y en el push a `develop` del merge. Los PRs de actions llegan de a uno por acción (cinco o seis por repo).

**Cuánto tarda de verdad cada job.** Medido sobre las últimas 15 corridas exitosas de cada repo:

| Job | Corrida más lenta | Promedio de ese repo |
|---|---|---|
| `test` de WebApp | 6,5 min | 5,3 min |
| `test` / `test-and-build` del resto | 3 min (Voice) | 2,6 min |
| `docker` | 2,3 min (Chat) | 1,6 min |

Un job sano nunca pasa de 7 minutos. Cualquier corrida de más de 15 es un cuelgue.

## Decisión
1. **`timeout-minutes` en todos los jobs del CI de cada servicio:**
   - `test` (`test-and-build` en Voice): 15 min. En WebApp, 20 min.
   - `docker`: 15 min.
   - `deploy`: sin cambios en el repo del servicio. Un job que llama a un workflow reutilizable no acepta `timeout-minutes`, y el job del `deploy.yml` del orquestador ya tiene 20 min.

   El margen es de unas 2 a 5 veces el job más lento: alcanza para un runner lento o un cache vacío sin que un cuelgue cueste más de 15 minutos. Se aplicó en `develop` de los ocho repos el 15/11/2026. Llega a `main` con el próximo pase a producción de cada servicio.
2. **Notification #16 (Fastify 5) se cerró sin `@dependabot ignore`.** Necesita una migración manual del monitor de `status-routes`. Sin `ignore`, Dependabot puede volver a ofrecer Fastify 5 cuando se encare esa migración.
3. **Mientras no haya minutos:**
   - No se mergea a `main`: sin el job `deploy`, `main` queda distinto de producción.
   - Se frenan los merges de Dependabot: sin CI no se prueban ni se publica la imagen.
   - Si hay una emergencia, se despliega a mano por SSM con `deploy-service.sh` (ADR 0010), que no usa Actions.
   - Si hace falta desplegar antes del día 1, un owner de la organización pone un presupuesto para Actions en *Settings → Billing → Budgets*. Linux cuesta USD 0,006 por minuto, así que USD 10 son unos 1.600 minutos.

## Alternativas Descartadas

1. **Hacer públicos los repos (minutos ilimitados):**

## Consecuencias

- **Positivas:**
  - Un job colgado cuesta como máximo 15 minutos (20 en el `test` de WebApp), no 360. El caso del 15/11 costaría 45 minutos en lugar de 1.080.
  - El job cortado queda en rojo con *"The job has exceeded the maximum execution time"*, que es más claro que un job cancelado a las 6 h.
  - Queda registrado qué hacer cuando se terminan los minutos y qué no tocar mientras tanto.
- **Negativas:**
  - Sin minutos no hay CD hasta el día 1. El timeout no resuelve el consumo de base.
  - Si un job crece y pasa el límite (por ejemplo, más tests en WebApp), va a fallar por timeout y hay que subir el valor.
- **Pendiente para gastar menos** (no se hizo todavía):
  - `concurrency` con `cancel-in-progress` en el CI de cada repo: un push nuevo cancela la corrida vieja de la misma rama.
  - Agrupar todas las actualizaciones de `github-actions` en un solo PR en el `dependabot.yml` de cada repo.
  - Pasar Dependabot de `weekly` a `monthly`.
  - No construir la imagen en los PRs que solo tocan tests o documentación.
  - Mergear los PRs de Dependabot de a uno y no pedir `rebase` en tanda: cada rebase es una corrida completa.
