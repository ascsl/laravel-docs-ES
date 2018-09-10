# Enrutamiento : Routing

- [Basic Routing](#basic-routing)
    - [Redirect Routes](#redirect-routes)
    - [View Routes](#view-routes)
- [Route Parameters](#route-parameters)
    - [Required Parameters](#required-parameters)
    - [Optional Parameters](#parameters-optional-parameters)
    - [Regular Expression Constraints](#parameters-regular-expression-constraints)
- [Named Routes](#named-routes)
- [Route Groups](#route-groups)
    - [Middleware](#route-group-middleware)
    - [Namespaces](#route-group-namespaces)
    - [Sub-Domain Routing](#route-group-sub-domain-routing)
    - [Route Prefixes](#route-group-prefixes)
    - [Route Name Prefixes](#route-group-name-prefixes)
- [Route Model Binding](#route-model-binding)
    - [Implicit Binding](#implicit-binding)
    - [Explicit Binding](#explicit-binding)
- [Fallback Routes](#fallback-routes)
- [Rate Limiting](#rate-limiting)
- [Form Method Spoofing](#form-method-spoofing)
- [Accessing The Current Route](#accessing-the-current-route)

<a name="basic-routing"></a>
## Enrutamiento básico : Basic Routing

Las rutas más básicas de Laravel aceptan un URI y un `Closure`, proporcionando un método muy simple y expresivo para definir rutas:
> > The most basic Laravel routes accept a URI and a `Closure`, providing a very simple and expressive method of defining routes:

    Route::get('foo', function () {
        return 'Hello World';
    });

#### Los archivos de ruta predeterminados : The Default Route Files

Todas las rutas de Laravel se definen en los archivos de ruta, que se encuentran en el directorio `routes`. Estos archivos son cargados automáticamente por el framework. El archivo `routes/web.php` define rutas que son para su interfaz web. A estas rutas se les asigna el grupo de middleware `web`, que proporciona características como estado de sesión y protección CSRF. Las rutas en `routes/api.php` son sin estado y se les asigna el grupo de middleware `api`.
> > All Laravel routes are defined in your route files, which are located in the `routes` directory. These files are automatically loaded by the framework. The `routes/web.php` file defines routes that are for your web interface. These routes are assigned the `web` middleware group, which provides features like session state and CSRF protection. The routes in `routes/api.php` are stateless and are assigned the `api` middleware group.

Para la mayoría de las aplicaciones, comenzará definiendo rutas en su archivo `routes/web.php`. Se puede acceder a las rutas definidas en `routes/web.php` ingresando la URL de la ruta definida en su navegador. Por ejemplo, puede acceder a la siguiente ruta navegando a `http://your-app.test/user` en su navegador:
> > For most applications, you will begin by defining routes in your `routes/web.php` file. The routes defined in `routes/web.php` may be accessed by entering the defined route's URL in your browser. For example, you may access the following route by navigating to `http://your-app.test/user` in your browser:

    Route::get('/user', 'UserController@index');

Las rutas definidas en el archivo `routes/api.php` están anidadas dentro de un grupo de rutas por `RouteServiceProvider`. Dentro de este grupo, el prefijo URI `/api` se aplica automáticamente, por lo que no es necesario aplicarlo manualmente a todas las rutas del archivo. Puede modificar el prefijo y otras opciones de grupo de ruta modificando su clase `RouteServiceProvider`.
> > Routes defined in the `routes/api.php` file are nested within a route group by the `RouteServiceProvider`. Within this group, the `/api` URI prefix is automatically applied so you do not need to manually apply it to every route in the file. You may modify the prefix and other route group options by modifying your `RouteServiceProvider` class.

#### Métodos de enrutador disponibles : Available Router Methods

El enrutador le permite registrar rutas que responden a cualquier verbo HTTP:
> > The router allows you to register routes that respond to any HTTP verb:

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

A veces puede necesitar registrar una ruta que responda a múltiples verbos HTTP. Puedes hacerlo usando el método `match`. O bien, puede incluso registrar una ruta que responda a todos los verbos HTTP utilizando el método `any`:
> > Sometimes you may need to register a route that responds to multiple HTTP verbs. You may do so using the `match` method. Or, you may even register a route that responds to all HTTP verbs using the `any` method:

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

#### Protección CSRF : CSRF Protection

Cualquier formulario HTML que apunte a las rutas `POST`, `PUT` o `DELETE` que están definidas en el archivo de rutas `web` debe incluir un campo de token CSRF. De lo contrario, la solicitud será rechazada. Puede leer más sobre la protección CSRF en la [documentación de CSRF](/docs/{{version}}/csrf):
> > Any HTML forms pointing to `POST`, `PUT`, or `DELETE` routes that are defined in the `web` routes file should include a CSRF token field. Otherwise, the request will be rejected. You can read more about CSRF protection in the [CSRF documentation](/docs/{{version}}/csrf):

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

<a name="redirect-routes"></a>
### Redirigir rutas : Redirect Routes

Si está definiendo una ruta que redirecciona a otro URI, puede usar el método `Route::redirect`. Este método proporciona un acceso directo conveniente para que no tenga que definir una ruta completa o un controlador para realizar una redirección simple:
> > If you are defining a route that redirects to another URI, you may use the `Route::redirect` method. This method provides a convenient shortcut so that you do not have to define a full route or controller for performing a simple redirect:

    Route::redirect('/here', '/there', 301);

<a name="view-routes"></a>
### Ver rutas : View Routes

Si su ruta solo necesita devolver una vista, puede usar el método `Route::view`. Al igual que el método `redirect`, este método proporciona un acceso directo simple para que no tenga que definir una ruta completa o un controlador. El método `view` acepta un URI como primer argumento y un nombre de vista como segundo argumento. Además, puede proporcionar un array de datos para pasar a la vista como tercer argumento opcional:
> > If your route only needs to return a view, you may use the `Route::view` method. Like the `redirect` method, this method provides a simple shortcut so that you do not have to define a full route or controller. The `view` method accepts a URI as its first argument and a view name as its second argument. In addition, you may provide an array of data to pass to the view as an optional third argument:

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

<a name="route-parameters"></a>
## Parámetros de ruta : Route Parameters

<a name="required-parameters"></a>
### Parámetros requeridos : Required Parameters

Por supuesto, a veces necesitará capturar segmentos del URI dentro de su ruta. Por ejemplo, puede que necesite capturar la ID de un usuario de la URL. Puede hacerlo definiendo parámetros de ruta:
> > Of course, sometimes you will need to capture segments of the URI within your route. For example, you may need to capture a user's ID from the URL. You may do so by defining route parameters:

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

Puede definir tantos parámetros de ruta como requiera su ruta:
> > You may define as many route parameters as required by your route:

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

Los parámetros de ruta siempre están encerrados dentro de llaves `{}` y deben consistir en caracteres alfabéticos, y no pueden contener un carácter `-`. En lugar de usar el carácter `-`, use un guión bajo (`_`). Los parámetros de ruta se inyectan en las devoluciones / controladores de ruta según su orden; los nombres de los argumentos de devolución de llamada / controlador no importan.
> > Route parameters are always encased within `{}` braces and should consist of alphabetic characters, and may not contain a `-` character. Instead of using the `-` character, use an underscore (`_`). Route parameters are injected into route callbacks / controllers based on their order - the names of the callback / controller arguments do not matter.

<a name="parameters-optional-parameters"></a>
### Parametros opcionales : Optional Parameters

Ocasionalmente, puede que necesite especificar un parámetro de ruta, pero haga que la presencia de ese parámetro de ruta sea opcional. Puede hacerlo colocando una marca `?` Después del nombre del parámetro. Asegúrese de dar a la variable correspondiente de la ruta un valor predeterminado:
> > Occasionally you may need to specify a route parameter, but make the presence of that route parameter optional. You may do so by placing a `?` mark after the parameter name. Make sure to give the route's corresponding variable a default value:

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### Restricciones de expresiones regulares : Regular Expression Constraints

Puede restringir el formato de los parámetros de ruta utilizando el método `where` en una instancia de ruta. El método `where` acepta el nombre del parámetro y una expresión regular que define cómo se debe restringir el parámetro:
> > You may constrain the format of your route parameters using the `where` method on a route instance. The `where` method accepts the name of the parameter and a regular expression defining how the parameter should be constrained:

    Route::get('user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

<a name="parameters-global-constraints"></a>
#### Restricciones globales : Global Constraints

Si desea que un parámetro de ruta siempre esté restringido por una expresión regular dada, puede usar el método `pattern`. Debe definir estos patrones en el método `boot` de `RouteServiceProvider`:
> > If you would like a route parameter to always be constrained by a given regular expression, you may use the `pattern` method. You should define these patterns in the `boot` method of your `RouteServiceProvider`:

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');

        parent::boot();
    }

Una vez que se ha definido el patrón, se aplica automáticamente a todas las rutas que usan ese nombre de parámetro:
> > Once the pattern has been defined, it is automatically applied to all routes using that parameter name:

    Route::get('user/{id}', function ($id) {
        // Only executed if {id} is numeric...
    });

<a name="named-routes"></a>
## Rutas con nombre : Named Routes

Las rutas con nombre permiten la generación conveniente de URL o redirecciones para rutas específicas. Puede especificar un nombre para una ruta encadenando el método `name` en la definición de ruta:
> > Named routes allow the convenient generation of URLs or redirects for specific routes. You may specify a name for a route by chaining the `name` method onto the route definition:

    Route::get('user/profile', function () {
        //
    })->name('profile');

También puede especificar nombres de ruta para las acciones del controlador:
> > You may also specify route names for controller actions:

    Route::get('user/profile', 'UserProfileController@show')->name('profile');

#### Generación de URL para rutas con nombre : Generating URLs To Named Routes

Una vez que haya asignado un nombre a una ruta determinada, puede usar el nombre de la ruta al generar URL o redirigir a través de la función global `route`:
> > Once you have assigned a name to a given route, you may use the route's name when generating URLs or redirects via the global `route` function:

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

Si la ruta especificada define parámetros, puede pasar los parámetros como segundo argumento a la función `route`. Los parámetros dados se insertarán automáticamente en la URL en sus posiciones correctas:
> > If the named route defines parameters, you may pass the parameters as the second argument to the `route` function. The given parameters will automatically be inserted into the URL in their correct positions:

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

#### Inspeccionando la ruta actual : Inspecting The Current Route

Si desea determinar si la solicitud actual fue enrutada a una ruta determinada, puede usar el método `named` en una instancia de Route. Por ejemplo, puede verificar el nombre de la ruta actual desde un middleware de ruta:
> > If you would like to determine if the current request was routed to a given named route, you may use the `named` method on a Route instance. For example, you may check the current route name from a route middleware:

    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->route()->named('profile')) {
            //
        }

        return $next($request);
    }

<a name="route-groups"></a>
## Grupos de rutas : Route Groups

Los grupos de ruta lo permiten compartir ruta, como middleware o namespaces, en una gran cantidad de rutas sin necesidad de definir los atributos en cada ruta individual. Los atributos compartidos se especifican en un formato de array como primer parámetro del método `Route::group`.
> > Route groups allow you to share route attributes, such as middleware or namespaces, across a large number of routes without needing to define those attributes on each individual route. Shared attributes are specified in an array format as the first parameter to the `Route::group` method.

<a name="route-group-middleware"></a>
### Middleware

Para asignar middleware a todas las rutas dentro de un grupo, puede usar el método `middleware` antes de definir el grupo. Middleware se ejecuta en el orden en que se enumeran en el array:
> > To assign middleware to all routes within a group, you may use the `middleware` method before defining the group. Middleware are executed in the order they are listed in the array:

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // Uses first & second Middleware
        });

        Route::get('user/profile', function () {
            // Uses first & second Middleware
        });
    });

<a name="route-group-namespaces"></a>
### Namespaces

Otro caso de uso común para grupos de rutas es asignar el mismo espacio de nombre PHP a un grupo de controladores que utilizan el método `namespace`:
> > Another common use-case for route groups is assigning the same PHP namespace to a group of controllers using the `namespace` method:

    Route::namespace('Admin')->group(function () {
        // Controllers Within The "App\Http\Controllers\Admin" Namespace
    });

Recuerde, de forma predeterminada, el `RouteServiceProvider` incluye sus archivos de ruta dentro de un grupo de espacio de nombres, lo que le permite registrar las rutas del controlador sin especificar el prefijo completo del namespace `App\Http\Controllers`. Por lo tanto, solo necesita especificar la porción del espacio de nombres que viene después del namespace base de la aplicación `App\Http\Controllers`.
> > Remember, by default, the `RouteServiceProvider` includes your route files within a namespace group, allowing you to register controller routes without specifying the full `App\Http\Controllers` namespace prefix. So, you only need to specify the portion of the namespace that comes after the base `App\Http\Controllers` namespace.

<a name="route-group-sub-domain-routing"></a>
### Enrutamiento de subdominio : Sub-Domain Routing

Los grupos de ruta también se pueden usar para gestionar el enrutamiento de subdominio. Los subdominios pueden tener asignados parámetros de ruta al igual que los URI de ruta, lo que le permite capturar una parte del subdominio para su uso en su ruta o controlador. El subdominio se puede especificar llamando al método `domain` antes de definir el grupo:
> > Route groups may also be used to handle sub-domain routing. Sub-domains may be assigned route parameters just like route URIs, allowing you to capture a portion of the sub-domain for usage in your route or controller. The sub-domain may be specified by calling the `domain` method before defining the group:

    Route::domain('{account}.myapp.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### Prefijos de ruta : Route Prefixes

El método `prefix` se puede usar para prefijar cada ruta en el grupo con un URI dado. Por ejemplo, puede prefijar todos los URI de ruta dentro del grupo con `admin`:
> > The `prefix` method may be used to prefix each route in the group with a given URI. For example, you may want to prefix all route URIs within the group with `admin`:

    Route::prefix('admin')->group(function () {
        Route::get('users', function () {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-group-name-prefixes"></a>
### Prefijos de nombre de ruta : Route Name Prefixes

El método `name` se puede usar para prefijar cada nombre de ruta en el grupo con una cadena dada. Por ejemplo, puede prefijar todos los nombres de las rutas agrupadas con `admin`. La cadena dada se antepone al nombre de la ruta exactamente como se especifica, por lo que nos aseguraremos de proporcionar el carácter `.` en el prefijo:
> > The `name` method may be used to prefix each route name in the group with a given string. For example, you may want to prefix all of the grouped route's names with `admin`. The given string is prefixed to the route name exactly as it is specified, so we will be sure to provide the trailing `.` character in the prefix:

    Route::name('admin.')->group(function () {
        Route::get('users', function () {
            // Route assigned name "admin.users"...
        })->name('users');
    });

<a name="route-model-binding"></a>
## Enlace al modelo de ruta : Route Model Binding

Cuando se inyecta una ID de modelo a una ruta o acción de controlador, con frecuencia se consultará para recuperar el modelo que corresponde a esa ID. La vinculación del modelo de ruta de Laravel proporciona una forma conveniente de inyectar automáticamente las instancias del modelo directamente en sus rutas. Por ejemplo, en lugar de inyectar la ID de un usuario, puede inyectar toda la instancia del modelo `User` que coincida con la ID dada.
> > When injecting a model ID to a route or controller action, you will often query to retrieve the model that corresponds to that ID. Laravel route model binding provides a convenient way to automatically inject the model instances directly into your routes. For example, instead of injecting a user's ID, you can inject the entire `User` model instance that matches the given ID.

<a name="implicit-binding"></a>
### Enlace implícito : Implicit Binding

Laravel resuelve automáticamente los modelos Eloquent definidos en rutas o acciones de controlador cuyos nombres de variables indicados coinciden con un nombre de segmento de ruta. Por ejemplo:
> > Laravel automatically resolves Eloquent models defined in routes or controller actions whose type-hinted variable names match a route segment name. For example:

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

Dado que la variable `$user` se indica como el modelo Eloquent `App\User` y el nombre de la variable coincide con el segmento URI `{user}`, Laravel inyectará automáticamente la instancia del modelo que tiene un ID que coincida con el valor correspondiente el URI de solicitud. Si no se encuentra una instancia de modelo coincidente en la base de datos, se generará automáticamente una respuesta HTTP 404.
> > Since the `$user` variable is type-hinted as the `App\User` Eloquent model and the variable name matches the `{user}` URI segment, Laravel will automatically inject the model instance that has an ID matching the corresponding value from the request URI. If a matching model instance is not found in the database, a 404 HTTP response will automatically be generated.

#### Personalización del nombre de la clave : Customizing The Key Name

Si desea que el enlace modelo utilice una columna de base de datos que no sea `id` al recuperar una clase de modelo determinada, puede anular el método `getRouteKeyName` en el modelo Eloquent:
> > If you would like model binding to use a database column other than `id` when retrieving a given model class, you may override the `getRouteKeyName` method on the Eloquent model:

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="explicit-binding"></a>
### Enlace explícito : Explicit Binding

Para registrar un enlace explícito, use el método `model` del enrutador para especificar la clase para un parámetro dado. Debes definir tus enlaces de modelo explícitos en el método `boot` de la clase `RouteServiceProvider`:
> > To register an explicit binding, use the router's `model` method to specify the class for a given parameter. You should define your explicit model bindings in the `boot` method of the `RouteServiceProvider` class:

    public function boot()
    {
        parent::boot();

        Route::model('user', App\User::class);
    }

A continuación, defina una ruta que contenga un parámetro `{user}`:
> > Next, define a route that contains a `{user}` parameter:

    Route::get('profile/{user}', function (App\User $user) {
        //
    });

Como hemos vinculado todos los parámetros `{user}` al modelo `App\User`, se inyectará una instancia `User` en la ruta. Entonces, por ejemplo, una solicitud a `profile/1` inyectará la instancia `User` de la base de datos que tiene una ID de `1`.
> > Since we have bound all `{user}` parameters to the `App\User` model, a `User` instance will be injected into the route. So, for example, a request to `profile/1` will inject the `User` instance from the database which has an ID of `1`.

Si no se encuentra una instancia de modelo coincidente en la base de datos, se generará automáticamente una respuesta HTTP 404.
> > If a matching model instance is not found in the database, a 404 HTTP response will be automatically generated.

#### Personalización de la lógica de resolución : Customizing The Resolution Logic

Si desea usar su propia lógica de resolución, puede usar el método `Route::bind`. El `Closure` que pasa al método `bind` recibirá el valor del segmento URI y debería devolver la instancia de la clase que se debe inyectar en la ruta:
> > If you wish to use your own resolution logic, you may use the `Route::bind` method. The `Closure` you pass to the `bind` method will receive the value of the URI segment and should return the instance of the class that should be injected into the route:

    public function boot()
    {
        parent::boot();

        Route::bind('user', function ($value) {
            return App\User::where('name', $value)->first() ?? abort(404);
        });
    }

<a name="fallback-routes"></a>
## Rutas de retorno : Fallback Routes

Con el método `Route::fallback`, puede definir una ruta que se ejecutará cuando ninguna otra ruta coincida con la solicitud entrante. Normalmente, las solicitudes no administradas generarán automáticamente una página "404" a través del manejador de excepciones de su aplicación. Sin embargo, dado que puede definir la ruta `fallback` dentro de su archivo `routes/web.php`, todo el middleware en el grupo de midddleware `web` se aplicará a la ruta. Por supuesto, puede agregar middleware adicional a esta ruta según sea necesario:
> > Using the `Route::fallback` method, you may define a route that will be executed when no other route matches the incoming request. Typically, unhandled requests will automatically render a "404" page via your application's exception handler. However, since you may define the `fallback` route within your `routes/web.php` file, all middleware in the `web` midddleware group will apply to the route. Of course, you are free to add additional middleware to this route as needed:

    Route::fallback(function () {
        //
    });

<a name="rate-limiting"></a>
## Límite de frecuencia : Rate Limiting

Laravel incluye un [middleware](/docs/{{version}}/middleware) para calificar el acceso limitado a las rutas dentro de su aplicación. Para comenzar, asigne el middleware `throttle` a una ruta o un grupo de rutas. El middleware `throttle` acepta dos parámetros que determinan la cantidad máxima de solicitudes que se pueden realizar en un número determinado de minutos. Por ejemplo, especifiquemos que un usuario autenticado puede acceder al siguiente grupo de rutas 60 veces por minuto:
> > Laravel includes a [middleware](/docs/{{version}}/middleware) to rate limit access to routes within your application. To get started, assign the `throttle` middleware to a route or a group of routes. The `throttle` middleware accepts two parameters that determine the maximum number of requests that can be made in a given number of minutes. For example, let's specify that an authenticated user may access the following group of routes 60 times per minute:

    Route::middleware('auth:api', 'throttle:60,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

#### Límite de velocidad dinámico : Dynamic Rate Limiting

Puede especificar un máximo de solicitud dinámico basado en un atributo del modelo autenticado de `User`. Por ejemplo, si su modelo `User` contiene un atributo `rate_limit`, puede pasar el nombre del atributo al middleware `throttle` para que se use para calcular el recuento máximo de solicitudes:
> > You may specify a dynamic request maximum based on an attribute of the authenticated `User` model. For example, if your `User` model contains a `rate_limit` attribute, you may pass the name of the attribute to the `throttle` middleware so that it is used to calculate the maximum request count:

    Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

<a name="form-method-spoofing"></a>
## Método de formulario Spoofing : Form Method Spoofing

Los formularios HTML no son compatibles con las acciones `PUT`, `PATCH` o `DELETE`. Por lo tanto, al definir las rutas `PUT`, `PATCH` o `DELETE` que se invocan desde un formulario HTML, deberá agregar un campo oculto `_method` al formulario. El valor enviado con el campo `_method` se usará como el método de solicitud HTTP:
> > HTML forms do not support `PUT`, `PATCH` or `DELETE` actions. So, when defining `PUT`, `PATCH` or `DELETE` routes that are called from an HTML form, you will need to add a hidden `_method` field to the form. The value sent with the `_method` field will be used as the HTTP request method:

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

Puede usar la directiva Blade `@method` para generar la entrada `_method`:
> > You may use the `@method` Blade directive to generate the `_method` input:

    <form action="/foo/bar" method="POST">
        @method('PUT')
        @csrf
    </form>

<a name="accessing-the-current-route"></a>
## Accediendo a la ruta actual : Accessing The Current Route

Puede usar los métodos `current`, `currentRouteName` y `currentRouteAction` en la fachada `Route` para acceder a la información sobre la ruta que maneja la solicitud entrante:
> > You may use the `current`, `currentRouteName`, and `currentRouteAction` methods on the `Route` facade to access information about the route handling the incoming request:

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

Consulte la documentación de la API para [la clase subyacente de la fachada de la ruta](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) y [Route instance](https://laravel.com/api/{{version}}/Iluminate/Routing/Route.html) para revisar todos los métodos accesibles.
> > Refer to the API documentation for both the [underlying class of the Route facade](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) and [Route instance](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) to review all accessible methods.
