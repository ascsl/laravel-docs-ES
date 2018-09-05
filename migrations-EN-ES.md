# Base de datos: migraciones : Database: Migrations

- [Introduction](#introduction)
- [Generating Migrations](#generating-migrations)
- [Migration Structure](#migration-structure)
- [Running Migrations](#running-migrations)
    - [Rolling Back Migrations](#rolling-back-migrations)
- [Tables](#tables)
    - [Creating Tables](#creating-tables)
    - [Renaming / Dropping Tables](#renaming-and-dropping-tables)
- [Columns](#columns)
    - [Creating Columns](#creating-columns)
    - [Column Modifiers](#column-modifiers)
    - [Modifying Columns](#modifying-columns)
    - [Dropping Columns](#dropping-columns)
- [Indexes](#indexes)
    - [Creating Indexes](#creating-indexes)
    - [Renaming Indexes](#renaming-indexes)
    - [Dropping Indexes](#dropping-indexes)
    - [Foreign Key Constraints](#foreign-key-constraints)

<a name="introduction"></a>
## Introducción : Introduction

Las migraciones son como el control de versiones para su base de datos, lo que le permite a su equipo modificar y compartir fácilmente el esquema de la base de datos de la aplicación. Las migraciones suelen combinarse con el generador de esquemas de Laravel para crear fácilmente el esquema de la base de datos de su aplicación. Si alguna vez ha tenido que decirle a un compañero de equipo que agregue manualmente una columna a su esquema de base de datos local, se ha enfrentado al problema que las migraciones de la base de datos resuelven.
> > Migrations are like version control for your database, allowing your team to easily modify and share the application's database schema. Migrations are typically paired with Laravel's schema builder to easily build your application's database schema. If you have ever had to tell a teammate to manually add a column to their local database schema, you've faced the problem that database migrations solve.

La [fachada](/docs/{{version}}/facades) `Schema` de Laravel proporciona soporte de base de datos independiente para crear y manipular tablas en todos los sistemas de bases de datos admitidos por Laravel.
> > The Laravel `Schema` [facade](/docs/{{version}}/facades) provides database agnostic support for creating and manipulating tables across all of Laravel's supported database systems.

<a name="generating-migrations"></a>
## Generación de migraciones : Generating Migrations

Para crear una migración, use el [comando Artisan](/docs/{{version}}/artisan) `make:migration`:
> > To create a migration, use the `make:migration` [Artisan command](/docs/{{version}}/artisan):

    php artisan make:migration create_users_table

La nueva migración se colocará en su directorio `database/migrations`. Cada nombre de archivo de migración contiene una marca de tiempo que le permite a Laravel determinar el orden de las migraciones.
> > The new migration will be placed in your `database/migrations` directory. Each migration file name contains a timestamp which allows Laravel to determine the order of the migrations.

Las opciones `--table` y `--create` también se pueden usar para indicar el nombre de la tabla y si la migración creará una nueva tabla. Estas opciones completan previamente el archivo de resguardo de migración generado con la tabla especificada:
> > The `--table` and `--create` options may also be used to indicate the name of the table and whether the migration will be creating a new table. These options pre-fill the generated migration stub file with the specified table:

    php artisan make:migration create_users_table --create=users

    php artisan make:migration add_votes_to_users_table --table=users

Si desea especificar una ruta de salida personalizada para la migración generada, puede usar la opción `--path` al ejecutar el comando `make:migration`. La ruta dada debe ser relativa a la ruta base de su aplicación.
> > If you would like to specify a custom output path for the generated migration, you may use the `--path` option when executing the `make:migration` command. The given path should be relative to your application's base path.

<a name="migration-structure"></a>
## Estructura de migración : Migration Structure

Una clase de migración contiene dos métodos: `up` y `down`. El método `up` se usa para agregar nuevas tablas, columnas o índices a su base de datos, mientras que el método `down` debe invertir las operaciones realizadas por el método `up`.
> > A migration class contains two methods: `up` and `down`. The `up` method is used to add new tables, columns, or indexes to your database, while the `down` method should reverse the operations performed by the `up` method.

Dentro de estos dos métodos, puede usar el generador de esquemas de Laravel para crear y modificar tablas expresivamente. Para conocer todos los métodos disponibles en el constructor `Schema`, [consulte su documentación](#creating-tables). Por ejemplo, este ejemplo de migración crea una tabla `flights`:
> > Within both of these methods you may use the Laravel schema builder to expressively create and modify tables. To learn about all of the methods available on the `Schema` builder, [check out its documentation](#creating-tables). For example, this migration example creates a `flights` table:

    <?php

    use Illuminate\Support\Facades\Schema;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;

    class CreateFlightsTable extends Migration
    {
        /**
         * Run the migrations.
         *
         * @return void
         */
        public function up()
        {
            Schema::create('flights', function (Blueprint $table) {
                $table->increments('id');
                $table->string('name');
                $table->string('airline');
                $table->timestamps();
            });
        }

        /**
         * Reverse the migrations.
         *
         * @return void
         */
        public function down()
        {
            Schema::drop('flights');
        }
    }

<a name="running-migrations"></a>
## Ejecución de migraciones : Running Migrations

Para ejecutar todas sus migraciones pendientes, ejecute el comando Artisan `migrate`:
> > To run all of your outstanding migrations, execute the `migrate` Artisan command:

    php artisan migrate

> {note} Si está utilizando la [máquina virtual Homestead](/docs/{{version}}/homestead), debe ejecutar este comando desde su máquina virtual.
> > > {note} If you are using the [Homestead virtual machine](/docs/{{version}}/homestead), you should run this command from within your virtual machine.

#### Forzar migraciones para ejecutar en producción : Forcing Migrations To Run In Production

Algunas operaciones de migración son destructivas, lo que significa que pueden hacer que pierdas datos. Para protegerlo de ejecutar estos comandos en su base de datos de producción, se le pedirá una confirmación antes de que se ejecuten los comandos. Para forzar que los comandos se ejecuten sin un aviso, use el indicador `--force`:
> > Some migration operations are destructive, which means they may cause you to lose data. In order to protect you from running these commands against your production database, you will be prompted for confirmation before the commands are executed. To force the commands to run without a prompt, use the `--force` flag:

    php artisan migrate --force

<a name="rolling-back-migrations"></a>
### Revertir migraciones : Rolling Back Migrations

Para deshacer la última operación de migración, puede usar el comando `rollback`. Este comando revierte el último "lote" de migraciones, que puede incluir varios archivos de migración:
> > To rollback the latest migration operation, you may use the `rollback` command. This command rolls back the last "batch" of migrations, which may include multiple migration files:

    php artisan migrate:rollback

Puede revertir un número limitado de migraciones al proporcionar la opción `step` al comando `rollback`. Por ejemplo, el siguiente comando revertirá las últimas cinco migraciones:
> > You may rollback a limited number of migrations by providing the `step` option to the `rollback` command. For example, the following command will rollback the last five migrations:

    php artisan migrate:rollback --step=5

El comando `migrate:reset` hará retroceder todas las migraciones de su aplicación:
> > The `migrate:reset` command will roll back all of your application's migrations:

    php artisan migrate:reset

#### Revertir y migrar en un solo comando : Rollback & Migrate In Single Command

El comando `migrate:refresh` revertirá todas sus migraciones y luego ejecutará el comando `migrate`. Este comando recrea de manera efectiva toda su base de datos:
> > The `migrate:refresh` command will roll back all of your migrations and then execute the `migrate` command. This command effectively re-creates your entire database:

    php artisan migrate:refresh

    // Refresh the database and run all database seeds...
    php artisan migrate:refresh --seed

Puede retrotraer y volver a migrar un número limitado de migraciones proporcionando la opción `step` al comando `refresh`. Por ejemplo, el siguiente comando revertirá y volverá a migrar las últimas cinco migraciones:
> > You may rollback & re-migrate a limited number of migrations by providing the `step` option to the `refresh` command. For example, the following command will rollback & re-migrate the last five migrations:

    php artisan migrate:refresh --step=5

#### Eliminar todas las tablas y migrar : Drop All Tables & Migrate

El comando `migrate:fresh` eliminara todas las tablas de la base de datos y luego ejecutará el comando `migrate`:
> > The `migrate:fresh` command will drop all tables from the database and then execute the `migrate` command:

    php artisan migrate:fresh

    php artisan migrate:fresh --seed

<a name="tables"></a>
## Tablas : Tables

<a name="creating-tables"></a>
### Creación de tablas : Creating Tables

Para crear una nueva tabla de base de datos, use el método `create` en la fachada `Schema`. El método `create` acepta dos argumentos. El primero es el nombre de la tabla, mientras que el segundo es un `Closure` que recibe un objeto `Blueprint` que puede usarse para definir la nueva tabla:
> > To create a new database table, use the `create` method on the `Schema` facade. The `create` method accepts two arguments. The first is the name of the table, while the second is a `Closure` which receives a `Blueprint` object that may be used to define the new table:

    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
    });

Por supuesto, al crear la tabla, puede usar cualquiera de los [métodos de columnas](#creating-columns) del constructor del esquema para definir las columnas de la tabla.
> > Of course, when creating the table, you may use any of the schema builder's [column methods] to define the table's columns.

#### Comprobación de la existencia de tabla / columna : Checking For Table / Column Existence

Puede verificar fácilmente la existencia de una tabla o columna utilizando los métodos `hasTable` y `hasColumn`:
> > You may easily check for the existence of a table or column using the `hasTable` and `hasColumn` methods:

    if (Schema::hasTable('users')) {
        //
    }

    if (Schema::hasColumn('users', 'email')) {
        //
    }

#### Opciones de tabla y conexión de base de datos : Database Connection & Table Options

Si desea realizar una operación de esquema en una conexión de base de datos que no sea su conexión predeterminada, use el método `connection`:
> > If you want to perform a schema operation on a database connection that is not your default connection, use the `connection` method:

    Schema::connection('foo')->create('users', function (Blueprint $table) {
        $table->increments('id');
    });

Puede usar los siguientes comandos en el generador de esquemas para definir las opciones de la tabla:
> > You may use the following commands on the schema builder to define the table's options:

Command  |  Descripción  |  Description
-------  |  -----------  |  -----------
`$table->engine = 'InnoDB';`  |  Especifique el motor de almacenamiento de tablas (MySQL).  |  Specify the table storage engine (MySQL).
`$table->charset = 'utf8';`  |  Especifique un juego de caracteres predeterminado para la tabla (MySQL).  |  Specify a default character set for the table (MySQL).
`$table->collation = 'utf8_unicode_ci';`  |  Especificar una intercalación predeterminada para la tabla (MySQL).  |  Specify a default collation for the table (MySQL).
`$table->temporary();`  |  Crear una tabla temporal (excepto SQL Server).  |  Create a temporary table (except SQL Server).

<a name="renaming-and-dropping-tables"></a>
### Renombrar / soltar tablas : Renaming / Dropping Tables

Para cambiar el nombre de una tabla de base de datos existente, use el método `rename`:
> > To rename an existing database table, use the `rename` method:

    Schema::rename($from, $to);

Para eliminar una tabla existente, puede usar los métodos `drop` o `dropIfExists`:
> > To drop an existing table, you may use the `drop` or `dropIfExists` methods:

    Schema::drop('users');

    Schema::dropIfExists('users');

#### Cambiar el nombre de las tablas con claves externas : Renaming Tables With Foreign Keys

Antes de renombrar una tabla, debe verificar que cualquier restricción de clave externa en la tabla tenga un nombre explícito en sus archivos de migración en lugar de dejar que Laravel asigne un nombre basado en la convención. De lo contrario, el nombre de la restricción de clave externa se referirá al nombre de la tabla anterior.
> > Before renaming a table, you should verify that any foreign key constraints on the table have an explicit name in your migration files instead of letting Laravel assign a convention based name. Otherwise, the foreign key constraint name will refer to the old table name.

<a name="columns"></a>
## Columnas : Columns

<a name="creating-columns"></a>
### Creación de columnas : Creating Columns

El método `table` de la fachada `Schema` se puede usar para actualizar las tablas existentes. Al igual que el método `create`, el método `table` acepta dos argumentos: el nombre de la tabla y un `Closure` que recibe una instancia `Blueprint` que puede usar para agregar columnas a la tabla:
> > The `table` method on the `Schema` facade may be used to update existing tables. Like the `create` method, the `table` method accepts two arguments: the name of the table and a `Closure` that receives a `Blueprint` instance you may use to add columns to the table:

    Schema::table('users', function (Blueprint $table) {
        $table->string('email');
    });

#### Tipos de columnas disponibles : Available Column Types

Por supuesto, el generador de esquemas contiene una variedad de tipos de columnas que puede especificar al crear sus tablas:
> > Of course, the schema builder contains a variety of column types that you may specify when building your tables:

Command  |  Description
-------  |  -----------
`$table->bigIncrements('id');`  |  Auto-incrementing UNSIGNED BIGINT (primary key) equivalent column.
`$table->bigInteger('votes');`  |  BIGINT equivalent column.
`$table->binary('data');`  |  BLOB equivalent column.
`$table->boolean('confirmed');`  |  BOOLEAN equivalent column.
`$table->char('name', 100);`  |  CHAR equivalent column with an optional length.
`$table->date('created_at');`  |  DATE equivalent column.
`$table->dateTime('created_at');`  |  DATETIME equivalent column.
`$table->dateTimeTz('created_at');`  |  DATETIME (with timezone) equivalent column.
`$table->decimal('amount', 8, 2);`  |  DECIMAL equivalent column with a precision (total digits) and scale (decimal digits).
`$table->double('amount', 8, 2);`  |  DOUBLE equivalent column with a precision (total digits) and scale (decimal digits).
`$table->enum('level', ['easy', 'hard']);`  |  ENUM equivalent column.
`$table->float('amount', 8, 2);`  |  FLOAT equivalent column with a precision (total digits) and scale (decimal digits).
`$table->geometry('positions');`  |  GEOMETRY equivalent column.
`$table->geometryCollection('positions');`  |  GEOMETRYCOLLECTION equivalent column.
`$table->increments('id');`  |  Auto-incrementing UNSIGNED INTEGER (primary key) equivalent column.
`$table->integer('votes');`  |  INTEGER equivalent column.
`$table->ipAddress('visitor');`  |  IP address equivalent column.
`$table->json('options');`  |  JSON equivalent column.
`$table->jsonb('options');`  |  JSONB equivalent column.
`$table->lineString('positions');`  |  LINESTRING equivalent column.
`$table->longText('description');`  |  LONGTEXT equivalent column.
`$table->macAddress('device');`  |  MAC address equivalent column.
`$table->mediumIncrements('id');`  |  Auto-incrementing UNSIGNED MEDIUMINT (primary key) equivalent column.
`$table->mediumInteger('votes');`  |  MEDIUMINT equivalent column.
`$table->mediumText('description');`  |  MEDIUMTEXT equivalent column.
`$table->morphs('taggable');`  |  Adds `taggable_id` UNSIGNED BIGINT and `taggable_type` VARCHAR equivalent columns.
`$table->multiLineString('positions');`  |  MULTILINESTRING equivalent column.
`$table->multiPoint('positions');`  |  MULTIPOINT equivalent column.
`$table->multiPolygon('positions');`  |  MULTIPOLYGON equivalent column.
`$table->nullableMorphs('taggable');`  |  Adds nullable versions of `morphs()` columns.
`$table->nullableTimestamps();`  |  Alias of `timestamps()` method.
`$table->point('position');`  |  POINT equivalent column.
`$table->polygon('positions');`  |  POLYGON equivalent column.
`$table->rememberToken();`  |  Adds a nullable `remember_token` VARCHAR(100) equivalent column.
`$table->smallIncrements('id');`  |  Auto-incrementing UNSIGNED SMALLINT (primary key) equivalent column.
`$table->smallInteger('votes');`  |  SMALLINT equivalent column.
`$table->softDeletes();`  |  Adds a nullable `deleted_at` TIMESTAMP equivalent column for soft deletes.
`$table->softDeletesTz();`  |  Adds a nullable `deleted_at` TIMESTAMP (with timezone) equivalent column for soft deletes.
`$table->string('name', 100);`  |  VARCHAR equivalent column with a optional length.
`$table->text('description');`  |  TEXT equivalent column.
`$table->time('sunrise');`  |  TIME equivalent column.
`$table->timeTz('sunrise');`  |  TIME (with timezone) equivalent column.
`$table->timestamp('added_on');`  |  TIMESTAMP equivalent column.
`$table->timestampTz('added_on');`  |  TIMESTAMP (with timezone) equivalent column.
`$table->timestamps();`  |  Adds nullable `created_at` and `updated_at` TIMESTAMP equivalent columns.
`$table->timestampsTz();`  |  Adds nullable `created_at` and `updated_at` TIMESTAMP (with timezone) equivalent columns.
`$table->tinyIncrements('id');`  |  Auto-incrementing UNSIGNED TINYINT (primary key) equivalent column.
`$table->tinyInteger('votes');`  |  TINYINT equivalent column.
`$table->unsignedBigInteger('votes');`  |  UNSIGNED BIGINT equivalent column.
`$table->unsignedDecimal('amount', 8, 2);`  |  UNSIGNED DECIMAL equivalent column with a precision (total digits) and scale (decimal digits).
`$table->unsignedInteger('votes');`  |  UNSIGNED INTEGER equivalent column.
`$table->unsignedMediumInteger('votes');`  |  UNSIGNED MEDIUMINT equivalent column.
`$table->unsignedSmallInteger('votes');`  |  UNSIGNED SMALLINT equivalent column.
`$table->unsignedTinyInteger('votes');`  |  UNSIGNED TINYINT equivalent column.
`$table->uuid('id');`  |  UUID equivalent column.
`$table->year('birth_year');`  |  YEAR equivalent column.

<a name="column-modifiers"></a>
### Modificadores de columna : Column Modifiers

Además de los tipos de columna enumerados anteriormente, hay varios "modificadores" de columna que puede usar al agregar una columna a una tabla de base de datos. Por ejemplo, para hacer que la columna permita "valores nulos", puedes usar el método `nullable`:
> > In addition to the column types listed above, there are several column "modifiers" you may use while adding a column to a database table. For example, to make the column "nullable", you may use the `nullable` method:

    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->nullable();
    });

A continuación se muestra una lista de todos los modificadores de columna disponibles. Esta lista no incluye los [modificadores de índice](#creating-indexes):
> > Below is a list of all the available column modifiers. This list does not include the [index modifiers](#creating-indexes):

Modifier  |  Description
--------  |  -----------
`->after('column')`  |  Place the column "after" another column (MySQL)
`->autoIncrement()`  |  Set INTEGER columns as auto-increment (primary key)
`->charset('utf8')`  |  Specify a character set for the column (MySQL)
`->collation('utf8_unicode_ci')`  |  Specify a collation for the column (MySQL/SQL Server)
`->comment('my comment')`  |  Add a comment to a column (MySQL)
`->default($value)`  |  Specify a "default" value for the column
`->first()`  |  Place the column "first" in the table (MySQL)
`->nullable($value = true)`  |  Allows (by default) NULL values to be inserted into the column
`->storedAs($expression)`  |  Create a stored generated column (MySQL)
`->unsigned()`  |  Set INTEGER columns as UNSIGNED (MySQL)
`->useCurrent()`  |  Set TIMESTAMP columns to use CURRENT_TIMESTAMP as default value
`->virtualAs($expression)`  |  Create a virtual generated column (MySQL)

<a name="modifying-columns"></a>
### Modificando columnas : Modifying Columns

#### Prerrequisitos : Prerequisites

Antes de modificar una columna, asegúrese de agregar la dependencia `doctrine/dbal` a su archivo `composer.json`. La biblioteca Doctrine DBAL se usa para determinar el estado actual de la columna y crear las consultas SQL necesarias para realizar los ajustes especificados en la columna:
> > Before modifying a column, be sure to add the `doctrine/dbal` dependency to your `composer.json` file. The Doctrine DBAL library is used to determine the current state of the column and create the SQL queries needed to make the specified adjustments to the column:

    composer require doctrine/dbal

#### Actualización de atributos de columna : Updating Column Attributes

El método `change` le permite modificar algunos tipos de columna existentes a un nuevo tipo o modificar los atributos de la columna. Por ejemplo, puede desear aumentar el tamaño de una columna de cadena. Para ver el método `change` en acción, aumentemos el tamaño de la columna `name` de 25 a 50:
> > The `change` method allows you to modify some existing column types to a new type or modify the column's attributes. For example, you may wish to increase the size of a string column. To see the `change` method in action, let's increase the size of the `name` column from 25 to 50:

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->change();
    });

También podríamos modificar una columna para que sea nullable:
> > We could also modify a column to be nullable:

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->nullable()->change();
    });

> {note} Solo se pueden "cambiar" los siguientes tipos de columnas: bigInteger, binary, boolean, date, dateTime, dateTimeTz, decimal, integer, json, longText, mediumText, smallInteger, string, text, time, unsignedBigInteger, unsignedInteger y unsignedSmallInteger.
> > > {note} Only the following column types can be "changed": bigInteger, binary, boolean, date, dateTime, dateTimeTz, decimal, integer, json, longText, mediumText, smallInteger, string, text, time, unsignedBigInteger, unsignedInteger and unsignedSmallInteger.

#### Cambiar el nombre de las columnas : Renaming Columns

Para cambiar el nombre de una columna, puede usar el método `renameColumn` en el generador de esquemas. Antes de renombrar una columna, asegúrese de agregar la dependencia `doctrine/dbal` a su archivo `composer.json`:
> > To rename a column, you may use the `renameColumn` method on the Schema builder. Before renaming a column, be sure to add the `doctrine/dbal` dependency to your `composer.json` file:

    Schema::table('users', function (Blueprint $table) {
        $table->renameColumn('from', 'to');
    });

> {nota} Cambiar el nombre de cualquier columna en una tabla que también tiene una columna de tipo `enum` no es compatible actualmente.
> > > {note} Renaming any column in a table that also has a column of type `enum` is not currently supported.

<a name="dropping-columns"></a>
### Eliminando columnas : Dropping Columns

Para eliminar una columna, use el método `dropColumn` en el generador de esquemas. Antes de eliminar columnas de una base de datos SQLite, deberá agregar la dependencia `doctrine/dbal` a su archivo `composer.json` y ejecutar el comando `composer update` en su terminal para instalar la biblioteca:
> > To drop a column, use the `dropColumn` method on the Schema builder. Before dropping columns from a SQLite database, you will need to add the `doctrine/dbal` dependency to your `composer.json` file and run the `composer update` command in your terminal to install the library:

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('votes');
    });

Puede eliminar varias columnas de una tabla pasando un array de nombres de columnas al método `dropColumn`:
> > You may drop multiple columns from a table by passing an array of column names to the `dropColumn` method:

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });

> {note} No se admite descartar o modificar varias columnas dentro de una única migración mientras se usa una base de datos SQLite.
> > > {note} Dropping or modifying multiple columns within a single migration while using a SQLite database is not supported.

#### Alias ​​de comando disponibles : Available Command Aliases

Command  |  Description
-------  |  -----------
`$table->dropRememberToken();`  |  Drop the `remember_token` column.
`$table->dropSoftDeletes();`  |  Drop the `deleted_at` column.
`$table->dropSoftDeletesTz();`  |  Alias of `dropSoftDeletes()` method.
`$table->dropTimestamps();`  |  Drop the `created_at` and `updated_at` columns.
`$table->dropTimestampsTz();` |  Alias of `dropTimestamps()` method.

<a name="indexes"></a>
## Índices : Indexes

<a name="creating-indexes"></a>
### Creando índices : Creating Indexes

El generador de esquemas admite varios tipos de índices. Primero, veamos un ejemplo que especifica que los valores de una columna deben ser únicos. Para crear el índice, podemos encadenar el método `unique` en la definición de la columna:
> > The schema builder supports several types of indexes. First, let's look at an example that specifies a column's values should be unique. To create the index, we can chain the `unique` method onto the column definition:

    $table->string('email')->unique();

Alternativamente, puede crear el índice después de definir la columna. Por ejemplo:
> > Alternatively, you may create the index after defining the column. For example:

    $table->unique('email');

Incluso puede pasar un array de columnas a un método de índice para crear un índice compuesto (o compuesto):
> > You may even pass an array of columns to an index method to create a compound (or composite) index:

    $table->index(['account_id', 'created_at']);

Laravel generará automáticamente un nombre de índice razonable, pero puede pasar un segundo argumento al método para especificar el nombre usted mismo:
> > Laravel will automatically generate a reasonable index name, but you may pass a second argument to the method to specify the name yourself:

    $table->unique('email', 'unique_email');

#### Tipos de índice disponibles : Available Index Types

Cada método de índice acepta un segundo argumento opcional para especificar el nombre del índice. Si se omite, el nombre se derivará de los nombres de la tabla y columna(s).
> > Each index method accepts an optional second argument to specify the name of the index. If omitted, the name will be derived from the names of the table and column(s).

Command  |  Description
-------  |  -----------
`$table->primary('id');`  |  Adds a primary key.
`$table->primary(['id', 'parent_id']);`  |  Adds composite keys.
`$table->unique('email');`  |  Adds a unique index.
`$table->index('state');`  |  Adds a plain index.
`$table->spatialIndex('location');`  |  Adds a spatial index. (except SQLite)

#### Longitudes de índice MySQL / MariaDB : Index Lengths & MySQL / MariaDB

Laravel usa el juego de caracteres `utf8mb4` de forma predeterminada, que incluye soporte para almacenar "emojis" en la base de datos. Si está ejecutando una versión de MySQL anterior a la versión 5.7.7 o MariaDB anterior a la versión 10.2.2, puede que necesite configurar manualmente la longitud de cadena predeterminada generada por las migraciones para que MySQL cree índices para ellas. Puede configurar esto llamando al método `Schema::defaultStringLength` dentro de su `AppServiceProvider`:
> > Laravel uses the `utf8mb4` character set by default, which includes support for storing "emojis" in the database. If you are running a version of MySQL older than the 5.7.7 release or MariaDB older than the 10.2.2 release, you may need to manually configure the default string length generated by migrations in order for MySQL to create indexes for them. You may configure this by calling the `Schema::defaultStringLength` method within your `AppServiceProvider`:

    use Illuminate\Support\Facades\Schema;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Schema::defaultStringLength(191);
    }

Alternativamente, puede habilitar la opción `innodb_large_prefix` para su base de datos. Consulte la documentación de su base de datos para obtener instrucciones sobre cómo habilitar correctamente esta opción.
> > Alternatively, you may enable the `innodb_large_prefix` option for your database. Refer to your database's documentation for instructions on how to properly enable this option.

<a name="renaming-indexes"></a>
### Renombrando índices : Renaming Indexes

Para cambiar el nombre de un índice, puede usar el método `renameIndex`. Este método acepta el nombre del índice actual como primer argumento y el nombre deseado como segundo argumento:
> > To rename an index, you may use the `renameIndex` method. This method accepts the current index name as its first argument and the desired name as its second argument:

    $table->renameIndex('from', 'to')

<a name="dropping-indexes"></a>
### Eliminando índices : Dropping Indexes

Para eliminar un índice, debe especificar el nombre del índice. Por defecto, Laravel asigna automáticamente un nombre razonable a los índices. Concatenar el nombre de la tabla, el nombre de la columna indexada y el tipo de índice. Aquí hay unos ejemplos:
> > To drop an index, you must specify the index's name. By default, Laravel automatically assigns a reasonable name to the indexes. Concatenate the table name, the name of the indexed column, and the index type. Here are some examples:

Command  |  Description
-------  |  -----------
`$table->dropPrimary('users_id_primary');`  |  Drop a primary key from the "users" table.
`$table->dropUnique('users_email_unique');`  |  Drop a unique index from the "users" table.
`$table->dropIndex('geo_state_index');`  |  Drop a basic index from the "geo" table.
`$table->dropSpatialIndex('geo_location_spatialindex');`  |  Drop a spatial index from the "geo" table  (except SQLite).

Si transfiere un array de columnas a un método que elimina índices, el nombre de índice convencional se generará en función del nombre de la tabla, las columnas y el tipo de clave:
> > If you pass an array of columns into a method that drops indexes, the conventional index name will be generated based on the table name, columns and key type:

    Schema::table('geo', function (Blueprint $table) {
        $table->dropIndex(['state']); // Drops index 'geo_state_index'
    });

<a name="foreign-key-constraints"></a>
### Restricciones de clave externa : Foreign Key Constraints

Laravel también proporciona soporte para crear restricciones de clave externa, que se utilizan para forzar la integridad referencial a nivel de base de datos. Por ejemplo, definamos una columna `user_id` en la tabla `posts` que hace referencia a la columna `id` en una tabla `users`:
> > Laravel also provides support for creating foreign key constraints, which are used to force referential integrity at the database level. For example, let's define a `user_id` column on the `posts` table that references the `id` column on a `users` table:

    Schema::table('posts', function (Blueprint $table) {
        $table->unsignedInteger('user_id');

        $table->foreign('user_id')->references('id')->on('users');
    });

También puede especificar la acción deseada para las propiedades "on delete" y "on update" de la restricción:
> > You may also specify the desired action for the "on delete" and "on update" properties of the constraint:

    $table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

Para soltar una clave externa, puede usar el método `dropForeign`. Las restricciones de clave externa usan la misma convención de nomenclatura como índices. Por lo tanto, concatenaremos el nombre de la tabla y las columnas en la restricción y luego el sufijo "_foreign":
> > To drop a foreign key, you may use the `dropForeign` method. Foreign key constraints use the same naming convention as indexes. So, we will concatenate the table name and the columns in the constraint then suffix the name with "_foreign":

    $table->dropForeign('posts_user_id_foreign');

O bien, puede pasar un valor de array que utilizará automáticamente el nombre de restricción convencional al soltar:
> > Or, you may pass an array value which will automatically use the conventional constraint name when dropping:

    $table->dropForeign(['user_id']);

Puede habilitar o deshabilitar restricciones de clave externa en sus migraciones utilizando los siguientes métodos:
> > You may enable or disable foreign key constraints within your migrations by using the following methods:

    Schema::enableForeignKeyConstraints();

    Schema::disableForeignKeyConstraints();
