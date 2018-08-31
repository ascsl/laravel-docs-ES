# Manejo de errores : Error Handling

- [Introduction](#introduction)
- [Configuration](#configuration)
- [The Exception Handler](#the-exception-handler)
    - [Report Method](#report-method)
    - [Render Method](#render-method)
    - [Reportable & Renderable Exceptions](#renderable-exceptions)
- [HTTP Exceptions](#http-exceptions)
    - [Custom HTTP Error Pages](#custom-http-error-pages)

<a name="introduction"></a>
## Introduction : Introducción

Cuando comienza un nuevo proyecto de Laravel, el manejo de errores y excepciones ya está configurado para usted. La clase `App\Exceptions\Handler` es donde todas las excepciones activadas por la aplicación se registran y luego se vuelven a representar para el usuario. Nos adentraremos más en esta clase a lo largo de esta documentación.
> > When you start a new Laravel project, error and exception handling is already configured for you. The `App\Exceptions\Handler` class is where all exceptions triggered by your application are logged and then rendered back to the user. We'll dive deeper into this class throughout this documentation.

<a name="configuration"></a>
## Configuración : Configuration

La opción `debug` en su archivo de configuración `config/app.php` determina cuánta información sobre un error se muestra realmente al usuario. Por defecto, esta opción está configurada para respetar el valor de la variable de entorno `APP_DEBUG`, que está almacenada en su archivo `.env`.
> > The `debug` option in your `config/app.php` configuration file determines how much information about an error is actually displayed to the user. By default, this option is set to respect the value of the `APP_DEBUG` environment variable, which is stored in your `.env` file.

Para el desarrollo local, debe establecer la variable de entorno `APP_DEBUG` en `true`. En su entorno de producción, este valor siempre debe ser `false`. Si el valor se establece en `true` en producción, corre el riesgo de exponer valores de configuración confidenciales a los usuarios finales de la aplicación.
> > For local development, you should set the `APP_DEBUG` environment variable to `true`. In your production environment, this value should always be `false`. If the value is set to `true` in production, you risk exposing sensitive configuration values to your application's end users.

<a name="the-exception-handler"></a>
## El controlador de excepciones : The Exception Handler

<a name="report-method"></a>
### El método de informe : The Report Method

Todas las excepciones son manejadas por la clase `App\Exceptions\Handler`. Esta clase contiene dos métodos: `report` y` render`. Examinaremos cada uno de estos métodos en detalle. El método `report` se usa para registrar excepciones o enviarlas a un servicio externo como [Bugsnag](https://bugsnag.com) o [Sentry](https://github.com/getsentry/sentry-laravel). Por defecto, el método `report` pasa la excepción a la clase base donde se registra la excepción. Sin embargo, puede registrar excepciones como lo desee.
> > All exceptions are handled by the `App\Exceptions\Handler` class. This class contains two methods: `report` and `render`. We'll examine each of these methods in detail. The `report` method is used to log exceptions or send them to an external service like [Bugsnag](https://bugsnag.com) or [Sentry](https://github.com/getsentry/sentry-laravel). By default, the `report` method passes the exception to the base class where the exception is logged. However, you are free to log exceptions however you wish.

Por ejemplo, si necesita informar diferentes tipos de excepciones de diferentes maneras, puede usar el operador de comparación `instanceof` de PHP:
> > For example, if you need to report different types of exceptions in different ways, you may use the PHP `instanceof` comparison operator:

    /**
     * Report or log an exception.
     *
     * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
     *
     * @param  \Exception  $exception
     * @return void
     */
    public function report(Exception $exception)
    {
        if ($exception instanceof CustomException) {
            //
        }

        return parent::report($exception);
    }

> {tip} En lugar de realizar una gran cantidad de verificaciones de `instanceof` en su método `report`, considere usar [excepciones reportables](/docs/{{version}}/errors#renderable-exceptions)
> > > {tip} Instead of making a lot of `instanceof` checks in your `report` method, consider using [reportable exceptions](/docs/{{version}}/errors#renderable-exceptions)

#### The `report` Helper
#### El helper `report`

A veces puede necesitar informar una excepción pero continuar manejando la solicitud actual. La función de ayuda `report` le permite reportar rápidamente una excepción usando el método `report` de su manejador de excepciones sin presentar una página de error:
> > Sometimes you may need to report an exception but continue handling the current request. The `report` helper function allows you to quickly report an exception using your exception handler's `report` method without rendering an error page:

    public function isValid($value)
    {
        try {
            // Validate the value...
        } catch (Exception $e) {
            report($e);

            return false;
        }
    }

#### Ignorar excepciones por tipo : Ignoring Exceptions By Type

La propiedad `$dontReport` del manejador de excepciones contiene un array de tipos de excepciones que no se registrarán. Por ejemplo, las excepciones que resultan de los errores 404, así como de otros muchos tipos de errores, no se escriben en sus archivos de registro. Puede agregar otros tipos de excepciones a este array según sea necesario:
> > The `$dontReport` property of the exception handler contains an array of exception types that will not be logged. For example, exceptions resulting from 404 errors, as well as several other types of errors, are not written to your log files. You may add other exception types to this array as needed:

    /**
     * A list of the exception types that should not be reported.
     *
     * @var array
     */
    protected $dontReport = [
        \Illuminate\Auth\AuthenticationException::class,
        \Illuminate\Auth\Access\AuthorizationException::class,
        \Symfony\Component\HttpKernel\Exception\HttpException::class,
        \Illuminate\Database\Eloquent\ModelNotFoundException::class,
        \Illuminate\Validation\ValidationException::class,
    ];

<a name="render-method"></a>
### El método Render : The Render Method

El método `render` es responsable de convertir una excepción dada en una respuesta HTTP que debe enviarse al navegador. Por defecto, la excepción se pasa a la clase base que genera una respuesta para usted. Sin embargo, puede verificar el tipo de excepción o devolver su propia respuesta personalizada:
> > The `render` method is responsible for converting a given exception into an HTTP response that should be sent back to the browser. By default, the exception is passed to the base class which generates a response for you. However, you are free to check the exception type or return your own custom response:

    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof CustomException) {
            return response()->view('errors.custom', [], 500);
        }

        return parent::render($request, $exception);
    }

<a name="renderable-exceptions"></a>
### Excepciones reportables y revocables : Reportable & Renderable Exceptions

En lugar de las excepciones de verificación de tipos en los métodos `report` y `render` del manejador de excepciones, puede definir los métodos `report` y `render` directamente en su excepción personalizada. Cuando existen estos métodos, el marco los llamará automáticamente:
> > Instead of type-checking exceptions in the exception handler's `report` and `render` methods, you may define `report` and `render` methods directly on your custom exception. When these methods exist, they will be called automatically by the framework:

    <?php

    namespace App\Exceptions;

    use Exception;

    class RenderException extends Exception
    {
        /**
         * Report the exception.
         *
         * @return void
         */
        public function report()
        {
            //
        }

        /**
         * Render the exception into an HTTP response.
         *
         * @param  \Illuminate\Http\Request
         * @return \Illuminate\Http\Response
         */
        public function render($request)
        {
            return response(...);
        }
    }

<a name="http-exceptions"></a>
## Excepciones HTTP : HTTP Exceptions

Algunas excepciones describen códigos de error HTTP del servidor. Por ejemplo, esto puede ser un error de "página no encontrada" (404), un "error no autorizado" (401) o incluso un error 500 generado por el desarrollador. Para generar dicha respuesta desde cualquier lugar de su aplicación, puede usar el helper `abort`:
> > Some exceptions describe HTTP error codes from the server. For example, this may be a "page not found" error (404), an "unauthorized error" (401) or even a developer generated 500 error. In order to generate such a response from anywhere in your application, you may use the `abort` helper:

    abort(404);

El helper `abort` generará inmediatamente una excepción que será procesada por el manejador de excepciones. Opcionalmente, puede proporcionar el texto de respuesta:
> > The `abort` helper will immediately raise an exception which will be rendered by the exception handler. Optionally, you may provide the response text:

    abort(403, 'Unauthorized action.');

<a name="custom-http-error-pages"></a>
### Páginas de error HTTP personalizadas : Custom HTTP Error Pages

Laravel facilita la visualización de páginas de error personalizadas para varios códigos de estado HTTP. Por ejemplo, si desea personalizar la página de error para códigos de estado HTTP 404, cree un `resources/views/errors/404.blade.php`. Este archivo se publicará en todos los errores 404 generados por su aplicación. Las vistas dentro de este directorio deben nombrarse para que coincidan con el código de estado HTTP al que corresponden. La instancia `HttpException` generada por la función `abort` se pasará a la vista como una variable `$exception`:
> > Laravel makes it easy to display custom error pages for various HTTP status codes. For example, if you wish to customize the error page for 404 HTTP status codes, create a `resources/views/errors/404.blade.php`. This file will be served on all 404 errors generated by your application. The views within this directory should be named to match the HTTP status code they correspond to. The `HttpException` instance raised by the `abort` function will be passed to the view as an `$exception` variable:

    <h2>{{ $exception->getMessage() }}</h2>
