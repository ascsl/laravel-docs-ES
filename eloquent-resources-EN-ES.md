# Eloquent: recursos de la API : Eloquent: API Resources

- [Introduction](#introduction)
- [Generating Resources](#generating-resources)
- [Concept Overview](#concept-overview)
- [Writing Resources](#writing-resources)
    - [Data Wrapping](#data-wrapping)
    - [Pagination](#pagination)
    - [Conditional Attributes](#conditional-attributes)
    - [Conditional Relationships](#conditional-relationships)
    - [Adding Meta Data](#adding-meta-data)
- [Resource Responses](#resource-responses)

<a name="introduction"></a>
## Introducción : Introduction

Al crear una API, es posible que necesite una capa de transformación entre sus modelos Eloquent y las respuestas JSON que realmente se devuelven a los usuarios de la aplicación. Las clases de recursos de Laravel le permiten transformar de manera expresa y fácil sus modelos y colecciones de modelos en JSON.
> > When building an API, you may need a transformation layer that sits between your Eloquent models and the JSON responses that are actually returned to your application's users. Laravel's resource classes allow you to expressively and easily transform your models and model collections into JSON.

<a name="generating-resources"></a>
## Generando Recursos : Generating Resources

Para generar una clase de recurso, puede usar el comando Artisan `make:resource`. Por defecto, los recursos se colocarán en el directorio `app/Http/Resources` de su aplicación. Los recursos amplían la clase `Illuminate\Http\Resources\Json\JsonResource`:
> > To generate a resource class, you may use the `make:resource` Artisan command. By default, resources will be placed in the `app/Http/Resources` directory of your application. Resources extend the `Illuminate\Http\Resources\Json\JsonResource` class:

    php artisan make:resource User

#### Colecciones de recursos : Resource Collections

Además de generar recursos que transforman modelos individuales, puede generar recursos que sean responsables de transformar colecciones de modelos. Esto permite que su respuesta incluya enlaces y otra metainformación que sea relevante para una colección completa de un recurso dado.
> > In addition to generating resources that transform individual models, you may generate resources that are responsible for transforming collections of models. This allows your response to include links and other meta information that is relevant to an entire collection of a given resource.

Para crear una colección de recursos, debe usar el indicador `--collection` al crear el recurso. O bien, incluir la palabra `Collection` en el nombre del recurso indicará a Laravel que debe crear un recurso de colección. Los recursos de la colección extienden la clase `Illuminate\Http\Resources\Json\ResourceCollection`:
> > To create a resource collection, you should use the `--collection` flag when creating the resource. Or, including the word `Collection` in the resource name will indicate to Laravel that it should create a collection resource. Collection resources extend the `Illuminate\Http\Resources\Json\ResourceCollection` class:

    php artisan make:resource Users --collection

    php artisan make:resource UserCollection

<a name="concept-overview"></a>
## Concepto general : Concept Overview

> {tip} Esta es una descripción general de alto nivel de recursos y colecciones de recursos. Le recomendamos que lea las otras secciones de esta documentación para obtener una comprensión más profunda de la personalización y la potencia que le ofrecen los recursos.
> > > {tip} This is a high-level overview of resources and resource collections. You are highly encouraged to read the other sections of this documentation to gain a deeper understanding of the customization and power offered to you by resources.

Antes de sumergirse en todas las opciones disponibles para usted al escribir recursos, primero observemos en detalle cómo se usan los recursos dentro de Laravel. Una clase de recurso representa un modelo único que debe transformarse en una estructura JSON. Por ejemplo, aquí hay una clase simple de recursos de `User`:
> > Before diving into all of the options available to you when writing resources, let's first take a high-level look at how resources are used within Laravel. A resource class represents a single model that needs to be transformed into a JSON structure. For example, here is a simple `User` resource class:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class User extends JsonResource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

Cada clase de recurso define un método `toArray` que devuelve la matriz de atributos que se deben convertir a JSON al enviar la respuesta. Tenga en cuenta que podemos acceder a las propiedades del modelo directamente desde la variable `$this`. Esto se debe a que una clase de recurso sustituirá automáticamente el acceso a propiedades y métodos por el modelo subyacente para un acceso conveniente. Una vez que se define el recurso, puede devolverse desde una ruta o controlador:
> > Every resource class defines a `toArray` method which returns the array of attributes that should be converted to JSON when sending the response. Notice that we can access model properties directly from the `$this` variable. This is because a resource class will automatically proxy property and method access down to the underlying model for convenient access. Once the resource is defined, it may be returned from a route or controller:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

### Colecciones de recursos : Resource Collections

Si está devolviendo una colección de recursos o una respuesta paginada, puede usar el método `collection` al crear la instancia de recurso en su ruta o controlador:
> > If you are returning a collection of resources or a paginated response, you may use the `collection` method when creating the resource instance in your route or controller:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

Por supuesto, esto no permite ninguna adición de metadatos que pueda ser necesario devolver con la colección. Si desea personalizar la respuesta de recopilación de recursos, puede crear un recurso dedicado para representar la colección:
> > Of course, this does not allow any addition of meta data that may need to be returned with the collection. If you would like to customize the resource collection response, you may create a dedicated resource to represent the collection:

    php artisan make:resource UserCollection

Una vez que se haya generado la clase de recopilación de recursos, puede definir fácilmente cualquier metadato que deba incluirse con la respuesta:
> > Once the resource collection class has been generated, you may easily define any meta data that should be included with the response:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

Después de definir su colección de recursos, se puede devolver desde una ruta o controlador:
> > After defining your resource collection, it may be returned from a route or controller:

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="writing-resources"></a>
## Escritura de recursos : Writing Resources

> {tip} Si no ha leído la [descripción general del concepto](#concept-overview), le recomendamos que lo haga antes de continuar con esta documentación.
> > > {tip} If you have not read the [concept overview](#concept-overview), you are highly encouraged to do so before proceeding with this documentation.

En esencia, los recursos son simples. Solo necesitan transformar un modelo dado en un array. Por lo tanto, cada recurso contiene un método `toArray` que traduce los atributos de su modelo en un array compatible con API que puede devolverse a sus usuarios:
> > In essence, resources are simple. They only need to transform a given model into an array. So, each resource contains a `toArray` method which translates your model's attributes into an API friendly array that can be returned to your users:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class User extends JsonResource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
                'name' => $this->name,
                'email' => $this->email,
                'created_at' => $this->created_at,
                'updated_at' => $this->updated_at,
            ];
        }
    }

Una vez que se ha definido un recurso, se puede devolver directamente desde una ruta o controlador:
> > Once a resource has been defined, it may be returned directly from a route or controller:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

#### Relaciones : Relationships

Si desea incluir recursos relacionados en su respuesta, puede agregarlos a la matriz devuelta por su método `toArray`. En este ejemplo, usaremos el método `collection` del recurso `Post` para agregar las publicaciones de blog del usuario a la respuesta del recurso:
> > If you would like to include related resources in your response, you may add them to the array returned by your `toArray` method. In this example, we will use the `Post` resource's `collection` method to add the user's blog posts to the resource response:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => PostResource::collection($this->posts),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

> {tip} Si desea incluir relaciones solo cuando ya se han cargado, consulte la documentación sobre [relaciones condicionales](#conditional-relationships).
> > > {tip} If you would like to include relationships only when they have already been loaded, check out the documentation on [conditional relationships](#conditional-relationships).

#### Colecciones de recursos : Resource Collections

Si bien los recursos traducen un único modelo en un array, las colecciones de recursos traducen una colección de modelos en un array. No es absolutamente necesario definir una clase de colección de recursos para cada uno de los tipos de modelo, ya que todos los recursos proporcionan un método `collection` para generar una colección de recursos "ad-hoc" sobre la marcha:
> > While resources translate a single model into an array, resource collections translate a collection of models into an array. It is not absolutely necessary to define a resource collection class for each one of your model types since all resources provide a `collection` method to generate an "ad-hoc" resource collection on the fly:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

Sin embargo, si necesita personalizar los metadatos devueltos con la colección, será necesario definir una colección de recursos:
> > However, if you need to customize the meta data returned with the collection, it will be necessary to define a resource collection:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

Al igual que los recursos singulares, las colecciones de recursos pueden devolverse directamente desde rutas o controladores:
> > Like singular resources, resource collections may be returned directly from routes or controllers:

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::all());
    });

<a name="data-wrapping"></a>
### Envoltura de datos : Data Wrapping

De forma predeterminada, su recurso externo está envuelto en una clave `data` cuando la respuesta del recurso se convierte a JSON. Entonces, por ejemplo, una respuesta típica de recolección de recursos se ve así:
> > By default, your outer-most resource is wrapped in a `data` key when the resource response is converted to JSON. So, for example, a typical resource collection response looks like the following:

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ]
    }

Si desea deshabilitar el ajuste del recurso externo, puede usar el método `withoutWrapping` en la clase de recurso base. Normalmente, debe llamar a este método desde su `AppServiceProvider` u otro [proveedor de servicios](/docs/{{version}}/providers) que se carga en cada solicitud a su aplicación:
> > If you would like to disable the wrapping of the outer-most resource, you may use the `withoutWrapping` method on the base resource class. Typically, you should call this method from your `AppServiceProvider` or another [service provider](/docs/{{version}}/providers) that is loaded on every request to your application:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Http\Resources\Json\Resource;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Resource::withoutWrapping();
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

> {note} El método `withoutWrapping` solo afecta la respuesta externa y no eliminará las claves `data` que usted agrega manualmente a sus propias colecciones de recursos.
> > > {note} The `withoutWrapping` method only affects the outer-most response and will not remove `data` keys that you manually add to your own resource collections.

### Envoltura de recursos anidados : Wrapping Nested Resources

Tienes libertad total para determinar cómo se envuelven las relaciones de tus recursos. Si desea que todas las colecciones de recursos se envuelvan en una clave `data`, independientemente de su anidación, debe definir una clase de colección de recursos para cada recurso y devolver la colección dentro de una clave `data`.
> > You have total freedom to determine how your resource's relationships are wrapped. If you would like all resource collections to be wrapped in a `data` key, regardless of their nesting, you should define a resource collection class for each resource and return the collection within a `data` key.

Por supuesto, es posible que se pregunte si esto causará que su recurso externo quede envuelto en dos claves de `data`. No se preocupe, Laravel nunca permitirá que sus recursos se doblen accidentalmente, por lo que no tiene que preocuparse por el nivel de anidación de la colección de recursos que está transformando:
> > Of course, you may be wondering if this will cause your outer-most resource to be wrapped in two `data` keys. Don't worry, Laravel will never let your resources be accidentally double-wrapped, so you don't have to be concerned about the nesting level of the resource collection you are transforming:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class CommentsCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return ['data' => $this->collection];
        }
    }

### Envoltura de datos y paginación : Data Wrapping And Pagination

Cuando devuelva colecciones paginadas en una respuesta de recursos, Laravel ajustará los datos de sus recursos en una clave `data` incluso si se ha llamado al método `withoutWrapping`. Esto se debe a que las respuestas paginadas siempre contienen claves `meta` y `links` con información sobre el estado del paginador:
> > When returning paginated collections in a resource response, Laravel will wrap your resource data in a `data` key even if the `withoutWrapping` method has been called. This is because paginated responses always contain `meta` and `links` keys with information about the paginator's state:

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }

<a name="pagination"></a>
### Paginación : Pagination

Siempre puede pasar una instancia de paginador al método `collection` de un recurso o a una colección de recursos personalizada:
> > You may always pass a paginator instance to the `collection` method of a resource or to a custom resource collection:

    use App\User;
    use App\Http\Resources\UserCollection;

    Route::get('/users', function () {
        return new UserCollection(User::paginate());
    });

Las respuestas paginadas siempre contienen claves `meta` y` links` con información sobre el estado del paginador:
> > Paginated responses always contain `meta` and `links` keys with information about the paginator's state:

    {
        "data": [
            {
                "id": 1,
                "name": "Eladio Schroeder Sr.",
                "email": "therese28@example.com",
            },
            {
                "id": 2,
                "name": "Liliana Mayert",
                "email": "evandervort@example.com",
            }
        ],
        "links":{
            "first": "http://example.com/pagination?page=1",
            "last": "http://example.com/pagination?page=1",
            "prev": null,
            "next": null
        },
        "meta":{
            "current_page": 1,
            "from": 1,
            "last_page": 1,
            "path": "http://example.com/pagination",
            "per_page": 15,
            "to": 10,
            "total": 10
        }
    }

<a name="conditional-attributes"></a>
### Atributos condicionales : Conditional Attributes

A veces puede desear incluir solo un atributo en una respuesta de recurso si se cumple una condición dada. Por ejemplo, puede incluir solo un valor si el usuario actual es un "administrador". Laravel proporciona una variedad de métodos de ayuda para ayudarlo en esta situación. El método `when` se puede usar para agregar condicionalmente un atributo a una respuesta de recursos:
> > Sometimes you may wish to only include an attribute in a resource response if a given condition is met. For example, you may wish to only include a value if the current user is an "administrator". Laravel provides a variety of helper methods to assist you in this situation. The `when` method may be used to conditionally add an attribute to a resource response:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'secret' => $this->when(Auth::user()->isAdmin(), 'secret-value'),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

En este ejemplo, la clave `secret` solo se devolverá en la respuesta de recurso final si el método `isAdmin` del usuario autenticado devuelve `true`. Si el método devuelve `false`, la clave `secret` se eliminará por completo de la respuesta al recurso antes de enviarla al cliente. El método `when` le permite definir expresivamente sus recursos sin recurrir a declaraciones condicionales al construir el conjunto.
> > In this example, the `secret` key will only be returned in the final resource response if the authenticated user's `isAdmin` method returns `true`. If the method returns `false`, the `secret` key will be removed from the resource response entirely before it is sent back to the client. The `when` method allows you to expressively define your resources without resorting to conditional statements when building the array.

El método `when` también acepta un Closure como segundo argumento, lo que le permite calcular el valor resultante solo si la condición dada es `true`:
> > The `when` method also accepts a Closure as its second argument, allowing you to calculate the resulting value only if the given condition is `true`:

    'secret' => $this->when(Auth::user()->isAdmin(), function () {
        return 'secret-value';
    }),

#### Fusionar atributos condicionales : Merging Conditional Attributes

En ocasiones, puede tener varios atributos que solo deberían incluirse en la respuesta del recurso en función de la misma condición. En este caso, puede usar el método `mergeWhen` para incluir los atributos en la respuesta solo cuando la condición dada sea `true`:
> > Sometimes you may have several attributes that should only be included in the resource response based on the same condition. In this case, you may use the `mergeWhen` method to include the attributes in the response only when the given condition is `true`:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            $this->mergeWhen(Auth::user()->isAdmin(), [
                'first-secret' => 'value',
                'second-secret' => 'value',
            ]),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

De nuevo, si la condición dada es `false`, estos atributos se eliminarán completamente de la respuesta al recurso antes de enviarse al cliente.
> > Again, if the given condition is `false`, these attributes will be removed from the resource response entirely before it is sent to the client.

> {note} El método `mergeWhen` no debe usarse en matrices que mezclan cadenas y teclas numéricas. Además, no se debe usar dentro de matrices con claves numéricas que no están ordenadas secuencialmente.
> > > {note} The `mergeWhen` method should not be used within arrays that mix string and numeric keys. Furthermore, it should not be used within arrays with numeric keys that are not ordered sequentially.

<a name="conditional-relationships"></a>
### Relaciones condicionales : Conditional Relationships

Además de cargar los atributos condicionalmente, puede incluir de forma condicional las relaciones en sus respuestas de recursos en función de si la relación ya se ha cargado en el modelo. Esto le permite a su controlador decidir qué relaciones se deben cargar en el modelo y su recurso puede incluirlas fácilmente solo cuando se hayan cargado realmente.
> > In addition to conditionally loading attributes, you may conditionally include relationships on your resource responses based on if the relationship has already been loaded on the model. This allows your controller to decide which relationships should be loaded on the model and your resource can easily include them only when they have actually been loaded.

En última instancia, esto hace que sea más fácil evitar problemas de consulta "N + 1" dentro de sus recursos. El método `whenLoaded` se puede usar para cargar una relación de manera condicional. Para evitar cargar innecesariamente las relaciones, este método acepta el nombre de la relación en lugar de la relación misma:
> > Ultimately, this makes it easier to avoid "N+1" query problems within your resources. The `whenLoaded` method may be used to conditionally load a relationship. In order to avoid unnecessarily loading relationships, this method accepts the name of the relationship instead of the relationship itself:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'posts' => PostResource::collection($this->whenLoaded('posts')),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

En este ejemplo, si la relación no se ha cargado, la clave `posts` se eliminará completamente de la respuesta al recurso antes de que se envíe al cliente.
> > In this example, if the relationship has not been loaded, the `posts` key will be removed from the resource response entirely before it is sent to the client.

#### Información de pivote condicional : Conditional Pivot Information

Además de incluir condicionalmente información de relación en sus respuestas de recursos, puede incluir de forma condicional los datos de las tablas intermedias de las relaciones de muchos a muchos utilizando el método `whenPivotLoaded`. El método `whenPivotLoaded` acepta el nombre de la tabla dinámica como primer argumento. El segundo argumento debe ser un Cierre que defina el valor que se devolverá si la información de pivote está disponible en el modelo:
> > In addition to conditionally including relationship information in your resource responses, you may conditionally include data from the intermediate tables of many-to-many relationships using the `whenPivotLoaded` method. The `whenPivotLoaded` method accepts the name of the pivot table as its first argument. The second argument should be a Closure that defines the value to be returned if the pivot information is available on the model:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'expires_at' => $this->whenPivotLoaded('role_users', function () {
                return $this->pivot->expires_at;
            }),
        ];
    }

<a name="adding-meta-data"></a>
### Agregar metadatos : Adding Meta Data

Algunos estándares de la API JSON requieren la adición de metadatos a sus respuestas de recursos y colecciones de recursos. Esto a menudo incluye cosas como `links` al recurso o recursos relacionados, o metadatos sobre el recurso en sí. Si necesita devolver metadatos adicionales sobre un recurso, inclúyalo en su método `toArray`. Por ejemplo, puede incluir información de `enlace` cuando se transforma una colección de recursos:
> > Some JSON API standards require the addition of meta data to your resource and resource collections responses. This often includes things like `links` to the resource or related resources, or meta data about the resource itself. If you need to return additional meta data about a resource, include it in your `toArray` method. For example, you might include `link` information when transforming a resource collection:

    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }

Al devolver metadatos adicionales de sus recursos, nunca tendrá que preocuparse por anular accidentalmente las teclas `links` o` meta` que Laravel agrega automáticamente cuando devuelve respuestas paginadas. Cualquier `links` adicional que defina se fusionará con los enlaces provistos por el paginador.
> > When returning additional meta data from your resources, you never have to worry about accidentally overriding the `links` or `meta` keys that are automatically added by Laravel when returning paginated responses. Any additional `links` you define will be merged with the links provided by the paginator.

#### Metadatos de nivel superior : Top Level Meta Data

En ocasiones, es posible que desee incluir ciertos metadatos con una respuesta de recursos si el recurso es el recurso externo que se devuelve. Típicamente, esto incluye metainformación sobre la respuesta como un todo. Para definir estos metadatos, agregue un método `with` a su clase de recurso. Este método debe devolver un array de metadatos para incluir con la respuesta de recursos solo cuando el recurso es el recurso más externo que se está representando:
> > Sometimes you may wish to only include certain meta data with a resource response if the resource is the outer-most resource being returned. Typically, this includes meta information about the response as a whole. To define this meta data, add a `with` method to your resource class. This method should return an array of meta data to be included with the resource response only when the resource is the outer-most resource being rendered:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * Transform the resource collection into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return parent::toArray($request);
        }

        /**
         * Get additional data that should be returned with the resource array.
         *
         * @param \Illuminate\Http\Request  $request
         * @return array
         */
        public function with($request)
        {
            return [
                'meta' => [
                    'key' => 'value',
                ],
            ];
        }
    }

#### Agregar metadatos al construir recursos : Adding Meta Data When Constructing Resources

También puede agregar datos de nivel superior al construir instancias de recursos en su ruta o controlador. El método `additional`, que está disponible en todos los recursos, acepta un array de datos que se debe agregar a la respuesta del recurso:
> > You may also add top-level data when constructing resource instances in your route or controller. The `additional` method, which is available on all resources, accepts an array of data that should be added to the resource response:

    return (new UserCollection(User::all()->load('roles')))
                    ->additional(['meta' => [
                        'key' => 'value',
                    ]]);

<a name="resource-responses"></a>
## Respuestas de recursos : Resource Responses

Como ya ha leído, los recursos pueden ser devueltos directamente desde rutas y controladores:
> > As you have already read, resources may be returned directly from routes and controllers:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return new UserResource(User::find(1));
    });

Sin embargo, a veces puede necesitar personalizar la respuesta HTTP saliente antes de enviarla al cliente. Hay dos maneras de lograr esto. Primero, puedes encadenar el método de `response` al recurso. Este método devolverá una instancia `Illuminate\Http\Response`, lo que le permitirá un control total de los encabezados de la respuesta:
> > However, sometimes you may need to customize the outgoing HTTP response before it is sent to the client. There are two ways to accomplish this. First, you may chain the `response` method onto the resource. This method will return an `Illuminate\Http\Response` instance, allowing you full control of the response's headers:

    use App\User;
    use App\Http\Resources\User as UserResource;

    Route::get('/user', function () {
        return (new UserResource(User::find(1)))
                    ->response()
                    ->header('X-Value', 'True');
    });

Alternativamente, puede definir un método `withResponse` dentro del propio recurso. Se llamará a este método cuando el recurso se devuelva como el recurso externo en una respuesta:
> > Alternatively, you may define a `withResponse` method within the resource itself. This method will be called when the resource is returned as the outer-most resource in a response:

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\JsonResource;

    class User extends JsonResource
    {
        /**
         * Transform the resource into an array.
         *
         * @param  \Illuminate\Http\Request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'id' => $this->id,
            ];
        }

        /**
         * Customize the outgoing response for the resource.
         *
         * @param  \Illuminate\Http\Request
         * @param  \Illuminate\Http\Response
         * @return void
         */
        public function withResponse($request, $response)
        {
            $response->header('X-Value', 'True');
        }
    }
