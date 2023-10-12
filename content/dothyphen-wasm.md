+++
title = "DotHyphen WebAssembly: De ASCII a Morse y de Morse a ASCII usando WebAssembly"
summary = "Segunda entrega de la serie de anotaciones en las que iré explicando cómo usar una librería desarrollada en Rust desde otros lenguajes. En esta anotación utilizaremos DotHyphen desde WebAssembly."
slug = "dotyyphen-ascii-morse-webassembly"
date = "2023-10-12"
categories = ['rust']
tags = ['rust', 'webassembly', 'ffi']
draft = true
+++

> Ésta es la segunda de una serie de anotaciones en las que iré explicando cómo usar una librería desarrollada en Rust desde otros lenguajes. Esta interoperabilidad con otros lenguajes es conocida como [FFI](https://en.wikipedia.org/wiki/Foreign_function_interface) (Foreign Function Interface). En esta anotación utilizaremos DotHyphen desde WebAssembly.
>
> * Parte 1: [DotHyphen y DoHy: De ASCII a Morse y de Morse a ASCII usando Rust](https://anedonia.website/dotyyphen-dohy-ascii-morse-rust/).
> * Parte 2: __DotHyphen WebAssembly: De ASCII a Morse y de Morse a ASCII usando WebAssembly.__
>
> Puesto que no soy ningún experto en la materia aviso que este escrito puede contener errores. Como sigo estudiando y aprendiendo lo iré corrigiendo y ampliando. También aviso que he simplificado algunas secciones para facilitar alguna explicación y mi propio entendimiento.

La manera más común de afrontar el uso de una librería desarrollada en Rust desde otros lenguajes, es la de construir un proyecto nuevo (un _crate_ de tipo _lib_) que tenga como dependencia esa librería y añadir cierto código "pegamento" (en Rust, principalmente) cuya finalidad es la de exponer los métodos que se deseen de la librería (_FFI_) y lo más importante, hacer de puente entre los tipos de un lenguaje (Rust) y el otro. Y con tipos me refiero a los tipos de datos (y cómo se gestionan en memoria). No hay tanta diferencia en tipos básicos y principalmente los numéricos, pero cuando entran en jeugo las cadenas de texto, eso es otra historia.

## DotHyphen WebAssembly


## Referencias

* https://developer.mozilla.org/en-US/docs/WebAssembly/Rust_to_wasm
* https://rustwasm.github.io/docs/wasm-bindgen/
* https://rustwasm.github.io/docs/wasm-pack/
* https://rustwasm.github.io/docs/book/
