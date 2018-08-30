# Eloquent: mutadores : Eloquent: Mutators

- [Introduction](#introduction)
- [Accessors & Mutators](#accessors-and-mutators)
    - [Defining An Accessor](#defining-an-accessor)
    - [Defining A Mutator](#defining-a-mutator)
- [Date Mutators](#date-mutators)
- [Attribute Casting](#attribute-casting)
    - [Array & JSON Casting](#array-and-json-casting)

<a name="introduction"></a>
## Introducción : Introduction

Los accesores y mutadores le permiten formatear valores de atributo Eloquent cuando los recupera o los configura en instancias de modelo. Por ejemplo, puede usar el [Encriptador de Laravel](/docs/{{version}}/encryption) para encriptar un valor mientras está almacenado en la base de datos, y luego descifrarlo automáticamente cuando acceda a él en un modelo Eloquent.
> > Accessors and mutators allow you to format Eloquent attribute values when you retrieve or set them on model instances. For example, you may want to use the [Laravel encrypter](/docs/{{version}}/encryption) to encrypt a value while it is stored in the database, and then automatically decrypt the attribute when you access it on an Eloquent model.

Además de los accesores y mutadores personalizados, Eloquent también puede crear automáticamente campos de fecha en instancias [Carbon](https://github.com/briannesbitt/Carbon) o incluso [enviar campos de texto a JSON](#attribute-casting).
> > In addition to custom accessors and mutators, Eloquent can also automatically cast date fields to [Carbon](https://github.com/briannesbitt/Carbon) instances or even [cast text fields to JSON](#attribute-casting).

<a name="accessors-and-mutators"></a>
## Accesores & Mutadores : Accessors & Mutators

<a name="defining-an-accessor"></a>
### Definición de un descriptor de acceso : Defining An Accessor

Para definir un descriptor de acceso, cree un método `getFooAttribute` en su modelo donde `Foo` es el nombre de la columna "studly" a la que desea acceder. En este ejemplo, definiremos un descriptor de acceso para el atributo `first_name`. Eloquent llamará automáticamente al descriptor de acceso cuando intente recuperar el valor del atributo `first_name`:
> > To define an accessor, create a `getFooAttribute` method on your model where `Foo` is the "studly" cased name of the column you wish to access. In this example, we'll define an accessor for the `first_name` attribute. The accessor will automatically be called by Eloquent when attempting to retrieve the value of the `first_name` attribute:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the user's first name.
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }

Como puede ver, el valor original de la columna se pasa al descriptor de acceso, lo que le permite manipular y devolver el valor. Para acceder al valor del acceso, puede acceder al atributo `first_name` en una instancia modelo:
> > As you can see, the original value of the column is passed to the accessor, allowing you to manipulate and return the value. To access the value of the accessor, you may access the `first_name` attribute on a model instance:

    $user = App\User::find(1);

    $firstName = $user->first_name;

Por supuesto, también puede usar accesores para devolver nuevos valores calculados de los atributos existentes:
> > Of course, you may also use accessors to return new, computed values from existing attributes:

    /**
     * Get the user's full name.
     *
     * @return string
     */
    public function getFullNameAttribute()
    {
        return "{$this->first_name} {$this->last_name}";
    }

<a name="defining-a-mutator"></a>
### Definición de un mutador : Defining A Mutator

Para definir un mutador, defina un método `setFooAttribute` en su modelo donde `Foo` es el nombre de la columna "studly" a la que desea acceder. Entonces, de nuevo, definamos un mutador para el atributo `first_name`. Este mutador se llamará automáticamente cuando intentemos establecer el valor del atributo `first_name` en el modelo:
> > To define a mutator, define a `setFooAttribute` method on your model where `Foo` is the "studly" cased name of the column you wish to access. So, again, let's define a mutator for the `first_name` attribute. This mutator will be automatically called when we attempt to set the value of the `first_name` attribute on the model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Set the user's first name.
         *
         * @param  string  $value
         * @return void
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }

El mutador recibirá el valor que se establece en el atributo, lo que le permite manipular el valor y establecer el valor manipulado en la propiedad interna `$attributes` del modelo Eloquent. Entonces, por ejemplo, si intentamos establecer el atributo `first_name` en `Sally`:
> > The mutator will receive the value that is being set on the attribute, allowing you to manipulate the value and set the manipulated value on the Eloquent model's internal `$attributes` property. So, for example, if we attempt to set the `first_name` attribute to `Sally`:

    $user = App\User::find(1);

    $user->first_name = 'Sally';

En este ejemplo, se llamará a la función `setFirstNameAttribute` con el valor `Sally`. El mutador aplicará entonces la función `strtolower` al nombre y establecerá su valor resultante en la matriz interna `$attributes`.
> > In this example, the `setFirstNameAttribute` function will be called with the value `Sally`. The mutator will then apply the `strtolower` function to the name and set its resulting value in the internal `$attributes` array.

<a name="date-mutators"></a>
## Mutadores de fechas : Date Mutators

Por defecto, Eloquent convertirá las columnas `created_at` y `updated_at` en instancias de [Carbon](https://github.com/briannesbitt/Carbon), que extiende la clase PHP `DateTime` para proporcionar una variedad de métodos útiles. Puede personalizar las fechas que se mutarán automáticamente e incluso desactivar completamente esta mutación anulando la propiedad `$dates` de su modelo:
> > By default, Eloquent will convert the `created_at` and `updated_at` columns to instances of [Carbon](https://github.com/briannesbitt/Carbon), which extends the PHP `DateTime` class to provide an assortment of helpful methods. You may customize which dates are automatically mutated, and even completely disable this mutation, by overriding the `$dates` property of your model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be mutated to dates.
         *
         * @var array
         */
        protected $dates = [
            'created_at',
            'updated_at',
            'deleted_at'
        ];
    }

Cuando una columna se considera una fecha, puede establecer su valor en una marca de tiempo UNIX, cadena de fecha (`Ymd`), cadena de fecha y hora, y por supuesto una instancia `DateTime` / `Carbon`, y el valor de la fecha será automáticamente estar correctamente almacenado en su base de datos:
> > When a column is considered a date, you may set its value to a UNIX timestamp, date string (`Y-m-d`), date-time string, and of course a `DateTime` / `Carbon` instance, and the date's value will automatically be correctly stored in your database:

    $user = App\User::find(1);

    $user->deleted_at = now();

    $user->save();

Como se señaló anteriormente, al recuperar los atributos que se enumeran en su propiedad `$dates`, se enviarán automáticamente a las instancias de [Carbon](https://github.com/briannesbitt/Carbon), lo que le permite utilizar cualquiera de los métodos de Carbon en tus atributos:
> > As noted above, when retrieving attributes that are listed in your `$dates` property, they will automatically be cast to [Carbon](https://github.com/briannesbitt/Carbon) instances, allowing you to use any of Carbon's methods on your attributes:

    $user = App\User::find(1);

    return $user->deleted_at->getTimestamp();

#### Formatos de fecha : Date Formats

Por defecto, las marcas de tiempo tienen el formato `'Y-m-d H:i:s'`. Si necesita personalizar el formato de marca de tiempo, configure la propiedad `$dateFormat` en su modelo. Esta propiedad determina cómo se almacenan los atributos de fecha en la base de datos, así como su formato cuando el modelo se serializa en una matriz o JSON:
> > By default, timestamps are formatted as `'Y-m-d H:i:s'`. If you need to customize the timestamp format, set the `$dateFormat` property on your model. This property determines how date attributes are stored in the database, as well as their format when the model is serialized to an array or JSON:

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

<a name="attribute-casting"></a>
## Atributo de Casting : Attribute Casting

La propiedad `$casts` en su modelo proporciona un método conveniente para convertir atributos a tipos de datos comunes. La propiedad `$casts` debe ser un array donde la clave es el nombre del atributo que se está emitiendo y el valor es el tipo al que se desea asignarle la columna. Los tipos de conversión compatibles son: `integer`, `real`, `float`, `double`, `string`, `boolean`, `object`, `array`, `collection`, `date`, `datetime`, y `timestamp`.
> > The `$casts` property on your model provides a convenient method of converting attributes to common data types. The `$casts` property should be an array where the key is the name of the attribute being cast and the value is the type you wish to cast the column to. The supported cast types are: `integer`, `real`, `float`, `double`, `string`, `boolean`, `object`, `array`, `collection`, `date`, `datetime`, and `timestamp`.

Por ejemplo, vamos a convertir el atributo `is_admin`, que está almacenado en nuestra base de datos como un entero (`0` o `1`) a un valor booleano:
> > For example, let's cast the `is_admin` attribute, which is stored in our database as an integer (`0` or `1`) to a boolean value:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast to native types.
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }

Ahora el atributo `is_admin` siempre se lanzará a un booleano cuando acceda a él, incluso si el valor subyacente se almacena en la base de datos como un entero:
> > Now the `is_admin` attribute will always be cast to a boolean when you access it, even if the underlying value is stored in the database as an integer:

    $user = App\User::find(1);

    if ($user->is_admin) {
        //
    }

<a name="array-and-json-casting"></a>
### Casting Array y JSON : Array & JSON Casting

El tipo de conversión `array` es particularmente útil cuando se trabaja con columnas almacenadas como JSON serializadas. Por ejemplo, si su base de datos tiene un tipo de campo `JSON` o `TEXT` que contiene JSON serializado, agregar el molde `array` a ese atributo deserializará automáticamente el atributo a un array PHP cuando acceda a él en su modelo Eloquent:
> > The `array` cast type is particularly useful when working with columns that are stored as serialized JSON. For example, if your database has a `JSON` or `TEXT` field type that contains serialized JSON, adding the `array` cast to that attribute will automatically deserialize the attribute to a PHP array when you access it on your Eloquent model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be cast to native types.
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

Una vez que se define la conversión, puede acceder al atributo `options` y se deserializará automáticamente de JSON a un array de PHP. Cuando establece el valor del atributo `options`, el array dado se serializará automáticamente en JSON para su almacenamiento:
> > Once the cast is defined, you may access the `options` attribute and it will automatically be deserialized from JSON into a PHP array. When you set the value of the `options` attribute, the given array will automatically be serialized back into JSON for storage:

    $user = App\User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();

<a name="date-casting"></a>
### Conversion de fecha : Date Casting

Al usar el tipo de conversión `date` o` datetime`, puede especificar el formato de la fecha. Este formato se usará cuando [el modelo se serialice en una matriz o JSON](/docs/{{version}}/eloquent-serialization):
> > When using the `date` or `datetime` cast type, you may specify the date's format. This format will be used when the [model is serialized to an array or JSON](/docs/{{version}}/eloquent-serialization):

    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = [
        'created_at' => 'datetime:Y-m-d',
    ];
