# Blog

Este repositorio contiene el código fuente y el contenido de mi blog personal construido mediante [Hugo](https://gohugo.io) y [Bulma](https://bulma.io).

## Entornos

### Desarrollo

Para montar este blog en un entorno de desarrollo es necesario:

* Instalar [Hugo](https://gohugo.io/getting-started/installing/) en su **versión _extended_**.
* Clonar este mismo repositorio.

Si no quieres instalar Hugo, puedes usar Visual Studio Code. En el repositorio verás la carpeta ``.devcontainer`` que contiene la configuración necesaria para ejecutar Hugo desde un contenedor.

### Producción

Este blog se publica en [Netlify](https://netlify.com/) de forma automática cada vez que se hace un _push_ a _main_ a modo de [Continuous Deployment](https://www.atlassian.com/continuous-delivery/continuous-deployment). La URL de este blog es [https://anedonia.website/](https://anedonia.website/).

[![Netlify Status](https://api.netlify.com/api/v1/badges/f1c7c22a-a9d6-4fb8-8972-c85810559521/deploy-status)](https://app.netlify.com/sites/anedonia/deploys)

## Contribuir

Cualquier aportación, a modo de _pull request_ será bienvenida. Este repositorio utiliza [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/).

Una vez se tiene el [entorno de desarrollo](#desarrollo) se puede contribuir de dos formas, mejorando el aspecto visual o mejorando/aportando contenidos.

### Aspecto visual

Para hacer cambios en el aspecto visual el directorio que te interesa es `themes/anedonia` y la documentación de Hugo al respecto de los _[Templates](https://gohugo.io/templates/)_.

### Contenidos

Para mejorar o aportar contenidos el directorio que te interesa es `content` y la documentación de Hugo al respecto del _[Content Management](https://gohugo.io/content-management/)_.

A modo de rápido resumen, en el directorio `content` hay tantos archivos _Markdown_ como entradas en el blog. A parte del contenido de la entrada, cada archivo tiene una cabecera, conocida como _[Front Matter](https://gohugo.io/content-management/front-matter/)_, que contiene ciertos metadatos que ayudan a Hugo a tomar cada archivo y darle forma de entrada de un blog.

## Uso

Para ver el blog en el entorno de desarrollo hay que ejecutar:

```bash
~ hugo server -D
```

Abrir en un navegador la URL: [http://localhost:1313/](http://localhost:1313/).

Todo cambio que se haga tanto en los contenidos como en el aspecto visual se verá reflejado en el navegador.

Si se quiere acceder desde otra máquina, hay que ejecutar:

```bash
hugo server -D --bind=0.0.0.0 --baseURL=<IP> 
# Donde IP puede ser, por ejemplo, http://192.168.1.20
```

Desde otra máquina, con acceso a la red 192.168.1.x, abrir en un navegador la URL: [http://192.168.1.20:1313](http://192.168.1.20:1313).