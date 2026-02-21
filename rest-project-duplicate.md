# Guía de duplicación del proyecto

## 1. Duplicar la carpeta del proyecto
- Copiar la carpeta original
- Renombrar la nueva carpeta con el nombre del nuevo servicio

## 2. Actualizar la nueva carpeta
- Eliminar archivos innecesarios:
  - `.git`
  - `.idea`
  - `target`
- Actualizar documentación:
  - `README.md`
- Modificar configuración del proyecto:
  - `pom.xml`
    - Cambiar `artifactId`
    - Establecer versión inicial: `1.0.0-SNAPSHOT`
  - `docker-compose.yml`

## 3. Actualizar ficheros de configuración
- Revisar y modificar:
  - `application.yml`
  - `application-dev.yml`
- Actualizar:
  - Nombre del servicio
  - Puertos
  - URLs de conexión
  - Variables de entorno

## 4. Limpiar el código base
- Eliminar clases innecesarias
- Mantener únicamente las clases relacionadas con el nuevo dominio (ej. `User...`)
- Revisar paquetes y estructura

## 5. Crear repositorios

### 5.1 Crear repositorio local
- `git init`
- `git add --all`
- `git commit -m "Initial commit - Goa-???"`

### 5.2 Crear rama principal de desarrollo
- `git checkout -b develop`

### 5.3 Crear repositorio remoto
- Crear nuevo repositorio en GitHub
- Conectar repositorio y subir la rama develop. En principio fallará el action
  - `git remote add origin <new-repository-url>`
  - `git push origin develop`

## 6. Configurar CI – Variables de entorno
- Configurar en GitHub Secrets:
  - `GHCR_PERSONAL_ACCESS_TOKEN`
  - `SONAR_TOKEN`

## 7. Configurar SonarQube
- Crear el proyecto manualmente:
  - Display Name: `goa-*`
  - Project Key: dejar el por defecto (`miw-upm-github_goa-*`)
- Establecer:
  - Rama por defecto: `develop`
  - Formato de ramas a analizar: `(develop|master|release-.*)`
- Ejecutar nuevamente el workflow de GitHub Actions para validar integración

## 8. Configurar CD – Variables de entorno
Configurar en GitHub Secrets:
- `API_CLIENT_ID`
- `API_CLIENT_SECRET`
- `MONGODB_URI`
- `AWS_LIGHTSAIL_IP`
- `AWS_LIGHTSAIL_USER`
- `AWS_LIGHTSAIL_SSH_KEY`
- ... la que fuesen necesarias

## 9. Crear una release
- Crear tag de versión
- Validar despliegue en AWS
  - Ver contenedores activos:
    - `docker ps`
  - Revisar logs:
    - `docker logs -f goa-*`
- Verificar que la aplicación responde correctamente
