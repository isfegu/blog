+++
title = "Static Site Generation con Next.js y SQLite"
summary = "Cómo utilizar SQLite desde Next.js para obtener datos que sirvan para generar contenido a la hora de construir páginas generadas estáticamente."
slug = "nextjs-sqlite-ssg-getstaticprops"
date = "2021-12-29"
categories = ['nextjs', 'sqlite']
tags = ['sqlite', 'ssg', 'getstaticprops']
+++

Tengo un proyecto web en mente que quiero que sea gestionado por un Static Site Generator. Inicialmente pensé e incluso empecé a hacerlo con [Hugo](https://gohugo.io) (el SSG que utilizo en este blog), pero vi que se me quedaba corto y buscando alternativas encontré [Next.js](https://nextjs.org). En el momento de escribir estas líneas parece ser que está muy de moda y pese estar en Javascript (lenguaje y ecosistema en el que nunca me he sentido cómodo), es el que me ofrece justo lo que necesito; la creación dinámica en tiempo de compilación de páginas y rutas (URLs) complejas que obtienen sus datos de archivos markdown y json que están en una estructura de directorios peculiar.

En este apunte no voy a explicar los pros y contras de Next.js, sino explicar algo que me ha hecho invertir mucho tiempo y quiero dejarlo por escrito para mi mismo y para aquel que le pueda interesar: Usar SQLite como fuente de datos para llenar de contenido las páginas que se generarán desde Next.js.

Leer una base de datos SQLite desde Javascript ejecutado desde Node.js es fácil. Simplemente usando, por ejemplo, [better-sqlite3](https://github.com/JoshuaWise/better-sqlite3). Según su documentación:

```javascript
const db = require('better-sqlite3')('foobar.db', options);

const row = db.prepare('SELECT * FROM users WHERE id = ?').get(userId);
console.log(row.firstName, row.lastName, row.email);
```

Pero esto mismo ejecutado en Next.js __no funciona__. Si ya sabes porqué no sigas leyendo, yo no tenía ni idea y mira que es obvio (cuando lo sabes).

No funciona porque intentas acceder a una base de datos que es un archivo que está en el sistema de ficheros y Next.js se ejecuta en un navegador y un navegador no tiene acceso al sistema de ficheros. Y pensé, pero si lo que yo quiero es usar Next.js para que me genere las páginas estáticas en tiempo de compilación, no quiero un navegador. Pues no, Next.js usa por debajo React y se ejecuta en un navegador.

Para entender claramente esta circunstancia nada mejor que la explicación oficial. Es indispensable su [explicación de pre-rendering, Static Generation y Server-side Rendering](https://nextjs.org/learn/basics/data-fetching/pre-rendering).

Y de esa explicación extraigo dos párrafos que me dan la pista de cómo hacer que funcione la conexión con SQLite:

> However, for some pages, you might not be able to render the HTML without first fetching some external data. Maybe you need to access the file system, fetch external API, or query your database at build time. Next.js supports this case — Static Generation with data — out of the box.

> Essentially, getStaticProps allows you to tell Next.js: “Hey, this page has some data dependencies — so when you pre-render this page at build time, make sure to resolve them first!”.

## getStaticProps, obtener datos antes de hacer el pre-render

getStaticProps es la manera que ofrece Next.js para obtener datos externos antes de hacer el pre-renderizado. Ese getStaticProps sucede en tiempo de construcción y no depende del navegador, es Node.js quien está detrás, por tanto se puede acceder al sistema de ficheros.

De nuevo, seguir la [explicación oficial sobre getStaticProps y Static Generation](https://nextjs.org/learn/basics/data-fetching/with-data), es mejor que cualquier cosa que pueda escribir yo mismo.

## Usar SQLite en getStaticProps

Con dos archivos tenemos suficiente, ``lib/db.js``:

```javascript
const sqlite = require('better-sqlite3')

export default function getDistros() {
    const db = sqlite('database/db.sqlite')
    const statement = db.prepare("SELECT * FROM distro")
    const distros = statement.all()
    return distros
}
```

y ``pages/distros.js``:

```javascript
import getDistros from "../lib/db"

export default function Distros(props) {
    const { distros } = props
    return (
        <>
            <h1>Distribuciones GNU/Linux</h1>
            <p>El siguiente listado se obtiene de una base de datos SQLite:</p>
            <ul>
                {
                    distros.map(
                        (item) => <>
                            <li key={item.id}>{item.name} {item.version}</li>
                        </>
                    )
                }
            </ul>

        </>
    )
}

export async function getStaticProps() {
    const distros = getDistros()
    return {
        props: {
            distros
        }
    }
}
```

En ``lib/db.js`` se expone el método ``getDistros()`` que hace la conexión contra la base de datos y obtiene los datos de una tabla. En ``pages/distros.js`` se crea una página que muestra los datos obtenidos de la tabla gracias a la implementación de ``getStaticProps()``.

Para no alargar este apunte he [publicado todo el código fuente del uso de SQLite desde Next.js](https://github.com/isfegu/nextjs-sqlite).

## Referencias

* Recomiendo a aquel que quiera introducirse en Next.js el [tutorial que hay es su propia página web](https://nextjs.org/learn/basics/create-nextjs-app).
* Un vídeo que explica cómo [conectar Next.js con MySQL](https://www.youtube.com/watch?v=uqlQB0oHSEE).
* [Este artículo](https://maikelveen.com/blog/how-to-solve-module-not-found-cant-resolve-fs-in-nextjs) tiene explicación que me ha sido muy útil para entender el funcionamiento de Next.js.
