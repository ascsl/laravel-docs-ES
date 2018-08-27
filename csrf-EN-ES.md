# Protección CSRF : CSRF Protection

- [Introduction](#csrf-introduction)
- [Excluding URIs](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## Introducción : Introduction

Laravel facilita la protección de su aplicación contra ataques de [cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF). Las falsificaciones de solicitudes entre sitios son un tipo de ataque malicioso por el cual se realizan comandos no autorizados en nombre de un usuario autenticado.
> > Laravel makes it easy to protect your application from [cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) attacks. Cross-site request forgeries are a type of malicious exploit whereby unauthorized commands are performed on behalf of an authenticated user.

Laravel genera automáticamente un "token" de CSRF para cada sesión de usuario activa administrada por la aplicación. Este token se utiliza para verificar que el usuario autenticado es el que hace las solicitudes a la aplicación.
> > Laravel automatically generates a CSRF "token" for each active user session managed by the application. This token is used to verify that the authenticated user is the one actually making the requests to the application.

Cada vez que defina un formulario HTML en su aplicación, debe incluir un campo de token CSRF oculto en el formulario para que el middleware de protección CSRF pueda validar la solicitud. Puede usar la directiva Blade `@csrf` para generar el campo de token:
> > Anytime you define a HTML form in your application, you should include a hidden CSRF token field in the form so that the CSRF protection middleware can validate the request. You may use the `@csrf` Blade directive to generate the token field:

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

El `VerifyCsrfToken` [middleware](/docs/{{version}}/middleware), que se incluye en el grupo de middleware `web`, verificará automáticamente que el token en la entrada de solicitud coincida con el token almacenado en la sesión.
> > The `VerifyCsrfToken` [middleware](/docs/{{version}}/middleware), which is included in the `web` middleware group, will automatically verify that the token in the request input matches the token stored in the session.

#### Tokens CSRF y JavaScript : CSRF Tokens & JavaScript

Al crear aplicaciones controladas por JavaScript, es conveniente que su biblioteca HTTP de JavaScript adjunte automáticamente el token CSRF a cada solicitud saliente. De forma predeterminada, el archivo `resources/assets/js/bootstrap.js` registra el valor de la etiqueta meta `csrf-token` con la biblioteca HTTP de Axios. Si no está utilizando esta biblioteca, tendrá que configurar manualmente este comportamiento para su aplicación.
> > When building JavaScript driven applications, it is convenient to have your JavaScript HTTP library automatically attach the CSRF token to every outgoing request. By default, the `resources/assets/js/bootstrap.js` file registers the value of the `csrf-token` meta tag with the Axios HTTP library. If you are not using this library, you will need to manually configure this behavior for your application.

<a name="csrf-excluding-uris"></a>
## Excluyendo las URIs de la protección CSRF : Excluding URIs From CSRF Protection

En ocasiones, es posible que desee excluir un conjunto de URI de la protección CSRF. Por ejemplo, si está utilizando [Stripe] (https://stripe.com) para procesar pagos y está utilizando su sistema webhook, deberá excluir su ruta del manejador webhook Stripe de la protección CSRF ya que Stripe no sabrá qué token CSRF para enviar a tus rutas
> > Sometimes you may wish to exclude a set of URIs from CSRF protection. For example, if you are using [Stripe](https://stripe.com) to process payments and are utilizing their webhook system, you will need to exclude your Stripe webhook handler route from CSRF protection since Stripe will not know what CSRF token to send to your routes.

Normalmente, debe colocar este tipo de rutas fuera del grupo de middleware `web` que el `RouteServiceProvider` se aplica a todas las rutas en el archivo `routes/web.php`. Sin embargo, también puede excluir las rutas agregando sus URIs a la propiedad `$except` del middleware `VerifyCsrfToken`:
> > Typically, you should place these kinds of routes outside of the `web` middleware group that the `RouteServiceProvider` applies to all routes in the `routes/web.php` file. However, you may also exclude the routes by adding their URIs to the `$except` property of the `VerifyCsrfToken` middleware:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

    class VerifyCsrfToken extends Middleware
    {
        /**
         * The URIs that should be excluded from CSRF verification.
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
            'http://example.com/foo/bar',
            'http://example.com/foo/*',
        ];
    }

> {tip} El middleware CSRF se desactiva automáticamente cuando [ejecuta pruebas](/docs/{{version}}/testing).
> > > {tip} The CSRF middleware is automatically disabled when [running tests](/docs/{{version}}/testing).

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

Además de verificar el token CSRF como un parámetro POST, el middleware `VerifyCsrfToken` también buscará el encabezado de solicitud `X-CSRF-TOKEN`. Podría, por ejemplo, almacenar el token en una etiqueta HTML `meta`:
> > In addition to checking for the CSRF token as a POST parameter, the `VerifyCsrfToken` middleware will also check for the `X-CSRF-TOKEN` request header. You could, for example, store the token in a HTML `meta` tag:

    <meta name="csrf-token" content="{{ csrf_token() }}">

Luego, una vez que haya creado la etiqueta `meta`, puede indicar a una biblioteca como jQuery que agregue automáticamente el token a todos los encabezados de solicitud. Esto proporciona una protección CSRF simple y conveniente para sus aplicaciones basadas en AJAX:
> > Then, once you have created the `meta` tag, you can instruct a library like jQuery to automatically add the token to all request headers. This provides simple, convenient CSRF protection for your AJAX based applications:

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });

> {tip} Por defecto, el archivo `resources/assets/js/bootstrap.js` registra el valor de la metaetiqueta `csrf-token` con la biblioteca HTTP de Axios. Si no está utilizando esta biblioteca, tendrá que configurar este comportamiento para su aplicación.
> > > {tip} By default, the `resources/assets/js/bootstrap.js` file registers the value of the `csrf-token` meta tag with the Axios HTTP library. If you are not using this library, you will need to manually configure this behavior for your application.

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel almacena el token CSRF actual en una cookie `XSRF-TOKEN` que se incluye con cada respuesta generada por el marco. Puede usar el valor de cookie para establecer el encabezado de solicitud `X-XSRF-TOKEN`.
> > Laravel stores the current CSRF token in a `XSRF-TOKEN` cookie that is included with each response generated by the framework. You can use the cookie value to set the `X-XSRF-TOKEN` request header.

Esta cookie se envía principalmente para su comodidad, ya que algunos marcos y bibliotecas JavaScript, como Angular y Axios, colocan automáticamente su valor en el encabezado `X-XSRF-TOKEN`.
> > This cookie is primarily sent as a convenience since some JavaScript frameworks and libraries, like Angular and Axios, automatically place its value in the `X-XSRF-TOKEN` header.
