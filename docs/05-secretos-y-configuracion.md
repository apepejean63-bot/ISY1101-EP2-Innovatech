# 05. Gestión de Secretos y Configuración

## Principio aplicado: mínimo privilegio

La credencial de la base de datos MySQL (contraseña) **nunca se escribe en texto plano** en el código fuente, en el Dockerfile, ni como variable de entorno plana en la Task Definition. En su lugar:

1. La contraseña se almacena en **AWS SSM Parameter Store** en la ruta **`/innovatech/db_password`**, como parámetro tipo **`SecureString`** (cifrado en reposo).
2. En la **Task Definition** de ECS, el parámetro se referencia mediante la sección `secrets`, no `environment`. ECS se encarga de inyectarlo de forma segura en el contenedor al momento de arrancar la tarea, sin que quede expuesto en logs ni en la consola.

```json
"secrets": [
  {
    "name": "DB_PASSWORD",
    "valueFrom": "arn:aws:ssm:us-east-1:<account-id>:parameter/innovatech/db_password"
  }
]
```

> **Acción pendiente:** reemplaza `<account-id>` con el ARN real (puedes ocultarlo en la presentación pública, pero debes tenerlo a mano para la defensa).

## Credenciales de AWS en el pipeline

Las credenciales usadas por GitHub Actions para autenticarse contra AWS (Access Key, Secret Key y Session Token, propios del entorno AWS Academy Learner Lab) se almacenan como **GitHub Secrets** del repositorio, y se referencian en el workflow mediante `${{ secrets.AWS_ACCESS_KEY_ID }}`, etc. Nunca se exponen en el código ni en los logs del pipeline.

**Limitación conocida:** al ser un entorno académico (Learner Lab), las credenciales expiran cada 4 horas junto con la sesión del laboratorio, lo que requiere refrescarlas manualmente en GitHub Secrets. Este es un problema real documentado (ver `08-analisis-critico-y-lecciones.md`), no un error de diseño del pipeline.

## Variables de entorno del frontend

Las URLs de las APIs de `ventas-service` y `despachos-service` se configuran mediante las variables de entorno de Vite **`VITE_VENTAS_URL`** y **`VITE_DESPACHOS_URL`**, definidas en `.env` y leídas en **tiempo de build**. Esto reemplazó una configuración previa con URLs de desarrollo local hardcodeadas (`192.168.x.x`), corregida durante el desarrollo del proyecto.

## Proyección a un entorno productivo real

En una cuenta AWS propia (no académica), se recomienda:

- Sustituir las Access Keys estáticas de GitHub Actions por **autenticación OIDC** entre GitHub Actions y AWS, eliminando la necesidad de credenciales de larga duración.
- Sustituir el `LabRole` (rol genérico de uso exclusivamente académico) por **roles IAM acotados** con permisos mínimos específicos para cada servicio (principio de mínimo privilegio real, no solo a nivel de secretos sino también de permisos de ejecución).
