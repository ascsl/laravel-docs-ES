# Proveedores de servicio : Service Providers

- [Introduction](#introduction)
- [Writing Service Providers](#writing-service-providers)
    - [The Register Method](#the-register-method)
    - [The Boot Method](#the-boot-method)
- [Registering Providers](#registering-providers)
- [Deferred Providers](#deferred-providers)

<a name="introduction"></a>
## Introducción : Introduction

Los proveedores de servicio son el lugar central de todas las aplicaciones de Laravel bootstrapping. Su propia aplicación, así como todos los servicios básicos de Laravel, se inician a través de proveedores de servicios.
> > Service providers are the central place of all Laravel application bootstrapping. Your own application, as well as all of Laravel's core services are bootstrapped via service providers.

Pero, ¿qué queremos decir con "bootstrapped"? En general, nos referimos a **registrar** cosas, incluido el registro de enlaces de contenedores de servicios, escuchas de eventos, middleware e incluso rutas. Los proveedores de servicios son el centro para configurar su aplicación.
> > But, what do we mean by "bootstrapped"? In general, we mean **registering** things, including registering service container bindings, event listeners, middleware, and even routes. Service providers are the central place to configure your application.

Si abre el archivo `config/app.php` incluido con Laravel, verá un array `providers`. Estas son todas las clases de proveedores de servicios que se cargarán para su aplicación. Por supuesto, muchos de estos son proveedores "diferidos", lo que significa que no se cargarán en cada solicitud, sino solo cuando realmente se necesiten los servicios que brindan.
> > If you open the `config/app.php` file included with Laravel, you will see a `providers` array. These are all of the service provider classes that will be loaded for your application. Of course, many of these are "deferred" providers, meaning they will not be loaded on every request, but only when the services they provide are actually needed.

En esta descripción, aprenderá a escribir sus propios proveedores de servicios y los registrará con su aplicación Laravel.
> > In this overview you will learn how to write your own service providers and register them with your Laravel application.

<a name="writing-service-providers"></a>
## Escribiendo Proveedores de servicio : Writing Service Providers

Todos los proveedores de servicios extienden la clase `Illuminate\Support\ServiceProvider`. La mayoría de los proveedores de servicio contienen un método de `register` y `boot`. Dentro del método `register`, debes **vincular cosas solo en el [service container](/docs/{{version}}/container)**. Nunca intente registrar ningún detector de eventos, rutas ni ninguna otra funcionalidad dentro del método `register`.
> > All service providers extend the `Illuminate\Support\ServiceProvider` class. Most service providers contain a `register` and a `boot` method. Within the `register` method, you should **only bind things into the [service container](/docs/{{version}}/container)**. You should never attempt to register any event listeners, routes, or any other piece of functionality within the `register` method.

La CLI de Artisan puede generar un nuevo proveedor a través del comando `make:provider`:
> > The Artisan CLI can generate a new provider via the `make:provider` command:

    php artisan make:provider RiakServiceProvider

<a name="the-register-method"></a>
### El método de registro : The Register Method

Como se mencionó anteriormente, dentro del método `register`, solo debe enlazar cosas en el [service container](/docs/{{version}}/container). Nunca intente registrar ningún detector de eventos, rutas ni ninguna otra funcionalidad dentro del método `register`. De lo contrario, puede usar accidentalmente un servicio proporcionado por un proveedor de servicios que todavía no se haya cargado.
> > As mentioned previously, within the `register` method, you should only bind things into the [service container](/docs/{{version}}/container). You should never attempt to register any event listeners, routes, or any other piece of functionality within the `register` method. Otherwise, you may accidentally use a service that is provided by a service provider which has not loaded yet.

Echemos un vistazo a un proveedor de servicios básicos. Dentro de cualquiera de sus métodos de proveedor de servicios, siempre tiene acceso a la propiedad `$app` que proporciona acceso al contenedor de servicios:
> > Let's take a look at a basic service provider. Within any of your service provider methods, you always have access to the `$app` property which provides access to the service container:

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection(config('riak'));
            });
        }
    }

Este proveedor de servicios solo define un método de `register` y usa ese método para definir una implementación de `Riak\Connection` en el contenedor de servicios. Si no comprende cómo funciona el contenedor de servicio, consulte [su documentación](/docs/{{version}}/container).
> > This service provider only defines a `register` method, and uses that method to define an implementation of `Riak\Connection` in the service container. If you don't understand how the service container works, check out [its documentation](/docs/{{version}}/container).

#### Las propiedades `bindings` y` singletons` : The `bindings` And `singletons` Properties

Si su proveedor de servicios registra muchos enlaces simples, puede usar las propiedades `bindings` y `singletons` en lugar de registrar manualmente cada enlace de contenedor. Cuando el proveedor de servicios carga el framework, automáticamente verificará estas propiedades y registrará sus enlaces:
> > If your service provider registers many simple bindings, you may wish to use the `bindings` and `singletons` properties instead of manually registering each container binding. When the service provider is loaded by the framework, it will automatically check for these properties and register their bindings:

    <?php

    namespace App\Providers;

    use App\Contracts\ServerProvider;
    use App\Contracts\DowntimeNotifier;
    use Illuminate\Support\ServiceProvider;
    use App\Services\PingdomDowntimeNotifier;
    use App\Services\DigitalOceanServerProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * All of the container bindings that should be registered.
         *
         * @var array
         */
        public $bindings = [
            ServerProvider::class => DigitalOceanServerProvider::class,
        ];

        /**
         * All of the container singletons that should be registered.
         *
         * @var array
         */
        public $singletons = [
            DowntimeNotifier::class => PingdomDowntimeNotifier::class,
        ];
    }

<a name="the-boot-method"></a>
### El método de arranque : The Boot Method

Entonces, ¿qué sucede si tenemos que registrar un compositor de vistas dentro de nuestro proveedor de servicios? Esto debe hacerse dentro del método `boot`. **Se llama a este método después de que se hayan registrado todos los demás proveedores de servicios**, lo que significa que tiene acceso a todos los demás servicios que el framework ha registrado:
> > So, what if we need to register a view composer within our service provider? This should be done within the `boot` method. **This method is called after all other service providers have been registered**, meaning you have access to all other services that have been registered by the framework:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            view()->composer('view', function () {
                //
            });
        }
    }

#### Boot Method Dependency Injection

Puede escribir dependencias de sugerencia para el método `boot` de su proveedor de servicios. El [contenedor de servicios](/docs/{{version}}/container) inyectará automáticamente las dependencias que necesite:
> > You may type-hint dependencies for your service provider's `boot` method. The [service container](/docs/{{version}}/container) will automatically inject any dependencies you need:

    use Illuminate\Contracts\Routing\ResponseFactory;

    public function boot(ResponseFactory $response)
    {
        $response->macro('caps', function ($value) {
            //
        });
    }

<a name="registering-providers"></a>
## Proveedores de registro : Registering Providers

Todos los proveedores de servicios están registrados en el archivo de configuración `config/app.php`. Este archivo contiene un array `providers` donde puede enumerar los nombres de clase de sus proveedores de servicios. De forma predeterminada, un conjunto de proveedores de servicios básicos de Laravel se enumeran en este array. Estos proveedores arrancan los principales componentes de Laravel, como el programa de correo, la cola, el caché y otros.
> > All service providers are registered in the `config/app.php` configuration file. This file contains a `providers` array where you can list the class names of your service providers. By default, a set of Laravel core service providers are listed in this array. These providers bootstrap the core Laravel components, such as the mailer, queue, cache, and others.

Para registrar su proveedor, agréguelo al array:
> > To register your provider, add it to the array:

    'providers' => [
        // Other Service Providers

        App\Providers\ComposerServiceProvider::class,
    ],

<a name="deferred-providers"></a>
## Proveedores diferidos : Deferred Providers

Si su proveedor está **solo** registrando enlaces en el [contenedor de servicio](/docs/{{version}}/container), puede optar por posponer su registro hasta que realmente se necesite uno de los enlaces registrados. Aplazar la carga de dicho proveedor mejorará el rendimiento de su aplicación, ya que no se carga desde el sistema de archivos en cada solicitud.
> > If your provider is **only** registering bindings in the [service container](/docs/{{version}}/container), you may choose to defer its registration until one of the registered bindings is actually needed. Deferring the loading of such a provider will improve the performance of your application, since it is not loaded from the filesystem on every request.

Laravel compila y almacena una lista de todos los servicios suministrados por los proveedores de servicios diferidos, junto con el nombre de su clase de proveedor de servicios. Entonces, solo cuando intenta resolver uno de estos servicios, Laravel carga el proveedor de servicios.
> > Laravel compiles and stores a list of all of the services supplied by deferred service providers, along with the name of its service provider class. Then, only when you attempt to resolve one of these services does Laravel load the service provider.

Para diferir la carga de un proveedor, establezca la propiedad `defer` en `true` y defina un método `provides`. El método `provides` debe devolver los enlaces del contenedor de servicio registrados por el proveedor:
> > To defer the loading of a provider, set the `defer` property to `true` and define a `provides` method. The `provides` method should return the service container bindings registered by the provider:

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Indicates if loading of the provider is deferred.
         *
         * @var bool
         */
        protected $defer = true;

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection($app['config']['riak']);
            });
        }

        /**
         * Get the services provided by the provider.
         *
         * @return array
         */
        public function provides()
        {
            return [Connection::class];
        }

    }
