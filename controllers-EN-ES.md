# Controladores : Controllers

- [Introduction](#introduction)
- [Basic Controllers](#basic-controllers)
    - [Defining Controllers](#defining-controllers)
    - [Controllers & Namespaces](#controllers-and-namespaces)
    - [Single Action Controllers](#single-action-controllers)
- [Controller Middleware](#controller-middleware)
- [Resource Controllers](#resource-controllers)
    - [Partial Resource Routes](#restful-partial-resource-routes)
    - [Naming Resource Routes](#restful-naming-resource-routes)
    - [Naming Resource Route Parameters](#restful-naming-resource-route-parameters)
    - [Localizing Resource URIs](#restful-localizing-resource-uris)
    - [Supplementing Resource Controllers](#restful-supplementing-resource-controllers)
- [Dependency Injection & Controllers](#dependency-injection-and-controllers)
- [Route Caching](#route-caching)

<a name="introduction"></a>
## Introducción : Introduction

En lugar de definir toda su lógica de gestión de solicitudes como cierres en archivos de ruta, puede organizar este comportamiento utilizando clases de controlador. Los controladores pueden agrupar la lógica de manejo de solicitudes relacionada en una sola clase. Los controladores se almacenan en el directorio `app/Http/Controllers`.
> > Instead of defining all of your request handling logic as Closures in route files, you may wish to organize this behavior using Controller classes. Controllers can group related request handling logic into a single class. Controllers are stored in the `app/Http/Controllers` directory.

<a name="basic-controllers"></a>
## Controladores básicos : Basic Controllers

<a name="defining-controllers"></a>
### Definición de controladores : Defining Controllers

A continuación se muestra un ejemplo de una clase de controlador básico. Tenga en cuenta que el controlador amplía la clase de controlador base incluida con Laravel. La clase base proporciona algunos métodos de conveniencia, como el método `middleware`, que se puede usar para adjuntar middleware a las acciones del controlador:
> > Below is an example of a basic controller class. Note that the controller extends the base controller class included with Laravel. The base class provides a few convenience methods such as the `middleware` method, which may be used to attach middleware to controller actions:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

Puede definir una ruta a esta acción del controlador de la siguiente manera:
> > You can define a route to this controller action like so:

    Route::get('user/{id}', 'UserController@show');

Ahora, cuando una solicitud coincida con la URI de ruta especificada, se ejecutará el método `show` en la clase `UserController`. Por supuesto, los parámetros de ruta también se pasarán al método.
> > Now, when a request matches the specified route URI, the `show` method on the `UserController` class will be executed. Of course, the route parameters will also be passed to the method.

> {tip} Los controladores no son **requridos** para extender una clase base. Sin embargo, no tendrá acceso a funciones de conveniencia como los métodos `middleware`, `validate` y `dispatch`.
> > > {tip} Controllers are not **required** to extend a base class. However, you will not have access to convenience features such as the `middleware`, `validate`, and `dispatch` methods.

<a name="controllers-and-namespaces"></a>
### Controladores y nombres de espacio : Controllers & Namespaces

Es muy importante tener en cuenta que no necesitamos especificar el nombre de espacio completo del controlador al definir la ruta del controlador. Dado que `RouteServiceProvider` carga sus archivos de ruta dentro de un grupo de ruta que contiene el nombre de espacio, solo especificamos la parte del nombre de clase que viene después de la porción `App\Http\Controllers` del nombre de espacio.
> > It is very important to note that we did not need to specify the full controller namespace when defining the controller route. Since the `RouteServiceProvider` loads your route files within a route group that contains the namespace, we only specified the portion of the class name that comes after the `App\Http\Controllers` portion of the namespace.

Si elige anidar sus controladores más profundamente en el directorio `App\Http\Controllers`, use el nombre de clase específico relativo al espacio de nombres raíz `App\Http\Controllers`. Entonces, si su clase de controlador completo es `App\Http\Controllers\Photos\AdminController`, debe registrar las rutas al controlador de la siguiente manera:
> > If you choose to nest your controllers deeper into the `App\Http\Controllers` directory, use the specific class name relative to the `App\Http\Controllers` root namespace. So, if your full controller class is `App\Http\Controllers\Photos\AdminController`, you should register routes to the controller like so:

    Route::get('foo', 'Photos\AdminController@method');

<a name="single-action-controllers"></a>
### Controladores de acción única : Single Action Controllers

Si desea definir un controlador que solo maneje una sola acción, puede colocar un solo método `__invoke` en el controlador:
> > If you would like to define a controller that only handles a single action, you may place a single `__invoke` method on the controller:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class ShowProfile extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function __invoke($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

Al registrar rutas para controladores de acción única, no necesita especificar un método:
> > When registering routes for single action controllers, you do not need to specify a method:

    Route::get('user/{id}', 'ShowProfile');

Puede generar un controlador invocable utilizando la opción `--invokable` del comando Artisan `make:controller`:
> > You may generate an invokable controller by using the `--invokable` option of the `make:controller` Artisan command:

    php artisan make:controller ShowProfile --invokable

<a name="controller-middleware"></a>
## Controlador Middleware : Controller Middleware

[Middleware](/docs/{{version}}/middleware) se pueden asignar a las rutas del controlador en sus archivos de ruta:
> > [Middleware](/docs/{{version}}/middleware) may be assigned to the controller's routes in your route files:

    Route::get('profile', 'UserController@show')->middleware('auth');

Sin embargo, es más conveniente especificar el middleware dentro del constructor de su controlador. Usando el método `middleware` del constructor de su controlador, puede asignar fácilmente middleware a la acción del controlador. Incluso puede restringir el middleware a solo ciertos métodos en la clase de controlador:
> > However, it is more convenient to specify middleware within your controller's constructor. Using the `middleware` method from your controller's constructor, you may easily assign middleware to the controller's action. You may even restrict the middleware to only certain methods on the controller class:

    class UserController extends Controller
    {
        /**
         * Instantiate a new controller instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');

            $this->middleware('log')->only('index');

            $this->middleware('subscribed')->except('store');
        }
    }

Los controladores también le permiten registrar middleware usando un Cierre. Esto proporciona una forma conveniente de definir un middleware para un solo controlador sin definir una clase completa de middleware:
> > Controllers also allow you to register middleware using a Closure. This provides a convenient way to define a middleware for a single controller without defining an entire middleware class:

    $this->middleware(function ($request, $next) {
        // ...

        return $next($request);
    });

> {tip} Puede asignar un middleware a un subconjunto de acciones del controlador; sin embargo, puede indicar que su controlador está creciendo demasiado. En su lugar, considere dividir su controlador en múltiples controladores más pequeños.
> > > {tip} You may assign middleware to a subset of controller actions; however, it may indicate your controller is growing too large. Instead, consider breaking your controller into multiple, smaller controllers.

<a name="resource-controllers"></a>
## Controladores de recursos : Resource Controllers

El enrutamiento de recursos de Laravel asigna las rutas "CRUD" típicas a un controlador con una sola línea de código. Por ejemplo, es posible que desee crear un controlador que maneje todas las solicitudes HTTP para "fotos" almacenadas por su aplicación. Usando el comando Artisan `make:controller`, podemos crear rápidamente dicho controlador:
> > Laravel resource routing assigns the typical "CRUD" routes to a controller with a single line of code. For example, you may wish to create a controller that handles all HTTP requests for "photos" stored by your application. Using the `make:controller` Artisan command, we can quickly create such a controller:

    php artisan make:controller PhotoController --resource

Este comando generará un controlador en `app/Http/Controllers/PhotoController.php`. El controlador contendrá un método para cada una de las operaciones de recursos disponibles.
> > This command will generate a controller at `app/Http/Controllers/PhotoController.php`. The controller will contain a method for each of the available resource operations.

A continuación, puede registrar una ruta ingeniosa para el controlador:
> > Next, you may register a resourceful route to the controller:

    Route::resource('photos', 'PhotoController');

Esta declaración de ruta única crea múltiples rutas para manejar una variedad de acciones en el recurso. El controlador generado ya tendrá métodos restringidos para cada una de estas acciones, incluidas notas que le informan sobre los verbos HTTP y las URI que manejan.
> > This single route declaration creates multiple routes to handle a variety of actions on the resource. The generated controller will already have methods stubbed for each of these actions, including notes informing you of the HTTP verbs and URIs they handle.

Puede registrar muchos controladores de recursos a la vez pasando un aray al método `resources`:
> > You may register many resource controllers at once by passing an array to the `resources` method:

    Route::resources([
        'photos' => 'PhotoController',
        'posts' => 'PostController'
    ]);

#### Acciones manejadas por el controlador de recursos : Actions Handled By Resource Controller

Verb      | URI                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | `/photos`              | index        | photos.index
GET       | `/photos/create`       | create       | photos.create
POST      | `/photos`              | store        | photos.store
GET       | `/photos/{photo}`      | show         | photos.show
GET       | `/photos/{photo}/edit` | edit         | photos.edit
PUT/PATCH | `/photos/{photo}`      | update       | photos.update
DELETE    | `/photos/{photo}`      | destroy      | photos.destroy

#### Especificación del modelo de recursos : Specifying The Resource Model

Si está utilizando el encuadernado del modelo de ruta y desea que los métodos del controlador de recursos indiquen una instancia del modelo, puede usar la opción `--model` al generar el controlador:
> > If you are using route model binding and would like the resource controller's methods to type-hint a model instance, you may use the `--model` option when generating the controller:

    php artisan make:controller PhotoController --resource --model=Photo

#### Métodos de formulario de suplantación : Spoofing Form Methods

Como los formularios HTML no pueden realizar solicitudes `PUT`,` PATCH` o `DELETE`, deberá agregar un campo oculto `_method` para suplantar estos verbos HTTP. La directiva Blade `@method` puede crear este campo para usted:
> > Since HTML forms can't make `PUT`, `PATCH`, or `DELETE` requests, you will need to add a hidden `_method` field to spoof these HTTP verbs. The `@method` Blade directive can create this field for you:

    <form action="/foo/bar" method="POST">
        @method('PUT')
    </form>

<a name="restful-partial-resource-routes"></a>
### Rutas de recursos parciales : Partial Resource Routes

Al declarar una ruta de recursos, puede especificar un subconjunto de acciones que el controlador debería manejar en lugar del conjunto completo de acciones predeterminadas:
> > When declaring a resource route, you may specify a subset of actions the controller should handle instead of the full set of default actions:

    Route::resource('photos', 'PhotoController')->only([
        'index', 'show'
    ]);

    Route::resource('photos', 'PhotoController')->except([
        'create', 'store', 'update', 'destroy'
    ]);

#### Rutas de recursos de la API : API Resource Routes

Al declarar las rutas de recursos que consumirán las API, normalmente querrá excluir las rutas que presenten plantillas HTML, como `create` y `edit`. Por convención, puede usar el método `apiResource` para excluir automáticamente estas dos rutas:
> > When declaring resource routes that will be consumed by APIs, you will commonly want to exclude routes that present HTML templates such as `create` and `edit`. For convenience, you may use the `apiResource` method to automatically exclude these two routes:

    Route::apiResource('photos', 'PhotoController');

Puede registrar muchos controladores de recursos API a la vez pasando un array al método `apiResources`:
> > You may register many API resource controllers at once by passing an array to the `apiResources` method:

    Route::apiResources([
        'photos' => 'PhotoController',
        'posts' => 'PostController'
    ]);

Para generar rápidamente un controlador de recursos API que no incluya los métodos `create` o `edit`, use el interruptor `--api` cuando ejecute el comando `make:controller`:
> > To quickly generate an API resource controller that does not include the `create` or `edit` methods, use the `--api` switch when executing the `make:controller` command:

    php artisan make:controller API/PhotoController --api

<a name="restful-naming-resource-routes"></a>
### Nombrar rutas de recursos : Naming Resource Routes

Por defecto, todas las acciones del controlador de recursos tienen un nombre de ruta; sin embargo, puede anular estos nombres pasando un array `names` con sus opciones:
> > By default, all resource controller actions have a route name; however, you can override these names by passing a `names` array with your options:

    Route::resource('photos', 'PhotoController')->names([
        'create' => 'photos.build'
    ]);

<a name="restful-naming-resource-route-parameters"></a>
### Nombrar parámetros de ruta de recursos : Naming Resource Route Parameters

Por defecto, `Route::resource` creará los parámetros de ruta para sus rutas de recursos basándose en la versión "singularizada" del nombre del recurso. Puede anular esto fácilmente sobreescribiendo el método `parameters`. El array pasado al método `parameters` debe ser un array asociativo de nombres de recursos y nombres de parámetros:
> > By default, `Route::resource` will create the route parameters for your resource routes based on the "singularized" version of the resource name. You can easily override this on a per resource basis by using the `parameters` method. The array passed into the `parameters` method should be an associative array of resource names and parameter names:

    Route::resource('users', 'AdminUserController')->parameters([
        'users' => 'admin_user'
    ]);

El ejemplo anterior genera los siguientes URI para la ruta `show` del recurso:
> > The example above generates the following URIs for the resource's `show` route:

    /users/{admin_user}

<a name="restful-localizing-resource-uris"></a>
### Localizar URIs de recursos : Localizing Resource URIs

Por defecto, `Route::resource` creará URIs de recursos usando verbos en inglés. Si necesita localizar los verbos de acción `create` y `edit`, puede usar el método `Route::resourceVerbs`. Esto se puede hacer en el método `boot` de su `AppServiceProvider`:
> > By default, `Route::resource` will create resource URIs using English verbs. If you need to localize the `create` and `edit` action verbs, you may use the `Route::resourceVerbs` method. This may be done in the `boot` method of your `AppServiceProvider`:

    use Illuminate\Support\Facades\Route;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Route::resourceVerbs([
            'create' => 'crear',
            'edit' => 'editar',
        ]);
    }

Una vez que los verbos se han personalizado, un registro de ruta de recursos como `Route::resource('fotos', 'PhotoController')` producirá los siguientes URIs:
> > Once the verbs have been customized, a resource route registration such as `Route::resource('fotos', 'PhotoController')` will produce the following URIs:

    /fotos/crear

    /fotos/{foto}/editar

<a name="restful-supplementing-resource-controllers"></a>
### Complementando los controladores de recursos : Supplementing Resource Controllers

Si necesita agregar rutas adicionales a un controlador de recursos más allá del conjunto predeterminado de rutas de recursos, debe definir esas rutas antes de su llamada a `Route::resource`; de lo contrario, las rutas definidas por el método `resource` pueden tener prioridad inintencionalmente sobre las rutas suplementarias:
> > If you need to add additional routes to a resource controller beyond the default set of resource routes, you should define those routes before your call to `Route::resource`; otherwise, the routes defined by the `resource` method may unintentionally take precedence over your supplemental routes:

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

> {tip} Recuerde mantener sus controladores enfocados. Si frecuentemente necesita métodos fuera del conjunto típico de acciones de recursos, considere dividir su controlador en dos controladores más pequeños.
> > > {tip} Remember to keep your controllers focused. If you find yourself routinely needing methods outside of the typical set of resource actions, consider splitting your controller into two, smaller controllers.

<a name="dependency-injection-and-controllers"></a>
## inyección de dependencias y controladores : Dependency Injection & Controllers

#### Inyección de constructor : Constructor Injection

El [contenedor de servicio](/docs/{{version}}/container) de Laravel se usa para resolver todos los controladores de Laravel. Como resultado, puede indicar cualquier dependencia que su controlador pueda necesitar en su constructor. Las dependencias declaradas se resolverán automáticamente y se inyectarán en la instancia del controlador:
> > The Laravel [service container](/docs/{{version}}/container) is used to resolve all Laravel controllers. As a result, you are able to type-hint any dependencies your controller may need in its constructor. The declared dependencies will automatically be resolved and injected into the controller instance:

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

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
    }

Por supuesto, también puede indicar cualquier [contrato de Laravel](/docs/{{version}}/contracts). Si el contenedor puede resolverlo, puede indicarlo. Dependiendo de su aplicación, inyectar sus dependencias en su controlador puede proporcionar una mejor capacidad de prueba.
> > Of course, you may also type-hint any [Laravel contract](/docs/{{version}}/contracts). If the container can resolve it, you can type-hint it. Depending on your application, injecting your dependencies into your controller may provide better testability.

#### Método de inyección : Method Injection

Además de la inyección de constructor, también puede indicar dependencias en los métodos de su controlador. Un caso de uso común para la inyección de método es inyectar la instancia `Illuminate\Http\Request` en sus métodos de controlador:
> > In addition to constructor injection, you may also type-hint dependencies on your controller's methods. A common use-case for method injection is injecting the `Illuminate\Http\Request` instance into your controller methods:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Store a new user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->name;

            //
        }
    }

Si su método de controlador también está esperando la entrada de un parámetro de ruta, enumere los argumentos de su ruta después de sus otras dependencias. Por ejemplo, si su ruta se define así:
> > If your controller method is also expecting input from a route parameter, list your route arguments after your other dependencies. For example, if your route is defined like so:

    Route::put('user/{id}', 'UserController@update');

Todavía puede indicar el `Illuminate\Http\Request` y acceder a su parámetro `id` definiendo su método de controlador de la siguiente manera:
> > You may still type-hint the `Illuminate\Http\Request` and access your `id` parameter by defining your controller method as follows:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Update the given user.
         *
         * @param  Request  $request
         * @param  string  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

<a name="route-caching"></a>
## Caché de ruta : Route Caching

> {note} Las rutas basadas en cierres no se pueden almacenar en caché. Para utilizar el almacenamiento en caché de rutas, debe convertir las rutas de cierre en clases de controlador.
> > > {note} Closure based routes cannot be cached. To use route caching, you must convert any Closure routes to controller classes.

Si su aplicación utiliza exclusivamente rutas basadas en controladores, debe aprovechar la caché de rutas de Laravel. Usar el caché de ruta reducirá drásticamente la cantidad de tiempo que lleva registrar todas las rutas de su aplicación. En algunos casos, su registro de ruta puede incluso ser hasta 100 veces más rápido. Para generar un caché de ruta, simplemente ejecuta el comando Artisan `route:cache`:
> > If your application is exclusively using controller based routes, you should take advantage of Laravel's route cache. Using the route cache will drastically decrease the amount of time it takes to register all of your application's routes. In some cases, your route registration may even be up to 100x faster. To generate a route cache, just execute the `route:cache` Artisan command:

    php artisan route:cache

Después de ejecutar este comando, su archivo de rutas en caché se cargará en cada solicitud. Recuerde que si agrega alguna ruta nueva, deberá generar una memoria caché de ruta nueva. Debido a esto, solo debe ejecutar el comando `route:cache` durante la implementación de su proyecto.
> > After running this command, your cached routes file will be loaded on every request. Remember, if you add any new routes you will need to generate a fresh route cache. Because of this, you should only run the `route:cache` command during your project's deployment.

Puede usar el comando `route:clear` para borrar la caché de ruta:
> > You may use the `route:clear` command to clear the route cache:

    php artisan route:clear
