+++
title = "Borrowing: o cómo se gestiona la memoria en Rust (parte II)"
summary = "En esta segunda entrega de mis anotaciones respecto la gestión de la memoria en Rust trato de explicar y entender cómo funciona el Copiar, el Mover y los Préstamos."
slug = "borrowing-gestion-de-memoria-rust"
date = "2021-12-27"
categories = ['rust']
tags = ['rust', 'memoria', 'borrowing', 'préstamos']
+++

> Este apunte es la segunda entrega de una serie de anotaciones que estoy haciendo respecto la gestión de la memoria en Rust.
>
> * [Parte 1: Ownership](../ownership-gestion-de-memoria-rust)
> * Parte 2: Borrowing
> Puesto que no soy ningún experto en la materia aviso que este escrito puede contener errores. Como sigo estudiando y aprendiendo lo iré corrigiendo y ampliando. También aviso que he simplificado algunas secciones para facilitar alguna explicación y mi propio entendimiento.

Esta es una lista de palabras inglesas muy utilizadas en el ámbito de la gestión de la memoria en Rust con la traducción que he utilizado para este documento.

* ***Borrowing***: Préstamo.
* ***Dereference***: Desreferenciar. Esta palabra no existe en español, pero la usaré.
* ***Owner***: Propietario.
* ***Ownership***: Propiedad.
* ***Reference***: Referencia.
* ***Trait***: Rasgo.

## Copiar y mover

En el [anterior apunte](../ownership-gestion-de-memoria-rust) comenté que cuando asignamos una variable a otra pueden suceder dos cosas:

1. si el tipo del dato **implementa** el rasgo (*trait*) *Copy*, se **crea una copia** en memoria del dato original y se enlaza a la nueva variable.
2. si el tipo del dato **no implementa** el rasgo *Copy*, se rompe el enlace con la actual variable y se crea un enlace con la nueva, haciendo que la propiedad (*ownership*) **se mueva** de una variable a la otra.

En el primer caso podemos seguir accediendo al dato original mediante la variable original, pero en el segundo caso no podemos usar la variable original ya que dejó de ser propietaria del dato, y solo podemos hacerlo a través de la segunda variable (la nueva propietaria).

En este escenario nos puede surgir una pregunta, ¿hay alguna manera de seguir usando el dato y la variable original tras una asignación?. La respuesta es, sí. Y podemos hacerlo de dos maneras, **implementando el rasgo *Copy*** o mediante los **préstamos** (*borrowing*).

## Implementar el rasgo Copy

Implementar el rasgo *Copy* significa que a partir de ese momento cuando se realize una asginación, en vez de que se mueva la propiedad se hagan copias. ¿Fácil?. Sí, con un pero. Solo podemos hacer que implementen el rasgo *Copy* los tipos que como programador podemos crear mediante estructuras (structs). No podemos implementarlo en los tipos ofrecidos por el lenguaje. Si el tipo *Vector* de Rust no implementa el rasgo *Copy*, no hay manera de hacer que lo implemente.

```rust
struct Persona {
    edad: i32
}

fn main () {
    let mut amparo = Persona { edad: 40 };
    
    mostrar_edad(amparo);
    
    amparo.edad = 41;
    
    mostrar_edad(amparo);
}

fn mostrar_edad(persona: Persona) {
    println!("Esta persona tiene {} años", persona.edad);
}

// Al ejecutar este código obtendremos el, ya conocido, resultado:

let mut amparo = Persona { edad: 40 };
      ---------- move occurs because `amparo` has type `Persona`, which does not implement the `Copy` trait

mostrar_edad(amparo);
             ------ value moved here

amparo.edad = 41;
^^^^^^^^^^^^^^^^ value partially assigned here after move
```

Implementemos ahora el rasgo *Copy* de la siguiente manera:

```rust
#[derive(Clone, Copy)]
struct Persona {
    edad: i32
}

fn main () {
    let mut amparo = Persona { edad: 40 };
    
    mostrar_edad(amparo);
    
    amparo.edad = 41;
    
    mostrar_edad(amparo);
}

fn mostrar_edad(persona: Persona) {
    println!("Esta persona tiene {} años", persona.edad);
}

// Al ejecutar este código obtendremos el siguiente resultado:

Esta persona tiene 40 años
Esta persona tiene 41 años
```

Ha sido muy sencillo, simplemente añadiendo *#[derive(Clone, Copy)]* antes de la declaración de la estructura, implementamos el rasgo *Copy* y el dato pasa de moverse a copiarse.

Y ahora nos puede surgir otra pregunta, ¿por qué no hacemos que todo se copie?. Primero que no podemos hacer que los tipos que proporciona Rust que no implementan el rasgo *Copy* pasen a implementarlo, y segundo, copiando estamos gastando tiempo y memoria. Para qué has de invertir el tiempo del procesador y la memoria en copiar un dato cada vez que hay un cambio de asignación cuando, por ejemplo, simplemente quieres que una función lea ese dato (como en el anterior *mostrar_edad()*).

Para estas y otras razones existen los préstamos (*borrowing*).

## Préstamos (*Borrowing*)

El propietario de un dato puede prestar el acceso a ese dato a otras variables sin perder en ningún momento su propiedad. Cada préstamo es un puntero al dato. A esos punteros se les denomina referencias (*references*).

```txt
       +---+----+-----+------+---+----+----+--------+
Pila   | * | 13  | 12 | hola | * | 13 | 12 | saludo |
       +-|-+----+-----+------+-|-+----+----+--------+
         |                     |
         | /-------------------/
         | |
         |/
       +-v-+---+---+---+---+---+---+---+---+---+---+---+
Montón | H | o | l | a | , |   | m | u | n | d | o |   |
       +---+---+---+---+---+---+---+---+---+---+---+---+

La variable 'hola' es la propietaria del dato "Hola, mundo"
y le presta el acceso a la variable 'saludo'.

Tanto 'hola' como 'saludo' apuntan al mismo dato.

La variable 'saludo' tiene una referencia (puntero) al dato.
```

Los préstamos se hacen mediante el operador **&**.

```rust
fn main () {
    let nombre: String = String::from("Amparo");
    // `nombre` es la propietaria del dato "Amparo".

    saludar(&nombre);  
    // Al añadir & en esta asignación implícita le estamos
    // pasando a la función una referencia al dato.
    // Lo estamos prestando.
    
    despedir(&nombre);
    // Aquí hacemos otro préstamo. Hemos hecho dos
    // préstamos.
    
    let palabra = &nombre;
    // Aquí hacemos un préstamo en una asignación explícita.
    
    println!("nombre es {}, palabra es {}", nombre, palabra);
    // `nombre` sigue siendo la propietaria y podemos usarla
    // después de haber sido asignada 3 veces anteriormente.
}

// En el caso de una asignación implícita, la función
// que recoge el dato también debe usar el operador &
// ya que el tipo de la variable no es de tipo String
// es del tipo Referencia a String.
fn saludar (persona: &String) {
    println!("Hola, {}", persona);
}

// De nuevo usamos el operador &
fn despedir (persona: &String) {
    println!("Adiós, {}", persona);
}

// La ejecución de este código nos devolverá:
Hola, Amparo
Adiós, Amparo
nombre es Amparo, palabra es Amparo
```

Los préstamos se pueden hacer de cualquier tipo de dato. No está limitado a tipos de datos que no implementen el rasgo *Copy* o basados en estructuras. Podemos tener préstamos de datos tanto de la pila como del montón.

```rust
fn main () {
    let i: i8 = 10;
    let j = &i;
    let k = &i;
    
    println!("i = {}, j = {}, k = {}, j + k = {}", i, j, k, j + k);
}

// Le ejecución nos devolverá:
i = 10, j = 10, k = 10, j + k = 20
```

```rust
// Veamos cómo haríamos el ejemplo que hemos visto
// en el apartado anterior sin usar el rasgo Copy.
struct Persona {
    edad: i32
}

fn main () {
    let mut amparo = Persona { edad: 40 };
    
    mostrar_edad(&amparo);
    
    amparo.edad = 41;
    
    mostrar_edad(&amparo);
}

fn mostrar_edad(persona: &Persona) {
    println!("Esta persona tiene {} años", persona.edad);
}

// Al ejecutar este código obtendremos el siguiente resultado:

Esta persona tiene 40 años
Esta persona tiene 41 años
```

No está de más recalcar que el tipo de una variable que guarda una referencia **no** es el mismo que el del dato al que referencia, su tipo es **referencia del tipo**.

```rust
fn main () {
    let i: i8 = 10; // i es de tipo i8 (entero de 8 bits)
    let j: &i8 = &i; // j es de tipo &i8 (referencia a un entero de 8 bits)
    let k: i8 = &i; // k es de tipo i8
    
    println!("i = {}, j = {}, k = {}, j + k = {}", i, j, k, j + k);
}

// Al ejecutar este código obtendremos el siguiente resultado:
let k: i8 = &i;
       --   ^^
        |    |
        |    expected `i8`, found `&i8`
        |    help: consider removing the borrow: `i`
        expected due to this

// De nuevo el compilador a nuestro rescate.
// La línea 13 nos dice qué sucede.
```

## Referencias mutables e inmutables

Al igual que hay variables mutables e inmutables, lo mismo sucede con los préstamos. Existen referencias mutables y referencias inmutables. Una referencia inmutable no puede modificar el dato prestado y una mutable sí que puede.

### Referencias inmutables

Son esas referencias que pueden acceder al dato pero no modificarlo. Se declaran usando el **&**.

```rust
fn main () {
    let i: i8 = 10;
    let j = &i;
    
    // Como j es del tipo &i8, espera almacenar un
    // dato del tipo referencia, por tanto le asignamos
    // una referencia a un número en este caso &4.
    j = &4;  
}

// Al querer darle el valor 4 a la variable j
// el compilador nos da el siguiente error:

let j = &i;
    -
    |
   first assignment to `j`
   help: make this binding mutable: `mut j`
    
j = &4;
^^^^^^ cannot assign twice to immutable variable

// Nos avisa de que una variable con una referencia inmutable no puede
// modificar el dato al que está referenciando.
```

Pueden haber tantas referencias inmutables como se quieran:

```rust
fn main () {
    let i: i8 = 10;
    let j = &i;
    let k = &i;
    let l = &i;
    
    println!("j: {}, k: {}, l: {}", j, k, l);
}

// El resultado es:

j: 10, k: 10, l: 10
```

### Referencias mutables

Son esas referencias que pueden acceder al dato y modificarlo. Se declaran usando **&mut**.

> Para que una referencia mutable pueda modficar el dato, el propietario del dato también tiene que ser mutable.

```rust
fn main () {
    let mut i: i8 = 10;
    let j = &mut i;

    println!("j: {}", j);

    *j = 4; // Esto lo comentó en la siguiente sección.
    
    println!("j: {}", j);
}

// Al ejecutar este código obtendremos el siguiente resultado:
j: 10
j: 4
```

Solo puede haber una referencia mutable a un mismo dato. Y, simultáneamente, pueden haber N referencias inmutables o una referencia mutable. Nunca se puede tener una referencia mutable y una referencia inmutable al mismo dato.

```rust
fn main () {
    let mut i: i8 = 10;
    let j = &mut i; // Creamos una referencia mutable
    let k = &i; // Creamos una referencia inmutable

    println!("j: {}", j);
    println!("k: {}", k);
}

// Al ejecutar este código obtendremos el siguiente resultado:

error[E0502]: cannot borrow `i` as immutable because it is also borrowed as mutable
let j = &mut i;
		------ mutable borrow occurs here
let k = &i;
        ^^ immutable borrow occurs here
println!("j: {}", j);
                  - mutable borrow later used here

// El compilador nos informa de la doble referencia
// mutable e inmutable.
```

## Desrefenciar (Dereference)

Una variable es un enlace a una parte de la memoria donde se almacena un dato. Una referencia es un enlace a esa misma parte de la memoria. Por lo tanto una referencia es una dirección de memoria, no es el dato en sí. Cuando una variable es una referencia mutable, si queremos manipular el dato al que apunta debemos "desreferenciar" la variable mediante un __*__ (llamado operador de indirección).

```rust
fn main () {
    let mut i: i8 = 10;
    let j = &mut i;

    println!("j: {}", j);

    *j = 4;
    // Mediante el operador de indirección
    // podemos acceder al dato.
    
    println!("j: {}", j);
}

// Al ejecutar este código obtendremos el siguiente resultado:

j: 4
```

```rust
fn main () {
    let mut i: i8 = 10;
    let j = &mut i;

    println!("j: {}", j);

    j = 4; // No usamos el operador de indirección
    
    println!("j: {}", j);
}

// Al ejecutar este código obtendremos el siguiente resultado:

j = 4;
    ^ expected `&mut i8`, found integer

help: consider dereferencing here to assign to the mutable borrowed piece of memory
*j = 4;

// j = 4 nos da un error ya que el tipo de j es una referencia mutable de i8
// y le estamos asignando un entero. El mismo compilador nos ofrece la solución.
```

## Copiar y mover referencias

Las referencias son variables y como tal se pueden asignar unas a otras. Y como ya expliqué en el [apunte anterior](../ownership-gestion-de-memoria-rust), asignar (implícita o explícitamente) variables unas a otras, en Rust, tiene sus implicaciones. Que son las siguientes:

### Las referencias inmutables son copiadas

```rust
fn main() {
    let saludo = String::from("Hola");
    let prestamo_1 = &saludo; // Referencia inmutable
    println!("préstamo 1: {}", prestamo_1); // Asignación implícita
    
    let prestamo_2 = prestamo_1; // Asignación explícita
    // Hemos podido asignar prestamo_1 a prestamo_2 sin que el compilador
    // nos avise de ningún error, incluso habiendo hecho una asignación 
    // implícita anteriormente. prestamo_2 tiene una copia de la
    // referencia que tiene prestamo_1.

    println!("préstamo 1: {}, préstamo 2: {}", prestamo_1, prestamo_2);
    println!("préstamo 2: {}", prestamo_2);
    // Podemos seguir haciendo asignaciones implícitas sin ningún problema,
    // Las referencias siguen copiándose.    
}

// Al ejecutar este código obtendremos el siguiente resultado:

préstamo 1: Hola
préstamo 1: Hola, préstamo 2: Hola
préstamo 1: Hola
```

```txt
       +---+---+---+--------+---+---+---+------------+---+---+---+------------+
Pila   | * | 5 | 4 | saludo | * | 5 | 4 | prestamo_1 | * | 5 | 4 | prestamo_2 |
       +-|-+---+---+--------+-|-+---+---+------------+-|-+---+---+------------+
         |                    |                        |
         | /------------------/                        |
         | | |-----------------------------------------/
         |/ /
       +-v-+---+---+---+---+
Montón | H | o | l | a |   |
       +---+---+---+---+---+

Tanto prestamo_1 como prestamo_2 son referencias al mismo dato
del que es propietaria la variable saludo.
```

### Las referencias mutables son movidas

```rust
fn main() {
    let mut saludo = String::from("Hola");
    let prestamo_1 = &mut saludo; // Referencia mutable
    let prestamo_2 = prestamo_1; // Asignación explícita

    *prestamo_1 = String::from("Amparo");
    // Modificamos el dato al que hace referencia prestamo_1

    println!("préstamo 1: {}", prestamo_1); // Asignación implícita
    println!("préstamo 2: {}", prestamo_2); // Asignación implícita
    println!("saludo: {}", saludo); // Asignación implícita
}

// Al ejecutar este código obtendremos el siguiente resultado:

let prestamo_1 = &mut saludo;
    ---------- move occurs because `prestamo_1` has type `&mut String`, which does not implement the `Copy` trait
let prestamo_2 = prestamo_1;
                 ---------- value moved here

*prestamo_1 = String::from("Amparo");
^^^^^^^^^^^ value used here after move

// El compilador nos avisa que no podemos usar prestamo_1 para
// modificar el dato ya que la referencia mutable se movió a
// prestamo_2.
// Sucede exactamente igual como hemos visto con la transferencia
// de la propiedad.

```

## Conclusión

En Rust todo dato tiene un único propietario y ese propietario puede prestar el acceso a ese dato tantas veces como sea necesario teniendo en cuenta que simultáneamente no puede haber un préstamo mutable y préstamos inmutables, pero sí únicamente uno o varios préstamos inmutables.