# Eloquent: Primeros pasos : Eloquent: Getting Started

- [Introduction](#introduction)
- [Defining Models](#defining-models)
    - [Eloquent Model Conventions](#eloquent-model-conventions)
- [Retrieving Models](#retrieving-models)
    - [Collections](#collections)
    - [Chunking Results](#chunking-results)
- [Retrieving Single Models / Aggregates](#retrieving-single-models)
    - [Retrieving Aggregates](#retrieving-aggregates)
- [Inserting & Updating Models](#inserting-and-updating-models)
    - [Inserts](#inserts)
    - [Updates](#updates)
    - [Mass Assignment](#mass-assignment)
    - [Other Creation Methods](#other-creation-methods)
- [Deleting Models](#deleting-models)
    - [Soft Deleting](#soft-deleting)
    - [Querying Soft Deleted Models](#querying-soft-deleted-models)
- [Query Scopes](#query-scopes)
    - [Global Scopes](#global-scopes)
    - [Local Scopes](#local-scopes)
- [Comparing Models](#comparing-models)
- [Events](#events)
    - [Observers](#observers)

<a name="introduction"></a>
## Introducción : Introduction

El ORM Eloquent incluido con Laravel proporciona una implementación de ActiveRecord bella y sencilla para trabajar con su base de datos. Cada tabla de base de datos tiene un "Modelo" correspondiente que se utiliza para interactuar con esa tabla. Los modelos le permiten consultar datos en sus tablas, así como insertar nuevos registros en la tabla.
> > The Eloquent ORM included with Laravel provides a beautiful, simple ActiveRecord implementation for working with your database. Each database table has a corresponding "Model" which is used to interact with that table. Models allow you to query for data in your tables, as well as insert new records into the table.

Antes de comenzar, asegúrese de configurar una conexión de base de datos en `config/database.php`. Para obtener más información sobre cómo configurar su base de datos, consulte [la documentación](/docs/{{version}}/database#configuration).
> > Before getting started, be sure to configure a database connection in `config/database.php`. For more information on configuring your database, check out [the documentation](/docs/{{version}}/database#configuration).

<a name="defining-models"></a>
## Definición de modelos : Defining Models

Para comenzar, creemos un modelo Eloquent. Los modelos generalmente viven en el directorio `app`, pero usted puede colocarlos en cualquier lugar que pueda cargarse automáticamente de acuerdo con su archivo `composer.json`. Todos los modelos Eloquent amplían la clase `Illuminate\Database\Eloquent\Model`.
> > To get started, let's create an Eloquent model. Models typically live in the `app` directory, but you are free to place them anywhere that can be auto-loaded according to your `composer.json` file. All Eloquent models extend `Illuminate\Database\Eloquent\Model` class.

La forma más fácil de crear una instancia de modelo es usar `make:model` [Artisan command](/docs/{{version}}/artisan):
> > The easiest way to create a model instance is using the `make:model` [Artisan command](/docs/{{version}}/artisan):

    php artisan make:model Flight

Si desea generar una [migración de base de datos](/docs/{{version}}/migrations) cuando genera el modelo, puede usar la opción `--migration` o` -m`:
> > If you would like to generate a [database migration](/docs/{{version}}/migrations) when you generate the model, you may use the `--migration` or `-m` option:

    php artisan make:model Flight --migration

    php artisan make:model Flight -m

<a name="eloquent-model-conventions"></a>
### Convenciones del modelo Eloquent : Eloquent Model Conventions

Ahora, veamos un ejemplo de modelo `Flight`, que usaremos para recuperar y almacenar información de nuestra tabla de base de datos `flights`:
> > Now, let's look at an example `Flight` model, which we will use to retrieve and store information from our `flights` database table:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        //
    }

#### Nombres de tabla : Table Names

Tenga en cuenta que no le dijimos a Eloquent qué tabla usar para nuestro modelo `Flight`. Por convención, el "snake case", se utilizará nombre plural de la clase como nombre de la tabla a menos que se especifique explícitamente otro nombre. Entonces, en este caso, Eloquent asumirá que el modelo `Flight` almacena registros en la tabla `flights`. Puede especificar una tabla personalizada definiendo una propiedad `table` en su modelo:
> > Note that we did not tell Eloquent which table to use for our `Flight` model. By convention, the "snake case", plural name of the class will be used as the table name unless another name is explicitly specified. So, in this case, Eloquent will assume the `Flight` model stores records in the `flights` table. You may specify a custom table by defining a `table` property on your model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The table associated with the model.
         *
         * @var string
         */
        protected $table = 'my_flights';
    }

#### Claves principales : Primary Keys

Eloquent también supondrá que cada tabla tiene una columna de clave principal llamada `id`. Puede definir una propiedad `$primaryKey` protegida para anular esta convención.
> > Eloquent will also assume that each table has a primary key column named `id`. You may define a protected `$primaryKey` property to override this convention.

Además, Eloquent asume que la clave primaria es un valor entero creciente, lo que significa que, por defecto, la clave primaria se convertirá en un `int` automáticamente. Si desea utilizar una clave primaria no incremental o no numérica, debe establecer la propiedad pública `$incrementing` en su modelo a `false`. Si su clave principal no es un número entero, debe establecer la propiedad `$keyType` protegida en su modelo en `string`.
> > In addition, Eloquent assumes that the primary key is an incrementing integer value, which means that by default the primary key will be cast to an `int` automatically. If you wish to use a non-incrementing or a non-numeric primary key you must set the public `$incrementing` property on your model to `false`. If your primary key is not an integer, you should set the protected `$keyType` property on your model to `string`.

#### Marcas de tiempo : Timestamps

Por defecto, Eloquent espera que las columnas `created_at` y `updated_at` existan en sus tablas. Si no desea que Eloquent administre automáticamente estas columnas, configure la propiedad `$timestamps` en su modelo en `false`:
> > By default, Eloquent expects `created_at` and `updated_at` columns to exist on your tables.  If you do not wish to have these columns automatically managed by Eloquent, set the `$timestamps` property on your model to `false`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Indicates if the model should be timestamped.
         *
         * @var bool
         */
        public $timestamps = false;
    }

Si necesita personalizar el formato de sus marcas de tiempo, configure la propiedad `$dateFormat` en su modelo. Esta propiedad determina cómo se almacenan los atributos de fecha en la base de datos, así como su formato cuando el modelo se serializa en un array o JSON:
> > If you need to customize the format of your timestamps, set the `$dateFormat` property on your model. This property determines how date attributes are stored in the database, as well as their format when the model is serialized to an array or JSON:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The storage format of the model's date columns.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

Si necesita personalizar los nombres de las columnas utilizadas para almacenar las marcas de tiempo, puede establecer las constantes `CREATED_AT` y` UPDATED_AT` en su modelo:
> > If you need to customize the names of the columns used to store the timestamps, you may set the `CREATED_AT` and `UPDATED_AT` constants in your model:

    <?php

    class Flight extends Model
    {
        const CREATED_AT = 'creation_date';
        const UPDATED_AT = 'last_update';
    }

#### Conexión a la base de datos : Database Connection

Por defecto, todos los modelos Eloquent usarán la conexión de base de datos predeterminada configurada para su aplicación. Si desea especificar una conexión diferente para el modelo, use la propiedad `$connection`:
> > By default, all Eloquent models will use the default database connection configured for your application. If you would like to specify a different connection for the model, use the `$connection` property:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The connection name for the model.
         *
         * @var string
         */
        protected $connection = 'connection-name';
    }

<a name="retrieving-models"></a>
## Recuperando modelos : Retrieving Models

Una vez que haya creado un modelo y [su tabla de base de datos asociada](/docs/{{version}}/migrations#writing-migrations), está listo para comenzar a recuperar datos de su base de datos. Piense en cada modelo Eloquent como un potente [generador de consultas](/docs/{{version}}/queries) que le permite consultar con fluidez la tabla de la base de datos asociada con el modelo. Por ejemplo:
> > Once you have created a model and [its associated database table](/docs/{{version}}/migrations#writing-migrations), you are ready to start retrieving data from your database. Think of each Eloquent model as a powerful [query builder](/docs/{{version}}/queries) allowing you to fluently query the database table associated with the model. For example:

    <?php

    use App\Flight;

    $flights = App\Flight::all();

    foreach ($flights as $flight) {
        echo $flight->name;
    }

#### Agregar restricciones adicionales : Adding Additional Constraints

El método `all` Eloquent devolverá todos los resultados en la tabla del modelo. Como cada modelo Eloquent sirve como [generador de consultas](/docs/{{version}}/queries), también puede agregar restricciones a las consultas, y luego usar el método `get` para recuperar los resultados:
> > The Eloquent `all` method will return all of the results in the model's table. Since each Eloquent model serves as a [query builder](/docs/{{version}}/queries), you may also add constraints to queries, and then use the `get` method to retrieve the results:

    $flights = App\Flight::where('active', 1)
                   ->orderBy('name', 'desc')
                   ->take(10)
                   ->get();

> {tip} Dado que los modelos Eloquent son constructores de consultas, debe revisar todos los métodos disponibles en [generador de consultas](/docs/{{version}}/queries). Puede usar cualquiera de estos métodos en sus consultas Eloquent.
> > > {tip} Since Eloquent models are query builders, you should review all of the methods available on the [query builder](/docs/{{version}}/queries). You may use any of these methods in your Eloquent queries.

<a name="collections"></a>
### Colecciones : Collections

Para métodos Eloquent como `all` y` get` que recuperan resultados múltiples, se devolverá una instancia de `Illuminate\Database\Eloquent\Collection`. La clase `Collection` proporciona [una variedad de métodos útiles](/docs/{{version}}/eloquent-collections#available-methods) para trabajar con sus resultados Eloquent:
> > For Eloquent methods like `all` and `get` which retrieve multiple results, an instance of `Illuminate\Database\Eloquent\Collection` will be returned. The `Collection` class provides [a variety of helpful methods](/docs/{{version}}/eloquent-collections#available-methods) for working with your Eloquent results:

    $flights = $flights->reject(function ($flight) {
        return $flight->cancelled;
    });

Por supuesto, también puede recorrer la colección como un array:
> > Of course, you may also loop over the collection like an array:

    foreach ($flights as $flight) {
        echo $flight->name;
    }

<a name="chunking-results"></a>
### Resultados de fragmentación : Chunking Results

Si necesita procesar miles de registros Eloquent, use el comando `chunk`. El método `chunk` recuperará un "trozo" de modelos Eloquent, alimentándolos a un `Closure` dado para su procesamiento. El uso del método `chunk` conservará la memoria cuando se trabaja con grandes conjuntos de resultados:
> > If you need to process thousands of Eloquent records, use the `chunk` command. The `chunk` method will retrieve a "chunk" of Eloquent models, feeding them to a given `Closure` for processing. Using the `chunk` method will conserve memory when working with large result sets:

    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });

El primer argumento que se pasa al método es la cantidad de registros que desea recibir por "porción". El cierre pasó ya que se llamará al segundo argumento para cada fragmento que se recupere de la base de datos. Se ejecutará una consulta de base de datos para recuperar cada fragmento de registros pasados ​​al Cierre.
> > The first argument passed to the method is the number of records you wish to receive per "chunk". The Closure passed as the second argument will be called for each chunk that is retrieved from the database. A database query will be executed to retrieve each chunk of records passed to the Closure.

#### Uso de cursores : Using Cursors

El método `cursor` le permite iterar a través de los registros de su base de datos usando un cursor, que solo ejecutará una sola consulta. Al procesar grandes cantidades de datos, el método `cursor` se puede usar para reducir en gran medida el uso de la memoria:
> > The `cursor` method allows you to iterate through your database records using a cursor, which will only execute a single query. When processing large amounts of data, the `cursor` method may be used to greatly reduce your memory usage:

    foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
        //
    }

<a name="retrieving-single-models"></a>
## Recuperando modelos únicos / agregados : Retrieving Single Models / Aggregates

Por supuesto, además de recuperar todos los registros de una tabla determinada, también puede recuperar registros individuales usando `find` o `first`. En lugar de devolver una colección de modelos, estos métodos devuelven una única instancia de modelo:
> > Of course, in addition to retrieving all of the records for a given table, you may also retrieve single records using `find` or `first`. Instead of returning a collection of models, these methods return a single model instance:

    // Retrieve a model by its primary key...
    $flight = App\Flight::find(1);

    // Retrieve the first model matching the query constraints...
    $flight = App\Flight::where('active', 1)->first();

También puede llamar al método `find` con un array de claves principales, que devolverá una colección de los registros coincidentes:
> > You may also call the `find` method with an array of primary keys, which will return a collection of the matching records:

    $flights = App\Flight::find([1, 2, 3]);

#### Excepciones no encontradas : Not Found Exceptions

En ocasiones, es posible que desee lanzar una excepción si no se encuentra un modelo. Esto es particularmente útil en rutas o controladores. Los métodos `findOrFail` y` firstOrFail` recuperarán el primer resultado de la consulta; sin embargo, si no se encuentra ningún resultado, se arrojará una 'Illuminate\Database\Eloquent\ModelNotFoundException`:
> > Sometimes you may wish to throw an exception if a model is not found. This is particularly useful in routes or controllers. The `findOrFail` and `firstOrFail` methods will retrieve the first result of the query; however, if no result is found, a `Illuminate\Database\Eloquent\ModelNotFoundException` will be thrown:

    $model = App\Flight::findOrFail(1);

    $model = App\Flight::where('legs', '>', 100)->firstOrFail();

Si no se detecta la excepción, se envía automáticamente una respuesta HTTP `404` al usuario. No es necesario escribir comprobaciones explícitas para devolver las respuestas `404` al usar estos métodos:
> > If the exception is not caught, a `404` HTTP response is automatically sent back to the user. It is not necessary to write explicit checks to return `404` responses when using these methods:

    Route::get('/api/flights/{id}', function ($id) {
        return App\Flight::findOrFail($id);
    });

<a name="retrieving-aggregates"></a>
### Recuperando agregados : Retrieving Aggregates

También puede usar `count`, `sum`, `max` y otros [métodos agregados](/docs/{{version}}/queries#aggregates) proporcionados por [query builder](/docs/{{version}}/queries). Estos métodos devuelven el valor escalar apropiado en lugar de una instancia de modelo completo:
> > You may also use the `count`, `sum`, `max`, and other [aggregate methods](/docs/{{version}}/queries#aggregates) provided by the [query builder](/docs/{{version}}/queries). These methods return the appropriate scalar value instead of a full model instance:

    $count = App\Flight::where('active', 1)->count();

    $max = App\Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## Inserción y actualización de modelos : Inserting & Updating Models

<a name="inserts"></a>
### Inserciones : Inserts

Para crear un nuevo registro en la base de datos, cree una nueva instancia de modelo, establezca atributos en el modelo y luego llame al método `save`:
> > To create a new record in the database, create a new model instance, set attributes on the model, then call the `save` method:

    <?php

    namespace App\Http\Controllers;

    use App\Flight;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class FlightController extends Controller
    {
        /**
         * Create a new flight instance.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate the request...

            $flight = new Flight;

            $flight->name = $request->name;

            $flight->save();
        }
    }

En este ejemplo, asignamos el parámetro `name` de la solicitud HTTP entrante al atributo `name` de la instancia del modelo `App\Flight`. Cuando llamamos al método `save`, se insertará un registro en la base de datos. Las marcas de tiempo `created_at` y` updated_at` se establecerán automáticamente cuando se llame al método `save`, por lo que no es necesario configurarlas manualmente.
> > In this example, we assign the `name` parameter from the incoming HTTP request to the `name` attribute of the `App\Flight` model instance. When we call the `save` method, a record will be inserted into the database. The `created_at` and `updated_at` timestamps will automatically be set when the `save` method is called, so there is no need to set them manually.

<a name="updates"></a>
### Actualizaciones : Updates

El método `save` también se puede usar para actualizar modelos que ya existen en la base de datos. Para actualizar un modelo, debe recuperarlo, establecer los atributos que desea actualizar y luego llamar al método `save`. De nuevo, la marca de tiempo `updated_at` se actualizará automáticamente, por lo que no es necesario establecer su valor manualmente:
> > The `save` method may also be used to update models that already exist in the database. To update a model, you should retrieve it, set any attributes you wish to update, and then call the `save` method. Again, the `updated_at` timestamp will automatically be updated, so there is no need to manually set its value:

    $flight = App\Flight::find(1);

    $flight->name = 'New Flight Name';

    $flight->save();

#### Actualizaciones masivas : Mass Updates

Las actualizaciones también pueden realizarse contra cualquier número de modelos que coincidan con una consulta determinada. En este ejemplo, todos los vuelos que estén `active` y tengan un `destination` de `San Diego` se marcarán como retrasados:
> > Updates can also be performed against any number of models that match a given query. In this example, all flights that are `active` and have a `destination` of `San Diego` will be marked as delayed:

    App\Flight::where('active', 1)
              ->where('destination', 'San Diego')
              ->update(['delayed' => 1]);

El método `update` espera que un array de pares de columnas y valores represente las columnas que deberían actualizarse.
> > The `update` method expects an array of column and value pairs representing the columns that should be updated.

> {note} Al emitir una actualización masiva a través de Eloquent, los eventos del modelo `saved` y `updated` no se dispararán para los modelos actualizados. Esto se debe a que los modelos nunca se recuperan realmente al emitir una actualización masiva.
> > > {note} When issuing a mass update via Eloquent, the `saved` and `updated` model events will not be fired for the updated models. This is because the models are never actually retrieved when issuing a mass update.

<a name="mass-assignment"></a>
### Asignación masiva : Mass Assignment

También puede usar el método `create` para guardar un nuevo modelo en una sola línea. La instancia del modelo insertado le será devuelta desde el método. Sin embargo, antes de hacerlo, deberá especificar un atributo `fillable` or `guarded` en el modelo, ya que todos los modelos Eloquent protegen contra la asignación masiva por defecto.
> > You may also use the `create` method to save a new model in a single line. The inserted model instance will be returned to you from the method. However, before doing so, you will need to specify either a `fillable` or `guarded` attribute on the model, as all Eloquent models protect against mass-assignment by default.

Una vulnerabilidad de asignación masiva ocurre cuando un usuario pasa un parámetro HTTP inesperado a través de una solicitud, y ese parámetro cambia una columna en su base de datos que usted no esperaba. Por ejemplo, un usuario malintencionado puede enviar un parámetro `is_admin` a través de una solicitud HTTP, que luego se pasa al método `create` de su modelo, lo que permite al usuario escalar a sí mismo a un administrador.
> > A mass-assignment vulnerability occurs when a user passes an unexpected HTTP parameter through a request, and that parameter changes a column in your database you did not expect. For example, a malicious user might send an `is_admin` parameter through an HTTP request, which is then passed into your model's `create` method, allowing the user to escalate themselves to an administrator.

Entonces, para comenzar, debe definir qué atributos de modelo desea asignar a la masa. Puede hacer esto usando la propiedad `$fillable` en el modelo. Por ejemplo, hagamos que el atributo `name` de nuestra asignación en masa del modelo `Flight`:
> > So, to get started, you should define which model attributes you want to make mass assignable. You may do this using the `$fillable` property on the model. For example, let's make the `name` attribute of our `Flight` model mass assignable:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The attributes that are mass assignable.
         *
         * @var array
         */
        protected $fillable = ['name'];
    }

Una vez que hemos asignado los atributos asignables en masa, podemos usar el método `create` para insertar un nuevo registro en la base de datos. El método `create` devuelve la instancia del modelo guardado:
> > Once we have made the attributes mass assignable, we can use the `create` method to insert a new record in the database. The `create` method returns the saved model instance:

    $flight = App\Flight::create(['name' => 'Flight 10']);

Si ya tiene una instancia modelo, puede usar el método `fill` para completarla con una matriz de atributos:
> > If you already have a model instance, you may use the `fill` method to populate it with an array of attributes:

    $flight->fill(['name' => 'Flight 22']);

#### Atributos de protección : Guarding Attributes

Si bien `$fillable` sirve como una "lista blanca" de atributos que deben asignarse en masa, también puede optar por usar `$guarded`. La propiedad `$guarded` debe contener un array de atributos que no desee que sean asignables en masa. Todos los demás atributos que no estén en el array se asignarán en masa. Entonces, `$guarded` funciona como una "lista negra". Por supuesto, debe usar `$fillable` o `$guarded` - no ambos. En el siguiente ejemplo, todos los atributos **excepto `price`** se asignarán en masa:
> > While `$fillable` serves as a "white list" of attributes that should be mass assignable, you may also choose to use `$guarded`. The `$guarded` property should contain an array of attributes that you do not want to be mass assignable. All other attributes not in the array will be mass assignable. So, `$guarded` functions like a "black list". Of course, you should use either `$fillable` or `$guarded` - not both. In the example below, all attributes **except for `price`** will be mass assignable:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The attributes that aren't mass assignable.
         *
         * @var array
         */
        protected $guarded = ['price'];
    }

Si desea que todos los atributos se puedan asignar en masa, puede definir la propiedad `$guarded` como una matriz vacía:
> > If you would like to make all attributes mass assignable, you may define the `$guarded` property as an empty array:

    /**
     * The attributes that aren't mass assignable.
     *
     * @var array
     */
    protected $guarded = [];

<a name="other-creation-methods"></a>
### Otros métodos de creación : Other Creation Methods

#### `firstOrCreate`/ `firstOrNew`

Existen otros dos métodos que puede usar para crear modelos mediante la asignación masiva de atributos: `firstOrCreate` y` firstOrNew`. El método `firstOrCreate` intentará localizar un registro de la base de datos utilizando los pares columna / valor dados. Si el modelo no se puede encontrar en la base de datos, se insertará un registro con los atributos del primer parámetro, junto con los del segundo parámetro opcional.
> > There are two other methods you may use to create models by mass assigning attributes: `firstOrCreate` and `firstOrNew`. The `firstOrCreate` method will attempt to locate a database record using the given column / value pairs. If the model can not be found in the database, a record will be inserted with the attributes from the first parameter, along with those in the optional second parameter.

El método `firstOrNew`, como `firstOrCreate` intentará ubicar un registro en la base de datos que coincida con los atributos dados. Sin embargo, si no se encuentra un modelo, se devolverá una nueva instancia del modelo. Tenga en cuenta que el modelo devuelto por `firstOrNew` aún no se ha conservado en la base de datos. Tendrá que llamar `save` manualmente para persistir:
> > The `firstOrNew` method, like `firstOrCreate` will attempt to locate a record in the database matching the given attributes. However, if a model is not found, a new model instance will be returned. Note that the model returned by `firstOrNew` has not yet been persisted to the database. You will need to call `save` manually to persist it:

    // Retrieve flight by name, or create it if it doesn't exist...
    $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

    // Retrieve flight by name, or create it with the name and delayed attributes...
    $flight = App\Flight::firstOrCreate(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );

    // Retrieve by name, or instantiate...
    $flight = App\Flight::firstOrNew(['name' => 'Flight 10']);

    // Retrieve by name, or instantiate with the name and delayed attributes...
    $flight = App\Flight::firstOrNew(
        ['name' => 'Flight 10'], ['delayed' => 1]
    );

#### `updateOrCreate`

También puede encontrar situaciones en las que desee actualizar un modelo existente o crear un nuevo modelo, si no existe ninguno. Laravel proporciona un método `updateOrCreate` para hacer esto en un solo paso. Al igual que el método `firstOrCreate`,` updateOrCreate` persiste en el modelo, por lo que no es necesario llamar a `save()`:
> > You may also come across situations where you want to update an existing model or create a new model if none exists. Laravel provides an `updateOrCreate` method to do this in one step. Like the `firstOrCreate` method, `updateOrCreate` persists the model, so there's no need to call `save()`:

    // If there's a flight from Oakland to San Diego, set the price to $99.
    // If no matching model exists, create one.
    $flight = App\Flight::updateOrCreate(
        ['departure' => 'Oakland', 'destination' => 'San Diego'],
        ['price' => 99]
    );

<a name="deleting-models"></a>
## Eliminación de modelos : Deleting Models

Para eliminar un modelo, llame al método `delete` en una instancia modelo:
> > To delete a model, call the `delete` method on a model instance:

    $flight = App\Flight::find(1);

    $flight->delete();

#### Eliminación de un modelo existente por clave : Deleting An Existing Model By Key

En el ejemplo anterior, estamos recuperando el modelo de la base de datos antes de llamar al método `delete`. Sin embargo, si conoce la clave principal del modelo, puede eliminar el modelo sin recuperarlo. Para hacerlo, llame al método `destroy`:
> > In the example above, we are retrieving the model from the database before calling the `delete` method. However, if you know the primary key of the model, you may delete the model without retrieving it. To do so, call the `destroy` method:

    App\Flight::destroy(1);

    App\Flight::destroy([1, 2, 3]);

    App\Flight::destroy(1, 2, 3);

#### Eliminar modelos por consulta : Deleting Models By Query

Por supuesto, también puede ejecutar una declaración de eliminación en un conjunto de modelos. En este ejemplo, borraremos todos los vuelos marcados como inactivos. Al igual que las actualizaciones masivas, las eliminaciones masivas no activarán ningún evento de modelo para los modelos que se eliminan:
> > Of course, you may also run a delete statement on a set of models. In this example, we will delete all flights that are marked as inactive. Like mass updates, mass deletes will not fire any model events for the models that are deleted:

    $deletedRows = App\Flight::where('active', 0)->delete();

> {note} Cuando se ejecuta una declaración de eliminación masiva a través de Eloquent, los eventos del modelo `deleting` y` deleted` no se dispararán para los modelos eliminados. Esto se debe a que los modelos nunca se recuperan cuando se ejecuta la declaración de eliminación.
> > > {note} When executing a mass delete statement via Eloquent, the `deleting` and `deleted` model events will not be fired for the deleted models. This is because the models are never actually retrieved when executing the delete statement.

<a name="soft-deleting"></a>
### Eliminación suave : Soft Deleting

Además de eliminar realmente los registros de su base de datos, Eloquent también puede "borrar por software" los modelos. Cuando los modelos son borrados por software, en realidad no se eliminan de su base de datos. En cambio, se establece un atributo `deleted_at` en el modelo y se inserta en la base de datos. Si un modelo tiene un valor 'deleted_at' no nulo, el modelo ha sido eliminado por software. Para habilitar eliminaciones suaves para un modelo, use el atributo `Illuminate\Database\Eloquent\SoftDeletes` en el modelo y agregue la columna `deleted_at` a su propiedad `$dates`:
> > In addition to actually removing records from your database, Eloquent can also "soft delete" models. When models are soft deleted, they are not actually removed from your database. Instead, a `deleted_at` attribute is set on the model and inserted into the database. If a model has a non-null `deleted_at` value, the model has been soft deleted. To enable soft deletes for a model, use the `Illuminate\Database\Eloquent\SoftDeletes` trait on the model and add the `deleted_at` column to your `$dates` property:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;

    class Flight extends Model
    {
        use SoftDeletes;

        /**
         * The attributes that should be mutated to dates.
         *
         * @var array
         */
        protected $dates = ['deleted_at'];
    }

Por supuesto, debe agregar la columna `deleted_at` a su tabla de base de datos. El [generador de esquemas](/docs/{{version}}/migrations) Laravel contiene un método auxiliar para crear esta columna:
> > Of course, you should add the `deleted_at` column to your database table. The Laravel [schema builder](/docs/{{version}}/migrations) contains a helper method to create this column:

    Schema::table('flights', function ($table) {
        $table->softDeletes();
    });

Ahora, cuando llame al método `delete` en el modelo, la columna `deleted_at` se establecerá en la fecha y hora actual. Y, al consultar un modelo que utiliza eliminaciones automáticas, los modelos borrados blandos se excluirán automáticamente de todos los resultados de la consulta.
> > Now, when you call the `delete` method on the model, the `deleted_at` column will be set to the current date and time. And, when querying a model that uses soft deletes, the soft deleted models will automatically be excluded from all query results.

Para determinar si una instancia de modelo determinada se ha eliminado por software, use el método `trashed`:
> > To determine if a given model instance has been soft deleted, use the `trashed` method:

    if ($flight->trashed()) {
        //
    }

<a name="querying-soft-deleted-models"></a>
### Consulta de modelos eliminados de software : Querying Soft Deleted Models

#### Including Soft Deleted Models : Incluyendo modelos borrados suaves

Como se indicó anteriormente, los modelos borrados blandos se excluirán automáticamente de los resultados de la consulta. Sin embargo, puede obligar a los modelos borrados blandos a aparecer en un conjunto de resultados usando el método `withTrashed` en la consulta:
> > As noted above, soft deleted models will automatically be excluded from query results. However, you may force soft deleted models to appear in a result set using the `withTrashed` method on the query:

    $flights = App\Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();

El método `withTrashed` también se puede usar en una consulta [relación](/docs/{{version}}/eloquent-relationships):
> > The `withTrashed` method may also be used on a [relationship](/docs/{{version}}/eloquent-relationships) query:

    $flight->history()->withTrashed()->get();

#### Recuperando solo modelos borrados suaves : Retrieving Only Soft Deleted Models

El método `onlyTrashed` recuperará **solo** modelos eliminados suaves:
> > The `onlyTrashed` method will retrieve **only** soft deleted models:

    $flights = App\Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();

#### Restauración de modelos borrados suaves : Restoring Soft Deleted Models

A veces puede desear "borrar" un modelo con borrado suave. Para restaurar un modelo borrado suave a un estado activo, use el método `restore` en una instancia modelo:
> > Sometimes you may wish to "un-delete" a soft deleted model. To restore a soft deleted model into an active state, use the `restore` method on a model instance:

    $flight->restore();

También puede usar el método `restore` en una consulta para restaurar rápidamente varios modelos. Nuevamente, al igual que otras operaciones "masivas", esto no disparará ningún evento modelo para los modelos que se restauran:
> > You may also use the `restore` method in a query to quickly restore multiple models. Again, like other "mass" operations, this will not fire any model events for the models that are restored:

    App\Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();

Al igual que el método `withTrashed`, el método `restore` también se puede usar en [relaciones] (/docs/{{version}}/eloquent-relationships):
> > Like the `withTrashed` method, the `restore` method may also be used on [relationships](/docs/{{version}}/eloquent-relationships):

    $flight->history()->restore();

#### Eliminar permanentemente los modelos : Permanently Deleting Models

A veces es posible que necesite eliminar realmente un modelo de su base de datos. Para eliminar permanentemente un modelo borrado suave de la base de datos, use el método `forceDelete`:
> > Sometimes you may need to truly remove a model from your database. To permanently remove a soft deleted model from the database, use the `forceDelete` method:

    // Force deleting a single model instance...
    $flight->forceDelete();

    // Force deleting all related models...
    $flight->history()->forceDelete();

<a name="query-scopes"></a>
## Ámbito de consultas : Query Scopes

<a name="global-scopes"></a>
### Ámbito global : Global Scopes

Los ámbitos globales le permiten agregar restricciones a todas las consultas para un modelo determinado. La propia funcionalidad de [borrado suave](#soft-deleting) de Laravel utiliza ámbitos globales para extraer únicamente modelos "no eliminados" de la base de datos. Escribir sus propios alcances globales puede proporcionar una manera conveniente y fácil de asegurarse de que cada consulta para un modelo determinado reciba ciertas restricciones.
> > Global scopes allow you to add constraints to all queries for a given model. Laravel's own [soft delete](#soft-deleting) functionality utilizes global scopes to only pull "non-deleted" models from the database. Writing your own global scopes can provide a convenient, easy way to make sure every query for a given model receives certain constraints.

#### Escritura de ámbitos globales : Writing Global Scopes

Escribir un alcance global es simple. Defina una clase que implemente la interfaz `Illuminate\Database\Eloquent\Scope`. Esta interfaz requiere que implemente un método: `apply`. El método `apply` puede agregar restricciones `where` a la consulta según sea necesario:
> > Writing a global scope is simple. Define a class that implements the `Illuminate\Database\Eloquent\Scope` interface. This interface requires you to implement one method: `apply`. The `apply` method may add `where` constraints to the query as needed:

    <?php

    namespace App\Scopes;

    use Illuminate\Database\Eloquent\Scope;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;

    class AgeScope implements Scope
    {
        /**
         * Apply the scope to a given Eloquent query builder.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $builder
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @return void
         */
        public function apply(Builder $builder, Model $model)
        {
            $builder->where('age', '>', 200);
        }
    }

> {tip} Si su ámbito global agrega columnas a la cláusula select de la consulta, debe usar el método `addSelect` en lugar de `select`. Esto evitará el reemplazo involuntario de la cláusula de selección existente de la consulta.
> > > {tip} If your global scope is adding columns to the select clause of the query, you should use the `addSelect` method instead of `select`. This will prevent the unintentional replacement of the query's existing select clause.

#### Aplicación de ámbitos globales : Applying Global Scopes

Para asignar un alcance global a un modelo, debe anular el método `boot` de un modelo dado y usar el método `addGlobalScope`:
> > To assign a global scope to a model, you should override a given model's `boot` method and use the `addGlobalScope` method:

    <?php

    namespace App;

    use App\Scopes\AgeScope;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The "booting" method of the model.
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();

            static::addGlobalScope(new AgeScope);
        }
    }

Después de agregar el alcance, una consulta a `User::all()` producirá el siguiente SQL:
> > After adding the scope, a query to `User::all()` will produce the following SQL:

    select * from `users` where `age` > 200

#### Ámbito global anónimo : Anonymous Global Scopes

Eloquent también le permite definir ámbitos globales utilizando Cierres, que es particularmente útil para ámbitos simples que no justifican una clase separada:
> > Eloquent also allows you to define global scopes using Closures, which is particularly useful for simple scopes that do not warrant a separate class:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;

    class User extends Model
    {
        /**
         * The "booting" method of the model.
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();

            static::addGlobalScope('age', function (Builder $builder) {
                $builder->where('age', '>', 200);
            });
        }
    }

#### Eliminación de ámbitos globales : Removing Global Scopes

Si desea eliminar un ámbito global para una consulta determinada, puede utilizar el método `withoutGlobalScope`. El método acepta el nombre de clase del alcance global como su único argumento:
> > If you would like to remove a global scope for a given query, you may use the `withoutGlobalScope` method. The method accepts the class name of the global scope as its only argument:

    User::withoutGlobalScope(AgeScope::class)->get();

O bien, si definió el alcance global utilizando un Cierre:
> > Or, if you defined the global scope using a Closure:

    User::withoutGlobalScope('age')->get();

Si desea eliminar varios o incluso todos los ámbitos globales, puede utilizar el método `withoutGlobalScopes`:
> > If you would like to remove several or even all of the global scopes, you may use the `withoutGlobalScopes` method:

    // Remove all of the global scopes...
    User::withoutGlobalScopes()->get();

    // Remove some of the global scopes...
    User::withoutGlobalScopes([
        FirstScope::class, SecondScope::class
    ])->get();

<a name="local-scopes"></a>
### Ámbitos locales : Local Scopes

Los ámbitos locales le permiten definir conjuntos comunes de restricciones que puede reutilizar fácilmente en toda su aplicación. Por ejemplo, es posible que necesite recuperar con frecuencia a todos los usuarios que se consideran "populares". Para definir un alcance, prefija un método modelo Eloquent con `scope`.
> > Local scopes allow you to define common sets of constraints that you may easily re-use throughout your application. For example, you may need to frequently retrieve all users that are considered "popular". To define a scope, prefix an Eloquent model method with `scope`.

Los ámbitos siempre deben devolver una instancia del generador de consultas:
> > Scopes should always return a query builder instance:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include popular users.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }

        /**
         * Scope a query to only include active users.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeActive($query)
        {
            return $query->where('active', 1);
        }
    }

#### Utilizando un alcance local : Utilizing A Local Scope

Una vez que se ha definido el ámbito, puede llamar a los métodos de alcance al consultar el modelo. Sin embargo, no debe incluir el prefijo `scope` cuando llame al método. Incluso puede encadenar llamadas a varios ámbitos, por ejemplo:
> > Once the scope has been defined, you may call the scope methods when querying the model. However, you should not include the `scope` prefix when calling the method. You can even chain calls to various scopes, for example:

    $users = App\User::popular()->active()->orderBy('created_at')->get();

#### Ámbitos dinámicos : Dynamic Scopes

A veces puede querer definir un alcance que acepte parámetros. Para comenzar, solo agregue sus parámetros adicionales a su alcance. Los parámetros del alcance se deben definir después del parámetro `$query`:
> > Sometimes you may wish to define a scope that accepts parameters. To get started, just add your additional parameters to your scope. Scope parameters should be defined after the `$query` parameter:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include users of a given type.
         *
         * @param \Illuminate\Database\Eloquent\Builder $query
         * @param mixed $type
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeOfType($query, $type)
        {
            return $query->where('type', $type);
        }
    }

Ahora, puede pasar los parámetros cuando llame al alcance:
> > Now, you may pass the parameters when calling the scope:

    $users = App\User::ofType('admin')->get();

<a name="comparing-models"></a>
## Comparación de modelos : Comparing Models

A veces puede necesitar determinar si dos modelos son el "mismo". El método `is` se puede usar para verificar rápidamente que dos modelos tengan la misma clave primaria, tabla y conexión de base de datos:
> > Sometimes you may need to determine if two models are the "same". The `is` method may be used to quickly verify two models have same primary key, table, and database connection:

    if ($post->is($anotherPost)) {
        //
    }

<a name="events"></a>
## Eventos : Events

Los modelos Eloquent activan varios eventos, lo que te permite conectar los siguientes puntos en el ciclo de vida de un modelo: `retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`. Los eventos le permiten ejecutar código fácilmente cada vez que se guarda o actualiza una clase de modelo específica en la base de datos.
> > Eloquent models fire several events, allowing you to hook into the following points in a model's lifecycle: `retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`. Events allow you to easily execute code each time a specific model class is saved or updated in the database.

El evento `retrieved` se activará cuando se recupere un modelo existente de la base de datos. Cuando se guarda un nuevo modelo por primera vez, los eventos `creating` y `created` se dispararán. Si un modelo ya existía en la base de datos y se llama al método `save`, los eventos `updating` / `updated` se activarán. Sin embargo, en ambos casos, se dispararán los eventos `saving` / `saved`.
> > The `retrieved` event will fire when an existing model is retrieved from the database. When a new model is saved for the first time, the `creating` and `created` events will fire. If a model already existed in the database and the `save` method is called, the `updating` / `updated` events will fire. However, in both cases, the `saving` / `saved` events will fire.

Para comenzar, defina una propiedad `$dispatchesEvents` en su modelo Eloquent que asigne varios puntos del ciclo de vida del modelo Eloquent a sus propias [clases de eventos](/docs/{{version}}/events):
> > To get started, define a `$dispatchesEvents` property on your Eloquent model that maps various points of the Eloquent model's lifecycle to your own [event classes](/docs/{{version}}/events):

    <?php

    namespace App;

    use App\Events\UserSaved;
    use App\Events\UserDeleted;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * The event map for the model.
         *
         * @var array
         */
        protected $dispatchesEvents = [
            'saved' => UserSaved::class,
            'deleted' => UserDeleted::class,
        ];
    }

<a name="observers"></a>
### Observadores : Observers

#### Definición de observadores : Defining Observers

Si está escuchando muchos eventos en un modelo dado, puede usar observadores para agrupar a todos sus oyentes en una sola clase. Las clases de observadores tienen nombres de métodos que reflejan los eventos elocuentes que desea escuchar. Cada uno de estos métodos recibe el modelo como su único argumento. El comando Artisan `make:observer` es la forma más fácil de crear una nueva clase de observador:
> > If you are listening for many events on a given model, you may use observers to group all of your listeners into a single class. Observers classes have method names which reflect the Eloquent events you wish to listen for. Each of these methods receives the model as their only argument. The `make:observer` Artisan command is the easiest way to create a new observer class:

    php artisan make:observer UserObserver --model=User

Este comando colocará al nuevo observador en su directorio `App/Observers`. Si este directorio no existe, Artisan lo creará para usted. Su observador nuevo tendrá el siguiente aspecto:
> > This command will place the new observer in your `App/Observers` directory. If this directory does not exist, Artisan will create it for you. Your fresh observer will look like the following:

    <?php

    namespace App\Observers;

    use App\User;

    class UserObserver
    {
        /**
         * Handle to the User "created" event.
         *
         * @param  \App\User  $user
         * @return void
         */
        public function created(User $user)
        {
            //
        }

        /**
         * Handle the User "updated" event.
         *
         * @param  \App\User  $user
         * @return void
         */
        public function updated(User $user)
        {
            //
        }

        /**
         * Handle the User "deleted" event.
         *
         * @param  \App\User  $user
         * @return void
         */
        public function deleted(User $user)
        {
            //
        }
    }

Para registrar un observador, use el método `observe` en el modelo que desea observar. Puede registrar observadores en el método `boot` de uno de sus proveedores de servicios. En este ejemplo, registraremos al observador en `AppServiceProvider`:
> > To register an observer, use the `observe` method on the model you wish to observe. You may register observers in the `boot` method of one of your service providers. In this example, we'll register the observer in the `AppServiceProvider`:

    <?php

    namespace App\Providers;

    use App\User;
    use App\Observers\UserObserver;
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
            User::observe(UserObserver::class);
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
