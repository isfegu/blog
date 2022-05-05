+++
title = "Tipos compuestos y colecciones en Rust"
summary = "Tanto los tipos compuestos como las colecciones son tipos de datos que agrupan múltiples valores (de un mismo o diferente tipo). La diferencia principal entre ambos es que los tipos compuestos se almacenan en la pila y tienen un tamaño conocido en tiempo de compilación mientras que las colecciones se almacenan en el montón y su tamaño puede variar en tiempo de ejecución."
description = "En rust, los tipos compuestos y las colecciones son tipos de datos que agrupan múltiples valores."
slug = "rust-tipos-compuestos-colecciones"
date = "2022-04-09"
categories = ['rust']
tags = ['rust', 'tipos compuestos', 'colecciones']
draft = true
+++

Tanto los __tipos compuestos__ como las __colecciones__ son tipos de datos que agrupan múltiples valores (de un mismo o diferente tipo). La diferencia principal entre ambos es que los tipos compuestos se almacenan en la pila y tienen un tamaño conocido en tiempo de compilación mientras que las colecciones se almacenan en el montón y su tamaño puede variar en tiempo de ejecución.

## Tipos compuestos

### Tupla

Una tupla almacena valores que pueden ser de diferente tipo. A cada valor se le llama __elemento__.

Una tupla se inicializa de la siguiente manera:

```rust
let tupla = ('A', "Alfa", 10i32, true);
```

Una tupla tiene un tamaño fijo y no puede aumentar o disminuir a lo largo de la ejecución del programa.

Para acceder a los valores de una tupla se usa un índice que indica su posición dentro de la tupla. El primer elemento de la tupla tiene el índice 0, el segundo el 1 y así sucesivamente.

```rust
println!("{:?}", tupla.0); // 'A'
println!("{:?}", tupla.1); // "Alfa"
println!("{:?}", tupla.2); // 10
println!("{:?}", tupla.3); // true
```

Tratar de acceder a un valor mediante un índice igual o mayor que el tamaño de la tupla dará un error en tiempo de compilación.

```rust
println!("{:?}", tupla.4);

// Al compilar este código obtenemos el siguiente error:

error[E0609]: no field `4` on type `(char, &str, i32, bool)`
 --> src/main.rs:8:28
  |
  |     println!("{:?}", tupla.4);
  |      

```

Es posible modificar el valor de un elemento.

```rust
tupla.2 = 6i32;
println!("{:?}", tupla.2); // Mostrará 6
```
Pero ha de ser por un valor del mismo tipo. La inicialización de una tupla marca el tipo de cada elemento a lo largo de todo el código. Y éste es evaluado en tiempo de compilación.

```rust
tupla.2 = "Beta";
println!("{:?}", tupla.2);

error[E0308]: mismatched types
 --> src/main.rs:8:15
  |
  |     tupla.2 = "Beta";
  |     -------   ^^^^^^ expected `i32`, found `&str`
  |     |
  |     expected due to the type of this binding
```

### Matriz

Una matriz almacena valores de un mismo tipo. A cada valor se le llama __elemento__.

Hay dos formas de inicializar una matriz:

- Como una lista de valores separados por comas.
- Especificando el valor inicial, seguido de punto y coma y, después, la longitud de la matriz.

```rust
let matriz_1 = ["Alfa", "Beta", "Gamma", "Delta", "Epsilon"];
let matriz_2 = [7; 10]; // Inicializa una matriz de 10 elementos en el que cada uno de ellos tiene el valor 7
```
Una matriz tiene un tamaño fijo y no puede aumentar o disminuir a lo largo de la ejecución del programa.

Para acceder a los valores de una matriz se usa un índice que indica su posición dentro de la tupla. El primer elemento de la matriz tiene el índice 0, el segundo el 1 y así sucesivamente.

```rust
println!("{:?}", matriz_1[0]); // "Alfa"
println!("{:?}", matriz_1[1]); // "Beta"
println!("{:?}", matriz_1[2]); // "Gamma"
println!("{:?}", matriz_1[3]); // "Delta"
println!("{:?}", matriz_1[4]); // "Epsilon"
```

Tratar de acceder a un valor mediante un índice igual o mayor que el tamaño de la matriz dará un error en tiempo de compilación.

```rust
println!("{:?}", matriz_1[8]);

// Al compilar este código obtenemos el siguiente error:

error: this operation will panic at runtime
  --> src/main.rs:10:18
   |
   | println!("{:?}", matriz_1[8]);
   |                  ^^^^^^^^^^^ index out of bounds: the length is 5 but the index is 8
   |
```

Es posible modificar el valor de un elemento (siempre que la matriz sea declarada como mutable, `mut`).

```rust
matriz_1[2] = "Phi";
println!("{:?}", matriz_1[2]); // Mostrará "Phi"
```
Pero ha de ser por un valor del mismo tipo. La inicialización de una matriz marca el tipo de cada elemento a lo largo de todo el código. Y éste es evaluado en tiempo de compilación.

```rust
matriz_1[2] = 10i32;
println!("{:?}", matriz_1[2]);

error[E0308]: mismatched types
  --> src/main.rs:10:15
   |
   | matriz_1[2] = 10i32;
   | -----------   ^^^^^ expected `&str`, found `i32`
   | |
   | expected due to the type of this binding
```

## Colecciones

### Vector

Un vector, `Vec<T>`, almacena valores de un mismo tipo.

Hay dos formas de declarar un vector:

- Mediante el método `new()` de la estructura `Vec`.

  ```rust
  let v = Vec::new();  
  ```

- Mediante la macro `vec!`.

  ```rust
  let v = vec![val1, val2, val3];
  ```

A diferencia de una matriz, un vector no tiene un tamaño fijo, por lo tanto pueden agregarse o eliminarse elementos a lo largo de la ejecución del programa.

Se usa el método `push` para añadir elementos.

```rust
let mut primos = Vec::new();
primos.push(2);
primos.push(3);
primos.push(5);

// El contenido de primos es:
// [2, 3, 5]
```

Solo se pueden añadir elementos del tipo adecuado. El tipo lo marca el primer elemento que se añade.

```rust
primos.push(3.4);

// Al compilar este código obtenemos el siguiente error:

  |
  |     primos.push(3.4);
  |                 ^^^ expected integer, found floating-point number

// El mismo error nos encontraremos a declarar un vector con la macro vec!

let mut primos = vec![2,3,5.5,7];

  |
  |     let mut primos = vec![2,3,5.5,7];
  |                               ^^^ expected integer, found floating-point number
```

Para acceder a los elementos de un vector se usa el índice que lo posiciona dentro del vector. El primer elemento tiene el índice 0, el segundo el 1 y así sucesivamente.

```rust
println!("{:?}", primos[0]); // 2
println!("{:?}", primos[1]); // 3
println!("{:?}", primos[2]); // 5
println!("{:?}", primos[4]); // 7
```

Tratar de acceder a un elemento mediante un índice igual o mayor que el tamaño del vector resultará en un error en tiempo de ejecución.

```rust
println!("{:?}", primos[9]);

// Al ejecutar este código obtendremos el siguiente error:

thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 9'
```

Para eliminar elementos se usa el método `remove` especificando el índice del elemento a eliminar.

```rust 
let mut primos = vec![2,3,5];

primos.remove(1);

// El contenido de primos es:
// [2, 5]
```

Tratar de eliminar un elemento mediante un índice igual o mayor que el tamaño del vector resultará en un error en tiempo de ejecución.

```rust 
let mut primos = vec![2,3,5];

primos.remove(4);

// Al ejecutar este código obtendremos el siguiente error:

thread 'main' panicked at 'removal index (is 4) should be < len (is 3)',
```

Es posible modificar el valor de un elemento (siempre que el vector sea declarado como mutable, `mut`).

```rust
primos[2] = 9;

// El contenido de primos es:
// [2, 3, 9]
```

### Mapa Hash

El tipo `HashMap<K, V>` almacena datos asignando cada clave _K_ con su valor _V_.

Un mapa hash se inicializa de la siguiente manera:

```rust
use std::collections::HashMap;
let mut films: HashMap<String, String> = HashMap::new(); 
// En este caso tanto la clave como el valor son del tipo String, pero pueden ser de cualquier otro tipo.
```

Un mapa hash no tiene un tamaño fijo, éste puede aumentar o dismuir a lo largo de la ejecución del programa.

Se usa el método `insert` para añadir elementos y el método `remove` para eliminarlos.

```rust
films.insert(String::from("El bueno, el feo y el malo"), String::from("Sergio Leone"));
films.insert(String::from("La noche del cazador"), String::from("Charles Laughton"));
films.insert(String::from("Los bingueros"), String::from("Mariano Ozores"));

// El contenido de films es:
// {"El bueno, el feo y el malo": "Sergio Leone", "Matar a un ruiseñor": "Robert Mulligan", "Los bingueros": "Mariano Ozores"}

films.remove("Los bingueros");

// El contenido de films es:
// {"El bueno, el feo y el malo": "Sergio Leone", "Matar a un ruiseñor": "Robert Mulligan"}
```

Solo se pueden añadir claves y valores del tipo estipulado en la declaración del mapa hash.

```rust
films.insert(String::from("Siete"), 7);

// Al compilar este código obtenemos el siguiente error:

   |
   | films.insert(String::from("Siete"), 1);
   |                                     ^- help: try using a conversion method: `.to_string()`
   |                                     |
   |                                     expected struct `String`, found integer
```

Se usa el método _get_ para obtener el valor asignado a una clave.

```rust
println!("{}", films.get("Los bingueros"));

// Nos devolverá:
// Some("Mariano Ozores")
```