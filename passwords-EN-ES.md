# Restablecimiento de contraseñas : Resetting Passwords

- [Introduction](#introduction)
- [Database Considerations](#resetting-database)
- [Routing](#resetting-routing)
- [Views](#resetting-views)
- [After Resetting Passwords](#after-resetting-passwords)
- [Customization](#password-customization)

<a name="introduction"></a>
## Introducción : Introduction

> {tip} **¿Desea comenzar rápido?** Simplemente ejecute `php artisan make:auth` en una nueva aplicación Laravel y navegue en su navegador a `http://your-app.test/register` o cualquier otra URL eso está asignado a tu aplicación. ¡Este comando único se encargará de andamiar todo su sistema de autenticación, incluido el restablecimiento de contraseñas!
> > > {tip} **Want to get started fast?** Just run `php artisan make:auth` in a fresh Laravel application and navigate your browser to `http://your-app.test/register` or any other URL that is assigned to your application. This single command will take care of scaffolding your entire authentication system, including resetting passwords!

La mayoría de las aplicaciones web proporcionan una forma para que los usuarios restablezcan sus contraseñas olvidadas. En lugar de forzarlo a volver a implementar esto en cada aplicación, Laravel proporciona métodos convenientes para enviar recordatorios de contraseñas y realizar restablecimientos de contraseñas.
> > Most web applications provide a way for users to reset their forgotten passwords. Rather than forcing you to re-implement this on each application, Laravel provides convenient methods for sending password reminders and performing password resets.

> {note} Antes de usar las funciones de restablecimiento de contraseña de Laravel, su usuario debe usar el atributo `Illuminate\Notifications\Notifiable`.
> > > {note} Before using the password reset features of Laravel, your user must use the `Illuminate\Notifications\Notifiable` trait.

<a name="resetting-database"></a>
## Consideraciones sobre la base de datos : Database Considerations

Para comenzar, verifique que su modelo `App\User` implemente el contrato `Illuminate\Contracts\Auth\CanResetPassword`. Por supuesto, el modelo `App\User` incluido con el framework ya implementa esta interfaz, y usa el rasgo `Illuminate\Auth\Passwords\CanResetPassword` para incluir los métodos necesarios para implementar la interfaz.
> > To get started, verify that your `App\User` model implements the `Illuminate\Contracts\Auth\CanResetPassword` contract. Of course, the `App\User` model included with the framework already implements this interface, and uses the `Illuminate\Auth\Passwords\CanResetPassword` trait to include the methods needed to implement the interface.

#### Generación de la migración de tabla de tokens de reinicio : Generating The Reset Token Table Migration

A continuación, se debe crear una tabla para almacenar los tokens de restablecimiento de contraseña. La migración para esta tabla se incluye con Laravel lista para usar, y reside en el directorio `database/migrations`. Entonces, todo lo que necesita hacer es ejecutar las migraciones de su base de datos:
> > Next, a table must be created to store the password reset tokens. The migration for this table is included with Laravel out of the box, and resides in the `database/migrations` directory. So, all you need to do is run your database migrations:

    php artisan migrate

<a name="resetting-routing"></a>
## Enrutamiento : Routing

Laravel incluye las clases `Auth\ForgotPasswordController` y `Auth\ResetPasswordController` que contienen la lógica necesaria para enviar por correo electrónico los enlaces de restablecimiento de contraseña y restablecer las contraseñas de los usuarios. Todas las rutas necesarias para realizar restablecimientos de contraseñas pueden generarse utilizando el comando Artisan `make:auth`:
> > Laravel includes `Auth\ForgotPasswordController` and `Auth\ResetPasswordController` classes that contains the logic necessary to e-mail password reset links and reset user passwords. All of the routes needed to perform password resets may be generated using the `make:auth` Artisan command:

    php artisan make:auth

<a name="resetting-views"></a>
## Vistas : Views

De nuevo, Laravel generará todas las vistas necesarias para restablecer la contraseña cuando se ejecute el comando `make:auth`. Estas vistas se colocan en `resources/views/auth/passwords`. Usted es libre de personalizarlos según sea necesario para su aplicación.
> > Again, Laravel will generate all of the necessary views for password reset when the `make:auth` command is executed. These views are placed in `resources/views/auth/passwords`. You are free to customize them as needed for your application.

<a name="after-resetting-passwords"></a>
## Después de restablecer contraseñas : After Resetting Passwords

Una vez que haya definido las rutas y las vistas para restablecer las contraseñas de sus usuarios, puede acceder a la ruta en su navegador en `/password/reset`. El `ForgotPasswordController` incluido con el marco ya incluye la lógica para enviar los correos electrónicos de enlace de restablecimiento de contraseña, mientras que `ResetPasswordController` incluye la lógica para restablecer las contraseñas de los usuarios.
> > Once you have defined the routes and views to reset your user's passwords, you may access the route in your browser at `/password/reset`. The `ForgotPasswordController` included with the framework already includes the logic to send the password reset link e-mails, while the `ResetPasswordController` includes the logic to reset user passwords.

Después de restablecer una contraseña, el usuario iniciará sesión automáticamente en la aplicación y se redirigirá a `/home`. Puede personalizar la ubicación de redirección de restablecimiento de contraseña después de definir una propiedad `redirectTo` en `ResetPasswordController`:
> > After a password is reset, the user will automatically be logged into the application and redirected to `/home`. You can customize the post password reset redirect location by defining a `redirectTo` property on the `ResetPasswordController`:

    protected $redirectTo = '/dashboard';

> {note} De manera predeterminada, los tokens de restablecimiento de contraseña caducan después de una hora. Puede cambiar esto a través de la opción de restablecimiento de contraseña `expire` en su archivo `config/auth.php`.
> > > {note} By default, password reset tokens expire after one hour. You may change this via the password reset `expire` option in your `config/auth.php` file.

<a name="password-customization"></a>
## Personalización : Customization

#### Authentication Guard Customization

En su archivo de configuración `auth.php`, puede configurar múltiples "guardias", que pueden usarse para definir el comportamiento de autenticación para múltiples tablas de usuario. Puede personalizar el `ResetPasswordController` incluido para usar el protector de su elección anulando el método `guard` en el controlador. Este método debe devolver una instancia de guardia:
> > In your `auth.php` configuration file, you may configure multiple "guards", which may be used to define authentication behavior for multiple user tables. You can customize the included `ResetPasswordController` to use the guard of your choice by overriding the `guard` method on the controller. This method should return a guard instance:

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### Personalización de Password Broker : Password Broker Customization

En su archivo de configuración `auth.php`, puede configurar varios "intermediarios" de contraseñas, que pueden usarse para restablecer contraseñas en varias tablas de usuarios. Puede personalizar el `ForgotPasswordController` y el `ResetPasswordController` incluidos para usar el intermediario de su elección anulando el método `broker`:
> > In your `auth.php` configuration file, you may configure multiple password "brokers", which may be used to reset passwords on multiple user tables. You can customize the included `ForgotPasswordController` and `ResetPasswordController` to use the broker of your choice by overriding the `broker` method:

    use Illuminate\Support\Facades\Password;

    /**
     * Get the broker to be used during password reset.
     *
     * @return PasswordBroker
     */
    protected function broker()
    {
        return Password::broker('name');
    }

#### Restablecer la personalización del correo electrónico : Reset Email Customization

Puede modificar fácilmente la clase de notificación utilizada para enviar el enlace de restablecimiento de contraseña al usuario. Para comenzar, anule el método `sendPasswordResetNotification` en su modelo` User`. Dentro de este método, puede enviar la notificación usando cualquier clase de notificación que elija. La contraseña restablecida `$token` es el primer argumento recibido por el método:
> > You may easily modify the notification class used to send the password reset link to the user. To get started, override the `sendPasswordResetNotification` method on your `User` model. Within this method, you may send the notification using any notification class you choose. The password reset `$token` is the first argument received by the method:

    /**
     * Send the password reset notification.
     *
     * @param  string  $token
     * @return void
     */
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new ResetPasswordNotification($token));
    }

