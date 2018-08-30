#  Eloquent: serialización : Eloquent: Serialization

- [Introduction](#introduction)
- [Serializing Models & Collections](#serializing-models-and-collections)
    - [Serializing To Arrays](#serializing-to-arrays)
    - [Serializing To JSON](#serializing-to-json)
- [Hiding Attributes From JSON](#hiding-attributes-from-json)
- [Appending Values To JSON](#appending-values-to-json)
- [Date Serialization](#date-serialization)

<a name="introduction"></a>
## Introducción : Introduction

Al construir APIs JSON, a menudo necesitará convertir sus modelos y relaciones a arrays o JSON. Eloquent incluye métodos convenientes para realizar estas conversiones, así como controlar qué atributos se incluyen en sus serializaciones.
> > When building JSON APIs, you will often need to convert your models and relationships to arrays or JSON. Eloquent includes convenient methods for making these conversions, as well as controlling which attributes are included in your serializations.

<a name="serializing-models-and-collections"></a>
## Serialización de modelos y colecciones : Serializing Models & Collections

<a name="serializing-to-arrays"></a>
### Serialización en arrays : Serializing To Arrays

Para convertir un modelo y sus [relaciones](/docs/{{version}}/eloquent-relationships) cargadas en un array, debe usar el método `toArray`. Este método es recursivo, por lo que todos los atributos y todas las relaciones (incluidas las relaciones de relaciones) se convertirán e arrays:
> > To convert a model and its loaded [relationships](/docs/{{version}}/eloquent-relationships) to an array, you should use the `toArray` method. This method is recursive, so all attributes and all relations (including the relations of relations) will be converted to arrays:

    $user = App\User::with('roles')->first();

    return $user->toArray();

También puede convertir [colecciones](/docs/{{version}}/eloquent-collections) completas de modelos en arrays:
> > You may also convert entire [collections](/docs/{{version}}/eloquent-collections) of models to arrays:

    $users = App\User::all();

    return $users->toArray();

<a name="serializing-to-json"></a>
### Serialización a JSON : Serializing To JSON

Para convertir un modelo a JSON, debe usar el método `toJson`. Al igual que `toArray`, el método` toJson` es recursivo, por lo que todos los atributos y relaciones se convertirán en JSON. También puede especificar las opciones de codificación JSON [compatibles con PHP](http://php.net/manual/en/function.json-encode.php):
> > To convert a model to JSON, you should use the `toJson` method. Like `toArray`, the `toJson` method is recursive, so all attributes and relations will be converted to JSON. You may also specify JSON encoding options [supported by PHP](http://php.net/manual/en/function.json-encode.php):

    $user = App\User::find(1);

    return $user->toJson();

    return $user->toJson(JSON_PRETTY_PRINT);

Alternativamente, puede lanzar un modelo o colección a una cadena, que automáticamente llamará al método `toJson` en el modelo o colección:
> > Alternatively, you may cast a model or collection to a string, which will automatically call the `toJson` method on the model or collection:

    $user = App\User::find(1);

    return (string) $user;

Como los modelos y colecciones se convierten a JSON cuando se convierten en una cadena, puede devolver objetos Eloquent directamente desde las rutas o controladores de la aplicación:
> > Since models and collections are converted to JSON when cast to a string, you can return Eloquent objects directly from your application's routes or controllers:

    Route::get('users', function () {
        return App\User::all();
    });

<a name="hiding-attributes-from-json"></a>
## Ocultar atributos de JSON : Hiding Attributes From JSON

En ocasiones, es posible que desee limitar los atributos, como las contraseñas, que se incluyen en la matriz de su modelo o la representación JSON. Para hacerlo, agrega una propiedad `$hidden` a tu modelo:
> > Sometimes you may wish to limit the attributes, such as passwords, that are included in your model's array or JSON representation. To do so, add a `$hidden` property to your model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be hidden for arrays.
         *
         * @var array
         */
        protected $hidden = ['password'];
    }

> {note} Cuando oculte relaciones, use el nombre del método de la relación.
> > > {note} When hiding relationships, use the relationship's method name.

Alternativamente, puede usar la propiedad `visible` para definir una lista blanca de atributos que deberían incluirse en el array de su modelo y la representación JSON. Todos los demás atributos estarán ocultos cuando el modelo se convierta en un array o JSON:
> > Alternatively, you may use the `visible` property to define a white-list of attributes that should be included in your model's array and JSON representation. All other attributes will be hidden when the model is converted to an array or JSON:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be visible in arrays.
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }

#### Modificando temporalmente la visibilidad del atributo : Temporarily Modifying Attribute Visibility

Si desea hacer visibles algunos atributos normalmente ocultos en una instancia de modelo determinada, puede usar el método `makeVisible`. El método `makeVisible` devuelve la instancia del modelo para el encadenamiento de métodos conveniente:
> > If you would like to make some typically hidden attributes visible on a given model instance, you may use the `makeVisible` method. The `makeVisible` method returns the model instance for convenient method chaining:

    return $user->makeVisible('attribute')->toArray();

Del mismo modo, si desea ocultar algunos atributos normalmente visibles en una instancia determinada del modelo, puede utilizar el método `makeHidden`.
> > Likewise, if you would like to make some typically visible attributes hidden on a given model instance, you may use the `makeHidden` method.

    return $user->makeHidden('attribute')->toArray();

<a name="appending-values-to-json"></a>
## Agregar valores a JSON : Appending Values To JSON

Ocasionalmente, cuando se lanzan modelos a un array o JSON, es posible que desee agregar atributos que no tienen una columna correspondiente en su base de datos. Para hacerlo, primero defina un [accesor](/docs/{{version}}/eloquent-mutators) para el valor:
> > Occasionally, when casting models to an array or JSON, you may wish to add attributes that do not have a corresponding column in your database. To do so, first define an [accessor](/docs/{{version}}/eloquent-mutators) for the value:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the administrator flag for the user.
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] == 'yes';
        }
    }

Después de crear el accesor, agregue el nombre del atributo a la propiedad `appends` en el modelo. Tenga en cuenta que los nombres de los atributos se mencionan normalmente en "snake case", aunque el acceso se define con "camel case":
> > After creating the accessor, add the attribute name to the `appends` property on the model. Note that attribute names are typically referenced in "snake case", even though the accessor is defined using "camel case":

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The accessors to append to the model's array form.
         *
         * @var array
         */
        protected $appends = ['is_admin'];
    }

Una vez que el atributo se haya agregado a la lista de 'Anexos', se incluirá tanto en la matriz del modelo como en las representaciones de JSON. Los atributos en el array `appends` también respetarán las configuraciones `visible` y `hidden` configuradas en el modelo.
> > Once the attribute has been added to the `appends` list, it will be included in both the model's array and JSON representations. Attributes in the `appends` array will also respect the `visible` and `hidden` settings configured on the model.

#### Agregar en tiempo de ejecución : Appending At Run Time

Puede indicar a una única instancia de modelo que anexe atributos utilizando el método `append`. O bien, puede usar el método `setAppends` para anular toda la matriz de propiedades adjuntas para una instancia de modelo determinada:
> > You may instruct a single model instance to append attributes using the `append` method. Or, you may use the `setAppends` method to override the entire array of appended properties for a given model instance:

    return $user->append('is_admin')->toArray();

    return $user->setAppends(['is_admin'])->toArray();

<a name="date-serialization"></a>
## Serialización de fecha : Date Serialization

#### Personalización del formato de fecha por atributo : Customizing The Date Format Per Attribute

Puede personalizar el formato de serialización de atributos de fecha Eloquent individuales especificando el formato de fecha en la [declaración de lanzamiento](/docs/{{version}}/eloquent-mutators#attribute-casting):
> > You may customize the serialization format of individual Eloquent date attributes by specifying the date format in the [cast declaration](/docs/{{version}}/eloquent-mutators#attribute-casting):

    protected $casts = [
        'birthday' => 'date:Y-m-d',
        'joined_at' => 'datetime:Y-m-d H:00',
    ];

#### Personalización global mediante Carbon : Global Customization Via Carbon

Laravel amplía la librería de fechas [Carbon](https://github.com/briannesbitt/Carbon) para proporcionar una personalización conveniente del formato de serialización JSON de Carbon. Para personalizar cómo se serializan todas las fechas de Carbon en su aplicación, use el método `Carbon::serializeUsing`. El método `serializeUsing` acepta un Cierre que devuelve una representación de cadena de la fecha para la serialización JSON:
> > Laravel extends the [Carbon](https://github.com/briannesbitt/Carbon) date library in order to provide convenient customization of Carbon's JSON serialization format. To customize how all Carbon dates throughout your application are serialized, use the `Carbon::serializeUsing` method. The `serializeUsing` method accepts a Closure which returns a string representation of the date for JSON serialization:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Carbon;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Carbon::serializeUsing(function ($carbon) {
                return $carbon->format('U');
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
