---
title: "Licencias consideradas dañinas"
date: 2023-03-27T19:12:45-03:00
draft: false
---


> *Next it'll be "pointers considered harmful", then "duck typing considered harmful". Then who will be left to defend you when they come to take away your unsafe programming construct? Eh?*
>
> Steve Jesop, respondiendo a [GOTO still considered harmfull?](https://stackoverflow.com/questions/46586/goto-still-considered-harmful)

Como en cualquier otra actividad, el programador tiene ante sí posibilidad
latente de equivocarse. Y ante un problema propenso a hacernos errar hay
(al menos) dos formas diferentes de actuar: ser extremadamente cuidadoso
o recurrir a una garantía ajena[^1]. Con garantía ajena me refiero a que otra
persona lo haga, pero dándome más seguridad que mi mismo. Por ejemplo, usar
un código ya testeado y desarrollado por gente suficientemente cuidadosa.

Esta segunda alternativa, desde luego, acelera el tiempo del desarrollo. Por
ejemplo, si en vez de implementar una
[*hash table*](https://es.wikipedia.org/wiki/Tabla_hash)
importo una cuyo uso sea estándar en la industria, voy a tardar menos tiempo
en desarrollar lo que tenía que hacer, obviamente. Es por eso que siempre se 
recomienda, de ser posible, tomar el segundo camino, y no **reinventar la rueda**.

A veces, sin embargo, se habla de *lo que se pierde* al hacerlo. Incluso hoy
podemos ver videos o posts mencionando casos en los que una implementación
*custom* de una hash table da una mejora en la *performance*. Pero me parece
que eso es más bien anecdótico.

Lo que parece ser recurrente es que cada tanto sale algo que viene y nos
dice: no hagas más tal cosa, metelo en una capa más abajo. Puede que haya
resistencia cada vez que eso pasa, pero en algún momento se define el conflicto.

Así, cuando se inventó el lenguaje [*fortran*](https://fortran-lang.org/en/),
dicen, había quienes desconfiaban de las ventajas de dejar de lado las
posibilidades de optimización a que daba lugar escribir directamente [*lenguaje
ensamblador*](https://es.wikipedia.org/wiki/Lenguaje_ensamblador).  Hoy en día,
*lo que se dice* es que al contrario nunca vas a ser más inteligente que el
compilador (o al menos sería rarísimo), así que dejá que el compilador optimice
(o sea, deja que una capa de más abajo lo haga por vos). Como dijo Knuth, la
optimización *prematura* es la fuente de todos los males, y esto podría ser un
ejemplo quizá.

En cuanto al *goto*, claramente hay casos y casos del uso del goto (por ejemplo,
puede que para simplificar el código de manejo de errores cuando se tienen que
manejar recursos pedidos al sistema sea mejor usarlo que no). Pero digamos que,
en general, el goto no ayuda si no que complica el código, favorece el hacerlo
difícil de entender, y la introducción de errores. Sobre todo si no se tiene
suficiente cuidado.

Los punteros suelen mencionarse también como problemáticos. Pero en realidad
los punteros están asociados a más de un problema. En
[*c*](https://es.wikipedia.org/wiki/C_(lenguaje_de_programaci%C3%B3n))
cuando se maneja manualmente la memoria (es decir, del heap) se usa un puntero.
Entonces los punteros se asocian a las dificultades vinculadas al manejo de
memoria. Por otra parte, cuando se *degrada* el sistema de tipos haciendo casteos
entre dos tipos cualesquiera, se usan punteros. Esto es algo distinto que manejar
memoria, pero es otro problema (que también se puede dar usando *uniones* por
otra parte).

Para la cuestión del manejo de memoria, la primera forma de meterlo un nivel
más abajo fue con el 
[*garbage collector*](https://es.wikipedia.org/wiki/Recolector_de_basura),
primero implementado para *lisp* y usado en muchos lenguajes muy usados
como *java*, *python*, *javascript*, *go*, *kotlin*,*R*, *haskell*, ...
En realidad son mucho menos los lenguajes que no lo usan. Fortran, por ejemplo
(que es anterior),
pero también c, c++, objective-c, [*swift*](https://developer.apple.com/swift/).
Existe un gc (garbage collector) para tanto 
[c como c++](https://github.com/ivmai/bdwgc), pero no siempre se lo usa. Por
ejemplo es c++ es más común usar
[RAII](https://es.wikipedia.org/wiki/RAII), y en c el manejo manual mediante
*malloc* y *free*. Objective-c y swift usan
[*reference countung*](https://es.wikipedia.org/wiki/Conteo_de_referencias).
Después existen, llamativamente, unos lenguajes más modernos que tampoco
delegan toda la responsabilidad en el gc. El más destacado hoy en día es,
claramente, *rust*, que usa una estrategia alternativa para lograr el mismo
objetivo de meter el problema en una capa más abajo, haiendo que el compilador
verifique estáticamente si la memoria tiene que liberarse en algún punto.
Pero no es la idea en este post hablar de este problema en particular y con detenimiento,
sino mencionarlo en la enumeración de ejemplos en los que la evolución viene
de la mano de la privación o limitación del programador.

Encontramos otro ejemplo de esto en el chequeo estricto de tipos, con el cual 
en vez de programar pensando en que las variables o los valores que guardan tienen
la estructura o los atributos que espero, que lo haga el compilador.

Más: las [*partes privadas*](https://en.cppreference.com/w/cpp/language/access)
que usan c++ y java por ejemplo, y que evitan que uno pueda
ver o modificar partes de una objeto desde código ajeno a su implementación, la
[transparencia referencial](https://es.wikipedia.org/wiki/Transparencia_referencial)
, que prohibe 
toda una categoría de rutinas o procedimientos que no se comportan como verdaderas
funciones.

Pese a todo esto, no es que estos cambios se impongan en forma ubicua.
A veces se recomienda usar goto para tener un
[punto de salida único](http://wiki.c2.com/?SingleFunctionExitPoint)
en una función para hacerla menos confusa.
También se diseñan
[lenguajes de programación nuevos](https://news.ycombinator.com/item?id=31151591)
con manejo manual de memoria para evitar el overhead en runtime del
gc pero mantener mayor simplicidad que rust. Muchos lenguajes usan duck typing
en lugar de chequeo estático de tipos (esto no sé por qué). En python (además
de usar duck typing), si bien tiene la convención de usar dos guiones bajos
para los métodos o variables "privados", no hay nada que impida al progamador
realmente usar o escibir ahí.

[^1]: Bueno, también se puede ser descuidado pero se entiende.
