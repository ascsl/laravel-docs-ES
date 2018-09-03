# Ayudantes : Helpers

- [Introduction](#introduction)
- [Available Methods](#available-methods)

<a name="introduction"></a>
## Introducción : Introduction

Laravel incluye una variedad de funciones PHP globales "auxiliares". Muchas de estas funciones son utilizadas por el propio framework; sin embargo, puede usarlos en sus propias aplicaciones si lo considera conveniente.
> > Laravel includes a variety of global "helper" PHP functions. Many of these functions are used by the framework itself; however, you are free to use them in your own applications if you find them convenient.

<a name="available-methods"></a>
## Métodos disponibles : Available Methods

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

### Arrays y objetos : Arrays & Objects

<div class="collection-method-list" markdown="1">

[array_add](#method-array-add)
[array_collapse](#method-array-collapse)
[array_divide](#method-array-divide)
[array_dot](#method-array-dot)
[array_except](#method-array-except)
[array_first](#method-array-first)
[array_flatten](#method-array-flatten)
[array_forget](#method-array-forget)
[array_get](#method-array-get)
[array_has](#method-array-has)
[array_last](#method-array-last)
[array_only](#method-array-only)
[array_pluck](#method-array-pluck)
[array_prepend](#method-array-prepend)
[array_pull](#method-array-pull)
[array_random](#method-array-random)
[array_set](#method-array-set)
[array_sort](#method-array-sort)
[array_sort_recursive](#method-array-sort-recursive)
[array_where](#method-array-where)
[array_wrap](#method-array-wrap)
[data_fill](#method-data-fill)
[data_get](#method-data-get)
[data_set](#method-data-set)
[head](#method-head)
[last](#method-last)
</div>

### Sendas : Paths

<div class="collection-method-list" markdown="1">

[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[mix](#method-mix)
[public_path](#method-public-path)
[resource_path](#method-resource-path)
[storage_path](#method-storage-path)

</div>

### Strings

<div class="collection-method-list" markdown="1">

[\__](#method-__)
[camel_case](#method-camel-case)
[class_basename](#method-class-basename)
[e](#method-e)
[ends_with](#method-ends-with)
[kebab_case](#method-kebab-case)
[preg_replace_array](#method-preg-replace-array)
[snake_case](#method-snake-case)
[starts_with](#method-starts-with)
[str_after](#method-str-after)
[str_before](#method-str-before)
[str_contains](#method-str-contains)
[str_finish](#method-str-finish)
[str_is](#method-str-is)
[str_limit](#method-str-limit)
[Str::orderedUuid](#method-str-ordered-uuid)
[str_plural](#method-str-plural)
[str_random](#method-str-random)
[str_replace_array](#method-str-replace-array)
[str_replace_first](#method-str-replace-first)
[str_replace_last](#method-str-replace-last)
[str_singular](#method-str-singular)
[str_slug](#method-str-slug)
[str_start](#method-str-start)
[studly_case](#method-studly-case)
[title_case](#method-title-case)
[trans](#method-trans)
[trans_choice](#method-trans-choice)
[Str::uuid](#method-str-uuid)

</div>

### URLs

<div class="collection-method-list" markdown="1">

[action](#method-action)
[asset](#method-asset)
[secure_asset](#method-secure-asset)
[route](#method-route)
[secure_url](#method-secure-url)
[url](#method-url)

</div>

### Varios : Miscellaneous

<div class="collection-method-list" markdown="1">

[abort](#method-abort)
[abort_if](#method-abort-if)
[abort_unless](#method-abort-unless)
[app](#method-app)
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[blank](#method-blank)
[broadcast](#method-broadcast)
[cache](#method-cache)
[class_uses_recursive](#method-class-uses-recursive)
[collect](#method-collect)
[config](#method-config)
[cookie](#method-cookie)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[dd](#method-dd)
[decrypt](#method-decrypt)
[dispatch](#method-dispatch)
[dispatch_now](#method-dispatch-now)
[dump](#method-dump)
[encrypt](#method-encrypt)
[env](#method-env)
[event](#method-event)
[factory](#method-factory)
[filled](#method-filled)
[info](#method-info)
[logger](#method-logger)
[method_field](#method-method-field)
[now](#method-now)
[old](#method-old)
[optional](#method-optional)
[policy](#method-policy)
[redirect](#method-redirect)
[report](#method-report)
[request](#method-request)
[rescue](#method-rescue)
[resolve](#method-resolve)
[response](#method-response)
[retry](#method-retry)
[session](#method-session)
[tap](#method-tap)
[today](#method-today)
[throw_if](#method-throw-if)
[throw_unless](#method-throw-unless)
[trait_uses_recursive](#method-trait-uses-recursive)
[transform](#method-transform)
[validator](#method-validator)
[value](#method-value)
[view](#method-view)
[with](#method-with)

</div>

<a name="method-listing"></a>
## Listado de métodos : Method Listing

<style>
    #collection-method code {
        font-size: 14px;
    }

    #collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="arrays"></a>
## Arrays y objetos : Arrays & Objects

<a name="method-array-add"></a>
#### `array_add()` {#collection-method .first-collection-method}

La función `array_add` agrega un par clave / valor dado a un array si la clave dada no existe en el array:
> > The `array_add` function adds a given key / value pair to an array if the given key doesn't already exist in the array:

    $array = array_add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>
#### `array_collapse()` {#collection-method}

La función `array_collapse` contrae un array de arrays en un único array:
> > The `array_collapse` function collapses an array of arrays into a single array:

    $array = array_collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-divide"></a>
#### `array_divide()` {#collection-method}

La función `array_divide` devuelve dos arrays, una que contiene las claves y la otra que contiene los valores del array dado:
> > The `array_divide` function returns two arrays, one containing the keys, and the other containing the values of the given array:

    [$keys, $values] = array_divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `array_dot()` {#collection-method}

La función `array_dot` aplana un array multidimensional en un array de un solo nivel que usa la notación del "punto" para indicar la profundidad:
> > The `array_dot` function flattens a multi-dimensional array into a single level array that uses "dot" notation to indicate depth:

    $array = ['products' => ['desk' => ['price' => 100]]];

    $flattened = array_dot($array);

    // ['products.desk.price' => 100]

<a name="method-array-except"></a>
#### `array_except()` {#collection-method}

La función `array_except` elimina los pares clave / valor dados de un array:
> > The `array_except` function removes the given key / value pairs from an array:

    $array = ['name' => 'Desk', 'price' => 100];

    $filtered = array_except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-first"></a>
#### `array_first()` {#collection-method}

La función `array_first` devuelve el primer elemento de un array que pasa una prueba de verdad dada:
> > The `array_first` function returns the first element of an array passing a given truth test:

    $array = [100, 200, 300];

    $first = array_first($array, function ($value, $key) {
        return $value >= 150;
    });

    // 200

Un valor predeterminado también se puede pasar como tercer parámetro del método. Este valor será devuelto si ningún valor pasa la prueba de verdad:
> > A default value may also be passed as the third parameter to the method. This value will be returned if no value passes the truth test:

    $first = array_first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `array_flatten()` {#collection-method}

La función `array_flatten` aplana un array multidimensional en un array de un único nivel:
> > The `array_flatten` function flattens a multi-dimensional array into a single level array:

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $flattened = array_flatten($array);

    // ['Joe', 'PHP', 'Ruby']

<a name="method-array-forget"></a>
#### `array_forget()` {#collection-method}

La función `array_forget` elimina un par clave / valor dado de un array anidado usando la notación "punto":
> > The `array_forget` function removes a given key / value pair from a deeply nested array using "dot" notation:

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `array_get()` {#collection-method}

La función `array_get` recupera un valor de un array anidado usando la notación "punto":
> > The `array_get` function retrieves a value from a deeply nested array using "dot" notation:

    $array = ['products' => ['desk' => ['price' => 100]]];

    $price = array_get($array, 'products.desk.price');

    // 100

La función `array_get` también acepta un valor predeterminado, que se devolverá si no se encuentra la clave específica:
> > The `array_get` function also accepts a default value, which will be returned if the specific key is not found:

    $discount = array_get($array, 'products.desk.discount', 0);

    // 0

<a name="method-array-has"></a>
#### `array_has()` {#collection-method}

La función `array_has` comprueba si un elemento o elementos dados existe en un array usando la notación "punto":
> > The `array_has` function checks whether a given item or items exists in an array using "dot" notation:

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = array_has($array, 'product.name');

    // true

    $contains = array_has($array, ['product.price', 'product.discount']);

    // false

<a name="method-array-last"></a>
#### `array_last()` {#collection-method}

La función `array_last` devuelve el último elemento de un array que pasa una prueba de verdad dada:
> > The `array_last` function returns the last element of an array passing a given truth test:

    $array = [100, 200, 300, 110];

    $last = array_last($array, function ($value, $key) {
        return $value >= 150;
    });

    // 300

Puede pasarse un valor predeterminado como tercer argumento para el método. Este valor será devuelto si ningún valor pasa la prueba de verdad:
> > A default value may be passed as the third argument to the method. This value will be returned if no value passes the truth test:

    $last = array_last($array, $callback, $default);

<a name="method-array-only"></a>
#### `array_only()` {#collection-method}

La función `array_only` devuelve solo los pares clave / valor especificados del array dado:
> > The `array_only` function returns only the specified key / value pairs from the given array:

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $slice = array_only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `array_pluck()` {#collection-method}

La función `array_pluck` recupera todos los valores para una clave determinada de un array:
> > The `array_pluck` function retrieves all of the values for a given key from an array:

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $names = array_pluck($array, 'developer.name');

    // ['Taylor', 'Abigail']

También puede especificar cómo desea que se codifique la lista resultante:
> > You may also specify how you wish the resulting list to be keyed:

    $names = array_pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail']

<a name="method-array-prepend"></a>
#### `array_prepend()` {#collection-method}

The `array_prepend` function will push an item onto the beginning of an array:

    $array = ['one', 'two', 'three', 'four'];

    $array = array_prepend($array, 'zero');

    // ['zero', 'one', 'two', 'three', 'four']

If needed, you may specify the key that should be used for the value:

    $array = ['price' => 100];

    $array = array_prepend($array, 'Desk', 'name');

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pull"></a>
#### `array_pull()` {#collection-method}

La función `array_pull` devuelve y elimina un par de clave / valor de un array:
> > The `array_pull` function returns and removes a key / value pair from an array:

    $array = ['name' => 'Desk', 'price' => 100];

    $name = array_pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

Puede pasarse un valor predeterminado como tercer argumento para el método. Este valor se devolverá si la clave no existe:
> > A default value may be passed as the third argument to the method. This value will be returned if the key doesn't exist:

    $value = array_pull($array, $key, $default);

<a name="method-array-random"></a>
#### `array_random()` {#collection-method}

La función `array_random` devuelve un valor aleatorio de un array:
> > The `array_random` function returns a random value from an array:

    $array = [1, 2, 3, 4, 5];

    $random = array_random($array);

    // 4 - (retrieved randomly)

También puede especificar la cantidad de elementos a devolver como segundo argumento opcional. Tenga en cuenta que proporcionar este argumento devolverá un array, incluso si solo se desea un elemento:
> > You may also specify the number of items to return as an optional second argument. Note that providing this argument will return an array, even if only one item is desired:

    $items = array_random($array, 2);

    // [2, 5] - (retrieved randomly)

<a name="method-array-set"></a>
#### `array_set()` {#collection-method}

La función `array_set` establece un valor dentro de un array anidado usando la notación "punto":
> > The `array_set` function sets a value within a deeply nested array using "dot" notation:

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-sort"></a>
#### `array_sort()` {#collection-method}

La función `array_sort` ordena un array por sus valores:
> > The `array_sort` function sorts an array by its values:

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = array_sort($array);

    // ['Chair', 'Desk', 'Table']

También puede ordenar el array por los resultados del Closure dado:
> > You may also sort the array by the results of the given Closure:

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(array_sort($array, function ($value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
            ['name' => 'Table'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `array_sort_recursive()` {#collection-method}

La función `array_sort_recursive` ordena recursivamente un array usando la función `sort`:
> > The `array_sort_recursive` function recursively sorts an array using the `sort` function:

    $array = [
        ['Roman', 'Taylor', 'Li'],
        ['PHP', 'Ruby', 'JavaScript'],
    ];

    $sorted = array_sort_recursive($array);

    /*
        [
            ['Li', 'Roman', 'Taylor'],
            ['JavaScript', 'PHP', 'Ruby'],
        ]
    */

<a name="method-array-where"></a>
#### `array_where()` {#collection-method}

La función `array_where` filtra un array usando el Closure dado:
> > The `array_where` function filters an array using the given Closure:

    $array = [100, '200', 300, '400', 500];

    $filtered = array_where($array, function ($value, $key) {
        return is_string($value);
    });

    // [1 => '200', 3 => '400']

<a name="method-array-wrap"></a>
#### `array_wrap()` {#collection-method}

La función `array_wrap` envuelve el valor dado en un array. Si el valor dado ya es un array, no se cambiará:
> > The `array_wrap` function wraps the given value in an array. If the given value is already an array it will not be changed:

    $string = 'Laravel';

    $array = array_wrap($string);

    // ['Laravel']

Si el valor dado es nulo, se devolverá un array vacío:
> > If the given value is null, an empty array will be returned:

    $nothing = null;

    $array = array_wrap($nothing);

    // []

<a name="method-data-fill"></a>
#### `data_fill()` {#collection-method}

La función `data_fill` establece un valor perdido dentro de un array anidado o un objeto usando la notación de "punto":
> > The `data_fill` function sets a missing value within a nested array or object using "dot" notation:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_fill($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 100]]]

    data_fill($data, 'products.desk.discount', 10);

    // ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]

Esta función también acepta asteriscos como comodines y cumplirá el objetivo en consecuencia:
> > This function also accepts asterisks as wildcards and will fill the target accordingly:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2'],
        ],
    ];

    data_fill($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 100],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

<a name="method-data-get"></a>
#### `data_get()` {#collection-method}

La función `data_get` recupera un valor de un array anidado u objeto usando notación de "punto":
> > The `data_get` function retrieves a value from a nested array or object using "dot" notation:

    $data = ['products' => ['desk' => ['price' => 100]]];

    $price = data_get($data, 'products.desk.price');

    // 100

La función `data_get` también acepta un valor predeterminado, que se devolverá si no se encuentra la clave especificada:
> > The `data_get` function also accepts a default value, which will be returned if the specified key is not found:

    $discount = data_get($data, 'products.desk.discount', 0);

    // 0

<a name="method-data-set"></a>
#### `data_set()` {#collection-method}

La función `data_set` establece un valor dentro de un array anidado u objeto usando notación de "punto":
> > The `data_set` function sets a value within a nested array or object using "dot" notation:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

Esta función también acepta comodines y establecerá valores en el objetivo en consecuencia:
> > This function also accepts wildcards and will set values on the target accordingly:

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 150],
        ],
    ];

    data_set($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 200],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

De manera predeterminada, todos los valores existentes se sobrescriben. Si solo desea establecer un valor si no existe, puede pasar `false` como tercer argumento:
> > By default, any existing values are overwritten. If you wish to only set a value if it doesn't exist, you may pass `false` as the third argument:

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200, false);

    // ['products' => ['desk' => ['price' => 100]]]

<a name="method-head"></a>
#### `head()` {#collection-method}

La función `head` devuelve el primer elemento en el array dado:
> > The `head` function returns the first element in the given array:

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()` {#collection-method}

La función `last` devuelve el último elemento en el array dado:
> > The `last` function returns the last element in the given array:

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="paths"></a>
## Sendas : Paths

<a name="method-app-path"></a>
#### `app_path()` {#collection-method}

La función `app_path` devuelve la ruta completa al directorio `app`. También puede usar la función `app_path` para generar una ruta completa a un archivo relacionado con el directorio de la aplicación:
> > The `app_path` function returns the fully qualified path to the `app` directory. You may also use the `app_path` function to generate a fully qualified path to a file relative to the application directory:

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {#collection-method}

La función `base_path` devuelve la ruta completa a la raíz del proyecto. También puede usar la función `base_path` para generar una ruta de acceso completa a un archivo dado en relación con el directorio raíz del proyecto:
> > The `base_path` function returns the fully qualified path to the project root. You may also use the `base_path` function to generate a fully qualified path to a given file relative to the project root directory:

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {#collection-method}

La función `config_path` devuelve la ruta completa al directorio `config`. También puede usar la función `config_path` para generar una ruta completa a un archivo determinado dentro del directorio de configuración de la aplicación:
> > The `config_path` function returns the fully qualified path to the `config` directory. You may also use the `config_path` function to generate a fully qualified path to a given file within the application's configuration directory:

    $path = config_path();

    $path = config_path('app.php');

<a name="method-database-path"></a>
#### `database_path()` {#collection-method}

La función `database_path` devuelve la ruta completa al directorio `database`. También puede usar la función `database_path` para generar una ruta de acceso completa a un archivo determinado dentro del directorio de la base de datos:
> > The `database_path` function returns the fully qualified path to the `database` directory. You may also use the `database_path` function to generate a fully qualified path to a given file within the database directory:

    $path = database_path();

    $path = database_path('factories/UserFactory.php');

<a name="method-mix"></a>
#### `mix()` {#collection-method}

La función `mix` devuelve la ruta a un [archivo Mix](/docs/{{version}}/mix):
> > The `mix` function returns the path to a [versioned Mix file](/docs/{{version}}/mix):

    $path = mix('css/app.css');

<a name="method-public-path"></a>
#### `public_path()` {#collection-method}

La función `public_path` devuelve la ruta completa al directorio `public`. También puede usar la función `public_path` para generar una ruta completa a un archivo determinado dentro del directorio público:
> > The `public_path` function returns the fully qualified path to the `public` directory. You may also use the `public_path` function to generate a fully qualified path to a given file within the public directory:

    $path = public_path();

    $path = public_path('css/app.css');

<a name="method-resource-path"></a>
#### `resource_path()` {#collection-method}

La función `resource_path` devuelve la ruta completa al directorio `resources`. También puede usar la función `resource_path` para generar una ruta completa a un archivo dado dentro del directorio de recursos:
> > The `resource_path` function returns the fully qualified path to the `resources` directory. You may also use the `resource_path` function to generate a fully qualified path to a given file within the resources directory:

    $path = resource_path();

    $path = resource_path('assets/sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

La función `storage_path` devuelve la ruta completa al directorio `storage`. También puede usar la función `storage_path` para generar una ruta completa a un archivo dado dentro del directorio de almacenamiento:
> > The `storage_path` function returns the fully qualified path to the `storage` directory. You may also use the `storage_path` function to generate a fully qualified path to a given file within the storage directory:

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## Cadenas : Strings

<a name="method-__"></a>
#### `__()` {#collection-method}

La función `__` traduce la cadena de traducción dada o la clave de traducción usando sus [archivos de localización](/docs/{{version}}/localization):
> > The `__` function translates the given translation string or translation key using your [localization files](/docs/{{version}}/localization):

    echo __('Welcome to our application');

    echo __('messages.welcome');

Si la cadena o la clave de traducción especificada no existe, la función `__` devolverá el valor dado. Entonces, usando el ejemplo anterior, la función `__` devolvería `messages.welcome` si esa clave de traducción no existe.
> > If the specified translation string or key does not exist, the `__` function will return the given value. So, using the example above, the `__` function would return `messages.welcome` if that translation key does not exist.

<a name="method-camel-case"></a>
#### `camel_case()` {#collection-method}

La función `camel_case` convierte la cadena dada en `camelCase`:
> > The `camel_case` function converts the given string to `camelCase`:

    $converted = camel_case('foo_bar');

    // fooBar

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

El `class_basename` devuelve el nombre de clase de la clase dada con el Namespace de la clase eliminado:
> > The `class_basename` returns the class name of the given class with the class' namespace removed:

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {#collection-method}

La función `e` ejecuta la función `htmlspecialchars` de PHP con la opción `double_encode` establecida en `true` de manera predeterminada:
> > The `e` function runs PHP's `htmlspecialchars` function with the `double_encode` option set to `true` by default:

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-ends-with"></a>
#### `ends_with()` {#collection-method}

La función `ends_with` determina si la cadena dada termina con el valor dado:
> > The `ends_with` function determines if the given string ends with the given value:

    $result = ends_with('This is my name', 'name');

    // true

<a name="method-kebab-case"></a>
#### `kebab_case()` {#collection-method}

La función `kebab_case` convierte la cadena dada en` kebab-case`:
> > The `kebab_case` function converts the given string to `kebab-case`:

    $converted = kebab_case('fooBar');

    // foo-bar

<a name="method-preg-replace-array"></a>
#### `preg_replace_array()` {#collection-method}

La función `preg_replace_array` reemplaza un patrón dado en la cadena de forma secuencial utilizando un array:
> > The `preg_replace_array` function replaces a given pattern in the string sequentially using an array:

    $string = 'The event will take place between :start and :end';

    $replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-snake-case"></a>
#### `snake_case()` {#collection-method}

La función `snake_case` convierte la cadena dada en `snake_case`:
> > The `snake_case` function converts the given string to `snake_case`:

    $converted = snake_case('fooBar');

    // foo_bar

<a name="method-starts-with"></a>
#### `starts_with()` {#collection-method}

La función `starts_with` determina si la cadena dada comienza con el valor dado:
> > The `starts_with` function determines if the given string begins with the given value:

    $result = starts_with('This is my name', 'This');

    // true

<a name="method-str-after"></a>
#### `str_after()` {#collection-method}

La función `str_after` devuelve todo después del valor dado en una cadena:
> > The `str_after` function returns everything after the given value in a string:

    $slice = str_after('This is my name', 'This is');

    // ' my name'

<a name="method-str-before"></a>
#### `str_before()` {#collection-method}

La función `str_before` devuelve todo antes del valor dado en una cadena:
The `str_before` function returns everything before the given value in a string:

    $slice = str_before('This is my name', 'my name');

    // 'This is '

<a name="method-str-contains"></a>
#### `str_contains()` {#collection-method}

La función `str_contains` determina si la cadena dada contiene el valor dado (distingue entre mayúsculas y minúsculas):
> > The `str_contains` function determines if the given string contains the given value (case sensitive):

    $contains = str_contains('This is my name', 'my');

    // true

También puede pasar un array de valores para determinar si la cadena dada contiene alguno de los valores:
> > You may also pass an array of values to determine if the given string contains any of the values:

    $contains = str_contains('This is my name', ['my', 'foo']);

    // true

<a name="method-str-finish"></a>
#### `str_finish()` {#collection-method}

La función `str_finish` agrega una sola instancia del valor dado a una cadena si aún no termina con el valor:
> > The `str_finish` function adds a single instance of the given value to a string if it does not already end with the value:

    $adjusted = str_finish('this/string', '/');

    // this/string/

    $adjusted = str_finish('this/string/', '/');

    // this/string/

<a name="method-str-is"></a>
#### `str_is()` {#collection-method}

La función `str_is` determina si una cadena dada concuerda con un patrón dado. Los asteriscos se pueden usar para indicar comodines:
> > The `str_is` function determines if a given string matches a given pattern. Asterisks may be used to indicate wildcards:

    $matches = str_is('foo*', 'foobar');

    // true

    $matches = str_is('baz*', 'foobar');

    // false

<a name="method-str-limit"></a>
#### `str_limit()` {#collection-method}

La función `str_limit` trunca la cadena dada en la longitud especificada:
> > The `str_limit` function truncates the given string at the specified length:

    $truncated = str_limit('The quick brown fox jumps over the lazy dog', 20);

    // The quick brown fox...

También puede pasar un tercer argumento para cambiar la cadena que se agregará al final:
> > You may also pass a third argument to change the string that will be appended to the end:

    $truncated = str_limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

    // The quick brown fox (...)

<a name="method-str-ordered-uuid"></a>
#### `Str::orderedUuid()` {#collection-method}

El método `Str::orderedUuid` genera un UUID "timestamp first" que puede almacenarse de manera eficiente en una columna de base de datos indexada:
> > The `Str::orderedUuid` method generates a "timestamp first" UUID that may be efficiently stored in an indexed database column:

    use Illuminate\Support\Str;

    return (string) Str::orderedUuid();

<a name="method-str-plural"></a>
#### `str_plural()` {#collection-method}

La función `str_plural` convierte una cadena en su forma plural. Esta función actualmente solo es compatible con el idioma inglés:
> > The `str_plural` function converts a string to its plural form. This function currently only supports the English language:

    $plural = str_plural('car');

    // cars

    $plural = str_plural('child');

    // children

Puede proporcionar un número entero como segundo argumento a la función para recuperar la forma singular o plural de la cadena:
> > You may provide an integer as a second argument to the function to retrieve the singular or plural form of the string:

    $plural = str_plural('child', 2);

    // children

    $plural = str_plural('child', 1);

    // child

<a name="method-str-random"></a>
#### `str_random()` {#collection-method}

La función `str_random` genera una cadena aleatoria de la longitud especificada. Esta función usa la función `random_bytes` de PHP:
> > The `str_random` function generates a random string of the specified length. This function uses PHP's `random_bytes` function:

    $random = str_random(40);

<a name="method-str-replace-array"></a>
#### `str_replace_array()` {#collection-method}

La función `str_replace_array` reemplaza un valor dado en la cadena de forma secuencial utilizando un array:
> > The `str_replace_array` function replaces a given value in the string sequentially using an array:

    $string = 'The event will take place between ? and ?';

    $replaced = str_replace_array('?', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-replace-first"></a>
#### `str_replace_first()` {#collection-method}

La función `str_replace_first` reemplaza la primera aparición de un valor dado en una cadena:
> > The `str_replace_first` function replaces the first occurrence of a given value in a string:

    $replaced = str_replace_first('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // a quick brown fox jumps over the lazy dog

<a name="method-str-replace-last"></a>
#### `str_replace_last()` {#collection-method}

La función `str_replace_last` reemplaza la última ocurrencia de un valor dado en una cadena:
> > The `str_replace_last` function replaces the last occurrence of a given value in a string:

    $replaced = str_replace_last('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // the quick brown fox jumps over a lazy dog

<a name="method-str-singular"></a>
#### `str_singular()` {#collection-method}

La función `str_singular` convierte una cadena en su forma singular. Esta función actualmente solo es compatible con el idioma inglés:
> > The `str_singular` function converts a string to its singular form. This function currently only supports the English language:

    $singular = str_singular('cars');

    // car

    $singular = str_singular('children');

    // child

<a name="method-str-slug"></a>
#### `str_slug()` {#collection-method}

La función `str_slug` genera un "slug" amigable con la URL a partir de la cadena dada:
> > The `str_slug` function generates a URL friendly "slug" from the given string:

    $slug = str_slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-str-start"></a>
#### `str_start()` {#collection-method}

La función `str_start` agrega una única instancia del valor dado a una cadena si aún no comienza con el valor:
> > The `str_start` function adds a single instance of the given value to a string if it does not already start with the value:

    $adjusted = str_start('this/string', '/');

    // /this/string

    $adjusted = str_start('/this/string', '/');

    // /this/string

<a name="method-studly-case"></a>
#### `studly_case()` {#collection-method}

La función `studly_case` convierte la cadena dada en `StudlyCase`:
> > The `studly_case` function converts the given string to `StudlyCase`:

    $converted = studly_case('foo_bar');

    // FooBar

<a name="method-title-case"></a>
#### `title_case()` {#collection-method}

La función `title_case` convierte la cadena dada en `Title Case`:
> > The `title_case` function converts the given string to `Title Case`:

    $converted = title_case('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-trans"></a>
#### `trans()` {#collection-method}

La función `trans` traduce la clave de traducción dada usando sus [archivos de localización](/docs/{{version}}/localization):
> > The `trans` function translates the given translation key using your [localization files](/docs/{{version}}/localization):

    echo trans('messages.welcome');

Si la clave de traducción especificada no existe, la función `trans` devolverá la clave dada. Entonces, usando el ejemplo anterior, la función `trans` devolvería `messages.welcome` si la clave de traducción no existe.
> > If the specified translation key does not exist, the `trans` function will return the given key. So, using the example above, the `trans` function would return `messages.welcome` if the translation key does not exist.

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

La función `trans_choice` traduce la clave de traducción dada con inflexión:
> > The `trans_choice` function translates the given translation key with inflection:

    echo trans_choice('messages.notifications', $unreadCount);

Si la clave de traducción especificada no existe, la función `trans_choice` devolverá la clave dada. Entonces, usando el ejemplo anterior, la función `trans_choice` devolvería `messages.notifications` si la clave de traducción no existe.
> > If the specified translation key does not exist, the `trans_choice` function will return the given key. So, using the example above, the `trans_choice` function would return `messages.notifications` if the translation key does not exist.

<a name="method-str-uuid"></a>
#### `Str::uuid()` {#collection-method}

El método `Str::uuid` genera un UUID (versión 4):
> > The `Str::uuid` method generates a UUID (version 4):

    use Illuminate\Support\Str;

    return (string) Str::uuid();

<a name="urls"></a>
## URLs

<a name="method-action"></a>
#### `action()` {#collection-method}

La función `action` genera una URL para la acción de controlador dada. No necesita pasar el espacio de nombre completo del controlador. En su lugar, pase el nombre de la clase del controlador en relación con el Namespace `App\Http\Controllers`:
> > The `action` function generates a URL for the given controller action. You do not need to pass the full namespace of the controller. Instead, pass the controller class name relative to the `App\Http\Controllers` namespace:

    $url = action('HomeController@index');

    $url = action([HomeController::class, 'index']);

Si el método acepta parámetros de ruta, puede pasarlos como el segundo argumento del método:
> > If the method accepts route parameters, you may pass them as the second argument to the method:

    $url = action('UserController@profile', ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {#collection-method}

La función `asset` genera una URL para un asset usando el esquema actual de la solicitud (HTTP o HTTPS):
> > The `asset` function generates a URL for an asset using the current scheme of the request (HTTP or HTTPS):

    $url = asset('img/photo.jpg');

<a name="method-secure-asset"></a>
#### `secure_asset()` {#collection-method}

La función `secure_asset` genera una URL para un activo usando HTTPS:
> > The `secure_asset` function generates a URL for an asset using HTTPS:

    $url = secure_asset('img/photo.jpg');

<a name="method-route"></a>
#### `route()` {#collection-method}

La función `route` genera una URL para la ruta nombrada dada:
> > The `route` function generates a URL for the given named route:

    $url = route('routeName');

Si la ruta acepta parámetros, puede pasarlos como segundo argumento del método:
> > If the route accepts parameters, you may pass them as the second argument to the method:

    $url = route('routeName', ['id' => 1]);

Por defecto, la función `route` genera una URL absoluta. Si desea generar una URL relativa, puede pasar `false` como tercer argumento:
> > By default, the `route` function generates an absolute URL. If you wish to generate a relative URL, you may pass `false` as the third argument:

    $url = route('routeName', ['id' => 1], false);

<a name="method-secure-url"></a>
#### `secure_url()` {#collection-method}

La función `secure_url` genera una URL HTTPS totalmente calificada para la ruta dada:
> > The `secure_url` function generates a fully qualified HTTPS URL to the given path:

    $url = secure_url('user/profile');

    $url = secure_url('user/profile', [1]);

<a name="method-url"></a>
#### `url()` {#collection-method}

La función `url` genera una URL completa para la ruta dada:
> > The `url` function generates a fully qualified URL to the given path:

    $url = url('user/profile');

    $url = url('user/profile', [1]);

Si no se proporciona ninguna ruta, se devuelve una instancia de `Illuminate\Routing\UrlGenerator`:
> > If no path is provided, a `Illuminate\Routing\UrlGenerator` instance is returned:

    $current = url()->current();

    $full = url()->full();

    $previous = url()->previous();

<a name="miscellaneous"></a>
## Miscellaneous

<a name="method-abort"></a>
#### `abort()` {#collection-method}

La función `abort` arroja [una excepción HTTP](/docs/{{version}}/errors#http-exceptions) que será procesada por el [manejador de excepciones](/docs/{{version}}/errors#the-exception-handler):
> > The `abort` function throws [an HTTP exception](/docs/{{version}}/errors#http-exceptions) which will be rendered by the [exception handler](/docs/{{version}}/errors#the-exception-handler):

    abort(403);

También puede proporcionar el texto de respuesta de la excepción y los encabezados de respuesta personalizados:
> > You may also provide the exception's response text and custom response headers:

    abort(403, 'Unauthorized.', $headers);

<a name="method-abort-if"></a>
#### `abort_if()` {#collection-method}

La función `abort_if` arroja una excepción HTTP si una expresión booleana dada se evalúa como `true`:
> > The `abort_if` function throws an HTTP exception if a given boolean expression evaluates to `true`:

    abort_if(! Auth::user()->isAdmin(), 403);

Al igual que el método `abort`, también puede proporcionar el texto de respuesta de la excepción como tercer argumento y un array de encabezados de respuesta personalizados como cuarto argumento.
> > Like the `abort` method, you may also provide the exception's response text as the third argument and an array of custom response headers as the fourth argument.

<a name="method-abort-unless"></a>
#### `abort_unless()` {#collection-method}

La función `abort_unless` arroja una excepción HTTP si una expresión booleana dada se evalúa como `false`:
> > The `abort_unless` function throws an HTTP exception if a given boolean expression evaluates to `false`:

    abort_unless(Auth::user()->isAdmin(), 403);

Al igual que el método `abort`, también puede proporcionar el texto de respuesta de la excepción como tercer argumento y un array de encabezados de respuesta personalizados como cuarto argumento.
> > Like the `abort` method, you may also provide the exception's response text as the third argument and an array of custom response headers as the fourth argument.

<a name="method-app"></a>
#### `app()` {#collection-method}

La función `app` devuelve la instancia del [contenedor de servicio](/docs/{{version}}/container):
> > The `app` function returns the [service container](/docs/{{version}}/container) instance:

    $container = app();

Puede pasar un nombre de clase o interfaz para resolverlo desde el contenedor:
> > You may pass a class or interface name to resolve it from the container:

    $api = app('HelpSpot\API');

<a name="method-auth"></a>
#### `auth()` {#collection-method}

La función `auth` devuelve una instancia [authenticator](/docs/{{version}}/authentication). Puede usarlo en lugar de la fachada `Auth` para mayor comodidad:
> > The `auth` function returns an [authenticator](/docs/{{version}}/authentication) instance. You may use it instead of the `Auth` facade for convenience:

    $user = auth()->user();

Si es necesario, puede especificar a qué instancia de guardia le gustaría acceder:
> > If needed, you may specify which guard instance you would like to access:

    $user = auth('admin')->user();

<a name="method-back"></a>
#### `back()` {#collection-method}

La función `back` genera una [redireción de la respuesta HTTP](/docs/{{version}}/responses#redirects) a la ubicación anterior del usuario:
> > The `back` function generates a [redirect HTTP response](/docs/{{version}}/responses#redirects) to the user's previous location:

    return back($status = 302, $headers = [], $fallback = false);

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {#collection-method}

La función `bcrypt` [hashes](/docs/{{version}}/hashing) el valor dado usando Bcrypt. Puede usarlo como una alternativa a la fachada `Hash`:
> > The `bcrypt` function [hashes](/docs/{{version}}/hashing) the given value using Bcrypt. You may use it as an alternative to the `Hash` facade:

    $password = bcrypt('my-secret-password');

<a name="method-broadcast"></a>
#### `broadcast()` {#collection-method}

La función `broadcast` [difunde](/docs/{{version}}/broadcasting) el [evento](/docs/{{version}}/events) dado a sus oyentes:
> > The `broadcast` function [broadcasts](/docs/{{version}}/broadcasting) the given [event](/docs/{{version}}/events) to its listeners:

    broadcast(new UserRegistered($user));

<a name="method-blank"></a>
#### `blank()` {#collection-method}

La función `blank` devuelve si el valor dado es "en blanco":
> > The `blank` function returns whether the given value is "blank":

    blank('');
    blank('   ');
    blank(null);
    blank(collect());

    // true

    blank(0);
    blank(true);
    blank(false);

    // false

Para el inverso de `blank`, vea el método [`filled`](#method-filled).
> > For the inverse of `blank`, see the [`filled`](#method-filled) method.

<a name="method-cache"></a>
#### `cache()` {#collection-method}

La función `cache` se puede usar para obtener valores de [cache](/docs/{{version}}/cache). Si la clave dada no existe en la memoria caché, se devolverá un valor predeterminado opcional:
> > The `cache` function may be used to get values from the [cache](/docs/{{version}}/cache). If the given key does not exist in the cache, an optional default value will be returned:

    $value = cache('key');

    $value = cache('key', 'default');

Puede agregar elementos a la memoria caché pasando un array de pares clave / valor a la función. También debe pasar la cantidad de minutos o la duración del valor en caché que se debe considerar válido:
> > You may add items to the cache by passing an array of key / value pairs to the function. You should also pass the number of minutes or duration the cached value should be considered valid:

    cache(['key' => 'value'], 5);

    cache(['key' => 'value'], now()->addSeconds(10));

<a name="method-class-uses-recursive"></a>
#### `class_uses_recursive()` {#collection-method}

La función `class_uses_recursive` devuelve todos los rasgos utilizados por una clase, incluidos los rasgos utilizados por todas sus clases principales:
> > The `class_uses_recursive` function returns all traits used by a class, including traits used by all of its parent classes:

    $traits = class_uses_recursive(App\User::class);

<a name="method-collect"></a>
#### `collect()` {#collection-method}

La función `collect` crea una instancia [collection](/docs/{{version}}/collections) del valor dado:
> > The `collect` function creates a [collection](/docs/{{version}}/collections) instance from the given value:

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {#collection-method}

La función `config` obtiene el valor de una variable [configuration](/docs/{{version}}/configuration). Se puede acceder a los valores de configuración usando la sintaxis "punto", que incluye el nombre del archivo y la opción a la que desea acceder. Se puede especificar un valor predeterminado y se devuelve si la opción de configuración no existe:
> > The `config` function gets the value of a [configuration](/docs/{{version}}/configuration) variable. The configuration values may be accessed using "dot" syntax, which includes the name of the file and the option you wish to access. A default value may be specified and is returned if the configuration option does not exist:

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

Puede establecer variables de configuración en el tiempo de ejecución pasando una matriz de pares clave / valor:
> > You may set configuration variables at runtime by passing an array of key / value pairs:

    config(['app.debug' => true]);

<a name="method-cookie"></a>
#### `cookie()` {#collection-method}

La función `cookie` crea una nueva instancia [cookie](/docs/{{version}}/requests#cookies):
> > The `cookie` function creates a new [cookie](/docs/{{version}}/requests#cookies) instance:

    $cookie = cookie('name', 'value', $minutes);

<a name="method-csrf-field"></a>
#### `csrf_field()` {#collection-method}

La función `csrf_field` genera un campo de entrada HTML oculto `hidden` que contiene el valor del token CSRF. Por ejemplo, usando [sintaxis Blade](/docs/{{version}}/blade):
> > The `csrf_field` function generates an HTML `hidden` input field containing the value of the CSRF token. For example, using [Blade syntax](/docs/{{version}}/blade):

    {{ csrf_field() }}

<a name="method-csrf-token"></a>
#### `csrf_token()` {#collection-method}

La función `csrf_token` recupera el valor del token CSRF actual:
> > The `csrf_token` function retrieves the value of the current CSRF token:

    $token = csrf_token();

<a name="method-dd"></a>
#### `dd()` {#collection-method}

La función `dd` vuelca las variables dadas y finaliza la ejecución del script:
> > The `dd` function dumps the given variables and ends execution of the script:

    dd($value);

    dd($value1, $value2, $value3, ...);

Si no desea detener la ejecución de su script, use la función [`dump`](#method-dump) en su lugar.
> > If you do not want to halt the execution of your script, use the [`dump`](#method-dump) function instead.

<a name="method-decrypt"></a>
#### `decrypt()` {#collection-method}

La función `decrypt` descifra el valor dado usando Laravel's [encrypter](/docs/{{version}}/encryption):
> > The `decrypt` function decrypts the given value using Laravel's [encrypter](/docs/{{version}}/encryption):

    $decrypted = decrypt($encrypted_value);

<a name="method-dispatch"></a>
#### `dispatch()` {#collection-method}

La función `dispatch` empuja el [trabajo](/docs/{{version}}/queues#creating-jobs) dado en la [cola de trabajos](/docs/{{version}}/queues) de Laravel:
> > The `dispatch` function pushes the given [job](/docs/{{version}}/queues#creating-jobs) onto the Laravel [job queue](/docs/{{version}}/queues):

    dispatch(new App\Jobs\SendEmails);

<a name="method-dispatch-now"></a>
#### `dispatch_now()` {#collection-method}

La función `dispatch_now` ejecuta el [trabajo](/docs/{{version}}/queues#creating-jobs) determinado y devuelve el valor de su método `handle`:
> > The `dispatch_now` function runs the given [job](/docs/{{version}}/queues#creating-jobs) immediately and returns the value from its `handle` method:

    $result = dispatch_now(new App\Jobs\SendEmails);

<a name="method-dump"></a>
#### `dump()` {#collection-method}

La función `dump` vuelca las variables dadas:
> > The `dump` function dumps the given variables:

    dump($value);

    dump($value1, $value2, $value3, ...);

Si desea detener el script después de eliminar las variables, use la función [`dd`](#method-dd) en su lugar.
> > If you want to stop executing the script after dumping the variables, use the [`dd`](#method-dd) function instead.

> {tip} Puedes usar el comando `dump-server` de Artisan para interceptar todas las llamadas `dump` y mostrarlas en la ventana de tu consola en lugar de en tu navegador.
> > > {tip} You may use Artisan's `dump-server` command to intercept all `dump` calls and display them in your console window instead of your browser.

<a name="method-encrypt"></a>
#### `encrypt()` {#collection-method}

La función `encrypt` encripta el valor dado usando el [encriptador](/docs/{{version}}/encryption) de Laravel:
> > The `encrypt` function encrypts the given value using Laravel's [encrypter](/docs/{{version}}/encryption):

    $encrypted = encrypt($unencrypted_value);

<a name="method-env"></a>
#### `env()` {#collection-method}

La función `env` recupera el valor de una [environment variable](/docs/{{version}}/configuration#environment-configuration) o devuelve un valor predeterminado:
> > The `env` function retrieves the value of an [environment variable](/docs/{{version}}/configuration#environment-configuration) or returns a default value:

    $env = env('APP_ENV');

    // Returns 'production' if APP_ENV is not set...
    $env = env('APP_ENV', 'production');

> {note} Si ejecuta el comando `config:cache` durante su proceso de implementación, debe asegurarse de llamar solo a la función `env` desde sus archivos de configuración. Una vez que la configuración ha sido almacenada en caché, el archivo `.env` no se cargará y todas las llamadas a la función `env` devolverán `null`.
> > > {note} If you execute the `config:cache` command during your deployment process, you should be sure that you are only calling the `env` function from within your configuration files. Once the configuration has been cached, the `.env` file will not be loaded and all calls to the `env` function will return `null`.

<a name="method-event"></a>
#### `event()` {#collection-method}

La función `event` distribuye el [evento](/docs/{{version}}/events) dado a sus oyentes:
> > The `event` function dispatches the given [event](/docs/{{version}}/events) to its listeners:

    event(new UserRegistered($user));

<a name="method-factory"></a>
#### `factory()` {#collection-method}

La función `factory` crea un constructor modelo de fábrica para una clase, nombre y cantidad determinada. Se puede usar durante las [pruebas](/docs/{{version}}/database-testing#writing-factories) o [seeding](/docs/{{version}}/seeding#using-model-factories):
> > The `factory` function creates a model factory builder for a given class, name, and amount. It can be used while [testing](/docs/{{version}}/database-testing#writing-factories) or [seeding](/docs/{{version}}/seeding#using-model-factories):

    $user = factory(App\User::class)->make();

<a name="method-filled"></a>
#### `filled()` {#collection-method}

La función `filled` devuelve si el valor dado no está "en blanco":
> > The `filled` function returns whether the given value is not "blank":

    filled(0);
    filled(true);
    filled(false);

    // true

    filled('');
    filled('   ');
    filled(null);
    filled(collect());

    // false

Para el inverso de `filled`, vea el método [`blank`](#method-blank).
> > For the inverse of `filled`, see the [`blank`](#method-blank) method.

<a name="method-info"></a>
#### `info()` {#collection-method}

La función `info` escribirá información en el registro [log](/docs/{{version}}/errors#logging):
> > The `info` function will write information to the [log](/docs/{{version}}/errors#logging):

    info('Some helpful information!');

También se puede pasar un array de datos contextuales a la función:
> > An array of contextual data may also be passed to the function:

    info('User login attempt failed.', ['id' => $user->id]);

<a name="method-logger"></a>
#### `logger()` {#collection-method}

La función `logger` se puede utilizar para escribir un mensaje de nivel `debug` al registro [log](/docs/{{version}}/errors#logging):
> > The `logger` function can be used to write a `debug` level message to the [log](/docs/{{version}}/errors#logging):

    logger('Debug message');

También se puede pasar una matriz de datos contextuales a la función:
> > An array of contextual data may also be passed to the function:

    logger('User has logged in.', ['id' => $user->id]);

Se devolverá una instancia [logger](/docs/{{version}}/errors#logging) si no se pasa ningún valor a la función:
> > A [logger](/docs/{{version}}/errors#logging) instance will be returned if no value is passed to the function:

    logger()->error('You are not allowed here.');

<a name="method-method-field"></a>
#### `method_field()` {#collection-method}

La función `method_field` genera un campo de entrada HTML oculto `hidden` que contiene el valor falso del verbo HTTP del formulario. Por ejemplo, usando [sintaxis Blade](/docs/{{version}}/blade):
> > The `method_field` function generates an HTML `hidden` input field containing the spoofed value of the form's HTTP verb. For example, using [Blade syntax](/docs/{{version}}/blade):

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>

<a name="method-now"></a>
#### `now()` {#collection-method}

La función `now` crea una nueva instancia `Illuminate\Support\Carbon` para la hora actual:
> > The `now` function creates a new `Illuminate\Support\Carbon` instance for the current time:

    $now = now();

<a name="method-old"></a>
#### `old()` {#collection-method}

La función `old` [recupera](/docs/{{version}}/requests#retrieving-input) un valor [old input](/docs/{{version}}/requests#old-input) destellado en la sesión :
> > The `old` function [retrieves](/docs/{{version}}/requests#retrieving-input) an [old input](/docs/{{version}}/requests#old-input) value flashed into the session:

    $value = old('value');

    $value = old('value', 'default');

<a name="method-optional"></a>
#### `optional()` {#collection-method}

La función `opcional` acepta cualquier argumento y le permite acceder a propiedades o métodos de llamada en ese objeto. Si el objeto dado es `nulo`, las propiedades y métodos devolverán `null` en lugar de causar un error:
> > The `optional` function accepts any argument and allows you to access properties or call methods on that object. If the given object is `null`, properties and methods will return `null` instead of causing an error:

    return optional($user->address)->street;

    {!! old('name', optional($user)->name) !!}

La función `opcional` también acepta un Closure como segundo argumento. El Closure se invocará si el valor proporcionado como primer argumento no es nulo:
> > The `optional` function also accepts a Closure as its second argument. The Closure will be invoked if the value provided as the first argument is not null:

    return optional(User::find($id), function ($user) {
        return new DummyUser;
    });

<a name="method-policy"></a>
#### `policy()` {#collection-method}

El método `policy` recupera una instancia [policy](/docs/{{version}}/authorization#creating-policies) para una clase determinada:
> > The `policy` method retrieves a [policy](/docs/{{version}}/authorization#creating-policies) instance for a given class:

    $policy = policy(App\User::class);

<a name="method-redirect"></a>
#### `redirect()` {#collection-method}

La función `redirect` devuelve un [redirector de respuesta HTTP](/docs/{{version}}/responses#redirects), o devuelve la instancia del redirector si se llama sin argumentos:
> > The `redirect` function returns a [redirect HTTP response](/docs/{{version}}/responses#redirects), or returns the redirector instance if called with no arguments:

    return redirect($to = null, $status = 302, $headers = [], $secure = null);

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>
#### `report()` {#collection-method}

La función `report` reportará una excepción usando el método `report` de su [manejador de excepciones](/docs/{{version}}/errors#the-exception-handler):
> > The `report` function will report an exception using your [exception handler](/docs/{{version}}/errors#the-exception-handler)'s `report` method:

    report($e);

<a name="method-request"></a>
#### `request()` {#collection-method}

La función `request` devuelve la instancia actual [request](/docs/{{version}}/requests) u obtiene un elemento de entrada:
> > The `request` function returns the current [request](/docs/{{version}}/requests) instance or obtains an input item:

    $request = request();

    $value = request('key', $default);

<a name="method-rescue"></a>
#### `rescue()` {#collection-method}

La función `rescue` ejecuta el Closure dado y detecta cualquier excepción que ocurra durante su ejecución. Todas las excepciones capturadas se enviarán a su método `report` del [manejador de excepciones](/docs/{{version}}/errors#the-exception-handler); sin embargo, la solicitud continuará procesándose:
> > The `rescue` function executes the given Closure and catches any exceptions that occur during its execution. All exceptions that are caught will be sent to your [exception handler](/docs/{{version}}/errors#the-exception-handler)'s `report` method; however, the request will continue processing:

    return rescue(function () {
        return $this->method();
    });

También puede pasar un segundo argumento a la función `rescue`. Este argumento será el valor "predeterminado" que se debe devolver si se produce una excepción al ejecutar el Closure:
> > You may also pass a second argument to the `rescue` function. This argument will be the "default" value that should be returned if an exception occurs while executing the Closure:

    return rescue(function () {
        return $this->method();
    }, false);

    return rescue(function () {
        return $this->method();
    }, function () {
        return $this->failure();
    });

<a name="method-resolve"></a>
#### `resolve()` {#collection-method}

La función `resolve` resuelve una clase dada o nombre de interfaz a su instancia utilizando el [service container](/docs/{{version}}/container):
> > The `resolve` function resolves a given class or interface name to its instance using the [service container](/docs/{{version}}/container):

    $api = resolve('HelpSpot\API');

<a name="method-response"></a>
#### `response()` {#collection-method}

La función `response` crea una instancia [respuesta](/docs/{{version}}/responses) u obtiene una instancia de la fábrica de respuestas:
> > The `response` function creates a [response](/docs/{{version}}/responses) instance or obtains an instance of the response factory:

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>
#### `retry()` {#collection-method}

La función `retry` intenta ejecutar el callback dado hasta que se alcanza el umbral de intento máximo dado. Si el callback no arroja una excepción, se devolverá su valor de retorno. Si el callback arroja una excepción, se volverá a intentar automáticamente. Si se excede el conteo máximo de intentos, se lanzará la excepción:
> > The `retry` function attempts to execute the given callback until the given maximum attempt threshold is met. If the callback does not throw an exception, its return value will be returned. If the callback throws an exception, it will automatically be retried. If the maximum attempt count is exceeded, the exception will be thrown:

    return retry(5, function () {
        // Attempt 5 times while resting 100ms in between attempts...
    }, 100);

<a name="method-session"></a>
#### `session()` {#collection-method}

La función `session` se puede usar para obtener o establecer valores [session](/docs/{{version}}/session):
> > The `session` function may be used to get or set [session](/docs/{{version}}/session) values:

    $value = session('key');

Puede establecer valores pasando un array de pares clave / valor a la función:
> > You may set values by passing an array of key / value pairs to the function:

    session(['chairs' => 7, 'instruments' => 3]);

El almacén de sesiones se devolverá si no se pasa ningún valor a la función:
> > The session store will be returned if no value is passed to the function:

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-tap"></a>
#### `tap()` {#collection-method}

La función `tap` acepta dos argumentos: un `$value` arbitrario y un Closure. El `$value` se pasará al Closure y luego será devuelto por la función `tap`. El valor de retorno del Closure es irrelevante:
> > The `tap` function accepts two arguments: an arbitrary `$value` and a Closure. The `$value` will be passed to the Closure and then be returned by the `tap` function. The return value of the Closure is irrelevant:

    $user = tap(User::first(), function ($user) {
        $user->name = 'taylor';

        $user->save();
    });

Si no se pasa ningún Closure a la función `tap`, puede llamar a cualquier método en el `$value` dado. El valor de retorno del método que llame siempre será `$value`, independientemente de lo que el método realmente devuelva en su definición. Por ejemplo, el método `update` de Eloquent normalmente devuelve un número entero. Sin embargo, podemos forzar al método a devolver el modelo mismo encadenando la llamada al método `update` a través de la función `tap`:
> > If no Closure is passed to the `tap` function, you may call any method on the given `$value`. The return value of the method you call will always be `$value`, regardless of what the method actually returns in its definition. For example, the Eloquent `update` method typically returns an integer. However, we can force the method to return the model itself by chaining the `update` method call through the `tap` function:

    $user = tap($user)->update([
        'name' => $name,
        'email' => $email,
    ]);

<a name="method-today"></a>
#### `today()` {#collection-method}

La función `today` crea una nueva instancia `Illuminate\Support\Carbon` para la fecha actual:
> > The `today` function creates a new `Illuminate\Support\Carbon` instance for the current date:

    $today = today();

<a name="method-throw-if"></a>
#### `throw_if()` {#collection-method}

La función `throw_if` arroja la excepción dada si una expresión booleana dada se evalúa como `true`:
> > The `throw_if` function throws the given exception if a given boolean expression evaluates to `true`:

    throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

    throw_if(
        ! Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );

<a name="method-throw-unless"></a>
#### `throw_unless()` {#collection-method}

La función `throw_unless` arroja la excepción dada si una expresión booleana dada se evalúa como `false`:
> > The `throw_unless` function throws the given exception if a given boolean expression evaluates to `false`:

    throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

    throw_unless(
        Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page'
    );

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()` {#collection-method}

La función `trait_uses_recursive` devuelve todos los rasgos utilizados por un rasgo:
> > The `trait_uses_recursive` function returns all traits used by a trait:

    $traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);

<a name="method-transform"></a>
#### `transform()` {#collection-method}

La función `transform` ejecuta un `Closure` en un valor dado si el valor no es [blank](# method-blank) y devuelve el resultado del `Closure`:
> > The `transform` function executes a `Closure` on a given value if the value is not [blank](#method-blank) and returns the result of the `Closure`:

    $callback = function ($value) {
        return $value * 2;
    };

    $result = transform(5, $callback);

    // 10

También se puede pasar un valor predeterminado o `Closure` como tercer parámetro del método. Este valor será devuelto si el valor dado está en blanco:
> > A default value or `Closure` may also be passed as the third parameter to the method. This value will be returned if the given value is blank:

    $result = transform(null, $callback, 'The value is blank');

    // The value is blank

<a name="method-validator"></a>
#### `validator()` {#collection-method}

La función `validator` crea una nueva instancia [validator](/docs/{{version}}/validation) con los argumentos dados. Puede usarlo en lugar de la fachada `Validator` para mayor comodidad:
> > The `validator` function creates a new [validator](/docs/{{version}}/validation) instance with the given arguments. You may use it instead of the `Validator` facade for convenience:

    $validator = validator($data, $rules, $messages);

<a name="method-value"></a>
#### `value()` {#collection-method}

La función `value` devuelve el valor que se le da. Sin embargo, si pasa un `Closure` a la función, se ejecutará el `Closure` y se devolverá su resultado:
> > The `value` function returns the value it is given. However, if you pass a `Closure` to the function, the `Closure` will be executed then its result will be returned:

    $result = value(true);

    // true

    $result = value(function () {
        return false;
    });

    // false

<a name="method-view"></a>
#### `view()` {#collection-method}

La función `view` recupera una instancia de [view](/docs/{{version}}/views):
> > The `view` function retrieves a [view](/docs/{{version}}/views) instance:

    return view('auth.login');

<a name="method-with"></a>
#### `with()` {#collection-method}

La función `with` devuelve el valor que se le da. Si se pasa un `Closure` como segundo argumento a la función, se ejecutará el `Closure` y se devolverá su resultado:
> > The `with` function returns the value it is given. If a `Closure` is passed as the second argument to the function, the `Closure` will be executed and its result will be returned:

    $callback = function ($value) {
        return (is_numeric($value)) ? $value * 2 : 0;
    };

    $result = with(5, $callback);

    // 10

    $result = with(null, $callback);

    // 0

    $result = with(5, null);

    // 5
