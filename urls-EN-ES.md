# Generación de URL : URL Generation

- [Introduction](#introduction)
- [The Basics](#the-basics)
    - [Generating Basic URLs](#generating-basic-urls)
    - [Accessing The Current URL](#accessing-the-current-url)
- [URLs For Named Routes](#urls-for-named-routes)
    - [Signed URLs](#signed-urls)
- [URLs For Controller Actions](#urls-for-controller-actions)
- [Default Values](#default-values)

<a name="introduction"></a>
## Introducción : Introduction

Laravel proporciona varios helpers para ayudarlo a generar URL para su aplicación. Por supuesto, estos son principalmente útiles al crear enlaces en sus plantillas y respuestas API, o al generar respuestas de redireccionamiento a otra parte de su aplicación.
> > Laravel provides several helpers to assist you in generating URLs for your application. Of course, these are mainly helpful when building links in your templates and API responses, or when generating redirect responses to another part of your application.

<a name="the-basics"></a>
## Lo básico : The Basics

<a name="generating-basic-urls"></a>
### Generando URLs básicas : Generating Basic URLs

El helper `url` se puede usar para generar URL arbitrarias para su aplicación. La URL generada utilizará automáticamente el esquema (HTTP o HTTPS) y el host de la solicitud actual:
> > The `url` helper may be used to generate arbitrary URLs for your application. The generated URL will automatically use the scheme (HTTP or HTTPS) and host from the current request:

    $post = App\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>
### Acceso a la URL actual : Accessing The Current URL

Si no se proporciona ninguna ruta al helper `url`, se devuelve una instancia `Illuminate\Routing\UrlGenerator`, que le permite acceder a información sobre la URL actual:
> > If no path is provided to the `url` helper, a `Illuminate\Routing\UrlGenerator` instance is returned, allowing you to access information about the current URL:

    // Get the current URL without the query string...
    echo url()->current();

    // Get the current URL including the query string...
    echo url()->full();

    // Get the full URL for the previous request...
    echo url()->previous();

También se puede acceder a cada uno de estos métodos a través de la [fachada](/docs/{{version}}/facades) `URL`:
> > Each of these methods may also be accessed via the `URL` [facade](/docs/{{version}}/facades):

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
## URLs para rutas con nombre : URLs For Named Routes

El helper `route` se puede usar para generar URL a rutas con nombre. Las rutas con nombre le permiten generar URL sin estar acopladas a la URL real definida en la ruta. Por lo tanto, si la URL de la ruta cambia, no es necesario realizar cambios en las llamadas a la función `route`. Por ejemplo, imagine que su aplicación contiene una ruta definida de la siguiente manera:
> > The `route` helper may be used to generate URLs to named routes. Named routes allow you to generate URLs without being coupled to the actual URL defined on the route. Therefore, if the route's URL changes, no changes need to be made to your `route` function calls. For example, imagine your application contains a route defined like the following:

    Route::get('/post/{post}', function () {
        //
    })->name('post.show');

Para generar una URL a esta ruta, puedes usar el helper `route` de la siguiente manera:
> > To generate a URL to this route, you may use the `route` helper like so:

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

A menudo generará URL utilizando la clave principal de [Modelos Eloquent](/docs/{{version}}/eloquent). Por esta razón, puede pasar modelos Eloquent como valores de parámetros. El helper `route` extraerá automáticamente la clave primaria del modelo:
> > You will often be generating URLs using the primary key of [Eloquent models](/docs/{{version}}/eloquent). For this reason, you may pass Eloquent models as parameter values. The `route` helper will automatically extract the model's primary key:

    echo route('post.show', ['post' => $post]);

<a name="signed-urls"></a>
### URLs firmadas : Signed URLs

Laravel le permite crear fácilmente URLs "firmadas" a rutas con nombre. Estas URL tienen un hash de "firma" adjunto a la cadena de consulta que permite a Laravel verificar que la URL no se haya modificado desde que se creó. Las URL firmadas son especialmente útiles para las rutas que son de acceso público pero necesitan una capa de protección contra la manipulación de URL.
> > Laravel allows you to easily create "signed" URLs to named routes. These URLs have a "signature" hash appended to the query string which allows Laravel to verify that the URL has not been modified since it was created. Signed URLs are especially useful for routes that are publicly accessible yet need a layer of protection against URL manipulation.

Por ejemplo, puede usar URL firmadas para implementar un enlace público de "cancelación de suscripción" que se envía por correo electrónico a sus clientes. Para crear una URL firmada para una ruta con nombre, use el método `signedRoute` de la fachada `URL`:
> > For example, you might use signed URLs to implement a public "unsubscribe" link that is emailed to your customers. To create a signed URL to a named route, use the `signedRoute` method of the `URL` facade:

    use Illuminate\Support\Facades\URL;

    return URL::signedRoute('unsubscribe', ['user' => 1]);

Si desea generar una URL de ruta firmada temporal que expira, puede usar el método `temporarySignedRoute`:
> > If you would like to generate a temporary signed route URL that expires, you may use the `temporarySignedRoute` method:

    use Illuminate\Support\Facades\URL;

    return URL::temporarySignedRoute(
        'unsubscribe', now()->addMinutes(30), ['user' => 1]
    );

#### Validación de solicitudes de ruta firmadas : Validating Signed Route Requests

Para verificar que una solicitud entrante tenga una firma válida, debe llamar al método `hasValidSignature` en la solicitud entrante `Request`:
> > To verify that an incoming request has a valid signature, you should call the `hasValidSignature` method on the incoming `Request`:

    use Illuminate\Http\Request;

    Route::get('/unsubscribe/{user}', function (Request $request) {
        if (! $request->hasValidSignature()) {
            abort(401);
        }

        // ...
    })->name('unsubscribe');

Alternativamente, puede asignar el middleware `Illuminate\Routing\Middleware\ValidateSignature` a la ruta. Si aún no está presente, debe asignarle a este middleware una clave en el array `routeMiddleware` del kernel HTTP:
> > Alternatively, you may assign the `Illuminate\Routing\Middleware\ValidateSignature` middleware to the route. If it is not already present, you should assign this middleware a key in your HTTP kernel's `routeMiddleware` array:

    /**
     * The application's route middleware.
     *
     * These middleware may be assigned to groups or used individually.
     *
     * @var array
     */
    protected $routeMiddleware = [
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    ];

Una vez que haya registrado el middleware en su kernel, puede adjuntarlo a una ruta. Si la solicitud entrante no tiene una firma válida, el middleware devolverá automáticamente una respuesta de error `403`:
> > Once you have registered the middleware in your kernel, you may attach it to a route. If the incoming request does not have a valid signature, the middleware will automatically return a `403` error response:

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed');

<a name="urls-for-controller-actions"></a>
## URLs para las acciones del controlador : URLs For Controller Actions

La función `action` genera una URL para la acción de controlador dada. No necesita pasar el namespace completo del controlador. En su lugar, pase el nombre de la clase del controlador relativo al namespace `App\Http\Controllers`:
> > The `action` function generates a URL for the given controller action. You do not need to pass the full namespace of the controller. Instead, pass the controller class name relative to the `App\Http\Controllers` namespace:

    $url = action('HomeController@index');

También puede hacer referencia a acciones con una sintaxis de array "invocable":
> > You may also reference actions with a "callable" array syntax:

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

Si el método del controlador acepta parámetros de ruta, puede pasarlos como segundo argumento de la función:
> > If the controller method accepts route parameters, you may pass them as the second argument to the function:

    $url = action('UserController@profile', ['id' => 1]);

<a name="default-values"></a>
## Valores predeterminados : Default Values

Para algunas aplicaciones, es posible que desee especificar valores predeterminados de solicitud para ciertos parámetros de URL. Por ejemplo, imagine que muchas de sus rutas definen un parámetro `{locale}`:
> > For some applications, you may wish to specify request-wide default values for certain URL parameters. For example, imagine many of your routes define a `{locale}` parameter:

    Route::get('/{locale}/posts', function () {
        //
    })->name('post.index');

Es engorroso pasar siempre el `locale` cada vez que llamas al asistente `route`. Por lo tanto, puede usar el método `URL::defaults` para definir un valor predeterminado para este parámetro que siempre se aplicará durante la solicitud actual. Es posible que desee llamar a este método desde un [middleware de ruta](/docs/{{version}}/middleware#assigning-middleware-to-routes) para que tenga acceso a la solicitud actual:
> > It is cumbersome to always pass the `locale` every time you call the `route` helper. So, you may use the `URL::defaults` method to define a default value for this parameter that will always be applied during the current request. You may wish to call this method from a [route middleware](/docs/{{version}}/middleware#assigning-middleware-to-routes) so that you have access to the current request:

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Support\Facades\URL;

    class SetDefaultLocaleForUrls
    {
        public function handle($request, Closure $next)
        {
            URL::defaults(['locale' => $request->user()->locale]);

            return $next($request);
        }
    }

Una vez que se ha establecido el valor predeterminado para el parámetro `locale`, ya no es necesario que pase su valor al generar URL a través del helper `route`.
> > Once the default value for the `locale` parameter has been set, you are no longer required to pass its value when generating URLs via the `route` helper.
