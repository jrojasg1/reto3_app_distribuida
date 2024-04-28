# Configuración

Este repositorio contiene la configuración necesaria para la implementación de Wordpress con Docker en VM Ubuntu 22.04.

## Especificaciones

- **AMI**: Se requiere una instancia basada en Ubuntu Server 22.04 LTS.
- **Key Pair**: Utiliza la misma clave PEM que se empleó en el reto 3.
- **VPC**: VPC por defecto que aparece en la configuración.
- **Puertos**: Abre el puerto 22 para SSH y el puerto 80 para la instancia en la configuración de seguridad.


## Instancias necesarias

Para montar el contenedor de Wordpress, necesitarás tener disponibles las siguientes instancias:

- Una instancia de MySQL.
- Una instancia de NFS.

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

## Montar NFS

Para montar NFS en la instancia, realiza los siguientes pasos:

1. **Instalación de nfs-common:**

    ```bash
    sudo apt update
    sudo apt install nfs-common -y
    ```

2. **Crear el directorio de montaje:**

    ```bash
    sudo mkdir -p /mnt/wordpress
    ```

3. **Montar el directorio:**

    ```bash
    sudo mount <ip_privada_nfs>:/mnt/wordpress /mnt/wordpress
    ```

## Configuración

Utiliza Docker Compose para crear un entorno de Wordpress utilizando contenedores Docker. Crea un archivo llamado `docker-compose.yml` con el siguiente contenido:

```yaml
version: "3.1"
services:
  wordpress:
    container_name: wordpress
    image: wordpress
    ports:
      - 80:80
    restart: always
    environment:
      WORDPRESS_DB_HOST: <ip-privada>
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - /mnt/wordpress:/var/www/html
```

#Despliegue
```bash
    docker-compose up -d
    ```
Esto creará y ejecutará los contenedores necesarios para WordPress. Una vez que el contenedor esté en funcionamiento, podrás acceder a WordPress a través del navegador web utilizando la dirección IP del host y el puerto 80.

Crear otra instancia con las mismas intrucciones de configuración.

