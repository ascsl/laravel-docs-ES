# Consola Artisan : Artisan Console

- [Introduction](#introduction)
- [Writing Commands](#writing-commands)
    - [Generating Commands](#generating-commands)
    - [Command Structure](#command-structure)
    - [Closure Commands](#closure-commands)
- [Defining Input Expectations](#defining-input-expectations)
    - [Arguments](#arguments)
    - [Options](#options)
    - [Input Arrays](#input-arrays)
    - [Input Descriptions](#input-descriptions)
- [Command I/O](#command-io)
    - [Retrieving Input](#retrieving-input)
    - [Prompting For Input](#prompting-for-input)
    - [Writing Output](#writing-output)
- [Registering Commands](#registering-commands)
- [Programmatically Executing Commands](#programmatically-executing-commands)
    - [Calling Commands From Other Commands](#calling-commands-from-other-commands)

<a name="introduction"></a>
## Introducción : Introduction

Artisan es la interfaz de línea de comandos incluida con Laravel. Proporciona una serie de útiles comandos que pueden ayudarle mientras construye su aplicación. Para ver una lista de todos los comandos Artisan disponibles, puede usar el comando `list`:
> > Artisan is the command-line interface included with Laravel. It provides a number of helpful commands that can assist you while you build your application. To view a list of all available Artisan commands, you may use the `list` command:

    php artisan list

Cada comando también incluye una pantalla de "ayuda" que muestra y describe los argumentos y opciones disponibles del comando. Para ver una pantalla de ayuda, preceda el nombre del comando con `help`:
> > Every command also includes a "help" screen which displays and describes the command's available arguments and options. To view a help screen, precede the name of the command with `help`:

    php artisan help migrate

#### Laravel REPL : Laravel REPL

Todas las aplicaciones de Laravel incluyen Tinker, un REPL impulsado por el paquete [PsySH] (https://github.com/bobthecow/psysh). Tinker te permite interactuar con toda tu aplicación Laravel en la línea de comandos, incluido el ORM Eloquent, trabajos, eventos y más. Para ingresar al entorno de Tinker, ejecuta el comando Artisan `tinker`:
> > All Laravel applications include Tinker, a REPL powered by the [PsySH](https://github.com/bobthecow/psysh) package. Tinker allows you to interact with your entire Laravel application on the command line, including the Eloquent ORM, jobs, events, and more. To enter the Tinker environment, run the `tinker` Artisan command:

    php artisan tinker

<a name="writing-commands"></a>
## Escribiendo comandos : Writing Commands

Además de los comandos provistos con Artisan, también puedes construir tus propios comandos personalizados. Los comandos se almacenan típicamente en el directorio `app/Console/Commands`; sin embargo, puede elegir tu propia ubicación de almacenamiento siempre que tus comandos puedan ser cargados por Composer.
> > In addition to the commands provided with Artisan, you may also build your own custom commands. Commands are typically stored in the `app/Console/Commands` directory; however, you are free to choose your own storage location as long as your commands can be loaded by Composer.

<a name="generating-commands"></a>
### Generación de comandos : Generating Commands

Para crear un nuevo comando, use el comando Artisan `make:command`. Este comando creará una nueva clase de comando en el directorio `app/Console/Commands`. No se preocupe si este directorio no existe en su aplicación, ya que se creará la primera vez que ejecute el comando Artisan `make:command`. El comando generado incluirá el conjunto predeterminado de propiedades y métodos que están presentes en todos los comandos:
> > To create a new command, use the `make:command` Artisan command. This command will create a new command class in the `app/Console/Commands` directory. Don't worry if this directory does not exist in your application, since it will be created the first time you run the `make:command` Artisan command. The generated command will include the default set of properties and methods that are present on all commands:

    php artisan make:command SendEmails

<a name="command-structure"></a>
### Estructura de comando : Command Structure

Después de generar su comando, debe completar las propiedades `signature` y `description` de la clase, que se usarán al mostrar su comando en la pantalla `list`. Se llamará al método `handle` cuando se ejecute su comando. Puede colocar su lógica de comando en este método.
> > After generating your command, you should fill in the `signature` and `description` properties of the class, which will be used when displaying your command on the `list` screen. The `handle` method will be called when your command is executed. You may place your command logic in this method.  

> {tip} Para una mayor reutilización del código, es una buena práctica mantener los comandos de la consola a la ligera y permitir que difieran de los servicios de la aplicación para realizar sus tareas. En el ejemplo a continuación, tenga en cuenta que inyectamos una clase de servicio para hacer el "levantamiento pesado" de enviar los correos electrónicos.
> > > {tip} For greater code reuse, it is good practice to keep your console commands light and let them defer to application services to accomplish their tasks. In the example below, note that we inject a service class to do the "heavy lifting" of sending the e-mails.  

Echemos un vistazo a un comando de ejemplo. Tenga en cuenta que podemos insertar cualquier dependencia que necesitemos en el constructor del comando o en el método `handle`. Laravel [service container] (/docs/{{version}}/container) inyectará automáticamente todas las dependencias de tipo insinuadas en el constructor o en el método `handle`:
> > Let's take a look at an example command. Note that we are able to inject any dependencies we need into the command's constructor or `handle` method. The Laravel [service container](/docs/{{version}}/container) will automatically inject all dependencies type-hinted in the constructor or `handle` method:  

    <?php

    namespace App\Console\Commands;

    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'email:send {user}';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Send drip e-mails to a user';

        /**
         * The drip e-mail service.
         *
         * @var DripEmailer
         */
        protected $drip;

        /**
         * Create a new command instance.
         *
         * @param  DripEmailer  $drip
         * @return void
         */
        public function __construct(DripEmailer $drip)
        {
            parent::__construct();

            $this->drip = $drip;
        }

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            $this->drip->send(User::find($this->argument('user')));
        }
    }

<a name="closure-commands"></a>
### Comandos de cierre : Closure Commands

Los comandos basados ​​en cierre proporcionan una alternativa para definir comandos de consola como clases. De la misma manera que los Closures de ruta son una alternativa a los controladores, piense en los cierres de comandos como una alternativa a las clases de comando. Dentro del método `commands` del archivo `app/Console/Kernel.php`, Laravel carga el archivo `routes/console.php`:
> > Closure based commands provide an alternative to defining console commands as classes. In the same way that route Closures are an alternative to controllers, think of command Closures as an alternative to command classes. Within the `commands` method of your `app/Console/Kernel.php` file, Laravel loads the `routes/console.php` file:  

    /**
     * Register the Closure based commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

Aunque este archivo no define rutas HTTP, define los puntos de entrada (rutas) basados ​​en la consola en su aplicación. Dentro de este archivo, puede definir todas sus rutas basadas en Closure utilizando el método `Artisan::command`. El método `command` acepta dos argumentos: el [command signature] (# definition-input-expectations) y un Closure que recibe los comandos argumentos y opciones:
> > Even though this file does not define HTTP routes, it defines console based entry points (routes) into your application. Within this file, you may define all of your Closure based routes using the `Artisan::command` method. The `command` method accepts two arguments: the [command signature](#defining-input-expectations) and a Closure which receives the commands arguments and options:  

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    });

The Closure está vinculado a la instancia de comando subyacente, por lo que tiene acceso completo a todos los métodos de ayuda a los que normalmente podría acceder en una clase de comando completa.
> > The Closure is bound to the underlying command instance, so you have full access to all of the helper methods you would typically be able to access on a full command class.  

#### Dependencias tipo Hinting (sugerencias) : Type-Hinting Dependencies

Además de recibir los argumentos y las opciones de su comando, los cierres de comandos también pueden escribir-indicar las dependencias adicionales que le gustaría resolver del [contenedor de servicios] (/docs/{{version}}/container):
> > In addition to receiving your command's arguments and options, command Closures may also type-hint additional dependencies that you would like resolved out of the [service container](/docs/{{version}}/container):  

    use App\User;
    use App\DripEmailer;

    Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

#### Descripciones de los comandos de cierre : Closure Command Descriptions

Al definir un comando basado en Closure, puede usar el método `describe` para agregar una descripción al comando. Esta descripción se mostrará cuando ejecute los comandos `php artisan list` o `php artisan help`:
> > When defining a Closure based command, you may use the `describe` method to add a description to the command. This description will be displayed when you run the `php artisan list` or `php artisan help` commands:  

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');

<a name="defining-input-expectations"></a>
## Definiendo las expectativas de entrada : Defining Input Expectations

Al escribir comandos de consola, es común recopilar información del usuario a través de argumentos u opciones. Laravel hace que sea muy conveniente definir la entrada que espera del usuario que utiliza la propiedad `signature` en sus comandos. La propiedad `signature` le permite definir el nombre, los argumentos y las opciones para el comando en una única sintaxis expresiva, similar a una ruta.
> > When writing console commands, it is common to gather input from the user through arguments or options. Laravel makes it very convenient to define the input you expect from the user using the `signature` property on your commands. The `signature` property allows you to define the name, arguments, and options for the command in a single, expressive, route-like syntax.  

<a name="arguments"></a>
### Argumentos : Arguments

Todos los argumentos y opciones suministrados por el usuario están envueltos en llaves. En el siguiente ejemplo, el comando define un argumento **required**: `user`:
> > All user supplied arguments and options are wrapped in curly braces. In the following example, the command defines one **required** argument: `user`:  

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

También puede hacer que los argumentos sean opcionales y definir valores predeterminados para los argumentos:
> > You may also make arguments optional and define default values for arguments:  

    // Optional argument...
    email:send {user?}

    // Optional argument with default value...
    email:send {user=foo}

<a name="options"></a>
### Opciones : Options

Las opciones, como los argumentos, son otra forma de entrada del usuario. Las opciones son prefijadas por dos guiones (`--`) cuando se especifican en la línea de comando. Hay dos tipos de opciones: las que reciben un valor y las que no. Las opciones que no reciben un valor sirven como un "interruptor" booleano. Echemos un vistazo a un ejemplo de este tipo de opción:
> > Options, like arguments, are another form of user input. Options are prefixed by two hyphens (`--`) when they are specified on the command line. There are two types of options: those that receive a value and those that don't. Options that don't receive a value serve as a boolean "switch". Let's take a look at an example of this type of option:  

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

En este ejemplo, el interruptor `--queue` se puede especificar al llamar al comando Artisan. Si se pasa el interruptor `--queue`, el valor de la opción será `true`. De lo contrario, el valor será `false`:
> > In this example, the `--queue` switch may be specified when calling the Artisan command. If the `--queue` switch is passed, the value of the option will be `true`. Otherwise, the value will be `false`:  

    php artisan email:send 1 --queue

<a name="options-with-values"></a>
#### Opciones con valores : Options With Values

A continuación, echemos un vistazo a una opción que espera un valor. Si el usuario debe especificar un valor para una opción, sufija el nombre de la opción con un signo `=`:
> > Next, let's take a look at an option that expects a value. If the user must specify a value for an option, suffix the option name with a `=` sign:  

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

En este ejemplo, el usuario puede pasar un valor para la opción como ese:
> > In this example, the user may pass a value for the option like so:  

    php artisan email:send 1 --queue=default

Puede asignar valores predeterminados a las opciones especificando el valor predeterminado después del nombre de la opción. Si el usuario no pasa ningún valor de opción, se usará el valor predeterminado:
> > You may assign default values to options by specifying the default value after the option name. If no option value is passed by the user, the default value will be used:  

    email:send {user} {--queue=default}

<a name="option-shortcuts"></a>
#### Accesos directos de opciones : Option Shortcuts

Para asignar un acceso directo al definir una opción, puede especificarla antes del nombre de la opción y usar a | delimitador para separar el atajo del nombre completo de la opción:
> > To assign a shortcut when defining an option, you may specify it before the option name and use a | delimiter to separate the shortcut from the full option name:  

    email:send {user} {--Q|queue}

<a name="input-arrays"></a>
### Arrays de entrada : Input Arrays

Si desea definir argumentos u opciones para esperar entradas en array, puede usar el caracter `*`. Primero, echemos un vistazo a un ejemplo que especifica un argumento en array:
> > If you would like to define arguments or options to expect array inputs, you may use the `*` character. First, let's take a look at an example that specifies an array argument:  

    email:send {user*}

Al llamar a este método, los argumentos `user` pueden pasarse en orden a la línea de comando. Por ejemplo, el siguiente comando establecerá el valor de `user` en `['foo', 'bar']`:
> > When calling this method, the `user` arguments may be passed in order to the command line. For example, the following command will set the value of `user` to `['foo', 'bar']`:  

    php artisan email:send foo bar

Al definir una opción que espera una entrada en array, cada valor de opción pasado al comando debe ir precedido del nombre de la opción:
> > When defining an option that expects an array input, each option value passed to the command should be prefixed with the option name:

    email:send {user} {--id=*}

    php artisan email:send --id=1 --id=2

<a name="input-descriptions"></a>
### Descripciones de entrada : Input Descriptions

Puede asignar descripciones para ingresar argumentos y opciones separando el parámetro de la descripción usando dos puntos. Si necesita un poco más de espacio para definir su comando, siéntase libre de difundir la definición en múltiples líneas:
> > You may assign descriptions to input arguments and options by separating the parameter from the description using a colon. If you need a little extra room to define your command, feel free to spread the definition across multiple lines:  

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';

<a name="command-io"></a>
## Comando I/O

<a name="retrieving-input"></a>
### Recuperación de entrada : Retrieving Input

Mientras se ejecuta su comando, obviamente necesitará acceder a los valores de los argumentos y opciones aceptados por su comando. Para hacerlo, puedes usar los métodos `argument` y` option`:
> > While your command is executing, you will obviously need to access the values for the arguments and options accepted by your command. To do so, you may use the `argument` and `option` methods:  

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

Si necesita recuperar todos los argumentos como un `array`, llame al método` arguments`:
> > If you need to retrieve all of the arguments as an `array`, call the `arguments` method:  

    $arguments = $this->arguments();

Las opciones se pueden recuperar tan fácilmente como los argumentos que usan el método `option`. Para recuperar todas las opciones como una matriz, llame al método `options`:
> > Options may be retrieved just as easily as arguments using the `option` method. To retrieve all of the options as an array, call the `options` method:  

    // Retrieve a specific option...
    $queueName = $this->option('queue');

    // Retrieve all options...
    $options = $this->options();

Si el argumento u opción no existe, se devolverá `null`.
> > If the argument or option does not exist, `null` will be returned.  

<a name="prompting-for-input"></a>
### Solicitud de entrada : Prompting For Input

Además de mostrar la salida, también puede pedir al usuario que proporcione información durante la ejecución de su comando. El método `ask` indicará al usuario la pregunta dada, aceptará su entrada y luego devolverá la entrada del usuario a su comando:
> > In addition to displaying output, you may also ask the user to provide input during the execution of your command. The `ask` method will prompt the user with the given question, accept their input, and then return the user's input back to your command:  

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

El método `secret` es similar a `ask`, pero la entrada del usuario no será visible para ellos mientras escriben en la consola. Este método es útil cuando se solicita información confidencial, como una contraseña:
> > The `secret` method is similar to `ask`, but the user's input will not be visible to them as they type in the console. This method is useful when asking for sensitive information such as a password:  

    $password = $this->secret('What is the password?');

#### Solicitud de confirmación : Asking For Confirmation

Si necesita solicitar al usuario una confirmación simple, puede usar el método `confirm`. Por defecto, este método devolverá `false`. Sin embargo, si el usuario ingresa `y` o` yes` en respuesta al prompt, el método devolverá `true`.
> > If you need to ask the user for a simple confirmation, you may use the `confirm` method. By default, this method will return `false`. However, if the user enters `y` or `yes` in response to the prompt, the method will return `true`.  

    if ($this->confirm('Do you wish to continue?')) {
        //
    }

#### Autocompletar : Auto-Completion

El método `anticipate` se puede usar para proporcionar autocompletado para posibles elecciones. El usuario aún puede elegir cualquier respuesta, independientemente de las sugerencias de autocompletado:
> > The `anticipate` method can be used to provide auto-completion for possible choices. The user can still choose any answer, regardless of the auto-completion hints:  

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

#### Preguntas de respuestas múltiples : Multiple Choice Questions

Si necesita darle al usuario un conjunto predefinido de opciones, puede usar el método `choice`. Puede establecer el índice del array del valor predeterminado que se devolverá si no se elige ninguna opción:
> > If you need to give the user a predefined set of choices, you may use the `choice` method. You may set the array index of the default value to be returned if no option is chosen:  

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);

<a name="writing-output"></a>
### Escritura de salida : Writing Output

Para enviar resultados a la consola, use los métodos `line`,` info`, `comment`, `question` y `error`. Cada uno de estos métodos utilizará los colores ANSI apropiados para su propósito. Por ejemplo, vamos a mostrar cierta información general para el usuario. Normalmente, el método `info` se mostrará en la consola como texto verde:
> > To send output to the console, use the `line`, `info`, `comment`, `question` and `error` methods. Each of these methods will use appropriate ANSI colors for their purpose. For example, let's display some general information to the user. Typically, the `info` method will display in the console as green text:  

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

Para mostrar un mensaje de error, use el método `error`. El texto del mensaje de error normalmente se muestra en rojo:
> > To display an error message, use the `error` method. Error message text is typically displayed in red:  

    $this->error('Something went wrong!');

Si desea mostrar la salida de consola sin color, utilice el método `line`:
> > If you would like to display plain, uncolored console output, use the `line` method:  

    $this->line('Display this on the screen');

#### Diseños de tabla : Table Layouts

El método `table` hace que sea fácil formatear correctamente varias filas / columnas de datos. Simplemente pase los encabezados y filas al método. El ancho y la altura se calcularán dinámicamente en función de los datos dados:
> > The `table` method makes it easy to correctly format multiple rows / columns of data. Just pass in the headers and rows to the method. The width and height will be dynamically calculated based on the given data:  

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### Barras de progreso : Progress Bars

Para tareas de larga ejecución, podría ser útil mostrar un indicador de progreso. Usando el objeto de salida, podemos iniciar, avanzar y detener la barra de progreso. Primero, defina el número total de pasos por los que el proceso se repetirá. A continuación, avance la barra de progreso después de procesar cada elemento:
> > For long running tasks, it could be helpful to show a progress indicator. Using the output object, we can start, advance and stop the Progress Bar. First, define the total number of steps the process will iterate through. Then, advance the Progress Bar after processing each item:  

    $users = App\User::all();

    $bar = $this->output->createProgressBar(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

Para obtener más opciones avanzadas, consulte la [Documentación del componente de la barra de progreso de Symfony] (https://symfony.com/doc/current/components/console/helpers/progressbar.html).
> > For more advanced options, check out the [Symfony Progress Bar component documentation](https://symfony.com/doc/current/components/console/helpers/progressbar.html).  

<a name="registering-commands"></a>
## Registro de comandos : Registering Commands

Debido a la llamada al método `load` en el método `commands` del kernel de su consola, todos los comandos dentro del directorio `app/Console/Commands` se registrarán automáticamente con Artisan. De hecho, puede realizar llamadas adicionales al método `load` para escanear otros directorios en busca de comandos de Artisan:
> > Because of the `load` method call in your console kernel's `commands` method, all commands within the `app/Console/Commands` directory will automatically be registered with Artisan. In fact, you are free to make additional calls to the `load` method to scan other directories for Artisan commands:  

    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
        $this->load(__DIR__.'/MoreCommands');

        // ...
    }

También puede registrar comandos manualmente al agregar su nombre de clase a la propiedad `$commands` de su archivo `app/Console/Kernel.php`. Cuando Artisan arranca, todos los comandos enumerados en esta propiedad serán resueltos por [contenedor de servicio] (/docs/{{version}}/container) y registrados con Artisan:
> > You may also manually register commands by adding its class name to the `$commands` property of your `app/Console/Kernel.php` file. When Artisan boots, all the commands listed in this property will be resolved by the [service container](/docs/{{version}}/container) and registered with Artisan:  

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>
## Ejecución programática de comandos : Programmatically Executing Commands

En ocasiones, es posible que desee ejecutar un comando de Artisan fuera de la CLI. Por ejemplo, puede desear disparar un comando de Artisan de una ruta o controlador. Puede usar el método `call` en la fachada `Artisan` para lograr esto. El método `call` acepta el nombre o la clase del comando como primer argumento y una matriz de parámetros de comando como el segundo argumento. El código de salida será devuelto:
> > Sometimes you may wish to execute an Artisan command outside of the CLI. For example, you may wish to fire an Artisan command from a route or controller. You may use the `call` method on the `Artisan` facade to accomplish this. The `call` method accepts either the command's name or class as the first argument, and an array of command parameters as the second argument. The exit code will be returned:  

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

Usando el método `queue` en la fachada `Artisan`, incluso puedes poner en cola los comandos de Artisan para que sean procesados ​​en segundo plano por tus [trabajadores de cola] (/docs/{{version}}/queues). Antes de usar este método, asegúrese de haber configurado su cola y ejecutar un detector de cola:
> > Using the `queue` method on the `Artisan` facade, you may even queue Artisan commands so they are processed in the background by your [queue workers](/docs/{{version}}/queues). Before using this method, make sure you have configured your queue and are running a queue listener:  

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

También puede especificar la conexión o cola a la que debe enviarse el comando de Artisan:
> > You may also specify the connection or queue the Artisan command should be dispatched to:  

    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');

#### Pasando valores en array : Passing Array Values

Si su comando define una opción que acepta un array, puede pasar un array de valores a esa opción:
> > If your command defines an option that accepts an array, you may pass an array of values to that option:  

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--id' => [5, 13]
        ]);
    });

#### Pasando valores booleanos : Passing Boolean Values

Si necesita especificar el valor de una opción que no acepta valores de cadena, como el indicador `--force` en el comando `migrate:refresh`, debe pasar `true` o` false`:
> > If you need to specify the value of an option that does not accept string values, such as the `--force` flag on the `migrate:refresh` command, you should pass `true` or `false`:

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="calling-commands-from-other-commands"></a>
### Comandos de llamada de otros comandos : Calling Commands From Other Commands

En ocasiones, es posible que desee llamar a otros comandos desde un comando Artisan existente. Puede hacerlo usando el método `call`. Este método `call` acepta el nombre del comando y un array de parámetros de comando:
> > Sometimes you may wish to call other commands from an existing Artisan command. You may do so using the `call` method. This `call` method accepts the command name and an array of command parameters:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    }

Si desea llamar a otro comando de consola y suprimir toda su salida, puede usar el método `callSilent`. El método `callSilent` tiene la misma firma que el método `call`:
> > If you would like to call another console command and suppress all of its output, you may use the `callSilent` method. The `callSilent` method has the same signature as the `call` method:

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
