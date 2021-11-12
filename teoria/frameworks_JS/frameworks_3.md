<!-- .slide: class="titulo" -->

# Tema 5: *Frameworks* JS en el cliente
## Parte III: Reactividad y _rendering_

---

<!-- .slide: class="titulo" -->

# Reactividad

---

Aunque **reactividad** es un término bastante amplio que tiene distintos significados en distintos contextos, en el contexto de los *frameworks* JS se suele entender como:

- **cambiar automáticamente el valor de una variable** cuando esta depende de otra
- **repintar la vista automáticamente cuando cambia el estado** del componente 

---

En su forma más básica, la reactividad no es nada nuevo

![](imag_3/hoja_calculo.png) <!-- .element class="stretch" -->


---

En *frameworks* JS se suelen distinguir dos tipos de reactividad:

- **Tipo *pull***: hay que llamar a un método para actualizar el estado, que a su vez "repinta" la vista. Por ejemplo: React, Svelte 2, Angular (aunque este último llama al método automáticamente por ejemplo en manejadores de evento)
- **Tipo *push***: al cambiar los datos se repinta la vista automáticamente. Ejemplos: Vue, Svelte 3, Knockout

---


## Ejemplos de reactividad tipo "pull"

- [Ejemplo de código "de juguete"](https://jsbin.com/cozigix/2/edit?html,js,console,output) 
  - Para probarlo escribir en la consola `setState({contador:0}`. A partir de ahí se puede usar el botón (que llama a `setState`) 
- [Ejemplo con el framework React](https://codepen.io/darylw/pen/vzKQNp?editors=0010) (podéis ver que la idea es la misma)


---

## Reactividad tipo "pull"

Implementación _naive_ y muy simplificada

```javascript
var update, state

//función a la que hay que pasar qué hacer cuando cambie el estado
var onStateChanged = function(update_fn)  {
  update = update_fn
}

//decimos: "cuando cambie el estado, queremos repintar la vista"
onStateChanged(function () {
  var view = render(state)
})

//función que sirve para cambiar el estado
var setState = function(newState) {
  state = newState
  update()
}


```

---

## Reactividad push

Como ya sabíais, Javascript no es reactivo:

```javascript
var a1 = 2
var a2 = 3
var total = a1 + a2
console.log(total)  //5
a1 = 10
console.log(total)  //sigue siendo 5 😕
```

sin embargo ya sabéis que Vue sí que lo es: [Ejemplo en JSBin](https://jsbin.com/nakomet/edit?html,js,output). Incluso podemos eliminar la parte de UI, y lo sigue siendo: [Ejemplo en JSBin](https://jsbin.com/nayojel/edit?html,js,output)


---

## Cómo funciona la reactividad en Vue 3

Cuando una operación es **reactiva**, queremos no solo ejecutarla, sino también guardarla para cuando cambien las dependencias ejecutarla de nuevo

```javascript
var a1 = 2
var a2 = 3
var total = 0
//"empaquetamos" la operación en una función para poder guardarla
const effect = function() {
  total = a1 + a2
}
//lo guardamos, suponemos que esto lo hace la función "track"
track(effect)
//ejecutamos la operación
effect()
...
//cuando cambien las dependencias, volvemos a ejecutar todos los "effects"
trigger()
```

(Los nombres `track`, `effect` y `trigger` son los que [usa Vue](https://v3.vuejs.org/guide/reactivity.html) internamente para estos conceptos)


---

## Ejemplo "de juguete"


```javascript
var a1 = 2
var a2 = 3
var total = 0
var effect = function() { return total = a1 + a2}
//guardaremos los effects en un Set de JS
var dep = new Set()
function track(un_effect) { dep.add(un_effect)}
function trigger() { dep.forEach(effect=>effect())}

//guardamos el effect
track(effect)
//lo ejecutamos
effect()

```

[Código en JSBin](https://jsbin.com/pirirup/edit?js,console)

de momento tanto el guardado de los effect como el trigger hay que ejecutarlos manualmente...en un poco veremos cómo resolverlo


---

## Escalando el ejemplo "de juguete"


En un caso real no tendremos variables simples, sino múltiples objetos reactivos, cada uno con múltiples propiedades reactivas para las que querremos almacenar los efectos

![](imag_3/deps.png) <!-- .element class="stretch" -->

[Del video "Reactivity in Vue 3: How does it work?"](https://www.youtube.com/watch?v=NZfNS4sJ8CI) <!-- .element class="caption" -->


---

## Falta automatizar la reactividad

```javascript
var datos = {a1:2, a2:3}
var total = 0
var effect = function() { return total = datos.a1 + datos.a2}
```

- Si al ejecutar un effect accedemos a una propiedad (*getter*) tendremos que hacer `track()` sobre ella
- Si una propiedad se cambia (*setter*) tendremos que ejecutar los efectos asociados (`trigger`)


Nos falta saber cómo **interceptar** los *getters* y *setters* de un objeto reactivo para además de ejecutar la operación original (get/set) ejecutar el `track/trigger`

---

## Proxies en JS


A partir de la versión ES6 me permiten "envolver" un objeto en otro e interceptar las llamadas al original


```javascript
var datos = {a1:2, a2:3}

var datosProxy = new Proxy(datos, {
  get (target,key) {
    //aquí hago esto, pero en Vue estaría llamando a "track"
    console.log('Estoy llamando al get')
    return target[key]
  }
})
```

- Igualmente se pueden interceptar los *setters*
- Más detalles: Reactivity in Vue 3: How does it work? [video1](https://www.youtube.com/watch?v=NZfNS4sJ8CI) [video2](https://www.vuemastery.com/courses/vue-3-reactivity/proxy-and-reflect/) (para ver el vídeo 2 tenéis que registraros en el sitio web del curso de Vue) 

---

## Otro ejemplo: Svelte 3

Svelte sigue un enfoque distinto, en lugar de hacer "la magia" en *runtime*, es un **compilador** que genera "código reactivo"

[Ejemplo online](https://svelte.dev/repl/a17ccc68af7d44948dee4b68256766dc?version=3.44.1)


---

<!-- .slide: class="titulo" -->

# *Rendering*

---


El *framework* debe permitirnos especificar cómo será la vista en función de los datos

```
vista = f(datos)
```

Aunque podríamos hacer muchas más diferencias hay *frameworks* que usan un lenguaje de **_templates_** (p.ej Angular, Svelte, Vue _en apariencia_) y otros usan directamente **Javascript** (p.ej React)

---

## Lenguajes de templates

- Problema: nos vemos limitados a la expresividad del lenguaje
- Ventaja: Los *framework* suelen incluir un *compilador* de *templates* que puede optimizar la detección de cambios para el repintado

```html
<div id="content">
  <p class="text">Lorem impsum</p>
  <p class="text">Lorem impsum</p>
  <p class="text">{{mensaje}}</p>
  <p class="text">Lorem impsum</p>
  <p class="text">Lorem impsum</p>
</div>
```

En el ejemplo, solo puede cambiar el tercer `<p>`, así que **para repintar el componente *como mucho* solo hace falta repintar ese `<p>`**


---

## Javascript para definir *templates*

- En React: `React.createElement(<tag>, <atributos>, <hijos>)`
- Luego veremos por qué no se usan directamente las funciones del DOM del mismo nombre. Por ahora podemos suponer que hacen lo mismo

```javascript
//esto es como la "f" de vista=f(datos) 
function HelloReact(props) {
  return React.createElement("div", null, 
            React.createElement("h1", null, props.saludo), 
            React.createElement("p", null, "Welcome to React"));
}
//maquinaria necesaria para pintar el componente React en la página
const element = <HelloReact saludo="Ehh!!"/>;
ReactDOM.render(
  element,
  //suponiendo que exista un tag con id="mountNode"
  document.getElementById('mountNode')
);
```
[Probar ejemplo](https://jscomplete.com/playground/s357745)

---

El ejemplo anterior puede parecer tedioso (¡y lo es!), pero usar JS para la función de *render* tiene la **ventaja** de que **podemos usar toda la expresividad de JS**

```javascript
//Aclaración: nadie programa así en React, normalmente se usa un formato llamado JSX,
//que permite poner las etiquetas en el JS y es mucho más legible
function Ejemplo(props) {
    const c = React.createElement
    const children = []
    for(var i=0; i<5; i++) {
        children.push(c('p', {class:'text'},
                       i == 2 ? props.mensaje : 'Lorem ipsum'))
    }
    return c('div', {id:'content'}, children)
}

const element = <Ejemplo mensaje="Hola amigos" />;
ReactDOM.render(
  element,
  document.getElementById('mountNode')
);
```

[Probar ejemplo](https://jscomplete.com/playground/s357763)

---

**Problema**: el lenguaje es tan expresivo que el *framework* no puede analizar qué estamos haciendo y no puede optimizar tanto el proceso de repintado.

Como hemos visto en los ejemplos, el desarrollador lo programa como si se repintara todo el árbol (*como los gráficos de un juego que se repintan enteros n veces por segundo*) 

¿Cómo reducir entonces el coste del repintado?

---

## DOM virtual

- Idea introducida por React
- La función `React.createElement` no genera nodos del DOM real, sino nodos en memoria (en un "árbol DOM virtual"), con un API más rápido
- En cada *render* se hace una especie de *diff* entre el DOM virtual actual y el anterior (denominada en React ["reconciliation"](https://reactjs.org/docs/reconciliation.html). Según la documentación de React el coste es lineal con el número de nodos
- **Solo se repintan en el DOM real los nodos que cambian**. 


---

[Ejemplo de reconciliation](https://codepen.io/ottocol/pen/QWWVWPa?editors=1010)

Para verlo hay que abrir la consola de desarrolladores del navegador, ir a ver el código fuente en "tiempo real" (pestaña "Elements" en Chrome, "Inspector" en Firefox) y buscar el div con id="root". Pese a que en el código de la función de render se repinta el componente entero, en el navegador solo se está cambiando un nodo.

---

## Vue y el Virtual DOM

- Curiosamente, aunque Vue usa plantillas para describir el HTML de los componentes, estas internamente se comportan como funciones JS, de hecho podemos escribir la función render() si las plantillas "se nos quedan pequeñas"
- Es por esto que Vue también usa un DOM virtual
- Por el contrario, frameworks como Svelte no lo necesitan

---


## Referencias

- 📺 [dotJS 2016 - Evan You - Reactivity in Frontend JavaScript Frameworks](https://www.youtube.com/watch?v=r4pNEdIt_l4)
- 📺 [Evan You on Vue.js: Seeking the Balance in Framework Design | JSConf.Asia 2019](https://www.youtube.com/watch?v=ANtSWq-zI0s)
- 📺 📄[Svelte 3: Rethinking reactivity](https://svelte.dev/blog/svelte-3-rethinking-reactivity)