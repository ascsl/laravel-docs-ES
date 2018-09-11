# Estructura de directorios : Directory Structure

- [Introduction](#introduction)
- [The Root Directory](#the-root-directory)
    - [The `app` Directory](#the-root-app-directory)
    - [The `bootstrap` Directory](#the-bootstrap-directory)
    - [The `config` Directory](#the-config-directory)
    - [The `database` Directory](#the-database-directory)
    - [The `public` Directory](#the-public-directory)
    - [The `resources` Directory](#the-resources-directory)
    - [The `routes` Directory](#the-routes-directory)
    - [The `storage` Directory](#the-storage-directory)
    - [The `tests` Directory](#the-tests-directory)
    - [The `vendor` Directory](#the-vendor-directory)
- [The App Directory](#the-app-directory)
    - [The `Broadcasting` Directory](#the-broadcasting-directory)
    - [The `Console` Directory](#the-console-directory)
    - [The `Events` Directory](#the-events-directory)
    - [The `Exceptions` Directory](#the-exceptions-directory)
    - [The `Http` Directory](#the-http-directory)
    - [The `Jobs` Directory](#the-jobs-directory)
    - [The `Listeners` Directory](#the-listeners-directory)
    - [The `Mail` Directory](#the-mail-directory)
    - [The `Notifications` Directory](#the-notifications-directory)
    - [The `Policies` Directory](#the-policies-directory)
    - [The `Providers` Directory](#the-providers-directory)
    - [The `Rules` Directory](#the-rules-directory)

<a name="introduction"></a>
## Introducción : Introduction

La estructura predeterminada de la aplicación Laravel está destinada a proporcionar un excelente punto de partida para aplicaciones tanto grandes como pequeñas. Por supuesto, puedes organizar tu aplicación como quieras. Laravel casi no impone restricciones sobre dónde se encuentra una clase determinada, siempre que Composer pueda autocargar la clase.
> > The default Laravel application structure is intended to provide a great starting point for both large and small applications. Of course, you are free to organize your application however you like. Laravel imposes almost no restrictions on where any given class is located - as long as Composer can autoload the class.

#### ¿Dónde está el directorio de modelos? : Where Is The Models Directory?

Al comenzar con Laravel, muchos desarrolladores se confunden por la falta de un directorio `modelos`. Sin embargo, la falta de dicho directorio es intencional. Encontramos la palabra "modelos" ambigua, ya que significa muchas cosas diferentes para muchas personas diferentes. Algunos desarrolladores se refieren al "modelo" de una aplicación como la totalidad de su lógica comercial, mientras que otros se refieren a "modelos" como clases que interactúan con una base de datos relacional.
> > When getting started with Laravel, many developers are confused by the lack of a `models` directory. However, the lack of such a directory is intentional. We find the word "models" ambiguous since it means many different things to many different people. Some developers refer to an application's "model" as the totality of all of its business logic, while others refer to "models" as classes that interact with a relational database.

Por esta razón, elegimos colocar modelos Eloquent en el directorio `app` de forma predeterminada, y permitimos que el desarrollador los coloque en otro lugar si así lo desean.
> > For this reason, we choose to place Eloquent models in the `app` directory by default, and allow the developer to place them somewhere else if they choose.

<a name="the-root-directory"></a>
## El directorio raíz : The Root Directory

<a name="the-root-app-directory"></a>
#### El directorio App : The App Directory

El directorio `app`, como era de esperar, contiene el código central de su aplicación. Exploraremos este directorio con más detalle pronto; sin embargo, casi todas las clases en su aplicación estarán en este directorio.
> > The `app` directory, as you might expect, contains the core code of your application. We'll explore this directory in more detail soon; however, almost all of the classes in your application will be in this directory.

<a name="the-bootstrap-directory"></a>
#### El directorio Bootstrap : The Bootstrap Directory

El directorio `bootstrap` contiene el archivo `app.php` que inicia el framework. Este directorio también alberga un directorio `cache` que contiene archivos generados por el framework para la optimización del rendimiento, como la ruta y los archivos de caché de servicios.
> > The `bootstrap` directory contains the `app.php` file which bootstraps the framework. This directory also houses a `cache` directory which contains framework generated files for performance optimization such as the route and services cache files.

<a name="the-config-directory"></a>
#### El directorio Config : The Config Directory

El directorio `config`, como su nombre lo indica, contiene todos los archivos de configuración de su aplicación. Es una gran idea leer todos estos archivos y familiarizarse con todas las opciones disponibles para usted.
> > The `config` directory, as the name implies, contains all of your application's configuration files. It's a great idea to read through all of these files and familiarize yourself with all of the options available to you.

<a name="the-database-directory"></a>
#### El directorio Database : The Database Directory

El directorio `database` contiene sus migraciones de bases de datos, fábricas de modelos y semillas. Si lo desea, también puede usar este directorio para mantener una base de datos SQLite.
> > The `database` directory contains your database migrations, model factories, and seeds. If you wish, you may also use this directory to hold an SQLite database.

<a name="the-public-directory"></a>
#### El directorio Public : The Public Directory

El directorio `public` contiene el archivo `index.php`, que es el punto de entrada para todas las solicitudes que ingresan a su aplicación y configura la carga automática. Este directorio también contiene sus activos como imágenes, JavaScript y CSS.
> > The `public` directory contains the `index.php` file, which is the entry point for all requests entering your application and configures autoloading. This directory also houses your assets such as images, JavaScript, and CSS.

<a name="the-resources-directory"></a>
#### El directorio Resources : The Resources Directory

El directorio `resources` contiene sus vistas, así como sus activos sin compilar, como LESS, SASS o JavaScript. Este directorio también alberga todos sus archivos de idiomas.
> > The `resources` directory contains your views as well as your raw, un-compiled assets such as LESS, SASS, or JavaScript. This directory also houses all of your language files.

<a name="the-routes-directory"></a>
#### El directorio Routes : The Routes Directory

El directorio `routes` contiene todas las definiciones de ruta para su aplicación. Por defecto, se incluyen varios archivos de ruta con Laravel: `web.php`, `api.php`, `console.php` y `channels.php`.
> > The `routes` directory contains all of the route definitions for your application. By default, several route files are included with Laravel: `web.php`, `api.php`, `console.php` and `channels.php`.

El archivo `web.php` contiene rutas que el `RouteServiceProvider` coloca en el grupo de middleware `web`, que proporciona el estado de la sesión, la protección CSRF y el cifrado de cookies. Si su aplicación no ofrece una API RESTABLECIDA y sin estado, es muy probable que todas sus rutas estén definidas en el archivo `web.php`.
> > The `web.php` file contains routes that the `RouteServiceProvider` places in the `web` middleware group, which provides session state, CSRF protection, and cookie encryption. If your application does not offer a stateless, RESTful API, all of your routes will most likely be defined in the `web.php` file.

El archivo `api.php` contiene rutas que el `RouteServiceProvider` coloca en el grupo de middleware `api`, que proporciona una limitación de velocidad. Estas rutas están destinadas a ser sin estado, por lo que las solicitudes que ingresan a la aplicación a través de estas rutas están destinadas a ser autenticadas mediante tokens y no tendrán acceso al estado de la sesión.
> > The `api.php` file contains routes that the `RouteServiceProvider` places in the `api` middleware group, which provides rate limiting. These routes are intended to be stateless, so requests entering the application through these routes are intended to be authenticated via tokens and will not have access to session state.

El archivo `console.php` es donde puede definir todos sus comandos de consola basados ​​en Closure. Cada Closure está vinculado a una instancia de comando que permite un enfoque simple para interactuar con los métodos IO de cada comando. Aunque este archivo no define rutas HTTP, define los puntos de entrada (rutas) basados ​​en la consola en su aplicación.
> > The `console.php` file is where you may define all of your Closure based console commands. Each Closure is bound to a command instance allowing a simple approach to interacting with each command's IO methods. Even though this file does not define HTTP routes, it defines console based entry points (routes) into your application.

El archivo `channels.php` es donde puede registrar todos los canales de difusión de eventos que admite su aplicación.
> > The `channels.php` file is where you may register all of the event broadcasting channels that your application supports.

<a name="the-storage-directory"></a>
#### El directorio Storage : The Storage Directory

El directorio `storage` contiene sus plantillas compiladas Blade, sesiones basadas en archivos, cachés de archivos y otros archivos generados por el framework. Este directorio está segregado en directorios `app`, `framework`, y `logs`. El directorio `app` se puede usar para almacenar cualquier archivo generado por su aplicación. El directorio `framework` se usa para almacenar archivos y cachés generados por el framework. Finalmente, el directorio `logs` contiene los archivos de registro de su aplicación.
> > The `storage` directory contains your compiled Blade templates, file based sessions, file caches, and other files generated by the framework. This directory is segregated into `app`, `framework`, and `logs` directories. The `app` directory may be used to store any files generated by your application. The `framework` directory is used to store framework generated files and caches. Finally, the `logs` directory contains your application's log files.

El directorio `storage/app/public` se puede usar para almacenar archivos generados por el usuario, como avatares de perfil, que deben ser de acceso público. Debería crear un enlace simbólico en `public/storage` que apunta a este directorio. Puede crear el enlace utilizando el comando `php artisan storage:link`.
> > The `storage/app/public` directory may be used to store user-generated files, such as profile avatars, that should be publicly accessible. You should create a symbolic link at `public/storage` which points to this directory. You may create the link using the `php artisan storage:link` command.

<a name="the-tests-directory"></a>
#### El directorio Test : The Tests Directory

El directorio `tests` contiene tus pruebas automáticas. Un ejemplo [PHPUnit](https://phpunit.de/) se proporciona de fábrica. Cada clase de prueba debe tener el sufijo con la palabra `Test`. Puede ejecutar sus pruebas utilizando los comandos `phpunit` o `php vendor/bin/phpunit`.
> > The `tests` directory contains your automated tests. An example [PHPUnit](https://phpunit.de/) is provided out of the box. Each test class should be suffixed with the word `Test`. You may run your tests using the `phpunit` or `php vendor/bin/phpunit` commands.

<a name="the-vendor-directory"></a>
#### El directorio Vendor : The Vendor Directory

El directorio `vendor` contiene sus dependencias [Composer](https://getcomposer.org).
> > The `vendor` directory contains your [Composer](https://getcomposer.org) dependencies.

<a name="the-app-directory"></a>
## El directorio App : The App Directory

La mayoría de su aplicación se encuentra en el directorio `app`. Por defecto, este directorio tiene el espacio de nombre en `App` y Composer lo carga automáticamente usando el [PSR-4 autocoloring standard](http://www.php-fig.org/psr/psr-4/).
> > The majority of your application is housed in the `app` directory. By default, this directory is namespaced under `App` and is autoloaded by Composer using the [PSR-4 autoloading standard](http://www.php-fig.org/psr/psr-4/).

El directorio `app` contiene una variedad de directorios adicionales tales como `Console`, `Http`, y `Providers`. Piense en los directorios `Console` y `Http` como una API en el núcleo de su aplicación. El protocolo HTTP y CLI son ambos mecanismos para interactuar con su aplicación, pero en realidad no contienen lógica de aplicación. En otras palabras, son dos formas de emitir comandos a su aplicación. El directorio `Console` contiene todos sus comandos Artisan, mientras que el directorio `Http` contiene sus controladores, middleware y solicitudes.
> > The `app` directory contains a variety of additional directories such as `Console`, `Http`, and `Providers`. Think of the `Console` and `Http` directories as providing an API into the core of your application. The HTTP protocol and CLI are both mechanisms to interact with your application, but do not actually contain application logic. In other words, they are two ways of issuing commands to your application. The `Console` directory contains all of your Artisan commands, while the `Http` directory contains your controllers, middleware, and requests.

Se generarán una variedad de directorios dentro del directorio `app` mientras usa los comandos Artisan `make` para generar clases. Por lo tanto, por ejemplo, el directorio `app/Jobs` no existirá hasta que ejecute el comando Artisan `make:job` para generar una clase de trabajo.
> > A variety of other directories will be generated inside the `app` directory as you use the `make` Artisan commands to generate classes. So, for example, the `app/Jobs` directory will not exist until you execute the `make:job` Artisan command to generate a job class.

> {tip} Muchas de las clases en el directorio `app` pueden ser generadas por Artisan a través de comandos. Para revisar los comandos disponibles, ejecute el comando `php artisan list make` en su terminal.
> > > {tip} Many of the classes in the `app` directory can be generated by Artisan via commands. To review the available commands, run the `php artisan list make` command in your terminal.

<a name="the-broadcasting-directory"></a>
#### El directorio Broadcasting : The Broadcasting Directory

El directorio `Broadcasting` contiene todas las clases de canales de difusión para su aplicación. Estas clases se generan utilizando el comando `make:channel`. Este directorio no existe de manera predeterminada, pero se creará para usted cuando cree su primer canal. Para obtener más información sobre los canales, consulte la documentación sobre [difusión de eventos](/docs/{{version}}/broadcasting).
> > The `Broadcasting` directory contains all of the broadcast channel classes for your application. These classes are generated using the `make:channel` command. This directory does not exist by default, but will be created for you when you create your first channel. To learn more about channels, check out the documentation on [event broadcasting](/docs/{{version}}/broadcasting).

<a name="the-console-directory"></a>
#### El directorio Console : The Console Directory

El directorio `Console` contiene todos los comandos personalizados de Artisan para su aplicación. Estos comandos se pueden generar con el comando `make:command`. Este directorio también alberga su kernel de consola, que es donde se registran los comandos personalizados de Artisan y se definen sus [tareas programadas](/docs/{{version}}/scheduling).
> > The `Console` directory contains all of the custom Artisan commands for your application. These commands may be generated using the `make:command` command. This directory also houses your console kernel, which is where your custom Artisan commands are registered and your [scheduled tasks](/docs/{{version}}/scheduling) are defined.

<a name="the-events-directory"></a>
#### El directorio Events : The Events Directory

Este directorio no existe de manera predeterminada, pero se creará para usted con los comandos Artisan `event:generate` y `make:event`. El directorio `Events`, como es de esperar, alberga [event classes](/docs/{{version}}/events). Los eventos se pueden utilizar para alertar a otras partes de su aplicación de que se ha producido una acción determinada, lo que proporciona una gran flexibilidad y desacoplamiento.
> > This directory does not exist by default, but will be created for you by the `event:generate` and `make:event` Artisan commands. The `Events` directory, as you might expect, houses [event classes](/docs/{{version}}/events). Events may be used to alert other parts of your application that a given action has occurred, providing a great deal of flexibility and decoupling.

<a name="the-exceptions-directory"></a>
#### El directorio Exceptions : The Exceptions Directory

El directorio `Exceptions` contiene el manejador de excepciones de su aplicación y también es un buen lugar para colocar cualquier excepción lanzada por su aplicación. Si desea personalizar cómo se registran o se procesan sus excepciones, debe modificar la clase `Handler` en este directorio.
> > The `Exceptions` directory contains your application's exception handler and is also a good place to place any exceptions thrown by your application. If you would like to customize how your exceptions are logged or rendered, you should modify the `Handler` class in this directory.

<a name="the-http-directory"></a>
#### El directorio Http : The Http Directory

El directorio `Http` contiene sus controladores, middleware y solicitudes de formularios. Casi toda la lógica para manejar las solicitudes que ingresen a su aplicación se colocará en este directorio.
> > The `Http` directory contains your controllers, middleware, and form requests. Almost all of the logic to handle requests entering your application will be placed in this directory.

<a name="the-jobs-directory"></a>
#### El directorio Jobs : The Jobs Directory

Este directorio no existe por defecto, pero se creará para usted si ejecuta el comando Artisan `make:job`. El directorio `Jobs` contiene los [trabajos em cola](/docs/{{version}}/queues) para su aplicación. La aplicación puede poner en cola trabajos o ejecutar de forma síncrona dentro del ciclo de vida de solicitud actual. Los trabajos que se ejecutan sincrónicamente durante la solicitud actual a veces se denominan "comandos" ya que son una implementación del [patrón de comando](https://en.wikipedia.org/wiki/Command_pattern).
> > This directory does not exist by default, but will be created for you if you execute the `make:job` Artisan command. The `Jobs` directory houses the [queueable jobs](/docs/{{version}}/queues) for your application. Jobs may be queued by your application or run synchronously within the current request lifecycle. Jobs that run synchronously during the current request are sometimes referred to as "commands" since they are an implementation of the [command pattern](https://en.wikipedia.org/wiki/Command_pattern).

<a name="the-listeners-directory"></a>
#### El directorio Listeners : The Listeners Directory

Este directorio no existe por defecto, pero se creará para usted si ejecuta los comandos Artisan `event:generate` o `make:listener`. El directorio `Listeners` contiene las clases que manejan sus [eventos](/docs/{{version}}/events). Los escuchas de eventos reciben una instancia de evento y realizan lógica en respuesta al evento que se está disparando. Por ejemplo, un evento `UserRegistered` podría ser manejado por un oyente `SendWelcomeEmail`.
> > This directory does not exist by default, but will be created for you if you execute the `event:generate` or `make:listener` Artisan commands. The `Listeners` directory contains the classes that handle your [events](/docs/{{version}}/events). Event listeners receive an event instance and perform logic in response to the event being fired. For example, a `UserRegistered` event might be handled by a `SendWelcomeEmail` listener.

<a name="the-mail-directory"></a>
#### El directorio Mail : The Mail Directory

Este directorio no existe por defecto, pero se creará para usted si ejecuta el comando Artisan `make:mail`. El directorio `Mail` contiene todas sus clases que representan los correos electrónicos enviados por su aplicación. Los objetos de correo le permiten encapsular toda la lógica de compilar un correo electrónico en una sola clase simple que se puede enviar utilizando el método `Mail::send`.
> > This directory does not exist by default, but will be created for you if you execute the `make:mail` Artisan command. The `Mail` directory contains all of your classes that represent emails sent by your application. Mail objects allow you to encapsulate all of the logic of building an email in a single, simple class that may be sent using the `Mail::send` method.

<a name="the-notifications-directory"></a>
#### El directorio de Notifications : The Notifications Directory

Este directorio no existe de manera predeterminada, pero se creará para usted si ejecuta el comando Artisan `make:notification`. El directorio `Notifications` contiene todas las notificaciones "transaccionales" que envía su aplicación, como notificaciones simples sobre eventos que suceden dentro de su aplicación. La notificación de Laravel presenta resúmenes que envían notificaciones a través de una variedad de controladores como correo electrónico, Slack, SMS o almacenados en una base de datos.
> > This directory does not exist by default, but will be created for you if you execute the `make:notification` Artisan command. The `Notifications` directory contains all of the "transactional" notifications that are sent by your application, such as simple notifications about events that happen within your application. Laravel's notification features abstracts sending notifications over a variety of drivers such as email, Slack, SMS, or stored in a database.

<a name="the-policies-directory"></a>
#### El directorio Policies : The Policies Directory

Este directorio no existe por defecto, pero se creará para usted si ejecuta el comando Artesano `make:policy`. El directorio `Policies` contiene las clases de política de autorización para su aplicación. Las políticas se usan para determinar si un usuario puede realizar una acción dada contra un recurso. Para obtener más información, consulte la [documentación de autorización](/docs/{{version}}/authorization).
> > This directory does not exist by default, but will be created for you if you execute the `make:policy` Artisan command. The `Policies` directory contains the authorization policy classes for your application. Policies are used to determine if a user can perform a given action against a resource. For more information, check out the [authorization documentation](/docs/{{version}}/authorization).

<a name="the-providers-directory"></a>
#### El directorio Providers : The Providers Directory

El directorio `Providers` contiene todos los [proveedores de servicios](/docs/{{version}}/providers) para su aplicación. Los proveedores de servicios inician su aplicación vinculando servicios en el contenedor de servicios, registrando eventos o realizando cualquier otra tarea para preparar su solicitud para las solicitudes entrantes.
> > The `Providers` directory contains all of the [service providers](/docs/{{version}}/providers) for your application. Service providers bootstrap your application by binding services in the service container, registering events, or performing any other tasks to prepare your application for incoming requests.

En una nueva aplicación Laravel, este directorio ya incluirá varios proveedores. Puede agregar sus propios proveedores a este directorio según sea necesario.
> > In a fresh Laravel application, this directory will already contain several providers. You are free to add your own providers to this directory as needed.

<a name="the-rules-directory"></a>
#### El directorio Rules : The Rules Directory

Este directorio no existe por defecto, pero se creará para usted si ejecuta el comando Artisan `make:rule`. El directorio `Rules` contiene los objetos de regla de validación personalizados para su aplicación. Las reglas se usan para encapsular lógica de validación complicada en un objeto simple. Para obtener más información, consulte la [documentación de validación](/docs/{{version}}/validation).
> > This directory does not exist by default, but will be created for you if you execute the `make:rule` Artisan command. The `Rules` directory contains the custom validation rule objects for your application. Rules are used to encapsulate complicated validation logic in a simple object. For more information, check out the [validation documentation](/docs/{{version}}/validation).
