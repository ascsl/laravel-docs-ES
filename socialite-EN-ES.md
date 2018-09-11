# Laravel Socialite

- [Introduction](#introduction)
- [Installation](#installation)
- [Configuration](#configuration)
- [Routing](#routing)
- [Optional Parameters](#optional-parameters)
- [Access Scopes](#access-scopes)
- [Stateless Authentication](#stateless-authentication)
- [Retrieving User Details](#retrieving-user-details)

<a name="introduction"></a>
## Introducción : Introduction

Además de la autenticación típica basada en formularios, Laravel también proporciona una manera simple y conveniente de autenticarse con proveedores de OAuth utilizando [Laravel Socialite](https://github.com/laravel/socialite). Socialite actualmente admite autenticación con Facebook, Twitter, LinkedIn, Google, GitHub y Bitbucket.
> > In addition to typical, form based authentication, Laravel also provides a simple, convenient way to authenticate with OAuth providers using [Laravel Socialite](https://github.com/laravel/socialite). Socialite currently supports authentication with Facebook, Twitter, LinkedIn, Google, GitHub and Bitbucket.

> {tip} Los adaptadores para otras plataformas se enumeran en el sitio web impulsado por la comunidad [Proveedores de Socialite](https://socialiteproviders.github.io/).
> > > {tip} Adapters for other platforms are listed at the community driven [Socialite Providers](https://socialiteproviders.github.io/) website.

<a name="installation"></a>
## Instalación : Installation

Para comenzar con Socialite, use Composer para agregar el paquete a las dependencias de su proyecto:
> > To get started with Socialite, use Composer to add the package to your project's dependencies:

    composer require laravel/socialite

<a name="configuration"></a>
## Configuración : Configuration

Antes de utilizar Socialite, también deberá agregar credenciales para los servicios OAuth que utiliza su aplicación. Estas credenciales deben colocarse en su archivo de configuración `config/services.php`, y deben usar la clave `facebook`, `twitter`, `linkedin`, `google`, `github` o `bitbucket`, según los proveedores que tu aplicación requiere. Por ejemplo:
> > Before using Socialite, you will also need to add credentials for the OAuth services your application utilizes. These credentials should be placed in your `config/services.php` configuration file, and should use the key `facebook`, `twitter`, `linkedin`, `google`, `github` or `bitbucket`, depending on the providers your application requires. For example:

    'github' => [
        'client_id' => env('GITHUB_CLIENT_ID'),         // Your GitHub Client ID
        'client_secret' => env('GITHUB_CLIENT_SECRET'), // Your GitHub Client Secret
        'redirect' => 'http://your-callback-url',
    ],

> {tip} Si la opción `redirect` contiene una ruta relativa, se resolverá automáticamente en una URL completa.
> > > {tip} If the `redirect` option contains a relative path, it will automatically be resolved to a fully qualified URL.

<a name="routing"></a>
## Enrutamiento ; Routing

¡Luego, estás listo para autenticar usuarios! Necesitará dos rutas: una para redireccionar al usuario al proveedor de OAuth, y otra para recibir el callback del proveedor después de la autenticación. Accederemos a Socialite usando la fachada `Socialite`:
> > Next, you are ready to authenticate users! You will need two routes: one for redirecting the user to the OAuth provider, and another for receiving the callback from the provider after authentication. We will access Socialite using the `Socialite` facade:

    <?php

    namespace App\Http\Controllers\Auth;

    use Socialite;

    class LoginController extends Controller
    {
        /**
         * Redirect the user to the GitHub authentication page.
         *
         * @return \Illuminate\Http\Response
         */
        public function redirectToProvider()
        {
            return Socialite::driver('github')->redirect();
        }

        /**
         * Obtain the user information from GitHub.
         *
         * @return \Illuminate\Http\Response
         */
        public function handleProviderCallback()
        {
            $user = Socialite::driver('github')->user();

            // $user->token;
        }
    }

El método `redirect` se encarga de enviar al usuario al proveedor de OAuth, mientras que el método `user` leerá la solicitud entrante y recuperará la información del usuario del proveedor.
> > The `redirect` method takes care of sending the user to the OAuth provider, while the `user` method will read the incoming request and retrieve the user's information from the provider.

Por supuesto, deberá definir rutas a sus métodos de controlador:
> > Of course, you will need to define routes to your controller methods:

    Route::get('login/github', 'Auth\LoginController@redirectToProvider');
    Route::get('login/github/callback', 'Auth\LoginController@handleProviderCallback');

<a name="optional-parameters"></a>
## Parámetros opcionales : Optional Parameters

Varios proveedores de OAuth admiten parámetros opcionales en la solicitud de redireccionamiento. Para incluir cualquier parámetro opcional en la solicitud, llame al método `with` con un array asociativo:
> > A number of OAuth providers support optional parameters in the redirect request. To include any optional parameters in the request, call the `with` method with an associative array:

    return Socialite::driver('google')
        ->with(['hd' => 'example.com'])
        ->redirect();

> {note} Cuando use el método `with`, tenga cuidado de no pasar palabras clave reservadas como `state` o `response_type`.
> > > {note} When using the `with` method, be careful not to pass any reserved keywords such as `state` or `response_type`.

<a name="access-scopes"></a>
## Access Scopes

Antes de redirigir al usuario, también puede agregar "ámbitos" adicionales en la solicitud utilizando el método `scopes`. Este método fusionará todos los ámbitos existentes con los que suministra:
> > Before redirecting the user, you may also add additional "scopes" on the request using the `scopes` method. This method will merge all existing scopes with the ones you supply:

    return Socialite::driver('github')
        ->scopes(['read:user', 'public_repo'])
        ->redirect();

Puede sobrescribir todos los ámbitos existentes utilizando el método `setScopes`:
> > You can overwrite all exisiting scopes using the `setScopes` method:

    return Socialite::driver('github')
        ->setScopes(['read:user', 'public_repo'])
        ->redirect();

<a name="stateless-authentication"></a>
## Autenticación sin estado : Stateless Authentication

El método `stateless` se puede usar para deshabilitar la verificación del estado de la sesión. Esto es útil al agregar autenticación social a una API:
> > The `stateless` method may be used to disable session state verification. This is useful when adding social authentication to an API:

    return Socialite::driver('google')->stateless()->user();

<a name="retrieving-user-details"></a>
## Recuperar detalles del usuario : Retrieving User Details

Una vez que tenga una instancia de usuario, puede obtener algunos detalles más sobre el usuario:
> > Once you have a user instance, you can grab a few more details about the user:

    $user = Socialite::driver('github')->user();

    // OAuth Two Providers
    $token = $user->token;
    $refreshToken = $user->refreshToken; // not always provided
    $expiresIn = $user->expiresIn;

    // OAuth One Providers
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // All Providers
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();

#### Recuperación de detalles de usuario de un token (OAuth2) : Retrieving User Details From A Token (OAuth2)

Si ya tiene un token de acceso válido para un usuario, puede recuperar sus detalles utilizando el método `userFromToken`:
> > If you already have a valid access token for a user, you can retrieve their details using the `userFromToken` method:

    $user = Socialite::driver('github')->userFromToken($token);
    
#### Recuperación de detalles de usuario de un token y un secreto (OAuth1) : Retrieving User Details From A Token And Secret (OAuth1)

Si ya tiene un par de tokens / secreto válido para un usuario, puede recuperar sus detalles utilizando el método `userFromTokenAndSecret`:
> > If you already have a valid pair of token / secret for a user, you can retrieve their details using the `userFromTokenAndSecret` method:

    $user = Socialite::driver('twitter')->userFromTokenAndSecret($token, $secret);
