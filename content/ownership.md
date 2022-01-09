+++
title = "Ownership: o cómo se gestiona la memoria en Rust (parte I)"
summary = "Este apunte surge de mi necesidad de querer entender bien las bases de cómo Rust gestiona la memoria. Siento que si esta parte no la comprendo lo acabaré pagando. En esta primera entrega me enfrento a la Propiedad."
description = "Una anotación respecto la gestión de la memoria enfocada en entender la propiedad."
slug = "ownership-gestion-de-memoria-rust"
date = "2021-12-26"
categories = ['rust']
tags = ['rust', 'memoria', 'ownership', 'propiedad']
+++

> Este apunte es la primera entrega de una serie de anotaciones que estoy haciendo respecto la gestión de la memoria en Rust.
>
> * Parte 1: Ownership
> * [Parte 2: Borrowing](../borrowing-gestion-de-memoria-rust)
>
> Puesto que no soy ningún experto en la materia aviso que este escrito puede contener errores. Como sigo estudiando y aprendiendo lo iré corrigiendo y ampliando. También aviso que he simplificado algunas secciones para facilitar alguna explicación y mi propio entendimiento.

Esta es una lista de palabras inglesas muy utilizadas en el ámbito de la gestión de la memoria en Rust con la traducción que he utilizado para este documento.

* ***Allocation***: Asignar.
* ***Bind***: Enlazar.
* ***Borrowing***: Préstamo.
* ***Copy***: Copiar.
* ***Dropped***: Soltado.
* ***Free***: Liberar.
* ***Garbage collection***: Recolección de basura.
* ***Heap***: Montón.
* ***Lifetime***: Tiempo de vida.
* ***Manual memory allocation***: Asignación manual de memoria.
* ***Owner***: Propietario.
* ***Ownership***: Propiedad.
* ***Pointer***: Puntero.
* ***Runtime***: Entorno de ejecución.
* ***Scope***: Ámbito.
* ***Size***: Tamaño.
* ***Stack***: Pila.
* ___Trait___: Rasgo.
* ***Unbind***: Desligar.

Para poder entender cómo se gestiona la memoria en Rust antes es necesario conocer, aunque sea de una manera superficial, cómo se usa la memoria de un ordenador.

## Variables y datos

Siempre imaginé una variable como una caja donde se guarda un dato. Y que esa caja era una porción de memoria. A esa caja se le podían ir guardando datos, unos reemplazando a los otros y cuando ya no los necesitaba, "destruía" esa caja, esa variable, y listos.

Frases que estaban en mi cabeza como "a la variable num se le asigna el valor 1" o "num vale 1", sumado a la sintaxis que utilizan muchos lenguajes de programación para declarar variables y asignarles un dato, me hacían pensar que primero existía la variable y luego el dato que se guarda en ella.

Pero lo que sucede es ligeramente diferente. Primero está el dato y luego está la variable. Primero el dato se guarda en una porción de la memoria y luego se crea una variable que se enlaza \(_bind_\) con esa porción de memoria. Ese enlace consiste en una dirección de memoria, que al igual que en un callejero, nos permite saber dónde de toda la memoria disponible está guardado el dato.

Cuando declaramos una variable y le asignamos un dato, estamos enlazando la variable al dato. De la misma manera que cuando pasamos una variable como parámetro a una función no pasamos el dato de una caja a otra, sino que enlazamos la variable que recibe el parámetro a ese dato. Lo mismo sucede con el retorno de funciones, enlazamos el dato de retorno a la variable que espera ese dato.

El cambio es sutil, pero el concepto de enlace es muy útil para entender ciertos aspectos de la gestión de la memoria que vamos a ver a continuación.

## Pila y montón

La pila \(_stack_\) y el montón \(_heap_\) son dos tipos de memoria donde podemos almacenar datos.

La pila tiene una estructura, justamente como su nombre indica, de pila. En la pila se guardan los datos uno encima del otro y se quitan de uno en uno empezando por el último que se puso. A este funcionamiento se le llama [LIFO](https://es.wikipedia.org/wiki/Last_in%2C_first_out), _Last In, First Out_. Podemos imaginarlo como una pila de libros, complicado quitar el libro que hay más abajo sin quitar antes los que hay encima.

El montón no tiene una estructura fija tan "estricta" como la pila, es más un espacio. En la que los datos se van guardando allí donde hay espacio libre. Siguiendo con la analogía anterior, podríamos decir que es una estantería donde podemos poner algunos libros continuos y otros no. Unos en un estante y otros en otro y no es necesario que sigan ninguna ordenación.

Qué va en la pila y qué va en el montón depende, generalizando, del tipo del dato que queremos almacenar y más precisamente del tamaño \(_size_\) de la porción de memoria necesaria para almacenar ese dato.

Todo dato que requiera de una cantidad de memoria que es conocida en tiempo de compilación y que no cambiará a lo largo de la ejecución del programa, se almacena en la pila. Cuando esa cantidad es desconocida en tiempo de compilación o cambiará a lo largo de la ejecución del programa, se almacena en el montón.

* Se guardan en la pila, por ejemplo: enteros, flotantes, booleanos, caracteres, punteros...
* Se guardan en el montón, por ejemplo: cadenas de texto, listas, vectores...

Veamos un ejemplo de dato almacenado en la pila:

```rust
let i: i32 = 10;

// El dato 10 es almacenado en la pila ya que conocemos la cantidad
// de bytes necesarios para almacenar ese dato (el mismo que para
// almacenar cualquier dato soportado por el tipo i32, ya sea un 200
// o un 1239).
```

```txt
      +----+---+
Pila  | 10 | i |
      +----+---+
```

Ahora veamos un ejemplo de dato almacenado en el montón:

```rust
let mut texto: String = String::from("Hola, mundo");

// El dato "Hola, mundo" es almacenado en el montón ya que la variable
// 'texto' al ser de tipo String puede cambiar su contenido, por tanto
// cambiar su tamaño.
// Es diferente el tamaño necesario para almacenar "Hola, mundo" que 
// para almacenar "Adiós mundo cruel".
```

La variable `texto` se guarda en memoria de la siguiente manera: en el montón se guarda el dato \(en este caso la cadena de texto\) y en la pila se almacena un puntero \(_pointer_\) a ese espacio en el montón junto con la capacidad de ese espacio y el tamaño del dato.

```txt
              puntero
            /    capacidad
           /    /     tamaño
          /    /     /   variable
         /    /     /   /
       +---+----+----+-------+
Pila   | * | 13 | 12 | texto |
       +-|-+----+----+-------+
         |
       [-|----------- capacidad -----------------------]
         |
       +-v-+---+---+---+---+---+---+---+---+---+---+---+
Montón | H | o | l | a | , |   | m | u | n | d | o |   |
       +---+---+---+---+---+---+---+---+---+---+---+---+

       [-------------- tamaño ---------------------]
```

La manera en cómo se almacenan y se borran los datos en el montón determina cómo gestiona la memoria un lenguaje de programación y marca cómo se programa en ese lenguaje.

## Gestión de la memoria

Todos los lenguajes de programación transfieren al programador, en mayor o menor medida, la gestión de la memoria del montón. Esa gestión se refiere, principalmente, a la responsabilidad de almacenar datos en memoria ocupando memoria libre \(_allocation\)_ y borrar esos datos cuando ya nos son necesarios, liberando la memoria ocupada \(_free_\).

Generalizando, esa gestión puede ser de dos maneras:

* mediante un **recolector de basura** \(_garbage collector_\), donde el programador no tiene que pensar ni preocuparse dónde los datos son almacenados ni cómo ni cuando son borrados, ya que es el propio entorno de ejecución \(_runtime_\) del lenguaje el que se preocupa por ti. Lenguajes como Javascript o Java entre muchos funcionan de esta manera.
* mediante la **asignación manual de memoria** \(_Manual memory allocation_\), en la que la gestión completa de la memoria recae sobre el programador. Es el programador quien tiene que especificar cómo y dónde almacenar los datos y cuando borrarlos. Lenguajes como C y C++ funcionan de esta manera.

El recolector de basura facilita la vida al desarrollador a costa de una pérdida de rendimiento y de control. Mediante la asignación manual de memoria tienes el rendimiento y control, a cambio de una mayor complejidad de código.

Pero existe una tercera manera de gestionar la memoria, la manera en cómo lo hace Rust, mediante la propiedad \(_ownership_\).

## Propiedad

La propiedad son una serie de reglas que hacen que el programador no tenga que pensar en cómo gestionar la memoria. Sin afectar a su rendimiento ni perder su control, como puede suceder con la recolección de basura y detectando en fase de compilación los posibles errores relacionados con la memoria que puedes encontrarte en fase de ejecución con la asignación manual de la memoria (*dangling pointers* o *double free*).

Y se reduce a algo muy simple, en Rust, todo dato tiene un único propietario \(_owner_\). Ser propietario de un dato implica ser el único que puede acceder y manipular el dato y determina el tiempo \(_lifetime_\) durante el que puedes hacerlo.

### La propiedad empieza con una asignación

Asignar un dato a una variable \(por tanto enlazar la variable a ese dato\) hace que esa variable sea la propietaria de ese dato.

```rust
fn main () {
    let num: i32 = 10;
    println!("El valor de num es: {}", num);
}

// El dato 10 está enlazado a la variable 'num'.
// La variable num es la propietaria del dato 10.
```

### La propiedad termina con el ámbito

Cuando termina el ámbito \(_scope\)_ de una variable, la variable se elimina, se rompe el enlace \(_unbind_\) entre la variable y el dato del que es propietaria y se borra automáticamente el dato de la memoria \(comportando la liberación de esa porción de memoria\). En Rust se dice que el dato es soltado \(_dropped_\).

```rust
fn main () {
    {
        let num: i32 = 10;
        println!("Este es el ámbito de num y su valor es: {}", num);
    }
    println!("Esto está fuera del ámbito de num y su valor es: {}", num);
}

// La declaración de la variable 'num' ocurre dentro de un bloque
// delimitado entre {}.
// El ámbito de la variable 'num' es ese bloque de código.

// Una vez se sale del ámbito, el dato enlazado con la variable 'num'
// (un 10) es borrado de la memoria (es soltado).

// No se puede usar la variable 'num' fuera de su ámbito por tanto
// no se puede acceder al dato 10 más allá de ese ámbito. La sentecia: 
// println!("Esto está fuera del ámbito de num y su valor es: {}", num); 
// da un error de compilación.
```

### La propiedad se mueve con el cambio de asignación

Asignar una variable a otra (asignación explícita) hace que la propiedad del dato pase de una variable a la otra.  Se rompe el enlace entre la variable original y el valor. En Rust se dice que la propiedad se ha movido (*move*, *moved*).

```rust
fn main () {
    let hola: String = String::from("Hola, mundo");
    // `hola` es la propietaria del dato "Hola, mundo".
    
    let saludo = hola;
    // `saludo` es ahora la propietaria del dato "Hola, mundo".
}
```

Sucede igual cuando pasamos una variable como parámetro de una función (asignación implícita). La propiedad pasa a la variable que "recoge" el dato.

```rust
fn main () {
    let hola: String = String::from("Hola, mundo");
    // `hola` es la propietaria del dato "Hola, mundo".
    
    saluda(hola);    
}

fn saluda (mensaje: String) {
    // `mensaje` es ahora la propietaria del dato "Hola, mundo".
    println!("{}", mensaje);
}
```

Como he escrito anteriormente, todo dato tiene un único propietario y el propietario es el único que puede acceder al dato. Por tanto a partir de un cambio de propiedad cualquier intento de acceder al dato mediante la variable original no será posible y comportará un error del que nos avisará, amablemente, el compilador.

```rust
fn main () {
    let hola: String = String::from("Hola, mundo");
    // `hola` es la propietaria del dato "Hola, mundo".
    
    let saludo = hola;
    // `saludo` es ahora la propietaria del dato "Hola, mundo".
    
    let hola_mundo = hola; 
    // Asignamos de nuevo `hola` a otra variable.
}
```

```rust
// El resultado de la compilación es el siguiente:
let hola: String = String::from("Hola, mundo");
    ---- move occurs because `hola` has type `String`, which does not implement the `Copy` trait
 let saludo = hola;
              ---- value moved here

 let hola_mundo = hola;
                  ^^^^ value used here after move

// Prestemos, atención de momento, solo a las líneas 5 y 8.
// La línea 5 nos marca dónde ha sucedido el movimiento de la 
// propiedad, y la línea 8 nos marca dónde se ha utilizado una
// variable que ya no tiene la propiedad de ningún dato.
```

De nuevo sucede lo mismo cuando tratamos de usar como parámetro de una función una variable que ya no está enlazada a ningún dato.

```rust
fn main () {
    let hola: String = String::from("Hola, mundo");
    // `hola` es la propietaria del dato "Hola, mundo".

    let saludo = hola;
    // `saludo` es ahora la propietaria del dato "Hola, mundo".

    saluda(hola);  
    // Al pasar la variable `hola` como parámetro de una función
    // estamos de nuevo asignándola a otra variable, en este
    // caso la variable `mensaje` de la función `saluda()`
}

fn saluda (mensaje: String) {
    println!("{}", mensaje);
}
```

```rust
let hola: String = String::from("Hola, mundo");
    ---- move occurs because `hola` has type `String`, which does not implement the `Copy` trait
let saludo = hola;
             ---- value moved here

saluda(hola);
       ^^^^ value used here after move
```

Este movimiento de la propiedad vinculado a un cambio de asignación tiene un aspecto muy importante, **únicamente sucede con datos almacenados en el montón**, y para ser más exactos **únicamente sucede con los datos cuyo tipo no implemente el rasgo (*trait*) *Copy***.

### Copiar

Todos los tipos de datos que se almacenan en la pila implementan el rasgo *Copy*. Esto comporta que al asignar una variable enlazada a un tipo de dato que implementa el rasgo *Copy* a otra variable, en vez de moverse la propiedad se crea una copia del dato y se enlaza con la nueva variable.

```rust
fn main () {
    let i: i32 = 10;
    // `i` es la propietaria del dato 10.
    
    let j = i;
    // `j` es la propietaria de otro dato 10,
    // una copia del dato 10 enlazado a `i`.
}
```

Lo mismo sucede cuando pasamos una variable como parámetro de una función.

```rust
fn main () {
    let i: i32 = 10;
	// `i` es la propietaria del dato 10.

    duplica(i);  
}

fn duplica (num: i32) {
    // `num` es la propietaria de otro dato 10
    // una copia del dato 10 enlazado a `i`.
    println!("{} * 2 = {}", num, num * 2);
}
```

```txt
      +----+---+
Pila  | 10 | i |
      +----+---+
      
Al asignar i a j, pasamos a tener:

      +----+---+
      | 10 | j |
Pila  +----+---+
      | 10 | i |
      +----+---+
```

Cuando hay cambios de asignación, Rust trata de copiar el dato y si no puede, mueve su propiedad. Veamos de nuevo el mensaje que nos daba anteriormente el compilador:

```rust
let hola: String = String::from("Hola, mundo");
    ---- move occurs because `hola` has type `String`, which does not implement the `Copy` trait
 let saludo = hola;
              ---- value moved here

 let hola_mundo = hola;
                  ^^^^ value used here after move

// Ahora entendemos la línea 2. Puesto que el tipo de dato String
// no implementa el rasgo Copy, la propiedad se ha movido.
```

```txt
       +---+----+----+--------+
       | - | -  | -  |  hola  |
Pila   +---+----+----+--------+
       | * | 13 | 12 | saludo |
       +-|-+----+----+--------+
         |
         |
       +-v-+---+---+---+---+---+---+---+---+---+---+---+
Montón | H | o | l | a | , |   | m | u | n | d | o |   |
       +---+---+---+---+---+---+---+---+---+---+---+---+
```

> Si quieres ver un listado de los tipos que implementan el rasgo *Copy* y profundizar más en este rasgo, nada mejor que la [documentación oficial](https://doc.rust-lang.org/core/marker/trait.Copy.html#implementors).

## Conceptos clave

* Cada dato tiene una variable enlazada que es propietaria de ese dato.
* Solo puede haber un único propietario de un dato al mismo tiempo.
* Cuando se acaba el ámbito del propietario el dato es eliminado de la memoria.
* Cuando se asigna una variable a otra:
  * Para datos que no implementen el rasgo *Copy* la propiedad se mueve de una variable a la otra
  * Para datos que implementen el rasgo *Copy*, los datos se copian de una variable a la otra

## Enlaces de referencia

Dejo a continuación un listado de todo aquello de lo que me he servido para aprender y poder escribir este apunte. Sincero agradecimiento a cada uno de sus autores.

* [https://www.softax.pl/blog/rust-lang-in-a-nutshell-1-introduction/](https://www.softax.pl/blog/rust-lang-in-a-nutshell-1-introduction/)
* https://blog.thoughtram.io/ownership-in-rust/
* [https://depth-first.com/articles/2020/01/27/rust-ownership-by-example/](https://depth-first.com/articles/2020/01/27/rust-ownership-by-example/)
* [https://medium.com/@bugaevc/understanding-rust-ownership-borrowing-lifetimes-ff9ee9f79a9c](https://medium.com/@bugaevc/understanding-rust-ownership-borrowing-lifetimes-ff9ee9f79a9c)
* [https://medium.com/@thomascountz/ownership-in-rust-part-1-112036b1126b](https://medium.com/@thomascountz/ownership-in-rust-part-1-112036b1126b)
* [https://medium.com/@thomascountz/ownership-in-rust-part-2-c3e1da89956e](https://medium.com/@thomascountz/ownership-in-rust-part-2-c3e1da89956e)
