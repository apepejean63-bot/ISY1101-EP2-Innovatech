# Documentación Técnica — Proyecto Innovatech Chile

**Asignatura:** Introducción a Herramientas DevOps (ISY1101)
**Evaluación:** Examen Final Transversal (EFT) — Entrega de Encargo con Presentación
**Autor:** Junior Altidor
**Institución:** DuocUC, 2026

## Descripción general

Innovatech Chile es una plataforma compuesta por tres microservicios (frontend, ventas y despachos) y una base de datos relacional MySQL, cuyo ciclo completo de integración y entrega continua (CI/CD) está automatizado con GitHub Actions y desplegado en un entorno de producción orquestado sobre **Amazon ECS Fargate**.

El proyecto evolucionó desde un despliegue manual en instancias EC2 (EP2) hacia una arquitectura completamente orquestada, con balanceo de carga, autoscaling real y despliegue automatizado (EP3 → EFT).

## Índice de documentos

| Documento | Contenido |
|---|---|
| [01-arquitectura.md](./01-arquitectura.md) | Arquitectura general del sistema, componentes, comunicación entre servicios, elección del proveedor cloud |
| [02-contenedores.md](./02-contenedores.md) | Dockerfiles, docker-compose, buenas prácticas de contenedorización |
| [03-cicd.md](./03-cicd.md) | Pipeline de GitHub Actions: build, test, push, deploy |
| [04-infraestructura-nube.md](./04-infraestructura-nube.md) | VPC, subredes, security groups, ALB, ECS/Fargate, autoscaling |
| [05-secretos-y-configuracion.md](./05-secretos-y-configuracion.md) | Gestión de secretos, variables de entorno, principio de mínimo privilegio |
| [06-seguridad.md](./06-seguridad.md) | Endurecimiento de imágenes, exposición mínima de puertos, reglas de seguridad |
| [07-observabilidad.md](./07-observabilidad.md) | Logs, métricas, evidencias de monitoreo |
| [08-analisis-critico-y-lecciones.md](./08-analisis-critico-y-lecciones.md) | Problemas reales enfrentados, soluciones aplicadas, proyección a producción |

## Enlaces del proyecto

- **Repositorio:** `github.com/apepejean63-bot/ISY1101-EP2-Innovatech`
- **Demo (Frontend vía ALB):** `innovatech-alb-142846777.us-east-1.elb.amazonaws.com`

> **Nota:** reemplaza estos enlaces si tu repositorio o el DNS del Load Balancer cambiaron desde la EP3.
