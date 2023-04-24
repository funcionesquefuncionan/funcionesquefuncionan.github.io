---
title: "Configurar vsftpd"
date: 2023-04-24T15:16:19-03:00
draft: false
---

`vsftp` es un servidor de ftp. En este post no voy a ir por todas las opciones de configuración sino simplemente una particular que es bastante directa y anda bien.

Obviamente primero hay que instalar `vsftpd`, por ejemplo en Ubuntu:

```
sudo apt install vsftpd
```

Vamos a configurar editando el archivo

```
/etc/vsftpd.conf
```

del cual mejor hacer una copia de backup antes.


Para que sea seguro vamos a hacer que un solo usuario del sistema pueda acceder al servidor por ftp. Ese usuario va a tener acceso sólo a un directorio específico (incluyendo sus subdirectorios, claro) y no va a poder subir a `/home` ni a ver los home de otros usuarios (en realidad ni a su propio home va a poder acceder, porque sólo va a ver un subdirectorio de su home).


Editamos entonces `/etc/vsftpd.conf`

```
sudo vim /etc/vsftpd.conf
```

Seteamos los siguientes flags:

```
# Allow anonymous FTP? (Disabled by default).
anonymous_enable=NO
#
# Uncomment this to allow local users to log in.
local_enable=YES
```

Esos son para que no pueda nadie acceder si no es un usuario local del sistema, o sea se tiene que loguear con el usuario y la contraseña del sistema.

Si queremos poder escribir (con ese usuario):
```
write_enable=YES
```

Ahora vamos a restringir al usuario en particular. Primero seteando:
```
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO
```

Con lo cual estamos haciendo que únicamente los usualos presentes en `/etc/vsftpd.userlist` puedan acceder (nosotros vamos a incluir uno sólo, como dije).

El flag `userlist_deny` es para cambiar el comportamiento con respecto a `userlist_file`. Si ambos `userlist_deny` y `userlist_enable` son `YES` se admiten todos los usuarios locales menos los que estén en la lista (lo que no sean usuario son rechazados por el flag `anonimous_enable=NO`).

Si en cambio (como es nuestro caso) tenemos `userlist_enable=YES` y `userlist_deny=NO`, entonces sólo los que estén en la lista van a poder ser autorizados.

Supongamos que le ponemos al usuario el nombre `ftpuser` (es muy feo el nombre, ustedes busquen otro).

Primero hay que crearlo, si es que no existe (esto es ajeno a vsftpd pero lo pongo acá por si alguien no se acuerda el comando y le ahorro googlearlo):
```
sudo adduser ftpuser
```

Y vamos a crear la carpeta en cuestión, y acá también bien un punto importante. El punto es que el directorio donde vamos a mandar al usuario no tiene que poder ser escrito por él, es decir, no tiene que tener permisos de escritura en el "root" de ftp. Pero como no queremos que un usuario no tenga permisos de escritura en su propio `$HOME` vamos a cambiar el "root" para que sea otro, el cual no pueda escribir.

```
sudo mkdir /home/ftpuser/ftp
sudo chown nobody:nogroup /home/ftpuser/ftp
```

Y para que pueda usar ese directorio vamos a hacerle un subdirectrio donde pueda subir cosas etc.

```
mkdir /home/ftpuser/ftp/files
sudo chown ftpuser:ftpuser /home/ftpuser/ftp/files
```

y finalmente vamos a configurar `vsftpd` para que use ese directorio como root:

(otra vez editamos `/etc/vsftpd.conf`)

```
user_sub_token=$USER
local_root=/home/$USER/ftp
```

Y listo. Ahora vamos a habilitar el servidor:
```
sudo systemctl start vsftpd
```

si es que no estaba corriendo y 
```
sudo systemctl restart vsftpd
```
si sí lo estaba.

Testeamos con 

```
ftp -p 0.0.0.0 # acá poner la dirección del servidor, obvio.
```

A lo cual le sigue poner nombre de usuario y contraseña etc:

```
Connected to 0.0.0.0.
220 (vsFTPd 3.0.3)
Name (203.0.113.0:default): ftpuser
331 Please specify the password.
Password: your_user's_password
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

Después pueden por ejemplo usar un cliente ftp en el celular para compartir archivos desde ahí a la compu y viceversa. Ambos tiene que estar, por su puesto, en la misma red. Por ejemplo, si están en la misma red wifi buscar la dirección ip con

```
ip a | grep inet
```
o buscar con un método [más espefíco acá](https://stackoverflow.com/questions/13322485/how-to-get-the-primary-ip-address-of-the-local-machine-on-linux-and-os-x).

Bueno esto sirve para tenerlo andando y usarlo, pero 
[acá hay más información](https://www.digitalocean.com/community/tutorials/how-to-set-up-vsftpd-for-a-user-s-directory-on-ubuntu-20-04), por ejemplo como usar ssl.
