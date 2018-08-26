# Plantillas Blade : Blade Templates

- [Introduction](#introduction)
- [Template Inheritance](#template-inheritance)
    - [Defining A Layout](#defining-a-layout)
    - [Extending A Layout](#extending-a-layout)
- [Components & Slots](#components-and-slots)
- [Displaying Data](#displaying-data)
    - [Blade & JavaScript Frameworks](#blade-and-javascript-frameworks)
- [Control Structures](#control-structures)
    - [If Statements](#if-statements)
    - [Switch Statements](#switch-statements)
    - [Loops](#loops)
    - [The Loop Variable](#the-loop-variable)
    - [Comments](#comments)
    - [PHP](#php)
- [Including Sub-Views](#including-sub-views)
    - [Rendering Views For Collections](#rendering-views-for-collections)
- [Stacks](#stacks)
- [Service Injection](#service-injection)
- [Extending Blade](#extending-blade)
    - [Custom If Statements](#custom-if-statements)

<a name="introduction"></a>
## Introducción : Introduction

Blade es el motor de creación de plantillas simple pero potente proporcionado con Laravel. A diferencia de otros populares motores de plantillas de PHP, Blade no le impide usar código PHP simple en sus vistas. De hecho, todas las vistas de Blade se compilan en código PHP simple y se almacenan en la memoria caché hasta que se modifiquen, lo que significa que Blade agrega esencialmente una sobrecarga a su aplicación. Los archivos Blade View usan la extensión de archivo `.blade.php` y típicamente se almacenan en el directorio `resources/views`.
> > Blade is the simple, yet powerful templating engine provided with Laravel. Unlike other popular PHP templating engines, Blade does not restrict you from using plain PHP code in your views. In fact, all Blade views are compiled into plain PHP code and cached until they are modified, meaning Blade adds essentially zero overhead to your application. Blade view files use the `.blade.php` file extension and are typically stored in the `resources/views` directory.

<a name="template-inheritance"></a>
## Herencia de plantilla : Template Inheritance

<a name="defining-a-layout"></a>
### Definición de un diseño : Defining A Layout

Dos de las principales ventajas de usar Blade son _template inheritance_ y _sections_. Para comenzar, echemos un vistazo a un ejemplo simple. Primero, examinaremos un diseño de página "maestro". Dado que la mayoría de las aplicaciones web mantienen el mismo diseño general en varias páginas, es conveniente definir este diseño como una sola vista de Blade:
> > Two of the primary benefits of using Blade are _template inheritance_ and _sections_. To get started, let's take a look at a simple example. First, we will examine a "master" page layout. Since most web applications maintain the same general layout across various pages, it's convenient to define this layout as a single Blade view:

    <!-- Stored in resources/views/layouts/app.blade.php -->

    <html>
        <head>
            <title>App Name - @yield('title')</title>
        </head>
        <body>
            @section('sidebar')
                This is the master sidebar.
            @show

            <div class="container">
                @yield('content')
            </div>
        </body>
    </html>

Como puede ver, este archivo contiene el marcado HTML típico. Sin embargo, tome nota de las directivas `@section` y `@yield`. La directiva `@section`, como su nombre lo indica, define una sección de contenido, mientras que la directiva `@yield` se utiliza para mostrar el contenido de una sección determinada.
> > As you can see, this file contains typical HTML mark-up. However, take note of the `@section` and `@yield` directives. The `@section` directive, as the name implies, defines a section of content, while the `@yield` directive is used to display the contents of a given section.

Ahora que hemos definido un diseño para nuestra aplicación, definamos una página secundaria que hereda el diseño.
> > Now that we have defined a layout for our application, let's define a child page that inherits the layout.

<a name="extending-a-layout"></a>
### Extendiendo un diseño : Extending A Layout

Al definir una vista secundaria, use la directiva Blade `@extends` para especificar qué diseño debe "heredarse" la vista secundaria. Las vistas que extienden un diseño de Blade pueden inyectar contenido en las secciones de diseño mediante las directivas `@section`. Recuerde, como se ve en el ejemplo anterior, los contenidos de estas secciones se mostrarán en el diseño usando `@yield`:
> > When defining a child view, use the Blade `@extends` directive to specify which layout the child view should "inherit". Views which extend a Blade layout may inject content into the layout's sections using `@section` directives. Remember, as seen in the example above, the contents of these sections will be displayed in the layout using `@yield`:

    <!-- Stored in resources/views/child.blade.php -->

    @extends('layouts.app')

    @section('title', 'Page Title')

    @section('sidebar')
        @@parent

        <p>This is appended to the master sidebar.</p>
    @endsection

    @section('content')
        <p>This is my body content.</p>
    @endsection

En este ejemplo, la sección `sidebar` utiliza la directiva `@@parent` para agregar (en lugar de sobreescribir) contenido a la barra lateral del diseño. La directiva `@@parent` se reemplazará por el contenido del diseño cuando se represente la vista.
> > In this example, the `sidebar` section is utilizing the `@@parent` directive to append (rather than overwriting) content to the layout's sidebar. The `@@parent` directive will be replaced by the content of the layout when the view is rendered.

> {tip} Contrariamente al ejemplo anterior, esta sección `sidebar` termina con `@endsection` en lugar de `@show`. La directiva `@endsection` solo definirá una sección, mientras que `@show` definirá y **generará de inmediato** la sección.
> > > {tip} Contrary to the previous example, this `sidebar` section ends with `@endsection` instead of `@show`. The `@endsection` directive will only define a section while `@show` will define and **immediately yield** the section.

Las vistas de Blade pueden devolverse desde rutas usando el helper `view` global:
> > Blade views may be returned from routes using the global `view` helper:

    Route::get('blade', function () {
        return view('child');
    });

<a name="components-and-slots"></a>
## Componentes y ranuras : Components & Slots

Los componentes y las ranuras ofrecen beneficios similares a las secciones y diseños; sin embargo, algunos pueden encontrar el modelo mental de componentes y slots más fácil de entender. Primero, imaginemos un componente de "alerta" reutilizable que nos gustaría volver a utilizar en nuestra aplicación:
> > Components and slots provide similar benefits to sections and layouts; however, some may find the mental model of components and slots easier to understand. First, let's imagine a reusable "alert" component we would like to reuse throughout our application:

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        {{ $slot }}
    </div>

La variable `{{$ slot}}` contendrá el contenido que deseamos inyectar en el componente. Ahora, para construir este componente, podemos usar la directiva Blade `@component`:
> > The `{{ $slot }}` variable will contain the content we wish to inject into the component. Now, to construct this component, we can use the `@component` Blade directive:

    @component('alert')
        <strong>Whoops!</strong> Something went wrong!
    @endcomponent

A veces es útil definir múltiples espacios para un componente. Modifiquemos nuestro componente de alerta para permitir la inyección de un "título". Las ranuras con nombre se pueden mostrar "haciendo eco" de la variable que coincide con su nombre:
> > Sometimes it is helpful to define multiple slots for a component. Let's modify our alert component to allow for the injection of a "title". Named slots may be displayed by "echoing" the variable that matches their name:

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        <div class="alert-title">{{ $title }}</div>

        {{ $slot }}
    </div>

Ahora, podemos inyectar contenido en la ranura nombrada usando la directiva `@slot`. Cualquier contenido que no esté dentro de una directiva `@slot` se pasará al componente en la variable `$slot`:
> > Now, we can inject content into the named slot using the `@slot` directive. Any content not within a `@slot` directive will be passed to the component in the `$slot` variable:

    @component('alert')
        @slot('title')
            Forbidden
        @endslot

        You are not allowed to access this resource!
    @endcomponent

#### Pasar datos adicionales a los componentes : Passing Additional Data To Components

A veces puede necesitar pasar datos adicionales a un componente. Por este motivo, puede pasar una matriz de datos como el segundo argumento de la directiva `@component`. Todos los datos estarán disponibles para la plantilla del componente como variables:
> > Sometimes you may need to pass additional data to a component. For this reason, you can pass an array of data as the second argument to the `@component` directive. All of the data will be made available to the component template as variables:

    @component('alert', ['foo' => 'bar'])
        ...
    @endcomponent

#### Componentes de aliasing : Aliasing Components

Si sus componentes Blade están almacenados en un subdirectorio, es posible que desee alisarlos para facilitar el acceso. Por ejemplo, imagine un componente Blade que se almacena en `resources/views/components/alert.blade.php`. Puede usar el método `component` para asignar el alias del componente de` components.alert` a `alert`. Normalmente, esto debe hacerse en el método `boot` de su `AppServiceProvider`:
> > If your Blade components are stored in a sub-directory, you may wish to alias them for easier access. For example, imagine a Blade component that is stored at `resources/views/components/alert.blade.php`. You may use the `component` method to alias the component from `components.alert` to `alert`. Typically, this should be done in the `boot` method of your `AppServiceProvider`:

    use Illuminate\Support\Facades\Blade;

    Blade::component('components.alert', 'alert');

Una vez que el componente ha recibido un alias, puede procesarlo usando una directiva:
> > Once the component has been aliased, you may render it using a directive:

    @alert(['type' => 'danger'])
        You are not allowed to access this resource!
    @endalert

Puede omitir los parámetros del componente si no tiene ranuras adicionales:
> > You may omit the component parameters if it has no additional slots:

    @alert
        You are not allowed to access this resource!
    @endalert

<a name="displaying-data"></a>
## Visualización de datos : Displaying Data

Puede mostrar datos pasados ​​a sus vistas de Blade envolviendo la variable entre llaves. Por ejemplo, dada la siguiente ruta:
> > You may display data passed to your Blade views by wrapping the variable in curly braces. For example, given the following route:

    Route::get('greeting', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

Puede mostrar el contenido de la variable `name` de la siguiente manera:
> > You may display the contents of the `name` variable like so:

    Hello, {{ $name }}.

Por supuesto, no está limitado a mostrar los contenidos de las variables pasadas a la vista. También puede repetir los resultados de cualquier función de PHP. De hecho, puedes poner cualquier código PHP que desees dentro de una declaración de eco Blade:
> > Of course, you are not limited to displaying the contents of the variables passed to the view. You may also echo the results of any PHP function. In fact, you can put any PHP code you wish inside of a Blade echo statement:

    The current UNIX timestamp is {{ time() }}.

> {tip} Las instrucciones Blade `{{ }}` se envían automáticamente a través de la función `htmlspecialchars` de PHP para evitar ataques XSS.
> > > {tip} Blade `{{ }}` statements are automatically sent through PHP's `htmlspecialchars` function to prevent XSS attacks.

#### Visualización de datos de Unescaped : Displaying Unescaped Data

Por defecto, las declaraciones Blade `{{ }}` se envían automáticamente a través de la función `htmlspecialchars` de PHP para evitar ataques XSS. Si no desea que se escapen sus datos, puede usar la siguiente sintaxis:
> > By default, Blade `{{ }}` statements are automatically sent through PHP's `htmlspecialchars` function to prevent XSS attacks. If you do not want your data to be escaped, you may use the following syntax:

    Hello, {!! $name !!}.

> {note} Tenga mucho cuidado al repetir el contenido proporcionado por los usuarios de su aplicación. Utilice siempre la sintaxis de paréntesis doble escapada para evitar ataques XSS cuando se muestran datos proporcionados por el usuario.
> > > {note} Be very careful when echoing content that is supplied by users of your application. Always use the escaped, double curly brace syntax to prevent XSS attacks when displaying user supplied data.

#### Presentando JSON : Rendering JSON

En ocasiones, puede pasar una matriz a su vista con la intención de representarla como JSON para inicializar una variable de JavaScript. Por ejemplo:
> > Sometimes you may pass an array to your view with the intention of rendering it as JSON in order to initialize a JavaScript variable. For example:

    <script>
        var app = <?php echo json_encode($array); ?>;
    </script>

Sin embargo, en lugar de llamar manualmente a `json_encode`, puede usar la directiva `@json` Blade:
> > However, instead of manually calling `json_encode`, you may use the `@json` Blade directive:

    <script>
        var app = @json($array);
    </script>

#### HTML Entity Encoding : HTML Entity Encoding

Por defecto, Blade (y el Laravel `e` helper) doblarán las entidades HTML. Si desea deshabilitar la codificación doble, llame al método `Blade::withoutDoubleEncoding` del método `boot` de `AppServiceProvider`:
> > By default, Blade (and the Laravel `e` helper) will double encode HTML entities. If you would like to disable double encoding, call the `Blade::withoutDoubleEncoding` method from the `boot` method of your `AppServiceProvider`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::withoutDoubleEncoding();
        }
    }

<a name="blade-and-javascript-frameworks"></a>
### Blade y Frameworks JavaScript : Blade & JavaScript Frameworks

Dado que muchos frameworks de JavaScript también usan llaves "rizadas" para indicar que una expresión dada debe mostrarse en el navegador, puede usar el símbolo `@` para informar al motor de renderizado Blade que una expresión debe permanecer intacta. Por ejemplo:
> > Since many JavaScript frameworks also use "curly" braces to indicate a given expression should be displayed in the browser, you may use the `@` symbol to inform the Blade rendering engine an expression should remain untouched. For example:

    <h1>Laravel</h1>

    Hello, @{{ name }}.

En este ejemplo, el símbolo `@` será eliminado por Blade; sin embargo, la expresión `{{name}}` no se verá afectada por el motor Blade, lo que le permitirá, en cambio, ser procesada por su framework JavaScript.
> > In this example, the `@` symbol will be removed by Blade; however, `{{ name }}` expression will remain untouched by the Blade engine, allowing it to instead be rendered by your JavaScript framework.

#### La directiva `@verbatim` : The `@verbatim` Directive

Si está visualizando variables de JavaScript en una gran parte de su plantilla, puede ajustar el HTML en la directiva `@verbatim` para que no tenga que prefijar cada instrucción de eco Blade con un símbolo `@`:
> > If you are displaying JavaScript variables in a large portion of your template, you may wrap the HTML in the `@verbatim` directive so that you do not have to prefix each Blade echo statement with an `@` symbol:

    @verbatim
        <div class="container">
            Hello, {{ name }}.
        </div>
    @endverbatim

<a name="control-structures"></a>
## Estructuras de Control : Control Structures

Además de la herencia de la plantilla y la visualización de datos, Blade también proporciona accesos directos convenientes para estructuras de control de PHP comunes, como sentencias y bucles condicionales. Estos accesos directos proporcionan una forma muy limpia y concisa de trabajar con las estructuras de control de PHP, al tiempo que permanecen familiares para sus contrapartes de PHP.
> > In addition to template inheritance and displaying data, Blade also provides convenient shortcuts for common PHP control structures, such as conditional statements and loops. These shortcuts provide a very clean, terse way of working with PHP control structures, while also remaining familiar to their PHP counterparts.

<a name="if-statements"></a>
### Sentencias If : If Statements

Puede construir sentencias `if` utilizando las directivas `@if`, `@elseif`, `@else` y `@endif`. Estas directivas funcionan de manera idéntica a sus contrapartes de PHP:
> > You may construct `if` statements using the `@if`, `@elseif`, `@else`, and `@endif` directives. These directives function identically to their PHP counterparts:

    @if (count($records) === 1)
        I have one record!
    @elseif (count($records) > 1)
        I have multiple records!
    @else
        I don't have any records!
    @endif

Para mayor comodidad, Blade también proporciona una directiva `@unless`:
> > For convenience, Blade also provides an `@unless` directive:

    @unless (Auth::check())
        You are not signed in.
    @endunless

Además de las directivas condicionales ya discutidas, las directivas `@isset` y `@empty` se pueden usar como accesos directos convenientes para sus respectivas funciones de PHP:
> > In addition to the conditional directives already discussed, the `@isset` and `@empty` directives may be used as convenient shortcuts for their respective PHP functions:

    @isset($records)
        // $records is defined and is not null...
    @endisset

    @empty($records)
        // $records is "empty"...
    @endempty

#### Directivas de autenticación : Authentication Directives

Las directivas `@auth` y `@guest` se pueden usar para determinar rápidamente si el usuario actual está autenticado o es un invitado:
> > The `@auth` and `@guest` directives may be used to quickly determine if the current user is authenticated or is a guest:

    @auth
        // The user is authenticated...
    @endauth

    @guest
        // The user is not authenticated...
    @endguest

Si es necesario, puede especificar el [protector de autenticación](/docs/{{version}}/authentication) que debe verificarse al usar las directivas `@auth` y `@guest`:
> > If needed, you may specify the [authentication guard](/docs/{{version}}/authentication) that should be checked when using the `@auth` and `@guest` directives:

    @auth('admin')
        // The user is authenticated...
    @endauth

    @guest('admin')
        // The user is not authenticated...
    @endguest

#### Directivas de sección : Section Directives

Puede verificar si una sección tiene contenido usando la directiva `@hasSection`:
> > You may check if a section has content using the `@hasSection` directive:

    @hasSection('navigation')
        <div class="pull-right">
            @yield('navigation')
        </div>

        <div class="clearfix"></div>
    @endif

<a name="switch-statements"></a>
### Sentencias de cambio : Switch Statements

Las sentencias Switch se pueden construir utilizando las directivas `@switch`, `@case`, `@break`, `@default` y `@endswitch`:
> > Switch statements can be constructed using the `@switch`, `@case`, `@break`, `@default` and `@endswitch` directives:

    @switch($i)
        @case(1)
            First case...
            @break

        @case(2)
            Second case...
            @break

        @default
            Default case...
    @endswitch

<a name="loops"></a>
### Bucles : Loops

Además de declaraciones condicionales, Blade proporciona directivas simples para trabajar con estructuras de bucle de PHP. Una vez más, cada una de estas directivas funciona de manera idéntica a sus homólogos de PHP:
> > In addition to conditional statements, Blade provides simple directives for working with PHP's loop structures. Again, each of these directives functions identically to their PHP counterparts:

    @for ($i = 0; $i < 10; $i++)
        The current value is {{ $i }}
    @endfor

    @foreach ($users as $user)
        <p>This is user {{ $user->id }}</p>
    @endforeach

    @forelse ($users as $user)
        <li>{{ $user->name }}</li>
    @empty
        <p>No users</p>
    @endforelse

    @while (true)
        <p>I'm looping forever.</p>
    @endwhile

> {tip} Cuando se repite, puede usar [loop variable](#the-loop-variable) para obtener información valiosa sobre el ciclo, como por ejemplo si está en la primera o la última iteración a través del ciclo.
> > > {tip} When looping, you may use the [loop variable](#the-loop-variable) to gain valuable information about the loop, such as whether you are in the first or last iteration through the loop.

Al usar bucles también puede finalizar el ciclo u omitir la iteración actual:
> > When using loops you may also end the loop or skip the current iteration:

    @foreach ($users as $user)
        @if ($user->type == 1)
            @continue
        @endif

        <li>{{ $user->name }}</li>

        @if ($user->number == 5)
            @break
        @endif
    @endforeach

También puede incluir la condición con la declaración de directiva en una línea:
> > You may also include the condition with the directive declaration in one line:

    @foreach ($users as $user)
        @continue($user->type == 1)

        <li>{{ $user->name }}</li>

        @break($user->number == 5)
    @endforeach

<a name="the-loop-variable"></a>
### La variable de bucle : The Loop Variable

Al realizar un bucle, una variable `$loop` estará disponible dentro de su bucle. Esta variable proporciona acceso a algunos bits de información útiles, como el índice de ciclo actual y si esta es la primera o la última iteración a través del ciclo:
> > When looping, a `$loop` variable will be available inside of your loop. This variable provides access to some useful bits of information such as the current loop index and whether this is the first or last iteration through the loop:

    @foreach ($users as $user)
        @if ($loop->first)
            This is the first iteration.
        @endif

        @if ($loop->last)
            This is the last iteration.
        @endif

        <p>This is user {{ $user->id }}</p>
    @endforeach

Si se encuentra en un bucle anidado, puede acceder a la variable `$loop` del bucle padre a través de la propiedad `parent`:
> > If you are in a nested loop, you may access the parent loop's `$loop` variable via the `parent` property:

    @foreach ($users as $user)
        @foreach ($user->posts as $post)
            @if ($loop->parent->first)
                This is first iteration of the parent loop.
            @endif
        @endforeach
    @endforeach

La variable `$loop` también contiene una variedad de otras propiedades útiles:
> > The `$loop` variable also contains a variety of other useful properties:

Property  | Description
------------- | -------------
`$loop->index`  |  The index of the current loop iteration (starts at 0).
`$loop->iteration`  |  The current loop iteration (starts at 1).
`$loop->remaining`  |  The iterations remaining in the loop.
`$loop->count`  |  The total number of items in the array being iterated.
`$loop->first`  |  Whether this is the first iteration through the loop.
`$loop->last`  |  Whether this is the last iteration through the loop.
`$loop->depth`  |  The nesting level of the current loop.
`$loop->parent`  |  When in a nested loop, the parent's loop variable.

Property  | Description
------------- | -------------
`$loop->index`  |  El índice de la iteración del ciclo actual (comienza en 0).
`$loop->iteration`  |  La iteración del bucle actual (comienza en 1).
`$loop->remaining`  |  Las iteraciones restantes en el bucle.
`$loop->count`  |  La cantidad total de elementos en la matriz que se itera.
`$loop->first`  |  Si esta es la primera iteración a través del bucle.
`$loop->last`  |  Si esta es la última iteración a través del bucle.
`$loop->depth`  |  El nivel de anidamiento del bucle actual.
`$loop->parent`  |  Cuando está en un bucle anidado, la variable de bucle del padre.

<a name="comments"></a>
### Comentarios : Comments

Blade también te permite definir comentarios en tus vistas. Sin embargo, a diferencia de los comentarios en HTML, los comentarios de Blade no están incluidos en el código HTML devuelto por su aplicación:
> > Blade also allows you to define comments in your views. However, unlike HTML comments, Blade comments are not included in the HTML returned by your application:

    {{-- This comment will not be present in the rendered HTML --}}

<a name="php"></a>
### PHP

En algunas situaciones, es útil insertar código PHP en sus vistas. Puede usar la directiva Blade `@php` para ejecutar un bloque de PHP simple dentro de su plantilla:
> > In some situations, it's useful to embed PHP code into your views. You can use the Blade `@php` directive to execute a block of plain PHP within your template:

    @php
        //
    @endphp

> {tip} Aunque Blade proporciona esta función, usarla con frecuencia puede ser una señal de que tiene demasiada lógica incrustada dentro de su plantilla.
> > > {tip} While Blade provides this feature, using it frequently may be a signal that you have too much logic embedded within your template.

<a name="including-sub-views"></a>
## Incluyendo Sub-Vistas : Including Sub-Views

La directiva `@include` de Blade le permite incluir una vista de Blade desde otra vista. Todas las variables que están disponibles para la vista primaria estarán disponibles para la vista incluida:
> > Blade's `@include` directive allows you to include a Blade view from within another view. All variables that are available to the parent view will be made available to the included view:

    <div>
        @include('shared.errors')

        <form>
            <!-- Form Contents -->
        </form>
    </div>

Aunque la vista incluida heredará todos los datos disponibles en la vista principal, también puede pasar una matriz de datos adicionales a la vista incluida:
> > Even though the included view will inherit all data available in the parent view, you may also pass an array of extra data to the included view:

    @include('view.name', ['some' => 'data'])

Por supuesto, si intenta incluir `@include` una vista que no existe, Laravel lanzará un error. Si desea incluir una vista que puede estar presente o no, debe usar la directiva `@includeIf`:
> > Of course, if you attempt to `@include` a view which does not exist, Laravel will throw an error. If you would like to include a view that may or may not be present, you should use the `@includeIf` directive:

    @includeIf('view.name', ['some' => 'data'])

Si desea incluir `@include` una vista dependiendo de una condición booleana dada, puede usar la directiva `@includeWhen`:
> > If you would like to `@include` a view depending on a given boolean condition, you may use the `@includeWhen` directive:

    @includeWhen($boolean, 'view.name', ['some' => 'data'])

Para incluir la primera vista que existe desde un conjunto de vistas determinado, puede usar la directiva `includeFirst`:
> > To include the first view that exists from a given array of views, you may use the `includeFirst` directive:

    @includeFirst(['custom.admin', 'admin'], ['some' => 'data'])

> {note} Debería evitar usar las constantes `__DIR__` y `__FILE__` en sus vistas de Blade, ya que se referirán a la ubicación de la vista compilada en caché.
> > > {note} You should avoid using the `__DIR__` and `__FILE__` constants in your Blade views, since they will refer to the location of the cached, compiled view.

<a name="rendering-views-for-collections"></a>
### Representación de vistas para colecciones : Rendering Views For Collections

Puede combinar bucles e incluirlos en una sola línea con la directiva `@each` de Blade:
> > You may combine loops and includes into one line with Blade's `@each` directive:

    @each('view.name', $jobs, 'job')

El primer argumento es la vista parcial para representar para cada elemento en la matriz o colección. El segundo argumento es la matriz o colección sobre la que desea iterar, mientras que el tercer argumento es el nombre de la variable que se asignará a la iteración actual dentro de la vista. Entonces, por ejemplo, si está iterando sobre una matriz de `jobs`, típicamente querrá acceder a cada trabajo como una variable `job` dentro de su vista parcial. La clave para la iteración actual estará disponible como la variable `key` dentro de su vista parcial.
> > The first argument is the view partial to render for each element in the array or collection. The second argument is the array or collection you wish to iterate over, while the third argument is the variable name that will be assigned to the current iteration within the view. So, for example, if you are iterating over an array of `jobs`, typically you will want to access each job as a `job` variable within your view partial. The key for the current iteration will be available as the `key` variable within your view partial.

You may also pass a fourth argument to the `@each` directive. This argument determines the view that will be rendered if the given array is empty.

    @each('view.name', $jobs, 'job', 'view.empty')

> {note} Las vistas renderizadas mediante `@each` no heredan las variables de la vista principal. Si la vista hija requiere estas variables, debería usar `@foreach` y` @include` en su lugar.
> > > {note} Views rendered via `@each` do not inherit the variables from the parent view. If the child view requires these variables, you should use `@foreach` and `@include` instead.

<a name="stacks"></a>
## Pilas : Stacks

Blade le permite presionar a las pilas con nombre que se pueden representar en otro lugar en otra vista o diseño. Esto puede ser particularmente útil para especificar cualquier biblioteca de JavaScript requerida por las vistas de su hijo:
> > Blade allows you to push to named stacks which can be rendered somewhere else in another view or layout. This can be particularly useful for specifying any JavaScript libraries required by your child views:

    @push('scripts')
        <script src="/example.js"></script>
    @endpush

Puede empujar a una pila tantas veces como sea necesario. Para representar el contenido completo de la pila, pase el nombre de la pila a la directiva `@stack`:
> > You may push to a stack as many times as needed. To render the complete stack contents, pass the name of the stack to the `@stack` directive:

    <head>
        <!-- Head Contents -->

        @stack('scripts')
    </head>

Si desea anteponer el contenido al comienzo de una pila, debe usar la directiva `@prepend`:
> > If you would like to prepend content onto the beginning of a stack, you should use the `@prepend` directive:

    @push('scripts')
        This will be second...
    @endpush

    // Later...

    @prepend('scripts')
        This will be first...
    @endprepend

<a name="service-injection"></a>
## Inyección de servicio : Service Injection

La directiva `@inject` se puede usar para recuperar un servicio de Laravel [service container](/docs/{{version}}/container). El primer argumento pasado a `@inject` es el nombre de la variable en la que se colocará el servicio, mientras que el segundo argumento es la clase o el nombre de la interfaz del servicio que desea resolver:
> > The `@inject` directive may be used to retrieve a service from the Laravel [service container](/docs/{{version}}/container). The first argument passed to `@inject` is the name of the variable the service will be placed into, while the second argument is the class or interface name of the service you wish to resolve:

    @inject('metrics', 'App\Services\MetricsService')

    <div>
        Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
    </div>

<a name="extending-blade"></a>
## Extending Blade

> > Blade allows you to define your own custom directives using the `directive` method. When the Blade compiler encounters the custom directive, it will call the provided callback with the expression that the directive contains.
## Extendiendo Blade

El siguiente ejemplo crea una directiva `@datetime($var)` que formatea un `$var` dado, que debería ser una instancia de `DateTime`:
> > The following example creates a `@datetime($var)` directive which formats a given `$var`, which should be an instance of `DateTime`:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Blade::directive('datetime', function ($expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
            });
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

Como puede ver, encadenaremos el método `format` en cualquier expresión que pase a la directiva. Entonces, en este ejemplo, el PHP final generado por esta directiva será:
> > As you can see, we will chain the `format` method onto whatever expression is passed into the directive. So, in this example, the final PHP generated by this directive will be:

    <?php echo ($var)->format('m/d/Y H:i'); ?>

> {note} Después de actualizar la lógica de una directiva Blade, deberá eliminar todas las vistas de Blade en caché. Las vistas de Blade almacenadas en caché se pueden eliminar utilizando el comando Artisan `view:clear`.
> > > {note} After updating the logic of a Blade directive, you will need to delete all of the cached Blade views. The cached Blade views may be removed using the `view:clear` Artisan command.

<a name="custom-if-statements"></a>
### Personalizado de sentencias If : Custom If Statements

La programación de una directiva personalizada a veces es más compleja de lo necesario al definir enunciados condicionales simples y personalizados. Por esa razón, Blade proporciona un método `Blade::if` que le permite definir rápidamente directivas condicionales personalizadas utilizando Closures. Por ejemplo, definamos un condicional personalizado que verifique el entorno de la aplicación actual. Podemos hacer esto en el método `boot` de nuestro `AppServiceProvider`:
> > Programming a custom directive is sometimes more complex than necessary when defining simple, custom conditional statements. For that reason, Blade provides a `Blade::if` method which allows you to quickly define custom conditional directives using Closures. For example, let's define a custom conditional that checks the current application environment. We may do this in the `boot` method of our `AppServiceProvider`:

    use Illuminate\Support\Facades\Blade;

    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

Una vez que se ha definido el condicional personalizado, podemos usarlo fácilmente en nuestras plantillas:
> > Once the custom conditional has been defined, we can easily use it on our templates:

    @env('local')
        // The application is in the local environment...
    @elseenv('testing')
        // The application is in the testing environment...
    @else
        // The application is not in the local or testing environment...
    @endenv
