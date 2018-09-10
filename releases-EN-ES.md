# Notas de lanzamiento : Release Notes

- [Versioning Scheme](#versioning-scheme)
- [Support Policy](#support-policy)
- [Laravel 5.7](#laravel-5.7)

<a name="versioning-scheme"></a>
## Esquema de control de versiones : Versioning Scheme

El esquema de control de versiones de Laravel mantiene la siguiente convención: `paradigm.major.minor`. Los lanzamientos principales del marco se lanzan cada seis meses (febrero y agosto), mientras que los lanzamientos de menor importancia se pueden lanzar tan a menudo como cada semana. Las versiones menores **nunca** deben contener cambios de última hora.
> > Laravel's versioning scheme maintains the following convention: `paradigm.major.minor`. Major framework releases are released every six months (February and August), while minor releases may be released as often as every week. Minor releases should **never** contain breaking changes.

Al hacer referencia al marco de Laravel o sus componentes desde su aplicación o paquete, siempre debe usar una restricción de versión como `5.7.*`, Ya que las versiones principales de Laravel incluyen cambios de última hora. Sin embargo, nos esforzamos por garantizar siempre que pueda actualizar a una nueva versión principal en un día o menos.
> > When referencing the Laravel framework or its components from your application or package, you should always use a version constraint such as `5.7.*`, since major releases of Laravel do include breaking changes. However, we strive to always ensure you may update to a new major release in one day or less.

Las versiones cambiantes de Paradigm están separadas por muchos años y representan cambios fundamentales en la arquitectura y las convenciones del marco. Actualmente, no hay un lanzamiento de cambio de paradigma en desarrollo.
> > Paradigm shifting releases are separated by many years and represent fundamental shifts in the framework's architecture and conventions. Currently, there is no paradigm shifting release under development.

<a name="support-policy"></a>
## Política de soporte : Support Policy

Para las versiones de LTS, como Laravel 5.5, se proporcionan correcciones de errores durante 2 años y se proporcionan soluciones de seguridad durante 3 años. Estas versiones proporcionan la ventana más larga de soporte y mantenimiento. Para las versiones generales, las correcciones de errores se proporcionan durante 6 meses y las correcciones de seguridad se proporcionan durante 1 año.
> > For LTS releases, such as Laravel 5.5, bug fixes are provided for 2 years and security fixes are provided for 3 years. These releases provide the longest window of support and maintenance. For general releases, bug fixes are provided for 6 months and security fixes are provided for 1 year.

| Version | Release | Bug Fixes Until | Security Fixes Until |
| --- | --- | --- | --- |
| 5.0 | February 4th, 2015 | August 4th, 2015 | February 4th, 2016 |
| 5.1 (LTS) | June 9th, 2015 | June 9th, 2017 | June 9th, 2018 |
| 5.2 | December 21st, 2015 | June 21st, 2016 | December 21st, 2016 |
| 5.3 | August 23rd, 2016 | February 23rd, 2017 | August 23rd, 2017 |
| 5.4 | January 24th, 2017 | July 24th, 2017 | January 24th, 2018 |
| 5.5 (LTS) | August 30th, 2017 | August 30th, 2019 | August 30th, 2020 |
| 5.6 | February 7th, 2018 | August 7th, 2018 | February 7th, 2019 |
| 5.7 | August 2018 | February 2019 | August 2019 |

<a name="laravel-5.7"></a>
## Laravel 5.7

Laravel 5.7 continúa las mejoras realizadas en Laravel 5.6 al presentar [Laravel Nova](https://nova.laravel.com), verificación de correo electrónico opcional para el andamiaje de autenticación, soporte para usuarios invitados en puertas de autorización y políticas, integración de Symfony `dump-server`, notificaciones localizables y una variedad de otras correcciones de errores y mejoras de usabilidad.
> > Laravel 5.7 continues the improvements made in Laravel 5.6 by introducing [Laravel Nova](https://nova.laravel.com), optional email verification to the authentication scaffolding, support for guest users in authorization gates and policies, Symfony `dump-server` integration, localizable notifications, and a variety of other bug fixes and usability improvements.

### Laravel Nova

[Laravel Nova](https://nova.laravel.com) es un tablero de administración bonito, el mejor de su clase para aplicaciones Laravel. Por supuesto, la característica principal de Nova es la capacidad de administrar sus registros de bases de datos subyacentes utilizando Eloquent. Además, Nova ofrece soporte para filtros, lentes, acciones, acciones en cola, métricas, autorizaciones, herramientas personalizadas, tarjetas personalizadas, campos personalizados y más.
> > [Laravel Nova](https://nova.laravel.com) is a beautiful, best-in-class administration dashboard for Laravel applications. Of course, the primary feature of Nova is the ability to administer your underlying database records using Eloquent. Additionally, Nova offers support for filters, lenses, actions, queued actions, metrics, authorization, custom tools, custom cards, custom fields, and more.

Para obtener más información sobre Laravel Nova, consulte el [sitio web de Nova](https://nova.laravel.com).
> > To learn more about Laravel Nova, check out the [Nova website](https://nova.laravel.com).

### Verificacion de email : Email Verification

Laravel 5.7 introduce la verificación de correo electrónico opcional para el andamio de autenticación incluido con el framework. Para adaptarse a esta función, se ha agregado una columna de indicación de fecha y hora `email_verified_at` a la migración predeterminada de la tabla `users` que se incluye con el framework.
> > Laravel 5.7 introduces optional email verification to the authentication scaffolding included with the framework. To accommodate this feature, an `email_verified_at` timestamp column has been added to the default `users` table migration that is included with the framework.

Para solicitar a los usuarios recién registrados que verifiquen su correo electrónico, el modelo `User` debe estar marcado con la interfaz `MustVerifyEmail`:
> > To prompt newly registered users to verify their email, the `User` model should be marked with the `MustVerifyEmail` interface:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Contracts\Auth\MustVerifyEmail;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable implements MustVerifyEmail
    {
        // ...
    }

Una vez que el modelo `User` está marcado con la interfaz `MustVerifyEmail`, los usuarios recién registrados recibirán un correo electrónico que contiene un enlace de verificación firmado. Una vez que se haya hecho clic en este enlace, Laravel registrará automáticamente el tiempo de verificación en la base de datos y redirigirá a los usuarios a la ubicación que usted elija.
> > Once the `User` model is marked with the `MustVerifyEmail` interface, newly registered users will receive an email containing a signed verification link. Once this link has been clicked, Laravel will automatically record the verification time in the database and redirect users to a location of your choosing.

Se ha agregado un middleware `verified` al kernel HTTP de la aplicación predeterminada. Este middleware puede adjuntarse a rutas que solo deberían permitir usuarios verificados:
> > A `verified` middleware has been added to the default application's HTTP kernel. This middleware may be attached to routes that should only allow verified users:

    'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,

> {tip} Para obtener más información sobre la verificación de correo electrónico, consulte la [documentación completa](/docs/{{version}}/verification).
> > > {tip} To learn more about email verification, check out the [complete documentation](/docs/{{version}}/verification).

### Guest User Gates / Policies

En versiones anteriores de Laravel, las puertas de autorización y las políticas devolvían automáticamente `false` para los visitantes no autenticados de su aplicación. Sin embargo, ahora puede permitir que los invitados pasen las verificaciones de autorización declarando una sugerencia de tipo "opcional" o proporcionando un valor predeterminado `null` para la definición del argumento del usuario:
> > In previous versions of Laravel, authorization gates and policies automatically returned `false` for unauthenticated visitors to your application. However, you may now allow guests to pass through authorization checks by declaring an "optional" type-hint or supplying a `null` default value for the user argument definition:

    Gate::define('update-post', function (?User $user, Post $post) {
        // ...
    });

### Symfony Dump Server

Laravel 5.7 ofrece integración con el comando `dump-server` de Symfony a través de [un paquete de Marcel Pociot](https://github.com/beyondcode/laravel-dump-server). Para comenzar, ejecute el comando Artisan `dump-server`:
> > Laravel 5.7 offers integration with Symfony's `dump-server` command via [a package by Marcel Pociot](https://github.com/beyondcode/laravel-dump-server). To get started, run the `dump-server` Artisan command:

    php artisan dump-server

Una vez que el servidor haya comenzado, todas las llamadas a `volcar` se mostrarán en la ventana de la consola `dump-server` en lugar de en su navegador, lo que le permitirá inspeccionar los valores sin modificar la salida de respuesta HTTP.
> > Once the server has started, all calls to `dump` will by displayed in the `dump-server` console window instead of in your browser, allowing you to inspect the values without mangling your HTTP response output.

### Localización de notificaciones : Notification Localization

Laravel ahora le permite enviar notificaciones en una configuración regional que no sea el idioma actual, e incluso recordará esta configuración regional si la notificación está en cola.
> > Laravel now allows you to send notifications in a locale other than the current language, and will even remember this locale if the notification is queued.

Para lograr esto, la clase `Illuminate\Notifications\Notification` ahora ofrece un método `locale` para establecer el idioma deseado. La aplicación cambiará a esta configuración regional cuando se esté formateando la notificación y luego volverá a la configuración regional anterior cuando se complete el formateo:
> > To accomplish this, the `Illuminate\Notifications\Notification` class now offers a `locale` method to set the desired language. The application will change into this locale when the notification is being formatted and then revert back to the previous locale when formatting is complete:

    $user->notify((new InvoicePaid($invoice))->locale('es'));

La localización de múltiples entradas de notificación obligatoria también se puede lograr a través de la fachada de `Notification`:
> > Localization of multiple notifiable entries may also be achieved via the `Notification` facade:

    Notification::locale('es')->send($users, new InvoicePaid($invoice));

### URL Generator & Callable Syntax

En lugar de solo aceptar cadenas, el generador de URL de Laravel ahora acepta la sintaxis "invocable" cuando genera direcciones URL para las acciones del controlador:
> > Instead of only accepting strings, Laravel's URL generator now accepts "callable" syntax when generating URLs to controller actions:

    action([UserController::class, 'index']);

### Enlaces de Paginator : Paginator Links

Laravel 5.7 le permite controlar la cantidad de enlaces adicionales que se muestran a cada lado de la "ventana" de la URL del paginador. De forma predeterminada, se muestran tres enlaces a cada lado de los enlaces del paginador principal. Sin embargo, puedes controlar este número usando el método `onEachSide`:
> > Laravel 5.7 allows you to control how many additional links are displayed on each side of the paginator's URL "window". By default, three links are displayed on each side of the primary paginator links. However, you may control this number using the `onEachSide` method:

    {{ $paginator->onEachSide(5)->links() }}

### Filesystem Read / Write Streams

La integración de Flysystem en Laravel ahora ofrece los métodos `readStream` y` writeStream`:
> > Laravel's Flysystem integration now offers `readStream` and `writeStream` methods:

    Storage::disk('s3')->writeStream(
        'remote-file.zip',
        Storage::disk('local')->readStream('local-file.zip')
    );
