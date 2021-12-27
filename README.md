# Blog

Este repositorio contiene el código fuente y el contenido de mi blog personal.

Este blog está gestionado mediante [Hugo](https://gohugo.io) y contiene, por un lado, una serie de anotaciones útiles en mi largo, lento y tortuoso camino de aprendizaje de Rust y por otro lado enlaces que encuentro interesantes o útiles.

## Entornos

### Desarrollo

Para montar este blog en un entorno de desarrollo es necesario:

* Instalar [Hugo](https://gohugo.io/getting-started/installing/).
* Clonar este mismo repositorio.

### Producción

Este blog se publica en Netlify de forma automática cada vez que se hace un _push_ a _main_ a modo de [Continuous Deployment](https://www.atlassian.com/continuous-delivery/continuous-deployment). La URL de este blog es [https://isfegu.netlify.app/](https://isfegu.netlify.app/).

[![Netlify Status](https://api.netlify.com/api/v1/badges/f1c7c22a-a9d6-4fb8-8972-c85810559521/deploy-status)](https://app.netlify.com/sites/isfegu/deploys)

## Contribuir

Cualquier aportación, a modo de _pull request_ será bienvenida.

Una vez se tiene el [entorno de desarrollo](#Development) se puede contribuir de dos formas, mejorando el aspecto visual o mejorando/aportando contenidos.

### Aspecto visual

Para hacer cambios en el aspecto visual el directorio que te interesa es `themes/hyde` y la documentación de Hugo al respecto de los _[Templates](https://gohugo.io/templates/)_.

### Contenidos

Para mejorar o aportar contenidos el directorio que te interesa es `content` y la documentación de Hugo al respecto del _[Content Management](https://gohugo.io/content-management/)_.

A modo de rápido resumen, en el directorio `content` hay tantos archivos _Markdown_ como entradas en el blog. A parte del contenido de la entrada, cada archivo tiene una cabecera, conocida como _[Front Matter](https://gohugo.io/content-management/front-matter/)_, que contiene ciertos metadatos que ayudan a Hugo a tomar cada archivo y darle forma de entrada de un blog.

## Uso

Para ver el blog en el entorno de desarrollo hay que ejecutar:

```bash
$ hugo server -D
```

Abrir en un navegador la URL: [http://localhost:1313/](http://localhost:1313/).

Todo cambio que se haga tanto en los contenidos como en el aspecto visual se verá reflejado en el navegador.
