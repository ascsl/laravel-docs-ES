# Authentication
# Autenticación

- [Introduction](#introduction)
    - [Database Considerations](#introduction-database-considerations)
- [Authentication Quickstart](#authentication-quickstart)
    - [Routing](#included-routing)
    - [Views](#included-views)
    - [Authenticating](#included-authenticating)
    - [Retrieving The Authenticated User](#retrieving-the-authenticated-user)
    - [Protecting Routes](#protecting-routes)
    - [Login Throttling](#login-throttling)
- [Manually Authenticating Users](#authenticating-users)
    - [Remembering Users](#remembering-users)
    - [Other Authentication Methods](#other-authentication-methods)
- [HTTP Basic Authentication](#http-basic-authentication)
    - [Stateless HTTP Basic Authentication](#stateless-http-basic-authentication)
- [Logging Out](#logging-out)
    - [Invalidating Sessions On Other Devices](#invalidating-sessions-on-other-devices)
- [Social Authentication](https://github.com/laravel/socialite)
- [Adding Custom Guards](#adding-custom-guards)
    - [Closure Request Guards](#closure-request-guards)
- [Adding Custom User Providers](#adding-custom-user-providers)
    - [The User Provider Contract](#the-user-provider-contract)
    - [The Authenticatable Contract](#the-authenticatable-contract)
- [Events](#events)

<a name="introduction"></a>
## Introducción
## Introduction

> {tip} **¿Quieres comenzar rápido?** Simplemente ejecuta `php artisan make:auth` y `php artisan migrate` en una nueva aplicación Laravel. Luego, navegue su navegador a `http://your-app.test/register` o cualquier otra URL que esté asignada a su aplicación. ¡Estos dos comandos se encargarán de andamiar todo tu sistema de autenticación!
> > > {tip} **Want to get started fast?** Just run `php artisan make:auth` and `php artisan migrate` in a fresh Laravel application. Then, navigate your browser to `http://your-app.test/register` or any other URL that is assigned to your application. These two commands will take care of scaffolding your entire authentication system!

Laravel hace que la implementación de autenticación sea muy simple. De hecho, casi todo está configurado para ti fuera de la caja. El archivo de configuración de autenticación se encuentra en `config/auth.php`, que contiene varias opciones bien documentadas para ajustar el comportamiento de los servicios de autenticación.
> > Laravel makes implementing authentication very simple. In fact, almost everything is configured for you out of the box. The authentication configuration file is located at `config/auth.php`, which contains several well documented options for tweaking the behavior of the authentication services.

En esencia, las instalaciones de autenticación de Laravel están formadas por "guardias" y "proveedores". Los guardias definen cómo se autentican los usuarios para cada solicitud. Por ejemplo, Laravel se envía con un guardia de "sesión" que mantiene el estado usando el almacenamiento de sesión y las cookies.
> > At its core, Laravel's authentication facilities are made up of "guards" and "providers". Guards define how users are authenticated for each request. For example, Laravel ships with a `session` guard which maintains state using session storage and cookies.

Los proveedores definen cómo se recuperan los usuarios de su almacenamiento persistente. Laravel se envía con soporte para recuperar usuarios usando Eloquent y el generador de consultas de bases de datos. Sin embargo, puede definir proveedores adicionales según sea necesario para su aplicación.
> > Providers define how users are retrieved from your persistent storage. Laravel ships with support for retrieving users using Eloquent and the database query builder. However, you are free to define additional providers as needed for your application.

¡No te preocupes si todo esto parece confuso ahora! Muchas aplicaciones nunca necesitarán modificar la configuración de autenticación predeterminada.
> > Don't worry if this all sounds confusing now! Many applications will never need to modify the default authentication configuration.

<a name="introduction-database-considerations"></a>
### Consideraciones de la base de datos
### Database Considerations

Por defecto, Laravel incluye una 'App\User' [Modelo Eloquent] (/docs/{{version}}/eloquent) en su directorio `app`. Este modelo se puede usar con el controlador de autenticación Eloquent predeterminado. Si su aplicación no está utilizando Eloquent, puede usar el controlador de autenticación `database` que usa el constructor de consultas Laravel.
> > By default, Laravel includes an `App\User` [Eloquent model](/docs/{{version}}/eloquent) in your `app` directory. This model may be used with the default Eloquent authentication driver. If your application is not using Eloquent, you may use the `database` authentication driver which uses the Laravel query builder.

Al crear el esquema de la base de datos para el modelo `App\User`, asegúrese de que la columna de contraseña tenga al menos 60 caracteres de longitud. Mantener una longitud de columna de cadena predeterminada de 255 caracteres sería una buena opción.
> > When building the database schema for the `App\User` model, make sure the password column is at least 60 characters in length. Maintaining the default string column length of 255 characters would be a good choice.

Además, debe verificar que su tabla `users` (o equivalente) contenga una columna nulo, de cadena `remember_token` de 100 caracteres. Esta columna se usará para almacenar un token para los usuarios que seleccionen la opción "recordarme" al iniciar sesión en su aplicación.
> > Also, you should verify that your `users` (or equivalent) table contains a nullable, string `remember_token` column of 100 characters. This column will be used to store a token for users that select the "remember me" option when logging into your application.

<a name="authentication-quickstart"></a>
## Autenticación Inicio rápido
## Authentication Quickstart

Laravel se envía con varios controladores de autenticación preconstruidos, que se encuentran en el espacio de nombres `App\Http\Controllers\Auth`. El `RegisterController` maneja el nuevo registro de usuario, el` LoginController` maneja la autenticación, el `ForgotPasswordController` maneja los enlaces de correo electrónico para restablecer las contraseñas, y `ResetPasswordController` contiene la lógica para restablecer las contraseñas. Cada uno de estos controladores usa un rasgo para incluir sus métodos necesarios. Para muchas aplicaciones, no necesitará modificar estos controladores en absoluto.
> > Laravel ships with several pre-built authentication controllers, which are located in the `App\Http\Controllers\Auth` namespace. The `RegisterController` handles new user registration, the `LoginController` handles authentication, the `ForgotPasswordController` handles e-mailing links for resetting passwords, and the `ResetPasswordController` contains the logic to reset passwords. Each of these controllers uses a trait to include their necessary methods. For many applications, you will not need to modify these controllers at all.

<a name="included-routing"></a>
### Enrutamiento
### Routing

Laravel proporciona una manera rápida de andamiar todas las rutas y vistas que necesita para la autenticación con un simple comando:
> > Laravel provides a quick way to scaffold all of the routes and views you need for authentication using one simple command:

    php artisan make:auth

Este comando se debe usar en aplicaciones nuevas e instalará una vista de diseño, vistas de registro e inicio de sesión, así como rutas para todos los puntos finales de autenticación. También se generará un `HomeController` para manejar las solicitudes posteriores al inicio de sesión en el tablero de su aplicación.
> > This command should be used on fresh applications and will install a layout view, registration and login views, as well as routes for all authentication end-points. A `HomeController` will also be generated to handle post-login requests to your application's dashboard.

<a name="included-views"></a>
### Vistas
### Views

Como se mencionó en la sección anterior, el comando `php artisan make:auth` creará todas las vistas que necesita para la autenticación y las colocará en el directorio `resources/views/auth`.
> > As mentioned in the previous section, the `php artisan make:auth` command will create all of the views you need for authentication and place them in the `resources/views/auth` directory.

El comando `make:auth` también creará un directorio `resources/views/layouts` que contiene un diseño base para su aplicación. Todas estas vistas usan el marco CSS de Bootstrap, pero usted es libre de personalizarlas como lo desee.
> > The `make:auth` command will also create a `resources/views/layouts` directory containing a base layout for your application. All of these views use the Bootstrap CSS framework, but you are free to customize them however you wish.

<a name="included-authenticating"></a>
### Autenticación
### Authenticating

Ahora que tiene configuraciones de rutas y vistas para los controladores de autenticación incluidos, ¡está listo para registrarse y autenticar nuevos usuarios para su aplicación! Puede acceder a su aplicación en un navegador ya que los controladores de autenticación ya contienen la lógica (a través de sus características) para autenticar usuarios existentes y almacenar nuevos usuarios en la base de datos.
> > Now that you have routes and views setup for the included authentication controllers, you are ready to register and authenticate new users for your application! You may access your application in a browser since the authentication controllers already contain the logic (via their traits) to authenticate existing users and store new users in the database.

#### Personalización de ruta
#### Path Customization

Cuando un usuario se autentica con éxito, se le redirigirá al URI `/home`. Puede personalizar la ubicación de redirección posterior a la autenticación definiendo una propiedad `redirectTo` en` LoginController`, `RegisterController` y` ResetPasswordController`:
> > When a user is successfully authenticated, they will be redirected to the `/home` URI. You can customize the post-authentication redirect location by defining a `redirectTo` property on the `LoginController`, `RegisterController`, and `ResetPasswordController`:

    protected $redirectTo = '/';

A continuación, debe modificar el método `handle` del middleware `RedirectIfAuthenticated` para usar su nuevo URI al redireccionar al usuario.
> > Next, you should modify the `RedirectIfAuthenticated` middleware's `handle` method to use your new URI when redirecting the user.

Si la ruta de redireccionamiento necesita lógica de generación personalizada, puede definir un método `redirectTo` en lugar de una propiedad` redirectTo`:
> > If the redirect path needs custom generation logic you may define a `redirectTo` method instead of a `redirectTo` property:

    protected function redirectTo()
    {
        return '/path';
    }

> {tip} The `redirectTo` method will take precedence over the `redirectTo` attribute.

#### Personalización del nombre de usuario
#### Username Customization

Por defecto, Laravel usa el campo `email` para autenticación. Si desea personalizar esto, puede definir un método `username` en su` LoginController`:
> > By default, Laravel uses the `email` field for authentication. If you would like to customize this, you may define a `username` method on your `LoginController`:

    public function username()
    {
        return 'username';
    }

#### Personalización de guardia
#### Guard Customization

También puede personalizar el "guardia" que se utiliza para autenticar y registrar usuarios. Para comenzar, defina un método `guard` en su` LoginController`, `RegisterController`, y` ResetPasswordController`. El método debe devolver una instancia de guardia:
> > You may also customize the "guard" that is used to authenticate and register users. To get started, define a `guard` method on your `LoginController`, `RegisterController`, and `ResetPasswordController`. The method should return a guard instance:

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### Validación / Personalización de almacenamiento
#### Validation / Storage Customization

Para modificar los campos de formulario que se requieren cuando un nuevo usuario se registra con su aplicación, o para personalizar cómo se almacenan los nuevos usuarios en su base de datos, puede modificar la clase `RegisterController`. Esta clase es responsable de validar y crear nuevos usuarios de su aplicación.
> > To modify the form fields that are required when a new user registers with your application, or to customize how new users are stored into your database, you may modify the `RegisterController` class. This class is responsible for validating and creating new users of your application.

El método `validator` de` RegisterController` contiene las reglas de validación para los nuevos usuarios de la aplicación. Usted es libre de modificar este método como desee.
> > The `validator` method of the `RegisterController` contains the validation rules for new users of the application. You are free to modify this method as you wish.

El método `create` de` RegisterController` es responsable de crear nuevos registros `App\User` en su base de datos usando el [Eloquent ORM] (/docs/{{version}}/eloquent). Usted es libre de modificar este método de acuerdo a las necesidades de su base de datos.
> > The `create` method of the `RegisterController` is responsible for creating new `App\User` records in your database using the [Eloquent ORM](/docs/{{version}}/eloquent). You are free to modify this method according to the needs of your database.

<a name="retrieving-the-authenticated-user"></a>
### Retrieving The Authenticated User
### Recuperando el usuario autenticado

Puede acceder al usuario autenticado a través de la fachada `Auth`:
> > You may access the authenticated user via the `Auth` facade:

    use Illuminate\Support\Facades\Auth;

    // Get the currently authenticated user...
    $user = Auth::user();

    // Get the currently authenticated user's ID...
    $id = Auth::id();

Alternativamente, una vez que un usuario es autenticado, puede acceder al usuario autenticado a través de una instancia `Illuminate\Http\Request`. Recuerde, las clases de tipo insinuado se inyectarán automáticamente en sus métodos de controlador:
> > Alternatively, once a user is authenticated, you may access the authenticated user via an `Illuminate\Http\Request` instance. Remember, type-hinted classes will automatically be injected into your controller methods:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class ProfileController extends Controller
    {
        /**
         * Update the user's profile.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // $request->user() returns an instance of the authenticated user...
        }
    }

#### Determinando si el usuario actual está autenticado
#### Determining If The Current User Is Authenticated

Para determinar si el usuario ya inició sesión en su aplicación, puede usar el método `check` en la fachada `Auth`, que devolverá `true` si el usuario está autenticado:
> > To determine if the user is already logged into your application, you may use the `check` method on the `Auth` facade, which will return `true` if the user is authenticated:

    use Illuminate\Support\Facades\Auth;

    if (Auth::check()) {
        // The user is logged in...
    }

> {tip} Aunque es posible determinar si un usuario se autentica usando el método `check`, normalmente usará un middleware para verificar que el usuario esté autenticado antes de permitir que el usuario acceda a ciertas rutas/controladores. Para obtener más información al respecto, consulte la documentación sobre [rutas de protección] (/docs/{{version}}/authentication#protecting-routes).
> > > {tip} Even though it is possible to determine if a user is authenticated using the `check` method, you will typically use a middleware to verify that the user is authenticated before allowing the user access to certain routes / controllers. To learn more about this, check out the documentation on [protecting routes](/docs/{{version}}/authentication#protecting-routes).

<a name="protecting-routes"></a>
### Protección de rutas
### Protecting Routes

[El middleware de ruta](/docs/{{version}}/middleware) solo se puede usar para permitir que los usuarios autenticados accedan a una ruta determinada. Laravel se envía con un middleware `auth`, que se define en `Illuminate\Auth\Middleware\Authenticate`. Dado que este middleware ya está registrado en su núcleo HTTP, todo lo que necesita hacer es adjuntar el middleware a una definición de ruta:
> > [Route middleware](/docs/{{version}}/middleware) can be used to only allow authenticated users to access a given route. Laravel ships with an `auth` middleware, which is defined at `Illuminate\Auth\Middleware\Authenticate`. Since this middleware is already registered in your HTTP kernel, all you need to do is attach the middleware to a route definition:

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth');

Por supuesto, si está utilizando [controladores] (/docs/{{version}}/controllers), puede llamar al método `middleware` desde el constructor del controlador en lugar de asociarlo directamente a la definición de ruta:
> > Of course, if you are using [controllers](/docs/{{version}}/controllers), you may call the `middleware` method from the controller's constructor instead of attaching it in the route definition directly:

    public function __construct()
    {
        $this->middleware('auth');
    }

#### Redirigir usuarios no autenticados
#### Redirecting Unauthenticated Users

Cuando el middleware `auth` detecta a un usuario no autorizado, devolverá una respuesta JSON `401` o, si la solicitud no era una solicitud AJAX, redireccionará al usuario al `login` [ruta especificada] (/docs/{{version}}/routing#named-routes).
> > When the `auth` middleware detects an unauthorized user, it will either return a JSON `401` response, or, if the request was not an AJAX request, redirect the user to the `login` [named route](/docs/{{version}}/routing#named-routes).

Puede modificar este comportamiento definiendo una función `no autenticada` en su archivo `app/Exceptions/Handler.php`:
> > You may modify this behavior by defining an `unauthenticated` function in your `app/Exceptions/Handler.php` file:

    use Illuminate\Auth\AuthenticationException;

    protected function unauthenticated($request, AuthenticationException $exception)
    {
        return $request->expectsJson()
                    ? response()->json(['message' => $exception->getMessage()], 401)
                    : redirect()->guest(route('login'));
    }

#### Especificar una guardia
#### Specifying A Guard

Al asociar el middleware `auth` a una ruta, también puede especificar qué guardia se debe usar para autenticar al usuario. El protector especificado debe corresponderse con una de las claves del array `guards` del archivo de configuración `auth.php`:
> > When attaching the `auth` middleware to a route, you may also specify which guard should be used to authenticate the user. The guard specified should correspond to one of the keys in the `guards` array of your `auth.php` configuration file:

    public function __construct()
    {
        $this->middleware('auth:api');
    }

<a name="login-throttling"></a>
### Regulación de inicio de sesión
### Login Throttling

Si está utilizando la clase incorporada `LoginController` de Laravel, el rasgo `Illuminate\Foundation\Auth\ThrottlesLogins` ya estará incluido en su controlador. Por defecto, el usuario no podrá iniciar sesión durante un minuto si no proporciona las credenciales correctas después de varios intentos. La aceleración es exclusiva del nombre de usuario / dirección de correo electrónico del usuario y su dirección IP.
> > If you are using Laravel's built-in `LoginController` class, the `Illuminate\Foundation\Auth\ThrottlesLogins` trait will already be included in your controller. By default, the user will not be able to login for one minute if they fail to provide the correct credentials after several attempts. The throttling is unique to the user's username / e-mail address and their IP address.

<a name="authenticating-users"></a>
## Autenticación manual de usuarios
## Manually Authenticating Users

Por supuesto, no está obligado a utilizar los controladores de autenticación incluidos con Laravel. Si decide eliminar estos controladores, deberá administrar la autenticación de usuario utilizando directamente las clases de autenticación de Laravel. No te preocupes, es muy fácil!
> > Of course, you are not required to use the authentication controllers included with Laravel. If you choose to remove these controllers, you will need to manage user authentication using the Laravel authentication classes directly. Don't worry, it's a cinch!

Accederemos a los servicios de autenticación de Laravel a través de `Auth` [fachada] (/docs/{{version}}/facades), por lo que necesitaremos asegurarnos de importar la fachada `Auth` en la parte superior de la clase. A continuación, veamos el método `attempt`:
> > We will access Laravel's authentication services via the `Auth` [facade](/docs/{{version}}/facades), so we'll need to make sure to import the `Auth` facade at the top of the class. Next, let's check out the `attempt` method:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;

    class LoginController extends Controller
    {
        /**
         * Handle an authentication attempt.
         *
         * @param  \Illuminate\Http\Request $request
         *
         * @return Response
         */
        public function authenticate(Request $request)
        {
            $credentials = $request->only('email', 'password');

            if (Auth::attempt($credentials)) {
                // Authentication passed...
                return redirect()->intended('dashboard');
            }
        }
    }

El método `attempt` acepta un array de pares clave / valor como primer argumento. Los valores del array se usarán para encontrar al usuario en la tabla de la base de datos. Entonces, en el ejemplo anterior, el usuario será recuperado por el valor de la columna `email`. Si se encuentra el usuario, la contraseña hash almacenada en la base de datos se comparará con el valor de `password` que se pasa al método a través del array. No debe hash la contraseña especificada como el valor de `password`, ya que el marco automáticamente reducirá el valor antes de compararlo con la contraseña hash en la base de datos. Si las dos contraseñas hash coinciden, se iniciará una sesión autenticada para el usuario.
> > The `attempt` method accepts an array of key / value pairs as its first argument. The values in the array will be used to find the user in your database table. So, in the example above, the user will be retrieved by the value of the `email` column. If the user is found, the hashed password stored in the database will be compared with the `password` value passed to the method via the array. You should not hash the password specified as the `password` value, since the framework will automatically hash the value before comparing it to the hashed password in the database. If the two hashed passwords match an authenticated session will be started for the user.

El método `attempt` devolverá `true` si la autenticación fue exitosa. De lo contrario, se devolverá `false`.
> > The `attempt` method will return `true` if authentication was successful. Otherwise, `false` will be returned.

El método `intended` en el redirector redirigirá al usuario a la URL a la que intentaron acceder antes de ser interceptado por el middleware de autenticación. Se puede dar un URI alternativo a este método en caso de que el destino previsto no esté disponible.
> > The `intended` method on the redirector will redirect the user to the URL they were attempting to access before being intercepted by the authentication middleware. A fallback URI may be given to this method in case the intended destination is not available.

#### Especificación de condiciones adicionales
#### Specifying Additional Conditions

Si lo desea, también puede agregar condiciones adicionales a la consulta de autenticación, además del correo electrónico y la contraseña del usuario. Por ejemplo, podemos verificar que el usuario esté marcado como "activo":
> > If you wish, you may also add extra conditions to the authentication query in addition to the user's e-mail and password. For example, we may verify that user is marked as "active":

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }

> {note} En estos ejemplos, `email` no es una opción obligatoria, simplemente se usa como ejemplo. Debe usar el nombre de columna que corresponda a un "nombre de usuario" en su base de datos.
> > > {note} In these examples, `email` is not a required option, it is merely used as an example. You should use whatever column name corresponds to a "username" in your database.

#### Acceso a instancias de Guardia Específica
#### Accessing Specific Guard Instances

Puede especificar qué instancia de guardia le gustaría utilizar utilizando el método `guard` en la fachada `Auth`. Esto le permite administrar la autenticación para partes separadas de su aplicación utilizando modelos autenticables o tablas de usuario completamente independientes.
> > You may specify which guard instance you would like to utilize using the `guard` method on the `Auth` facade. This allows you to manage authentication for separate parts of your application using entirely separate authenticatable models or user tables.

El nombre del guardián pasado al método `guard` debe corresponder a uno de los guardias configurados en su archivo de configuración `auth.php`:
> > The guard name passed to the `guard` method should correspond to one of the guards configured in your `auth.php` configuration file:

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### Saliendo de tu cuenta
#### Logging Out

Para desconectar usuarios de su aplicación, puede usar el método `logout` en la fachada `Auth`. Esto borrará la información de autenticación en la sesión del usuario:
> > To log users out of your application, you may use the `logout` method on the `Auth` facade. This will clear the authentication information in the user's session:

    Auth::logout();

<a name="remembering-users"></a>
### Recordar usuarios
### Remembering Users

Si desea proporcionar la funcionalidad "recordarme" en su aplicación, puede pasar un valor booleano como el segundo argumento para el método `attempt`, que mantendrá al usuario autenticado indefinidamente, o hasta que cierre la sesión manualmente. Por supuesto, su tabla `users` debe incluir la columna `remember_token`, que se usará para almacenar el token "recordarme".
> > If you would like to provide "remember me" functionality in your application, you may pass a boolean value as the second argument to the `attempt` method, which will keep the user authenticated indefinitely, or until they manually logout. Of course, your `users` table must include the string `remember_token` column, which will be used to store the "remember me" token.

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

> {tip} Si está utilizando el `LoginController` incorporado con Laravel, la lógica adecuada para "recordar" a los usuarios ya está implementada por los rasgos utilizados por el controlador.
> > > {tip} If you are using the built-in `LoginController` that is shipped with Laravel, the proper logic to "remember" users is already implemented by the traits used by the controller.

Si está "recordando" a los usuarios, puede usar el método `viaRemember` para determinar si el usuario fue autenticado usando la cookie "recordarme":
> > If you are "remembering" users, you may use the `viaRemember` method to determine if the user was authenticated using the "remember me" cookie:

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### Otros métodos de autenticación
### Other Authentication Methods

#### Autenticar una instancia de usuario
#### Authenticate A User Instance

Si necesita registrar una instancia de usuario existente en su aplicación, puede llamar al método `login` con la instancia del usuario. El objeto dado debe ser una implementación de `Illuminate\Contracts\Auth\Authenticatable` [contract](/docs/{{version}}/contracts). Por supuesto, el modelo `App\User` incluido con Laravel ya implementa esta interfaz:
> > If you need to log an existing user instance into your application, you may call the `login` method with the user instance. The given object must be an implementation of the `Illuminate\Contracts\Auth\Authenticatable` [contract](/docs/{{version}}/contracts). Of course, the `App\User` model included with Laravel already implements this interface:

    Auth::login($user);

    // Login and "remember" the given user...
    Auth::login($user, true);

Por supuesto, puede especificar la instancia de guardia que le gustaría usar:
> > Of course, you may specify the guard instance you would like to use:

    Auth::guard('admin')->login($user);

#### Autenticar un usuario por ID
#### Authenticate A User By ID

Para registrar a un usuario en la aplicación por su ID, puede usar el método `loginUsingId`. Este método acepta la clave principal del usuario que desea autenticar:
> > To log a user into the application by their ID, you may use the `loginUsingId` method. This method accepts the primary key of the user you wish to authenticate:

    Auth::loginUsingId(1);

    // Login and "remember" the given user...
    Auth::loginUsingId(1, true);

#### Autenticar un usuario una vez
#### Authenticate A User Once

Puede usar el método `once` para registrar a un usuario en la aplicación para una única solicitud. No se utilizarán sesiones ni cookies, lo que significa que este método puede ser útil al crear una API sin estado:
> > You may use the `once` method to log a user into the application for a single request. No sessions or cookies will be utilized, which means this method may be helpful when building a stateless API:

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## Autenticación básica HTTP
## HTTP Basic Authentication

[Autenticación básica HTTP](https://en.wikipedia.org/wiki/Basic_access_authentication) proporciona una forma rápida de autenticar usuarios de su aplicación sin configurar una página dedicada de "inicio de sesión". Para comenzar, adjunte `auth.basic` [middleware](/docs/{{version}}/middleware) a su ruta. El middleware `auth.basic` está incluido con el framework de Laravel, por lo que no necesita definirlo:
> > [HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) provides a quick way to authenticate users of your application without setting up a dedicated "login" page. To get started, attach the `auth.basic` [middleware](/docs/{{version}}/middleware) to your route. The `auth.basic` middleware is included with the Laravel framework, so you do not need to define it:

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic');

Una vez que el middleware se haya adjuntado a la ruta, se le solicitarán automáticamente las credenciales al acceder a la ruta en su navegador. Por defecto, el middleware `auth.basic` usará la columna `email` en el registro del usuario como el "nombre de usuario".
> > Once the middleware has been attached to the route, you will automatically be prompted for credentials when accessing the route in your browser. By default, the `auth.basic` middleware will use the `email` column on the user record as the "username".

#### Una nota sobre FastCGI
#### A Note On FastCGI

Si está utilizando PHP FastCGI, la autenticación HTTP básica puede no funcionar correctamente de fábrica. Las siguientes líneas se deben agregar a su archivo `.htaccess`:
> > If you are using PHP FastCGI, HTTP Basic authentication may not work correctly out of the box. The following lines should be added to your `.htaccess` file:

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### Autenticación básica HTTP sin estado
### Stateless HTTP Basic Authentication

También puede usar Autenticación básica HTTP sin establecer una cookie de identificador de usuario en la sesión, que es particularmente útil para la autenticación API. Para hacerlo, [defina un middleware] (/docs/{{version}}/middleware) que llame al método `onceBasic`. Si el método `onceBasic` no devuelve ninguna respuesta, la solicitud puede pasarse a la aplicación:
> > You may also use HTTP Basic Authentication without setting a user identifier cookie in the session, which is particularly useful for API authentication. To do so, [define a middleware](/docs/{{version}}/middleware) that calls the `onceBasic` method. If no response is returned by the `onceBasic` method, the request may be passed further into the application:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Support\Facades\Auth;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

A continuación, [registre el middleware de la ruta](/docs/{{version}}/middleware#registration-middleware) y adjúntelo a una ruta:
> > Next, [register the route middleware](/docs/{{version}}/middleware#registering-middleware) and attach it to a route:

    Route::get('api/user', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic.once');

<a name="logging-out"></a>
## Saliendo de tu cuenta
## Logging Out

Para registrar manualmente usuarios fuera de su aplicación, puede usar el método `logout` en la fachada` Auth`. Esto borrará la información de autenticación en la sesión del usuario:
> > To manually log users out of your application, you may use the `logout` method on the `Auth` facade. This will clear the authentication information in the user's session:

    use Illuminate\Support\Facades\Auth;

    Auth::logout();

<a name="invalidating-sessions-on-other-devices"></a>
### Invalidar sesiones en otros dispositivos
### Invalidating Sessions On Other Devices

Laravel también proporciona un mecanismo para invalidar y "cerrar sesión" las sesiones de un usuario que están activas en otros dispositivos sin invalidar la sesión en su dispositivo actual. Antes de comenzar, debe asegurarse de que el middleware `Illuminate\Session\Middleware\AuthenticateSession` esté presente y no comentado en su grupo de middleware `app/Http/Kernel.php` class' `web`:
> > Laravel also provides a mechanism for invalidating and "logging out" a user's sessions that are active on other devices without invalidating the session on their current device. Before getting started, you should make sure that the `Illuminate\Session\Middleware\AuthenticateSession` middleware is present and un-commented in your `app/Http/Kernel.php` class' `web` middleware group:

    'web' => [
        // ...
        \Illuminate\Session\Middleware\AuthenticateSession::class,
        // ...
    ],

Luego, puede usar el método `logoutOtherDevices` en la fachada `Auth`. Este método requiere que el usuario proporcione su contraseña actual, que su aplicación debe aceptar a través de un formulario de entrada:
> > Then, you may use the `logoutOtherDevices` method on the `Auth` facade. This method requires the user to provide their current password, which your application should accept through an input form:

    use Illuminate\Support\Facades\Auth;

    Auth::logoutOtherDevices($password);

> {note} Cuando se invoca el método `logoutOtherDevices`, las demás sesiones del usuario se invalidarán por completo, lo que significa que serán "desconectadas" de todas las guardias con las que fueron previamente autenticadas.
> > > {note} When the `logoutOtherDevices` method is invoked, the user's other sessions will be invalidated entirely, meaning they will be "logged out" of all guards they were previously authenticated by.

<a name="adding-custom-guards"></a>
## Agregar guardias personalizados
## Adding Custom Guards

Puede definir sus propias protecciones de autenticación utilizando el método `extender` en la fachada `Auth`. Debería realizar esta llamada para "extender" dentro de un [proveedor de servicios] (/docs/{{version}}/providers). Dado que Laravel ya se envía con un `AuthServiceProvider`, podemos colocar el código en ese proveedor:
> > You may define your own authentication guards using the `extend` method on the `Auth` facade. You should place this call to `extend` within a [service provider](/docs/{{version}}/providers). Since Laravel already ships with an `AuthServiceProvider`, we can place the code in that provider:

    <?php

    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
    use Illuminate\Support\Facades\Auth;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::extend('jwt', function ($app, $name, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\Guard...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }

Como puede ver en el ejemplo anterior, la devolución de llamada pasada al método `extender` debe devolver una implementación de `Iluminar\Contratos\Auth\Guard`. Esta interfaz contiene algunos métodos que deberá implementar para definir una guardia personalizada. Una vez que haya definido su guardia personalizada, puede usar esta protección en la configuración `guards` de su archivo de configuración `auth.php`:
> > As you can see in the example above, the callback passed to the `extend` method should return an implementation of `Illuminate\Contracts\Auth\Guard`. This interface contains a few methods you will need to implement to define a custom guard. Once your custom guard has been defined, you may use this guard in the `guards` configuration of your `auth.php` configuration file:

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="closure-request-guards"></a>
### Guardias de solicitud de cierre
### Closure Request Guards

La forma más sencilla de implementar un sistema de autenticación basado en solicitudes HTTP personalizado es mediante el método `Auth::viaRequest`. Este método le permite definir rápidamente su proceso de autenticación utilizando un solo cierre.
> > The simplest way to implement a custom, HTTP request based authentication system is by using the `Auth::viaRequest` method. This method allows you to quickly define your authentication process using a single Closure.

Para comenzar, llame al método `Auth::viaRequest` dentro del método `boot` de su `AuthServiceProvider`. El método `viaRequest` acepta un nombre de guardia como su primer argumento. Este nombre puede ser cualquier cadena que describa tu guardia personalizada. El segundo argumento que se pasa al método debe ser un Cierre que recibe la solicitud HTTP entrante y devuelve una instancia de usuario o, si la autenticación falla, `null`:
> > To get started, call the `Auth::viaRequest` method within the `boot` method of your `AuthServiceProvider`. The `viaRequest` method accepts a guard name as its first argument. This name can be any string that describes your custom guard. The second argument passed to the method should be a Closure that receives the incoming HTTP request and returns a user instance or, if authentication fails, `null`:

    use App\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;

    /**
     * Register any application authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Auth::viaRequest('custom-token', function ($request) {
            return User::where('token', $request->token)->first();
        });
    }

Una vez que haya definido su guardia personalizada, puede usar esta protección en la configuración `guards` de su archivo de configuración `auth.php`:
> > Once your custom guard has been defined, you may use this guard in the `guards` configuration of your `auth.php` configuration file:

    'guards' => [
        'api' => [
            'driver' => 'custom-token',
        ],
    ],

<a name="adding-custom-user-providers"></a>
## Agregar proveedores de usuario personalizados
## Adding Custom User Providers

Si no está utilizando una base de datos relacional tradicional para almacenar a sus usuarios, deberá extenderla Laravel con su propio proveedor de autenticación. Usaremos el método `provider` en la fachada `Auth` para definir un proveedor de usuario personalizado:
> > If you are not using a traditional relational database to store your users, you will need to extend Laravel with your own authentication user provider. We will use the `provider` method on the `Auth` facade to define a custom user provider:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::provider('riak', function ($app, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...

                return new RiakUserProvider($app->make('riak.connection'));
            });
        }
    }

Después de que haya registrado el proveedor utilizando el método `provider`, puede cambiar al nuevo proveedor de usuario en su archivo de configuración `auth.php`. Primero, defina un `provider` que use su nuevo controlador:
> > After you have registered the provider using the `provider` method, you may switch to the new user provider in your `auth.php` configuration file. First, define a `provider` that uses your new driver:

    'providers' => [
        'users' => [
            'driver' => 'riak',
        ],
    ],

Finalmente, puede usar este proveedor en su configuración `guards`:
> > Finally, you may use this provider in your `guards` configuration:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

<a name="the-user-provider-contract"></a>
### El contrato de proveedor de usuario
### The User Provider Contract

Las implementaciones `Illuminate\Contracts\Auth\UserProvider` solo son responsables de obtener una implementación `Illuminate\Contracts\Auth\Authenticatable` de un sistema de almacenamiento persistente, como MySQL, Riak, etc. Estas dos interfaces permiten los mecanismos de autenticación de Laravel para continuar funcionando independientemente de cómo se almacenan los datos del usuario o qué tipo de clase se utiliza para representarlo.
> > The `Illuminate\Contracts\Auth\UserProvider` implementations are only responsible for fetching a `Illuminate\Contracts\Auth\Authenticatable` implementation out of a persistent storage system, such as MySQL, Riak, etc. These two interfaces allow the Laravel authentication mechanisms to continue functioning regardless of how the user data is stored or what type of class is used to represent it.

Echemos un vistazo al contrato `Illuminate\Contracts\Auth\UserProvider`:
> > Let's take a look at the `Illuminate\Contracts\Auth\UserProvider` contract:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

La función `retrieveById` generalmente recibe una clave que representa al usuario, como una identificación autoincrementada de una base de datos MySQL. La implementación `Authenticatable` que coincida con la ID debe ser recuperada y devuelta por el método.
> > The `retrieveById` function typically receives a key representing the user, such as an auto-incrementing ID from a MySQL database. The `Authenticatable` implementation matching the ID should be retrieved and returned by the method.

La función `retrieveByToken` recupera a un usuario por su `$identifier` único y "recuerdame" `$token`, almacenado en un campo `remember_token`. Al igual que con el método anterior, debe devolverse la implementación `Authenticatable`.
> > The `retrieveByToken` function retrieves a user by their unique `$identifier` and "remember me" `$token`, stored in a field `remember_token`. As with the previous method, the `Authenticatable` implementation should be returned.

El método `updateRememberToken` actualiza el campo `$user` `remember_token` con el nuevo `$token`. Se asigna un token nuevo en un intento exitoso de "recordarme" o cuando el usuario está cerrando sesión.
> > The `updateRememberToken` method updates the `$user` field `remember_token` with the new `$token`. A fresh token is assigned on a successful "remember me" login attempt or when the user is logging out.

El método `retrieveByCredentials` recibe la matriz de credenciales pasadas al método `Auth::attempt` cuando intenta iniciar sesión en una aplicación. El método debería entonces "consultar" el almacenamiento persistente subyacente para el usuario que coincida con esas credenciales. Normalmente, este método ejecutará una consulta con una condición "where" en `$credentials['username']`. El método debería devolver una implementación de `Authenticatable`. ** Este método no debe intentar realizar ninguna validación o autenticación con contraseña. **
> > The `retrieveByCredentials` method receives the array of credentials passed to the `Auth::attempt` method when attempting to sign into an application. The method should then "query" the underlying persistent storage for the user matching those credentials. Typically, this method will run a query with a "where" condition on `$credentials['username']`. The method should then return an implementation of `Authenticatable`. **This method should not attempt to do any password validation or authentication.**

El método `validateCredentials` debe comparar el `$user` dado con las `$credentials` para autenticar al usuario. Por ejemplo, este método probablemente debería usar `Hash::check` para comparar el valor de `$user->getAuthPassword()` con el valor de `$credentials['password']`. Este método debe devolver `true` o` false` indicando si la contraseña es válida.
> > The `validateCredentials` method should compare the given `$user` with the `$credentials` to authenticate the user. For example, this method should probably use `Hash::check` to compare the value of `$user->getAuthPassword()` to the value of `$credentials['password']`. This method should return `true` or `false` indicating on whether the password is valid.

<a name="the-authenticatable-contract"></a>
### El Contrato Authenticatable
### The Authenticatable Contract

Ahora que hemos explorado cada uno de los métodos en el `UserProvider`, echemos un vistazo al contrato `Authenticatable`. Recuerde, el proveedor debe devolver las implementaciones de esta interfaz de los métodos `retrieveById`, `retrieveByToken`, y `retrieveByCredentials`:
> > Now that we have explored each of the methods on the `UserProvider`, let's take a look at the `Authenticatable` contract. Remember, the provider should return implementations of this interface from the `retrieveById`, `retrieveByToken`, and `retrieveByCredentials` methods:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

Esta interfaz es simple. El método `getAuthIdentifierName` debe devolver el nombre del campo "clave principal" del usuario y el método `getAuthIdentifier` debe devolver la "clave primaria" del usuario. En un back-end de MySQL, nuevamente, esta sería la clave primaria de incremento automático. La `getAuthPassword` debe devolver la contraseña hash del usuario. Esta interfaz permite que el sistema de autenticación funcione con cualquier clase de usuario, independientemente de qué capa de abstracción de almacenamiento o ORM está utilizando. Por defecto, Laravel incluye una clase `User` en el directorio `app` que implementa esta interfaz, por lo que puede consultar esta clase para obtener un ejemplo de implementación.
> > This interface is simple. The `getAuthIdentifierName` method should return the name of the "primary key" field of the user and the `getAuthIdentifier` method should return the "primary key" of the user. In a MySQL back-end, again, this would be the auto-incrementing primary key. The `getAuthPassword` should return the user's hashed password. This interface allows the authentication system to work with any User class, regardless of what ORM or storage abstraction layer you are using. By default, Laravel includes a `User` class in the `app` directory which implements this interface, so you may consult this class for an implementation example.

<a name="events"></a>
## Eventos
## Events

Laravel genera una variedad de [eventos](/docs/{{version}}/eventos) durante el proceso de autenticación. Puede adjuntar oyentes a estos eventos en su `EventServiceProvider`:
> > Laravel raises a variety of [events](/docs/{{version}}/events) during the authentication process. You may attach listeners to these events in your `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Registered' => [
            'App\Listeners\LogRegisteredUser',
        ],

        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],

        'Illuminate\Auth\Events\Authenticated' => [
            'App\Listeners\LogAuthenticated',
        ],

        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],

        'Illuminate\Auth\Events\Failed' => [
            'App\Listeners\LogFailedLogin',
        ],

        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],

        'Illuminate\Auth\Events\Lockout' => [
            'App\Listeners\LogLockout',
        ],

        'Illuminate\Auth\Events\PasswordReset' => [
            'App\Listeners\LogPasswordReset',
        ],
    ];
