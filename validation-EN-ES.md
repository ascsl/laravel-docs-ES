# Validación : Validation

- [Introduction](#introduction)
- [Validation Quickstart](#validation-quickstart)
    - [Defining The Routes](#quick-defining-the-routes)
    - [Creating The Controller](#quick-creating-the-controller)
    - [Writing The Validation Logic](#quick-writing-the-validation-logic)
    - [Displaying The Validation Errors](#quick-displaying-the-validation-errors)
    - [A Note On Optional Fields](#a-note-on-optional-fields)
- [Form Request Validation](#form-request-validation)
    - [Creating Form Requests](#creating-form-requests)
    - [Authorizing Form Requests](#authorizing-form-requests)
    - [Customizing The Error Messages](#customizing-the-error-messages)
- [Manually Creating Validators](#manually-creating-validators)
    - [Automatic Redirection](#automatic-redirection)
    - [Named Error Bags](#named-error-bags)
    - [After Validation Hook](#after-validation-hook)
- [Working With Error Messages](#working-with-error-messages)
    - [Custom Error Messages](#custom-error-messages)
- [Available Validation Rules](#available-validation-rules)
- [Conditionally Adding Rules](#conditionally-adding-rules)
- [Validating Arrays](#validating-arrays)
- [Custom Validation Rules](#custom-validation-rules)
    - [Using Rule Objects](#using-rule-objects)
    - [Using Closures](#using-closures)
    - [Using Extensions](#using-extensions)

<a name="introduction"></a>
## Introducción : Introduction

Laravel proporciona varios enfoques diferentes para validar los datos entrantes de su aplicación. Por defecto, la clase de controlador base de Laravel usa un rasgo `ValidatesRequests` que proporciona un método conveniente para validar la solicitud HTTP entrante con una variedad de poderosas reglas de validación.
> > Laravel provides several different approaches to validate your application's incoming data. By default, Laravel's base controller class uses a `ValidatesRequests` trait which provides a convenient method to validate incoming HTTP request with a variety of powerful validation rules.

<a name="validation-quickstart"></a>
## Inicio rápido de validación : Validation Quickstart

Para conocer las poderosas funciones de validación de Laravel, veamos un ejemplo completo de validación de un formulario y visualización de los mensajes de error al usuario.
> > To learn about Laravel's powerful validation features, let's look at a complete example of validating a form and displaying the error messages back to the user.

<a name="quick-defining-the-routes"></a>
### Definir las rutas : Defining The Routes

Primero, supongamos que tenemos las siguientes rutas definidas en nuestro archivo `routes/web.php`:
> > First, let's assume we have the following routes defined in our `routes/web.php` file:

    Route::get('post/create', 'PostController@create');

    Route::post('post', 'PostController@store');

Por supuesto, la ruta `GET` mostrará un formulario para que el usuario cree una nueva publicación de blog, mientras que la ruta `POST` almacenará la nueva publicación de blog en la base de datos.
> > Of course, the `GET` route will display a form for the user to create a new blog post, while the `POST` route will store the new blog post in the database.

<a name="quick-creating-the-controller"></a>
### Creando el controlador : Creating The Controller

A continuación, echemos un vistazo a un controlador simple que maneja estas rutas. Dejaremos el método `store` vacío por ahora:
> > Next, let's take a look at a simple controller that handles these routes. We'll leave the `store` method empty for now:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Show the form to create a new blog post.
         *
         * @return Response
         */
        public function create()
        {
            return view('post.create');
        }

        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate and store the blog post...
        }
    }

<a name="quick-writing-the-validation-logic"></a>
### Escribir la lógica de validación : Writing The Validation Logic

Ahora estamos listos para completar nuestro método `store` con la lógica para validar la nueva publicación de blog. Para hacer esto, usaremos el método `validate` provisto por el objeto `Illuminate\Http\Request`. Si se aprueban las reglas de validación, su código seguirá ejecutándose normalmente; sin embargo, si falla la validación, se lanzará una excepción y la respuesta de error correcta se enviará automáticamente al usuario. En el caso de una solicitud HTTP tradicional, se generará una respuesta de redireccionamiento, mientras que se enviará una respuesta JSON para las solicitudes AJAX.
> > Now we are ready to fill in our `store` method with the logic to validate the new blog post. To do this, we will use the `validate` method provided by the `Illuminate\Http\Request` object. If the validation rules pass, your code will keep executing normally; however, if validation fails, an exception will be thrown and the proper error response will automatically be sent back to the user. In the case of a traditional HTTP request, a redirect response will be generated, while a JSON response will be sent for AJAX requests.

Para obtener una mejor comprensión del método `validate`, volvamos al método `store`:
> > To get a better understanding of the `validate` method, let's jump back into the `store` method:

    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $validatedData = $request->validate([
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        // The blog post is valid...
    }

Como puede ver, pasamos las reglas de validación deseadas al método `validate`. De nuevo, si la validación falla, la respuesta apropiada se generará automáticamente. Si la validación pasa, nuestro controlador continuará ejecutándose normalmente.
> > As you can see, we pass the desired validation rules into the `validate` method. Again, if the validation fails, the proper response will automatically be generated. If the validation passes, our controller will continue executing normally.

#### Detener en el primer fallo de validación : Stopping On First Validation Failure

A veces puede dejar de ejecutar reglas de validación en un atributo después del primer fallo de validación. Para hacerlo, asigne la regla `bail` al atributo:
> > Sometimes you may wish to stop running validation rules on an attribute after the first validation failure. To do so, assign the `bail` rule to the attribute:

    $request->validate([
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
    ]);

En este ejemplo, si la regla `unique` en el atributo` title` falla, la regla `max` no se comprobará. Las reglas se validarán en el orden en que se asignaron.
> > In this example, if the `unique` rule on the `title` attribute fails, the `max` rule will not be checked. Rules will be validated in the order they are assigned.

#### Una nota sobre los atributos anidados : A Note On Nested Attributes

Si su solicitud HTTP contiene parámetros "anidados", puede especificarlos en sus reglas de validación utilizando la sintaxis del "punto":
> > If your HTTP request contains "nested" parameters, you may specify them in your validation rules using "dot" syntax:

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>
### Visualización de los errores de validación : Displaying The Validation Errors

Entonces, ¿qué pasa si los parámetros de solicitud entrantes no pasan las reglas de validación dadas? Como se mencionó anteriormente, Laravel automáticamente redirigirá al usuario a su ubicación anterior. Además, todos los errores de validación serán [flasheados automáticamente a la sesión](/docs/{{version}}/session#flash-data).
> > So, what if the incoming request parameters do not pass the given validation rules? As mentioned previously, Laravel will automatically redirect the user back to their previous location. In addition, all of the validation errors will automatically be [flashed to the session](/docs/{{version}}/session#flash-data).

De nuevo, observe que no teníamos que vincular explícitamente los mensajes de error a la vista en nuestra ruta `GET`. Esto se debe a que Laravel verificará los errores en los datos de la sesión y los vinculará automáticamente a la vista si están disponibles. La variable `$errors` será una instancia de `Illuminate\Support\MessageBag`. Para obtener más información sobre cómo trabajar con este objeto, [consulte su documentación](#working-with-error-messages).
> > Again, notice that we did not have to explicitly bind the error messages to the view in our `GET` route. This is because Laravel will check for errors in the session data, and automatically bind them to the view if they are available. The `$errors` variable will be an instance of `Illuminate\Support\MessageBag`. For more information on working with this object, [check out its documentation](#working-with-error-messages).

> {tip} La variable `$errors` está vinculada a la vista por el middleware `Illuminate\View\Middleware\ShareErrorsFromSession`, que es proporcionado por el grupo de middleware `web`. **Cuando se aplica este middleware, siempre estará disponible una variable `$errors` en sus vistas**, lo que le permite asumir convenientemente que la variable `$errors` está siempre definida y puede usarse con seguridad.
> > > {tip} The `$errors` variable is bound to the view by the `Illuminate\View\Middleware\ShareErrorsFromSession` middleware, which is provided by the `web` middleware group. **When this middleware is applied an `$errors` variable will always be available in your views**, allowing you to conveniently assume the `$errors` variable is always defined and can be safely used.

Entonces, en nuestro ejemplo, el usuario será redirigido al método `create` de nuestro controlador cuando la validación falla, lo que nos permite mostrar los mensajes de error en la vista:
> > So, in our example, the user will be redirected to our controller's `create` method when validation fails, allowing us to display the error messages in the view:

    <!-- /resources/views/post/create.blade.php -->

    <h1>Create Post</h1>

    @if ($errors->any())
        <div class="alert alert-danger">
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    <!-- Create Post Form -->

<a name="a-note-on-optional-fields"></a>
### Una nota sobre campos opcionales : A Note On Optional Fields

Por defecto, Laravel incluye el middleware `TrimStrings` y `ConvertEmptyStringsToNull` en la pila de middleware global de su aplicación. Estos middleware se enumeran en la pila por la clase `App\Http\Kernel`. Debido a esto, a menudo tendrá que marcar sus campos de solicitud "opcionales" como `nullable` si no desea que el validador considere que los valores `null` no son válidos. Por ejemplo:
> > By default, Laravel includes the `TrimStrings` and `ConvertEmptyStringsToNull` middleware in your application's global middleware stack. These middleware are listed in the stack by the `App\Http\Kernel` class. Because of this, you will often need to mark your "optional" request fields as `nullable` if you do not want the validator to consider `null` values as invalid. For example:

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

En este ejemplo, estamos especificando que el campo `publish_at` puede ser `null` o una representación de fecha válida. Si el modificador `nullable` no se agrega a la definición de la regla, el validador consideraría `null` una fecha inválida.
> > In this example, we are specifying that the `publish_at` field may be either `null` or a valid date representation. If the `nullable` modifier is not added to the rule definition, the validator would consider `null` an invalid date.

<a name="quick-ajax-requests-and-validation"></a>
#### Solicitudes y Validación de AJAX : AJAX Requests & Validation

En este ejemplo, usamos una forma tradicional para enviar datos a la aplicación. Sin embargo, muchas aplicaciones usan solicitudes AJAX. Al utilizar el método `validate` durante una solicitud AJAX, Laravel no generará una respuesta de redirección. En cambio, Laravel genera una respuesta JSON que contiene todos los errores de validación. Esta respuesta JSON se enviará con un código de estado HTTP 422.
> > In this example, we used a traditional form to send data to the application. However, many applications use AJAX requests. When using the `validate` method during an AJAX request, Laravel will not generate a redirect response. Instead, Laravel generates a JSON response containing all of the validation errors. This JSON response will be sent with a 422 HTTP status code.

<a name="form-request-validation"></a>
## Validación de solicitud de formulario : Form Request Validation

<a name="creating-form-requests"></a>
### Creating Form Requests

Para escenarios de validación más complejos, es posible que desee crear una "solicitud de formulario". Las solicitudes de formulario son clases de solicitud personalizadas que contienen lógica de validación. Para crear una clase de solicitud de formulario, use el comando Artisan CLI `make:request`:
> > For more complex validation scenarios, you may wish to create a "form request". Form requests are custom request classes that contain validation logic. To create a form request class, use the `make:request` Artisan CLI command:

    php artisan make:request StoreBlogPost

La clase generada se colocará en el directorio `app/Http/Requests`. Si este directorio no existe, se creará cuando ejecute el comando `make:request`. Agreguemos algunas reglas de validación al método `rules`:
> > The generated class will be placed in the `app/Http/Requests` directory. If this directory does not exist, it will be created when you run the `make:request` command. Let's add a few validation rules to the `rules` method:

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

> {tip} Puede indicar cualquier dependencia que necesite dentro de la firma del método `rules`. Se resolverán automáticamente a través del [contenedor de servicios](/docs/{{version}}/container) de Laravel.
> > > {tip} You may type-hint any dependencies you need within the `rules` method's signature. They will automatically be resolved via the Laravel [service container](/docs/{{version}}/container).

Entonces, ¿cómo se evalúan las reglas de validación? Todo lo que necesita hacer es insinuar la solicitud en su método de controlador. La solicitud de formulario entrante se valida antes de llamar al método del controlador, lo que significa que no necesita saturar su controlador con ninguna lógica de validación:
> > So, how are the validation rules evaluated? All you need to do is type-hint the request on your controller method. The incoming form request is validated before the controller method is called, meaning you do not need to clutter your controller with any validation logic:

    /**
     * Store the incoming blog post.
     *
     * @param  StoreBlogPost  $request
     * @return Response
     */
    public function store(StoreBlogPost $request)
    {
        // The incoming request is valid...

        // Retrieve the validated input data...
        $validated = $request->validated();
    }

Si la validación falla, se generará una respuesta de redireccionamiento para enviar al usuario a su ubicación anterior. Los errores también se mostrarán en la sesión para que estén disponibles para su visualización. Si la solicitud fue una solicitud AJAX, se devolverá al usuario una respuesta HTTP con un código de estado 422, incluida una representación JSON de los errores de validación.
> > If validation fails, a redirect response will be generated to send the user back to their previous location. The errors will also be flashed to the session so they are available for display. If the request was an AJAX request, a HTTP response with a 422 status code will be returned to the user including a JSON representation of the validation errors.

#### Agregar después de enganches a solicitudes de formulario : Adding After Hooks To Form Requests

Si desea agregar un enlace "después" a una solicitud de formulario, puede usar el método `withValidator`. Este método recibe el validador totalmente construido, lo que le permite llamar a cualquiera de sus métodos antes de que las reglas de validación sean realmente evaluadas:
> > If you would like to add an "after" hook to a form request, you may use the `withValidator` method. This method receives the fully constructed validator, allowing you to call any of its methods before the validation rules are actually evaluated:

    /**
     * Configure the validator instance.
     *
     * @param  \Illuminate\Validation\Validator  $validator
     * @return void
     */
    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add('field', 'Something is wrong with this field!');
            }
        });
    }

<a name="authorizing-form-requests"></a>
### Autorización de solicitudes de formularios : Authorizing Form Requests

La clase de solicitud de formulario también contiene un método `authorize`. Dentro de este método, puede verificar si el usuario autenticado en realidad tiene la autoridad para actualizar un recurso dado. Por ejemplo, puede determinar si un usuario posee realmente un comentario de blog que intentan actualizar:
> > The form request class also contains an `authorize` method. Within this method, you may check if the authenticated user actually has the authority to update a given resource. For example, you may determine if a user actually owns a blog comment they are attempting to update:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        $comment = Comment::find($this->route('comment'));

        return $comment && $this->user()->can('update', $comment);
    }

Como todas las solicitudes de formulario extienden la clase de solicitud básica de Laravel, podemos utilizar el método `user` para acceder al usuario autenticado actualmente. También tenga en cuenta la llamada al método `route` en el ejemplo anterior. Este método le otorga acceso a los parámetros de URI definidos en la ruta que se llama, como el parámetro `{comment}` en el siguiente ejemplo:
> > Since all form requests extend the base Laravel request class, we may use the `user` method to access the currently authenticated user. Also note the call to the `route` method in the example above. This method grants you access to the URI parameters defined on the route being called, such as the `{comment}` parameter in the example below:

    Route::post('comment/{comment}');

Si el método `authorize` devuelve `false`, se devolverá automáticamente una respuesta HTTP con un código de estado 403 y su método de controlador no se ejecutará.
> > If the `authorize` method returns `false`, a HTTP response with a 403 status code will automatically be returned and your controller method will not execute.

Si planea tener una lógica de autorización en otra parte de su aplicación, devuelva `true` del método `authorize`:
> > If you plan to have authorization logic in another part of your application, return `true` from the `authorize` method:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

> {tip} Puede indicar cualquier dependencia que necesite dentro de la firma del método `authorize`. Se resolverán automáticamente a través del [contenedor de servicios](/docs/{{version}}/container) de Laravel.
> > > {tip} You may type-hint any dependencies you need within the `authorize` method's signature. They will automatically be resolved via the Laravel [service container](/docs/{{version}}/container).

<a name="customizing-the-error-messages"></a>
### Personalizando los mensajes de error : Customizing The Error Messages

Puede personalizar los mensajes de error utilizados por la solicitud de formulario anulando el método `messages`. Este método debería devolver un array de pares de atributos / reglas y sus correspondientes mensajes de error:
> > You may customize the error messages used by the form request by overriding the `messages` method. This method should return an array of attribute / rule pairs and their corresponding error messages:

    /**
     * Get the error messages for the defined validation rules.
     *
     * @return array
     */
    public function messages()
    {
        return [
            'title.required' => 'A title is required',
            'body.required'  => 'A message is required',
        ];
    }

<a name="manually-creating-validators"></a>
## Creación manual de validadores : Manually Creating Validators

Si no desea utilizar el método `validate` en la solicitud, puede crear una instancia de validador manualmente usando la [fachada](/docs/{{version}}/facades) `Validator`. El método `make` de la fachada genera una nueva instancia de validador:
> > If you do not want to use the `validate` method on the request, you may create a validator instance manually using the `Validator` [facade](/docs/{{version}}/facades). The `make` method on the facade generates a new validator instance:

    <?php

    namespace App\Http\Controllers;

    use Validator;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $validator = Validator::make($request->all(), [
                'title' => 'required|unique:posts|max:255',
                'body' => 'required',
            ]);

            if ($validator->fails()) {
                return redirect('post/create')
                            ->withErrors($validator)
                            ->withInput();
            }

            // Store the blog post...
        }
    }

El primer argumento pasado al método `make` es la información bajo validación. El segundo argumento es las reglas de validación que se deben aplicar a los datos.
> > The first argument passed to the `make` method is the data under validation. The second argument is the validation rules that should be applied to the data.

Después de verificar si la validación de la solicitud falló, puede usar el método `withErrors` para mostrar los mensajes de error en la sesión. Al utilizar este método, la variable `$errors` se compartirá automáticamente con sus vistas después de la redirección, lo que le permitirá mostrarlas fácilmente al usuario. El método `withErrors` acepta un validador, un `MessageBag`, o un array `PHP`.
> > After checking if the request validation failed, you may use the `withErrors` method to flash the error messages to the session. When using this method, the `$errors` variable will automatically be shared with your views after redirection, allowing you to easily display them back to the user. The `withErrors` method accepts a validator, a `MessageBag`, or a PHP `array`.

<a name="automatic-redirection"></a>
### Redirección automática : Automatic Redirection

Si desea crear una instancia de validador de forma manual pero aún así aprovechar la redirección automática que ofrece el método `validate` de las solicitudes, puede llamar al método `validate` en una instancia de validador existente. Si la validación falla, el usuario será redireccionado automáticamente o, en el caso de una solicitud AJAX, se devolverá una respuesta JSON:
> > If you would like to create a validator instance manually but still take advantage of the automatic redirection offered by the requests's `validate` method, you may call the `validate` method on an existing validator instance. If validation fails, the user will automatically be redirected or, in the case of an AJAX request, a JSON response will be returned:

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validate();

<a name="named-error-bags"></a>
### Bolsas de error con nombre : Named Error Bags

Si tiene varios formularios en una sola página, le recomendamos que nombre el `MessageBag` de los errores, lo que le permite recuperar los mensajes de error de un formulario específico. Pase un nombre como segundo argumento a `withErrors`:
> > If you have multiple forms on a single page, you may wish to name the `MessageBag` of errors, allowing you to retrieve the error messages for a specific form. Pass a name as the second argument to `withErrors`:

    return redirect('register')
                ->withErrors($validator, 'login');

A continuación, puede acceder a la instancia denominada `MessageBag` de la variable `$errors`:
> > You may then access the named `MessageBag` instance from the `$errors` variable:

    {{ $errors->login->first('email') }}

<a name="after-validation-hook"></a>
### Después del gancho de validación : After Validation Hook

El validador también le permite adjuntar devoluciones de llamadas para que se ejecuten después de que se complete la validación. Esto le permite realizar fácilmente una validación adicional e incluso agregar más mensajes de error a la colección de mensajes. Para comenzar, use el método `after` en una instancia de validador:
> > The validator also allows you to attach callbacks to be run after validation is completed. This allows you to easily perform further validation and even add more error messages to the message collection. To get started, use the `after` method on a validator instance:

    $validator = Validator::make(...);

    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });

    if ($validator->fails()) {
        //
    }

<a name="working-with-error-messages"></a>
## Trabajar con mensajes de error : Working With Error Messages

Después de llamar al método `errors` en una instancia `Validator`, recibirá una instancia `Illuminate\Support\MessageBag`, que tiene una variedad de métodos convenientes para trabajar con mensajes de error. La variable `$errors` que está automáticamente disponible para todas las vistas también es una instancia de la clase `MessageBag`.
> > After calling the `errors` method on a `Validator` instance, you will receive an `Illuminate\Support\MessageBag` instance, which has a variety of convenient methods for working with error messages. The `$errors` variable that is automatically made available to all views is also an instance of the `MessageBag` class.

#### Recuperando el primer mensaje de error para un campo : Retrieving The First Error Message For A Field

Para recuperar el primer mensaje de error para un campo dado, use el método `first`:
> > To retrieve the first error message for a given field, use the `first` method:

    $errors = $validator->errors();

    echo $errors->first('email');

#### Recuperación de todos los mensajes de error para un campo : Retrieving All Error Messages For A Field

Si necesita recuperar un array con todos los mensajes para un campo dado, use el método `get`:
> > If you need to retrieve an array of all the messages for a given field, use the `get` method:

    foreach ($errors->get('email') as $message) {
        //
    }

Si está validando un array de campos de formulario, puede recuperar todos los mensajes para cada uno de los elementos del array utilizando el carácter `*`:
> > If you are validating an array form field, you may retrieve all of the messages for each of the array elements using the `*` character:

    foreach ($errors->get('attachments.*') as $message) {
        //
    }

#### Recuperación de todos los mensajes de error para todos los campos : Retrieving All Error Messages For All Fields

Para recuperar un array con todos los mensajes para todos los campos, use el método `all`:
> > To retrieve an array of all messages for all fields, use the `all` method:

    foreach ($errors->all() as $message) {
        //
    }

#### Determinar si existen mensajes para un campo : Determining If Messages Exist For A Field

El método `has` se puede usar para determinar si existe algún mensaje de error para un campo dado:
> > The `has` method may be used to determine if any error messages exist for a given field:

    if ($errors->has('email')) {
        //
    }

<a name="custom-error-messages"></a>
### Mensajes de error personalizados : Custom Error Messages

Si es necesario, puede usar mensajes de error personalizados para la validación en lugar de los valores predeterminados. Hay varias formas de especificar mensajes personalizados. Primero, puede pasar los mensajes personalizados como tercer argumento del método `Validator::make`:
> > If needed, you may use custom error messages for validation instead of the defaults. There are several ways to specify custom messages. First, you may pass the custom messages as the third argument to the `Validator::make` method:

    $messages = [
        'required' => 'The :attribute field is required.',
    ];

    $validator = Validator::make($input, $rules, $messages);

En este ejemplo, el marcador de posición `:attribute` será reemplazado por el nombre real del campo bajo validación. También puede utilizar otros marcadores de posición en mensajes de validación. Por ejemplo:
> > In this example, the `:attribute` place-holder will be replaced by the actual name of the field under validation. You may also utilize other place-holders in validation messages. For example:

    $messages = [
        'same'    => 'The :attribute and :other must match.',
        'size'    => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute value :input is not between :min - :max.',
        'in'      => 'The :attribute must be one of the following types: :values',
    ];

#### Especificando un mensaje personalizado para un atributo determinado : Specifying A Custom Message For A Given Attribute

En ocasiones, es posible que desee especificar un mensaje de error personalizado solo para un campo específico. Puedes hacerlo usando la notación "punto". Especifique primero el nombre del atributo, seguido de la regla:
> > Sometimes you may wish to specify a custom error messages only for a specific field. You may do so using "dot" notation. Specify the attribute's name first, followed by the rule:

    $messages = [
        'email.required' => 'We need to know your e-mail address!',
    ];

<a name="localization"></a>
#### Especificando mensajes personalizados en archivos de idioma : Specifying Custom Messages In Language Files

En la mayoría de los casos, probablemente especifique sus mensajes personalizados en un archivo de idioma en lugar de pasarlos directamente al `Validator`. Para hacerlo, agregue sus mensajes al array `custom` en el archivo de idioma `resources/lang/xx/validation.php`.
> > In most cases, you will probably specify your custom messages in a language file instead of passing them directly to the `Validator`. To do so, add your messages to `custom` array in the `resources/lang/xx/validation.php` language file.

    'custom' => [
        'email' => [
            'required' => 'We need to know your e-mail address!',
        ],
    ],

#### Especificando atributos personalizados en archivos de idioma : Specifying Custom Attributes In Language Files

Si desea que la porción `:attribute` de su mensaje de validación se sustituya por un nombre de atributo personalizado, puede especificar el nombre personalizado en el array `attributes` del archivo de idioma `resources/lang/xx/validation.php` :
> > If you would like the `:attribute` portion of your validation message to be replaced with a custom attribute name, you may specify the custom name in the `attributes` array of your `resources/lang/xx/validation.php` language file:

    'attributes' => [
        'email' => 'email address',
    ],

<a name="available-validation-rules"></a>
## Reglas de validación disponibles : Available Validation Rules

A continuación hay una lista de todas las reglas de validación disponibles y su función:
> > Below is a list of all available validation rules and their function:

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

[Accepted](#rule-accepted)
[Active URL](#rule-active-url)
[After (Date)](#rule-after)
[After Or Equal (Date)](#rule-after-or-equal)
[Alpha](#rule-alpha)
[Alpha Dash](#rule-alpha-dash)
[Alpha Numeric](#rule-alpha-num)
[Array](#rule-array)
[Bail](#rule-bail)
[Before (Date)](#rule-before)
[Before Or Equal (Date)](#rule-before-or-equal)
[Between](#rule-between)
[Boolean](#rule-boolean)
[Confirmed](#rule-confirmed)
[Date](#rule-date)
[Date Equals](#rule-date-equals)
[Date Format](#rule-date-format)
[Different](#rule-different)
[Digits](#rule-digits)
[Digits Between](#rule-digits-between)
[Dimensions (Image Files)](#rule-dimensions)
[Distinct](#rule-distinct)
[E-Mail](#rule-email)
[Exists (Database)](#rule-exists)
[File](#rule-file)
[Filled](#rule-filled)
[Greater Than](#rule-gt)
[Greater Than Or Equal](#rule-gte)
[Image (File)](#rule-image)
[In](#rule-in)
[In Array](#rule-in-array)
[Integer](#rule-integer)
[IP Address](#rule-ip)
[JSON](#rule-json)
[Less Than](#rule-lt)
[Less Than Or Equal](#rule-lte)
[Max](#rule-max)
[MIME Types](#rule-mimetypes)
[MIME Type By File Extension](#rule-mimes)
[Min](#rule-min)
[Not In](#rule-not-in)
[Not Regex](#rule-not-regex)
[Nullable](#rule-nullable)
[Numeric](#rule-numeric)
[Present](#rule-present)
[Regular Expression](#rule-regex)
[Required](#rule-required)
[Required If](#rule-required-if)
[Required Unless](#rule-required-unless)
[Required With](#rule-required-with)
[Required With All](#rule-required-with-all)
[Required Without](#rule-required-without)
[Required Without All](#rule-required-without-all)
[Same](#rule-same)
[Size](#rule-size)
[String](#rule-string)
[Timezone](#rule-timezone)
[Unique (Database)](#rule-unique)
[URL](#rule-url)

</div>

<a name="rule-accepted"></a>
#### accepted

El campo bajo validación debe ser _yes_, _on_, _1_, o _true_. Esto es útil para validar la aceptación de los "Términos de servicio".
> > The field under validation must be _yes_, _on_, _1_, or _true_. This is useful for validating "Terms of Service" acceptance.

<a name="rule-active-url"></a>
#### active_url

El campo bajo validación debe tener un registro A o AAAA válido según la función PHP `dns_get_record`.
> > The field under validation must have a valid A or AAAA record according to the `dns_get_record` PHP function.

<a name="rule-after"></a>
#### after:_date_

El campo bajo validación debe ser un valor después de una fecha determinada. Las fechas se pasarán a la función PHP `strtotime`:
> > The field under validation must be a value after a given date. The dates will be passed into the `strtotime` PHP function:

    'start_date' => 'required|date|after:tomorrow'

En lugar de pasar una cadena de fecha para ser evaluada por `strtotime`, puede especificar otro campo para comparar con la fecha:
> > Instead of passing a date string to be evaluated by `strtotime`, you may specify another field to compare against the date:

    'finish_date' => 'required|date|after:start_date'

<a name="rule-after-or-equal"></a>
#### after\_or\_equal:_date_

El campo validado debe ser un valor igual o posterior a la fecha dada. Para obtener más información, consulte la regla [after](#rule-after).
> > The field under validation must be a value after or equal to the given date. For more information, see the [after](#rule-after) rule.

<a name="rule-alpha"></a>
#### alpha

El campo validado debe ser completamente alfabético.
> > The field under validation must be entirely alphabetic characters.

<a name="rule-alpha-dash"></a>
#### alpha_dash

El campo validado puede tener caracteres alfanuméricos, así como guiones y guiones bajos.
> > The field under validation may have alpha-numeric characters, as well as dashes and underscores.

<a name="rule-alpha-num"></a>
#### alpha_num

El campo validado debe ser completamente alfanumérico.
> > The field under validation must be entirely alpha-numeric characters.

<a name="rule-array"></a>
#### array

El campo validado debe ser un `array` PHP.
> > The field under validation must be a PHP `array`.

<a name="rule-bail"></a>
#### bail

Detener las reglas de validación después del primer fallo de validación.
> > Stop running validation rules after the first validation failure.

<a name="rule-before"></a>
#### before:_date_

El campo validado debe ser un valor anterior a la fecha dada. Las fechas se pasarán a la función `strtotime` de PHP.
> > The field under validation must be a value preceding the given date. The dates will be passed into the PHP `strtotime` function.

<a name="rule-before-or-equal"></a>
#### before\_or\_equal:_date_

El campo validado debe ser un valor anterior o igual a la fecha dada. Las fechas se pasarán a la función `strtotime` de PHP.
> > The field under validation must be a value preceding or equal to the given date. The dates will be passed into the PHP `strtotime` function.

<a name="rule-between"></a>
#### between:_min_,_max_

El campo validado debe tener un tamaño entre _min_ y _max_. Las cadenas, los números, los arrays y los archivos se evalúan de la misma manera que la regla [`size`](#rule-size).
> > The field under validation must have a size between the given _min_ and _max_. Strings, numerics, arrays, and files are evaluated in the same fashion as the [`size`](#rule-size) rule.

<a name="rule-boolean"></a>
#### boolean

El campo validado se debe poder convertir como booleano. La entrada aceptada es `true`, `false`, `1`, `0`, `"1"`, y `"0"`.
> > The field under validation must be able to be cast as a boolean. Accepted input are `true`, `false`, `1`, `0`, `"1"`, and `"0"`.

<a name="rule-confirmed"></a>
#### confirmed

El campo validado debe tener un campo coincidente de `foo_confirmation`. Por ejemplo, si el campo bajo validación es `password`, un campo coincidente `password_confirmation` debe estar presente en la entrada.
> > The field under validation must have a matching field of `foo_confirmation`. For example, if the field under validation is `password`, a matching `password_confirmation` field must be present in the input.

<a name="rule-date"></a>
#### date

El campo validado debe ser una fecha válida de acuerdo con la función PHP `strtotime`.
> > The field under validation must be a valid date according to the `strtotime` PHP function.

<a name="rule-date-equals"></a>
#### date_equals:_date_

El campo validado debe ser igual a la fecha dada. Las fechas se pasarán a la función `strtotime` de PHP.
> > The field under validation must be equal to the given date. The dates will be passed into the PHP `strtotime` function.

<a name="rule-date-format"></a>
#### date_format:_format_

El campo validado debe coincidir con el _format_ dado. Debe usar **o** `date` o `date_format` cuando valide un campo, no ambos.
> > The field under validation must match the given _format_. You should use **either** `date` or `date_format` when validating a field, not both.

<a name="rule-different"></a>
#### different:_field_

El campo validado debe tener un valor diferente de _field_.
> > The field under validation must have a different value than _field_.

<a name="rule-digits"></a>
#### digits:_value_

El campo validado debe ser _numeric_ y debe tener una longitud exacta de _value_.
> > The field under validation must be _numeric_ and must have an exact length of _value_.

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

El campo validado debe tener una longitud entre los _min_ y _max_ dados.
> > The field under validation must have a length between the given _min_ and _max_.

<a name="rule-dimensions"></a>
#### dimensions

El archivo validado debe ser una imagen que cumpla con las restricciones de dimensión especificadas por los parámetros de la regla:
> > The file under validation must be an image meeting the dimension constraints as specified by the rule's parameters:

    'avatar' => 'dimensions:min_width=100,min_height=200'

Las restricciones disponibles son: _min\_width_, _max\_width_, _min\_height_, _max\_height_, _width_, _height_, _ratio_.
> > Available constraints are: _min\_width_, _max\_width_, _min\_height_, _max\_height_, _width_, _height_, _ratio_.

Una restricción _ratio_ debe representarse como ancho dividido por altura. Esto se puede especificar mediante una instrucción como `3/2` o una coma flotante como `1.5`:
> > A _ratio_ constraint should be represented as width divided by height. This can be specified either by a statement like `3/2` or a float like `1.5`:

    'avatar' => 'dimensions:ratio=3/2'

Como esta regla requiere varios argumentos, puede usar el método `Rule::dimensions` para construir con fluidez la regla:
> > Since this rule requires several arguments, you may use the `Rule::dimensions` method to fluently construct the rule:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'avatar' => [
            'required',
            Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
        ],
    ]);

<a name="rule-distinct"></a>
#### distinct

Al trabajar con arrays, el campo validado no debe tener ningún valor duplicado.
> > When working with arrays, the field under validation must not have any duplicate values.

    'foo.*.id' => 'distinct'

<a name="rule-email"></a>
#### email

El campo validado se debe formatear como una dirección de correo electrónico.
> > The field under validation must be formatted as an e-mail address.

<a name="rule-exists"></a>
#### exists:_table_,_column_

El campo validado debe existir en una tabla de base de datos dada.
> > The field under validation must exist on a given database table.

#### Uso básico de la regla existente : Basic Usage Of Exists Rule

    'state' => 'exists:states'

Si no se especifica la opción `column`, se usará el nombre del campo.
> > If the `column` option is not specified, the field name will be used.

#### Especificando un nombre de columna personalizado : Specifying A Custom Column Name

    'state' => 'exists:states,abbreviation'

Ocasionalmente, es posible que deba especificar una conexión de base de datos específica para usar en la consulta `exists`. Puede lograr esto anteponiendo el nombre de la conexión al nombre de la tabla usando la sintaxis del "punto":
> > Occasionally, you may need to specify a specific database connection to be used for the `exists` query. You can accomplish this by prepending the connection name to the table name using "dot" syntax:

    'email' => 'exists:connection.staff,email'

Si desea personalizar la consulta ejecutada por la regla de validación, puede usar la clase `Rule` para definir con fluidez la regla. En este ejemplo, también especificaremos las reglas de validación como un array en lugar de usar el carácter `|` para delimitarlas:
> > If you would like to customize the query executed by the validation rule, you may use the `Rule` class to fluently define the rule. In this example, we'll also specify the validation rules as an array instead of using the `|` character to delimit them:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::exists('staff')->where(function ($query) {
                $query->where('account_id', 1);
            }),
        ],
    ]);

<a name="rule-file"></a>
#### file

El campo validado debe ser un archivo cargado correctamente.
> > The field under validation must be a successfully uploaded file.

<a name="rule-filled"></a>
#### filled

El campo validado no debe estar vacío cuando está presente.
> > The field under validation must not be empty when it is present.

<a name="rule-gt"></a>
#### gt:_field_

El campo validado debe ser mayor que el _field_ dado. Los dos campos deben ser del mismo tipo. Cadenas, números, arrays y archivos se evalúan utilizando las mismas convenciones que la regla `size`.
> > The field under validation must be greater than the given _field_. The two fields must be of the same type. Strings, numerics, arrays, and files are evaluated using the same conventions as the `size` rule.

<a name="rule-gte"></a>
#### gte:_field_

El campo validado debe ser mayor o igual que el _field_ dado. Los dos campos deben ser del mismo tipo. Cadenas, números, arrays y archivos se evalúan utilizando las mismas convenciones que la regla `size`.
> > The field under validation must be greater than or equal to the given _field_. The two fields must be of the same type. Strings, numerics, arrays, and files are evaluated using the same conventions as the `size` rule.

<a name="rule-image"></a>
#### image

El archivo validado debe ser una imagen (jpeg, png, bmp, gif o svg)
> > The file under validation must be an image (jpeg, png, bmp, gif, or svg)

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

El campo validado se debe incluir en la lista de valores dada. Como esta regla a menudo requiere que `implode` un array, el método `Rule::in` se puede usar para construir con fluidez la regla:
> > The field under validation must be included in the given list of values. Since this rule often requires you to `implode` an array, the `Rule::in` method may be used to fluently construct the rule:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'zones' => [
            'required',
            Rule::in(['first-zone', 'second-zone']),
        ],
    ]);

<a name="rule-in-array"></a>
#### in_array:_anotherfield_

El campo validado debe existir en los valores de _anotherfield_.
> > The field under validation must exist in _anotherfield_'s values.

<a name="rule-integer"></a>
#### integer

El campo validado debe ser un número entero.
> > The field under validation must be an integer.

<a name="rule-ip"></a>
#### ip

El campo validado debe ser una dirección IP.
> > The field under validation must be an IP address.

#### ipv4

El campo validado debe ser una dirección IPv4.
> > The field under validation must be an IPv4 address.

#### ipv6

El campo validado debe ser una dirección IPv6.
> > The field under validation must be an IPv6 address.

<a name="rule-json"></a>
#### json

El campo validado debe ser una cadena JSON válida.
> > The field under validation must be a valid JSON string.

<a name="rule-lt"></a>
#### lt:_field_

El campo validado debe ser menor que el _field_ dado. Los dos campos deben ser del mismo tipo. Cadenas, números, arrays y archivos se evalúan utilizando las mismas convenciones que la regla `size`.
> > The field under validation must be less than the given _field_. The two fields must be of the same type. Strings, numerics, arrays, and files are evaluated using the same conventions as the `size` rule.

<a name="rule-lte"></a>
#### lte:_field_

El campo validado debe ser menor o igual que el _field_ dado. Los dos campos deben ser del mismo tipo. Cadenas, números, arrays y archivos se evalúan utilizando las mismas convenciones que la regla `size`.
> > The field under validation must be less than or equal to the given _field_. The two fields must be of the same type. Strings, numerics, arrays, and files are evaluated using the same conventions as the `size` rule.

<a name="rule-max"></a>
#### max:_value_

El campo validado debe ser menor o igual a un _value_ máximo. Cadenas, números, arrays y archivos se evalúan de la misma manera que la regla [`size`](#rule-size).
> > The field under validation must be less than or equal to a maximum _value_. Strings, numerics, arrays, and files are evaluated in the same fashion as the [`size`](#rule-size) rule.

<a name="rule-mimetypes"></a>
#### mimetypes:_text/plain_,...

El archivo validado debe coincidir con uno de los tipos MIME dados:
> > The file under validation must match one of the given MIME types:

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

To determine the MIME type of the uploaded file, the file's contents will be read and the framework will attempt to guess the MIME type, which may be different from the client provided MIME type.

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

El archivo validado debe tener un tipo MIME correspondiente a una de las extensiones enumeradas.
> > The file under validation must have a MIME type corresponding to one of the listed extensions.

#### Uso básico de la regla MIME : Basic Usage Of MIME Rule

    'photo' => 'mimes:jpeg,bmp,png'

Aunque solo necesita especificar las extensiones, esta regla en realidad se valida con el tipo MIME del archivo leyendo el contenido del archivo y adivinando su tipo MIME.
> > Even though you only need to specify the extensions, this rule actually validates against the MIME type of the file by reading the file's contents and guessing its MIME type.

Se puede encontrar una lista completa de los tipos MIME y sus extensiones correspondientes en la siguiente ubicación: [https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)
> > A full listing of MIME types and their corresponding extensions may be found at the following location: [https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="rule-min"></a>
#### min:_value_

El campo validado debe tener un _value_ mínimo. Cadenas, números, arrays y archivos se evalúan de la misma manera que la regla [`size`](#rule-size).
> > The field under validation must have a minimum _value_. Strings, numerics, arrays, and files are evaluated in the same fashion as the [`size`](#rule-size) rule.

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

El campo validado no debe incluirse en la lista de valores dada. El método `Rule::notIn` se puede usar para construir con fluidez la regla:
> > The field under validation must not be included in the given list of values. The `Rule::notIn` method may be used to fluently construct the rule:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'toppings' => [
            'required',
            Rule::notIn(['sprinkles', 'cherries']),
        ],
    ]);

<a name="rule-not-regex"></a>
#### not_regex:_pattern_

El campo validado no debe coincidir con la expresión regular dada.
> > The field under validation must not match the given regular expression.

**Nota:** Al usar los patrones `regex` / `not_regex`, puede ser necesario especificar reglas en un array en lugar de usar delimitadores de pipa, especialmente si la expresión regular contiene un carácter de pipa.
> > **Note:** When using the `regex` / `not_regex` patterns, it may be necessary to specify rules in an array instead of using pipe delimiters, especially if the regular expression contains a pipe character.

<a name="rule-nullable"></a>
#### nullable

El campo validado puede ser `null`. Esto es particularmente útil cuando se valida una primitiva como cadenas y enteros que pueden contener valores `null`.
> > The field under validation may be `null`. This is particularly useful when validating primitive such as strings and integers that can contain `null` values.

<a name="rule-numeric"></a>
#### numeric

El campo validado debe ser numérico.
> > The field under validation must be numeric.

<a name="rule-present"></a>
#### present

El campo validado debe estar presente en los datos de entrada, pero puede estar vacío.
> > The field under validation must be present in the input data but can be empty.

<a name="rule-regex"></a>
#### regex:_pattern_

El campo validado debe coincidir con la expresión regular dada.
> > The field under validation must match the given regular expression.

**Nota:** Al usar los patrones `regex` / `not_regex`, puede ser necesario especificar reglas en un array en lugar de usar delimitadores de pipa, especialmente si la expresión regular contiene un carácter de pipa.
> > **Note:** When using the `regex` / `not_regex` patterns, it may be necessary to specify rules in an array instead of using pipe delimiters, especially if the regular expression contains a pipe character.

<a name="rule-required"></a>
#### required

El campo bajo validación debe estar presente en los datos de entrada y no está vacío. Un campo se considera "vacío" si una de las siguientes condiciones es verdadera:
> > The field under validation must be present in the input data and not empty. A field is considered "empty" if one of the following conditions are true:

<div class="content-list" markdown="1">

- El valor es `nulo`.
- El valor es una cadena vacía.
- El valor es un array vacía o un objeto `Contable` vacío.
- El valor es un archivo cargado sin ruta.
> > - The value is `null`.
> > - The value is an empty string.
> > - The value is an empty array or empty `Countable` object.
> > - The value is an uploaded file with no path.

</div>

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,...

El campo validado debe estar presente y no está vacío si el campo _anotherfield_ es igual a cualquier _value_.
> > The field under validation must be present and not empty if the _anotherfield_ field is equal to any _value_.

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

El campo validado debe estar presente y no estar vacío a menos que el campo _anotherfield_ sea igual a cualquier _value_.
> > The field under validation must be present and not empty unless the _anotherfield_ field is equal to any _value_.

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

El campo validado debe estar presente y no está vacío, solo si está presente alguno de los otros campos especificados.
> > The field under validation must be present and not empty _only if_ any of the other specified fields are present.

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

El campo validado debe estar presente y no estar vacío _sólo si_ están presentes todos los demás campos especificados.
> > The field under validation must be present and not empty _only if_ all of the other specified fields are present.

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

El campo validado debe estar presente y no está vacío _sólo cuando_ ninguno de los otros campos especificados estén presentes.
> > The field under validation must be present and not empty _only when_ any of the other specified fields are not present.

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

El campo validado debe estar presente y no estar vacío, solo cuando todos los demás campos especificados no están presentes.
> > The field under validation must be present and not empty _only when_ all of the other specified fields are not present.

<a name="rule-same"></a>
#### same:_field_

El _field_ dado debe coincidir con el campo validado.
> > The given _field_ must match the field under validation.

<a name="rule-size"></a>
#### size:_value_

El campo validado debe tener un tamaño que coincida con el _value_ dado. Para datos de cadena, _value_ corresponde al número de caracteres. Para datos numéricos, _value_ corresponde a un valor entero dado. Para un array, _size_ corresponde al `count` del array. Para los archivos, _size_ corresponde al tamaño del archivo en kilobytes.
> > The field under validation must have a size matching the given _value_. For string data, _value_ corresponds to the number of characters. For numeric data, _value_ corresponds to a given integer value. For an array, _size_ corresponds to the `count` of the array. For files, _size_ corresponds to the file size in kilobytes.

<a name="rule-string"></a>
#### string

El campo validado debe ser una cadena. Si desea permitir que el campo también sea `null`, debe asignar la regla` nullable` al campo.
> > The field under validation must be a string. If you would like to allow the field to also be `null`, you should assign the `nullable` rule to the field.

<a name="rule-timezone"></a>
#### timezone

El campo validado debe ser un identificador de zona horaria válido según la función PHP `timezone_identifiers_list`.
> > The field under validation must be a valid timezone identifier according to the `timezone_identifiers_list` PHP function.

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

El campo validado debe ser único en una tabla de base de datos dada. Si no se especifica la opción `column`, se usará el nombre del campo.
> > The field under validation must be unique in a given database table. If the `column` option is not specified, the field name will be used.

**Especificando un nombre de columna personalizado:**
> > **Specifying A Custom Column Name:**

    'email' => 'unique:users,email_address'

**Conexión de base de datos personalizada**
> > **Custom Database Connection**

Ocasionalmente, es posible que deba establecer una conexión personalizada para las consultas de la base de datos realizadas por el Validador. Como se vio anteriormente, al establecer `unique:users` como una regla de validación se usará la conexión de base de datos predeterminada para consultar la base de datos. Para anular esto, especifique la conexión y el nombre de la tabla usando la sintaxis del "punto":
> > Occasionally, you may need to set a custom connection for database queries made by the Validator. As seen above, setting `unique:users` as a validation rule will use the default database connection to query the database. To override this, specify the connection and the table name using "dot" syntax:

    'email' => 'unique:connection.users,email_address'

**Forzar una regla única para ignorar un ID dado:**
> > **Forcing A Unique Rule To Ignore A Given ID:**

A veces, es posible que desee ignorar una identificación dada durante el control único. Por ejemplo, considere una pantalla de "perfil de actualización" que incluya el nombre del usuario, la dirección de correo electrónico y la ubicación. Por supuesto, querrá verificar que la dirección de correo electrónico sea única. Sin embargo, si el usuario solo cambia el campo de nombre y no el campo de correo electrónico, no desea que se genere un error de validación porque el usuario ya es el propietario de la dirección de correo electrónico.
> > Sometimes, you may wish to ignore a given ID during the unique check. For example, consider an "update profile" screen that includes the user's name, e-mail address, and location. Of course, you will want to verify that the e-mail address is unique. However, if the user only changes the name field and not the e-mail field, you do not want a validation error to be thrown because the user is already the owner of the e-mail address.

Para indicar al validador que ignore la identificación del usuario, usaremos la clase `Rule` para definir la regla de manera fluida. En este ejemplo, también especificaremos las reglas de validación como una matriz en lugar de usar el carácter `|` para delimitar las reglas:
> > To instruct the validator to ignore the user's ID, we'll use the `Rule` class to fluently define the rule. In this example, we'll also specify the validation rules as an array instead of using the `|` character to delimit the rules:

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::unique('users')->ignore($user->id),
        ],
    ]);

Si su tabla utiliza un nombre de columna principal que no sea `id`, puede especificar el nombre de la columna cuando llame al método `ignore`:
> > If your table uses a primary key column name other than `id`, you may specify the name of the column when calling the `ignore` method:

    'email' => Rule::unique('users')->ignore($user->id, 'user_id')

**Agregar cláusulas Where adicionales:**
> > **Adding Additional Where Clauses:**

También puede especificar restricciones de consulta adicionales personalizando la consulta utilizando el método `where`. Por ejemplo, agreguemos una restricción que verifica que `account_id` es `1`:
> > You may also specify additional query constraints by customizing the query using the `where` method. For example, let's add a constraint that verifies the `account_id` is `1`:

    'email' => Rule::unique('users')->where(function ($query) {
        return $query->where('account_id', 1);
    })

<a name="rule-url"></a>
#### url

El campo validado debe ser una URL válida.
> > The field under validation must be a valid URL.

<a name="conditionally-adding-rules"></a>
## Reglas de adición condicional : Conditionally Adding Rules

#### Validar cuando es presente : Validating When Present

En algunas situaciones, es posible que desee ejecutar comprobaciones de validación en un campo **solo** si ese campo está presente en el array de entrada. Para lograr esto rápidamente, agregue la regla `sometimes` a su lista de reglas:
> > In some situations, you may wish to run validation checks against a field **only** if that field is present in the input array. To quickly accomplish this, add the `sometimes` rule to your rule list:

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

En el ejemplo anterior, el campo `email` solo se validará si está presente en array `$data`.
> > In the example above, the `email` field will only be validated if it is present in the `$data` array.

> {tip} Si está intentando validar un campo que siempre debe estar presente pero puede estar vacío, consulte [esta nota en campos opcionales](#a-note-on-optional-fields)
> > > {tip} If you are attempting to validate a field that should always be present but may be empty, check out [this note on optional fields](#a-note-on-optional-fields)

#### Validación Condicional Compleja : Complex Conditional Validation

En ocasiones, es posible que desee agregar reglas de validación basadas en una lógica condicional más compleja. Por ejemplo, es posible que desee solicitar un campo determinado solo si otro campo tiene un valor mayor que 100. O bien, puede necesitar dos campos para tener un valor determinado solo cuando haya otro campo presente. Agregar estas reglas de validación no tiene que ser un problema. Primero, crea una instancia `Validator` con tus _estatic rules_ que nunca cambian:
> > Sometimes you may wish to add validation rules based on more complex conditional logic. For example, you may wish to require a given field only if another field has a greater value than 100. Or, you may need two fields to have a given value only when another field is present. Adding these validation rules doesn't have to be a pain. First, create a `Validator` instance with your _static rules_ that never change:

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

Supongamos que nuestra aplicación web es para coleccionistas de juegos. Si un coleccionista de juegos se registra con nuestra aplicación y posee más de 100 juegos, queremos que explique por qué posee tantos juegos. Por ejemplo, tal vez tengan una tienda de reventa de juegos, o tal vez simplemente disfrutan coleccionar. Para agregar de forma condicional este requisito, podemos usar el método `sometimes` en la instancia `Validator`.
> > Let's assume our web application is for game collectors. If a game collector registers with our application and they own more than 100 games, we want them to explain why they own so many games. For example, perhaps they run a game resale shop, or maybe they just enjoy collecting. To conditionally add this requirement, we can use the `sometimes` method on the `Validator` instance.

    $v->sometimes('reason', 'required|max:500', function ($input) {
        return $input->games >= 100;
    });

El primer argumento pasado al método `sometimes` es el nombre del campo que validamos condicionalmente. El segundo argumento es las reglas que queremos agregar. Si el `Closure` pasado como tercer argumento devuelve `true`, las reglas se agregarán. Este método facilita la creación de validaciones condicional complejas. Incluso puede agregar validaciones condicionales para varios campos a la vez:
> > The first argument passed to the `sometimes` method is the name of the field we are conditionally validating. The second argument is the rules we want to add. If the `Closure` passed as the third argument returns `true`, the rules will be added. This method makes it a breeze to build complex conditional validations. You may even add conditional validations for several fields at once:

    $v->sometimes(['reason', 'cost'], 'required', function ($input) {
        return $input->games >= 100;
    });

> {tip} El parámetro `$input` pasado a su `Closure` será una instancia de `Illuminate\Support\Fluent` y se puede usar para acceder a sus entradas y archivos.
> > > {tip} The `$input` parameter passed to your `Closure` will be an instance of `Illuminate\Support\Fluent` and may be used to access your input and files.

<a name="validating-arrays"></a>
## Validar matrices : Validating Arrays

La validación de los campos de entrada de formularios basados ​​en arreglos no tiene que ser un problema. Puede usar "notación de puntos" para validar atributos dentro de un array. Por ejemplo, si la solicitud HTTP entrante contiene un campo `photos[profile]`, puede validarlo así:
> > Validating array based form input fields doesn't have to be a pain. You may use "dot notation" to validate attributes within an array. For example, if the incoming HTTP request contains a `photos[profile]` field, you may validate it like so:

    $validator = Validator::make($request->all(), [
        'photos.profile' => 'required|image',
    ]);

También puede validar cada elemento de un array. Por ejemplo, para validar que cada correo electrónico en un campo de entrada de array determinado sea único, puede hacer lo siguiente:
> > You may also validate each element of an array. For example, to validate that each e-mail in a given array input field is unique, you may do the following:

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);

Del mismo modo, puede usar el caracter `*` cuando especifique sus mensajes de validación en sus archivos de idioma, por lo que es muy fácil utilizar un solo mensaje de validación para los campos basados ​​en arrays:
> > Likewise, you may use the `*` character when specifying your validation messages in your language files, making it a breeze to use a single validation message for array based fields:

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],

<a name="custom-validation-rules"></a>
## Reglas de validación personalizadas : Custom Validation Rules

<a name="using-rule-objects"></a>
### Using Rule Objects

Laravel proporciona una variedad de reglas de validación útiles; Sin embargo, es posible que desee especificar algunos de los suyos. Un método para registrar reglas de validación personalizadas es usar objetos de reglas. Para generar un nuevo objeto de regla, puede usar el comando Artisan `make:rule`. Usemos este comando para generar una regla que verifique que una cadena esté en mayúscula. Laravel colocará la nueva regla en el directorio `app/Rules`:
> > Laravel provides a variety of helpful validation rules; however, you may wish to specify some of your own. One method of registering custom validation rules is using rule objects. To generate a new rule object, you may use the `make:rule` Artisan command. Let's use this command to generate a rule that verifies a string is uppercase. Laravel will place the new rule in the `app/Rules` directory:

    php artisan make:rule Uppercase

Una vez que se ha creado la regla, estamos listos para definir su comportamiento. Un objeto de regla contiene dos métodos: `passes` y `message`. El método `passes` recibe el nombre y el valor del atributo, y debe devolver `true` o `false` en función de si el valor del atributo es válido o no. El método `message` debe devolver el mensaje de error de validación que se debe usar cuando la validación falla:
> > Once the rule has been created, we are ready to define its behavior. A rule object contains two methods: `passes` and `message`. The `passes` method receives the attribute value and name, and should return `true` or `false` depending on whether the attribute value is valid or not. The `message` method should return the validation error message that should be used when validation fails:

    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\Rule;

    class Uppercase implements Rule
    {
        /**
         * Determine if the validation rule passes.
         *
         * @param  string  $attribute
         * @param  mixed  $value
         * @return bool
         */
        public function passes($attribute, $value)
        {
            return strtoupper($value) === $value;
        }

        /**
         * Get the validation error message.
         *
         * @return string
         */
        public function message()
        {
            return 'The :attribute must be uppercase.';
        }
    }

Por supuesto, puede llamar al asistente `trans` desde su método `message` si desea devolver un mensaje de error de sus archivos de traducción:
> > Of course, you may call the `trans` helper from your `message` method if you would like to return an error message from your translation files:

    /**
     * Get the validation error message.
     *
     * @return string
     */
    public function message()
    {
        return trans('validation.uppercase');
    }

Una vez que se ha definido la regla, puede adjuntarla a un validador pasando una instancia del objeto de regla con sus otras reglas de validación:
> > Once the rule has been defined, you may attach it to a validator by passing an instance of the rule object with your other validation rules:

    use App\Rules\Uppercase;

    $request->validate([
        'name' => ['required', 'string', new Uppercase],
    ]);

<a name="using-closures"></a>
### Uso de Closures : Using Closures

Si solo necesita la funcionalidad de una regla personalizada una vez en su aplicación, puede usar un Cierre en lugar de un objeto de regla. El Cierre recibe el nombre del atributo, el valor del atributo y una devolución de llamada `$fail` que debe invocarse si falla la validación:
> > If you only need the functionality of a custom rule once throughout your application, you may use a Closure instead of a rule object. The Closure receives the attribute's name, the attribute's value, and a `$fail` callback that should be called if validation fails:

    $validator = Validator::make($request->all(), [
        'title' => [
            'required',
            'max:255',
            function($attribute, $value, $fail) {
                if ($value === 'foo') {
                    return $fail($attribute.' is invalid.');
                }
            },
        ],
    ]);

<a name="using-extensions"></a>
### Uso de extensiones : Using Extensions

Otro método para registrar reglas de validación personalizadas es usar el método `extend` en la [fachada](/docs/{{version}}/facades) `Validator`. Usemos este método dentro de un [proveedor de servicios](/docs/{{version}}/providers) para registrar una regla de validación personalizada:
> > Another method of registering custom validation rules is using the `extend` method on the `Validator` [facade](/docs/{{version}}/facades). Let's use this method within a [service provider](/docs/{{version}}/providers) to register a custom validation rule:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Validator;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
                return $value == 'foo';
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

El Closure del validador personalizado recibe cuatro argumentos: el nombre del `$attribute` que se valida, el `$value` del atributo, un array de `$parameters` pasada a la regla y la instancia `Validator`.
> > The custom validator Closure receives four arguments: the name of the `$attribute` being validated, the `$value` of the attribute, an array of `$parameters` passed to the rule, and the `Validator` instance.

También puede pasar una clase y un método al método `extend` en lugar de un Closure:
> > You may also pass a class and method to the `extend` method instead of a Closure:

    Validator::extend('foo', 'FooValidator@validate');

#### Definición del mensaje de error : Defining The Error Message

También necesitará definir un mensaje de error para su regla personalizada. Puede hacerlo utilizando una matriz de mensajes personalizados en línea o agregando una entrada en el archivo de idioma de validación. Este mensaje debe colocarse en el primer nivel del array, no dentro del array `custom`, que es solo para mensajes de error específicos del atributo:
> > You will also need to define an error message for your custom rule. You can do so either using an inline custom message array or by adding an entry in the validation language file. This message should be placed in the first level of the array, not within the `custom` array, which is only for attribute-specific error messages:

    "foo" => "Your input was invalid!",

    "accepted" => "The :attribute must be accepted.",

    // The rest of the validation error messages...

Al crear una regla de validación personalizada, es posible que a veces necesite definir reemplazos de marcador de posición personalizados para los mensajes de error. Puede hacerlo creando un Validator personalizado como se describe arriba y luego realizando una llamada al método `replacer` en la fachada `Validator`. Puede hacer esto dentro del método `boot` de un [proveedor de servicios](/docs/{{version}}/providers):
> > When creating a custom validation rule, you may sometimes need to define custom place-holder replacements for error messages. You may do so by creating a custom Validator as described above then making a call to the `replacer` method on the `Validator` facade. You may do this within the `boot` method of a [service provider](/docs/{{version}}/providers):

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend(...);

        Validator::replacer('foo', function ($message, $attribute, $rule, $parameters) {
            return str_replace(...);
        });
    }

#### Extensiones implícitas : Implicit Extensions

Por defecto, cuando un atributo que se está validando no está presente o contiene un valor vacío como se define en la regla [`required`](#rule-required), las reglas de validación normales, incluidas las extensiones personalizadas, no se ejecutan. Por ejemplo, la regla [`unique`](#rule-unique) no se ejecutará contra un valor `null`:
> > By default, when an attribute being validated is not present or contains an empty value as defined by the [`required`](#rule-required) rule, normal validation rules, including custom extensions, are not run. For example, the [`unique`](#rule-unique) rule will not be run against a `null` value:

    $rules = ['name' => 'unique'];

    $input = ['name' => null];

    Validator::make($input, $rules)->passes(); // true

Para que una regla se ejecute incluso cuando un atributo está vacío, la regla debe implicar que el atributo es obligatorio. Para crear una extensión "implícita", use el método `Validator::extendImplicit()`:
> > For a rule to run even when an attribute is empty, the rule must imply that the attribute is required. To create such an "implicit" extension, use the `Validator::extendImplicit()` method:

    Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });

> {note} Una extensión "implícita" solo _implies_ que el atributo es obligatorio. Si usted realmente invalida un atributo faltante o vacío depende de usted.
> > > {note} An "implicit" extension only _implies_ that the attribute is required. Whether it actually invalidates a missing or empty attribute is up to you.
