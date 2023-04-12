---
title: "Listas y memoria"
date: 2023-04-11T17:12:45-03:00
draft: false
---

Cuando uno está empezando a programar (pero después también), siempre es mejor
encarar los problemas de a uno y simplificarlos en lo posible. El principio es
bastante convincente y dudo que muchos quieran contradecirlo. Sin embargo, la
manera de hacer esto no es siempre evidente. O quizás la separación en sí no es
posible si no que se hace posible después de cierto trabajo.

Por ejemplo, supongamos que queremos implementar una lista enlazada. Para eso
podemos decir simplemente que una lista puede ser o una lista vacía o un par
que tiene un primer elemento (`head`) y el resto (`tail`) que es en sí mismo
una lista (a este par `head` y `tail` lo llamamos `Cons`). Usando `haskell` se
podía ver algo así:

```
data Lista a = Vacia | Cons a (Lista a) deriving Show
```

Lo que se ve bastante simple. Pero para que eso funcione hay muchas cosas que
tienen que estar en su lugar. Supongamos que hacemos una lista para ir al chino
y no olvidarnos

```
let compras = Cons "agua" (Cons "yerba" (Cons "miel" (Cons "alfajores" ...)))
```

(los `...` son para indicar que la lista sigue, no es parte de la sintaxis).
Con eso estamos guardando en la memoria (de la computadora) nuestra lista, en
`compras`. Pero una vez que hicimos la compra ya no necesitamos recordar la
lista, entonces podemos olvidarla. Sin embargo, no tenemos que olvidarla antes
de terminar nuestra compra, porque si no podemos tener que volver a ir si es
que nos olvidamos algo. Todo esto de la memoria no es algo que sea inherente al
problema de qué es una lista, y de hecho podemos abstraernos de él. Sin
embargo, esto es porque ya se está manejando. Si existiera algo que se ocupara
de eso, tendíamos que hacerlo nosotros.

La misma idea de lista, si fuésemos a usar `c`, sería algo como:

```
typedef struct Lista Lista;
typedef struct Lista { char* head; Lista* tail; } Lista;
```

Esto quizá invoque al lector el tema no sólo de la memoria si no también el de
los punteros (que está relacionado) y puede que otras cosas como que esta lista
es menos genérica. Pero vamos de a una. Lo que en `haskell` era escribir `let
vacia = Vacia` acá sería algo como:

```
Lista vacia = 0;
```

Pero la función `cons` en `c` no vine dada, como en `haskell` (donde la
definimos al definir el tipo Lista). Podríamos hacer algo así:

```
Lista* cons(char* head, Lista* tail) {
    Lista* lista = malloc(sizeof (Lista));
    if (!lista)
        return 0;
    *lista = (Lista) { .head = head, .tail = tail };
    return lista;
}
```

Donde llamamos a la función `malloc`. Entonces tenemos (al menos) dos problemas
mezclados, el manejo de la memoria y la construcción de una lista. Si `malloc`
devuelve `0`, eso significa que no disponemos de memoria para satisfacer a quien
llamó a `cons` y no podemos hacer nada. Todo lo que podemos hacer, en todo
caso, es avisarle que en tal situación se lo vamos a indicar devolviéndole a a
él también `0`, con lo cual él también va a tener que preocuparse por la
memoria (cuando probablemente su problema concreto haya sido otro).

Pero hay otro problema, y es que la memoria pedida con `malloc` no se va a
liberar hasta que alguien llame a `free`. Es decir, tenemos que hacer otra
función como

```
void free_lista(Lista* ls) {
    if (ls) {
        free_lista(ls->tail);
        free(ls);
    }
}
```

Esto, suponiendo que las `string`s son literales porque si no también
tendríamos que fijarnos si no hay que liberar su memoria también... (podría
pasar que otra lista también apunta a la misma `string` así que habría que ver
eso, que no es fácil, y menos si no era el problema que había que resolver en
un principio).

Entonces, es `c` un lenguaje obsoleto al lado de las opciones modernas que
tienen muchas más soluciones? Bueno, hay gente que dice eso. Pero yo creo que
es no es así.

Podemos hacer algo como `haskell` (o `java`, o `python`, etc) con lo de que
`malloc` no nos de memoria. Quedaría así:


```
Lista* cons(char* head, Lista* tail) {
    Lista* lista = malloc(sizeof (Lista));
    if (!lista) {
        perror("not enough memory!");
        abort();
    }
    *lista = (Lista) { .head = head, .tail = tail };
    return lista;
}
```

Con esto evitamos que el usuario se tenga que fijar si recibió o no una lista.
Simplemente su programa se interrumpe.

En cuanto al otro problema podemos usar [Bohem](https://www.hboehm.info/gc/).

```
Lista* cons(char* head, Lista* tail) {
    Lista* lista = GC_malloc(sizeof (Lista));
    if (!lista) {
        perror("not enough memory!");
        abort(); }
    *lista = (Lista) { .head = head, .tail = tail };
    return lista;
}
```

Y listo, el /garbage collector/ se ocupa de liberar la memoria cuando no se la
necesite más. Lo único es que para compilar el programa hay que conseguir la
librería (o biblioteca).

Acá hay más de una forma de hacer esto (y en otro momento podría hacer un post
al respecto), pero digamos que la más fácil quizá sea usar el gestor de
paquetes de nuestra distribución (si usamos ubuntu apt, en mac brew etc.).

En ubuntu: `sudo apt install libgc-dev`. Después en `main.c` agregamos
`#include <gc/gc.h>` y le pasamos al compilador `` `pgk-config --libs bdw-gc` ``.

Claro que esto no es un argument decisivo, ni mucho menos, para el "debate"
sobre la obsolescencia de c, que además tiene otros problemas, como un sistema
de tipos tipado tan debilmente por ejemplo. Pero es que por algún motivo que
desconozco últimamente me he cruzado con muchas publicaciones de gente con
bastantes críticas contra c (y también otros lenguajes con bastantes años, como
c++, incluso java) y reclamando dejarlos de lado para usar *lo nuevo*.  Supongo
que el paso del tiempo impone la ocurrencia de estos cambios, y las nuevas
generaciones adoptan lenguajes que cuando las anteriores empezaron toavía no
existían.

De todas formas creo que cuando la discusión adopta la forma tribalista es una
pérdida de tiempo. No por ello me opongo rotundamente a observar su devenir, ya
que perder el tiempo es una parte importante de la vida, al menos para algunos
de nosotros. Sí recomendaría al que preguntase no atarse a un lenguaje de
programación ya que de todas formas es una manera de codificar un programa y
programar va más allá de dominar esa herramienta en particular.

Por último, quizá muchos vean en este ejemplo de listas otro caso más contra
c (y en particular a favor de haskell). Sin embargo, considero que previamente
a sacar esa conclusión habría que chequear que no se trate que justamente 
haskell es justamente práctico para hacer listas y no concluyamos demasiado
rápido generalizando ese punto. Antes habría que llevar a cabo una comparación 
mejor.
