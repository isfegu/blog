+++
title = "Usar Bulma con Next.js"
summary = "Cómo montar una aplicación básica Next.js que utilice el framework CSS Bulma para facilitar la maquetación."
description = "Cómo montar una aplicación básica Next.js que utilice el framework CSS Bulma para facilitar la maquetación."
slug = "nextjs-bulma"
date = "2022-01-26"
categories = ['nextjs', 'bulma']
tags = ['css', 'sass', 'ssg']
+++

Para un proyecto muy sencillo al que ando dando vueltas me he planteado usar [Next.js](https://nextjs.org) y [Bulma](https://bulma.io/), como excusa para aprender un poquito de ambos. Aquí dejo anotado los pasos que he hecho para tener el entorno de desarrollo montado.

Empezaremos [iniciando una aplicación Next.js](https://nextjs.org/docs/getting-started), en este caso con soporte de Typescript:

```bash
yarn create next-app --typescript
```

> También se puede hacer lo mismo sin Typescript. En ese caso quita el parámetro __--typescript__ y en el resto de esta anotación dónde veas archivos `.tsx`, cámbialo por `.js`:

Seguidamente instalamos Bulma:

```bash
yarn add bulma
```

Ahora toca decidir si queremos utilizar los estilos de Bulma y hacer los ajustes que necesitemos mediante nuestra propia hoja de estilos o vamos un paso más allá y queremos poder modificar los estilos de Bulma mediante la [modificación de sus propiedades](https://bulma.io/documentation/customize/variables/) gracias a [sass](https://sass-lang.com/).

## Opción 1: Usar Bulma y nuestra hoja de estilos

En el archivo `pages/_app.tsx` añadir:

```typescript
import 'bulma/css/bulma.css'
```

Con esto estamos usando la hoja de estilos de Bulma que hay en el directorio `node_modules` (Next.js se encarga de saber que ha de buscarlo allí). También podemos importar la versión comprimida en formato `min`, `bulma.min.css` o la versión `rtl`, `bulma-rtl`.

Ahora ya podemos usar las clases css de Bulma, por ejemplo añadiendo en `pages/index.tsx`:

```typescript
return (
  ...

  <button className="button is-primary">Primary</button>

  ...
)
```

### Opción 1.1: Usar Bulma desde un CDN y nuestra hoja de estilos

Si no queremos usar Bulma desde `node_modules` podemos usar un CDN. En ese caso podemos importar Bulma en el `head` del `html`. En el archivo `pages/index.tsx`:

```typescript
const Home: NextPage = () => {
  return (
    ...
      <Head>
        <title>Create Next App</title>
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@0.9.3/css/bulma.min.css">
      </Head>
    ...
```

O importar Bulma desde nuestra hoja de estilos. En el archivo `styles/styles.css`:

```css
@import "https://cdn.jsdelivr.net/npm/bulma@0.9.3/css/bulma.min.css";
```

## Opción 2: Usar Bulma, adaptarla mediante propiedades y aplicar nuestra hoja de estilos

Primero hemos de instalar sass:

```bash
yarn add sass
```

Añadir al archivo `_app.tsx`:

```typescript
import '../styles/globals.scss'
import '../styles/styles.scss'
```

Seguidamente renombrar `styles/globals.css` por `styles/globals.scss` y `styles/styles.css` por `styles/styles.scss`.

Eliminar el contenido de `styles/globals.scss` y añadir:

```scss
$primary: #900; // Aquí se sobreescriben todas las variables que se quieran

@import 'bulma/bulma.sass';
```

En `styles/styles.scss` añadir:

```scss
@import 'globals.scss'

html {} // Añadir los estilos que se quieran
```
