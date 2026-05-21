# ISY1101 - EP2 Innovatech Chile
## Contenedorización y CI/CD

Sistema de gestión de ventas y despachos desplegado en AWS con Docker y CI/CD automatizado.

## Tecnologías utilizadas
- Java Spring Boot (Backend Ventas y Despachos)
- React.js (Frontend)
- MySQL 8.0 (Base de datos)
- Docker y Docker Hub
- GitHub Actions (CI/CD)
- AWS EC2 (3 instancias)

## Arquitectura
- EC2 Frontend → React app (puerto 80)
- EC2 Backend → Spring Boot x2 (puertos 8080 y 8081)
- EC2 Data → MySQL (puerto 3306)

## CI/CD Pipeline
Pipeline automatizado con GitHub Actions que:
1. Construye imágenes Docker
2. Las publica en Docker Hub
3. Despliega automáticamente en AWS EC2 vía SSM

## Demo
Frontend en producción: http://3.80.47.21

## Desarrollador
Junior Altidor - Full Stack Developer
Santiago, Chile
