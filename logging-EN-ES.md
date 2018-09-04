# Logging

- [Introduction](#introduction)
- [Configuration](#configuration)
    - [Building Log Stacks](#building-log-stacks)
- [Writing Log Messages](#writing-log-messages)
    - [Writing To Specific Channels](#writing-to-specific-channels)
- [Advanced Monolog Channel Customization](#advanced-monolog-channel-customization)
    - [Customizing Monolog For Channels](#customizing-monolog-for-channels)
    - [Creating Monolog Handler Channels](#creating-monolog-handler-channels)
    - [Creating Channels Via Factories](#creating-channels-via-factories)

<a name="introduction"></a>
## Introducción : Introduction

Para ayudarlo a obtener más información sobre lo que sucede dentro de su aplicación, Laravel provee sólidos servicios de registro que le permiten registrar mensajes en los archivos, el registro de errores del sistema e incluso Slack para notificar a todo su equipo.
> > To help you learn more about what's happening within your application, Laravel provides robust logging services that allow you to log messages to files, the system error log, and even to Slack to notify your entire team.

Internamente, Laravel utiliza la biblioteca [Monolog](https://github.com/Seldaek/monolog), que brinda soporte para una variedad de potentes manejadores de registros. Laravel hace que sea muy fácil configurar estos manejadores, lo que le permite mezclarlos y combinarlos para personalizar el manejo de log de su aplicación.
> > Under the hood, Laravel utilizes the [Monolog](https://github.com/Seldaek/monolog) library, which provides support for a variety of powerful log handlers. Laravel makes it a cinch to configure these handlers, allowing you to mix and match them to customize your application's log handling.

<a name="configuration"></a>
## Configuración : Configuration

Toda la configuración para el sistema de registro de su aplicación se encuentra en el archivo de configuración `config/logging.php`. Este archivo le permite configurar los canales de registro de su aplicación, así que asegúrese de revisar cada uno de los canales disponibles y sus opciones. Por supuesto, revisaremos algunas opciones comunes a continuación.
> > All of the configuration for your application's logging system is housed in the `config/logging.php` configuration file. This file allows you to configure your application's log channels, so be sure to review each of the available channels and their options. Of course, we'll review a few common options below.

Por defecto, Laravel usará el canal `stack` cuando inicie sesión. El canal `stack` se usa para agregar múltiples canales de registro en un solo canal. Para obtener más información sobre la construcción de pilas, consulte la [documentación a continuación](#building-log-stacks).
> > By default, Laravel will use the `stack` channel when logging messages. The `stack` channel is used to aggregate multiple log channels into a single channel. For more information on building stacks, check out the [documentation below](#building-log-stacks).

#### Configurando el nombre del canal : Configuring The Channel Name

Por defecto, Monolog se instancia con un "nombre de canal" que coincide con el entorno actual, como `production` o `local`. Para cambiar este valor, agrega una opción `name` a la configuración de tu canal:
> > By default, Monolog is instantiated with a "channel name" that matches the current environment, such as `production` or `local`. To change this value, add a `name` option to your channel's configuration:

    'stack' => [
        'driver' => 'stack',
        'name' => 'channel-name',
        'channels' => ['single', 'slack'],
    ],

#### Controladores de canal disponibles : Available Channel Drivers

Name | Descripción | Description
------------- | ------------- | -------------
`stack` | Un contenedor para facilitar la creación de canales "multicanal" | A wrapper to facilitate creating "multi-channel" channels
`single` | Un archivo único o canal de registrador basado en ruta (`StreamHandler`) | A single file or path based logger channel (`StreamHandler`)
`daily` | Un driver Monolog basado en `RotatingFileHandler` que gira a diario | A `RotatingFileHandler` based Monolog driver which rotates daily
`slack` | Un driver de Monolog basado en `SlackWebhookHandler` | A `SlackWebhookHandler` based Monolog driver
`syslog` | Un driver Monolog basado en `SyslogHandler` | A `SyslogHandler` based Monolog driver
`errorlog` | Un driver Monolog basado en `ErrorLogHandler` | A `ErrorLogHandler` based Monolog driver
`monolog` | Un driver de fábrica de Monolog que puede usar cualquier controlador de Monolog compatible | A Monolog factory driver that may use any supported Monolog handler
`custom` | Un driver que llama a una fábrica especificada para crear un canal | A driver that calls a specified factory to create a channel

> {tip} Consulte la documentación sobre [personalización avanzada del canal](#advanced-monolog-channel-customization) para obtener más información sobre los drivers `monolog` y` custom`.
> > > {tip} Check out the documentation on [advanced channel customization](#advanced-monolog-channel-customization) to learn more about the `monolog` and `custom` drivers.

#### Configurando el canal Slack : Configuring The Slack Channel

El canal `slack` requiere una opción de configuración `url`. Esta URL debe coincidir con una URL para un [enlace web entrante](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks) que haya configurado para su equipo Slack.
> > The `slack` channel requires a `url` configuration option. This URL should match a URL for an [incoming webhook](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks) that you have configured for your Slack team.

<a name="building-log-stacks"></a>
### Creación de pilas de registro : Building Log Stacks

Como se mencionó anteriormente, el controlador `stack` le permite combinar múltiples canales en un solo canal de registro. Para ilustrar cómo usar las pilas de registros, echemos un vistazo a una configuración de ejemplo que puede ver en una aplicación de producción:
> > As previously mentioned, the `stack` driver allows you to combine multiple channels into a single log channel. To illustrate how to use log stacks, let's take a look at an example configuration that you might see in a production application:

    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => ['syslog', 'slack'],
        ],

        'syslog' => [
            'driver' => 'syslog',
            'level' => 'debug',
        ],

        'slack' => [
            'driver' => 'slack',
            'url' => env('LOG_SLACK_WEBHOOK_URL'),
            'username' => 'Laravel Log',
            'emoji' => ':boom:',
            'level' => 'critical',
        ],
    ],

Vamos a diseccionar esta configuración. Primero, observe que nuestro canal `stack` agrega otros dos canales a través de su opción `channels`: `syslog` y `slack`. Por lo tanto, al registrar mensajes, ambos canales tendrán la oportunidad de registrar el mensaje.
> > Let's dissect this configuration. First, notice our `stack` channel aggregates two other channels via its `channels` option: `syslog` and `slack`. So, when logging messages, both of these channels will have the opportunity to log the message.

#### Niveles de registro : Log Levels

Tome nota de la opción de configuración de `level` presente en las configuraciones de canal `syslog` y `slack` en el ejemplo anterior. Esta opción determina el "nivel" mínimo que debe tener un mensaje para que el canal lo registre. Monolog, que alimenta los servicios de registro de Laravel, ofrece todos los niveles de registro definidos en la [especificación RFC 5424](https://tools.ietf.org/html/rfc5424): **emergencia**, **alerta**, **crítico**, **error**, **advertencia**, **aviso**, **información** y **depuración**.
> > Take note of the `level` configuration option present on the `syslog` and `slack` channel configurations in the example above. This option determines the minimum "level" a message must be in order to be logged by the channel. Monolog, which powers Laravel's logging services, offers all of the log levels defined in the [RFC 5424 specification](https://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info**, and **debug**.

Entonces, imagina que registramos un mensaje usando el método `debug`:
> > So, imagine we log a message using the `debug` method:

    Log::debug('An informational message.');

Dada nuestra configuración, el canal `syslog` escribirá el mensaje en el registro del sistema; sin embargo, dado que el mensaje de error no es "crítico" o superior, no se enviará a Slack. Sin embargo, si registramos un mensaje `emergency`, se enviará tanto al registro del sistema como ¿¿¿a la holgura??? ya que el nivel `emergency` está por encima de nuestro umbral de nivel mínimo para ambos canales:
> > Given our configuration, the `syslog` channel will write the message to the system log; however, since the error message is not `critical` or above, it will not be sent to Slack. However, if we log an `emergency` message, it will be sent to both the system log and Slack since the `emergency` level is above our minimum level threshold for both channels:

    Log::emergency('The system is down!');

<a name="writing-log-messages"></a>
## Escritura de mensajes de registro : Writing Log Messages

Puede escribir información en los registros usando la [fachada](/docs/{{version}}/facades) `Log`. Como se mencionó anteriormente, el registrador proporciona los ocho niveles de registro definidos en la [especificación RFC 5424](https://tools.ietf.org/html/rfc5424): **emergencia**, **alerta**, **crítica**, **error**, **advertencia**, **aviso**, **información** y **depuración**:
> > You may write information to the logs using the `Log` [facade](/docs/{{version}}/facades). As previously mentioned, the logger provides the eight logging levels defined in the [RFC 5424 specification](https://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** and **debug**:

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

Entonces, puede llamar a cualquiera de estos métodos para registrar un mensaje para el nivel correspondiente. De forma predeterminada, el mensaje se escribirá en el canal de registro predeterminado según lo configurado por su archivo de configuración `config/logging.php`:
> > So, you may call any of these methods to log a message for the corresponding level. By default, the message will be written to the default log channel as configured by your `config/logging.php` configuration file:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Support\Facades\Log;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            Log::info('Showing user profile for user: '.$id);

            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

#### Información contextual : Contextual Information

También se puede pasar una matriz de datos contextuales a los métodos de registro. Esta información contextual se formateará y se mostrará con el mensaje de registro:
> > An array of contextual data may also be passed to the log methods. This contextual data will be formatted and displayed with the log message:

    Log::info('User failed to login.', ['id' => $user->id]);

<a name="writing-to-specific-channels"></a>
### Escritura en canales específicos : Writing To Specific Channels

En ocasiones, es posible que desee registrar un mensaje en un canal que no sea el predeterminado de su aplicación. Puede usar el método `channel` en la fachada `Log` para recuperar y registrar en cualquier canal definido en su archivo de configuración:
> > Sometimes you may wish to log a message to a channel other than your application's default channel. You may use the `channel` method on the `Log` facade to retrieve and log to any channel defined in your configuration file:

    Log::channel('slack')->info('Something happened!');

Si desea crear una pila de registro a pedido que consta de múltiples canales, puede usar el método `stack`:
> > If you would like to create an on-demand logging stack consisting of multiple channels, you may use the `stack` method:

    Log::stack(['single', 'slack'])->info('Something happened!');


<a name="advanced-monolog-channel-customization"></a>
## Advanced Monolog Channel Customization

<a name="customizing-monolog-for-channels"></a>
### Personalización de Monolog para canales : Customizing Monolog For Channels

A veces puede necesitar un control total sobre cómo está configurado Monolog para un canal existente. Por ejemplo, es posible que desee configurar una implementación de Monolog `FormatterInterface` personalizada para los manejadores de un canal determinado.
> > Sometimes you may need complete control over how Monolog is configured for an existing channel. For example, you may want to configure a custom Monolog `FormatterInterface` implementation for a given channel's handlers.

Para comenzar, defina un array `tap` en la configuración del canal. El array `tap` debe contener una lista de clases que deberían tener la oportunidad de personalizar (o "tocar") la instancia de Monolog una vez creada:
> > To get started, define a `tap` array on the channel's configuration. The `tap` array should contain a list of classes that should have an opportunity to customize (or "tap" into) the Monolog instance after it is created:

    'single' => [
        'driver' => 'single',
        'tap' => [App\Logging\CustomizeFormatter::class],
        'path' => storage_path('logs/laravel.log'),
        'level' => 'debug',
    ],

Una vez que hayas configurado la opción `tap` en tu canal, estás listo para definir la clase que personalizará tu instancia de Monolog. Esta clase solo necesita un único método: `__invoke`, que recibe una instancia `Illuminate\Log\Logger`. La instancia `Illuminate\Log\Logger` representa todas las llamadas de método a la instancia de Monolog subyacente:
> > Once you have configured the `tap` option on your channel, you're ready to define the class that will customize your Monolog instance. This class only needs a single method: `__invoke`, which receives an `Illuminate\Log\Logger` instance. The `Illuminate\Log\Logger` instance proxies all method calls to the underlying Monolog instance:

    <?php

    namespace App\Logging;

    class CustomizeFormatter
    {
        /**
         * Customize the given logger instance.
         *
         * @param  \Illuminate\Log\Logger  $logger
         * @return void
         */
        public function __invoke($logger)
        {
            foreach ($logger->getHandlers() as $handler) {
                $handler->setFormatter(...);
            }
        }
    }

> {tip} Todas sus clases de "tap" son resueltas por [contenedor de servicio](/docs/{{version}}/container), por lo que cualquier dependencia de constructor que requieran será inyectada automáticamente.
> > > {tip} All of your "tap" classes are resolved by the [service container](/docs/{{version}}/container), so any constructor dependencies they require will automatically be injected.

<a name="creating-monolog-handler-channels"></a>
### Creating Monolog Handler Channels

Monolog tiene una variedad de [controladores disponibles](https://github.com/Seldaek/monolog/tree/master/src/Monolog/Handler). En algunos casos, el tipo de registrador que desea crear es simplemente un controlador de Monolog con una instancia de un controlador específico. Estos canales se pueden crear con el controlador `monolog`.
> > Monolog has a variety of [available handlers](https://github.com/Seldaek/monolog/tree/master/src/Monolog/Handler). In some cases, the type of logger you wish to create is merely a Monolog driver with an instance of a specific handler.  These channels can be created using the `monolog` driver.

Cuando se utiliza el controlador `monolog`, la opción de configuración `handler` se usa para especificar qué controlador se creará una instancia. Opcionalmente, cualquier parámetro de constructor que el manejador necesite se puede especificar usando la opción de configuración `handler_with`:
> > When using the `monolog` driver, the `handler` configuration option is used to specify which handler will be instantiated. Optionally, any constructor parameters the handler needs may be specified using the `handler_with` configuration option:

    'logentries' => [
        'driver'  => 'monolog',
        'handler' => Monolog\Handler\SyslogUdpHandler::class,
        'handler_with' => [
            'host' => 'my.logentries.internal.datahubhost.company.com',
            'port' => '10000',
        ],
    ],

#### Formateadores de Monolog : Monolog Formatters

Cuando se utiliza el controlador `monolog`, el `LineFormatter` de Monolog se usará como el formateador predeterminado. Sin embargo, puede personalizar el tipo de formateador que se pasa al controlador utilizando las opciones de configuración `formatter` y `formatter_with`:
> > When using the `monolog` driver, the Monolog `LineFormatter` will be used as the default formatter. However, you may customize the type of formatter passed to the handler using the `formatter` and `formatter_with` configuration options:

    'browser' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\BrowserConsoleHandler::class,
        'formatter' => Monolog\Formatter\HtmlFormatter::class,
        'formatter_with' => [
            'dateFormat' => 'Y-m-d',
        ],
    ],

Si está utilizando un controlador Monolog que es capaz de proporcionar su propio formateador, puede establecer el valor de la opción de configuración `formatter` en `default`:
> > If you are using a Monolog handler that is capable of providing its own formatter, you may set the value of the `formatter` configuration option to `default`:

    'newrelic' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\NewRelicHandler::class,
        'formatter' => 'default',
    ],

<a name="creating-channels-via-factories"></a>
### Creación de canales a través de fábricas : Creating Channels Via Factories

Si desea definir un canal completamente personalizado en el que tenga control total sobre la creación de instancias y la configuración de Monolog, puede especificar un tipo de controlador `personalizado` en su archivo de configuración `config/logging.php`. Su configuración debe incluir una opción `via` para apuntar a la clase de fábrica que se invocará para crear la instancia de Monolog:
> > If you would like to define an entirely custom channel in which you have full control over Monolog's instantiation and configuration, you may specify a `custom` driver type in your `config/logging.php` configuration file. Your configuration should include a `via` option to point to the factory class which will be invoked to create the Monolog instance:

    'channels' => [
        'custom' => [
            'driver' => 'custom',
            'via' => App\Logging\CreateCustomLogger::class,
        ],
    ],

Una vez que haya configurado el canal `custom`, estará listo para definir la clase que creará su instancia de Monolog. Esta clase solo necesita un método único: `__invoke`, que debe devolver la instancia de Monolog:
> > Once you have configured the `custom` channel, you're ready to define the class that will create your Monolog instance. This class only needs a single method: `__invoke`, which should return the Monolog instance:

    <?php

    namespace App\Logging;

    use Monolog\Logger;

    class CreateCustomLogger
    {
        /**
         * Create a custom Monolog instance.
         *
         * @param  array  $config
         * @return \Monolog\Logger
         */
        public function __invoke(array $config)
        {
            return new Logger(...);
        }
    }
