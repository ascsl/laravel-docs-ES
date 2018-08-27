# Contenedor de servicio : Service Container

- [Introduction](#introduction)
- [Binding](#binding)
    - [Binding Basics](#binding-basics)
    - [Binding Interfaces To Implementations](#binding-interfaces-to-implementations)
    - [Contextual Binding](#contextual-binding)
    - [Tagging](#tagging)
    - [Extending Bindings](#extending-bindings)
- [Resolving](#resolving)
    - [The Make Method](#the-make-method)
    - [Automatic Injection](#automatic-injection)
- [Container Events](#container-events)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## Introducción : Introduction

El contenedor de servicios de Laravel es una poderosa herramienta para administrar dependencias de clases y realizar inyecciones de dependencia. La inyección de dependencia es una frase sofisticada que básicamente significa esto: las dependencias de clase se "inyectan" en la clase a través del constructor o, en algunos casos, de los métodos "setter".
> > The Laravel service container is a powerful tool for managing class dependencies and performing dependency injection. Dependency injection is a fancy phrase that essentially means this: class dependencies are "injected" into the class via the constructor or, in some cases, "setter" methods.

Veamos un ejemplo simple:
> > Let's look at a simple example:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Repositories\UserRepository;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            $user = $this->users->find($id);

            return view('user.profile', ['user' => $user]);
        }
    }

En este ejemplo, el `UserController` necesita recuperar usuarios de una fuente de datos. Por lo tanto, **inyectaremos** un servicio que pueda recuperar usuarios. En este contexto, es muy probable que nuestro `UserRepository` use [Eloquent](/docs/{{version}}/eloquent) para recuperar información del usuario de la base de datos. Sin embargo, dado que el repositorio se inyecta, podemos intercambiarlo fácilmente con otra implementación. También podemos "simular" fácilmente o crear una implementación ficticia del `UserRepository` al probar nuestra aplicación.
> > In this example, the `UserController` needs to retrieve users from a data source. So, we will **inject** a service that is able to retrieve users. In this context, our `UserRepository` most likely uses [Eloquent](/docs/{{version}}/eloquent) to retrieve user information from the database. However, since the repository is injected, we are able to easily swap it out with another implementation. We are also able to easily "mock", or create a dummy implementation of the `UserRepository` when testing our application.

Una comprensión profunda del contenedor de servicios de Laravel es esencial para construir una aplicación poderosa y grande, así como para contribuir con el núcleo de Laravel.
> > A deep understanding of the Laravel service container is essential to building a powerful, large application, as well as for contributing to the Laravel core itself.

<a name="binding"></a>
## Unión : Binding

<a name="binding-basics"></a>
### Enlaces básicos : Binding Basics

Casi todos sus enlaces de contenedor de servicio se registrarán en [proveedores de servicios](/docs/{{version}}/providers), por lo que la mayoría de estos ejemplos demostrarán el uso del contenedor en ese contexto.
> > Almost all of your service container bindings will be registered within [service providers](/docs/{{version}}/providers), so most of these examples will demonstrate using the container in that context.

> {tip} No es necesario vincular clases al contenedor si no dependen de ninguna interfaz. El contenedor no necesita instrucciones sobre cómo construir estos objetos, ya que puede resolver automáticamente estos objetos mediante el reflejo.
> > > {tip} There is no need to bind classes into the container if they do not depend on any interfaces. The container does not need to be instructed on how to build these objects, since it can automatically resolve these objects using reflection.

#### Enlaces simples : Simple Bindings

Dentro de un proveedor de servicios, siempre tiene acceso al contenedor a través de la propiedad `$this->app`. Podemos registrar un enlace usando el método `bind`, pasando el nombre de clase o interfaz que deseamos registrar junto con un `Closure` que devuelve una instancia de la clase:
> > Within a service provider, you always have access to the container via the `$this->app` property. We can register a binding using the `bind` method, passing the class or interface name that we wish to register along with a `Closure` that returns an instance of the class:

    $this->app->bind('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

Tenga en cuenta que recibimos el contenedor en sí como un argumento para el resolver. Luego podemos usar el contenedor para resolver las subdependencias del objeto que estamos construyendo.
> > Note that we receive the container itself as an argument to the resolver. We can then use the container to resolve sub-dependencies of the object we are building.

#### Enlace a Singleton : Binding A Singleton

El método `singleton` vincula una clase o interfaz en el contenedor que solo se debe resolver una vez. Una vez que se resuelve un enlace único, la misma instancia de objeto se devolverá en las siguientes llamadas al contenedor:
> > The `singleton` method binds a class or interface into the container that should only be resolved one time. Once a singleton binding is resolved, the same object instance will be returned on subsequent calls into the container:

    $this->app->singleton('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

#### Instancias de enlace : Binding Instances

También puede vincular una instancia de objeto existente en el contenedor utilizando el método `instance`. La instancia dada siempre se devolverá en las siguientes llamadas al contenedor: 
> > You may also bind an existing object instance into the container using the `instance` method. The given instance will always be returned on subsequent calls into the container:

    $api = new HelpSpot\API(new HttpClient);

    $this->app->instance('HelpSpot\API', $api);

#### Enlaces primitivos : Binding Primitives

A veces puede tener una clase que recibe algunas clases inyectadas, pero también necesita un valor primitivo inyectado como un entero. Puede usar fácilmente el enlace contextual para inyectar cualquier valor que su clase pueda necesitar:
> > Sometimes you may have a class that receives some injected classes, but also needs an injected primitive value such as an integer. You may easily use contextual binding to inject any value your class may need:

    $this->app->when('App\Http\Controllers\UserController')
              ->needs('$variableName')
              ->give($value);

<a name="binding-interfaces-to-implementations"></a>
### Enlace de interfaces a implementaciones : Binding Interfaces To Implementations

Una característica muy poderosa del contenedor de servicios es su capacidad para vincular una interfaz a una implementación determinada. Por ejemplo, supongamos que tenemos una interfaz `EventPusher` y una implementación `RedisEventPusher`. Una vez que hayamos codificado nuestra implementación `RedisEventPusher` de esta interfaz, podemos registrarla en el contenedor de servicios de la siguiente manera:
> > A very powerful feature of the service container is its ability to bind an interface to a given implementation. For example, let's assume we have an `EventPusher` interface and a `RedisEventPusher` implementation. Once we have coded our `RedisEventPusher` implementation of this interface, we can register it with the service container like so:

    $this->app->bind(
        'App\Contracts\EventPusher',
        'App\Services\RedisEventPusher'
    );

Esta declaración le dice al contenedor que debe inyectar el `RedisEventPusher` cuando una clase necesita una implementación de `EventPusher`. Ahora podemos indicar la interfaz `EventPusher` en un constructor, o en cualquier otra ubicación donde el contenedor del servicio inyecte dependencias:
> > This statement tells the container that it should inject the `RedisEventPusher` when a class needs an implementation of `EventPusher`. Now we can type-hint the `EventPusher` interface in a constructor, or any other location where dependencies are injected by the service container:

    use App\Contracts\EventPusher;

    /**
     * Create a new class instance.
     *
     * @param  EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

<a name="contextual-binding"></a>
### Contextual Binding

A veces puede tener dos clases que utilizan la misma interfaz, pero desea inyectar implementaciones diferentes en cada clase. Por ejemplo, dos controladores pueden depender de diferentes implementaciones del `Illuminate\Contracts\Filesystem\Filesystem` [contract](/docs/{{version}}/contracts). Laravel proporciona una interfaz simple y fluida para definir este comportamiento:
> > Sometimes you may have two classes that utilize the same interface, but you wish to inject different implementations into each class. For example, two controllers may depend on different implementations of the `Illuminate\Contracts\Filesystem\Filesystem` [contract](/docs/{{version}}/contracts). Laravel provides a simple, fluent interface for defining this behavior:

    use Illuminate\Support\Facades\Storage;
    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\VideoController;
    use Illuminate\Contracts\Filesystem\Filesystem;

    $this->app->when(PhotoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('local');
              });

    $this->app->when(VideoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('s3');
              });

<a name="tagging"></a>
### Etiquetado : Tagging

Ocasionalmente, es posible que deba resolver toda una determinada "categoría" de enlace. Por ejemplo, tal vez está creando un agregador de informes que recibe un array de muchas implementaciones de interfaz diferentes de `Report`. Después de registrar las implementaciones de `Report`, puede asignarles una etiqueta usando el método `tag`:
> > Occasionally, you may need to resolve all of a certain "category" of binding. For example, perhaps you are building a report aggregator that receives an array of many different `Report` interface implementations. After registering the `Report` implementations, you can assign them a tag using the `tag` method:

    $this->app->bind('SpeedReport', function () {
        //
    });

    $this->app->bind('MemoryReport', function () {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

Una vez que los servicios han sido etiquetados, puede resolverlos fácilmente a través del método `tagged`:
> > Once the services have been tagged, you may easily resolve them all via the `tagged` method:

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="extending-bindings"></a>
### Extendiendo enlaces : Extending Bindings

El método `extend` permite la modificación de servicios resueltos. Por ejemplo, cuando se resuelve un servicio, puede ejecutar código adicional para decorar o configurar el servicio. El método `extend` acepta un Cierre, que debe devolver el servicio modificado, como su único argumento:
> > The `extend` method allows the modification of resolved services. For example, when a service is resolved, you may run additional code to decorate or configure the service. The `extend` method accepts a Closure, which should return the modified service, as its only argument:

    $this->app->extend(Service::class, function($service) {
        return new DecoratedService($service);
    });

<a name="resolving"></a>
## Resolviendo : Resolving

<a name="the-make-method"></a>
#### El método `make` : The `make` Method

Puede usar el método `make` para resolver una instancia de clase fuera del contenedor. El método `make` acepta el nombre de la clase o interfaz que desea resolver:
> > You may use the `make` method to resolve a class instance out of the container. The `make` method accepts the name of the class or interface you wish to resolve:

    $api = $this->app->make('HelpSpot\API');

Si se encuentra en una ubicación de su código que no tiene acceso a la variable `$app`, puede usar el helper `resolve` global:
> > If you are in a location of your code that does not have access to the `$app` variable, you may use the global `resolve` helper:

    $api = resolve('HelpSpot\API');

Si algunas de las dependencias de su clase no se pueden resolver a través del contenedor, puede inyectarlas pasándolas como un array asociativa en el método `makeWith`:
> > If some of your class' dependencies are not resolvable via the container, you may inject them by passing them as an associative array into the `makeWith` method:

    $api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);

<a name="automatic-injection"></a>
#### Inyección automática : Automatic Injection

Alternativamente, y lo que es más importante, puede "indicar" la dependencia en el constructor de una clase que el contenedor resuelve, incluidos [controladores](/docs/{{version}}/controllers), [detectores de eventos](/docs/{{version}}/events), [trabajos de cola](/docs/{{version}}/queues), [middleware](/docs/{{version}}/middleware) y más. En la práctica, así es como el contenedor debe resolver la mayoría de sus objetos.
> > Alternatively, and importantly, you may "type-hint" the dependency in the constructor of a class that is resolved by the container, including [controllers](/docs/{{version}}/controllers), [event listeners](/docs/{{version}}/events), [queue jobs](/docs/{{version}}/queues), [middleware](/docs/{{version}}/middleware), and more. In practice, this is how most of your objects should be resolved by the container.

Por ejemplo, puede indicar un repositorio definido por su aplicación en el constructor de un controlador. El repositorio se resolverá automáticamente y se inyectará en la clase:
> > For example, you may type-hint a repository defined by your application in a controller's constructor. The repository will automatically be resolved and injected into the class:

    <?php

    namespace App\Http\Controllers;

    use App\Users\Repository as UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }

        /**
         * Show the user with the given ID.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            //
        }
    }

<a name="container-events"></a>
## Eventos de contenedores : Container Events

El contenedor de servicio dispara un evento cada vez que resuelve un objeto. Puedes escuchar este evento usando el método `resolving`:
> > The service container fires an event each time it resolves an object. You may listen to this event using the `resolving` method:

    $this->app->resolving(function ($object, $app) {
        // Called when container resolves object of any type...
    });

    $this->app->resolving(HelpSpot\API::class, function ($api, $app) {
        // Called when container resolves objects of type "HelpSpot\API"...
    });

Como puede ver, el objeto que se está resolviendo se pasará a la devolución de llamada, lo que le permite establecer cualquier propiedad adicional en el objeto antes de que se le dé a su consumidor.
> > As you can see, the object being resolved will be passed to the callback, allowing you to set any additional properties on the object before it is given to its consumer.

<a name="psr-11"></a>
## PSR-11

El contenedor de servicios de Laravel implementa la interfaz [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md). Por lo tanto, puede indicar la interfaz del contenedor del PSR-11 para obtener una instancia del contenedor Laravel:
> > Laravel's service container implements the [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md) interface. Therefore, you may type-hint the PSR-11 container interface to obtain an instance of the Laravel container:

    use Psr\Container\ContainerInterface;

    Route::get('/', function (ContainerInterface $container) {
        $service = $container->get('Service');

        //
    });

> {note} Llamar al método `get` generará una excepción si el identificador no se ha vinculado explícitamente al contenedor.
>>> {note} Calling the `get` method will throw an exception if the identifier has not been explicitly bound into the container.
