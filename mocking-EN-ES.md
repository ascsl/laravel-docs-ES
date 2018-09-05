# Simulación : Mocking

- [Introduction](#introduction)
- [Bus Fake](#bus-fake)
- [Event Fake](#event-fake)
    - [Scoped Event Fakes](#scoped-event-fakes)
- [Mail Fake](#mail-fake)
- [Notification Fake](#notification-fake)
- [Queue Fake](#queue-fake)
- [Storage Fake](#storage-fake)
- [Facades](#mocking-facades)

<a name="introduction"></a>
## Introducción : Introduction

Al probar las aplicaciones de Laravel, es posible que desee "simular" ciertos aspectos de su aplicación para que no se ejecuten realmente durante una prueba determinada. Por ejemplo, cuando se prueba un controlador que distribuye un evento, es posible que desee simular los detectores de eventos para que no se ejecuten realmente durante la prueba. Esto le permite probar únicamente la respuesta HTTP del controlador sin preocuparse por la ejecución de los detectores de eventos, ya que los detectores de eventos se pueden probar en su propio caso de prueba.
> > When testing Laravel applications, you may wish to "mock" certain aspects of your application so they are not actually executed during a given test. For example, when testing a controller that dispatches an event, you may wish to mock the event listeners so they are not actually executed during the test. This allows you to only test the controller's HTTP response without worrying about the execution of the event listeners, since the event listeners can be tested in their own test case.

Laravel proporciona ayudantes para burlarse de eventos, trabajos y fachadas listos para usar. Estos ayudantes proporcionan principalmente una capa de conveniencia sobre Mockery para que no tengas que realizar manualmente llamadas de método de Mockery complicadas. Por supuesto, puede usar [Mockery](http://docs.mockery.io/en/latest/) o PHPUnit para crear sus propios simulacros o espías.
> > Laravel provides helpers for mocking events, jobs, and facades out of the box. These helpers primarily provide a convenience layer over Mockery so you do not have to manually make complicated Mockery method calls. Of course, you are free to use [Mockery](http://docs.mockery.io/en/latest/) or PHPUnit to create your own mocks or spies.

<a name="bus-fake"></a>
## Bus falso : Bus Fake

Como alternativa a la simulación, puede usar el método `fake` de la fachada `Bus` para evitar que se despachen trabajos. Cuando se utilizan falsificaciones, las afirmaciones se realizan después de que se ejecuta el código bajo prueba:
> > As an alternative to mocking, you may use the `Bus` facade's `fake` method to prevent jobs from being dispatched. When using fakes, assertions are made after the code under test is executed:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Jobs\ShipOrder;
    use Illuminate\Support\Facades\Bus;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Bus::fake();

            // Perform order shipping...

            Bus::assertDispatched(ShipOrder::class, function ($job) use ($order) {
                return $job->order->id === $order->id;
            });

            // Assert a job was not dispatched...
            Bus::assertNotDispatched(AnotherJob::class);
        }
    }

<a name="event-fake"></a>
## Evento falso : Event Fake

Como alternativa a la simulación, puede usar el método `fake` de la fachada `Event` para evitar que se ejecuten todos los oyentes de eventos. A continuación, puede afirmar que los eventos fueron enviados e incluso inspeccionar los datos que recibieron. Cuando se utilizan falsificaciones, las afirmaciones se realizan después de que se ejecuta el código bajo prueba:
> > As an alternative to mocking, you may use the `Event` facade's `fake` method to prevent all event listeners from executing. You may then assert that events were dispatched and even inspect the data they received. When using fakes, assertions are made after the code under test is executed:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Events\OrderShipped;
    use App\Events\OrderFailedToShip;
    use Illuminate\Support\Facades\Event;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        /**
         * Test order shipping.
         */
        public function testOrderShipping()
        {
            Event::fake();

            // Perform order shipping...

            Event::assertDispatched(OrderShipped::class, function ($e) use ($order) {
                return $e->order->id === $order->id;
            });

            // Assert an event was dispatched twice...
            Event::assertDispatched(OrderShipped::class, 2);

            // Assert an event was not dispatched...
            Event::assertNotDispatched(OrderFailedToShip::class);
        }
    }

> {note} Después de llamar a `Event::fake()`, no se ejecutarán escuchas de eventos. Entonces, si sus pruebas usan fábricas modelo que dependen de eventos, como la creación de un UUID durante el evento `creating` de un modelo, debe llamar a `Event::fake()` **después de** usar sus fábricas.
> > > {note} After calling `Event::fake()`, no event listeners will be executed. So, if your tests use model factories that rely on events, such as creating a UUID during a model's `creating` event, you should call `Event::fake()` **after** using your factories.

<a name="scoped-event-fakes"></a>
### Scoped Event Fakes

Si solo quiere falsificar oyentes de eventos para una parte de su prueba, puede usar el método `fakeFor`:
> > If you only want to fake event listeners for a portion of your test, you may use the `fakeFor` method:

    <?php

    namespace Tests\Feature;

    use App\Order;
    use Tests\TestCase;
    use App\Events\OrderCreated;
    use Illuminate\Support\Facades\Event;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        /**
         * Test order process.
         */
        public function testOrderProcess()
        {
            $order = Event::fakeFor(function () {
                $order = factory(Order::class)->create();

                Event::assertDispatched(OrderCreated::class);

                return $order;
            });

            // Events are dispatched as normal and observers will run ...
            $order->update([...]);
        }
    }

<a name="mail-fake"></a>
## Correo falso : Mail Fake

Puede usar el método `fake` de la fachada `Mail` para evitar que se envíe el correo. A continuación, puede afirmar que [mailables](/docs/{{version}}/mail) se enviaron a los usuarios e incluso inspeccionar los datos que recibieron. Cuando se utilizan falsificaciones, las afirmaciones se realizan después de que se ejecuta el código bajo prueba:
> > You may use the `Mail` facade's `fake` method to prevent mail from being sent. You may then assert that [mailables](/docs/{{version}}/mail) were sent to users and even inspect the data they received. When using fakes, assertions are made after the code under test is executed:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Mail\OrderShipped;
    use Illuminate\Support\Facades\Mail;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Mail::fake();

            // Perform order shipping...

            Mail::assertSent(OrderShipped::class, function ($mail) use ($order) {
                return $mail->order->id === $order->id;
            });

            // Assert a message was sent to the given users...
            Mail::assertSent(OrderShipped::class, function ($mail) use ($user) {
                return $mail->hasTo($user->email) &&
                       $mail->hasCc('...') &&
                       $mail->hasBcc('...');
            });

            // Assert a mailable was sent twice...
            Mail::assertSent(OrderShipped::class, 2);

            // Assert a mailable was not sent...
            Mail::assertNotSent(AnotherMailable::class);
        }
    }

Si está poniendo en cola correos para la entrega en segundo plano, debe usar el método `assertQueued` en lugar de `assertSent`:
> > If you are queueing mailables for delivery in the background, you should use the `assertQueued` method instead of `assertSent`:

    Mail::assertQueued(...);
    Mail::assertNotQueued(...);

<a name="notification-fake"></a>
## Notificación falsa
## Notification Fake

Puede usar el método `fake` de la fachada `Notification` para evitar que se envíen notificaciones. A continuación, puede afirmar que [notificaciones](/docs/{{version}}/notifications) se enviaron a los usuarios e incluso inspeccionar los datos que recibieron. Cuando se utilizan falsificaciones, las afirmaciones se realizan después de que se ejecuta el código bajo prueba:
> > You may use the `Notification` facade's `fake` method to prevent notifications from being sent. You may then assert that [notifications](/docs/{{version}}/notifications) were sent to users and even inspect the data they received. When using fakes, assertions are made after the code under test is executed:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Notifications\OrderShipped;
    use Illuminate\Support\Facades\Notification;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Notification::fake();

            // Perform order shipping...

            Notification::assertSentTo(
                $user,
                OrderShipped::class,
                function ($notification, $channels) use ($order) {
                    return $notification->order->id === $order->id;
                }
            );

            // Assert a notification was sent to the given users...
            Notification::assertSentTo(
                [$user], OrderShipped::class
            );

            // Assert a notification was not sent...
            Notification::assertNotSentTo(
                [$user], AnotherNotification::class
            );
        }
    }

<a name="queue-fake"></a>
## Cola falsa : Queue Fake

Como alternativa a la simulación, puede usar el método `fake` de la fachada `Queue` para evitar que se pongan en cola los trabajos. A continuación, puede afirmar que los trabajos se enviaron a la cola e incluso inspeccionar los datos que recibieron. Cuando se utilizan falsificaciones, las afirmaciones se realizan después de que se ejecuta el código bajo prueba:
> > As an alternative to mocking, you may use the `Queue` facade's `fake` method to prevent jobs from being queued. You may then assert that jobs were pushed to the queue and even inspect the data they received. When using fakes, assertions are made after the code under test is executed:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use App\Jobs\ShipOrder;
    use Illuminate\Support\Facades\Queue;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            Queue::fake();

            // Perform order shipping...

            Queue::assertPushed(ShipOrder::class, function ($job) use ($order) {
                return $job->order->id === $order->id;
            });

            // Assert a job was pushed to a given queue...
            Queue::assertPushedOn('queue-name', ShipOrder::class);

            // Assert a job was pushed twice...
            Queue::assertPushed(ShipOrder::class, 2);

            // Assert a job was not pushed...
            Queue::assertNotPushed(AnotherJob::class);
        }
    }

<a name="storage-fake"></a>
## Almacenamiento falso : Storage Fake

El método `fake` de la fachada `Storage` le permite generar fácilmente un disco falso que, combinado con las utilidades de generación de archivos de la clase `UploadedFile`, simplifica enormemente la prueba de las cargas de archivos. Por ejemplo:
> > The `Storage` facade's `fake` method allows you to easily generate a fake disk that, combined with the file generation utilities of the `UploadedFile` class, greatly simplifies the testing of file uploads. For example:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Http\UploadedFile;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class ExampleTest extends TestCase
    {
        public function testAvatarUpload()
        {
            Storage::fake('avatars');

            $response = $this->json('POST', '/avatar', [
                'avatar' => UploadedFile::fake()->image('avatar.jpg')
            ]);

            // Assert the file was stored...
            Storage::disk('avatars')->assertExists('avatar.jpg');

            // Assert a file does not exist...
            Storage::disk('avatars')->assertMissing('missing.jpg');
        }
    }

> {tip} By default, the `fake` method will delete all files in its temporary directory. If you would like to keep these files, you may use the "persistentFake" method instead.

<a name="mocking-facades"></a>
## Fachadas : Facades

A diferencia de las llamadas a métodos estáticos tradicionales, [fachadas](/docs/{{version}}/facades) se pueden simular. Esto proporciona una gran ventaja sobre los métodos estáticos tradicionales y le otorga la misma capacidad de prueba que tendría si estuviera usando la inyección de dependencia. Al probar, a menudo puede querer burlarse de una llamada a una fachada de Laravel en uno de sus controladores. Por ejemplo, considere la siguiente acción de controlador:
> > Unlike traditional static method calls, [facades](/docs/{{version}}/facades) may be mocked. This provides a great advantage over traditional static methods and grants you the same testability you would have if you were using dependency injection. When testing, you may often want to mock a call to a Laravel facade in one of your controllers. For example, consider the following controller action:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

Podemos simular la llamada a la fachada `Cache` utilizando el método` shouldReceive`, que devolverá una instancia de un simulacro [Mockery](https://github.com/padraic/mockery). Dado que las fachadas en realidad se resuelven y administran por medio de Laravel [contenedor de servicios](/docs/{{version}}/container), tienen mucha más capacidad de prueba que una clase estática típica. Por ejemplo, simulamos nuestra llamada al método `get` de la fachada `Cache`:
> > We can mock the call to the `Cache` facade by using the `shouldReceive` method, which will return an instance of a [Mockery](https://github.com/padraic/mockery) mock. Since facades are actually resolved and managed by the Laravel [service container](/docs/{{version}}/container), they have much more testability than a typical static class. For example, let's mock our call to the `Cache` facade's `get` method:

    <?php

    namespace Tests\Feature;

    use Tests\TestCase;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;

    class UserControllerTest extends TestCase
    {
        public function testGetIndex()
        {
            Cache::shouldReceive('get')
                        ->once()
                        ->with('key')
                        ->andReturn('value');

            $response = $this->get('/users');

            // ...
        }
    }

> {note} No deberías simular la fachada `Request`. En su lugar, pase la entrada que desee a los métodos de ayuda de HTTP como `get` y `post` al ejecutar su prueba. Del mismo modo, en lugar de simular la fachada `Config`, llame al método `Config::set` en sus pruebas.
> > > {note} You should not mock the `Request` facade. Instead, pass the input you desire into the HTTP helper methods such as `get` and `post` when running your test. Likewise, instead of mocking the `Config` facade, call the `Config::set` method in your tests.
