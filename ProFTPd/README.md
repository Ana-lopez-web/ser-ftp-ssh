# Parte 1

## Instalación del servidor

Empezamos, como casi siempre, instalando nuestro servicio.

Conviene primero comprobar que todo esté al día:

> `sudo apt update`
> `sudo apt upgrade`

Y luego ya instalamos el servicio:

> `sudo apt install proftpd`

## Configurando el servidor

Para configurar el servidor, debemos modificar el fichero `/etc/proftpd/proftpd.conf`. Deberemos tocar o añadir distintas líneas para lograr lo pedido:

- **Los usuarios no deben poder subir de su directorio personal (cada uno el suyo).** Deben descomentarse la línea `DefaultRoot ~`.
- **Configura el rango de puertos para el modo pasivo del 62000 al 63000.** Descomentar la línea `PassivePorts` y poner bien los puertos.
- **Permite un máximo de 30 conexiones simultáneas, 5 de las cuales pueden ser anónimas.** Por defecto ya tenemos el `MaxInstances` a 30 (¡pero siempre debemos comprobarlo!) y para las conexiones anónimas, deberemos descomentar el bloque de `<Anonymous ~ftp>(...)</Anonymous>` que hay de ejemplo y cambiar la línea de `MaxInstances` a 5.

## Creando y configurando los usuarios

Para los 3 usuarios que tenemos que crear, nos tocará hacer 3 nuevos usuarios de nuestra máquina Ubuntu. Lo ejemplificaremos con el user admin, siendo igual para los 3.

> `sudo adduser admin`

El comando nos pedirá varios campos, como la contraseña. Deberemos comprobar que se crean los directorios home y que se permite loguerse en el sistema.

Nos quedaría, todavía, hacer un paso más de configuración:

**Configura al usuario anónimo para que pueda descargar archivos existentes en el directorio /srv/public del servidor (debe configurar el directorio también).** En el bloque `<Anonymous ~ftp>(...)</Anonymous>` viene definido por defecto que el usuario anónimo será el usuario ftp. Si creamos el directorio `/srv/public` y se lo asignamos como home al usuario ftp, ya estaría configurado.

> `sudo mkdir /srv/public`
> `sudo usermod -m -d /srv/public ftp`

## Crear un certificado auto firmado

Por hacer...
