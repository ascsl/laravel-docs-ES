# Envoy Task Runner

- [Introduction](#introduction)
    - [Installation](#installation)
- [Writing Tasks](#writing-tasks)
    - [Setup](#setup)
    - [Variables](#variables)
    - [Stories](#stories)
    - [Multiple Servers](#multiple-servers)
- [Running Tasks](#running-tasks)
    - [Confirming Task Execution](#confirming-task-execution)
- [Notifications](#notifications)
    - [Slack](#slack)

<a name="introduction"></a>
## Introducción : Introduction

[Laravel Envoy](https://github.com/laravel/envoy) proporciona una sintaxis mínima y clara para definir las tareas comunes que ejecuta en sus servidores remotos. Usando la sintaxis de estilo Blade, puede configurar fácilmente tareas para implementación, comandos de Artisan y más. Actualmente, Envoy solo es compatible con los sistemas operativos Mac y Linux.
> > [Laravel Envoy](https://github.com/laravel/envoy) provides a clean, minimal syntax for defining common tasks you run on your remote servers. Using Blade style syntax, you can easily setup tasks for deployment, Artisan commands, and more. Currently, Envoy only supports the Mac and Linux operating systems.

<a name="installation"></a>
### Instalación : Installation

Primero, instale Envoy usando el comando Composer `global require`:
> > First, install Envoy using the Composer `global require` command:

    composer global require laravel/envoy

Dado que las librerías globales de Composer a veces pueden causar conflictos en la versión del paquete, puede considerar usar `cgr`, que es un reemplazo directo para el comando `composer global require`. Las instrucciones de instalación de la librería `cgr` se pueden encontrar [en GitHub](https://github.com/consolidation-org/cgr).
> > Since global Composer libraries can sometimes cause package version conflicts, you may wish to consider using `cgr`, which is a drop-in replacement for the `composer global require` command. The `cgr` library's installation instructions can be [found on GitHub](https://github.com/consolidation-org/cgr).

> {note} Asegúrese de colocar el directorio `~/.composer/vendor/bin` en su PATH para que el ejecutable `envoy` se encuentre al ejecutar el comando `envoy` en su terminal.
> > > {note} Make sure to place the `~/.composer/vendor/bin` directory in your PATH so the `envoy` executable is found when running the `envoy` command in your terminal.

#### Actualización de Envoy : Updating Envoy

También puede usar Composer para mantener su instalación de Envoy actualizada. Al emitir el comando `composer global update` se actualizarán todos los paquetes de Composer instalados globalmente:
> > You may also use Composer to keep your Envoy installation up to date. Issuing the `composer global update` command will update all of your globally installed Composer packages:

    composer global update

<a name="writing-tasks"></a>
## Tareas de escritura : Writing Tasks

Todas las tareas de Envoy se deben definir en un archivo `Envoy.blade.php` en la raíz de su proyecto. Aquí hay un ejemplo para comenzar:
> > All of your Envoy tasks should be defined in an `Envoy.blade.php` file in the root of your project. Here's an example to get you started:

    @servers(['web' => ['user@192.168.1.1']])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask

Como puede ver, una serie de `@servers` se define en la parte superior del archivo, lo que le permite hacer referencia a estos servidores en la opción `on` de sus declaraciones de tareas. Dentro de sus declaraciones `@task ', debe colocar el código Bash que debería ejecutarse en su servidor cuando se ejecuta la tarea.
> > As you can see, an array of `@servers` is defined at the top of the file, allowing you to reference these servers in the `on` option of your task declarations. Within your `@task` declarations, you should place the Bash code that should run on your server when the task is executed.

Puede forzar que un script se ejecute localmente especificando la dirección IP del servidor como `127.0.0.1`:
> > You can force a script to run locally by specifying the server's IP address as `127.0.0.1`:

    @servers(['localhost' => '127.0.0.1'])

<a name="setup"></a>
### Preparar : Setup

A veces, es posible que deba ejecutar algún código PHP antes de ejecutar sus tareas Envoy. Puede usar la directiva ```@setup``` para declarar variables y realizar otro trabajo PHP general antes de que se ejecute cualquiera de sus otras tareas:
> > Sometimes, you may need to execute some PHP code before executing your Envoy tasks. You may use the ```@setup``` directive to declare variables and do other general PHP work before any of your other tasks are executed:

    @setup
        $now = new DateTime();

        $environment = isset($env) ? $env : "testing";
    @endsetup

Si necesita requerir otros archivos PHP antes de ejecutar su tarea, puede usar la directiva `@include` en la parte superior de su archivo `Envoy.blade.php`:
> > If you need to require other PHP files before your task is executed, you may use the `@include` directive at the top of your `Envoy.blade.php` file:

    @include('vendor/autoload.php')

    @task('foo')
        # ...
    @endtask

<a name="variables"></a>
### Variables : Variables

Si es necesario, puede pasar valores de opciones a las tareas de Envoy usando la línea de comando:
> > If needed, you may pass option values into Envoy tasks using the command line:

    envoy run deploy --branch=master

Puede acceder a las opciones en sus tareas a través de la sintaxis "echo" de Blade. Por supuesto, también puede usar declaraciones y bucles `if` dentro de sus tareas. Por ejemplo, verifiquemos la presencia de la variable `$branch` antes de ejecutar el comando `git pull`:
> > You may access the options in your tasks via Blade's "echo" syntax. Of course, you may also use `if` statements and loops within your tasks. For example, let's verify the presence of the `$branch` variable before executing the `git pull` command:

    @servers(['web' => '192.168.1.1'])

    @task('deploy', ['on' => 'web'])
        cd site

        @if ($branch)
            git pull origin {{ $branch }}
        @endif

        php artisan migrate
    @endtask

<a name="stories"></a>
### Historias : Stories

Las historias agrupan un conjunto de tareas con un nombre único y conveniente, lo que le permite agrupar tareas pequeñas y enfocadas en grandes tareas. Por ejemplo, una historia de `deploy` puede ejecutar las tareas `git` y` composer` al enumerar los nombres de las tareas dentro de su definición:
> > Stories group a set of tasks under a single, convenient name, allowing you to group small, focused tasks into large tasks. For instance, a `deploy` story may run the `git` and `composer` tasks by listing the task names within its definition:

    @servers(['web' => '192.168.1.1'])

    @story('deploy')
        git
        composer
    @endstory

    @task('git')
        git pull origin master
    @endtask

    @task('composer')
        composer install
    @endtask

Una vez que la historia ha sido escrita, puede ejecutarla como una tarea típica:
> > Once the story has been written, you may run it just like a typical task:

    envoy run deploy

<a name="multiple-servers"></a>
### Varios servidores : Multiple Servers

Envoy le permite ejecutar fácilmente una tarea en varios servidores. Primero, agregue servidores adicionales a su declaración `@servers`. A cada servidor se le debe asignar un nombre único. Una vez que haya definido sus servidores adicionales, enumere cada uno de los servidores en el array `on` de la tarea:
> > Envoy allows you to easily run a task across multiple servers. First, add additional servers to your `@servers` declaration. Each server should be assigned a unique name. Once you have defined your additional servers, list each of the servers in the task's `on` array:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2']])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

#### Ejecución paralela : Parallel Execution

Por defecto, las tareas se ejecutarán en cada servidor en serie. En otras palabras, una tarea terminará ejecutándose en el primer servidor antes de proceder a ejecutar en el segundo servidor. Si desea ejecutar una tarea en varios servidores en paralelo, agregue la opción `parallel` a su declaración de tarea:
> > By default, tasks will be executed on each server serially. In other words, a task will finish running on the first server before proceeding to execute on the second server. If you would like to run a task across multiple servers in parallel, add the `parallel` option to your task declaration:

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="running-tasks"></a>
## Ejecución de tareas : Running Tasks

Para ejecutar una tarea o historia que esté definida en su archivo `Envoy.blade.php`, ejecute el comando `run` de Envoy, pasando el nombre de la tarea o historia que le gustaría ejecutar. Envoy ejecutará la tarea y mostrará el resultado de los servidores mientras se ejecuta la tarea:
> > To run a task or story that is defined in your `Envoy.blade.php` file, execute Envoy's `run` command, passing the name of the task or story you would like to execute. Envoy will run the task and display the output from the servers as the task is running:

    envoy run task

<a name="confirming-task-execution"></a>
### Confirmación de la ejecución de tareas : Confirming Task Execution

Si desea que se le solicite confirmación antes de ejecutar una tarea determinada en sus servidores, debe agregar la directiva `confirm` a su declaración de tareas. Esta opción es particularmente útil para operaciones destructivas:
> > If you would like to be prompted for confirmation before running a given task on your servers, you should add the `confirm` directive to your task declaration. This option is particularly useful for destructive operations:

    @task('deploy', ['on' => 'web', 'confirm' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="notifications"></a>
## Notificaciones : Notifications

<a name="slack"></a>
### Slack

Envoy también admite el envío de notificaciones a [Slack](https://slack.com) después de que se ejecuta cada tarea. La directiva `@slack` acepta una URL de gancho Slack y un nombre de canal. Puede recuperar su URL de webhook creando una integración de "WebHooks entrantes" en su panel de control de Slack. Debe pasar todo el URL de webhook en la directiva `@slack`:
> > Envoy also supports sending notifications to [Slack](https://slack.com) after each task is executed. The `@slack` directive accepts a Slack hook URL and a channel name. You may retrieve your webhook URL by creating an "Incoming WebHooks" integration in your Slack control panel. You should pass the entire webhook URL into the `@slack` directive:

    @finished
        @slack('webhook-url', '#bots')
    @endfinished

Puede proporcionar uno de los siguientes como el argumento del canal:
> > You may provide one of the following as the channel argument:

<div class="content-list" markdown="1">
- Para enviar la notificación a un canal: `#channel`
- Para enviar la notificación a un usuario: `@user`
</div>
<div class="content-list" markdown="1">
> > - To send the notification to a channel: `#channel`
> > - To send the notification to a user: `@user`
</div>
