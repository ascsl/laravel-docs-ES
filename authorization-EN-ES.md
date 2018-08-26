# Autorización
# Authorization

- [Introduction](#introduction)
- [Gates](#gates)
    - [Writing Gates](#writing-gates)
    - [Authorizing Actions](#authorizing-actions-via-gates)
    - [Intercepting Gate Checks](#intercepting-gate-checks)
- [Creating Policies](#creating-policies)
    - [Generating Policies](#generating-policies)
    - [Registering Policies](#registering-policies)
- [Writing Policies](#writing-policies)
    - [Policy Methods](#policy-methods)
    - [Methods Without Models](#methods-without-models)
    - [Guest Users](#guest-users)
    - [Policy Filters](#policy-filters)
- [Authorizing Actions Using Policies](#authorizing-actions-using-policies)
    - [Via The User Model](#via-the-user-model)
    - [Via Middleware](#via-middleware)
    - [Via Controller Helpers](#via-controller-helpers)
    - [Via Blade Templates](#via-blade-templates)

<a name="introduction"></a>
## Introducción
## Introduction

Además de proporcionar servicios de [autenticación](/docs/{{version}}/authentication) de fábrica, Laravel también proporciona una forma simple de autorizar acciones del usuario contra un recurso dado. Al igual que la autenticación, el enfoque de la autorización de Laravel es simple y existen dos formas principales de autorizar acciones: puertas y políticas.
> > In addition to providing [authentication](/docs/{{version}}/authentication) services out of the box, Laravel also provides a simple way to authorize user actions against a given resource. Like authentication, Laravel's approach to authorization is simple, and there are two primary ways of authorizing actions: gates and policies.

Piense en puertas y políticas como rutas y controladores. Gates proporciona un enfoque simple basado en el cierre para la autorización, mientras que las políticas, como los controladores, agrupan su lógica en torno a un modelo o recurso en particular. Primero exploraremos las puertas y luego examinaremos las políticas.
> > Think of gates and policies like routes and controllers. Gates provide a simple, Closure based approach to authorization while policies, like controllers, group their logic around a particular model or resource. We'll explore gates first and then examine policies.

No es necesario elegir entre utilizar puertas de forma exclusiva o utilizar políticas exclusivamente al crear una aplicación. La mayoría de las aplicaciones probablemente contengan una combinación de puertas y políticas, ¡y eso está perfectamente bien! Las puertas son más aplicables a acciones que no están relacionadas con ningún modelo o recurso, como ver un tablero de administrador. Por el contrario, las políticas deben usarse cuando desee autorizar una acción para un modelo o recurso en particular.
> > You do not need to choose between exclusively using gates or exclusively using policies when building an application. Most applications will most likely contain a mixture of gates and policies, and that is perfectly fine! Gates are most applicable to actions which are not related to any model or resource, such as viewing an administrator dashboard. In contrast, policies should be used when you wish to authorize an action for a particular model or resource.

<a name="gates"></a>
## Puertas
## Gates

<a name="writing-gates"></a>
### Escritura de puertas
### Writing Gates

Las puertas son cierres que determinan si un usuario está autorizado a realizar una acción determinada y, por lo general, se definen en la clase `App\Providers\AuthServiceProvider` utilizando la fachada `Gate`. Gates siempre recibe una instancia de usuario como su primer argumento, y opcionalmente puede recibir argumentos adicionales, como un modelo de Eloquent relevante:
> > Gates are Closures that determine if a user is authorized to perform a given action and are typically defined in the `App\Providers\AuthServiceProvider` class using the `Gate` facade. Gates always receive a user instance as their first argument, and may optionally receive additional arguments such as a relevant Eloquent model:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', function ($user, $post) {
            return $user->id == $post->user_id;
        });
    }

Las puertas también se pueden definir utilizando una cadena de devolución de llamada de estilo `Class@method`, como controladores:
> > Gates may also be defined using a `Class@method` style callback string, like controllers:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', 'App\Policies\PostPolicy@update');
    }

#### Puertas de recursos
#### Resource Gates

También puedes definir múltiples habilidades de Gate a la vez usando el método `resource`:
> > You may also define multiple Gate abilities at once using the `resource` method:

    Gate::resource('posts', 'App\Policies\PostPolicy');

Esto es idéntico a la definición manual de las siguientes definiciones de puerta:
> > This is identical to manually defining the following Gate definitions:

    Gate::define('posts.view', 'App\Policies\PostPolicy@view');
    Gate::define('posts.create', 'App\Policies\PostPolicy@create');
    Gate::define('posts.update', 'App\Policies\PostPolicy@update');
    Gate::define('posts.delete', 'App\Policies\PostPolicy@delete');

Por defecto, se definirán las habilidades `view`, `create`, `update`, y `delete`. Puede anular las habilidades predeterminadas pasando una matriz como tercer argumento al método `resource`. Las teclas de la matriz definen los nombres de las habilidades mientras que los valores definen los nombres de los métodos. Por ejemplo, el siguiente código solo creará dos nuevas definiciones de puerta: `posts.image` y `posts.photo`:
> > By default, the `view`, `create`, `update`, and `delete` abilities will be defined. You may override the default abilities by passing an array as a third argument to the `resource` method. The keys of the array define the names of the abilities while the values define the method names. For example, the following code will only create two new Gate definitions - `posts.image` and `posts.photo`:

    Gate::resource('posts', 'PostPolicy', [
        'image' => 'updateImage',
        'photo' => 'updatePhoto',
    ]);

<a name="authorizing-actions-via-gates"></a>
### Autorización de acciones
### Authorizing Actions

Para autorizar una acción usando puertas, debe usar los métodos `allows` o `denies`. Tenga en cuenta que no es necesario pasar el usuario autenticado actualmente a estos métodos. Laravel se encargará automáticamente de pasar al usuario al cierre de la puerta:
> > To authorize an action using gates, you should use the `allows` or `denies` methods. Note that you are not required to pass the currently authenticated user to these methods. Laravel will automatically take care of passing the user into the gate Closure:

    if (Gate::allows('update-post', $post)) {
        // The current user can update the post...
    }

    if (Gate::denies('update-post', $post)) {
        // The current user can't update the post...
    }

Si desea determinar si un usuario en particular está autorizado a realizar una acción, puede usar el método `forUser` en la fachada `Gate`:
> > If you would like to determine if a particular user is authorized to perform an action, you may use the `forUser` method on the `Gate` facade:

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // The user can update the post...
    }

    if (Gate::forUser($user)->denies('update-post', $post)) {
        // The user can't update the post...
    }

<a name="intercepting-gate-checks"></a>
#### Interceptación de controles de puerta
#### Intercepting Gate Checks

En ocasiones, puede otorgar todas las habilidades a un usuario específico. Puede usar el método `before` para definir una devolución de llamada que se ejecuta antes de todas las demás comprobaciones de autorización:
> > Sometimes, you may wish to grant all abilities to a specific user. You may use the `before` method to define a callback that is run before all other authorization checks:

    Gate::before(function ($user, $ability) {
        if ($user->isSuperAdmin()) {
            return true;
        }
    });

Si la devolución de llamada `before` devuelve un resultado no nulo, el resultado se considerará el resultado de la comprobación.
> > If the `before` callback returns a non-null result that result will be considered the result of the check.

Puede usar el método `after` para definir una devolución de llamada que se ejecutará después de cada comprobación de autorización. Sin embargo, no puede modificar el resultado de la verificación de autorización de una devolución de llamada `after`:
> > You may use the `after` method to define a callback to be executed after every authorization check. However, you may not modify the result of the authorization check from an `after` callback:

    Gate::after(function ($user, $ability, $result, $arguments) {
        //
    });

<a name="creating-policies"></a>
## Creando Políticas
## Creating Policies

<a name="generating-policies"></a>
### Generando Políticas
### Generating Policies

Las políticas son clases que organizan la lógica de autorización en torno a un modelo o recurso en particular. Por ejemplo, si su aplicación es un blog, puede tener un modelo `Post` y una `PostPolicy` correspondiente para autorizar acciones del usuario, como crear o actualizar publicaciones.
> > Policies are classes that organize authorization logic around a particular model or resource. For example, if your application is a blog, you may have a `Post` model and a corresponding `PostPolicy` to authorize user actions such as creating or updating posts.

Puede generar una política utilizando `make:policy` [artisan command](/docs/{{version}}/artesanal). La política generada se colocará en el directorio `app/Policies`. Si este directorio no existe en su aplicación, Laravel lo creará por usted:
> > You may generate a policy using the `make:policy` [artisan command](/docs/{{version}}/artisan). The generated policy will be placed in the `app/Policies` directory. If this directory does not exist in your application, Laravel will create it for you:

    php artisan make:policy PostPolicy

El comando `make:policy` generará una clase de política vacía. Si desea generar una clase con los métodos de política básica "CRUD" ya incluidos en la clase, puede especificar un `--model` al ejecutar el comando:
> > The `make:policy` command will generate an empty policy class. If you would like to generate a class with the basic "CRUD" policy methods already included in the class, you may specify a `--model` when executing the command:

    php artisan make:policy PostPolicy --model=Post

> {tip} Todas las políticas se resuelven a través del Laravel [service container](/docs/{{version}}/container), lo que le permite indicar cualquier dependencia necesaria en el constructor de la política para que se les inyecte automáticamente.
> > > {tip} All policies are resolved via the Laravel [service container](/docs/{{version}}/container), allowing you to type-hint any needed dependencies in the policy's constructor to have them automatically injected.

<a name="registering-policies"></a>
### Políticas de registro
### Registering Policies

Una vez que existe la política, debe registrarse. El `AuthServiceProvider` incluido en las aplicaciones Laravel nuevas contiene una propiedad `policies` que asigna sus modelos Eloquent a sus políticas correspondientes. El registro de una política le indicará a Laravel qué política utilizar al autorizar acciones contra un modelo dado:
> > Once the policy exists, it needs to be registered. The `AuthServiceProvider` included with fresh Laravel applications contains a `policies` property which maps your Eloquent models to their corresponding policies. Registering a policy will instruct Laravel which policy to utilize when authorizing actions against a given model:

    <?php

    namespace App\Providers;

    use App\Post;
    use App\Policies\PostPolicy;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * The policy mappings for the application.
         *
         * @var array
         */
        protected $policies = [
            Post::class => PostPolicy::class,
        ];

        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            //
        }
    }

<a name="writing-policies"></a>
## Políticas de escritura
## Writing Policies

<a name="policy-methods"></a>
### Métodos de política
### Policy Methods

Una vez que se haya registrado la política, puede agregar métodos para cada acción que autorice. Por ejemplo, definamos un método `update` en nuestra` PostPolicy` que determina si un `User` dado puede actualizar una instancia `Post` determinada.
> > Once the policy has been registered, you may add methods for each action it authorizes. For example, let's define an `update` method on our `PostPolicy` which determines if a given `User` can update a given `Post` instance.

El método `update` recibirá una instancia `User` y `Post` como sus argumentos, y debería devolver `true` o `false` indicando si el usuario está autorizado a actualizar el `Post` dado. Por lo tanto, para este ejemplo, verifiquemos que el `id` del usuario coincida con el `user_id` en la publicación:
> > The `update` method will receive a `User` and a `Post` instance as its arguments, and should return `true` or `false` indicating whether the user is authorized to update the given `Post`. So, for this example, let's verify that the user's `id` matches the `user_id` on the post:

    <?php

    namespace App\Policies;

    use App\User;
    use App\Post;

    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\User  $user
         * @param  \App\Post  $post
         * @return bool
         */
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }

Puede continuar definiendo métodos adicionales en la política según sea necesario para las diversas acciones que autoriza. Por ejemplo, puede definir los métodos `view` o `delete` para autorizar varias acciones `Post`, pero recuerde que puede dar a los métodos de su política el nombre que desee.
> > You may continue to define additional methods on the policy as needed for the various actions it authorizes. For example, you might define `view` or `delete` methods to authorize various `Post` actions, but remember you are free to give your policy methods any name you like.

> {tip} Si usó la opción `--model` al generar su política a través de la consola Artisan, ya incluirá métodos para las acciones `view`, `create`,` update`, y `delete`.
> > > {tip} If you used the `--model` option when generating your policy via the Artisan console, it will already contain methods for the `view`, `create`, `update`, and `delete` actions.

<a name="methods-without-models"></a>
### Métodos sin modelos
### Methods Without Models

Algunos métodos de políticas solo reciben el usuario autenticado actualmente y no una instancia del modelo que autorizan. Esta situación es más común al autorizar acciones `create`. Por ejemplo, si está creando un blog, es posible que desee verificar si un usuario está autorizado a crear publicaciones.
> > Some policy methods only receive the currently authenticated user and not an instance of the model they authorize. This situation is most common when authorizing `create` actions. For example, if you are creating a blog, you may wish to check if a user is authorized to create any posts at all.

Al definir métodos de política que no recibirán una instancia de modelo, como un método `create`, no recibirá una instancia de modelo. En su lugar, debe definir el método como el único que espera al usuario autenticado:
> > When defining policy methods that will not receive a model instance, such as a `create` method, it will not receive a model instance. Instead, you should define the method as only expecting the authenticated user:

    /**
     * Determine if the given user can create posts.
     *
     * @param  \App\User  $user
     * @return bool
     */
    public function create(User $user)
    {
        //
    }

<a name="guest-users"></a>
### Usuarios Invitados
### Guest Users

Por defecto, todas las puertas y políticas devuelven `false` automáticamente si la solicitud HTTP entrante no fue iniciada por un usuario autenticado. Sin embargo, puede permitir que estas verificaciones de autorización pasen a sus puertas y políticas declarando una sugerencia de tipo "opcional" o suministrando un valor predeterminado `null` para la definición del argumento del usuario:
> > By default, all gates and policies automatically return `false` if the incoming HTTP request was not initiated by an authenticated user. However, you may allow these authorization checks to pass through to your gates and policies by declaring an "optional" type-hint or supplying a `null` default value for the user argument definition:

    <?php

    namespace App\Policies;

    use App\User;
    use App\Post;

    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\User  $user
         * @param  \App\Post  $post
         * @return bool
         */
        public function update(?User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }

<a name="policy-filters"></a>
### Filtros de política
### Policy Filters

Para ciertos usuarios, es posible que desee autorizar todas las acciones dentro de una política determinada. Para lograr esto, defina un método `before` en la política. El método `before` se ejecutará antes que cualquier otro método de la política, lo que le brinda la oportunidad de autorizar la acción antes de que realmente se llame al método de política previsto. Esta característica se usa con más frecuencia para autorizar a los administradores de aplicaciones a realizar cualquier acción:
> > For certain users, you may wish to authorize all actions within a given policy. To accomplish this, define a `before` method on the policy. The `before` method will be executed before any other methods on the policy, giving you an opportunity to authorize the action before the intended policy method is actually called. This feature is most commonly used for authorizing application administrators to perform any action:

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

Si desea denegar todas las autorizaciones para un usuario, debe devolver `false` del método `before`. Si se devuelve `null`, la autorización corresponderá al método de política.
> > If you would like to deny all authorizations for a user you should return `false` from the `before` method. If `null` is returned, the authorization will fall through to the policy method.

> {note} El método `before` de una clase de política no se invocará si la clase no contiene un método con un nombre que coincida con el nombre de la habilidad que se verifica.
> > > {note} The `before` method of a policy class will not be called if the class doesn't contain a method with a name matching the name of the ability being checked.

<a name="authorizing-actions-using-policies"></a>
## Autorización de acciones usando políticas
## Authorizing Actions Using Policies

<a name="via-the-user-model"></a>
### A través del modelo de usuario
### Via The User Model

El modelo `User` que se incluye con su aplicación Laravel incluye dos métodos útiles para autorizar acciones: `can` y `cant`. El método `can` recibe la acción que desea autorizar y el modelo relevante. Por ejemplo, determinemos si un usuario está autorizado a actualizar un modelo de `Post` dado:
> > The `User` model that is included with your Laravel application includes two helpful methods for authorizing actions: `can` and `cant`. The `can` method receives the action you wish to authorize and the relevant model. For example, let's determine if a user is authorized to update a given `Post` model:

    if ($user->can('update', $post)) {
        //
    }

Si una [policy is registered](#registering-policies) para el modelo dado, el método `can` llamará automáticamente a la política apropiada y devolverá el resultado booleano. Si no se registra ninguna política para el modelo, el método `can` intentará llamar a la puerta basada en Closure que coincida con el nombre de la acción dada.
> > If a [policy is registered](#registering-policies) for the given model, the `can` method will automatically call the appropriate policy and return the boolean result. If no policy is registered for the model, the `can` method will attempt to call the Closure based Gate matching the given action name.

#### Acciones que no requieren modelos
#### Actions That Don't Require Models

Recuerde, algunas acciones como `create` pueden no requerir una instancia modelo. En estas situaciones, puede pasar un nombre de clase al método `can`. El nombre de la clase se usará para determinar qué política usar al autorizar la acción:
> > Remember, some actions like `create` may not require a model instance. In these situations, you may pass a class name to the `can` method. The class name will be used to determine which policy to use when authorizing the action:

    use App\Post;

    if ($user->can('create', Post::class)) {
        // Executes the "create" method on the relevant policy...
    }

<a name="via-middleware"></a>
### A través de Middleware
### Via Middleware

Laravel incluye un middleware que puede autorizar acciones antes de que la solicitud entrante llegue a sus rutas o controladores. Por defecto, el middleware `Illuminate\Auth\Middleware\Authorize` tiene asignada la tecla `can` en su clase `App\Http\Kernel`. Analicemos un ejemplo del uso del middleware `can` para autorizar que un usuario pueda actualizar una publicación de blog:
> > Laravel includes a middleware that can authorize actions before the incoming request even reaches your routes or controllers. By default, the `Illuminate\Auth\Middleware\Authorize` middleware is assigned the `can` key in your `App\Http\Kernel` class. Let's explore an example of using the `can` middleware to authorize that a user can update a blog post:

    use App\Post;

    Route::put('/post/{post}', function (Post $post) {
        // The current user may update the post...
    })->middleware('can:update,post');

En este ejemplo, estamos pasando al middleware `can` dos argumentos. El primero es el nombre de la acción que deseamos autorizar y el segundo es el parámetro de ruta que deseamos pasar al método de política. En este caso, dado que estamos utilizando [vinculación de modelo implícita] (/docs/{{version}}/routing#implicit-binding), un modelo de `Post` se pasará al método de política. Si el usuario no está autorizado para realizar la acción dada, el middleware generará una respuesta HTTP con un código de estado `403`.
> > In this example, we're passing the `can` middleware two arguments. The first is the name of the action we wish to authorize and the second is the route parameter we wish to pass to the policy method. In this case, since we are using [implicit model binding](/docs/{{version}}/routing#implicit-binding), a `Post` model will be passed to the policy method. If the user is not authorized to perform the given action, a HTTP response with a `403` status code will be generated by the middleware.

#### Acciones que no requieren modelos
#### Actions That Don't Require Models

Nuevamente, algunas acciones como `create` pueden no requerir una instancia modelo. En estas situaciones, puede pasar un nombre de clase al middleware. El nombre de la clase se usará para determinar qué política usar al autorizar la acción:
> > Again, some actions like `create` may not require a model instance. In these situations, you may pass a class name to the middleware. The class name will be used to determine which policy to use when authorizing the action:

    Route::post('/post', function () {
        // The current user may create posts...
    })->middleware('can:create,App\Post');

<a name="via-controller-helpers"></a>
### Via Controller Helpers

Además de los métodos útiles proporcionados al modelo `User`, Laravel proporciona un método útil de `authorize` a cualquiera de sus controladores que extiende la clase base `App\Http\Controllers\Controller`. Al igual que el método `can`, este método acepta el nombre de la acción que desea autorizar y el modelo relevante. Si la acción no está autorizada, el método `authorize` lanzará un `Illuminate\Auth\Access\AuthorizationException`, que el manejador de excepciones Laravel predeterminado convertirá a una respuesta HTTP con un código de estado `403`:
> > In addition to helpful methods provided to the `User` model, Laravel provides a helpful `authorize` method to any of your controllers which extend the `App\Http\Controllers\Controller` base class. Like the `can` method, this method accepts the name of the action you wish to authorize and the relevant model. If the action is not authorized, the `authorize` method will throw an `Illuminate\Auth\Access\AuthorizationException`, which the default Laravel exception handler will convert to an HTTP response with a `403` status code:

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given blog post.
         *
         * @param  Request  $request
         * @param  Post  $post
         * @return Response
         * @throws \Illuminate\Auth\Access\AuthorizationException
         */
        public function update(Request $request, Post $post)
        {
            $this->authorize('update', $post);

            // The current user can update the blog post...
        }
    }

#### Acciones que no requieren modelos
#### Actions That Don't Require Models

Como se discutió anteriormente, algunas acciones como `create` pueden no requerir una instancia modelo. En estas situaciones, puede pasar un nombre de clase al método `authorize`. El nombre de la clase se usará para determinar qué política usar al autorizar la acción:
> > As previously discussed, some actions like `create` may not require a model instance. In these situations, you may pass a class name to the `authorize` method. The class name will be used to determine which policy to use when authorizing the action:

    /**
     * Create a new blog post.
     *
     * @param  Request  $request
     * @return Response
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);

        // The current user can create blog posts...
    }

<a name="via-blade-templates"></a>
### Plantillas de Via Blade
### Via Blade Templates

Al escribir plantillas Blade, es posible que desee mostrar una parte de la página solo si el usuario está autorizado para realizar una acción determinada. Por ejemplo, es posible que desee mostrar un formulario de actualización para una publicación de blog solo si el usuario puede actualizar la publicación. En esta situación, puede usar la familia de directivas `@can` y `@cannot`:
> > When writing Blade templates, you may wish to display a portion of the page only if the user is authorized to perform a given action. For example, you may wish to show an update form for a blog post only if the user can actually update the post. In this situation, you may use the `@can` and `@cannot` family of directives:

    @can('update', $post)
        <!-- The Current User Can Update The Post -->
    @elsecan('create', App\Post::class)
        <!-- The Current User Can Create New Post -->
    @endcan

    @cannot('update', $post)
        <!-- The Current User Can't Update The Post -->
    @elsecannot('create', App\Post::class)
        <!-- The Current User Can't Create New Post -->
    @endcannot

Estas directivas son accesos directos convenientes para escribir declaraciones `@if` y `@unless`. Las sentencias `@can` y `@cannot` mencionadas anteriormente se traducen a las siguientes declaraciones:
> > These directives are convenient shortcuts for writing `@if` and `@unless` statements. The `@can` and `@cannot` statements above respectively translate to the following statements:

    @if (Auth::user()->can('update', $post))
        <!-- The Current User Can Update The Post -->
    @endif

    @unless (Auth::user()->can('update', $post))
        <!-- The Current User Can't Update The Post -->
    @endunless

#### Acciones que no requieren modelos
#### Actions That Don't Require Models

Al igual que la mayoría de los otros métodos de autorización, puede pasar un nombre de clase a las directivas `@can` y `@cannot` si la acción no requiere una instancia modelo:
> > Like most of the other authorization methods, you may pass a class name to the `@can` and `@cannot` directives if the action does not require a model instance:

    @can('create', App\Post::class)
        <!-- The Current User Can Create Posts -->
    @endcan

    @cannot('create', App\Post::class)
        <!-- The Current User Can't Create Posts -->
    @endcannot
