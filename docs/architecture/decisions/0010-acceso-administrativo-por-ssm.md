# ADR 0010: Acceso Administrativo por SSM, sin Puertos de Administración Expuestos

## Estado
Aceptado

## Contexto
La plataforma corre en tres EC2 (edge, realtime, infra) más un RDS, en una VPC propia. Además del tráfico de los usuarios, hay tres tipos de acceso administrativo:

- **Operar un nodo:** leer logs, editar `/etc/discordia/nodes.env`, reiniciar contenedores.
- **Desplegar:** el CD de cada servicio y `deploy-config.yml` tienen que ejecutar los scripts de deploy en el nodo.
- **Mirar la observabilidad:** Grafana (3030) y Prometheus (9090) en el nodo infra.

La forma habitual sería abrir el puerto 22 para SSH y los puertos 3030 y 9090 para los paneles, restringidos por IP. Para este equipo eso tiene tres problemas:

- Seis personas con IPs domiciliarias y móviles que cambian: la lista de IPs autorizadas queda desactualizada enseguida, y la salida rápida es abrir a `0.0.0.0/0`.
- Grafana se protege con un único usuario `admin` compartido, sin MFA. Prometheus no tiene autenticación: cualquiera que llegue al 9090 lee todas las métricas y puede lanzar consultas pesadas sobre un nodo de 1 GB.
- Las claves SSH habría que repartirlas, rotarlas y revocarlas a mano cuando alguien deja el proyecto.

## Decisión
1. **Ningún puerto de administración abierto en los Security Groups.** No hay regla para el 22, ni para el 3030 ni para el 9090 (`infra/terraform/security.tf`). Desde internet solo están abiertos el 443 de edge, restringido a los rangos de Cloudflare, y los puertos de media de LiveKit (7881/tcp y 7882/udp), que requieren un token propio.
2. **Shell en los nodos por SSM Session Manager.** Cada EC2 tiene un rol con `AmazonSSMManagedInstanceCore` (`infra/terraform/modules/node/main.tf`). El agente de SSM abre la conexión desde el nodo hacia AWS, así que el nodo no necesita ningún puerto de entrada. Se entra con `aws ssm start-session --target <id>`.
3. **Paneles por port forwarding de SSM.** Grafana y Prometheus publican solo en `127.0.0.1` del nodo infra. Para verlos se abre un túnel con el documento `AWS-StartPortForwardingSession` y se navega a `http://localhost:3030` o `http://localhost:9090`.
4. **El CD también entra por SSM.** GitHub Actions asume un rol por OIDC (sin claves guardadas en GitHub) que solo puede ejecutar `ssm:SendCommand` con `AWS-RunShellScript` sobre las tres instancias (`infra/terraform/iam-cd.tf`).
5. **El tráfico entre nodos se limita por grupo de origen.** Cuando un puerto tiene que publicarse para otro nodo (por ejemplo, los exportadores 9100 y 9180 o las métricas de Kong en el 8100), la regla acepta solo el Security Group del nodo que lo consume, nunca un rango de IPs (ADR-O3 y ADR-O4 de `observability`).
6. **Las interfaces de administración de los componentes no salen del contenedor o del host.** El Admin API de Kong escucha en `127.0.0.1:8001` dentro del contenedor, y el puerto HTTP sin TLS de Kong (8000) solo en `127.0.0.1` del host.

## Alternativas Descartadas
1. **SSH con puerto 22 restringido por IP:**
   - *Descartada:* Exige mantener la lista de IPs de cada integrante y gestionar claves SSH. Además, el puerto queda expuesto a escaneos y a vulnerabilidades del servidor SSH.
2. **Bastion host o VPN (por ejemplo, WireGuard):**
   - *Pros:* Un único punto de entrada auditado, que permite exponer los paneles dentro de la red privada.
   - *Contras:* Una máquina más que pagar, parchear y monitorear, para un problema que SSM ya resuelve sin costo extra.
3. **Exponer Grafana y Prometheus con su propio login:**
   - *Descartada:* La seguridad dependería de una contraseña compartida y de mantener Grafana parcheado. Prometheus no tiene autenticación nativa. Los paneles muestran IPs privadas, nombres de servicios, carga y errores, información útil para planear un ataque.

## Consecuencias
- **Positivas:**
  - Superficie de ataque mínima: no hay puertos de administración que escanear ni que atacar por fuerza bruta.
  - El acceso depende de credenciales de IAM y no de la IP de origen. Revocar a alguien es revocar sus credenciales de AWS.
  - Cada `StartSession` y cada `SendCommand` queda registrado en CloudTrail, con quién lo hizo y cuándo.
  - No hay claves SSH ni bastion que mantener.
- **Negativas:**
  - Para ver Grafana hay que tener el AWS CLI con el `session-manager-plugin` y abrir un túnel. El link "Abrir Grafana" del backoffice apunta a `http://localhost:3030` y solo funciona con el túnel abierto.
  - Hoy el equipo entra con un usuario IAM de permisos amplios, no con un rol acotado a `ssm:StartSession` sobre los tres nodos. Tampoco se guarda el contenido de las sesiones (comandos ejecutados), solo el hecho de que se abrieron.
  - Si alguna vez hace falta que Grafana se vea desde el navegador sin túnel, el camino es ponerlo detrás de un proxy con autenticación (por ejemplo, Cloudflare Access con el login de Auth0), no abrir el puerto.
