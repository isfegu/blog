+++
title = "DotHyphen C: De ASCII a Morse y de Morse a ASCII usando C/C++"
summary = "Cuarta entrega de la serie de anotaciones en las que explico cómo usar una librería desarrollada en Rust desde otros lenguajes. En esta anotación utilizaré DotHyphen desde un código C/C++."
slug = "dothyphen-ascii-morse-c-c++"
date = "2023-12-23"
categories = ['rust']
tags = ['rust', 'webassembly', 'ffi', 'c/c++']
+++

> Cuarta entrega de la serie de anotaciones en las que explico cómo usar una librería desarrollada en Rust desde otros lenguajes. Esta interoperabilidad con otros lenguajes es conocida como [FFI](https://en.wikipedia.org/wiki/Foreign_function_interface) (Foreign Function Interface). En esta anotación utilizaré DotHyphen desde un código C/C++.
>
> 1. [DotHyphen y DoHy: De ASCII a Morse y de Morse a ASCII usando Rust](/dothyphen-dohy-ascii-morse-rust/).
> 2. [DotHyphen WebAssembly: De ASCII a Morse y de Morse a ASCII usando WebAssembly (I)](/dothyphen-ascii-morse-rust-webassembly-wasi/).
> 3. [DotHyphen WebAssembly: De ASCII a Morse y de Morse a ASCII usando WebAssembly (II)](/dothyphen-ascii-morse-webassembly-javascript/).
>
> 4. __DotHyphen C: De ASCII a Morse y de Morse a ASCII usando C/C++__
>
> Puesto que no soy ningún experto en la materia aviso que este escrito puede contener errores. Como sigo estudiando y aprendiendo lo iré corrigiendo y ampliando. También aviso que he simplificado algunas secciones para facilitar alguna explicación y mi propio entendimiento.

La manera más común de afrontar el uso de una librería desarrollada en Rust desde otros lenguajes, es la de construir un proyecto nuevo (un _crate_ de tipo _lib_) que tenga como dependencia esa librería y añadir cierto código "pegamento" (el _FFI_, en Rust principalmente) cuya finalidad es la de exponer los métodos que se deseen de la librería y lo más importante, hacer de puente entre los tipos de Rust y los del otro lenguaje.

Y con tipos me refiero a los tipos de datos. No hay tanta diferencia entre los tipos básicos y principalmente los numéricos, pero cuando entran en juego las cadenas de texto y los tipos complejos, eso es otra historia (principalmente por cómo se gestionan en memoria).

_DotHyphen C_ es ese código "pegamento" que hace que _DotHyphen_ se pueda compilar como librería  estática para ser usada desde C/C++.

## De Rust a C/C++

Para utilizar código Rust desde C/C++ es necesario:

1. definir qué parte de nuestro código será susceptible de ser accesible desde un código C/C++
2. compilar nuestro código Rust como librería estática
3. generar el archivo de las cabeceras (un archivo `.h`) adecuado

### Exponer código

En un crate de tipo _lib_ toda función que se quiera exponer (que queremos que sea accesible desde un código C/C++) debe ser declarada:

1. añadiendo el atributo `#[no_mangle]`
2. como pública
3. añadiendo `extern "C"` en la firma de la función

```rust
#[no_mangle]
pub extern "C" fn sum (x: i32, y: i32) -> i32 {
  x + y
}
```

### Compilar como librería estática

Para generar una librería estática debemos añadir al `cargo.toml` lo siguiente:

```toml
[lib]
name = "XXX"
crate-type = ["staticlib"]
```

Donde _XXX_ debe reemplazarse por el nombre que queramos darle a la librería resultante.

Una vez ejecutemos `cargo build` se generará una librería estática en el directorio `target/<debug|release>/lib<XXX>.a` (_XXX_ será substituido por el nombre dado en el `cargo.toml`). Esta librería es una librería nativa, por tanto es dependiente del sistema operativo en el que ha sido construida. En el caso de GNU/Linux y macOS será un archivo `.a` y en MS Windows un archivo `.lib`. No puedes utilizar una librería de un sistema operativo en otro.

### cbindgen

Por último necesitamos crear el archivo con las cabeceras, el archivo `.h` que acompaña a las librerías nativas. Para esto nos podemos ayudar de [`cbindgen`](https://github.com/mozilla/cbindgen).

`cbindgen` nos ahorra tener que escribir a mano el archivo `.h` adecuado a nuestra librería nativa. Podemos utilizar `cbindgen` como dependencia o como aplicación. En el caso de usarlo como dependencia, deberemos añadirla al `cargo.toml`:

```toml
[build-dependencies]
cbindgen = "0.26"
```

Y añadir en la raíz del crate un archivo `build.rs` con el siguiente contenido:

```rust
extern crate cbindgen;

use std::env;

fn main() {
    let crate_dir = env::var("CARGO_MANIFEST_DIR").unwrap();

    cbindgen::Builder::new()
        .with_crate(crate_dir)
        .generate()
        .expect("Unable to generate bindings")
        .write_to_file("libXXX.h");
}
```

Donde _XXX_ debe reemplazarse por el nombre que le hemos dado a la librería. Cuando ejecutemos `cargo build` automáticamente generará el archivo `.h`.

> En el ejemplo he utilizado una función `sum` que recibe y retorna enteros. Esta es la forma más sencilla de usar Rust como lenguaje para hacer librerías nativas para ser usadas desde C/C++. Pero es solo la punta del iceberg. La diferencia en la gestión de la memoria que tienen ambos lenguajes, así como la diferencia en los tipos de datos comporta que hacer este tipo de librerías no sea algo trivial. En mi caso, tras más de 20 años sin tocar C/C++ he tenido que darme unas cuantas veces contra la pared y leer bastante documentación para conseguir ir más allá de usar enteros y poder llevar _DotHyphen_ a C/C++.

## DotHyphen C

Para poder ofrecer _DotHyphen_ como librería estática que pueda ser usado desde C/C++ he añadido un _member_ al _workspace_ llamado `dothyphen-c`, que es un _crate_ de tipo _lib_ que tiene como dependencia `dothyphen`.

`dothyphen-c` es principalmente un FFI que genera una librería estática llamada `libdotcyphen` y sus cabeceras en el archivo `libdotcyphen.h`.

Mejor que extenderme en esta anotación creo más interesante que le eches un vistazo el repositorio [_Samuel_](https://github.com/isfegu/samuel) y en particular al _member_ `dothyphen-c` (y la documentación que lo acompaña).

## Referencias

* <https://github.com/mozilla/cbindgen/blob/master/docs.md>
* <https://doc.rust-lang.org/nomicon/ffi.html>
* <https://doc.rust-lang.org/std/ffi/index.html>