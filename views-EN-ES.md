# Vistas  : Views

- [Creating Views](#creating-views)
- [Passing Data To Views](#passing-data-to-views)
    - [Sharing Data With All Views](#sharing-data-with-all-views)
- [View Composers](#view-composers)

<a name="creating-views"></a>
## Creando Vistas : Creating Views

> {tip} ¿Está buscando más información sobre cómo escribir plantillas Blade? Consulte la [documentación de Blade](/docs/{{version}}/blade) completa para comenzar.
> > > {tip} Looking for more information on how to write Blade templates? Check out the full [Blade documentation](/docs/{{version}}/blade) to get started.

Las vistas contienen el HTML servido por su aplicación y separa la lógica de su controlador / aplicación de su lógica de presentación. Las vistas se almacenan en el directorio `resources/views`. Una vista simple podría verse más o menos así:
> > Views contain the HTML served by your application and separate your controller / application logic from your presentation logic. Views are stored in the `resources/views` directory. A simple view might look something like this:

    <!-- View stored in resources/views/greeting.blade.php -->

    <html>
        <body>
            <h1>Hello, {{ $name }}</h1>
        </body>
    </html>

Dado que esta vista se almacena en `resources/views/greeting.blade.php`, podemos devolverla usando el helper global `view` de la siguiente manera:
> > Since this view is stored at `resources/views/greeting.blade.php`, we may return it using the global `view` helper like so:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

Como puede ver, el primer argumento pasado al helper `view` corresponde al nombre del archivo de vista en el directorio `resources/views`. El segundo argumento es un array de datos que debería estar disponible para la vista. En este caso, estamos pasando la variable `name`, que se muestra en la vista usando [Sintaxis Blade](/docs/{{version}}/blade).
> > As you can see, the first argument passed to the `view` helper corresponds to the name of the view file in the `resources/views` directory. The second argument is an array of data that should be made available to the view. In this case, we are passing the `name` variable, which is displayed in the view using [Blade syntax](/docs/{{version}}/blade).

Por supuesto, las vistas también pueden estar anidadas dentro de los subdirectorios del directorio `resources/views`. La notación "Dot" se puede usar para hacer referencia a vistas anidadas. Por ejemplo, si su vista está almacenada en `resources/views/admin/profile.blade.php`, puede referenciarla de esta manera:
> > Of course, views may also be nested within sub-directories of the `resources/views` directory. "Dot" notation may be used to reference nested views. For example, if your view is stored at `resources/views/admin/profile.blade.php`, you may reference it like so:

    return view('admin.profile', $data);

#### Determinando si existe una vista : Determining If A View Exists

Si necesita determinar si existe una vista, puede usar la fachada `View`. El método `exists` devolverá `true` si la vista existe:
> > If you need to determine if a view exists, you may use the `View` facade. The `exists` method will return `true` if the view exists:

    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        //
    }

#### Creando la primera vista disponible : Creating The First Available View

Utilizando el método `first`, puede crear la primera vista que existe en un array de vistas determinado. Esto es útil si su aplicación o paquete permite personalizar o sobrescribir las vistas:
> > Using the `first` method, you may create the first view that exists in a given array of views. This is useful if your application or package allows views to be customized or overwritten:

    return view()->first(['custom.admin', 'admin'], $data);

Por supuesto, también puede llamar a este método a través de `View` [facade](/docs/{{version}}/facades):
> > Of course, you may also call this method via the `View` [facade](/docs/{{version}}/facades):

    use Illuminate\Support\Facades\View;

    return View::first(['custom.admin', 'admin'], $data);

<a name="passing-data-to-views"></a>
## Pasar datos a las vistas : Passing Data To Views

Como viste en los ejemplos anteriores, puedes pasar una matriz de datos a las vistas:
> > As you saw in the previous examples, you may pass an array of data to views:

    return view('greetings', ['name' => 'Victoria']);

Al pasar información de esta manera, los datos deben ser una matriz con pares clave / valor. Dentro de su vista, puede acceder a cada valor usando su clave correspondiente, como `<?php echo $key; ?>`. Como alternativa a pasar un array completo de datos a la función de ayuda `view`, puede usar el método `with` para agregar piezas de datos individuales a la vista:
> > When passing information in this manner, the data should be an array with key / value pairs. Inside your view, you can then access each value using its corresponding key, such as `<?php echo $key; ?>`. As an alternative to passing a complete array of data to the `view` helper function, you may use the `with` method to add individual pieces of data to the view:

    return view('greeting')->with('name', 'Victoria');

<a name="sharing-data-with-all-views"></a>
#### Compartir datos con todas las vistas : Sharing Data With All Views

Ocasionalmente, puede que necesite compartir un dato con todas las vistas que su aplicación presenta. Puede hacerlo utilizando el método `share` de la fachada de la vista. Por lo general, debe realizar llamadas a `share` dentro del método `boot` de un proveedor de servicios. Puede agregarlos al `AppServiceProvider` o generar un proveedor de servicios independiente para alojarlos:
> > Occasionally, you may need to share a piece of data with all views that are rendered by your application. You may do so using the view facade's `share` method. Typically, you should place calls to `share` within a service provider's `boot` method. You are free to add them to the `AppServiceProvider` or generate a separate service provider to house them:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            View::share('key', 'value');
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="view-composers"></a>
## Compositores de vistas : View Composers

Los compositores de vista son callbacks o métodos de clase que se invocan cuando se procesa una vista. Si tiene datos que desea vincular a una vista cada vez que se representa esa vista, un compositor de vistas puede ayudarlo a organizar esa lógica en una única ubicación.
> > View composers are callbacks or class methods that are called when a view is rendered. If you have data that you want to be bound to a view each time that view is rendered, a view composer can help you organize that logic into a single location.

Para este ejemplo, registremos los compositores de vista dentro de un [proveedor de servicios](/docs/{{version}}/providers). Usaremos la fachada `View` para acceder a la implementación subyacente del contrato `Illuminate\Contracts\View\Factory`. Recuerde, Laravel no incluye un directorio predeterminado para ver compositores. Usted es libre de organizarlos como lo desee. Por ejemplo, podría crear un directorio `app/Http/ViewComposers`:
> > For this example, let's register the view composers within a [service provider](/docs/{{version}}/providers). We'll use the `View` facade to access the underlying `Illuminate\Contracts\View\Factory` contract implementation. Remember, Laravel does not include a default directory for view composers. You are free to organize them however you wish. For example, you could create an `app/Http/ViewComposers` directory:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function boot()
        {
            // Using class based composers...
            View::composer(
                'profile', 'App\Http\ViewComposers\ProfileComposer'
            );

            // Using Closure based composers...
            View::composer('dashboard', function ($view) {
                //
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

> {note} Recuerde, si crea un nuevo proveedor de servicios para contener sus registros de compositores de vista, deberá agregar el proveedor de servicios al array `providers` en el archivo de configuración `config/app.php`.
> > > {note} Remember, if you create a new service provider to contain your view composer registrations, you will need to add the service provider to the `providers` array in the `config/app.php` configuration file.

Ahora que hemos registrado el compositor, el método `ProfileComposer@compose` se ejecutará cada vez que se muestre la vista de `profile`. Entonces, definamos la clase de compositor:
> > Now that we have registered the composer, the `ProfileComposer@compose` method will be executed each time the `profile` view is being rendered. So, let's define the composer class:

    <?php

    namespace App\Http\ViewComposers;

    use Illuminate\View\View;
    use App\Repositories\UserRepository;

    class ProfileComposer
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new profile composer.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            // Dependencies automatically resolved by service container...
            $this->users = $users;
        }

        /**
         * Bind data to the view.
         *
         * @param  View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }

Justo antes de que se visualice la vista, se llama al método `compose` del compositor con la instancia `Illuminate\View\View`. Puede usar el método `with` para enlazar datos a la vista.
> > Just before the view is rendered, the composer's `compose` method is called with the `Illuminate\View\View` instance. You may use the `with` method to bind data to the view.

> {tip} Todos los compositores de vistas se resuelven a través del [contenedor de servicios](/docs/{{version}}/container), por lo que puede indicar cualquier dependencia que necesite dentro del constructor de un compositor.
> > > {tip} All view composers are resolved via the [service container](/docs/{{version}}/container), so you may type-hint any dependencies you need within a composer's constructor.

#### Adjuntar un compositor a múltiples vistas : Attaching A Composer To Multiple Views

Puede adjuntar un compositor de vistas a varias vistas a la vez al pasar un array de vistas como primer argumento del método `composer`:
> > You may attach a view composer to multiple views at once by passing an array of views as the first argument to the `composer` method:

    View::composer(
        ['profile', 'dashboard'],
        'App\Http\ViewComposers\MyViewComposer'
    );

El método `composer` también acepta el carácter `*` como comodín, lo que le permite adjuntar un compositor a todas las vistas:
> > The `composer` method also accepts the `*` character as a wildcard, allowing you to attach a composer to all views:

    View::composer('*', function ($view) {
        //
    });

#### Creadores de vistas : View Creators

Los **creadores** de vistas son muy similares a los compositores de vistas; sin embargo, se ejecutan inmediatamente después de que se crea una instancia de la vista en lugar de esperar hasta que la vista esté por renderizarse. Para registrar un creador de vista, use el método `creator`:
View **creators** are very similar to view composers; however, they are executed immediately after the view is instantiated instead of waiting until the view is about to render. To register a view creator, use the `creator` method:

    View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');
