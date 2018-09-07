# API Authentication (Passport)

- [Introduction](#introduction)
- [Installation](#installation)
    - [Frontend Quickstart](#frontend-quickstart)
    - [Deploying Passport](#deploying-passport)
- [Configuration](#configuration)
    - [Token Lifetimes](#token-lifetimes)
- [Issuing Access Tokens](#issuing-access-tokens)
    - [Managing Clients](#managing-clients)
    - [Requesting Tokens](#requesting-tokens)
    - [Refreshing Tokens](#refreshing-tokens)
- [Password Grant Tokens](#password-grant-tokens)
    - [Creating A Password Grant Client](#creating-a-password-grant-client)
    - [Requesting Tokens](#requesting-password-grant-tokens)
    - [Requesting All Scopes](#requesting-all-scopes)
- [Implicit Grant Tokens](#implicit-grant-tokens)
- [Client Credentials Grant Tokens](#client-credentials-grant-tokens)
- [Personal Access Tokens](#personal-access-tokens)
    - [Creating A Personal Access Client](#creating-a-personal-access-client)
    - [Managing Personal Access Tokens](#managing-personal-access-tokens)
- [Protecting Routes](#protecting-routes)
    - [Via Middleware](#via-middleware)
    - [Passing The Access Token](#passing-the-access-token)
- [Token Scopes](#token-scopes)
    - [Defining Scopes](#defining-scopes)
    - [Assigning Scopes To Tokens](#assigning-scopes-to-tokens)
    - [Checking Scopes](#checking-scopes)
- [Consuming Your API With JavaScript](#consuming-your-api-with-javascript)
- [Events](#events)
- [Testing](#testing)

<a name="introduction"></a>
## Introducción : Introduction

Laravel ya facilita la autenticación a través de los formularios de inicio de sesión tradicionales, pero ¿qué pasa con las API? Las API generalmente usan tokens para autenticar usuarios y no mantienen el estado de la sesión entre las solicitudes. Laravel hace que la autenticación de la API sea muy sencilla utilizando Laravel Passport, que proporciona una implementación completa del servidor OAuth2 para su aplicación Laravel en cuestión de minutos. Passport está construido sobre el servidor [League OAuth2](https://github.com/thephpleague/oauth2-server) mantenido por Andy Millington y Simon Hamp.
> > Laravel already makes it easy to perform authentication via traditional login forms, but what about APIs? APIs typically use tokens to authenticate users and do not maintain session state between requests. Laravel makes API authentication a breeze using Laravel Passport, which provides a full OAuth2 server implementation for your Laravel application in a matter of minutes. Passport is built on top of the [League OAuth2 server](https://github.com/thephpleague/oauth2-server) that is maintained by Andy Millington and Simon Hamp.

> {note} Esta documentación supone que ya está familiarizado con OAuth2. Si no sabe nada sobre OAuth2, considere familiarizarse con la terminología general y las características de OAuth2 antes de continuar.
> > > {note} This documentation assumes you are already familiar with OAuth2. If you do not know anything about OAuth2, consider familiarizing yourself with the general terminology and features of OAuth2 before continuing.

<a name="installation"></a>
## Instalación : Installation

Para comenzar, instale Passport a través del administrador de paquetes de Composer:
> > To get started, install Passport via the Composer package manager:

    composer require laravel/passport

El proveedor de servicios de Passport registra su propio directorio de migración de bases de datos con el framework, por lo que debe migrar su base de datos después de registrar el proveedor. Las migraciones de Passport crearán las tablas que su aplicación necesita para almacenar clientes y tokens de acceso:
> > The Passport service provider registers its own database migration directory with the framework, so you should migrate your database after registering the provider. The Passport migrations will create the tables your application needs to store clients and access tokens:

    php artisan migrate

> {note} Si no va a utilizar las migraciones predeterminadas de Passport, debe llamar al método `Passport::ignoreMigrations` en el método `register` de `AppServiceProvider`. Puede exportar las migraciones predeterminadas utilizando `php artisan vendor:publish --tag=passport-migrations`.
> > > {note} If you are not going to use Passport's default migrations, you should call the `Passport::ignoreMigrations` method in the `register` method of your `AppServiceProvider`. You may export the default migrations using `php artisan vendor:publish --tag=passport-migrations`.

A continuación, debe ejecutar el comando `passport:install`. Este comando creará las claves de cifrado necesarias para generar tokens de acceso seguro. Además, el comando creará clientes de "acceso personal" y "concesión de contraseña" que se usarán para generar tokens de acceso:
> > Next, you should run the `passport:install` command. This command will create the encryption keys needed to generate secure access tokens. In addition, the command will create "personal access" and "password grant" clients which will be used to generate access tokens:

    php artisan passport:install

Después de ejecutar este comando, agregue el rasgo `Laravel\Passport\HasApiTokens` a su modelo `App\User`. Este rasgo proporcionará algunos métodos de ayuda a su modelo que le permiten inspeccionar el token y los ámbitos del usuario autenticado:
> > After running this command, add the `Laravel\Passport\HasApiTokens` trait to your `App\User` model. This trait will provide a few helper methods to your model which allow you to inspect the authenticated user's token and scopes:

    <?php

    namespace App;

    use Laravel\Passport\HasApiTokens;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;
    }

A continuación, debe llamar al método `Passport::routes` dentro del método `boot` de su `AuthServiceProvider`. Este método registrará las rutas necesarias para emitir tokens de acceso y revocar tokens de acceso, clientes y tokens de acceso personal:
> > Next, you should call the `Passport::routes` method within the `boot` method of your `AuthServiceProvider`. This method will register the routes necessary to issue access tokens and revoke access tokens, clients, and personal access tokens:

    <?php

    namespace App\Providers;

    use Laravel\Passport\Passport;
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
            'App\Model' => 'App\Policies\ModelPolicy',
        ];

        /**
         * Register any authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Passport::routes();
        }
    }

Finalmente, en su archivo de configuración `config/auth.php`, debe establecer la opción `driver` del protector de autenticación `api` en `passport`. Esto le indicará a su aplicación que use `TokenGuard` de Passport al autenticar las solicitudes API entrantes:
> > Finally, in your `config/auth.php` configuration file, you should set the `driver` option of the `api` authentication guard to `passport`. This will instruct your application to use Passport's `TokenGuard` when authenticating incoming API requests:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],

<a name="frontend-quickstart"></a>
### Frontend Quickstart

> {note} Para usar los componentes de Passport Vue, debe usar el framework de JavaScript [Vue](https://vuejs.org). Estos componentes también usan el marco CSS de Bootstrap. Sin embargo, incluso si no está utilizando estas herramientas, los componentes sirven como una referencia valiosa para su propia implementación frontend.
> > > {note} In order to use the Passport Vue components, you must be using the [Vue](https://vuejs.org) JavaScript framework. These components also use the Bootstrap CSS framework. However, even if you are not using these tools, the components serve as a valuable reference for your own frontend implementation.

Passport se envía con una API JSON que puede usar para permitir a sus usuarios crear clientes y tokens de acceso personal. Sin embargo, puede llevar mucho tiempo codificar una interfaz para interactuar con estas API. Por lo tanto, Passport también incluye componentes precompilados [Vue](https://vuejs.org) que puede usar como ejemplo de implementación o punto de partida para su propia implementación.
> > Passport ships with a JSON API that you may use to allow your users to create clients and personal access tokens. However, it can be time consuming to code a frontend to interact with these APIs. So, Passport also includes pre-built [Vue](https://vuejs.org) components you may use as an example implementation or starting point for your own implementation.

Para publicar los componentes de Passport Vue, use el comando Artisan `vendor:publish`:
> > To publish the Passport Vue components, use the `vendor:publish` Artisan command:

    php artisan vendor:publish --tag=passport-components

Los componentes publicados se colocarán en su directorio `resources/assets/js/components`. Una vez que los componentes se hayan publicado, debe registrarlos en su archivo `resources/assets/js/app.js`:
> > The published components will be placed in your `resources/assets/js/components` directory. Once the components have been published, you should register them in your `resources/assets/js/app.js` file:

    Vue.component(
        'passport-clients',
        require('./components/passport/Clients.vue')
    );

    Vue.component(
        'passport-authorized-clients',
        require('./components/passport/AuthorizedClients.vue')
    );

    Vue.component(
        'passport-personal-access-tokens',
        require('./components/passport/PersonalAccessTokens.vue')
    );

Después de registrar los componentes, asegúrese de ejecutar `npm run dev` para recompilar sus assets. Una vez que haya recompilado sus assets, puede soltar los componentes en una de las plantillas de su aplicación para comenzar a crear clientes y tokens de acceso personal:
> > After registering the components, make sure to run `npm run dev` to recompile your assets. Once you have recompiled your assets, you may drop the components into one of your application's templates to get started creating clients and personal access tokens:

    <passport-clients></passport-clients>
    <passport-authorized-clients></passport-authorized-clients>
    <passport-personal-access-tokens></passport-personal-access-tokens>

<a name="deploying-passport"></a>
### Implementando Passport : Deploying Passport

Al implementar Passport en sus servidores de producción por primera vez, es probable que necesite ejecutar el comando `passport:keys`. Este comando genera las claves de encriptación que Passport necesita para generar token de acceso. Las claves generadas no suelen mantenerse en control de origen:
> > When deploying Passport to your production servers for the first time, you will likely need to run the `passport:keys` command. This command generates the encryption keys Passport needs in order to generate access token. The generated keys are not typically kept in source control:

    php artisan passport:keys

<a name="configuration"></a>
## Configuración : Configuration

<a name="token-lifetimes"></a>
### Duración de token : Token Lifetimes

Por defecto, Passport emite tokens de acceso de larga duración que caducan después de un año. Si desea configurar una vida útil de token más larga o más corta, puede usar los métodos `tokensExpireIn` y `refreshTokensExpireIn`. Estos métodos deben invocarse desde el método `boot` de su `AuthServiceProvider`:
> > By default, Passport issues long-lived access tokens that expire after one year. If you would like to configure a longer / shorter token lifetime, you may use the `tokensExpireIn` and `refreshTokensExpireIn` methods. These methods should be called from the `boot` method of your `AuthServiceProvider`:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::tokensExpireIn(now()->addDays(15));

        Passport::refreshTokensExpireIn(now()->addDays(30));
    }

<a name="issuing-access-tokens"></a>
## Emisión de tokens de acceso : Issuing Access Tokens

El uso de OAuth2 con códigos de autorización es la forma en que la mayoría de los desarrolladores están familiarizados con OAuth2. Al usar códigos de autorización, una aplicación cliente redireccionará a un usuario a su servidor donde aprobará o denegará la solicitud de emitir un token de acceso al cliente.
> > Using OAuth2 with authorization codes is how most developers are familiar with OAuth2. When using authorization codes, a client application will redirect a user to your server where they will either approve or deny the request to issue an access token to the client.

<a name="managing-clients"></a>
### Gestión de clientes : Managing Clients

En primer lugar, los desarrolladores que crean aplicaciones que necesitan interactuar con la API de su aplicación deberán registrar su aplicación con la suya creando un "cliente". Normalmente, esto consiste en proporcionar el nombre de su aplicación y una URL a la que su aplicación puede redirigir después de que los usuarios aprueban su solicitud de autorización.
> > First, developers building applications that need to interact with your application's API will need to register their application with yours by creating a "client". Typically, this consists of providing the name of their application and a URL that your application can redirect to after users approve their request for authorization.

#### El comando `pasaporte: cliente` : The `passport:client` Command

La forma más sencilla de crear un cliente es usar el comando Artisan `passport:client`. Este comando puede usarse para crear sus propios clientes para probar su funcionalidad OAuth2. Cuando ejecuta el comando `client`, Passport le solicitará más información sobre su cliente y le proporcionará una identificación y un secreto del cliente:
> > The simplest way to create a client is using the `passport:client` Artisan command. This command may be used to create your own clients for testing your OAuth2 functionality. When you run the `client` command, Passport will prompt you for more information about your client and will provide you with a client ID and secret:

    php artisan passport:client

#### API JSON : JSON API

Como sus usuarios no podrán utilizar el comando `client`, Passport proporciona una API JSON que puede usar para crear clientes. Esto le ahorra la molestia de tener que codificar manualmente los controladores para crear, actualizar y eliminar clientes.
> > Since your users will not be able to utilize the `client` command, Passport provides a JSON API that you may use to create clients. This saves you the trouble of having to manually code controllers for creating, updating, and deleting clients.

Sin embargo, deberá emparejar la API JSON de Passport con su propia interfaz para proporcionar un tablero para que los usuarios administren a sus clientes. A continuación, revisaremos todos los puntos finales API para administrar clientes. Para mayor comodidad, utilizaremos [Axios](https://github.com/mzabriskie/axios) para demostrar la realización de solicitudes HTTP a los puntos finales.
> > However, you will need to pair Passport's JSON API with your own frontend to provide a dashboard for your users to manage their clients. Below, we'll review all of the API endpoints for managing clients. For convenience, we'll use [Axios](https://github.com/mzabriskie/axios) to demonstrate making HTTP requests to the endpoints.

> {tip} Si no quiere implementar la interfaz completa de administración de clientes usted mismo, puede usar [inicio rápido de frontend](#frontend-quickstart) para tener una interfaz completamente funcional en cuestión de minutos.
> > > {tip} If you don't want to implement the entire client management frontend yourself, you can use the [frontend quickstart](#frontend-quickstart) to have a fully functional frontend in a matter of minutes.

#### `GET /oauth/clients`

Esta ruta devuelve todos los clientes para el usuario autenticado. Esto es principalmente útil para enumerar todos los clientes del usuario para que puedan editarlos o eliminarlos:
> > This route returns all of the clients for the authenticated user. This is primarily useful for listing all of the user's clients so that they may edit or delete them:

    axios.get('/oauth/clients')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/clients`

Esta ruta se usa para crear nuevos clientes. Requiere dos datos: el `name` del cliente y una URL `redirect`. La URL `redirect` es donde el usuario será redirigido después de aprobar o denegar una solicitud de autorización.
> > This route is used to create new clients. It requires two pieces of data: the client's `name` and a `redirect` URL. The `redirect` URL is where the user will be redirected after approving or denying a request for authorization.

Cuando se crea un cliente, se emitirá un ID de cliente y un secreto de cliente. Estos valores se usarán cuando solicite tokens de acceso desde su aplicación. La ruta de creación del cliente devolverá la nueva instancia del cliente:
> > When a client is created, it will be issued a client ID and client secret. These values will be used when requesting access tokens from your application. The client creation route will return the new client instance:

    const data = {
        name: 'Client Name',
        redirect: 'http://example.com/callback'
    };

    axios.post('/oauth/clients', data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });

#### `PUT /oauth/clients/{client-id}`

Esta ruta se usa para actualizar clientes. Requiere dos datos: el `name` del cliente y una URL `redirect`. La URL `redirect` es donde el usuario será redirigido después de aprobar o denegar una solicitud de autorización. La ruta devolverá la instancia del cliente actualizada:
> > This route is used to update clients. It requires two pieces of data: the client's `name` and a `redirect` URL. The `redirect` URL is where the user will be redirected after approving or denying a request for authorization. The route will return the updated client instance:

    const data = {
        name: 'New Client Name',
        redirect: 'http://example.com/callback'
    };

    axios.put('/oauth/clients/' + clientId, data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });

#### `DELETE /oauth/clients/{client-id}`

Esta ruta se usa para eliminar clientes:
> > This route is used to delete clients:

    axios.delete('/oauth/clients/' + clientId)
        .then(response => {
            //
        });

<a name="requesting-tokens"></a>
### Solicitud de tokens : Requesting Tokens

#### Redireccionamiento para la autorización : Redirecting For Authorization

Una vez que se ha creado un cliente, los desarrolladores pueden usar su ID de cliente y su secreto para solicitar un código de autorización y un token de acceso desde su aplicación. En primer lugar, la aplicación consumidora debe hacer una solicitud de redireccionamiento a la ruta `/oauth/authorize` de su aplicación de la siguiente manera:
> > Once a client has been created, developers may use their client ID and secret to request an authorization code and access token from your application. First, the consuming application should make a redirect request to your application's `/oauth/authorize` route like so:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} Recuerde, la ruta `/oauth/authorize` ya está definida por el método `Passport::routes`. No necesita definir manualmente esta ruta.
> > > {tip} Remember, the `/oauth/authorize` route is already defined by the `Passport::routes` method. You do not need to manually define this route.

#### Aprobar la solicitud : Approving The Request

Al recibir solicitudes de autorización, Passport mostrará automáticamente una plantilla al usuario, lo que le permitirá aprobar o denegar la solicitud de autorización. Si aprueban la solicitud, se les redirigirá al `redirect_uri` especificado por la aplicación consumidora. El `redirect_uri` debe coincidir con la URL `redirect` que se especificó cuando se creó el cliente.
> > When receiving authorization requests, Passport will automatically display a template to the user allowing them to approve or deny the authorization request. If they approve the request, they will be redirected back to the `redirect_uri` that was specified by the consuming application. The `redirect_uri` must match the `redirect` URL that was specified when the client was created.

Si desea personalizar la pantalla de aprobación de autorización, puede publicar las vistas de Passport utilizando el comando Artisan 'vendor:publish`. Las vistas publicadas se colocarán en `resources/views/vendor/passport`:
> > If you would like to customize the authorization approval screen, you may publish Passport's views using the `vendor:publish` Artisan command. The published views will be placed in `resources/views/vendor/passport`:

    php artisan vendor:publish --tag=passport-views

#### Conversión de códigos de autorización para acceder a tokens : Converting Authorization Codes To Access Tokens

Si el usuario aprueba la solicitud de autorización, se le redirigirá a la aplicación que lo consume. El consumidor debe emitir una solicitud `POST` a su aplicación para solicitar un token de acceso. La solicitud debe incluir el código de autorización emitido por su aplicación cuando el usuario aprobó la solicitud de autorización. En este ejemplo, usaremos la biblioteca Guzzle HTTP para realizar la solicitud `POST`:
> > If the user approves the authorization request, they will be redirected back to the consuming application. The consumer should then issue a `POST` request to your application to request an access token. The request should include the authorization code that was issued by your application when the user approved the authorization request. In this example, we'll use the Guzzle HTTP library to make the `POST` request:

    Route::get('/callback', function (Request $request) {
        $http = new GuzzleHttp\Client;

        $response = $http->post('http://your-app.com/oauth/token', [
            'form_params' => [
                'grant_type' => 'authorization_code',
                'client_id' => 'client-id',
                'client_secret' => 'client-secret',
                'redirect_uri' => 'http://example.com/callback',
                'code' => $request->code,
            ],
        ]);

        return json_decode((string) $response->getBody(), true);
    });

Esta ruta `/oauth/token` devolverá una respuesta JSON que contiene los atributos `access_token`, `refresh_token`, y `expires_in`. El atributo `expires_in` contiene el número de segundos hasta que caduque el token de acceso.
> > This `/oauth/token` route will return a JSON response containing `access_token`, `refresh_token`, and `expires_in` attributes. The `expires_in` attribute contains the number of seconds until the access token expires.

> {tip} Al igual que en la ruta `/oauth/authorize`, la ruta `/oauth/token` está definida para usted por el método `Passport::routes`. No es necesario definir manualmente esta ruta.
> > > {tip} Like the `/oauth/authorize` route, the `/oauth/token` route is defined for you by the `Passport::routes` method. There is no need to manually define this route.

<a name="refreshing-tokens"></a>
### Resfrescando Tokens : Refreshing Tokens

Si su aplicación emite tokens de acceso de corta duración, los usuarios necesitarán actualizar sus tokens de acceso a través del token de actualización que se les proporcionó cuando se emitió el token de acceso. En este ejemplo, usaremos la biblioteca Guzzle HTTP para actualizar el token:
> > If your application issues short-lived access tokens, users will need to refresh their access tokens via the refresh token that was provided to them when the access token was issued. In this example, we'll use the Guzzle HTTP library to refresh the token:

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'refresh_token',
            'refresh_token' => 'the-refresh-token',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

Esta ruta `/oauth/token` devolverá una respuesta JSON que contiene los atributos `access_token`, `refresh_token`, y `expires_in`. El atributo `expires_in` contiene el número de segundos hasta que caduque el token de acceso.
> > This `/oauth/token` route will return a JSON response containing `access_token`, `refresh_token`, and `expires_in` attributes. The `expires_in` attribute contains the number of seconds until the access token expires.

<a name="password-grant-tokens"></a>
## Tokens de concesión de contraseña : Password Grant Tokens

La concesión de contraseña de OAuth2 permite que sus otros clientes propios, como una aplicación móvil, obtengan un token de acceso usando una dirección de correo electrónico / nombre de usuario y contraseña. Esto le permite emitir tokens de acceso de forma segura a sus clientes propios sin que los usuarios tengan que pasar por el flujo completo de redireccionamiento del código de autorización de OAuth2.
> > The OAuth2 password grant allows your other first-party clients, such as a mobile application, to obtain an access token using an e-mail address / username and password. This allows you to issue access tokens securely to your first-party clients without requiring your users to go through the entire OAuth2 authorization code redirect flow.

<a name="creating-a-password-grant-client"></a>
### Creación de un cliente de concesión de contraseña : Creating A Password Grant Client

Antes de que su aplicación pueda emitir tokens mediante la concesión de contraseñas, deberá crear un cliente de concesión de contraseñas. Puede hacer esto usando el comando `passport:client` con la opción `--password`. Si ya ha ejecutado el comando `pasaport:install`, no necesita ejecutar este comando:
> > Before your application can issue tokens via the password grant, you will need to create a password grant client. You may do this using the `passport:client` command with the `--password` option. If you have already run the `passport:install` command, you do not need to run this command:

    php artisan passport:client --password

<a name="requesting-password-grant-tokens"></a>
### Solicitud de tokens : Requesting Tokens

Una vez que haya creado un cliente de concesión de contraseñas, puede solicitar un token de acceso emitiendo una solicitud `POST` a la ruta `/oauth/token` con la dirección de correo electrónico y la contraseña del usuario. Recuerde, esta ruta ya está registrada por el método `Passport::routes`, por lo que no es necesario definirla manualmente. Si la solicitud es exitosa, recibirá un `access_token` y `refresh_token` en la respuesta JSON del servidor:
> > Once you have created a password grant client, you may request an access token by issuing a `POST` request to the `/oauth/token` route with the user's email address and password. Remember, this route is already registered by the `Passport::routes` method so there is no need to define it manually. If the request is successful, you will receive an `access_token` and `refresh_token` in the JSON response from the server:

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

> {tip} Recuerde, los tokens de acceso son de larga duración por defecto. Sin embargo, puede [configurar su vida útil máxima de token de acceso](#configuration) si es necesario.
> > > {tip} Remember, access tokens are long-lived by default. However, you are free to [configure your maximum access token lifetime](#configuration) if needed.

<a name="requesting-all-scopes"></a>
### Solicitud de todos los ámbitos : Requesting All Scopes

Al utilizar la concesión de contraseña, es posible que desee autorizar el token para todos los ámbitos admitidos por su aplicación. Puede hacer esto solicitando el alcance `*`. Si solicita el alcance `*`, el método `can` en la instancia del token siempre devolverá `true`. Este alcance solo se puede asignar a un token que se emite utilizando la concesión `password`:
> > When using the password grant, you may wish to authorize the token for all of the scopes supported by your application. You can do this by requesting the `*` scope. If you request the `*` scope, the `can` method on the token instance will always return `true`. This scope may only be assigned to a token that is issued using the `password` grant:

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '*',
        ],
    ]);

<a name="implicit-grant-tokens"></a>
## Tokens de concesión implícitos : Implicit Grant Tokens

La concesión implícita es similar a la concesión del código de autorización; sin embargo, el token se devuelve al cliente sin intercambiar un código de autorización. Esta subvención se utiliza con mayor frecuencia para JavaScript o aplicaciones móviles donde las credenciales del cliente no se pueden almacenar de forma segura. Para habilitar la concesión, llame al método `enableImplicitGrant` en su` AuthServiceProvider`:
> > The implicit grant is similar to the authorization code grant; however, the token is returned to the client without exchanging an authorization code. This grant is most commonly used for JavaScript or mobile applications where the client credentials can't be securely stored. To enable the grant, call the `enableImplicitGrant` method in your `AuthServiceProvider`:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::enableImplicitGrant();
    }

Una vez que ha habilitado una concesión, los usuarios pueden usar su ID de cliente para solicitar un acceso desde su aplicación. La aplicación consumidora debe hacer una solicitud de redireccionamiento a la ruta `/oauth/authorize` de su application de la siguiente manera:
> > Once a grant has been enabled, developers may use their client ID to request an access token from your application. The consuming application should make a redirect request to your application's `/oauth/authorize` route like so:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'token',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} Recuerde, la ruta `/oauth/authorize` ya está definida por el método `Passport::routes`. No necesita definir manualmente esta ruta.
> > > {tip} Remember, the `/oauth/authorize` route is already defined by the `Passport::routes` method. You do not need to manually define this route.

<a name="client-credentials-grant-tokens"></a>
## Client Credentials Grant Tokens

La concesión de credenciales del cliente es adecuada para la autenticación de máquina a máquina. Por ejemplo, puede usar esta concesión en un trabajo programado que realiza tareas de mantenimiento en una API. Para utilizar este método, primero debe agregar un nuevo middleware a su `$routeMiddleware` en `app/Http/Kernel.php`:
> > The client credentials grant is suitable for machine-to-machine authentication. For example, you might use this grant in a scheduled job which is performing maintenance tasks over an API. To use this method you first need to add new middleware to your `$routeMiddleware` in `app/Http/Kernel.php`:

    use Laravel\Passport\Http\Middleware\CheckClientCredentials;

    protected $routeMiddleware = [
        'client' => CheckClientCredentials::class,
    ];

A continuación, conecte este middleware a una ruta:
> > Then attach this middleware to a route:

    Route::get('/user', function(Request $request) {
        ...
    })->middleware('client');

Para recuperar un token, realice una solicitud al punto final `oauth/token`:
> > To retrieve a token, make a request to the `oauth/token` endpoint:

    $guzzle = new GuzzleHttp\Client;

    $response = $guzzle->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'client_credentials',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => 'your-scope',
        ],
    ]);

    return json_decode((string) $response->getBody(), true)['access_token'];

<a name="personal-access-tokens"></a>
## Tokens de acceso personal : Personal Access Tokens

En ocasiones, es posible que los usuarios deseen emitir tokens de acceso a sí mismos sin pasar por el flujo de redirección de código de autorización típico. Permitir que los usuarios emitan tokens a sí mismos a través de la IU de su aplicación puede ser útil para permitir que los usuarios experimenten con su API o puede servir como un enfoque más simple para emitir tokens de acceso en general.
> > Sometimes, your users may want to issue access tokens to themselves without going through the typical authorization code redirect flow. Allowing users to issue tokens to themselves via your application's UI can be useful for allowing users to experiment with your API or may serve as a simpler approach to issuing access tokens in general.

> {note} Los tokens de acceso personal siempre son de larga duración. Su duración no se modifica cuando se utilizan los métodos `tokensExpireIn` o `refreshTokensExpireIn`.
> > > {note} Personal access tokens are always long-lived. Their lifetime is not modified when using the `tokensExpireIn` or `refreshTokensExpireIn` methods.

<a name="creating-a-personal-access-client"></a>
### Creando un cliente de acceso personal : Creating A Personal Access Client

Antes de que su aplicación pueda emitir tokens de acceso personal, deberá crear un cliente de acceso personal. Puede hacer esto usando el comando `passport:client` con la opción `--personal`. Si ya ha ejecutado el comando `pasaport:install`, no necesita ejecutar este comando:
> > Before your application can issue personal access tokens, you will need to create a personal access client. You may do this using the `passport:client` command with the `--personal` option. If you have already run the `passport:install` command, you do not need to run this command:

    php artisan passport:client --personal

<a name="managing-personal-access-tokens"></a>
### Administrar tokens de acceso personal : Managing Personal Access Tokens

Una vez que haya creado un cliente de acceso personal, puede emitir tokens para un usuario dado utilizando el método `createToken` en la instancia del modelo `User`. El método `createToken` acepta el nombre del token como primer argumento y un array opcional de [scopes](#token-scopes) como segundo argumento:
> > Once you have created a personal access client, you may issue tokens for a given user using the `createToken` method on the `User` model instance. The `createToken` method accepts the name of the token as its first argument and an optional array of [scopes](#token-scopes) as its second argument:

    $user = App\User::find(1);

    // Creating a token without scopes...
    $token = $user->createToken('Token Name')->accessToken;

    // Creating a token with scopes...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

#### API JSON : JSON API

Passport también incluye una API JSON para administrar tokens de acceso personal. Puede emparejar esto con su propia interfaz para ofrecer a sus usuarios un panel para administrar tokens de acceso personal. A continuación, revisaremos todos los puntos finales API para administrar tokens de acceso personal. Para mayor comodidad, utilizaremos [Axios](https://github.com/mzabriskie/axios) para demostrar la realización de solicitudes HTTP a los puntos finales.
> > Passport also includes a JSON API for managing personal access tokens. You may pair this with your own frontend to offer your users a dashboard for managing personal access tokens. Below, we'll review all of the API endpoints for managing personal access tokens. For convenience, we'll use [Axios](https://github.com/mzabriskie/axios) to demonstrate making HTTP requests to the endpoints.

> {tip} Si no desea implementar personalmente el frontend de token de acceso personal, puede usar [inicio rápido de frontend](#frontend-quickstart) para tener un frontend completamente funcional en cuestión de minutos.
> > > {tip} If you don't want to implement the personal access token frontend yourself, you can use the [frontend quickstart](#frontend-quickstart) to have a fully functional frontend in a matter of minutes.

#### `GET /oauth/scopes`

Esta ruta devuelve todos los [ámbitos](#token-scopes) definidos para su aplicación. Puede usar esta ruta para enumerar los ámbitos que un usuario puede asignar a un token de acceso personal:
> > This route returns all of the [scopes](#token-scopes) defined for your application. You may use this route to list the scopes a user may assign to a personal access token:

    axios.get('/oauth/scopes')
        .then(response => {
            console.log(response.data);
        });

#### `GET /oauth/personal-access-tokens`

Esta ruta devuelve todos los tokens de acceso personal que ha creado el usuario autenticado. Esto es principalmente útil para enumerar todos los tokens del usuario para que puedan editarlos o eliminarlos:
> > This route returns all of the personal access tokens that the authenticated user has created. This is primarily useful for listing all of the user's tokens so that they may edit or delete them:

    axios.get('/oauth/personal-access-tokens')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/personal-access-tokens`

Esta ruta crea nuevos tokens de acceso personal. Requiere dos datos: el `name` del token y los `scopes` que se deben asignar al token:
> > This route creates new personal access tokens. It requires two pieces of data: the token's `name` and the `scopes` that should be assigned to the token:

    const data = {
        name: 'Token Name',
        scopes: []
    };

    axios.post('/oauth/personal-access-tokens', data)
        .then(response => {
            console.log(response.data.accessToken);
        })
        .catch (response => {
            // List errors on response...
        });

#### `DELETE /oauth/personal-access-tokens/{token-id}`

Esta ruta se puede usar para eliminar tokens de acceso personal:
> > This route may be used to delete personal access tokens:

    axios.delete('/oauth/personal-access-tokens/' + tokenId);

<a name="protecting-routes"></a>
## Protecting Routes

<a name="via-middleware"></a>
### A través de Middleware : Via Middleware

Passport incluye un [protector de autenticación](/docs/{{version}}/authentication#adding-custom-guard) que validará los tokens de acceso en las solicitudes entrantes. Una vez que haya configurado el protector `api` para usar el controlador `passport`, solo necesita especificar el middleware `auth:api` en las rutas que requieren un token de acceso válido:
> > Passport includes an [authentication guard](/docs/{{version}}/authentication#adding-custom-guards) that will validate access tokens on incoming requests. Once you have configured the `api` guard to use the `passport` driver, you only need to specify the `auth:api` middleware on any routes that require a valid access token:

    Route::get('/user', function () {
        //
    })->middleware('auth:api');

<a name="passing-the-access-token"></a>
### Pasando el token de acceso : Passing The Access Token

Al llamar rutas que están protegidas por Passport, los consumidores API de su aplicación deben especificar su token de acceso como un token `Bearer` en el encabezado `Authorization` de su solicitud. Por ejemplo, cuando se utiliza la biblioteca HTTP Guzzle:
> > When calling routes that are protected by Passport, your application's API consumers should specify their access token as a `Bearer` token in the `Authorization` header of their request. For example, when using the Guzzle HTTP library:

    $response = $client->request('GET', '/api/user', [
        'headers' => [
            'Accept' => 'application/json',
            'Authorization' => 'Bearer '.$accessToken,
        ],
    ]);

<a name="token-scopes"></a>
## Token Scopes

<a name="defining-scopes"></a>
### Definición de ámbitos : Defining Scopes

Los ámbitos permiten a los clientes de API solicitar un conjunto específico de permisos cuando solicitan autorización para acceder a una cuenta. Por ejemplo, si está creando una aplicación de comercio electrónico, no todos los consumidores API necesitarán la posibilidad de realizar pedidos. En su lugar, puede permitir que los consumidores solo soliciten autorización para acceder a los estados de envío de pedidos. En otras palabras, los ámbitos permiten a los usuarios de su aplicación limitar las acciones que una aplicación de terceros puede realizar en su nombre.
> > Scopes allow your API clients to request a specific set of permissions when requesting authorization to access an account. For example, if you are building an e-commerce application, not all API consumers will need the ability to place orders. Instead, you may allow the consumers to only request authorization to access order shipment statuses. In other words, scopes allow your application's users to limit the actions a third-party application can perform on their behalf.

Puede definir los ámbitos de su API utilizando el método `Passport::tokensCan` en el método `boot` de su `AuthServiceProvider`. El método `tokensCan` acepta una matriz de nombres de ámbito y descripciones de ámbito. La descripción del alcance puede ser cualquier cosa que desee y se mostrará a los usuarios en la pantalla de aprobación de autorización:
> > You may define your API's scopes using the `Passport::tokensCan` method in the `boot` method of your `AuthServiceProvider`. The `tokensCan` method accepts an array of scope names and scope descriptions. The scope description may be anything you wish and will be displayed to users on the authorization approval screen:

    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

<a name="assigning-scopes-to-tokens"></a>
### Asignación de ámbitos a fichas : Assigning Scopes To Tokens

#### Al solicitar códigos de autorización : When Requesting Authorization Codes

Al solicitar un token de acceso utilizando la concesión del código de autorización, los consumidores deben especificar sus ámbitos deseados como el parámetro de cadena de consulta `scope`. El parámetro `scope` debe ser una lista de ámbitos delimitada por espacios:
> > When requesting an access token using the authorization code grant, consumers should specify their desired scopes as the `scope` query string parameter. The `scope` parameter should be a space-delimited list of scopes:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

#### Al emitir tokens de acceso personales : When Issuing Personal Access Tokens

Si está emitiendo tokens de acceso personal utilizando el método `createToken` del modelo `User`, puede pasar el array de ámbitos deseados como segundo argumento del método:
> > If you are issuing personal access tokens using the `User` model's `createToken` method, you may pass the array of desired scopes as the second argument to the method:

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="checking-scopes"></a>
### Comprobación de ámbitos : Checking Scopes

Passport incluye dos middleware que se pueden usar para verificar que una solicitud entrante se autentica con un token que ha recibido un alcance determinado. Para comenzar, agregue el siguiente middleware a la propiedad `$routeMiddleware` de su archivo `app/Http/Kernel.php`:
> > Passport includes two middleware that may be used to verify that an incoming request is authenticated with a token that has been granted a given scope. To get started, add the following middleware to the `$routeMiddleware` property of your `app/Http/Kernel.php` file:

    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,

#### Verificar todos los ámbitos : Check For All Scopes

El middleware `scopes` se puede asignar una ruta para verificar que el token de acceso de la solicitud entrante tenga *todos* los ámbitos enumerados:
> > The `scopes` middleware may be assigned to a route to verify that the incoming request's access token has *all* of the listed scopes:

    Route::get('/orders', function () {
        // Access token has both "check-status" and "place-orders" scopes...
    })->middleware('scopes:check-status,place-orders');

#### Verificar cualquier ámbito : Check For Any Scopes

El middleware `scope` se puede asignar a una ruta para verificar que el token de acceso de la solicitud entrante tenga *al menos uno* de los ámbitos listados:
> > The `scope` middleware may be assigned to a route to verify that the incoming request's access token has *at least one* of the listed scopes:

    Route::get('/orders', function () {
        // Access token has either "check-status" or "place-orders" scope...
    })->middleware('scope:check-status,place-orders');

#### Comprobación de ámbitos en una instancia de token : Checking Scopes On A Token Instance

Una vez que una solicitud autenticada de token de acceso ha ingresado a su aplicación, aún puede verificar si el token tiene un alcance determinado usando el método `tokenCan` en la instancia `User` autenticada:
> > Once an access token authenticated request has entered your application, you may still check if the token has a given scope using the `tokenCan` method on the authenticated `User` instance:

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });

<a name="consuming-your-api-with-javascript"></a>
## Consumir su API con JavaScript : Consuming Your API With JavaScript

Al construir una API, puede ser extremadamente útil poder consumir tu propia API desde tu aplicación de JavaScript. Este enfoque para el desarrollo de API le permite a su propia aplicación consumir la misma API que está compartiendo con el mundo. La misma API puede ser consumida por su aplicación web, aplicaciones móviles, aplicaciones de terceros y cualquier SDK que pueda publicar en varios administradores de paquetes.
> > When building an API, it can be extremely useful to be able to consume your own API from your JavaScript application. This approach to API development allows your own application to consume the same API that you are sharing with the world. The same API may be consumed by your web application, mobile applications, third-party applications, and any SDKs that you may publish on various package managers.

Normalmente, si desea consumir su API desde la aplicación JavaScript, deberá enviar manualmente un token de acceso a la aplicación y pasarlo con cada solicitud a su aplicación. Sin embargo, Passport incluye un middleware que puede manejar esto por usted. Todo lo que necesita hacer es agregar el middleware `CreateFreshApiToken` a su grupo de middleware `web` en su archivo `app/Http/Kernel.php`:
> > Typically, if you want to consume your API from your JavaScript application, you would need to manually send an access token to the application and pass it with each request to your application. However, Passport includes a middleware that can handle this for you. All you need to do is add the `CreateFreshApiToken` middleware to your `web` middleware group in your `app/Http/Kernel.php` file:

    'web' => [
        // Other middleware...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],

Este middleware de Passport adjuntará una cookie `laravel_token` a sus respuestas salientes. Esta cookie contiene un JWT encriptado que Passport utilizará para autenticar las solicitudes de la API desde su aplicación de JavaScript. Ahora, puede realizar solicitudes a la API de su aplicación sin pasar explícitamente un token de acceso:
> > This Passport middleware will attach a `laravel_token` cookie to your outgoing responses. This cookie contains an encrypted JWT that Passport will use to authenticate API requests from your JavaScript application. Now, you may make requests to your application's API without explicitly passing an access token:

    axios.get('/api/user')
        .then(response => {
            console.log(response.data);
        });

Al usar este método de autenticación, el andamio de Laravel JavaScript predeterminado ordena a Axios que siempre envíe los encabezados `X-CSRF-TOKEN` y `X-Requested-With`. Sin embargo, debe asegurarse de incluir su token CSRF en una [metaetiqueta HTML](/docs/{{version}}/csrf#csrf-x-csrf-token):
> > When using this method of authentication, the default Laravel JavaScript scaffolding instructs Axios to always send the `X-CSRF-TOKEN` and `X-Requested-With` headers. However, you should be sure to include your CSRF token in a [HTML meta tag](/docs/{{version}}/csrf#csrf-x-csrf-token):

    window.axios.defaults.headers.common = {
        'X-Requested-With': 'XMLHttpRequest',
    };

> {note} Si está utilizando un framework de JavaScript diferente, debe asegurarse de que esté configurado para enviar los encabezados `X-CSRF-TOKEN` y `X-Requested-With` con cada solicitud saliente.
> > > {note} If you are using a different JavaScript framework, you should make sure it is configured to send the `X-CSRF-TOKEN` and `X-Requested-With` headers with every outgoing request.

<a name="events"></a>
## Eventos : Events

Passport genera eventos al emitir tokens de acceso y tokens de actualización. Puede usar estos eventos para eliminar o revocar otros tokens de acceso en su base de datos. Puede adjuntar oyentes a estos eventos en el `EventServiceProvider` de su aplicación:
> > Passport raises events when issuing access tokens and refresh tokens. You may use these events to prune or revoke other access tokens in your database. You may attach listeners to these events in your application's `EventServiceProvider`:

```php
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'Laravel\Passport\Events\AccessTokenCreated' => [
        'App\Listeners\RevokeOldTokens',
    ],

    'Laravel\Passport\Events\RefreshTokenCreated' => [
        'App\Listeners\PruneOldTokens',
    ],
];
```

<a name="testing"></a>
## Probando : Testing

El método `actingAs` de Passport se puede usar para especificar el usuario autenticado actualmente así como sus ámbitos. El primer argumento dado al método `actingAs` es la instancia del usuario y el segundo es un array de ámbitos que debe otorgarse al token del usuario:
> > Passport's `actingAs` method may be used to specify the currently authenticated user as well as its scopes. The first argument given to the `actingAs` method is the user instance and the second is an array of scopes that should be granted to the user's token:

    public function testServerCreation()
    {
        Passport::actingAs(
            factory(User::class)->create(),
            ['create-servers']
        );

        $response = $this->post('/api/create-server');

        $response->assertStatus(200);
    }
