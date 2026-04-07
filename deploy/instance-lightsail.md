# Documentación de Implementación: GOA - Despacho de Abogados

## 1️⃣ Crear instancia AWS LightSail

### Especificaciones
- Operating system (OS) only: Ubuntu 22.04 LTS
- General purpose
- 2GB RAM (preferable 4GB) (First 90 days free)
- Establecer nombre: goa
- Crear una IP estática y asignarla (Networking/attach static IP)

### Firewall. Abrir (IPv4 & IPv6)
- SSH TCP 22 (Lightsail browser SSH)
- HTTP TCP 80 (Any IPv4 address)
- HTTPS TCP 443 (Any IPv4 address)

## 2️⃣ Configuración DNS

### Configuración de registros DNS
1. Añadir registro A: `gestion.ocanabogados.es. 0 A 34.246.233.225`
2. Verificar la configuración DNS:
   ```bash
   nslookup gestion.ocanabogados.es
   ```

## 3️⃣ Preparar servidor Ubuntu

### Verificación del sistema
```bash
lsb_release -a
```

### Actualización del sistema
```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

## 4️⃣ Instalar NGINX

### Instalación de NGINX
```bash
sudo apt update
sudo apt install nginx -y
```

### Verificar estado
```bash
sudo systemctl status nginx
```

## 5️⃣ Configuración inicial NGINX (HTTP)
1. Editar el archivo de configuración:
   ```bash
   sudo nano /etc/nginx/sites-available/default
   ```
   
2. Reemplazar el contenido con la siguiente configuración:
   ```nginx
   server {
       listen 80;
       server_name gestion.ocanabogados.es;

       root /var/www/goa-front;
       index index.html;
       server_tokens off;
       autoindex off;

       location ~* \.(php|pl|py|sh|cgi|rb)$ {
           return 403;
       }

       location / {
           try_files $uri $uri/ /index.html;
       }

   	   # /api/goa-document/* -> App Runner (strip /api/goa-document)
       location ^~ /api/goa-document/ {
           proxy_pass https://qxjem7fxjz.eu-west-1.awsapprunner.com/;
           proxy_http_version 1.1;
		   proxy_set_header Host qxjem7fxjz.eu-west-1.awsapprunner.com;
           proxy_ssl_server_name on;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
       location = /api/goa-document {
          return 301 /api/goa-document/;
       }

       location /api/ {
           proxy_pass http://127.0.0.1:8080;
           proxy_http_version 1.1;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header Cookie $http_cookie;
       }
   
	   location /login {
           proxy_pass http://127.0.0.1:8080/goa-user/login;
           proxy_http_version 1.1;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;        
       }

       location /default-ui.css/ {
           proxy_pass http://127.0.0.1:8080/default-ui.css;
           proxy_http_version 1.1;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       }
       location /oauth2/ {
           proxy_pass http://127.0.0.1:8080/oauth2/;
           proxy_http_version 1.1;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header Cookie $http_cookie;
       }
       gzip on;
       gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
   }

   ```
## 6️⃣. Instalación de Certificados SSL

### Instalación de Certbot
```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

### Obtención de certificado SSL
```bash
sudo certbot --nginx -d gestion.ocanabogados.es
j.bernal@upm.es Y ...
```

### Verificación de renovación automática
```bash
sudo certbot renew --dry-run
```

### Verificación de SSL
1. Editar el archivo de configuración y Y comprobar que todo ha ido bien y Cerbot lo ha actualizado
   ```bash
   sudo nano /etc/nginx/sites-available/default
   ```
2. Verificar la configuración y reiniciar NGINX:
   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

## 7️⃣. Desplegar Angular
- Configurar variables de entorno AWS

## 8️⃣. Instalación y Configuración de Docker

## Instalación de Docker
```bash
sudo apt update
sudo apt install -y docker.io 
sudo usermod -aG docker $USER
newgrp docker
```

### Verificación de la versión de Docker
```bash
docker --version
```

### Instalación de Docker Compose v2
```bash
sudo mkdir -p /usr/local/lib/docker/cli-plugins
sudo curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
```

### Verificación de la versión de Docker Compose
```bash
docker compose version
```

### Configuración de red Docker
```bash
# Crear red
docker network create goa

# Verificar redes
docker network ls

# Inspeccionar red
docker network inspect goa
```

## 9️⃣ Optimización del Servidor

### Verificación de memoria disponible
```bash
free -h
```

### Configuración de memoria virtual (swap)
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### Hacer el swap permanente
```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

## Notas adicionales
- Esta configuración establece un servidor para la aplicación GOA de Ocaña Abogados
- NGINX actúa como proxy inverso para el backend que corre en el puerto 8080
- Los certificados SSL se renuevan automáticamente con Certbot
- La aplicación utiliza Docker para su despliegue y gestión
