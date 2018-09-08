# Database: Query Builder

- [Introduction](#introduction)
- [Retrieving Results](#retrieving-results)
    - [Chunking Results](#chunking-results)
    - [Aggregates](#aggregates)
- [Selects](#selects)
- [Raw Expressions](#raw-expressions)
- [Joins](#joins)
- [Unions](#unions)
- [Where Clauses](#where-clauses)
    - [Parameter Grouping](#parameter-grouping)
    - [Where Exists Clauses](#where-exists-clauses)
    - [JSON Where Clauses](#json-where-clauses)
- [Ordering, Grouping, Limit, & Offset](#ordering-grouping-limit-and-offset)
- [Conditional Clauses](#conditional-clauses)
- [Inserts](#inserts)
- [Updates](#updates)
    - [Updating JSON Columns](#updating-json-columns)
    - [Increment & Decrement](#increment-and-decrement)
- [Deletes](#deletes)
- [Pessimistic Locking](#pessimistic-locking)

<a name="introduction"></a>
## Introducción : Introduction

El generador de consultas de base de datos de Laravel proporciona una interfaz conveniente y fluida para crear y ejecutar consultas de bases de datos. Se puede usar para realizar la mayoría de las operaciones de bases de datos en su aplicación y funciona en todos los sistemas de bases de datos compatibles.
> > Laravel's database query builder provides a convenient, fluent interface to creating and running database queries. It can be used to perform most database operations in your application and works on all supported database systems.

El generador de consultas de Laravel utiliza el enlace de parámetros de PDO para proteger su aplicación contra los ataques de inyección de SQL. No es necesario limpiar las cadenas que se pasan como enlaces.
> > The Laravel query builder uses PDO parameter binding to protect your application against SQL injection attacks. There is no need to clean strings being passed as bindings.

<a name="retrieving-results"></a>
## Recuperando resultados : Retrieving Results

#### Recuperación de todas las filas de una tabla : Retrieving All Rows From A Table

Puede usar el método `table` en la fachada `DB` para comenzar una consulta. El método `table` devuelve una instancia de generador de consultas para la tabla dada, lo que le permite encadenar más restricciones a la consulta y finalmente obtener los resultados utilizando el método `get`:
> > You may use the `table` method on the `DB` facade to begin a query. The `table` method returns a fluent query builder instance for the given table, allowing you to chain more constraints onto the query and then finally get the results using the `get` method:

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
            $users = DB::table('users')->get();

            return view('user.index', ['users' => $users]);
        }
    }

El método `get` devuelve una colección `Illuminate\Support\Collection` que contiene los resultados donde cada resultado es una instancia del objeto PHP `StdClass`. Puede acceder al valor de cada columna accediendo a la columna como una propiedad del objeto:
> > The `get` method returns an `Illuminate\Support\Collection` containing the results where each result is an instance of the PHP `StdClass` object. You may access each column's value by accessing the column as a property of the object:

    foreach ($users as $user) {
        echo $user->name;
    }

#### Recuperando una sola fila / columna de una tabla : Retrieving A Single Row / Column From A Table

Si solo necesita recuperar una sola fila de la tabla de la base de datos, puede usar el método `first`. Este método devolverá un único objeto `StdClass`:
> > If you just need to retrieve a single row from the database table, you may use the `first` method. This method will return a single `StdClass` object:

    $user = DB::table('users')->where('name', 'John')->first();

    echo $user->name;

Si ni siquiera necesita una fila completa, puede extraer un único valor de un registro utilizando el método `value`. Este método devolverá el valor de la columna directamente:
> > If you don't even need an entire row, you may extract a single value from a record using the `value` method. This method will return the value of the column directly:

    $email = DB::table('users')->where('name', 'John')->value('email');

#### Recuperando una lista de valores de columna : Retrieving A List Of Column Values

Si desea recuperar una Colección que contiene los valores de una sola columna, puede usar el método `pluck`. En este ejemplo, recuperaremos una Colección de títulos de roles:
> > If you would like to retrieve a Collection containing the values of a single column, you may use the `pluck` method. In this example, we'll retrieve a Collection of role titles:

    $titles = DB::table('roles')->pluck('title');

    foreach ($titles as $title) {
        echo $title;
    }

También puede especificar una columna de clave personalizada para la Colección devuelta:
> > You may also specify a custom key column for the returned Collection:

    $roles = DB::table('roles')->pluck('title', 'name');

    foreach ($roles as $name => $title) {
        echo $title;
    }

<a name="chunking-results"></a>
### Fragmentando resultados : Chunking Results

Si necesita trabajar con miles de registros de bases de datos, considere usar el método `chunk`. Este método recupera una pequeña porción de los resultados a la vez y alimenta cada fragmento en un `Closure` para su procesamiento. Este método es muy útil para escribir [comandos Artisan](/docs/{{version}}/artisan) que procesan miles de registros. Por ejemplo, trabajemos con toda la tabla `users` en trozos de 100 registros a la vez:
> > If you need to work with thousands of database records, consider using the `chunk` method. This method retrieves a small chunk of the results at a time and feeds each chunk into a `Closure` for processing. This method is very useful for writing [Artisan commands](/docs/{{version}}/artisan) that process thousands of records. For example, let's work with the entire `users` table in chunks of 100 records at a time:

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        foreach ($users as $user) {
            //
        }
    });

Puede evitar que otros fragmentos se procesen devolviendo `false` desde el `Closure`:
> > You may stop further chunks from being processed by returning `false` from the `Closure`:

    DB::table('users')->orderBy('id')->chunk(100, function ($users) {
        // Process the records...

        return false;
    });

<a name="aggregates"></a>
### Agregados : Aggregates

El generador de consultas también proporciona una variedad de métodos agregados como `count`, `max`, `min`, `avg` y `sum`. Puede llamar a cualquiera de estos métodos después de construir su consulta:
> > The query builder also provides a variety of aggregate methods such as `count`, `max`, `min`, `avg`, and `sum`. You may call any of these methods after constructing your query:

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

Por supuesto, puede combinar estos métodos con otras cláusulas:
> > Of course, you may combine these methods with other clauses:

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

#### Determinando si existen registros : Determining If Records Exist

En lugar de utilizar el método `count` para determinar si existe algún registro que coincida con las restricciones de su consulta, puede usar los métodos `exists` y `doesntExist`:
> > Instead of using the `count` method to determine if any records exist that match your query's constraints, you may use the `exists` and `doesntExist` methods:

    return DB::table('orders')->where('finalized', 1)->exists();

    return DB::table('orders')->where('finalized', 1)->doesntExist();

<a name="selects"></a>
## Selects

#### Especificar una cláusula Select : Specifying A Select Clause

Por supuesto, puede que no siempre quiera seleccionar todas las columnas de una tabla de base de datos. Usando el método `select`, puede especificar una cláusula `select` personalizada para la consulta:
> > Of course, you may not always want to select all columns from a database table. Using the `select` method, you can specify a custom `select` clause for the query:

    $users = DB::table('users')->select('name', 'email as user_email')->get();

El método `distinct` le permite forzar a la consulta a devolver resultados distintos:
> > The `distinct` method allows you to force the query to return distinct results:

    $users = DB::table('users')->distinct()->get();

Si ya tiene una instancia del generador de consultas y desea agregar una columna a su cláusula de selección existente, puede usar el método `addSelect`:
> > If you already have a query builder instance and you wish to add a column to its existing select clause, you may use the `addSelect` method:

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

<a name="raw-expressions"></a>
## Expresiones sin procesar : Raw Expressions

A veces puede necesitar usar una expresión sin formato en una consulta. Para crear una expresión sin formato, puede usar el método `DB::raw`:
> > Sometimes you may need to use a raw expression in a query. To create a raw expression, you may use the `DB::raw` method:

    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();

> {note} Las sentencias Raw se inyectarán en la consulta como cadenas, por lo que debe tener mucho cuidado de no crear vulnerabilidades de inyección de SQL.
> > > {note} Raw statements will be injected into the query as strings, so you should be extremely careful to not create SQL injection vulnerabilities.

<a name="raw-methods"></a>
### Métodos sin procesar : Raw Methods

En lugar de usar `DB::raw`, también puede usar los siguientes métodos para insertar una expresión sin formato en varias partes de su consulta.
> > Instead of using `DB::raw`, you may also use the following methods to insert a raw expression into various parts of your query.

#### `selectRaw`

El método `selectRaw` se puede usar en lugar de `select(DB::raw(...))`. Este método acepta un array opcional de enlaces como su segundo argumento:
> > The `selectRaw` method can be used in place of `select(DB::raw(...))`. This method accepts an optional array of bindings as its second argument:

    $orders = DB::table('orders')
                    ->selectRaw('price * ? as price_with_tax', [1.0825])
                    ->get();

#### `whereRaw / orWhereRaw`

Los métodos `whereRaw` y `orWhereRaw` se pueden usar para inyectar una cláusula `where` en su consulta. Estos métodos aceptan un array opcional de enlaces como segundo argumento:
> > The `whereRaw` and `orWhereRaw` methods can be used to inject a raw `where` clause into your query. These methods accept an optional array of bindings as their second argument:

    $orders = DB::table('orders')
                    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
                    ->get();

#### `havingRaw / orHavingRaw`

Los métodos `havingRaw` y `orHavingRaw` se pueden usar para establecer una cadena sin procesar como el valor de la cláusula `having`. Estos métodos aceptan un array opcional de enlaces como segundo argumento:
> > The `havingRaw` and `orHavingRaw` methods may be used to set a raw string as the value of the `having` clause. These methods accept an optional array of bindings as their second argument:

    $orders = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > ?', [2500])
                    ->get();

#### `orderByRaw`

El método `orderByRaw` se puede usar para establecer una cadena sin procesar como el valor de la cláusula `order by`:
> > The `orderByRaw` method may be used to set a raw string as the value of the `order by` clause:

    $orders = DB::table('orders')
                    ->orderByRaw('updated_at - created_at DESC')
                    ->get();

<a name="joins"></a>
## Uniones : Joins

#### Cláusula de unión interna : Inner Join Clause

El generador de consultas también se puede usar para escribir instrucciones de combinación. Para realizar una "combinación interna" básica, puede usar el método `join` en una instancia del generador de consultas. El primer argumento que se pasa al método `join` es el nombre de la tabla a la que debe unirse, mientras que los argumentos restantes especifican las restricciones de columna para la combinación. Por supuesto, como puede ver, puede unirse a varias tablas en una sola consulta:
> > The query builder may also be used to write join statements. To perform a basic "inner join", you may use the `join` method on a query builder instance. The first argument passed to the `join` method is the name of the table you need to join to, while the remaining arguments specify the column constraints for the join. Of course, as you can see, you can join to multiple tables in a single query:

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

#### Cláusula Left Join : Left Join Clause

Si desea realizar una "combinación izquierda" en lugar de una "unión interna", use el método `leftJoin`. El método `leftJoin` tiene la misma firma que el método `join`:
> > If you would like to perform a "left join" instead of an "inner join", use the `leftJoin` method. The `leftJoin` method has the same signature as the `join` method:

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

#### Cláusula Cross Join : Cross Join Clause

Para realizar una "combinación cruzada" use el método `crossJoin` con el nombre de la tabla a la que desea unirse. Las combinaciones cruzadas generan un producto cartesiano entre la primera tabla y la tabla unida:
> > To perform a "cross join" use the `crossJoin` method with the name of the table you wish to cross join to. Cross joins generate a cartesian product between the first table and the joined table:

    $users = DB::table('sizes')
                ->crossJoin('colours')
                ->get();

#### Cláusulas avanzadas Join : Advanced Join Clauses

También puede especificar cláusulas de unión más avanzadas. Para comenzar, pase un `Closure` como segundo argumento en el método `join`. El `Closure` recibirá un objeto `JoinClause` que le permite especificar restricciones en la cláusula `join`:
> > You may also specify more advanced join clauses. To get started, pass a `Closure` as the second argument into the `join` method. The `Closure` will receive a `JoinClause` object which allows you to specify constraints on the `join` clause:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();

Si desea utilizar una cláusula de estilo "where" en sus combinaciones, puede usar los métodos `where` y `orWhere` en una combinación. En lugar de comparar dos columnas, estos métodos compararán la columna con un valor:
> > If you would like to use a "where" style clause on your joins, you may use the `where` and `orWhere` methods on a join. Instead of comparing two columns, these methods will compare the column against a value:

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

#### Sub-Query Joins

Puede usar los métodos `joinSub`,` leftJoinSub`, y `rightJoinSub` para unir una consulta a una subconsulta. Cada uno de estos métodos recibe tres argumentos: la subconsulta, su alias de tabla y un Closure que define las columnas relacionadas:
> > You may use the `joinSub`, `leftJoinSub`, and `rightJoinSub` methods to join a query to a sub-query. Each of these methods receive three arguments: the sub-query, its table alias, and a Closure that defines the related columns:

    $latestPosts = DB::table('posts')
                       ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
                       ->where('is_published', true)
                       ->groupBy('user_id');

    $users = DB::table('users')
            ->joinSub($latestPosts, 'latest_posts', function($join) {
                $join->on('users.id', '=', 'latest_posts.user_id');
            })->get();

<a name="unions"></a>
## Unions

El generador de consultas también proporciona una forma rápida de "unir" dos consultas juntas. Por ejemplo, puede crear una consulta inicial y usar el método `union` para unirla con una segunda consulta:
> > The query builder also provides a quick way to "union" two queries together. For example, you may create an initial query and use the `union` method to union it with a second query:

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

> {tip} El método `unionAll` también está disponible y tiene la misma firma de método que `union`.
> > > {tip} The `unionAll` method is also available and has the same method signature as `union`.

<a name="where-clauses"></a>
## Cláusulas Where : Where Clauses

#### Cláusulas Where simples : Simple Where Clauses

Puede usar el método `where` en una instancia del generador de consultas para agregar cláusulas `where` a la consulta. La llamada más básica a `where` requiere tres argumentos. El primer argumento es el nombre de la columna. El segundo argumento es un operador, que puede ser cualquiera de los operadores compatibles de la base de datos. Finalmente, el tercer argumento es el valor para evaluar contra la columna.
> > You may use the `where` method on a query builder instance to add `where` clauses to the query. The most basic call to `where` requires three arguments. The first argument is the name of the column. The second argument is an operator, which can be any of the database's supported operators. Finally, the third argument is the value to evaluate against the column.

Por ejemplo, aquí hay una consulta que verifica que el valor de la columna de "votos" es igual a 100:
> > For example, here is a query that verifies the value of the "votes" column is equal to 100:

    $users = DB::table('users')->where('votes', '=', 100)->get();

Por comodidad, si desea verificar que una columna es igual a un valor dado, puede pasar el valor directamente como el segundo argumento al método `where`:
> > For convenience, if you want to verify that a column is equal to a given value, you may pass the value directly as the second argument to the `where` method:

    $users = DB::table('users')->where('votes', 100)->get();

Por supuesto, puede usar otros operadores al escribir una cláusula `where`:
> > Of course, you may use a variety of other operators when writing a `where` clause:

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

También puede pasar una serie de condiciones a la función `where`:
> > You may also pass an array of conditions to the `where` function:

    $users = DB::table('users')->where([
        ['status', '=', '1'],
        ['subscribed', '<>', '1'],
    ])->get();

#### Sentencias Or : Or Statements

Puedes encadenar las restricciones juntas y agregar cláusulas `or` a la consulta. El método `orWhere` acepta los mismos argumentos que el método `where`:
> > You may chain where constraints together as well as add `or` clauses to the query. The `orWhere` method accepts the same arguments as the `where` method:

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

#### Cláusulas adicionales Where : Additional Where Clauses

**whereBetween**

El método `whereBetween` verifica que el valor de una columna esté entre dos valores:
> > The `whereBetween` method verifies that a column's value is between two values:

    $users = DB::table('users')
                        ->whereBetween('votes', [1, 100])->get();

**whereNotBetween**

El método `whereNotBetween` verifica que el valor de una columna se encuentre fuera de dos valores:
> > The `whereNotBetween` method verifies that a column's value lies outside of two values:

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();

**whereIn / whereNotIn**

El método `whereIn` verifica que el valor de una columna dada se encuentre dentro del array dado:
> > The `whereIn` method verifies that a given column's value is contained within the given array:

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

El método `whereNotIn` verifica que el valor de la columna dada **no** esté contenido en el array dado:
> > The `whereNotIn` method verifies that the given column's value is **not** contained in the given array:

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

**whereNull / whereNotNull**

El método `whereNull` verifica que el valor de la columna dada sea `NULL`:
> > The `whereNull` method verifies that the value of the given column is `NULL`:

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();

El método `whereNotNull` verifica que el valor de la columna no sea `NULL`:
> > The `whereNotNull` method verifies that the column's value is not `NULL`:

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();

**whereDate / whereMonth / whereDay / whereYear / whereTime**

El método `whereDate` se puede usar para comparar el valor de una columna con una fecha:
> > The `whereDate` method may be used to compare a column's value against a date:

    $users = DB::table('users')
                    ->whereDate('created_at', '2016-12-31')
                    ->get();

El método `whereMonth` se puede usar para comparar el valor de una columna con un mes específico de un año:
> > The `whereMonth` method may be used to compare a column's value against a specific month of a year:

    $users = DB::table('users')
                    ->whereMonth('created_at', '12')
                    ->get();

El método `whereDay` se puede usar para comparar el valor de una columna con un día específico de un mes:
> > The `whereDay` method may be used to compare a column's value against a specific day of a month:

    $users = DB::table('users')
                    ->whereDay('created_at', '31')
                    ->get();

El método `whereYear` se puede usar para comparar el valor de una columna con un año específico:
> > The `whereYear` method may be used to compare a column's value against a specific year:

    $users = DB::table('users')
                    ->whereYear('created_at', '2016')
                    ->get();

El método `whereTime` se puede usar para comparar el valor de una columna con un tiempo específico:
> > The `whereTime` method may be used to compare a column's value against a specific time:

    $users = DB::table('users')
                    ->whereTime('created_at', '=', '11:20:45')
                    ->get();

**whereColumn**

El método `whereColumn` se puede usar para verificar que dos columnas son iguales:
> > The `whereColumn` method may be used to verify that two columns are equal:

    $users = DB::table('users')
                    ->whereColumn('first_name', 'last_name')
                    ->get();

También puede pasarle un operador de comparación al método:
> > You may also pass a comparison operator to the method:

    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at')
                    ->get();

El método `whereColumn` también se puede pasar a un array de múltiples condiciones. Estas condiciones se unirán usando el operador `y`:
> > The `whereColumn` method can also be passed an array of multiple conditions. These conditions will be joined using the `and` operator:

    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', '=', 'last_name'],
                        ['updated_at', '>', 'created_at']
                    ])->get();

<a name="parameter-grouping"></a>
### Agrupación de parámetros : Parameter Grouping

A veces puede necesitar crear cláusulas where más avanzadas como cláusulas "where exists" o agrupaciones de parámetros anidados. El generador de consultas de Laravel también puede manejar estos. Para comenzar, veamos un ejemplo de restricciones de agrupación entre paréntesis:
> > Sometimes you may need to create more advanced where clauses such as "where exists" clauses or nested parameter groupings. The Laravel query builder can handle these as well. To get started, let's look at an example of grouping constraints within parenthesis:

    DB::table('users')
                ->where('name', '=', 'John')
                ->where(function ($query) {
                    $query->where('votes', '>', 100)
                          ->orWhere('title', '=', 'Admin');
                })
                ->get();

Como puede ver, al pasar un `Closure` al método `where` se le ordena al generador de consultas que comience un grupo de restricción. El `Closure` recibirá una instancia del generador de consultas que puede usar para establecer las restricciones que deberían estar contenidas dentro del grupo de paréntesis. El ejemplo anterior producirá el siguiente SQL:
> > As you can see, passing a `Closure` into the `where` method instructs the query builder to begin a constraint group. The `Closure` will receive a query builder instance which you can use to set the constraints that should be contained within the parenthesis group. The example above will produce the following SQL:

    select * from users where name = 'John' and (votes > 100 or title = 'Admin')

> {tip} Siempre debe agrupar llamadas `orWhere` para evitar comportamientos inesperados cuando se aplican ámbitos globales.
> > > {tip} You should always group `orWhere` calls in order to avoid unexpected behavior when global scopes are applied.

<a name="where-exists-clauses"></a>
### Where Exists Clauses

El método `whereExists` le permite escribir cláusulas SQL `where exists`. El método `whereExists` acepta un argumento `Closure`, que recibirá una instancia del generador de consultas que le permite definir la consulta que debe colocarse dentro de la cláusula "exists":
> > The `whereExists` method allows you to write `where exists` SQL clauses. The `whereExists` method accepts a `Closure` argument, which will receive a query builder instance allowing you to define the query that should be placed inside of the "exists" clause:

    DB::table('users')
                ->whereExists(function ($query) {
                    $query->select(DB::raw(1))
                          ->from('orders')
                          ->whereRaw('orders.user_id = users.id');
                })
                ->get();

La consulta anterior producirá el siguiente SQL:
> > The query above will produce the following SQL:

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )

<a name="json-where-clauses"></a>
### JSON Where Clauses

Laravel también admite la consulta de tipos de columnas JSON en bases de datos que brindan soporte para tipos de columnas JSON. Actualmente, esto incluye MySQL 5.7, PostgreSQL y SQL Server 2016. Para consultar una columna JSON, use el operador `->`:
> > Laravel also supports querying JSON column types on databases that provide support for JSON column types. Currently, this includes MySQL 5.7, PostgreSQL, and SQL Server 2016. To query a JSON column, use the `->` operator:

    $users = DB::table('users')
                    ->where('options->language', 'en')
                    ->get();

    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();
                    
Puede usar `whereJsonContains` para consultar arrays JSON:
> > You may use `whereJsonContains` to query JSON arrays:
                    
    $users = DB::table('users')
                    ->whereJsonContains('options->languages', 'en')
                    ->get();

MySQL y PostgreSQL admiten `whereJsonContains` con múltiples valores:
> > MySQL and PostgreSQL support `whereJsonContains` with multiple values:

    $users = DB::table('users')
                    ->whereJsonContains('options->languages', ['en', 'de'])
                    ->get();                    

<a name="ordering-grouping-limit-and-offset"></a>
## Ordenando, agrupando, límite y compensación : Ordering, Grouping, Limit, & Offset

#### orderBy

El método `orderBy` le permite ordenar el resultado de la consulta por una columna determinada. El primer argumento para el método `orderBy` debe ser la columna por la que desea ordenar, mientras que el segundo argumento controla la dirección del género y puede ser `asc` o `desc`:
> > The `orderBy` method allows you to sort the result of the query by a given column. The first argument to the `orderBy` method should be the column you wish to sort by, while the second argument controls the direction of the sort and may be either `asc` or `desc`:

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

#### Últimos / más antiguos : latest / oldest

Los métodos `latest` y `oldest` le permiten ordenar fácilmente los resultados por fecha. Por defecto, el resultado será ordenado por la columna `created_at`. O bien, puede pasar el nombre de la columna que desea ordenar por:
> > The `latest` and `oldest` methods allow you to easily order results by date. By default, result will be ordered by the `created_at` column. Or, you may pass the column name that you wish to sort by:

    $user = DB::table('users')
                    ->latest()
                    ->first();

#### Orden aleatorio : inRandomOrder

El método `inRandomOrder` se puede usar para ordenar los resultados de la consulta aleatoriamente. Por ejemplo, puede usar este método para buscar un usuario aleatorio:
> > The `inRandomOrder` method may be used to sort the query results randomly. For example, you may use this method to fetch a random user:

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();

#### groupBy / having

Los métodos `groupBy` y `having` se pueden usar para agrupar los resultados de la consulta. La firma del método `having` es similar a la del método `where`:
> > The `groupBy` and `having` methods may be used to group the query results. The `having` method's signature is similar to that of the `where` method:

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

Puede pasar múltiples argumentos al método `groupBy` para agrupar por varias columnas:
> > You may pass multiple arguments to the `groupBy` method to group by multiple columns:

    $users = DB::table('users')
                    ->groupBy('first_name', 'status')
                    ->having('account_id', '>', 100)
                    ->get();

Para las declaraciones `having` más avanzadas, vea el método [`havingRaw`](#raw-methods).
> > For more advanced `having` statements, see the [`havingRaw`](#raw-methods) method.

#### skip / take

Para limitar el número de resultados devueltos por la consulta, o para omitir un número determinado de resultados en la consulta, puede usar los métodos `skip` y `take`:
> > To limit the number of results returned from the query, or to skip a given number of results in the query, you may use the `skip` and `take` methods:

    $users = DB::table('users')->skip(10)->take(5)->get();

Alternativamente, puede usar los métodos `limit` y `offset`:
> > Alternatively, you may use the `limit` and `offset` methods:

    $users = DB::table('users')
                    ->offset(10)
                    ->limit(5)
                    ->get();

<a name="conditional-clauses"></a>
## Cláusulas condicionales : Conditional Clauses

En ocasiones, es posible que desee que las cláusulas se apliquen a una consulta solo cuando algo más sea verdadero. Por ejemplo, puede que solo desee aplicar una declaración `where` si un valor de entrada dado está presente en la solicitud entrante. Puede lograr esto usando el método `when`:
> > Sometimes you may want clauses to apply to a query only when something else is true. For instance you may only want to apply a `where` statement if a given input value is present on the incoming request. You may accomplish this using the `when` method:

    $role = $request->input('role');

    $users = DB::table('users')
                    ->when($role, function ($query, $role) {
                        return $query->where('role_id', $role);
                    })
                    ->get();

El método `when` solo ejecuta el Closure dado cuando el primer parámetro es `true`. Si el primer parámetro es `false`, el Closure no se ejecutará.
> > The `when` method only executes the given Closure when the first parameter is `true`. If the first parameter is `false`, the Closure will not be executed.

Puede pasar otro Closure como tercer parámetro al método `when`. Este cierre se ejecutará si el primer parámetro se evalúa como `false`. Para ilustrar cómo se puede usar esta característica, la usaremos para configurar la clasificación predeterminada de una consulta:
> > You may pass another Closure as the third parameter to the `when` method. This Closure will execute if the first parameter evaluates as `false`. To illustrate how this feature may be used, we will use it to configure the default sorting of a query:

    $sortBy = null;

    $users = DB::table('users')
                    ->when($sortBy, function ($query, $sortBy) {
                        return $query->orderBy($sortBy);
                    }, function ($query) {
                        return $query->orderBy('name');
                    })
                    ->get();

<a name="inserts"></a>
## Inserts

El generador de consultas también proporciona un método `insert` para insertar registros en la tabla de la base de datos. El método `insert` acepta un array de nombres y valores de columnas:
> > The query builder also provides an `insert` method for inserting records into the database table. The `insert` method accepts an array of column names and values:

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );

Incluso puede insertar varios registros en la tabla con una sola llamada a `insert` pasando un array de arrays. Cada array representa una fila para insertar en la tabla:
> > You may even insert several records into the table with a single call to `insert` by passing an array of arrays. Each array represents a row to be inserted into the table:

    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);

#### IDs autoincrementales : Auto-Incrementing IDs

Si la tabla tiene un ID autoincremental, use el método `insertGetId` para insertar un registro y luego recuperar la ID:
> > If the table has an auto-incrementing id, use the `insertGetId` method to insert a record and then retrieve the ID:

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> {note} Cuando se usa PostgreSQL, el método `insertGetId` espera que la columna de incremento automático se denomine `id`. Si desea recuperar la ID de una "secuencia" diferente, puede pasar el nombre de la columna como segundo parámetro al método `insertGetId`.
> > > {note} When using PostgreSQL the `insertGetId` method expects the auto-incrementing column to be named `id`. If you would like to retrieve the ID from a different "sequence", you may pass the column name as the second parameter to the `insertGetId` method.

<a name="updates"></a>
## Updates

Por supuesto, además de insertar registros en la base de datos, el generador de consultas también puede actualizar los registros existentes usando el método `update`. El método `update`, al igual que el método `insert`, acepta un array de columnas y pares de valores que contienen las columnas que se actualizarán. Puede restringir la consulta `update` usando las cláusulas `where`:
> > Of course, in addition to inserting records into the database, the query builder can also update existing records using the `update` method. The `update` method, like the `insert` method, accepts an array of column and value pairs containing the columns to be updated. You may constrain the `update` query using `where` clauses:

    DB::table('users')
                ->where('id', 1)
                ->update(['votes' => 1]);

<a name="updating-json-columns"></a>
### Updating JSON Columns

Al actualizar una columna JSON, debe usar la sintaxis `->` para acceder a la clave apropiada en el objeto JSON. Esta operación solo se admite en bases de datos que admiten columnas JSON:
> > When updating a JSON column, you should use `->` syntax to access the appropriate key in the JSON object. This operation is only supported on databases that support JSON columns:

    DB::table('users')
                ->where('id', 1)
                ->update(['options->enabled' => true]);

<a name="increment-and-decrement"></a>
### Incremento y Decremento : Increment & Decrement

El generador de consultas también proporciona métodos convenientes para incrementar o disminuir el valor de una columna determinada. Este es un acceso directo, que proporciona una interfaz más expresiva y concisa en comparación con la escritura manual de la declaración `update`.
> > The query builder also provides convenient methods for incrementing or decrementing the value of a given column. This is a shortcut, providing a more expressive and terse interface compared to manually writing the `update` statement.

Ambos métodos aceptan al menos un argumento: la columna a modificar. Opcionalmente, se puede pasar un segundo argumento para controlar la cantidad por la cual la columna debe incrementarse o decrementarse:
> > Both of these methods accept at least one argument: the column to modify. A second argument may optionally be passed to control the amount by which the column should be incremented or decremented:

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

También puede especificar columnas adicionales para actualizar durante la operación:
> > You may also specify additional columns to update during the operation:

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

<a name="deletes"></a>
## Deletes

El generador de consultas también se puede usar para eliminar registros de la tabla mediante el método `delete`. Puede restringir las instrucciones `delete` agregando `where` clauses antes de llamar al método `delete`:
> > The query builder may also be used to delete records from the table via the `delete` method. You may constrain `delete` statements by adding `where` clauses before calling the `delete` method:

    DB::table('users')->delete();

    DB::table('users')->where('votes', '>', 100)->delete();

Si desea truncar toda la tabla, lo que eliminará todas las filas y restablecerá el ID de incremento automático a cero, puede usar el método `truncate`:
> > If you wish to truncate the entire table, which will remove all rows and reset the auto-incrementing ID to zero, you may use the `truncate` method:

    DB::table('users')->truncate();

<a name="pessimistic-locking"></a>
## Bloqueo pesimista : Pessimistic Locking

El generador de consultas también incluye algunas funciones para ayudarlo a realizar un "bloqueo pesimista" en sus declaraciones `select`. Para ejecutar la declaración con un "bloqueo compartido", puede usar el método `sharedLock` en una consulta. Un bloqueo compartido impide que las filas seleccionadas se modifiquen hasta que la transacción se comprometa:
> > The query builder also includes a few functions to help you do "pessimistic locking" on your `select` statements. To run the statement with a "shared lock", you may use the `sharedLock` method on a query. A shared lock prevents the selected rows from being modified until your transaction commits:

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

Alternativamente, puede usar el método `lockForUpdate`. Un bloqueo "para actualizar" impide que las filas se modifiquen o que se seleccionen con otro bloqueo compartido:
> > Alternatively, you may use the `lockForUpdate` method. A "for update" lock prevents the rows from being modified or from being selected with another shared lock:

    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
