Cuando uno está empezando a programar (pero después también), siempre es mejor
encarar los problemas de a uno y simplificarlos en lo posible. El principio
es bastante convincente y dudo que muchos quieran contradecirlo. Sin embargo,
la manera de hacer esto no es siempre evidente. O quizás la separación en sí
no es posible si no que se hace posible después de cierto trabajo.

Por ejemplo, supongamos que queremos implementar una lista enlazada. Para eso
podemos decir simplemente que una lista puede ser o una lista vacia o un par 
que tiene un primer elemento (`head`) y el resto (`tail`) que es en sí mismo
una lista (a este par `head` y `tail` lo llamamos `Cons`). Usando `haskell` se
podía ver algo así:

```
data Lista a = Vacia | Cons a (Lista a) deriving Show
```

Lo que se ve bastante simple. Pero para que eso funcione hay muchas cosas que
tienen que estar en su lugar. Supongamos que hacemos una lista para ir al chino
y no olvidarmos

```
let compras = Cons "agua" (Cons "yerba" (Cons "miel" (Cons "alfajores" ...)))
```

(los `...` son para indicar que la lista sigue, no es parte de la sintaxis).
Con eso estamos guardando en la memoria (de la computadora) nuestra lista,
en `compras`. Pero una vez que hicimos la compra ya no necesitamos recordar
la lista, entonces podemos olvidarla. Sin embargo, no tenemos que olvidarla
antes de terminar nuestra compra, porque si no podemos tener que volver a ir
si es que nos olvidamos algo. Todo esto de la memoria no es algo que sea 
inherente al problema de qué es una lista, y de hecho podemos abstraernos
de él. Sin embargo, esto es porque ya se está manejando. Si existiera algo
que se ocupara de eso, tendíamos que hacerlo nosotros.

La misma idea de lista, si fuésemos a usar `c`, sería algo como:

```
typedef struct Lista Lista;
typedef struct Lista {
    char* head;
    Lista* tail;
} Lista;
```

Esto quizá invoque al lector el tema no sólo de la memoria si no también el
de los punteros (que está relacionado) y puede que otras cosas como que esta
lista es menos genérica. Pero vamos de a una. Lo que en `haskell` era escribir
`let vacia = Vacia` acá sería algo como:

```
Lista vacia = 0;
```

Pero la función `cons` en `c` no vine dada, como en `haskell` (donde la definimos
al definir el tipo Lista). Podríamos hacer algo así:

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
devuelve `0`, eso significa que no disponemos de memoria para satifacer a quien
llamó a `cons` y no podemos hacer nada. Todo lo que podemos hacer, en todo caso,
es avisarle que en tal situación se lo vamos a indicar devolviéndole a a él 
también `0`, con lo cual él también va a tener que preocuparse por la memoria
(cuando probablemente su problema concreto haya sdo otro).

Pero hay otro problema, y es que la memoria pedida con `malloc` no se va a liberar
hasta que alguien llame a `free`. Es decir, tenemos que hacer otra función como

```
void free_lista(Lista* ls) {
    if (ls) {
        free_lista(ls->tail);
        free(ls);
    }
}
```

Esto, suponiendo que las `string`s son literales porque si no también tendríamos 
que fijarnos si no hay que liberar su memoria también... (podría pasar que otra
lista también apunta a la misma `string` así que habría que ver eso, que no es
facil, y menos si no era el problema que había que resolver en un principio).

Entonces `c` es un lenguaje obsoleto al lado de las opciones modernas que tienen
muchas más soluciones? Bueno, hay gente que dice eso. Pero yo creo que es no es
así.

Podemos hacer algo como `haskell` (o `java`, o `python`, etc) con lo de que `malloc`
no nos de memoria. Quedaría así:


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
Simplemente su programa se interrumple.

En cuanto al otro problema podemos usar [Bohem](https://www.hboehm.info/gc/).

```
Lista* cons(char* head, Lista* tail) {
    Lista* lista = GC_malloc(sizeof (Lista));
    if (!lista) {
        perror("not enough memory!");
        abort();
    }
    *lista = (Lista) { .head = head, .tail = tail };
    return lista;
}
```

Y listo, el /garbage collector/ se ocupa de liberar la memoria cuando no se la
necesite más. Lo único es que para compilar el programa hay que conseguir la librería 
(o biblioteca).

Acá hay más de una forma de hacer esto (y en otro momento podría hacer un post al respecto),
pero digamos que la más fácil quizá sea usar el gestor de paquetes de nuestra distribución
(si usamos ubuntu apt, en mac brew etc.).

En ubuntu: `sudo apt install libgc-dev`. Después en `main.c` agregamos `#include <gc/gc.h>`
y le pasamos al compilador `\`pgk-config --libs bdw-gc\``.

