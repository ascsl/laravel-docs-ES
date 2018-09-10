# Programación de tareas : Task Scheduling

- [Introduction](#introduction)
- [Defining Schedules](#defining-schedules)
    - [Scheduling Artisan Commands](#scheduling-artisan-commands)
    - [Scheduling Queued Jobs](#scheduling-queued-jobs)
    - [Scheduling Shell Commands](#scheduling-shell-commands)
    - [Schedule Frequency Options](#schedule-frequency-options)
    - [Timezones](#timezones)
    - [Preventing Task Overlaps](#preventing-task-overlaps)
    - [Running Tasks On One Server](#running-tasks-on-one-server)
    - [Maintenance Mode](#maintenance-mode)
- [Task Output](#task-output)
- [Task Hooks](#task-hooks)

<a name="introduction"></a>
## Introducción : Introduction

En el pasado, puede haber generado una entrada cron para cada tarea que necesita programar en su servidor. Sin embargo, esto puede convertirse rápidamente en un problema, ya que su agenda de tareas ya no está en control de fuente y debe SSH en su servidor para agregar entradas Cron adicionales.
> > In the past, you may have generated a Cron entry for each task you needed to schedule on your server. However, this can quickly become a pain, because your task schedule is no longer in source control and you must SSH into your server to add additional Cron entries.

El programador de comandos de Laravel le permite definir de manera fluida y expresiva su cronograma de comandos dentro de Laravel. Cuando se utiliza el planificador, solo se necesita una sola entrada cron en su servidor. Su programa de tareas se define en el método `schedule` del archivo `app/Console/Kernel.php`. Para ayudarlo a comenzar, se define un ejemplo simple dentro del método.
> > Laravel's command scheduler allows you to fluently and expressively define your command schedule within Laravel itself. When using the scheduler, only a single Cron entry is needed on your server. Your task schedule is defined in the `app/Console/Kernel.php` file's `schedule` method. To help you get started, a simple example is defined within the method.

### Iniciar el programador : Starting The Scheduler

Al utilizar el programador, solo necesita agregar la siguiente entrada de Cron a su servidor. Si no sabe cómo agregar entradas de Cron a su servidor, considere usar un servicio como [Laravel Forge](https://forge.laravel.com) que puede administrar las entradas de Cron por usted:
> > When using the scheduler, you only need to add the following Cron entry to your server. If you do not know how to add Cron entries to your server, consider using a service such as [Laravel Forge](https://forge.laravel.com) which can manage the Cron entries for you:

    * * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1

Este Cron llamará al programador de comandos de Laravel cada minuto. Cuando se ejecuta el comando `schedule:run`, Laravel evaluará sus tareas programadas y ejecutará las tareas vencidas.
> > This Cron will call the Laravel command scheduler every minute. When the `schedule:run` command is executed, Laravel will evaluate your scheduled tasks and runs the tasks that are due.

<a name="defining-schedules"></a>
## Definición de horarios : Defining Schedules

Puede definir todas sus tareas programadas en el método `schedule` de la clase `App\Console\Kernel`. Para comenzar, veamos un ejemplo de programar una tarea. En este ejemplo, programaremos un `Closure` para llamar todos los días a la medianoche. Dentro del `Closure` vamos a ejecutar una consulta de base de datos para borrar una tabla:
> > You may define all of your scheduled tasks in the `schedule` method of the `App\Console\Kernel` class. To get started, let's look at an example of scheduling a task. In this example, we will schedule a `Closure` to be called every day at midnight. Within the `Closure` we will execute a database query to clear a table:

    <?php

    namespace App\Console;

    use DB;
    use Illuminate\Console\Scheduling\Schedule;
    use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

    class Kernel extends ConsoleKernel
    {
        /**
         * The Artisan commands provided by your application.
         *
         * @var array
         */
        protected $commands = [
            //
        ];

        /**
         * Define the application's command schedule.
         *
         * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
         * @return void
         */
        protected function schedule(Schedule $schedule)
        {
            $schedule->call(function () {
                DB::table('recent_users')->delete();
            })->daily();
        }
    }

Además de programar usando Closures, también puedes usar [objetos invocables](http://php.net/manual/en/language.oop5.magic.php#object.invoke). Los objetos invocables son clases PHP simples que contienen un método `__invoke`:
> > In addition to scheduling using Closures, you may also using [invokable objects](http://php.net/manual/en/language.oop5.magic.php#object.invoke). Invokable objects are simple PHP classes that contain an `__invoke` method:

    $schedule->call(new DeleteRecentUsers)->daily();

<a name="scheduling-artisan-commands"></a>
### Programación de comandos Artisan : Scheduling Artisan Commands

Además de programar llamadas de Closure, también puede programar [comandos de Artisan](/docs/{{version}}/artisan) y comandos del sistema operativo. Por ejemplo, puede usar el método `command` para programar un comando de Artisan utilizando el nombre o la clase del comando:
> > In addition to scheduling Closure calls, you may also schedule [Artisan commands](/docs/{{version}}/artisan) and operating system commands. For example, you may use the `command` method to schedule an Artisan command using either the command's name or class:

    $schedule->command('emails:send --force')->daily();

    $schedule->command(EmailsCommand::class, ['--force'])->daily();

<a name="scheduling-queued-jobs"></a>
### Programación de trabajos en cola : Scheduling Queued Jobs

El método `job` se puede usar para programar un [trabajo en cola](/docs/{{version}}/queues). Este método proporciona una forma conveniente de programar trabajos sin utilizar el método `call` para crear Closures manualmente para poner en cola el trabajo:
> > The `job` method may be used to schedule a [queued job](/docs/{{version}}/queues). This method provides a convenient way to schedule jobs without using the `call` method to manually create Closures to queue the job:

    $schedule->job(new Heartbeat)->everyFiveMinutes();

    // Dispatch the job to the "heartbeats" queue...
    $schedule->job(new Heartbeat, 'heartbeats')->everyFiveMinutes();

<a name="scheduling-shell-commands"></a>
### Programación de comandos de shell : Scheduling Shell Commands

El método `exec` se puede usar para emitir un comando al sistema operativo:
> > The `exec` method may be used to issue a command to the operating system:

    $schedule->exec('node /home/forge/script.js')->daily();

<a name="schedule-frequency-options"></a>
### Opciones de frecuencia de programación : Schedule Frequency Options

Por supuesto, hay una variedad de horarios que puede asignar a su tarea:
> > Of course, there are a variety of schedules you may assign to your task:

Method  | Descripción | Description
------------- | ------------- | -------------
`->cron('* * * * *');`  |  Ejecute la tarea en un cronograma personalizado de Cron  |  Run the task on a custom Cron schedule
`->everyMinute();`  |  Ejecute la tarea cada minuto  |  Run the task every minute
`->everyFiveMinutes();`  |  Ejecute la tarea cada cinco minutos  |  Run the task every five minutes
`->everyTenMinutes();`  |  Ejecute la tarea cada diez minutos  |  Run the task every ten minutes
`->everyFifteenMinutes();`  |  Ejecute la tarea cada quince minutos  |  Run the task every fifteen minutes
`->everyThirtyMinutes();`  |  Ejecute la tarea cada treinta minutos  |  Run the task every thirty minutes
`->hourly();`  |  Ejecute la tarea cada hora  |  Run the task every hour
`->hourlyAt(17);`  |  Ejecute la tarea cada hora a las 17 minutos después de la hora  |  Run the task every hour at 17 mins past the hour
`->daily();`  |  Ejecute la tarea todos los días a la medianoche  |  Run the task every day at midnight
`->dailyAt('13:00');`  |  Ejecute la tarea todos los días a las 13:00  |  Run the task every day at 13:00
`->twiceDaily(1, 13);`  |  Ejecute la tarea a diario a la 1:00 y 13:00  |  Run the task daily at 1:00 & 13:00
`->weekly();`  |  Ejecute la tarea cada semana  |  Run the task every week
`->weeklyOn(1, '8:00');`  |  Ejecute la tarea todas las semanas el lunes a las 8:00  |  Run the task every week on Monday at 8:00
`->monthly();`  |  Ejecute la tarea cada mes  |  Run the task every month
`->monthlyOn(4, '15:00');`  |  Ejecute la tarea todos los meses el día 4 a las 15:00  |  Run the task every month on the 4th at 15:00
`->quarterly();` |  Ejecute la tarea cada trimestre  |  Run the task every quarter
`->yearly();`  |  Ejecute la tarea cada año  |  Run the task every year
`->timezone('America/New_York');` |  Establecer la zona horaria  |  Set the timezone

Estos métodos se pueden combinar con restricciones adicionales para crear horarios aún más ajustados que solo se ejecutan en ciertos días de la semana. Por ejemplo, para programar un comando para que se ejecute semanalmente el lunes:
> > These methods may be combined with additional constraints to create even more finely tuned schedules that only run on certain days of the week. For example, to schedule a command to run weekly on Monday:

    // Run once per week on Monday at 1 PM...
    $schedule->call(function () {
        //
    })->weekly()->mondays()->at('13:00');

    // Run hourly from 8 AM to 5 PM on weekdays...
    $schedule->command('foo')
              ->weekdays()
              ->hourly()
              ->timezone('America/Chicago')
              ->between('8:00', '17:00');

A continuación hay una lista de las restricciones de cronograma adicionales:
> > Below is a list of the additional schedule constraints:

Method  | Descripción | Description
------------- | ------------- | -------------
`->weekdays();`  |  Limite la tarea a los días de la semana  |  Limit the task to weekdays
`->sundays();`  |  Limite la tarea a los días al Domingo  |  Limit the task to Sunday
`->mondays();`  |  Limite la tarea a los días al Lunes  |  Limit the task to Monday
`->tuesdays();`  |  Limite la tarea a los días al Martes  |  Limit the task to Tuesday
`->wednesdays();`  |  Limite la tarea a los días al Miercoles  |  Limit the task to Wednesday
`->thursdays();`  |  Limite la tarea a los días al Jueves  |  Limit the task to Thursday
`->fridays();`  |  Limite la tarea a los días al Viernes  |  Limit the task to Friday
`->saturdays();`  |  Limite la tarea a los días al Sabado  |  Limit the task to Saturday
`->between($start, $end);`  |  Limite la tarea para ejecutarse entre los tiempos de inicio y fin  |  Limit the task to run between start and end times
`->when(Closure);`  |  Limite la tarea en base a una prueba de verdad  |  Limit the task based on a truth test

#### Entre restricciones de tiempo : Between Time Constraints

El método `between` se puede usar para limitar la ejecución de una tarea en función de la hora del día:
> > The `between` method may be used to limit the execution of a task based on the time of day:

    $schedule->command('reminders:send')
                        ->hourly()
                        ->between('7:00', '22:00');

Del mismo modo, el método `unlessBetween` se puede usar para excluir la ejecución de una tarea por un período de tiempo:
> > Similarly, the `unlessBetween` method can be used to exclude the execution of a task for a period of time:

    $schedule->command('reminders:send')
                        ->hourly()
                        ->unlessBetween('23:00', '4:00');

#### Restricciones de prueba de verdad : Truth Test Constraints

El método `when` se puede usar para limitar la ejecución de una tarea en función del resultado de una prueba de verdad dada. En otras palabras, si el `Closure` dado devuelve `true`, la tarea se ejecutará siempre que ninguna otra condición restrictiva impida la ejecución de la tarea:
> > The `when` method may be used to limit the execution of a task based on the result of a given truth test. In other words, if the given `Closure` returns `true`, the task will execute as long as no other constraining conditions prevent the task from running:

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });

El método `skip` puede verse como el inverso de `when`. Si el método `skip` devuelve `true`, la tarea programada no se ejecutará:
> > The `skip` method may be seen as the inverse of `when`. If the `skip` method returns `true`, the scheduled task will not be executed:

    $schedule->command('emails:send')->daily()->skip(function () {
        return true;
    });

Al usar métodos encadenados `when`, el comando programado solo se ejecutará si todas las condiciones` when` devuelven `true`.
> > When using chained `when` methods, the scheduled command will only execute if all `when` conditions return `true`.

<a name="timezones"></a>
### Zonas horarias : Timezones

Usando el método `timezone`, puede especificar que el tiempo de una tarea programada se debe interpretar dentro de una zona horaria dada:
> > Using the `timezone` method, you may specify that a scheduled task's time should be interpreted within a given timezone:

    $schedule->command('report:generate')
             ->timezone('America/New_York')
             ->at('02:00')

> {note} Recuerde que algunas zonas horarias utilizan el horario de verano. Cuando se producen cambios en el horario de verano, la tarea programada puede ejecutarse dos veces o incluso no ejecutarse. Por esta razón, recomendamos evitar la programación de la zona horaria cuando sea posible.
> > > {note} Remember that some timezones utilize daylight savings time. When daylight saving time changes occur, your scheduled task may run twice or even not run at all. For this reason, we recommend avoiding timezone scheduling when possible.

<a name="preventing-task-overlaps"></a>
### Prevención de superposiciones de tareas : Preventing Task Overlaps

Por defecto, las tareas programadas se ejecutarán incluso si la instancia anterior de la tarea aún se está ejecutando. Para evitar esto, puede usar el método `withoutOverlapping`:
> > By default, scheduled tasks will be run even if the previous instance of the task is still running. To prevent this, you may use the `withoutOverlapping` method:

    $schedule->command('emails:send')->withoutOverlapping();

En este ejemplo, el `emails:send` [comando Artisan](/docs/{{version}}/artisan) se ejecutará cada minuto si aún no se está ejecutando. El método `withoutOverlapping` es especialmente útil si tiene tareas que varían drásticamente en el tiempo de ejecución, lo que le impide predecir exactamente cuánto tiempo llevará una tarea determinada.
> > In this example, the `emails:send` [Artisan command](/docs/{{version}}/artisan) will be run every minute if it is not already running. The `withoutOverlapping` method is especially useful if you have tasks that vary drastically in their execution time, preventing you from predicting exactly how long a given task will take.

Si es necesario, puede especificar cuántos minutos deben transcurrir antes de que caduque el bloqueo "sin superposición". Por defecto, el bloqueo caducará después de 24 horas:
> > If needed, you may specify how many minutes must pass before the "without overlapping" lock expires. By default, the lock will expire after 24 hours:

    $schedule->command('emails:send')->withoutOverlapping(10);

<a name="running-tasks-on-one-server"></a>
### Ejecución de tareas en un servidor : Running Tasks On One Server

> {note} Para utilizar esta característica, su aplicación debe usar el controlador de caché `memcached` o `redis` como el controlador de caché predeterminado de su aplicación. Además, todos los servidores deben comunicarse con el mismo servidor de caché central.
> > > {note} To utilize this feature, your application must be using the `memcached` or `redis` cache driver as your application's default cache driver. In addition, all servers must be communicating with the same central cache server.

Si su aplicación se ejecuta en varios servidores, puede limitar un trabajo programado para que se ejecute solo en un único servidor. Por ejemplo, supongamos que tiene una tarea programada que genera un nuevo informe todos los viernes por la noche. Si el planificador de tareas se ejecuta en tres servidores de trabajo, la tarea programada se ejecutará en los tres servidores y generará el informe tres veces. ¡No está bien!
> > If your application is running on multiple servers, you may limit a scheduled job to only execute on a single server. For instance, assume you have a scheduled task that generates a new report every Friday night. If the task scheduler is running on three worker servers, the scheduled task will run on all three servers and generate the report three times. Not good!

Para indicar que la tarea debe ejecutarse en un solo servidor, use el método `onOneServer` al definir la tarea programada. El primer servidor que obtenga la tarea asegurará un bloqueo atómico en el trabajo para evitar que otros servidores ejecuten la misma tarea al mismo tiempo:
> > To indicate that the task should run on only one server, use the `onOneServer` method when defining the scheduled task. The first server to obtain the task will secure an atomic lock on the job to prevent other servers from running the same task at the same time:

    $schedule->command('report:generate')
                    ->fridays()
                    ->at('17:00')
                    ->onOneServer();

<a name="maintenance-mode"></a>
### Modo de mantenimiento : Maintenance Mode

Las tareas programadas de Laravel no se ejecutarán cuando Laravel esté en [modo de mantenimiento](/docs/{{version}}/configuration#maintenance-mode), ya que no queremos que sus tareas interfieran con el mantenimiento pendiente que pueda estar realizando en tu servidor Sin embargo, si desea obligar a una tarea a ejecutarse incluso en modo de mantenimiento, puede utilizar el método `evenInMaintenanceMode`:
> > Laravel's scheduled tasks will not run when Laravel is in [maintenance mode](/docs/{{version}}/configuration#maintenance-mode), since we don't want your tasks to interfere with any unfinished maintenance you may be performing on your server. However, if you would like to force a task to run even in maintenance mode, you may use the `evenInMaintenanceMode` method:

    $schedule->command('emails:send')->evenInMaintenanceMode();

<a name="task-output"></a>
## Tarea de salida : Task Output

El programador de Laravel proporciona varios métodos convenientes para trabajar con la salida generada por las tareas programadas. Primero, utilizando el método `sendOutputTo`, puede enviar la salida a un archivo para su posterior inspección:
> > The Laravel scheduler provides several convenient methods for working with the output generated by scheduled tasks. First, using the `sendOutputTo` method, you may send the output to a file for later inspection:

    $schedule->command('emails:send')
             ->daily()
             ->sendOutputTo($filePath);

Si desea agregar el resultado a un archivo dado, puede usar el método `appendOutputTo`:
> > If you would like to append the output to a given file, you may use the `appendOutputTo` method:

    $schedule->command('emails:send')
             ->daily()
             ->appendOutputTo($filePath);

Usando el método `emailOutputTo`, puede enviar el resultado por correo electrónico a una dirección de email de su elección. Antes de enviar por correo electrónico el resultado de una tarea, debe configurar los [servicios de correo electrónico](/docs/{{version}}/mail) de Laravel:
> > Using the `emailOutputTo` method, you may e-mail the output to an e-mail address of your choice. Before e-mailing the output of a task, you should configure Laravel's [e-mail services](/docs/{{version}}/mail):

    $schedule->command('foo')
             ->daily()
             ->sendOutputTo($filePath)
             ->emailOutputTo('foo@example.com');

> {note} Los métodos `emailOutputTo`, `sendOutputTo` y `appendOutputTo` son exclusivos de los métodos `command` y `exec`.
> > > {note} The `emailOutputTo`, `sendOutputTo` and `appendOutputTo` methods are exclusive to the `command` and `exec` methods.

<a name="task-hooks"></a>
## Tareas Ganchos : Task Hooks

Usando los métodos `before` y `after`, puede especificar que el código se ejecute antes y después de que se complete la tarea programada:
> > Using the `before` and `after` methods, you may specify code to be executed before and after the scheduled task is complete:

    $schedule->command('emails:send')
             ->daily()
             ->before(function () {
                 // Task is about to start...
             })
             ->after(function () {
                 // Task is complete...
             });

#### Pinging URLs

Usando los métodos `pingBefore` y `thenPing`, el planificador puede hacer ping automáticamente a una URL determinada antes o después de que se complete una tarea. Este método es útil para notificar a un servicio externo, como [Laravel Envoyer](https://envoyer.io), que su tarea programada está comenzando o ha finalizado su ejecución:
> > Using the `pingBefore` and `thenPing` methods, the scheduler can automatically ping a given URL before or after a task is complete. This method is useful for notifying an external service, such as [Laravel Envoyer](https://envoyer.io), that your scheduled task is commencing or has finished execution:

    $schedule->command('emails:send')
             ->daily()
             ->pingBefore($url)
             ->thenPing($url);

El uso de la función `pingBefore($url)` o `thenPing($url)` requiere la biblioteca HTTP de Guzzle. Puede agregar Guzzle a su proyecto utilizando el administrador de paquetes Composer:
> > Using either the `pingBefore($url)` or `thenPing($url)` feature requires the Guzzle HTTP library. You can add Guzzle to your project using the Composer package manager:

    composer require guzzlehttp/guzzle
