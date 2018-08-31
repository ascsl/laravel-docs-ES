# Eventos : Events

- [Introduction](#introduction)
- [Registering Events & Listeners](#registering-events-and-listeners)
    - [Generating Events & Listeners](#generating-events-and-listeners)
    - [Manually Registering Events](#manually-registering-events)
- [Defining Events](#defining-events)
- [Defining Listeners](#defining-listeners)
- [Queued Event Listeners](#queued-event-listeners)
    - [Manually Accessing The Queue](#manually-accessing-the-queue)
    - [Handling Failed Jobs](#handling-failed-jobs)
- [Dispatching Events](#dispatching-events)
- [Event Subscribers](#event-subscribers)
    - [Writing Event Subscribers](#writing-event-subscribers)
    - [Registering Event Subscribers](#registering-event-subscribers)

<a name="introduction"></a>
## Introducción : Introduction

Los eventos de Laravel proporcionan una implementación simple del observador, lo que le permite suscribirse y escuchar varios eventos que ocurren en su aplicación. Las clases de eventos normalmente se almacenan en el directorio `app/Events`, mientras que sus oyentes se almacenan en `app/Listeners`. No se preocupe si no ve estos directorios en su aplicación, ya que se crearán para usted a medida que genera eventos y oyentes utilizando los comandos de la consola Artisan.
> > Laravel's events provides a simple observer implementation, allowing you to subscribe and listen for various events that occur in your application. Event classes are typically stored in the `app/Events` directory, while their listeners are stored in `app/Listeners`. Don't worry if you don't see these directories in your application, since they will be created for you as you generate events and listeners using Artisan console commands.

Los eventos son una excelente manera de desacoplar diversos aspectos de su aplicación, ya que un único evento puede tener múltiples oyentes que no dependen el uno del otro. Por ejemplo, es posible que desee enviar una notificación de Slack a su usuario cada vez que se envíe un pedido. En lugar de acoplar su código de procesamiento de pedidos a su código de notificación de Slack, puede generar un evento `OrderShipped`, que un oyente puede recibir y transformar en una notificación de Slack.
> > Events serve as a great way to decouple various aspects of your application, since a single event can have multiple listeners that do not depend on each other. For example, you may wish to send a Slack notification to your user each time an order has shipped. Instead of coupling your order processing code to your Slack notification code, you can raise an `OrderShipped` event, which a listener can receive and transform into a Slack notification.

<a name="registering-events-and-listeners"></a>
## Registro de eventos y oyentes : Registering Events & Listeners

El `EventServiceProvider` incluido con su aplicación Laravel proporciona un lugar conveniente para registrar todas las escuchas de eventos de su aplicación. La propiedad `listen` contiene un array de todos los eventos (claves) y sus oyentes (valores). Por supuesto, puede agregar tantos eventos a este array como requiera su aplicación. Por ejemplo, agreguemos un evento `OrderShipped`:
> > The `EventServiceProvider` included with your Laravel application provides a convenient place to register all of your application's event listeners. The `listen` property contains an array of all events (keys) and their listeners (values). Of course, you may add as many events to this array as your application requires. For example, let's add a `OrderShipped` event:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'App\Events\OrderShipped' => [
            'App\Listeners\SendShipmentNotification',
        ],
    ];

<a name="generating-events-and-listeners"></a>
### Generación de eventos y oyentes : Generating Events & Listeners

Por supuesto, crear manualmente los archivos para cada evento y oyente es engorroso. En su lugar, agregue escuchas y eventos a su `EventServiceProvider` y use el comando `event:generate`. Este comando generará cualquier evento u oyente que esté listado en su `EventServiceProvider`. Por supuesto, los eventos y oyentes que ya existen quedarán intactos:
> > Of course, manually creating the files for each event and listener is cumbersome. Instead, add listeners and events to your `EventServiceProvider` and use the `event:generate` command. This command will generate any events or listeners that are listed in your `EventServiceProvider`. Of course, events and listeners that already exist will be left untouched:

    php artisan event:generate

<a name="manually-registering-events"></a>
### Registro manual de eventos : Manually Registering Events

Por lo general, los eventos deben registrarse a través del array `EventServiceProvider` `$listen`; sin embargo, también puede registrar eventos basados ​​en Closure manualmente en el método `boot` de `EventServiceProvider`:
> > Typically, events should be registered via the `EventServiceProvider` `$listen` array; however, you may also register Closure based events manually in the `boot` method of your `EventServiceProvider`:

    /**
     * Register any other events for your application.
     *
     * @return void
     */
    public function boot()
    {
        parent::boot();

        Event::listen('event.name', function ($foo, $bar) {
            //
        });
    }

#### Oyentes de eventos con comodines : Wildcard Event Listeners

Incluso puede registrar oyentes utilizando `*` como parámetro comodín, lo que le permite capturar múltiples eventos en el mismo oyente. Los oyentes comodines reciben el nombre del evento como primer argumento y toda el array de datos del evento como segundo argumento:
> > You may even register listeners using the `*` as a wildcard parameter, allowing you to catch multiple events on the same listener. Wildcard listeners receive the event name as their first argument, and the entire event data array as their second argument:

    Event::listen('event.*', function ($eventName, array $data) {
        //
    });

<a name="defining-events"></a>
## Definición de eventos : Defining Events

Una clase de evento es un contenedor de datos que contiene la información relacionada con el evento. Por ejemplo, supongamos que nuestro evento `OrderShipped` generado recibe un objeto [Eloquent ORM](/docs/{{version}}/eloquent):
> > An event class is a data container which holds the information related to the event. For example, let's assume our generated `OrderShipped` event receives an [Eloquent ORM](/docs/{{version}}/eloquent) object:

    <?php

    namespace App\Events;

    use App\Order;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped
    {
        use SerializesModels;

        public $order;

        /**
         * Create a new event instance.
         *
         * @param  \App\Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }
    }

Como puede ver, esta clase de evento no contiene lógica. Es un contenedor para la instancia `Order` que se compró. El rasgo `SerializesModels` utilizado por el evento serializará elegantemente cualquier modelo Eloquent si el objeto del evento se serializa usando la función `serialize` de PHP.
> > As you can see, this event class contains no logic. It is a container for the `Order` instance that was purchased. The `SerializesModels` trait used by the event will gracefully serialize any Eloquent models if the event object is serialized using PHP's `serialize` function.

<a name="defining-listeners"></a>
## Definición de oyentes : Defining Listeners

A continuación, echemos un vistazo al oyente para nuestro evento de ejemplo. Los oyentes de eventos reciben la instancia del evento en su método `handle`. El comando `event:generate` importará automáticamente la clase de evento apropiada e indicará el evento en el método `handle`. Dentro del método `handle`, puede realizar las acciones necesarias para responder al evento:
> > Next, let's take a look at the listener for our example event. Event listeners receive the event instance in their `handle` method. The `event:generate` command will automatically import the proper event class and type-hint the event on the `handle` method. Within the `handle` method, you may perform any actions necessary to respond to the event:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;

    class SendShipmentNotification
    {
        /**
         * Create the event listener.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Handle the event.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            // Access the order using $event->order...
        }
    }

> {tip} Sus oyentes de eventos también pueden indicar cualquier dependencia que necesiten en sus constructores. Todos los detectores de eventos se resuelven a través del [contenedor de servicios](/docs/{{version}}/container) de Laravel, por lo que las dependencias se inyectarán automáticamente.
> > > {tip} Your event listeners may also type-hint any dependencies they need on their constructors. All event listeners are resolved via the Laravel [service container](/docs/{{version}}/container), so dependencies will be injected automatically.

#### Detener la propagación de un evento : Stopping The Propagation Of An Event

A veces, es posible que desee detener la propagación de un evento a otros oyentes. Puede hacerlo devolviendo `false` del método `handle` de su oyente.
> > Sometimes, you may wish to stop the propagation of an event to other listeners. You may do so by returning `false` from your listener's `handle` method.

<a name="queued-event-listeners"></a>
## Oyentes de eventos en cola : Queued Event Listeners

Los oyentes en cola pueden ser beneficiosos si su oyente va a realizar una tarea lenta, como enviar un correo electrónico o realizar una solicitud HTTP. Antes de comenzar con escuchas en cola, asegúrese de [configurar su cola](/docs/{{version}}/queues) y comience a escuchar la cola en su servidor o entorno de desarrollo local.
> > Queueing listeners can be beneficial if your listener is going to perform a slow task such as sending an e-mail or making an HTTP request. Before getting started with queued listeners, make sure to [configure your queue](/docs/{{version}}/queues) and start a queue listener on your server or local development environment.

Para especificar que un oyente debe estar en cola, agregue la interfaz `ShouldQueue` a la clase de oyente. Los oyentes generados por el comando Artisan `event:generate` ya tienen esta interfaz importada en el espacio de nombres actual, por lo que puede usarla inmediatamente:
> > To specify that a listener should be queued, add the `ShouldQueue` interface to the listener class. Listeners generated by the `event:generate` Artisan command already have this interface imported into the current namespace, so you can use it immediately:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        //
    }

¡Eso es! Ahora, cuando se llama a este oyente para un evento, el asignador de eventos lo pondrá en cola automáticamente utilizando el [sistema de cola](/docs/{{version}}/queues) de Laravel. Si no se lanzan excepciones cuando la cola ejecuta el oyente, el trabajo en cola se eliminará automáticamente después de que haya finalizado el proceso.
> > That's it! Now, when this listener is called for an event, it will be automatically queued by the event dispatcher using Laravel's [queue system](/docs/{{version}}/queues). If no exceptions are thrown when the listener is executed by the queue, the queued job will automatically be deleted after it has finished processing.

#### Personalización de la conexión de cola y el nombre de cola : Customizing The Queue Connection & Queue Name

Si desea personalizar la conexión de la cola y el nombre de la cola utilizados por un detector de eventos, puede definir las propiedades `$connection` y `$queue` en su clase de escucha:
> > If you would like to customize the queue connection and queue name used by an event listener, you may define `$connection` and `$queue` properties on your listener class:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        /**
         * The name of the connection the job should be sent to.
         *
         * @var string|null
         */
        public $connection = 'sqs';

        /**
         * The name of the queue the job should be sent to.
         *
         * @var string|null
         */
        public $queue = 'listeners';
    }

<a name="manually-accessing-the-queue"></a>
### Acceso manual a la cola : Manually Accessing The Queue

Si necesita acceder manualmente a los métodos `delete` y `release` del trabajo de cola subyacente del oyente, puede hacerlo utilizando el rasgo `Illuminate\Queue\InteractsWithQueue`. Este rasgo se importa por defecto en los oyentes generados y proporciona acceso a estos métodos:
> > If you need to manually access the listener's underlying queue job's `delete` and `release` methods, you may do so using the `Illuminate\Queue\InteractsWithQueue` trait. This trait is imported by default on generated listeners and provides access to these methods:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * Handle the event.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="handling-failed-jobs"></a>
### Manejo de trabajos fallidos : Handling Failed Jobs

En ocasiones, los oyentes de su evento en cola pueden fallar. Si el oyente en cola excede el número máximo de intentos según lo definido por su trabajador de cola, se llamará al método `failed` en su oyente. El método `failed` recibe la instancia del evento y la excepción que causó la falla:
> > Sometimes your queued event listeners may fail. If queued listener exceeds the maximum number of attempts as defined by your queue worker, the `failed` method will be called on your listener. The `failed` method receives the event instance and the exception that caused the failure:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * Handle the event.
         *
         * @param  \App\Events\OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            //
        }

        /**
         * Handle a job failure.
         *
         * @param  \App\Events\OrderShipped  $event
         * @param  \Exception  $exception
         * @return void
         */
        public function failed(OrderShipped $event, $exception)
        {
            //
        }
    }

<a name="dispatching-events"></a>
## Despacho de eventos : Dispatching Events

Para enviar un evento, puede pasar una instancia del evento al helper `event`. El asistente enviará el evento a todos sus oyentes registrados. Dado que el helper `event` está disponible globalmente, puede llamarlo desde cualquier lugar de su aplicación:
> > To dispatch an event, you may pass an instance of the event to the `event` helper. The helper will dispatch the event to all of its registered listeners. Since the `event` helper is globally available, you may call it from anywhere in your application:

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  int  $orderId
         * @return Response
         */
        public function ship($orderId)
        {
            $order = Order::findOrFail($orderId);

            // Order shipment logic...

            event(new OrderShipped($order));
        }
    }

> {tip} Cuando se prueba, puede ser útil afirmar que se enviaron ciertos eventos sin activar realmente a sus oyentes. Los [ayudantes de prueba incorporados](/docs/{{version}}/mocking#event-fake) en Laravel lo hacen fácil.
> > > {tip} When testing, it can be helpful to assert that certain events were dispatched without actually triggering their listeners. Laravel's [built-in testing helpers](/docs/{{version}}/mocking#event-fake) makes it a cinch.

<a name="event-subscribers"></a>
## Suscriptores de eventos : Event Subscribers

<a name="writing-event-subscribers"></a>
### Escritura de suscriptores de eventos : Writing Event Subscribers

Los suscriptores de eventos son clases que pueden suscribirse a múltiples eventos desde la propia clase, lo que le permite definir varios manejadores de eventos dentro de una sola clase. Los suscriptores deben definir un método `subscribe`, que se pasará a una instancia de despachador de eventos. Puede llamar al método `listen` en el despachador dado para registrar oyentes de eventos:
> > Event subscribers are classes that may subscribe to multiple events from within the class itself, allowing you to define several event handlers within a single class. Subscribers should define a `subscribe` method, which will be passed an event dispatcher instance. You may call the `listen` method on the given dispatcher to register event listeners:

    <?php

    namespace App\Listeners;

    class UserEventSubscriber
    {
        /**
         * Handle user login events.
         */
        public function onUserLogin($event) {}

        /**
         * Handle user logout events.
         */
        public function onUserLogout($event) {}

        /**
         * Register the listeners for the subscriber.
         *
         * @param  \Illuminate\Events\Dispatcher  $events
         */
        public function subscribe($events)
        {
            $events->listen(
                'Illuminate\Auth\Events\Login',
                'App\Listeners\UserEventSubscriber@onUserLogin'
            );

            $events->listen(
                'Illuminate\Auth\Events\Logout',
                'App\Listeners\UserEventSubscriber@onUserLogout'
            );
        }

    }

<a name="registering-event-subscribers"></a>
### Registro de suscriptores de eventos : Registering Event Subscribers

Después de escribir el suscriptor, está listo para registrarlo con el despachador de eventos. Puede registrar suscriptores utilizando la propiedad `$subscribe` en `EventServiceProvider`. Por ejemplo, agreguemos el `UserEventSubscriber` a la lista:
> > After writing the subscriber, you are ready to register it with the event dispatcher. You may register subscribers using the `$subscribe` property on the `EventServiceProvider`. For example, let's add the `UserEventSubscriber` to the list:

    <?php

    namespace App\Providers;

    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * The event listener mappings for the application.
         *
         * @var array
         */
        protected $listen = [
            //
        ];

        /**
         * The subscriber classes to register.
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventSubscriber',
        ];
    }
