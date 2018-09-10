# Solicitudes HTTP : HTTP Requests

- [Accessing The Request](#accessing-the-request)
    - [Request Path & Method](#request-path-and-method)
    - [PSR-7 Requests](#psr7-requests)
- [Input Trimming & Normalization](#input-trimming-and-normalization)
- [Retrieving Input](#retrieving-input)
    - [Old Input](#old-input)
    - [Cookies](#cookies)
- [Files](#files)
    - [Retrieving Uploaded Files](#retrieving-uploaded-files)
    - [Storing Uploaded Files](#storing-uploaded-files)
- [Configuring Trusted Proxies](#configuring-trusted-proxies)

<a name="accessing-the-request"></a>
## Accediendo a la Solicitud : Accessing The Request

Para obtener una instancia de la solicitud HTTP actual a través de la inyección de dependencia, debe escribir-insinuar la clase `Illuminate\Http\Request` en su método de controlador. La instancia de solicitud entrante será inyectada automáticamente por [contenedor de servicio](/docs/{{version}}/container):
> > To obtain an instance of the current HTTP request via dependency injection, you should type-hint the `Illuminate\Http\Request` class on your controller method. The incoming request instance will automatically be injected by the [service container](/docs/{{version}}/container):

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
            $name = $request->input('name');

            //
        }
    }

#### Inyección de dependencia y parámetros de ruta : Dependency Injection & Route Parameters

Si el método de su controlador también espera la entrada de un parámetro de ruta, debe enumerar los parámetros de ruta después de sus otras dependencias. Por ejemplo, si su ruta se define así:
> > If your controller method is also expecting input from a route parameter you should list your route parameters after your other dependencies. For example, if your route is defined like so:

    Route::put('user/{id}', 'UserController@update');

Todavía puede escribir-insinuar el `Illuminate\Http\Request` y acceder a su parámetro de ruta `id` definiendo su método de controlador de la siguiente manera:
> > You may still type-hint the `Illuminate\Http\Request` and access your route parameter `id` by defining your controller method as follows:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Update the specified user.
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

#### Accediendo a la solicitud a través de los Closures de ruta : Accessing The Request Via Route Closures

También puede indicar la clase `Illuminate\Http\Request` en un Closure de ruta. El contenedor de servicio inyectará automáticamente la solicitud entrante en el Closure cuando se ejecute:
> > You may also type-hint the `Illuminate\Http\Request` class on a route Closure. The service container will automatically inject the incoming request into the Closure when it is executed:

    use Illuminate\Http\Request;

    Route::get('/', function (Request $request) {
        //
    });

<a name="request-path-and-method"></a>
### Solicitud de ruta y método : Request Path & Method

La instancia `Illuminate\Http\Request` proporciona una variedad de métodos para examinar la solicitud HTTP para su aplicación y amplía la clase `Symfony\Component\HttpFoundation\Request`. Discutiremos algunos de los métodos más importantes a continuación.
> > The `Illuminate\Http\Request` instance provides a variety of methods for examining the HTTP request for your application and extends the `Symfony\Component\HttpFoundation\Request` class. We will discuss a few of the most important methods below.

#### Recuperación de ruta de la solicitud : Retrieving The Request Path

El método `path` devuelve la información de ruta de la solicitud. Por lo tanto, si la solicitud entrante está dirigida a `http://domain.com/foo/bar`, el método `path` devolverá `foo/bar`:
> > The `path` method returns the request's path information. So, if the incoming request is targeted at `http://domain.com/foo/bar`, the `path` method will return `foo/bar`:

    $uri = $request->path();

El método `is` le permite verificar que la ruta de la solicitud entrante coincida con un patrón dado. Puedes usar el caracter `*` como comodín cuando utilizas este método:
> > The `is` method allows you to verify that the incoming request path matches a given pattern. You may use the `*` character as a wildcard when utilizing this method:

    if ($request->is('admin/*')) {
        //
    }

#### Recuperando la URL de solicitud : Retrieving The Request URL

Para recuperar la URL completa de la solicitud entrante, puede usar los métodos `url` o `fullUrl`. El método `url` devolverá la URL sin la cadena de consulta, mientras que el método `fullUrl` incluye la cadena de consulta:
> > To retrieve the full URL for the incoming request you may use the `url` or `fullUrl` methods. The `url` method will return the URL without the query string, while the `fullUrl` method includes the query string:

    // Without Query String...
    $url = $request->url();

    // With Query String...
    $url = $request->fullUrl();

#### Recuperando el método de solicitud : Retrieving The Request Method

El método `method` devolverá el verbo HTTP para la solicitud. Puede usar el método `isMethod` para verificar que el verbo HTTP coincida con una cadena dada:
> > The `method` method will return the HTTP verb for the request. You may use the `isMethod` method to verify that the HTTP verb matches a given string:

    $method = $request->method();

    if ($request->isMethod('post')) {
        //
    }

<a name="psr7-requests"></a>
### Peticiones PSR-7 : PSR-7 Requests

El [estándar PSR-7](http://www.php-fig.org/psr/psr-7/) especifica interfaces para mensajes HTTP, incluidas solicitudes y respuestas. Si desea obtener una instancia de una solicitud de PSR-7 en lugar de una solicitud de Laravel, primero deberá instalar algunas librerias. Laravel usa el componente * Symfony HTTP Message Bridge * para convertir las solicitudes y respuestas típicas de Laravel en implementaciones compatibles con PSR-7:
> > The [PSR-7 standard](http://www.php-fig.org/psr/psr-7/) specifies interfaces for HTTP messages, including requests and responses. If you would like to obtain an instance of a PSR-7 request instead of a Laravel request, you will first need to install a few libraries. Laravel uses the *Symfony HTTP Message Bridge* component to convert typical Laravel requests and responses into PSR-7 compatible implementations:

    composer require symfony/psr-http-message-bridge
    composer require zendframework/zend-diactoros

Una vez que haya instalado estas bibliotecas, puede obtener una solicitud de PSR-7 al inscribir por tipo la interfaz de solicitud en el método de cierre o control de su ruta:
> > Once you have installed these libraries, you may obtain a PSR-7 request by type-hinting the request interface on your route Closure or controller method:

    use Psr\Http\Message\ServerRequestInterface;

    Route::get('/', function (ServerRequestInterface $request) {
        //
    });

> {tip} Si devuelve una instancia de respuesta del PSR-7 de una ruta o controlador, se convertirá automáticamente a una instancia de respuesta de Laravel y se mostrará en el framework.
> > > {tip} If you return a PSR-7 response instance from a route or controller, it will automatically be converted back to a Laravel response instance and be displayed by the framework.

<a name="input-trimming-and-normalization"></a>
## Input Trimming & Normalization

Por defecto, Laravel incluye el middleware `TrimStrings` y` ConvertEmptyStringsToNull` en la pila de middleware global de su aplicación. Estos middleware se enumeran en la pila por la clase `App\Http\Kernel`. Este middleware recortará automáticamente todos los campos de cadena entrantes en la solicitud, así como también convertirá los campos de cadena vacía en `null`. Esto le permite no tener que preocuparse por estas preocupaciones de normalización en sus rutas y controladores.
> > By default, Laravel includes the `TrimStrings` and `ConvertEmptyStringsToNull` middleware in your application's global middleware stack. These middleware are listed in the stack by the `App\Http\Kernel` class. These middleware will automatically trim all incoming string fields on the request, as well as convert any empty string fields to `null`. This allows you to not have to worry about these normalization concerns in your routes and controllers.

Si desea deshabilitar este comportamiento, puede eliminar los dos middleware de la pila de middleware de su aplicación eliminándolos de la propiedad `$middleware` de su clase `App\Http\Kernel`.
> > If you would like to disable this behavior, you may remove the two middleware from your application's middleware stack by removing them from the `$middleware` property of your `App\Http\Kernel` class.

<a name="retrieving-input"></a>
## Recuperando entrada : Retrieving Input

#### Recuperación de todos los datos de entrada : Retrieving All Input Data

También puede recuperar todos los datos de entrada como un `array` usando el método `all`:
> > You may also retrieve all of the input data as an `array` using the `all` method:

    $input = $request->all();

#### Recuperando un valor de entrada : Retrieving An Input Value

Usando algunos métodos simples, puede acceder a toda la entrada del usuario desde su instancia `Illuminate\Http\Request` sin preocuparse por qué verbo HTTP se utilizó para la solicitud. Independientemente del verbo HTTP, el método `input` se puede usar para recuperar la entrada del usuario:
> > Using a few simple methods, you may access all of the user input from your `Illuminate\Http\Request` instance without worrying about which HTTP verb was used for the request. Regardless of the HTTP verb, the `input` method may be used to retrieve user input:

    $name = $request->input('name');

Puede pasar un valor predeterminado como segundo argumento para el método `input`. Este valor se devolverá si el valor de entrada solicitado no está presente en la solicitud:
> > You may pass a default value as the second argument to the `input` method. This value will be returned if the requested input value is not present on the request:

    $name = $request->input('name', 'Sally');

Cuando trabaje con formularios que contienen entradas de array, use la notación de "punto" para acceder a los arrays:
> > When working with forms that contain array inputs, use "dot" notation to access the arrays:

    $name = $request->input('products.0.name');

    $names = $request->input('products.*.name');

#### Recuperación de entrada de la cadena de consulta : Retrieving Input From The Query String

Mientras que el método `input` recupera valores de toda la carga útil de la solicitud (incluida la cadena de consulta), el método `query` solo recuperará los valores de la cadena de consulta:
> > While the `input` method retrieves values from entire request payload (including the query string), the `query` method will only retrieve values from the query string:

    $name = $request->query('name');

Si los datos de valor de cadena de consulta solicitados no están presentes, se devolverá el segundo argumento a este método:
> > If the requested query string value data is not present, the second argument to this method will be returned:

    $name = $request->query('name', 'Helen');

Puede llamar al método `query` sin ningún argumento para recuperar todos los valores de cadena de consulta como un array asociativo:
> > You may call the `query` method without any arguments in order to retrieve all of the query string values as an associative array:

    $query = $request->query();

#### Recuperación de entrada a través de propiedades dinámicas : Retrieving Input Via Dynamic Properties

También puede acceder a la entrada del usuario utilizando propiedades dinámicas en la instancia `Illuminate\Http\Request`. Por ejemplo, si uno de los formularios de su aplicación contiene un campo `name`, puede acceder al valor del campo de la siguiente manera:
> > You may also access user input using dynamic properties on the `Illuminate\Http\Request` instance. For example, if one of your application's forms contains a `name` field, you may access the value of the field like so:

    $name = $request->name;

Al usar propiedades dinámicas, Laravel primero buscará el valor del parámetro en la carga útil de la solicitud. Si no está presente, Laravel buscará el campo en los parámetros de ruta.
> > When using dynamic properties, Laravel will first look for the parameter's value in the request payload. If it is not present, Laravel will search for the field in the route parameters.

#### Recuperación de valores de entrada JSON : Retrieving JSON Input Values

Al enviar solicitudes JSON a su aplicación, puede acceder a los datos JSON a través del método `input` siempre que el encabezado `Content-Type` de la solicitud esté configurado correctamente en `application/json`. Incluso puede usar la sintaxis del "punto" para profundizar en los arrays JSON:
> > When sending JSON requests to your application, you may access the JSON data via the `input` method as long as the `Content-Type` header of the request is properly set to `application/json`. You may even use "dot" syntax to dig into JSON arrays:

    $name = $request->input('user.name');

#### Recuperación de una porción de los datos de entrada : Retrieving A Portion Of The Input Data

Si necesita recuperar un subconjunto de los datos de entrada, puede usar los métodos `only` y `except`. Ambos métodos aceptan un solo `array` o una lista dinámica de argumentos:
> > If you need to retrieve a subset of the input data, you may use the `only` and `except` methods. Both of these methods accept a single `array` or a dynamic list of arguments:

    $input = $request->only(['username', 'password']);

    $input = $request->only('username', 'password');

    $input = $request->except(['credit_card']);

    $input = $request->except('credit_card');

> {tip} El método `only` devuelve todos los pares clave / valor que usted solicita; sin embargo, no devolverá pares clave / valor que no estén presentes en la solicitud.
> > > {tip} The `only` method returns all of the key / value pairs that you request; however, it will not return key / value pairs that are not present on the request.

#### Determinando si hay un valor de entrada presente : Determining If An Input Value Is Present

Debe usar el método `has` para determinar si hay un valor presente en la solicitud. El método `has` devuelve `true` si el valor está presente en la solicitud:
> > You should use the `has` method to determine if a value is present on the request. The `has` method returns `true` if the value is present on the request:

    if ($request->has('name')) {
        //
    }

Cuando se le da un array, el método `has` determinará si todos los valores especificados están presentes:
> > When given an array, the `has` method will determine if all of the specified values are present:

    if ($request->has(['name', 'email'])) {
        //
    }

Si desea determinar si un valor está presente en la solicitud y no está vacío, puede utilizar el método `filled`:
> > If you would like to determine if a value is present on the request and is not empty, you may use the `filled` method:

    if ($request->filled('name')) {
        //
    }

<a name="old-input"></a>
### Entrada antigua : Old Input

Laravel le permite mantener la entrada de una solicitud durante la próxima solicitud. Esta característica es particularmente útil para rellenar formularios después de detectar errores de validación. Sin embargo, si está utilizando las [funciones de validación incluidas](/docs/{{version}}/validation) de Laravel, es poco probable que necesite usar estos métodos manualmente, ya que algunas de las funciones de validación integradas de Laravel los llamarán automáticamente. .
> > Laravel allows you to keep input from one request during the next request. This feature is particularly useful for re-populating forms after detecting validation errors. However, if you are using Laravel's included [validation features](/docs/{{version}}/validation), it is unlikely you will need to manually use these methods, as some of Laravel's built-in validation facilities will call them automatically.

#### Entrada intermitente a la sesión : Flashing Input To The Session

El método `flash` en la clase `Illuminate\Http\Request` mostrará la entrada actual en la [sesión](/docs/{{version}}/session) para que esté disponible durante la siguiente solicitud del usuario a la aplicación:
> > The `flash` method on the `Illuminate\Http\Request` class will flash the current input to the [session](/docs/{{version}}/session) so that it is available during the user's next request to the application:

    $request->flash();

También puede usar los métodos `flashOnly` y` flashExcept` para mostrar un subconjunto de los datos de solicitud a la sesión. Estos métodos son útiles para mantener la información confidencial, como las contraseñas fuera de la sesión:
> > You may also use the `flashOnly` and `flashExcept` methods to flash a subset of the request data to the session. These methods are useful for keeping sensitive information such as passwords out of the session:

    $request->flashOnly(['username', 'email']);

    $request->flashExcept('password');

#### Entrada intermitente y luego redireccionamiento : Flashing Input Then Redirecting

Dado que a menudo deseará introducir información en la sesión y luego redirigirla a la página anterior, puede encadenar fácilmente el destello de entrada en una redirección utilizando el método `withInput`:
> > Since you often will want to flash input to the session and then redirect to the previous page, you may easily chain input flashing onto a redirect using the `withInput` method:

    return redirect('form')->withInput();

    return redirect('form')->withInput(
        $request->except('password')
    );

#### Recuperación de entradas antiguas : Retrieving Old Input

Para recuperar la entrada flasheada de la solicitud anterior, use el método `old` en la instancia `Request`. El método `old` extraerá los datos de entrada previamente flasheados de la [sesión](/docs/{{version}}/session):
> > To retrieve flashed input from the previous request, use the `old` method on the `Request` instance. The `old` method will pull the previously flashed input data from the [session](/docs/{{version}}/session):

    $username = $request->old('username');

Laravel también proporciona un helper global `old`. Si está visualizando una entrada anterior dentro de una [Plantilla Blade](/docs/{{version}}/blade), es más conveniente usar el helper `old`. Si no existe una entrada anterior para el campo dado, se devolverá `null`:
> > Laravel also provides a global `old` helper. If you are displaying old input within a [Blade template](/docs/{{version}}/blade), it is more convenient to use the `old` helper. If no old input exists for the given field, `null` will be returned:

    <input type="text" name="username" value="{{ old('username') }}">

<a name="cookies"></a>
### Galletas : Cookies

#### Recuperación de cookies de las solicitudes : Retrieving Cookies From Requests

Todas las cookies creadas por el framework de Laravel están encriptadas y firmadas con un código de autenticación, lo que significa que serán consideradas inválidas si el cliente las ha modificado. Para recuperar un valor de cookie de la solicitud, utilice el método `cookie` en una instancia `Illuminate\Http\Request`:
> > All cookies created by the Laravel framework are encrypted and signed with an authentication code, meaning they will be considered invalid if they have been changed by the client. To retrieve a cookie value from the request, use the `cookie` method on a `Illuminate\Http\Request` instance:

    $value = $request->cookie('name');

Alternativamente, puede usar la fachada `Cookie` para acceder a los valores de las cookies:
> > Alternatively, you may use the `Cookie` facade to access cookie values:

    $value = Cookie::get('name');

#### Adjuntar cookies a las respuestas : Attaching Cookies To Responses

Puede adjuntar una cookie a una instancia saliente `Illuminate\Http\Response` utilizando el método `cookie`. Debe pasar el nombre, el valor y la cantidad de minutos que la cookie debe considerarse válida para este método:
> > You may attach a cookie to an outgoing `Illuminate\Http\Response` instance using the `cookie` method. You should pass the name, value, and number of minutes the cookie should be considered valid to this method:

    return response('Hello World')->cookie(
        'name', 'value', $minutes
    );

El método `cookie` también acepta algunos argumentos más que se utilizan con menos frecuencia. En general, estos argumentos tienen el mismo propósito y significado que los argumentos que se les daría al método nativo de PHP [setcookie](https://secure.php.net/manual/en/function.setcookie.php):
> > The `cookie` method also accepts a few more arguments which are used less frequently. Generally, these arguments have the same purpose and meaning as the arguments that would be given to PHP's native [setcookie](https://secure.php.net/manual/en/function.setcookie.php) method:

    return response('Hello World')->cookie(
        'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
    );

Alternativamente, puede usar la fachada `Cookie` para "poner en cola" las cookies para adjuntarlas a la respuesta saliente de su aplicación. El método `queue` acepta una instancia `Cookie` o los argumentos necesarios para crear una instancia `Cookie`. Estas cookies se adjuntarán a la respuesta de salida antes de enviarse al navegador:
> > Alternatively, you can use the `Cookie` facade to "queue" cookies for attachment to the outgoing response from your application. The `queue` method accepts a `Cookie` instance or the arguments needed to create a `Cookie` instance. These cookies will be attached to the outgoing response before it is sent to the browser:

    Cookie::queue(Cookie::make('name', 'value', $minutes));

    Cookie::queue('name', 'value', $minutes);

#### Generación de instancias de cookies : Generating Cookie Instances

Si desea generar una instancia `Symfony\Component\HttpFoundation\Cookie` que se le pueda dar a una instancia de respuesta más adelante, puede usar el helper global `cookie`. Esta cookie no se enviará de vuelta al cliente a menos que esté adjuntada a una instancia de respuesta:
> > If you would like to generate a `Symfony\Component\HttpFoundation\Cookie` instance that can be given to a response instance at a later time, you may use the global `cookie` helper. This cookie will not be sent back to the client unless it is attached to a response instance:

    $cookie = cookie('name', 'value', $minutes);

    return response('Hello World')->cookie($cookie);

<a name="files"></a>
## Ficheros : Files

<a name="retrieving-uploaded-files"></a>
### Recuperación de archivos cargados : Retrieving Uploaded Files

Puede acceder a los archivos cargados desde una instancia `Illuminate\Http\Request` utilizando el método `file` o usando propiedades dinámicas. El método `file` devuelve una instancia de la clase `Illuminate\Http\UploadedFile`, que amplía la clase PHP `SplFileInfo` y proporciona una variedad de métodos para interactuar con el archivo:
> > You may access uploaded files from a `Illuminate\Http\Request` instance using the `file` method or using dynamic properties. The `file` method returns an instance of the `Illuminate\Http\UploadedFile` class, which extends the PHP `SplFileInfo` class and provides a variety of methods for interacting with the file:

    $file = $request->file('photo');

    $file = $request->photo;

Puede determinar si un archivo está presente en la solicitud utilizando el método `hasFile`:
> > You may determine if a file is present on the request using the `hasFile` method:

    if ($request->hasFile('photo')) {
        //
    }

#### Validar cargas exitosas : Validating Successful Uploads

Además de verificar si el archivo está presente, puede verificar que no haya ningún problema para subir el archivo a través del método `isValid`:
> > In addition to checking if the file is present, you may verify that there were no problems uploading the file via the `isValid` method:

    if ($request->file('photo')->isValid()) {
        //
    }

#### Extensiones y rutas de archivos : File Paths & Extensions

La clase `UploadedFile` también contiene métodos para acceder a la ruta completa del archivo y su extensión. El método `extension` intentará adivinar la extensión del archivo en función de su contenido. Esta extensión puede ser diferente de la extensión que fue proporcionada por el cliente:
> > The `UploadedFile` class also contains methods for accessing the file's fully-qualified path and its extension. The `extension` method will attempt to guess the file's extension based on its contents. This extension may be different from the extension that was supplied by the client:

    $path = $request->photo->path();

    $extension = $request->photo->extension();

#### Otros métodos de archivo : Other File Methods

Hay una variedad de otros métodos disponibles en las instancias `UploadedFile`. Consulte la [documentación de API para la clase](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html) para obtener más información sobre estos métodos.
> > There are a variety of other methods available on `UploadedFile` instances. Check out the [API documentation for the class](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html) for more information regarding these methods.

<a name="storing-uploaded-files"></a>
### Almacenamiento de archivos cargados : Storing Uploaded Files

Para almacenar un archivo cargado, generalmente usará uno de sus [filesystems](/docs/{{version}}/filesystem) configurados. La clase `UploadedFile` tiene un método `store` que moverá un archivo cargado a uno de sus discos, que puede ser una ubicación en su sistema de archivos local o incluso una ubicación de almacenamiento en la nube como Amazon S3.
> > To store an uploaded file, you will typically use one of your configured [filesystems](/docs/{{version}}/filesystem). The `UploadedFile` class has a `store` method which will move an uploaded file to one of your disks, which may be a location on your local filesystem or even a cloud storage location like Amazon S3.

El método `store` acepta la ruta donde el archivo debe almacenarse en relación con el directorio raíz configurado del sistema de archivos. Esta ruta no debe contener un nombre de archivo, ya que una ID única se generará automáticamente para servir como nombre de archivo.
> > The `store` method accepts the path where the file should be stored relative to the filesystem's configured root directory. This path should not contain a file name, since a unique ID will automatically be generated to serve as the file name.

El método `store` también acepta un segundo argumento opcional para el nombre del disco que se debe usar para almacenar el archivo. El método devolverá la ruta del archivo en relación con la raíz del disco:
> > The `store` method also accepts an optional second argument for the name of the disk that should be used to store the file. The method will return the path of the file relative to the disk's root:

    $path = $request->photo->store('images');

    $path = $request->photo->store('images', 's3');

Si no desea que se genere automáticamente un nombre de archivo, puede usar el método `storeAs`, que acepta la ruta, el nombre del archivo y el nombre del disco como sus argumentos:
> > If you do not want a file name to be automatically generated, you may use the `storeAs` method, which accepts the path, file name, and disk name as its arguments:

    $path = $request->photo->storeAs('images', 'filename.jpg');

    $path = $request->photo->storeAs('images', 'filename.jpg', 's3');

<a name="configuring-trusted-proxies"></a>
## Configuración de servidores proxy de confianza : Configuring Trusted Proxies

Cuando ejecuta sus aplicaciones detrás de un equilibrador de carga que finaliza los certificados TLS / SSL, puede observar que a veces su aplicación no genera enlaces HTTPS. Normalmente, esto se debe a que su aplicación está siendo reenviada al tráfico de su equilibrador de carga en el puerto 80 y no sabe que debería generar enlaces seguros.
> > When running your applications behind a load balancer that terminates TLS / SSL certificates, you may notice your application sometimes does not generate HTTPS links. Typically this is because your application is being forwarded traffic from your load balancer on port 80 and does not know it should generate secure links.

Para solucionar esto, puede usar el middleware `App\Http\Middleware\TrustProxies` que se incluye en su aplicación Laravel, que le permite personalizar rápidamente los equilibradores de carga o los proxies que su aplicación debe confiar. Sus proxies de confianza deberían aparecer como un array en la propiedad `$proxies` de este middleware. Además de configurar los proxies confiables, puede configurar los proxy `$headers` que deberían ser de confianza:
> > To solve this, you may use the `App\Http\Middleware\TrustProxies` middleware that is included in your Laravel application, which allows you to quickly customize the load balancers or proxies that should be trusted by your application. Your trusted proxies should be listed as an array on the `$proxies` property of this middleware. In addition to configuring the trusted proxies, you may configure the proxy `$headers` that should be trusted:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Http\Request;
    use Fideloper\Proxy\TrustProxies as Middleware;

    class TrustProxies extends Middleware
    {
        /**
         * The trusted proxies for this application.
         *
         * @var array
         */
        protected $proxies = [
            '192.168.1.1',
            '192.168.1.2',
        ];

        /**
         * The headers that should be used to detect proxies.
         *
         * @var string
         */
        protected $headers = Request::HEADER_X_FORWARDED_ALL;
    }

> {tip} Si usa AWS Elastic Load Balancing, su valor `$headers` debe ser `Request::HEADER_X_FORWARDED_AWS_ELB`. Para obtener más información sobre las constantes que se pueden usar en la propiedad `$headers`, consulte la documentación de Symfony en [proxies confiables](http://symfony.com/doc/current/deployment/proxies.html).
> > > {tip} If you are using AWS Elastic Load Balancing, your `$headers` value should be `Request::HEADER_X_FORWARDED_AWS_ELB`. For more information on the constants that may be used in the `$headers` property, check out Symfony's documentation on [trusting proxies](http://symfony.com/doc/current/deployment/proxies.html).

#### Confiando en todos los poderes : Trusting All Proxies

Si está utilizando Amazon AWS u otro proveedor de equilibrador de carga "en la nube", es posible que no conozca las direcciones IP de sus equilibradores reales. En este caso, puede usar `*` para confiar en todos los proxies:
> > If you are using Amazon AWS or another "cloud" load balancer provider, you may not know the IP addresses of your actual balancers. In this case, you may use `*` to trust all proxies:

    /**
     * The trusted proxies for this application.
     *
     * @var array
     */
    protected $proxies = '*';
