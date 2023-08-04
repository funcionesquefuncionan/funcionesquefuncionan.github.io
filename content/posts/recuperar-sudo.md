---
title: "Recuperar sudo"
date: 2023-08-02T14:26:14-03:00
draft: false
---

El otro día se me apagó la compu por una falla eléctrica. Después de
un rato (porque no andaba inmediatamente) reinició. Pero al poco
tiempo noté que ya que no tenía sudo. O sea, el shell me decía, para
mi sorpresa:

```
pgassendi is not in the sudoers file
This incident has been reported to the administrator
```

Es interesante que efectivamente el incidente había sido reportado al
administrador, o sea yo. Sólo que, paradojicamente, había perdido al
parecer algunos privilegios de administrador.

No sé qué fue lo que paso. Como justo había pasado lo de la
electricidad supuse que al apagarse de forma `ungracefully` la pc
podría tener que ver con eso. Pero la vedad que ni si quiera se hace
cuanto que había perdido sudo (no más de un día, supongo, por la
frecuencia en que los uso).

Pero el tema era recuperarlo y por suerte es bastante facil y aparece
tan solo googleando pero lo resumo acá de todas formas porque me
molesta bastante que casi todas (si no todas) las páginas de hoy están
llenas de popups, cookies, publicidades por doquier. Y yo a esta
altura ya tengo hasta nostalgia por las páginas de los noventa,
escritas en html y sin tantos carteles como la calle Lugones. Bueno,
también tenemos stack overflow askubuntu etc. que están bien, pero
igual me pintó (después de todo está lleno de páginas que reproducen
una y otra vez lo mismo). Y esto por lo menos me funcionó a mí y me da
fiaca loguearme solo para upvotar el post específic oque me sirvió a
mi.

Por otra parte, si bien mi usuario tenía sudo (o estaba en el grupo sudo),
sin embargo yo no tenía la contraseña de `root`. Esto es por como viene configurado 
ubuntu y [se explica acá](https://help.ubuntu.com/community/RootSudo). La idea es 
que las "tareas administrativas" las haga un usuario que tenga sudo, como era mi caso,
pero que la cuenta `root` no se usa. Pero se puede habilitar, teniendo `sudo`,
simplemente cambiando la contraseña de `root`:

```
sudo passwd root
```

Pero para eso tengo que primero recuperar `sudo`.

Entonces,
[en resumen, siguiendo esto](https://askubuntu.com/questions/1229628/my-user-was-deleted-from-sudo-group)
, lo que hice:
- Prender la computadora
- Apretar `Esc` mientras empieza a bootear para ir al GRUB (en otros casos puede ser
  que haya que apretar `Shift` u otra cosa).
- Elegir `advanced options`
- Elegir una opción con `(recovery mode)`. Después de esto esperar que
  bootea,
- Elegir `drop to root shell prompt`
- Al ver que dice `press Enter for maintenance` apretar `Enter`
- Escribir el comando: `mount -o rw,remount /`
- Finalmente: `sudo adduser $MI_USUARIO sudo`

y listo.

Vi después (recién mientras buscaba el post que askubuntu que cito
arriba) 
[otra que decía como cambiar la contraseña de root](https://www.solucionex.com/blog/recuperar-la-contrasena-de-root-en-ubuntu).
No lo probé, pero lo pego acá para probar otro día a ver si funciona. En ese caso la idea sería
cambiar directamente la contraseña de `root` para con ella agragar el usuario que quiera a
`sudo` (con `adduser`).
