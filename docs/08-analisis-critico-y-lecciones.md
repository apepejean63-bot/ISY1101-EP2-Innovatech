# 08. Análisis Crítico y Lecciones Aprendidas

## Problemas reales enfrentados y su solución

| Problema | Causa | Solución aplicada |
|---|---|---|
| Error 502 en el ALB | El health check revisaba `/` en lugar de un endpoint real de la API | Se cambió el health check a `/api/v1/ventas` y `/api/v1/despachos` |
| Ciclo infinito de reemplazo de tareas | Sin *grace period*, ECS mataba la tarea antes de que Spring Boot terminara de arrancar | Se configuró `health-check-grace-period-seconds = 90` |
| `Connection refused` a MySQL | El contenedor de MySQL se detuvo al reiniciarse la sesión del Lab | Reinicio del contenedor + política `restart: always` |
| CORS / `ERR_CONNECTION_TIMED_OUT` en el navegador | Los puertos 8080/8081 del ALB solo aceptaban tráfico interno, pero el navegador del cliente llama directo a las APIs (arquitectura SPA) | Se abrieron 8080/8081 a `0.0.0.0/0` en `alb-sg` (con la limitación de diseño documentada) |
| Pipeline con error de IAM (ECR deny) | La sesión del AWS Academy Learner Lab había terminado, no solo las credenciales | Reinicio del Lab + refresco de Secrets en GitHub |
| URLs de API hardcodeadas en React | Restos de configuración de desarrollo local (`192.168.x.x`) | Variables de entorno `VITE_VENTAS_URL` / `VITE_DESPACHOS_URL` leídas en build time |
| `git push` rechazado (non-fast-forward) | La rama `deploy` en GitHub tenía commits que no estaban en local | `git pull` + resolución de conflicto en `deploy.yml` |

Este análisis es importante para la defensa: demuestra **comprensión real del sistema**, no solo que "funciona", que es exactamente lo que exige la rúbrica (IE10 — Defensa técnica, e IE5 — Verificación y funcionalidad).

## Lecciones aprendidas y proyección a un escenario productivo real

1. **Base de datos gestionada:** migrar MySQL de un contenedor en EC2 a **Amazon RDS**, ganando backups automáticos y alta disponibilidad multi-AZ.
2. **Comunicación interna real:** reemplazar el patrón actual (ALB multi-puerto) por **AWS Service Connect** o un patrón *Backend for Frontend (BFF)* — no se implementó en este entorno por restricciones de permisos del Lab académico, no por decisión de diseño.
3. **Identidad sin claves estáticas:** en una cuenta AWS propia, usar **OIDC** entre GitHub Actions y AWS en lugar de access keys que expiran cada 4 horas.
4. **Permisos de mínimo privilegio real:** sustituir el `LabRole` genérico (de uso exclusivamente académico) por **roles IAM acotados** a cada servicio específico.

## Conclusión del proyecto

- Clúster ECS Fargate operativo con 3 microservicios desacoplados.
- Autoscaling automático verificado con carga real (1 → 2 tareas).
- Pipeline CI/CD que despliega solo con cada commit, sin pasos manuales.
- Arquitectura documentada, con sus límites reconocidos y su camino claro hacia un entorno productivo real.

Esta sección es la más valiosa para diferenciarse en la nota de "profesionalismo": no basta con decir "funcionó", hay que mostrar que se entendieron los problemas y sus causas raíz.
