# Configuración

Este repositorio contiene la configuración necesaria para la implementación de NFS en VM Ubuntu 22.04.

## Especificaciones

- **AMI**: Se requiere una instancia basada en Ubuntu Server 22.04 LTS.
- **Key Pair**: Utiliza la misma clave PEM que se empleó en el reto 3.
- **VPC**: VPC por defecto que aparece en la configuración.
- **Puertos**: Abre el puerto 22 para SSH y el puerto 2049 para NFS en la configuración de seguridad.

# Paso a Paso de la Configuración

Para establecer el entorno necesario, sigue estos pasos:

1. **Instalación de nfs-kernel-server**

    Ejecuta los siguientes comandos en tu terminal:

    ```bash
    sudo apt update
    sudo apt install nfs-kernel-server -y
    ```

2. **Creación del directorio de WordPress**

    Crea un directorio para WordPress con el siguiente comando:

    ```bash
    sudo mkdir -p /mnt/wordpress
    ```

3. **Cambio de propietario y permisos**

    Ajusta el propietario y los permisos del directorio recién creado:

    ```bash
    sudo chown nobody:nogroup /mnt/wordpress
    sudo chmod 777 /mnt/wordpress
    ```

4. **Modificación de los exports**

    Edita el archivo `/etc/exports` utilizando el editor de texto de tu elección. Añade la siguiente línea:

    ```
    /mnt/wordpress <ip-privada-server>(rw,sync,no_subtree_check)
    ```

5. **Reinicio del servicio de NFS**

    Una vez hecho, reinicia el servicio de NFS con el siguiente comando:

    ```bash
    sudo systemctl restart nfs-kernel-server
    ```

6. **Configuración del firewall**

    Activa el firewall y añade las reglas necesarias:

    ```bash
    sudo ufw enable
    sudo ufw allow nfs
    sudo ufw allow ssh
    ```

Estos pasos te ayudarán a configurar correctamente el entorno para el servicio de NFS en tu sistema.

