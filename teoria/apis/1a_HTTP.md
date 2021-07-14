<!-- .slide: class="titulo" -->

# Tema 1
## Introducción a los APIs web
# Parte I
## HTTP y aplicaciones web

---

## Indice

1.  HTTP para sitios web estáticos
2.  HTTP para aplicaciones web
3.  Aplicaciones web "modernas"


---

<!-- .slide: class="titulo" -->

# 1. HTTP para sitios web estáticos


---

## Petición/respuesta HTTP

Un servidor web está a la escucha por un **puerto**, aceptando **peticiones** de recursos (p.ej. archivos HTML) y devolviéndolos como **respuestas**

<!-- .element class="stretch" -->
![](img_1a/http.png)

---

## Observar el tráfico HTTP

* En HTTP1 el intercambio de información es en **modo texto**. En HTTP2 es **binario**
* Herramientas de desarrollador del navegador 

![](img_1a/firefox_devtools.png)

Firefox developer tools: Menú "Tools > Web Developer > Network"
<!-- .element class="caption" -->

---

## Petición HTTP

-  Típicamente una petición contiene el **método** (tipo), la **URL** solicitada y **cabeceras** con información adicional. 
- En HTTP2 el método y la URL se tratan también como cabeceras
  
![](img_1a/peticion_anotada.png) 

---

## Métodos de petición

*   **GET** solicitar un recurso 
*   **POST** enviar datos al servidor
*   **PUT** actualizar recurso, **DELETE**: eliminar recurso
    * No permitidos en la mayoría de recursos por motivos evidentes
    * Los navegadores no los usan en la navegación "normal", se necesita Javascript para lanzar estos métodos

---

## Respuesta HTTP


Una respuesta contiene un **código de estado**, unas **cabeceras** con información adicional y normalmente los **datos** solicitados por el cliente

```http
HTTP/1.1 200 OK
Date: Wed, 06 Sep 2017 10:38:21 GMT
Server: Apache
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Length: 27003
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html

<!DOCTYPE html>
<html lang="es">
<head>
  <title> Universidad de Alicante</title>
    <meta charset="utf-8">
    ...
```

<div class="caption">Respuesta del servidor web de la UA a una petición GET a https://www.ua.es</div>


---

## Códigos de estado

Diferentes rangos numéricos indican distintos tipos de resultados
*   1xx _informational_
*   2xx _success_ (p.ej. `200 OK`)
*   3xx _redirection_ (p. ej. `301 MOVED PERMANENTLY`)
*   4xx _client error_ (p. ej. `404 NOT FOUND`, `400 BAD REQUEST`, `403 FORBIDDEN`, [418 I’M A TEAPOT](http://tools.ietf.org/html/rfc2324) :) )
*   5xx server error

Consultar más en [http://httpstatus.es](http://httpstatus.es)

En la actualidad la mayoría son ignorados por el navegador, que se limita a   mostrar el cuerpo de la respuesta.

---

<!-- .slide: class="titulo" -->

# 1.2 HTTP básico para aplicaciones web

---

¿Podemos usar un protocolo diseñado inicialmente para navegar por repositorios remotos de documentos para, en vez de eso, **ejecutar aplicaciones remotas**?

![](img_1a/sujetame.jpg)<!-- .element: class="fragment" data-fragment-index="1"-->

---

## Aplicaciones web y HTTP

* Una aplicación web es una **colección de "programitas"** o "rutinas". A cada uno se accede a través de una URL
* La comunicación con las rutinas se hace a través de **HTTP** 
    *   Una petición GET ya no significa "devuelve un recurso", sino **"ejecuta un programa y devuelve el resultado"**
    *   El código de estado se puede interpretar como el **resultado de la ejecución**. p. ej, un 500 se debe a que el programa ha abortado
* Al igual que en línea de comandos podemos pasar **parámetros** 

<span class="fragment" data-fragment-index="1">http://www.miapp.com/verNota</span><span class="fragment" data-fragment-index="2">?dni=222333444</span>

---

## Hipotética *app* web para consultar notas

![](img_1a/app_web.png)

---

## GET vs. POST

*   En aplicaciones web "clásicas" **tienen la misma semántica, ejecutar un programa remoto pasándole datos** en forma `parametro=valor&parametro2=valor2...`
*   Algunas diferencias "prácticas"
    *   Los parámetros en POST tienen longitud ilimitada
    *   Los parámetros en GET se ven en la barra de direcciones del navegador

---

## Por cierto, ¿De dónde salen los parámetros? 🤔

* De los formularios

```html
<form action="login.php" method="post">
  Usuario: <input type="text" name="login">
  Contraseña: <input type="password" name="password">
  <input type="submit" value="Entrar">
</form>
```

* "Embebidos" en los enlaces

```html
<a href="verUsuario?dni=11222333">Luis Ricardo Borriquero</a>
```

---

## Plantillas HTML en el servidor

*   Facilitan la tarea de generar HTML dinámicamente, ya que generar todo el HTML a base de algo estilo `printf` sería engorroso
*   Mezclan bloques de **HTML** "estático" con **sentencias** de algún lenguaje de programación

---

## Ejemplo: PHP

![](img_1a/php.png)

---

## Javascript

![](img_1a/js.png)
*   El código se descarga junto con el HTML y se interpreta en el navegador **después de** la petición/respuesta HTTP
   -  Recordemos que el código en el servidor se ejecuta **antes de** enviar la respuesta  
*   Inicialmente se usaba para pequeños cálculos, validación de formularios, [efectos tontos](http://www.javascript-fx.com/mouse_trail/pinwheel/demo.html)

---

## Frontend vs. Backend

Con Javascript aparece el mundo del **desarrollo frontend**

["Picking a Technology Stack"](https://docs.google.com/presentation/d/1pA6reUNKqkfupSogZB4Q42Tk98VAq_loqkIDE-HSxAE/present#slide=id.p), Pamela Fox
<!-- .element: class="caption" --> 

![](img_1a/front_vs_back.png)


---

## Arquitectura "clásica" para apps web

Charla: [The New Application Architectures](http://www.infoq.com/presentations/SpringOne-2GX-2012-Keynote-2), Adrian Colyer
<!-- .element: class="caption" --> 

![](img_1a/app_clasica.png)

---

## Resumiendo: apps web "clásicas"

- Casi todo el trabajo lo hace el servidor
- Las páginas son en su mayoría estáticas, con pequeños fragmentos dinámicos
- Cada "pantalla" de la app es una página  distinta (un `.html`)
- Casi todo el estado se mantiene en el servidor


---

<!-- .slide: class="titulo" -->

## 3. Aplicaciones web "modernas"

---

## AJAX

Varias tecnologías (un API JS + formato JSON/XML) que permiten **hacer peticiones al servidor sin cambiar de URL, y refrescar solo parte de la página**

Omnipresente en la actualidad. Salvo las webs estáticas prácticamente **todo es AJAX**

---



![](img_1a/AJAX_vs_NO.png)
<br>[Geek & Poke, sin AJAX vs. con AJAX](http://geekandpoke.typepad.com/geekandpoke/2012/01/simply-explained.html)
<!-- .element: class="caption" --> 


---

## Arquitectura de app web "moderna"

Gracias a AJAX y a Javascript nos podemos llevar **casi todo el código de la aplicación al navegador**, convirtiendo el servidor simplemente en un **API remoto** para guardar/recuperar datos

Charla: [The New Application Architectures](http://www.infoq.com/presentations/SpringOne-2GX-2012-Keynote-2), Adrian Colyer
<!-- .element: class="caption" -->


![](img_1a/modern_webapp.png) 


---

## El estado se traslada al cliente

Datos como el carro de la compra, el listado que estamos viendo en pantalla y podemos modificar, el usuario actual, etc, se guardan en el **cliente**

Técnicamente posible gracias a una serie de APIs Javascript del navegador
  - Local Storage: almacenar pares clave/valor
  - IndexedDB: almacenar bases de datos en el cliente

---

## Single Page Applications

* AJAX "llevado al extremo"
* La aplicación es **un único .HTML** y los cambios en la interfaz se hacen cambiando dinámicamente fragmentos de HTML gracias al JS, no navegando a otras páginas


---

## Ventajas de las SPA (I)

Una SPA proporciona una **experiencia de usuario** mucho más cercana a una app nativa que la web "tradicional"

Ejemplo: la versión "estándar" de GMail vs. la [vista "básica" HTML](https://support.google.com/mail/answer/15049?hl=es)

---

## Ventajas de las SPA (II)

Al extraer la interfaz y gran parte de la lógica al *frontend* **el *backend* es el mismo** para todos los clientes (web, app móvil, escritorio, integración con otros sistemas...)

![](img_1a/linkedin_services.png)


---

## Problemas de las SPA (I)

La **carga inicial** del sitio es mucho más lenta. Gran problema en móviles

![](img_1a/spa_speed.png)

---

## "Moderno" no siempre quiere decir mejor

En 2010 Twitter [cambió su arquitectura *frontend*](https://blog.twitter.com/engineering/en_us/a/2010/the-tech-behind-the-new-twittercom.html), desplazando la creación de interfaz y parte de lógica al cliente, con Javascript. PERO...

>[...] we discovered that the raw parsing and execution of JavaScript caused massive outliers in perceived rendering speed. **In our fully client-side architecture, you don’t see anything until our JavaScript is downloaded and executed**. The problem is further exacerbated if you do not have a high-specification machine or if you’re running an older browser. The bottom line is that **a client-side architecture leads to slower performance** [...]

["Improving performance on Twitter.com"](https://blog.twitter.com/2012/improving-performance-on-twittercom), del [Blog de *engineering* de Twitter](https://blog.twitter.com/engineering/en_us.html)
<!-- .element: class="caption" -->


---

## Problemas de las SPA (II): **SEO**

Aunque Google indexa y renderiza los sitios dinámicos [desde 2015](https://webmasters.googleblog.com/2015/10/deprecating-our-ajax-crawling-scheme.html) como lo haría un navegador, ejecutando el JS, puede haber problemas. Por ejemplo no sabe cuándo se ha cargado todo el contenido.

---

## Posible solución: *rendering* "híbrido"

En el **primer acceso** el servidor envía **HTML** "estático", como en una app "clásica". Una vez cargado el contenido, empieza a actuar como una SPA

Los *frameworks* de desarrollo en el cliente ya suelen tener extensiones que facilitan implementar esto, p.ej. [Nuxt.js](https://nuxtjs.org/) para Vue

![](img_1a/react_ssr.png) <!-- .element: class="r-stretch" -->

---

## Resumiendo: aplicaciones web "modernas"

- El **trabajo** se reparte mucho más entre cliente y servidor
- El cliente almacena también *estado*
- El servidor se convierte en una especie de **"APÏ remoto"** para almacenar/recuperar datos
- En el extremo, todas las "pantallas" de la app están en un único `.html` y cambian dinámicamente con JS


---

<!-- .slide: class="titulo" -->

## ¿Preguntas?



