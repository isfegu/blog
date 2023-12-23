+++
title = "DotHyphen WASI: De ASCII a Morse y de Morse a ASCII usando WebAssembly (I)"
summary = "Segunda entrega de la serie de anotaciones en las que explico cómo usar una librería desarrollada en Rust desde otros lenguajes. En esta anotación utilizaré DotHyphen desde un runtime WebAssembly fuera de un ecosistema Javascript."
slug = "dothyphen-ascii-morse-rust-webassembly-wasi"
date = "2023-10-12"
categories = ['rust']
tags = ['rust', 'webassembly', 'wasi', 'ffi']
+++

> Segunda entrega de la serie de anotaciones en las que explico cómo usar una librería desarrollada en Rust desde otros lenguajes. Esta interoperabilidad con otros lenguajes es conocida como [FFI](https://en.wikipedia.org/wiki/Foreign_function_interface) (Foreign Function Interface). En esta anotación utilizaré DotHyphen desde un runtime WebAssembly fuera de un ecosistema Javascript.
>
> 1. [DotHyphen y DoHy: De ASCII a Morse y de Morse a ASCII usando Rust](/dothyphen-dohy-ascii-morse-rust/).
> 2. __DotHyphen WebAssembly: De ASCII a Morse y de Morse a ASCII usando WebAssembly (I).__
> 3. [DotHyphen WebAssembly: De ASCII a Morse y de Morse a ASCII usando WebAssembly (II)](/dothyphen-ascii-morse-webassembly-javascript/).
> 4. [DotHyphen C: De ASCII a Morse y de Morse a ASCII usando C/C++](/dothyphen-ascii-morse-c-c++/).
>
> Puesto que no soy ningún experto en la materia aviso que este escrito puede contener errores. Como sigo estudiando y aprendiendo lo iré corrigiendo y ampliando. También aviso que he simplificado algunas secciones para facilitar alguna explicación y mi propio entendimiento.

El primer paso que voy a dar para utilizar _DotHyphen_ más allá de Rust es utilizarlo desde un entorno de ejecución (_runtime_) WebAssembly. Y no lo haré desde un navegador ([capaces en su mayoría de ejecutar WebAssembly](https://caniuse.com/wasm)) o mediante Node.js (también capaz de ejecutar código WebAssembly), sino desde un entorno de ejecución WebAssembly/WASI instalado en mi ordenador.

## Entorno de ejecución WebAssembly y WASI

Pese a que [WebAssembly](https://webassembly.org/) inició su andadura para ejecutarse en un navegador (ya su nombre nos da una pista), una de sus bondades es su capacidad de portabilidad, ya que un código WebAssembly puede ser ejecutado allí donde haya un entorno de ejecución WebAssembly (algo muy parecido a Java y la máquina virtual de Java). Los navegadores tienen un entorno de ejecución WebAssembly.

WebAssembly no puede, entre otras cosas, acceder a nuestro sistema de ficheros, hacer peticiones por la red o imprimir por pantalla entre otras cosas. Dicho mal y rápido, [WASI](https://wasi.dev/) provee la forma de hacerlo. Es una API de acceso al sistema, que utiliza nuestro código WebAssembly y que deben implementar los entornos de ejecución WebAssembly.

Ahora mismo los entornos de ejecución WebAssembly de los navegadores proveen de funciones de acceso al sistema sin usar WASI.

Hay unos cuantos entornos de ejecución de WebAssembly, los hay para poder ejecutar WebAssembly desde nuestra terminal ([Wasmtime](https://wasmtime.dev/), [Wasmer](https://wasmer.io/), [Wasmedge](https://wasmedge.org), etc) y los hay para ejecutarlo desde un lenguaje de programación (por ejemplo desde un código Rust cargar código WebAssembly).

## De Rust a WebAssembly

Para que un _crate_ sea susceptible de ejecutarse desde un entorno de ejecución WebAssembly/WASI debe contener un archivo `main.rs` con su función `main`.

```rust
pub fn main() {
  println!("Me gusta que los planes salgan bién");
}
```

La manera que tenemos de "transformar" nuestro código Rust en un paquete WebAssembly es compilándolo con el _target_ `wasm32-wasi`. De esta forma Rust tomará todo el código Rust y gracias a WASI creará un paquete WebAssembly capaz, si es necesario, de leer del sistema de ficheros o de imprimir por pantalla.

```bash
~ cargo build --target wasm32-wasi --release
```

Para poder ejecutarlo necesitamos hacerlo mediante el entorno de ejecución WebAssembly/WASI (en este caso Wasmtime).

```bash
~ wasmtime target/wasm32-wasi/release/anibal.wasm
```

### Apunte

Se puede tener un _crate_ de tipo _lib_ y no tener ningún archivo _main_ y compilarlo como WebAssembly con el _target_ `wasm32-wasi`. Y luego ejecutar uno de las funciones públicas expuestas desde el entorno de ejecución WebAssembly.

```rust
#[no_mangle]
pub fn sum (x: i32, y: i32) {
  println!("{}", x + y);
}
```

```bash
~ wasmtime run foo.wasm --invoke sum 3 4
# 7
```

Pero esto sólo es posible cuando las funciones esperan dígitos como parámetros, si son cadenas de texto (o tipos complejos), [entonces no funcionará](https://github.com/wasmerio/wasmer/issues/1205#issuecomment-584785545).

## DotHypen WASI

Para poder ofrecer _DotHyphen_ como paquete WebAssembly que pueda ser usado desde un entorno de ejecución WebAssembly/WASI he añadido un _member_ al _workspace_ llamado `dothyphen-wasi`, que es un _crate_ de tipo _bin_ que tiene como dependencia `dothyphen`.

`dothyphen-wasi` utiliza `clap` y espera los parámetros `--translate` y `--output` al ejecutarse.

```bash
samuel~ wasmtime target/wasm32-wasi/release/dothyphen-wasi.wasm -- --translate "Hello world" --output morse
# .... . .-.. .-.. --- / .-- --- .-. .-.. -..
```

Mejor que extenderme en esta anotación creo más interesante que le eches un vistazo el repositorio [_Samuel_](https://github.com/isfegu/samuel) y en particular al _member_ `dothyphen-wasi` (y la documentación que lo acompaña).
