+++
title = "Next.js y Bulma"
summary = ""
description = ""
slug = "nextjs-bulma"
date = "2022-01-14"
categories = ['']
tags = ['']
draft = true
+++

https://nextjs.org/docs/getting-started

yarn create next-app --typescript

https://bulma.io/

yarn add bulma

## Si se quiere poder modificar los estilos de Bulma mediante propiedades

yarn add sass

Añadir:

import '../styles/globals.scss'
import '../styles/styles.scss'

en _app.tsx

Renombrar '../styles/globals.css' a '../styles/globals.scss'

En '../styles/globals.scss', eliminar el contenido y añadir:

$primary: #900; // Aquí se sobreescriben todas las variables que se quieran

@import 'bulma/bulma.sass';

En '../styles/styles.scss' añadir:

@import 'globals.scss'

html {} // Añadir los estilos que se quieran

## Si no se quiere modificar los estilos de Bulma

yarn add sass

Añadir:

import 'bulma/bulma.sass'

en _app.tsx

## Si no se quiere usar sass