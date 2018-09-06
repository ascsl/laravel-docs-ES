# Notificaciones : Notifications

- [Introduction](#introduction)
- [Creating Notifications](#creating-notifications)
- [Sending Notifications](#sending-notifications)
    - [Using The Notifiable Trait](#using-the-notifiable-trait)
    - [Using The Notification Facade](#using-the-notification-facade)
    - [Specifying Delivery Channels](#specifying-delivery-channels)
    - [Queueing Notifications](#queueing-notifications)
    - [On-Demand Notifications](#on-demand-notifications)
- [Mail Notifications](#mail-notifications)
    - [Formatting Mail Messages](#formatting-mail-messages)
    - [Customizing The Recipient](#customizing-the-recipient)
    - [Customizing The Subject](#customizing-the-subject)
    - [Customizing The Templates](#customizing-the-templates)
- [Markdown Mail Notifications](#markdown-mail-notifications)
    - [Generating The Message](#generating-the-message)
    - [Writing The Message](#writing-the-message)
    - [Customizing The Components](#customizing-the-components)
- [Database Notifications](#database-notifications)
    - [Prerequisites](#database-prerequisites)
    - [Formatting Database Notifications](#formatting-database-notifications)
    - [Accessing The Notifications](#accessing-the-notifications)
    - [Marking Notifications As Read](#marking-notifications-as-read)
- [Broadcast Notifications](#broadcast-notifications)
    - [Prerequisites](#broadcast-prerequisites)
    - [Formatting Broadcast Notifications](#formatting-broadcast-notifications)
    - [Listening For Notifications](#listening-for-notifications)
- [SMS Notifications](#sms-notifications)
    - [Prerequisites](#sms-prerequisites)
    - [Formatting SMS Notifications](#formatting-sms-notifications)
    - [Customizing The "From" Number](#customizing-the-from-number)
    - [Routing SMS Notifications](#routing-sms-notifications)
- [Slack Notifications](#slack-notifications)
    - [Prerequisites](#slack-prerequisites)
    - [Formatting Slack Notifications](#formatting-slack-notifications)
    - [Slack Attachments](#slack-attachments)
    - [Routing Slack Notifications](#routing-slack-notifications)
- [Notification Events](#notification-events)
- [Custom Channels](#custom-channels)

<a name="introduction"></a>
## Introducción : Introduction

Además del soporte para [enviar correo electrónico](/docs/{{version}}/mail), Laravel brinda asistencia para enviar notificaciones a través de una variedad de canales de entrega, incluyendo correo, SMS (a través de [Nexmo](https://www).nexmo.com/)) y [Slack](https://slack.com). Las notificaciones también pueden almacenarse en una base de datos para que se muestren en su interfaz web.
> > In addition to support for [sending email](/docs/{{version}}/mail), Laravel provides support for sending notifications across a variety of delivery channels, including mail, SMS (via [Nexmo](https://www.nexmo.com/)), and [Slack](https://slack.com). Notifications may also be stored in a database so they may be displayed in your web interface.

Por lo general, las notificaciones deben ser breves, mensajes informativos que notifican a los usuarios de algo que ocurrió en su aplicación. Por ejemplo, si está escribiendo una aplicación de facturación, puede enviar una notificación de "Factura pagada" a sus usuarios a través del correo electrónico y los canales de SMS.
> > Typically, notifications should be short, informational messages that notify users of something that occurred in your application. For example, if you are writing a billing application, you might send an "Invoice Paid" notification to your users via the email and SMS channels.

<a name="creating-notifications"></a>
## Creando notificaciones : Creating Notifications

En Laravel, cada notificación está representada por una única clase (normalmente almacenada en el directorio `app/Notifications`). No se preocupe si no ve este directorio en su aplicación, se creará para usted cuando ejecute el comando Artisan `make:notification`:
> > In Laravel, each notification is represented by a single class (typically stored in the `app/Notifications` directory). Don't worry if you don't see this directory in your application, it will be created for you when you run the `make:notification` Artisan command:

    php artisan make:notification InvoicePaid

Este comando colocará una nueva clase de notificación en su directorio `app/Notifications`. Cada clase de notificación contiene un método `via` y una cantidad variable de métodos de creación de mensajes (como `toMail` o `toDatabase`) que convierten la notificación en un mensaje optimizado para ese canal en particular.
> > This command will place a fresh notification class in your `app/Notifications` directory. Each notification class contains a `via` method and a variable number of message building methods (such as `toMail` or `toDatabase`) that convert the notification to a message optimized for that particular channel.

<a name="sending-notifications"></a>
## Envío de notificaciones : Sending Notifications

<a name="using-the-notifiable-trait"></a>
### Usando el rasgo Notifiable : Using The Notifiable Trait

Las notificaciones pueden enviarse de dos maneras: utilizando el método `notify` del rasgo `Notifiable` o usando la [fachada](/docs/{{version}}/facades) `Notification`. Primero, exploremos el uso del rasgo:
> > Notifications may be sent in two ways: using the `notify` method of the `Notifiable` trait or using the `Notification` [facade](/docs/{{version}}/facades). First, let's explore using the trait:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;
    }

Este rasgo es utilizado por el modelo predeterminado `App\User` y contiene un método que puede usarse para enviar notificaciones: `notify`. El método `notify` espera recibir una instancia de notificación:
> > This trait is utilized by the default `App\User` model and contains one method that may be used to send notifications: `notify`. The `notify` method expects to receive a notification instance:

    use App\Notifications\InvoicePaid;

    $user->notify(new InvoicePaid($invoice));

> {tip} Recuerde, puede usar el atributo `Illuminate\Notifications\Notifiable` en cualquiera de sus modelos. No está limitado a solo incluirlo en su modelo `User`.
> > > {tip} Remember, you may use the `Illuminate\Notifications\Notifiable` trait on any of your models. You are not limited to only including it on your `User` model.

<a name="using-the-notification-facade"></a>
### Uso de la fachada Notification : Using The Notification Facade

Alternativamente, puede enviar notificaciones a través de la [fachada](/docs/{{version}}/facades) `Notification`. Esto es útil principalmente cuando necesita enviar una notificación a varias entidades de notificación obligatoria, como una colección de usuarios. Para enviar notificaciones usando la fachada, pase todas las entidades de declaración obligatoria y la instancia de notificación al método `send`:
> > Alternatively, you may send notifications via the `Notification` [facade](/docs/{{version}}/facades). This is useful primarily when you need to send a notification to multiple notifiable entities such as a collection of users. To send notifications using the facade, pass all of the notifiable entities and the notification instance to the `send` method:

    Notification::send($users, new InvoicePaid($invoice));

<a name="specifying-delivery-channels"></a>
### Especificando los canales de entrega : Specifying Delivery Channels

Cada clase de notificación tiene un método `via` que determina en qué canales se entregará la notificación. Fuera de la caja, las notificaciones se pueden enviar en los canales `mail`, `database`, `broadcast`, `nexmo`, y `slack`.
> > Every notification class has a `via` method that determines on which channels the notification will be delivered. Out of the box, notifications may be sent on the `mail`, `database`, `broadcast`, `nexmo`, and `slack` channels.

> {tip} Si desea utilizar otros canales de entrega como Telegram o Pusher, consulte el sitio web [Laravel Notification Channels](http://laravel-notification-channels.com) dirigido por la comunidad.
> > > {tip} If you would like to use other delivery channels such as Telegram or Pusher, check out the community driven [Laravel Notification Channels website](http://laravel-notification-channels.com).

El método `via` recibe una instancia `$notifiable`, que será una instancia de la clase a la que se envía la notificación. Puede usar `$notificable` para determinar en qué canales debe entregarse la notificación:
> > The `via` method receives a `$notifiable` instance, which will be an instance of the class to which the notification is being sent. You may use `$notifiable` to determine which channels the notification should be delivered on:

    /**
     * Get the notification's delivery channels.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return $notifiable->prefers_sms ? ['nexmo'] : ['mail', 'database'];
    }

<a name="queueing-notifications"></a>
### Notificaciones en cola : Queueing Notifications

> {note} Antes de poner en cola las notificaciones, debe configurar su cola y [iniciar un trabajador](/docs/{{version}}/queues).
> > > {note} Before queueing notifications you should configure your queue and [start a worker](/docs/{{version}}/queues).

Enviar notificaciones puede llevar tiempo, especialmente si el canal necesita una llamada API externa para entregar la notificación. Para acelerar el tiempo de respuesta de su aplicación, permita que su notificación sea puesta en cola agregando la interfaz `ShouldQueue` y el rasgo `Queueable` a su clase. La interfaz y el rasgo ya se importaron para todas las notificaciones generadas mediante `make:notification`, por lo que puedes agregarlas inmediatamente a tu clase de notificación:
> > Sending notifications can take time, especially if the channel needs an external API call to deliver the notification. To speed up your application's response time, let your notification be queued by adding the `ShouldQueue` interface and `Queueable` trait to your class. The interface and trait are already imported for all notifications generated using `make:notification`, so you may immediately add them to your notification class:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        // ...
    }

Una vez que se haya agregado la interfaz `ShouldQueue` a su notificación, puede enviar la notificación como de costumbre. Laravel detectará la interfaz `ShouldQueue` en la clase y automáticamente pondrá en cola la entrega de la notificación:
> > Once the `ShouldQueue` interface has been added to your notification, you may send the notification like normal. Laravel will detect the `ShouldQueue` interface on the class and automatically queue the delivery of the notification:

    $user->notify(new InvoicePaid($invoice));

Si desea retrasar la entrega de la notificación, puede encadenar el método `demora` en su instanciación de notificación:
> > If you would like to delay the delivery of the notification, you may chain the `delay` method onto your notification instantiation:

    $when = now()->addMinutes(10);

    $user->notify((new InvoicePaid($invoice))->delay($when));

<a name="on-demand-notifications"></a>
### Notificaciones bajo demanda : On-Demand Notifications

A veces puede necesitar enviar una notificación a alguien que no está almacenado como un "usuario" de su aplicación. Usando el método `Notification::route`, puede especificar información de enrutamiento de notificación ad-hoc antes de enviar la notificación:
> > Sometimes you may need to send a notification to someone who is not stored as a "user" of your application. Using the `Notification::route` method, you may specify ad-hoc notification routing information before sending the notification:

    Notification::route('mail', 'taylor@example.com')
                ->route('nexmo', '5555555555')
                ->notify(new InvoicePaid($invoice));

<a name="mail-notifications"></a>
## Notificaciones de correo : Mail Notifications

<a name="formatting-mail-messages"></a>
### Formateo de mensajes de correo : Formatting Mail Messages

Si una notificación admite que se envíe como un correo electrónico, debe definir un método `toMail` en la clase de notificación. Este método recibirá una entidad `$notifiable` y debería devolver una instancia `Illuminate\Notifications\Messages\MailMessage`. Los mensajes de correo pueden contener líneas de texto, así como una "llamada a la acción". Echemos un vistazo a un ejemplo del método `toMail`:
> > If a notification supports being sent as an email, you should define a `toMail` method on the notification class. This method will receive a `$notifiable` entity and should return a `Illuminate\Notifications\Messages\MailMessage` instance. Mail messages may contain lines of text as well as a "call to action". Let's take a look at an example `toMail` method:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->greeting('Hello!')
                    ->line('One of your invoices has been paid!')
                    ->action('View Invoice', $url)
                    ->line('Thank you for using our application!');
    }

> {tip} Tenga en cuenta que estamos usando `$this->invoice->id` en nuestro método `toMail`. Puede pasar cualquier dato que su notificación necesite para generar su mensaje en el constructor de la notificación.
> > > {tip} Note we are using `$this->invoice->id` in our `toMail` method. You may pass any data your notification needs to generate its message into the notification's constructor.

En este ejemplo, registramos un saludo, una línea de texto, un llamado a la acción y luego otra línea de texto. Estos métodos proporcionados por el objeto `MailMessage` hacen que sea sencillo y rápido formatear pequeños correos electrónicos transaccionales. El canal de correo traducirá los componentes del mensaje en una plantilla de correo electrónico HTML agradable y receptiva con una contraparte de texto sin formato. Aquí hay un ejemplo de un correo electrónico generado por el canal `mail`:
> > In this example, we register a greeting, a line of text, a call to action, and then another line of text. These methods provided by the `MailMessage` object make it simple and fast to format small transactional emails. The mail channel will then translate the message components into a nice, responsive HTML email template with a plain-text counterpart. Here is an example of an email generated by the `mail` channel:

<img src="https://laravel.com/assets/img/notification-example.png" width="551" height="596">

> {tip} Cuando envíe notificaciones por correo, asegúrese de establecer el valor `name` en su archivo de configuración `config/app.php`. Este valor se usará en el encabezado y pie de página de sus mensajes de notificación por correo.
> > > {tip} When sending mail notifications, be sure to set the `name` value in your `config/app.php` configuration file. This value will be used in the header and footer of your mail notification messages.

#### Otras opciones de formato de notificación : Other Notification Formatting Options

En lugar de definir las "líneas" de texto en la clase de notificación, puede usar el método `view` para especificar una plantilla personalizada que se debe usar para representar el correo electrónico de notificación:
> > Instead of defining the "lines" of text in the notification class, you may use the `view` method to specify a custom template that should be used to render the notification email:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)->view(
            'emails.name', ['invoice' => $this->invoice]
        );
    }

Además, puede devolver un [objeto de correo](/docs/{{version}}/mail) desde el método `toMail`:
> > In addition, you may return a [mailable object](/docs/{{version}}/mail) from the `toMail` method:

    use App\Mail\InvoicePaid as Mailable;

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return Mailable
     */
    public function toMail($notifiable)
    {
        return (new Mailable($this->invoice))->to($this->user->email);
    }

<a name="error-messages"></a>
#### Messages de error : Error Messages

Algunas notificaciones informan a los usuarios de errores, como un pago fallido de factura. Puede indicar que un mensaje de correo está relacionado con un error llamando al método `error` al compilar su mensaje. Al usar el método `error` en un mensaje de correo, el botón de llamar a la acción será rojo en lugar de azul:
> > Some notifications inform users of errors, such as a failed invoice payment. You may indicate that a mail message is regarding an error by calling the `error` method when building your message. When using the `error` method on a mail message, the call to action button will be red instead of blue:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Message
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->error()
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-recipient"></a>
### Personalización del destinatario : Customizing The Recipient

Al enviar notificaciones a través del canal `mail`, el sistema de notificación buscará automáticamente una propiedad `email` en su entidad de notificación obligatoria. Puede personalizar qué dirección de correo electrónico se utiliza para entregar la notificación definiendo un método `routeNotificationForMail` en la entidad:
> > When sending notifications via the `mail` channel, the notification system will automatically look for an `email` property on your notifiable entity. You may customize which email address is used to deliver the notification by defining a `routeNotificationForMail` method on the entity:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the mail channel.
         *
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return string
         */
        public function routeNotificationForMail($notification)
        {
            return $this->email_address;
        }
    }

<a name="customizing-the-subject"></a>
### Personalización del sujeto : Customizing The Subject

Por defecto, el asunto del correo electrónico es el nombre de la clase de la notificación formateada para "caso del título". Entonces, si su clase de notificación se llama `InvoicePaid`, el asunto del correo electrónico será `Invoice Paid`. Si desea especificar un tema explícito para el mensaje, puede llamar al método `subject` cuando construya su mensaje:
> > By default, the email's subject is the class name of the notification formatted to "title case". So, if your notification class is named `InvoicePaid`, the email's subject will be `Invoice Paid`. If you would like to specify an explicit subject for the message, you may call the `subject` method when building your message:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-templates"></a>
### Personalizar las plantillas : Customizing The Templates

Puede modificar el HTML y la plantilla de texto sin formato utilizados por las notificaciones de correo publicando los recursos del paquete de notificación. Después de ejecutar este comando, las plantillas de notificación de correo se ubicarán en el directorio `resources/views/vendor/notifications`:
> > You can modify the HTML and plain-text template used by mail notifications by publishing the notification package's resources. After running this command, the mail notification templates will be located in the `resources/views/vendor/notifications` directory:

    php artisan vendor:publish --tag=laravel-notifications

<a name="markdown-mail-notifications"></a>
## Notificaciones de correo Markdown : Markdown Mail Notifications

Las notificaciones de marcado de Markdown le permiten aprovechar las plantillas preconstruidas de notificaciones por correo, a la vez que le da más libertad para escribir mensajes más largos y personalizados. Dado que los mensajes se escriben en Markdown, Laravel puede generar plantillas HTML atractivas y bonitas para los mensajes y generar automáticamente una contraparte de texto sin formato.
> > Markdown mail notifications allow you to take advantage of the pre-built templates of mail notifications, while giving you more freedom to write longer, customized messages. Since the messages are written in Markdown, Laravel is able to render beautiful, responsive HTML templates for the messages while also automatically generating a plain-text counterpart.

<a name="generating-the-message"></a>
### Generando el mensaje : Generating The Message

Para generar una notificación con una plantilla de rebajas correspondiente, puede usar la opción `--markdown` del comando Artisan `make:notification`:
> > To generate a notification with a corresponding Markdown template, you may use the `--markdown` option of the `make:notification` Artisan command:

    php artisan make:notification InvoicePaid --markdown=mail.invoice.paid

Al igual que todas las demás notificaciones de correo, las notificaciones que usan plantillas de Markdown deben definir un método `toMail` en su clase de notificación. Sin embargo, en lugar de usar los métodos `line` y `action` para construir la notificación, use el método `markdown` para especificar el nombre de la plantilla de Markdown que se debe usar:
> > Like all other mail notifications, notifications that use Markdown templates should define a `toMail` method on their notification class. However, instead of using the `line` and `action` methods to construct the notification, use the `markdown` method to specify the name of the Markdown template that should be used:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="writing-the-message"></a>
### Escribir el mensaje : Writing The Message

Las notificaciones de marcado de Markdown utilizan una combinación de componentes Blade y sintaxis Markdown que le permiten construir fácilmente notificaciones mientras aprovecha los componentes de notificación prefabricados de Laravel:
> > Markdown mail notifications use a combination of Blade components and Markdown syntax which allow you to easily construct notifications while leveraging Laravel's pre-crafted notification components:

    @component('mail::message')
    # Invoice Paid

    Your invoice has been paid!

    @component('mail::button', ['url' => $url])
    View Invoice
    @endcomponent

    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent

#### Componente botón : Button Component

El componente de botón representa un enlace de botón centrado. El componente acepta dos argumentos, un `url` y un `color` opcional. Los colores compatibles son `blue`, `green`, y `red`. Puede agregar tantos componentes de botón a una notificación como desee:
> > The button component renders a centered button link. The component accepts two arguments, a `url` and an optional `color`. Supported colors are `blue`, `green`, and `red`. You may add as many button components to a notification as you wish:

    @component('mail::button', ['url' => $url, 'color' => 'green'])
    View Invoice
    @endcomponent

#### Componente panel : Panel Component

El componente panel representa el bloque de texto dado en un panel que tiene un color de fondo ligeramente diferente que el resto de la notificación. Esto le permite llamar la atención sobre un bloque de texto dado:
> > The panel component renders the given block of text in a panel that has a slightly different background color than the rest of the notification. This allows you to draw attention to a given block of text:

    @component('mail::panel')
    This is the panel content.
    @endcomponent

#### Componente tabla : Table Component

El componente tabla le permite transformar una tabla de rebajas en una tabla HTML. El componente acepta la tabla de rebajas como su contenido. La alineación de columna de tabla es compatible con la sintaxis de alineación de tabla de marcación predeterminada:
> > The table component allows you to transform a Markdown table into an HTML table. The component accepts the Markdown table as its content. Table column alignment is supported using the default Markdown table alignment syntax:

    @component('mail::table')
    | Laravel       | Table         | Example  |
    | ------------- |:-------------:| --------:|
    | Col 2 is      | Centered      | $10      |
    | Col 3 is      | Right-Aligned | $20      |
    @endcomponent

<a name="customizing-the-components"></a>
### Personalización de los componentes : Customizing The Components

Puede exportar todos los componentes de notificación de reducción a su propia aplicación para personalización. Para exportar los componentes, use el comando Artisan `vendor:publish` para publicar la etiqueta de elemento `laravel-mail`:
> > You may export all of the Markdown notification components to your own application for customization. To export the components, use the `vendor:publish` Artisan command to publish the `laravel-mail` asset tag:

    php artisan vendor:publish --tag=laravel-mail

Este comando publicará los componentes de correo Markdown en el directorio `resources/views/vendor/mail`. El directorio `mail` contendrá un directorio `html` y `markdown`, cada uno con sus respectivas representaciones de cada componente disponible. Usted es libre de personalizar estos componentes como desee.
> > This command will publish the Markdown mail components to the `resources/views/vendor/mail` directory. The `mail` directory will contain a `html` and a `markdown` directory, each containing their respective representations of every available component. You are free to customize these components however you like.

#### Personalizar el CSS : Customizing The CSS

Después de exportar los componentes, el directorio `resources/views/vendor/mail/html/themes` contendrá un archivo `default.css`. Puede personalizar el CSS en este archivo y sus estilos se alinearán automáticamente dentro de las representaciones HTML de sus notificaciones de reducción.
> > After exporting the components, the `resources/views/vendor/mail/html/themes` directory will contain a `default.css` file. You may customize the CSS in this file and your styles will automatically be in-lined within the HTML representations of your Markdown notifications.

> {tip} Si desea crear un tema completamente nuevo para los componentes Markdown, escriba un nuevo archivo CSS dentro del directorio `html/themes` y cambie la opción `theme` de su archivo de configuración `mail`.
> > > {tip} If you would like to build an entirely new theme for the Markdown components, write a new CSS file within the `html/themes` directory and change the `theme` option of your `mail` configuration file.

<a name="database-notifications"></a>
## Notificaciones de base de datos : Database Notifications

<a name="database-prerequisites"></a>
### Prerrequisitos : Prerequisites

El canal de notificación `database` almacena la información de notificación en una tabla de base de datos. Esta tabla contendrá información, como el tipo de notificación, así como datos JSON personalizados que describen la notificación.
> > The `database` notification channel stores the notification information in a database table. This table will contain information such as the notification type as well as custom JSON data that describes the notification.

Puede consultar la tabla para mostrar las notificaciones en la interfaz de usuario de su aplicación. Pero, antes de que pueda hacer eso, necesitará crear una tabla de base de datos para contener sus notificaciones. Puede usar el comando `notifications:table` para generar una migración con el esquema de tabla apropiado:
> > You can query the table to display the notifications in your application's user interface. But, before you can do that, you will need to create a database table to hold your notifications. You may use the `notifications:table` command to generate a migration with the proper table schema:

    php artisan notifications:table

    php artisan migrate

<a name="formatting-database-notifications"></a>
### Formateando notificaciones de la base de datos : Formatting Database Notifications

Si una notificación admite el almacenamiento en una tabla de base de datos, debe definir un método `toDatabase` o `toArray` en la clase de notificación. Este método recibirá una entidad `$notifiable` y debería devolver un array PHP simple. El array devuelto se codificará como JSON y se almacenará en la columna `data` de su tabla `notifications`. Echemos un vistazo a un ejemplo de método `toArray`:
> > If a notification supports being stored in a database table, you should define a `toDatabase` or `toArray` method on the notification class. This method will receive a `$notifiable` entity and should return a plain PHP array. The returned array will be encoded as JSON and stored in the `data` column of your `notifications` table. Let's take a look at an example `toArray` method:

    /**
     * Get the array representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toArray($notifiable)
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ];
    }

#### `toDatabase` Vs. `toArray`

The `toArray` method is also used by the `broadcast` channel to determine which data to broadcast to your JavaScript client. If you would like to have two different array representations for the `database` and `broadcast` channels, you should define a `toDatabase` method instead of a `toArray` method.

<a name="accessing-the-notifications"></a>
### Acceso a las notificaciones : Accessing The Notifications

Una vez que las notificaciones se almacenan en la base de datos, necesita una forma conveniente de acceder a ellas desde sus entidades de notificación obligatoria. El atributo `Illuminate\Notifications\Notifiable`, que se incluye en el modelo `App\User` predeterminado de Laravel, incluye una relación Eloquent `notifications` que devuelve las notificaciones para la entidad. Para buscar notificaciones, puede acceder a este método como cualquier otra relación Eloquent. Por defecto, las notificaciones se ordenarán por la marca de tiempo `created_at`:
> > Once notifications are stored in the database, you need a convenient way to access them from your notifiable entities. The `Illuminate\Notifications\Notifiable` trait, which is included on Laravel's default `App\User` model, includes a `notifications` Eloquent relationship that returns the notifications for the entity. To fetch notifications, you may access this method like any other Eloquent relationship. By default, notifications will be sorted by the `created_at` timestamp:

    $user = App\User::find(1);

    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }

Si desea recuperar solo las notificaciones "no leídas", puede usar la relación `unreadNotifications`. Nuevamente, estas notificaciones serán ordenadas por la marca de tiempo `created_at`:
> > If you want to retrieve only the "unread" notifications, you may use the `unreadNotifications` relationship. Again, these notifications will be sorted by the `created_at` timestamp:

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }

> {tip} Para acceder a sus notificaciones desde su cliente de JavaScript, debe definir un controlador de notificaciones para su aplicación que devuelve las notificaciones para una entidad de declaración obligatoria, como el usuario actual. A continuación, puede realizar una solicitud HTTP al URI de ese controlador desde su cliente de JavaScript.
> > > {tip} To access your notifications from your JavaScript client, you should define a notification controller for your application which returns the notifications for a notifiable entity, such as the current user. You may then make an HTTP request to that controller's URI from your JavaScript client.

<a name="marking-notifications-as-read"></a>
### Marcado de notificaciones como leídas : Marking Notifications As Read

Normalmente, querrá marcar una notificación como "leer" cuando un usuario la vea. El rasgo `Illuminate\Notifications\Notifiable` proporciona un método `markAsRead`, que actualiza la columna `read_at` en el registro de la base de datos de la notificación:
> > Typically, you will want to mark a notification as "read" when a user views it. The `Illuminate\Notifications\Notifiable` trait provides a `markAsRead` method, which updates the `read_at` column on the notification's database record:

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }

Sin embargo, en lugar de recorrer cada notificación, puede usar el método `markAsRead` directamente en una colección de notificaciones:
> > However, instead of looping through each notification, you may use the `markAsRead` method directly on a collection of notifications:

    $user->unreadNotifications->markAsRead();

También puede usar una consulta de actualización masiva para marcar todas las notificaciones como leídas sin recuperarlas de la base de datos:
> > You may also use a mass-update query to mark all of the notifications as read without retrieving them from the database:

    $user = App\User::find(1);

    $user->unreadNotifications()->update(['read_at' => now()]);

Por supuesto, puede borrar las notificaciones con `delete` para eliminarlas de la tabla por completo:
> > Of course, you may `delete` the notifications to remove them from the table entirely:

    $user->notifications()->delete();

<a name="broadcast-notifications"></a>
## Difusión de notificaciones : Broadcast Notifications

<a name="broadcast-prerequisites"></a>
### Prerrequisitos : Prerequisites

Antes de transmitir notificaciones, debe configurar y familiarizarse con los servicios de [difusión de eventos](/docs/{{version}}/broadcasting) de Laravel. La transmisión de eventos proporciona una forma de reaccionar a los eventos de Laravel disparados desde el lado del servidor desde su cliente de JavaScript.
> > Before broadcasting notifications, you should configure and be familiar with Laravel's [event broadcasting](/docs/{{version}}/broadcasting) services. Event broadcasting provides a way to react to server-side fired Laravel events from your JavaScript client.

<a name="formatting-broadcast-notifications"></a>
### Formatting Broadcast Notifications

El canal `broadcast` difunde notificaciones utilizando los servicios de [difusión de eventos](/docs/{{version}}/broadcasting) de Laravel, permitiendo que su cliente de JavaScript capte las notificaciones en tiempo real. Si una notificación admite la transmisión, debe definir un método `toBroadcast` en la clase de notificación. Este método recibirá una entidad `$notifiable` y debería devolver una instancia `BroadcastMessage`. Los datos devueltos se codificarán como JSON y se transmitirán a su cliente de JavaScript. Echemos un vistazo a un ejemplo del método `toBroadcast`:
> > The `broadcast` channel broadcasts notifications using Laravel's [event broadcasting](/docs/{{version}}/broadcasting) services, allowing your JavaScript client to catch notifications in realtime. If a notification supports broadcasting, you should define a `toBroadcast` method on the notification class. This method will receive a `$notifiable` entity and should return a `BroadcastMessage` instance. The returned data will be encoded as JSON and broadcast to your JavaScript client. Let's take a look at an example `toBroadcast` method:

    use Illuminate\Notifications\Messages\BroadcastMessage;

    /**
     * Get the broadcastable representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return BroadcastMessage
     */
    public function toBroadcast($notifiable)
    {
        return new BroadcastMessage([
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ]);
    }

#### Configuración de cola de difusión : Broadcast Queue Configuration

Todas las notificaciones de difusión están en cola para su transmisión. Si desea configurar la cola o el nombre de cola que se utiliza para poner en cola la operación de difusión, puede usar los métodos `onConnection` y `onQueue` de `BroadcastMessage`:
> > All broadcast notifications are queued for broadcasting. If you would like to configure the queue connection or queue name that is used to the queue the broadcast operation, you may use the `onConnection` and `onQueue` methods of the `BroadcastMessage`:

    return (new BroadcastMessage($data))
                    ->onConnection('sqs')
                    ->onQueue('broadcasts');

> {tip} Además de los datos que especifique, la difusión de notificaciones también contendrán un campo `type` que contiene el nombre de clase de la notificación.
> > > {tip} In addition to the data you specify, broadcast notifications will also contain a `type` field containing the class name of the notification.

<a name="listening-for-notifications"></a>
### Escuchando notificaciones : Listening For Notifications

Las notificaciones se transmitirán en un canal privado formateado utilizando una convención `{notifiable}.{Id}`. Por lo tanto, si está enviando una notificación a una instancia de `App\User` con un ID de `1`, la notificación se emitirá en el canal privado `App.User.1`. Al usar [Laravel Echo](/docs/{{version}}/broadcasting), puede escuchar fácilmente las notificaciones en un canal utilizando el método de ayuda `notification`:
> > Notifications will broadcast on a private channel formatted using a `{notifiable}.{id}` convention. So, if you are sending a notification to a `App\User` instance with an ID of `1`, the notification will be broadcast on the `App.User.1` private channel. When using [Laravel Echo](/docs/{{version}}/broadcasting), you may easily listen for notifications on a channel using the `notification` helper method:

    Echo.private('App.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });

#### Personalización del canal de notificación : Customizing The Notification Channel

Si desea personalizar los canales a los que una entidad notificable recibe sus notificaciones de difusión, puede definir un método `receivesBroadcastNotificationsOn` en la entidad de declaración obligatoria:
> > If you would like to customize which channels a notifiable entity receives its broadcast notifications on, you may define a `receivesBroadcastNotificationsOn` method on the notifiable entity:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * The channels the user receives notification broadcasts on.
         *
         * @return string
         */
        public function receivesBroadcastNotificationsOn()
        {
            return 'users.'.$this->id;
        }
    }

<a name="sms-notifications"></a>
## Notificaciones SMS : SMS Notifications

<a name="sms-prerequisites"></a>
### Prerrequisitos : Prerequisites

El envío de notificaciones por SMS en Laravel está impulsado por [Nexmo](https://www.nexmo.com/). Antes de poder enviar notificaciones a través de Nexmo, debe instalar el paquete Composer `nexmo/client` y agregar algunas opciones de configuración a su archivo de configuración `config/services.php`. Puede copiar la siguiente configuración de ejemplo para comenzar:
> > Sending SMS notifications in Laravel is powered by [Nexmo](https://www.nexmo.com/). Before you can send notifications via Nexmo, you need to install the `nexmo/client` Composer package and add a few configuration options to your `config/services.php` configuration file. You may copy the example configuration below to get started:

    'nexmo' => [
        'key' => env('NEXMO_KEY'),
        'secret' => env('NEXMO_SECRET'),
        'sms_from' => '15556666666',
    ],

La opción `sms_from` es el número de teléfono del que se enviarán sus mensajes SMS. Debe generar un número de teléfono para su aplicación en el panel de control de Nexmo.
> > The `sms_from` option is the phone number that your SMS messages will be sent from. You should generate a phone number for your application in the Nexmo control panel.

<a name="formatting-sms-notifications"></a>
### Formateo de notificaciones SMS : Formatting SMS Notifications

Si una notificación admite que se envíe como un SMS, debe definir un método `toNexmo` en la clase de notificación. Este método recibirá una entidad `$notifiable` y debería devolver una instancia `Illuminate\Notifications\Messages\NexmoMessage`:
> > If a notification supports being sent as an SMS, you should define a `toNexmo` method on the notification class. This method will receive a `$notifiable` entity and should return a `Illuminate\Notifications\Messages\NexmoMessage` instance:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content');
    }

#### Contenido Unicode : Unicode Content

Si su mensaje SMS contendrá caracteres Unicode, debe llamar al método `unicode` cuando construya la instancia `NexmoMessage`:
> > If your SMS message will contain unicode characters, you should call the `unicode` method when constructing the `NexmoMessage` instance:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your unicode message')
                    ->unicode();
    }

<a name="customizing-the-from-number"></a>
### Personalizar el número "Desde" : Customizing The "From" Number

Si desea enviar algunas notificaciones desde un número de teléfono diferente al número de teléfono especificado en su archivo `config/services.php`, puede usar el método `from` en una instancia `NexmoMessage`:
> > If you would like to send some notifications from a phone number that is different from the phone number specified in your `config/services.php` file, you may use the `from` method on a `NexmoMessage` instance:

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content')
                    ->from('15554443333');
    }

<a name="routing-sms-notifications"></a>
### Enrutamiento de notificaciones SMS : Routing SMS Notifications

Al enviar notificaciones a través del canal `nexmo`, el sistema de notificación buscará automáticamente un atributo `phone_number` en la entidad de notificación obligatoria. Si desea personalizar el número de teléfono al que se entrega la notificación, defina un método `routeNotificationForNexmo` en la entidad:
> > When sending notifications via the `nexmo` channel, the notification system will automatically look for a `phone_number` attribute on the notifiable entity. If you would like to customize the phone number the notification is delivered to, define a `routeNotificationForNexmo` method on the entity:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Nexmo channel.
         *
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return string
         */
        public function routeNotificationForNexmo($notification)
        {
            return $this->phone;
        }
    }

<a name="slack-notifications"></a>
## Notificaciones Slack : Slack Notifications

<a name="slack-prerequisites"></a>
### Prerrequisitos : Prerequisites

Antes de poder enviar notificaciones a través de Slack, debe instalar la biblioteca HTTP de Guzzle a través de Composer:
> > Before you can send notifications via Slack, you must install the Guzzle HTTP library via Composer:

    composer require guzzlehttp/guzzle

También deberá configurar una integración ["Webhook entrante"](https://api.slack.com/incoming-webhooks) para su equipo de Slack. Esta integración le proporcionará una URL que puede usar cuando [enruta las notificaciones de Slack](#routing-slack-notifications).
> > You will also need to configure an ["Incoming Webhook"](https://api.slack.com/incoming-webhooks) integration for your Slack team. This integration will provide you with a URL you may use when [routing Slack notifications](#routing-slack-notifications).

<a name="formatting-slack-notifications"></a>
### Formateo de notificaciones Slack : Formatting Slack Notifications

Si una notificación admite que se envíe como mensaje Slack, debe definir un método `toSlack` en la clase de notificación. Este método recibirá una entidad `$notifiable` y debería devolver una instancia `Illuminate\Notifications\Messages\SlackMessage`. Los mensajes Slack pueden contener contenido de texto, así como un "archivo adjunto" que formatea texto adicional o un array de campos. Echemos un vistazo a un ejemplo básico de `toSlack`:
> > If a notification supports being sent as a Slack message, you should define a `toSlack` method on the notification class. This method will receive a `$notifiable` entity and should return a `Illuminate\Notifications\Messages\SlackMessage` instance. Slack messages may contain text content as well as an "attachment" that formats additional text or an array of fields. Let's take a look at a basic `toSlack` example:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->content('One of your invoices has been paid!');
    }

En este ejemplo, solo estamos enviando una sola línea de texto a Slack, que creará un mensaje similar al siguiente:
> > In this example we are just sending a single line of text to Slack, which will create a message that looks like the following:

<img src="https://laravel.com/assets/img/basic-slack-notification.png">

#### Personalización del remitente y el destinatario : Customizing The Sender & Recipient

Puede usar los métodos `from` y `to` para personalizar el remitente y el destinatario. El método `from` acepta un nombre de usuario y un identificador de emoji, mientras que el método `to` acepta un canal o nombre de usuario:
> > You may use the `from` and `to` methods to customize the sender and recipient. The `from` method accepts a username and emoji identifier, while the `to` method accepts a channel or username:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Ghost', ':ghost:')
                    ->to('#other')
                    ->content('This will be sent to #other');
    }

También puede usar una imagen como su logotipo en lugar de un emoji:
> > You may also use an image as your logo instead of an emoji:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Laravel')
                    ->image('https://laravel.com/favicon.png')
                    ->content('This will display the Laravel logo next to the message');
    }

<a name="slack-attachments"></a>
### Adjuntos Slack : Slack Attachments

También puede agregar "archivos adjuntos" a los mensajes de Slack. Los archivos adjuntos ofrecen opciones de formato más completas que los mensajes de texto simples. En este ejemplo, enviaremos una notificación de error sobre una excepción que ocurrió en una aplicación, incluido un enlace para ver más detalles sobre la excepción:
> > You may also add "attachments" to Slack messages. Attachments provide richer formatting options than simple text messages. In this example, we will send an error notification about an exception that occurred in an application, including a link to view more details about the exception:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was not found.');
                    });
    }

El ejemplo anterior generará un mensaje Slack que se parece a lo siguiente:
> > The example above will generate a Slack message that looks like the following:

<img src="https://laravel.com/assets/img/basic-slack-attachment.png">

Los archivos adjuntos también le permiten especificar un array de datos que se deben presentar al usuario. Los datos proporcionados se presentarán en formato de tabla para facilitar la lectura:
> > Attachments also allow you to specify an array of data that should be presented to the user. The given data will be presented in a table-style format for easy reading:

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/invoices/'.$this->invoice->id);

        return (new SlackMessage)
                    ->success()
                    ->content('One of your invoices has been paid!')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Invoice 1322', $url)
                                   ->fields([
                                        'Title' => 'Server Expenses',
                                        'Amount' => '$1,234',
                                        'Via' => 'American Express',
                                        'Was Overdue' => ':-1:',
                                    ]);
                    });
    }

El ejemplo anterior creará un mensaje Slack que se parece a lo siguiente:
> > The example above will create a Slack message that looks like the following:

<img src="https://laravel.com/assets/img/slack-fields-attachment.png">

#### Markdown Attachment Content

Si algunos de sus archivos adjuntos contienen Markdown, puede usar el método `markdown` para indicar a Slack que analice y muestre los campos adjuntos dados como texto con formato Markdown. Los valores aceptados por este método son: `pretext`,` text`, y / o `fields`. Para obtener más información sobre el formato de archivo adjunto Slack, consulte la [documentación de API de Slack](https://api.slack.com/docs/message-formatting#message_formatting):
> > If some of your attachment fields contain Markdown, you may use the `markdown` method to instruct Slack to parse and display the given attachment fields as Markdown formatted text. The values accepted by this method are: `pretext`, `text`, and / or `fields`. For more information about Slack attachment formatting, check out the [Slack API documentation](https://api.slack.com/docs/message-formatting#message_formatting):

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was *not found*.')
                                   ->markdown(['text']);
                    });
    }

<a name="routing-slack-notifications"></a>
### Enrutamiento de notificaciones Slack : Routing Slack Notifications

Para enrutar las notificaciones de Slack a la ubicación correcta, defina un método `routeNotificationForSlack` en su entidad de notificación obligatoria. Esto debería devolver la URL webhook a la que se debe enviar la notificación. Las URL de Webhook pueden generarse al agregar un servicio de "Webhook entrante" a su equipo de Slack:
> > To route Slack notifications to the proper location, define a `routeNotificationForSlack` method on your notifiable entity. This should return the webhook URL to which the notification should be delivered. Webhook URLs may be generated by adding an "Incoming Webhook" service to your Slack team:

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Slack channel.
         *
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return string
         */
        public function routeNotificationForSlack($notification)
        {
            return $this->slack_webhook_url;
        }
    }

<a name="notification-events"></a>
## Eventos de notificación : Notification Events

Cuando se envía una notificación, el sistema de notificaciones dispara el evento `Illuminate\Notifications\Events\NotificationSent`. Este contiene la entidad "notificable" y la instancia de notificación en sí misma. Puede registrar oyentes para este evento en su `EventServiceProvider`:
> > When a notification is sent, the `Illuminate\Notifications\Events\NotificationSent` event is fired by the notification system. This contains the "notifiable" entity and the notification instance itself. You may register listeners for this event in your `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Notifications\Events\NotificationSent' => [
            'App\Listeners\LogNotification',
        ],
    ];

> {tip} Después de registrar oyentes en su `EventServiceProvider`, use el comando Artisan `event:generate` para generar rápidamente clases de escucha.
> > > {tip} After registering listeners in your `EventServiceProvider`, use the `event:generate` Artisan command to quickly generate listener classes.

Dentro de un detector de eventos, puede acceder a las propiedades `notifiable`, `notification`, y `channel` en el evento para obtener más información sobre el destinatario de la notificación o la notificación en sí:
> > Within an event listener, you may access the `notifiable`, `notification`, and `channel` properties on the event to learn more about the notification recipient or the notification itself:

    /**
     * Handle the event.
     *
     * @param  NotificationSent  $event
     * @return void
     */
    public function handle(NotificationSent $event)
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
    }

<a name="custom-channels"></a>
## Canales personalizados : Custom Channels

Laravel incluye un puñado de canales de notificación, pero es posible que desee escribir sus propios controladores para enviar notificaciones a través de otros canales. Laravel lo hace simple. Para comenzar, defina una clase que contenga un método `send`. El método debe recibir dos argumentos: `$notifiable` y `$notification`:
> > Laravel ships with a handful of notification channels, but you may want to write your own drivers to deliver notifications via other channels. Laravel makes it simple. To get started, define a class that contains a `send` method. The method should receive two arguments: a `$notifiable` and a `$notification`:

    <?php

    namespace App\Channels;

    use Illuminate\Notifications\Notification;

    class VoiceChannel
    {
        /**
         * Send the given notification.
         *
         * @param  mixed  $notifiable
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return void
         */
        public function send($notifiable, Notification $notification)
        {
            $message = $notification->toVoice($notifiable);

            // Send notification to the $notifiable instance...
        }
    }

Una vez que se haya definido su clase de canal de notificación, puede devolver el nombre de clase del método `via` de cualquiera de sus notificaciones:
> > Once your notification channel class has been defined, you may return the class name from the `via` method of any of your notifications:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use App\Channels\VoiceChannel;
    use App\Channels\Messages\VoiceMessage;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class InvoicePaid extends Notification
    {
        use Queueable;

        /**
         * Get the notification channels.
         *
         * @param  mixed  $notifiable
         * @return array|string
         */
        public function via($notifiable)
        {
            return [VoiceChannel::class];
        }

        /**
         * Get the voice representation of the notification.
         *
         * @param  mixed  $notifiable
         * @return VoiceMessage
         */
        public function toVoice($notifiable)
        {
            // ...
        }
    }
