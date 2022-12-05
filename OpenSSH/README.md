# Parte 2

Para esta parte necesitaremos dos máquinas distintas. Podemos crearlas de nuevo o reutilizarlas desde prácticas anteriores.

Para dejar bien claras estas instrucciones, a la máquina que tiene el usuario ubuntu la llamaremos _CLIENTE_ y a la que tiene el usuario admin la llamaremos _SERVIDOR_. Cuando veas _IP CLIENTE_ o _IP SERVIDOR_ nos referiremos a las IP de cada una de esas máquinas.

## Crear el usuario en _CLIENTE_ y los certificados

En la máquina que hayamos elegido como _CLIENTE_ deberemos crear el usuario Ubuntu (si no lo hemos creado ya en la instalacción).

Llegados a este punto, primero deberemos comprobar si nuestro usuario ya tiene certificados creados en la máquina _CLIENTE_. Para ello, listamos el contenido de del directorio `/home/ubuntu/.ssh/`. Si aparecieran los ficheros y por tanto ya estuvieran creados, pasamos a la siguiente sección.

Si no los tuviera creados, en la máquina _CLIENTE_, estando logueados con el usuario ubuntu, ejecutamos el siguiente comando en la terminal:

> `ssh-keygen`

Nos preguntará dónde queremos guardar la clave, dándonos la opción de dejarlo en blanco para coger la opción por defecto (`/home/ubuntu/.ssh/id_rsa`). 

Nos preguntará también si queremos utilizar una passphrase. Esta es una opción que añade una capa extra de seguridad, haciendo que tengamos que escribir una contraseña cada vez que queremos usar nuestra clave privada, evitando que alguien con acceso físico a nuestra máquina pueda usar nuestra clave. Como es habitual, ese extra de seguridad tiene como precio mayor incomodidad en el uso.

## Configurar el servidor SSH en _SERVIDOR_

Vamos ahora la máquina de _SERVIDOR_. Deberíamos comprobar que está instalado el servidor SSH, porque sin él no podremos conectarnos.

> `ssh -V`

Si nos devuelve una versión, está instalado. Si no, nos tocará ejecutar:

> `sudo apt install openssh-server`

## Copiar la clave al _SERVIDOR_

La manera más fácil de lograr este paso es, desde _CLIENTE_ ejecutar el siguiente comando:

> `ssh-copy-id admin@IP_SERVIDOR`

Este comando, incluido por defecto en las instalaciones de openssh, se encarga de copia nuestra clave pública de la cuenta local al servidor, guardándola en el fichero correspondiente.

Si no estuviera disponible, deberíamos ejecutar los siguientes comandos:

> `cat ~/.ssh/id_rsa.pub | ssh admin@IP_SERVIDOR "cat >> ~/.ssh/authorized_keys"`

Básicamente, leemos el contenido de la clave pública en _CLIENTE_ para escribirlo en el fichero `~/.ssh/authorized_key` en _SERVIDOR_, pues es este fichero el que contiene las claves privadas que pueden loguearse.

## Comprobar que se ha agregado la clave correctamente

Desde _CLIENTE_ ejecutamos el siguiente comando:

> `ssh admin@IP_SERVIDOR`

Si nos pide contraseña, algo ha ido mal. Si nos logueamos correctamente, todo está bien.

## Impedir el acceso SALVO por clave pública/privada

Nos queda ya solo el último paso: deshabilitar el acceso por contraseña para el usuario admin.

Para ello, debemos modificar el fichero de configuración del servidor ssh `/etc/ssh/sshd_config` en _SERVIDOR_ (aunque como ya tenemos conexión SSH habilitada, podemos hacerlo desde cualquier máquina).

En dicho fichero, tenemos dos opciones:

- Lo más directo sería ir a la última línea y modificar el `PasswordAuthentication yes` por un `PasswordAuthentication no`.
- Pero también podemos hacer algo más fino: en el fichero de configuración del servicio ssh podemos poner reglas por usuarios. En el fichero que está en este repositorio se puede ver cómo se haría.

Por último, reiniciamos el servido sshd:

> `sudo systemctl restart sshd`

## Comprobando que solo se permite el acceso por clave pública/privada

La manera más fácil de probar que hemos configurado bien este último requisito es irnos a _CLIENTE_ y desde ahí ejecutar nuestro comando de conexión ssh pero esta vez forzando a que use usuario/contraseña:

> `ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no admin@IP_SERVIDOR`

Debería darnos un error, no pedirnos la contraseña.

Si nos conectamos por las opciones por defecto, deberíamos seguir pudiendo.

