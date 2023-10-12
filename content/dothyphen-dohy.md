+++
title = "DotHyphen y DoHy: De ASCII a Morse y de Morse a ASCII usando Rust"
summary = "Ésta es la primera de una serie de anotaciones en las que iré explicando cómo usar una librería desarrollada en Rust desde otros lenguajes. Esta interoperabilidad con otros lenguajes es conocida como FFI (Foreign Function Interface)."
slug = "dotyyphen-dohy-ascii-morse-rust"
date = "2023-09-24"
categories = ['rust']
tags = ['rust', 'ffi']
+++

> Ésta es la primera de una serie de anotaciones en las que iré explicando cómo usar una librería desarrollada en Rust desde otros lenguajes. Esta interoperabilidad con otros lenguajes es conocida como [FFI](https://en.wikipedia.org/wiki/Foreign_function_interface) (Foreign Function Interface).
>
> * Parte 1: __DotHyphen y DoHy: De ASCII a Morse y de Morse a ASCII usando Rust.__
>
> Puesto que no soy ningún experto en la materia aviso que este escrito puede contener errores. Como sigo estudiando y aprendiendo lo iré corrigiendo y ampliando. También aviso que he simplificado algunas secciones para facilitar alguna explicación y mi propio entendimiento.

Quiero saber cómo utilizar Rust como lenguaje principal para desarrollar librerías que puedan ser utilizadas desde otros lenguajes. Como necesitaba tener "algo" con lo que probar esta interoperabilidad he desarrollado un sencillo transformador de ASCII a código Morse y de código Morse a ASCII, al que he llamado _DotHyphen_. Mi intención no es desarrollar un gran _crate_ para trabajar con código Morse, esta es la pieza menos importante del proyecto. Lo que me interesa realmente es cómo usarlo desde otros lenguajes y diferentes ecosistemas. A todo este proyecto le he llamado [_Samuel_](https://github.com/isfegu/samuel/).

## DotHyphen

`dothyphen` es un _crate_ sencillo de tipo _lib_ que tiene toda la lógica de transformación expuesta mediante las funciones públicas `translate::to_morse` y `translate::to_ascii`.

```rust
pub fn to_morse(input: &str) -> String
```

```rust
pub fn to_ascii(input: &str) -> String
```

### Recursos

* [Código fuente](https://github.com/isfegu/samuel/tree/main/dothyphen)
* [Documentación](https://docs.rs/dothyphen/latest/dothyphen/)
* [Crate](https://crates.io/crates/dothyphen)

## DoHy

`dohy` es un CLI para `dothyphen`.

```bash
~ dohy --translate "Hello world" --output morse
```

```bash
~ dohy --translate ".... . .-.. .-.. --- / .-- --- .-. .-.. -.." --output ascii
```

### Recursos

* [Código fuente](https://github.com/isfegu/samuel/tree/main/dohy)
* [Crate](https://crates.io/crates/dohy)
