# Práctica evaluable 3: Frameworks Javascript

El objetivo de esta práctica es desarrollar una web que sirva de cliente al API de la práctica 1, pero a diferencia de la práctica anterior, **en la implementación se debe usar un *framework* Javascript. Nosotros en clase hemos visto brevemente Vue pero si lo preferís podéis usar otros como React, Angular, Svelte o algún otro similar**.

## Organización del código

 - Usad una estructura de proyecto similar a la práctica anterior, con una carpeta `client` y una `server`
 - Necesitaréis modificar el servidor **para que use CORS**, como hicimos en el servidor de la lista de la compra
 - En el proyecto cliente tenéis 3 opciones:
   
     1. No usar ninguna herramienta adicional en el cliente, como hicimos la semana pasada en la introducción a Vue.
     2. Usar *parcel*, como hicimos en la práctica de la lista de la compra de hace dos semanas. En la documentación se explica [cómo usarlo en conjunto con Vue](https://parceljs.org/languages/vue/), y también con otros frameworks (tendríais 0.25 puntos adicionales por hacerlo así)
     3. Usar [`vue-cli`](https://cli.vuejs.org/) y *single file components* (tendríais 0.5 puntos adicionales por hacerlo así). En otros frameworks hay herramientas equivalentes (por ejemplo en React está `create-react-app`)

## Requisitos mínimos

La implementación correcta de estos requisitos es necesaria para poder aprobar la práctica. Con estos requisitos podéis optar hasta un 6 si además de la implementación comentáis adecuadamente el código fuente, indicando para cada componente: qué hace en líneas generales, qué eventos genera o procesa, qué representa cada variable del estado (`data` en Vue) , cada prop, y cada método (descripción de lo que hace, parámetros, efecto en el estado si lo tiene).

Debéis desarrollar una web en la que como mínimo se pueda hacer:

- Login/logout
- Listado de items: pueden ser todos o puede haber un cuadro de búsqueda asociado
- Eliminación de items (similar al botón "x" de la lista de la compra)
- Ver detalles: al clicar en un item o en un botón asociado a cada item, se muestra más información sobre el mismo (sin cambiar de página)
- Creación de items

> Pongo "items" como término genérico porque cada uno estáis haciendo una aplicación distinta, en vuestro caso concreto serán "productos" o "usuarios" o "películas" o lo que proceda.

### Cómo implementar el login/logout

Aunque en un API REST "puro" no existe formalmente la idea de "hacer login/hacer *logout*", para el usuario de la web es útil que existan las operaciones de *login* y el *logout*. De lo contrario tendría que estar introduciendo continuamente *login* y *password*. Debes implementar las siguientes funcionalidades:

- En algún sitio debes mostrar un formulario para hacer *login*.  
- Una vez comprobado que *login* y *password* son correctos, y obtenido el *token* se guardará este último en el *local storage*
- Cada vez que se haga una petición al servidor con Javascript para una operación restringida enviaréis el *token* en la cabecera `Authorization`
- La operación de *logout* simplemente borrará el *token* del *local storage*. 
- Una vez hecho el login con éxito se debería mostrar en algún sitio de la página el login del usuario o algún mensaje que le indique que ya está logueado y un botón/enlace para hacer *logout*. Cuando se hace el logout con éxito se debería mostrar el formulario de login.

## Requisitos adicionales

Además de los 6 puntos de los requisitos mínimos, elegir de entre los siguientes:

- hasta *0.5 puntos* si usáis la estructura de proyecto que genera la herramienta `vue-CLI` y *single file components* (podéis mirar su uso más detallado en la documentación):
    + [Instalación](https://cli.vuejs.org/guide/installation.html) de `vue-cli`
    + [Crear un proyecto](https://cli.vuejs.org/guide/creating-a-project.html)
    + [*Single file components*](https://v3.vuejs.org/guide/single-file-component.html)
- hasta *1 punto*: implementar el listado de otro recurso incluyendo también "ver detalles" y "eliminar".
- hasta *1 punto*: edición de datos: cada elemento del listado debería tener un botón o enlace "editar" para cambiar su contenido. Al pulsarlo aparecerá un formulario para escribir los nuevos datos. Dónde y cómo aparezca (y desaparezca al acabar la edición) queda a vuestra elección.
- Hasta *0.5 puntos* introducir transiciones/animaciones cuando aparezcan/desaparezcan elementos de la página. Añadir además el típico efecto de que el botón/formulario "tiembla" cuando introducimos un login/password incorrectos. Vue incorpora ya integrada la [funcionalidad](https://v3.vuejs.org/guide/transitions-overview.html#class-based-animations-transitions) para hacer estas transiciones
- hasta *1 punto* si usáis algún *framework* de componentes visuales. Para Vue tenéis por ejemplo [vuetify](https://vuetifyjs.com/en/) o [bootstrap-vue](https://bootstrap-vue.js.org)
- hasta *1.5 puntos*: usar una librería para gestionar el estado de vuestra aplicación de manera centralizada. En Vue se usaría [Vuex](https://vuex.vuejs.org), en otros *frameworks* hay equivalentes.
- hasta *1 punto*: usar un *router* para gestionar la navegación por la *app*. En el caso de Vue se usaría [Vue Router](https://router.vuejs.org)

Cualquier otra funcionalidad que queráis añadir consultadla antes conmigo para ver cuánto se podría valorar en el baremo.

## Entrega

El plazo de entrega concluye el lunes **13 de diciembre de 2021** a las 23:59 horas. La entrega se realizará como es habitual a través de Moodle. Entregad una carpeta comprimida con el proyecto de esta práctica y el del servidor (modificado para usar CORS) en dos subcarpetas, ya que ambos son necesarios para ejecutar la aplicación. 

Entregad también un LEEME.txt donde expliquéis brevemente las partes optativas que habéis hecho y cualquier detalle que consideréis necesario.