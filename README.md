# ISY1101 - EP3 Innovatech Chile
## Orquestación y Automatización en AWS ECS Fargate

Sistema de gestión de ventas y despachos desplegado en un clúster de contenedores orquestado (AWS ECS + Fargate), con autoscaling, balanceo de carga y pipeline CI/CD totalmente automatizado.

Este proyecto es la evolución directa del EP2 (contenedorización en EC2 sueltas). El EP3 reemplaza ese despliegue manual por un entorno de orquestación real: clúster ECS, Load Balancer, autoscaling y pipeline que despliega solo con cada commit.

## Tecnologías utilizadas

- Java 21 + Spring Boot 3.4 (Backend Ventas y Despachos)
- React 18 + Vite + Nginx (Frontend)
- MySQL 8.0
- Docker (multi-stage builds, usuario no root)
- Amazon ECR (registro de imágenes — antes Docker Hub en EP2)
- AWS ECS + Fargate (orquestación de contenedores — antes EC2 sueltas en EP2)
- Application Load Balancer (balanceo y URL pública estable)
- AWS Application Auto Scaling (Target Tracking por CPU)
- AWS Systems Manager Parameter Store (gestión de secrets)
- GitHub Actions (CI/CD)

## Arquitectura

```
Internet
   │
   ▼
Application Load Balancer (innovatech-alb)
   ├── :80   → tg-frontend   → ECS Service: frontend-service  (1-2 tasks, Fargate)
   ├── :8080 → tg-ventas     → ECS Service: ventas-service     (1-4 tasks, autoscaling)
   └── :8081 → tg-despachos  → ECS Service: despachos-service  (1-4 tasks, autoscaling)
                                        │
                                        ▼
                              EC2 (MySQL 8.0, contenedor Docker, restart:always)
```

- **Clúster:** `innovatech-cluster`, capacity provider `FARGATE`
- **VPC:** default (`172.31.0.0/16`), 2 subredes públicas (`us-east-1a`, `us-east-1b`)
- **Security Groups:** `alb-sg` (público en 80/8080/8081 — ver nota de diseño abajo), `tasks-sg` (interno)
- **IAM:** `LabRole` (cuenta de AWS Academy Learner Lab)

### Nota de diseño: por qué 8080/8081 son públicos en el ALB

El frontend es una SPA de React: las llamadas a la API ocurren desde el navegador del usuario, no desde un servidor. Por eso el ALB necesita exponer también los puertos de los backends, no solo el 80. En un escenario productivo real esto se resolvería mejor con un patrón BFF (Backend for Frontend) o ruteo por path bajo un solo puerto público — quedó documentado como oportunidad de mejora.

## CI/CD Pipeline

Pipeline en `.github/workflows/deploy.yml`, disparado por push a la rama `deploy`:

1. Checkout del código
2. Configura credenciales AWS (Secrets de GitHub)
3. Login a Amazon ECR
4. Build + push de las 3 imágenes (frontend, ventas, despachos)
5. `aws ecs update-service --force-new-deployment` en los 3 servicios

**Nota sobre credenciales:** como el proyecto corre en AWS Academy Learner Lab, las credenciales son temporales (expiran ~4h). Los Secrets `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` y `AWS_SESSION_TOKEN` deben actualizarse en GitHub antes de cada sesión de trabajo o demo.

## Autoscaling

Target Tracking Scaling configurado en `ventas-service` y `despachos-service`:

- **Métrica:** `ECSServiceAverageCPUUtilization`
- **Umbral:** 50%
- **Min capacity:** 1 / **Max capacity:** 4

Probado con carga sostenida (4 minutos de requests concurrentes): el `desiredCount` subió de 1 a 2 automáticamente.

## Gestión de Secrets

La contraseña de la base de datos no está hardcodeada: se almacena en SSM Parameter Store (`/innovatech/db_password`, tipo `SecureString`) y se inyecta en los contenedores vía el campo `secrets` de la Task Definition.

## Problemas encontrados y cómo se resolvieron

| Problema | Causa | Solución |
|---|---|---|
| Health checks fallando (502 en el ALB) | El Target Group revisaba `/` pero la API no responde ahí | Se cambió el health check path a `/api/v1/ventas` y `/api/v1/despachos` |
| ECS reemplazaba las tareas en bucle infinito | Sin grace period, ECS mataba la tarea antes de que Spring Boot terminara de arrancar | Se agregó `--health-check-grace-period-seconds 90` |
| Connection refused a MySQL | El contenedor de MySQL se detuvo cuando la sesión del Lab se reinició | Se reinició el contenedor y se agregó `--restart=always` |
| CORS / ERR_CONNECTION_TIMED_OUT en el navegador | Los puertos 8080/8081 del ALB solo aceptaban tráfico interno, pero el navegador del cliente llama directo | Se abrieron 8080/8081 a `0.0.0.0/0` en `alb-sg` (con la salvedad documentada arriba) |
| Pipeline fallaba con `ecr:GetAuthorizationToken... explicit deny` | La sesión del Lab de Academy había terminado (no solo credenciales vencidas) | Se reinició el Lab, se refrescaron credenciales en local y en GitHub Secrets |
| URLs de API hardcodeadas en el frontend (192.168.x.x) | Restos de configuración de desarrollo local | Se migraron a variables de entorno Vite (`VITE_VENTAS_URL`, `VITE_DESPACHOS_URL`) definidas en `.env`, leídas en build time |
| git push rechazado (non-fast-forward) | La rama `deploy` en GitHub tenía commits que no estaban en local | `git pull` + resolución de conflicto en `deploy.yml` |

## Cómo correr localmente

Cada servicio mantiene su propio Dockerfile. Para levantar todo junto en local:

```bash
docker compose up --build
```

## Documentación técnica completa

Ver la carpeta [`docs/`](./docs) para el detalle completo de arquitectura, contenedorización, CI/CD, infraestructura en la nube, gestión de secretos, seguridad, observabilidad y análisis crítico del proyecto.

## Oportunidades de mejora (trabajo futuro)

- Migrar MySQL a Amazon RDS (gestionado, con backups automáticos)
- Reemplazar el ALB multi-puerto por un patrón BFF o Cloud Map / Service Connect (bloqueado en este proyecto por permisos del Lab de Academy)
- Usar OIDC entre GitHub Actions y AWS en vez de access keys (viable en una cuenta personal, no en el Lab de Academy)
- Acotar las políticas IAM a permisos mínimos en vez de `LabRole`
- Agregar tags de versión (SHA del commit) a las imágenes de ECR para trazabilidad
- Incorporar una etapa de test automatizado en el pipeline

## Demo

- **Frontend:** http://innovatech-alb-142846777.us-east-1.elb.amazonaws.com
- **Repositorio:** github.com/apepejean63-bot/ISY1101-EP2-Innovatech

## Desarrollador

Junior Altidor - Full Stack Developer
Santiago, Chile
