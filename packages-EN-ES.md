# Desarrollo de paquetes : Package Development

- [Introduction](#introduction)
    - [A Note On Facades](#a-note-on-facades)
- [Package Discovery](#package-discovery)
- [Service Providers](#service-providers)
- [Resources](#resources)
    - [Configuration](#configuration)
    - [Migrations](#migrations)
    - [Routes](#routes)
    - [Translations](#translations)
    - [Views](#views)
- [Commands](#commands)
- [Public Assets](#public-assets)
- [Publishing File Groups](#publishing-file-groups)

<a name="introduction"></a>
## Introducción : Introduction

Los paquetes son la principal forma de agregar funcionalidad a Laravel. Los paquetes pueden ser cualquier cosa, desde una gran forma de trabajar con fechas como [Carbon](https://github.com/briannesbitt/Carbon), o todo un framework de prueba de BDD como [Behat](https://github.com/Behat)/Behat).
> > Packages are the primary way of adding functionality to Laravel. Packages might be anything from a great way to work with dates like [Carbon](https://github.com/briannesbitt/Carbon), or an entire BDD testing framework like [Behat](https://github.com/Behat/Behat).

Por supuesto, hay diferentes tipos de paquetes. Algunos paquetes son independientes, lo que significa que funcionan con cualquier marco PHP. Carbon y Behat son ejemplos de paquetes independientes. Cualquiera de estos paquetes se puede usar con Laravel solicitándolos en su archivo `composer.json`.
> > Of course, there are different types of packages. Some packages are stand-alone, meaning they work with any PHP framework. Carbon and Behat are examples of stand-alone packages. Any of these packages may be used with Laravel by requesting them in your `composer.json` file.

Por otro lado, otros paquetes están específicamente destinados para su uso con Laravel. Estos paquetes pueden tener rutas, controladores, vistas y configuraciones específicamente diseñadas para mejorar una aplicación Laravel. Esta guía cubre principalmente el desarrollo de aquellos paquetes que son específicos de Laravel.
> > On the other hand, other packages are specifically intended for use with Laravel. These packages may have routes, controllers, views, and configuration specifically intended to enhance a Laravel application. This guide primarily covers the development of those packages that are Laravel specific.

<a name="a-note-on-facades"></a>
### Una nota sobre fachadas : A Note On Facades

Al escribir una aplicación Laravel, generalmente no importa si usa contratos o fachadas ya que ambos brindan niveles esencialmente iguales de capacidad de prueba. Sin embargo, al escribir paquetes, su paquete normalmente no tendrá acceso a todos los asistentes de prueba de Laravel. Si desea poder escribir sus pruebas de paquete como si existieran dentro de una aplicación Laravel típica, puede usar el paquete [Orchestral Testbench](https://github.com/orchestral/testbench).
> > When writing a Laravel application, it generally does not matter if you use contracts or facades since both provide essentially equal levels of testability. However, when writing packages, your package will not typically have access to all of Laravel's testing helpers. If you would like to be able to write your package tests as if they existed inside a typical Laravel application, you may use the [Orchestral Testbench](https://github.com/orchestral/testbench) package.

<a name="package-discovery"></a>
## Package Discovery

En el archivo de configuración `config/app.php` de la aplicación Laravel, la opción `providers` define una lista de proveedores de servicios que Laravel debe cargar. Cuando alguien instala su paquete, normalmente querrá que su proveedor de servicio se incluya en esta lista. En lugar de requerir que los usuarios agreguen manualmente su proveedor de servicios a la lista, puede definir el proveedor en la sección `extra` del archivo `composer.json` de su paquete. Además de los proveedores de servicios, también puede enumerar las [fachadas](/docs/{{version}}/facades) que desea registrar:
> > In a Laravel application's `config/app.php` configuration file, the `providers` option defines a list of service providers that should be loaded by Laravel. When someone installs your package, you will typically want your service provider to be included in this list. Instead of requiring users to manually add your service provider to the list, you may define the provider in the `extra` section of your package's `composer.json` file. In addition to service providers, you may also list any [facades](/docs/{{version}}/facades) you would like to be registered:

    "extra": {
        "laravel": {
            "providers": [
                "Barryvdh\\Debugbar\\ServiceProvider"
            ],
            "aliases": {
                "Debugbar": "Barryvdh\\Debugbar\\Facade"
            }
        }
    },

Una vez que su paquete se haya configurado para su descubrimiento, Laravel registrará automáticamente sus proveedores de servicios y fachadas cuando esté instalado, creando una buena experiencia de instalación para los usuarios de su paquete.
> > Once your package has been configured for discovery, Laravel will automatically register its service providers and facades when it is installed, creating a convenient installation experience for your package's users.

### Opción de exclusión del descubrimiento del paquete : Opting Out Of Package Discovery

Si usted es el consumidor de un paquete y desea deshabilitar el descubrimiento de paquetes para un paquete, puede incluir el nombre del paquete en la sección `extra` del archivo `composer.json` de su aplicación:
> > If you are the consumer of a package and would like to disable package discovery for a package, you may list the package name in the `extra` section of your application's `composer.json` file:

    "extra": {
        "laravel": {
            "dont-discover": [
                "barryvdh/laravel-debugbar"
            ]
        }
    },

Puede deshabilitar el descubrimiento de paquetes para todos los paquetes que usan el carácter `*` dentro de la directiva `dont-discover` de su aplicación:
> > You may disable package discovery for all packages using the `*` character inside of your application's `dont-discover` directive:

    "extra": {
        "laravel": {
            "dont-discover": [
                "*"
            ]
        }
    },

<a name="service-providers"></a>
## Proveedores de servicio : Service Providers

[Proveedores de servicios](/docs/{{version}}/providers) son los puntos de conexión entre su paquete y Laravel. Un proveedor de servicios es responsable de vincular las cosas en el [contenedor de servicios](/docs/{{version}}/container) de Laravel e informar a Laravel dónde cargar los recursos del paquete, como vistas, configuración y archivos de localización.
> > [Service providers](/docs/{{version}}/providers) are the connection points between your package and Laravel. A service provider is responsible for binding things into Laravel's [service container](/docs/{{version}}/container) and informing Laravel where to load package resources such as views, configuration, and localization files.

Un proveedor de servicios hereda la clase `Illuminate\Support\ServiceProvider` y contiene dos métodos: `register` y `boot`. La clase `ServiceProvider` base se encuentra en el paquete Composer `illuminate/support`, que debe agregar a las dependencias de su propio paquete. Para obtener más información sobre la estructura y el propósito de los proveedores de servicios, consulte [su documentación](/docs/{{version}}/providers).
> > A service provider extends the `Illuminate\Support\ServiceProvider` class and contains two methods: `register` and `boot`. The base `ServiceProvider` class is located in the `illuminate/support` Composer package, which you should add to your own package's dependencies. To learn more about the structure and purpose of service providers, check out [their documentation](/docs/{{version}}/providers).

<a name="resources"></a>
## Recursos : Resources

<a name="configuration"></a>
### Configuración : Configuration

Por lo general, deberá publicar el archivo de configuración de su paquete en el propio directorio `config` de la aplicación. Esto permitirá a los usuarios de su paquete anular fácilmente sus opciones de configuración predeterminadas. Para permitir que se publiquen sus archivos de configuración, llame al método `publishes` desde el método `boot` de su proveedor de servicios:
> > Typically, you will need to publish your package's configuration file to the application's own `config` directory. This will allow users of your package to easily override your default configuration options. To allow your configuration files to be published, call the `publishes` method from the `boot` method of your service provider:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
        ]);
    }

Ahora, cuando los usuarios de su paquete ejecutan el comando `vendor:publish` de Laravel, su archivo se copiará a la ubicación de publicación especificada. Por supuesto, una vez que se haya publicado su configuración, se podrá acceder a sus valores como cualquier otro archivo de configuración:
> > Now, when users of your package execute Laravel's `vendor:publish` command, your file will be copied to the specified publish location. Of course, once your configuration has been published, its values may be accessed like any other configuration file:

    $value = config('courier.option');

> {note} No debe definir Closures en sus archivos de configuración. No se pueden serializar correctamente cuando los usuarios ejecutan el comando Artisan `config:cache`.
> > > {note} You should not define Closures in your configuration files. They can not be serialized correctly when users execute the `config:cache` Artisan command.

#### Configuración predeterminada del paquete : Default Package Configuration

También puede fusionar su propio archivo de configuración de paquete con la copia publicada de la aplicación. Esto permitirá que los usuarios definan solo las opciones que realmente desean anular en la copia publicada de la configuración. Para fusionar las configuraciones, use el método `mergeConfigFrom` dentro del método `register` de su proveedor de servicios:
> > You may also merge your own package configuration file with the application's published copy. This will allow your users to define only the options they actually want to override in the published copy of the configuration. To merge the configurations, use the `mergeConfigFrom` method within your service provider's `register` method:

    /**
     * Register bindings in the container.
     *
     * @return void
     */
    public function register()
    {
        $this->mergeConfigFrom(
            __DIR__.'/path/to/config/courier.php', 'courier'
        );
    }

> {note} Este método solo combina el primer nivel del array de configuración. Si los usuarios definen parcialmente un array de configuración multidimensional, las opciones faltantes no se fusionarán.
> > > {note} This method only merges the first level of the configuration array. If your users partially define a multi-dimensional configuration array, the missing options will not be merged.

<a name="routes"></a>
### Rutas : Routes

Si su paquete contiene rutas, puede cargarlas usando el método `loadRoutesFrom`. Este método determinará automáticamente si las rutas de la aplicación se almacenan en caché y no cargarán el archivo de rutas si las rutas ya se han almacenado en caché:
> > If your package contains routes, you may load them using the `loadRoutesFrom` method. This method will automatically determine if the application's routes are cached and will not load your routes file if the routes have already been cached:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadRoutesFrom(__DIR__.'/routes.php');
    }

<a name="migrations"></a>
### Migraciones : Migrations

Si su paquete contiene [migraciones de base de datos](/docs/{{version}}/migrations), puede usar el método `loadMigrationsFrom` para informarle a Laravel cómo cargarlas. El método `loadMigrationsFrom` acepta la ruta a las migraciones de su paquete como único argumento:
> > If your package contains [database migrations](/docs/{{version}}/migrations), you may use the `loadMigrationsFrom` method to inform Laravel how to load them. The `loadMigrationsFrom` method accepts the path to your package's migrations as its only argument:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadMigrationsFrom(__DIR__.'/path/to/migrations');
    }

Una vez que se hayan registrado las migraciones de su paquete, se ejecutarán automáticamente cuando se ejecute el comando `php artisan migrate`. No necesita exportarlos al directorio principal de `database/migrations` de la aplicación.
> > Once your package's migrations have been registered, they will automatically be run when the `php artisan migrate` command is executed. You do not need to export them to the application's main `database/migrations` directory.

<a name="translations"></a>
### Traducciones : Translations

Si su paquete contiene [archivos de traducción](/docs/{{version}}/localization), puede usar el método `loadTranslationsFrom` para informarle a Laravel cómo cargarlos. Por ejemplo, si su paquete se llama `courier`, debe agregar lo siguiente al método `boot` de su proveedor de servicios:
> > If your package contains [translation files](/docs/{{version}}/localization), you may use the `loadTranslationsFrom` method to inform Laravel how to load them. For example, if your package is named `courier`, you should add the following to your service provider's `boot` method:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
    }

Las traducciones de paquetes se referencian usando la convención de sintaxis `package::file.line`. Por lo tanto, puede cargar la línea `welcome` del paquete `courier` del archivo `messages` de la siguiente manera:
> > Package translations are referenced using the `package::file.line` syntax convention. So, you may load the `courier` package's `welcome` line from the `messages` file like so:

    echo trans('courier::messages.welcome');

#### Publicación de traducciones : Publishing Translations

Si desea publicar las traducciones de su paquete en el directorio `resources/lang/vendor` de la aplicación, puede usar el método `publishes` del proveedor de servicios. El método `publishes` acepta un array de rutas de paquetes y sus ubicaciones de publicación deseadas. Por ejemplo, para publicar los archivos de traducción para el paquete `courier`, puede hacer lo siguiente:
> > If you would like to publish your package's translations to the application's `resources/lang/vendor` directory, you may use the service provider's `publishes` method. The `publishes` method accepts an array of package paths and their desired publish locations. For example, to publish the translation files for the `courier` package, you may do the following:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');

        $this->publishes([
            __DIR__.'/path/to/translations' => resource_path('lang/vendor/courier'),
        ]);
    }

Ahora, cuando los usuarios de su paquete ejecutan el comando Artisan `vendor:publish` de Laravel, las traducciones de su paquete se publicarán en la ubicación de publicación especificada.
> > Now, when users of your package execute Laravel's `vendor:publish` Artisan command, your package's translations will be published to the specified publish location.

<a name="views"></a>
### Vistas : Views

Para registrar las [vistas](/docs/{{version}}/views) de su paquete con Laravel, necesita decirle a Laravel dónde están ubicadas las vistas. Puede hacerlo utilizando el método `loadViewsFrom` del proveedor de servicios. El método `loadViewsFrom` acepta dos argumentos: la ruta a sus plantillas de vista y el nombre de su paquete. Por ejemplo, si el nombre de su paquete es `courier`, debe agregar lo siguiente al método `boot` de su proveedor de servicios:
> > To register your package's [views](/docs/{{version}}/views) with Laravel, you need to tell Laravel where the views are located. You may do this using the service provider's `loadViewsFrom` method. The `loadViewsFrom` method accepts two arguments: the path to your view templates and your package's name. For example, if your package's name is `courier`, you would add the following to your service provider's `boot` method:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
    }

Las vistas de paquete se referencian usando la convención de sintaxis `package::view`. Entonces, una vez que su ruta de vista se registra en un proveedor de servicios, puede cargar la vista `admin` del paquete `courier` de la siguiente forma:
> > Package views are referenced using the `package::view` syntax convention. So, once your view path is registered in a service provider, you may load the `admin` view from the `courier` package like so:

    Route::get('admin', function () {
        return view('courier::admin');
    });

#### Overriding Package Views 

Cuando utiliza el método `loadViewsFrom`, Laravel en realidad registra dos ubicaciones para sus vistas: el directorio `resources/views/vendor` de la aplicación y el directorio que usted especifique. Entonces, usando el ejemplo `courier`, Laravel primero comprobará si el desarrollador ha proporcionado una versión personalizada de la vista en `resources/views/vendor/courier`. Entonces, si la vista no se ha personalizado, Laravel buscará en el directorio de vista del paquete que especificó en su llamada a `loadViewsFrom`. Esto facilita a los usuarios del paquete personalizar / anular las vistas de su paquete.
> > When you use the `loadViewsFrom` method, Laravel actually registers two locations for your views: the application's `resources/views/vendor` directory and the directory you specify. So, using the `courier` example, Laravel will first check if a custom version of the view has been provided by the developer in `resources/views/vendor/courier`. Then, if the view has not been customized, Laravel will search the package view directory you specified in your call to `loadViewsFrom`. This makes it easy for package users to customize / override your package's views.

#### Publicación de vistas : Publishing Views

Si desea que sus vistas estén disponibles para su publicación en el directorio `resources/views/vendor` de la aplicación, puede usar el método `publishes` del proveedor de servicios. El método `publishes` acepta un array de rutas de vista de paquete y sus ubicaciones de publicación deseadas:
> > If you would like to make your views available for publishing to the application's `resources/views/vendor` directory, you may use the service provider's `publishes` method. The `publishes` method accepts an array of package view paths and their desired publish locations:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

        $this->publishes([
            __DIR__.'/path/to/views' => resource_path('views/vendor/courier'),
        ]);
    }

Ahora, cuando los usuarios de su paquete ejecutan el comando Artisan `vendor:publish` de Laravel, las vistas de su paquete se copiarán en la ubicación de publicación especificada.
> > Now, when users of your package execute Laravel's `vendor:publish` Artisan command, your package's views will be copied to the specified publish location.

<a name="commands"></a>
## Comandos : Commands

Para registrar los comandos Artisan de su paquete con Laravel, puede usar el método `commands`. Este método espera un array de nombres de clase de comando. Una vez que los comandos han sido registrados, puede ejecutarlos usando [Artisan CLI](/docs/{{version}}/artisan):
> > To register your package's Artisan commands with Laravel, you may use the `commands` method. This method expects an array of command class names. Once the commands have been registered, you may execute them using the [Artisan CLI](/docs/{{version}}/artisan):

    /**
     * Bootstrap the application services.
     *
     * @return void
     */
    public function boot()
    {
        if ($this->app->runningInConsole()) {
            $this->commands([
                FooCommand::class,
                BarCommand::class,
            ]);
        }
    }

<a name="public-assets"></a>
## Public Assets

Su paquete puede tener assets JavaScript, CSS e imágenes. Para publicar estos assets en el directorio `public` de la aplicación, use el método `publishes` del proveedor de servicios. En este ejemplo, también agregaremos una etiqueta de grupo de asset `public`, que se puede usar para publicar grupos de assets relacionados:
> > Your package may have assets such as JavaScript, CSS, and images. To publish these assets to the application's `public` directory, use the service provider's `publishes` method. In this example, we will also add a `public` asset group tag, which may be used to publish groups of related assets:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/assets' => public_path('vendor/courier'),
        ], 'public');
    }

Ahora, cuando los usuarios de su paquete ejecutan el comando `vendor:publish`, sus assets se copiarán en la ubicación de publicación especificada. Como normalmente necesitará sobrescribir los activos cada vez que se actualice el paquete, puede usar el indicador `--force`:
> > Now, when your package's users execute the `vendor:publish` command, your assets will be copied to the specified publish location. Since you will typically need to overwrite the assets every time the package is updated, you may use the `--force` flag:

    php artisan vendor:publish --tag=public --force

<a name="publishing-file-groups"></a>
## Publicar grupos de archivos : Publishing File Groups

Es posible que desee publicar grupos de assets y recursos de paquetes por separado. Por ejemplo, es posible que desee permitir que los usuarios publiquen los archivos de configuración de su paquete sin verse obligados a publicar los assets de su paquete. Puede hacer esto "etiquetándolos" cuando llama al método `publishes` del proveedor de servicios de un paquete. Por ejemplo, usemos etiquetas para definir dos grupos de publicación en el método `boot` de un proveedor de servicios de paquetes:
> > You may want to publish groups of package assets and resources separately. For instance, you might want to allow your users to publish your package's configuration files without being forced to publish your package's assets. You may do this by "tagging" them when calling the `publishes` method from a package's service provider. For example, let's use tags to define two publish groups in the `boot` method of a package service provider:

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/../config/package.php' => config_path('package.php')
        ], 'config');

        $this->publishes([
            __DIR__.'/../database/migrations/' => database_path('migrations')
        ], 'migrations');
    }

Ahora sus usuarios pueden publicar estos grupos por separado al hacer referencia a su etiqueta al ejecutar el comando `vendor:publish`:
> > Now your users may publish these groups separately by referencing their tag when executing the `vendor:publish` command:

    php artisan vendor:publish --tag=config
