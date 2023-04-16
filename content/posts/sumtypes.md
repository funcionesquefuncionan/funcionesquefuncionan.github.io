---
title: "Suma de tipos"
date: 2023-04-14T09:06:30-03:00
draft: false
---

A veces necesitamos un tipo que adopte alternativamente la forma de un conjunto
de tipos. O sea, dados los tipos `A, B, C, ...`, queremos un tipo `T` cuyos
valores sean de culquiera de estos tipos `A, B, C, ...`.

Esto puede ser útil en distintas situaciones. Por ejemplo, cuando tenemos una 
función que devuelve un tipo, pero que puede fallar, o mejor dicho, una función 
parcial. La función suma, desde el punto de vista matemático, no es parcial. O
sea, para cualquier par de números, existe su suma. Pero las computadoras son
muy limitadas y sólo trabajan con algunos números, no con todos. Esto significa
que tiene que haber un mínimo, y un máximo. Alguno dirá que puede implementarse
un tipo de número que "no tenga" máximo. Por ejemplo, yo puedo escribir en python
algo como 

```
>>> x = 3 ** 77
>>> y = x ** x
```

Pero esto no está libre de problemas. El más obvio es algo que está fuera de la
operación en sí misma y es que el tiempo que puede tardar eso varía mucho
dependiendo dónde se ejecute ese código. Y quizá sea lo suficientemente grande
como para disparar un *timeout* de alguien que estaba esperando un resultado, o
cansar a un cliente etc.

Por como están construídas las computadoras, además, resulta mucho más conveniente
usar números de tamaño fijo. Por ejemplo 64 bits. En este caso el mínimo y el 
máximo están claramente señalados de antemano. Ejemplo, tomemos el tipo 
`long long` en c:

```
$ man limits.h | grep -A3 "\<LLONG"
       {LLONG_MAX}
             Maximum value for an object of type long long.
             Minimum Acceptable Value: +9223372036854775807

       {LLONG_MIN}
             Minimum value for an object of type long long.
             Maximum Acceptable Value: -9223372036854775807

```

¿Qué pasa si escribimos la expresión `1 + 9223372036854775807`? Tenemos también
problemas. Una simple búsqueda de google nos da
[una respuesta](https://www.gnu.org/software/autoconf/manual/autoconf-2.63/html_node/Integer-Overflow-Basics.html)[^1]:

> the C standard says that signed integer overflow leads to undefined behavior
> where a program can do anything, including dumping core or overrunning a
> buffer. The misbehavior can even precede the overflow. Such an overflow can
> occur during addition, subtraction, multiplication, division, and left shift.

O sea que conviene no usarla en ningún programa.

Con esto en realidad voy a que si uno quisiera sumar `int` y no tener 
comportamiento indefinido, entonces no tiene más remedio que garantizar
que nunca va a haber un overflow. Si se da el caso de que hay algún valor
que de obtiene en tiempo de ejecución, por ejemplo el input del usuario
o una respuesta de ChatGPT, entonces habrá que chequearlos antes de
hacer la suma. Y si entonces ocurre que los números son demasiado grandes,
hacer algo (algo distinto a sumarlos, digo). Obvio que acá se puede elegir 
qué hacer pero una de las opciones se, valga la redundancia, usar un 
`Optional`.
[`Optional<T>` es un tipo en `java`](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html):

> A container object which may or may not contain a non-null value. If a value
> is present, isPresent() will return true and get() will return the value. 

Lo que hace es agarrar un tipo `T` (`int` en nuestro ejemplo) y construir un nuevo
tipo que funciona como la unión de `T` más un valor más llamado `null`. Entonces,
la suma "segura" lo que puede hacer es chequear los argumentos y si está todo ok,
entonces devoler `Optional.of(a+b)` pero si no `Optional.empty()`. Con esto no 
resolvemos el problema, pero sí evitamos el comportamiento indefinido y le damos
la posibilidad de resolverlo a quien usa nuestra función. Y habría que agregar
que en este caso en java, una variable de tipo `Optional<T>` puede ser ella
misma `null` lamentablemente, lo hace que uno tenga igualmente q chequear
contra null si recibe este de una función y pensar que entonces para qué
"wrappear" todo en un optional al final?

Continuando con el ejemplo, podría darse el caso de que llamamos a una función
que puede fallar por varios motivos diferentes y queremos saber cuál. Para ese
esenario puede usarse un tipo que combine `T` junto con `string` y use el
primero para el resultado cuando no hay problema y una string para indicar el
problema, si es que lo hubo.

Pero tiene muchas más aplicaciones. Por ejemplo, el tipo `List` que vimos en 
[este otro post]( {{< ref "/posts/gc-malloc" >}} ) es la unión de una lista vacía o 
un elemento agregado a una lista. También sirve para árboles combinando 
una hoja con un nodo que combina árboles, etc.

Lo que también vimos ahí es que así como hay lenguajes como haskell donde se puede
definir esto directamente (me refiero a una unión de tipos), mientras que otros
como c donde no ocurre lo mismo. Hace poco leí que mientras que existen lenguajes
como c++ o rust donde a sus respectivas comunidades les gusta ir agregando
features interesantes que no tenían, en c el lenguaje no cambia, o lo hace muy poco
en comparación, pero lo que sí, se lo usa para implementar todas esas cosas.

Ahora bien, c incluye la `union`, pero tiene una limitación bastante grande:
dado un valor de tipo unión de `A, B, C, ...`, no hay forma de saber qué tipo
está efectivamente guardado ahí. Los lenguajes más modernos usan `pattern matching`
para eso, pero no está disponible en c.

Existe una librería que implementa esto usando macros que se llama
[datatype99](https://github.com/Hirrolot/datatype99) y que define una macro `datatype`
para definir unión de tipos y `match ... of` para *pattern matching* que acabo de 
conocer mientras escribo estas líneas (así que la dejo en todo caso para otro post).

Quienes usan *plain c* lo que hacen para implementar una unión de tipos es
definir un enum con un valor para cada uno de los tipos de la unión, luego
definir el tipo como una `struct` que tiene una etiqueta con ese tipo y
después una unión con los tipos a unir. Finalmente, para pattern matching
usan `switch` en la etiqueta. Todo eso es más engorroso y da lugar a errores como
olvidarse de un caso en un switch o poner un valor inválido en la etiqueta
del tipo, pero es lo que hay. Veamos un ejemplo.

El protoclo de internet define dos versiones para las direcciones
de internet: 4 y 6. Sería razonable que una implementación defina un tipo para
cada versión y un tipo para ambas.

El header `<sys/sockaddr.h>` provee los tipos[^2].

```
typedef unsigned short int sa_family_t;

struct sockaddr {
    sa_family_t sa_family; /* Address family */
    char        sa_data[]; /* Socket address */
};

```

Y `<netinet/in.h>`:

```
struct sockaddr_in {
    sa_family_t    sin_family;
    in_port_t      sin_port;    /* Port number.  */
    struct in_addr sin_addr;    /* Internet address.  */
    unsigned char  sin_zero[8]; /* Pad to size of `struct sockaddr'.  */
};

struct sockaddr_in6 {
    sa_family_t     sin6_family;
    in_port_t       sin6_port;        /* Transport layer port # */
    uint32_t        sin6_flowinfo;    /* IPv6 flow information */
    struct in6_addr sin6_addr;        /* IPv6 address */
    uint32_t        sin6_scope_id;    /* IPv6 scope-id */
};

```

En `sockaddr` hay espacio suficiente tanto para una `struct sockaddr_in` como
para `sockaddr_in6`. Acá no se usa una `union` si no que se obliga al cliente
a castear el tipo cada vez, pero se puede llegar al mismo resultado
(de una forma tal vez un poco más fea aún).
De cualquier manera, podemos así definir por ejemplo una función para imprimir una
dirección, ya sea ipv4 o ipv6, de esta forma:

```
void print_addr(struct sockaddr* sa) {
    char buf[INET6_ADDRSTRLEN];
    char const* addr = NULL;

    if (sa) {
        switch (sa->sa-family) {
            case AF_INET:   { 
                struct sockaddr_in* sin = (struct sockaddr_in*) sa;
                addr = inet_ntop(sin->sin_family, &(sin->sin_addr), buf, INET_ADDRSTRLEN);
		break;
            } 
            case AF_INET6:  { 
                struct sockaddr_in6* sin6 = (struct sockaddr_in6*) sa;
                addr = inet_ntop(sin6->sin6_family, &(sin6->sin6_addr), buf, INET6_ADDRSTRLEN);
		break;
            } 
        }
    }

    if (addr) {
        return printf("addr: %s,  port: %d\n", buf, sa->sin_port);
    } else {
        puts("invalid sockaddr family");
    }
}
```

La función `inet_ntop` así como `INET6_ADDRSTRLEN` están
en `arpa/inet.h`

En el caso de la lista (así como el del árbol) no se usa una etiqueta para taguear
si es una lista vacía o no (o un nodo a diferencia de uan hoja en el árbol), ya que
uno se puede "ahorrar" ese campo. En vez de 

```
struct Empty { ListType t; };
struct List { ListType t; T head; List* tail; }
```

y matchear con 
```
switch(mi_lista->t) {
    case EMPTY:
        //...
    case NONEMPTY:
        //...
}
```


Basta con 

```
struct Node { T head; Node* tail; }
```

Y usar `NULL` para la lista vacía:

```
if (!mi_lista) {
    // lista vacía
} else {
    // lista no vacía
}
```

También se usa `struct List { Node* list; }` para evitar tener que pasar
`Node**` a una función que pueda tener que modificar la lista.

[^1]: Hubiera citado directamente el standard de c, pero no lo tengo a mano así.
[^2]: Omito mencionar que la implmementación al menos que uso en mi computadora
usa [`__SOCKADDR_COMMON`](https://stackoverflow.com/questions/34539564/what-does-sockaddr-common-sin-mean).
