# Sesión HTTP : HTTP Session

- [Introduction](#introduction)
    - [Configuration](#configuration)
    - [Driver Prerequisites](#driver-prerequisites)
- [Using The Session](#using-the-session)
    - [Retrieving Data](#retrieving-data)
    - [Storing Data](#storing-data)
    - [Flash Data](#flash-data)
    - [Deleting Data](#deleting-data)
    - [Regenerating The Session ID](#regenerating-the-session-id)
- [Adding Custom Session Drivers](#adding-custom-session-drivers)
    - [Implementing The Driver](#implementing-the-driver)
    - [Registering The Driver](#registering-the-driver)

<a name="introduction"></a>
## Introducción : Introduction

Dado que las aplicaciones impulsadas por HTTP son sin estado, las sesiones proporcionan una forma de almacenar información sobre el usuario en múltiples solicitudes. Laravel se envía con una variedad de backends de sesión a los que se accede a través de una API expresiva y unificada. El soporte para backends populares como [Memcached](https://memcached.org), [Redis](https://redis.io) y bases de datos se incluye de fábrica.
> > Since HTTP driven applications are stateless, sessions provide a way to store information about the user across multiple requests. Laravel ships with a variety of session backends that are accessed through an expressive, unified API. Support for popular backends such as [Memcached](https://memcached.org), [Redis](https://redis.io), and databases is included out of the box.

<a name="configuration"></a>
### Configuración : Configuration

El archivo de configuración de la sesión se almacena en `config/session.php`. Asegúrese de revisar las opciones disponibles para usted en este archivo. Por defecto, Laravel está configurado para usar el controlador de sesión `file`, que funcionará bien para muchas aplicaciones. En aplicaciones de producción, puede considerar el uso de los controladores `memcached` o `redis` para un rendimiento de sesión incluso más rápido.
> > The session configuration file is stored at `config/session.php`. Be sure to review the options available to you in this file. By default, Laravel is configured to use the `file` session driver, which will work well for many applications. In production applications, you may consider using the `memcached` or `redis` drivers for even faster session performance.

The session `driver` configuration option defines where session data will be stored for each request. Laravel ships with several great drivers out of the box:

<div class="content-list" markdown="1">
- `file` - sessions are stored in `storage/framework/sessions`.
- `cookie` - sessions are stored in secure, encrypted cookies.
- `database` - sessions are stored in a relational database.
- `memcached` / `redis` - sessions are stored in one of these fast, cache based stores.
- `array` - sessions are stored in a PHP array and will not be persisted.
</div>

> {tip} The array driver is used during [testing](/docs/{{version}}/testing) and prevents the data stored in the session from being persisted.

<a name="driver-prerequisites"></a>
### Requisitos previos del Driver : Driver Prerequisites

#### Base de datos : Database

Al utilizar el controlador de sesión `database`, deberá crear una tabla para contener los elementos de la sesión. A continuación se muestra un ejemplo de declaración `Schema` para la tabla:
> > When using the `database` session driver, you will need to create a table to contain the session items. Below is an example `Schema` declaration for the table:

    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->unsignedInteger('user_id')->nullable();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity');
    });

Puede usar el comando Artisan `session:table` para generar esta migración:
> > You may use the `session:table` Artisan command to generate this migration:

    php artisan session:table

    php artisan migrate

#### Redis

Antes de utilizar las sesiones de Redis con Laravel, deberá instalar el paquete `predis/predis` (~1.0) a través de Composer. Puede configurar sus conexiones Redis en el archivo de configuración `database`. En el archivo de configuración `session`, la opción `connection` se puede usar para especificar qué conexión Redis usa la sesión.
> > Before using Redis sessions with Laravel, you will need to install the `predis/predis` package (~1.0) via Composer. You may configure your Redis connections in the `database` configuration file. In the `session` configuration file, the `connection` option may be used to specify which Redis connection is used by the session.

<a name="using-the-session"></a>
## Uso de la sesión : Using The Session

<a name="retrieving-data"></a>
### Recuperando datos : Retrieving Data

Hay dos formas principales de trabajar con datos de sesión en Laravel: el helper global `session` y a través de una instancia `Request`. Primero, veamos el acceso a la sesión a través de una instancia `Request`, que puede ser indicada por tipo en un método de controlador. Recuerde, las dependencias de los métodos del controlador se inyectan automáticamente a través de Laravel [service container](/docs/{{version}}/container):
> > There are two primary ways of working with session data in Laravel: the global `session` helper and via a `Request` instance. First, let's look at accessing the session via a `Request` instance, which can be type-hinted on a controller method. Remember, controller method dependencies are automatically injected via the Laravel [service container](/docs/{{version}}/container):

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function show(Request $request, $id)
        {
            $value = $request->session()->get('key');

            //
        }
    }

Cuando recupera un artículo de la sesión, también puede pasar un valor predeterminado como segundo argumento para el método `get`. Este valor predeterminado se devolverá si la clave especificada no existe en la sesión. Si pasa un `Closure` como valor predeterminado al método` get` y la clave solicitada no existe, el `Closure` se ejecutará y se devolverá su resultado:
> > When you retrieve an item from the session, you may also pass a default value as the second argument to the `get` method. This default value will be returned if the specified key does not exist in the session. If you pass a `Closure` as the default value to the `get` method and the requested key does not exist, the `Closure` will be executed and its result returned:

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function () {
        return 'default';
    });

#### The Global Session Helper

También puede usar la función global PHP `session` para recuperar y almacenar datos en la sesión. Cuando se llama al asistente `session` con un solo argumento de cadena, devolverá el valor de esa clave de sesión. Cuando se llama al ayudante con un array de pares clave / valor, esos valores se almacenarán en la sesión:
> > You may also use the global `session` PHP function to retrieve and store data in the session. When the `session` helper is called with a single, string argument, it will return the value of that session key. When the helper is called with an array of key / value pairs, those values will be stored in the session:

    Route::get('home', function () {
        // Retrieve a piece of data from the session...
        $value = session('key');

        // Specifying a default value...
        $value = session('key', 'default');

        // Store a piece of data in the session...
        session(['key' => 'value']);
    });

> {tip} Hay poca diferencia práctica entre el uso de la sesión a través de una instancia de solicitud HTTP o el uso del helper global `session`. Ambos métodos son [comprobables](/docs/{{version}}/testing) a través del método `assertSessionHas` que está disponible en todos los casos de prueba.
> > > {tip} There is little practical difference between using the session via an HTTP request instance versus using the global `session` helper. Both methods are [testable](/docs/{{version}}/testing) via the `assertSessionHas` method which is available in all of your test cases.

#### Recuperación de todos los datos de la sesión : Retrieving All Session Data

Si desea recuperar todos los datos en la sesión, puede usar el método `all`:
> > If you would like to retrieve all the data in the session, you may use the `all` method:

    $data = $request->session()->all();

#### Determinando si existe un elemento en la sesión : Determining If An Item Exists In The Session

Para determinar si un elemento está presente en la sesión, puede usar el método `has`. El método `has` devuelve `true` si el elemento está presente y su valor no es `null`:
> > To determine if an item is present in the session, you may use the `has` method. The `has` method returns `true` if the item is present and is not `null`:

    if ($request->session()->has('users')) {
        //
    }

Para determinar si un elemento está presente en la sesión, incluso si su valor es `null`, puede usar el método `exists`. El método `exists` devuelve `true` si el elemento está presente:
> > To determine if an item is present in the session, even if its value is `null`, you may use the `exists` method. The `exists` method returns `true` if the item is present:

    if ($request->session()->exists('users')) {
        //
    }

<a name="storing-data"></a>
### Almacenamiento de datos : Storing Data

Para almacenar datos en la sesión, normalmente usará el método `put` o el helper `session`:
> > To store data in the session, you will typically use the `put` method or the `session` helper:

    // Via a request instance...
    $request->session()->put('key', 'value');

    // Via the global helper...
    session(['key' => 'value']);

#### Empujar para organizar valores de sesión : Pushing To Array Session Values

El método `push` se puede usar para insertar un nuevo valor en un valor de sesión que sea un array. Por ejemplo, si la clave `user.teams` contiene un array de nombres de equipos, puede insertar un nuevo valor en el array de la siguiente manera:
> > The `push` method may be used to push a new value onto a session value that is an array. For example, if the `user.teams` key contains an array of team names, you may push a new value onto the array like so:

    $request->session()->push('user.teams', 'developers');

#### Recuperación y eliminación de un elemento : Retrieving & Deleting An Item

El método `pull` recuperará y eliminará un elemento de la sesión en una única declaración:
> > The `pull` method will retrieve and delete an item from the session in a single statement:

    $value = $request->session()->pull('key', 'default');

<a name="flash-data"></a>
### Datos Flash : Flash Data

A veces puede desear almacenar elementos en la sesión solo para la próxima solicitud. Puedes hacerlo usando el método `flash`. Los datos almacenados en la sesión utilizando este método solo estarán disponibles durante la solicitud HTTP posterior, y luego serán eliminados. Los datos Flash son principalmente útiles para mensajes de estado de corta duración:
> > Sometimes you may wish to store items in the session only for the next request. You may do so using the `flash` method. Data stored in the session using this method will only be available during the subsequent HTTP request, and then will be deleted. Flash data is primarily useful for short-lived status messages:

    $request->session()->flash('status', 'Task was successful!');

Si necesita mantener sus datos flash para varias solicitudes, puede usar el método `reflash`, que conservará todos los datos flash para una solicitud adicional. Si solo necesita mantener datos flash específicos, puede usar el método `keep`:
> > If you need to keep your flash data around for several requests, you may use the `reflash` method, which will keep all of the flash data for an additional request. If you only need to keep specific flash data, you may use the `keep` method:

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

<a name="deleting-data"></a>
### Eliminar datos : Deleting Data

El método `forget` eliminará un dato de la sesión. Si desea eliminar todos los datos de la sesión, puede usar el método `flush`:
> > The `forget` method will remove a piece of data from the session. If you would like to remove all data from the session, you may use the `flush` method:

    $request->session()->forget('key');

    $request->session()->flush();

<a name="regenerating-the-session-id"></a>
### Regeneración del ID de sesión : Regenerating The Session ID

La regeneración del ID de sesión a menudo se realiza para evitar que los usuarios malintencionados exploten un ataque de [fijación de sesión](https://en.wikipedia.org/wiki/Session_fixation) en su aplicación.
> > Regenerating the session ID is often done in order to prevent malicious users from exploiting a [session fixation](https://en.wikipedia.org/wiki/Session_fixation) attack on your application.

Laravel regenera automáticamente el ID de sesión durante la autenticación si está utilizando el `LoginController` incorporado; sin embargo, si necesita regenerar manualmente la ID de la sesión, puede usar el método `regenerate`.
> > Laravel automatically regenerates the session ID during authentication if you are using the built-in `LoginController`; however, if you need to manually regenerate the session ID, you may use the `regenerate` method.

    $request->session()->regenerate();

<a name="adding-custom-session-drivers"></a>
## Agregar controladores de sesión personalizados : Adding Custom Session Drivers

<a name="implementing-the-driver"></a>
#### Implementando el Driver : Implementing The Driver

Su controlador de sesión personalizado debería implementar `SessionHandlerInterface`. Esta interfaz contiene solo algunos métodos simples que debemos implementar. Una implementación obsoleta de MongoDB se ve así:
> > Your custom session driver should implement the `SessionHandlerInterface`. This interface contains just a few simple methods we need to implement. A stubbed MongoDB implementation looks something like this:

    <?php

    namespace App\Extensions;

    class MongoSessionHandler implements \SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }

> {tip} Laravel no incluye un directorio para contener sus extensiones. Usted es libre de colocarlos en cualquier lugar que desee. En este ejemplo, hemos creado un directorio `Extensions` para alojar `MongoSessionHandler`.
> > > {tip} Laravel does not ship with a directory to contain your extensions. You are free to place them anywhere you like. In this example, we have created an `Extensions` directory to house the `MongoSessionHandler`.

Dado que el propósito de estos métodos no es fácil de entender, cubramos rápidamente lo que hace cada uno de los métodos:
> > Since the purpose of these methods is not readily understandable, let's quickly cover what each of the methods do:

<div class="content-list" markdown="1">
- El método `open` normalmente se usaría en sistemas de almacenamiento de sesiones basados ​​en archivos. Dado que Laravel se envía con un controlador de sesión `file`, casi nunca tendrás que poner nada en este método. Puedes dejarlo como un trozo vacío. Es un hecho del diseño de interfaz deficiente (que discutiremos más adelante) que PHP requiere que implementemos este método.
- El método `close`, como el método `open`, generalmente también puede descartarse. Para la mayoría de los drivers, no es necesario.
- El método `read` debe devolver la versión de cadena de los datos de sesión asociados con el `$sessionId` dado. No es necesario hacer ninguna serialización u otra codificación al recuperar o almacenar datos de sesión en su controlador, ya que Laravel realizará la serialización por usted.
- El método `write` debe escribir la cadena `$data` dada asociada con `$sessionId` a algún sistema de almacenamiento persistente, como MongoDB, Dynamo, etc. Nuevamente, no debe realizar ninguna serialización - Laravel ya habrá manejado eso para ti.
- El método `destroy` debe eliminar los datos asociados con `$sessionId` del almacenamiento persistente.
- El método `gc` debe destruir todos los datos de sesión que sean más antiguos que el `$lifetime` dado, que es una marca de tiempo UNIX. Para sistemas que expiran, como Memcached y Redis, este método puede dejarse vacío.
> > - The `open` method would typically be used in file based session store systems. Since Laravel ships with a `file` session driver, you will almost never need to put anything in this method. You can leave it as an empty stub. It is a fact of poor interface design (which we'll discuss later) that PHP requires us to implement this method.
> > - The `close` method, like the `open` method, can also usually be disregarded. For most drivers, it is not needed.
> > - The `read` method should return the string version of the session data associated with the given `$sessionId`. There is no need to do any serialization or other encoding when retrieving or storing session data in your driver, as Laravel will perform the serialization for you.
> > - The `write` method should write the given `$data` string associated with the `$sessionId` to some persistent storage system, such as MongoDB, Dynamo, etc.  Again, you should not perform any serialization - Laravel will have already handled that for you.
> > - The `destroy` method should remove the data associated with the `$sessionId` from persistent storage.
> > - The `gc` method should destroy all session data that is older than the given `$lifetime`, which is a UNIX timestamp. For self-expiring systems like Memcached and Redis, this method may be left empty.
</div>

<a name="registering-the-driver"></a>
#### Registrando el Driver : Registering The Driver

Una vez que se haya implementado su controlador, está listo para registrarlo en el marco. Para agregar controladores adicionales al back-end de la sesión de Laravel, puede usar el método `extend` en la [fachada](/docs/{{version}}/facades) `Session`. Debe llamar al método `extend` del método `boot` de un [proveedor de servicios](/docs/{{version}}/providers). Puede hacer esto desde el `AppServiceProvider` existente o crear un proveedor completamente nuevo:
> > Once your driver has been implemented, you are ready to register it with the framework. To add additional drivers to Laravel's session backend, you may use the `extend` method on the `Session` [facade](/docs/{{version}}/facades). You should call the `extend` method from the `boot` method of a [service provider](/docs/{{version}}/providers). You may do this from the existing `AppServiceProvider` or create an entirely new provider:

    <?php

    namespace App\Providers;

    use App\Extensions\MongoSessionHandler;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Session::extend('mongo', function ($app) {
                // Return implementation of SessionHandlerInterface...
                return new MongoSessionHandler;
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Una vez que se haya registrado el controlador de sesión, puede usar el controlador `mongo` en su archivo de configuración `config/session.php`.
> > Once the session driver has been registered, you may use the `mongo` driver in your `config/session.php` configuration file.
