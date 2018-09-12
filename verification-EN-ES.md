# Verificacion de email : Email Verification

- [Introduction](#introduction)
- [Database Considerations](#verification-database)
- [Routing](#verification-routing)
    - [Protecting Routes](#protecting-routes)
- [Views](#verification-views)
- [After Verifying Emails](#after-verifying-emails)

<a name="introduction"></a>
## Introducción : Introduction

Muchas aplicaciones web requieren que los usuarios verifiquen sus direcciones de correo electrónico antes de usar la aplicación. En lugar de forzarlo a volver a implementar esto en cada aplicación, Laravel proporciona métodos convenientes para enviar y verificar solicitudes de verificación de correo electrónico.
> > Many web applications requires users to verify their email addresses before using the application. Rather than forcing you to re-implement this on each application, Laravel provides convenient methods for sending and verifying email verification requests.

<a name="verification-database"></a>
## Consideraciones sobre la base de datos : Database Considerations

> {note} Para comenzar, verifique que su modelo `App\User` implemente el contrato `Illuminate\Contracts\Auth\MustVerifyEmail`.
> > > {note} To get started, verify that your `App\User` model implements the `Illuminate\Contracts\Auth\MustVerifyEmail` contract.

#### La columna de verificación de correo electrónico : The Email Verification Column

A continuación, su tabla `user` debe contener una columna `email_verified_at` para almacenar la fecha y la hora en que se verificó la dirección de correo electrónico. De forma predeterminada, la migración de la tabla `users` incluida con el marco de Laravel ya incluye esta columna. Entonces, todo lo que necesita hacer es ejecutar las migraciones de su base de datos:
> > Next, your `user` table must contain an `email_verified_at` column to store the date and time that the email address was verified. By default, the `users` table migration included with the Laravel framework already includes this column. So, all you need to do is run your database migrations:

    php artisan migrate

<a name="verification-routing"></a>
## Enrutamiento : Routing

Laravel incluye la clase `Auth\VerificationController` que contiene la lógica necesaria para enviar enlaces de verificación y verificar correos electrónicos. Para registrar las rutas necesarias para este controlador, pase la opción `verify` al método `Auth::routes`:
> > Laravel includes the `Auth\VerificationController` class that contains the necessary logic to send verification links and verify emails. To register the necessary routes for this controller, pass the `verify` option to the `Auth::routes` method:

    Auth::routes(['verify' => true]);

<a name="protecting-routes"></a>
### Protección de rutas : Protecting Routes

[El middleware de ruta](/docs/{{version}}/middleware) se puede usar para permitir que usuarios verificados accedan a una ruta determinada. Laravel se envía con un middleware `verified`, que se define en `Illuminate\Auth\Middleware\EnsureEmailIsVerified`. Como este middleware ya está registrado en el kernel HTTP de su aplicación, todo lo que necesita hacer es adjuntar el middleware a una definición de ruta:
> > [Route middleware](/docs/{{version}}/middleware) can be used to only allow verified users to access a given route. Laravel ships with a `verified` middleware, which is defined at `Illuminate\Auth\Middleware\EnsureEmailIsVerified`. Since this middleware is already registered in your application's HTTP kernel, all you need to do is attach the middleware to a route definition:

    Route::get('profile', function () {
        // Only verified users may enter...
    })->middleware('verified');

<a name="verification-views"></a>
## Vistas : Views

Laravel generará toda la vista necesaria de verificación de correo electrónico cuando se ejecute el comando `make:auth`. Esta vista se coloca en `resources/views/auth/verify.blade.php`. Usted es libre de personalizar esta vista según sea necesario para su aplicación.
> > Laravel will generate all of the necessary email verification view when the `make:auth` command is executed. This views is placed in `resources/views/auth/verify.blade.php`. You are free to customize this view as needed for your application.

<a name="after-verifying-emails"></a>
## Después de verificar los correos electrónicos : After Verifying Emails

Después de que se verifica una dirección de correo electrónico, el usuario será redirigido automáticamente a `/home`. Puede personalizar la ubicación de redirección de verificación posterior definiendo un método o propiedad `redirectTo` en `VerificationController`:
> > After an email address is verified, the user will automatically be redirected to `/home`. You can customize the post verification redirect location by defining a `redirectTo` method or property on the `VerificationController`:

    protected $redirectTo = '/dashboard';
