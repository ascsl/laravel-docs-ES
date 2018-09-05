# Mail

- [Introduction](#introduction)
    - [Driver Prerequisites](#driver-prerequisites)
- [Generating Mailables](#generating-mailables)
- [Writing Mailables](#writing-mailables)
    - [Configuring The Sender](#configuring-the-sender)
    - [Configuring The View](#configuring-the-view)
    - [View Data](#view-data)
    - [Attachments](#attachments)
    - [Inline Attachments](#inline-attachments)
    - [Customizing The SwiftMailer Message](#customizing-the-swiftmailer-message)
- [Markdown Mailables](#markdown-mailables)
    - [Generating Markdown Mailables](#generating-markdown-mailables)
    - [Writing Markdown Messages](#writing-markdown-messages)
    - [Customizing The Components](#customizing-the-components)
- [Sending Mail](#sending-mail)
    - [Queueing Mail](#queueing-mail)
- [Rendering Mailables](#rendering-mailables)
    - [Previewing Mailables In The Browser](#previewing-mailables-in-the-browser)
- [Mail & Local Development](#mail-and-local-development)
- [Events](#events)

<a name="introduction"></a>
## Introducción : Introduction

Laravel proporciona una API limpia y simple sobre la popular biblioteca [SwiftMailer](https://swiftmailer.symfony.com/) con controladores para SMTP, Mailgun, SparkPost, Amazon SES, la función `mail` de PHP y `sendmail`, lo que permite comenzar rápidamente a enviar correos a través de un servicio local o en la nube de su elección.
> > Laravel provides a clean, simple API over the popular [SwiftMailer](https://swiftmailer.symfony.com/) library with drivers for SMTP, Mailgun, SparkPost, Amazon SES, PHP's `mail` function, and `sendmail`, allowing you to quickly get started sending mail through a local or cloud based service of your choice.

<a name="driver-prerequisites"></a>
### Prerrequisitos del driver : Driver Prerequisites

Los controladores basados ​​en API como Mailgun y SparkPost suelen ser más simples y rápidos que los servidores SMTP. Si es posible, debe usar uno de estos controladores. Todos los controladores API requieren la biblioteca Guzzle HTTP, que puede instalarse a través del gestor de paquetes Composer:
> > The API based drivers such as Mailgun and SparkPost are often simpler and faster than SMTP servers. If possible, you should use one of these drivers. All of the API drivers require the Guzzle HTTP library, which may be installed via the Composer package manager:

    composer require guzzlehttp/guzzle

#### Driver para Mailgun : Mailgun Driver

Para usar el driver Mailgun, primero instale Guzzle, luego configure la opción `driver` en su archivo de configuración `config/mail.php` en `mailgun`. Luego, verifique que su archivo de configuración `config/services.php` contiene las siguientes opciones:
> > To use the Mailgun driver, first install Guzzle, then set the `driver` option in your `config/mail.php` configuration file to `mailgun`. Next, verify that your `config/services.php` configuration file contains the following options:

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
    ],

#### Driver para SparkPost : SparkPost Driver

Para usar el controlador SparkPost, primero instale Guzzle, luego configure la opción `driver` en su archivo de configuración `config/mail.php` en `sparkpost`. Luego, verifique que su archivo de configuración `config/services.php` contiene las siguientes opciones:
> > To use the SparkPost driver, first install Guzzle, then set the `driver` option in your `config/mail.php` configuration file to `sparkpost`. Next, verify that your `config/services.php` configuration file contains the following options:

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
    ],

Si es necesario, también puede configurar qué [punto final API](https://developers.sparkpost.com/api/#header-endpoints) se debe utilizar:
> > If necessary, you may also configure which [API endpoint](https://developers.sparkpost.com/api/#header-endpoints) should be used:

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
        'options' => [
            'endpoint' => 'https://api.eu.sparkpost.com/api/v1/transmissions',
        ],
    ],

#### Driver Amazon SES : SES Driver

Para usar el controlador de Amazon SES, primero debe instalar Amazon AWS SDK para PHP. Puede instalar esta libreria agregando la siguiente línea a la sección `require` del archivo `composer.json` y ejecutando el comando `composer update`:
> > To use the Amazon SES driver you must first install the Amazon AWS SDK for PHP. You may install this library by adding the following line to your `composer.json` file's `require` section and running the `composer update` command:

    "aws/aws-sdk-php": "~3.0"

A continuación, configure la opción `driver` en su archivo de configuración `config/mail.php` en `ses` y verifique que su archivo de configuración `config/services.php` contiene las siguientes opciones:
> > Next, set the `driver` option in your `config/mail.php` configuration file to `ses` and verify that your `config/services.php` configuration file contains the following options:

    'ses' => [
        'key' => 'your-ses-key',
        'secret' => 'your-ses-secret',
        'region' => 'ses-region',  // e.g. us-east-1
    ],

<a name="generating-mailables"></a>
## Generando Mailables : Generating Mailables

En Laravel, cada tipo de correo electrónico enviado por su aplicación se representa como una clase "mailable". Estas clases se almacenan en el directorio `app/Mail`. No se preocupe si no ve este directorio en su aplicación, ya que se generará para usted cuando cree su primera clase mailable usando el comando `make:mail`:
> > In Laravel, each type of email sent by your application is represented as a "mailable" class. These classes are stored in the `app/Mail` directory. Don't worry if you don't see this directory in your application, since it will be generated for you when you create your first mailable class using the `make:mail` command:

    php artisan make:mail OrderShipped

<a name="writing-mailables"></a>
## Writing Mailables

Toda la configuración de una clase mailable se realiza en el método `build`. Dentro de este método, puede llamar a varios métodos como `from`,` subject`, `view` y `attach` para configurar la presentación y entrega del correo electrónico.
> > All of a mailable class' configuration is done in the `build` method. Within this method, you may call various methods such as `from`, `subject`, `view`, and `attach` to configure the email's presentation and delivery.

<a name="configuring-the-sender"></a>
### Configuración del remitente : Configuring The Sender

#### Usando el método `from` : Using The `from` Method

Primero, exploremos la configuración del remitente del correo electrónico. O, en otras palabras, quién será el correo electrónico "de". Hay dos formas de configurar el remitente. En primer lugar, puede usar el método `from` dentro de su método `build` de la clase mailable:
> > First, let's explore configuring the sender of the email. Or, in other words, who the email is going to be "from". There are two ways to configure the sender. First, you may use the `from` method within your mailable class' `build` method:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->view('emails.orders.shipped');
    }

#### Uso de una dirección global `from` : Using A Global `from` Address

Sin embargo, si su aplicación utiliza la misma dirección "de" para todos sus correos electrónicos, puede resultar engorroso llamar al método `from` en cada clase mailable que genere. En su lugar, puede especificar una dirección global "de" en su archivo de configuración `config/mail.php`. Esta dirección se usará si no se especifica ninguna otra dirección "desde" dentro de la clase mailable:
> > However, if your application uses the same "from" address for all of its emails, it can become cumbersome to call the `from` method in each mailable class you generate. Instead, you may specify a global "from" address in your `config/mail.php` configuration file. This address will be used if no other "from" address is specified within the mailable class:

    'from' => ['address' => 'example@example.com', 'name' => 'App Name'],

<a name="configuring-the-view"></a>
### Configuración de la vista : Configuring The View

Dentro de un método `build` de la clase mailable, puede usar el método `view` para especificar qué plantilla se debe usar al representar los contenidos del correo electrónico. Dado que cada correo electrónico generalmente usa una [Plantilla Blade](/docs/{{version}}/blade) para representar sus contenidos, usted tiene toda la potencia y conveniencia del motor de plantillas Blade al construir el HTML de su correo electrónico:
> > Within a mailable class' `build` method, you may use the `view` method to specify which template should be used when rendering the email's contents. Since each email typically uses a [Blade template](/docs/{{version}}/blade) to render its contents, you have the full power and convenience of the Blade templating engine when building your email's HTML:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped');
    }

> {tip} Es posible que desee crear un directorio `resources/views/emails` para albergar todas sus plantillas de correo electrónico; sin embargo, puede colocarlos donde desee en su directorio `resources/views`.
> > > {tip} You may wish to create a `resources/views/emails` directory to house all of your email templates; however, you are free to place them wherever you wish within your `resources/views` directory.

#### Correos electrónicos de texto sin formato : Plain Text Emails

Si desea definir una versión de texto plano de su correo electrónico, puede usar el método `text`. Al igual que el método `view`, el método `text` acepta un nombre de plantilla que se usará para representar el contenido del correo electrónico. Usted es libre de definir una versión HTML y de texto sin formato de su mensaje:
> > If you would like to define a plain-text version of your email, you may use the `text` method. Like the `view` method, the `text` method accepts a template name which will be used to render the contents of the email. You are free to define both a HTML and plain-text version of your message:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->text('emails.orders.shipped_plain');
    }

<a name="view-data"></a>
### View Data

#### A través de las propiedades públicas : Via Public Properties

Por lo general, querrás pasar algunos datos a tu vista que puedes utilizar al representar el HTML del correo electrónico. Hay dos maneras de hacer que los datos estén disponibles para su vista. Primero, cualquier propiedad pública definida en su clase mailable se pondrá automáticamente a disposición de la vista. Así, por ejemplo, puede pasar datos al constructor de su clase mailable y establecer esos datos a propiedades públicas definidas en la clase:
> > Typically, you will want to pass some data to your view that you can utilize when rendering the email's HTML. There are two ways you may make data available to your view. First, any public property defined on your mailable class will automatically be made available to the view. So, for example, you may pass data into your mailable class' constructor and set that data to public properties defined on the class:

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var Order
         */
        public $order;

        /**
         * Create a new message instance.
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped');
        }
    }

Una vez que los datos se han asignado a una propiedad pública, estarán disponibles en la vista, por lo que puede acceder a ella como si tuviera acceso a cualquier otro dato en sus plantillas Blade:
> > Once the data has been set to a public property, it will automatically be available in your view, so you may access it like you would access any other data in your Blade templates:

    <div>
        Price: {{ $order->price }}
    </div>

#### A través del método `with` : Via The `with` Method

Si desea personalizar el formato de los datos de su correo electrónico antes de enviarlo a la plantilla, puede pasar manualmente sus datos a la vista mediante el método `with`. Por lo general, aún pasarás datos a través del constructor de la clase mailable; sin embargo, debe asignar estos datos en propiedades `protected` o `private` para que los datos no estén disponibles para la plantilla. Luego, cuando llame al método `with`, pase un array de datos que desee poner a disposición de la plantilla:
> > If you would like to customize the format of your email's data before it is sent to the template, you may manually pass your data to the view via the `with` method. Typically, you will still pass data via the mailable class' constructor; however, you should set this data to `protected` or `private` properties so the data is not automatically made available to the template. Then, when calling the `with` method, pass an array of data that you wish to make available to the template:

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var Order
         */
        protected $order;

        /**
         * Create a new message instance.
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->with([
                            'orderName' => $this->order->name,
                            'orderPrice' => $this->order->price,
                        ]);
        }
    }

Una vez que los datos se han pasado al método `with`, estarán disponibles en la vista, por lo que puede acceder a ellos como lo haría con cualquier otro dato en sus plantillas Blade:
> > Once the data has been passed to the `with` method, it will automatically be available in your view, so you may access it like you would access any other data in your Blade templates:

    <div>
        Price: {{ $orderPrice }}
    </div>

<a name="attachments"></a>
### Archivos adjuntos : Attachments

Para agregar archivos adjuntos a un correo electrónico, use el método `attach` dentro del método `build` de la clase mailable. El método `attach` acepta la ruta completa al archivo como primer argumento:
> > To add attachments to an email, use the `attach` method within the mailable class' `build` method. The `attach` method accepts the full path to the file as its first argument:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file');
        }

Al adjuntar archivos a un mensaje, también puede especificar el nombre para mostrar y / o el tipo MIME pasando una `array` como segundo argumento al método `attach`:
> > When attaching files to a message, you may also specify the display name and / or MIME type by passing an `array` as the second argument to the `attach` method:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file', [
                            'as' => 'name.pdf',
                            'mime' => 'application/pdf',
                        ]);
        }

#### Datos adjuntos brutos : Raw Data Attachments

El método `attachData` se puede usar para adjuntar una cadena de bytes sin formato como archivo adjunto. Por ejemplo, puede usar este método si ha generado un PDF en la memoria y desea adjuntarlo al correo electrónico sin escribirlo en el disco. El método `attachData` acepta los bytes de datos brutos como primer argumento, el nombre del archivo como segundo argumento y un array de opciones como tercer argumento:
> > The `attachData` method may be used to attach a raw string of bytes as an attachment. For example, you might use this method if you have generated a PDF in memory and want to attach it to the email without writing it to disk. The `attachData` method accepts the raw data bytes as its first argument, the name of the file as its second argument, and an array of options as its third argument:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attachData($this->pdf, 'name.pdf', [
                            'mime' => 'application/pdf',
                        ]);
        }

<a name="inline-attachments"></a>
### Adjuntos en línea : Inline Attachments

La incrustación de imágenes en línea en sus correos electrónicos suele ser engorrosa; sin embargo, Laravel proporciona una forma de adjuntar imágenes a sus correos electrónicos y recuperar el CID apropiado. Para incrustar una imagen en línea, use el método `embed` en la variable `$message` dentro de su plantilla de correo electrónico. Laravel automáticamente hace que la variable `$message` esté disponible para todas sus plantillas de correo electrónico, por lo que no tiene que preocuparse por pasarla manualmente:
> > Embedding inline images into your emails is typically cumbersome; however, Laravel provides a convenient way to attach images to your emails and retrieving the appropriate CID. To embed an inline image, use the `embed` method on the `$message` variable within your email template. Laravel automatically makes the `$message` variable available to all of your email templates, so you don't need to worry about passing it in manually:

    <body>
        Here is an image:

        <img src="{{ $message->embed($pathToFile) }}">
    </body>

> {note} La variable `$message` no está disponible en los mensajes markdown.
> > > {note} `$message` variable is not available in markdown messages.

#### Integración de datos adjuntos brutos : Embedding Raw Data Attachments

Si ya tiene una cadena de datos brutos que desea incorporar a una plantilla de correo electrónico, puede usar el método `embedData` en la variable `$message`:
> > If you already have a raw data string you wish to embed into an email template, you may use the `embedData` method on the `$message` variable:

    <body>
        Here is an image from raw data:

        <img src="{{ $message->embedData($data, $name) }}">
    </body>

<a name="customizing-the-swiftmailer-message"></a>
### Personalización del mensaje de SwiftMailer : Customizing The SwiftMailer Message

El método `withSwiftMessage` de la clase base `Mailable` le permite registrar una rellamada que se invocará con la instancia de mensaje de SwiftMailer sin procesar antes de enviar el mensaje. Esto le da la oportunidad de personalizar el mensaje antes de que se entregue:
> > The `withSwiftMessage` method of the `Mailable` base class allows you to register a callback which will be invoked with the raw SwiftMailer message instance before sending the message. This gives you an opportunity to customize the message before it is delivered:

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            $this->view('emails.orders.shipped');

            $this->withSwiftMessage(function ($message) {
                $message->getHeaders()
                        ->addTextHeader('Custom-Header', 'HeaderValue');
            });
        }

<a name="markdown-mailables"></a>
## Markdown Mailables

Los mensajes Markdown mailables le permiten aprovechar las plantillas y los componentes precompilados de las notificaciones por correo en sus documentos. Dado que los mensajes se escriben en Markdown, Laravel puede generar plantillas HTML atractivas y bonitas para los mensajes y generar automáticamente una contraparte de texto sin formato.
> > Markdown mailable messages allow you to take advantage of the pre-built templates and components of mail notifications in your mailables. Since the messages are written in Markdown, Laravel is able to render beautiful, responsive HTML templates for the messages while also automatically generating a plain-text counterpart.

<a name="generating-markdown-mailables"></a>
### Generating Markdown Mailables

Para generar un mailable con una plantilla de rebajas correspondiente, puede usar la opción `--markdown` del comando Artisan `make:mail`:
> > To generate a mailable with a corresponding Markdown template, you may use the `--markdown` option of the `make:mail` Artisan command:

    php artisan make:mail OrderShipped --markdown=emails.orders.shipped

Luego, al configurar el mailable dentro de su método `build`, llame al método `markdown` en lugar del método `view`. Los métodos `markdown` aceptan el nombre de la plantilla Markdown y un array opcional de datos para poner a disposición de la plantilla:
> > Then, when configuring the mailable within its `build` method, call the `markdown` method instead of the `view` method. The `markdown` methods accepts the name of the Markdown template and an optional array of data to make available to the template:

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->markdown('emails.orders.shipped');
    }

<a name="writing-markdown-messages"></a>
### Escribir mensajes Markdown : Writing Markdown Messages

Markdown mailables utiliza una combinación de componentes Blade y sintaxis Markdown que le permiten construir fácilmente mensajes de correo al tiempo que aprovecha los componentes prefabricados de Laravel:
> > Markdown mailables use a combination of Blade components and Markdown syntax which allow you to easily construct mail messages while leveraging Laravel's pre-crafted components:

    @component('mail::message')
    # Order Shipped

    Your order has been shipped!

    @component('mail::button', ['url' => $url])
    View Order
    @endcomponent

    Thanks,<br>
    {{ config('app.name') }}
    @endcomponent

> {tip} No use una sangría excesiva al escribir correos electrónicos de Markdown. Los analizadores de Markdown renderizarán contenido sangrado como bloques de código.
> > > {tip} Do not use excess indentation when writing Markdown emails. Markdown parsers will render indented content as code blocks.

#### Componente de botón : Button Component

El componente de botón representa un enlace de botón centrado. El componente acepta dos argumentos, una `url` y un `color` opcional. Los colores admitidos son `primary`, `success`, y `error`. Puede agregar tantos componentes de botón a un mensaje como desee:
> > The button component renders a centered button link. The component accepts two arguments, a `url` and an optional `color`. Supported colors are `primary`, `success`, and `error`. You may add as many button components to a message as you wish:

    @component('mail::button', ['url' => $url, 'color' => 'success'])
    View Order
    @endcomponent

#### Componente de panel : Panel Component

El componente de panel representa el bloque de texto dado en un panel que tiene un color de fondo ligeramente diferente que el resto del mensaje. Esto le permite llamar la atención sobre un bloque de texto dado:
> > The panel component renders the given block of text in a panel that has a slightly different background color than the rest of the message. This allows you to draw attention to a given block of text:

    @component('mail::panel')
    This is the panel content.
    @endcomponent

#### Componente de tabla : Table Component

El componente de tabla le permite transformar una tabla Markdown en una tabla HTML. El componente acepta la tabla Markdown como contenido. La alineación de columna de tabla es compatible con la sintaxis de alineación de la tabla Markdown predeterminada:
> > The table component allows you to transform a Markdown table into an HTML table. The component accepts the Markdown table as its content. Table column alignment is supported using the default Markdown table alignment syntax:

    @component('mail::table')
    | Laravel       | Table         | Example  |
    | ------------- |:-------------:| --------:|
    | Col 2 is      | Centered      | $10      |
    | Col 3 is      | Right-Aligned | $20      |
    @endcomponent

<a name="customizing-the-components"></a>
### Personalización de los componentes : Customizing The Components

Puede exportar todos los componentes de correo Markdown a su propia aplicación para personalización. Para exportar los componentes, use el comando Artisan `vendor:publish` para publicar la etiqueta de elemento `laravel-mail`:
> > You may export all of the Markdown mail components to your own application for customization. To export the components, use the `vendor:publish` Artisan command to publish the `laravel-mail` asset tag:

    php artisan vendor:publish --tag=laravel-mail

Este comando publicará los componentes de correo Markdown en el directorio `resources/views/vendor/mail`. El directorio `mail` contendrá un directorio `html` y `markdown`, cada uno con sus respectivas representaciones de cada componente disponible. Los componentes en el directorio `html` se usan para generar la versión HTML de su correo electrónico, y sus contrapartes en el directorio `markdown` se usan para generar la versión de texto sin formato. Usted es libre de personalizar estos componentes como desee.
> > This command will publish the Markdown mail components to the `resources/views/vendor/mail` directory. The `mail` directory will contain a `html` and a `markdown` directory, each containing their respective representations of every available component. The components in the `html` directory are used to generate the HTML version of your email, and their counterparts in the `markdown` directory are used to generate the plain-text version. You are free to customize these components however you like.

#### Personalizar el CSS : Customizing The CSS

Después de exportar los componentes, el directorio `resources/views/vendor/mail/html/themes` contendrá un archivo `default.css`. Puede personalizar el CSS en este archivo y sus estilos se alinearán automáticamente en las representaciones HTML de sus mensajes de correo Markdown.
> > After exporting the components, the `resources/views/vendor/mail/html/themes` directory will contain a `default.css` file. You may customize the CSS in this file and your styles will automatically be in-lined within the HTML representations of your Markdown mail messages.

> {tip} Si desea crear un tema completamente nuevo para los componentes Markdown, escriba un nuevo archivo CSS dentro del directorio `html/themes` y cambie la opción `theme` de su archivo de configuración `mail`.
> > > {tip} If you would like to build an entirely new theme for the Markdown components, write a new CSS file within the `html/themes` directory and change the `theme` option of your `mail` configuration file.

<a name="sending-mail"></a>
## Enviando correo : Sending Mail

Para enviar un mensaje, use el método `to` en la [fachada](/docs/{{version}}/facades) `Mail`. El método `to` acepta una dirección de correo electrónico, una instancia de usuario o una colección de usuarios. Si pasa un objeto o una colección de objetos, el remitente utilizará automáticamente sus propiedades de `email` y `name` cuando configure los destinatarios del correo electrónico, por lo tanto, asegúrese de que estos atributos estén disponibles en sus objetos. Una vez que haya especificado sus destinatarios, puede pasar una instancia de su clase mailable al método `send`:
> > To send a message, use the `to` method on the `Mail` [facade](/docs/{{version}}/facades). The `to` method accepts an email address, a user instance, or a collection of users. If you pass an object or collection of objects, the mailer will automatically use their `email` and `name` properties when setting the email recipients, so make sure these attributes are available on your objects. Once you have specified your recipients, you may pass an instance of your mailable class to the `send` method:

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Mail\OrderShipped;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Mail;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  Request  $request
         * @param  int  $orderId
         * @return Response
         */
        public function ship(Request $request, $orderId)
        {
            $order = Order::findOrFail($orderId);

            // Ship order...

            Mail::to($request->user())->send(new OrderShipped($order));
        }
    }

Por supuesto, no está limitado a especificar los destinatarios "to" al enviar un mensaje. Usted es libre de configurar los destinatarios "to", "cc" y "bcc", todo dentro de una única llamada a un método encadenado:
> > Of course, you are not limited to just specifying the "to" recipients when sending a message. You are free to set "to", "cc", and "bcc" recipients all within a single, chained method call:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="rendering-mailables"></a>
## Rendering Mailables

A veces puede desear capturar el contenido HTML de un mailable sin enviarlo. Para lograr esto, puede llamar al método `render` del mailable. Este método devolverá los contenidos evaluados del mailable como una cadena:
> > Sometimes you may wish to capture the HTML content of a mailable without sending it. To accomplish this, you may call the `render` method of the mailable. This method will return the evaluated contents of the mailable as a string:

    $invoice = App\Invoice::find(1);

    return (new App\Mail\InvoicePaid($invoice))->render();

<a name="previewing-mailables-in-the-browser"></a>
### Vista previa de Mailables en el navegador : Previewing Mailables In The Browser

Al diseñar una plantilla de mailable, es conveniente obtener una vista previa rápida en su navegador como una plantilla típica de Blade. Por esta razón, Laravel le permite devolver cualquier mailable directamente desde un Closure de ruta o controlador. Cuando se devuelve un mailable, se procesará y se mostrará en el navegador, lo que le permite obtener una vista previa rápida de su diseño sin necesidad de enviarlo a una dirección real de correo electrónico:
> > When designing a mailable's template, it is convenient to quickly preview the rendered mailable in your browser like a typical Blade template. For this reason, Laravel allows you to return any mailable directly from a route Closure or controller. When a mailable is returned, it will be rendered and displayed in the browser, allowing you to quickly preview its design without needing to send it to an actual email address:

    Route::get('/mailable', function () {
        $invoice = App\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

<a name="queueing-mail"></a>
### Correo en cola : Queueing Mail

#### Poner un mensaje de correo en cola : Queueing A Mail Message

Dado que el envío de mensajes de correo electrónico puede alargar drásticamente el tiempo de respuesta de su aplicación, muchos desarrolladores eligen poner en cola los mensajes de correo electrónico para el envío en segundo plano. Laravel hace esto fácil usando su [API de cola unificada](/docs/{{version}}/queues) incorporada. Para poner en cola un mensaje de correo, use el método `queue` en la fachada `Mail` después de especificar los destinatarios del mensaje:
> > Since sending email messages can drastically lengthen the response time of your application, many developers choose to queue email messages for background sending. Laravel makes this easy using its built-in [unified queue API](/docs/{{version}}/queues). To queue a mail message, use the `queue` method on the `Mail` facade after specifying the message's recipients:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

Este método se encargará automáticamente de insertar un trabajo en la cola para que el mensaje se envíe en segundo plano. Por supuesto, necesitará [configurar sus colas](/docs/{{version}}/queues) antes de usar esta función.
> > This method will automatically take care of pushing a job onto the queue so the message is sent in the background. Of course, you will need to [configure your queues](/docs/{{version}}/queues) before using this feature.

#### Cola de mensajes retrasada : Delayed Message Queueing

Si desea retrasar la entrega de un mensaje de correo electrónico en cola, puede usar el método `later`. Como primer argumento, el método `later` acepta una instancia `DateTime` que indica cuándo se debe enviar el mensaje:
> > If you wish to delay the delivery of a queued email message, you may use the `later` method. As its first argument, the `later` method accepts a `DateTime` instance indicating when the message should be sent:

    $when = now()->addMinutes(10);

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later($when, new OrderShipped($order));

#### Empujar a colas específicas : Pushing To Specific Queues

Como todas las clases mailable generadas usando el comando `make:mail` hacen uso del rasgo `Illuminate\Bus\Queueable`, puede llamar a los métodos `onQueue` y `onConnection` en cualquier instancia de clase mailable, lo que le permite especificar la conexión y nombre de cola para el mensaje:
> > Since all mailable classes generated using the `make:mail` command make use of the `Illuminate\Bus\Queueable` trait, you may call the `onQueue` and `onConnection` methods on any mailable class instance, allowing you to specify the connection and queue name for the message:

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

#### Colas por defecto : Queueing By Default

Si tiene clases mailable que desea que estén siempre en cola, puede implementar el contrato `ShouldQueue` en la clase. Ahora, incluso si llama al método `send` cuando envía correos, el mailable seguirá en cola ya que implementa el contrato:
> > If you have mailable classes that you want to always be queued, you may implement the `ShouldQueue` contract on the class. Now, even if you call the `send` method when mailing, the mailable will still be queued since it implements the contract:

    use Illuminate\Contracts\Queue\ShouldQueue;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        //
    }

<a name="mail-and-local-development"></a>
## Mail y desarrollo local : Mail & Local Development

Al desarrollar una aplicación que envía correos electrónicos, probablemente no desee enviar correos electrónicos a direcciones de correo electrónico en vivo. Laravel proporciona varias formas de "desactivar" el envío real de correos electrónicos durante el desarrollo local.
> > When developing an application that sends email, you probably don't want to actually send emails to live email addresses. Laravel provides several ways to "disable" the actual sending of emails during local development.

#### Log Driver : Log Driver

En lugar de enviar sus correos electrónicos, el controlador de correo `log` escribirá todos los mensajes de correo electrónico en sus archivos de registro para su inspección. Para obtener más información sobre cómo configurar su aplicación por entorno, consulte la [documentación de configuración](/docs/{{version}}/configuration#environment-configuration).
> > Instead of sending your emails, the `log` mail driver will write all email messages to your log files for inspection. For more information on configuring your application per environment, check out the [configuration documentation](/docs/{{version}}/configuration#environment-configuration).

#### Universal To

Otra solución proporcionada por Laravel es establecer un destinatario universal de todos los correos electrónicos enviados por el framework. De esta forma, todos los correos electrónicos generados por su aplicación serán enviados a una dirección específica, en lugar de la dirección realmente especificada al enviar el mensaje. Esto se puede hacer a través de la opción `to` en su archivo de configuración `config/mail.php`:
> > Another solution provided by Laravel is to set a universal recipient of all emails sent by the framework. This way, all the emails generated by your application will be sent to a specific address, instead of the address actually specified when sending the message. This can be done via the `to` option in your `config/mail.php` configuration file:

    'to' => [
        'address' => 'example@example.com',
        'name' => 'Example'
    ],

#### Mailtrap

Finalmente, puede usar un servicio como [Mailtrap](https://mailtrap.io) y el controlador `smtp` para enviar sus mensajes de correo electrónico a un buzón "ficticio" donde puede verlos en un verdadero cliente de correo electrónico. Este enfoque tiene el beneficio de permitirle inspeccionar realmente los correos electrónicos finales en el visor de mensajes de Mailtrap.
> > Finally, you may use a service like [Mailtrap](https://mailtrap.io) and the `smtp` driver to send your email messages to a "dummy" mailbox where you may view them in a true email client. This approach has the benefit of allowing you to actually inspect the final emails in Mailtrap's message viewer.

<a name="events"></a>
## Eventos : Events

Laravel dispara dos eventos durante el proceso de envío de mensajes de correo. El evento `MessageSending` se dispara antes de que se envíe un mensaje, mientras que el evento `MessageSent` se dispara después de que se ha enviado un mensaje. Recuerde, estos eventos se disparan cuando el correo *se envía*, no cuando se pone en cola. Puede registrar un detector de eventos para este evento en su `EventServiceProvider`:
> > Laravel fires two events during the process of sending mail messages. The `MessageSending` event is fired prior to a message being sent, while the `MessageSent` event is fired after a message has been sent. Remember, these events are fired when the mail is being *sent*, not when it is queued. You may register an event listener for this event in your `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Mail\Events\MessageSending' => [
            'App\Listeners\LogSendingMessage',
        ],
        'Illuminate\Mail\Events\MessageSent' => [
            'App\Listeners\LogSentMessage',
        ],
    ];
