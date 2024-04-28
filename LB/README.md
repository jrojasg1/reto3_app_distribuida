# Configuración

Este repositorio contiene la configuración necesaria para la implementación de balanceador de carga en una VM Ubuntu 22.04.

## Especificaciones

- **AMI**: Se requiere una instancia basada en Ubuntu Server 22.04 LTS.
- **Key Pair**: Utiliza la misma clave PEM que se empleó en el reto 3.
- **VPC**: VPC por defecto que aparece en la configuración.
- **Puertos**: Abre el puerto 22 para SSH, el puerto 80 para http y el puerto 443 para https en la configuración de seguridad.

# Configuración

Para configurar correctamente el sistema, se deben seguir los siguientes pasos:

## Configuración del DNS

Es necesario crear las entradas en el DNS de tu proveedor de domino para los subdominios:

- `reto3.sudominio.com` -> IP Elástica de LB
- `www.reto3.sudominio.com` -> reto3.sudominio.com.

## Instalación y configuración del certbot

1. Primero, es necesario instalar el certbot y otros paquetes necesarios:

```bash
sudo apt update
sudo add-apt-repository ppa:certbot/certbot
sudo apt install letsencrypt -y
sudo apt install nginx -y
```
2. A continuación, se debe generar el certificado utilizando el certbot:

```bash
sudo mkdir -p /var/www/letsencrypt
sudo certbot --server https://acme-v02.api.letsencrypt.org/directory -d *.sudominio.xyz --manual --preferred-challenges dns-01 certonly
```
Sigue las instrucciones proporcionadas por el certbot.Se genera clave y valor para un registro TXT que se debe agregar en el DNS de tu proveedor de dominio, una vez agregado, presina ENTER y se realiza validacion exitosa del certificado.

## Preparación para el Docker Compose

Es necesario crear los directorios y copiar los certificados necesarios:

```bash
mkdir loadbalancer
mkdir loadbalancer/ssl
sudo su
cp /etc/letsencrypt/live/sudominio.com/* /home/ubuntu/loadbalancer/ssl
``` 
## Configuración de Nginx

```bash
nano loadbalancer/nginx.conf
```
Agrega las siguientes configuraciones en el archivo. Cambia `<wordpress_ip_1>` y `<wordpress_ip_2>` por las IPs públicas de tus instancias Wordpress.

```nginx
   worker_processes auto;
   error_log /var/log/nginx/error.log;
   pid /run/nginx.pid;

   events {
      worker_connections 1024;  ## Default: 1024
   }

   http {
      upstream backend {
         server <wordpress_ip_1>;
         server <wordpress_ip_2>;
      }

      server {
         listen 80;
         listen [::]:80;
         server_name _;
         rewrite ^ https://$host$request_uri permanent;
      }

      server {
         listen 443 ssl http2 default_server;
         listen [::]:443 ssl http2 default_server;
         server_name _;

         # enable subfolder method reverse proxy confs
         #include /config/nginx/proxy-confs/*.subfolder.conf;

         # all ssl related config moved to ssl.conf
         include /etc/nginx/ssl.conf;

         client_max_body_size 0;

         location / {
               proxy_pass http://backend;
               proxy_redirect off;
               proxy_set_header Host $host;
               proxy_set_header X-Real-IP $remote_addr;
               proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
               proxy_set_header X-Forwarded-Host $host;
               proxy_set_header X-Forwarded-Server $host;
               proxy_set_header X-Forwarded-Proto $scheme;
         }
      }
   }
```
Luego, crea el archivo de configuración de SSL:
```bash
nano loadbalancer/ssl.conf
``` 

con las siguientes instrucciones:

```nginx
   ## Version 2018/05/31 - Changelog: https://github.com/linuxserver/docker-letsencrypt/commits/master/root/defaults/ssl.conf

   # session settings
   ssl_session_timeout 1d;
   ssl_session_cache shared:SSL:50m;
   ssl_session_tickets off;

   # Diffie-Hellman parameter for DHE cipher suites
   # ssl_dhparam /etc/nginx/ssl/ssl-dhparams.pem;

   # ssl certs
   ssl_certificate /etc/nginx/ssl/fullchain.pem;
   ssl_certificate_key /etc/nginx/ssl/privkey.pem;

   # protocols
   ssl_protocols TLSv1.1 TLSv1.2;
   ssl_prefer_server_ciphers on;
   ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';

   # HSTS, remove # from the line below to enable HSTS
   add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;

   # OCSP Stapling
   ssl_stapling on;
   ssl_stapling_verify on;

   # Optional additional headers
   #add_header Content-Security-Policy "upgrade-insecure-requests";
   #add_header X-Frame-Options "SAMEORIGIN" always;
   #add_header X-XSS-Protection "1; mode=block" always;
   #add_header X-Content-Type-Options "nosniff" always;
   #add_header X-UA-Compatible "IE=Edge" always;
   #add_header Cache-Control "no-transform" always;
   #add_header Referrer-Policy "same-origin" always;
```

## Configuración del Docker Compose

Crea el archivo de Docker Compose en el directorio loadbalancer:

```bash
nano loadbalancer/docker-compose.yml
```
agrega estás instrucciones

```yaml
   version: "3.1"
   services:
   nginx:
     container_name: nginx
     restart: always
     image: nginx
     volumes:
       - ./nginx.conf:/etc/nginx/nginx.conf:ro
       - ./ssl:/etc/nginx/ssl
       - ./ssl.conf:/etc/nginx/ssl.conf
     ports:
       - 80:80
       - 443:443
```

Se debe detener los proceso de nginx en la instancia:

```bash
   ps ax | grep nginx
   netstat -an | grep 80

   sudo systemctl disable nginx
   sudo systemctl stop nginx
   exit
```

## Despliegue

```bash
cd loadbalancer
sudo docker-compose up -d
```

Accede al sitio con reto3.tudominio.xyz para visualizarlo. Agrega algún cambio en el sitio si quieres ver que el balanceador distribuya la carga correctamente, puedes mostrar la IP de la instancia donde se está ejecutando.





