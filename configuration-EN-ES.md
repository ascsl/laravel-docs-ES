# Configuración :  Configuration

- [Introduction](#introduction)
- [Environment Configuration](#environment-configuration)
    - [Environment Variable Types](#environment-variable-types)
    - [Retrieving Environment Configuration](#retrieving-environment-configuration)
    - [Determining The Current Environment](#determining-the-current-environment)
- [Accessing Configuration Values](#accessing-configuration-values)
- [Configuration Caching](#configuration-caching)
- [Maintenance Mode](#maintenance-mode)

<a name="introduction"></a>
## Introducción : Introduction

Todos los archivos de configuración para el marco de Laravel se almacenan en el directorio `config`. Cada opción está documentada, así que no dude en consultar los archivos y familiarizarse con las opciones disponibles para usted.
> > All of the configuration files for the Laravel framework are stored in the `config` directory. Each option is documented, so feel free to look through the files and get familiar with the options available to you.

<a name="environment-configuration"></a>
## Configuración del entorno : Environment Configuration

A menudo es útil tener diferentes valores de configuración basados ​​en el entorno donde se ejecuta la aplicación. Por ejemplo, puede desear usar un controlador de caché diferente localmente que en el servidor de producción.
> > It is often helpful to have different configuration values based on the environment where the application is running. For example, you may wish to use a different cache driver locally than you do on your production server.

Para hacer esto más fácil, Laravel utiliza la biblioteca PHP [DotEnv](https://github.com/vlucas/phpdotenv) de Vance Lucas. En una instalación nueva de Laravel, el directorio raíz de su aplicación contendrá un archivo `.env.example`. Si instala Laravel mediante Composer, este archivo cambiará automáticamente de nombre a `.env`. De lo contrario, debe cambiar el nombre del archivo manualmente.
> > To make this a cinch, Laravel utilizes the [DotEnv](https://github.com/vlucas/phpdotenv) PHP library by Vance Lucas. In a fresh Laravel installation, the root directory of your application will contain a `.env.example` file. If you install Laravel via Composer, this file will automatically be renamed to `.env`. Otherwise, you should rename the file manually.

Su archivo `.env` no debe comprometerse con el control de origen de su aplicación, ya que cada desarrollador / servidor que use su aplicación podría requerir una configuración de entorno diferente. Además, esto supondría un riesgo para la seguridad en caso de que un intruso obtenga acceso a su repositorio de control de origen, ya que cualquier credencial confidencial quedaría expuesta.
> > Your `.env` file should not be committed to your application's source control, since each developer / server using your application could require a different environment configuration. Furthermore, this would be a security risk in the event an intruder gains access to your source control repository, since any sensitive credentials would get exposed.

Si está desarrollando en equipo, puede continuar incluyendo un archivo `.env.example` con su aplicación. Al poner valores de marcador de posición en el archivo de configuración de ejemplo, otros desarrolladores en su equipo pueden ver claramente qué variables de entorno son necesarias para ejecutar su aplicación. También puede crear un archivo `.env.testing`. Este archivo anulará el archivo `.env` cuando ejecute las pruebas de PHPUnit o ejecute los comandos de Artisan con la opción `--env=testing`.
> > If you are developing with a team, you may wish to continue including a `.env.example` file with your application. By putting place-holder values in the example configuration file, other developers on your team can clearly see which environment variables are needed to run your application. You may also create a `.env.testing` file. This file will override the `.env` file when running PHPUnit tests or executing Artisan commands with the `--env=testing` option.

> {tip} Cualquier variable en su archivo `.env` puede ser anulada por variables de entorno externas, como variables de entorno a nivel de servidor o de nivel de sistema.
> > > {tip} Any variable in your `.env` file can be overridden by external environment variables such as server-level or system-level environment variables.

<a name="environment-variable-types"></a>
### Tipos de variables de entorno : Environment Variable Types

Todas las variables en sus archivos `.env` se analizan como cadenas, por lo que algunos valores reservados se han creado para permitirle devolver un rango más amplio de tipos desde la función `env()`:
> > All variables in your `.env` files are parsed as strings, so some reserved values have been created to allow you to return a wider range of types from the `env()` function:

`.env` Value  | `env()` Value
------------- | -------------
true | (bool) true
(true) | (bool) true
false | (bool) false
(false) | (bool) false
empty | (string) ''
(empty) | (string) ''
null | (null) null
(null) | (null) null

Si necesita definir una variable de entorno con un valor que contenga espacios, puede hacerlo encerrando el valor entre comillas dobles.
> > If you need to define an environment variable with a value that contains spaces, you may do so by enclosing the value in double quotes.

    APP_NAME="My Application"

<a name="retrieving-environment-configuration"></a>
### Recuperación de la configuración del entorno : Retrieving Environment Configuration

Todas las variables enumeradas en este archivo se cargarán en el PHP `$_ENV` súper global cuando su aplicación reciba una solicitud. Sin embargo, puede usar el helper `env` para recuperar valores de estas variables en sus archivos de configuración. De hecho, si revisa los archivos de configuración de Laravel, notará varias de las opciones que ya usan esta ayuda:
> > All of the variables listed in this file will be loaded into the `$_ENV` PHP super-global when your application receives a request. However, you may use the `env` helper to retrieve values from these variables in your configuration files. In fact, if you review the Laravel configuration files, you will notice several of the options already using this helper:

    'debug' => env('APP_DEBUG', false),

El segundo valor pasado a la función `env` es el "valor predeterminado". Este valor se usará si no existe una variable de entorno para la clave dada.
> > The second value passed to the `env` function is the "default value". This value will be used if no environment variable exists for the given key.

<a name="determining-the-current-environment"></a>
### Determinación del entorno actual : Determining The Current Environment

El entorno de aplicación actual se determina a través de la variable `APP_ENV` de su archivo` .env`. Puede acceder a este valor a través del método `environment` en `App` [facade](/docs/{{version}}/facades):
> > The current application environment is determined via the `APP_ENV` variable from your `.env` file. You may access this value via the `environment` method on the `App` [facade](/docs/{{version}}/facades):

    $environment = App::environment();

También puede pasar argumentos al método `environment` para verificar si el entorno coincide con un valor determinado. El método devolverá `true` si el entorno coincide con cualquiera de los valores dados:
> > You may also pass arguments to the `environment` method to check if the environment matches a given value. The method will return `true` if the environment matches any of the given values:

    if (App::environment('local')) {
        // The environment is local
    }

    if (App::environment(['local', 'staging'])) {
        // The environment is either local OR staging...
    }

> {tip} La detección del entorno de aplicación actual puede ser anulada por una variable de entorno `APP_ENV` de nivel de servidor. Esto puede ser útil cuando necesite compartir la misma aplicación para diferentes configuraciones de entorno, para que pueda configurar un host determinado para que coincida con un entorno determinado en las configuraciones de su servidor.
> > > {tip} The current application environment detection can be overridden by a server-level `APP_ENV` environment variable. This can be useful when you need to share the same application for different environment configurations, so you can set up a given host to match a given environment in your server's configurations.

<a name="accessing-configuration-values"></a>
## Acceder a los valores de configuración : Accessing Configuration Values

Puede acceder fácilmente a sus valores de configuración utilizando la función global de ayuda `config` desde cualquier lugar de su aplicación. Se puede acceder a los valores de configuración usando la sintaxis "punto", que incluye el nombre del archivo y la opción a la que desea acceder. También se puede especificar un valor predeterminado y se devolverá si la opción de configuración no existe:
> > You may easily access your configuration values using the global `config` helper function from anywhere in your application. The configuration values may be accessed using "dot" syntax, which includes the name of the file and option you wish to access. A default value may also be specified and will be returned if the configuration option does not exist:

    $value = config('app.timezone');

Para establecer valores de configuración en tiempo de ejecución, pase una matriz al helper `config`:
> > To set configuration values at runtime, pass an array to the `config` helper:

    config(['app.timezone' => 'America/Chicago']);

<a name="configuration-caching"></a>
## Caché de configuración : Configuration Caching

Para darle a su aplicación un aumento de velocidad, debe almacenar en caché todos sus archivos de configuración en un solo archivo usando el comando Artisan `config:cache`. Esto combinará todas las opciones de configuración para su aplicación en un único archivo que el marco cargará rápidamente.
> > To give your application a speed boost, you should cache all of your configuration files into a single file using the `config:cache` Artisan command. This will combine all of the configuration options for your application into a single file which will be loaded quickly by the framework.

Normalmente debería ejecutar el comando `php artisan config:cache` como parte de su rutina de implementación de producción. El comando no se debe ejecutar durante el desarrollo local ya que las opciones de configuración con frecuencia deberán cambiarse durante el desarrollo de la aplicación.
> > You should typically run the `php artisan config:cache` command as part of your production deployment routine. The command should not be run during local development as configuration options will frequently need to be changed during the course of your application's development.

> {note} Si ejecuta el comando `config:cache` durante su proceso de implementación, debe asegurarse de llamar solo a la función `env` desde sus archivos de configuración. Una vez que la configuración ha sido almacenada en caché, el archivo `.env` no se cargará y todas las llamadas a la función `env` devolverán `null`.
> > > {note} If you execute the `config:cache` command during your deployment process, you should be sure that you are only calling the `env` function from within your configuration files. Once the configuration has been cached, the `.env` file will not be loaded and all calls to the `env` function will return `null`.

<a name="maintenance-mode"></a>
## Modo de mantenimiento : Maintenance Mode

Cuando su aplicación se encuentre en modo de mantenimiento, se mostrará una vista personalizada para todas las solicitudes en su aplicación. Esto facilita la "desactivación" de su aplicación mientras se actualiza o cuando se realiza mantenimiento. Se incluye una verificación de modo de mantenimiento en la pila de middleware predeterminada para su aplicación. Si la aplicación está en modo de mantenimiento, se lanzará una `MaintenanceModeException` con un código de estado de 503.
> > When your application is in maintenance mode, a custom view will be displayed for all requests into your application. This makes it easy to "disable" your application while it is updating or when you are performing maintenance. A maintenance mode check is included in the default middleware stack for your application. If the application is in maintenance mode, a `MaintenanceModeException` will be thrown with a status code of 503.

Para habilitar el modo de mantenimiento, ejecute el comando `down` Artisan:
> > To enable maintenance mode, execute the `down` Artisan command:

    php artisan down

También puede proporcionar las opciones `message` y `retry` al comando `down`. El valor de `message` se puede usar para mostrar o registrar un mensaje personalizado, mientras que el valor `retry` se establecerá como el valor del encabezado HTTP `Retry-After`:
> > You may also provide `message` and `retry` options to the `down` command. The `message` value may be used to display or log a custom message, while the `retry` value will be set as the `Retry-After` HTTP header's value:

    php artisan down --message="Upgrading Database" --retry=60

Incluso en el modo de mantenimiento, las direcciones o redes IP específicas pueden acceder a la aplicación utilizando la opción `allow` del comando:
> > Even while in maintenance mode, specific IP addresses or networks may be allowed to access the application using the command's `allow` option:

    php artisan down --allow=127.0.0.1 --allow=192.168.0.0/16

Para deshabilitar el modo de mantenimiento, use el comando `up`:
> > To disable maintenance mode, use the `up` command:

    php artisan up

> {tip} Puede personalizar la plantilla del modo de mantenimiento predeterminado definiendo su propia plantilla en `resources/views/errors/503.blade.php`.
> > > {tip} You may customize the default maintenance mode template by defining your own template at `resources/views/errors/503.blade.php`.

#### Modo de mantenimiento y colas : Maintenance Mode & Queues

Mientras su aplicación esté en modo de mantenimiento, no se manejarán [trabajos en cola](/docs/{{version}}/queues). Los trabajos seguirán siendo manejados de forma normal una vez que la aplicación esté fuera del modo de mantenimiento.
> > While your application is in maintenance mode, no [queued jobs](/docs/{{version}}/queues) will be handled. The jobs will continue to be handled as normal once the application is out of maintenance mode.

#### Alternativas al modo de mantenimiento : Alternatives To Maintenance Mode

Como el modo de mantenimiento requiere que su aplicación tenga varios segundos de tiempo de inactividad, considere alternativas como [Envoyer](https://envoyer.io) para lograr una implementación de tiempo de inactividad cero con Laravel.
> > Since maintenance mode requires your application to have several seconds of downtime, consider alternatives like [Envoyer](https://envoyer.io) to accomplish zero-downtime deployment with Laravel.
