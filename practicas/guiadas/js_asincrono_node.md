# Código asíncrono en Javascript

En Javascript no hay varios hilos, todo el código se ejecuta en el mismo hilo. Por ello la mayoría de operaciones que pueden tardar cierto tiempo se ejecutan de forma *asíncrona*, para no bloquear el intérprete. Esto incluye cosas como consultas a bases de datos, peticiones a otros servidores, operaciones criptográficas, etc.

## Código asíncrono vs. síncrono

Por ejemplo [knex](http://knexjs.org) es una librería para trabajar con bases de datos relacionales en JS. La mayoría de operaciones son asíncronas, al igual que en todas las demás librerías de este estilo. Veamos cómo se haría por ejemplo un SELECT para comprobar que existe un usuario con un login y un password determinados.

> Para instalar knex en nuestro proyecto, estando en su directorio hacer `npm i knex` y luego instalar el *driver* de la BD a usar (en nuestro caso SQLite, es decir `npm i sqlite3`). Más detalles en las [instrucciones de instalación de knex para node](http://knexjs.org/#Installation-node)

```javascript
//knex puede trabajar con varios motores de BD distintos, hay que configurarlo
const knex = require('knex')({
    //tipo de BD a usar. Podría ser también postgres, oracle, mysql,...
    client: 'sqlite3',
    //En SQLite podemos usar un fichero directamente sin necesidad de servidor
    connection: {
      filename: "./bd.sqlite"
    }
});

var login_busc = "pepe"
var password_busc = "pepe"

knex.select()
    .from('usuarios')
    .where({login:login_busc, password:password_busc})
    .asCallback(function(error, filas){
        //filas es un array de objetos con los datos. Si está vacío es que no existía ese usuario con ese password
        if (filas.length>0)
          console.log('Usuario OK')
        else
          console.log('El usuario y/o la contraseña son incorrectos')
        knex.destroy()
    })

```
`
> NOTA: el `knex.destroy()` del final fuerza a que termine el `script`, en caso contrario no lo haría mientras knex mantenga una conexión con la BD. En un `script` como el anterior esto no es adecuado, pero en un servidor Express por ejemplo no importaría mucho ya que el propio servidor debe estar ejecutándose todo el tiempo.

Si quisiéramos reutilizar el código anterior para chequear una operación de login en una aplicación web parecería bastante lógico encapsularlo en una función y utilizarlo de este modo:

> AVISO IMPORTANTE: el siguiente código NO FUNCIONA

```javascript
//faltaría toda la parte de inicialización de Knex
//...

function chequeaCredenciales(un_login, un_password) {
  knex.select()
    .from('usuarios')
    .where({login:un_login, password:un_password})
    .asCallback(function(error, filas){
        //filas es un array de objetos con los datos. Si está vacío es que no existía ese usuario con ese password
        if (filas.length>0)
          return true
        else
          return false
    })
    console.log("fin de chequeo")
}

//Faltaría la parte de configuración e inicialización de express
//...
app.post('/usuarios/check', function(pet, resp) {
  //supongamos que el login y password a chequear vienen en el body en JSON
  var body = pet.body
  var result = chequeaCredenciales(body.login, body.password)
  if (result)
     resp.status(200)
     resp.send("Usuario OK")
  else
     resp.status(401)
     resp.send("Usuario y/o contraseña erróneos")
})
```
> ¿Por qué crees que este código no funciona?. Pistas sobre los problemas: 1) ¿dónde están los `return` de `chequeaCredenciales`, son de verdad de esta función? 2) ¿Qué crees que se ejecutará antes, el mensaje de "fin de chequeo" o el código que devuelve true/false?.

El problema es que el código asíncrono es totalmente distinto al código secuencial al que estamos acostumbrados. Con él nos tenemos que olvidar de la idea de "llamo a una función y obtengo el valor que habrá devuelto con `return`".

La clave es que hay que realizar el envío de la respuesta inmediatamente tras darnos cuenta de que las credenciales son correctas, algo como:

```javascript
app.post('/usuarios/check', function(pet, resp) {
  var body = pet.body
  knex.select()
    .from('usuarios')
    .where({login:body.login, password:body.password})
    .asCallback(function(error, filas){
      if (filas.length>0) {
        resp.status(200)
        resp.send("Usuario OK")
      }
      else {
        resp.status(401)
        resp.send("Usuario y/o contraseña incorrectos")
      }  
    })
})

app.listen(3000, function(){
  console.log("Servidor inicializado en el puerto 3000...")
})
```

El problema del código anterior es que no es modular, no hemos separado la funcionalidad de `chequearCredenciales` como antes. La clave para poder modularizar de nuevo el código es que el `resp.status()` y el `resp.send()` se deben ejecutar **desde dentro de `chequearCredenciales`**. Una posibilidad sería pasarle el objeto `resp` a `chequearCredenciales`, pero es una solución un poco "sucia" ya que estamos introduciendo dependencias extrañas en la función. Lo que podemos hacer es de nuevo usar la idea de *callback*, pasándole a `chequearCredenciales` una función a ejecutar cuando acabe el chequeo. Puede parecer raro o retorcido pero en realidad así es como trabajan las funciones asíncronas de knex. El código podría quedar como:

```javascript
function chequeaCredenciales(login_busc, password_busc, success_callback, error_callback) {
  knex.select()
    .from('usuarios')
    .where({login:login_busc, password:password_busc})
    .asCallback(function(error, filas){
        if (filas.length>0)
          success_callback()
        else
          error_callback()     
    })
}

app.post('/usuarios/check', function(pet, resp) {
  var body = pet.body
  chequeaCredenciales(body.login, body.password, 
    function(){
      resp.status(200)
      resp.send("usuario OK")
    },
    function() {
      resp.status(401)
      resp.send("Usuario y/o contraseña incorrectos")
    })
})
```


## Mecanismos para la ejecución de código asíncrono en JS

Vamos a ver los tres mecanismos más comunes que nos permiten ejecutar código asíncrono en Javascript: los *callbacks*, las *promesas* y el *async/await*.

### Callbacks

Ya hemos visto bastantes ejemplos de *callbacks* en node/express, son un concepto muy común no solo en Javascript sino también en muchos otros lenguajes. Básicamente son funciones a las que no llamamos nosotros, sino que las llama típicamente el sistema en respuesta a ciertos eventos o a la finalización de ciertas tareas. 

Por ejemplo ya hemos visto que en una *app* de Express el segundo parámetro del método `get` (idem con `post`, `put`,...) es un *callback*. Express lo ejecuta cuando se recibe la correspondiente petición HTTP. 

```javascript
app.get('/saludo', function(pet, resp){
  response.send("¡Hola!")
})
```

#### ¿Por qué los *callbacks* son incómodos para programar?

Con una única llamada asíncrona no hay problema, pero **encadenar** varias llamadas asíncronas se hace muy tedioso. Acabamos teniendo un *callback* dentro de un *callback* dentro de un *callback*... (alias [*callback hell*](http://callbackhell.com/)).

Por ejemplo, supongamos que para registrar un nuevo usuario en nuestra aplicación tenemos que insertar al usuario en la base de datos y mandarle un email de confirmación. Ambas van a ser operaciones asíncronas. En el siguiente ejemplo (simplificado) hacemos uso de knex para insertar en la BD y de un servicio de terceros llamado [SendGrid](http://www.sendgrid.com) para enviar el *email*. 

```javascript
app.post('/api/v1/usuarios', function(pet, resp){
   nuevoUsuario = pet.body
   //en un caso real por supuesto habría que validar los datos
   knex('usuarios').insert(nuevoUsuario).asCallback(function(result){
        //Le enviamos al usuario un email de confirmación usando un servicio externo
        //Falta rellenar el cuerpo del mensaje con los datos
        //mensaje=...
        request.post({
            url:'https://api.sendgrid.com/v3/mail/send',
            json: true,
            body: mensaje
        }, function(error, result, body){
            //Para simplificar supongamos que el mail siempre se va a enviar OK
            //Usuario ya insertado y mensaje ya enviado, todo OK
            resp.status(201)
            resp.end()
        })            
    })
})
```

Como vemos, el *callback* del envío del *email*, que es cuando se puede dar por finalizada la operación, está anidado a un "tercer nivel", ya que por cada operación asíncrona hay un callback dentro de otro. Se ve que cuantas más operaciones asíncronas encadenemos, más difícil va a ser entender el código.

### Promesas

Las promesas son otro mecanismo para la ejecución de código asíncrono que pretende evitar el *callback hell*.

Una **promesa** (`Promise`) es un objeto que representa el resultado de una operación asíncrona. Puede estar en estado pendiente (*pending*), resuelta con éxito (*resolved*) o haber fallado (*rejected*).

Por ejemplo en el navegador para hacer peticiones HTTP se usa el API `fetch` que hace una petición y devuelve una promesa que eventualmente se resolverá con la respuesta o fallará con un error. 

```javascript
var promesa = fetch('https://api.github.com/users/octocat')
```

En node este API no es estándar pero podemos instalar algunos paquetes de terceros que lo implementan, como `node-fetch`.

La clase `Promise` tiene dos métodos fundamentales: `then()` y `catch()`:

  -  A `then` le podemos pasar dos funciones, que se ejecutarán cuando la promesa pase a *resolved* y a *rejected* respectivamente. La segunda es opcional, de hecho habitualmente solo se le pasa la primera.
  -  A `catch` le podemos pasar una función que se ejecutará cuando la promesa pase a *rejected*. En realidad es "azúcar sintáctico" para `then(null,...)`

```javascript
fetch('https://api.github.com/users/octocat')
   .then(function(){

   })
   .catch(function(){

   })
```

En un manejador de un `then` podemos: 

1. Devolver otra promesa
2. Devolver un valor cualquiera que no sea una promesa (o ninguno)
3. Lanzar un error con `throw`


**1. Devolver otra promesa**

Es lo más común. Nos permite ir *encadenando* promesas, es decir, **ir encadenando operaciones asíncronas, sin tener que anidar los *callbacks***, con lo que el código queda mucho más "limpio".

```javascript
//https://repl.it/@ottocol/node-fetch
const fetch = require('node-fetch')
fetch('https://api.github.com/users/octocat')
  .then(function(respuesta){
       //El método json() convierte la respuesta del servidor a JSON
       //pero es asíncrono, no síncrono!!! (y devuelve una promesa)
       return respuesta.json()
  }).then(function(objeto){
       alert("El nombre real del usuario es: " + objeto.name)
  }).catch(function(error){
      console.log(error)
  })    
```

> Aclaración: **los parámetros** de los *callbacks*  pasados a los `then`  **los están devolviendo `fetch()` y `json()`**. Es decir, depende del API que estemos usando qué parámetros recibirán nuestros *callbacks*

Volvemos al ejemplo de antes de los *callbacks* anidados pero ahora con promesas, que también soporta knex (de hecho es la forma recomendada de usar su API).

```javascript
app.post('/api/v1/usuarios', function(pet, resp){
   nuevoUsuario = pet.body
   //en un caso real por supuesto habría que validar los datos
   knex('usuarios').insert(nuevoUsuario)
     .then(function(result){
        //Le enviamos al usuario un email de confirmación usando un servicio externo
        //Para que funcionara de verdad falta rellenar el cuerpo del mensaje con los datos
        ///https://sendgrid.com/docs/API_Reference/Web_API_v3/Mail/index.html
        return fetch('https://api.sendgrid.com/v3/mail/send', {
            method:'post',
            body: JSON.stringify(mensaje),
            headers: { 'Content-Type': 'application/json' },
        
        })            
    }).then(function(resp_sendgrid) {
        if (resp_sendgrid.ok) { // res.status >= 200 && res.status < 300
          return resp_sendgrid;
        } else {
          throw Error(res.statusText);
        }
    }).then(function(resp_sendgrid){
        resp.status(200)
        resp.send("Usuario creado correctamente")
    }).catch(function(error){
        console.log(error)
    })
})
```

**2. Devolver un valor que no sea una promesa (o ninguno)**

Es una forma de "convertir" un valor síncrono en asíncrono, ya que el `then` crea una nueva promesa resuelta cuyo valor es el valor devuelto.

```javascript
//https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html
getUserByName('nolan').then(function (user) {
  if (inMemoryCache[user.id]) {
    return inMemoryCache[user.id];    // returning a synchronous value!
  }
  return getUserAccountById(user.id); // returning a promise!
}).then(function (userAccount) {
  // I got a user account!
});
```

**3. Lanzar un error con `throw`**

Se irá al `catch` del final de la cadena, similar a la construcción tradicional de programación secuencial

```javascript
fetch('https://api.github.com/users/octocat')
  .then(function(respuesta){
       if (respuesta.ok)
          return respuesta.json()
       else throw new Error("Problema en la respuesta")
  }).then(function(objeto){
       alert("El nombre real es: " + objeto.name)
  }).catch(function(error){
       console.log(error.message)  //"Problema en la respuesta"
  })
```

### Async/Await

Nos permite escribir código asíncrono como si fuera secuencial.

Si ponemos `await` delante de una función que devuelva una promesa nos esperaremos a que se resuelva o rechace. Cuidado, `await` solo puede usarse dentro de funciones marcadas como `async`.
 
```javascript
const fetch = require('node-fetch')

app.post('/api/v1/usuarios', async function(pet, resp){
   nuevoUsuario = pet.body
   //en un caso real por supuesto antes habría que validar los datos
   try {
     var result_knex = await knex('usuarios').insert(nuevoUsuario)
     var resp_sendgrid = await fetch('https://api.sendgrid.com/v3/mail/send', {
              method:'post',
              body: JSON.stringify(mensaje),
              headers: { 'Content-Type': 'application/json' },
          })    
     if (resp_sendgrid.ok) {
        resp.status(200)
        resp.send("Usuario creado OK")
     }
     else {
        resp.status(500)
        resp.send("Usuario OK, pero sin mensaje de confirmación")
     } 
   }  
   catch(e) {
      resp.status(500)
      resp.send("Error al insertar usuario en la BD")
   }
})         
```
