# Respuestas HTTP : HTTP Responses

- [Creating Responses](#creating-responses)
    - [Attaching Headers To Responses](#attaching-headers-to-responses)
    - [Attaching Cookies To Responses](#attaching-cookies-to-responses)
    - [Cookies & Encryption](#cookies-and-encryption)
- [Redirects](#redirects)
    - [Redirecting To Named Routes](#redirecting-named-routes)
    - [Redirecting To Controller Actions](#redirecting-controller-actions)
    - [Redirecting To External Domains](#redirecting-external-domains)
    - [Redirecting With Flashed Session Data](#redirecting-with-flashed-session-data)
- [Other Response Types](#other-response-types)
    - [View Responses](#view-responses)
    - [JSON Responses](#json-responses)
    - [File Downloads](#file-downloads)
    - [File Responses](#file-responses)
- [Response Macros](#response-macros)

<a name="creating-responses"></a>
## Creando respuestas : Creating Responses

#### Cadenas y arrays : Strings & Arrays

Todas las rutas y controladores deben devolver una respuesta para ser enviados de vuelta al navegador del usuario. Laravel proporciona varias formas diferentes de devolver respuestas. La respuesta más básica es devolver una cadena de una ruta o controlador. El framework convertirá automáticamente la cadena en una respuesta HTTP completa:
> > All routes and controllers should return a response to be sent back to the user's browser. Laravel provides several different ways to return responses. The most basic response is returning a string from a route or controller. The framework will automatically convert the string into a full HTTP response:

    Route::get('/', function () {
        return 'Hello World';
    });

Además de devolver cadenas desde sus rutas y controladores, también puede devolver arrays. El framework convertirá automáticamente el array en una respuesta JSON:
> > In addition to returning strings from your routes and controllers, you may also return arrays. The framework will automatically convert the array into a JSON response:

    Route::get('/', function () {
        return [1, 2, 3];
    });

> {tip} ¿Sabía que también puede devolver [colecciones Eloquent](/docs/{{version}}/eloquent-collections) de sus rutas o controladores? Se convertirán automáticamente a JSON. ¡Dale un tiro!
> > > {tip} Did you know you can also return [Eloquent collections](/docs/{{version}}/eloquent-collections) from your routes or controllers? They will automatically be converted to JSON. Give it a shot!

#### Objetos de respuesta : Response Objects

Por lo general, no solo devolverá cadenas simples o matrices de sus acciones de ruta. En su lugar, devolverá instancias completas de `Illuminate\Http\Response` o [views](/docs/{{version}}/views).
> > Typically, you won't just be returning simple strings or arrays from your route actions. Instead, you will be returning full `Illuminate\Http\Response` instances or [views](/docs/{{version}}/views).

Devolver una instancia completa de `Response` le permite personalizar el código de estado HTTP y los encabezados de la respuesta. Una instancia `Response` hereda de la clase `Symfony\Component\HttpFoundation\Response`, que proporciona una variedad de métodos para generar respuestas HTTP:
> > Returning a full `Response` instance allows you to customize the response's HTTP status code and headers. A `Response` instance inherits from the `Symfony\Component\HttpFoundation\Response` class, which provides a variety of methods for building HTTP responses:

    Route::get('home', function () {
        return response('Hello World', 200)
                      ->header('Content-Type', 'text/plain');
    });

<a name="attaching-headers-to-responses"></a>
#### Adjuntar encabezados a las respuestas : Attaching Headers To Responses

Tenga en cuenta que la mayoría de los métodos de respuesta son encadenables, lo que permite la construcción fluida de las instancias de respuesta. Por ejemplo, puede usar el método `header` para agregar una serie de encabezados a la respuesta antes de devolverla al usuario:
> > Keep in mind that most response methods are chainable, allowing for the fluent construction of response instances. For example, you may use the `header` method to add a series of headers to the response before sending it back to the user:

    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');

O bien, puede usar el método `withHeaders` para especificar un array de encabezados que se agregarán a la respuesta:
> > Or, you may use the `withHeaders` method to specify an array of headers to be added to the response:

    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-One' => 'Header Value',
                    'X-Header-Two' => 'Header Value',
                ]);

<a name="attaching-cookies-to-responses"></a>
#### Adjuntar cookies a las respuestas : Attaching Cookies To Responses

El método `cookie` en las instancias de respuesta le permite adjuntar fácilmente cookies a la respuesta. Por ejemplo, puede usar el método `cookie` para generar una cookie y adjuntarla de manera fluida a la instancia de respuesta de la siguiente manera:
> > The `cookie` method on response instances allows you to easily attach cookies to the response. For example, you may use the `cookie` method to generate a cookie and fluently attach it to the response instance like so:

    return response($content)
                    ->header('Content-Type', $type)
                    ->cookie('name', 'value', $minutes);

El método `cookie` también acepta algunos argumentos más que se utilizan con menos frecuencia. En general, estos argumentos tienen el mismo propósito y significado que los argumentos que se les daría al método nativo de PHP [setcookie](https://secure.php.net/manual/en/function.setcookie.php):
> > The `cookie` method also accepts a few more arguments which are used less frequently. Generally, these arguments have the same purpose and meaning as the arguments that would be given to PHP's native [setcookie](https://secure.php.net/manual/en/function.setcookie.php) method:

    ->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)

Alternativamente, puede usar la fachada `Cookie` para "poner en cola" las cookies para adjuntarlas a la respuesta saliente de su aplicación. El método `queue` acepta una instancia `Cookie` o los argumentos necesarios para crear una instancia `Cookie`. Estas cookies se adjuntarán a la respuesta de salida antes de enviarse al navegador:
> > Alternatively, you can use the `Cookie` facade to "queue" cookies for attachment to the outgoing response from your application. The `queue` method accepts a `Cookie` instance or the arguments needed to create a `Cookie` instance. These cookies will be attached to the outgoing response before it is sent to the browser:

    Cookie::queue(Cookie::make('name', 'value', $minutes));

    Cookie::queue('name', 'value', $minutes);

<a name="cookies-and-encryption"></a>
#### Cookies y encriptación : Cookies & Encryption

Por defecto, todas las cookies generadas por Laravel están encriptadas y firmadas para que el cliente no pueda modificarlas ni leerlas. Si desea deshabilitar el cifrado de un subconjunto de cookies generado por su aplicación, puede usar la propiedad `$except` del middleware `App\Http\Middleware\EncryptCookies`, que se encuentra en el directorio `app/Http/Middleware`:
> > By default, all cookies generated by Laravel are encrypted and signed so that they can't be modified or read by the client. If you would like to disable encryption for a subset of cookies generated by your application, you may use the `$except` property of the `App\Http\Middleware\EncryptCookies` middleware, which is located in the `app/Http/Middleware` directory:

    /**
     * The names of the cookies that should not be encrypted.
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];

<a name="redirects"></a>
## Redirecciones : Redirects

Las respuestas de redireccionamiento son instancias de la clase `Illuminate\Http\RedirectResponse` y contienen los encabezados adecuados necesarios para redirigir al usuario a otra URL. Hay varias formas de generar una instancia `RedirectResponse`. El método más simple es usar el helper global `redirect`:
> > Redirect responses are instances of the `Illuminate\Http\RedirectResponse` class, and contain the proper headers needed to redirect the user to another URL. There are several ways to generate a `RedirectResponse` instance. The simplest method is to use the global `redirect` helper:

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

En ocasiones, es posible que desee redireccionar al usuario a su ubicación anterior, como cuando un formulario enviado no es válido. Puedes hacerlo usando la función del helper global `back`. Como esta característica utiliza [session](/docs/{{version}}/session), asegúrese de que la ruta que llama a la función `back` esté utilizando el grupo de middleware `web` o que se haya aplicado todo el middleware de sesión:
> > Sometimes you may wish to redirect the user to their previous location, such as when a submitted form is invalid. You may do so by using the global `back` helper function. Since this feature utilizes the [session](/docs/{{version}}/session), make sure the route calling the `back` function is using the `web` middleware group or has all of the session middleware applied:

    Route::post('user/profile', function () {
        // Validate the request...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
### Redirigir a rutas con nombre : Redirecting To Named Routes

Cuando llamas al asistente `redirect` sin parámetros, se devuelve una instancia de `Illuminate\Routing\Redirector`, que te permite llamar a cualquier método en la instancia `Redirector`. Por ejemplo, para generar un `RedirectResponse` a una ruta con nombre, puede usar el método `route`:
> > When you call the `redirect` helper with no parameters, an instance of `Illuminate\Routing\Redirector` is returned, allowing you to call any method on the `Redirector` instance. For example, to generate a `RedirectResponse` to a named route, you may use the `route` method:

    return redirect()->route('login');

Si su ruta tiene parámetros, puede pasarlos como el segundo argumento al método `route`:
> > If your route has parameters, you may pass them as the second argument to the `route` method:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### Rellenar parámetros a través de modelos Eloquent : Populating Parameters Via Eloquent Models

Si está redireccionando a una ruta con un parámetro "ID" que se está rellenando desde un modelo Eloquent, puede pasar el modelo mismo. La identificación se extraerá automáticamente:
> > If you are redirecting to a route with an "ID" parameter that is being populated from an Eloquent model, you may pass the model itself. The ID will be extracted automatically:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', [$user]);

Si desea personalizar el valor que se coloca en el parámetro de ruta, debe anular el método `getRouteKey` en su modelo Eloquent:
> > If you would like to customize the value that is placed in the route parameter, you should override the `getRouteKey` method on your Eloquent model:

    /**
     * Get the value of the model's route key.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### Redirigir a acciones del controlador : Redirecting To Controller Actions

También puede generar redirecciones a [acciones del controlador](/docs/{{version}}/controllers). Para hacerlo, pase el nombre del controlador y la acción al método `action`. Recuerde, no necesita especificar el espacio de nombres completo para el controlador ya que `RouteServiceProvider` de Laravel configurará automáticamente el namespace del controlador base:
> > You may also generate redirects to [controller actions](/docs/{{version}}/controllers). To do so, pass the controller and action name to the `action` method. Remember, you do not need to specify the full namespace to the controller since Laravel's `RouteServiceProvider` will automatically set the base controller namespace:

    return redirect()->action('HomeController@index');

Si la ruta de su controlador requiere parámetros, puede pasarlos como el segundo argumento para el método `action`:
> > If your controller route requires parameters, you may pass them as the second argument to the `action` method:

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );

<a name="redirecting-external-domains"></a>
### Redirigir a dominios externos : Redirecting To External Domains

A veces puede necesitar redirigir a un dominio fuera de su aplicación. Puede hacerlo llamando al método `away`, que crea un `RedirectResponse` sin ninguna codificación URL adicional, validación o verificación:
> > Sometimes you may need to redirect to a domain outside of your application. You may do so by calling the `away` method, which creates a `RedirectResponse` without any additional URL encoding, validation, or verification:

    return redirect()->away('https://www.google.com');

<a name="redirecting-with-flashed-session-data"></a>
### Redirigir con datos de sesión parpadeados : Redirecting With Flashed Session Data

El redireccionamiento a una nueva URL y [datos intermitentes a la sesión](/docs/{{version}}/session#flash-data) generalmente se realizan al mismo tiempo. Por lo general, esto se hace después de realizar con éxito una acción cuando muestra un mensaje de éxito en la sesión. Para su comodidad, puede crear una instancia `RedirectResponse` y actualizar datos a la sesión en una única cadena de métodos fluida:
> > Redirecting to a new URL and [flashing data to the session](/docs/{{version}}/session#flash-data) are usually done at the same time. Typically, this is done after successfully performing an action when you flash a success message to the session. For convenience, you may create a `RedirectResponse` instance and flash data to the session in a single, fluent method chain:

    Route::post('user/profile', function () {
        // Update the user's profile...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

Después de redirigir al usuario, puede mostrar el mensaje de la [sesión](/docs/{{version}}/sesión). Por ejemplo, usando [sintaxis Blade](/docs/{{version}}/blade):
> > After the user is redirected, you may display the flashed message from the [session](/docs/{{version}}/session). For example, using [Blade syntax](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

<a name="other-response-types"></a>
## Otros tipos de respuesta : Other Response Types

El helper `response` se puede usar para generar otros tipos de instancias de respuesta. Cuando se llama al ayudante `response` sin argumentos, se devuelve una implementación de `Illuminate\Contracts\Routing\ResponseFactory` [contract](/docs/{{version}}/contracts). Este contrato proporciona varios métodos útiles para generar respuestas.
> > The `response` helper may be used to generate other types of response instances. When the `response` helper is called without arguments, an implementation of the `Illuminate\Contracts\Routing\ResponseFactory` [contract](/docs/{{version}}/contracts) is returned. This contract provides several helpful methods for generating responses.

<a name="view-responses"></a>
### Ver las respuestas : View Responses

Si necesita control sobre el estado y los encabezados de la respuesta, pero también necesita devolver una [vista](/docs/{{version}}/views) como contenido de la respuesta, debe usar el método `view`:
> > If you need control over the response's status and headers but also need to return a [view](/docs/{{version}}/views) as the response's content, you should use the `view` method:

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);

Por supuesto, si no necesita pasar un código de estado HTTP personalizado o encabezados personalizados, debe usar la función del helper global `view`.
> > Of course, if you do not need to pass a custom HTTP status code or custom headers, you should use the global `view` helper function.

<a name="json-responses"></a>
### Respuestas JSON : JSON Responses

El método `json` configurará automáticamente el encabezado `Content-Type` en `application/json`, y también convertirá el array dado a JSON utilizando la función PHP `json_encode`:
> > The `json` method will automatically set the `Content-Type` header to `application/json`, as well as convert the given array to JSON using the `json_encode` PHP function:

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA'
    ]);

Si desea crear una respuesta JSONP, puede usar el método `json` en combinación con el método `withCallback`:
> > If you would like to create a JSONP response, you may use the `json` method in combination with the `withCallback` method:

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));

<a name="file-downloads"></a>
### Descargas de archivos : File Downloads

El método `download` se puede usar para generar una respuesta que obligue al navegador del usuario a descargar el archivo en la ruta determinada. El método `download` acepta un nombre de archivo como segundo argumento para el método, que determinará el nombre de archivo que ve el usuario que descarga el archivo. Finalmente, puede pasar un array de encabezados HTTP como tercer argumento del método:
> > The `download` method may be used to generate a response that forces the user's browser to download the file at the given path. The `download` method accepts a file name as the second argument to the method, which will determine the file name that is seen by the user downloading the file. Finally, you may pass an array of HTTP headers as the third argument to the method:

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);

    return response()->download($pathToFile)->deleteFileAfterSend(true);

> {note} Symfony HttpFoundation, que administra descargas de archivos, requiere que el archivo que se está descargando tenga un nombre de archivo ASCII.
> > > {note} Symfony HttpFoundation, which manages file downloads, requires the file being downloaded to have an ASCII file name.

#### Descargas por streaming : Streamed Downloads

A veces puede desear convertir la respuesta de cadena de una operación dada en una respuesta descargable sin tener que escribir el contenido de la operación en el disco. Puede usar el método `streamDownload` en este escenario. Este método acepta una devolución de llamada, un nombre de archivo y un array opcional de encabezados como argumentos:
> > Sometimes you may wish to turn the string response of a given operation into a downloadable response without having to write the contents of the operation to disk. You may use the `streamDownload` method in this scenario. This method accepts a callback, file name, and an optional array of headers as its arguments:

    return response()->streamDownload(function () {
        echo GitHub::api('repo')
                    ->contents()
                    ->readme('laravel', 'laravel')['contents'];
    }, 'laravel-readme.md');

<a name="file-responses"></a>
### Respuestas de archivos : File Responses

El método `file` se puede usar para mostrar un archivo, como una imagen o PDF, directamente en el navegador del usuario en lugar de iniciar una descarga. Este método acepta la ruta al archivo como primer argumento y un array de encabezados como segundo argumento:
> > The `file` method may be used to display a file, such as an image or PDF, directly in the user's browser instead of initiating a download. This method accepts the path to the file as its first argument and an array of headers as its second argument:

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

<a name="response-macros"></a>
## Macros de respuesta : Response Macros

Si desea definir una respuesta personalizada que pueda volver a utilizar en una variedad de rutas y controladores, puede usar el método `macro` en la fachada `Response`. Por ejemplo, desde el método `boot` de un [proveedor de servicios](/docs/{{version}}/providers):
> > If you would like to define a custom response that you can re-use in a variety of your routes and controllers, you may use the `macro` method on the `Response` facade. For example, from a [service provider's](/docs/{{version}}/providers) `boot` method:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Response;

    class ResponseMacroServiceProvider extends ServiceProvider
    {
        /**
         * Register the application's response macros.
         *
         * @return void
         */
        public function boot()
        {
            Response::macro('caps', function ($value) {
                return Response::make(strtoupper($value));
            });
        }
    }

La función `macro` acepta un nombre como primer argumento, y un Closure como segundo. El cierre de la macro se ejecutará al invocar el nombre de la macro de una implementación de `ResponseFactory` o del helper `response`:
> > The `macro` function accepts a name as its first argument, and a Closure as its second. The macro's Closure will be executed when calling the macro name from a `ResponseFactory` implementation or the `response` helper:

    return response()->caps('foo');
