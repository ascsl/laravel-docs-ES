# Redis

- [Introduction](#introduction)
    - [Configuration](#configuration)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [Interacting With Redis](#interacting-with-redis)
    - [Pipelining Commands](#pipelining-commands)
- [Pub / Sub](#pubsub)

<a name="introduction"></a>
## Introducción : Introduction

[Redis](https://redis.io) es un almacén de clave-valor avanzado de código abierto. A menudo se lo denomina servidor de estructura de datos ya que las claves pueden contener [cadenas](https://redis.io/topics/data-types#strings), [hashes](https://redis.io/topics/data)-types#hashes), [listas](https://redis.io/topics/data-types#lists), [sets](https://redis.io/topics/data-types#sets), y [conjuntos ordenados](https://redis.io/topics/data-types#sorted-sets).
> > [Redis](https://redis.io) is an open source, advanced key-value store. It is often referred to as a data structure server since keys can contain [strings](https://redis.io/topics/data-types#strings), [hashes](https://redis.io/topics/data-types#hashes), [lists](https://redis.io/topics/data-types#lists), [sets](https://redis.io/topics/data-types#sets), and [sorted sets](https://redis.io/topics/data-types#sorted-sets).

Antes de usar Redis con Laravel, deberá instalar el paquete `predis/predis` mediante Composer:
> > Before using Redis with Laravel, you will need to install the `predis/predis` package via Composer:

    composer require predis/predis

Alternativamente, puede instalar la extensión PHP [PhpRedis](https://github.com/phpredis/phpredis) a través de PECL. La extensión es más compleja de instalar, pero puede ofrecer un mejor rendimiento para las aplicaciones que hacen un uso intensivo de Redis.
> > Alternatively, you may install the [PhpRedis](https://github.com/phpredis/phpredis) PHP extension via PECL. The extension is more complex to install but may yield better performance for applications that make heavy use of Redis.

<a name="configuration"></a>
### Configuración : Configuration

La configuración de Redis para su aplicación se encuentra en el archivo de configuración `config/database.php`. Dentro de este archivo, verá un array `redis` que contiene los servidores Redis utilizados por su aplicación:
> > The Redis configuration for your application is located in the `config/database.php` configuration file. Within this file, you will see a `redis` array containing the Redis servers utilized by your application:

    'redis' => [

        'client' => 'predis',

        'default' => [
            'host' => env('REDIS_HOST', 'localhost'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => 0,
        ],

    ],

La configuración del servidor por defecto debería ser suficiente para el desarrollo. Sin embargo, puede modificar este array según su entorno. Cada servidor Redis definido en su archivo de configuración debe tener un nombre, un host y un puerto.
> > The default server configuration should suffice for development. However, you are free to modify this array based on your environment. Each Redis server defined in your configuration file is required to have a name, host, and port.

#### Configuración de clústeres : Configuring Clusters

Si su aplicación está utilizando un clúster de servidores Redis, debe definir estos clústeres dentro de una clave `clusters` de su configuración de Redis:
> > If your application is utilizing a cluster of Redis servers, you should define these clusters within a `clusters` key of your Redis configuration:

    'redis' => [

        'client' => 'predis',

        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', 'localhost'),
                    'password' => env('REDIS_PASSWORD', null),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],

    ],

Por defecto, los clústeres realizarán la división del lado del cliente en sus nodos, lo que le permite agrupar nodos y crear una gran cantidad de RAM disponible. Sin embargo, tenga en cuenta que la fragmentación del lado del cliente no gestiona la conmutación por error; por lo tanto, es principalmente adecuado para datos en caché que están disponibles en otro almacén de datos primario. Si desea utilizar el agrupamiento de Redis nativo, debe especificarlo en la clave `options` de su configuración de Redis:
> > By default, clusters will perform client-side sharding across your nodes, allowing you to pool nodes and create a large amount of available RAM. However, note that client-side sharding does not handle failover; therefore, is primarily suited for cached data that is available from another primary data store. If you would like to use native Redis clustering, you should specify this in the `options` key of your Redis configuration:

    'redis' => [

        'client' => 'predis',

        'options' => [
            'cluster' => 'redis',
        ],

        'clusters' => [
            // ...
        ],

    ],

<a name="predis"></a>
### Predis

Además de las opciones predeterminadas de configuración del servidor `host`, `port`, `database` y` password`, Predis admite [parámetros de conexión](https://github.com/nrk/predis/wiki/Connection-Parameters) adicionales que se pueden definir para cada uno de sus servidores Redis. Para utilizar estas opciones de configuración adicionales, agréguelos a la configuración de su servidor Redis en el archivo de configuración `config/database.php`:
> > In addition to the default `host`, `port`, `database`, and `password` server configuration options, Predis supports additional [connection parameters](https://github.com/nrk/predis/wiki/Connection-Parameters) that may be defined for each of your Redis servers. To utilize these additional configuration options, add them to your Redis server configuration in the `config/database.php` configuration file:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_write_timeout' => 60,
    ],

<a name="phpredis"></a>
### PhpRedis

> {note} Si tiene la extensión PhpRedis PHP instalada a través de PECL, necesitará cambiar el nombre del alias `Redis` en su archivo de configuración `config/app.php`.
> > > {note} If you have the PhpRedis PHP extension installed via PECL, you will need to rename the `Redis` alias in your `config/app.php` configuration file.

Para utilizar la extensión PhpRedis, debe cambiar la opción `client` de su configuración de Redis a `phpredis`. Esta opción se encuentra en su archivo de configuración `config/database.php`:
> > To utilize the PhpRedis extension, you should change the `client` option of your Redis configuration to `phpredis`. This option is found in your `config/database.php` configuration file:

    'redis' => [

        'client' => 'phpredis',

        // Rest of Redis configuration...
    ],

Además de las opciones predeterminadas de configuración del servidor `host`, `port`, `database` y` password`, PhpRedis admite los siguientes parámetros de conexión adicionales: `persistent`, `prefix`, `read_timeout` y` timeout`. Puede agregar cualquiera de estas opciones a la configuración de su servidor Redis en el archivo de configuración `config/database.php`:
> > In addition to the default `host`, `port`, `database`, and `password` server configuration options, PhpRedis supports the following additional connection parameters: `persistent`, `prefix`, `read_timeout` and `timeout`. You may add any of these options to your Redis server configuration in the `config/database.php` configuration file:

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_timeout' => 60,
    ],

<a name="interacting-with-redis"></a>
## Interactuando con Redis : Interacting With Redis

Puede interactuar con Redis llamando a varios métodos en la [fachada](/docs/{{version}}/facades) `Redis`. La fachada `Redis` admite métodos dinámicos, lo que significa que puede llamar a cualquier [comando Redis](https://redis.io/commands) en la fachada y el comando se pasará directamente a Redis. En este ejemplo, llamaremos al comando `GET` de Redis llamando al método `get` en la fachada `Redis`:
> > You may interact with Redis by calling various methods on the `Redis` [facade](/docs/{{version}}/facades). The `Redis` facade supports dynamic methods, meaning you may call any [Redis command](https://redis.io/commands) on the facade and the command will be passed directly to Redis. In this example, we will call the Redis `GET` command by calling the `get` method on the `Redis` facade:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Redis;

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
            $user = Redis::get('user:profile:'.$id);

            return view('user.profile', ['user' => $user]);
        }
    }

Por supuesto, como se mencionó anteriormente, puede llamar a cualquiera de los comandos de Redis en la fachada `Redis`. Laravel usa métodos mágicos para pasar los comandos al servidor Redis, así que pase los argumentos que el comando Redis espera:
> > Of course, as mentioned above, you may call any of the Redis commands on the `Redis` facade. Laravel uses magic methods to pass the commands to the Redis server, so pass the arguments the Redis command expects:

    Redis::set('name', 'Taylor');

    $values = Redis::lrange('names', 5, 10);

Alternativamente, también puede pasar comandos al servidor utilizando el método `command`, que acepta el nombre del comando como su primer argumento, y una matriz de valores como su segundo argumento:
> > Alternatively, you may also pass commands to the server using the `command` method, which accepts the name of the command as its first argument, and an array of values as its second argument:

    $values = Redis::command('lrange', ['name', 5, 10]);

#### Uso de varias conexiones Redis : Using Multiple Redis Connections

Puede obtener una instancia de Redis llamando al método `Redis::connection`:
> > You may get a Redis instance by calling the `Redis::connection` method:

    $redis = Redis::connection();

This will give you an instance of the default Redis server. You may also pass the connection or cluster name to the `connection` method to get a specific server or cluster as defined in your Redis configuration:
> > This will give you an instance of the default Redis server. You may also pass the connection or cluster name to the `connection` method to get a specific server or cluster as defined in your Redis configuration:

    $redis = Redis::connection('my-connection');

<a name="pipelining-commands"></a>
### Pipelining Commands

La canalización debe usarse cuando necesite enviar muchos comandos al servidor en una sola operación. El método `pipeline` acepta un argumento: un `Closure` que recibe una instancia de Redis. Puede emitir todos sus comandos a esta instancia de Redis y se ejecutarán en una única operación:
> > Pipelining should be used when you need to send many commands to the server in one operation. The `pipeline` method accepts one argument: a `Closure` that receives a Redis instance. You may issue all of your commands to this Redis instance and they will all be executed within a single operation:

    Redis::pipeline(function ($pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>
## Pub / Sub

Laravel proporciona una interfaz conveniente para los comandos `publish` y `subscribe` de Redis. Estos comandos Redis le permiten escuchar mensajes en un "canal" dado. Puede publicar mensajes en el canal desde otra aplicación, o incluso utilizando otro lenguaje de programación, lo que permite una comunicación sencilla entre aplicaciones y procesos.
> > Laravel provides a convenient interface to the Redis `publish` and `subscribe` commands. These Redis commands allow you to listen for messages on a given "channel". You may publish messages to the channel from another application, or even using another programming language, allowing easy communication between applications and processes.

Primero, configuremos un oyente de canal usando el método `suscribe`. Colocaremos esta llamada al método dentro de un [comando Artisan](/docs/{{version}}/artisan) ya que llamar al método `subscribe` comienza un proceso de larga ejecución:
> > First, let's setup a channel listener using the `subscribe` method. We'll place this method call within an [Artisan command](/docs/{{version}}/artisan) since calling the `subscribe` method begins a long-running process:

    <?php

    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Support\Facades\Redis;

    class RedisSubscribe extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Subscribe to a Redis channel';

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            Redis::subscribe(['test-channel'], function ($message) {
                echo $message;
            });
        }
    }

Ahora podemos publicar mensajes en el canal usando el método `publish`:
> > Now we may publish messages to the channel using the `publish` method:

    Route::get('publish', function () {
        // Route logic...

        Redis::publish('test-channel', json_encode(['foo' => 'bar']));
    });

#### Suscripciones de comodines : Wildcard Subscriptions

Usando el método `psubscribe`, puede suscribirse a un canal comodín, que puede ser útil para capturar todos los mensajes en todos los canales. El nombre `$channel` se pasará como segundo argumento para el callback `Closure`:
> > Using the `psubscribe` method, you may subscribe to a wildcard channel, which may be useful for catching all messages on all channels. The `$channel` name will be passed as the second argument to the provided callback `Closure`:

    Redis::psubscribe(['*'], function ($message, $channel) {
        echo $message;
    });

    Redis::psubscribe(['users.*'], function ($message, $channel) {
        echo $message;
    });
