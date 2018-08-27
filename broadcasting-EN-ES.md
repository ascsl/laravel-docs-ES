# Transmisión : Broadcasting

- [Introduction](#introduction)
    - [Configuration](#configuration)
    - [Driver Prerequisites](#driver-prerequisites)
- [Concept Overview](#concept-overview)
    - [Using An Example Application](#using-example-application)
- [Defining Broadcast Events](#defining-broadcast-events)
    - [Broadcast Name](#broadcast-name)
    - [Broadcast Data](#broadcast-data)
    - [Broadcast Queue](#broadcast-queue)
    - [Broadcast Conditions](#broadcast-conditions)
- [Authorizing Channels](#authorizing-channels)
    - [Defining Authorization Routes](#defining-authorization-routes)
    - [Defining Authorization Callbacks](#defining-authorization-callbacks)
    - [Defining Channel Classes](#defining-channel-classes)
- [Broadcasting Events](#broadcasting-events)
    - [Only To Others](#only-to-others)
- [Receiving Broadcasts](#receiving-broadcasts)
    - [Installing Laravel Echo](#installing-laravel-echo)
    - [Listening For Events](#listening-for-events)
    - [Leaving A Channel](#leaving-a-channel)
    - [Namespaces](#namespaces)
- [Presence Channels](#presence-channels)
    - [Authorizing Presence Channels](#authorizing-presence-channels)
    - [Joining Presence Channels](#joining-presence-channels)
    - [Broadcasting To Presence Channels](#broadcasting-to-presence-channels)
- [Client Events](#client-events)
- [Notifications](#notifications)

<a name="introduction"></a>
## Introducción : Introduction

En muchas aplicaciones web modernas, los WebSockets se utilizan para implementar interfaces de usuario en tiempo real y actualizadas. Cuando se actualizan algunos datos en el servidor, típicamente se envía un mensaje a través de una conexión WebSocket para ser manejado por el cliente. Esto proporciona una alternativa más robusta y eficiente para realizar un sondeo continuo de cambios en su aplicación.
> > In many modern web applications, WebSockets are used to implement realtime, live-updating user interfaces. When some data is updated on the server, a message is typically sent over a WebSocket connection to be handled by the client. This provides a more robust, efficient alternative to continually polling your application for changes.

Para ayudarlo a crear este tipo de aplicaciones, Laravel facilita la "transmisión" de sus [eventos](/docs/{{version}}/events) a través de una conexión WebSocket. Transmitir sus eventos de Laravel le permite compartir los mismos nombres de eventos entre su código del lado del servidor y su aplicación de JavaScript del lado del cliente.
> > To assist you in building these types of applications, Laravel makes it easy to "broadcast" your [events](/docs/{{version}}/events) over a WebSocket connection. Broadcasting your Laravel events allows you to share the same event names between your server-side code and your client-side JavaScript application.

> {tip} Antes de sumergirte en la transmisión de eventos, asegúrate de haber leído toda la documentación relacionada con Laravel [events and listeners](/docs/{{version}}/events).
> > > {tip} Before diving into event broadcasting, make sure you have read all of the documentation regarding Laravel [events and listeners](/docs/{{version}}/events).

<a name="configuration"></a>
### Configuración : Configuration

Toda la configuración de transmisión de eventos de su aplicación se almacena en el archivo de configuración `config/broadcasting.php`. Laravel es compatible con varios controladores de transmisión: [Empujador](https://pusher.com), [Redis](/docs/{{version}}/redis) y un controlador `log` para el desarrollo local y la depuración. Además, se incluye un controlador `null` que le permite deshabilitar totalmente la transmisión. Se incluye un ejemplo de configuración para cada uno de estos controladores en el archivo de configuración `config/broadcasting.php`.
> > All of your application's event broadcasting configuration is stored in the `config/broadcasting.php` configuration file. Laravel supports several broadcast drivers out of the box: [Pusher](https://pusher.com), [Redis](/docs/{{version}}/redis), and a `log` driver for local development and debugging. Additionally, a `null` driver is included which allows you to totally disable broadcasting. A configuration example is included for each of these drivers in the `config/broadcasting.php` configuration file.

#### Proveedor de servicios de difusión : Broadcast Service Provider

Antes de transmitir cualquier evento, primero deberá registrar la `App\Providers\BroadcastServiceProvider`. En las aplicaciones nuevas de Laravel, solo necesita descomentar este proveedor en la matriz `providers` de su archivo de configuración `config/app.php`. Este proveedor le permitirá registrar las rutas de autorización de transmisión y las devoluciones de llamada.
> > Before broadcasting any events, you will first need to register the `App\Providers\BroadcastServiceProvider`. In fresh Laravel applications, you only need to uncomment this provider in the `providers` array of your `config/app.php` configuration file. This provider will allow you to register the broadcast authorization routes and callbacks.

#### Símbolo CSRF : CSRF Token

[Laravel Echo](#installation-laravel-echo) necesitará acceder al token CSRF de la sesión actual. Debe verificar que el elemento HTML `head` de su aplicación define una etiqueta `meta` que contiene el token CSRF:
> > [Laravel Echo](#installing-laravel-echo) will need access to the current session's CSRF token. You should verify that your application's `head` HTML element defines a `meta` tag containing the CSRF token:

    <meta name="csrf-token" content="{{ csrf_token() }}">

<a name="driver-prerequisites"></a>
### Prerrequisitos del Driver : Driver Prerequisites

#### Pusher

Si está transmitiendo sus eventos a través de [Pusher](https://pusher.com), debe instalar el SDK PHP de Pusher usando el administrador de paquetes Composer:
> > If you are broadcasting your events over [Pusher](https://pusher.com), you should install the Pusher PHP SDK using the Composer package manager:

    composer require pusher/pusher-php-server "~3.0"

A continuación, debe configurar sus credenciales de Pusher en el archivo de configuración `config/broadcasting.php`. Ya está incluido en este archivo un ejemplo de configuración de Pusher, lo que le permite especificar rápidamente su clave de Pusher, secreto e ID de aplicación. La configuración `pusher` del archivo `config/broadcasting.php` también le permite especificar `options` adicionales soportadas por Pusher, como el clúster:
> > Next, you should configure your Pusher credentials in the `config/broadcasting.php` configuration file. An example Pusher configuration is already included in this file, allowing you to quickly specify your Pusher key, secret, and application ID. The `config/broadcasting.php` file's `pusher` configuration also allows you to specify additional `options` that are supported by Pusher, such as the cluster:

    'options' => [
        'cluster' => 'eu',
        'encrypted' => true
    ],

Cuando utilice Pusher y [Laravel Echo] (#installation-laravel-echo), debe especificar `pusher` como su emisora ​​deseada al crear la instancia de Echo en su archivo `resources/assets/js/bootstrap.js`:
> > When using Pusher and [Laravel Echo](#installing-laravel-echo), you should specify `pusher` as your desired broadcaster when instantiating the Echo instance in your `resources/assets/js/bootstrap.js` file:

    import Echo from "laravel-echo"

    window.Pusher = require('pusher-js');

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key'
    });

#### Redis

Si está utilizando la emisora ​​Redis, debe instalar la biblioteca Predis:
> > If you are using the Redis broadcaster, you should install the Predis library:

    composer require predis/predis

La emisora ​​Redis transmitirá mensajes utilizando la función pub / sub de Redis; sin embargo, deberá emparejar esto con un servidor WebSocket que pueda recibir los mensajes de Redis y transmitirlos a sus canales WebSocket.
> > The Redis broadcaster will broadcast messages using Redis' pub / sub feature; however, you will need to pair this with a WebSocket server that can receive the messages from Redis and broadcast them to your WebSocket channels.

Cuando el presentador de Redis publique un evento, se publicará en los nombres de canal especificados del evento y la carga será una cadena codificada JSON que contiene el nombre del evento, una carga de datos `data` y el usuario que generó el ID del socket del evento (si corresponde) )
> > When the Redis broadcaster publishes an event, it will be published on the event's specified channel names and the payload will be a JSON encoded string containing the event name, a `data` payload, and the user that generated the event's socket ID (if applicable).

#### Socket.IO

Si va a vincular la emisora ​​Redis con un servidor Socket.IO, deberá incluir la biblioteca cliente Socket.IO JavaScript en su aplicación. Puede instalarlo a través del administrador de paquetes de NPM:
> > If you are going to pair the Redis broadcaster with a Socket.IO server, you will need to include the Socket.IO JavaScript client library in your application. You may install it via the NPM package manager:

    npm install --save socket.io-client

Next, you will need to instantiate Echo with the `socket.io` connector and a `host`.
A continuación, deberá crear una instancia de Echo con el conector `socket.io` y un `host`.

    import Echo from "laravel-echo"

    window.io = require('socket.io-client');

    window.Echo = new Echo({
        broadcaster: 'socket.io',
        host: window.location.hostname + ':6001'
    });

Finalmente, deberá ejecutar un servidor Socket.IO compatible. Laravel no incluye una implementación de servidor Socket.IO; sin embargo, actualmente se mantiene un servidor Socket.IO gestionado por la comunidad en el repositorio de [tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server) GitHub.
> > Finally, you will need to run a compatible Socket.IO server. Laravel does not include a Socket.IO server implementation; however, a community driven Socket.IO server is currently maintained at the [tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server) GitHub repository.

#### Prerrequisitos de cola : Queue Prerequisites

Antes de transmitir eventos, también deberá configurar y ejecutar un [escuchador de cola](/docs/{{version}}/queues). Toda la transmisión de eventos se realiza a través de trabajos en cola para que el tiempo de respuesta de su aplicación no se vea seriamente afectado.
> > Before broadcasting events, you will also need to configure and run a [queue listener](/docs/{{version}}/queues). All event broadcasting is done via queued jobs so that the response time of your application is not seriously affected.

<a name="concept-overview"></a>
## Concepto general : Concept Overview

La transmisión de eventos de Laravel le permite transmitir sus eventos Laravel del lado del servidor a su aplicación de JavaScript del lado del cliente utilizando un enfoque basado en controladores para WebSockets. Actualmente, Laravel se envía con [Pusher] (https://pusher.com) y los controladores de Redis. Los eventos pueden consumirse fácilmente en el lado del cliente utilizando el paquete [Laravel Echo](#instalación-laravel-echo) Javascript.
> > Laravel's event broadcasting allows you to broadcast your server-side Laravel events to your client-side JavaScript application using a driver-based approach to WebSockets. Currently, Laravel ships with [Pusher](https://pusher.com) and Redis drivers. The events may be easily consumed on the client-side using the [Laravel Echo](#installing-laravel-echo) Javascript package.

Los eventos se transmiten a través de "canales", que pueden especificarse como públicos o privados. Cualquier visitante a su aplicación puede suscribirse a un canal público sin ninguna autenticación o autorización; sin embargo, para suscribirse a un canal privado, un usuario debe estar autenticado y autorizado para escuchar en ese canal.
> > Events are broadcast over "channels", which may be specified as public or private. Any visitor to your application may subscribe to a public channel without any authentication or authorization; however, in order to subscribe to a private channel, a user must be authenticated and authorized to listen on that channel.

<a name="using-example-application"></a>
### Usando una aplicación de ejemplo : Using An Example Application

Antes de sumergirnos en cada componente de la transmisión de eventos, tomemos una descripción general de alto nivel usando una tienda de comercio electrónico como ejemplo. No discutiremos los detalles de la configuración de [Pusher](https://pusher.com) o [Laravel Echo](#installation-laravel-echo) ya que se analizará en detalle en otras secciones de esta documentación.
> > Before diving into each component of event broadcasting, let's take a high level overview using an e-commerce store as an example. We won't discuss the details of configuring [Pusher](https://pusher.com) or [Laravel Echo](#installing-laravel-echo) since that will be discussed in detail in other sections of this documentation.

En nuestra aplicación, supongamos que tenemos una página que permite a los usuarios ver el estado de envío de sus pedidos. Supongamos también que un evento `ShippingStatusUpdated` se activa cuando la aplicación procesa una actualización de estado de envío:
> > In our application, let's assume we have a page that allows users to view the shipping status for their orders. Let's also assume that a `ShippingStatusUpdated` event is fired when a shipping status update is processed by the application:

    event(new ShippingStatusUpdated($update));

#### La interfaz `ShouldBroadcast` : The `ShouldBroadcast` Interface

Cuando un usuario está viendo una de sus órdenes, no queremos que tenga que actualizar la página para ver las actualizaciones de estado. En cambio, queremos transmitir las actualizaciones a la aplicación a medida que se crean. Entonces, necesitamos marcar el evento `ShippingStatusUpdated` con la interfaz `ShouldBroadcast`. Esto le indicará a Laravel que transmita el evento cuando se dispare:
> > When a user is viewing one of their orders, we don't want them to have to refresh the page to view status updates. Instead, we want to broadcast the updates to the application as they are created. So, we need to mark the `ShippingStatusUpdated` event with the `ShouldBroadcast` interface. This will instruct Laravel to broadcast the event when it is fired:

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ShippingStatusUpdated implements ShouldBroadcast
    {
        /**
         * Information about the shipping status update.
         *
         * @var string
         */
        public $update;
    }

La interfaz `ShouldBroadcast` requiere que nuestro evento defina un método `broadcastOn`. Este método es responsable de devolver los canales que el evento debería transmitir. Ya se ha definido un trozo vacío de este método en las clases de eventos generados, por lo que solo debemos completar sus detalles. Solo queremos que el creador del pedido pueda ver las actualizaciones de estado, por lo que transmitiremos el evento en un canal privado que esté vinculado al pedido:
> > The `ShouldBroadcast` interface requires our event to define a `broadcastOn` method. This method is responsible for returning the channels that the event should broadcast on. An empty stub of this method is already defined on generated event classes, so we only need to fill in its details. We only want the creator of the order to be able to view status updates, so we will broadcast the event on a private channel that is tied to the order:

    /**
     * Get the channels the event should broadcast on.
     *
     * @return array
     */
    public function broadcastOn()
    {
        return new PrivateChannel('order.'.$this->update->order_id);
    }

#### Autorización de canales : Authorizing Channels

Recuerde, los usuarios deben estar autorizados para escuchar en canales privados. Podemos definir nuestras reglas de autorización de canal en el archivo `routes/channels.php`. En este ejemplo, debemos verificar que cualquier usuario que intente escuchar en el canal privado `order.1` sea en realidad el creador del pedido:
> > Remember, users must be authorized to listen on private channels. We may define our channel authorization rules in the `routes/channels.php` file. In this example, we need to verify that any user attempting to listen on the private `order.1` channel is actually the creator of the order:

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

El método `channel` acepta dos argumentos: el nombre del canal y una devolución de llamada que devuelve `true` o `false` indicando si el usuario está autorizado a escuchar en el canal.
> > The `channel` method accepts two arguments: the name of the channel and a callback which returns `true` or `false` indicating whether the user is authorized to listen on the channel.

Todas las rellamadas de autorización reciben al usuario autenticado actualmente como su primer argumento y cualquier parámetro de comodín adicional como argumentos posteriores. En este ejemplo, estamos usando el marcador de posición `{orderId}` para indicar que la porción "ID" del nombre del canal es un comodín.
> > All authorization callbacks receive the currently authenticated user as their first argument and any additional wildcard parameters as their subsequent arguments. In this example, we are using the `{orderId}` placeholder to indicate that the "ID" portion of the channel name is a wildcard.

#### Escuchando transmisiones de eventos : Listening For Event Broadcasts

A continuación, todo lo que queda es escuchar el evento en nuestra aplicación de JavaScript. Podemos hacer esto usando Laravel Echo. Primero, usaremos el método `private` para suscribirnos al canal privado. Entonces, podemos usar el método `listen` para escuchar el evento `ShippingStatusUpdated`. Por defecto, todas las propiedades públicas del evento se incluirán en el evento de difusión:
> > Next, all that remains is to listen for the event in our JavaScript application. We can do this using Laravel Echo. First, we'll use the `private` method to subscribe to the private channel. Then, we may use the `listen` method to listen for the `ShippingStatusUpdated` event. By default, all of the event's public properties will be included on the broadcast event:

    Echo.private(`order.${orderId}`)
        .listen('ShippingStatusUpdated', (e) => {
            console.log(e.update);
        });

<a name="defining-broadcast-events"></a>
## Definición de eventos de difusión : Defining Broadcast Events

Para informarle a Laravel que un evento determinado debe transmitirse, implemente la interfaz `Illuminate\Contracts\Broadcasting\ShouldBroadcast` en la clase de evento. Esta interfaz ya está importada en todas las clases de eventos generadas por el marco, por lo que puede agregarla fácilmente a cualquiera de sus eventos.
> > To inform Laravel that a given event should be broadcast, implement the `Illuminate\Contracts\Broadcasting\ShouldBroadcast` interface on the event class. This interface is already imported into all event classes generated by the framework so you may easily add it to any of your events.

La interfaz `ShouldBroadcast` requiere que implementes un único método: `broadcastOn`. El método `broadcastOn` debe devolver un canal o conjunto de canales que el evento debería transmitir. Los canales deben ser instancias de `Channel`, `PrivateChannel`, o `PresenceChannel`. Las instancias de `Channel` representan canales públicos a los que cualquier usuario puede suscribirse, mientras que `PrivateChannels` y `PresenceChannels` representan canales privados que requieren [autorización de canal](#authorizing-channels):
> > The `ShouldBroadcast` interface requires you to implement a single method: `broadcastOn`. The `broadcastOn` method should return a channel or array of channels that the event should broadcast on. The channels should be instances of `Channel`, `PrivateChannel`, or `PresenceChannel`. Instances of `Channel` represent public channels that any user may subscribe to, while `PrivateChannels` and `PresenceChannels` represent private channels that require [channel authorization](#authorizing-channels):

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ServerCreated implements ShouldBroadcast
    {
        use SerializesModels;

        public $user;

        /**
         * Create a new event instance.
         *
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * Get the channels the event should broadcast on.
         *
         * @return Channel|array
         */
        public function broadcastOn()
        {
            return new PrivateChannel('user.'.$this->user->id);
        }
    }

Entonces, solo necesita [activar el evento](/docs/{{version}}/events) como lo haría normalmente. Una vez que se ha disparado el evento, un [trabajo en cola](/docs/{{version}}/queues) transmitirá automáticamente el evento a través del controlador de transmisión especificado.
> > Then, you only need to [fire the event](/docs/{{version}}/events) as you normally would. Once the event has been fired, a [queued job](/docs/{{version}}/queues) will automatically broadcast the event over your specified broadcast driver.

<a name="broadcast-name"></a>
### Nombre de transmisión : Broadcast Name

Por defecto, Laravel transmitirá el evento utilizando el nombre de clase del evento. Sin embargo, puede personalizar el nombre de la emisión definiendo un método `broadcastAs` en el evento:
> > By default, Laravel will broadcast the event using the event's class name. However, you may customize the broadcast name by defining a `broadcastAs` method on the event:

    /**
     * The event's broadcast name.
     *
     * @return string
     */
    public function broadcastAs()
    {
        return 'server.created';
    }

Si personaliza el nombre de la emisión usando el método `broadcastAs`, debe asegurarse de registrar su oyente con un caracter `.`. Esto indicará a Echo que no anteponga el espacio de nombre de la aplicación al evento:
> > If you customize the broadcast name using the `broadcastAs` method, you should make sure to register your listener with a leading `.` character. This will instruct Echo to not prepend the application's namespace to the event:

    .listen('.server.created', function (e) {
        ....
    });

<a name="broadcast-data"></a>
### Transimión de datos : Broadcast Data

Cuando se transmite un evento, todas sus propiedades `public` se serializan automáticamente y se transmiten como la carga útil del evento, lo que le permite acceder a cualquiera de sus datos públicos desde su aplicación de JavaScript. Entonces, por ejemplo, si su evento tiene una sola propiedad pública `$user` que contiene un modelo Eloquent, la carga útil de difusión del evento sería:
> > When an event is broadcast, all of its `public` properties are automatically serialized and broadcast as the event's payload, allowing you to access any of its public data from your JavaScript application. So, for example, if your event has a single public `$user` property that contains an Eloquent model, the event's broadcast payload would be:

    {
        "user": {
            "id": 1,
            "name": "Patrick Stewart"
            ...
        }
    }

Sin embargo, si desea tener un control más detallado sobre la carga de difusión, puede agregar un método `broadcastWith` a su evento. Este método debe devolver la matriz de datos que desea transmitir como la carga del evento:
> > However, if you wish to have more fine-grained control over your broadcast payload, you may add a `broadcastWith` method to your event. This method should return the array of data that you wish to broadcast as the event payload:

    /**
     * Get the data to broadcast.
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['id' => $this->user->id];
    }

<a name="broadcast-queue"></a>
### Colas de transmisión : Broadcast Queue

Por defecto, cada evento de difusión se coloca en la cola predeterminada para la conexión de cola predeterminada especificada en el archivo de configuración `queue.php`. Puede personalizar la cola utilizada por la emisora ​​definiendo una propiedad `broadcastQueue` en su clase de evento. Esta propiedad debe especificar el nombre de la cola que desea utilizar al transmitir:
> > By default, each broadcast event is placed on the default queue for the default queue connection specified in your `queue.php` configuration file. You may customize the queue used by the broadcaster by defining a `broadcastQueue` property on your event class. This property should specify the name of the queue you wish to use when broadcasting:

    /**
     * The name of the queue on which to place the event.
     *
     * @var string
     */
    public $broadcastQueue = 'your-queue-name';

Si desea transmitir su evento utilizando la cola `sync` en lugar del controlador de cola predeterminado, puede implementar la interfaz `ShouldBroadcastNow` en lugar de `ShouldBroadcast`:
> > If you want to broadcast your event using the `sync` queue instead of the default queue driver, you can implement the `ShouldBroadcastNow` interface instead of `ShouldBroadcast`:

    <?php

    use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

    class ShippingStatusUpdated implements ShouldBroadcastNow
    {
        //
    }
<a name="broadcast-conditions"></a>
### Condiciones de transmisión : Broadcast Conditions

Algunas veces desea transmitir su evento solo si una condición dada es verdadera. Puede definir estas condiciones agregando un método `broadcastWhen` a su clase de evento:
> > Sometimes you want to broadcast your event only if a given condition is true. You may define these conditions by adding a `broadcastWhen` method to your event class:

    /**
     * Determine if this event should broadcast.
     *
     * @return bool
     */
    public function broadcastWhen()
    {
        return $this->value > 100;
    }

<a name="authorizing-channels"></a>
## Autorización de canales : Authorizing Channels

Los canales privados requieren que usted autorice que el usuario actualmente autenticado pueda realmente escuchar en el canal. Esto se logra haciendo una solicitud HTTP a su aplicación Laravel con el nombre del canal y permitiendo que su aplicación determine si el usuario puede escuchar en ese canal. Al usar [Laravel Echo](#installation-laravel-echo), la solicitud de HTTP para autorizar las suscripciones a canales privados se realizará automáticamente; sin embargo, debe definir las rutas adecuadas para responder a estas solicitudes.
> > Private channels require you to authorize that the currently authenticated user can actually listen on the channel. This is accomplished by making an HTTP request to your Laravel application with the channel name and allowing your application to determine if the user can listen on that channel. When using [Laravel Echo](#installing-laravel-echo), the HTTP request to authorize subscriptions to private channels will be made automatically; however, you do need to define the proper routes to respond to these requests.

<a name="defining-authorization-routes"></a>
### Definición de rutas de autorización : Defining Authorization Routes

Afortunadamente, Laravel facilita la definición de las rutas para responder a las solicitudes de autorización de canales. En el `BroadcastServiceProvider` incluido con su aplicación Laravel, verá una llamada al método `Broadcast::routes`. Este método registrará la ruta `/broadcasting/auth` para manejar solicitudes de autorización:
> > Thankfully, Laravel makes it easy to define the routes to respond to channel authorization requests. In the `BroadcastServiceProvider` included with your Laravel application, you will see a call to the `Broadcast::routes` method. This method will register the `/broadcasting/auth` route to handle authorization requests:

    Broadcast::routes();

El método `Broadcast::routes` colocará automáticamente sus rutas dentro del grupo de middleware `web`; sin embargo, puede pasar una matriz de atributos de ruta al método si desea personalizar los atributos asignados:
> > The `Broadcast::routes` method will automatically place its routes within the `web` middleware group; however, you may pass an array of route attributes to the method if you would like to customize the assigned attributes:

    Broadcast::routes($attributes);

<a name="defining-authorization-callbacks"></a>
### Definición de rellamadas de autorización : Defining Authorization Callbacks

A continuación, debemos definir la lógica que realmente realizará la autorización del canal. Esto se hace en el archivo `routes/channels.php` que se incluye con su aplicación. En este archivo, puede usar el método `Broadcast::channel` para registrar las devoluciones de llamadas de autorizaciones de canales:
> > Next, we need to define the logic that will actually perform the channel authorization. This is done in the `routes/channels.php` file that is included with your application. In this file, you may use the `Broadcast::channel` method to register channel authorization callbacks:

    Broadcast::channel('order.{orderId}', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

El método `channel` acepta dos argumentos: el nombre del canal y una devolución de llamada que devuelve `true` o `false` indicando si el usuario está autorizado a escuchar en el canal.
> > The `channel` method accepts two arguments: the name of the channel and a callback which returns `true` or `false` indicating whether the user is authorized to listen on the channel.

Todas las rellamadas de autorización reciben al usuario autenticado actualmente como su primer argumento y cualquier parámetro de comodín adicional como argumentos posteriores. En este ejemplo, estamos usando el marcador de posición `{orderId}` para indicar que la porción "ID" del nombre del canal es un comodín.
> > All authorization callbacks receive the currently authenticated user as their first argument and any additional wildcard parameters as their subsequent arguments. In this example, we are using the `{orderId}` placeholder to indicate that the "ID" portion of the channel name is a wildcard.

#### Autorización Retrollamada del modelo de rellamada : Authorization Callback Model Binding

Al igual que las rutas HTTP, las rutas de los canales también pueden aprovechar la [vinculación del modelo de ruta] implícita y explícita (/docs/{{version}}/routing#route-model-binding). Por ejemplo, en lugar de recibir la identificación de la cadena o de la orden numérica, puede solicitar una instancia del modelo `Order` real:
> > Just like HTTP routes, channel routes may also take advantage of implicit and explicit [route model binding](/docs/{{version}}/routing#route-model-binding). For example, instead of receiving the string or numeric order ID, you may request an actual `Order` model instance:

    use App\Order;

    Broadcast::channel('order.{order}', function ($user, Order $order) {
        return $user->id === $order->user_id;
    });

<a name="defining-channel-classes"></a>
### Definición de clases de canales : Defining Channel Classes

Si su aplicación consume muchos canales diferentes, su archivo `routes/channels.php` podría volverse voluminoso. Por lo tanto, en lugar de utilizar Closures para autorizar canales, puede usar clases de canales. Para generar una clase de canal, use el comando Artesanal `make:channel`. Este comando colocará una nueva clase de canal en el directorio `App/Broadcasting`.
> > If your application is consuming many different channels, your `routes/channels.php` file could become bulky. So, instead of using Closures to authorize channels, you may use channel classes. To generate a channel class, use the `make:channel` Artisan command. This command will place a new channel class in the `App/Broadcasting` directory.

    php artisan make:channel OrderChannel

Luego, registre su canal en su archivo `routes / channels.php`:
> > Next, register your channel in your `routes/channels.php` file:

    use App\Broadcasting\OrderChannel;

    Broadcast::channel('order.{order}', OrderChannel::class);

Finally, you may place the authorization logic for your channel in the channel class' `join` method. This `join` method will house the same logic you would have typically placed in your channel authorization Closure. Of course, you may also take advantage of channel model binding:
Finalmente, puede colocar la lógica de autorización para su canal en el método `join` de la clase del canal. Este método `join` albergará la misma lógica que normalmente habría colocado en el cierre de autorización de su canal. Por supuesto, también puede aprovechar el enlace de modelo de canal:

    <?php

    namespace App\Broadcasting;

    use App\User;
    use App\Order;

    class OrderChannel
    {
        /**
         * Create a new channel instance.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Authenticate the user's access to the channel.
         *
         * @param  \App\User  $user
         * @param  \App\Order  $order
         * @return array|bool
         */
        public function join(User $user, Order $order)
        {
            return $user->id === $order->user_id;
        }
    }

> {tip} Como muchas otras clases en Laravel, las clases de canales se resolverán automáticamente por [contenedor de servicio](/docs/{{version}}/container). Por lo tanto, puede escribir-insinuar cualquier dependencia requerida por su canal en su constructor.
> > > {tip} Like many other classes in Laravel, channel classes will automatically be resolved by the [service container](/docs/{{version}}/container). So, you may type-hint any dependencies required by your channel in its constructor.

<a name="broadcasting-events"></a>
## Eventos de transmisión : Broadcasting Events

Una vez que haya definido un evento y lo haya marcado con la interfaz `ShouldBroadcast`, solo tendrá que iniciar el evento con la función `event`. El despachador del evento notará que el evento está marcado con la interfaz `ShouldBroadcast` y pondrá en cola el evento para la transmisión:
> > Once you have defined an event and marked it with the `ShouldBroadcast` interface, you only need to fire the event using the `event` function. The event dispatcher will notice that the event is marked with the `ShouldBroadcast` interface and will queue the event for broadcasting:

    event(new ShippingStatusUpdated($update));

<a name="only-to-others"></a>
### Solo para otros : Only To Others

Al crear una aplicación que utiliza transmisión de eventos, puede sustituir la función `event` con la función `broadcast`. Al igual que la función `event`, la función` broadcast` distribuye el evento a los oyentes del lado del servidor:
> > When building an application that utilizes event broadcasting, you may substitute the `event` function with the `broadcast` function. Like the `event` function, the `broadcast` function dispatches the event to your server-side listeners:

    broadcast(new ShippingStatusUpdated($update));

Sin embargo, la función `broadcast` también expone el método` toOthers` que le permite excluir al usuario actual de los destinatarios de la transmisión:
> > However, the `broadcast` function also exposes the `toOthers` method which allows you to exclude the current user from the broadcast's recipients:

    broadcast(new ShippingStatusUpdated($update))->toOthers();

Para comprender mejor cuándo puede usar el método `toOthers`, imaginemos una aplicación de lista de tareas donde un usuario puede crear una nueva tarea al ingresar un nombre de tarea. Para crear una tarea, su aplicación puede hacer una solicitud a un punto final `/task` que difunde la creación de la tarea y devuelve una representación JSON de la nueva tarea. Cuando su aplicación JavaScript recibe la respuesta desde el punto final, puede insertar directamente la nueva tarea en su lista de tareas de la siguiente manera:
> > To better understand when you may want to use the `toOthers` method, let's imagine a task list application where a user may create a new task by entering a task name. To create a task, your application might make a request to a `/task` end-point which broadcasts the task's creation and returns a JSON representation of the new task. When your JavaScript application receives the response from the end-point, it might directly insert the new task into its task list like so:

    axios.post('/task', task)
        .then((response) => {
            this.tasks.push(response.data);
        });

Sin embargo, recuerde que también transmitimos la creación de la tarea. Si su aplicación de JavaScript está escuchando este evento para agregar tareas a la lista de tareas, tendrá tareas duplicadas en su lista: una desde el punto final y otra desde la transmisión. Puede resolver esto utilizando el método `toOthers` para indicar al emisor que no transmita el evento al usuario actual.
> > However, remember that we also broadcast the task's creation. If your JavaScript application is listening for this event in order to add tasks to the task list, you will have duplicate tasks in your list: one from the end-point and one from the broadcast. You may solve this by using the `toOthers` method to instruct the broadcaster to not broadcast the event to the current user.

> {note} Su evento debe usar el rasgo `Illuminate\Broadcasting \InteractsWithSockets` para llamar al método `toOthers`.
> > > {note} Your event must use the `Illuminate\Broadcasting\InteractsWithSockets` trait in order to call the `toOthers` method.

#### Configuración : Configuration

Cuando inicializa una instancia de Laravel Echo, se asigna una ID de socket a la conexión. Si está utilizando [Vue](https://vuejs.org) y [Axios](https://github.com/mzabriskie/axios), el ID del socket se adjuntará automáticamente a cada solicitud de salida como un `X-Socket-ID` Luego, cuando llame al método `toOthers`, Laravel extraerá la ID del socket del encabezado e instruirá al broadcaster para que no se transmita a ninguna conexión con ese ID de socket.
> > When you initialize a Laravel Echo instance, a socket ID is assigned to the connection. If you are using [Vue](https://vuejs.org) and [Axios](https://github.com/mzabriskie/axios), the socket ID will automatically be attached to every outgoing request as a `X-Socket-ID` header. Then, when you call the `toOthers` method, Laravel will extract the socket ID from the header and instruct the broadcaster to not broadcast to any connections with that socket ID.

Si no está utilizando Vue y Axios, deberá configurar manualmente su aplicación JavaScript para enviar el encabezado `X-Socket-ID`. Puede recuperar el ID del socket utilizando el método `Echo.socketId`:
> > If you are not using Vue and Axios, you will need to manually configure your JavaScript application to send the `X-Socket-ID` header. You may retrieve the socket ID using the `Echo.socketId` method:

    var socketId = Echo.socketId();

<a name="receiving-broadcasts"></a>
## Recepción de transmisiones : Receiving Broadcasts

<a name="installing-laravel-echo"></a>
### Instalación de Laravel Echo : Installing Laravel Echo

Laravel Echo es una biblioteca de JavaScript que facilita la suscripción a canales y escucha eventos transmitidos por Laravel. Puede instalar Echo a través del administrador de paquetes de NPM. En este ejemplo, también instalaremos el paquete `pusher-js` ya que usaremos la emisora ​​Pusher:
> > Laravel Echo is a JavaScript library that makes it painless to subscribe to channels and listen for events broadcast by Laravel. You may install Echo via the NPM package manager. In this example, we will also install the `pusher-js` package since we will be using the Pusher broadcaster:

    npm install --save laravel-echo pusher-js

Una vez que se haya instalado Echo, está listo para crear una nueva instancia de Echo en el JavaScript de su aplicación. Un excelente lugar para hacerlo es en la parte inferior del archivo `resources/assets/js/bootstrap.js` que se incluye con el marco de Laravel:
> > Once Echo is installed, you are ready to create a fresh Echo instance in your application's JavaScript. A great place to do this is at the bottom of the `resources/assets/js/bootstrap.js` file that is included with the Laravel framework:

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key'
    });

Al crear una instancia de Echo que utiliza el conector `pusher`, también puede especificar un `cluster`, así como si la conexión debe cifrarse:
> > When creating an Echo instance that uses the `pusher` connector, you may also specify a `cluster` as well as whether the connection should be encrypted:

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key',
        cluster: 'eu',
        encrypted: true
    });

<a name="listening-for-events"></a>
### Escuchar eventos : Listening For Events

Una vez que haya instalado y ejecutado Echo, estará listo para comenzar a escuchar transmisiones de eventos. Primero, use el método `channel` para recuperar una instancia de un canal, luego llame al método `listen` para escuchar un evento específico:
> > Once you have installed and instantiated Echo, you are ready to start listening for event broadcasts. First, use the `channel` method to retrieve an instance of a channel, then call the `listen` method to listen for a specified event:

    Echo.channel('orders')
        .listen('OrderShipped', (e) => {
            console.log(e.order.name);
        });

Si desea escuchar eventos en un canal privado, use el método `private` en su lugar. Puede continuar las llamadas en cadena al método `listen` para escuchar múltiples eventos en un solo canal:
> > If you would like to listen for events on a private channel, use the `private` method instead. You may continue to chain calls to the `listen` method to listen for multiple events on a single channel:

    Echo.private('orders')
        .listen(...)
        .listen(...)
        .listen(...);

<a name="leaving-a-channel"></a>
### Dejando un canal : Leaving A Channel

Para abandonar un canal, puede llamar al método `leave` en su instancia de Echo:
> > To leave a channel, you may call the `leave` method on your Echo instance:

    Echo.leave('orders');

<a name="namespaces"></a>
### Nombres de espacios : Namespaces

Es posible que haya notado en los ejemplos anteriores que no especificamos el nombre de espacio completo para las clases de eventos. Esto se debe a que Echo asumirá automáticamente que los eventos se encuentran en el espacio de nombres `App\Events`. Sin embargo, puede configurar el nombre de espacio raíz cuando crea una instancia de Echo pasando una opción de configuración `namespace`:
> > You may have noticed in the examples above that we did not specify the full namespace for the event classes. This is because Echo will automatically assume the events are located in the `App\Events` namespace. However, you may configure the root namespace when you instantiate Echo by passing a `namespace` configuration option:

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key',
        namespace: 'App.Other.Namespace'
    });

Alternativamente, puede prefijar las clases de evento con un `.` cuando se suscribe a ellas usando Echo. Esto le permitirá especificar siempre el nombre de clase completo:
> > Alternatively, you may prefix event classes with a `.` when subscribing to them using Echo. This will allow you to always specify the fully-qualified class name:

    Echo.channel('orders')
        .listen('.Namespace.Event.Class', (e) => {
            //
        });

<a name="presence-channels"></a>
## Canales de presencia : Presence Channels

Los canales de presencia se basan en la seguridad de canales privados al tiempo que exponen la característica adicional de conocimiento de quién está suscrito al canal. Esto facilita la creación de características de aplicaciones de colaboración potentes, como notificar a los usuarios cuando otro usuario está viendo la misma página.
> > Presence channels build on the security of private channels while exposing the additional feature of awareness of who is subscribed to the channel. This makes it easy to build powerful, collaborative application features such as notifying users when another user is viewing the same page.

<a name="authorizing-presence-channels"></a>
### Autorización de canales de presencia : Authorizing Presence Channels

Todos los canales de presencia son también canales privados; por lo tanto, los usuarios deben estar [autorizados a acceder a ellos](#authorizing-channels). Sin embargo, al definir devoluciones de llamadas de autorización para canales de presencia, no devolverá `true` si el usuario está autorizado para unirse al canal. En su lugar, debe devolver una serie de datos sobre el usuario.
> > All presence channels are also private channels; therefore, users must be [authorized to access them](#authorizing-channels). However, when defining authorization callbacks for presence channels, you will not return `true` if the user is authorized to join the channel. Instead, you should return an array of data about the user.

Los datos devueltos por la devolución de llamada de autorización estarán disponibles para los oyentes de eventos del canal de presencia en su aplicación de JavaScript. Si el usuario no está autorizado para unirse al canal de presencia, debe devolver `false` o `null`:
> > The data returned by the authorization callback will be made available to the presence channel event listeners in your JavaScript application. If the user is not authorized to join the presence channel, you should return `false` or `null`:

    Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
        if ($user->canJoinRoom($roomId)) {
            return ['id' => $user->id, 'name' => $user->name];
        }
    });

<a name="joining-presence-channels"></a>
### Unir canales de presencia : Joining Presence Channels

Para unirte a un canal de presencia, puedes usar el método `join` de Echo. El método `join` devolverá una implementación `PresenceChannel` que, junto con la exposición del método `listen`, le permite suscribirse a los eventos `here`, `joining`, y `leaving`.
> > To join a presence channel, you may use Echo's `join` method. The `join` method will return a `PresenceChannel` implementation which, along with exposing the `listen` method, allows you to subscribe to the `here`, `joining`, and `leaving` events.

    Echo.join(`chat.${roomId}`)
        .here((users) => {
            //
        })
        .joining((user) => {
            console.log(user.name);
        })
        .leaving((user) => {
            console.log(user.name);
        });

La rellamada `here` se ejecutará inmediatamente una vez que el canal se una satisfactoriamente, y recibirá una matriz que contiene la información del usuario para todos los demás usuarios actualmente suscritos al canal. El método `joining` se ejecutará cuando un nuevo usuario se una a un canal, mientras que el método `leaving` se ejecutará cuando un usuario abandone el canal.
> > The `here` callback will be executed immediately once the channel is joined successfully, and will receive an array containing the user information for all of the other users currently subscribed to the channel. The `joining` method will be executed when a new user joins a channel, while the `leaving` method will be executed when a user leaves the channel.

<a name="broadcasting-to-presence-channels"></a>
### Transmisión a canales de presencia : Broadcasting To Presence Channels

Los canales de presencia pueden recibir eventos al igual que canales públicos o privados. Usando el ejemplo de una sala de chat, es posible que deseemos transmitir eventos `NewMessage` al canal de presencia de la sala. Para hacerlo, devolveremos una instancia de `PresenceChannel` del método `broadcastOn` del evento:
> > Presence channels may receive events just like public or private channels. Using the example of a chatroom, we may want to broadcast `NewMessage` events to the room's presence channel. To do so, we'll return an instance of `PresenceChannel` from the event's `broadcastOn` method:

    /**
     * Get the channels the event should broadcast on.
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PresenceChannel('room.'.$this->message->room_id);
    }

Al igual que los eventos públicos o privados, los eventos del canal de presencia se pueden transmitir usando la función `broadcast`. Al igual que con otros eventos, puede usar el método `toOthers` para excluir que el usuario actual reciba la transmisión:
> > Like public or private events, presence channel events may be broadcast using the `broadcast` function. As with other events, you may use the `toOthers` method to exclude the current user from receiving the broadcast:

    broadcast(new NewMessage($message));

    broadcast(new NewMessage($message))->toOthers();

Puede escuchar el evento join mediante el método de `listen` de Echo:
> > You may listen for the join event via Echo's `listen` method:

    Echo.join(`chat.${roomId}`)
        .here(...)
        .joining(...)
        .leaving(...)
        .listen('NewMessage', (e) => {
            //
        });

<a name="client-events"></a>
## Eventos de clientes : Client Events

> {tip} Al usar [Pusher](https://pusher.com), debe habilitar la opción "Eventos de cliente" en la sección "Configuración de la aplicación" de su [panel de control de la aplicación](https://dashboard.pusher.com/) para enviar eventos del cliente.
> > > {tip} When using [Pusher](https://pusher.com), you must enable the "Client Events" option in the "App Settings" section of your [application dashboard](https://dashboard.pusher.com/) in order to send client events.

A veces puede desear transmitir un evento a otros clientes conectados sin tocar su aplicación Laravel en absoluto. Esto puede ser particularmente útil para cosas como notificaciones de "tipeo", donde desea alertar a los usuarios de su aplicación que otro usuario está escribiendo un mensaje en una pantalla determinada.
> > Sometimes you may wish to broadcast an event to other connected clients without hitting your Laravel application at all. This can be particularly useful for things like "typing" notifications, where you want to alert users of your application that another user is typing a message on a given screen.

Para transmitir eventos del cliente, puedes usar el método `whisper` de Echo:
> > To broadcast client events, you may use Echo's `whisper` method:

    Echo.private('chat')
        .whisper('typing', {
            name: this.user.name
        });

Para escuchar los eventos del cliente, puede usar el método `listenForWhisper`:
> > To listen for client events, you may use the `listenForWhisper` method:

    Echo.private('chat')
        .listenForWhisper('typing', (e) => {
            console.log(e.name);
        });

<a name="notifications"></a>
## Notificaciones : Notifications

Al combinar la transmisión de eventos con [notificaciones](/docs/{{version}}/notifications), su aplicación JavaScript puede recibir nuevas notificaciones a medida que ocurren sin necesidad de actualizar la página. En primer lugar, asegúrese de leer la documentación sobre el uso de [el canal de notificación de difusión](/docs/{{version}}/notifications#broadcast-notifications).
> > By pairing event broadcasting with [notifications](/docs/{{version}}/notifications), your JavaScript application may receive new notifications as they occur without needing to refresh the page. First, be sure to read over the documentation on using [the broadcast notification channel](/docs/{{version}}/notifications#broadcast-notifications).

Una vez que haya configurado una notificación para usar el canal de transmisión, puede escuchar los eventos de transmisión usando el método `notification` de Echo. Recuerde, el nombre del canal debe coincidir con el nombre de la clase de la entidad que recibe las notificaciones:
> > Once you have configured a notification to use the broadcast channel, you may listen for the broadcast events using Echo's `notification` method. Remember, the channel name should match the class name of the entity receiving the notifications:

    Echo.private(`App.User.${userId}`)
        .notification((notification) => {
            console.log(notification.type);
        });

En este ejemplo, todas las notificaciones enviadas a instancias de `App\User` a través del canal `broadcast` serían recibidas por la devolución de llamada. Se incluye una devolución de llamada de autorización de canal para el canal `App.User.{id}` en el `BroadcastServiceProvider` predeterminado que se envía con el marco de Laravel.
> > In this example, all notifications sent to `App\User` instances via the `broadcast` channel would be received by the callback. A channel authorization callback for the `App.User.{id}` channel is included in the default `BroadcastServiceProvider` that ships with the Laravel framework.
