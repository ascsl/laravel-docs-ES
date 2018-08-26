# Cajero Laravel
# Laravel Cashier

- [Introduction](#introduction)
- [Configuration](#configuration)
    - [Stripe](#stripe-configuration)
    - [Braintree](#braintree-configuration)
    - [Currency Configuration](#currency-configuration)
- [Subscriptions](#subscriptions)
    - [Creating Subscriptions](#creating-subscriptions)
    - [Checking Subscription Status](#checking-subscription-status)
    - [Changing Plans](#changing-plans)
    - [Subscription Quantity](#subscription-quantity)
    - [Subscription Taxes](#subscription-taxes)
    - [Cancelling Subscriptions](#cancelling-subscriptions)
    - [Resuming Subscriptions](#resuming-subscriptions)
    - [Updating Credit Cards](#updating-credit-cards)
- [Subscription Trials](#subscription-trials)
    - [With Credit Card Up Front](#with-credit-card-up-front)
    - [Without Credit Card Up Front](#without-credit-card-up-front)
- [Handling Stripe Webhooks](#handling-stripe-webhooks)
    - [Defining Webhook Event Handlers](#defining-webhook-event-handlers)
    - [Failed Subscriptions](#handling-failed-subscriptions)
- [Handling Braintree Webhooks](#handling-braintree-webhooks)
    - [Defining Webhook Event Handlers](#defining-braintree-webhook-event-handlers)
    - [Failed Subscriptions](#handling-braintree-failed-subscriptions)
- [Single Charges](#single-charges)
- [Invoices](#invoices)
    - [Generating Invoice PDFs](#generating-invoice-pdfs)

<a name="introduction"></a>
## Introduction
## Introducción

Laravel Cashier ofrece una interfaz expresiva y fluida para los servicios de facturación por suscripción [Stripe's] (https://stripe.com) y [Braintree's](https://www.braintreepayments.com). Maneja casi todo el código de facturación de suscripción repetitivo que temes escribir. Además de la administración básica de suscripciones, Cashier puede manejar cupones, intercambiar subscripciones, "cantidades" de suscripciones, períodos de gracia de cancelación e incluso generar PDF de facturas.
> > Laravel Cashier provides an expressive, fluent interface to [Stripe's](https://stripe.com) and [Braintree's](https://www.braintreepayments.com) subscription billing services. It handles almost all of the boilerplate subscription billing code you are dreading writing. In addition to basic subscription management, Cashier can handle coupons, swapping subscription, subscription "quantities", cancellation grace periods, and even generate invoice PDFs.

> {note} Si solo realiza cargos "únicos" y no ofrece suscripciones, no debe usar Cashier. En su lugar, use los SDK Stripe y Braintree directamente.
> > > {note} If you're only performing "one-off" charges and do not offer subscriptions, you should not use Cashier. Instead, use the Stripe and Braintree SDKs directly.

<a name="configuration"></a>
## Configuración
## Configuration

<a name="stripe-configuration"></a>
### Stripe

#### Composer

Primero, agregue el paquete de cajero para Stripe a sus dependencias:
> > First, add the Cashier package for Stripe to your dependencies:

    composer require "laravel/cashier":"~7.0"

#### Migraciones de base de datos
#### Database Migrations

Antes de usar Cashier, también necesitaremos [preparar la base de datos] (/docs/{{version}}/migrations). Necesitamos agregar varias columnas a su tabla `users` y crear una nueva tabla `subscriptions` para contener todas las suscripciones de nuestros clientes:
> > Before using Cashier, we'll also need to [prepare the database](/docs/{{version}}/migrations). We need to add several columns to your `users` table and create a new `subscriptions` table to hold all of our customer's subscriptions:

    Schema::table('users', function ($table) {
        $table->string('stripe_id')->nullable();
        $table->string('card_brand')->nullable();
        $table->string('card_last_four')->nullable();
        $table->timestamp('trial_ends_at')->nullable();
    });

    Schema::create('subscriptions', function ($table) {
        $table->increments('id');
        $table->unsignedInteger('user_id');
        $table->string('name');
        $table->string('stripe_id');
        $table->string('stripe_plan');
        $table->integer('quantity');
        $table->timestamp('trial_ends_at')->nullable();
        $table->timestamp('ends_at')->nullable();
        $table->timestamps();
    });

Una vez que las migraciones han sido creadas, ejecute el comando `migrate` Artisan.
> > Once the migrations have been created, run the `migrate` Artisan command.

#### Modelo facturable
#### Billable Model

A continuación, agregue el rasgo `Billable` a la definición de su modelo. Este rasgo proporciona varios métodos que le permiten realizar tareas de facturación comunes, como crear suscripciones, aplicar cupones y actualizar la información de la tarjeta de crédito:
> > Next, add the `Billable` trait to your model definition. This trait provides various methods to allow you to perform common billing tasks, such as creating subscriptions, applying coupons, and updating credit card information:

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### Claves API
#### API Keys

Finalmente, debe configurar su clave Stripe en su archivo de configuración `services.php`. Puede recuperar sus claves de Stripe API desde el panel de control de Stripe:
> > Finally, you should configure your Stripe key in your `services.php` configuration file. You can retrieve your Stripe API keys from the Stripe control panel:

    'stripe' => [
        'model'  => App\User::class,
        'key' => env('STRIPE_KEY'),
        'secret' => env('STRIPE_SECRET'),
    ],

<a name="braintree-configuration"></a>
### Braintree

#### Advertencias de Braintree
#### Braintree Caveats

Para muchas operaciones, las implementaciones de cajero de Stripe y Braintree funcionan de la misma manera. Ambos servicios ofrecen facturación de suscripción con tarjetas de crédito, pero Braintree también admite pagos a través de PayPal. Sin embargo, Braintree también carece de algunas características compatibles con Stripe. Debe tener en cuenta lo siguiente cuando decida usar Stripe o Braintree:
> > For many operations, the Stripe and Braintree implementations of Cashier function the same. Both services provide subscription billing with credit cards but Braintree also supports payments via PayPal. However, Braintree also lacks some features that are supported by Stripe. You should keep the following in mind when deciding to use Stripe or Braintree:

<div class="content-list" markdown="1">
- Braintree admite PayPal, mientras que Stripe no.
- Braintree no admite los métodos `increment` y `decrement` en las suscripciones. Esta es una limitación de Braintree, no una limitación de cajero.
- Braintree no admite descuentos basados ​​en porcentajes. Esta es una limitación de Braintree, no una limitación de cajero.
</div>
<div class="content-list" markdown="1">
- Braintree supports PayPal while Stripe does not.
- Braintree does not support the `increment` and `decrement` methods on subscriptions. This is a Braintree limitation, not a Cashier limitation.
- Braintree does not support percentage based discounts. This is a Braintree limitation, not a Cashier limitation.
</div>

#### Composer

Primero, agregue el paquete de Cajero para Braintree a sus dependencias:
> > First, add the Cashier package for Braintree to your dependencies:

    composer require "laravel/cashier-braintree":"~2.0"

#### Proveedor de servicio
#### Service Provider

A continuación, registre `Laravel\Cashier\CashierServiceProvider` [proveedor de servicios](/docs/{{version}}/providers) en su archivo de configuración `config/app.php`:
> > Next, register the `Laravel\Cashier\CashierServiceProvider` [service provider](/docs/{{version}}/providers) in your `config/app.php` configuration file:

    Laravel\Cashier\CashierServiceProvider::class

#### Plan de cupón de crédito
#### Plan Credit Coupon

Antes de utilizar Cashier con Braintree, deberá definir un descuento de "plan-crédito" en su panel de control Braintree. Este descuento se usará para prorratear adecuadamente las suscripciones que cambian de facturación anual a mensual o de facturación mensual a anual.
> > Before using Cashier with Braintree, you will need to define a `plan-credit` discount in your Braintree control panel. This discount will be used to properly prorate subscriptions that change from yearly to monthly billing, or from monthly to yearly billing.

El monto de descuento configurado en el panel de control de Braintree puede ser cualquier valor que desee, ya que el Cajero anulará la cantidad definida con nuestro propio monto personalizado cada vez que apliquemos el cupón. Este cupón es necesario ya que Braintree no es compatible de forma nativa con las suscripciones a través de las frecuencias de suscripción.
> > The discount amount configured in the Braintree control panel can be any value you wish, as Cashier will override the defined amount with our own custom amount each time we apply the coupon. This coupon is needed since Braintree does not natively support prorating subscriptions across subscription frequencies.

#### Migraciones de base de datos
#### Database Migrations

Antes de usar Cashier, necesitaremos [preparar la base de datos] (/docs/{{version}}/migrations). Necesitamos agregar varias columnas a su tabla `users` y crear una nueva tabla `subscriptions` para contener todas las suscripciones de nuestros clientes:
> > Before using Cashier, we'll need to [prepare the database](/docs/{{version}}/migrations). We need to add several columns to your `users` table and create a new `subscriptions` table to hold all of our customer's subscriptions:

    Schema::table('users', function ($table) {
        $table->string('braintree_id')->nullable();
        $table->string('paypal_email')->nullable();
        $table->string('card_brand')->nullable();
        $table->string('card_last_four')->nullable();
        $table->timestamp('trial_ends_at')->nullable();
    });

    Schema::create('subscriptions', function ($table) {
        $table->increments('id');
        $table->unsignedInteger('user_id');
        $table->string('name');
        $table->string('braintree_id');
        $table->string('braintree_plan');
        $table->integer('quantity');
        $table->timestamp('trial_ends_at')->nullable();
        $table->timestamp('ends_at')->nullable();
        $table->timestamps();
    });

Una vez que las migraciones han sido creadas, ejecute el comando `migrate` Artisan.
> > Once the migrations have been created, run the `migrate` Artisan command.

#### Modelo facturable
#### Billable Model

A continuación, agregue el rasgo `Billable` a su definición de modelo:
> > Next, add the `Billable` trait to your model definition:

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

#### Claves API
#### API Keys

A continuación, debe configurar las siguientes opciones en su archivo `services.php`:
> > Next, you should configure the following options in your `services.php` file:

    'braintree' => [
        'model'  => App\User::class,
        'environment' => env('BRAINTREE_ENV'),
        'merchant_id' => env('BRAINTREE_MERCHANT_ID'),
        'public_key' => env('BRAINTREE_PUBLIC_KEY'),
        'private_key' => env('BRAINTREE_PRIVATE_KEY'),
    ],

Luego, debe agregar las siguientes llamadas de Braintree SDK al método `boot` del proveedor de servicios `AppServiceProvider`:
> > Then you should add the following Braintree SDK calls to your `AppServiceProvider` service provider's `boot` method:

    \Braintree_Configuration::environment(config('services.braintree.environment'));
    \Braintree_Configuration::merchantId(config('services.braintree.merchant_id'));
    \Braintree_Configuration::publicKey(config('services.braintree.public_key'));
    \Braintree_Configuration::privateKey(config('services.braintree.private_key'));

<a name="currency-configuration"></a>
### Configuración de moneda
### Currency Configuration

La moneda de cajero predeterminada es dólares estadounidenses (USD). Puede cambiar la moneda predeterminada llamando al método `Cashier::useCurrency` desde el método `boot` de uno de sus proveedores de servicios. El método `useCurrency` acepta dos parámetros de cadena: la moneda y el símbolo de la moneda:
> > The default Cashier currency is United States Dollars (USD). You can change the default currency by calling the `Cashier::useCurrency` method from within the `boot` method of one of your service providers. The `useCurrency` method accepts two string parameters: the currency and the currency's symbol:

    use Laravel\Cashier\Cashier;

    Cashier::useCurrency('eur', '€');

<a name="subscriptions"></a>
## Subscripciones
## Subscriptions

<a name="creating-subscriptions"></a>
### Creando suscripciones
### Creating Subscriptions

Para crear una suscripción, primero recupere una instancia de su modelo facturable, que típicamente será una instancia de `App\User`. Una vez que haya recuperado la instancia del modelo, puede usar el método `newSubscription` para crear la suscripción del modelo:
> > To create a subscription, first retrieve an instance of your billable model, which typically will be an instance of `App\User`. Once you have retrieved the model instance, you may use the `newSubscription` method to create the model's subscription:

    $user = User::find(1);

    $user->newSubscription('main', 'premium')->create($stripeToken);

El primer argumento pasado al método `newSubscription` debe ser el nombre de la suscripción. Si su aplicación solo ofrece una suscripción única, puede llamarla `main` o `primary`. El segundo argumento es el plan específico de Stripe / Braintree al que el usuario se está suscribiendo. Este valor debe corresponderse con el identificador del plan en Stripe o Braintree.
> > The first argument passed to the `newSubscription` method should be the name of the subscription. If your application only offers a single subscription, you might call this `main` or `primary`. The second argument is the specific Stripe / Braintree plan the user is subscribing to. This value should correspond to the plan's identifier in Stripe or Braintree.

El método `create`, que acepta un token de tarjeta de crédito / fuente Stripe, comenzará la suscripción y actualizará su base de datos con la identificación del cliente y otra información de facturación relevante.
> > The `create` method, which accepts a Stripe credit card / source token, will begin the subscription as well as update your database with the customer ID and other relevant billing information.

#### Detalles de usuario adicionales
#### Additional User Details

Si desea especificar detalles adicionales del cliente, puede hacerlo pasándolos como el segundo argumento del método `create`:
> > If you would like to specify additional customer details, you may do so by passing them as the second argument to the `create` method:

    $user->newSubscription('main', 'monthly')->create($stripeToken, [
        'email' => $email,
    ]);

Para obtener más información acerca de los campos adicionales admitidos por Stripe o Braintree, consulte Stripe's [documentación sobre creación de clientes](https://stripe.com/docs/api#create_customer) o la correspondiente [documentación de Braintree](https://developers.braintreepayments.com/reference/request/customer/create/php).
> > To learn more about the additional fields supported by Stripe or Braintree, check out Stripe's [documentation on customer creation](https://stripe.com/docs/api#create_customer) or the corresponding [Braintree documentation](https://developers.braintreepayments.com/reference/request/customer/create/php).

#### Cupones
#### Coupons

Si desea aplicar un cupón al crear la suscripción, puede usar el método `withCoupon`:
> > If you would like to apply a coupon when creating the subscription, you may use the `withCoupon` method:

    $user->newSubscription('main', 'monthly')
         ->withCoupon('code')
         ->create($stripeToken);

<a name="checking-subscription-status"></a>
### Comprobación del estado de la suscripción
### Checking Subscription Status

Una vez que un usuario está suscrito a su aplicación, puede verificar fácilmente el estado de su suscripción utilizando una variedad de métodos convenientes. Primero, el método `subscribed` devuelve `true` si el usuario tiene una suscripción activa, incluso si la suscripción se encuentra actualmente dentro de su período de prueba:
> > Once a user is subscribed to your application, you may easily check their subscription status using a variety of convenient methods. First, the `subscribed` method returns `true` if the user has an active subscription, even if the subscription is currently within its trial period:

    if ($user->subscribed('main')) {
        //
    }

El método `subscribed` también es un gran candidato para [middleware de ruta](/docs/{{version}}/middleware), lo que le permite filtrar el acceso a rutas y controladores en función del estado de suscripción del usuario:
> > The `subscribed` method also makes a great candidate for a [route middleware](/docs/{{version}}/middleware), allowing you to filter access to routes and controllers based on the user's subscription status:

    public function handle($request, Closure $next)
    {
        if ($request->user() && ! $request->user()->subscribed('main')) {
            // This user is not a paying customer...
            return redirect('billing');
        }

        return $next($request);
    }

Si desea determinar si un usuario aún se encuentra dentro de su período de prueba, puede usar el método `onTrial`. Este método puede ser útil para mostrar una advertencia al usuario de que todavía están en período de prueba:
> > If you would like to determine if a user is still within their trial period, you may use the `onTrial` method. This method can be useful for displaying a warning to the user that they are still on their trial period:

    if ($user->subscription('main')->onTrial()) {
        //
    }

El método `suscribedToPlan` se puede usar para determinar si el usuario está suscrito a un plan determinado en función de una ID de plan de Stripe / Braintree dada. En este ejemplo, determinaremos si la suscripción `main` del usuario está activamente suscrita al plan `monthly`:
> > The `subscribedToPlan` method may be used to determine if the user is subscribed to a given plan based on a given Stripe / Braintree plan ID. In this example, we will determine if the user's `main` subscription is actively subscribed to the `monthly` plan:

    if ($user->subscribedToPlan('monthly', 'main')) {
        //
    }

#### Estado de suscripción cancelado
#### Cancelled Subscription Status

Para determinar si el usuario alguna vez fue un suscriptor activo, pero ha cancelado su suscripción, puede usar el método `cancelled`:
> > To determine if the user was once an active subscriber, but has cancelled their subscription, you may use the `cancelled` method:

    if ($user->subscription('main')->cancelled()) {
        //
    }

También puede determinar si un usuario ha cancelado su suscripción, pero aún se encuentra en su "período de gracia" hasta que la suscripción caduque por completo. Por ejemplo, si un usuario cancela una suscripción el 5 de marzo que originalmente estaba programado para vencer el 10 de marzo, el usuario está en su "período de gracia" hasta el 10 de marzo. Tenga en cuenta que el método `subscribed` aún devuelve `true` durante este tiempo:
> > You may also determine if a user has cancelled their subscription, but are still on their "grace period" until the subscription fully expires. For example, if a user cancels a subscription on March 5th that was originally scheduled to expire on March 10th, the user is on their "grace period" until March 10th. Note that the `subscribed` method still returns `true` during this time:

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

<a name="changing-plans"></a>
### Cambio de planes
### Changing Plans

Después de que un usuario esté suscrito a su aplicación, ocasionalmente querrá cambiar a un nuevo plan de suscripción. Para cambiar a un usuario a una nueva suscripción, pase el identificador del plan al método `swap`:
> > After a user is subscribed to your application, they may occasionally want to change to a new subscription plan. To swap a user to a new subscription, pass the plan's identifier to the `swap` method:

    $user = App\User::find(1);

    $user->subscription('main')->swap('provider-plan-id');

Si el usuario está en prueba, se mantendrá el período de prueba. Además, si existe una "cantidad" para la suscripción, esa cantidad también se mantendrá.
> > If the user is on trial, the trial period will be maintained. Also, if a "quantity" exists for the subscription, that quantity will also be maintained.

Si desea cambiar de plan y cancelar cualquier período de prueba que el usuario tenga actualmente, puede usar el método `skipTrial`:
> > If you would like to swap plans and cancel any trial period the user is currently on, you may use the `skipTrial` method:

    $user->subscription('main')
            ->skipTrial()
            ->swap('provider-plan-id');

<a name="subscription-quantity"></a>
### Cantidad de suscripciones
### Subscription Quantity

> {note} Las cantidades de suscripción solo son compatibles con la edición de Stripe de Cashier. Braintree no tiene una función que corresponda a la "cantidad" de Stripe.
> > > {note} Subscription quantities are only supported by the Stripe edition of Cashier. Braintree does not have a feature that corresponds to Stripe's "quantity".

En ocasiones, las suscripciones se ven afectadas por la "cantidad". Por ejemplo, su aplicación puede cobrar $ 10 por mes ** por usuario ** en una cuenta. Para incrementar o disminuir fácilmente su cantidad de suscripción, use los métodos `incrementQuantity` y `decrementQuantity`:
> > Sometimes subscriptions are affected by "quantity". For example, your application might charge $10 per month **per user** on an account. To easily increment or decrement your subscription quantity, use the `incrementQuantity` and `decrementQuantity` methods:

    $user = User::find(1);

    $user->subscription('main')->incrementQuantity();

    // Add five to the subscription's current quantity...
    $user->subscription('main')->incrementQuantity(5);

    $user->subscription('main')->decrementQuantity();

    // Subtract five to the subscription's current quantity...
    $user->subscription('main')->decrementQuantity(5);

Alternativamente, puede establecer una cantidad específica usando el método `updateQuantity`:
> > Alternatively, you may set a specific quantity using the `updateQuantity` method:

    $user->subscription('main')->updateQuantity(10);

El método `noProrate` se puede usar para actualizar la cantidad de la suscripción sin prorratear los cargos:
> > The `noProrate` method may be used to update the subscription's quantity without pro-rating the charges:

    $user->subscription('main')->noProrate()->updateQuantity(10);

Para obtener más información sobre las cantidades de suscripción, consulte la [documentación de Stripe](https://stripe.com/docs/subscriptions/quantities).
> > For more information on subscription quantities, consult the [Stripe documentation](https://stripe.com/docs/subscriptions/quantities).

<a name="subscription-taxes"></a>
### Impuestos de suscripción
### Subscription Taxes

Para especificar el porcentaje de impuestos que un usuario paga en una suscripción, implemente el método `taxPercentage` en su modelo facturable y devuelva un valor numérico entre 0 y 100, con no más de 2 lugares decimales.
> > To specify the tax percentage a user pays on a subscription, implement the `taxPercentage` method on your billable model, and return a numeric value between 0 and 100, with no more than 2 decimal places.

    public function taxPercentage() {
        return 20;
    }

El método `taxPercentage` le permite aplicar una tasa impositiva modelo por modelo, que puede ser útil para una base de usuarios que abarca varios países y tasas impositivas.
> > The `taxPercentage` method enables you to apply a tax rate on a model-by-model basis, which may be helpful for a user base that spans multiple countries and tax rates.

> {note} El método `taxPercentage` solo se aplica a los cargos de suscripción. Si usa el Cajero para hacer cargos "únicos", deberá especificar manualmente la tasa de impuestos en ese momento.
> > > {note} The `taxPercentage` method only applies to subscription charges. If you use Cashier to make "one off" charges, you will need to manually specify the tax rate at that time.

<a name="cancelling-subscriptions"></a>
### Cancelar suscripciones
### Cancelling Subscriptions

Para cancelar una suscripción, llame al método `cancel` en la suscripción del usuario:
> > To cancel a subscription, call the `cancel` method on the user's subscription:

    $user->subscription('main')->cancel();

Cuando se cancela una suscripción, Cashier automáticamente configurará la columna `ends_at` en su base de datos. Esta columna se usa para saber cuándo el método `subscribed` debería comenzar a devolver `false`. Por ejemplo, si un cliente cancela una suscripción el 1 de marzo, pero la suscripción no estaba programada para finalizar hasta el 5 de marzo, el método `subscribed` continuará siendo `true` hasta el 5 de marzo.
> > When a subscription is cancelled, Cashier will automatically set the `ends_at` column in your database. This column is used to know when the `subscribed` method should begin returning `false`. For example, if a customer cancels a subscription on March 1st, but the subscription was not scheduled to end until March 5th, the `subscribed` method will continue to return `true` until March 5th.

Puede determinar si un usuario ha cancelado su suscripción pero aún está en su "período de gracia" utilizando el método `onGracePeriod`:
> > You may determine if a user has cancelled their subscription but are still on their "grace period" using the `onGracePeriod` method:

    if ($user->subscription('main')->onGracePeriod()) {
        //
    }

Si desea cancelar una suscripción de inmediato, llame al método `cancelNow` en la suscripción del usuario:
> > If you wish to cancel a subscription immediately, call the `cancelNow` method on the user's subscription:

    $user->subscription('main')->cancelNow();

<a name="resuming-subscriptions"></a>
### Reanudar suscripciones
### Resuming Subscriptions

Si un usuario ha cancelado su suscripción y desea reanudarla, utilice el método `resume`. El usuario **debe** seguir estando en su período de gracia para poder reanudar una suscripción:
> > If a user has cancelled their subscription and you wish to resume it, use the `resume` method. The user **must** still be on their grace period in order to resume a subscription:

    $user->subscription('main')->resume();

Si el usuario cancela una suscripción y luego reanuda esa suscripción antes de que la suscripción haya expirado por completo, no se le cobrará inmediatamente. En cambio, su suscripción se volverá a activar y se les facturará en el ciclo de facturación original.
> > If the user cancels a subscription and then resumes that subscription before the subscription has fully expired, they will not be billed immediately. Instead, their subscription will be re-activated, and they will be billed on the original billing cycle.

<a name="updating-credit-cards"></a>
### Actualización de tarjetas de crédito
### Updating Credit Cards

El método `updateCard` se puede usar para actualizar la información de la tarjeta de crédito de un cliente. Este método acepta un token Stripe y asignará la nueva tarjeta de crédito como fuente de facturación predeterminada:
> > The `updateCard` method may be used to update a customer's credit card information. This method accepts a Stripe token and will assign the new credit card as the default billing source:

    $user->updateCard($stripeToken);

<a name="subscription-trials"></a>
## Ensayos de suscripción
## Subscription Trials

<a name="with-credit-card-up-front"></a>
### Con tarjeta de crédito en el frente
### With Credit Card Up Front

Si desea ofrecer períodos de prueba a sus clientes al mismo tiempo que recopila la información del método de pago por adelantado, debe usar el método `testDays` al crear sus suscripciones:
> > If you would like to offer trial periods to your customers while still collecting payment method information up front, you should use the `trialDays` method when creating your subscriptions:

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')
                ->trialDays(10)
                ->create($stripeToken);

Este método establecerá la fecha de finalización del período de prueba en el registro de suscripción dentro de la base de datos, así como también le indicará a Stripe / Braintree que no comience a facturar al cliente hasta después de esta fecha.
> > This method will set the trial period ending date on the subscription record within the database, as well as instruct Stripe / Braintree to not begin billing the customer until after this date.

> {note} Si la suscripción del cliente no se cancela antes de la fecha de finalización de la prueba, se le cobrará tan pronto como expire la versión de prueba, por lo que debe asegurarse de notificar a los usuarios sobre la fecha de finalización de la prueba.
> > > {note} If the customer's subscription is not cancelled before the trial ending date they will be charged as soon as the trial expires, so you should be sure to notify your users of their trial ending date.

El método `trialUntil` le permite proporcionar una instancia `DateTime` para especificar cuándo debe finalizar el período de prueba:
> > The `trialUntil` method allows you to provide a `DateTime` instance to specify when the trial period should end:

    use Carbon\Carbon;

    $user->newSubscription('main', 'monthly')
                ->trialUntil(Carbon::now()->addDays(10))
                ->create($stripeToken);

Puede determinar si el usuario está dentro de su período de prueba utilizando el método `onTrial` de la instancia del usuario o el método `onTrial` de la instancia de suscripción. Los dos ejemplos a continuación son idénticos:
> > You may determine if the user is within their trial period using either the `onTrial` method of the user instance, or the `onTrial` method of the subscription instance. The two examples below are identical:

    if ($user->onTrial('main')) {
        //
    }

    if ($user->subscription('main')->onTrial()) {
        //
    }

<a name="without-credit-card-up-front"></a>
### Sin tarjeta de crédito en la parte delantera
### Without Credit Card Up Front

Si desea ofrecer periodos de prueba sin recopilar previamente la información del método de pago del usuario, puede configurar la columna `trial_ends_at` en el registro del usuario a la fecha de finalización de la prueba que desee. Esto se hace típicamente durante el registro del usuario:
> > If you would like to offer trial periods without collecting the user's payment method information up front, you may set the `trial_ends_at` column on the user record to your desired trial ending date. This is typically done during user registration:

    $user = User::create([
        // Populate other user properties...
        'trial_ends_at' => now()->addDays(10),
    ]);

> {note} Asegúrese de agregar un [date mutator](/docs/{{version}}/eloquent-mutators#date-mutators) para `trial_ends_at` a la definición de su modelo.
> > > {note}  Be sure to add a [date mutator](/docs/{{version}}/eloquent-mutators#date-mutators) for `trial_ends_at` to your model definition.

El cajero se refiere a este tipo de prueba como una "prueba genérica", ya que no está asociada a ninguna suscripción existente. El método `onTrial` en la instancia `User` devolverá `true` si la fecha actual no está más allá del valor de `trial_ends_at`:
> > Cashier refers to this type of trial as a "generic trial", since it is not attached to any existing subscription. The `onTrial` method on the `User` instance will return `true` if the current date is not past the value of `trial_ends_at`:

    if ($user->onTrial()) {
        // User is within their trial period...
    }

También puede utilizar el método `onGenericTrial `si desea saber específicamente que el usuario se encuentra dentro de su período de prueba" genérico "y aún no ha creado una suscripción real:
> > You may also use the `onGenericTrial` method if you wish to know specifically that the user is within their "generic" trial period and has not created an actual subscription yet:

    if ($user->onGenericTrial()) {
        // User is within their "generic" trial period...
    }

Una vez que esté listo para crear una suscripción real para el usuario, puede usar el método `newSubscription` como de costumbre:
> > Once you are ready to create an actual subscription for the user, you may use the `newSubscription` method as usual:

    $user = User::find(1);

    $user->newSubscription('main', 'monthly')->create($stripeToken);

<a name="handling-stripe-webhooks"></a>
## Manejo de Webhooks de banda
## Handling Stripe Webhooks

Tanto Stripe como Braintree pueden notificar a su aplicación una variedad de eventos a través de webhooks. Para manejar los webhooks de Stripe, defina una ruta que apunte al controlador webhook de Cashier. Este controlador manejará todas las solicitudes entrantes de webhook y las enviará al método de controlador apropiado:
> > Both Stripe and Braintree can notify your application of a variety of events via webhooks. To handle Stripe webhooks, define a route that points to Cashier's webhook controller. This controller will handle all incoming webhook requests and dispatch them to the proper controller method:

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

> {note} Una vez que haya registrado su ruta, asegúrese de configurar la URL del webhook en la configuración de su panel de control Stripe.
> > > {note} Once you have registered your route, be sure to configure the webhook URL in your Stripe control panel settings.

Por defecto, este controlador manejará automáticamente la cancelación de suscripciones que tienen demasiados cargos fallidos (según lo definido por su configuración de Stripe); sin embargo, como pronto descubriremos, puede extender este controlador para manejar cualquier evento webhook que desee.
> > By default, this controller will automatically handle cancelling subscriptions that have too many failed charges (as defined by your Stripe settings); however, as we'll soon discover, you can extend this controller to handle any webhook event you like.

#### Webhooks y protección CSRF
#### Webhooks & CSRF Protection

Como los webhooks de Stripe necesitan eludir la [protección CSRF] de Laravel (/docs/{{version}}/csrf), asegúrese de incluir el URI como excepción en su middleware `VerifyCsrfToken` o haga una lista de la ruta fuera del grupo middleware `web`:
> > Since Stripe webhooks need to bypass Laravel's [CSRF protection](/docs/{{version}}/csrf), be sure to list the URI as an exception in your `VerifyCsrfToken` middleware or list the route outside of the `web` middleware group:

    protected $except = [
        'stripe/*',
    ];

<a name="defining-webhook-event-handlers"></a>
### Definición de manejadores de eventos Webhook
### Defining Webhook Event Handlers

El cajero maneja automáticamente la cancelación de suscripción en los cargos fallidos, pero si tiene eventos adicionales de webhook de Stripe que le gustaría manejar, extienda el controlador Webhook. Los nombres de sus métodos deben corresponderse con la convención esperada del Cajero, específicamente, los métodos deben tener el prefijo `handle` y el nombre de "camel case" del webhook Stripe que desea manejar. Por ejemplo, si desea manejar el webhook `invoice.payment_succeeded`, debe agregar un método `handleInvoicePaymentSucceeded` al controlador:
> > Cashier automatically handles subscription cancellation on failed charges, but if you have additional Stripe webhook events you would like to handle, extend the Webhook controller. Your method names should correspond to Cashier's expected convention, specifically, methods should be prefixed with `handle` and the "camel case" name of the Stripe webhook you wish to handle. For example, if you wish to handle the `invoice.payment_succeeded` webhook, you should add a `handleInvoicePaymentSucceeded` method to the controller:

    <?php

    namespace App\Http\Controllers;

    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * Handle a Stripe webhook.
         *
         * @param  array  $payload
         * @return Response
         */
        public function handleInvoicePaymentSucceeded($payload)
        {
            // Handle The Event
        }
    }

A continuación, defina una ruta a su controlador de cajero dentro de su archivo `routes/web.php`:
> > Next, define a route to your Cashier controller within your `routes/web.php` file:

    Route::post(
        'stripe/webhook',
        '\App\Http\Controllers\WebhookController@handleWebhook'
    );

<a name="handling-failed-subscriptions"></a>
### Suscripciones fallidas
### Failed Subscriptions

¿Qué pasa si la tarjeta de crédito de un cliente expira? Sin preocupaciones: Cashier incluye un controlador Webhook que puede cancelar fácilmente la suscripción del cliente por usted. Como se señaló anteriormente, todo lo que necesita hacer es señalar una ruta al controlador:
> > What if a customer's credit card expires? No worries - Cashier includes a Webhook controller that can easily cancel the customer's subscription for you. As noted above, all you need to do is point a route to the controller:

    Route::post(
        'stripe/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

¡Eso es! Los pagos fallidos serán capturados y manejados por el controlador. El controlador cancelará la suscripción del cliente cuando Stripe determine que la suscripción ha fallado (normalmente después de tres intentos fallidos de pago).
> > That's it! Failed payments will be captured and handled by the controller. The controller will cancel the customer's subscription when Stripe determines the subscription has failed (normally after three failed payment attempts).

<a name="handling-braintree-webhooks"></a>
## Manejo de Webhooks de Braintree
## Handling Braintree Webhooks

Tanto Stripe como Braintree pueden notificar a su aplicación una variedad de eventos a través de webhooks. Para manejar webhooks de Braintree, defina una ruta que apunte al controlador webhook de Cashier. Este controlador manejará todas las solicitudes entrantes de webhook y las enviará al método de controlador apropiado:
> > Both Stripe and Braintree can notify your application of a variety of events via webhooks. To handle Braintree webhooks, define a route that points to Cashier's webhook controller. This controller will handle all incoming webhook requests and dispatch them to the proper controller method:

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

> {note} Una vez que haya registrado su ruta, asegúrese de configurar la URL del webhook en la configuración del panel de control de Braintree.
> > > {note} Once you have registered your route, be sure to configure the webhook URL in your Braintree control panel settings.

Por defecto, este controlador manejará automáticamente la cancelación de suscripciones que tienen demasiados cargos fallidos (según lo definido por su configuración de Braintree); sin embargo, como pronto descubriremos, puede extender este controlador para manejar cualquier evento webhook que desee.
> > By default, this controller will automatically handle cancelling subscriptions that have too many failed charges (as defined by your Braintree settings); however, as we'll soon discover, you can extend this controller to handle any webhook event you like.

#### Webhooks y protección CSRF
#### Webhooks & CSRF Protection

Como los webhooks de Braintree deben eludir la [protección CSRF] de Laravel (/docs/{{version}}/csrf), asegúrese de incluir el URI como excepción en su middleware `VerifyCsrfToken` o haga una lista de la ruta fuera del grupo middleware `web`:
> > Since Braintree webhooks need to bypass Laravel's [CSRF protection](/docs/{{version}}/csrf), be sure to list the URI as an exception in your `VerifyCsrfToken` middleware or list the route outside of the `web` middleware group:

    protected $except = [
        'braintree/*',
    ];

<a name="defining-braintree-webhook-event-handlers"></a>
### Definición de manejadores de eventos Webhook
### Defining Webhook Event Handlers

El cajero maneja automáticamente la cancelación de suscripción en los cargos fallidos, pero si tiene eventos webhook Braintree adicionales que le gustaría manejar, extienda el controlador Webhook. Los nombres de sus métodos deben corresponder a la convención esperada del Cajero, específicamente, los métodos deben tener el prefijo `handle` y el nombre de "camel case" del webhook de Braintree que desea manejar. Por ejemplo, si desea manejar el webhook `dispute_opened`, debe agregar un método `handleDisputeOpened` al controlador:
> > Cashier automatically handles subscription cancellation on failed charges, but if you have additional Braintree webhook events you would like to handle, extend the Webhook controller. Your method names should correspond to Cashier's expected convention, specifically, methods should be prefixed with `handle` and the "camel case" name of the Braintree webhook you wish to handle. For example, if you wish to handle the `dispute_opened` webhook, you should add a `handleDisputeOpened` method to the controller:

    <?php

    namespace App\Http\Controllers;

    use Braintree\WebhookNotification;
    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * Handle a Braintree webhook.
         *
         * @param  WebhookNotification  $webhook
         * @return Response
         */
        public function handleDisputeOpened(WebhookNotification $notification)
        {
            // Handle The Event
        }
    }

<a name="handling-braintree-failed-subscriptions"></a>
### Suscripciones fallidas
### Failed Subscriptions

¿Qué pasa si la tarjeta de crédito de un cliente expira? Sin preocupaciones: Cashier incluye un controlador Webhook que puede cancelar fácilmente la suscripción del cliente por usted. Simplemente señale una ruta al controlador:
> > What if a customer's credit card expires? No worries - Cashier includes a Webhook controller that can easily cancel the customer's subscription for you. Just point a route to the controller:

    Route::post(
        'braintree/webhook',
        '\Laravel\Cashier\Http\Controllers\WebhookController@handleWebhook'
    );

¡Eso es! Los pagos fallidos serán capturados y manejados por el controlador. El controlador cancelará la suscripción del cliente cuando Braintree determine que la suscripción ha fallado (normalmente después de tres intentos fallidos de pago). No lo olvide: deberá configurar el URI de webhook en la configuración del panel de control de Braintree.
> > That's it! Failed payments will be captured and handled by the controller. The controller will cancel the customer's subscription when Braintree determines the subscription has failed (normally after three failed payment attempts). Don't forget: you will need to configure the webhook URI in your Braintree control panel settings.

<a name="single-charges"></a>
## Cargos individuales
## Single Charges

### Cargo simple
### Simple Charge

> {note} Al usar Stripe, el método `charge` acepta la cantidad que le gustaría cobrar en el **denominador más bajo de la moneda utilizada por su aplicación**. Sin embargo, al usar Braintree, debe pasar el monto total en dólares al método de `charge`:
> > > {note} When using Stripe, the `charge` method accepts the amount you would like to charge in the **lowest denominator of the currency used by your application**. However, when using Braintree, you should pass the full dollar amount to the `charge` method:

Si desea hacer un cargo "único" contra la tarjeta de crédito de un cliente suscrito, puede usar el método `charge` en una instancia de modelo facturable.
> > If you would like to make a "one off" charge against a subscribed customer's credit card, you may use the `charge` method on a billable model instance.

    // Stripe Accepts Charges In Cents...
    $user->charge(100);

    // Braintree Accepts Charges In Dollars...
    $user->charge(1);

El método `charge` acepta una matriz como segundo argumento, lo que le permite pasar cualquier opción que desee a la creación subyacente de carga Stripe / Braintree. Consulte la documentación de Stripe o Braintree con respecto a las opciones disponibles para crear cargos:
> > The `charge` method accepts an array as its second argument, allowing you to pass any options you wish to the underlying Stripe / Braintree charge creation. Consult the Stripe or Braintree documentation regarding the options available to you when creating charges:

    $user->charge(100, [
        'custom_option' => $value,
    ]);

El método `charge` lanzará una excepción si la carga falla. Si la carga es exitosa, la respuesta completa de Stripe / Braintree será devuelta por el método:
> > The `charge` method will throw an exception if the charge fails. If the charge is successful, the full Stripe / Braintree response will be returned from the method:

    try {
        $response = $user->charge(100);
    } catch (Exception $e) {
        //
    }

### Cargo con factura
### Charge With Invoice

En ocasiones, es posible que tenga que realizar un cargo único, pero también generar una factura por el cargo para que pueda ofrecer un recibo en PDF a su cliente. El método `invoiceFor` te permite hacer eso. Por ejemplo, facturemos al cliente $ 5.00 por una "Tarifa única":
> > Sometimes you may need to make a one-time charge but also generate an invoice for the charge so that you may offer a PDF receipt to your customer. The `invoiceFor` method lets you do just that. For example, let's invoice the customer $5.00 for a "One Time Fee":

    // Stripe Accepts Charges In Cents...
    $user->invoiceFor('One Time Fee', 500);

    // Braintree Accepts Charges In Dollars...
    $user->invoiceFor('One Time Fee', 5);

La factura se cargará inmediatamente contra la tarjeta de crédito del usuario. El método `invoiceFor` también acepta una matriz como su tercer argumento, lo que le permite pasar cualquier opción que desee a la creación de cargos subyacente de Stripe / Braintree:
> > The invoice will be charged immediately against the user's credit card. The `invoiceFor` method also accepts an array as its third argument, allowing you to pass any options you wish to the underlying Stripe / Braintree charge creation:

    $user->invoiceFor('One Time Fee', 500, [
        'custom-option' => $value,
    ]);

Si está utilizando Braintree como su proveedor de facturación, debe incluir una opción `description` cuando llame al método `invoiceFor`:
> > If you are using Braintree as your billing provider, you must include a `description` option when calling the `invoiceFor` method:

    $user->invoiceFor('One Time Fee', 500, [
        'description' => 'your invoice description here',
    ]);


> {note} El método `invoiceFor` creará una factura Stripe que reintentará los intentos de facturación fallidos. Si no desea que las facturas reintenten los cargos fallidos, deberá cerrarlos con la API de Stripe después del primer cargo fallido.
> > > {note} The `invoiceFor` method will create a Stripe invoice which will retry failed billing attempts. If you do not want invoices to retry failed charges, you will need to close them using the Stripe API after the first failed charge.

<a name="invoices"></a>
## Facturas
## Invoices

Puede recuperar fácilmente una matriz de facturas de un modelo facturable utilizando el método `invoices`:
> > You may easily retrieve an array of a billable model's invoices using the `invoices` method:

    $invoices = $user->invoices();

    // Include pending invoices in the results...
    $invoices = $user->invoicesIncludingPending();

Al enumerar las facturas para el cliente, puede usar los métodos auxiliares de la factura para mostrar la información relevante de la factura. Por ejemplo, puede desear enumerar cada factura en una tabla, lo que permite al usuario descargar fácilmente cualquiera de ellas:
> > When listing the invoices for the customer, you may use the invoice's helper methods to display the relevant invoice information. For example, you may wish to list every invoice in a table, allowing the user to easily download any of them:

    <table>
        @foreach ($invoices as $invoice)
            <tr>
                <td>{{ $invoice->date()->toFormattedDateString() }}</td>
                <td>{{ $invoice->total() }}</td>
                <td><a href="/user/invoice/{{ $invoice->id }}">Download</a></td>
            </tr>
        @endforeach
    </table>

<a name="generating-invoice-pdfs"></a>
### Generación de PDF de facturas
### Generating Invoice PDFs

Desde dentro de una ruta o controlador, use el método `downloadInvoice` para generar una descarga de PDF de la factura. Este método generará automáticamente la respuesta HTTP adecuada para enviar la descarga al navegador:
> > From within a route or controller, use the `downloadInvoice` method to generate a PDF download of the invoice. This method will automatically generate the proper HTTP response to send the download to the browser:

    use Illuminate\Http\Request;

    Route::get('user/invoice/{invoice}', function (Request $request, $invoiceId) {
        return $request->user()->downloadInvoice($invoiceId, [
            'vendor'  => 'Your Company',
            'product' => 'Your Product',
        ]);
    });
