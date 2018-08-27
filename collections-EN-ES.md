# Colecciones : Collections

- [Introduction](#introduction)
    - [Creating Collections](#creating-collections)
    - [Extending Collections](#extending-collections)
- [Available Methods](#available-methods)
- [Higher Order Messages](#higher-order-messages)

<a name="introduction"></a>
## Introducción : Introduction

La clase `Illuminate\Support\Collection` proporciona un contenedor fluido y conveniente para trabajar arrays de datos. Por ejemplo, mira el siguiente código. Usaremos el helper `collect` para crear una nueva instancia de colección del array, ejecutaremos la función `strtoupper` en cada elemento y luego eliminaremos todos los elementos vacíos:
> > The `Illuminate\Support\Collection` class provides a fluent, convenient wrapper for working with arrays of data. For example, check out the following code. We'll use the `collect` helper to create a new collection instance from the array, run the `strtoupper` function on each element, and then remove all empty elements:

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });

Como puede ver, la clase `Colección` le permite encadenar sus métodos para realizar una asignación fluida y una reducción de la matriz subyacente. En general, las colecciones son inmutables, lo que significa que cada método de "Colección" devuelve una instancia de "Colección" completamente nueva.
> > As you can see, the `Collection` class allows you to chain its methods to perform fluent mapping and reducing of the underlying array. In general, collections are immutable, meaning every `Collection` method returns an entirely new `Collection` instance.

<a name="creating-collections"></a>
### Creando colecciones : Creating Collections

Como se mencionó anteriormente, el helper `collect` devuelve una nueva instancia `Illuminate\Support\Collection` para el array dado. Entonces, crear una colección es tan simple como:
> > As mentioned above, the `collect` helper returns a new `Illuminate\Support\Collection` instance for the given array. So, creating a collection is as simple as:

    $collection = collect([1, 2, 3]);

> {tip} Los resultados de las consultas [Eloquent](/docs/{{version}}/eloquent) siempre se devuelven como instancias de `Collection`.
> > > {tip} The results of [Eloquent](/docs/{{version}}/eloquent) queries are always returned as `Collection` instances.

<a name="extending-collections"></a>
### Extendiendo colecciones : Extending Collections

Las colecciones son "macroable", lo que le permite agregar métodos adicionales a la clase `Collection` en tiempo de ejecución. Por ejemplo, el siguiente código agrega un método `toUpper` a la clase `Collection`:
> > Collections are "macroable", which allows you to add additional methods to the `Collection` class at run time. For example, the following code adds a `toUpper` method to the `Collection` class:

    use Illuminate\Support\Str;

    Collection::macro('toUpper', function () {
        return $this->map(function ($value) {
            return Str::upper($value);
        });
    });

    $collection = collect(['first', 'second']);

    $upper = $collection->toUpper();

    // ['FIRST', 'SECOND']

Normalmente, debe declarar las macros de recopilación en un [proveedor de servicios](/docs/{{version}}/providers).
> > Typically, you should declare collection macros in a [service provider](/docs/{{version}}/providers).

<a name="available-methods"></a>
## Métodos disponibles : Available Methods

Para el resto de esta documentación, discutiremos cada método disponible en la clase `Collection`. Recuerde, todos estos métodos pueden estar encadenados para manipular con fluidez la matriz subyacente. Además, casi todos los métodos devuelven una nueva instancia de `Collection`, lo que le permite conservar la copia original de la colección cuando sea necesario:
> > For the remainder of this documentation, we'll discuss each method available on the `Collection` class. Remember, all of these methods may be chained to fluently manipulate the underlying array. Furthermore, almost every method returns a new `Collection` instance, allowing you to preserve the original copy of the collection when necessary:

<style>
    #collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[collapse](#method-collapse)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsStrict](#method-containsstrict)
[count](#method-count)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffKeys](#method-diffkeys)
[dump](#method-dump)
[each](#method-each)
[eachSpread](#method-eachspread)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[macro](#method-macro)
[make](#method-make)
[map](#method-map)
[mapInto](#method-mapinto)
[mapSpread](#method-mapspread)
[mapToGroups](#method-maptogroups)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[min](#method-min)
[mode](#method-mode)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[pipe](#method-pipe)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[reverse](#method-reverse)
[search](#method-search)
[shift](#method-shift)
[shuffle](#method-shuffle)
[slice](#method-slice)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[sortKeys](#method-sortkeys)
[sortKeysDesc](#method-sortkeysdesc)
[splice](#method-splice)
[split](#method-split)
[sum](#method-sum)
[take](#method-take)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unwrap](#method-unwrap)
[values](#method-values)
[when](#method-when)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereInstanceOf](#method-whereinstanceof)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[wrap](#method-wrap)
[zip](#method-zip)

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

<a name="method-all"></a>
#### `all()` {#collection-method .first-collection-method}

El método `all` devuelve el array subyacente representado por la colección:
> > The `all` method returns the underlying array represented by the collection:

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-average"></a>
#### `average()` {#collection-method}

Alias ​​para el método [`avg`](#method-avg).
> > Alias for the [`avg`](#method-avg) method.

<a name="method-avg"></a>
#### `avg()` {#collection-method}

El método `avg` devuelve el [valor promedio](https://en.wikipedia.org/wiki/Average) de una clave determinada:
> > The `avg` method returns the [average value](https://en.wikipedia.org/wiki/Average) of a given key:

    $average = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->avg('foo');

    // 20

    $average = collect([1, 1, 2, 4])->avg();

    // 2

<a name="method-chunk"></a>
#### `chunk()` {#collection-method}

El método `chunk` divide la colección en múltiples colecciones más pequeñas de un tamaño dado:
> > The `chunk` method breaks the collection into multiple, smaller collections of a given size:

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->toArray();

    // [[1, 2, 3, 4], [5, 6, 7]]

Este método es especialmente útil en [views](/docs/{{version}}/views) cuando se trabaja con un sistema de cuadrícula como [Bootstrap](https://getbootstrap.com/css/#grid). Imagine que tiene una colección de modelos [Eloquent](/docs/{{version}}/elocuent) que desea mostrar en una grilla:
> > This method is especially useful in [views](/docs/{{version}}/views) when working with a grid system such as [Bootstrap](https://getbootstrap.com/css/#grid). Imagine you have a collection of [Eloquent](/docs/{{version}}/eloquent) models you want to display in a grid:

    @foreach ($products->chunk(3) as $chunk)
        <div class="row">
            @foreach ($chunk as $product)
                <div class="col-xs-4">{{ $product->name }}</div>
            @endforeach
        </div>
    @endforeach

<a name="method-collapse"></a>
#### `collapse()` {#collection-method}

El método `collapse` colapsa una colección de arrays en una sola colección plana:
> > The `collapse` method collapses a collection of arrays into a single, flat collection:

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-combine"></a>
#### `combine()` {#collection-method}

El método `combine` combina las claves de la colección con los valores de otra matriz o colección:
> > The `combine` method combines the keys of the collection with the values of another array or collection:

    $collection = collect(['name', 'age']);

    $combined = $collection->combine(['George', 29]);

    $combined->all();

    // ['name' => 'George', 'age' => 29]

<a name="method-concat"></a>
#### `concat()` {#collection-method}

El método `concat` agrega los `array` dados o valores de colección al final de la colección:
> > The `concat` method appends the given `array` or collection values onto the end of the collection:

    $collection = collect(['John Doe']);

    $concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

    $concatenated->all();

    // ['John Doe', 'Jane Doe', 'Johnny Doe']

<a name="method-contains"></a>
#### `contains()` {#collection-method}

El método `contains` determina si la colección contiene un elemento dado:
> > The `contains` method determines whether the collection contains a given item:

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    // true

    $collection->contains('New York');

    // false

También puede pasar un par de clave / valor al método `contains`, que determinará si el par dado existe en la colección:
> > You may also pass a key / value pair to the `contains` method, which will determine if the given pair exists in the collection:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->contains('product', 'Bookcase');

    // false

Finalmente, también puede pasar una devolución de llamada al método `contains` para realizar su propia prueba de verdad:
> > Finally, you may also pass a callback to the `contains` method to perform your own truth test:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function ($value, $key) {
        return $value > 5;
    });

    // false

El método `contains` usa comparaciones" sueltas "al verificar los valores de los elementos, lo que significa que una cadena con un valor entero se considerará igual a un entero del mismo valor. Utilice el método [`containsStrict`](#method-containsstric) para filtrar utilizando comparaciones "estrictas".
> > The `contains` method uses "loose" comparisons when checking item values, meaning a string with an integer value will be considered equal to an integer of the same value. Use the [`containsStrict`](#method-containsstrict) method to filter using "strict" comparisons.

<a name="method-containsstrict"></a>
#### `containsStrict()` {#collection-method}

Este método tiene la misma firma que el método [`contains`] (#method-contains); sin embargo, todos los valores se comparan utilizando comparaciones "estrictas".
> > This method has the same signature as the [`contains`](#method-contains) method; however, all values are compared using "strict" comparisons.

<a name="method-count"></a>
#### `count()` {#collection-method}

El método `count` devuelve la cantidad total de elementos en la colección:
> > The `count` method returns the total number of items in the collection:

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-crossjoin"></a>
#### `crossJoin()` {#collection-method}

El método `crossJoin` se une a los valores de la colección entre las matrices o colecciones dadas, devolviendo un producto cartesiano con todas las permutaciones posibles:
> > The `crossJoin` method cross joins the collection's values among the given arrays or collections, returning a Cartesian product with all possible permutations:

    $collection = collect([1, 2]);

    $matrix = $collection->crossJoin(['a', 'b']);

    $matrix->all();

    /*
        [
            [1, 'a'],
            [1, 'b'],
            [2, 'a'],
            [2, 'b'],
        ]
    */

    $collection = collect([1, 2]);

    $matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);

    $matrix->all();

    /*
        [
            [1, 'a', 'I'],
            [1, 'a', 'II'],
            [1, 'b', 'I'],
            [1, 'b', 'II'],
            [2, 'a', 'I'],
            [2, 'a', 'II'],
            [2, 'b', 'I'],
            [2, 'b', 'II'],
        ]
    */

<a name="method-dd"></a>
#### `dd()` {#collection-method}

El método `dd` vacía los elementos de la colección y finaliza la ejecución del script:
> > The `dd` method dumps the collection's items and ends execution of the script:

    $collection = collect(['John Doe', 'Jane Doe']);

    $collection->dd();

    /*
        Collection {
            #items: array:2 [
                0 => "John Doe"
                1 => "Jane Doe"
            ]
        }
    */

Si no quiere dejar de ejecutar el script, use el método [`dump`](#method-dump) en su lugar.
> > If you do not want to stop executing the script, use the [`dump`](#method-dump) method instead.

<a name="method-diff"></a>
#### `diff()` {#collection-method}

El método `diff` compara la colección con otra colección o un `array` PHP simple basado en sus valores. Este método devolverá los valores en la colección original que no están presentes en la colección dada:
> > The `diff` method compares the collection against another collection or a plain PHP `array` based on its values. This method will return the values in the original collection that are not present in the given collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

<a name="method-diffassoc"></a>
#### `diffAssoc()` {#collection-method}

El método `diffAssoc` compara la colección con otra colección o un `array` PHP simple basado en sus claves y valores. Este método devolverá los pares clave / valor en la colección original que no están presentes en la colección dada:
> > The `diffAssoc` method compares the collection against another collection or a plain PHP `array` based on its keys and values. This method will return the key / value pairs in the original collection that are not present in the given collection:

    $collection = collect([
        'color' => 'orange',
        'type' => 'fruit',
        'remain' => 6
    ]);

    $diff = $collection->diffAssoc([
        'color' => 'yellow',
        'type' => 'fruit',
        'remain' => 3,
        'used' => 6
    ]);

    $diff->all();

    // ['color' => 'orange', 'remain' => 6]

<a name="method-diffkeys"></a>
#### `diffKeys()` {#collection-method}

El método `diffKeys` compara la colección con otra colección o un `array` PHP simple basado en sus claves. Este método devolverá los pares clave / valor en la colección original que no están presentes en la colección dada:
> > The `diffKeys` method compares the collection against another collection or a plain PHP `array` based on its keys. This method will return the key / value pairs in the original collection that are not present in the given collection:

    $collection = collect([
        'one' => 10,
        'two' => 20,
        'three' => 30,
        'four' => 40,
        'five' => 50,
    ]);

    $diff = $collection->diffKeys([
        'two' => 2,
        'four' => 4,
        'six' => 6,
        'eight' => 8,
    ]);

    $diff->all();

    // ['one' => 10, 'three' => 30, 'five' => 50]

<a name="method-dump"></a>
#### `dump()` {#collection-method}

El método `dump` vuelca los elementos de la colección:
> > The `dump` method dumps the collection's items:

    $collection = collect(['John Doe', 'Jane Doe']);

    $collection->dump();

    /*
        Collection {
            #items: array:2 [
                0 => "John Doe"
                1 => "Jane Doe"
            ]
        }
    */

Si desea dejar de ejecutar el script después de volcar la colección, use el método [`dd`](#method-dd) en su lugar.
> > If you want to stop executing the script after dumping the collection, use the [`dd`](#method-dd) method instead.

<a name="method-each"></a>
#### `each()` {#collection-method}

El método `each` itera sobre los elementos de la colección y pasa cada elemento a una devolución de llamada:
> > The `each` method iterates over the items in the collection and passes each item to a callback:

    $collection->each(function ($item, $key) {
        //
    });

Si desea detener la iteración a través de los elementos, puede devolver `false` de su devolución de llamada:
> > If you would like to stop iterating through the items, you may return `false` from your callback:

    $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-eachspread"></a>
#### `eachSpread()` {#collection-method}

El método `eachSpread` itera sobre los elementos de la colección, pasando cada valor de elemento anidado a la devolución de llamada dada:
> > The `eachSpread` method iterates over the collection's items, passing each nested item value into the given callback:

    $collection = collect([['John Doe', 35], ['Jane Doe', 33]]);

    $collection->eachSpread(function ($name, $age) {
        //
    });

Puede detener la iteración a través de los elementos al devolver `false` de la devolución de llamada:
> > You may stop iterating through the items by returning `false` from the callback:

    $collection->eachSpread(function ($name, $age) {
        return false;
    });

<a name="method-every"></a>
#### `every()` {#collection-method}

El método `every` se puede usar para verificar que todos los elementos de una colección pasen una prueba de verdad dada:
> > The `every` method may be used to verify that all elements of a collection pass a given truth test:

    collect([1, 2, 3, 4])->every(function ($value, $key) {
        return $value > 2;
    });

    // false

<a name="method-except"></a>
#### `except()` {#collection-method}

El método `except` devuelve todos los elementos de la colección, excepto aquellos con las claves especificadas:
> > The `except` method returns all items in the collection except for those with the specified keys:

    $collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

    $filtered = $collection->except(['price', 'discount']);

    $filtered->all();

    // ['product_id' => 1]

Para el inverso de `except`, vea el método [only](#method-only).
> > For the inverse of `except`, see the [only](#method-only) method.

<a name="method-filter"></a>
#### `filter()` {#collection-method}

El método `filter` filtra la colección usando la devolución de llamada dada, manteniendo solo aquellos elementos que pasan una prueba de verdad dada:
> > The `filter` method filters the collection using the given callback, keeping only those items that pass a given truth test:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [3, 4]

Si no se proporciona una devolución de llamada, se eliminarán todas las entradas de la colección que son equivalentes a `false`:
> > If no callback is supplied, all entries of the collection that are equivalent to `false` will be removed:

    $collection = collect([1, 2, 3, null, false, '', 0, []]);

    $collection->filter()->all();

    // [1, 2, 3]

Para el inverso de `filter`, vea el método [rechazar](#method-reject).
> > For the inverse of `filter`, see the [reject](#method-reject) method.

<a name="method-first"></a>
#### `first()` {#collection-method}

El método `first` devuelve el primer elemento en la colección que pasa una prueba de verdad dada:
> > The `first` method returns the first element in the collection that passes a given truth test:

    collect([1, 2, 3, 4])->first(function ($value, $key) {
        return $value > 2;
    });

    // 3

También puede llamar al método `first` sin argumentos para obtener el primer elemento de la colección. Si la colección está vacía, se devuelve `null`:
> > You may also call the `first` method with no arguments to get the first element in the collection. If the collection is empty, `null` is returned:

    collect([1, 2, 3, 4])->first();

    // 1

<a name="method-first-where"></a>
#### `firstWhere()` {#collection-method}

The `firstWhere` method returns the first element in the collection with the given key / value pair:
> > El método `firstWhere` devuelve el primer elemento de la colección con el par clave / valor proporcionado:

    $collection = collect([
        ['name' => 'Regena', 'age' => 12],
        ['name' => 'Linda', 'age' => 14],
        ['name' => 'Diego', 'age' => 23],
        ['name' => 'Linda', 'age' => 84],
    ]);

    $collection->firstWhere('name', 'Linda');

    // ['name' => 'Linda', 'age' => 14]

También puede llamar al método `firstWhere` con un operador:
> > You may also call the `firstWhere` method with an operator:

    $collection->firstWhere('age', '>=', 18);

    // ['name' => 'Diego', 'age' => 23]

<a name="method-flatmap"></a>
#### `flatMap()` {#collection-method}

El método `flatMap` itera a través de la colección y pasa cada valor a la devolución de llamada dada. La devolución de llamada es libre de modificar el elemento y devolverlo, formando así una nueva colección de elementos modificados. Entonces, el array se aplana a un nivel:
> > The `flatMap` method iterates through the collection and passes each value to the given callback. The callback is free to modify the item and return it, thus forming a new collection of modified items. Then, the array is flattened by a level:

    $collection = collect([
        ['name' => 'Sally'],
        ['school' => 'Arkansas'],
        ['age' => 28]
    ]);

    $flattened = $collection->flatMap(function ($values) {
        return array_map('strtoupper', $values);
    });

    $flattened->all();

    // ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];

<a name="method-flatten"></a>
#### `flatten()` {#collection-method}

El método `flatten` aplana una colección multidimensional en una sola dimensión:
> > The `flatten` method flattens a multi-dimensional collection into a single dimension:

    $collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];

Opcionalmente, puede pasarle a la función un argumento de "profundidad":
> > You may optionally pass the function a "depth" argument:

    $collection = collect([
        'Apple' => [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ],
        'Samsung' => [
            ['name' => 'Galaxy S7', 'brand' => 'Samsung']
        ],
    ]);

    $products = $collection->flatten(1);

    $products->values()->all();

    /*
        [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
            ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
        ]
    */

En este ejemplo, al llamar a `flatten` sin proporcionar la profundidad también se aplanarían las matrices anidadas, lo que da como resultado `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`. Proporcionar una profundidad le permite restringir los niveles de matrices anidadas que se aplanarán.
> > In this example, calling `flatten` without providing the depth would have also flattened the nested arrays, resulting in `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`. Providing a depth allows you to restrict the levels of nested arrays that will be flattened.

<a name="method-flip"></a>
#### `flip()` {#collection-method}

El método `flip` intercambia las claves de la colección con sus valores correspondientes:
The `flip` method swaps the collection's keys with their corresponding values:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` {#collection-method}

El método `forget` elimina un elemento de la colección por su clave:
> > The `forget` method removes an item from the collection by its key:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $collection->forget('name');

    $collection->all();

    // ['framework' => 'laravel']

> {note} A diferencia de la mayoría de los otros métodos de recolección, `forget` no devuelve una nueva colección modificada; modifica la colección a la que se llama.
> > > {note} Unlike most other collection methods, `forget` does not return a new modified collection; it modifies the collection it is called on.

<a name="method-forpage"></a>
#### `forPage()` {#collection-method}

El método `forPage` devuelve una nueva colección que contiene los elementos que estarían presentes en un número de página determinado. El método acepta el número de página como primer argumento y la cantidad de elementos para mostrar por página como segundo argumento:
> > The `forPage` method returns a new collection containing the items that would be present on a given page number. The method accepts the page number as its first argument and the number of items to show per page as its second argument:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunk = $collection->forPage(2, 3);

    $chunk->all();

    // [4, 5, 6]

<a name="method-get"></a>
#### `get()` {#collection-method}

El método `get` devuelve el elemento en una clave determinada. Si la clave no existe, se devuelve `null`:
> > The `get` method returns the item at a given key. If the key does not exist, `null` is returned:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('name');

    // taylor

Opcionalmente, puede pasar un valor predeterminado como segundo argumento:
> > You may optionally pass a default value as the second argument:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('foo', 'default-value');

    // default-value

Incluso puede pasar una rellamada como el valor predeterminado. El resultado de la rellamada se devolverá si la clave especificada no existe:
> > You may even pass a callback as the default value. The result of the callback will be returned if the specified key does not exist:

    $collection->get('email', function () {
        return 'default-value';
    });

    // default-value

<a name="method-groupby"></a>
#### `groupBy()` {#collection-method}

El método `groupBy` agrupa los elementos de la colección con una clave determinada:
> > The `groupBy` method groups the collection's items by a given key:

    $collection = collect([
        ['account_id' => 'account-x10', 'product' => 'Chair'],
        ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ['account_id' => 'account-x11', 'product' => 'Desk'],
    ]);

    $grouped = $collection->groupBy('account_id');

    $grouped->toArray();

    /*
        [
            'account-x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'account-x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

En lugar de pasar una cadena `key`, puede pasar una devolución de llamada. La devolución de llamada debe devolver el valor que desea que el grupo clave:
> > Instead of passing a string `key`, you may pass a callback. The callback should return the value you wish to key the group by:

    $grouped = $collection->groupBy(function ($item, $key) {
        return substr($item['account_id'], -3);
    });

    $grouped->toArray();

    /*
        [
            'x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

Se pueden pasar múltiples criterios de agrupamiento como un array. Cada elemento del array se aplicará al nivel correspondiente dentro de un array multidimensional:
> > Multiple grouping criteria may be passed as an array. Each array element will be applied to the corresponding level within a multi-dimensional array:

    $data = new Collection([
        10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
        20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
        40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
    ]);

    $result = $data->groupBy([
        'skill',
        function ($item) {
            return $item['roles'];
        },
    ], $preserveKeys = true);

    /*
    [
        1 => [
            'Role_1' => [
                10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
                20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
            ],
            'Role_2' => [
                20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
            ],
            'Role_3' => [
                10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
            ],
        ],
        2 => [
            'Role_1' => [
                30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
            ],
            'Role_2' => [
                40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
            ],
        ],
    ];
    */

<a name="method-has"></a>
#### `has()` {#collection-method}

El método `has` determina si existe una clave dada en la colección:
> > The `has` method determines if a given key exists in the collection:

    $collection = collect(['account_id' => 1, 'product' => 'Desk']);

    $collection->has('product');

    // true

<a name="method-implode"></a>
#### `implode()` {#collection-method}

El método `implode` se une a los elementos de una colección. Sus argumentos dependen del tipo de elementos en la colección. Si la colección contiene arrays u objetos, debe pasar la clave de los atributos que desea unir y la cadena "pegamento" que desea colocar entre los valores:
> > The `implode` method joins the items in a collection. Its arguments depend on the type of items in the collection. If the collection contains arrays or objects, you should pass the key of the attributes you wish to join, and the "glue" string you wish to place between the values:

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair

Si la colección contiene cadenas simples o valores numéricos, pase el "pegamento" como el único argumento para el método:
> > If the collection contains simple strings or numeric values, pass the "glue" as the only argument to the method:

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()` {#collection-method}

El método `intersect` elimina cualquier valor de la colección original que no esté presente en la `array` o colección dada. La colección resultante conservará las claves de la colección original:
> > The `intersect` method removes any values from the original collection that are not present in the given `array` or collection. The resulting collection will preserve the original collection's keys:

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

<a name="method-intersectbykeys"></a>
#### `intersectByKeys()` {#collection-method}

El método `intersectByKeys` elimina cualquier clave de la colección original que no esté presente en el `array` o colección dada:
> > The `intersectByKeys` method removes any keys from the original collection that are not present in the given `array` or collection:

    $collection = collect([
        'serial' => 'UX301', 'type' => 'screen', 'year' => 2009
    ]);

    $intersect = $collection->intersectByKeys([
        'reference' => 'UX404', 'type' => 'tab', 'year' => 2011
    ]);

    $intersect->all();

    // ['type' => 'screen', 'year' => 2009]

<a name="method-isempty"></a>
#### `isEmpty()` {#collection-method}

El método `isEmpty` devuelve `true` si la colección está vacía; de lo contrario, se devuelve `false`:
> > The `isEmpty` method returns `true` if the collection is empty; otherwise, `false` is returned:

    collect([])->isEmpty();

    // true

<a name="method-isnotempty"></a>
#### `isNotEmpty()` {#collection-method}

El método `isNotEmpty` devuelve `true` si la colección no está vacía; de lo contrario, se devuelve `false`:
> > The `isNotEmpty` method returns `true` if the collection is not empty; otherwise, `false` is returned:

    collect([])->isNotEmpty();

    // false

<a name="method-keyby"></a>
#### `keyBy()` {#collection-method}

El método `keyBy` teclea la colección con la clave dada. Si varios elementos tienen la misma clave, solo el último aparecerá en la nueva colección:
> > The `keyBy` method keys the collection by the given key. If multiple items have the same key, only the last one will appear in the new collection:

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keyed = $collection->keyBy('product_id');

    $keyed->all();

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

También puede pasar una devolución de llamada al método. La devolución de llamada debe devolver el valor de la clave de la colección al:
> > You may also pass a callback to the method. The callback should return the value to key the collection by:

    $keyed = $collection->keyBy(function ($item) {
        return strtoupper($item['product_id']);
    });

    $keyed->all();

    /*
        [
            'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

<a name="method-keys"></a>
#### `keys()` {#collection-method}

El método `keys` devuelve todas las claves de la colección:
> > The `keys` method returns all of the collection's keys:

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keys = $collection->keys();

    $keys->all();

    // ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` {#collection-method}

El método `last` devuelve el último elemento en la colección que pasa una prueba de verdad dada:
> > The `last` method returns the last element in the collection that passes a given truth test:

    collect([1, 2, 3, 4])->last(function ($value, $key) {
        return $value < 3;
    });

    // 2

También puede llamar al método `last` sin argumentos para obtener el último elemento de la colección. Si la colección está vacía, se devuelve `null`:
> > You may also call the `last` method with no arguments to get the last element in the collection. If the collection is empty, `null` is returned:

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-macro"></a>
#### `macro()` {#collection-method}

El método estático `macro` le permite agregar métodos a la clase` Collection` en tiempo de ejecución. Consulte la documentación sobre [extender colecciones](#extending-collections) para obtener más información.
> > The static `macro` method allows you to add methods to the `Collection` class at run time. Refer to the documentation on [extending collections](#extending-collections) for more information.

<a name="method-make"></a>
#### `make()` {#collection-method}

El método estático `make` crea una nueva instancia de colección. Consulte la sección [Creación de colecciones](#creating-collections).
> > The static `make` method creates a new collection instance. See the [Creating Collections](#creating-collections) section.

<a name="method-map"></a>
#### `map()` {#collection-method}

El método `map` itera a través de la colección y pasa cada valor a la rellamada dada. La rellamada es libre de modificar el elemento y devolverlo, formando así una nueva colección de elementos modificados:
> > The `map` method iterates through the collection and passes each value to the given callback. The callback is free to modify the item and return it, thus forming a new collection of modified items:

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });

    $multiplied->all();

    // [2, 4, 6, 8, 10]

> {note} Como la mayoría de los otros métodos de recolección, `map` devuelve una nueva instancia de colección; no modifica la colección a la que se llama. Si desea transformar la colección original, use el método [`transform`](#method-transform).
> > > {note} Like most other collection methods, `map` returns a new collection instance; it does not modify the collection it is called on. If you want to transform the original collection, use the [`transform`](#method-transform) method.

<a name="method-mapinto"></a>
#### `mapInto()` {#collection-method}

El método `mapInto()` itera sobre la colección, creando una nueva instancia de la clase dada pasando el valor al constructor:
> > The `mapInto()` method iterates over the collection, creating a new instance of the given class by passing the value into the constructor:

    class Currency
    {
        /**
         * Create a new currency instance.
         *
         * @param  string  $code
         * @return void
         */
        function __construct(string $code)
        {
            $this->code = $code;
        }
    }

    $collection = collect(['USD', 'EUR', 'GBP']);

    $currencies = $collection->mapInto(Currency::class);

    $currencies->all();

    // [Currency('USD'), Currency('EUR'), Currency('GBP')]

<a name="method-mapspread"></a>
#### `mapSpread()` {#collection-method}

El método `mapSpread` itera sobre los elementos de la colección, pasando cada valor de elemento anidado a la rellamada dada. La rellamada es libre de modificar el elemento y devolverlo, formando así una nueva colección de elementos modificados:
> > The `mapSpread` method iterates over the collection's items, passing each nested item value into the given callback. The callback is free to modify the item and return it, thus forming a new collection of modified items:

    $collection = collect([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunks = $collection->chunk(2);

    $sequence = $chunks->mapSpread(function ($odd, $even) {
        return $odd + $even;
    });

    $sequence->all();

    // [1, 5, 9, 13, 17]

<a name="method-maptogroups"></a>
#### `mapToGroups()` {#collection-method}

El método `mapToGroups` agrupa los elementos de la colección por la devolución de llamada dada. La devolución de llamada debería devolver un array asociativo que contenga un único par clave / valor, formando así una nueva colección de valores agrupados:
> > The `mapToGroups` method groups the collection's items by the given callback. The callback should return an associative array containing a single key / value pair, thus forming a new collection of grouped values:

    $collection = collect([
        [
            'name' => 'John Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Jane Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Johnny Doe',
            'department' => 'Marketing',
        ]
    ]);

    $grouped = $collection->mapToGroups(function ($item, $key) {
        return [$item['department'] => $item['name']];
    });

    $grouped->toArray();

    /*
        [
            'Sales' => ['John Doe', 'Jane Doe'],
            'Marketing' => ['Johhny Doe'],
        ]
    */

    $grouped->get('Sales')->all();

    // ['John Doe', 'Jane Doe']

<a name="method-mapwithkeys"></a>
#### `mapWithKeys()` {#collection-method}

El método `mapWithKeys` itera a través de la colección y pasa cada valor a la devolución de llamada dada. La devolución de llamada debería devolver una matriz asociativa que contenga un único par clave / valor:
> > The `mapWithKeys` method iterates through the collection and passes each value to the given callback. The callback should return an associative array containing a single key / value pair:

    $collection = collect([
        [
            'name' => 'John',
            'department' => 'Sales',
            'email' => 'john@example.com'
        ],
        [
            'name' => 'Jane',
            'department' => 'Marketing',
            'email' => 'jane@example.com'
        ]
    ]);

    $keyed = $collection->mapWithKeys(function ($item) {
        return [$item['email'] => $item['name']];
    });

    $keyed->all();

    /*
        [
            'john@example.com' => 'John',
            'jane@example.com' => 'Jane',
        ]
    */

<a name="method-max"></a>
#### `max()` {#collection-method}

El método `max` devuelve el valor máximo de una clave determinada:
> > The `max` method returns the maximum value of a given key:

    $max = collect([['foo' => 10], ['foo' => 20]])->max('foo');

    // 20

    $max = collect([1, 2, 3, 4, 5])->max();

    // 5

<a name="method-median"></a>
#### `median()` {#collection-method}

El método `median` devuelve el [valor medio] (https://en.wikipedia.org/wiki/Median) de una clave determinada:
> > The `median` method returns the [median value](https://en.wikipedia.org/wiki/Median) of a given key:

    $median = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->median('foo');

    // 15

    $median = collect([1, 1, 2, 4])->median();

    // 1.5

<a name="method-merge"></a>
#### `merge()` {#collection-method}

El método `merge` combina la matriz o colección dada con la colección original. Si una clave de cadena en los elementos dados coincide con una clave de cadena en la colección original, el valor de los elementos dados sobrescribirá el valor en la colección original:
> > The `merge` method merges the given array or collection with the original collection. If a string key in the given items matches a string key in the original collection, the given items's value will overwrite the value in the original collection:

    $collection = collect(['product_id' => 1, 'price' => 100]);

    $merged = $collection->merge(['price' => 200, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'price' => 200, 'discount' => false]

Si las claves de los elementos son numéricas, los valores se agregarán al final de la colección:
> > If the given items's keys are numeric, the values will be appended to the end of the collection:

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-min"></a>
#### `min()` {#collection-method}

The `min` method returns the minimum value of a given key:
El método `min` devuelve el valor mínimo de una clave determinada:

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

    // 10

    $min = collect([1, 2, 3, 4, 5])->min();

    // 1

<a name="method-mode"></a>
#### `mode()` {#collection-method}

El método `mode` devuelve el [valor de modo] (https://en.wikipedia.org/wiki/Mode_(statistics)) de una clave determinada:
> > The `mode` method returns the [mode value](https://en.wikipedia.org/wiki/Mode_(statistics)) of a given key:

    $mode = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->mode('foo');

    // [10]

    $mode = collect([1, 1, 2, 4])->mode();

    // [1]

<a name="method-nth"></a>
#### `nth()` {#collection-method}

El método `nth` crea una nueva colección que consta de cada n-ésimo elemento:
> > The `nth` method creates a new collection consisting of every n-th element:

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->nth(4);

    // ['a', 'e']

Opcionalmente puede pasar un desplazamiento como segundo argumento:
> > You may optionally pass an offset as the second argument:

    $collection->nth(4, 1);

    // ['b', 'f']

<a name="method-only"></a>
#### `only()` {#collection-method}

El método `only` devuelve los elementos en la colección con las claves especificadas:
> > The `only` method returns the items in the collection with the specified keys:

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->only(['product_id', 'name']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

Para el inverso de `only`, vea el método [except](#method-except).
> > For the inverse of `only`, see the [except](#method-except) method.

<a name="method-pad"></a>
#### `pad()` {#collection-method}

El método `pad` llenará el array con el valor dado hasta que el array alcance el tamaño especificado. Este método se comporta como la función de PHP [array_pad] (https://secure.php.net/manual/en/function.array-pad.php).
> > The `pad` method will fill the array with the given value until the array reaches the specified size. This method behaves like the [array_pad](https://secure.php.net/manual/en/function.array-pad.php) PHP function.

Para rellenar a la izquierda, debe especificar un tamaño negativo. No se realizará ningún relleno si el valor absoluto del tamaño dado es menor o igual que la longitud del conjunto:
> > To pad to the left, you should specify a negative size. No padding will take place if the absolute value of the given size is less than or equal to the length of the array:

    $collection = collect(['A', 'B', 'C']);

    $filtered = $collection->pad(5, 0);

    $filtered->all();

    // ['A', 'B', 'C', 0, 0]

    $filtered = $collection->pad(-5, 0);

    $filtered->all();

    // [0, 0, 'A', 'B', 'C']

<a name="method-partition"></a>
#### `partition()` {#collection-method}

El método `partition` se puede combinar con la función PHP `list` para separar los elementos que pasan una prueba de verdad determinada de aquellos que no:
> > The `partition` method may be combined with the `list` PHP function to separate elements that pass a given truth test from those that do not:

    $collection = collect([1, 2, 3, 4, 5, 6]);

    list($underThree, $aboveThree) = $collection->partition(function ($i) {
        return $i < 3;
    });

    $underThree->all();

    // [1, 2]

    $aboveThree->all();

    // [3, 4, 5, 6]

<a name="method-pipe"></a>
#### `pipe()` {#collection-method}

El método `pipe` pasa la colección a la rellamada dada y devuelve el resultado:
> > The `pipe` method passes the collection to the given callback and returns the result:

    $collection = collect([1, 2, 3]);

    $piped = $collection->pipe(function ($collection) {
        return $collection->sum();
    });

    // 6

<a name="method-pluck"></a>
#### `pluck()` {#collection-method}

El método `pluck` recupera todos los valores para una clave dada:
> > The `pluck` method retrieves all of the values for a given key:

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $plucked = $collection->pluck('name');

    $plucked->all();

    // ['Desk', 'Chair']

También puede especificar cómo desea que se codifique la colección resultante:
> > You may also specify how you wish the resulting collection to be keyed:

    $plucked = $collection->pluck('name', 'product_id');

    $plucked->all();

    // ['prod-100' => 'Desk', 'prod-200' => 'Chair']

Si existen claves duplicadas, el último elemento coincidente se insertará en la colección punteada:
> > If duplicate keys exist, the last matching element will be inserted into the plucked collection:

    $collection = collect([
        ['brand' => 'Tesla',  'color' => 'red'],
        ['brand' => 'Pagani', 'color' => 'white'],
        ['brand' => 'Tesla',  'color' => 'black'],
        ['brand' => 'Pagani', 'color' => 'orange'],
    ]);

    $plucked = $collection->pluck('color', 'brand');

    $plucked->all();

    // ['Tesla' => 'black', 'Pagani' => 'orange']

<a name="method-pop"></a>
#### `pop()` {#collection-method}

El método `pop` elimina y devuelve el último elemento de la colección:
> > The `pop` method removes and returns the last item from the collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop();

    // 5

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-prepend"></a>
#### `prepend()` {#collection-method}

El método `prepend` agrega un elemento al comienzo de la colección:
> > The `prepend` method adds an item to the beginning of the collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

También puede pasar un segundo argumento para establecer la clave del elemento antepuesto:
> > You may also pass a second argument to set the key of the prepended item:

    $collection = collect(['one' => 1, 'two' => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two' => 2]

<a name="method-pull"></a>
#### `pull()` {#collection-method}

El método `pull` elimina y devuelve un elemento de la colección por su clave:
> > The `pull` method removes and returns an item from the collection by its key:

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` {#collection-method}

El método `push` agrega un elemento al final de la colección:
> > The `push` method appends an item to the end of the collection:

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` {#collection-method}

El método `put` establece la clave y el valor dados en la colección:
> > The `put` method sets the given key and value in the collection:

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {#collection-method}

El método `random` devuelve un elemento aleatorio de la colección:
> > The `random` method returns a random item from the collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (retrieved randomly)

Opcionalmente, puede pasar un número entero a `random` para especificar cuántos elementos desea recuperar al azar. Siempre se devuelve una colección de elementos cuando se pasa explícitamente la cantidad de elementos que desea recibir:
> > You may optionally pass an integer to `random` to specify how many items you would like to randomly retrieve. A collection of items is always returned when explicitly passing the number of items you wish to receive:

    $random = $collection->random(3);

    $random->all();

    // [2, 4, 5] - (retrieved randomly)

Si la Colección tiene menos elementos de los solicitados, el método lanzará una `InvalidArgumentException`.
> > If the Collection has fewer items than requested, the method will throw an `InvalidArgumentException`.

<a name="method-reduce"></a>
#### `reduce()` {#collection-method}

El método `reduce` reduce la colección a un único valor, pasando el resultado de cada iteración a la iteración siguiente:
> > The `reduce` method reduces the collection to a single value, passing the result of each iteration into the subsequent iteration:

    $collection = collect([1, 2, 3]);

    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });

    // 6

El valor de `$carry` en la primera iteración es `null`; sin embargo, puede especificar su valor inicial pasando un segundo argumento a `reduce`:
> > The value for `$carry` on the first iteration is `null`; however, you may specify its initial value by passing a second argument to `reduce`:

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);

    // 10

<a name="method-reject"></a>
#### `reject()` {#collection-method}

El método `reject` filtra la colección usando la rellamada dada. La rellamada debería devolver `true` si el elemento debe eliminarse de la colección resultante:
> > The `reject` method filters the collection using the given callback. The callback should return `true` if the item should be removed from the resulting collection:

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [1, 2]

Para el inverso del método `reject`, vea el método [`filter`](# method-filter).
> > For the inverse of the `reject` method, see the [`filter`](#method-filter) method.

<a name="method-reverse"></a>
#### `reverse()` {#collection-method}

El método `reverse` invierte el orden de los elementos de la colección, conservando las claves originales:
> > The `reverse` method reverses the order of the collection's items, preserving the original keys:

    $collection = collect(['a', 'b', 'c', 'd', 'e']);

    $reversed = $collection->reverse();

    $reversed->all();

    /*
        [
            4 => 'e',
            3 => 'd',
            2 => 'c',
            1 => 'b',
            0 => 'a',
        ]
    */

<a name="method-search"></a>
#### `search()` {#collection-method}

El método `search` busca en la colección el valor dado y devuelve su clave si se encuentra. Si el artículo no se encuentra, se devuelve `false`.
> > The `search` method searches the collection for the given value and returns its key if found. If the item is not found, `false` is returned.

    $collection = collect([2, 4, 6, 8]);

    $collection->search(4);

    // 1

La búsqueda se realiza usando una comparación "suelta", lo que significa que una cadena con un valor entero se considerará igual a un número entero del mismo valor. Para usar una comparación "estricta", pase `true` como segundo argumento del método:
> > The search is done using a "loose" comparison, meaning a string with an integer value will be considered equal to an integer of the same value. To use "strict" comparison, pass `true` as the second argument to the method:

    $collection->search('4', true);

    // false

Alternativamente, puede pasar su propia devolución de llamada para buscar el primer elemento que aprueba su prueba de verdad:
> > Alternatively, you may pass in your own callback to search for the first item that passes your truth test:

    $collection->search(function ($item, $key) {
        return $item > 5;
    });

    // 2

<a name="method-shift"></a>
#### `shift()` {#collection-method}

El método `shift` elimina y devuelve el primer elemento de la colección:
> > The `shift` method removes and returns the first item from the collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->shift();

    // 1

    $collection->all();

    // [2, 3, 4, 5]

<a name="method-shuffle"></a>
#### `shuffle()` {#collection-method}

El método `shuffle` mezcla aleatoriamente los elementos en la colección:
> > The `shuffle` method randomly shuffles the items in the collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    // [3, 2, 5, 1, 4] - (generated randomly)

<a name="method-slice"></a>
#### `slice()` {#collection-method}

El método `slice` devuelve un segmento de la colección que comienza en el índice dado:
> > The `slice` method returns a slice of the collection starting at the given index:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

Si desea limitar el tamaño del segmento devuelto, pase el tamaño deseado como el segundo argumento del método:
> > If you would like to limit the size of the returned slice, pass the desired size as the second argument to the method:

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

El segmento devuelto conservará las claves de forma predeterminada. Si no desea conservar las claves originales, puede usar el método [`values`](#method-values) para reindexarlas.
> > The returned slice will preserve keys by default. If you do not wish to preserve the original keys, you can use the [`values`](#method-values) method to reindex them.

<a name="method-sort"></a>
#### `sort()` {#collection-method}

El método `sort` ordena la colección. La colección ordenada conserva las claves de matriz originales, por lo que en este ejemplo utilizaremos el método [`values`](#method-values) para restablecer las claves a los índices numerados consecutivamente:
> > The `sort` method sorts the collection. The sorted collection keeps the original array keys, so in this example we'll use the [`values`](#method-values) method to reset the keys to consecutively numbered indexes:

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

Si sus necesidades de clasificación son más avanzadas, puede pasar una devolución de llamada a `sort` con su propio algoritmo. Consulte la documentación de PHP en [`uasort`](https://secure.php.net/manual/en/function.uasort.php#refsect1-function.uasort-parameters), que es el método `sort` de la colección llama internamente.
> > If your sorting needs are more advanced, you may pass a callback to `sort` with your own algorithm. Refer to the PHP documentation on [`uasort`](https://secure.php.net/manual/en/function.uasort.php#refsect1-function.uasort-parameters), which is what the collection's `sort` method calls under the hood.

> {tip} Si necesita ordenar una colección de matrices u objetos anidados, consulte los métodos [`sortBy`](#method-sortby) y [`sortByDesc`] (#method-sortbydesc).
> > > {tip} If you need to sort a collection of nested arrays or objects, see the [`sortBy`](#method-sortby) and [`sortByDesc`](#method-sortbydesc) methods.

<a name="method-sortby"></a>
#### `sortBy()` {#collection-method}

El método `sortBy` ordena la colección con la clave dada. La colección ordenada conserva las claves originales del array, por lo que en este ejemplo utilizaremos el método [`values`](#method-values) para restablecer las claves a los índices numerados consecutivamente:
> > The `sortBy` method sorts the collection by the given key. The sorted collection keeps the original array keys, so in this example we'll use the [`values`](#method-values) method to reset the keys to consecutively numbered indexes:

    $collection = collect([
        ['name' => 'Desk', 'price' => 200],
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
    ]);

    $sorted = $collection->sortBy('price');

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'price' => 100],
            ['name' => 'Bookcase', 'price' => 150],
            ['name' => 'Desk', 'price' => 200],
        ]
    */

También puede pasar su rellamada para determinar cómo ordenar los valores de la colección:
> > You can also pass your own callback to determine how to sort the collection values:

    $collection = collect([
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $sorted = $collection->sortBy(function ($product, $key) {
        return count($product['colors']);
    });

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'colors' => ['Black']],
            ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
            ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
        ]
    */

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {#collection-method}

Este método tiene la misma firma que el método [`sortBy`](#method-sortby), pero clasificará la colección en el orden opuesto.
> > This method has the same signature as the [`sortBy`](#method-sortby) method, but will sort the collection in the opposite order.

<a name="method-sortkeys"></a>
#### `sortKeys()` {#collection-method}

El método `sortKeys` ordena la colección mediante las teclas de la matriz asociativa subyacente:
> > The `sortKeys` method sorts the collection by the keys of the underlying associative array:

    $collection = collect([
        'id' => 22345,
        'first' => 'John',
        'last' => 'Doe',
    ]);

    $sorted = $collection->sortKeys();

    $sorted->all();

    /*
        [
            'first' => 'John',
            'id' => 22345,
            'last' => 'Doe',
        ]
    */

<a name="method-sortkeysdesc"></a>
#### `sortKeysDesc()` {#collection-method}

Este método tiene la misma firma que el método [`sortKeys`](#method-sortkeys), pero clasificará la colección en el orden opuesto.
> > This method has the same signature as the [`sortKeys`](#method-sortkeys) method, but will sort the collection in the opposite order.

<a name="method-splice"></a>
#### `splice()` {#collection-method}

El método `splice` elimina y devuelve un segmento de elementos comenzando en el índice especificado:
> > The `splice` method removes and returns a slice of items starting at the specified index:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

Puede pasar un segundo argumento para limitar el tamaño del fragmento resultante:
> > You may pass a second argument to limit the size of the resulting chunk:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

Además, puede pasar un tercer argumento que contenga los nuevos elementos para reemplazar los elementos eliminados de la colección:
> > In addition, you can pass a third argument containing the new items to replace the items removed from the collection:

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]

<a name="method-split"></a>
#### `split()` {#collection-method}

El método `split` divide una colección en el número de grupos dado:
> > The `split` method breaks a collection into the given number of groups:

    $collection = collect([1, 2, 3, 4, 5]);

    $groups = $collection->split(3);

    $groups->toArray();

    // [[1, 2], [3, 4], [5]]

<a name="method-sum"></a>
#### `sum()` {#collection-method}

El método `sum` devuelve la suma de todos los elementos en la colección:
> > The `sum` method returns the sum of all items in the collection:

    collect([1, 2, 3, 4, 5])->sum();

    // 15

Si la colección contiene arrays u objetos anidados, debe pasar una clave para usar para determinar qué valores sumar:
> > If the collection contains nested arrays or objects, you should pass a key to use for determining which values to sum:

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    // 1272

Además, puede pasar su rellamada para determinar qué valores de la colección sumar:
> > In addition, you may pass your own callback to determine which values of the collection to sum:

    $collection = collect([
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $collection->sum(function ($product) {
        return count($product['colors']);
    });

    // 6

<a name="method-take"></a>
#### `take()` {#collection-method}

El método `take` devuelve una nueva colección con el número especificado de elementos:
> > The `take` method returns a new collection with the specified number of items:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

También puede pasar un número entero negativo para tomar la cantidad especificada de elementos del final de la colección:
> > You may also pass a negative integer to take the specified amount of items from the end of the collection:

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-tap"></a>
#### `tap()` {#collection-method}

El método `tap` pasa la colección a la rellamada dada, lo que le permite "tocar" la colección en un punto específico y hacer algo con los elementos sin afectar la colección en sí:
> > The `tap` method passes the collection to the given callback, allowing you to "tap" into the collection at a specific point and do something with the items while not affecting the collection itself:

    collect([2, 4, 3, 1, 5])
        ->sort()
        ->tap(function ($collection) {
            Log::debug('Values after sorting', $collection->values()->toArray());
        })
        ->shift();

    // 1

<a name="method-times"></a>
#### `times()` {#collection-method}

El método static `times` crea una nueva colección invocando la devolución de llamada una cantidad determinada de veces:
> > The static `times` method creates a new collection by invoking the callback a given amount of times:

    $collection = Collection::times(10, function ($number) {
        return $number * 9;
    });

    $collection->all();

    // [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]

Este método puede ser útil cuando se combina con fábricas para crear modelos [Eloquentes](/docs/{{version}}/eloquent):
> > This method can be useful when combined with factories to create [Eloquent](/docs/{{version}}/eloquent) models:

    $categories = Collection::times(3, function ($number) {
        return factory(Category::class)->create(['name' => 'Category #'.$number]);
    });

    $categories->all();

    /*
        [
            ['id' => 1, 'name' => 'Category #1'],
            ['id' => 2, 'name' => 'Category #2'],
            ['id' => 3, 'name' => 'Category #3'],
        ]
    */

<a name="method-toarray"></a>
#### `toArray()` {#collection-method}

El método `toArray` convierte la colección en un simple `array` de PHP. Si los valores de la colección son modelos [Eloquent](/docs/{{version}}/eloquent), los modelos también se convertirán en arrays:
> > The `toArray` method converts the collection into a plain PHP `array`. If the collection's values are [Eloquent](/docs/{{version}}/eloquent) models, the models will also be converted to arrays:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> {note} `toArray` también convierte todos los objetos anidados de la colección en una matriz. Si desea obtener el conjunto subyacente sin procesar, use el método [`all`](# method-all) en su lugar.
> > > {note} `toArray` also converts all of the collection's nested objects to an array. If you want to get the raw underlying array, use the [`all`](#method-all) method instead.

<a name="method-tojson"></a>
#### `toJson()` {#collection-method}

El método `toJson` convierte la colección en una cadena serializada JSON:
> > The `toJson` method converts the collection into a JSON serialized string:

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk", "price":200}'

<a name="method-transform"></a>
#### `transform()` {#collection-method}

El método `transform` itera sobre la colección y llama a la rellamada dada con cada elemento de la colección. Los elementos en la colección serán reemplazados por los valores devueltos por la rellamada:
> > The `transform` method iterates over the collection and calls the given callback with each item in the collection. The items in the collection will be replaced by the values returned by the callback:

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });

    $collection->all();

    // [2, 4, 6, 8, 10]

> {note} A diferencia de la mayoría de los demás métodos de recopilación, `transform` modifica la colección en sí. Si desea crear una nueva colección, utilice el método [`map`](#method-map).
> > > {note} Unlike most other collection methods, `transform` modifies the collection itself. If you wish to create a new collection instead, use the [`map`](#method-map) method.

<a name="method-union"></a>
#### `union()` {#collection-method}

El método `union` agrega la matriz dada a la colección. Si la matriz dada contiene claves que ya están en la colección original, se preferirán los valores de la colección original:
> > The `union` method adds the given array to the collection. If the given array contains keys that are already in the original collection, the original collection's values will be preferred:

    $collection = collect([1 => ['a'], 2 => ['b']]);

    $union = $collection->union([3 => ['c'], 1 => ['b']]);

    $union->all();

    // [1 => ['a'], 2 => ['b'], 3 => ['c']]

<a name="method-unique"></a>
#### `unique()` {#collection-method}

El método `unique` devuelve todos los elementos únicos en la colección. La colección devuelta conserva las claves de matriz originales, por lo que en este ejemplo utilizaremos el método [`values`](#method-values) para restablecer las claves a índices numerados consecutivamente:
> > The `unique` method returns all of the unique items in the collection. The returned collection keeps the original array keys, so in this example we'll use the [`values`](#method-values) method to reset the keys to consecutively numbered indexes:

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

Cuando se trata de matrices u objetos anidados, puede especificar la clave utilizada para determinar la singularidad:
> > When dealing with nested arrays or objects, you may specify the key used to determine uniqueness:

    $collection = collect([
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]);

    $unique = $collection->unique('brand');

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ]
    */

También puede pasar su devolución de llamada para determinar la singularidad del elemento:
> > You may also pass your own callback to determine item uniqueness:

    $unique = $collection->unique(function ($item) {
        return $item['brand'].$item['type'];
    });

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
            ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
        ]
    */

El método `unique` utiliza comparaciones "sueltas" cuando se comprueban los valores de los elementos, lo que significa que una cadena con un valor entero se considerará igual a un número entero del mismo valor. Utilice el método [`uniqueStrict`](#method-uniquestrict) para filtrar utilizando comparaciones "estrictas".
> > The `unique` method uses "loose" comparisons when checking item values, meaning a string with an integer value will be considered equal to an integer of the same value. Use the [`uniqueStrict`](#method-uniquestrict) method to filter using "strict" comparisons.

<a name="method-uniquestrict"></a>
#### `uniqueStrict()` {#collection-method}

Este método tiene la misma firma que el método [`unique`](#method-unique); sin embargo, todos los valores se comparan utilizando comparaciones "estrictas".
> > This method has the same signature as the [`unique`](#method-unique) method; however, all values are compared using "strict" comparisons.

<a name="method-unless"></a>
#### `unless()` {#collection-method}

El método `unless` ejecutará la devolución de llamada dada a menos que el primer argumento dado al método se evalúe como `true`:
> > The `unless` method will execute the given callback unless the first argument given to the method evaluates to `true`:

    $collection = collect([1, 2, 3]);

    $collection->unless(true, function ($collection) {
        return $collection->push(4);
    });

    $collection->unless(false, function ($collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 5]

Para el inverso de `unless`, vea el método [`when`](#method-when).
> > For the inverse of `unless`, see the [`when`](#method-when) method.

<a name="method-unwrap"></a>
#### `unwrap()` {#collection-method}

El método estático `unwrap` devuelve los elementos subyacentes de la colección del valor dado cuando corresponda:
> > The static `unwrap` method returns the collection's underlying items from the given value when applicable:

    Collection::unwrap(collect('John Doe'));

    // ['John Doe']

    Collection::unwrap(['John Doe']);

    // ['John Doe']

    Collection::unwrap('John Doe');

    // 'John Doe'

<a name="method-values"></a>
#### `values()` {#collection-method}

El método `values` devuelve una nueva colección con las claves restablecidas en enteros consecutivos:
> > The `values` method returns a new collection with the keys reset to consecutive integers:

    $collection = collect([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200]
    ]);

    $values = $collection->values();

    $values->all();

    /*
        [
            0 => ['product' => 'Desk', 'price' => 200],
            1 => ['product' => 'Desk', 'price' => 200],
        ]
    */

<a name="method-when"></a>
#### `when()` {#collection-method}

El método `when` ejecutará la devolución de llamada dada cuando el primer argumento dado al método se evalúa como `true`:
> > The `when` method will execute the given callback when the first argument given to the method evaluates to `true`:

    $collection = collect([1, 2, 3]);

    $collection->when(true, function ($collection) {
        return $collection->push(4);
    });

    $collection->when(false, function ($collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 4]

Para el inverso de `when`, vea el método [`unless`](# method-unless).
> > For the inverse of `when`, see the [`unless`](#method-unless) method.

<a name="method-where"></a>
#### `where()` {#collection-method}

El método `where` filtra la colección por un par clave / valor dado:
> > The `where` method filters the collection by a given key / value pair:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->where('price', 100);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

El método `where` usa comparaciones "sueltas" cuando se comprueban los valores de los elementos, lo que significa que una cadena con un valor entero se considerará igual a un entero del mismo valor. Utilice el método [`whereStrict`](#method-wherestrict) para filtrar utilizando comparaciones "estrictas".
> > The `where` method uses "loose" comparisons when checking item values, meaning a string with an integer value will be considered equal to an integer of the same value. Use the [`whereStrict`](#method-wherestrict) method to filter using "strict" comparisons.

<a name="method-wherestrict"></a>
#### `whereStrict()` {#collection-method}

Este método tiene la misma firma que el método [`where`](#method-where); sin embargo, todos los valores se comparan utilizando comparaciones "estrictas".
> > This method has the same signature as the [`where`](#method-where) method; however, all values are compared using "strict" comparisons.

<a name="method-wherein"></a>
#### `whereIn()` {#collection-method}

El método `whereIn` filtra la colección por una clave / valor dado contenido dentro del array dado:
> > The `whereIn` method filters the collection by a given key / value contained within the given array:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Bookcase', 'price' => 150],
            ['product' => 'Desk', 'price' => 200],
        ]
    */

El método `whereIn` usa comparaciones" sueltas "cuando se comprueban los valores de los elementos, lo que significa que una cadena con un valor entero se considerará igual a un número entero del mismo valor. Utilice el método [`whereInStrict`](#method-whereinstrict) para filtrar utilizando comparaciones "estrictas".
> > The `whereIn` method uses "loose" comparisons when checking item values, meaning a string with an integer value will be considered equal to an integer of the same value. Use the [`whereInStrict`](#method-whereinstrict) method to filter using "strict" comparisons.

<a name="method-whereinstrict"></a>
#### `whereInStrict()` {#collection-method}

Este método tiene la misma firma que el método [`whereIn`](#method-donde); sin embargo, todos los valores se comparan utilizando comparaciones "estrictas".
> > This method has the same signature as the [`whereIn`](#method-wherein) method; however, all values are compared using "strict" comparisons.

<a name="method-whereinstanceof"></a>
#### `whereInstanceOf()` {#collection-method}

El método `whereInstanceOf` filtra la colección por un tipo de clase dado:
> > The `whereInstanceOf` method filters the collection by a given class type:

    $collection = collect([
        new User,
        new User,
        new Post,
    ]);

    return $collection->whereInstanceOf(User::class);

<a name="method-wherenotin"></a>
#### `whereNotIn()` {#collection-method}

El método `whereNotIn` filtra la colección por una clave / valor dado que no está contenido dentro del array dado:
> > The `whereNotIn` method filters the collection by a given key / value not contained within the given array:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereNotIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

El método `whereNotIn` utiliza comparaciones "sueltas" al verificar los valores de los elementos, lo que significa que una cadena con un valor entero se considerará igual a un número entero del mismo valor. Utilice el método [`whereNotInStrict`](#method-wherenotinstrict) para filtrar utilizando comparaciones "estrictas".
> > The `whereNotIn` method uses "loose" comparisons when checking item values, meaning a string with an integer value will be considered equal to an integer of the same value. Use the [`whereNotInStrict`](#method-wherenotinstrict) method to filter using "strict" comparisons.

<a name="method-wherenotinstrict"></a>
#### `whereNotInStrict()` {#collection-method}

Este método tiene la misma firma que el método [`whereNotIn`](#method-wherenotin); sin embargo, todos los valores se comparan utilizando comparaciones "estrictas".
> > This method has the same signature as the [`whereNotIn`](#method-wherenotin) method; however, all values are compared using "strict" comparisons.

<a name="method-wrap"></a>
#### `wrap()` {#collection-method}

El método estático `wrap` envuelve el valor dado en una colección cuando corresponda:
> > The static `wrap` method wraps the given value in a collection when applicable:

    $collection = Collection::wrap('John Doe');

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(['John Doe']);

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(collect('John Doe'));

    $collection->all();

    // ['John Doe']

<a name="method-zip"></a>
#### `zip()` {#collection-method}

El método `zip` combina los valores de la matriz dada con los valores de la colección original en el índice correspondiente:
> > The `zip` method merges together the values of the given array with the values of the original collection at the corresponding index:

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]

<a name="higher-order-messages"></a>
## Higher Order Messages

Las colecciones también brindan soporte para "mensajes de orden superior", que son atajos para realizar acciones comunes en las colecciones. Los métodos de recopilación que proporcionan mensajes de orden superior son: [`average`](#method-average), [`avg`](#method-avg), [`contains`](#method-contains), [`each`](#method-each), [`every`](#method-every), [`filter`](#method-filter), [`first`](#method-first), [`flatMap`](#method-flatmap), [`groupBy`](#method-groupby), [`keyBy`](#method-keyby), [`map`](#method-map), [`max`](#method-max), [`min`](#method-min), [`partition`](#method-partition), [`reject`](#method-reject), [`sortBy`](#method-sortby), [`sortByDesc`](#method-sortbydesc), [`sum`](#method-sum), and [`unique`](#method-unique).
> > Collections also provide support for "higher order messages", which are short-cuts for performing common actions on collections. The collection methods that provide higher order messages are: [`average`](#method-average), [`avg`](#method-avg), [`contains`](#method-contains), [`each`](#method-each), [`every`](#method-every), [`filter`](#method-filter), [`first`](#method-first), [`flatMap`](#method-flatmap), [`groupBy`](#method-groupby), [`keyBy`](#method-keyby), [`map`](#method-map), [`max`](#method-max), [`min`](#method-min), [`partition`](#method-partition), [`reject`](#method-reject), [`sortBy`](#method-sortby), [`sortByDesc`](#method-sortbydesc), [`sum`](#method-sum), and [`unique`](#method-unique).

Se puede acceder a cada mensaje de orden superior como una propiedad dinámica en una instancia de colección. Por ejemplo, usemos el mensaje `each` orden superior para llamar a un método en cada objeto dentro de una colección:
> > Each higher order message can be accessed as a dynamic property on a collection instance. For instance, let's use the `each` higher order message to call a method on each object within a collection:

    $users = User::where('votes', '>', 500)->get();

    $users->each->markAsVip();

Del mismo modo, podemos usar el mensaje de orden superior `sum` para reunir el número total de "votos" para una colección de usuarios:
> > Likewise, we can use the `sum` higher order message to gather the total number of "votes" for a collection of users:

    $users = User::where('group', 'Development')->get();

    return $users->sum->votes;
