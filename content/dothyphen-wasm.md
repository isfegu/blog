+++
title = "DotHyphen WebAssembly: De ASCII a Morse y de Morse a ASCII usando WebAssembly (II)"
summary = "Tercera entrega de la serie de anotaciones en las que explico cómo usar una librería desarrollada en Rust desde otros lenguajes. En esta anotación utilizaré DotHyphen desde un runtime WebAssembly en un ecosistema Javascript (Navegador y Node.js)."
slug = "dothyphen-ascii-morse-webassembly-javascript"
date = "2023-10-14"
categories = ['rust']
tags = ['rust', 'webassembly', 'ffi', 'javascript', 'nodejs']
+++

> Tercera entrega de la serie de anotaciones en las que explico cómo usar una librería desarrollada en Rust desde otros lenguajes. Esta interoperabilidad con otros lenguajes es conocida como [FFI](https://en.wikipedia.org/wiki/Foreign_function_interface) (Foreign Function Interface). En esta anotación utilizaré DotHyphen tanto desde un navegador como desde Node.js.
>
> 1. [DotHyphen y DoHy: De ASCII a Morse y de Morse a ASCII usando Rust](/dothyphen-dohy-ascii-morse-rust/).
> 2. [DotHyphen WebAssembly: De ASCII a Morse y de Morse a ASCII usando WebAssembly (I)](/dothyphen-ascii-morse-rust-webassembly-wasi/).
> 3. __DotHyphen WebAssembly: De ASCII a Morse y de Morse a ASCII usando WebAssembly (II).__
>
> Puesto que no soy ningún experto en la materia aviso que este escrito puede contener errores. Como sigo estudiando y aprendiendo lo iré corrigiendo y ampliando. También aviso que he simplificado algunas secciones para facilitar alguna explicación y mi propio entendimiento.

La manera más común de afrontar el uso de una librería desarrollada en Rust desde otros lenguajes, es la de construir un proyecto nuevo (un _crate_ de tipo _lib_) que tenga como dependencia esa librería y añadir cierto código "pegamento" (el _FFI_, en Rust principalmente) cuya finalidad es la de exponer los métodos que se deseen de la librería y lo más importante, hacer de puente entre los tipos de Rust y los del otro lenguaje.

Y con tipos me refiero a los tipos de datos. No hay tanta diferencia entre los tipos básicos y principalmente los numéricos, pero cuando entran en juego las cadenas de texto y los tipos complejos, eso es otra historia (principalmente por cómo se gestionan en memoria).

_DotHyphen WebAssembly_ es ese código "pegamento" que hace que _DotHyphen_ se pueda compilar como _WebAssembly_ para ser usado desde un ecosistema Javascript.

## De Rust a WebAssembly

Tres "piezas" son clave para "transformar" nuestro código Rust en un paquete WebAssembly susceptible de ser ejecutado en un ecosistema Javascript:

### wasm-bindgen

`wasm-bindgen` es un _crate_ que, _grosso modo_, es el encargado que el código WebAssembly pueda ser utilizado desde Javascript (y también Typescript) o que el código WebAssembly pueda utilizar funciones Javascript. Permitiendo principalmente:

* Importar funcionalidades de Javascript en Rust. Como pueden ser la manipulación del DOM o _log_ por consola.
* Exportar funcionalidades del código Rust a Javascript como clases, funciones, etc.
* Trabajar con tipos complejos como cadenas de texto, números, clases, objetos, etc.

Y todo esto, "simplemente", añadiendo el atributo `#[wasm_bindgen]` en el lugar adecuado del código Rust.

A modo de ejemplo. Este código a Rust:

```rust
pub fn sum (x: i32, y: i32) -> i32 {
  x + y
}
```

Le añadimos lo necesario para que se pueda compilar como WebAssembly y ser ejecutado en un ecosistema Javascript:

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn sum (x: i32, y: i32) -> i32 {
  x + y
}
```

El siguiente paso es compilarlo, y para ello entra en juego la segunda "pieza", `wasm32-unknown-unknown`.

### El _target_ `wasm32-unknown-unknown`

`wasm32-unknown-unknown` es el _target_ necesario para compilar Rust a un WebAssembly que pueda ser ejecutado en un ecosistema Javascript. Este _target_ debe instalarse mediante:

```bash
rustup target add wasm32-unknown-unknown
```

Si ahora compilásemos el código de ejemplo:

```bash
cargo build --target wasm32-unknown-unknown
```

Se generaría un paquete `.wasm`. Pero para hacerlo funcionar desde un ecosistema Javascript, aún es necesario cierto código Javascript capaz de "entender" e interactuar con ese paquete `.wasm`. Y para ello entra en juego la tercera "pieza", `wasm-pack`.

### wasm-pack

`wasm-pack` es una herramienta que:

* compila el _crate_ como WebAssembly.
* genera el código Javascript (y Typescript) necesario para poder utilizarlo.
* genera un `package.json` listo para publicar el resultado como paquete npm.

Y todo con un simple comando:

```bash
wasm-pack build
```

Ahora tendríamos un directorio `pkg` con los siguientes archivos:

```plain
pkg/
├── package.json
├── README.md
├── sum_bg.wasm
├── sum.d.ts
└── sum.js
```

El mejor ejemplo de cómo funciona lo explicado es viendo _DotHyphen WebAssembly_.

## DotHyphen WebAssembly

Para poder ofrecer _DotHyphen_ como paquete WebAssembly que pueda ser usado desde un ecosistema Javascript he añadido un _member_ al _workspace_ llamado `dothyphe-wasm`, que es un _crate_ de tipo _lib_ que tiene como dependencia `dothyphen`.

El resultado es un paquete WebAssembly que puede ser utilizado desde un código Javascript ejecutado desde un navegador:

```html
<!DOCTYPE html>
<html lang="en-US">

<head>
  <meta charset="utf-8" />
  <title>dothyphen example</title>
</head>

<body>
  <script type="module">
    import init, { to_morse, to_ascii } from "./lib/dothyphen_wasm.js";
    init().then(() => {
      alert(to_morse("Hello World"));
      alert(to_ascii(".... . .-.. .-.. --- / ... .- -- ..- . .-.."));
    });
  </script>
</body>

</html>
```

 o un paquete npm que puede ser utilizado desde Node.js:

 ```javascript
import * as dothyphen from "@isfegu/dothyphen-wasm";

console.log(dothyphen.to_morse("Hello World"));
console.log(dothyphen.to_ascii(".... . .-.. .-.. --- / ... .- -- ..- . .-.."));
 ```

Mejor que extenderme en esta anotación creo más interesante que le eches un vistazo el repositorio [_Samuel_](https://github.com/isfegu/samuel) y en particular al _member_ `dothyphen-wasm` (y la documentación que lo acompaña).

## Referencias

* <https://developer.mozilla.org/en-US/docs/WebAssembly/Rust_to_wasm>
* <https://rustwasm.github.io/docs/wasm-bindgen/>
* <https://rustwasm.github.io/docs/wasm-pack/>
* <https://rustwasm.github.io/docs/book/>
