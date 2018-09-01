# Fachadas : Facades

- [Introduction](#introduction)
- [When To Use Facades](#when-to-use-facades)
    - [Facades Vs. Dependency Injection](#facades-vs-dependency-injection)
    - [Facades Vs. Helper Functions](#facades-vs-helper-functions)
- [How Facades Work](#how-facades-work)
- [Real-Time Facades](#real-time-facades)
- [Facade Class Reference](#facade-class-reference)

<a name="introduction"></a>
## Introducción : Introduction

Las fachadas proporcionan una interfaz "estática" para las clases que están disponibles en el [contenedor de servicios](/docs/{{version}}/container) de la aplicación. Laravel se envía con muchas fachadas que proporcionan acceso a casi todas las características de Laravel. Las fachadas de Laravel sirven como "servidores proxy estáticos" para las clases subyacentes en el contenedor de servicios, proporcionando el beneficio de una sintaxis concisa y expresiva a la vez que mantienen una mayor capacidad de prueba y flexibilidad que los métodos estáticos tradicionales.
> > Facades provide a "static" interface to classes that are available in the application's [service container](/docs/{{version}}/container). Laravel ships with many facades which provide access to almost all of Laravel's features. Laravel facades serve as "static proxies" to underlying classes in the service container, providing the benefit of a terse, expressive syntax while maintaining more testability and flexibility than traditional static methods.

Todas las fachadas de Laravel se definen en el espacio de nombres `Illuminate\Support\Facades`. Entonces, podemos acceder fácilmente a una fachada como esta:
> > All of Laravel's facades are defined in the `Illuminate\Support\Facades` namespace. So, we can easily access a facade like so:

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

A lo largo de la documentación de Laravel, muchos de los ejemplos utilizarán fachadas para demostrar diversas características del framework.
> > Throughout the Laravel documentation, many of the examples will use facades to demonstrate various features of the framework.

<a name="when-to-use-facades"></a>
## Cuándo usar fachadas : When To Use Facades

Las fachadas tienen muchos beneficios. Proporcionan una sintaxis breve y memorable que le permite utilizar las características de Laravel sin recordar los nombres largos de clase que deben ser inyectados o configurados manualmente. Además, debido a su uso exclusivo de los métodos dinámicos de PHP, son fáciles de probar.
> > Facades have many benefits. They provide a terse, memorable syntax that allows you to use Laravel's features without remembering long class names that must be injected or configured manually. Furthermore, because of their unique usage of PHP's dynamic methods, they are easy to test.

Sin embargo, se debe tener cuidado al usar fachadas. El principal peligro de las fachadas es el deslizamiento del alcance de clase. Como las fachadas son tan fáciles de usar y no requieren inyección, puede ser fácil dejar que sus clases sigan creciendo y usar muchas fachadas en una sola clase. Al usar la inyección de dependencia, este potencial se ve mitigado por los comentarios visuales que un gran constructor le indica que su clase está creciendo demasiado. Por lo tanto, al usar fachadas, preste especial atención al tamaño de su clase para que su alcance de responsabilidad se mantenga estrecho.
> > However, some care must be taken when using facades. The primary danger of facades is class scope creep. Since facades are so easy to use and do not require injection, it can be easy to let your classes continue to grow and use many facades in a single class. Using dependency injection, this potential is mitigated by the visual feedback a large constructor gives you that your class is growing too large. So, when using facades, pay special attention to the size of your class so that its scope of responsibility stays narrow.

> {tip} Al construir un paquete de terceros que interactúa con Laravel, es mejor inyectar [contratos de Laravel](/docs/{{version}}/contratos) en lugar de utilizar fachadas. Dado que los paquetes se construyen fuera de Laravel, no tendrá acceso a los ayudantes de pruebas de fachada de Laravel.
> > > {tip} When building a third-party package that interacts with Laravel, it's better to inject [Laravel contracts](/docs/{{version}}/contracts) instead of using facades. Since packages are built outside of Laravel itself, you will not have access to Laravel's facade testing helpers.

<a name="facades-vs-dependency-injection"></a>
### Fachadas vs. Inyección de dependencia : Facades Vs. Dependency Injection

Uno de los principales beneficios de la inyección de dependencia es la capacidad de intercambiar implementaciones de la clase inyectada. Esto es útil durante las pruebas ya que puede inyectar un simulacro o un talón y afirmar que se han llamado varios métodos en el talón.
> > One of the primary benefits of dependency injection is the ability to swap implementations of the injected class. This is useful during testing since you can inject a mock or stub and assert that various methods were called on the stub.

Normalmente, no sería posible simular o burlar un método de clase verdaderamente estático. Sin embargo, dado que las fachadas usan métodos dinámicos para realizar llamadas de métodos proxy a objetos resueltos desde el contenedor de servicios, en realidad podemos probar fachadas de la misma manera que probaríamos una instancia de clase inyectada. Por ejemplo, dada la siguiente ruta:
> > Typically, it would not be possible to mock or stub a truly static class method. However, since facades use dynamic methods to proxy method calls to objects resolved from the service container, we actually can test facades just as we would test an injected class instance. For example, given the following route:

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

Podemos escribir la siguiente prueba para verificar que se invocó el método `Cache::get` con el argumento que esperábamos:
> > We can write the following test to verify that the `Cache::get` method was called with the argument we expected:

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $this->visit('/cache')
             ->see('value');
    }

<a name="facades-vs-helper-functions"></a>
### Fachadas vs. Funciones helper : Facades Vs. Helper Functions

Además de las fachadas, Laravel incluye una variedad de funciones de "ayuda" o helpers que pueden realizar tareas comunes como generar vistas, disparar eventos, despachar trabajos o enviar respuestas HTTP. Muchas de estas funciones auxiliares realizan la misma función que una fachada correspondiente. Por ejemplo, esta llamada de fachada y la llamada de helper son equivalentes:
> > In addition to facades, Laravel includes a variety of "helper" functions which can perform common tasks like generating views, firing events, dispatching jobs, or sending HTTP responses. Many of these helper functions perform the same function as a corresponding facade. For example, this facade call and helper call are equivalent:

    return View::make('profile');

    return view('profile');

No hay absolutamente ninguna diferencia práctica entre fachadas y funciones auxiliares. Al usar funciones auxiliares, aún puede probarlas exactamente como lo haría con la fachada correspondiente. Por ejemplo, dada la siguiente ruta:
> > There is absolutely no practical difference between facades and helper functions. When using helper functions, you may still test them exactly as you would the corresponding facade. For example, given the following route:

    Route::get('/cache', function () {
        return cache('key');
    });

Internamente, el helper `cache` va a llamar al método `get` en la clase subyacente a la fachada `Cache`. Entonces, aunque estamos usando la función auxiliar, podemos escribir la siguiente prueba para verificar que el método fue llamado con el argumento que esperábamos:
> > Under the hood, the `cache` helper is going to call the `get` method on the class underlying the `Cache` facade. So, even though we are using the helper function, we can write the following test to verify that the method was called with the argument we expected:

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $this->visit('/cache')
             ->see('value');
    }

<a name="how-facades-work"></a>
## Cómo funcionan las fachadas : How Facades Work

En una aplicación Laravel, una fachada es una clase que proporciona acceso a un objeto desde el contenedor. La maquinaria que hace este trabajo está en la clase `Fachada`. Las fachadas de Laravel y las fachadas personalizadas que crees extenderán la base de la clase `Illuminate\Support\Facades\Facade`.
> > In a Laravel application, a facade is a class that provides access to an object from the container. The machinery that makes this work is in the `Facade` class. Laravel's facades, and any custom facades you create, will extend the base `Illuminate\Support\Facades\Facade` class.

La clase base `Facade` hace uso del método mágico `__callStatic()` para aplazar las llamadas desde su fachada a un objeto resuelto desde el contenedor. En el siguiente ejemplo, se realiza una llamada al sistema de caché de Laravel. Al echar un vistazo a este código, uno podría suponer que el método estático `get` se llama en la clase `Cache`:
> > The `Facade` base class makes use of the `__callStatic()` magic-method to defer calls from your facade to an object resolved from the container. In the example below, a call is made to the Laravel cache system. By glancing at this code, one might assume that the static method `get` is being called on the `Cache` class:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Cache::get('user:'.$id);

            return view('profile', ['user' => $user]);
        }
    }

Tenga en cuenta que en la parte superior del archivo estamos "importando" la fachada `Cache`. Esta fachada sirve como proxy para acceder a la implementación subyacente de la interfaz `Illuminate\Contracts\Cache\Factory`. Todas las llamadas que hagamos usando la fachada se pasarán a la instancia subyacente del servicio de caché de Laravel.
> > Notice that near the top of the file we are "importing" the `Cache` facade. This facade serves as a proxy to accessing the underlying implementation of the `Illuminate\Contracts\Cache\Factory` interface. Any calls we make using the facade will be passed to the underlying instance of Laravel's cache service.

Si observamos la clase `Illuminate\Support\Facades\Cache`, verá que no hay un método estático `get`:
> > If we look at that `Illuminate\Support\Facades\Cache` class, you'll see that there is no static method `get`:

    class Cache extends Facade
    {
        /**
         * Get the registered name of the component.
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }

En cambio, la fachada `Cache` extiende la clase base `Facade` y define el método `getFacadeAccessor()`. El trabajo de este método es devolver el nombre de un enlace de contenedor de servicio. Cuando un usuario hace referencia a cualquier método estático en la fachada `Cache`, Laravel resuelve el enlace `cache` del [service container](/docs/{{version}}/container) y ejecuta el método solicitado (en este caso, `get`) contra ese objeto.
> > Instead, the `Cache` facade extends the base `Facade` class and defines the method `getFacadeAccessor()`. This method's job is to return the name of a service container binding. When a user references any static method on the `Cache` facade, Laravel resolves the `cache` binding from the [service container](/docs/{{version}}/container) and runs the requested method (in this case, `get`) against that object.

<a name="real-time-facades"></a>
## Fachadas en tiempo real : Real-Time Facades

Usando fachadas en tiempo real, puede tratar cualquier clase en su aplicación como si fuera una fachada. Para ilustrar cómo se puede usar esto, examinemos una alternativa. Por ejemplo, supongamos que nuestro modelo `Podcast` tiene un método `publish`. Sin embargo, para publicar el podcast, necesitamos inyectar una instancia `Publisher`:
> > Using real-time facades, you may treat any class in your application as if it were a facade. To illustrate how this can be used, let's examine an alternative. For example, let's assume our `Podcast` model has a `publish` method. However, in order to publish the podcast, we need to inject a `Publisher` instance:

    <?php

    namespace App;

    use App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         *
         * @param  Publisher  $publisher
         * @return void
         */
        public function publish(Publisher $publisher)
        {
            $this->update(['publishing' => now()]);

            $publisher->publish($this);
        }
    }

Inyectar una implementación del editor en el método nos permite probar fácilmente el método de forma aislada ya que podemos simular el editor inyectado. Sin embargo, nos exige pasar siempre una instancia de editor cada vez que llamamos al método `publish`. Al usar fachadas en tiempo real, podemos mantener la misma capacidad de prueba sin tener que pasar explícitamente una instancia de `Publisher`. Para generar una fachada en tiempo real, prefija el nombre de espacio de la clase importada con `Facades`:
> > Injecting a publisher implementation into the method allows us to easily test the method in isolation since we can mock the injected publisher. However, it requires us to always pass a publisher instance each time we call the `publish` method. Using real-time facades, we can maintain the same testability while not being required to explicitly pass a `Publisher` instance. To generate a real-time facade, prefix the namespace of the imported class with `Facades`:

    <?php

    namespace App;

    use Facades\App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         *
         * @return void
         */
        public function publish()
        {
            $this->update(['publishing' => now()]);

            Publisher::publish($this);
        }
    }

Cuando se utiliza la fachada en tiempo real, la implementación del publicador se resolverá fuera del contenedor de servicio utilizando la parte de la interfaz o nombre de clase que aparece después del prefijo `Facades`. Al probar, podemos utilizar los helpers de prueba de fachada incorporados de Laravel para simular esta llamada al método:
> > When the real-time facade is used, the publisher implementation will be resolved out of the service container using the portion of the interface or class name that appears after the `Facades` prefix. When testing, we can use Laravel's built-in facade testing helpers to mock this method call:

    <?php

    namespace Tests\Feature;

    use App\Podcast;
    use Tests\TestCase;
    use Facades\App\Contracts\Publisher;
    use Illuminate\Foundation\Testing\RefreshDatabase;

    class PodcastTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * A test example.
         *
         * @return void
         */
        public function test_podcast_can_be_published()
        {
            $podcast = factory(Podcast::class)->create();

            Publisher::shouldReceive('publish')->once()->with($podcast);

            $podcast->publish();
        }
    }

<a name="facade-class-reference"></a>
## Facade Class Reference

A continuación encontrará cada fachada y su clase subyacente. Esta es una herramienta útil para profundizar rápidamente en la documentación API para una raíz de fachada determinada. La clave [service container binding](/docs/{{version}}/container) también se incluye cuando corresponde.
> > Below you will find every facade and its underlying class. This is a useful tool for quickly digging into the API documentation for a given facade root. The [service container binding](/docs/{{version}}/container) key is also included where applicable.

Facade  |  Class  |  Service Container Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](https://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)  |  `app`
Artisan  |  [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](https://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Contracts\Auth\Guard](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Guard.html)  |  `auth.driver`
Blade  |  [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Broadcast  |  [Illuminate\Contracts\Broadcasting\Factory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Factory.html)  |  &nbsp;
Broadcast (Instance)  |  [Illuminate\Contracts\Broadcasting\Broadcaster](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Broadcaster.html)  |  &nbsp;
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |  &nbsp;
Cache  |  [Illuminate\Cache\CacheManager](https://laravel.com/api/{{version}}/Illuminate/Cache/CacheManager.html)  |  `cache`
Cache (Instance)  |  [Illuminate\Cache\Repository](https://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache.store`
Config  |  [Illuminate\Config\Repository](https://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](https://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](https://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](https://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](https://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |  `db.connection`
Event  |  [Illuminate\Events\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html)  |  &nbsp;
Hash  |  [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Lang  |  [Illuminate\Translation\Translator](https://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Logger](https://laravel.com/api/{{version}}/Illuminate/Log/Logger.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](https://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Notification  |  [Illuminate\Notifications\ChannelManager](https://laravel.com/api/{{version}}/Illuminate/Notifications/ChannelManager.html)  |  &nbsp;
Password  |  [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html)  |  `auth.password`
Password (Instance)  |  [Illuminate\Auth\Passwords\PasswordBroker](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password.broker`
Queue  |  [Illuminate\Queue\QueueManager](https://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance)  |  [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html)  |  `queue.connection`
Queue (Base Class)  |  [Illuminate\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |  &nbsp;
Redirect  |  [Illuminate\Routing\Redirector](https://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\RedisManager](https://laravel.com/api/{{version}}/Illuminate/Redis/RedisManager.html)  |  `redis`
Redis (Instance)  |  [Illuminate\Redis\Connections\Connection](https://laravel.com/api/{{version}}/Illuminate/Redis/Connections/Connection.html)  |  `redis.connection`
Request  |  [Illuminate\Http\Request](https://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |  &nbsp;
Response (Instance)  |  [Illuminate\Http\Response](https://laravel.com/api/{{version}}/Illuminate/Http/Response.html)  |  &nbsp;
Route  |  [Illuminate\Routing\Router](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Builder](https://laravel.com/api/{{version}}/Illuminate/Database/Schema/Builder.html)  |  &nbsp;
Session  |  [Illuminate\Session\SessionManager](https://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](https://laravel.com/api/{{version}}/Illuminate/Session/Store.html)  |  `session.store`
Storage  |  [Illuminate\Filesystem\FilesystemManager](https://laravel.com/api/{{version}}/Illuminate/Filesystem/FilesystemManager.html)  |  `filesystem`
Storage (Instance)  |  [Illuminate\Contracts\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Filesystem.html)  |  `filesystem.disk`
URL  |  [Illuminate\Routing\UrlGenerator](https://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](https://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](https://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html)  |  &nbsp;
View  |  [Illuminate\View\Factory](https://laravel.com/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](https://laravel.com/api/{{version}}/Illuminate/View/View.html)  |  &nbsp;
