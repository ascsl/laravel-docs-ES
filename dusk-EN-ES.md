# Pruebas del navegador (Laravel Dusk) : Browser Tests (Laravel Dusk)

- [Introduction](#introduction)
- [Installation](#installation)
    - [Using Other Browsers](#using-other-browsers)
- [Getting Started](#getting-started)
    - [Generating Tests](#generating-tests)
    - [Running Tests](#running-tests)
    - [Environment Handling](#environment-handling)
    - [Creating Browsers](#creating-browsers)
    - [Authentication](#authentication)
    - [Database Migrations](#migrations)
- [Interacting With Elements](#interacting-with-elements)
    - [Dusk Selectors](#dusk-selectors)
    - [Clicking Links](#clicking-links)
    - [Text, Values, & Attributes](#text-values-and-attributes)
    - [Using Forms](#using-forms)
    - [Attaching Files](#attaching-files)
    - [Using The Keyboard](#using-the-keyboard)
    - [Using The Mouse](#using-the-mouse)
    - [Scoping Selectors](#scoping-selectors)
    - [Waiting For Elements](#waiting-for-elements)
    - [Making Vue Assertions](#making-vue-assertions)
- [Available Assertions](#available-assertions)
- [Pages](#pages)
    - [Generating Pages](#generating-pages)
    - [Configuring Pages](#configuring-pages)
    - [Navigating To Pages](#navigating-to-pages)
    - [Shorthand Selectors](#shorthand-selectors)
    - [Page Methods](#page-methods)
- [Components](#components)
    - [Generating Components](#generating-components)
    - [Using Components](#using-components)
- [Continuous Integration](#continuous-integration)
    - [CircleCI](#running-tests-on-circle-ci)
    - [Codeship](#running-tests-on-codeship)
    - [Heroku CI](#running-tests-on-heroku-ci)
    - [Travis CI](#running-tests-on-travis-ci)

<a name="introduction"></a>
## Introducción : Introduction

Laravel Dusk proporciona una API expresiva y fácil de usar de automatización y prueba del navegador. De forma predeterminada, Dusk no requiere que instale JDK o Selenium en su máquina. En cambio, Dusk utiliza una instalación independiente de [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home). Sin embargo, puede utilizar cualquier otro controlador compatible con Selenium que desee.
> > Laravel Dusk provides an expressive, easy-to-use browser automation and testing API. By default, Dusk does not require you to install JDK or Selenium on your machine. Instead, Dusk uses a standalone [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home) installation. However, you are free to utilize any other Selenium compatible driver you wish.

<a name="installation"></a>
## Instalación : Installation

Para comenzar, debe agregar la dependencia de Composer `laravel/dusk` a su proyecto:
> > To get started, you should add the `laravel/dusk` Composer dependency to your project:

    composer require --dev laravel/dusk

Una vez que se instala Dusk, debe registrar el proveedor de servicio `Laravel\Dusk\DuskServiceProvider`. Por lo general, esto se realizará automáticamente mediante el registro automático de proveedores de servicios de Laravel.
> > Once Dusk is installed, you should register the `Laravel\Dusk\DuskServiceProvider` service provider. Typically, this will be done automatically via Laravel's automatic service provider registration.

> {note} Si está registrando manualmente el proveedor de servicios de Dusk, **nunca** debe registrarlo en su entorno de producción, ya que hacerlo podría provocar que usuarios arbitrarios puedan autenticarse con su aplicación.
> > > {note} If you are manually registering Dusk's service provider, you should **never** register it in your production environment, as doing so could lead to arbitrary users being able to authenticate with your application.

Después de instalar el paquete Dusk, ejecute el comando Artisan 'dusk: install`:
> > After installing the Dusk package, run the `dusk:install` Artisan command:

    php artisan dusk:install

Se creará un directorio `Browser` dentro de su directorio `tests` y contendrá una prueba de ejemplo. A continuación, establezca la variable de entorno `APP_URL` en su archivo `.env`. Este valor debe coincidir con la URL que usa para acceder a su aplicación en un navegador.
> > A `Browser` directory will be created within your `tests` directory and will contain an example test. Next, set the `APP_URL` environment variable in your `.env` file. This value should match the URL you use to access your application in a browser.

Para ejecutar tus pruebas, utiliza el comando Artisan `dusk`. El comando `dusk` acepta cualquier argumento que también sea aceptado por el comando `phpunit`:
> > To run your tests, use the `dusk` Artisan command. The `dusk` command accepts any argument that is also accepted by the `phpunit` command:

    php artisan dusk

<a name="using-other-browsers"></a>
### Uso de otros navegadores : Using Other Browsers

Por defecto, Dusk utiliza Google Chrome y una instalación independiente de [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home) para ejecutar las pruebas de su navegador. Sin embargo, puede iniciar su propio servidor de Selenium y ejecutar sus pruebas contra cualquier navegador que desee.
> > By default, Dusk uses Google Chrome and a standalone [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home) installation to run your browser tests. However, you may start your own Selenium server and run your tests against any browser you wish.

Para comenzar, abra su archivo `tests/DuskTestCase.php`, que es el caso base de la prueba Dusk para su aplicación. Dentro de este archivo, puede eliminar la llamada al método `startChromeDriver`. Esto impedirá que Dusk inicie automáticamente el ChromeDriver:
> > To get started, open your `tests/DuskTestCase.php` file, which is the base Dusk test case for your application. Within this file, you can remove the call to the `startChromeDriver` method. This will stop Dusk from automatically starting the ChromeDriver:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

A continuación, puede modificar el método `driver` para conectarse a la URL y al puerto de su elección. Además, puede modificar las "capacidades deseadas" que se deben pasar al WebDriver:
> > Next, you may modify the `driver` method to connect to the URL and port of your choice. In addition, you may modify the "desired capabilities" that should be passed to the WebDriver:

    /**
     * Create the RemoteWebDriver instance.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
        );
    }

<a name="getting-started"></a>
## Empezando : Getting Started

<a name="generating-tests"></a>
### Generación de pruebas : Generating Tests

Para generar una prueba de Dusk, use el comando Artisan `dusk:make`. La prueba generada se colocará en el directorio `tests/Browser`:
> > To generate a Dusk test, use the `dusk:make` Artisan command. The generated test will be placed in the `tests/Browser` directory:

    php artisan dusk:make LoginTest

<a name="running-tests"></a>
### Ejecución de pruebas : Running Tests

Para ejecutar las pruebas de su navegador, use el comando Artisan `dusk`:
> > To run your browser tests, use the `dusk` Artisan command:

    php artisan dusk

El comando `dusk` acepta cualquier argumento que sea normalmente aceptado por el corrector de prueba PHPUnit, lo que le permite ejecutar solo las pruebas para un [grupo](https://phpunit.de/manual/current/en/appendixes.annotations.html#appendixes.annotations.group) determinado, etc.
> > The `dusk` command accepts any argument that is normally accepted by the PHPUnit test runner, allowing you to only run the tests for a given [group](https://phpunit.de/manual/current/en/appendixes.annotations.html#appendixes.annotations.group), etc:

    php artisan dusk --group=foo

#### Inicio manual de ChromeDriver : Manually Starting ChromeDriver

Por defecto, Dusk automáticamente intentará iniciar ChromeDriver. Si esto no funciona para su sistema en particular, puede iniciar ChromeDriver manualmente antes de ejecutar el comando `dusk`. Si elige iniciar ChromeDriver manualmente, debe comentar la siguiente línea de su archivo `tests/DuskTestCase.php`:
> > By default, Dusk will automatically attempt to start ChromeDriver. If this does not work for your particular system, you may manually start ChromeDriver before running the `dusk` command. If you choose to start ChromeDriver manually, you should comment out the following line of your `tests/DuskTestCase.php` file:

    /**
     * Prepare for Dusk test execution.
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

Además, si inicia ChromeDriver en un puerto que no sea 9515, debe modificar el método `driver` de la misma clase:
> > In addition, if you start ChromeDriver on a port other than 9515, you should modify the `driver` method of the same class:

    /**
     * Create the RemoteWebDriver instance.
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:9515', DesiredCapabilities::chrome()
        );
    }

<a name="environment-handling"></a>
### Manejo del entorno : Environment Handling

Para forzar a Dusk a usar su propio archivo de entorno al ejecutar pruebas, cree un archivo `.env.dusk.{environment}` en la raíz de su proyecto. Por ejemplo, si va a iniciar el comando `dusk` desde su entorno `local`, debe crear un archivo `.env.dusk.local`.
> > To force Dusk to use its own environment file when running tests, create a `.env.dusk.{environment}` file in the root of your project. For example, if you will be initiating the `dusk` command from your `local` environment, you should create a `.env.dusk.local` file.

Al ejecutar las pruebas, Dusk realizará una copia de seguridad de su archivo `.env` y cambiará el nombre de su entorno Dusk a `.env`. Una vez que las pruebas se hayan completado, su archivo `.env` se restaurará.
> > When running tests, Dusk will back-up your `.env` file and rename your Dusk environment to `.env`. Once the tests have completed, your `.env` file will be restored.

<a name="creating-browsers"></a>
### Crear navegadores : Creating Browsers

Para comenzar, vamos a escribir una prueba que verifique que podemos iniciar sesión en nuestra aplicación. Después de generar una prueba, podemos modificarla para navegar a la página de inicio de sesión, ingresar algunas credenciales y hacer clic en el botón "Iniciar sesión". Para crear una instancia del navegador, llame al método `browse`:
> > To get started, let's write a test that verifies we can log into our application. After generating a test, we can modify it to navigate to the login page, enter some credentials, and click the "Login" button. To create a browser instance, call the `browse` method:

    <?php

    namespace Tests\Browser;

    use App\User;
    use Tests\DuskTestCase;
    use Laravel\Dusk\Chrome;
    use Illuminate\Foundation\Testing\DatabaseMigrations;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;

        /**
         * A basic browser test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $user = factory(User::class)->create([
                'email' => 'taylor@laravel.com',
            ]);

            $this->browse(function ($browser) use ($user) {
                $browser->visit('/login')
                        ->type('email', $user->email)
                        ->type('password', 'secret')
                        ->press('Login')
                        ->assertPathIs('/home');
            });
        }
    }

Como puede ver en el ejemplo anterior, el método `browse` acepta una rellamada. A Dusk se le pasará automáticamente una instancia del navegador a esta rellamada y es el objeto principal utilizado para interactuar y realizar afirmaciones en contra de su aplicación.
> > As you can see in the example above, the `browse` method accepts a callback. A browser instance will automatically be passed to this callback by Dusk and is the main object used to interact with and make assertions against your application.

> {tip} Esta prueba se puede usar para probar la pantalla de inicio de sesión generada por el comando Artisan `make:auth`.
> > > {tip} This test can be used to test the login screen generated by the `make:auth` Artisan command.

#### Creación de múltiples navegadores : Creating Multiple Browsers

A veces puede necesitar múltiples navegadores para realizar una prueba correctamente. Por ejemplo, es posible que se necesiten varios navegadores para probar una pantalla de chat que interactúa con websockets. Para crear varios navegadores, "preguntar" por más de un navegador en la firma de la rellamada dada al método `browse`:
> > Sometimes you may need multiple browsers in order to properly carry out a test. For example, multiple browsers may be needed to test a chat screen that interacts with websockets. To create multiple browsers, "ask" for more than one browser in the signature of the callback given to the `browse` method:

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home')
              ->waitForText('Message');

        $second->loginAs(User::find(2))
               ->visit('/home')
               ->waitForText('Message')
               ->type('message', 'Hey Taylor')
               ->press('Send');

        $first->waitForText('Hey Taylor')
              ->assertSee('Jeffrey Way');
    });

#### Cambiar el tamaño del navegador de Windows : Resizing Browser Windows

Puede usar el método `resize` para ajustar el tamaño de la ventana del navegador:
> > You may use the `resize` method to adjust the size of the browser window:

    $browser->resize(1920, 1080);

El método `maximize` se puede usar para maximizar la ventana del navegador:
> > The `maximize` method may be used to maximize the browser window:

    $browser->maximize();

<a name="authentication"></a>
### Autenticación : Authentication

A menudo, probará páginas que requieren autenticación. Puede utilizar el método `loginAs` de Dusk para evitar interactuar con la pantalla de inicio de sesión durante cada prueba. El método `loginAs` acepta una ID de usuario o una instancia de modelo de usuario:
> > Often, you will be testing pages that require authentication. You can use Dusk's `loginAs` method in order to avoid interacting with the login screen during every test. The `loginAs` method accepts a user ID or user model instance:

    $this->browse(function ($first, $second) {
        $first->loginAs(User::find(1))
              ->visit('/home');
    });

> {note} Después de usar el método `loginAs`, la sesión del usuario se mantendrá para todas las pruebas dentro del archivo.
> > > {note} After using the `loginAs` method, the user session will be maintained for all tests within the file.

<a name="migrations"></a>
### Migraciones de base de datos : Database Migrations

Cuando su prueba requiere migraciones, como el ejemplo de autenticación anterior, nunca debe usar el rasgo `RefreshDatabase`. El rasgo `RefreshDatabase` aprovecha las transacciones de la base de datos que no se aplicarán en las solicitudes HTTP. En su lugar, use el rasgo `DatabaseMigrations`:
> > When your test requires migrations, like the authentication example above, you should never use the `RefreshDatabase` trait. The `RefreshDatabase` trait leverages database transactions which will not be applicable across HTTP requests. Instead, use the `DatabaseMigrations` trait:

    <?php

    namespace Tests\Browser;

    use App\User;
    use Tests\DuskTestCase;
    use Laravel\Dusk\Chrome;
    use Illuminate\Foundation\Testing\DatabaseMigrations;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;
    }

<a name="interacting-with-elements"></a>
## Interactuando con los elementos : Interacting With Elements

<a name="dusk-selectors"></a>
### Selectores de Dusk : Dusk Selectors

Elegir buenos selectores de CSS para interactuar con los elementos es una de las partes más difíciles de escribir las pruebas de Dusk. Con el tiempo, los cambios frontend pueden provocar selectores CSS como los siguientes para romper sus pruebas:
> > Choosing good CSS selectors for interacting with elements is one of the hardest parts of writing Dusk tests. Over time, frontend changes can cause CSS selectors like the following to break your tests:

    // HTML...

    <button>Login</button>

    // Test...

    $browser->click('.login-page .container div > button');

Los selectores de Dusk le permiten concentrarse en escribir pruebas efectivas en lugar de recordar los selectores de CSS. Para definir un selector, agrega un atributo `dusk` a tu elemento HTML. Luego, prefija el selector con `@` para manipular el elemento adjunto dentro de una prueba de Dusk:
> > Dusk selectors allow you to focus on writing effective tests rather than remembering CSS selectors. To define a selector, add a `dusk` attribute to your HTML element. Then, prefix the selector with `@` to manipulate the attached element within a Dusk test:

    // HTML...

    <button dusk="login-button">Login</button>

    // Test...

    $browser->click('@login-button');

<a name="clicking-links"></a>
### Hacer clic en Enlaces : Clicking Links

Para hacer clic en un enlace, puede usar el método `clickLink` en la instancia del navegador. El método `clickLink` hará clic en el enlace que tiene el texto de visualización dado:
> > To click a link, you may use the `clickLink` method on the browser instance. The `clickLink` method will click the link that has the given display text:

    $browser->clickLink($linkText);

> {note} Este método interactúa con jQuery. Si jQuery no está disponible en la página, Dusk lo inyectará automáticamente en la página para que esté disponible para la duración de la prueba.
> > > {note} This method interacts with jQuery. If jQuery is not available on the page, Dusk will automatically inject it into the page so it is available for the test's duration.

<a name="text-values-and-attributes"></a>
### Texto, valores y atributos : Text, Values, & Attributes

#### Recuperación y configuración de valores : Retrieving & Setting Values

Dusk proporciona varios métodos para interactuar con el texto de visualización actual, el valor y los atributos de los elementos en la página. Por ejemplo, para obtener el "valor" de un elemento que coincida con un selector dado, use el método `valor`:
> > Dusk provides several methods for interacting with the current display text, value, and attributes of elements on the page. For example, to get the "value" of an element that matches a given selector, use the `value` method:

    // Retrieve the value...
    $value = $browser->value('selector');

    // Set the value...
    $browser->value('selector', 'value');

#### Recuperación de texto : Retrieving Text

El método `text` se puede usar para recuperar el texto de visualización de un elemento que coincida con el selector dado:
> > The `text` method may be used to retrieve the display text of an element that matches the given selector:

    $text = $browser->text('selector');

#### Recuperando atributos : Retrieving Attributes

Finalmente, el método `attribute` se puede usar para recuperar un atributo de un elemento que coincida con el selector dado:
> > Finally, the `attribute` method may be used to retrieve an attribute of an element matching the given selector:

    $attribute = $browser->attribute('selector', 'value');

<a name="using-forms"></a>
### Uso de formularios : Using Forms

#### Valores de escritura : Typing Values

Dusk proporciona una variedad de métodos para interactuar con formularios y elementos de entrada. Primero, echemos un vistazo a un ejemplo de tipeo de texto en un campo de entrada:
> > Dusk provides a variety of methods for interacting with forms and input elements. First, let's take a look at an example of typing text into an input field:

    $browser->type('email', 'taylor@laravel.com');

Tenga en cuenta que, aunque el método acepta uno si es necesario, no estamos obligados a pasar un selector CSS al método `type`. Si no se proporciona un selector de CSS, Dusk buscará un campo de entrada con el atributo `name` dado. Finalmente, Dusk intentará encontrar un `textarea` con el atributo `name` dado.
> > Note that, although the method accepts one if necessary, we are not required to pass a CSS selector into the `type` method. If a CSS selector is not provided, Dusk will search for an input field with the given `name` attribute. Finally, Dusk will attempt to find a `textarea` with the given `name` attribute.

Para agregar texto a un campo sin borrar su contenido, puede usar el método `append`:
> > To append text to a field without clearing its content, you may use the `append` method:

    $browser->type('tags', 'foo')
            ->append('tags', ', bar, baz');

Puede borrar el valor de una entrada usando el método `clear`:
> > You may clear the value of an input using the `clear` method:

    $browser->clear('email');

#### Listas deplegables : Dropdowns

Para seleccionar un valor en un cuadro de selección desplegable, puede usar el método `select`. Al igual que el método `type`, el método `select` no requiere un selector completo de CSS. Al pasar un valor al método `select`, debe pasar el valor de la opción subyacente en lugar del texto de visualización:
> > To select a value in a dropdown selection box, you may use the `select` method. Like the `type` method, the `select` method does not require a full CSS selector. When passing a value to the `select` method, you should pass the underlying option value instead of the display text:

    $browser->select('size', 'Large');

Puede seleccionar una opción aleatoria omitiendo el segundo parámetro:
> > You may select a random option by omitting the second parameter:

    $browser->select('size');

#### Casillas de verificación : Checkboxes

Para "marcar" un campo de casilla de verificación, puede usar el método `check`. Al igual que muchos otros métodos relacionados con la entrada, no se requiere un selector completo de CSS. Si no se puede encontrar una coincidencia de selector exacta, Dusk buscará una casilla con el atributo `name` correspondiente:
> > To "check" a checkbox field, you may use the `check` method. Like many other input related methods, a full CSS selector is not required. If an exact selector match can't be found, Dusk will search for a checkbox with a matching `name` attribute:

    $browser->check('terms');

    $browser->uncheck('terms');

#### Botones de radio : Radio Buttons

Para "seleccionar" una opción de botón de radio, puede usar el método `radio`. Al igual que muchos otros métodos relacionados con la entrada, no se requiere un selector completo de CSS. Si no se puede encontrar una coincidencia de selector exacta, Dusk buscará una radio con los atributos `name` y `value` coincidentes:
> > To "select" a radio button option, you may use the `radio` method. Like many other input related methods, a full CSS selector is not required. If an exact selector match can't be found, Dusk will search for a radio with matching `name` and `value` attributes:

    $browser->radio('version', 'php7');

<a name="attaching-files"></a>
### Adjuntar archivos : Attaching Files

El método `attach` se puede usar para adjuntar un archivo a un elemento de entrada `file`. Al igual que muchos otros métodos relacionados con la entrada, no se requiere un selector completo de CSS. Si no se puede encontrar una coincidencia de selector exacta, Dusk buscará una entrada de archivo con el atributo `name` correspondiente:
> > The `attach` method may be used to attach a file to a `file` input element. Like many other input related methods, a full CSS selector is not required. If an exact selector match can't be found, Dusk will search for a file input with matching `name` attribute:

    $browser->attach('photo', __DIR__.'/photos/me.png');

> {note} La función de adjuntar requiere que la extensión de PHP `Zip` esté instalada y habilitada en su servidor.
> > > {note} The attach function requires the `Zip` PHP extension to be installed and enabled on your server.

<a name="using-the-keyboard"></a>
### Uso del teclado : Using The Keyboard

El método `keys` le permite proporcionar secuencias de entrada más complejas a un elemento dado de lo que normalmente permite el método `type`. Por ejemplo, puede mantener las teclas modificadoras introduciendo valores. En este ejemplo, la tecla `shift` se mantendrá mientras se introduce `taylor` en el elemento que coincide con el selector dado. Después de tipear `taylor`, `otwell` se tipeará sin ninguna tecla modificadora:
> > The `keys` method allows you to provide more complex input sequences to a given element than normally allowed by the `type` method. For example, you may hold modifier keys entering values. In this example, the `shift` key will be held while `taylor` is entered into the element matching the given selector. After `taylor` is typed, `otwell` will be typed without any modifier keys:

    $browser->keys('selector', ['{shift}', 'taylor'], 'otwell');

Incluso puede enviar una "tecla de acceso rápido" al selector de CSS principal que contiene su aplicación:
> > You may even send a "hot key" to the primary CSS selector that contains your application:

    $browser->keys('.app', ['{command}', 'j']);

> {tip} Todas las teclas modificadoras están envueltas en caracteres `{}`, y coinciden con las constantes definidas en la clase `Facebook\WebDriver\WebDriverKeys`, que se pueden encontrar [en GitHub](https://github.com/facebook/php-webdriver/blob/community/lib/WebDriverKeys.php).
> > > {tip} All modifier keys are wrapped in `{}` characters, and match the constants defined in the `Facebook\WebDriver\WebDriverKeys` class, which can be [found on GitHub](https://github.com/facebook/php-webdriver/blob/community/lib/WebDriverKeys.php).

<a name="using-the-mouse"></a>
### Usando el ratón : Using The Mouse

#### Hacer clic en elementos : Clicking On Elements

El método `click` se puede usar para "hacer clic" en un elemento que coincida con el selector dado:
> > The `click` method may be used to "click" on an element matching the given selector:

    $browser->click('.selector');

#### Ratón sobre : Mouseover

El método `mouseover` se puede usar cuando necesita mover el mouse sobre un elemento que coincida con el selector dado:
> > The `mouseover` method may be used when you need to move the mouse over an element matching the given selector:

    $browser->mouseover('.selector');

#### Arrastrar y soltar : Drag & Drop

El método `drag` se puede usar para arrastrar un elemento que coincida con el selector dado a otro elemento:
> > The `drag` method may be used to drag an element matching the given selector to another element:

    $browser->drag('.from-selector', '.to-selector');

O bien, puede arrastrar un elemento en una sola dirección:
> > Or, you may drag an element in a single direction:

    $browser->dragLeft('.selector', 10);
    $browser->dragRight('.selector', 10);
    $browser->dragUp('.selector', 10);
    $browser->dragDown('.selector', 10);

<a name="scoping-selectors"></a>
### Selectores de ámbito : Scoping Selectors

En ocasiones, es posible que desee realizar varias operaciones al determinar el alcance de todas las operaciones dentro de un selector determinado. Por ejemplo, puede desear afirmar que algún texto existe solo dentro de una tabla y luego hacer clic en un botón dentro de esa tabla. Puede usar el método `with` para lograr esto. Todas las operaciones que se realicen dentro de la devolución de llamada otorgada al método `with` se asignarán al selector original:
> > Sometimes you may wish to perform several operations while scoping all of the operations within a given selector. For example, you may wish to assert that some text exists only within a table and then click a button within that table. You may use the `with` method to accomplish this. All operations performed within the callback given to the `with` method will be scoped to the original selector:

    $browser->with('.table', function ($table) {
        $table->assertSee('Hello World')
              ->clickLink('Delete');
    });

<a name="waiting-for-elements"></a>
### Esperando elementos : Waiting For Elements

Al probar aplicaciones que usan JavaScript extensivamente, a menudo se hace necesario "esperar" que ciertos elementos o datos estén disponibles antes de continuar con una prueba. Dusk hace que esto sea fácil. Usando una variedad de métodos, puede esperar a que los elementos sean visibles en la página o incluso esperar hasta que una expresión de JavaScript determinada sea `true`.
> > When testing applications that use JavaScript extensively, it often becomes necessary to "wait" for certain elements or data to be available before proceeding with a test. Dusk makes this a cinch. Using a variety of methods, you may wait for elements to be visible on the page or even wait until a given JavaScript expression evaluates to `true`.

#### Esperando : Waiting

Si necesita detener la prueba durante un número determinado de milisegundos, use el método `pause`:
> > If you need to pause the test for a given number of milliseconds, use the `pause` method:

    $browser->pause(1000);

#### Esperando selectores : Waiting For Selectors

El método `waitFor` se puede usar para pausar la ejecución de la prueba hasta que el elemento que coincida con el selector CSS dado se muestre en la página. De forma predeterminada, esto detendrá la prueba durante un máximo de cinco segundos antes de lanzar una excepción. Si es necesario, puede pasar un umbral de tiempo de espera personalizado como segundo argumento para el método:
> > The `waitFor` method may be used to pause the execution of the test until the element matching the given CSS selector is displayed on the page. By default, this will pause the test for a maximum of five seconds before throwing an exception. If necessary, you may pass a custom timeout threshold as the second argument to the method:

    // Wait a maximum of five seconds for the selector...
    $browser->waitFor('.selector');

    // Wait a maximum of one second for the selector...
    $browser->waitFor('.selector', 1);

También puede esperar hasta que el selector dado falta en la página:
> > You may also wait until the given selector is missing from the page:

    $browser->waitUntilMissing('.selector');

    $browser->waitUntilMissing('.selector', 1);

#### Selectores de ámbito cuando están disponibles : Scoping Selectors When Available

Ocasionalmente, puede esperar un selector determinado y luego interactuar con el elemento que coincide con el selector. Por ejemplo, es posible que desee esperar hasta que esté disponible una ventana modal y luego presione el botón "Aceptar" dentro del modal. El método `whenAvailable` se puede usar en este caso. Todas las operaciones de elemento realizadas dentro de la rellamada dada se ajustarán al selector original:
> > Occasionally, you may wish to wait for a given selector and then interact with the element matching the selector. For example, you may wish to wait until a modal window is available and then press the "OK" button within the modal. The `whenAvailable` method may be used in this case. All element operations performed within the given callback will be scoped to the original selector:

    $browser->whenAvailable('.modal', function ($modal) {
        $modal->assertSee('Hello World')
              ->press('OK');
    });

#### Esperando texto : Waiting For Text

El método `waitForText` se puede usar para esperar hasta que se muestre el texto en la página:
> > The `waitForText` method may be used to wait until the given text is displayed on the page:

    // Wait a maximum of five seconds for the text...
    $browser->waitForText('Hello World');

    // Wait a maximum of one second for the text...
    $browser->waitForText('Hello World', 1);

#### Esperando enlaces : Waiting For Links

El método `waitForLink` se puede usar para esperar hasta que el texto del enlace se muestre en la página:
> > The `waitForLink` method may be used to wait until the given link text is displayed on the page:

    // Wait a maximum of five seconds for the link...
    $browser->waitForLink('Create');

    // Wait a maximum of one second for the link...
    $browser->waitForLink('Create', 1);

#### Esperando en la ubicación de la página : Waiting On The Page Location

Al realizar una aserción de ruta como `$browser->assertPathIs('/ home')`, la aserción puede fallar si `window.location.pathname` se actualiza de forma asincrónica. Puede usar el método `waitForLocation` para esperar que la ubicación tenga un valor determinado:
> > When making a path assertion such as `$browser->assertPathIs('/home')`, the assertion can fail if `window.location.pathname` is being updated asynchronously. You may use the `waitForLocation` method to wait for the location to be a given value:

    $browser->waitForLocation('/secret');

También puede esperar la ubicación de una ruta con nombre:
> > You may also wait for a named route's location:

    $browser->waitForRoute($routeName, $parameters);

#### Esperando la recarga de página : Waiting for Page Reloads

Si necesita hacer afirmaciones después de que una página ha sido recargada, use el método `waitForReload`:
> > If you need to make assertions after a page has been reloaded, use the `waitForReload` method:

    $browser->click('.some-action')
            ->waitForReload()
            ->assertSee('something');

#### Esperando en expresiones de JavaScript : Waiting On JavaScript Expressions

En ocasiones, es posible que desee pausar la ejecución de una prueba hasta que una expresión de JavaScript dada se evalúe como 'verdadera'. Puede lograrlo fácilmente utilizando el método `waitUntil`. Al pasar una expresión a este método, no es necesario que incluya la palabra clave `return` o un punto y coma final:
> > Sometimes you may wish to pause the execution of a test until a given JavaScript expression evaluates to `true`. You may easily accomplish this using the `waitUntil` method. When passing an expression to this method, you do not need to include the `return` keyword or an ending semi-colon:

    // Wait a maximum of five seconds for the expression to be true...
    $browser->waitUntil('App.dataLoaded');

    $browser->waitUntil('App.data.servers.length > 0');

    // Wait a maximum of one second for the expression to be true...
    $browser->waitUntil('App.data.servers.length > 0', 1);

#### Esperando con una rellamada : Waiting With A Callback

Muchos de los métodos de "espera" en Dusk dependen del método subyacente de 'waitUsing'. Puede usar este método directamente para esperar a que una devolución de llamada determinada devuelva `true`. El método `waitUsing` acepta la cantidad máxima de segundos para esperar, el intervalo en el que se debe evaluar el Cierre, el Cierre y un mensaje opcional de falla:
> > Many of the "wait" methods in Dusk rely on the underlying `waitUsing` method. You may use this method directly to wait for a given callback to return `true`. The `waitUsing` method accepts the maximum number of seconds to wait, the interval at which the Closure should be evaluated, the Closure, and an optional failure message:

    $browser->waitUsing(10, 1, function () use ($something) {
        return $something->isReady();
    }, "Something wasn't ready in time.");

<a name="making-vue-assertions"></a>
### Hacer afirmaciones Vue : Making Vue Assertions

Dusk incluso te permite hacer afirmaciones sobre el estado de los datos del componente [Vue](https://vuejs.org). Por ejemplo, imagine que su aplicación contiene el siguiente componente Vue:
> > Dusk even allows you to make assertions on the state of [Vue](https://vuejs.org) component data. For example, imagine your application contains the following Vue component:

    // HTML...

    <profile dusk="profile-component"></profile>

    // Component Definition...

    Vue.component('profile', {
        template: '<div>{{ user.name }}</div>',

        data: function () {
            return {
                user: {
                  name: 'Taylor'
                }
            };
        }
    });

Puede confirmar el estado del componente Vue de la siguiente manera:
> > You may assert on the state of the Vue component like so:

    /**
     * A basic Vue test example.
     *
     * @return void
     */
    public function testVue()
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                    ->assertVue('user.name', 'Taylor', '@profile-component');
        });
    }

<a name="available-assertions"></a>
## Afirmaciones disponibles : Available Assertions

Dusk proporciona una variedad de afirmaciones que puede realizar en contra de su aplicación. Todas las aserciones disponibles están documentadas en la lista a continuación:
> > Dusk provides a variety of assertions that you may make against your application. All of the available assertions are documented in the list below:

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">
[assertTitle](#assert-title)
[assertTitleContains](#assert-title-contains)
[assertUrlIs](#assert-url-is)
[assertPathBeginsWith](#assert-path-begins-with)
[assertPathIs](#assert-path-is)
[assertPathIsNot](#assert-path-is-not)
[assertRouteIs](#assert-route-is)
[assertQueryStringHas](#assert-query-string-has)
[assertQueryStringMissing](#assert-query-string-missing)
[assertFragmentIs](#assert-fragment-is)
[assertFragmentBeginsWith](#assert-fragment-begins-with)
[assertFragmentIsNot](#assert-fragment-is-not)
[assertHasCookie](#assert-has-cookie)
[assertCookieMissing](#assert-cookie-missing)
[assertCookieValue](#assert-cookie-value)
[assertPlainCookieValue](#assert-plain-cookie-value)
[assertSee](#assert-see)
[assertDontSee](#assert-dont-see)
[assertSeeIn](#assert-see-in)
[assertDontSeeIn](#assert-dont-see-in)
[assertSourceHas](#assert-source-has)
[assertSourceMissing](#assert-source-missing)
[assertSeeLink](#assert-see-link)
[assertDontSeeLink](#assert-dont-see-link)
[assertInputValue](#assert-input-value)
[assertInputValueIsNot](#assert-input-value-is-not)
[assertChecked](#assert-checked)
[assertNotChecked](#assert-not-checked)
[assertRadioSelected](#assert-radio-selected)
[assertRadioNotSelected](#assert-radio-not-selected)
[assertSelected](#assert-selected)
[assertNotSelected](#assert-not-selected)
[assertSelectHasOptions](#assert-select-has-options)
[assertSelectMissingOptions](#assert-select-missing-options)
[assertSelectHasOption](#assert-select-has-option)
[assertValue](#assert-value)
[assertVisible](#assert-visible)
[assertPresent](#assert-present)
[assertMissing](#assert-missing)
[assertDialogOpened](#assert-dialog-opened)
[assertEnabled](#assert-enabled)
[assertDisabled](#assert-disabled)
[assertFocused](#assert-focused)
[assertNotFocused](#assert-not-focused)
[assertVue](#assert-vue)
[assertVueIsNot](#assert-vue-is-not)
[assertVueContains](#assert-vue-contains)
[assertVueDoesNotContain](#assert-vue-does-not-contain)
</div>

<a name="assert-title"></a>
#### assertTitle

Afirma que el título de la página coincide con el texto dado:
> > Assert the page title matches the given text:

    $browser->assertTitle($title);

<a name="assert-title-contains"></a>
#### assertTitleContains

Afirma que el título de la página contiene el texto dado:
> > Assert the page title contains the given text:

    $browser->assertTitleContains($title);

<a name="assert-url-is"></a>
#### assertUrlIs

Afirma que la URL actual (sin la cadena de consulta) coincide con la cadena dada:
> > Assert that the current URL (without the query string) matches the given string:

    $browser->assertUrlIs($url);

<a name="assert-path-begins-with"></a>
#### assertPathBeginsWith

Afirma que la ruta actual de la URL comienza con una ruta determinada:
> > Assert that the current URL path begins with given path:

    $browser->assertPathBeginsWith($path);

<a name="assert-path-is"></a>
#### assertPathIs

Afirma que la ruta actual coincide con la ruta dada:
> > Assert the current path matches the given path:

    $browser->assertPathIs('/home');

<a name="assert-path-is-not"></a>
#### assertPathIsNot

Afirma que la ruta actual no coincide con la ruta dada:
>>Assert the current path does not match the given path:

    $browser->assertPathIsNot('/home');

<a name="assert-route-is"></a>
#### assertRouteIs

Afirma que la URL actual coincida con la URL de la ruta con nombre especificada:
> > Assert the current URL matches the given named route's URL:

    $browser->assertRouteIs($name, $parameters);

<a name="assert-query-string-has"></a>
#### assertQueryStringHas

Afirma que el parámetro de cadena de consulta dado está presente:
> > Assert the given query string parameter is present:

    $browser->assertQueryStringHas($name);

Afirma que el parámetro de cadena de consulta dado está presente y tiene un valor dado:
> > Assert the given query string parameter is present and has a given value:

    $browser->assertQueryStringHas($name, $value);

<a name="assert-query-string-missing"></a>
#### assertQueryStringMissing

Afirma que falta el parámetro de cadena de consulta dado:
> > Assert the given query string parameter is missing:

    $browser->assertQueryStringMissing($name);
    
<a name="assert-fragment-is"></a>
#### assertFragmentIs

Afirma que el fragmento actual coincide con el fragmento dado:
> > Assert the current fragment matches the given fragment:

    $browser->assertFragmentIs('anchor');
    
<a name="assert-fragment-begins-with"></a>
#### assertFragmentBeginsWith

Afirma de que el fragmento actual comience con un fragmento dado:
> > Assert that the current fragment begins with given fragment:

    $browser->assertFragmentBeginsWith('anchor');
    
<a name="assert-fragment-is-not"></a>
#### assertFragmentIsNot

Afirma que el fragmento actual no coincide con el fragmento dado:
> > Assert the current fragment does not match the given fragment:

    $browser->assertFragmentIsNot('anchor');

<a name="assert-has-cookie"></a>
#### assertHasCookie

Afirma que la cookie dada está presente:
> > Assert the given cookie is present:

    $browser->assertHasCookie($name);

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

Afirma de que la cookie dada no está presente:
> > Assert that the given cookie is not present:

    $browser->assertCookieMissing($name);

<a name="assert-cookie-value"></a>
#### assertCookieValue

Afirma que una cookie tiene un valor dado:
> > Assert a cookie has a given value:

    $browser->assertCookieValue($name, $value);

<a name="assert-plain-cookie-value"></a>
#### assertPlainCookieValue

Afirma una cookie no encriptada tiene un valor dado:
> > Assert an unencrypted cookie has a given value:

    $browser->assertPlainCookieValue($name, $value);

<a name="assert-see"></a>
#### assertSee

Afirma que el texto dado está presente en la página:
> > Assert the given text is present on the page:

    $browser->assertSee($text);

<a name="assert-dont-see"></a>
#### assertDontSee

Afirma que el texto dado no está presente en la página:
> > Assert the given text is not present on the page:

    $browser->assertDontSee($text);

<a name="assert-see-in"></a>
#### assertSeeIn

Afirma que el texto dado está presente dentro del selector:
> > Assert the given text is present within the selector:

    $browser->assertSeeIn($selector, $text);

<a name="assert-dont-see-in"></a>
#### assertDontSeeIn

Afirma que el texto dado no está presente en el selector:
> > Assert the given text is not present within the selector:

    $browser->assertDontSeeIn($selector, $text);

<a name="assert-source-has"></a>
#### assertSourceHas

Afirma de que el código fuente dado esté presente en la página:
> > Assert that the given source code is present on the page:

    $browser->assertSourceHas($code);

<a name="assert-source-missing"></a>
#### assertSourceMissing

Afirma que el código fuente dado no está presente en la página:
> > Assert that the given source code is not present on the page:

    $browser->assertSourceMissing($code);

<a name="assert-see-link"></a>
#### assertSeeLink

Afirma que el enlace dado está presente en la página:
> > Assert the given link is present on the page:

    $browser->assertSeeLink($linkText);

<a name="assert-dont-see-link"></a>
#### assertDontSeeLink

Afirma que el enlace dado no está presente en la página:
> > Assert the given link is not present on the page:

    $browser->assertDontSeeLink($linkText);

<a name="assert-input-value"></a>
#### assertInputValue

Afirma que el campo de entrada dado tiene el valor dado:
> > Assert the given input field has the given value:

    $browser->assertInputValue($field, $value);

<a name="assert-input-value-is-not"></a>
#### assertInputValueIsNot

Afirma que el campo de entrada dado no tiene el valor dado:
> > Assert the given input field does not have the given value:

    $browser->assertInputValueIsNot($field, $value);

<a name="assert-checked"></a>
#### assertChecked

Afirma que la casilla de verificación está marcada:
> > Assert the given checkbox is checked:

    $browser->assertChecked($field);

<a name="assert-not-checked"></a>
#### assertNotChecked

Afirma que la casilla de verificación dada no está marcada:
> > Assert the given checkbox is not checked:

    $browser->assertNotChecked($field);

<a name="assert-radio-selected"></a>
#### assertRadioSelected

Afirma que el campo de radio dado está seleccionado:
> > Assert the given radio field is selected:

    $browser->assertRadioSelected($field, $value);

<a name="assert-radio-not-selected"></a>
#### assertRadioNotSelected

Afirma que el campo de radio dado no está seleccionado:
> > Assert the given radio field is not selected:

    $browser->assertRadioNotSelected($field, $value);

<a name="assert-selected"></a>
#### assertSelected

Afirma que el menú desplegable dado tiene el valor dado seleccionado:
> > Assert the given dropdown has the given value selected:

    $browser->assertSelected($field, $value);

<a name="assert-not-selected"></a>
#### assertNotSelected

Afirma que el menú desplegable dado no tiene el valor dado seleccionado:
> > Assert the given dropdown does not have the given value selected:

    $browser->assertNotSelected($field, $value);

<a name="assert-select-has-options"></a>
#### assertSelectHasOptions

Afirma que el array de valores dado esté disponible para ser seleccionado:
> > Assert that the given array of values are available to be selected:

    $browser->assertSelectHasOptions($field, $values);

<a name="assert-select-missing-options"></a>
#### assertSelectMissingOptions

Afirma que el array de valores dado no esté disponible para ser seleccionado:
> > Assert that the given array of values are not available to be selected:

    $browser->assertSelectMissingOptions($field, $values);

<a name="assert-select-has-option"></a>
#### assertSelectHasOption

Afirma que el valor dado está disponible para ser seleccionado en el campo dado:
> > Assert that the given value is available to be selected on the given field:

    $browser->assertSelectHasOption($field, $value);

<a name="assert-value"></a>
#### assertValue

Afirma que el elemento que coincide con el selector dado tiene el valor dado:
> > Assert the element matching the given selector has the given value:

    $browser->assertValue($selector, $value);

<a name="assert-visible"></a>
#### assertVisible

Afirma que el elemento que coincide con el selector dado es visible:
> > Assert the element matching the given selector is visible:

    $browser->assertVisible($selector);

<a name="assert-present"></a>
#### assertPresent

Afirma que el elemento que coincide con el selector dado está presente:
> > Assert the element matching the given selector is present:

    $browser->assertPresent($selector);

<a name="assert-missing"></a>
#### assertMissing

Afirma que el elemento que coincide con el selector dado no está visible:
> > Assert the element matching the given selector is not visible:

    $browser->assertMissing($selector);

<a name="assert-dialog-opened"></a>
#### assertDialogOpened

Afirma que se ha abierto un cuadro de diálogo de JavaScript con un mensaje determinado:
> > Assert that a JavaScript dialog with given message has been opened:

    $browser->assertDialogOpened($message);

<a name="assert-enabled"></a>
#### assertEnabled

Afirma de que el campo dado esté habilitado:
> > Assert that the given field is enabled:

    $browser->assertEnabled($field);

<a name="assert-disabled"></a>
#### assertDisabled

Afirma que el campo dado esté deshabilitado:
> > Assert that the given field is disabled:

    $browser->assertDisabled($field);

<a name="assert-focused"></a>
#### assertFocused

Afirma que el campo dado tiene el foco:
> > Assert that the given field is focused:

    $browser->assertFocused($field);

<a name="assert-not-focused"></a>
#### assertNotFocused

Afirma que el campo dado no tiene el foco:
> > Assert that the given field is not focused:

    $browser->assertNotFocused($field);

<a name="assert-vue"></a>
#### assertVue

Afirma que una propiedad de datos del componente Vue dada coincide con el valor dado:
> > Assert that a given Vue component data property matches the given value:

    $browser->assertVue($property, $value, $componentSelector = null);

<a name="assert-vue-is-not"></a>
#### assertVueIsNot

Afirma que una propiedad de datos del componente Vue dada no coincide con el valor dado:
> > Assert that a given Vue component data property does not match the given value:

    $browser->assertVueIsNot($property, $value, $componentSelector = null);

<a name="assert-vue-contains"></a>
#### assertVueContains

Afirma que una propiedad de datos del componente Vue dada es un array y contiene el valor dado:
> > Assert that a given Vue component data property is an array and contains the given value:

    $browser->assertVueContains($property, $value, $componentSelector = null);

<a name="assert-vue-does-not-contain"></a>
#### assertVueDoesNotContain

Afirma que una propiedad de datos del componente Vue dada es un array y no contiene el valor dado:
> > Assert that a given Vue component data property is an array and does not contain the given value:

    $browser->assertVueDoesNotContain($property, $value, $componentSelector = null);

<a name="pages"></a>
## Páginas : Pages

A veces, las pruebas requieren varias acciones complicadas para realizarse en secuencia. Esto puede hacer que tus pruebas sean más difíciles de leer y entender. Las páginas le permiten definir acciones expresivas que luego pueden realizarse en una página determinada con un único método. Las páginas también le permiten definir accesos directos a selectores comunes para su aplicación o una sola página.
> > Sometimes, tests require several complicated actions to be performed in sequence. This can make your tests harder to read and understand. Pages allow you to define expressive actions that may then be performed on a given page using a single method. Pages also allow you to define short-cuts to common selectors for your application or a single page.

<a name="generating-pages"></a>
### Generando páginas : Generating Pages

Para generar un objeto de página, use el comando Artisan 'dusk:page`. Todos los objetos de la página se colocarán en el directorio `tests/Browser/Pages`:
> > To generate a page object, use the `dusk:page` Artisan command. All page objects will be placed in the `tests/Browser/Pages` directory:

    php artisan dusk:page Login

<a name="configuring-pages"></a>
### Configuración de páginas : Configuring Pages

Por defecto, las páginas tienen tres métodos: `url`,` assert` y `elements`. Discutiremos los métodos `url` y `assert` ahora. El método de los "elementos" será [discutido en más detalle a continuación](#shorthand-selectors).
> > By default, pages have three methods: `url`, `assert`, and `elements`. We will discuss the `url` and `assert` methods now. The `elements` method will be [discussed in more detail below](#shorthand-selectors).

#### El método `url` : The `url` Method

El método `url` debe devolver la ruta de la URL que representa la página. Dusk usará esta URL cuando navegue a la página en el navegador:
> > The `url` method should return the path of the URL that represents the page. Dusk will use this URL when navigating to the page in the browser:

    /**
     * Get the URL for the page.
     *
     * @return string
     */
    public function url()
    {
        return '/login';
    }

#### El método 'assert' : The `assert` Method

El método 'assert' puede hacer cualquier afirmación necesaria para verificar que el navegador está realmente en la página dada. Completar este método no es necesario; sin embargo, puede hacer estas afirmaciones si lo desea. Estas afirmaciones se ejecutarán automáticamente al navegar a la página:
> > The `assert` method may make any assertions necessary to verify that the browser is actually on the given page. Completing this method is not necessary; however, you are free to make these assertions if you wish. These assertions will be run automatically when navigating to the page:

    /**
     * Assert that the browser is on the page.
     *
     * @return void
     */
    public function assert(Browser $browser)
    {
        $browser->assertPathIs($this->url());
    }

<a name="navigating-to-pages"></a>
### Navegación a páginas : Navigating To Pages

Una vez que se ha configurado una página, puede navegar hacia ella utilizando el método `visit`:
> > Once a page has been configured, you may navigate to it using the `visit` method:

    use Tests\Browser\Pages\Login;

    $browser->visit(new Login);

En ocasiones, es posible que ya esté en una página determinada y necesite "cargar" los selectores y métodos de la página en el contexto de prueba actual. Esto es común cuando se presiona un botón y se lo redirecciona a una página dada sin navegar explícitamente hacia él. En esta situación, puede usar el método `on` para cargar la página:
> > Sometimes you may already be on a given page and need to "load" the page's selectors and methods into the current test context. This is common when pressing a button and being redirected to a given page without explicitly navigating to it. In this situation, you may use the `on` method to load the page:

    use Tests\Browser\Pages\CreatePlaylist;

    $browser->visit('/dashboard')
            ->clickLink('Create Playlist')
            ->on(new CreatePlaylist)
            ->assertSee('@create');

<a name="shorthand-selectors"></a>
### Selectores abreviados : Shorthand Selectors

El método `elements` de páginas le permite definir accesos directos rápidos y fáciles de recordar para cualquier selector de CSS en su página. Por ejemplo, definamos un atajo para el campo de entrada "email" de la página de inicio de sesión de la aplicación:
> > The `elements` method of pages allows you to define quick, easy-to-remember shortcuts for any CSS selector on your page. For example, let's define a shortcut for the "email" input field of the application's login page:

    /**
     * Get the element shortcuts for the page.
     *
     * @return array
     */
    public function elements()
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }

Ahora, puedes usar este selector abreviado en cualquier lugar donde uses un selector completo de CSS:
> > Now, you may use this shorthand selector anywhere you would use a full CSS selector:

    $browser->type('@email', 'taylor@laravel.com');

#### Selectores abreviados globales : Global Shorthand Selectors

Después de instalar Dusk, se colocará una clase base de "Página" en el directorio `tests/Browser/Pages`. Esta clase contiene un método `siteElements` que se puede utilizar para definir selectores abreviados globales que deberían estar disponibles en cada página de la aplicación:
> > After installing Dusk, a base `Page` class will be placed in your `tests/Browser/Pages` directory. This class contains a `siteElements` method which may be used to define global shorthand selectors that should be available on every page throughout your application:

    /**
     * Get the global element shortcuts for the site.
     *
     * @return array
     */
    public static function siteElements()
    {
        return [
            '@element' => '#selector',
        ];
    }

<a name="page-methods"></a>
### Métodos de página : Page Methods

Además de los métodos predeterminados definidos en las páginas, puede definir métodos adicionales que pueden usarse a lo largo de sus pruebas. Por ejemplo, imaginemos que estamos construyendo una aplicación de administración de música. Una acción común para una página de la aplicación podría ser crear una lista de reproducción. En lugar de volver a escribir la lógica para crear una lista de reproducción en cada prueba, puede definir un método `createPlaylist` en una clase de página:
> > In addition to the default methods defined on pages, you may define additional methods which may be used throughout your tests. For example, let's imagine we are building a music management application. A common action for one page of the application might be to create a playlist. Instead of re-writing the logic to create a playlist in each test, you may define a `createPlaylist` method on a page class:

    <?php

    namespace Tests\Browser\Pages;

    use Laravel\Dusk\Browser;

    class Dashboard extends Page
    {
        // Other page methods...

        /**
         * Create a new playlist.
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  string  $name
         * @return void
         */
        public function createPlaylist(Browser $browser, $name)
        {
            $browser->type('name', $name)
                    ->check('share')
                    ->press('Create Playlist');
        }
    }

Una vez que se haya definido el método, puede usarlo dentro de cualquier prueba que utilice la página. La instancia del navegador se pasará automáticamente al método de la página:
> > Once the method has been defined, you may use it within any test that utilizes the page. The browser instance will automatically be passed to the page method:

    use Tests\Browser\Pages\Dashboard;

    $browser->visit(new Dashboard)
            ->createPlaylist('My Playlist')
            ->assertSee('My Playlist');

<a name="components"></a>
## Componentes : Components

Los componentes son similares a los "objetos de página" de Dusk, pero están destinados a piezas de UI y funcionalidades que se reutilizan en toda la aplicación, como una barra de navegación o una ventana de notificación. Como tal, los componentes no están vinculados a URL específicas.
> > Components are similar to Dusk’s “page objects”, but are intended for pieces of UI and functionality that are re-used throughout your application, such as a navigation bar or notification window. As such, components are not bound to specific URLs.

<a name="generating-components"></a>
### Generación de componentes : Generating Components

Para generar un componente, use el comando Artisan 'dusk:component`. Los nuevos componentes se colocan en el directorio `test/Browser/Components`:
> > To generate a component, use the `dusk:component` Artisan command. New components are placed in the `test/Browser/Components` directory:

    php artisan dusk:component DatePicker

Como se muestra arriba, un "selector de fecha" es un ejemplo de un componente que podría existir en toda su aplicación en una variedad de páginas. Puede ser engorroso escribir manualmente la lógica de automatización del navegador para seleccionar una fecha en docenas de pruebas en todo su conjunto de pruebas. En su lugar, podemos definir un componente Dusk para representar el selector de fecha, lo que nos permite encapsular esa lógica dentro del componente:
> > As shown above, a "date picker" is an example of a component that might exist throughout your application on a variety of pages. It can become cumbersome to manually write the browser automation logic to select a date in dozens of tests throughout your test suite. Instead, we can define a Dusk component to represent the date picker, allowing us to encapsulate that logic within the component:

    <?php

    namespace Tests\Browser\Components;

    use Laravel\Dusk\Browser;
    use Laravel\Dusk\Component as BaseComponent;

    class DatePicker extends BaseComponent
    {
        /**
         * Get the root selector for the component.
         *
         * @return string
         */
        public function selector()
        {
            return '.date-picker';
        }

        /**
         * Assert that the browser page contains the component.
         *
         * @param  Browser  $browser
         * @return void
         */
        public function assert(Browser $browser)
        {
            $browser->assertVisible($this->selector());
        }

        /**
         * Get the element shortcuts for the component.
         *
         * @return array
         */
        public function elements()
        {
            return [
                '@date-field' => 'input.datepicker-input',
                '@month-list' => 'div > div.datepicker-months',
                '@day-list' => 'div > div.datepicker-days',
            ];
        }

        /**
         * Select the given date.
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  int  $month
         * @param  int  $year
         * @return void
         */
        public function selectDate($browser, $month, $year)
        {
            $browser->click('@date-field')
                    ->within('@month-list', function ($browser) use ($month) {
                        $browser->click($month);
                    })
                    ->within('@day-list', function ($browser) use ($day) {
                        $browser->click($day);
                    });
        }
    }

<a name="using-components"></a>
### Uso de componentes : Using Components

Una vez que se ha definido el componente, podemos seleccionar fácilmente una fecha dentro del selector de fecha de cualquier prueba. Y, si la lógica necesaria para seleccionar una fecha cambia, solo necesitamos actualizar el componente:
> > Once the component has been defined, we can easily select a date within the date picker from any test. And, if the logic necessary to select a date changes, we only need to update the component:

    <?php

    namespace Tests\Browser;

    use Tests\DuskTestCase;
    use Laravel\Dusk\Browser;
    use Tests\Browser\Components\DatePicker;
    use Illuminate\Foundation\Testing\DatabaseMigrations;

    class ExampleTest extends DuskTestCase
    {
        /**
         * A basic component test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->browse(function (Browser $browser) {
                $browser->visit('/')
                        ->within(new DatePicker, function ($browser) {
                            $browser->selectDate(1, 2018);
                        })
                        ->assertSee('January');
            });
        }
    }

<a name="continuous-integration"></a>
## Integración continua : Continuous Integration

<a name="running-tests-on-circle-ci"></a>
### CircleCI

#### CircleCI 1.0

Si está utilizando CircleCI 1.0 para ejecutar sus pruebas de Dusk, puede usar este archivo de configuración como punto de partida. Al igual que TravisCI, usaremos el comando `php artisan serve` para iniciar el servidor web incorporado de PHP:
> > If you are using CircleCI 1.0 to run your Dusk tests, you may use this configuration file as a starting point. Like TravisCI, we will use the `php artisan serve` command to launch PHP's built-in web server:

    dependencies:
      pre:
          - curl -L -o google-chrome.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
          - sudo dpkg -i google-chrome.deb
          - sudo sed -i 's|HERE/chrome\"|HERE/chrome\" --disable-setuid-sandbox|g' /opt/google/chrome/google-chrome
          - rm google-chrome.deb

    test:
        pre:
            - "./vendor/laravel/dusk/bin/chromedriver-linux":
                background: true
            - cp .env.testing .env
            - "php artisan serve":
                background: true

        override:
            - php artisan dusk

#### CircleCI 2.0

Si está utilizando CircleCI 2.0 para ejecutar sus pruebas de Dusk, puede agregar estos pasos a su compilación:
> > If you are using CircleCI 2.0 to run your Dusk tests, you may add these steps to your build:

     version: 2
     jobs:
         build:
             steps:
                - run: sudo apt-get install -y libsqlite3-dev
                - run: cp .env.testing .env
                - run: composer install -n --ignore-platform-reqs
                - run: npm install
                - run: npm run production
                - run: vendor/bin/phpunit

                - run:
                   name: Start Chrome Driver
                   command: ./vendor/laravel/dusk/bin/chromedriver-linux
                   background: true

                - run:
                   name: Run Laravel Server
                   command: php artisan serve
                   background: true

                - run:
                   name: Run Laravel Dusk Tests
                   command: php artisan dusk

<a name="running-tests-on-codeship"></a>
### Codeship

Para ejecutar las pruebas de Dusk en [Codeship](https://codeship.com), agregue los siguientes comandos a su proyecto de Codeship. Por supuesto, estos comandos son un punto de partida y puede agregar comandos adicionales según sea necesario:
> > To run Dusk tests on [Codeship](https://codeship.com), add the following commands to your Codeship project. Of course, these commands are a starting point and you are free to add additional commands as needed:

    phpenv local 7.1
    cp .env.testing .env
    composer install --no-interaction
    nohup bash -c "./vendor/laravel/dusk/bin/chromedriver-linux 2>&1 &"
    nohup bash -c "php artisan serve 2>&1 &" && sleep 5
    php artisan dusk

<a name="running-tests-on-heroku-ci"></a>
### Heroku CI

Para ejecutar las pruebas de Dusk en [Heroku CI](https://www.heroku.com/continuous-integration), agregue los siguientes buildpack y scripts de Google Chrome a su archivo Heroku `app.json`:
> > To run Dusk tests on [Heroku CI](https://www.heroku.com/continuous-integration), add the following Google Chrome buildpack and scripts to your Heroku `app.json` file:

    {
      "environments": {
        "test": {
          "buildpacks": [
            { "url": "heroku/php" },
            { "url": "https://github.com/heroku/heroku-buildpack-google-chrome" }
          ],
          "scripts": {
            "test-setup": "cp .env.testing .env",
            "test": "nohup bash -c './vendor/laravel/dusk/bin/chromedriver-linux > /dev/null 2>&1 &' && nohup bash -c 'php artisan serve > /dev/null 2>&1 &' && php artisan dusk"
          }
        }
      }
    }

<a name="running-tests-on-travis-ci"></a>
### Travis CI

Para ejecutar tus pruebas de Dusk en Travis CI, necesitaremos usar el entorno "Ubuntu habilitado para sudo" de Ubuntu 14.04 (Trusty). Debido a que Travis CI no es un entorno gráfico, tendremos que dar algunos pasos adicionales para poder abrir un navegador Chrome. Además, usaremos `php artisan serve` para lanzar el servidor web incorporado de PHP:
> > To run your Dusk tests on Travis CI, we will need to use the "sudo-enabled" Ubuntu 14.04 (Trusty) environment. Since Travis CI is not a graphical environment, we will need to take some extra steps in order to launch a Chrome browser. In addition, we will use `php artisan serve` to launch PHP's built-in web server:

    sudo: required
    dist: trusty

    addons:
       chrome: stable

    install:
       - cp .env.testing .env
       - travis_retry composer install --no-interaction --prefer-dist --no-suggest

    before_script:
       - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
       - php artisan serve &

    script:
       - php artisan dusk
