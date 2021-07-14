
<!-- .slide: class="titulo" -->

# Tema 2
## Diseño y desarrollo de APIs REST
### (sesión 2)


---


## Contenidos de hoy

4. **Colecciones: filtrados, búsquedas y paginación**
5. Buenas prácticas en el diseño de APIs REST
6. Buenas prácticas a nivel técnico
7. Hipermedia

---

## Filtrar colecciones

- Es poco habitual obtener **todos** los elementos de un tipo, es más común
  + Filtrar resultados especificando condiciones
  + Paginar los resultados, limitando así su número
- Ambos casos se suelen representar con una *query string* (grupo de parámetros HTTP precedido de `?`)

```http
http://mislibros.com/libros?titulo_contiene=tronos&autor_contiene=Martin
https://graph.facebook.com/JustinBieber/photos?offset=5&limit=2
https://api.github.com/users/octocat/repos?page=2&per_page=5
```


---

En algunos APIs como el de Github, la búsqueda se hace mediante URLs especiales

```bash
# buscar todos los repositorios con la palabra clave "node"
https://api.github.com/search/repositories?q=node
```

Esto no es nada REST (¡Un verbo en una URL!), pero qué le vamos a hacer, se ve por ahí

---

## Paginación basada en offset

- Suponiendo que los resultados están ordenados, especificamos la *cantidad* deseada y el *offset* (a partir de qué dato queremos)
- Los parámetros son propios de cada API (`page`&`per_page`, `page`&`count`, `offset`&`limit`...) normalmente el significado es fácil de deducir por el nombre

```http
https://gateway.marvel.com:443/v1/public/characters?limit=10&offset=0
```

- La respuesta suele contener **metadatos** con página actual, total, etc.

```javascript
{   "offset": 0,
    "limit": 10,
    "total": 1491,
    "count": 10,
    "results": [
      {
        "id": 1011334,
        "name": "3-D Man",
        "description": "",
        "modified": "2014-04-29T14:18:17-0400",
        "thumbnail": {
          "path": "http://i.annihil.us/u/prod/marvel/i/mg/c/e0/535fecbbb9784",
          "extension": "jpg"
        }
        ...
      }
        ...
      ]
}
```

---

## Problemas de la paginación por offset

- En SQL, las consultas con un *offset* grande son ineficientes (la BD tiene que leer y descartar las filas)
- En datos que puedan cambiar en tiempo real, añadir/eliminar datos puede significar que se dupliquen/eliminen resultados


---

## Paginación con cursores

- Inicialmente el cliente pide solo una cantidad de datos. En la respuesta viene un **cursor**: identificador único que marca un dato concreto de la lista. 
- En cada respuesta se envía generalmente un cursor para ir "atrás" y otro para "adelante"

```json
{
  "data": [
     ... aquí estarían los datos en sí de lo que hemos pedido
  ],
  "paging": {
    "cursors": {
      "after": "MTAxNTExOTQ1MjAwNzI5NDE=",
      "before": "NDMyNzQyODI3OTQw"
    },
    "previous": "https://graph.facebook.com/me/albums?limit=25&before=NDMyNzQyODI3OTQw",
    "next": "https://graph.facebook.com/me/albums?limit=25&after=MTAxNTExOTQ1MjAwNzI5NDE="
  }
}
```

Problema: el cliente no puede saltar a una página concreta, solo ir adelante y atrás


---

En cuanto a la **implementación** el cursor puede ser un *timestamp*, un *id* o similar, que permita ordenar e indexar los datos y que nos asegure de que no van a aparecer datos nuevos "por enmedio"

Ejemplo: algunos *endpoint* del graph API de FB usan *timestamp* UNIX con los parámetros `since` o `until`

```http
https://graph.facebook.com/me/feed?limit=25&since=1364849754"
```

Cursores "opacos": internamente pueden ser *timestamp*, *id* o lo que sea (combinación de ambos, p.ej.) pero al cliente no le importa. Además evita que el cliente "haga trampas" usando valores "inventados" para los cursores

```http
https://graph.facebook.com/me/albums?limit=25&after=MTAxNTExOTQ1MjAwNzI5NDE=
```

---

<!-- .slide: class="titulo" -->
# 5. Buenas prácticas en el diseño de APIs REST


---

## Algunas buenas prácticas

Primero: los usuarios del API son los **desarrolladores** que van a implementar *apps* o webs que llamen al API, no los usuarios finales de las *apps* o de las webs

- **Consistencia** en el diseño del API, no "sorprender" al usuario
- Centrarse en los **casos de uso**
- **Formalizar la API** antes de implementarla (diseño por contrato, *design first*)

Así especificadas son aplicables al diseño de cualquier tipo de API, no solo REST

---

## No sorprender al usuario

Seguir en lo posible las **convenciones estándar** (en nuestro caso, REST)

Pregunta: ¿Qué convenciones REST **no** se están respetando en esta llamada al API "REST" de Twitter que borra un tweet dado su *id*?

![](img_2/twitter_API.png)

De la [documentación del API de Twitter](https://dev.twitter.com/rest/reference)
<!-- .element: class="caption" -->


---

El objetivo no es ser puristas REST ([Restafarians](https://www.ben-morris.com/the-restafarian-flame-wars-common-points-of-disagreement-over-rest-api-design/)) solo por esnobismo, es para facilitar la vida a los usuarios del API

![](img_2/restafarian.jpg)

---

## Centrarse en los casos de uso

El API debe **facilitar los casos de uso típicos**, no ser un reflejo del funcionamiento interno del sistema, ni de la base de datos

![](img_2/phil_tweet.png)

---

## Especificación formal de APIs REST

Existen varios lenguajes especialmente creados para **especificar APIs REST** que en general nos permiten:

- Obtener la especificación formal antes de implementar
- Generar automáticamente un servidor *mock* a partir de la especificación
- Validar automáticamente que la implementación cumple con la especificación
- Generar un esqueleto de implementación para servidor o cliente 
- Documentar automáticamente el API si ya está implementado

---

## Algunos lenguajes de especificación

<div class="row clearfix">
    <div class="column third">
      <img src="img_2/apiblueprint-logo-2.png">
      <a href="https://apiblueprint.org/" class="caption">API Blueprint]</a>
    </div>
    <div class="column third">
      <img src="img_2/swagger.png"> 
      <a href="https://swagger.io/" class="caption">Swagger</a>
    </div>
    <div class="column third">
      <img src="img_2/raml.png">   
      <a href="https://raml.org/" class="caption">RAML</a>
      <a href="http://static-anypoint-mulesoft-com.s3.amazonaws.com/API_examples_notebooks/raml-design3.html" class="caption">Ejemplo de API</a>
      </div>
</div>


---

## Ejemplo con API Blueprint

```markdown
FORMAT: 1A

# Lista de la compra

API muy sencillo para manejar una lista de la compra

# Group Items

Los productos que hay en la lista de la compra

## Lista de Items [/items]

### Listar todos los items [GET]
+ Response 200 (application/json)

        [ 
           {"nombre":"Patatas", "cantidad":"1kg"},
           {"nombre":"leche", "cantidad":"1L"}
        ]

### Añadir item [POST]

+ Request (application/json)

        {"nombre":"vino", "cantidad":"1 botella"}
        
+ Response 201 (application/json)

    + Headers

            Location: /items/1
```

Hay [muchas herramientas](https://apiblueprint.org/tools.html) que trabajan con este formato, por ejemplo [https://getsandbox.com/](https://getsandbox.com/)

---


<!-- .slide: class="titulo" -->

# 6. Buenas prácticas a nivel técnico

---

## Versionar el API

- Si las URL se mantienen igual entre versiones del API pero cambiamos su comportamiento, "romperemos" los clientes
- Necesitamos que el cliente pueda **especificar a qué versión del API está llamando**

Solución 1: "incrustar" el número de versión en la URL

```http
https://graph.facebook.com/v2.5/me
https://api.twitter.com/1.1/search/tweets.json
```

Solución 2: (Ortodoxia REST): el cliente envía la cabecera `Accept` indicando qué versión del API quiere. Usada por ejemplo por el API de Github

```http
//con esto indicamos que queremos usar la versión 3 del API y queremos JSON
Accept: application/vnd.github.v3+json
```

---

## Cómo generar los id de los recursos

- **Autonuméricos** de la BD. Solución sencilla, pero puede dar problemas si tenemos más de un servidor de BD para repartir la carga (*shard*)
- **UUIDS**: podemos "asegurar" que son únicos aunque se generen desde distintas instancias de servidor

```javascript
//https://repl.it/LQjU
var uuid = require('uuid/v1')
console.log(uuid())   //176c8ba0-9cb6-11e7-abab-7df31ea5be22
```

---

## "Negociación" de contenido

- Hace unos años, todos los APIs devolvían solo XML, ahora casi todos usan solo JSON

![](https://img.devrant.io/devrant/rant/r_111482_unPBx.jpg)

- Si queremos que nuestro API admita y sirva varios formatos de datos, se considera una buena práctica que cliente y servidor puedan "**negociar**" el formato más adecuado para ambos


---

## Cómo negociar el formato según la ortodoxia REST

- Cliente -> servidor: cabecera `Accept` en la petición
```http
Accept: application/json;q=1.0, application/xml;q=0.5, */*;q=0.0
```

- Servidor -> cliente: 
  - Si no se han podido satisfacer las preferencias, código de estado `406 NOT ACCEPTABLE`
  - Indica el formato que ha enviado con la cabecera `Content-Type`

---

## Cómo se negocia el formato en el "mundo real"

Depende del API, en algunos se hace con **parámetros HTTP**, en otros es **parte de la URL** (generalmente como si fuera una extensión de archivo: `.xml`, `.json`,...)


```http
https://api.twitter.com/1.1/search/tweets.json
https://api.flickr.com/services?format=XML...
```


---

<!-- .slide: class="titulo" -->
<!-- .slide: id="sect_hipermedia" -->

# 7. Hipermedia en APIs REST



---

## Problema: dependencia de las URL

*   Tal y como hemos implementado hasta ahora los servicios REST, para realizar una operación necesitamos **conocer previamente la URL** del recurso
*   Esto presenta el problema de que **no podemos modificar las URL** sin "romper" los clientes actuales

---


## HATEOAS

**H**ypermedia **A**s **T**he **E**ngine **O**f **A**pplication **S**tate

Imitar el funcionamiento de la web, en la que para seguir los pasos en un flujo de trabajo vamos saltando entre enlaces, sin necesidad de conocer previamente las URL

![](img_2/amazon.png)<!-- .element: class="r-stretch" -->

---

## Hipermedia en REST

- En cada respuesta debemos incluir enlaces con las **operaciones** posibles y los **recursos** directamente relacionados
- Aunque en JSON no hay un tipo de datos URL, hay formatos más o menos difundidos para representar vínculos, por ejemplo [HAL](http://stateless.co/hal_specification.html)

```javascript
{
  "id":"1",
  "items": [
    {"id":"12", "cantidad":"1"},
    {"id":"1123", "cantidad":"2"}
  ]
  "_links": {
    "self": {
       "href": "http://miapi.com/pedidos/1"  
    },  
    "items": {
      "href": "http://miapi.com/pedidos/1/items"  
    },
    "pagar": {
      "href": "http://miapi.com/pagos/pedidos/1"  
    }
  }
}
```

---

La idea de HATEOAS es que al final el API es como una máquina de estados y pasamos de un estado a otro haciendo peticiones

![](https://eamodeorubio.files.wordpress.com/2012/09/order_state_machine.png) <!-- .element: class="r-stretch" -->

---

## Paginación

Es un ejemplo apropiado para hipermedia, ya que es recomendable incluir **vínculos** a los datos de (al menos) la página anterior y siguiente

```javascript
{
    "_links": {
        "self": {
            "href": "http://example.org/api/books?page=3"
        },
        "first": {
            "href": "http://example.org/api/books"
        },
        "prev": {
            "href": "http://example.org/api/books?page=2"
        },
        "next": {
            "href": "http://example.org/api/books?page=4"
        },
        "last": {
            "href": "http://example.org/api/books?page=133"
        }
    }
    "count": 2,
    "total": 498,
    "data": {
        { title: "Canción de hielo y fuego", author: "George R.R. Martin"}
        { title: "A vuestros cuerpos dispersos", author: "Philip J. Farmer"}
    }
}
```

---

## Referencias

- API design patterns, JJ Geewax


---

<!-- .slide: class="titulo" -->

# ¿Alguna duda?






