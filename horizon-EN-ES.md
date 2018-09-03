# Laravel Horizon

- [Introduction](#introduction)
- [Installation](#installation)
    - [Configuration](#configuration)
    - [Dashboard Authentication](#dashboard-authentication)
- [Running Horizon](#running-horizon)
    - [Deploying Horizon](#deploying-horizon)
- [Tags](#tags)
- [Notifications](#notifications)
- [Metrics](#metrics)

<a name="introduction"></a>
## Introducción : Introduction

Horizon proporciona un tablero de instrumentos bonito y una configuración basada en código para sus colas Redis potenciadas por Laravel. Horizon le permite monitorear fácilmente las métricas clave de su sistema de colas, como el rendimiento del trabajo, el tiempo de ejecución y las fallas en el trabajo.
> > Horizon provides a beautiful dashboard and code-driven configuration for your Laravel powered Redis queues. Horizon allows you to easily monitor key metrics of your queue system such as job throughput, runtime, and job failures.

Toda la configuración de su trabajador se almacena en un único archivo de configuración simple, lo que permite que su configuración permanezca en control de origen donde todo su equipo puede colaborar.
> > All of your worker configuration is stored in a single, simple configuration file, allowing your configuration to stay in source control where your entire team can collaborate.

<a name="installation"></a>
## Instalación : Installation

> {note} Debido a su uso de señales de proceso asíncronas, Horizon requiere PHP 7.1+. En segundo lugar, debe asegurarse de que su controlador de cola esté configurado en `redis` en su archivo de configuración `queue`.
> > > {note} Due to its usage of async process signals, Horizon requires PHP 7.1+. Secondly, you should ensure that your queue driver is set to `redis` in your `queue` configuration file.

Puede usar Composer para instalar Horizon en su proyecto Laravel:
> > You may use Composer to install Horizon into your Laravel project:

    composer require laravel/horizon

Después de instalar Horizon, publique sus activos utilizando el comando Artisan 'vendor:publish`:
> > After installing Horizon, publish its assets using the `vendor:publish` Artisan command:

    php artisan vendor:publish --provider="Laravel\Horizon\HorizonServiceProvider"

<a name="configuration"></a>
### Configuración : Configuration

Después de publicar los activos de Horizon, su archivo de configuración principal estará ubicado en `config/horizon.php`. Este archivo de configuración le permite configurar sus opciones de trabajador y cada opción de configuración incluye una descripción de su propósito, así que asegúrese de explorar a fondo este archivo.
> > After publishing Horizon's assets, its primary configuration file will be located at `config/horizon.php`. This configuration file allows you to configure your worker options and each configuration option includes a description of its purpose, so be sure to thoroughly explore this file.

#### Opciones de saldo : Balance Options

Horizon le permite elegir entre tres estrategias de equilibrio: `simple`,` auto` y `false`. La estrategia `simple`, que es la predeterminada, divide los trabajos entrantes de manera uniforme entre los procesos:
> > Horizon allows you to choose from three balancing strategies: `simple`, `auto`, and `false`. The `simple` strategy, which is the default, splits incoming jobs evenly between processes:

    'balance' => 'simple',

La estrategia `auto` ajusta la cantidad de procesos de trabajo por cola en función de la carga de trabajo actual de la cola. Por ejemplo, si su cola de `notifications` tiene 1,000 trabajos en espera mientras su cola `render` está vacía, Horizon asignará más trabajadores a su cola de `notifications` hasta que esté vacía. Cuando la opción `balance` se establece en `false`, se utilizará el comportamiento de Laravel predeterminado, que procesa las colas en el orden en que se enumeran en su configuración.
> > The `auto` strategy adjusts the number of worker processes per queue based on the current workload of the queue. For example, if your `notifications` queue has 1,000 waiting jobs while your `render` queue is empty, Horizon will allocate more workers to your `notifications` queue until it is empty. When the `balance` option is set to `false`, the default Laravel behavior will be used, which processes queues in the order they are listed in your configuration.

<a name="dashboard-authentication"></a>
### Autenticación del tablero : Dashboard Authentication

Horizon expone un tablero en `/horizon`. Por defecto, solo podrá acceder a este panel en el entorno `local`. Para definir una política de acceso más específica para el tablero, debe usar el método `Horizon::auth`. El método `auth` acepta una rellamada que debe devolver `true` o `false`, lo que indica si el usuario debería tener acceso al panel de Horizon. Por lo general, debe llamar a `Horizon::auth` en el método `boot` de su `AppServiceProvider`:
> > Horizon exposes a dashboard at `/horizon`. By default, you will only be able to access this dashboard in the `local` environment. To define a more specific access policy for the dashboard, you should use the `Horizon::auth` method. The `auth` method accepts a callback which should return `true` or `false`, indicating whether the user should have access to the Horizon dashboard. Typically, you should call `Horizon::auth` in the `boot` method of your `AppServiceProvider`:

    Horizon::auth(function ($request) {
        // return true / false;
    });

<a name="running-horizon"></a>
## Ejecutando Horizon : Running Horizon

Una vez que haya configurado sus trabajadores en el archivo de configuración `config/horizon.php`, puede iniciar Horizon usando el comando` horizon` Artisan. Este único comando iniciará todos sus trabajadores configurados:
> > Once you have configured your workers in the `config/horizon.php` configuration file, you may start Horizon using the `horizon` Artisan command. This single command will start all of your configured workers:

    php artisan horizon

Puede pausar el proceso Horizon e indicarle que continúe procesando trabajos usando los comandos Artisan `horizon:pause` y `horizon:continue`:
> > You may pause the Horizon process and instruct it to continue processing jobs using the `horizon:pause` and `horizon:continue` Artisan commands:

    php artisan horizon:pause

    php artisan horizon:continue

Puede finalizar con gracia el proceso maestro de Horizon en su máquina utilizando el comando Artesano `horizon:terminate`. Todos los trabajos que Horizon está procesando actualmente se completarán y luego Horizon saldrá:
> > You may gracefully terminate the master Horizon process on your machine using the `horizon:terminate` Artisan command. Any jobs that Horizon is currently processing will be completed and then Horizon will exit:

    php artisan horizon:terminate

<a name="deploying-horizon"></a>
### Implementando Horizon : Deploying Horizon

Si está implementando Horizon en un servidor activo, debe configurar un monitor de proceso para monitorear el comando `php artisan horizon` y reiniciarlo si se cierra inesperadamente. Al implementar código nuevo en su servidor, deberá indicarle al proceso principal de Horizon que finalice para que el monitor de procesos pueda reiniciarlo y recibir los cambios en su código.
> > If you are deploying Horizon to a live server, you should configure a process monitor to monitor the `php artisan horizon` command and restart it if it quits unexpectedly. When deploying fresh code to your server, you will need to instruct the master Horizon process to terminate so it can be restarted by your process monitor and receive your code changes.

#### Configuración del supervisor : Supervisor Configuration

Si está utilizando el supervisor de proceso Supervisor para administrar su proceso `horizon`, el siguiente archivo de configuración debería ser suficiente:
> > If you are using the Supervisor process monitor to manage your `horizon` process, the following configuration file should suffice:

    [program:horizon]
    process_name=%(program_name)s
    command=php /home/forge/app.com/artisan horizon
    autostart=true
    autorestart=true
    user=forge
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/horizon.log

> {tip} Si no se siente cómodo administrando sus propios servidores, considere usar [Laravel Forge](https://forge.laravel.com). Forge aprovisiona servidores PHP 7+ con todo lo que necesita para ejecutar aplicaciones modernas y robustas de Laravel con Horizon.
> > > {tip} If you are uncomfortable managing your own servers, consider using [Laravel Forge](https://forge.laravel.com). Forge provisions PHP 7+ servers with everything you need to run modern, robust Laravel applications with Horizon.

<a name="tags"></a>
## Etiquetas : Tags

Horizon le permite asignar "etiquetas" a trabajos, incluidos elementos almacenables, difusiones de eventos, notificaciones y escuchas de eventos en cola. De hecho, Horizon etiquetará de manera inteligente y automática la mayoría de los trabajos dependiendo de los modelos Eloquent que están adjuntos al trabajo. Por ejemplo, eche un vistazo al siguiente trabajo:
> > Horizon allows you to assign “tags” to jobs, including mailables, event broadcasts, notifications, and queued event listeners. In fact, Horizon will intelligently and automatically tag most jobs depending on the Eloquent models that are attached to the job. For example, take a look at the following job:

    <?php

    namespace App\Jobs;

    use App\Video;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;

    class RenderVideo implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * The video instance.
         *
         * @var \App\Video
         */
        public $video;

        /**
         * Create a new job instance.
         *
         * @param  \App\Video  $video
         * @return void
         */
        public function __construct(Video $video)
        {
            $this->video = $video;
        }

        /**
         * Execute the job.
         *
         * @return void
         */
        public function handle()
        {
            //
        }
    }

Si este trabajo se pone en cola con una instancia de `App\Video` que tiene un `id` de `1`, recibirá automáticamente la etiqueta `App\Video:1`. Esto se debe a que Horizon examinará las propiedades del trabajo para cualquier modelo Eloquent. Si se encuentran modelos Eloquent, Horizon etiquetará inteligentemente el trabajo utilizando el nombre de clase y la clave principal del modelo:
> > If this job is queued with an `App\Video` instance that has an `id` of `1`, it will automatically receive the tag `App\Video:1`. This is because Horizon will examine the job's properties for any Eloquent models. If Eloquent models are found, Horizon will intelligently tag the job using the model's class name and primary key:

    $video = App\Video::find(1);

    App\Jobs\RenderVideo::dispatch($video);

#### Etiquetado manual : Manually Tagging

Si desea definir manualmente las etiquetas para uno de sus objetos que se pueden poner en cola, puede definir un método `tags` en la clase:
> > If you would like to manually define the tags for one of your queueable objects, you may define a `tags` method on the class:

    class RenderVideo implements ShouldQueue
    {
        /**
         * Get the tags that should be assigned to the job.
         *
         * @return array
         */
        public function tags()
        {
            return ['render', 'video:'.$this->video->id];
        }
    }

<a name="notifications"></a>
## Notificaciones : Notifications

> **Nota:** Antes de usar notificaciones, debe agregar el paquete Composer 'guzzlehttp/guzzle` a su proyecto. Al configurar Horizon para enviar notificaciones por SMS, también debe consultar los [requisitos previos para el controlador de notificación de Nexmo](https://laravel.com/docs/5.6/notifications#sms-notifications).
> **Note:** Before using notifications, you should add the `guzzlehttp/guzzle` Composer package to your project. When configuring Horizon to send SMS notifications, you should also review the [prerequisites for the Nexmo notification driver](https://laravel.com/docs/5.6/notifications#sms-notifications).

Si desea recibir una notificación cuando una de sus colas tiene un tiempo de espera prolongado, puede utilizar los métodos `Horizon::routeMailNotificationsTo`, `Horizon::routeSlackNotificationsTo` y `Horizon::routeSmsNotificationsTo`. Puede llamar a estos métodos desde el `AppServiceProvider` de su aplicación:
> > If you would like to be notified when one of your queues has a long wait time, you may use the `Horizon::routeMailNotificationsTo`, `Horizon::routeSlackNotificationsTo`, and `Horizon::routeSmsNotificationsTo` methods. You may call these methods from your application's `AppServiceProvider`:

    Horizon::routeMailNotificationsTo('example@example.com');
    Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
    Horizon::routeSmsNotificationsTo('15556667777');

#### Configuración de umbrales de tiempo de espera de la notificación : Configuring Notification Wait Time Thresholds

Puede configurar cuántos segundos se consideran una "larga espera" dentro de su archivo de configuración `config/horizon.php`. La opción de configuración `wait` dentro de este archivo le permite controlar el umbral de espera largo para cada combinación de conexión / cola:
> > You may configure how many seconds are considered a "long wait" within your `config/horizon.php` configuration file. The `waits` configuration option within this file allows you to control the long wait threshold for each connection / queue combination:

    'waits' => [
        'redis:default' => 60,
    ],

<a name="metrics"></a>
## Métricas : Metrics

Horizon incluye un panel de indicadores que proporciona información sobre su trabajo y los tiempos de espera y el rendimiento de la cola. Para rellenar este panel, debe configurar el comando Artisan `snapshot` de Horizon para que se ejecute cada cinco minutos a través del [planificador](/docs/{{version}}/scheduling) de la aplicación:
> > Horizon includes a metrics dashboard which provides information on your job and queue wait times and throughput. In order to populate this dashboard, you should configure Horizon's `snapshot` Artisan command to run every five minutes via your application's [scheduler](/docs/{{version}}/scheduling):

    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('horizon:snapshot')->everyFiveMinutes();
    }
