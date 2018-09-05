# Middleware

- [Introduction](#introduction)
- [Defining Middleware](#defining-middleware)
- [Registering Middleware](#registering-middleware)
    - [Global Middleware](#global-middleware)
    - [Assigning Middleware To Routes](#assigning-middleware-to-routes)
    - [Middleware Groups](#middleware-groups)
- [Middleware Parameters](#middleware-parameters)
- [Terminable Middleware](#terminable-middleware)

<a name="introduction"></a>
## Introducción : Introduction

El Middleware proporciona un mecanismo para filtrar las solicitudes HTTP que ingresan a su aplicación. Por ejemplo, Laravel incluye un middleware que verifica que el usuario de su aplicación esté autenticado. Si el usuario no está autenticado, el middleware redirigirá al usuario a la pantalla de inicio de sesión. Sin embargo, si el usuario está autenticado, el middleware permitirá que la solicitud proceda más adelante en la aplicación.
> > Middleware provide a convenient mechanism for filtering HTTP requests entering your application. For example, Laravel includes a middleware that verifies the user of your application is authenticated. If the user is not authenticated, the middleware will redirect the user to the login screen. However, if the user is authenticated, the middleware will allow the request to proceed further into the application.

Por supuesto, se puede escribir un middleware adicional para realizar gran variedad de tareas además de la autenticación. Un middleware CORS podría ser responsable de agregar los encabezados adecuados a todas las respuestas que dejen su aplicación. Un middleware de registro podría registrar todas las solicitudes entrantes a su aplicación.
> > Of course, additional middleware can be written to perform a variety of tasks besides authentication. A CORS middleware might be responsible for adding the proper headers to all responses leaving your application. A logging middleware might log all incoming requests to your application.

Hay varios middleware incluidos en el framework de Laravel, incluido el middleware para autenticación y protección CSRF. Todos estos middleware se encuentran en el directorio `app/Http/Middleware`.
> > There are several middleware included in the Laravel framework, including middleware for authentication and CSRF protection. All of these middleware are located in the `app/Http/Middleware` directory.

<a name="defining-middleware"></a>
## Definición de Middleware : Defining Middleware

Para crear un nuevo middleware, use el comando Artisan `make:middleware`:
> > To create a new middleware, use the `make:middleware` Artisan command:

    php artisan make:middleware CheckAge

Este comando colocará una nueva clase `CheckAge` dentro de su directorio `app/Http/Middleware`. En este middleware, solo permitiremos el acceso a la ruta si la edad `age` suministrada es superior a 200. De lo contrario, redirigiremos a los usuarios de vuelta al URI `home`:
> > This command will place a new `CheckAge` class within your `app/Http/Middleware` directory. In this middleware, we will only allow access to the route if the supplied `age` is greater than 200. Otherwise, we will redirect the users back to the `home` URI:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckAge
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->age <= 200) {
                return redirect('home');
            }

            return $next($request);
        }
    }

Como puede ver, si la edad `age` dada es menor o igual a `200`, el middleware devolverá un redireccionamiento HTTP al cliente; de lo contrario, la solicitud se transferirá a la aplicación. Para pasar la solicitud más profundamente en la aplicación (permitiendo que el middleware "pase"), llame al callback `$next` con `$request`.
> > As you can see, if the given `age` is less than or equal to `200`, the middleware will return an HTTP redirect to the client; otherwise, the request will be passed further into the application. To pass the request deeper into the application (allowing the middleware to "pass"), call the `$next` callback with the `$request`.

Lo mejor es visualizar el middleware como una serie de "capas" que las solicitudes HTTP deben pasar antes de que lleguen a su aplicación. Cada capa puede examinar la solicitud e incluso rechazarla por completo.
> > It's best to envision middleware as a series of "layers" HTTP requests must pass through before they hit your application. Each layer can examine the request and even reject it entirely.

> {tip} Todos los middleware se resuelven a través del [service container](/docs/{{version}}/container), de modo que puede indicar cualquier dependencia que necesite dentro del constructor de middleware.
> > > {tip} All middleware are resolved via the [service container](/docs/{{version}}/container), so you may type-hint any dependencies you need within a middleware's constructor.

### Antes y después del Middleware : Before & After Middleware

Si un middleware se ejecuta antes o después de una solicitud depende del middleware en sí. Por ejemplo, el siguiente middleware realizaría alguna tarea **antes de** que la aplicación maneje la solicitud:
> > Whether a middleware runs before or after a request depends on the middleware itself. For example, the following middleware would perform some task **before** the request is handled by the application:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class BeforeMiddleware
    {
        public function handle($request, Closure $next)
        {
            // Perform action

            return $next($request);
        }
    }

Sin embargo, este middleware realizaría su tarea **después** de que la solicitud maneje la solicitud:
> > However, this middleware would perform its task **after** the request is handled by the application:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AfterMiddleware
    {
        public function handle($request, Closure $next)
        {
            $response = $next($request);

            // Perform action

            return $response;
        }
    }

<a name="registering-middleware"></a>
## Registro de Middleware : Registering Middleware

<a name="global-middleware"></a>
### Middleware global : Global Middleware

Si desea que se ejecute un middleware para cualquier solicitud HTTP a su aplicación, liste la clase middleware en la propiedad `$middleware` de su clase `app/Http/Kernel.php`.
If you want a middleware to run during every HTTP request to your application, list the middleware class in the `$middleware` property of your `app/Http/Kernel.php` class.

<a name="assigning-middleware-to-routes"></a>
### Asignación de middleware a las rutas : Assigning Middleware To Routes

Si desea asignar un middleware a rutas específicas, primero debe asignar una clave al middleware en su archivo `app/Http/Kernel.php`. Por defecto, la propiedad `$routeMiddleware` de esta clase contiene entradas para el middleware incluido con Laravel. Para agregar el suyo, agréguelo a esta lista y asígnele una clave de su elección. Por ejemplo:
> > If you would like to assign middleware to specific routes, you should first assign the middleware a key in your `app/Http/Kernel.php` file. By default, the `$routeMiddleware` property of this class contains entries for the middleware included with Laravel. To add your own, append it to this list and assign it a key of your choosing. For example:

    // Within App\Http\Kernel Class...

    protected $routeMiddleware = [
        'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    ];

Una vez que el middleware ha sido definido en el núcleo HTTP, puede usar el método `middleware` para asignar middleware a una ruta:
> > Once the middleware has been defined in the HTTP kernel, you may use the `middleware` method to assign middleware to a route:

    Route::get('admin/profile', function () {
        //
    })->middleware('auth');

También puede asignar múltiples middleware a la ruta:
> > You may also assign multiple middleware to the route:

    Route::get('/', function () {
        //
    })->middleware('first', 'second');

Al asignar middleware, también puede pasar el nombre completo de la clase:
> > When assigning middleware, you may also pass the fully qualified class name:

    use App\Http\Middleware\CheckAge;

    Route::get('admin/profile', function () {
        //
    })->middleware(CheckAge::class);

<a name="middleware-groups"></a>
### Grupos de middleware : Middleware Groups

A veces puede querer agrupar varios middleware con una sola clave para facilitar la asignación de rutas. Puede hacer esto usando la propiedad `$middlewareGroups` de su kernel HTTP.
> > Sometimes you may want to group several middleware under a single key to make them easier to assign to routes. You may do this using the `$middlewareGroups` property of your HTTP kernel.

Fuera de la caja, Laravel viene con grupos de middleware `web` y `api` que contienen un middleware común que tal vez quiera aplicar a su UI web y rutas API:
> > Out of the box, Laravel comes with `web` and `api` middleware groups that contain common middleware you may want to apply to your web UI and API routes:

    /**
     * The application's route middleware groups.
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],

        'api' => [
            'throttle:60,1',
            'auth:api',
        ],
    ];

Los grupos de middleware pueden asignarse a rutas y acciones de controlador utilizando la misma sintaxis que el middleware individual. Una vez más, los grupos de middleware hacen que sea más conveniente asignar muchos middleware a una ruta a la vez:
> > Middleware groups may be assigned to routes and controller actions using the same syntax as individual middleware. Again, middleware groups make it more convenient to assign many middleware to a route at once:

    Route::get('/', function () {
        //
    })->middleware('web');

    Route::group(['middleware' => ['web']], function () {
        //
    });

> {tip} Fuera de la caja, el grupo de middleware `web` se aplica automáticamente a su archivo `routes/web.php` por `RouteServiceProvider`.
> > > {tip} Out of the box, the `web` middleware group is automatically applied to your `routes/web.php` file by the `RouteServiceProvider`.

<a name="middleware-parameters"></a>
## Parámetros de Middleware : Middleware Parameters

El Middleware también puede recibir parámetros adicionales. Por ejemplo, si su aplicación necesita verificar que el usuario autenticado tiene un "rol" dado antes de realizar una acción determinada, puede crear un middleware `CheckRole` que reciba un nombre de función como argumento adicional.
> > Middleware can also receive additional parameters. For example, if your application needs to verify that the authenticated user has a given "role" before performing a given action, you could create a `CheckRole` middleware that receives a role name as an additional argument.

Los parámetros de middleware adicionales se pasarán al middleware después del argumento `$next`:
> > Additional middleware parameters will be passed to the middleware after the `$next` argument:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckRole
    {
        /**
         * Handle the incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // Redirect...
            }

            return $next($request);
        }

    }

Los parámetros de middleware pueden especificarse al definir la ruta al separar el nombre del middleware y los parámetros con un `:`. Múltiples parámetros deben estar delimitados por comas:
> > Middleware parameters may be specified when defining the route by separating the middleware name and parameters with a `:`. Multiple parameters should be delimited by commas:

    Route::put('post/{id}', function ($id) {
        //
    })->middleware('role:editor');

<a name="terminable-middleware"></a>
## Terminable Middleware

A veces, un middleware puede necesitar algún trabajo después de que se haya preparado la respuesta HTTP. Por ejemplo, el middleware de "sesión" incluido con Laravel escribe los datos de la sesión en el almacenamiento una vez que la respuesta ha sido completamente preparada. Si define un método `terminate` en su middleware, se llamará automáticamente después de que la respuesta esté lista para enviarse al navegador.
> > Sometimes a middleware may need to do some work after the HTTP response has been prepared. For example, the "session" middleware included with Laravel writes the session data to storage after the response has been fully prepared. If you define a `terminate` method on your middleware, it will automatically be called after the response is ready to be sent to the browser.

    <?php

    namespace Illuminate\Session\Middleware;

    use Closure;

    class StartSession
    {
        public function handle($request, Closure $next)
        {
            return $next($request);
        }

        public function terminate($request, $response)
        {
            // Store the session data...
        }
    }

El método `terminate` debe recibir tanto la solicitud como la respuesta. Una vez que haya definido un middleware terminable, debe agregarlo a la lista de rutas o middleware global en el archivo `app/Http/Kernel.php`.
> > The `terminate` method should receive both the request and the response. Once you have defined a terminable middleware, you should add it to the list of route or global middleware in the `app/Http/Kernel.php` file.

Al invocar el método `terminate` en su middleware, Laravel resolverá una nueva instancia del middleware del [service container](/docs/{{version}}/container). Si desea utilizar la misma instancia de middleware cuando se invocan los métodos `handle` y `terminate`, registre el middleware con el contenedor utilizando el método `singleton` del contenedor.
> > When calling the `terminate` method on your middleware, Laravel will resolve a fresh instance of the middleware from the [service container](/docs/{{version}}/container). If you would like to use the same middleware instance when the `handle` and `terminate` methods are called, register the middleware with the container using the container's `singleton` method.
