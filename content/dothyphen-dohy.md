+++
title = "DotHyphen y DoHy: De ASCII a Morse usando Rust"
summary = "Ésta es la primera de una serie de anotaciones en las que iré explicando cómo usar una librería desarrollada en Rust desde otros lenguajes. Esta interoperabilidad con otros lenguajes es conocida como FFI (Foreign Function Interface)."
slug = "dotyyphen-dohy-ascii-morse-rust"
date = "2023-09-24"
categories = ['rust']
tags = ['rust', 'ffi']
+++

> Ésta es la primera de una serie de anotaciones en las que iré explicando cómo usar una librería desarrollada en Rust desde otros lenguajes. Esta interoperabilidad con otros lenguajes es conocida como [FFI](https://en.wikipedia.org/wiki/Foreign_function_interface) (Foreign Function Interface).
>
> * Parte 1: DotHyphen y DoHy: De ASCII a Morse usando Rust.
>
> Puesto que no soy ningún experto en la materia aviso que este escrito puede contener errores. Como sigo estudiando y aprendiendo lo iré corrigiendo y ampliando. También aviso que he simplificado algunas secciones para facilitar alguna explicación y mi propio entendimiento.

Para tener "algo" con lo que probar la interoperabilidad entre Rust y otros lenguajes he desarrollado un sencillo transformador de ASCII a código Morse, llamado _DotHyphen_.

## DotHyphen

`dothyphen` es un _crate_ sencillo de tipo _lib_ que tiene toda la lógica de transformación de ASCII a Morse expuesta mediante la función pública `translate`.

```rust
pub fn translate(input: &str) -> String
```

### Recursos

* [Código fuente](https://github.com/isfegu/samuel/tree/main/dothyphen)
* [Documentación](https://docs.rs/dothyphen/latest/dothyphen/)
* [Crate](https://crates.io/crates/dothyphen)

## DoHy

`dohy` es un CLI para `dothyphen`.

```bash
~ dohy --translate "Hello world"
```

### Recursos

* [Código fuente](https://github.com/isfegu/samuel/tree/main/dohy)
* [Crate](https://crates.io/crates/dohy)
