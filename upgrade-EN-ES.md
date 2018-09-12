# Guía de actualización : Upgrade Guide

- [Upgrading To 5.7.0 From 5.6](#upgrade-5.7.0)

<a name="upgrade-5.7.0"></a>
## Actualizando a 5.7.0 desde 5.6 : Upgrading To 5.7.0 From 5.6

#### Tiempo estimado de actualización: 10 - 15 minutos : Estimated Upgrade Time: 10 - 15 Minutes

> {note} Intentamos documentar cada posible cambio de ruptura. Dado que algunos de estos cambios importantes se encuentran en partes oscuras del framework, solo una parte de estos cambios puede afectar su aplicación.
> > > {note} We attempt to document every possible breaking change. Since some of these breaking changes are in obscure parts of the framework only a portion of these changes may actually affect your application.

### Actualización de dependencias : Updating Dependencies

Actualice su dependencia `laravel/framework` a `5.7.*` en su archivo `composer.json`.
> > Update your `laravel/framework` dependency to `5.7.*` in your `composer.json` file.

Por supuesto, no olvide examinar los paquetes de terceros consumidos por su aplicación y verificar que está utilizando la versión adecuada para la compatibilidad con Laravel 5.7.
> > Of course, don't forget to examine any 3rd party packages consumed by your application and verify you are using the proper version for Laravel 5.7 support.

### Aplicació : Application

#### El método 'register' : The `register` Method

**Likelihood Of Impact: Very Low**

El argumento `options` no utilizado del método `registration` de la clase `Illuminate\Foundation\Application` ha sido eliminado. Si está anulando este método, debe actualizar la firma de su método:
> > The unused `options` argument of the `Illuminate\Foundation\Application` class' `register` method has been removed. If you are overriding this method, you should update your method's signature:

    /**
     * Register a service provider with the application.
     *
     * @param  \Illuminate\Support\ServiceProvider|string  $provider
     * @param  bool   $force
     * @return \Illuminate\Support\ServiceProvider
     */
    public function register($provider, $force = false);

### Artisan

#### Conexión y colas de trabajos programadas : Scheduled Job Connection & Queues

**Likelihood Of Impact: Low**

El método `$schedule->job` ahora respeta las propiedades `queue` y `connection` en la clase de trabajo si una conexión / tarea no se pasa explícitamente al método `job`.
> > The `$schedule->job` method now respects the `queue` and `connection` properties on the job class if a connection / job is not explicitly passed into the `job` method.

En general, esto se debe considerar una corrección de errores; sin embargo, aparece como un cambio de rotura por precaución. [Infórmenos si encuentra algún problema relacionado con este cambio](https://github.com/laravel/framework/pull/25216).
> > Generally, this should be considered a bug fix; however, it is listed as a breaking change out of caution. [Please let us know if you encounter any issues surrounding this change](https://github.com/laravel/framework/pull/25216).

### Autenticación : Authentication

#### El middleware `Authenticate` : The `Authenticate` Middleware

**Likelihood Of Impact: Low**

El método `authenticate` del middleware `Illuminate\Auth\Middleware\Authenticate` se ha actualizado para aceptar el `$request` entrante como primer argumento. Si está reemplazando este método en su propio middleware `Authenticate`, debe actualizar la firma de su middleware:
> > The `authenticate` method of the `Illuminate\Auth\Middleware\Authenticate` middleware has been updated to accept the incoming `$request` as its first argument. If you are overriding this method in your own `Authenticate` middleware, you should update your middleware's signature:

    /**
     * Determine if the user is logged in to any of the given guards.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  array  $guards
     * @return void
     *
     * @throws \Illuminate\Auth\AuthenticationException
     */
    protected function authenticate($request, array $guards)

#### El rasgo `ResetsPasswords` : The `ResetsPasswords` Trait

**Likelihood Of Impact: Low**

El método protegido `sendResetResponse` del rasgo `ResetsPasswords` ahora acepta la entrada `Illuminate\Http\Request` como primer argumento. Si está anulando este método, debe actualizar la firma de su método:
> > The protected `sendResetResponse` method of the `ResetsPasswords` trait now accepts the incoming `Illuminate\Http\Request` as its first argument. If you are overriding this method, you should update your method's signature:

    /**
     * Get the response for a successful password reset.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  string  $response
     * @return \Illuminate\Http\RedirectResponse|\Illuminate\Http\JsonResponse
     */
    protected function sendResetResponse(Request $request, $response)

#### El rasgo `SendsPasswordResetEmails` : The `SendsPasswordResetEmails` Trait

**Likelihood Of Impact: Low**

El método protegido `sendResetLinkResponse` del rasgo `SendsPasswordResetEmails` ahora acepta la entrada `Illuminate\Http\Request` como primer argumento. Si está anulando este método, debe actualizar la firma de su método:
> > The protected `sendResetLinkResponse` method of the `SendsPasswordResetEmails` trait now accepts the incoming `Illuminate\Http\Request` as its first argument. If you are overriding this method, you should update your method's signature:

    /**
     * Get the response for a successful password reset link.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  string  $response
     * @return \Illuminate\Http\RedirectResponse|\Illuminate\Http\JsonResponse
     */
    protected function sendResetLinkResponse(Request $request, $response)

### Autorización : Authorization

#### El contrato `Gate` : The `Gate` Contract

**Likelihood Of Impact: Very Low**

El método `raw` fue cambiado de visibilidad `protected` a `public`. Además, se [agregó al contrato `Illuminate/Contracts/Auth/Access/Gate`](https://github.com/laravel/framework/pull/25143):
> > The `raw` was changed from `protected` to `public` visibility. In addition, it [was added to the `Illuminate/Contracts/Auth/Access/Gate` contract](https://github.com/laravel/framework/pull/25143):

    /**
     * Get the raw result from the authorization callback.
     *
     * @param  string  $ability
     * @param  array|mixed  $arguments
     * @return mixed
     */
    public function raw($ability, $arguments = []);

Si está implementando esta interfaz, debe agregar este método a su implementación.
> > If you are implementing this interface, you should add this method to your implementation.

### Blade

#### El operador `or` : The `or` Operator

**Likelihood Of Impact: High**

El operador Blade "or" ha sido eliminado en favor del operador `??` "nulo de fusión" incorporado de PHP, que tiene el mismo propósito y funcionalidad:
> > The Blade "or" operator has been removed in favor of PHP's built-in `??` "null coalesce" operator, which has the same purpose and functionality:

    // Laravel 5.6...
    {{ $foo or 'default' }}

    // Laravel 5.7...
    {{ $foo ?? 'default' }}

### Carbon

**Likelihood Of Impact: Very Low**

Las "macros" de Carbon ahora son manejadas directamente por la librería de Carbon en lugar de la extensión de la librería de Laravel. No esperamos que esto rompa su código; sin embargo, [infórmenos de cualquier problema que encuentre relacionado con este cambio](https://github.com/laravel/framework/pull/23938).
> > Carbon "macros" are now handled by the Carbon library directly instead of Laravel's extension of the library. We do not expect this to break your code; however, [please make us aware of any problems you encounter related to this change](https://github.com/laravel/framework/pull/23938).

### Colecciones : Collections

#### El método 'split' : The `split` Method

**Likelihood Of Impact: Low**

El método `split` [se ha actualizado para que siempre devuelva el número solicitado de "grupos"](https://github.com/laravel/framework/pull/24088), a menos que el número total de elementos en la colección original sea menor que el recuento de la colección solicitada. En general, esto se debe considerar una corrección de errores; sin embargo, aparece como un cambio de rotura por precaución.
> > The `split` method [has been updated to always return the requested number of "groups"](https://github.com/laravel/framework/pull/24088), unless the total number of items in the original collection is less than the requested collection count. Generally, this should be considered a bug fix; however, it is listed as a breaking change out of caution.

### Galleta : Cookie

#### `Factory` Contract Method Signature

**Likelihood Of Impact: Very Low**

Las firmas de los métodos `make` y `forever` de la interfaz `Illuminate/Contracts/Cookie/Factory` [se han cambiado](https://github.com/laravel/framework/pull/23200). Si está implementando esta interfaz, debe actualizar estos métodos en su implementación.
> > The signatures of the `make` and `forever` methods of the `Illuminate/Contracts/Cookie/Factory` interface [have been changed](https://github.com/laravel/framework/pull/23200). If you are implementing this interface, you should update these methods in your implementation.

### Base de datos : Database

#### El método de migración `softDeletesTz` : The `softDeletesTz` Migration Method

**Likelihood Of Impact: Low**

El método `softDeletesTz` del constructor de tablas de esquema ahora acepta el nombre de columna como primer argumento, mientras que `$precision` se ha movido a la posición del segundo argumento:
> > The schema table builder's `softDeletesTz` method now accepts the column name as its first argument, while the `$precision` has been moved to the second argument position:

    /**
     * Add a "deleted at" timestampTz for the table.
     *
     * @param  string  $column
     * @param  int  $precision
     * @return \Illuminate\Support\Fluent
     */
    public function softDeletesTz($column = 'deleted_at', $precision = 0)

#### El contrato `ConnectionInterface` : The `ConnectionInterface` Contract

**Likelihood Of Impact: Very Low**

Las firmas del método `select` y `selectOne` del contrato `Illuminate\Contracts\Database\ConnectionInterface` se han actualizado para acomodar el nuevo argumento `$useReadPdo`:
> > The `Illuminate\Contracts\Database\ConnectionInterface` contract's `select` and `selectOne` method signatures have been updated to accommodate the new `$useReadPdo` argument:

    /**
     * Run a select statement and return a single result.
     *
     * @param  string  $query
     * @param  array   $bindings
     * @param  bool  $useReadPdo
     * @return mixed
     */
    public function selectOne($query, $bindings = [], $useReadPdo = true);

    /**
     * Run a select statement against the database.
     *
     * @param  string  $query
     * @param  array   $bindings
     * @param  bool  $useReadPdo
     * @return array
     */
    public function select($query, $bindings = [], $useReadPdo = true);

Además, el método `cursor` se agregó al contrato:
> > In addition, the `cursor` method was added to the contract:

    /**
     * Run a select statement against the database and returns a generator.
     *
     * @param  string  $query
     * @param  array  $bindings
     * @param  bool  $useReadPdo
     * @return \Generator
     */
    public function cursor($query, $bindings = [], $useReadPdo = true);

Si está implementando esta interfaz, debe agregar este método a su implementación.
> > If you are implementing this interface, you should add this method to your implementation.

#### Prioridad del driver SQL Server : SQL Server Driver Priority

**Likelihood Of Impact: Low**

Antes de Laravel 5.7, el controlador `PDO_DBLIB` se usaba como el controlador PDO de SQL Server predeterminado. Este controlador se considera obsoleto por Microsoft. A partir de Laravel 5.7, `PDO_SQLSRV` se usará como el controlador predeterminado si está disponible. Alternativamente, puede optar por usar el controlador `PDO_ODBC`:
> > Prior to Laravel 5.7, the `PDO_DBLIB` driver was used as the default SQL Server PDO driver. This driver is considered deprecated by Microsoft. As of Laravel 5.7, `PDO_SQLSRV` will be used as the default driver if it is available. Alternatively, you may choose to use the `PDO_ODBC` driver:

    'sqlsrv' => [
        // ...
        'odbc' => true,
        'odbc_datasource_name' => 'your-odbc-dsn',
    ],

Si ninguno de estos controladores está disponible, Laravel utilizará el controlador `PDO_DBLIB`.
> > If neither of these drivers are available, Laravel will use the `PDO_DBLIB` driver.

### Depurar : Debug

#### Clases de volcado : Dumper Classes

**Likelihood Of Impact: Very Low**

Las clases `Illuminate\Support\Debug\Dumper` y `Illuminate\Support\Debug\HtmlDumper` han sido eliminadas a favor del uso de volcadoras de variables nativas de Symfony: `Symfony\Component\VarDumper\VarDumper` y `Symfony\Component\VarDumper\Dumper\HtmlDumper`.
> > The `Illuminate\Support\Debug\Dumper` and `Illuminate\Support\Debug\HtmlDumper` classes have been removed in favor of using Symfony's native variable dumpers: `Symfony\Component\VarDumper\VarDumper` and `Symfony\Component\VarDumper\Dumper\HtmlDumper`.

### Eloquent

#### Los métodos `latest` / `oldest` : The `latest` / `oldest` Methods

**Likelihood Of Impact: Low**

Los métodos `latest` y `oldest` del generador de consultas Eloquent se han actualizado para respetar las columnas "timestamp" personalizadas "creadas en" que se pueden especificar en sus modelos Eloquent. En general, esto se debe considerar una corrección de errores; sin embargo, aparece como un cambio de rotura por precaución.
> > The Eloquent query builder's `latest` and `oldest` methods have been updated to respect custom "created at" timestamp columns that may be specified on your Eloquent models. Generally, this should be considered a bug fix; however, it is listed as a breaking change out of caution.

#### El método `wasChanged` : The `wasChanged` Method

**Likelihood Of Impact: Very Low**

Los cambios de un modelo Eloquent ahora están disponibles para el método `wasChanged` **antes de** activar el evento del modelo `updated`. En general, esto se debe considerar una corrección de errores; sin embargo, aparece como un cambio de rotura por precaución. [Por favor, háganos saber si encuentra algún problema relacionado con este cambio](https://github.com/laravel/framework/pull/25026).
> > An Eloquent model's changes are now available to the `wasChanged` method **before** firing the `updated` model event. Generally, this should be considered a bug fix; however, it is listed as a breaking change out of caution. [Please let us know if you encounter any issues surrounding this change](https://github.com/laravel/framework/pull/25026).

#### Valores flotantes especiales de PostgreSQL : PostgreSQL Special Float Values

**Likelihood Of Impact: Low**

PostgreSQL admite los valores de flotación `Infinity`, `-Infinity` y `NaN`. Antes de Laravel 5.7, estos se convertían en `0` cuando el tipo de conversión Eloquent para la columna era `float`, `double` o `real`.
> > PostgreSQL supports the float values `Infinity`, `-Infinity` and `NaN`. Prior to Laravel 5.7, these were cast to `0` when the Eloquent casting type for the column was `float`, `double`, or `real`.

A partir de Laravel 5.7, estos valores se convertirán en las constantes PHP correspondientes `INF`,` -INF` y `NAN`.
> > As of Laravel 5.7, these values will be cast to the corresponding PHP constants `INF`, `-INF`, and `NAN`.

### Verificación de email : Email Verification

**Likelihood Of Impact: Optional**

Si opta por utilizar los nuevos [servicios de verificación de correo electrónico](/docs/{{version}}/verification) de Laravel, deberá agregar andamios adicionales a su aplicación. Primero, agregue el `VerificationController` a su aplicación: [App\Http\Controllers\Auth\VerificationController](https://github.com/laravel/laravel/blob/develop/app/Http/Controllers/Auth/VerificationController.php )
> > If you choose to use Laravel's new [email verification services](/docs/{{version}}/verification), you will need to add additional scaffolding to your application. First, add the `VerificationController` to your application: [App\Http\Controllers\Auth\VerificationController](https://github.com/laravel/laravel/blob/develop/app/Http/Controllers/Auth/VerificationController.php).

También necesitará el talón de la vista de verificación. Esta vista debe colocarse en `resources/views/auth/verify.blade.php`. Puede obtener los contenidos de la vista [en GitHub](https://github.com/laravel/framework/blob/5.7/src/Illuminate/Auth/Console/stubs/make/views/auth/verify.stub).
> > You will also need the verification view stub. This view should be placed at `resources/views/auth/verify.blade.php`. You may obtain the view's contents [on GitHub](https://github.com/laravel/framework/blob/5.7/src/Illuminate/Auth/Console/stubs/make/views/auth/verify.stub).

Finalmente, cuando llamas al método `Auth::routes`, debes pasar la opción `verify` al método:
> > Finally, when calling the `Auth::routes` method, you should pass the `verify` option to the method:

    Auth::routes(['verify' => true]);

### Filesystem

#### `Filesystem` Contract Methods

**Likelihood Of Impact: Low**

Los métodos `readStream` y `writeStream` [se han agregado al contrato `Illuminate\Contracts\Filesystem\Filesystem`](https://github.com/laravel/framework/pull/23755). Si está implementando esta interfaz, debe agregar estos métodos a su implementación.
> > The `readStream` and `writeStream` methods [have been added to the `Illuminate\Contracts\Filesystem\Filesystem` contract](https://github.com/laravel/framework/pull/23755). If you are implementing this interface, you should add these methods to your implementation.

### Mail

#### Mailable Dynamic Variable Casing

**Likelihood Of Impact: Medium**

Las variables que se pasan dinámicamente a vistas mailable [ahora son automáticamente "camel caseadas"](https://github.com/laravel/framework/pull/24232), lo que hace que el comportamiento de la variable dinámica mailable sea consistente con las variables de vista dinámica. Las variables de mailable dinámico no son una característica de Laravel documentada, por lo que la probabilidad de impacto para su aplicación es baja.
> > Variables that are dynamically passed to mailable views [are now automatically "camel cased"](https://github.com/laravel/framework/pull/24232), which makes mailable dynamic variable behavior consistent with dynamic view variables. Dynamic mailable variables are not a documented Laravel feature, so likelihood of impact to your application is low.

### Enrutamiento : Routing

#### El método `Route::redirect` : The `Route::redirect` Method

**Likelihood Of Impact: High**

El método `Route::redirect` ahora devuelve una redirección de código de estado HTTP `302`. El método `permanentRedirect` se ha agregado para permitir las redirecciones `301`.
> > The `Route::redirect` method now returns a `302` HTTP status code redirect. The `permanentRedirect` method has been added to allow `301` redirects.

    // Return a 302 redirect...
    Route::redirect('/foo', '/bar');

    // Return a 301 redirect...
    Route::redirect('/foo', '/bar', 301);

    // Return a 301 redirect...
    Route::permanentRedirect('/foo', '/bar');

#### El método `addRoute` : The `addRoute` Method

**Likelihood Of Impact: Low**

El método `addRoute` de la clase `Illuminate\Routing\Router` se ha cambiado de `protected` a `public`.
> > The `addRoute` method of the `Illuminate\Routing\Router` class has been changed from `protected` to `public`.

### Validación : Validation

#### Datos de validación anidados : Nested Validation Data

**Likelihood Of Impact: Medium**

En versiones anteriores de Laravel, el método `validate` no devolvía los datos correctos para las reglas de validación anidadas. Esto ha sido corregido en Laravel 5.7:
> > In previous versions of Laravel, the `validate` method did not return the correct data for nested validation rules. This has been corrected in Laravel 5.7:

    $data = Validator::make([
        'person' => [
            'name' => 'Taylor',
            'job' => 'Developer'
        ]
    ], ['person.name' => 'required'])->validate();

    dump($data);

    // Prior Behavior...
    ['person' => ['name' => 'Taylor', 'job' => 'Developer']]

    // New Behavior...
    ['person' => ['name' => 'Taylor']]

### Varios : Miscellaneous

También le recomendamos que vea los cambios en `laravel/laravel` [repositorio de GitHub](https://github.com/laravel/laravel). Si bien muchos de estos cambios no son necesarios, es posible que desee mantener estos archivos sincronizados con su aplicación. Algunos de estos cambios se cubrirán en esta guía de actualización, pero otros, como cambios en los archivos de configuración o comentarios, no lo serán. Puede ver fácilmente los cambios con la [herramienta de comparación GitHub](https://github.com/laravel/laravel/compare/5.6...master) y elegir qué actualizaciones son importantes para usted.
> > We also encourage you to view the changes in the `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). While many of these changes are not required, you may wish to keep these files in sync with your application. Some of these changes will be covered in this upgrade guide, but others, such as changes to configuration files or comments, will not be. You can easily view the changes with the [GitHub comparison tool](https://github.com/laravel/laravel/compare/5.6...master) and choose which updates are important to you.
