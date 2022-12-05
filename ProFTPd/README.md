# Parte 1

## Instalación del servidor

Empezamos, como casi siempre, instalando nuestro servicio.

Conviene primero comprobar que todo esté al día:

> `sudo apt update`
> 
> `sudo apt upgrade`

Y luego ya instalamos el servicio:

> `sudo apt install proftpd`

## Configurando el servidor

Para configurar el servidor, debemos modificar el fichero `/etc/proftpd/proftpd.conf`. Deberemos tocar o añadir distintas líneas para lograr lo pedido:

- **Los usuarios no deben poder subir de su directorio personal (cada uno el suyo).** Deben descomentarse la línea `DefaultRoot ~`.
- **Configura el rango de puertos para el modo pasivo del 62000 al 63000.** Descomentar la línea `PassivePorts` y poner bien los puertos.
- **Permite un máximo de 30 conexiones simultáneas, 5 de las cuales pueden ser anónimas.** Por defecto ya tenemos el `MaxInstances` a 30 (¡pero siempre debemos comprobarlo!) y para las conexiones anónimas, deberemos descomentar el bloque de `<Anonymous ~ftp>(...)</Anonymous>` que hay de ejemplo y cambiar la línea de `MaxClients` a 5.

## Creando y configurando los usuarios

Para los 3 usuarios que tenemos que crear, nos tocará hacer 3 nuevos usuarios de nuestra máquina Ubuntu. Lo ejemplificaremos con el user admin, siendo igual para los 3.

> `sudo adduser admin`

_Nota: utilizar `useradd` hace el camino más costoso, pues toca hacer más pasos, como crear a mano el directorio home y darle los permisoso adecuados._

El comando nos pedirá varios campos, como la contraseña. Deberemos comprobar que se crean los directorios home y que se permite loguerse en el sistema.

Nos quedaría, todavía, hacer un paso más de configuración:

**Configura al usuario anónimo para que pueda descargar archivos existentes en el directorio /srv/public del servidor (debe configurar el directorio también).** En el bloque `<Anonymous ~ftp>(...)</Anonymous>` viene definido por defecto que el usuario anónimo será el usuario ftp. Si creamos el directorio `/srv/public` y se lo asignamos como home al usuario ftp, ya estaría configurado.

> `sudo mkdir /srv/public`
> 
> `sudo usermod -m -d /srv/public ftp`

## Crear un certificado auto firmado

Primero, debemos crear los certificados. Con openssl se incluye una herramienta para hacerlo:

> `sudo openssl req -x509 -newkey rsa:2048 -keyout /etc/ssl/private/proftpd.key -out /etc/ssl/certs/proftpd.crt -nodes -days 365`

Además, deberemos cambiar los permisos (sobre todo, para la clave privada)

>`sudo chmod 600 /etc/ssl/certs/proftpd.crt`
>
>`sudo chmod 600 /etc/ssl/private/proftpd.key`

Llegados a este punto, deberemos modificar **tres** ficheros:

- `/etc/proftpd/proftpd.conf` para descomentar la línea del `Include /etc/proftpd/tls.conf`
- `/etc/proftpd/tls.conf` para descomentar las líneas necesarias de dentro del fichero.
- `/etc/proftpd/modelus.conf` para que se cargue el módulo `mod_tls.c` al que se hace referencia en el fichero anterior.

Este último fichero, además, nos indica que deberemos instalar un paquete más de proftpd:

> `sudo apt install proftpd-mod-crypto`

Reiniciamos el servicio y probamos con Filezilla que todo vaya bien. Deberíamos recibir, la primera vez, un aviso preguntando si confiamos en el certificado. Deberemos comprobar que los datos del certificado se corresponden con los datos del certificado que hemos creado antes.