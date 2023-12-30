---
title: "Rerust. libcurl y la vida de los lenguajes"
date: 2023-12-30T12:13:34-03:00
draft: false
---

> The rewrite-it-in-rust mantra is mostly repeated by rust fans and people who think this is an easy answer to fixing the share of security problems that are due to C mistakes. Typically, the kind who has no desire or plans to participate in said venture.
> 
> Daniel Stenberg, [Making it harder to do wrong](https://daniel.haxx.se/blog/2023/12/13/making-it-harder-to-do-wrong/)

La nota linkeada en la cita de arriba es del principal programador de curl.  Habla de las
vulerabilidades que fueron apareciendo *durante la vida de curl*, en qué medida son
atribuíbles al lenguaje de programación usado (C) y la idea de reescribir libcurl en otro
lenguaje más seguro.

Al final se lee un comentario de alguien que pasó:

> It’s time to grow up and start Rust rewrite for Curl.

A lo cual responde 

>  go ahead, there’s nothing preventing you!

Quizá ni leyó la nota, o quizá quería sólo molestar (como suele pasar en lugares como
twitter), porque desde el principio aclara que él *no va a reescribir curl* y explica por qué.
Reescribir un proyecto tan enorme, en primer lugar, trae aparejado un montón de problemas
nuevos. Se tardaría años hasta que el proyecto nuevo tenga un nivel similar al de curl
hoy. Además, durante los años iniciales en los que el proyecto esté en beta, el proyecto
en C seguiría, duplicando los esfuerzos necesarios por el equipo, que por otra parte es un
equipo experto en C, no necesariamente va a ser igual de eficiente en cualquier otro
lenguaje de programación.

Obviamente que no niega los problemas intrínsecos al uso de C, y hace una análisis al
respecto. Al pasar hace una observación interesante: mientras que en muchas fuentes online
se dice que el código en C y en C++ la proporción de los problemas debidos a la falta de
*memory-safety* está en el rango de los 60-70%, en curl las fallas debidas a C nunca
estuvieron arriba del 50%, tomadas por el momento en que fueron *introducidas*, mientas
que si se calcula la proporción por el momento en que fueron reportadas se observa que,
por ejemplo, en 2010, estuvieron sobre el 60%. Esto de por sí no dice mucho, y él mismo
observa para matizar que es el análisis de un único proyecto, pero muestra quizá que hay
involucrada una tendencia de los programadores a poner la lupa en estos defectos de C que
ayudó a mejorar muchas vulnerabilidades de este tipo.

Lista, además, algunos puntos que le parece que vale la pena resaltar.

Áreas problemáticas:
1. **strings** sin restricciones de tamaño que pueden causar *overflows* en enteros,
1. **reallocs**, en particular sin restricciones de tamaño y asociado a overflows de 32
1. bits,
1. copias de memoria y de strings, antecedidas de *malloc* 
1. *strncpy* es en sí misma complicada por el
  [padding](https://en.cppreference.com/w/c/string/byte/strncpy) y porque puede generar
  string que no terminen en null.

Para prevenir errores usan algunas funciones auxiliares que tiene en cuenta:
1. restricciones generales de la longitud en las strings pasadas a la API de libcurl, así
   como limites para todas las creadas internamente.
1. evitar reallocs usando buffers dinámicos (siempre es bueno evitar pedir memoria).
1. de ser posible, usar un puntero (junto con el tamaño de la memoria apuntada) en ver de
1  una copia. Si es necesario copiar, hacerlo cerca de chequear los border.
1. evitar *strncpy*. En la mayoría de los casos, es mejor devolver un error diciendo
   *too long input* y usar en cambio *strcpy* co nel tamaño exacto. Idealmente, usar sólo
   un puntero con la longitud.


## El reemplazo de C y C++

Entones, qué tan razonable es la demanda, no del todo infrecuente en internet, por la
reescritura de todo en rust? 

Bueno, considerando el análisis de Daniel Stenberg diría que no mucho. Lo que parece más
verosímil es que aparezcan proyectos nuevos que sean escritos en rust, pero no el
reemplazo de todo lo que está en C o en C++, que no es poco e incluye por ejemplo, además
de libcurl, la totalidad o gran parte del código de otros proyectos como el kernel de
linux, git, cpython, numpy, vim, emacs, redis, nodejs, macos, windows, chrome, v8, qemu,
ffmpega, texlive, openssl, gpg, ...

De este modo, proyectos escritos en lenguajes más modernos como rust, zig, go, etc., van
a vivir apoyados en infraestructura desarrollada en C por un buen tiempo. Distinto
sería si hubiese una necesidad real de reescribir todo. Cuando el proyecto GNU se propuso
(también) en su momento reescribir todo era una necesidad para ellos ya que de otra forma
no contaban con la infraestructura. Pero esto es muy distinto a volver a escribir todo en
rust, sería ir para atrás y no para adelante.
