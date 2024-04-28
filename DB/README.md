# Configuración

Este repositorio contiene la configuración necesaria para la implementación de base de datos MySQL con Docker en una VM Ubuntu 22.04.

## Especificaciones

- **AMI**: Se requiere una instancia basada en Ubuntu Server 22.04 LTS.
- **Key Pair**: Utiliza la misma clave PEM que se empleó en el reto 3.
- **VPC**: VPC por defecto que aparece en la configuración.
- **Puertos**: Abre el puerto 22 para SSH y el puerto 3306 para la instancia en la configuración de seguridad.

## Instalación de Docker y Docker Compose

Para instalar Docker y Docker Compose en la instancia de Ubuntu, sigue estos pasos:

1. **Instalación de Docker:**

    ```bash
    sudo apt update
    sudo apt install docker.io -y
    sudo apt install docker-compose -y
    sudo systemctl enable docker
    sudo systemctl start docker
    ```

2. **Agregar el usuario al grupo de Docker:**

    ```bash
    sudo usermod -aG docker $USER
    ```

Luego, cierra la sesión y vuelve a iniciarla.

## Configuración

Utiliza Docker Compose para crear un entorno de MySQL utilizando contenedores Docker. Crea un archivo llamado `docker-compose.yml` con el siguiente contenido:

```yaml
version: "3.1"
services:
  db:
    image: mysql:5.7
    restart: always
    ports:
      - 3306:3306
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: "1"
    volumes:
      - db:/var/lib/mysql
  volumes:
    - db:
```
#Despliegue
```bash
    docker-compose up -d
```
Esto creará y ejecutará los contenedores necesarios para MySQL. Una vez que el contenedor esté en funcionamiento, podrás vincular la ip privada al WordPress a través del archivo `docker-compose.yml` creado para desplegar el sitio wordpress.


