# Eloquent: Relaciones : Eloquent: Relationships

- [Introduction](#introduction)
- [Defining Relationships](#defining-relationships)
    - [One To One](#one-to-one)
    - [One To Many](#one-to-many)
    - [One To Many (Inverse)](#one-to-many-inverse)
    - [Many To Many](#many-to-many)
    - [Has Many Through](#has-many-through)
    - [Polymorphic Relations](#polymorphic-relations)
    - [Many To Many Polymorphic Relations](#many-to-many-polymorphic-relations)
- [Querying Relations](#querying-relations)
    - [Relationship Methods Vs. Dynamic Properties](#relationship-methods-vs-dynamic-properties)
    - [Querying Relationship Existence](#querying-relationship-existence)
    - [Querying Relationship Absence](#querying-relationship-absence)
    - [Counting Related Models](#counting-related-models)
- [Eager Loading](#eager-loading)
    - [Constraining Eager Loads](#constraining-eager-loads)
    - [Lazy Eager Loading](#lazy-eager-loading)
- [Inserting & Updating Related Models](#inserting-and-updating-related-models)
    - [The `save` Method](#the-save-method)
    - [The `create` Method](#the-create-method)
    - [Belongs To Relationships](#updating-belongs-to-relationships)
    - [Many To Many Relationships](#updating-many-to-many-relationships)
- [Touching Parent Timestamps](#touching-parent-timestamps)

<a name="introduction"></a>
## Introducción : Introduction

Las tablas de la base de datos a menudo están relacionadas entre sí. Por ejemplo, una publicación de blog puede tener muchos comentarios, o una orden podría estar relacionada con el usuario que la colocó. Eloquent facilita la gestión y el trabajo con estas relaciones, y admite varios tipos diferentes de relaciones:
> > Database tables are often related to one another. For example, a blog post may have many comments, or an order could be related to the user who placed it. Eloquent makes managing and working with these relationships easy, and supports several different types of relationships:

- [Uno a uno](#one-to-one) [One To One]
- [Uno a muchos](#one-to-many) [One To Many]
- [Muchos a muchos](#many-to-many) [Many To Many]
- [Tiene muchos a través](#has-many-through) [Has Many Through]
- [Relaciones polimórficas](#polymorphic-relations) [Polymorphic Relations]
- [Muchos a muchos relaciones polimórficas](#many-to-many-polymorphic-relations) [Many To Many Polymorphic Relations]

<a name="defining-relationships"></a>
## Definición de relaciones : Defining Relationships

Las relaciones Eloquent se definen como métodos en sus clases modelo Eloquent. Dado que, al igual que los modelos Eloquent, las relaciones también sirven como potentes [constructores de consultas](/docs/{{version}}/queries), la definición de relaciones como métodos proporciona potentes funciones de encadenamiento y consulta de métodos. Por ejemplo, podemos encadenar restricciones adicionales en esta relación de `posts`:
> > Eloquent relationships are defined as methods on your Eloquent model classes. Since, like Eloquent models themselves, relationships also serve as powerful [query builders](/docs/{{version}}/queries), defining relationships as methods provides powerful method chaining and querying capabilities. For example, we may chain additional constraints on this `posts` relationship:

    $user->posts()->where('active', 1)->get();

Pero, antes de sumergirnos demasiado en el uso de las relaciones, aprendamos a definir cada tipo.
> > But, before diving too deep into using relationships, let's learn how to define each type.

<a name="one-to-one"></a>
### Uno a uno : One To One

Una relación uno a uno es una relación muy básica. Por ejemplo, un modelo `User` podría estar asociado con un `Phone`. Para definir esta relación, colocamos un método `phone` en el modelo `User`. El método `phone` debe llamar al método `hasOne` y devolver su resultado:
> > A one-to-one relationship is a very basic relation. For example, a `User` model might be associated with one `Phone`. To define this relationship, we place a `phone` method on the `User` model. The `phone` method should call the `hasOne` method and return its result:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the phone record associated with the user.
         */
        public function phone()
        {
            return $this->hasOne('App\Phone');
        }
    }

El primer argumento pasado al método `hasOne` es el nombre del modelo relacionado. Una vez que se define la relación, podemos recuperar el registro relacionado utilizando las propiedades dinámicas de Eloquent. Las propiedades dinámicas le permiten acceder a los métodos de relación como si fueran propiedades definidas en el modelo:
> > The first argument passed to the `hasOne` method is the name of the related model. Once the relationship is defined, we may retrieve the related record using Eloquent's dynamic properties. Dynamic properties allow you to access relationship methods as if they were properties defined on the model:

    $phone = User::find(1)->phone;

Eloquent determina la clave externa de la relación basada en el nombre del modelo. En este caso, se asume automáticamente que el modelo `Phone` tiene una clave externa `user_id`. Si desea anular esta convención, puede pasar un segundo argumento al método `hasOne`:
> > Eloquent determines the foreign key of the relationship based on the model name. In this case, the `Phone` model is automatically assumed to have a `user_id` foreign key. If you wish to override this convention, you may pass a second argument to the `hasOne` method:

    return $this->hasOne('App\Phone', 'foreign_key');

Además, Eloquent asume que la clave externa debe tener un valor que coincida con la columna `id` (o la `$primaryKey` personalizada) del padre. En otras palabras, Eloquent buscará el valor de la columna `id` del usuario en la columna `user_id` del registro `Phone`. Si desea que la relación utilice un valor que no sea `id`, puede pasar un tercer argumento al método `hasOne` que especifica su clave personalizada:
> > Additionally, Eloquent assumes that the foreign key should have a value matching the `id` (or the custom `$primaryKey`) column of the parent. In other words, Eloquent will look for the value of the user's `id` column in the `user_id` column of the `Phone` record. If you would like the relationship to use a value other than `id`, you may pass a third argument to the `hasOne` method specifying your custom key:

    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

#### Definición del inverso de la relación : Defining The Inverse Of The Relationship

Entonces, podemos acceder al modelo `Phone` de nuestro `User`. Ahora, definamos una relación en el modelo `Phone` que nos permitirá acceder al `User` que posee el teléfono. Podemos definir el inverso de una relación `hasOne` utilizando el método `belongsTo`:
> > So, we can access the `Phone` model from our `User`. Now, let's define a relationship on the `Phone` model that will let us access the `User` that owns the phone. We can define the inverse of a `hasOne` relationship using the `belongsTo` method:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Phone extends Model
    {
        /**
         * Get the user that owns the phone.
         */
        public function user()
        {
            return $this->belongsTo('App\User');
        }
    }

En el ejemplo anterior, Eloquent intentará hacer coincidir el `user_id` del modelo `Phone` con un `id` en el modelo `User`. Eloquent determina el nombre de la clave foránea por defecto examinando el nombre del método de relación y el sufijo del nombre del método con `_id`. Sin embargo, si la clave externa en el modelo `Phone` no es `user_id`, puede pasar un nombre de clave personalizado como segundo argumento al método `belongsTo`:
> > In the example above, Eloquent will try to match the `user_id` from the `Phone` model to an `id` on the `User` model. Eloquent determines the default foreign key name by examining the name of the relationship method and suffixing the method name with `_id`. However, if the foreign key on the `Phone` model is not `user_id`, you may pass a custom key name as the second argument to the `belongsTo` method:

    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key');
    }

Si su modelo principal no usa `id` como su clave principal, o si desea unir el modelo secundario a una columna diferente, puede pasar un tercer argumento al método `belongsTo` especificando la clave personalizada de su tabla primaria:
> > If your parent model does not use `id` as its primary key, or you wish to join the child model to a different column, you may pass a third argument to the `belongsTo` method specifying your parent table's custom key:

    /**
     * Get the user that owns the phone.
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key', 'other_key');
    }

<a name="one-to-many"></a>
### Uno a muchos : One To Many

Una relación "uno a muchos" se usa para definir las relaciones en las que un único modelo posee cualquier cantidad de otros modelos. Por ejemplo, una publicación de blog puede tener una cantidad infinita de comentarios. Como todas las otras relaciones Eloquent, las relaciones uno a muchos se definen al colocar una función en su modelo Eloquent:
> > A "one-to-many" relationship is used to define relationships where a single model owns any amount of other models. For example, a blog post may have an infinite number of comments. Like all other Eloquent relationships, one-to-many relationships are defined by placing a function on your Eloquent model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * Get the comments for the blog post.
         */
        public function comments()
        {
            return $this->hasMany('App\Comment');
        }
    }

Recuerde, Eloquent determinará automáticamente la columna de clave externa apropiada en el modelo `Comment`. Por convención, Eloquent tomará el nombre de "snake case" del modelo propietario y lo agregará con `_id`. Entonces, para este ejemplo, Eloquent asumirá que la clave externa en el modelo `Comment` es `post_id`.
> > Remember, Eloquent will automatically determine the proper foreign key column on the `Comment` model. By convention, Eloquent will take the "snake case" name of the owning model and suffix it with `_id`. So, for this example, Eloquent will assume the foreign key on the `Comment` model is `post_id`.

Una vez que se ha definido la relación, podemos acceder al conjunto de comentarios accediendo a la propiedad `comments`. Recuerde, dado que Eloquent proporciona "propiedades dinámicas", podemos acceder a los métodos de relación como si estuvieran definidos como propiedades en el modelo:
> > Once the relationship has been defined, we can access the collection of comments by accessing the `comments` property. Remember, since Eloquent provides "dynamic properties", we can access relationship methods as if they were defined as properties on the model:

    $comments = App\Post::find(1)->comments;

    foreach ($comments as $comment) {
        //
    }

Por supuesto, dado que todas las relaciones también sirven como constructores de consultas, puede agregar restricciones adicionales a los comentarios que se recuperan llamando al método `comments` y continuando las condiciones de la cadena en la consulta:
> > Of course, since all relationships also serve as query builders, you can add further constraints to which comments are retrieved by calling the `comments` method and continuing to chain conditions onto the query:

    $comment = App\Post::find(1)->comments()->where('title', 'foo')->first();

Al igual que el método `hasOne`, también puede anular las claves externas y locales pasando argumentos adicionales al método `hasMany`:
> > Like the `hasOne` method, you may also override the foreign and local keys by passing additional arguments to the `hasMany` method:

    return $this->hasMany('App\Comment', 'foreign_key');

    return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

<a name="one-to-many-inverse"></a>
### Uno a muchos (inverso) : One To Many (Inverse)

Ahora que podemos acceder a todos los comentarios de una publicación, definamos una relación para permitir que un comentario acceda a su publicación principal. Para definir el inverso de una relación `hasMany`, defina una función de relación en el modelo hijo que llama al método `belongsTo`:
> > Now that we can access all of a post's comments, let's define a relationship to allow a comment to access its parent post. To define the inverse of a `hasMany` relationship, define a relationship function on the child model which calls the `belongsTo` method:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Get the post that owns the comment.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

Una vez que se ha definido la relación, podemos recuperar el modelo `Post` para un `Comment` accediendo a la "propiedad dinámica" `post`:
> > Once the relationship has been defined, we can retrieve the `Post` model for a `Comment` by accessing the `post` "dynamic property":

    $comment = App\Comment::find(1);

    echo $comment->post->title;

En el ejemplo anterior, Eloquent intentará hacer coincidir el `post_id` del modelo `Comment` con un `id` en el modelo `Post`. Eloquent determina el nombre predeterminado de la clave externa examinando el nombre del método de relación y agregando el sufijo al nombre del método con un `_` seguido del nombre de la columna de la clave principal. Sin embargo, si la clave externa en el modelo `Comment` no es `post_id`, puede pasar un nombre de clave personalizado como segundo argumento al método `belongsTo`:
> > In the example above, Eloquent will try to match the `post_id` from the `Comment` model to an `id` on the `Post` model. Eloquent determines the default foreign key name by examining the name of the relationship method and suffixing the method name with a `_` followed by the name of the primary key column. However, if the foreign key on the `Comment` model is not `post_id`, you may pass a custom key name as the second argument to the `belongsTo` method:

    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key');
    }

Si su modelo principal no usa `id` como su clave principal, o si desea unir el modelo secundario a una columna diferente, puede pasar un tercer argumento al método `belongsTo` especificando la clave personalizada de su tabla primaria:
> > If your parent model does not use `id` as its primary key, or you wish to join the child model to a different column, you may pass a third argument to the `belongsTo` method specifying your parent table's custom key:

    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
    }

<a name="many-to-many"></a>
### Muchos a muchos : Many To Many

Las relaciones de muchos a muchos son un poco más complicadas que las relaciones `hasOne` y `hasMany`. Un ejemplo de tal relación es un usuario con muchos roles, donde los roles también son compartidos por otros usuarios. Por ejemplo, muchos usuarios pueden tener el rol de "Administrador". Para definir esta relación, se necesitan tres tablas de base de datos: `users`,` roles`, y `role_user`. La tabla `role_user` se deriva del orden alfabético de los nombres de modelos relacionados, y contiene las columnas `user_id` y `role_id`.
> > Many-to-many relations are slightly more complicated than `hasOne` and `hasMany` relationships. An example of such a relationship is a user with many roles, where the roles are also shared by other users. For example, many users may have the role of "Admin". To define this relationship, three database tables are needed: `users`, `roles`, and `role_user`. The `role_user` table is derived from the alphabetical order of the related model names, and contains the `user_id` and `role_id` columns.

Las relaciones de muchos a muchos se definen escribiendo un método que devuelve el resultado del método `belongsToMany`. Por ejemplo, definamos el método `roles` en nuestro modelo `User`:
> > Many-to-many relationships are defined by writing a method that returns the result of the `belongsToMany` method. For example, let's define the `roles` method on our `User` model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The roles that belong to the user.
         */
        public function roles()
        {
            return $this->belongsToMany('App\Role');
        }
    }

Una vez que se define la relación, puede acceder a los roles del usuario utilizando la propiedad dinámica `roles`:
> > Once the relationship is defined, you may access the user's roles using the `roles` dynamic property:

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        //
    }

Por supuesto, como todos los demás tipos de relaciones, puede llamar al método `roles` para continuar encadenando las restricciones de consulta a la relación:
> > Of course, like all other relationship types, you may call the `roles` method to continue chaining query constraints onto the relationship:

    $roles = App\User::find(1)->roles()->orderBy('name')->get();

Como se mencionó anteriormente, para determinar el nombre de la tabla de unión de la relación, Eloquent unirá los dos nombres de modelos relacionados en orden alfabético. Sin embargo, eres libre de anular esta convención. Puede hacerlo pasando un segundo argumento al método `belongsToMany`:
> > As mentioned previously, to determine the table name of the relationship's joining table, Eloquent will join the two related model names in alphabetical order. However, you are free to override this convention. You may do so by passing a second argument to the `belongsToMany` method:

    return $this->belongsToMany('App\Role', 'role_user');

Además de personalizar el nombre de la tabla de unión, también puede personalizar los nombres de columna de las claves en la tabla pasando argumentos adicionales al método `belongsToMany`. El tercer argumento es el nombre de la clave externa del modelo en el que está definiendo la relación, mientras que el cuarto argumento es el nombre de la clave externa del modelo al que se está uniendo:
> > In addition to customizing the name of the joining table, you may also customize the column names of the keys on the table by passing additional arguments to the `belongsToMany` method. The third argument is the foreign key name of the model on which you are defining the relationship, while the fourth argument is the foreign key name of the model that you are joining to:

    return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');

#### Definición del inverso de la relación : Defining The Inverse Of The Relationship

Para definir el inverso de una relación de muchos a muchos, realiza otra llamada a `belongsToMany` en su modelo relacionado. Para continuar nuestro ejemplo de roles de usuario, definamos el método `users` en el modelo `Role`:
> > To define the inverse of a many-to-many relationship, you place another call to `belongsToMany` on your related model. To continue our user roles example, let's define the `users` method on the `Role` model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\User');
        }
    }

Como puede ver, la relación se define exactamente igual que su contraparte `User`, con la excepción de hacer referencia al modelo `App\User`. Como estamos reutilizando el método `belongsToMany`, todas las opciones habituales de personalización de tablas y teclas están disponibles cuando se define el inverso de las relaciones entre varios.
> > As you can see, the relationship is defined exactly the same as its `User` counterpart, with the exception of referencing the `App\User` model. Since we're reusing the `belongsToMany` method, all of the usual table and key customization options are available when defining the inverse of many-to-many relationships.

#### Recuperación de columnas de tablas intermedias : Retrieving Intermediate Table Columns

Como ya aprendió, trabajar con relaciones de muchos a muchos requiere la presencia de una tabla intermedia. Eloquent proporciona algunas formas muy útiles de interactuar con esta tabla. Por ejemplo, supongamos que nuestro objeto `User` tiene muchos objetos `Role` con los que está relacionado. Después de acceder a esta relación, podemos acceder a la tabla intermedia utilizando el atributo `pivot` en los modelos:
> > As you have already learned, working with many-to-many relations requires the presence of an intermediate table. Eloquent provides some very helpful ways of interacting with this table. For example, let's assume our `User` object has many `Role` objects that it is related to. After accessing this relationship, we may access the intermediate table using the `pivot` attribute on the models:

    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

Tenga en cuenta que a cada modelo `Role` que recuperamos se le asigna automáticamente un atributo `pivot`. Este atributo contiene un modelo que representa la tabla intermedia y puede usarse como cualquier otro modelo de Eloquent.
> > Notice that each `Role` model we retrieve is automatically assigned a `pivot` attribute. This attribute contains a model representing the intermediate table, and may be used like any other Eloquent model.

Por defecto, solo las claves del modelo estarán presentes en el objeto `pivot`. Si su tabla dinámica contiene atributos adicionales, debe especificarlos al definir la relación:
> > By default, only the model keys will be present on the `pivot` object. If your pivot table contains extra attributes, you must specify them when defining the relationship:

    return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');

Si desea que su tabla pivote mantenga automáticamente las marcas de tiempo `created_at` y `updated_at`, use el método `withTimestamps` en la definición de la relación:
> > If you want your pivot table to have automatically maintained `created_at` and `updated_at` timestamps, use the `withTimestamps` method on the relationship definition:

    return $this->belongsToMany('App\Role')->withTimestamps();

#### Personalización del nombre del atributo `pivot` : Customizing The `pivot` Attribute Name

Como se señaló anteriormente, se puede acceder a los atributos de la tabla intermedia en los modelos que usan el atributo `pivot`. Sin embargo, puede personalizar el nombre de este atributo para reflejar mejor su propósito dentro de su aplicación.
> > As noted earlier, attributes from the intermediate table may be accessed on models using the `pivot` attribute. However, you are free to customize the name of this attribute to better reflect its purpose within your application.

Por ejemplo, si su aplicación contiene usuarios que pueden suscribirse a podcasts, probablemente tenga una relación muchos a muchos entre usuarios y podcasts. Si este es el caso, es posible que desee cambiar el nombre de su acceso a la tabla intermedia a `subscription` en lugar de `pivot`. Esto se puede hacer usando el método `as` al definir la relación:
> > For example, if your application contains users that may subscribe to podcasts, you probably have a many-to-many relationship between users and podcasts. If this is the case, you may wish to rename your intermediate table accessor to `subscription` instead of `pivot`. This can be done using the `as` method when defining the relationship:

    return $this->belongsToMany('App\Podcast')
                    ->as('subscription')
                    ->withTimestamps();

Una vez hecho esto, puede acceder a los datos de la tabla intermedia utilizando el nombre personalizado:
> > Once this is done, you may access the intermediate table data using the customized name:

    $users = User::with('podcasts')->get();

    foreach ($users->flatMap->podcasts as $podcast) {
        echo $podcast->subscription->created_at;
    }

#### Filtrado de relaciones a través de columnas de tablas intermedias : Filtering Relationships Via Intermediate Table Columns

También puede filtrar los resultados devueltos por `belongsToMany` utilizando los métodos `wherePivot` y `wherePivotIn` al definir la relación:
> > You can also filter the results returned by `belongsToMany` using the `wherePivot` and `wherePivotIn` methods when defining the relationship:

    return $this->belongsToMany('App\Role')->wherePivot('approved', 1);

    return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);

#### Definición de modelos de tablas intermedias personalizadas : Defining Custom Intermediate Table Models

Si desea definir un modelo personalizado para representar la tabla intermedia de su relación, puede llamar al método `using` cuando defina la relación. Los modelos pivote personalizados de muchos a muchos deberían ampliar la clase `Illuminate\Database\Eloquent\Relations\Pivot`, mientras que los modelos pivotantes polimórficos personalizados muchos a muchos deberían extender la clase `Illuminate\Database\Eloquent\Relations\MorphPivot`. Por ejemplo, podemos definir un `Role` que utiliza un modelo de pivote `UserRole` personalizado:
> > If you would like to define a custom model to represent the intermediate table of your relationship, you may call the `using` method when defining the relationship. Custom many-to-many pivot models should extend the `Illuminate\Database\Eloquent\Relations\Pivot` class while custom polymorphic many-to-many pivot models should extend the `Illuminate\Database\Eloquent\Relations\MorphPivot` class. For example, we may define a `Role` which uses a custom `UserRole` pivot model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\User')->using('App\UserRole');
        }
    }

Al definir el modelo `UserRole`, ampliaremos la clase `Pivot`:
> > When defining the `UserRole` model, we will extend the `Pivot` class:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Relations\Pivot;

    class UserRole extends Pivot
    {
        //
    }

<a name="has-many-through"></a>
### Tiene muchos a través : Has Many Through

La relación "tiene muchos a través" proporciona un atajo para acceder a relaciones distantes a través de una relación intermedia. Por ejemplo, un modelo `Country` podría tener muchos modelos `Post` a través de un modelo `User` intermedio. En este ejemplo, puede reunir fácilmente todas las publicaciones de blog para un país determinado. Veamos las tablas requeridas para definir esta relación:
> > The "has-many-through" relationship provides a convenient shortcut for accessing distant relations via an intermediate relation. For example, a `Country` model might have many `Post` models through an intermediate `User` model. In this example, you could easily gather all blog posts for a given country. Let's look at the tables required to define this relationship:

    countries
        id - integer
        name - string

    users
        id - integer
        country_id - integer
        name - string

    posts
        id - integer
        user_id - integer
        title - string

Aunque `posts` no contiene una columna `country_id`, la relación `hasManyThrough` proporciona acceso a las publicaciones de un país a través de `$country->posts`. Para realizar esta consulta, Eloquent inspecciona `country_id` en la tabla intermedia `users`. Después de encontrar las ID de usuario coincidentes, se utilizan para consultar la tabla `posts`.
> > Though `posts` does not contain a `country_id` column, the `hasManyThrough` relation provides access to a country's posts via `$country->posts`. To perform this query, Eloquent inspects the `country_id` on the intermediate `users` table. After finding the matching user IDs, they are used to query the `posts` table.

Ahora que hemos examinado la estructura de la tabla para la relación, vamos a definirla en el modelo `Country`:
> > Now that we have examined the table structure for the relationship, let's define it on the `Country` model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Country extends Model
    {
        /**
         * Get all of the posts for the country.
         */
        public function posts()
        {
            return $this->hasManyThrough('App\Post', 'App\User');
        }
    }

El primer argumento pasado al método `hasManyThrough` es el nombre del modelo final al que deseamos acceder, mientras que el segundo argumento es el nombre del modelo intermedio.
> > The first argument passed to the `hasManyThrough` method is the name of the final model we wish to access, while the second argument is the name of the intermediate model.

Las convenciones Eloquent de clave externa típicas se usarán al realizar las consultas de la relación. Si desea personalizar las claves de la relación, puede pasarlas como tercer y cuarto argumentos del método `hasManyThrough`. El tercer argumento es el nombre de la clave externa en el modelo intermedio. El cuarto argumento es el nombre de la clave externa en el modelo final. El quinto argumento es la clave local, mientras que el sexto argumento es la clave local del modelo intermedio:
> > Typical Eloquent foreign key conventions will be used when performing the relationship's queries. If you would like to customize the keys of the relationship, you may pass them as the third and fourth arguments to the `hasManyThrough` method. The third argument is the name of the foreign key on the intermediate model. The fourth argument is the name of the foreign key on the final model. The fifth argument is the local key, while the sixth argument is the local key of the intermediate model:

    class Country extends Model
    {
        public function posts()
        {
            return $this->hasManyThrough(
                'App\Post',
                'App\User',
                'country_id', // Foreign key on users table...
                'user_id', // Foreign key on posts table...
                'id', // Local key on countries table...
                'id' // Local key on users table...
            );
        }
    }

<a name="polymorphic-relations"></a>
### Relaciones polimórficas : Polymorphic Relations

#### Estructura de tabla : Table Structure

Las relaciones polimórficas permiten que un modelo pertenezca a más de otro modelo en una sola asociación. Por ejemplo, imagine que los usuarios de su aplicación pueden "comentar" tanto en publicaciones como en videos. Usando relaciones polimórficas, puede usar una sola tabla `comments` para ambos escenarios. Primero, examinemos la estructura de la tabla requerida para construir esta relación:
> > Polymorphic relations allow a model to belong to more than one other model on a single association. For example, imagine users of your application can "comment" on both posts and videos. Using polymorphic relationships, you can use a single `comments` table for both of these scenarios. First, let's examine the table structure required to build this relationship:

    posts
        id - integer
        title - string
        body - text

    videos
        id - integer
        title - string
        url - string

    comments
        id - integer
        body - text
        commentable_id - integer
        commentable_type - string

Dos columnas importantes a tener en cuenta son las columnas `commentable_id` y` commentable_type` en la tabla `comments`. La columna `commentable_id` contendrá el valor ID de la publicación o video, mientras que la columna `commentable_type` contendrá el nombre de la clase del modelo propietario. La columna `commentable_type` es cómo el ORM determina qué "tipo" de modelo propietario devolverá cuando acceda a la relación `commentable`.
> > Two important columns to note are the `commentable_id` and `commentable_type` columns on the `comments` table. The `commentable_id` column will contain the ID value of the post or video, while the `commentable_type` column will contain the class name of the owning model. The `commentable_type` column is how the ORM determines which "type" of owning model to return when accessing the `commentable` relation.

#### Estructura del modelo : Model Structure

A continuación, examinemos las definiciones de modelo necesarias para construir esta relación:
> > Next, let's examine the model definitions needed to build this relationship:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * Get all of the owning commentable models.
         */
        public function commentable()
        {
            return $this->morphTo();
        }
    }

    class Post extends Model
    {
        /**
         * Get all of the post's comments.
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

    class Video extends Model
    {
        /**
         * Get all of the video's comments.
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

#### Recuperación de relaciones polimórficas : Retrieving Polymorphic Relations

Una vez que su tabla de base de datos y sus modelos están definidos, puede acceder a las relaciones a través de sus modelos. Por ejemplo, para acceder a todos los comentarios de una publicación, podemos usar la propiedad dinámica `comments`:
> > Once your database table and models are defined, you may access the relationships via your models. For example, to access all of the comments for a post, we can use the `comments` dynamic property:

    $post = App\Post::find(1);

    foreach ($post->comments as $comment) {
        //
    }

También puede recuperar al propietario de una relación polimórfica del modelo polimórfico accediendo al nombre del método que realiza la llamada a `morphTo`. En nuestro caso, ese es el método `commentable` en el modelo `Comment`. Entonces, accederemos a ese método como una propiedad dinámica:
> > You may also retrieve the owner of a polymorphic relation from the polymorphic model by accessing the name of the method that performs the call to `morphTo`. In our case, that is the `commentable` method on the `Comment` model. So, we will access that method as a dynamic property:

    $comment = App\Comment::find(1);

    $commentable = $comment->commentable;

La relación `commentable` en el modelo `Comment` devolverá una instancia `Post` o `Video`, según el tipo de modelo que tenga el comentario.
> > The `commentable` relation on the `Comment` model will return either a `Post` or `Video` instance, depending on which type of model owns the comment.

#### Tipos polimórficos personalizados : Custom Polymorphic Types

Por defecto, Laravel utilizará el nombre de clase totalmente calificado para almacenar el tipo del modelo relacionado. Por ejemplo, dado el ejemplo anterior donde un `Comentario` puede pertenecer a un `Post` o `Video`, el `commentable_type` predeterminado sería `App\Post` o `App\Video`, respectivamente. Sin embargo, es posible que desee desacoplar su base de datos de la estructura interna de su aplicación. En ese caso, puede definir una relación "morph map" para indicar a Eloquent que use un nombre personalizado para cada modelo en lugar del nombre de la clase:
> > By default, Laravel will use the fully qualified class name to store the type of the related model. For instance, given the example above where a `Comment` may belong to a `Post` or a `Video`, the default `commentable_type` would be either `App\Post` or `App\Video`, respectively. However, you may wish to decouple your database from your application's internal structure. In that case, you may define a relationship "morph map" to instruct Eloquent to use a custom name for each model instead of the class name:

    use Illuminate\Database\Eloquent\Relations\Relation;

    Relation::morphMap([
        'posts' => 'App\Post',
        'videos' => 'App\Video',
    ]);

Puede registrar el `morphMap` en la función `boot` de `AppServiceProvider` o crear un proveedor de servicios por separado si lo desea.
> > You may register the `morphMap` in the `boot` function of your `AppServiceProvider` or create a separate service provider if you wish.

<a name="many-to-many-polymorphic-relations"></a>
### Relaciones polimórficas de muchos a muchos : Many To Many Polymorphic Relations

#### Estructura de tabla : Table Structure

Además de las relaciones polimórficas tradicionales, también puede definir relaciones polimórficas de "muchos a muchos". Por ejemplo, un modelo de blog 'Post' y 'Video' podría compartir una relación polimórfica con un modelo `Tag`. El uso de una relación polimórfica de muchos a muchos le permite tener una lista única de etiquetas únicas que se comparten entre publicaciones de blogs y videos. Primero, examinemos la estructura de la tabla:
> > In addition to traditional polymorphic relations, you may also define "many-to-many" polymorphic relations. For example, a blog `Post` and `Video` model could share a polymorphic relation to a `Tag` model. Using a many-to-many polymorphic relation allows you to have a single list of unique tags that are shared across blog posts and videos. First, let's examine the table structure:

    posts
        id - integer
        name - string

    videos
        id - integer
        name - string

    tags
        id - integer
        name - string

    taggables
        tag_id - integer
        taggable_id - integer
        taggable_type - string

#### Estructura del modelo : Model Structure

Luego, estamos listos para definir las relaciones en el modelo. Los modelos `Post` y` Video` tendrán ambos un método `tags` que llama al método `morphToMany` en la clase Eloquent base:
> > Next, we're ready to define the relationships on the model. The `Post` and `Video` models will both have a `tags` method that calls the `morphToMany` method on the base Eloquent class:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * Get all of the tags for the post.
         */
        public function tags()
        {
            return $this->morphToMany('App\Tag', 'taggable');
        }
    }

#### Definición del inverso de la relación : Defining The Inverse Of The Relationship

Luego, en el modelo `Tag`, debe definir un método para cada uno de sus modelos relacionados. Entonces, para este ejemplo, definiremos un método `posts` y un método `videos`:
> > Next, on the `Tag` model, you should define a method for each of its related models. So, for this example, we will define a `posts` method and a `videos` method:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Tag extends Model
    {
        /**
         * Get all of the posts that are assigned this tag.
         */
        public function posts()
        {
            return $this->morphedByMany('App\Post', 'taggable');
        }

        /**
         * Get all of the videos that are assigned this tag.
         */
        public function videos()
        {
            return $this->morphedByMany('App\Video', 'taggable');
        }
    }

#### Recuperando la relación : Retrieving The Relationship

Una vez que su tabla de base de datos y sus modelos están definidos, puede acceder a las relaciones a través de sus modelos. Por ejemplo, para acceder a todas las etiquetas de una publicación, puede usar la propiedad dinámica `tags`:
> > Once your database table and models are defined, you may access the relationships via your models. For example, to access all of the tags for a post, you can use the `tags` dynamic property:

    $post = App\Post::find(1);

    foreach ($post->tags as $tag) {
        //
    }

También puede recuperar al propietario de una relación polimórfica del modelo polimórfico accediendo al nombre del método que realiza la llamada a `morphedByMany`. En nuestro caso, esos son los métodos `posts` o` videos` en el modelo `Tag`. Por lo tanto, accederá a esos métodos como propiedades dinámicas:
> > You may also retrieve the owner of a polymorphic relation from the polymorphic model by accessing the name of the method that performs the call to `morphedByMany`. In our case, that is the `posts` or `videos` methods on the `Tag` model. So, you will access those methods as dynamic properties:

    $tag = App\Tag::find(1);

    foreach ($tag->videos as $video) {
        //
    }

<a name="querying-relations"></a>
## Consultando relaciones : Querying Relations

Como todos los tipos de relaciones Eloquent se definen a través de métodos, puede llamar a esos métodos para obtener una instancia de la relación sin ejecutar realmente las consultas de relación. Además, todos los tipos de relaciones Eloquent también sirven como [constructores de consultas](/docs/{{version}}/queries), lo que le permite continuar las restricciones de cadena en la consulta de relación antes de finalmente ejecutar el SQL en su base de datos.
> > Since all types of Eloquent relationships are defined via methods, you may call those methods to obtain an instance of the relationship without actually executing the relationship queries. In addition, all types of Eloquent relationships also serve as [query builders](/docs/{{version}}/queries), allowing you to continue to chain constraints onto the relationship query before finally executing the SQL against your database.

Por ejemplo, imagina un sistema de blog en el que un modelo `User` tiene muchos modelos asociados de `Post`:
> > For example, imagine a blog system in which a `User` model has many associated `Post` models:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get all of the posts for the user.
         */
        public function posts()
        {
            return $this->hasMany('App\Post');
        }
    }

Puede consultar la relación `publicaciones` y agregar restricciones adicionales a la relación de esta manera:
> > You may query the `posts` relationship and add additional constraints to the relationship like so:

    $user = App\User::find(1);

    $user->posts()->where('active', 1)->get();

Puede utilizar cualquiera de los métodos [generador de consultas](/docs/{{version}}/queries) en la relación, por lo que debe asegurarse de explorar la documentación del generador de consultas para obtener más información sobre todos los métodos disponibles.
> > You are able to use any of the [query builder](/docs/{{version}}/queries) methods on the relationship, so be sure to explore the query builder documentation to learn about all of the methods that are available to you.

<a name="relationship-methods-vs-dynamic-properties"></a>
### Métodos de relación vs. Propiedades dinámicas : Relationship Methods Vs. Dynamic Properties

Si no necesita agregar restricciones adicionales a una consulta de relación Eloquent, puede acceder a la relación como si fuera una propiedad. Por ejemplo, al seguir usando nuestros modelos de ejemplo `User` y `Post`, podemos acceder a todas las publicaciones de un usuario de la siguiente manera:
> > If you do not need to add additional constraints to an Eloquent relationship query, you may access the relationship as if it were a property. For example, continuing to use our `User` and `Post` example models, we may access all of a user's posts like so:

    $user = App\User::find(1);

    foreach ($user->posts as $post) {
        //
    }

Las propiedades dinámicas son "carga lenta", lo que significa que solo cargarán sus datos de relación cuando realmente accedas a ellos. Debido a esto, los desarrolladores a menudo usan [carga ansiosa](#eager-loading) para precargar las relaciones que saben que se accederán después de cargar el modelo. La carga ansiosa proporciona una reducción significativa en las consultas SQL que se deben ejecutar para cargar las relaciones de un modelo.
> > Dynamic properties are "lazy loading", meaning they will only load their relationship data when you actually access them. Because of this, developers often use [eager loading](#eager-loading) to pre-load relationships they know will be accessed after loading the model. Eager loading provides a significant reduction in SQL queries that must be executed to load a model's relations.

<a name="querying-relationship-existence"></a>
### Consultando la existencia de la relación : Querying Relationship Existence

Al acceder a los registros de un modelo, puede limitar sus resultados en función de la existencia de una relación. Por ejemplo, imagine que desea recuperar todas las publicaciones de blog que tienen al menos un comentario. Para hacerlo, puede pasar el nombre de la relación a los métodos `has` y `orHas`:
> > When accessing the records for a model, you may wish to limit your results based on the existence of a relationship. For example, imagine you want to retrieve all blog posts that have at least one comment. To do so, you may pass the name of the relationship to the `has` and `orHas` methods:

    // Retrieve all posts that have at least one comment...
    $posts = App\Post::has('comments')->get();

También puede especificar un operador y contar para personalizar aún más la consulta:
> > You may also specify an operator and count to further customize the query:

    // Retrieve all posts that have three or more comments...
    $posts = App\Post::has('comments', '>=', 3)->get();

Las sentencias `has` anidadas también pueden construirse usando la notación "punto". Por ejemplo, puede recuperar todas las publicaciones que tienen al menos un comentario y voto:
> > Nested `has` statements may also be constructed using "dot" notation. For example, you may retrieve all posts that have at least one comment and vote:

    // Retrieve all posts that have at least one comment with votes...
    $posts = App\Post::has('comments.votes')->get();

Si necesita aún más potencia, puede usar los métodos `whereHas` y` orWhereHas` para poner condiciones de "dónde" en sus consultas `has`. Estos métodos le permiten agregar restricciones personalizadas a una restricción de relación, como verificar el contenido de un comentario:
> > If you need even more power, you may use the `whereHas` and `orWhereHas` methods to put "where" conditions on your `has` queries. These methods allow you to add customized constraints to a relationship constraint, such as checking the content of a comment:

    // Retrieve all posts with at least one comment containing words like foo%
    $posts = App\Post::whereHas('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

<a name="querying-relationship-absence"></a>
### Consultando ausencia de relación : Querying Relationship Absence

Al acceder a los registros de un modelo, puede limitar sus resultados según la ausencia de una relación. Por ejemplo, imagine que desea recuperar todas las publicaciones de blog **que no** tienen ningún comentario. Para hacerlo, puede pasar el nombre de la relación a los métodos `doesntHave` y `orDoesntHave`:
> > When accessing the records for a model, you may wish to limit your results based on the absence of a relationship. For example, imagine you want to retrieve all blog posts that **don't** have any comments. To do so, you may pass the name of the relationship to the `doesntHave` and `orDoesntHave` methods:

    $posts = App\Post::doesntHave('comments')->get();

Si necesita aún más potencia, puede usar los métodos `whereDoesntHave` y `orWhereDoesntHave` para poner condiciones de "dónde" en sus consultas `doesntHave`. Estos métodos le permiten agregar restricciones personalizadas a una restricción de relación, como verificar el contenido de un comentario:
> > If you need even more power, you may use the `whereDoesntHave` and `orWhereDoesntHave` methods to put "where" conditions on your `doesntHave` queries. These methods allows you to add customized constraints to a relationship constraint, such as checking the content of a comment:

    $posts = App\Post::whereDoesntHave('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

Puede usar la notación "punto" para ejecutar una consulta en una relación anidada. Por ejemplo, la siguiente consulta recuperará todas las publicaciones con comentarios de autores que no están prohibidos:
> > You may use "dot" notation to execute a query against a nested relationship. For example, the following query will retrieve all posts with comments from authors that are not banned:

    $posts = App\Post::whereDoesntHave('comments.author', function ($query) {
        $query->where('banned', 1);
    })->get();

<a name="counting-related-models"></a>
### Contando modelos relacionados : Counting Related Models

Si desea contar el número de resultados de una relación sin cargarlos en realidad, puede usar el método `withCount`, que colocará una columna `{relation}_count` en sus modelos resultantes. Por ejemplo:
> > If you want to count the number of results from a relationship without actually loading them you may use the `withCount` method, which will place a `{relation}_count` column on your resulting models. For example:

    $posts = App\Post::withCount('comments')->get();

    foreach ($posts as $post) {
        echo $post->comments_count;
    }

Puede agregar los "recuentos" para relaciones múltiples y agregar restricciones a las consultas:
> > You may add the "counts" for multiple relations as well as add constraints to the queries:

    $posts = App\Post::withCount(['votes', 'comments' => function ($query) {
        $query->where('content', 'like', 'foo%');
    }])->get();

    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;

También puede alias el resultado del conteo de relaciones, permitiendo múltiples conteos en la misma relación:
> > You may also alias the relationship count result, allowing multiple counts on the same relationship:

    $posts = App\Post::withCount([
        'comments',
        'comments as pending_comments_count' => function ($query) {
            $query->where('approved', false);
        }
    ])->get();

    echo $posts[0]->comments_count;

    echo $posts[0]->pending_comments_count;

<a name="eager-loading"></a>
## Eager Loading : Eager Loading

Al acceder a las relaciones Eloquent como propiedades, los datos de la relación se "cargan de forma diferida". Esto significa que los datos de la relación no se cargan realmente hasta que primero acceda a la propiedad. Sin embargo, Eloquent puede relaciones de "carga ansiosa" en el momento de consultar el modelo principal. La carga ansiosa alivia el problema de consulta N + 1. Para ilustrar el problema de consulta N + 1, considere un modelo `Book` que se relaciona con `Author`:
> > When accessing Eloquent relationships as properties, the relationship data is "lazy loaded". This means the relationship data is not actually loaded until you first access the property. However, Eloquent can "eager load" relationships at the time you query the parent model. Eager loading alleviates the N + 1 query problem. To illustrate the N + 1 query problem, consider a `Book` model that is related to `Author`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Book extends Model
    {
        /**
         * Get the author that wrote the book.
         */
        public function author()
        {
            return $this->belongsTo('App\Author');
        }
    }

Ahora, recuperemos todos los libros y sus autores:
> > Now, let's retrieve all books and their authors:

    $books = App\Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

Este ciclo ejecutará 1 consulta para recuperar todos los libros en la tabla, luego otra consulta para cada libro para recuperar el autor. Entonces, si tenemos 25 libros, este ciclo ejecutará 26 consultas: 1 para el libro original y 25 consultas adicionales para recuperar al autor de cada libro.
> > This loop will execute 1 query to retrieve all of the books on the table, then another query for each book to retrieve the author. So, if we have 25 books, this loop would run 26 queries: 1 for the original book, and 25 additional queries to retrieve the author of each book.

Afortunadamente, podemos utilizar la carga ansiosa para reducir esta operación a solo 2 consultas. Al realizar consultas, puede especificar qué relaciones se deben cargar con el método `with`:
> > Thankfully, we can use eager loading to reduce this operation to just 2 queries. When querying, you may specify which relationships should be eager loaded using the `with` method:

    $books = App\Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

Para esta operación, solo se ejecutarán dos consultas:
> > For this operation, only two queries will be executed:

    select * from books

    select * from authors where id in (1, 2, 3, 4, 5, ...)

#### Eager Loading Multiple Relationships

A veces puede necesitar cargar varias relaciones diferentes en una sola operación. Para hacerlo, simplemente pase argumentos adicionales al método `with`:
> > Sometimes you may need to eager load several different relationships in a single operation. To do so, just pass additional arguments to the `with` method:

    $books = App\Book::with(['author', 'publisher'])->get();

#### Nested Eager Loading

Para carga ansiosa de relaciones anidadas, puede usar sintaxis "punto". Por ejemplo, anhelemos cargar todos los autores del libro y todos los contactos personales del autor en una declaración Eloquent:
> > To eager load nested relationships, you may use "dot" syntax. For example, let's eager load all of the book's authors and all of the author's personal contacts in one Eloquent statement:

    $books = App\Book::with('author.contacts')->get();

#### Eager Loading Specific Columns

Es posible que no siempre necesite todas las columnas de las relaciones que está recuperando. Por esta razón, Eloquent le permite especificar qué columnas de la relación le gustaría recuperar:
> > You may not always need every column from the relationships you are retrieving. For this reason, Eloquent allows you to specify which columns of the relationship you would like to retrieve:

    $users = App\Book::with('author:id,name')->get();

> {note} Cuando use esta función, siempre debe incluir la columna `id` en la lista de columnas que desea recuperar.
> > > {note} When using this feature, you should always include the `id` column in the list of columns you wish to retrieve.

<a name="constraining-eager-loads"></a>
### Constraining Eager Loads

A veces puede desear cargar una relación, pero también especificar restricciones de consulta adicionales para la consulta de carga ansiosa. Aquí hay un ejemplo:
> > Sometimes you may wish to eager load a relationship, but also specify additional query constraints for the eager loading query. Here's an example:

    $users = App\User::with(['posts' => function ($query) {
        $query->where('title', 'like', '%first%');
    }])->get();

En este ejemplo, Eloquent solo estará ansioso por cargar publicaciones donde la columna `title` de la publicación contenga la palabra` first`. Por supuesto, puede llamar a otros métodos [generador de consultas] (/ docs / {{version}} / consultas) para personalizar aún más la operación de carga ansiosa:
> > In this example, Eloquent will only eager load posts where the post's `title` column contains the word `first`. Of course, you may call other [query builder](/docs/{{version}}/queries) methods to further customize the eager loading operation:

    $users = App\User::with(['posts' => function ($query) {
        $query->orderBy('created_at', 'desc');
    }])->get();

<a name="lazy-eager-loading"></a>
### Lazy Eager Loading

En ocasiones, es posible que tenga que cargar una relación después de que el modelo principal ya se haya recuperado. Por ejemplo, esto puede ser útil si necesita decidir dinámicamente si cargar modelos relacionados:
> > Sometimes you may need to eager load a relationship after the parent model has already been retrieved. For example, this may be useful if you need to dynamically decide whether to load related models:

    $books = App\Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

Si necesita establecer restricciones de consulta adicionales en la consulta de carga ansiosa, puede pasar una matriz marcada por las relaciones que desea cargar. Los valores del array deben ser instancias `Closure` que reciben la instancia de consulta:
> > If you need to set additional query constraints on the eager loading query, you may pass an array keyed by the relationships you wish to load. The array values should be `Closure` instances which receive the query instance:

    $books->load(['author' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }]);

Para cargar una relación solo cuando todavía no se ha cargado, use el método `loadMissing`:
> > To load a relationship only when it has not already been loaded, use the `loadMissing` method:

    public function format(Book $book)
    {
        $book->loadMissing('author');

        return [
            'name' => $book->name,
            'author' => $book->author->name
        ];
    }

<a name="inserting-and-updating-related-models"></a>
## Inserción y actualización de modelos relacionados : Inserting & Updating Related Models

<a name="the-save-method"></a>
### El método Save : The Save Method

Eloquent proporciona métodos convenientes para agregar nuevos modelos a las relaciones. Por ejemplo, quizás necesite insertar un nuevo `Comment` para un modelo `Post`. En lugar de configurar manualmente el atributo `post_id` en `Comment`, puede insertar el `Comment` directamente del método `save` de la relación:
> > Eloquent provides convenient methods for adding new models to relationships. For example, perhaps you need to insert a new `Comment` for a `Post` model. Instead of manually setting the `post_id` attribute on the `Comment`, you may insert the `Comment` directly from the relationship's `save` method:

    $comment = new App\Comment(['message' => 'A new comment.']);

    $post = App\Post::find(1);

    $post->comments()->save($comment);

Tenga en cuenta que no accedimos a la relación `comments` como una propiedad dinámica. En cambio, llamamos al método `comments` para obtener una instancia de la relación. El método `save` agregará automáticamente el valor `post_id` apropiado al nuevo modelo `Comment`.
> > Notice that we did not access the `comments` relationship as a dynamic property. Instead, we called the `comments` method to obtain an instance of the relationship. The `save` method will automatically add the appropriate `post_id` value to the new `Comment` model.

Si necesita guardar varios modelos relacionados, puede usar el método `saveMany`:
> > If you need to save multiple related models, you may use the `saveMany` method:

    $post = App\Post::find(1);

    $post->comments()->saveMany([
        new App\Comment(['message' => 'A new comment.']),
        new App\Comment(['message' => 'Another comment.']),
    ]);

<a name="the-create-method"></a>
### El método Create : The Create Method

Además de los métodos `save` y `saveMany`, también puede usar el método `create`, que acepta un array de atributos, crea un modelo y lo inserta en la base de datos. Nuevamente, la diferencia entre `save` y `create` es que `save` acepta una instancia de modelo Eloquent completa, mientras que `create` acepta un `array` simple de PHP:
> > In addition to the `save` and `saveMany` methods, you may also use the `create` method, which accepts an array of attributes, creates a model, and inserts it into the database. Again, the difference between `save` and `create` is that `save` accepts a full Eloquent model instance while `create` accepts a plain PHP `array`:

    $post = App\Post::find(1);

    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

> {tip} Antes de usar el método `create`, asegúrese de revisar la documentación en el atributo [asignación masiva](/docs/{{version}}/eloquent#mass-assignment).
> > > {tip} Before using the `create` method, be sure to review the documentation on attribute [mass assignment](/docs/{{version}}/eloquent#mass-assignment).

Puede usar el método `createMany` para crear múltiples modelos relacionados:
> > You may use the `createMany` method to create multiple related models:

    $post = App\Post::find(1);

    $post->comments()->createMany([
        [
            'message' => 'A new comment.',
        ],
        [
            'message' => 'Another new comment.',
        ],
    ]);

<a name="updating-belongs-to-relationships"></a>
### Pertenece a las relaciones : Belongs To Relationships

Al actualizar una relación `belongsTo`, puede usar el método `associate`. Este método establecerá la clave externa en el modelo hijo:
> > When updating a `belongsTo` relationship, you may use the `associate` method. This method will set the foreign key on the child model:

    $account = App\Account::find(10);

    $user->account()->associate($account);

    $user->save();

Al eliminar una relación `belongsTo`, puede usar el método `dissociate`. Este método establecerá la clave externa de la relación en `null`:
> > When removing a `belongsTo` relationship, you may use the `dissociate` method. This method will set the relationship's foreign key to `null`:

    $user->account()->dissociate();

    $user->save();

<a name="default-models"></a>
#### Modelos predeterminados : Default Models

La relación `belongsTo` le permite definir un modelo predeterminado que se devolverá si la relación dada es `null`. Este patrón a menudo se conoce como [Patrón de objeto nulo](https://en.wikipedia.org/wiki/Null_Object_pattern) y puede ayudar a eliminar las verificaciones condicionales en su código. En el siguiente ejemplo, la relación `user` devolverá un modelo `App\User` vacío si no se adjunta `user` a la publicación:
> > The `belongsTo` relationship allows you to define a default model that will be returned if the given relationship is `null`. This pattern is often referred to as the [Null Object pattern](https://en.wikipedia.org/wiki/Null_Object_pattern) and can help remove conditional checks in your code. In the following example, the `user` relation will return an empty `App\User` model if no `user` is attached to the post:

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault();
    }

Para completar el modelo predeterminado con atributos, puede pasar una matriz o cierre al método `withDefault`:
> > To populate the default model with attributes, you may pass an array or Closure to the `withDefault` method:

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault([
            'name' => 'Guest Author',
        ]);
    }

    /**
     * Get the author of the post.
     */
    public function user()
    {
        return $this->belongsTo('App\User')->withDefault(function ($user) {
            $user->name = 'Guest Author';
        });
    }

<a name="updating-many-to-many-relationships"></a>
### Relaciones de muchos a muchos : Many To Many Relationships

#### Adjuntar / Separar : Attaching / Detaching

Eloquent también proporciona algunos métodos auxiliares adicionales para hacer que trabajar con modelos relacionados sea más conveniente. Por ejemplo, imaginemos que un usuario puede tener muchos roles y un rol puede tener muchos usuarios. Para adjuntar un rol a un usuario al insertar un registro en la tabla intermedia que une los modelos, use el método `attach`:
> > Eloquent also provides a few additional helper methods to make working with related models more convenient. For example, let's imagine a user can have many roles and a role can have many users. To attach a role to a user by inserting a record in the intermediate table that joins the models, use the `attach` method:

    $user = App\User::find(1);

    $user->roles()->attach($roleId);

Al asociar una relación a un modelo, también puede pasar una matriz de datos adicionales para insertar en la tabla intermedia:
> > When attaching a relationship to a model, you may also pass an array of additional data to be inserted into the intermediate table:

    $user->roles()->attach($roleId, ['expires' => $expires]);

Por supuesto, a veces puede ser necesario eliminar un rol de un usuario. Para eliminar un registro de relación de muchos a muchos, use el método `detach`. El método `detach` eliminará el registro apropiado de la tabla intermedia; sin embargo, ambos modelos permanecerán en la base de datos:
> > Of course, sometimes it may be necessary to remove a role from a user. To remove a many-to-many relationship record, use the `detach` method. The `detach` method will remove the appropriate record out of the intermediate table; however, both models will remain in the database:

    // Detach a single role from the user...
    $user->roles()->detach($roleId);

    // Detach all roles from the user...
    $user->roles()->detach();

Por convención, `attach` y` detach` también aceptan arrays de IDs como entrada:
> > For convenience, `attach` and `detach` also accept arrays of IDs as input:

    $user = App\User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([
        1 => ['expires' => $expires],
        2 => ['expires' => $expires]
    ]);

#### Sincronización de asociaciones : Syncing Associations

También puede usar el método `sync` para construir asociaciones de muchos a muchos. El método `sync` acepta un array de IDs para colocar en la tabla intermedia. Cualquier ID que no esté en el array dado se eliminará de la tabla intermedia. Entonces, después de completar esta operación, solo los IDs en la matriz dada existirán en la tabla intermedia:
> > You may also use the `sync` method to construct many-to-many associations. The `sync` method accepts an array of IDs to place on the intermediate table. Any IDs that are not in the given array will be removed from the intermediate table. So, after this operation is complete, only the IDs in the given array will exist in the intermediate table:

    $user->roles()->sync([1, 2, 3]);

También puede pasar valores de tabla intermedia adicionales con los IDs:
> > You may also pass additional intermediate table values with the IDs:

    $user->roles()->sync([1 => ['expires' => true], 2, 3]);

Si no desea separar los ID existentes, puede usar el método `syncWithoutDetaching`:
> > If you do not want to detach existing IDs, you may use the `syncWithoutDetaching` method:

    $user->roles()->syncWithoutDetaching([1, 2, 3]);

#### Alternar asociaciones : Toggling Associations

La relación muchos a muchos también proporciona un método de alternancia que "alterna" el estado del archivo adjunto de los IDs dados. Si la identificación dada está actualmente adjunta, se desconectará. Del mismo modo, si está actualmente separado, se adjuntará:
> > The many-to-many relationship also provides a `toggle` method which "toggles" the attachment status of the given IDs. If the given ID is currently attached, it will be detached. Likewise, if it is currently detached, it will be attached:

    $user->roles()->toggle([1, 2, 3]);

#### Guardar datos adicionales en una tabla pivote : Saving Additional Data On A Pivot Table

Al trabajar con una relación de muchos a muchos, el método `save` acepta una matriz de atributos de tablas intermedias adicionales como su segundo argumento:
> > When working with a many-to-many relationship, the `save` method accepts an array of additional intermediate table attributes as its second argument:

    App\User::find(1)->roles()->save($role, ['expires' => $expires]);

#### Actualización de un registro en una tabla pivote : Updating A Record On A Pivot Table

Si necesita actualizar una fila existente en su tabla pivote, puede usar el método `updateExistingPivot`. Este método acepta la clave foránea de registro dinámico y un array de atributos para actualizar:
> > If you need to update an existing row in your pivot table, you may use `updateExistingPivot` method. This method accepts the pivot record foreign key and an array of attributes to update:

    $user = App\User::find(1);

    $user->roles()->updateExistingPivot($roleId, $attributes);

<a name="touching-parent-timestamps"></a>
## Touching Parent Timestamps

Cuando un modelo `belongsTo` o `belongsToMany` otro modelo, como `Comment` que pertenece a un `Post`, a veces es útil actualizar la marca de tiempo del padre cuando se actualiza el modelo hijo. Por ejemplo, cuando se actualiza un modelo `Comment`, es posible que desee "tocar" automáticamente la marca de tiempo `updated_at` del `Post` propietario. Eloquent lo hace fácil. Simplemente agregue una propiedad `touches` que contenga los nombres de las relaciones con el modelo hijo:
> > When a model `belongsTo` or `belongsToMany` another model, such as a `Comment` which belongs to a `Post`, it is sometimes helpful to update the parent's timestamp when the child model is updated. For example, when a `Comment` model is updated, you may want to automatically "touch" the `updated_at` timestamp of the owning `Post`. Eloquent makes it easy. Just add a `touches` property containing the names of the relationships to the child model:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * All of the relationships to be touched.
         *
         * @var array
         */
        protected $touches = ['post'];

        /**
         * Get the post that the comment belongs to.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

Ahora, cuando actualice un `Comment`, también se actualizará la columna `updated_at` de su propietario, haciendo que sea más conveniente saber cuándo invalidar un caché del modelo `Post`:
> > Now, when you update a `Comment`, the owning `Post` will have its `updated_at` column updated as well, making it more convenient to know when to invalidate a cache of the `Post` model:

    $comment = App\Comment::find(1);

    $comment->text = 'Edit to this comment!';

    $comment->save();
