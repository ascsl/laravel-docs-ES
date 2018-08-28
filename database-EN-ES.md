# Base de datos: Primeros pasos : Database: Getting Started

- [Introduction](#introduction)
    - [Configuration](#configuration)
    - [Read & Write Connections](#read-and-write-connections)
    - [Using Multiple Database Connections](#using-multiple-database-connections)
- [Running Raw SQL Queries](#running-queries)
    - [Listening For Query Events](#listening-for-query-events)
- [Database Transactions](#database-transactions)

<a name="introduction"></a>
## Introducción : Introduction

Laravel hace que la interacción con bases de datos sea extremadamente simple en una variedad de bases de datos de bases de datos utilizando SQL sin procesar, el [generador de consultas fluido](/docs/{{version}}/queries), y [Eloquent ORM](/docs/{{version}}/eloquent). Actualmente, Laravel es compatible con cuatro bases de datos:
> > Laravel makes interacting with databases extremely simple across a variety of database backends using either raw SQL, the [fluent query builder](/docs/{{version}}/queries), and the [Eloquent ORM](/docs/{{version}}/eloquent). Currently, Laravel supports four databases:

<div class="content-list" markdown="1">
- MySQL
- PostgreSQL
- SQLite
- SQL Server
</div>

<a name="configuration"></a>
### Configuración : Configuration

La configuración de la base de datos para su aplicación se encuentra en `config/database.php`. En este archivo, puede definir todas las conexiones de su base de datos, así como especificar qué conexión se debe usar de forma predeterminada. Ejemplos de la mayoría de los sistemas de bases de datos compatibles se proporcionan en este archivo.
> > The database configuration for your application is located at `config/database.php`. In this file you may define all of your database connections, as well as specify which connection should be used by default. Examples for most of the supported database systems are provided in this file.

Por defecto, el ejemplo de Laravel [configuración del entorno](/docs/{{version}}/configuration#environment-configuration) está listo para usar con [Laravel Homestead](/docs/{{version}}/homestead), que es conveniente una máquina virtual para hacer el desarrollo de Laravel en su máquina local. Por supuesto, puede modificar esta configuración según sea necesario para su base de datos local.
> > By default, Laravel's sample [environment configuration](/docs/{{version}}/configuration#environment-configuration) is ready to use with [Laravel Homestead](/docs/{{version}}/homestead), which is a convenient virtual machine for doing Laravel development on your local machine. Of course, you are free to modify this configuration as needed for your local database.

#### Configuración SQLite : SQLite Configuration

Después de crear una nueva base de datos SQLite con un comando como `touch database/database.sqlite`, puede configurar fácilmente las variables de entorno para que apunten a esta base de datos recién creada utilizando la ruta absoluta de la base de datos:
> > After creating a new SQLite database using a command such as `touch database/database.sqlite`, you can easily configure your environment variables to point to this newly created database by using the database's absolute path:

    DB_CONNECTION=sqlite
    DB_DATABASE=/absolute/path/to/database.sqlite

<a name="read-and-write-connections"></a>
### Conexiones de lectura y escritura : Read & Write Connections

En ocasiones, es posible que desee utilizar una conexión de base de datos para las sentencias SELECT y otra para las instrucciones INSERT, UPDATE y DELETE. Laravel hace que esto sea muy sencillo, y las conexiones adecuadas siempre se usarán si está utilizando consultas sin formato, el generador de consultas o el ORM Eloquent.
> > Sometimes you may wish to use one database connection for SELECT statements, and another for INSERT, UPDATE, and DELETE statements. Laravel makes this a breeze, and the proper connections will always be used whether you are using raw queries, the query builder, or the Eloquent ORM.

Para ver cómo deben configurarse las conexiones de lectura / escritura, veamos este ejemplo:
> > To see how read / write connections should be configured, let's look at this example:

    'mysql' => [
        'read' => [
            'host' => ['192.168.1.1'],
        ],
        'write' => [
            'host' => ['196.168.1.2'],
        ],
        'sticky'    => true,
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset'   => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
        'prefix'    => '',
    ],

Tenga en cuenta que se han agregado tres claves al array de configuración: `read`, `write` y `sticky`. Las claves `read` y `write` tienen valores de array que contienen una sola clave: `host`. El resto de las opciones de base de datos para las conexiones `read` y `write` se fusionarán desde el array principal `mysql`.
> > Note that three keys have been added to the configuration array: `read`, `write` and `sticky`. The `read` and `write` keys have array values containing a single key: `host`. The rest of the database options for the `read` and `write` connections will be merged from the main `mysql` array.

Solo necesita colocar elementos en los arrays `read` y `write` si desea anular los valores del array principal. En este caso, `192.168.1.1` se usará como host para la conexión de "lectura", mientras que `192.168.1.2` se usará para la conexión de "escritura". Las credenciales de la base de datos, el prefijo, el conjunto de caracteres y todas las demás opciones del array principal `mysql` se compartirán en ambas conexiones.
> > You only need to place items in the `read` and `write` arrays if you wish to override the values from the main array. So, in this case, `192.168.1.1` will be used as the host for the "read" connection, while `192.168.1.2` will be used for the "write" connection. The database credentials, prefix, character set, and all other options in the main `mysql` array will be shared across both connections.

#### La opción `sticky` : The `sticky` Option

La opción `sticky` es un valor *opcional* que se puede usar para permitir la lectura inmediata de los registros que se han escrito en la base de datos durante el ciclo de solicitud actual. Si la opción `sticky` está habilitada y se ha realizado una operación de "escritura" contra la base de datos durante el ciclo de solicitud actual, cualquier otra operación de "lectura" utilizará la conexión de "escritura". Esto garantiza que cualquier dato escrito durante el ciclo de solicitud pueda leerse inmediatamente desde la base de datos durante la misma solicitud. Depende de usted decidir si este es el comportamiento deseado para su aplicación.
> > The `sticky` option is an *optional* value that can be used to allow the immediate reading of records that have been written to the database during the current request cycle. If the `sticky` option is enabled and a "write" operation has been performed against the database during the current request cycle, any further "read" operations will use the "write" connection. This ensures that any data written during the request cycle can be immediately read back from the database during that same request. It is up to you to decide if this is the desired behavior for your application.

<a name="using-multiple-database-connections"></a>
### Uso de múltiples conexiones de bases de datos : Using Multiple Database Connections

Al usar conexiones múltiples, puede acceder a cada conexión a través del método `connection` en la fachada `DB`. El `name` pasado al método `connection` debe corresponder a una de las conexiones listadas en su archivo de configuración `config/database.php`:
> > When using multiple connections, you may access each connection via the `connection` method on the `DB` facade. The `name` passed to the `connection` method should correspond to one of the connections listed in your `config/database.php` configuration file:

    $users = DB::connection('foo')->select(...);

También puede acceder a la instancia de PDO subyacente sin formato utilizando el método `getPdo` en una instancia de conexión:
> > You may also access the raw, underlying PDO instance using the `getPdo` method on a connection instance:

    $pdo = DB::connection()->getPdo();

<a name="running-queries"></a>
## Ejecución de consultas SQL sin procesar : Running Raw SQL Queries

Una vez que haya configurado su conexión de base de datos, puede ejecutar consultas utilizando la fachada `DB`. La fachada `DB` proporciona métodos para cada tipo de consulta: `select`, `update`, `insert`, `delete`, y `statement`.
> > Once you have configured your database connection, you may run queries using the `DB` facade. The `DB` facade provides methods for each type of query: `select`, `update`, `insert`, `delete`, and `statement`.

#### Ejecución de una consulta de selección : Running A Select Query

Para ejecutar una consulta básica, puede usar el método `select` en la fachada `DB`:
> > To run a basic query, you may use the `select` method on the `DB` facade:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::select('select * from users where active = ?', [1]);

            return view('user.index', ['users' => $users]);
        }
    }

El primer argumento que se pasa al método `select` es la consulta SQL sin formato, mientras que el segundo argumento es cualquier enlace de parámetro que deba vincularse a la consulta. Típicamente, estos son los valores de las restricciones de la cláusula `where`. El enlace de parámetros proporciona protección contra la inyección de SQL.
> > The first argument passed to the `select` method is the raw SQL query, while the second argument is any parameter bindings that need to be bound to the query. Typically, these are the values of the `where` clause constraints. Parameter binding provides protection against SQL injection.

El método `select` siempre devolverá un `array` de resultados. Cada resultado dentro del array será un objeto PHP `StdClass`, lo que le permitirá acceder a los valores de los resultados:
> > The `select` method will always return an `array` of results. Each result within the array will be a PHP `StdClass` object, allowing you to access the values of the results:

    foreach ($users as $user) {
        echo $user->name;
    }

#### Uso de enlaces con nombre : Using Named Bindings

En lugar de usar `?` para representar sus enlaces de parámetros, puede ejecutar una consulta utilizando enlaces con nombre:
> > Instead of using `?` to represent your parameter bindings, you may execute a query using named bindings:

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

#### Ejecución de una sentencia de inserción : Running An Insert Statement

Para ejecutar una instrucción `insert`, puede usar el método `insert` en la fachada `DB`. Como `select`, este método toma la consulta SQL sin procesar como primer argumento y los enlaces como segundo argumento:
> > To execute an `insert` statement, you may use the `insert` method on the `DB` facade. Like `select`, this method takes the raw SQL query as its first argument and bindings as its second argument:

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

#### Ejecución de una instrucción de actualización : Running An Update Statement

El método `update` se debe usar para actualizar los registros existentes en la base de datos. Se devolverá el número de filas afectadas por la declaración:
> > The `update` method should be used to update existing records in the database. The number of rows affected by the statement will be returned:

    $affected = DB::update('update users set votes = 100 where name = ?', ['John']);

#### Ejecución de una sentencia de eliminación : Running A Delete Statement

El método `delete` se debe usar para eliminar registros de la base de datos. Al igual que `update`, se devolverá el número de filas afectadas:
> > The `delete` method should be used to delete records from the database. Like `update`, the number of rows affected will be returned:

    $deleted = DB::delete('delete from users');

#### Ejecución de una sentencia general : Running A General Statement

Algunas sentencias de base de datos no devuelven ningún valor. Para este tipo de operaciones, puede usar el método `statement` en la fachada `DB`:
> > Some database statements do not return any value. For these types of operations, you may use the `statement` method on the `DB` facade:

    DB::statement('drop table users');

<a name="listening-for-query-events"></a>
### Escuchar eventos de consulta : Listening For Query Events

Si desea recibir cada consulta SQL ejecutada por su aplicación, puede usar el método `listen`. Este método es útil para registrar consultas o depuración. Puede registrar su oyente de consulta en un [proveedor de servicios] (/docs/{{version}}/providers):
> > If you would like to receive each SQL query executed by your application, you may use the `listen` method. This method is useful for logging queries or debugging. You may register your query listener in a [service provider](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\DB;
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
            DB::listen(function ($query) {
                // $query->sql
                // $query->bindings
                // $query->time
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

<a name="database-transactions"></a>
## Transacciones de base de datos : Database Transactions

Puede usar el método `transaction` en la fachada `DB` para ejecutar un conjunto de operaciones dentro de una transacción de base de datos. Si se lanza una excepción dentro de la transacción `Closure`, la transacción se retrotraerá automáticamente. Si el `Closure` se ejecuta con éxito, la transacción se confirmará automáticamente. No necesita preocuparse de deshacer manualmente o cometer mientras usa el método `transaction`:
> > You may use the `transaction` method on the `DB` facade to run a set of operations within a database transaction. If an exception is thrown within the transaction `Closure`, the transaction will automatically be rolled back. If the `Closure` executes successfully, the transaction will automatically be committed. You don't need to worry about manually rolling back or committing while using the `transaction` method:

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    });

#### Manejo de bloqueos : Handling Deadlocks

El método `transaction` acepta un segundo argumento opcional que define la cantidad de veces que una transacción debe ser reintentada cuando ocurre un bloqueo. Una vez que estos intentos se hayan agotado, se lanzará una excepción:
> > The `transaction` method accepts an optional second argument which defines the number of times a transaction should be reattempted when a deadlock occurs. Once these attempts have been exhausted, an exception will be thrown:

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    }, 5);

#### Uso manual de transacciones : Manually Using Transactions

Si desea comenzar una transacción de forma manual y tener control completo sobre las reversiones y confirmaciones, puede usar el método `beginTransaction` en la fachada `DB`:
> > If you would like to begin a transaction manually and have complete control over rollbacks and commits, you may use the `beginTransaction` method on the `DB` facade:

    DB::beginTransaction();

Puede deshacer la transacción mediante el método `rollBack`:
> > You can rollback the transaction via the `rollBack` method:

    DB::rollBack();

Por último, puede confirmar una transacción a través del método `commit`:
> > Lastly, you can commit a transaction via the `commit` method:

    DB::commit();

> {tip} Los métodos de transacción de la fachada `DB` controlan las transacciones para [query builder](/docs/{{version}}/queries) y [Eloquent ORM](/docs/{{version}}/eloquent).
> > > {tip} The `DB` facade's transaction methods control the transactions for both the [query builder](/docs/{{version}}/queries) and [Eloquent ORM](/docs/{{version}}/eloquent).
