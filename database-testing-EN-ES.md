# Prueba de base de datos : Database Testing

- [Introduction](#introduction)
- [Generating Factories](#generating-factories)
- [Resetting The Database After Each Test](#resetting-the-database-after-each-test)
- [Writing Factories](#writing-factories)
    - [Factory States](#factory-states)
    - [Factory Callbacks](#factory-callbacks)
- [Using Factories](#using-factories)
    - [Creating Models](#creating-models)
    - [Persisting Models](#persisting-models)
    - [Relationships](#relationships)
- [Available Assertions](#available-assertions)

<a name="introduction"></a>
## Introducción : Introduction

Laravel proporciona una variedad de herramientas útiles para que sea más fácil probar sus aplicaciones basadas en bases de datos. Primero, puede usar el helper `assertDatabaseHas` para afirmar que los datos existen en la base de datos que coinciden con un conjunto determinado de criterios. Por ejemplo, si desea verificar que hay un registro en la tabla `users` con el valor `email` de `sally@example.com`, puede hacer lo siguiente:
> > Laravel provides a variety of helpful tools to make it easier to test your database driven applications. First, you may use the `assertDatabaseHas` helper to assert that data exists in the database matching a given set of criteria. For example, if you would like to verify that there is a record in the `users` table with the `email` value of `sally@example.com`, you can do the following:

    public function testDatabase()
    {
        // Make call to application...

        $this->assertDatabaseHas('users', [
            'email' => 'sally@example.com'
        ]);
    }

También puede usar el ayudante `assertDatabaseMissing` para afirmar que los datos no existen en la base de datos.
> > You can also use the `assertDatabaseMissing` helper to assert that data does not exist in the database.

Por supuesto, el método `assertDatabaseHas` y otros ayudantes como este son por conveniencia. Puede usar cualquiera de los métodos de aserción incorporados de PHPUnit para complementar sus pruebas.
> > Of course, the `assertDatabaseHas` method and other helpers like it are for convenience. You are free to use any of PHPUnit's built-in assertion methods to supplement your tests.

<a name="generating-factories"></a>
## Generación de fábricas : Generating Factories

Para crear una fábrica, use `make:factory` [Artisan command](/docs/{{version}}/artisan):
> > To create a factory, use the `make:factory` [Artisan command](/docs/{{version}}/artisan):

    php artisan make:factory PostFactory

La nueva fábrica se colocará en el directorio `database/factories`.
The new factory will be placed in your `database/factories` directory.

La opción `--model` se puede usar para indicar el nombre del modelo creado por la fábrica. Esta opción completará previamente el archivo de fábrica generado con el modelo dado:
> > The `--model` option may be used to indicate the name of the model created by the factory. This option will pre-fill the generated factory file with the given model:

    php artisan make:factory PostFactory --model=Post

<a name="resetting-the-database-after-each-test"></a>
## Restablecer la base de datos después de cada prueba : Resetting The Database After Each Test

A menudo es útil restablecer su base de datos después de cada prueba para que los datos de una prueba anterior no interfieran con las pruebas posteriores. El rasgo `RefreshDatabase` toma el enfoque más óptimo para migrar su base de datos de prueba dependiendo de si está usando una base de datos en memoria o una base de datos tradicional. Use el rasgo en su clase de prueba y todo se manejará por usted:
> > It is often useful to reset your database after each test so that data from a previous test does not interfere with subsequent tests. The `RefreshDatabase` trait takes the most optimal approach to migrating your test database depending on if you are using an in-memory database or a traditional database. Use the trait on your test class and everything will be handled for you:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->get('/');

            // ...
        }
    }

<a name="writing-factories"></a>
## Escritura de fábricas : Writing Factories

Al realizar pruebas, es posible que deba insertar algunos registros en su base de datos antes de ejecutar su prueba. En lugar de especificar manualmente el valor de cada columna cuando crea estos datos de prueba, Laravel le permite definir un conjunto predeterminado de atributos para cada uno de sus [modelos Eloquent](/docs/{{version}}/eloquent) utilizando fábricas de modelos. Para comenzar, eche un vistazo al archivo `database/factories/UserFactory.php` en su aplicación. Fuera de la caja, este archivo contiene una definición de fábrica:
> > When testing, you may need to insert a few records into your database before executing your test. Instead of manually specifying the value of each column when you create this test data, Laravel allows you to define a default set of attributes for each of your [Eloquent models](/docs/{{version}}/eloquent) using model factories. To get started, take a look at the `database/factories/UserFactory.php` file in your application. Out of the box, this file contains one factory definition:

    use Faker\Generator as Faker;

    $factory->define(App\User::class, function (Faker $faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->unique()->safeEmail,
            'password' => '$2y$10$TKh8H1.PfQx37YgCzwiKb.KjNyWgaHb9cbcoQgdIVFlYg7B77UdFm', // secret
            'remember_token' => str_random(10),
        ];
    });

Dentro del Cierre, que sirve como definición de fábrica, puede devolver los valores de prueba predeterminados de todos los atributos en el modelo. El cierre recibirá una instancia de la librería PHP [Faker](https://github.com/fzaninotto/Faker), que le permite generar convenientemente varios tipos de datos aleatorios para las pruebas.
> > Within the Closure, which serves as the factory definition, you may return the default test values of all attributes on the model. The Closure will receive an instance of the [Faker](https://github.com/fzaninotto/Faker) PHP library, which allows you to conveniently generate various kinds of random data for testing.

También puede crear archivos de fábrica adicionales para cada modelo para una mejor organización. Por ejemplo, podría crear archivos `UserFactory.php` y `CommentFactory.php` en su directorio `database/factories`. Todos los archivos dentro del directorio `factories` serán cargados automáticamente por Laravel.
> > You may also create additional factory files for each model for better organization. For example, you could create `UserFactory.php` and `CommentFactory.php` files within your `database/factories` directory. All of the files within the `factories` directory will automatically be loaded by Laravel.

> {tip} Puede establecer la configuración regional de Faker agregando una opción `faker_locale` a su archivo de configuración `config/app.php`.
> > > {tip} You can set the Faker locale by adding a `faker_locale` option to your `config/app.php` configuration file.

<a name="factory-states"></a>
### Estados de fábrica : Factory States

Los estados le permiten definir modificaciones discretas que se pueden aplicar a sus fábricas modelo en cualquier combinación. Por ejemplo, su modelo `User` podría tener un estado `delinquent` que modifique uno de sus valores de atributo predeterminados. Puede definir sus transformaciones de estado usando el método `state`. Para estados simples, puede pasar un array de modificaciones de atributos:
> > States allow you to define discrete modifications that can be applied to your model factories in any combination. For example, your `User` model might have a `delinquent` state that modifies one of its default attribute values. You may define your state transformations using the `state` method. For simple states, you may pass an array of attribute modifications:

    $factory->state(App\User::class, 'delinquent', [
        'account_status' => 'delinquent',
    ]);

Si su estado requiere un cálculo o una instancia `$faker`, puede usar un Cierre para calcular las modificaciones de atributo del estado:
> > If your state requires calculation or a `$faker` instance, you may use a Closure to calculate the state's attribute modifications:

    $factory->state(App\User::class, 'address', function ($faker) {
        return [
            'address' => $faker->address,
        ];
    });

<a name="factory-callbacks"></a>
### Rellamadas de fábrica : Factory Callbacks

Las rellamadas de fábrica se registran utilizando los métodos `afterMaking` y `afterCreating`, y le permiten realizar tareas adicionales después de crear o crear un modelo. Por ejemplo, puede usar rellamadas para relacionar modelos adicionales con el modelo creado:
> > Factory callbacks are registered using the `afterMaking` and `afterCreating` methods, and allow you to perform additional tasks after making or creating a model. For example, you may use callbacks to relate additional models to the created model:

    $factory->afterMaking(App\User::class, function ($user, $faker) {
        // ...
    });

    $factory->afterCreating(App\User::class, function ($user, $faker) {
        $user->accounts()->save(factory(App\Account::class)->make());
    });

También puede definir rellamadas para [estados de fábrica](#factory-states):
> > You may also define callbacks for [factory states](#factory-states):

    $factory->afterMakingState(App\User::class, 'delinquent', function ($user, $faker) {
        // ...
    });

    $factory->afterCreatingState(App\User::class, 'delinquent', function ($user, $faker) {
        // ...
    });

<a name="using-factories"></a>
## Utilizando fábricas : Using Factories

<a name="creating-models"></a>
### Creación de modelos : Creating Models

Una vez que haya definido sus fábricas, puede usar la función global `factory` en sus pruebas o archivos semilla para generar instancias modelo. Entonces, echemos un vistazo a algunos ejemplos de creación de modelos. Primero, usaremos el método `make` para crear modelos pero no los guardaremos en la base de datos:
> > Once you have defined your factories, you may use the global `factory` function in your tests or seed files to generate model instances. So, let's take a look at a few examples of creating models. First, we'll use the `make` method to create models but not save them to the database:

    public function testDatabase()
    {
        $user = factory(App\User::class)->make();

        // Use model in tests...
    }

También puede crear una Colección de muchos modelos o crear modelos de un tipo determinado:
> > You may also create a Collection of many models or create models of a given type:

    // Create three App\User instances...
    $users = factory(App\User::class, 3)->make();

#### Aplicando estados : Applying States

También puede aplicar cualquiera de sus [estados](#factory-states) a los modelos. Si desea aplicar transformaciones de estado múltiples a los modelos, debe especificar el nombre de cada estado que desea aplicar:
> > You may also apply any of your [states](#factory-states) to the models. If you would like to apply multiple state transformations to the models, you should specify the name of each state you would like to apply:

    $users = factory(App\User::class, 5)->states('delinquent')->make();

    $users = factory(App\User::class, 5)->states('premium', 'delinquent')->make();

#### Atributos principales : Overriding Attributes

Si desea anular algunos de los valores predeterminados de sus modelos, puede pasar un array de valores al método `make`. Solo los valores especificados serán reemplazados mientras el resto de los valores permanecen establecidos en sus valores predeterminados según lo especificado por la fábrica:
> > If you would like to override some of the default values of your models, you may pass an array of values to the `make` method. Only the specified values will be replaced while the rest of the values remain set to their default values as specified by the factory:

    $user = factory(App\User::class)->make([
        'name' => 'Abigail',
    ]);

<a name="persisting-models"></a>
### Modelos persistentes : Persisting Models

El método `create` no solo crea las instancias del modelo sino que también las guarda en la base de datos utilizando el método `save` de Eloquent:
> > The `create` method not only creates the model instances but also saves them to the database using Eloquent's `save` method:

    public function testDatabase()
    {
        // Create a single App\User instance...
        $user = factory(App\User::class)->create();

        // Create three App\User instances...
        $users = factory(App\User::class, 3)->create();

        // Use model in tests...
    }

Puede anular atributos en el modelo pasando un array al método `create`:
> > You may override attributes on the model by passing an array to the `create` method:

    $user = factory(App\User::class)->create([
        'name' => 'Abigail',
    ]);

<a name="relationships"></a>
### Relaciones : Relationships

En este ejemplo, adjuntaremos una relación a algunos modelos creados. Cuando se utiliza el método `create` para crear varios modelos, se devuelve una [instancia de colección](/docs/{{version}}/eloquent-collections) Eloquent, que le permite utilizar cualquiera de las funciones convenientes proporcionadas por la colección, tales como `each`:
> > In this example, we'll attach a relation to some created models. When using the `create` method to create multiple models, an Eloquent [collection instance](/docs/{{version}}/eloquent-collections) is returned, allowing you to use any of the convenient functions provided by the collection, such as `each`:

    $users = factory(App\User::class, 3)
               ->create()
               ->each(function ($u) {
                    $u->posts()->save(factory(App\Post::class)->make());
                });

#### Cierres de relaciones y atributos : Relations & Attribute Closures

También puede adjuntar relaciones a modelos utilizando los atributos de Cierre en las definiciones de fábrica. Por ejemplo, si desea crear una nueva instancia de `User` al crear un `Post`, puede hacer lo siguiente:
> > You may also attach relationships to models using Closure attributes in your factory definitions. For example, if you would like to create a new `User` instance when creating a `Post`, you may do the following:

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            }
        ];
    });

Estos cierres también reciben el array de atributos evaluados de la fábrica que los define:
> > These Closures also receive the evaluated attribute array of the factory that defines them:

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            },
            'user_type' => function (array $post) {
                return App\User::find($post['user_id'])->type;
            }
        ];
    });

<a name="available-assertions"></a>
## Afirmaciones disponibles : Available Assertions

Laravel provides several database assertions for your [PHPUnit](https://phpunit.de/) tests:

Method  | Description
------------- | -------------
`$this->assertDatabaseHas($table, array $data);`  |  Confirma que una tabla en la base de datos contiene los datos proporcionados.
  |  Assert that a table in the database contains the given data.
`$this->assertDatabaseMissing($table, array $data);`  |  Confirma que una tabla en la base de datos no contiene los datos proporcionados
  |  Assert that a table in the database does not contain the given data.
`$this->assertSoftDeleted($table, array $data);`  |  Confirma que el registro dado ha sido eliminado suave  |  Assert that the given record has been soft deleted.
