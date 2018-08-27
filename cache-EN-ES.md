# Cache

- [Configuration](#configuration)
    - [Driver Prerequisites](#driver-prerequisites)
- [Cache Usage](#cache-usage)
    - [Obtaining A Cache Instance](#obtaining-a-cache-instance)
    - [Retrieving Items From The Cache](#retrieving-items-from-the-cache)
    - [Storing Items In The Cache](#storing-items-in-the-cache)
    - [Removing Items From The Cache](#removing-items-from-the-cache)
    - [Atomic Locks](#atomic-locks)
    - [The Cache Helper](#the-cache-helper)
- [Cache Tags](#cache-tags)
    - [Storing Tagged Cache Items](#storing-tagged-cache-items)
    - [Accessing Tagged Cache Items](#accessing-tagged-cache-items)
    - [Removing Tagged Cache Items](#removing-tagged-cache-items)
- [Adding Custom Cache Drivers](#adding-custom-cache-drivers)
    - [Writing The Driver](#writing-the-driver)
    - [Registering The Driver](#registering-the-driver)
- [Events](#events)

<a name="configuration"></a>
## Configuración : Configuration

Laravel proporciona una API expresiva y unificada para varios backends de almacenamiento en caché. La configuración de la memoria caché se encuentra en `config/cache.php`. En este archivo, puede especificar qué controlador de caché desea usar de manera predeterminada en su aplicación. Laravel admite backends de caché populares como [Memcached](https://memcached.org) y [Redis](https://redis.io) de fábrica.
> > Laravel provides an expressive, unified API for various caching backends. The cache configuration is located at `config/cache.php`. In this file you may specify which cache driver you would like to be used by default throughout your application. Laravel supports popular caching backends like [Memcached](https://memcached.org) and [Redis](https://redis.io) fuera de la caja.

El archivo de configuración de caché también contiene otras opciones, que están documentadas dentro del archivo, así que asegúrese de leer estas opciones. Por defecto, Laravel está configurado para usar el controlador de caché `file`, que almacena los objetos en caché serializados en el sistema de archivos. Para aplicaciones más grandes, se recomienda que use un controlador más robusto como Memcached o Redis. Incluso puede configurar múltiples configuraciones de caché para el mismo controlador.
> > The cache configuration file also contains various other options, which are documented within the file, so make sure to read over these options. By default, Laravel is configured to use the `file` cache driver, which stores the serialized, cached objects in the filesystem. For larger applications, it is recommended that you use a more robust driver such as Memcached or Redis. You may even configure multiple cache configurations for the same driver.

<a name="driver-prerequisites"></a>
### Prerrequisitos del Driver : Driver Prerequisites

#### Base de datos : Database

Cuando utilice el controlador de caché `database`, necesitará configurar una tabla para contener los elementos de caché. Encontrará un ejemplo de declaración `Schema` para la siguiente tabla:
> > When using the `database` cache driver, you will need to setup a table to contain the cache items. You'll find an example `Schema` declaration for the table below:

    Schema::create('cache', function ($table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });

> {tip} También puede usar el comando Artisan `php artisan cache:table` para generar una migración con el esquema apropiado.
> > > {tip} You may also use the `php artisan cache:table` Artisan command to generate a migration with the proper schema.

#### Memcached

El uso del controlador Memcached requiere la instalación del [paquete Memcached PECL](https://pecl.php.net/package/memcached). Puede listar todos sus servidores Memcached en el archivo de configuración `config/cache.php`:
> > Using the Memcached driver requires the [Memcached PECL package](https://pecl.php.net/package/memcached) to be installed. You may list all of your Memcached servers in the `config/cache.php` configuration file:

    'memcached' => [
        [
            'host' => '127.0.0.1',
            'port' => 11211,
            'weight' => 100
        ],
    ],

También puede establecer la opción `host` en una ruta de socket UNIX. Si haces esto, la opción `port` se debe establecer en `0`:
> > You may also set the `host` option to a UNIX socket path. If you do this, the `port` option should be set to `0`:

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],

#### Redis

Antes de usar un caché Redis con Laravel, deberá instalar el paquete `predis/predis` (~1.0) a través de Composer o instalar la extensión PhpRedis PHP a través de PECL.
> > Before using a Redis cache with Laravel, you will need to either install the `predis/predis` package (~1.0) via Composer or install the PhpRedis PHP extension via PECL.

Para obtener más información sobre la configuración de Redis, consulte su [página de documentación de Laravel](/docs/{{version}}/redis#configuration).
> > For more information on configuring Redis, consult its [Laravel documentation page](/docs/{{version}}/redis#configuration).

<a name="cache-usage"></a>
## Uso de caché : Cache Usage

<a name="obtaining-a-cache-instance"></a>
### Obtención de una instancia de caché : Obtaining A Cache Instance

Los `Illuminate\Contracts\Cache\Factory` y `Illuminate\Contracts\Cache\Repository` [contratos](/docs/{{version}}/contracts) proporcionan acceso a los servicios de caché de Laravel. El contrato `Factory` proporciona acceso a todos los controladores de caché definidos para su aplicación. El contrato `Repository` es típicamente una implementación del controlador de caché predeterminado para su aplicación según lo especificado por su archivo de configuración `cache`.
> > The `Illuminate\Contracts\Cache\Factory` and `Illuminate\Contracts\Cache\Repository` [contracts](/docs/{{version}}/contracts) provide access to Laravel's cache services. The `Factory` contract provides access to all cache drivers defined for your application. The `Repository` contract is typically an implementation of the default cache driver for your application as specified by your `cache` configuration file.

Sin embargo, también puede usar la fachada `Caché`, que es lo que usaremos a lo largo de esta documentación. La fachada `Cache` proporciona acceso conveniente y directo a las implementaciones subyacentes de los contratos de caché de Laravel:
> > However, you may also use the `Cache` facade, which is what we will use throughout this documentation. The `Cache` facade provides convenient, terse access to the underlying implementations of the Laravel cache contracts:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

#### Acceso a múltiples almacenes de caché : Accessing Multiple Cache Stores

Usando la fachada `Cache`, puede acceder a varias tiendas de caché mediante el método `store`. La clave pasada al método `store` debe corresponder a una de las tiendas listadas en la matriz de configuración `stores` en su archivo de configuración `cache`:
> > Using the `Cache` facade, you may access various cache stores via the `store` method. The key passed to the `store` method should correspond to one of the stores listed in the `stores` configuration array in your `cache` configuration file:

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 10);

<a name="retrieving-items-from-the-cache"></a>
### Recuperar elementos de la memoria caché : Retrieving Items From The Cache

El método `get` en la fachada `Cache` se utiliza para recuperar elementos de la memoria caché. Si el elemento no existe en la memoria caché, se devolverá `null`. Si lo desea, puede pasar un segundo argumento al método `get` que especifica el valor predeterminado que desea que se devuelva si el elemento no existe:
> > The `get` method on the `Cache` facade is used to retrieve items from the cache. If the item does not exist in the cache, `null` will be returned. If you wish, you may pass a second argument to the `get` method specifying the default value you wish to be returned if the item doesn't exist:

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');

Incluso puede pasar un `Closure` como valor predeterminado. El resultado de `Closure` se devolverá si el elemento especificado no existe en la memoria caché. Pasar un cierre le permite diferir la recuperación de valores predeterminados de una base de datos u otro servicio externo:
> > You may even pass a `Closure` as the default value. The result of the `Closure` will be returned if the specified item does not exist in the cache. Passing a Closure allows you to defer the retrieval of default values from a database or other external service:

    $value = Cache::get('key', function () {
        return DB::table(...)->get();
    });

#### Comprobación de la existencia del artículo : Checking For Item Existence

El método `has` se puede usar para determinar si un elemento existe en la memoria caché. Este método devolverá `false` si el valor es `null` o `false`:
> > The `has` method may be used to determine if an item exists in the cache. This method will return `false` if the value is `null` or `false`:

    if (Cache::has('key')) {
        //
    }

#### Incrementando / Decrementando Valores : Incrementing / Decrementing Values

Los métodos `increment` y` decrement` se pueden usar para ajustar el valor de los elementos enteros en la memoria caché. Ambos métodos aceptan un segundo argumento opcional que indica la cantidad por la cual incrementar o disminuir el valor del elemento:
> > The `increment` and `decrement` methods may be used to adjust the value of integer items in the cache. Both of these methods accept an optional second argument indicating the amount by which to increment or decrement the item's value:

    Cache::increment('key');
    Cache::increment('key', $amount);
    Cache::decrement('key');
    Cache::decrement('key', $amount);

#### Recuperar y almacenar : Retrieve & Store

En ocasiones, es posible que desee recuperar un elemento de la memoria caché, pero también almacenar un valor predeterminado si el elemento solicitado no existe. Por ejemplo, puede desear recuperar todos los usuarios del caché o, si no existen, recuperarlos de la base de datos y agregarlos al caché. Puedes hacer esto usando el método `Cache::remember`:
> > Sometimes you may wish to retrieve an item from the cache, but also store a default value if the requested item doesn't exist. For example, you may wish to retrieve all users from the cache or, if they don't exist, retrieve them from the database and add them to the cache. You may do this using the `Cache::remember` method:

    $value = Cache::remember('users', $minutes, function () {
        return DB::table('users')->get();
    });

Si el elemento no existe en la memoria caché, se ejecutará el `Closure` pasado al método `remember` y su resultado se colocará en la memoria caché.
> > If the item does not exist in the cache, the `Closure` passed to the `remember` method will be executed and its result will be placed in the cache.

Puede usar el método `rememberForever` para recuperar un elemento del caché o almacenarlo para siempre:
> > You may use the `rememberForever` method to retrieve an item from the cache or store it forever:

    $value = Cache::rememberForever('users', function() {
        return DB::table('users')->get();
    });

#### Recuperar y eliminar : Retrieve & Delete

Si necesita recuperar un elemento del caché y luego eliminarlo, puede usar el método `pull`. Al igual que el método `get`, `null` se devolverá si el elemento no existe en la memoria caché:
> > If you need to retrieve an item from the cache and then delete the item, you may use the `pull` method. Like the `get` method, `null` will be returned if the item does not exist in the cache:

    $value = Cache::pull('key');

<a name="storing-items-in-the-cache"></a>
### Almacenamiento de elementos en el caché : Storing Items In The Cache

Puede usar el método `put` en la fachada `Cache` para almacenar elementos en la caché. Cuando coloca un elemento en la memoria caché, necesita especificar el número de minutos para los cuales el valor debe almacenarse en caché:
> > You may use the `put` method on the `Cache` facade to store items in the cache. When you place an item in the cache, you need to specify the number of minutes for which the value should be cached:

    Cache::put('key', 'value', $minutes);

En lugar de pasar el número de minutos como un entero, también puede pasar una instancia de `DateTime` que representa el tiempo de caducidad del elemento en caché:
> > Instead of passing the number of minutes as an integer, you may also pass a `DateTime` instance representing the expiration time of the cached item:

    $expiresAt = now()->addMinutes(10);

    Cache::put('key', 'value', $expiresAt);

#### Tienda si no está presente : Store If Not Present

El método `add` solo agregará el elemento a la caché si aún no existe en la memoria caché. El método devolverá `true` si el elemento se agrega realmente al caché. De lo contrario, el método devolverá `false`:
> > The `add` method will only add the item to the cache if it does not already exist in the cache store. The method will return `true` if the item is actually added to the cache. Otherwise, the method will return `false`:

    Cache::add('key', 'value', $minutes);

#### Almacenamiento de elementos para siempre : Storing Items Forever

El método `forever` se puede usar para almacenar un elemento en la memoria caché de forma permanente. Como estos elementos no caducarán, se deben eliminar manualmente de la memoria caché utilizando el método `forget`:
> > The `forever` method may be used to store an item in the cache permanently. Since these items will not expire, they must be manually removed from the cache using the `forget` method:

    Cache::forever('key', 'value');

> {tip} Si está utilizando el controlador de Memcached, los elementos que están almacenados "para siempre" pueden eliminarse cuando el caché alcanza su límite de tamaño.
> > > {tip} If you are using the Memcached driver, items that are stored "forever" may be removed when the cache reaches its size limit.

<a name="removing-items-from-the-cache"></a>
### Eliminar elementos de la memoria caché : Removing Items From The Cache

Puede eliminar elementos del caché utilizando el método `forget`:
> > You may remove items from the cache using the `forget` method:

    Cache::forget('key');

Puede borrar todo el caché utilizando el método `flush`:
> > You may clear the entire cache using the `flush` method:

    Cache::flush();

> {note} La limpieza del caché no respeta el prefijo de caché y eliminará todas las entradas del caché. Considere esto cuidadosamente cuando borre un caché compartido por otras aplicaciones.
> > > {note} Flushing the cache does not respect the cache prefix and will remove all entries from the cache. Consider this carefully when clearing a cache which is shared by other applications.

<a name="atomic-locks"></a>
### Bloqueos atómicos : Atomic Locks

> {note} Para utilizar esta característica, su aplicación debe usar el controlador de caché `memcached` o `redis` como el controlador de caché predeterminado de su aplicación. Además, todos los servidores deben comunicarse con el mismo servidor de caché central.
> > > {note} To utilize this feature, your application must be using the `memcached` or `redis` cache driver as your application's default cache driver. In addition, all servers must be communicating with the same central cache server.

Los bloqueos atómicos permiten la manipulación de bloqueos distribuidos sin preocuparse por las condiciones de carrera. Por ejemplo, [Laravel Forge](https://forge.laravel.com) utiliza bloqueos atómicos para garantizar que solo se ejecute una tarea remota en un servidor a la vez. Puede crear y administrar bloqueos usando el método `Cache::lock`:
> > Atomic locks allow for the manipulation of distributed locks without worrying about race conditions. For example, [Laravel Forge](https://forge.laravel.com) uses atomic locks to ensure that only one remote task is being executed on a server at a time. You may create and manage locks using the `Cache::lock` method:

    if (Cache::lock('foo', 10)->get()) {
        // Lock acquired for 10 seconds...

        Cache::lock('foo')->release();
    }

El método `get` también acepta un Cierre. Después de que se ejecuta el Cierre, Laravel liberará automáticamente el bloqueo:
> > The `get` method also accepts a Closure. After the Closure is executed, Laravel will automatically release the lock:

    Cache::lock('foo')->get(function () {
        // Lock acquired indefinitely and automatically released...
    });

Si el bloqueo no está disponible en el momento en que lo solicita, puede indicar a Laravel que espere hasta que esté disponible:
> > If the lock is not available at the moment you request it, you may instruct Laravel to wait until it becomes available:

    if (Cache::lock('foo', 10)->block()) {
        // Lock acquired after waiting...
    }

Por defecto, el método `block` esperará indefinidamente hasta que el bloqueo esté disponible. Puede usar el método `blockFor` para esperar solo la cantidad de segundos especificada. Si no se puede adquirir el bloqueo dentro del límite de tiempo especificado, se lanzará un `Illuminate\Contracts\Cache\LockTimeoutException`:
> > By default, the `block` method will wait indefinitely until the lock is available. You may use the `blockFor` method to only wait for the specified number of seconds. If the lock can not be acquired within the specified time limit, an `Illuminate\Contracts\Cache\LockTimeoutException` will be thrown:

    if (Cache::lock('foo', 10)->blockFor(5)) {
        // Lock acquired after waiting maximum of 5 seconds...
    }

    Cache::lock('foo', 10)->blockFor(5, function () {
        // Lock acquired after waiting maximum of 5 seconds...
    });


<a name="the-cache-helper"></a>
### The Cache Helper

Además de usar la fachada `Caché` o [contrato de caché](/docs/{{versión}}/contracts), también puede usar la función `caché` global para recuperar y almacenar datos a través de la caché. Cuando se llama a la función `cache` con un solo argumento de cadena, devolverá el valor de la clave dada:
> > In addition to using the `Cache` facade or [cache contract](/docs/{{version}}/contracts), you may also use the global `cache` function to retrieve and store data via the cache. When the `cache` function is called with a single, string argument, it will return the value of the given key:

    $value = cache('key');

Si proporciona un array de pares de clave / valor y un tiempo de expiración a la función, almacenará los valores en la memoria caché durante la duración especificada:
> > If you provide an array of key / value pairs and an expiration time to the function, it will store values in the cache for the specified duration:

    cache(['key' => 'value'], $minutes);

    cache(['key' => 'value'], now()->addSeconds(10));

> {tip} Al realizar una llamada de prueba a la función global `cache`, puede usar el método `Cache::shouldReceive` como si estuviera [probando una fachada](/docs/{{version}}/mocking#mocking-facades).
> > > {tip} When testing call to the global `cache` function, you may use the `Cache::shouldReceive` method just as if you were [testing a facade](/docs/{{version}}/mocking#mocking-facades).

<a name="cache-tags"></a>
## Etiquetas de caché : Cache Tags

> {note} Las etiquetas de caché no son compatibles cuando se utilizan los controladores de caché `file` o `database`. Además, cuando se usan etiquetas múltiples con cachés que se almacenan "para siempre", el rendimiento será mejor con un controlador como "memcached", que purga automáticamente los registros obsoletos.
> > > {note} Cache tags are not supported when using the `file` or `database` cache drivers. Furthermore, when using multiple tags with caches that are stored "forever", performance will be best with a driver such as `memcached`, which automatically purges stale records.

<a name="storing-tagged-cache-items"></a>
### Almacenamiento de elementos de caché etiquetados : Storing Tagged Cache Items

Las etiquetas de caché le permiten etiquetar elementos relacionados en la memoria caché y luego eliminar todos los valores almacenados en caché a los que se les ha asignado una etiqueta determinada. Puede acceder a un caché etiquetado al pasar una matriz ordenada de nombres de etiqueta. Por ejemplo, accedamos a un caché etiquetado y al valor `put` en el caché:
> > Cache tags allow you to tag related items in the cache and then flush all cached values that have been assigned a given tag. You may access a tagged cache by passing in an ordered array of tag names. For example, let's access a tagged cache and `put` value in the cache:

    Cache::tags(['people', 'artists'])->put('John', $john, $minutes);

    Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);

<a name="accessing-tagged-cache-items"></a>
### Acceso a elementos de caché etiquetados : Accessing Tagged Cache Items

Para recuperar un elemento de caché etiquetado, pase la misma lista ordenada de etiquetas al método `tags` y luego llame al método `get` con la clave que desea recuperar:
> > To retrieve a tagged cache item, pass the same ordered list of tags to the `tags` method and then call the `get` method with the key you wish to retrieve:

    $john = Cache::tags(['people', 'artists'])->get('John');

    $anne = Cache::tags(['people', 'authors'])->get('Anne');

<a name="removing-tagged-cache-items"></a>
### Eliminar elementos de caché etiquetados : Removing Tagged Cache Items

Puede enjuagar todos los elementos a los que se les asigna una etiqueta o una lista de etiquetas. Por ejemplo, esta declaración eliminaría todos los cachés etiquetados con `people`, `authors` o ambos. Por lo tanto, tanto `Anne` como `John` se eliminarán de la memoria caché:
> > You may flush all items that are assigned a tag or list of tags. For example, this statement would remove all caches tagged with either `people`, `authors`, or both. So, both `Anne` and `John` would be removed from the cache:

    Cache::tags(['people', 'authors'])->flush();

Por el contrario, esta declaración eliminaría solo los cachés etiquetados con `authors`, por lo que `Anne` se eliminaría, pero no `John`:
> > In contrast, this statement would remove only caches tagged with `authors`, so `Anne` would be removed, but not `John`:

    Cache::tags('authors')->flush();

<a name="adding-custom-cache-drivers"></a>
## Agregar Drivers de caché personalizados : Adding Custom Cache Drivers

<a name="writing-the-driver"></a>
### Escribir el Driver : Writing The Driver

Para crear nuestro controlador de caché personalizado, primero debemos implementar `Illuminate\Contracts\Cache\Store` [contract](/docs/{{version}}/contracts). Entonces, una implementación de caché MongoDB se vería así:
> > To create our custom cache driver, we first need to implement the `Illuminate\Contracts\Cache\Store` [contract](/docs/{{version}}/contracts). So, a MongoDB cache implementation would look something like this:

    <?php

    namespace App\Extensions;

    use Illuminate\Contracts\Cache\Store;

    class MongoStore implements Store
    {
        public function get($key) {}
        public function many(array $keys);
        public function put($key, $value, $minutes) {}
        public function putMany(array $values, $minutes);
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}
        public function getPrefix() {}
    }

Solo necesitamos implementar cada uno de estos métodos usando una conexión MongoDB. Para ver un ejemplo de cómo implementar cada uno de estos métodos, eche un vistazo a `Illuminate\Cache\MemcachedStore` en el código fuente del framework. Una vez que se complete nuestra implementación, podemos finalizar nuestro registro de controlador personalizado.
> > We just need to implement each of these methods using a MongoDB connection. For an example of how to implement each of these methods, take a look at the `Illuminate\Cache\MemcachedStore` in the framework source code. Once our implementation is complete, we can finish our custom driver registration.

    Cache::extend('mongo', function ($app) {
        return Cache::repository(new MongoStore);
    });

> {tip} Si se pregunta dónde colocar su código de controlador de caché personalizado, puede crear un espacio de nombres `Extensions` en su directorio `app`. Sin embargo, tenga en cuenta que Laravel no tiene una estructura de aplicación rígida y que es libre de organizar su aplicación de acuerdo con sus preferencias.
> > > {tip} If you're wondering where to put your custom cache driver code, you could create an `Extensions` namespace within your `app` directory. However, keep in mind that Laravel does not have a rigid application structure and you are free to organize your application according to your preferences.

<a name="registering-the-driver"></a>
### Registrar el controlador : Registering The Driver

Para registrar el controlador de caché personalizado con Laravel, usaremos el método `extend` en la fachada `Cache`. La llamada a `Cache::extend` podría hacerse en el método `boot` de la `App\Providers\AppServiceProvider` predeterminada que se envía con las aplicaciones Laravel nuevas, o puede crear su propio proveedor de servicios para alojar la extensión. Simplemente don no olvide registrar el proveedor en la matriz de proveedores `config/app.php`:
> > To register the custom cache driver with Laravel, we will use the `extend` method on the `Cache` facade. The call to `Cache::extend` could be done in the `boot` method of the default `App\Providers\AppServiceProvider` that ships with fresh Laravel applications, or you may create your own service provider to house the extension - just don't forget to register the provider in the `config/app.php` provider array:

    <?php

    namespace App\Providers;

    use App\Extensions\MongoStore;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\ServiceProvider;

    class CacheServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Cache::extend('mongo', function ($app) {
                return Cache::repository(new MongoStore);
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

El primer argumento pasado al método `extend` es el nombre del controlador. Esto corresponderá a su opción `driver` en el archivo de configuración `config/cache.php`. El segundo argumento es un Cierre que debería devolver una instancia `Illuminate\Cache\Repository`. Al cierre se le pasará una instancia `$app`, que es una instancia del [contenedor de servicios](/docs/{{version}}/container).
> > The first argument passed to the `extend` method is the name of the driver. This will correspond to your `driver` option in the `config/cache.php` configuration file. The second argument is a Closure that should return an `Illuminate\Cache\Repository` instance. The Closure will be passed an `$app` instance, which is an instance of the [service container](/docs/{{version}}/container).

Una vez que se haya registrado su extensión, actualice la opción `driver` del archivo de configuración `config/cache.php` al nombre de su extensión.
> > Once your extension is registered, update your `config/cache.php` configuration file's `driver` option to the name of your extension.

<a name="events"></a>
## Eventos : Events

Para ejecutar código en cada operación de caché, puede escuchar los [eventos](/docs/{{version}}/events) activados por la caché. Normalmente, debe colocar estos oyentes de eventos dentro de su `EventServiceProvider`:
> > To execute code on every cache operation, you may listen for the [events](/docs/{{version}}/events) fired by the cache. Typically, you should place these event listeners within your `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Cache\Events\CacheHit' => [
            'App\Listeners\LogCacheHit',
        ],

        'Illuminate\Cache\Events\CacheMissed' => [
            'App\Listeners\LogCacheMissed',
        ],

        'Illuminate\Cache\Events\KeyForgotten' => [
            'App\Listeners\LogKeyForgotten',
        ],

        'Illuminate\Cache\Events\KeyWritten' => [
            'App\Listeners\LogKeyWritten',
        ],
    ];
