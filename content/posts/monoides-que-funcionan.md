---
title: "Monoides que funcionan"
date: 2023-04-30T12:01:21-03:00
draft: true
---

Los lenguajes de programación suelen diferenciar los tipos atómicos, o
*escalares*, de los que son colecciones. Esto es natural, porque las
computadoras suelen estar dotadas de unos cuantos *registros*, en los cuales se
puede "poner" (guardar, escribir) un valor atómico y realizar operaciones sobre
él. Por ejemplo, puedo poner un número entero `n`
y aplicarle la función sucesor que devuelve el siguiente número (o cero si es el
último que entra en el registro). Pero por otra parte, es común necesitar hacer
cosas con colecciones de números, como aplicarles a cada uno una función, o
realizar una operación sobre la colección en sí, como obtener el máximo etc.
También podemos querer aplicar una función que toma un par de escalares y
devuelve uno a lo largo de toda la colección.

En C, por ejemplo, un entero es un `unsigned n` mientras que una colección  de
`K` enteros (un arreglo en este caso) es `unsigned ns[K]`. Para aplicar la
función sucesor podemos escribir `int m = n + 1`, pero para aplicar esa misma
función a todos los números de la colección `ns` no es tan directo, tenemos que
usar un `for` e *iterar*. Nos queda algo así:

```
unsigned sucs[K];
for (size_t index = 0; index < K; ++index) {
	sucs[index] = ns[index] + 1;
}
```

Es decir, lo que hicimos fue usar una función que ya no es sucesor de un número,
si no que es una función que dado un *índice* te da el sucesor del elemento
indexado por ese índice en la colección. Sería mucho más cómodo simplemente
hacer algo así: `unsinged sucs[K] = aplicar ns (+ 1)`.

Lamentablemente, eso no puede hacerse en C, pero casi todos los demás lenguajes
si tienen algo de eso. La operación de aplicar una función a cada uno de los
elementos de una colección se llama `map`. Por ejemplo en haskell es:

```
map (+1) ns
```

Hay algunas cosas cómodas en ese ejemplo. Primero, `(+1)` es una función,
definida *al vuelo* y que se le pasa como argumento a la función map. Eso en C
no se puede, las funciones hay que definirlas con nombre y todo en su propio
bloque, no adentro de otra función etc. Segundo, esa función se definió
simplemente agarrando otra que ya existía, la función `+`, que toma dos números
y devuelve uno, y fijándole uno de los dos número a sumar. Sería como hacer en C
algo como

```
unsigned sucesor(unsigned n) { return n + 1; }
...
map(sucesor, ns, &res);
```

si es que ya tenemos definido `map` en algún lado,
que es lo mismo pero más verborrágico.

Tercero, usamos `map` y no necesitamos hacer eso de los índices.

Además de map, podemos querer hacer otro tipo de operaciones sobre los elementos
de la colección, como sumarlos:

```
unsigned res = 0;
for (size_t index = 0; index < K; ++index) {
	res = ns[index] + res;
}
```

O también multiplicarlos 
```
unsigned res = 0;
for (size_t index = 0; index < K; ++index) {
	res = ns[index] * res;
}
```

Obtener el mínimo
```
unsigned res = MAX_UINT;
for (size_t index = 0; index < K; ++index) {
	res = min(ns[index], res);
}
```

Se va fácilmente el patrón. Primero, se inicializa `res` con un valor que no
altera el resultado. Después se realiza la operación con el resultado parcial
por cada uno de los elementos de la colección. No sería bueno tener algo
parecido a map que sea tipo `combinar operación colección` y no tener que estar
con el índice y todo lo demás?

Ahora bien, el título del post menciona a los monoides, qué tienen que ver con
esto? Bueno, un monoide es justamente un conjunto con una operación binaria que
agarra dos cosas de ese conjunto y que cumple tres propiedades:
1. el valor que devuelve está en ese mismo conjunto
2. si aplicamos la operación a dos elementos y luego a este resultado con un
   tercer, el resultado es el mismo si lo aplicábamos primero al último con el
   segundo y a este resultado con el primero, o sea: `(a op b) op c = a op (b op
   c)`.
3. hay un elemento "neutro" que si es argumento de la operación hace que lo que
   devuelve sea el otro sin cambiar, o sea: `x op Neutro = x`.

Esto se relaciona con lo que estábamos diciendo. En los ejemplos de suma,
producto y mínimo, tenemos una operación que toma dos elementos de un conjunto y
devuelve otro del mismo conjunto. Además, en cada caso hay uno neutro: en la
suma el cero y en el producto el 1. En mínimo también! Si nuestro conjunto tiene
un máximo (como para con el tipo `unsigned`), entonces `min(x, máximo) = x`.


Esto dio lugar a la `typeclass`
[`Monoid` en haskell](https://wiki.haskell.org/Monoid) (no confundir con mónada!).

Por cada operación sobre un tipo que cumpla estas tres propiedades tenemos un
monoide. En el link se explica pero voy a mencionar únicamente de ahí un
ejemplo de aplicación (sí, una aplicación de la matemática a la computación que 
no es la aproximación de una función real ...).

En haskell existe el tipo `Ordering` y la función `compare` que dado dos objetos
ordenables, te dice si el primero es mas grande, más chico, o son iguales.
Ejemplos de `compare`:

```
ghci> compare 1 2
LT
ghci> compare 0 0
EQ
ghci> compare "haskell" "c"
GT
```

Estos
tres valores (`GT`, `LT`, `EQ` por sus siglas en inglés (?) son el tipo `Ordering`.
`Ordering` es una instancia de `Monoid`. El neutro es `EQ`. La operación en
cuestión (en haskell esta operación se escribe `<>`) es el primer argumento, si
no es `EQ`, o el segundo, si el primero es `EQ`. O sea

|   | EQ| LT| GQ|
|---|---|---|---|
|EQ|EQ|LT|GT|
|LT|LT|LT|LT|
|GT|GT|GT|GT|

La columna de la izquerda es el primer parámetro, la primer fila el segundo.
Otro ejemplo de función que devuelve `Ordering`s:

```
ghci> comparing length "monoide" "mónada"
GT
ghci> comparing (!!0) "c" "c++"
EQ
```
Acá `comparing` lo que hace es usar su primer parámetro `length` para proyectar
los valores a comparar (en este caso strings a enteros) y usar `compare` en
ellos.

Pero como `Ordering` es un monoide, los valores de `Ordering` se pueden
combinar y además, pueden también combinarse funciones que devuelven
un `Ordering` aplicándola y combinando sus resultados. O sea: podemos combinar
`compare` con `comparing length`. Y así podemos escribir una función que compare
primero por tamaño y después (si dos string tiene el mismo tamaño)
lexicográficamente:

```
sortStrings = sortBy (comparing length <> compare)
ghci> sortStrings ["EQ", "LT", "GT"]
["EQ","GT","LT"]
```


Por último, existe una función que agarra una colección y la *reduce* tomando
sus elementos de a dos y aplicando una operación binaria. En haskell están para
eso las funciones `foldl` y `foldr`. Reciben como primer parámetro la operación
binaria, como segundo un valor inicial y finalmente la coleccón. Ejemplos:

```
ghci> foldMin = foldr min (maxBound :: Int)
ghci> foldSum = foldr (+) 0
ghci> foldProd = foldr (*) 1
```

Pero el valor incial y el resultado de la operación binaria no tienen que ser,
necesariamente, del mismo tipo (como occurre en el monoide)
