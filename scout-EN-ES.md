# Laravel Scout

- [Introduction](#introduction)
- [Installation](#installation)
    - [Queueing](#queueing)
    - [Driver Prerequisites](#driver-prerequisites)
- [Configuration](#configuration)
    - [Configuring Model Indexes](#configuring-model-indexes)
    - [Configuring Searchable Data](#configuring-searchable-data)
    - [Configuring The Model ID](#configuring-the-model-id)
- [Indexing](#indexing)
    - [Batch Import](#batch-import)
    - [Adding Records](#adding-records)
    - [Updating Records](#updating-records)
    - [Removing Records](#removing-records)
    - [Pausing Indexing](#pausing-indexing)
    - [Conditionally Searchable Model Instances](#conditionally-searchable-model-instances)
- [Searching](#searching)
    - [Where Clauses](#where-clauses)
    - [Pagination](#pagination)
    - [Soft Deleting](#soft-deleting)
- [Custom Engines](#custom-engines)

<a name="introduction"></a>
## Introducción : Introduction

Laravel Scout proporciona una solución simple basada en controladores para agregar búsquedas de texto completo a sus [modelos Eloquent](/docs/{{version}}/eloquent). Usando observadores modelo, Scout mantendrá tus índices de búsqueda sincronizados con sus registros Elocuentes.
> > Laravel Scout provides a simple, driver based solution for adding full-text search to your [Eloquent models](/docs/{{version}}/eloquent). Using model observers, Scout will automatically keep your search indexes in sync with your Eloquent records.

Actualmente, Scout se envía con un controlador [Algolia](https://www.algolia.com/); sin embargo, escribir controladores personalizados es simple y usted es libre de extender Scout con sus propias implementaciones de búsqueda.
> > Currently, Scout ships with an [Algolia](https://www.algolia.com/) driver; however, writing custom drivers is simple and you are free to extend Scout with your own search implementations.

<a name="installation"></a>
## Instalación : Installation

Primero, instale Scout a través del administrador de paquetes Composer:
> > First, install Scout via the Composer package manager:

    composer require laravel/scout

Después de instalar Scout, debe publicar la configuración Scout utilizando el comando Artesano `vendor:publish`. Este comando publicará el archivo de configuración `scout.php` en su directorio `config`:
> > After installing Scout, you should publish the Scout configuration using the `vendor:publish` Artisan command. This command will publish the `scout.php` configuration file to your `config` directory:

    php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"

Finalmente, agregue el rasgo `Laravel\Scout\Searchable` al modelo que le gustaría hacer búsquedas. Este rasgo registrará un observador modelo para mantener el modelo sincronizado con su controlador de búsqueda:
> > Finally, add the `Laravel\Scout\Searchable` trait to the model you would like to make searchable. This trait will register a model observer to keep the model in sync with your search driver:

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;
    }

<a name="queueing"></a>
### Cola : Queueing

Si bien no se requiere estrictamente el uso de Scout, se debe considerar la configuración de un [controlador de cola](/docs/{{version}}/queues) antes de usar la biblioteca. La ejecución de un trabajador de cola permitirá que Scout ponga en cola todas las operaciones que sincronizan la información de su modelo con sus índices de búsqueda, proporcionando tiempos de respuesta mucho mejores para la interfaz web de su aplicación.
> > While not strictly required to use Scout, you should strongly consider configuring a [queue driver](/docs/{{version}}/queues) before using the library. Running a queue worker will allow Scout to queue all operations that sync your model information to your search indexes, providing much better response times for your application's web interface.

Una vez que haya configurado un controlador de cola, establezca el valor de la opción `queue` en su archivo de configuración `config/scout.php` en `true`:
> > Once you have configured a queue driver, set the value of the `queue` option in your `config/scout.php` configuration file to `true`:

    'queue' => true,

<a name="driver-prerequisites"></a>
### Requisitos previos del Driver : Driver Prerequisites

#### Algolia

When using the Algolia driver, you should configure your Algolia `id` and `secret` credentials in your `config/scout.php` configuration file. Once your credentials have been configured, you will also need to install the Algolia PHP SDK via the Composer package manager:

    composer require algolia/algoliasearch-client-php

<a name="configuration"></a>
## Configuración : Configuration

<a name="configuring-model-indexes"></a>
### Configuración de índices de modelo : Configuring Model Indexes

Cada modelo Eloquent se sincroniza con un "índice" de búsqueda dado, que contiene todos los registros de búsqueda para ese modelo. En otras palabras, puedes pensar en cada índice como una tabla MySQL. De forma predeterminada, cada modelo se mantendrá en un índice que coincida con el nombre de la "tabla" típica del modelo. Normalmente, esta es la forma plural del nombre del modelo; sin embargo, puede personalizar el índice del modelo anulando el método `searchableAs` en el modelo:
> > Each Eloquent model is synced with a given search "index", which contains all of the searchable records for that model. In other words, you can think of each index like a MySQL table. By default, each model will be persisted to an index matching the model's typical "table" name. Typically, this is the plural form of the model name; however, you are free to customize the model's index by overriding the `searchableAs` method on the model:

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * Get the index name for the model.
         *
         * @return string
         */
        public function searchableAs()
        {
            return 'posts_index';
        }
    }

<a name="configuring-searchable-data"></a>
### Configuración de datos de búsqueda : Configuring Searchable Data

Por defecto, toda la forma `toArray` de un modelo dado se mantendrá en su índice de búsqueda. Si desea personalizar los datos que están sincronizados con el índice de búsqueda, puede anular el método `toSearchableArray` en el modelo:
> > By default, the entire `toArray` form of a given model will be persisted to its search index. If you would like to customize the data that is synchronized to the search index, you may override the `toSearchableArray` method on the model:

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * Get the indexable data array for the model.
         *
         * @return array
         */
        public function toSearchableArray()
        {
            $array = $this->toArray();

            // Customize array...

            return $array;
        }
    }

<a name="configuring-the-model-id"></a>
### Configurando el ID del modelo : Configuring The Model ID

Por defecto, Scout usará la clave principal del modelo como ID única almacenada en el índice de búsqueda. Si necesita personalizar este comportamiento, puede anular el método `getScoutKey` en el modelo:
> > By default, Scout will use the primary key of the model as the unique ID stored in the search index. If you need to customize this behavior, you may override the `getScoutKey` method on the model:

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        use Searchable;

        /**
         * Get the value used to index the model.
         *
         * @return mixed
         */
        public function getScoutKey()
        {
            return $this->email;
        }
    }

<a name="indexing"></a>
## Indexación : Indexing

<a name="batch-import"></a>
### Importación por lotes : Batch Import

Si está instalando Scout en un proyecto existente, es posible que ya tenga registros de la base de datos que necesite importar a su controlador de búsqueda. Scout proporciona un comando Artisan `import` que puede usar para importar todos sus registros existentes en sus índices de búsqueda:
> > If you are installing Scout into an existing project, you may already have database records you need to import into your search driver. Scout provides an `import` Artisan command that you may use to import all of your existing records into your search indexes:

    php artisan scout:import "App\Post"

El comando `flush` se puede usar para eliminar todos los registros de un modelo de sus índices de búsqueda:
> > The `flush` command may be used to remove all of a model's records from your search indexes:

    php artisan scout:flush "App\Post"

<a name="adding-records"></a>
### Agregar registros : Adding Records

Una vez que haya agregado el rasgo `Laravel\Scout\Searchable` a un modelo, todo lo que necesita hacer es guardar `save` una instancia modelo y se agregará automáticamente a su índice de búsqueda. Si ha configurado Scout para [usar colas](#queueing), su trabajador de cola realizará esta operación en segundo plano:
> > Once you have added the `Laravel\Scout\Searchable` trait to a model, all you need to do is `save` a model instance and it will automatically be added to your search index. If you have configured Scout to [use queues](#queueing) this operation will be performed in the background by your queue worker:

    $order = new App\Order;

    // ...

    $order->save();

#### Agregar vía consulta : Adding Via Query

Si desea agregar una colección de modelos a su índice de búsqueda a través de una consulta Eloquent, puede encadenar el método `searchable` en una consulta Eloquent. El método `searchable` [dividirá los resultados](/docs/{{version}}/eloquent#chunking-results) de la consulta y agrega los registros a su índice de búsqueda. Nuevamente, si configuró Scout para usar colas, todos los fragmentos serán agregados en segundo plano por los trabajadores de la cola:
> > If you would like to add a collection of models to your search index via an Eloquent query, you may chain the `searchable` method onto an Eloquent query. The `searchable` method will [chunk the results](/docs/{{version}}/eloquent#chunking-results) of the query and add the records to your search index. Again, if you have configured Scout to use queues, all of the chunks will be added in the background by your queue workers:

    // Adding via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();

    // You may also add records via relationships...
    $user->orders()->searchable();

    // You may also add records via collections...
    $orders->searchable();

El método `searchable` se puede considerar una operación "upsert". En otras palabras, si el registro del modelo ya está en su índice, se actualizará. Si no existe en el índice de búsqueda, se agregará al índice.
> > The `searchable` method can be considered an "upsert" operation. In other words, if the model record is already in your index, it will be updated. If it does not exist in the search index, it will be added to the index.

<a name="updating-records"></a>
### Actualización de registros : Updating Records

Para actualizar un modelo de búsqueda, solo necesita actualizar las propiedades de la instancia del modelo y `save` el modelo en su base de datos. Scout automáticamente persistirá en los cambios a su índice de búsqueda:
> > To update a searchable model, you only need to update the model instance's properties and `save` the model to your database. Scout will automatically persist the changes to your search index:

    $order = App\Order::find(1);

    // Update the order...

    $order->save();

También puede usar el método `searchable` en una consulta Eloquent para actualizar una colección de modelos. Si los modelos no existen en su índice de búsqueda, se crearán:
> > You may also use the `searchable` method on an Eloquent query to update a collection of models. If the models do not exist in your search index, they will be created:

    // Updating via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();

    // You may also update via relationships...
    $user->orders()->searchable();

    // You may also update via collections...
    $orders->searchable();

<a name="removing-records"></a>
### Eliminación de registros : Removing Records

Para eliminar un registro de su índice, `delete` el modelo de la base de datos. Esta forma de eliminación es incluso compatible con los modelos [soft deleted](/docs/{{version}}/eloquent#soft-deleting):
> > To remove a record from your index, `delete` the model from the database. This form of removal is even compatible with [soft deleted](/docs/{{version}}/eloquent#soft-deleting) models:

    $order = App\Order::find(1);

    $order->delete();

Si no desea recuperar el modelo antes de eliminar el registro, puede usar el método `unsearchable` en una instancia o colección de consulta Eloquent:
> > If you do not want to retrieve the model before deleting the record, you may use the `unsearchable` method on an Eloquent query instance or collection:

    // Removing via Eloquent query...
    App\Order::where('price', '>', 100)->unsearchable();

    // You may also remove via relationships...
    $user->orders()->unsearchable();

    // You may also remove via collections...
    $orders->unsearchable();

<a name="pausing-indexing"></a>
### Pausar la indexación : Pausing Indexing

A veces puede necesitar realizar un lote de operaciones Eloquent en un modelo sin sincronizar los datos del modelo con su índice de búsqueda. Puedes hacer esto usando el método `withoutSyncingToSearch`. Este método acepta un solo callback que se ejecutará inmediatamente. Cualquier operación de modelo que ocurra dentro del callback no se sincronizará con el índice del modelo:
> > Sometimes you may need to perform a batch of Eloquent operations on a model without syncing the model data to your search index. You may do this using the `withoutSyncingToSearch` method. This method accepts a single callback which will be immediately executed. Any model operations that occur within the callback will not be synced to the model's index:

    App\Order::withoutSyncingToSearch(function () {
        // Perform model actions...
    });

<a name="conditionally-searchable-model-instances"></a>
### Instancias de modelo con capacidad de búsqueda condicional : Conditionally Searchable Model Instances

En ocasiones, es posible que solo necesite hacer que un modelo se pueda buscar bajo ciertas condiciones. Por ejemplo, imagine que tiene el modelo `App\Post` que puede estar en uno de dos estados: "borrador" y "publicado". Es posible que solo desee permitir que las publicaciones "publicadas" puedan buscarse. Para lograr esto, puede definir un método `shouldBeSearchable` en su modelo:
> > Sometimes you may need to only make a model searchable under certain conditions. For example, imagine you have `App\Post` model that may be in one of two states: "draft" and "published". You may only want to allow "published" posts to be searchable. To accomplish this, you may define a `shouldBeSearchable` method on your model:

    public function shouldBeSearchable()
    {
        return $this->isPublished();
    }

<a name="searching"></a>
## Buscando : Searching

Puede comenzar a buscar un modelo utilizando el método `search`. El método de búsqueda acepta una sola cadena que se utilizará para buscar sus modelos. A continuación, debe encadenar el método `get` en la consulta de búsqueda para recuperar los modelos Eloquent que coincidan con la consulta de búsqueda proporcionada:
> > You may begin searching a model using the `search` method. The search method accepts a single string that will be used to search your models. You should then chain the `get` method onto the search query to retrieve the Eloquent models that match the given search query:

    $orders = App\Order::search('Star Trek')->get();

Como las búsquedas Scout devuelven una colección de modelos Eloquent, incluso puede devolver los resultados directamente desde una ruta o controlador y se convertirán automáticamente a JSON:
> > Since Scout searches return a collection of Eloquent models, you may even return the results directly from a route or controller and they will automatically be converted to JSON:

    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return App\Order::search($request->search)->get();
    });

Si desea obtener los resultados sin procesar antes de convertirlos a modelos Eloquent, debe usar el método `raw`:
> > If you would like to get the raw results before they are converted to Eloquent models, you should use the `raw` method:

    $orders = App\Order::search('Star Trek')->raw();

Las consultas de búsqueda normalmente se realizarán en el índice especificado por el método [`searchableAs`](#configuring-model-indexes) del modelo. Sin embargo, puede usar el método `within` para especificar un índice personalizado que debe buscarse en su lugar:
> > Search queries will typically be performed on the index specified by the model's [`searchableAs`](#configuring-model-indexes) method. However, you may use the `within` method to specify a custom index that should be searched instead:

    $orders = App\Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();

<a name="where-clauses"></a>
### Cláusulas Where : Where Clauses

Scout le permite agregar cláusulas simples de "dónde" a sus consultas de búsqueda. Actualmente, estas cláusulas solo admiten verificaciones de igualdad numéricas básicas, y son principalmente útiles para el alcance de las consultas de búsqueda por un ID de inquilino. Dado que un índice de búsqueda no es una base de datos relacional, las cláusulas "donde" más avanzadas no son compatibles actualmente:
> > Scout allows you to add simple "where" clauses to your search queries. Currently, these clauses only support basic numeric equality checks, and are primarily useful for scoping search queries by a tenant ID. Since a search index is not a relational database, more advanced "where" clauses are not currently supported:

    $orders = App\Order::search('Star Trek')->where('user_id', 1)->get();

<a name="pagination"></a>
### Paginación : Pagination

Además de recuperar una colección de modelos, puede paginar los resultados de búsqueda utilizando el método `paginate`. Este método devolverá una instancia `Paginator` como si hubiera [paginado una consulta Eloquent tradicional](/docs/{{version}}/pagination):
> > In addition to retrieving a collection of models, you may paginate your search results using the `paginate` method. This method will return a `Paginator` instance just as if you had [paginated a traditional Eloquent query](/docs/{{version}}/pagination):

    $orders = App\Order::search('Star Trek')->paginate();

Puede especificar cuántos modelos recuperar por página al pasar la cantidad como primer argumento al método `paginate`:
> > You may specify how many models to retrieve per page by passing the amount as the first argument to the `paginate` method:

    $orders = App\Order::search('Star Trek')->paginate(15);

Una vez que haya recuperado los resultados, puede mostrar los resultados y renderizar los enlaces de la página utilizando [Blade](/docs/{{version}}/blade) como si hubiera paginado una consulta Eloquent tradicional:
> > Once you have retrieved the results, you may display the results and render the page links using [Blade](/docs/{{version}}/blade) just as if you had paginated a traditional Eloquent query:

    <div class="container">
        @foreach ($orders as $order)
            {{ $order->price }}
        @endforeach
    </div>

    {{ $orders->links() }}

<a name="soft-deleting"></a>
### Soft Deleting

Si sus modelos indexados son [borrado suave](/docs/{{version}}/eloquent#soft-deleting) y necesita buscar sus modelos borrados, configure la opción `soft_delete` del archivo de configuración `config/scout.php` a `true`:
> > If your indexed models are [soft deleting](/docs/{{version}}/eloquent#soft-deleting) and you need to search your soft deleted models, set the `soft_delete` option of the `config/scout.php` configuration file to `true`:

    'soft_delete' => true,

Cuando esta opción de configuración es `true`, Scout no eliminará los modelos borrados suaves del índice de búsqueda. En cambio, establecerá un atributo oculto `__soft_deleted` en el registro indexado. Luego, puede usar los métodos `withTrashed` o` onlyTrashed` para recuperar los registros borrados suaves al buscar:
> > When this configuration option is `true`, Scout will not remove soft deleted models from the search index. Instead, it will set a hidden `__soft_deleted` attribute on the indexed record. Then, you may use the `withTrashed` or `onlyTrashed` methods to retrieve the soft deleted records when searching:

    // Include trashed records when retrieving results...
    $orders = App\Order::withTrashed()->search('Star Trek')->get();

    // Only include trashed records when retrieving results...
    $orders = App\Order::onlyTrashed()->search('Star Trek')->get();

> {tip} Cuando un modelo borrado suave se elimina permanentemente usando `forceDelete`, Scout lo eliminará automáticamente del índice de búsqueda.
> > > {tip} When a soft deleted model is permanently deleted using `forceDelete`, Scout will remove it from the search index automatically.

<a name="custom-engines"></a>
## Motores personalizados : Custom Engines

#### Escribiendo el motor : Writing The Engine

Si uno de los motores de búsqueda Scout integrados no se ajusta a sus necesidades, puede escribir su propio motor personalizado y registrarlo con Scout. Su motor debería extender la clase abstracta `Laravel\Scout\Engines\Engine`. Esta clase abstracta contiene siete métodos que su motor personalizado debe implementar:
> > If one of the built-in Scout search engines doesn't fit your needs, you may write your own custom engine and register it with Scout. Your engine should extend the `Laravel\Scout\Engines\Engine` abstract class. This abstract class contains seven methods your custom engine must implement:

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function mapIds($results);
    abstract public function map($results, $model);
    abstract public function getTotalCount($results);

Puede que le resulte útil revisar las implementaciones de estos métodos en la clase `Laravel\Scout\Engines\AlgoliaEngine`. Esta clase le proporcionará un buen punto de partida para aprender a implementar cada uno de estos métodos en su propio motor.
> > You may find it helpful to review the implementations of these methods on the `Laravel\Scout\Engines\AlgoliaEngine` class. This class will provide you with a good starting point for learning how to implement each of these methods in your own engine.

#### Registrar el motor : Registering The Engine

Una vez que haya escrito su motor personalizado, puede registrarlo con Scout usando el método `extend` del administrador del motor Scout. Debe llamar al método `extend` desde el método `boot` de su `AppServiceProvider` o cualquier otro proveedor de servicios utilizado por su aplicación. Por ejemplo, si ha escrito un `MySqlSearchEngine`, puede registrarlo así:
> > Once you have written your custom engine, you may register it with Scout using the `extend` method of the Scout engine manager. You should call the `extend` method from the `boot` method of your `AppServiceProvider` or any other service provider used by your application. For example, if you have written a `MySqlSearchEngine`, you may register it like so:

    use Laravel\Scout\EngineManager;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }

Una vez que su motor ha sido registrado, puede especificarlo como su "controlador" Scout predeterminado en su archivo de configuración `config/scout.php`:
> > Once your engine has been registered, you may specify it as your default Scout `driver` in your `config/scout.php` configuration file:

    'driver' => 'mysql',
