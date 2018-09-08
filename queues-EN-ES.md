# Colas : Queues

- [Introduction](#introduction)
    - [Connections Vs. Queues](#connections-vs-queues)
    - [Driver Notes & Prerequisites](#driver-prerequisites)
- [Creating Jobs](#creating-jobs)
    - [Generating Job Classes](#generating-job-classes)
    - [Class Structure](#class-structure)
- [Dispatching Jobs](#dispatching-jobs)
    - [Delayed Dispatching](#delayed-dispatching)
    - [Job Chaining](#job-chaining)
    - [Customizing The Queue & Connection](#customizing-the-queue-and-connection)
    - [Specifying Max Job Attempts / Timeout Values](#max-job-attempts-and-timeout)
    - [Rate Limiting](#rate-limiting)
    - [Error Handling](#error-handling)
- [Running The Queue Worker](#running-the-queue-worker)
    - [Queue Priorities](#queue-priorities)
    - [Queue Workers & Deployment](#queue-workers-and-deployment)
    - [Job Expirations & Timeouts](#job-expirations-and-timeouts)
- [Supervisor Configuration](#supervisor-configuration)
- [Dealing With Failed Jobs](#dealing-with-failed-jobs)
    - [Cleaning Up After Failed Jobs](#cleaning-up-after-failed-jobs)
    - [Failed Job Events](#failed-job-events)
    - [Retrying Failed Jobs](#retrying-failed-jobs)
- [Job Events](#job-events)

<a name="introduction"></a>
## Introducción : Introduction

> {tip} Laravel ahora ofrece Horizon, un hermoso tablero y sistema de configuración para sus colas de Redis. Consulte la [documentación de Horizon](/docs/{{version}}/horizon) completa para obtener más información.
> > > {tip} Laravel now offers Horizon, a beautiful dashboard and configuration system for your Redis powered queues. Check out the full [Horizon documentation](/docs/{{version}}/horizon) for more information.

Las colas de Laravel proporcionan una API unificada en una variedad de backends de cola diferentes, como Beanstalk, Amazon SQS, Redis o incluso una base de datos relacional. Las colas le permiten diferir el procesamiento de una tarea que consume mucho tiempo, como enviar un correo electrónico, hasta un momento posterior. Aplazar estas tareas que consumen mucho tiempo acelera drásticamente las solicitudes web a su aplicación.
> > Laravel queues provide a unified API across a variety of different queue backends, such as Beanstalk, Amazon SQS, Redis, or even a relational database. Queues allow you to defer the processing of a time consuming task, such as sending an email, until a later time. Deferring these time consuming tasks drastically speeds up web requests to your application.

El archivo de configuración de la cola se almacena en `config/queue.php`. En este archivo encontrará configuraciones de conexión para cada uno de los controladores de cola que se incluyen con el marco, que incluye una base de datos, [Beanstalkd](https://kr.github.io/beanstalkd/), [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io) y un controlador síncrono que ejecutará trabajos inmediatamente (para uso local). También se incluye un controlador de cola `null` que descarta los trabajos en cola.
> > The queue configuration file is stored in `config/queue.php`. In this file you will find connection configurations for each of the queue drivers that are included with the framework, which includes a database, [Beanstalkd](https://kr.github.io/beanstalkd/), [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io),  and a synchronous driver that will execute jobs immediately (for local use). A `null` queue driver is also included which discards queued jobs.

<a name="connections-vs-queues"></a>
### Conexiones vs. Colas : Connections Vs. Queues

Antes de comenzar con las colas de Laravel, es importante comprender la distinción entre "conexiones" y "colas". En su archivo de configuración `config/queue.php`, hay una opción de configuración `connections`. Esta opción define una conexión particular a un servicio back-end como Amazon SQS, Beanstalk o Redis. Sin embargo, cualquier conexión de cola determinada puede tener múltiples "colas" que pueden considerarse como diferentes pilas o montones de trabajos en cola.
> > Before getting started with Laravel queues, it is important to understand the distinction between "connections" and "queues". In your `config/queue.php` configuration file, there is a `connections` configuration option. This option defines a particular connection to a backend service such as Amazon SQS, Beanstalk, or Redis. However, any given queue connection may have multiple "queues" which may be thought of as different stacks or piles of queued jobs.

Tenga en cuenta que cada ejemplo de configuración de conexión en el archivo de configuración `queue` contiene un atributo `queue`. Esta es la cola predeterminada a la que se enviarán los trabajos cuando se envíen a una conexión determinada. En otras palabras, si envía un trabajo sin definir explícitamente a qué cola debe enviarse, el trabajo se colocará en la cola definida en el atributo `queue` de la configuración de conexión:
> > Note that each connection configuration example in the `queue` configuration file contains a `queue` attribute. This is the default queue that jobs will be dispatched to when they are sent to a given connection. In other words, if you dispatch a job without explicitly defining which queue it should be dispatched to, the job will be placed on the queue that is defined in the `queue` attribute of the connection configuration:

    // This job is sent to the default queue...
    Job::dispatch();

    // This job is sent to the "emails" queue...
    Job::dispatch()->onQueue('emails');

Es posible que algunas aplicaciones no necesiten insertar trabajos en varias colas, sino que prefieran tener una cola simple. Sin embargo, enviar trabajos a varias colas puede ser especialmente útil para las aplicaciones que desean priorizar o segmentar cómo se procesan los trabajos, ya que el trabajador de cola de Laravel le permite especificar qué colas debe procesar por prioridad. Por ejemplo, si envía trabajos a una cola `high`, puede ejecutar un trabajador que les dé mayor prioridad de procesamiento:
> > Some applications may not need to ever push jobs onto multiple queues, instead preferring to have one simple queue. However, pushing jobs to multiple queues can be especially useful for applications that wish to prioritize or segment how jobs are processed, since the Laravel queue worker allows you to specify which queues it should process by priority. For example, if you push jobs to a `high` queue, you may run a worker that gives them higher processing priority:

    php artisan queue:work --queue=high,default

<a name="driver-prerequisites"></a>
### Notas del conductor y requisitos previos : Driver Notes & Prerequisites

#### Base de datos : Database

Para utilizar el controlador de cola `database`, necesitará una tabla de base de datos para contener los trabajos. Para generar una migración que crea esta tabla, ejecute el comando Artesanal `queue:table`. Una vez que se ha creado la migración, puede migrar su base de datos utilizando el comando `migrate`:
> > In order to use the `database` queue driver, you will need a database table to hold the jobs. To generate a migration that creates this table, run the `queue:table` Artisan command. Once the migration has been created, you may migrate your database using the `migrate` command:

    php artisan queue:table

    php artisan migrate

#### Redis

Para utilizar el controlador de cola `redis`, debe configurar una conexión de base de datos Redis en su archivo de configuración `config/database.php`.
> > In order to use the `redis` queue driver, you should configure a Redis database connection in your `config/database.php` configuration file.

**Redis Cluster**

Si su conexión de cola Redis utiliza un cluster Redis, sus nombres de cola deben contener una [etiqueta de clave hash](https://redis.io/topics/cluster-spec#keys-hash-tags). Esto es necesario para garantizar que todas las claves de Redis para una cola determinada se coloquen en la misma ranura hash:
> > If your Redis queue connection uses a Redis Cluster, your queue names must contain a [key hash tag](https://redis.io/topics/cluster-spec#keys-hash-tags). This is required in order to ensure all of the Redis keys for a given queue are placed into the same hash slot:

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => '{default}',
        'retry_after' => 90,
    ],

**Blocking**

Al usar la cola de Redis, puede usar la opción de configuración `block_for` para especificar cuánto tiempo debe esperar el controlador para que un trabajo esté disponible antes de recorrer el bucle de trabajo y volver a sondear la base de datos de Redis.
> > When using the Redis queue, you may use the `block_for` configuration option to specify how long the driver should wait for a job to become available before iterating through the worker loop and re-polling the Redis database.

Ajustar este valor en función de la carga de la cola puede ser más eficaz que consultar continuamente la base de datos Redis para nuevos trabajos. Por ejemplo, puede establecer el valor en `5` para indicar que el controlador debe bloquearse durante cinco segundos mientras espera que un trabajo esté disponible:
> > Adjusting this value based on your queue load can be more efficient than continually polling the Redis database for new jobs. For instance, you may set the value to `5` to indicate that the driver should block for five seconds while waiting for a job to become available:

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => 'default',
        'retry_after' => 90,
        'block_for' => 5,
    ],

> {note} Bloquear pop es una característica experimental. Existe una pequeña posibilidad de que un trabajo en cola se pierda si el servidor o el trabajador de Redis falla al mismo tiempo que se recupera el trabajo.
> > > {note} Blocking pop is an experimental feature. There is a small chance that a queued job could be lost if the Redis server or worker crashes at the same time the job is retrieved.

#### Otros requisitos previos de los Driver : Other Driver Prerequisites

Las siguientes dependencias son necesarias para los controladores de cola listados:
> > The following dependencies are needed for the listed queue drivers:

<div class="content-list" markdown="1">
- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~3.0`
- Redis: `predis/predis ~1.0`
</div>

<a name="creating-jobs"></a>
## Creación de trabajos : Creating Jobs

<a name="generating-job-classes"></a>
### Generación de clases de trabajo : Generating Job Classes

De forma predeterminada, todos los trabajos queables para su aplicación se almacenan en el directorio `app/Jobs`. Si el directorio `app/Jobs` no existe, se creará cuando ejecute el comando `make:job` Artisan. Puede generar un nuevo trabajo en cola utilizando la CLI de Artisan:
> > By default, all of the queueable jobs for your application are stored in the `app/Jobs` directory. If the `app/Jobs` directory doesn't exist, it will be created when you run the `make:job` Artisan command. You may generate a new queued job using the Artisan CLI:

    php artisan make:job ProcessPodcast

La clase generada implementará la interfaz `Illuminate\Contracts\Queue\ShouldQueue`, lo que indicará a Laravel que el trabajo debe colocarse en la cola para ejecutarse de forma asíncrona.
> > The generated class will implement the `Illuminate\Contracts\Queue\ShouldQueue` interface, indicating to Laravel that the job should be pushed onto the queue to run asynchronously.

<a name="class-structure"></a>
### Estructura de clase : Class Structure

Las clases de trabajo son muy simples, y normalmente solo contienen un método de "control" que se invoca cuando la cola procesa el trabajo. Para comenzar, echemos un vistazo a una clase de trabajo de ejemplo. En este ejemplo, pretendemos que administramos un servicio de publicación de podcasts y necesitamos procesar los archivos de podcast cargados antes de que se publiquen:
> > Job classes are very simple, normally containing only a `handle` method which is called when the job is processed by the queue. To get started, let's take a look at an example job class. In this example, we'll pretend we manage a podcast publishing service and need to process the uploaded podcast files before they are published:

    <?php

    namespace App\Jobs;

    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;

    class ProcessPodcast implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * Create a new job instance.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * Execute the job.
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Process uploaded podcast...
        }
    }

En este ejemplo, tenga en cuenta que pudimos pasar un [modelo elocuente](/docs/{{version}}/eloquent) directamente en el constructor del trabajo en cola. Debido al rasgo `SerializesModels` que está utilizando el trabajo, los modelos Eloquent se serializarán con gracia y se deserializarán cuando se procese el trabajo. Si su trabajo en cola acepta un modelo de Eloquent en su constructor, solo el identificador del modelo se serializará en la cola. Cuando el trabajo se gestiona realmente, el sistema de cola recuperará automáticamente la instancia del modelo completo de la base de datos. Todo es totalmente transparente para su aplicación y evita los problemas que pueden surgir de la serialización de las instancias de modelo Eloquent completas.
> > In this example, note that we were able to pass an [Eloquent model](/docs/{{version}}/eloquent) directly into the queued job's constructor. Because of the `SerializesModels` trait that the job is using, Eloquent models will be gracefully serialized and unserialized when the job is processing. If your queued job accepts an Eloquent model in its constructor, only the identifier for the model will be serialized onto the queue. When the job is actually handled, the queue system will automatically re-retrieve the full model instance from the database. It's all totally transparent to your application and prevents issues that can arise from serializing full Eloquent model instances.

El método `handle` se invoca cuando la cola procesa el trabajo. Tenga en cuenta que podemos escribir dependencias de sugerencia en el método `handle` del trabajo. Laravel [contenedor de servicios](/docs/{{version}}/container) inyecta automáticamente estas dependencias.
> > The `handle` method is called when the job is processed by the queue. Note that we are able to type-hint dependencies on the `handle` method of the job. The Laravel [service container](/docs/{{version}}/container) automatically injects these dependencies.

> {note} Los datos binarios, como el contenido de la imagen sin formato, se deben pasar a través de la función `base64_encode` antes de pasarlos a un trabajo en cola. De lo contrario, es posible que el trabajo no se serialice correctamente en JSON cuando se coloca en la cola.
> > > {note} Binary data, such as raw image contents, should be passed through the `base64_encode` function before being passed to a queued job. Otherwise, the job may not properly serialize to JSON when being placed on the queue.

<a name="dispatching-jobs"></a>
## Despacho de trabajos : Dispatching Jobs

Una vez que haya escrito su clase de trabajo, puede enviarla utilizando el método `dispatch` en el trabajo mismo. Los argumentos pasados ​​al método `dispatch` se darán al constructor del trabajo:
> > Once you have written your job class, you may dispatch it using the `dispatch` method on the job itself. The arguments passed to the `dispatch` method will be given to the job's constructor:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast);
        }
    }

<a name="delayed-dispatching"></a>
### Delayed Dispatching

Si desea retrasar la ejecución de un trabajo en cola, puede usar el método `delay` al despachar un trabajo. Por ejemplo, especifiquemos que un trabajo no debería estar disponible para su procesamiento hasta 10 minutos después de su envío:
> > If you would like to delay the execution of a queued job, you may use the `delay` method when dispatching a job. For example, let's specify that a job should not be available for processing until 10 minutes after it has been dispatched:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast)
                    ->delay(now()->addMinutes(10));
        }
    }

> {note} El servicio de cola de Amazon SQS tiene un tiempo de demora máximo de 15 minutos.
> > > {note} The Amazon SQS queue service has a maximum delay time of 15 minutes.

<a name="job-chaining"></a>
### Job Chaining

El encadenamiento de trabajos le permite especificar una lista de trabajos en cola que deben ejecutarse en secuencia. Si un trabajo en la secuencia falla, el resto de los trabajos no se ejecutarán. Para ejecutar una cadena de trabajos en cola, puede usar el método `withChain` en cualquiera de sus trabajos despachables:
> > Job chaining allows you to specify a list of queued jobs that should be run in sequence. If one job in the sequence fails, the rest of the jobs will not be run. To execute a queued job chain, you may use the `withChain` method on any of your dispatchable jobs:

    ProcessPodcast::withChain([
        new OptimizePodcast,
        new ReleasePodcast
    ])->dispatch();

#### Chain Connection & Queue

Si desea especificar la conexión y la cola predeterminadas que se deben usar para los trabajos encadenados, puede usar los métodos `allOnConnection` y `allOnQueue`. Estos métodos especifican la conexión en cola y el nombre de la cola que se deben usar a menos que el trabajo en cola tenga asignada explícitamente una conexión / cola diferente:
> > If you would like to specify the default connection and queue that should be used for the chained jobs, you may use the `allOnConnection` and `allOnQueue` methods. These methods specify the queue connection and queue name that should be used unless the queued job is explicitly assigned a different connection / queue:

    ProcessPodcast::withChain([
        new OptimizePodcast,
        new ReleasePodcast
    ])->dispatch()->allOnConnection('redis')->allOnQueue('podcasts');

<a name="customizing-the-queue-and-connection"></a>
### Personalizar la cola y la conexión : Customizing The Queue & Connection

#### Envío a una cola particular : Dispatching To A Particular Queue

Al enviar trabajos a diferentes colas, puede "categorizar" sus trabajos en cola e incluso priorizar la cantidad de trabajadores que asigna a varias colas. Tenga en cuenta que esto no envía trabajos a diferentes "conexiones" de cola, tal como lo define su archivo de configuración de cola, sino solo a colas específicas dentro de una única conexión. Para especificar la cola, use el método `onQueue` cuando envíe el trabajo:
> > By pushing jobs to different queues, you may "categorize" your queued jobs and even prioritize how many workers you assign to various queues. Keep in mind, this does not push jobs to different queue "connections" as defined by your queue configuration file, but only to specific queues within a single connection. To specify the queue, use the `onQueue` method when dispatching the job:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast)->onQueue('processing');
        }
    }

#### Envío a una conexión particular : Dispatching To A Particular Connection

Si está trabajando con múltiples conexiones de cola, puede especificar a qué conexión debe enviar un trabajo. Para especificar la conexión, use el método `onConnection` cuando envíe el trabajo:
> > If you are working with multiple queue connections, you may specify which connection to push a job to. To specify the connection, use the `onConnection` method when dispatching the job:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            ProcessPodcast::dispatch($podcast)->onConnection('sqs');
        }
    }

Por supuesto, puede encadenar los métodos `onConnection` y `onQueue` para especificar la conexión y la cola para un trabajo:
> > Of course, you may chain the `onConnection` and `onQueue` methods to specify the connection and the queue for a job:

    ProcessPodcast::dispatch($podcast)
                  ->onConnection('sqs')
                  ->onQueue('processing');

<a name="max-job-attempts-and-timeout"></a>
### Especificación de los valores máximos de intentos de trabajo / tiempo de espera : Specifying Max Job Attempts / Timeout Values

#### Intentos máximos : Max Attempts

Un enfoque para especificar el número máximo de veces que se puede intentar un trabajo a través del interruptor `--tries` en la línea de comando Artisan:
> > One approach to specifying the maximum number of times a job may be attempted is via the `--tries` switch on the Artisan command line:

    php artisan queue:work --tries=3

Sin embargo, puede tomar un enfoque más granular al definir la cantidad máxima de intentos en la clase de trabajo en sí. Si se especifica el número máximo de intentos en el trabajo, tendrá prioridad sobre el valor proporcionado en la línea de comando:
> > However, you may take a more granular approach by defining the maximum number of attempts on the job class itself. If the maximum number of attempts is specified on the job, it will take precedence over the value provided on the command line:

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * The number of times the job may be attempted.
         *
         * @var int
         */
        public $tries = 5;
    }

<a name="time-based-attempts"></a>
#### Intentos basados ​​en el tiempo : Time Based Attempts

Como alternativa a la definición de cuántas veces se puede intentar un trabajo antes de que falle, puede definir una hora en la que el trabajo debe expirar. Esto permite que un trabajo se intente varias veces dentro de un marco de tiempo determinado. Para definir la hora a la que un trabajo debe expirar, agregue un método `retryUntil` a su clase de trabajo:
> > As an alternative to defining how many times a job may be attempted before it fails, you may define a time at which the job should timeout. This allows a job to be attempted any number of times within a given time frame. To define the time at which a job should timeout, add a `retryUntil` method to your job class:

    /**
     * Determine the time at which the job should timeout.
     *
     * @return \DateTime
     */
    public function retryUntil()
    {
        return now()->addSeconds(5);
    }

> {tip} También puede definir un método `retryUntil` en sus escuchas de eventos en cola.
> > > {tip} You may also define a `retryUntil` method on your queued event listeners.

#### Se acabó el tiempo : Timeout

> {note} La función `timeout` está optimizada para PHP 7.1+ y la extensión PHP `pcntl`.
> > > {note} The `timeout` feature is optimized for PHP 7.1+ and the `pcntl` PHP extension.

Del mismo modo, la cantidad máxima de segundos que se pueden ejecutar trabajos se puede especificar utilizando el interruptor `--timeout` en la línea de comandos de Artisan:
> > Likewise, the maximum number of seconds that jobs can run may be specified using the `--timeout` switch on the Artisan command line:

    php artisan queue:work --timeout=30

Sin embargo, también puede definir la cantidad máxima de segundos que se debe permitir que un trabajo se ejecute en la clase de trabajo misma. Si el tiempo de espera se especifica en el trabajo, tendrá prioridad sobre cualquier tiempo de espera especificado en la línea de comando:
> > However, you may also define the maximum number of seconds a job should be allowed to run on the job class itself. If the timeout is specified on the job, it will take precedence over any timeout specified on the command line:

    <?php

    namespace App\Jobs;

    class ProcessPodcast implements ShouldQueue
    {
        /**
         * The number of seconds the job can run before timing out.
         *
         * @var int
         */
        public $timeout = 120;
    }

<a name="rate-limiting"></a>
### Límite de velocidad : Rate Limiting

> {note} Esta característica requiere que su aplicación pueda interactuar con un [servidor Redis](/docs/{{version}}/redis).
> > > {note} This feature requires that your application can interact with a [Redis server](/docs/{{version}}/redis).

Si su aplicación interactúa con Redis, puede acelerar sus trabajos en cola por tiempo o concurrencia. Esta función puede ser útil cuando tus trabajos en cola interactúen con API que también tienen un límite de velocidad. Por ejemplo, con el método `throttle`, puede estrangular un tipo determinado de trabajo para que se ejecute solo 10 veces cada 60 segundos. Si no se puede obtener un bloqueo, normalmente debe volver a liberar el trabajo en la cola para que pueda volver a intentarlo más adelante:
> > If your application interacts with Redis, you may throttle your queued jobs by time or concurrency. This feature can be of assistance when your queued jobs are interacting with APIs that are also rate limited. For example, using the `throttle` method, you may throttle a given type of job to only run 10 times every 60 seconds. If a lock can not be obtained, you should typically release the job back onto the queue so it can be retried later:

    Redis::throttle('key')->allow(10)->every(60)->then(function () {
        // Job logic...
    }, function () {
        // Could not obtain lock...

        return $this->release(10);
    });

> {tip} En el ejemplo anterior, la clave `key` puede ser cualquier cadena que identifique de manera única el tipo de trabajo al que le gustaría calificar el límite. Por ejemplo, es posible que desee construir la clave según el nombre de la clase del trabajo y los ID de los modelos Eloquent en los que opera.
> > > {tip} In the example above, the `key` may be any string that uniquely identifies the type of job you would like to rate limit. For example, you may wish to construct the key based on the class name of the job and the IDs of the Eloquent models it operates on.

Alternativamente, puede especificar la cantidad máxima de trabajadores que pueden procesar simultáneamente un trabajo determinado. Esto puede ser útil cuando un trabajo en cola está modificando un recurso que solo debería ser modificado por un trabajo a la vez. Por ejemplo, al usar el método `funnel`, puede limitar trabajos de un tipo dado para que solo sean procesados ​​por un trabajador a la vez:
> > Alternatively, you may specify the maximum number of workers that may simultaneously process a given job. This can be helpful when a queued job is modifying a resource that should only be modified by one job at a time. For example, using the `funnel` method, you may limit jobs of a given type to only be processed by one worker at a time:

    Redis::funnel('key')->limit(1)->then(function () {
        // Job logic...
    }, function () {
        // Could not obtain lock...

        return $this->release(10);
    });

> {tip} Cuando se usa la limitación de velocidad, es difícil determinar la cantidad de intentos que su trabajo deberá ejecutar con éxito. Por lo tanto, es útil combinar la limitación de velocidad con [intentos basados ​​en el tiempo](#time-based-attempts).
> > > {tip} When using rate limiting, the number of attempts your job will need to run successfully can be hard to determine. Therefore, it is useful to combine rate limiting with [time based attempts](#time-based-attempts).

<a name="error-handling"></a>
### Manejo de errores : Error Handling

Si se lanza una excepción mientras se está procesando el trabajo, el trabajo se volverá a liberar automáticamente en la cola para que se pueda intentar de nuevo. El trabajo continuará siendo liberado hasta que se haya intentado la cantidad máxima de veces permitida por su aplicación. El número máximo de intentos se define mediante el modificador `--tries` utilizado en el comando Artisan `queue:work`. Alternativamente, la cantidad máxima de intentos puede definirse en la clase de trabajo misma. Más información sobre cómo ejecutar el trabajador de cola [se puede encontrar a continuación](#running-the-queue-worker).
> > If an exception is thrown while the job is being processed, the job will automatically be released back onto the queue so it may be attempted again. The job will continue to be released until it has been attempted the maximum number of times allowed by your application. The maximum number of attempts is defined by the `--tries` switch used on the `queue:work` Artisan command. Alternatively, the maximum number of attempts may be defined on the job class itself. More information on running the queue worker [can be found below](#running-the-queue-worker).

<a name="running-the-queue-worker"></a>
## Running The Queue Worker

Laravel incluye un trabajador de cola que procesará los nuevos trabajos a medida que se colocan en la cola. Puede ejecutar al trabajador utilizando el comando Artisan `queue:work`. Tenga en cuenta que una vez que el comando `queue:work` ha comenzado, continuará ejecutándose hasta que se detenga manualmente o cierre su terminal:
> > Laravel includes a queue worker that will process new jobs as they are pushed onto the queue. You may run the worker using the `queue:work` Artisan command. Note that once the `queue:work` command has started, it will continue to run until it is manually stopped or you close your terminal:

    php artisan queue:work

> {tip} Para mantener el proceso `queue:work` ejecutándose permanentemente en segundo plano, debe usar un monitor de proceso como [Supervisor](#supervisor-configuration) para asegurarse de que el trabajador de colas no deje de ejecutarse.
> > > {tip} To keep the `queue:work` process running permanently in the background, you should use a process monitor such as [Supervisor](#supervisor-configuration) to ensure that the queue worker does not stop running.

Recuerde, los trabajadores de cola son procesos de larga duración y almacenan el estado de la aplicación iniciada en la memoria. Como resultado, no notarán cambios en su base de código una vez que se hayan iniciado. Por lo tanto, durante el proceso de implementación, asegúrese de [reiniciar los trabajadores de la cola](#queue-workers-and-deployment).
> > Remember, queue workers are long-lived processes and store the booted application state in memory. As a result, they will not notice changes in your code base after they have been started. So, during your deployment process, be sure to [restart your queue workers](#queue-workers-and-deployment).

#### Procesamiento de un solo trabajo : Processing A Single Job

La opción `--once` se puede usar para indicar al trabajador que solo procese un único trabajo de la cola:
> > The `--once` option may be used to instruct the worker to only process a single job from the queue:

    php artisan queue:work --once

#### Especificando la conexión y la cola : Specifying The Connection & Queue

También puede especificar qué conexión de cola debe utilizar el trabajador. El nombre de conexión pasado al comando `work` debe corresponder a una de las conexiones definidas en su archivo de configuración `config/queue.php`:
> > You may also specify which queue connection the worker should utilize. The connection name passed to the `work` command should correspond to one of the connections defined in your `config/queue.php` configuration file:

    php artisan queue:work redis

Puede personalizar aún más su cola de trabajo solo procesando colas particulares para una conexión determinada. Por ejemplo, si todos sus correos electrónicos se procesan en una cola `emails` en su conexión de cola `redis`, puede emitir el siguiente comando para iniciar un trabajador que solo procesa esa cola:
> > You may customize your queue worker even further by only processing particular queues for a given connection. For example, if all of your emails are processed in an `emails` queue on your `redis` queue connection, you may issue the following command to start a worker that only processes only that queue:

    php artisan queue:work redis --queue=emails

#### Consideraciones de recursos : Resource Considerations

Los trabajadores de cola de Daemon no "reinician" el framework antes de procesar cada trabajo. Por lo tanto, debe liberar todos los recursos pesados ​​después de cada trabajo. Por ejemplo, si está haciendo manipulación de imágenes con la biblioteca GD, debería liberar la memoria con `imagedestroy` cuando haya terminado.
> > Daemon queue workers do not "reboot" the framework before processing each job. Therefore, you should free any heavy resources after each job completes. For example, if you are doing image manipulation with the GD library, you should free the memory with `imagedestroy` when you are done.

<a name="queue-priorities"></a>
### Prioridades de cola : Queue Priorities

A veces puede querer priorizar cómo se procesan sus colas. Por ejemplo, en su `config/queue.php` puede establecer `queue` predeterminada para su conexión `redis` a `low`. Sin embargo, de vez en cuando es posible que desee enviar un trabajo a una cola de prioridad `high` como esta:
> > Sometimes you may wish to prioritize how your queues are processed. For example, in your `config/queue.php` you may set the default `queue` for your `redis` connection to `low`. However, occasionally you may wish to push a job to a `high` priority queue like so:

    dispatch((new Job)->onQueue('high'));

Para iniciar un trabajador que verifica que todos los trabajos de cola `high` se procesen antes de continuar con cualquier trabajo en la cola `low`, pase una lista delimitada por comas de nombres de cola al comando `work`:
> > To start a worker that verifies that all of the `high` queue jobs are processed before continuing to any jobs on the `low` queue, pass a comma-delimited list of queue names to the `work` command:

    php artisan queue:work --queue=high,low

<a name="queue-workers-and-deployment"></a>
### Queue Workers & Deployment

Como los trabajadores de cola son procesos de larga duración, no recibirán cambios en su código sin reiniciarse. Por lo tanto, la forma más sencilla de implementar una aplicación utilizando los trabajadores de cola es reiniciar los trabajadores durante el proceso de implementación. Puede reiniciar graciosamente a todos los trabajadores emitiendo el comando `queue:restart`:
> > Since queue workers are long-lived processes, they will not pick up changes to your code without being restarted. So, the simplest way to deploy an application using queue workers is to restart the workers during your deployment process. You may gracefully restart all of the workers by issuing the `queue:restart` command:

    php artisan queue:restart

Este comando indicará a todos los trabajadores de cola que "mueran" elegantemente después de que terminen de procesar su trabajo actual para que no se pierdan trabajos existentes. Como los trabajadores de la cola morirán cuando se ejecuta el comando `queue:restart`, debe ejecutar un administrador de procesos como [Supervisor](#supervisor-configuración) para reiniciar automáticamente los trabajadores de la cola.
> > This command will instruct all queue workers to gracefully "die" after they finish processing their current job so that no existing jobs are lost. Since the queue workers will die when the `queue:restart` command is executed, you should be running a process manager such as [Supervisor](#supervisor-configuration) to automatically restart the queue workers.

> {tip} La cola usa [caché](/docs/{{version}}/cache) para almacenar señales de reinicio, por lo que debe verificar que el controlador de caché esté configurado correctamente para su aplicación antes de usar esta característica.
> > > {tip} The queue uses the [cache](/docs/{{version}}/cache) to store restart signals, so you should verify a cache driver is properly configured for your application before using this feature.

<a name="job-expirations-and-timeouts"></a>
### Expiración de trabajos y tiempos de espera : Job Expirations & Timeouts

#### Expiración de trabajos : Job Expiration

En su archivo de configuración `config/queue.php`, cada conexión de cola define una opción `retry_after`. Esta opción especifica cuántos segundos debe esperar la conexión de cola antes de reintentar un trabajo que se está procesando. Por ejemplo, si el valor de `retry_after` se establece en `90`, el trabajo se volverá a publicar en la cola si se ha procesado durante 90 segundos sin haber sido eliminado. Por lo general, debe establecer el valor `retry_after` en la cantidad máxima de segundos que sus trabajos deberían razonablemente tomar para completar el procesamiento.
> > In your `config/queue.php` configuration file, each queue connection defines a `retry_after` option. This option specifies how many seconds the queue connection should wait before retrying a job that is being processed. For example, if the value of `retry_after` is set to `90`, the job will be released back onto the queue if it has been processing for 90 seconds without being deleted. Typically, you should set the `retry_after` value to the maximum number of seconds your jobs should reasonably take to complete processing.

> {note} La única conexión de cola que no contiene un valor `retry_after` es Amazon SQS. SQS volverá a intentar el trabajo según el [Tiempo de espera de visibilidad predeterminado](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html) que se administra dentro de la consola de AWS.
> > > {note} The only queue connection which does not contain a `retry_after` value is Amazon SQS. SQS will retry the job based on the [Default Visibility Timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html) which is managed within the AWS console.

#### Worker Timeouts

El comando Artisan `queue:work` expone una opción `--timeout`. La opción `--timeout` especifica cuánto tiempo esperará el proceso maestro de cola de Laravel antes de matar a un trabajador de cola hijo que está procesando un trabajo. A veces, un proceso de cola hijo puede "congelarse" por varias razones, como una llamada HTTP externa que no responde. La opción `--timeout` elimina los procesos inmovilizados que han excedido el límite de tiempo especificado:
> > The `queue:work` Artisan command exposes a `--timeout` option. The `--timeout` option specifies how long the Laravel queue master process will wait before killing off a child queue worker that is processing a job. Sometimes a child queue process can become "frozen" for various reasons, such as an external HTTP call that is not responding. The `--timeout` option removes frozen processes that have exceeded that specified time limit:

    php artisan queue:work --timeout=60

La opción de configuración `retry_after` y la opción `--timeout` de la CLI son diferentes, pero funcionan juntas para garantizar que no se pierdan trabajos y que los trabajos se procesen con éxito solo una vez.
> > The `retry_after` configuration option and the `--timeout` CLI option are different, but work together to ensure that jobs are not lost and that jobs are only successfully processed once.

> {note} El valor `--timeout` siempre debe ser al menos varios segundos más corto que su valor de configuración `retry_after`. Esto asegurará que un trabajador que procesa un trabajo determinado siempre muera antes de volver a intentarlo. Si su opción `--timeout` es más larga que su valor de configuración `retry_after`, sus trabajos pueden procesarse dos veces.
> > > {note} The `--timeout` value should always be at least several seconds shorter than your `retry_after` configuration value. This will ensure that a worker processing a given job is always killed before the job is retried. If your `--timeout` option is longer than your `retry_after` configuration value, your jobs may be processed twice.

#### Duración del sueño del trabajador : Worker Sleep Duration

Cuando hay trabajos disponibles en la cola, el trabajador seguirá procesando trabajos sin demora entre ellos. Sin embargo, la opción `sleep` determina cuánto tiempo (en segundos) "dormirá" el trabajador si no hay nuevos trabajos disponibles. Mientras duerme, el trabajador no procesará ningún trabajo nuevo; los trabajos se procesarán después de que el trabajador se despierte nuevamente.
> > When jobs are available on the queue, the worker will keep processing jobs with no delay in between them. However, the `sleep` option determines how long (in seconds) the worker will "sleep" if there are no new jobs available. While sleeping, the worker will not process any new jobs - the jobs will be processed after the worker wakes up again.

    php artisan queue:work --sleep=3

<a name="supervisor-configuration"></a>
## Configuración de Supervisor : Supervisor Configuration

#### Instalación de Supervisor : Installing Supervisor

Supervisor es un monitor de procesos para el sistema operativo Linux y reiniciará automáticamente su proceso `queue:work` si falla. Para instalar Supervisor en Ubuntu, puede usar el siguiente comando:
> > Supervisor is a process monitor for the Linux operating system, and will automatically restart your `queue:work` process if it fails. To install Supervisor on Ubuntu, you may use the following command:

    sudo apt-get install supervisor

> {tip} Si configurar Supervisor por su cuenta suena abrumador, considere usar [Laravel Forge](https://forge.laravel.com), que automáticamente instalará y configurará el Supervisor para sus proyectos Laravel.
> > > {tip} If configuring Supervisor yourself sounds overwhelming, consider using [Laravel Forge](https://forge.laravel.com), which will automatically install and configure Supervisor for your Laravel projects.

#### Configurando Supervisor : Configuring Supervisor

Los archivos de configuración de Supervisor normalmente se almacenan en el directorio `/etc/supervisor/conf.d`. Dentro de este directorio, puede crear cualquier cantidad de archivos de configuración que instruyan al supervisor sobre cómo deben monitorearse sus procesos. Por ejemplo, creemos un archivo `laravel-worker.conf` que inicia y supervisa un proceso `queue:work`:
> > Supervisor configuration files are typically stored in the `/etc/supervisor/conf.d` directory. Within this directory, you may create any number of configuration files that instruct supervisor how your processes should be monitored. For example, let's create a `laravel-worker.conf` file that starts and monitors a `queue:work` process:

    [program:laravel-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3
    autostart=true
    autorestart=true
    user=forge
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/worker.log

En este ejemplo, la directiva `numprocs` le indicará al supervisor que ejecute 8 procesos `queue: work` y que los supervise a todos, reiniciándolos automáticamente si fallan. Por supuesto, debe cambiar la parte `queue:work sqs` de la directiva `command` para reflejar su conexión de cola deseada.
> > In this example, the `numprocs` directive will instruct Supervisor to run 8 `queue:work` processes and monitor all of them, automatically restarting them if they fail. Of course, you should change the `queue:work sqs` portion of the `command` directive to reflect your desired queue connection.

#### Iniciando Supervisor : Starting Supervisor

Una vez que se haya creado el archivo de configuración, puede actualizar la configuración de Supervisor e iniciar los procesos utilizando los siguientes comandos:
> > Once the configuration file has been created, you may update the Supervisor configuration and start the processes using the following commands:

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start laravel-worker:*

Para obtener más información sobre Supervisor, consulte la [Documentación de Supervisor](http://supervisord.org/index.html).
> > For more information on Supervisor, consult the [Supervisor documentation](http://supervisord.org/index.html).

<a name="dealing-with-failed-jobs"></a>
## Tratando con trabajos fallidos : Dealing With Failed Jobs

Algunas veces sus trabajos en cola fallarán. ¡No te preocupes, las cosas no siempre salen según lo planeado! Laravel incluye una forma conveniente de especificar la cantidad máxima de intentos de trabajo. Después de que un trabajo haya excedido esta cantidad de intentos, se insertará en la tabla de la base de datos `failed_jobs`. Para crear una migración para la tabla `failed_jobs`, puede usar el comando `queue:failed-table`:
> > Sometimes your queued jobs will fail. Don't worry, things don't always go as planned! Laravel includes a convenient way to specify the maximum number of times a job should be attempted. After a job has exceeded this amount of attempts, it will be inserted into the `failed_jobs` database table. To create a migration for the `failed_jobs` table, you may use the `queue:failed-table` command:

    php artisan queue:failed-table

    php artisan migrate

Luego, cuando ejecuta su [queue worker](#running-the-queue-worker), debe especificar la cantidad máxima de veces que se intentará un trabajo utilizando el interruptor `--tries` en el comando `queue:work`. Si no especifica un valor para la opción `--tries`, los trabajos se intentarán indefinidamente:
> > Then, when running your [queue worker](#running-the-queue-worker), you should specify the maximum number of times a job should be attempted using the `--tries` switch on the `queue:work` command. If you do not specify a value for the `--tries` option, jobs will be attempted indefinitely:

    php artisan queue:work redis --tries=3

<a name="cleaning-up-after-failed-jobs"></a>
### Limpieza después de trabajos fallidos : Cleaning Up After Failed Jobs

Puede definir un método `failed` directamente en su clase de trabajo, lo que le permite realizar tareas específicas de limpieza cuando ocurre una falla. Esta es la ubicación perfecta para enviar una alerta a sus usuarios o revertir cualquier acción realizada por el trabajo. La `Excepción` que provocó la falla del trabajo se pasará al método `failed`:
> > You may define a `failed` method directly on your job class, allowing you to perform job specific clean-up when a failure occurs. This is the perfect location to send an alert to your users or revert any actions performed by the job. The `Exception` that caused the job to fail will be passed to the `failed` method:

    <?php

    namespace App\Jobs;

    use Exception;
    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class ProcessPodcast implements ShouldQueue
    {
        use InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * Create a new job instance.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * Execute the job.
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Process uploaded podcast...
        }

        /**
         * The job failed to process.
         *
         * @param  Exception  $exception
         * @return void
         */
        public function failed(Exception $exception)
        {
            // Send user notification of failure, etc...
        }
    }

<a name="failed-job-events"></a>
### Eventos de trabajos fallidos : Failed Job Events

Si desea registrar un evento que será invocado cuando falla un trabajo, puede usar el método `Queue::failing`. Este evento es una gran oportunidad para notificar a su equipo por correo electrónico o [Stride](https://www.stride.com). Por ejemplo, podemos adjuntar una devolución de llamada a este evento desde `AppServiceProvider` que se incluye con Laravel:
> > If you would like to register an event that will be called when a job fails, you may use the `Queue::failing` method. This event is a great opportunity to notify your team via email or [Stride](https://www.stride.com). For example, we may attach a callback to this event from the `AppServiceProvider` that is included with Laravel:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Queue\Events\JobFailed;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Queue::failing(function (JobFailed $event) {
                // $event->connectionName
                // $event->job
                // $event->exception
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="retrying-failed-jobs"></a>
### Reintentar trabajos fallidos : Retrying Failed Jobs

Para ver todos sus trabajos fallidos que se han insertado en su tabla de base de datos `failed_jobs`, puede usar el comando Artisan `queue:failed`:
> > To view all of your failed jobs that have been inserted into your `failed_jobs` database table, you may use the `queue:failed` Artisan command:

    php artisan queue:failed

El comando `queue:failed` mostrará la identificación del trabajo, la conexión, la cola y el tiempo de fallo. La ID del trabajo se puede usar para volver a intentar el trabajo fallido. Por ejemplo, para volver a intentar un trabajo fallido que tiene una ID de `5`, emita el siguiente comando:
> > The `queue:failed` command will list the job ID, connection, queue, and failure time. The job ID may be used to retry the failed job. For instance, to retry a failed job that has an ID of `5`, issue the following command:

    php artisan queue:retry 5

Para volver a intentar todos sus trabajos fallidos, ejecute el comando `queue:retry` y pase `all` como ID:
> > To retry all of your failed jobs, execute the `queue:retry` command and pass `all` as the ID:

    php artisan queue:retry all

Si desea eliminar un trabajo fallido, puede usar el comando `queue:forget`:
> > If you would like to delete a failed job, you may use the `queue:forget` command:

    php artisan queue:forget 5

Para eliminar todos sus trabajos fallidos, puede usar el comando `queue:flush`:
> > To delete all of your failed jobs, you may use the `queue:flush` command:

    php artisan queue:flush

<a name="job-events"></a>
## Eventos de trabajo : Job Events

Usando los métodos `before` y `after` en `Queue` [facade](/docs/{{version}}/facades), puede especificar que los callbacks se ejecuten antes o después de que se procese un trabajo en cola. Estas devoluciones de llamada son una gran oportunidad para realizar estadísticas de incremento o registro adicionales para un tablero de instrumentos. Por lo general, debe llamar a estos métodos desde un [proveedor de servicios](/docs/{{version}}/providers). Por ejemplo, podemos usar el `AppServiceProvider` que se incluye con Laravel:
> > Using the `before` and `after` methods on the `Queue` [facade](/docs/{{version}}/facades), you may specify callbacks to be executed before or after a queued job is processed. These callbacks are a great opportunity to perform additional logging or increment statistics for a dashboard. Typically, you should call these methods from a [service provider](/docs/{{version}}/providers). For example, we may use the `AppServiceProvider` that is included with Laravel:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobProcessed;
    use Illuminate\Queue\Events\JobProcessing;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Queue::before(function (JobProcessing $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });

            Queue::after(function (JobProcessed $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

Usando el método `looping` en `Queue` [facade](/docs/{{version}}/facades), puede especificar devoluciones de llamada que se ejecuten antes de que el trabajador intente recuperar un trabajo de una cola. Por ejemplo, puede registrar un Cierre para deshacer cualquier transacción que haya quedado abierta por un trabajo que falló anteriormente:
> > Using the `looping` method on the `Queue` [facade](/docs/{{version}}/facades), you may specify callbacks that execute before the worker attempts to fetch a job from a queue. For example, you might register a Closure to rollback any transactions that were left open by a previously failed job:

    Queue::looping(function () {
        while (DB::transactionLevel() > 0) {
            DB::rollBack();
        }
    });
