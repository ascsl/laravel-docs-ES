# Redirecciones HTTP : HTTP Redirects

- [Creating Redirects](#creating-redirects)
- [Redirecting To Named Routes](#redirecting-named-routes)
- [Redirecting To Controller Actions](#redirecting-controller-actions)
- [Redirecting With Flashed Session Data](#redirecting-with-flashed-session-data)

<a name="creating-redirects"></a>
## Creación de redirecciones : Creating Redirects

Las respuestas de redireccionamiento son instancias de la clase `Illuminate\Http\RedirectResponse` y contienen los encabezados adecuados necesarios para redirigir al usuario a otra URL. Hay varias formas de generar una instancia `RedirectResponse`. El método más simple es usar el helper global `redirect`:
> > Redirect responses are instances of the `Illuminate\Http\RedirectResponse` class, and contain the proper headers needed to redirect the user to another URL. There are several ways to generate a `RedirectResponse` instance. The simplest method is to use the global `redirect` helper:

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

En ocasiones, es posible que desee redireccionar al usuario a su ubicación anterior, como cuando un formulario enviado no es válido. Puedes hacerlo usando la función de ayuda global `back`. Como esta característica utiliza [session](/docs/{{version}}/session), asegúrese de que la ruta que llama a la función `back` esté utilizando el grupo de middleware `web` o que se haya aplicado todo el middleware de sesión:
> > Sometimes you may wish to redirect the user to their previous location, such as when a submitted form is invalid. You may do so by using the global `back` helper function. Since this feature utilizes the [session](/docs/{{version}}/session), make sure the route calling the `back` function is using the `web` middleware group or has all of the session middleware applied:

    Route::post('user/profile', function () {
        // Validate the request...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
## Redirigir a rutas específicas : Redirecting To Named Routes

Cuando llamas al asistente `redirect` sin parámetros, se devuelve una instancia de `Illuminate\Routing\Redirector`, que te permite llamar a cualquier método en la instancia `Redirector`. Por ejemplo, para generar un `RedirectResponse` a una ruta con nombre, puede usar el método `route`:
> > When you call the `redirect` helper with no parameters, an instance of `Illuminate\Routing\Redirector` is returned, allowing you to call any method on the `Redirector` instance. For example, to generate a `RedirectResponse` to a named route, you may use the `route` method:

    return redirect()->route('login');

Si su ruta tiene parámetros, puede pasarlos como el segundo argumento al método `route`:
> > If your route has parameters, you may pass them as the second argument to the `route` method:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### Rellenar parámetros a través de modelos Eloquent : Populating Parameters Via Eloquent Models

Si está redireccionando a una ruta con un parámetro "ID" que se está rellenando desde un modelo Eloquent, puede pasar el modelo mismo. La identificación se extraerá automáticamente:
> > If you are redirecting to a route with an "ID" parameter that is being populated from an Eloquent model, you may pass the model itself. The ID will be extracted automatically:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', [$user]);

Si desea personalizar el valor que se coloca en el parámetro de ruta, debe anular el método `getRouteKey` en su modelo Eloquent:
> > If you would like to customize the value that is placed in the route parameter, you should override the `getRouteKey` method on your Eloquent model:

    /**
     * Get the value of the model's route key.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
## Redirigir a las acciones del controlador : Redirecting To Controller Actions

También puede generar redirecciones a [acciones del controlador](/docs/{{version}}/controllers). Para hacerlo, pase el nombre del controlador y la acción al método `action`. Recuerde, no necesita especificar el espacio de nombres completo para el controlador ya que `RouteServiceProvider` de Laravel configurará automáticamente el nombre de espacio del controlador base:
> > You may also generate redirects to [controller actions](/docs/{{version}}/controllers). To do so, pass the controller and action name to the `action` method. Remember, you do not need to specify the full namespace to the controller since Laravel's `RouteServiceProvider` will automatically set the base controller namespace:

    return redirect()->action('HomeController@index');

Si la ruta de su controlador requiere parámetros, puede pasarlos como el segundo argumento para el método `acción`:
> > If your controller route requires parameters, you may pass them as the second argument to the `action` method:

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );

<a name="redirecting-with-flashed-session-data"></a>
## Redirigir con datos de sesión parpadeados : Redirecting With Flashed Session Data

El redireccionamiento a una nueva URL y [datos intermitentes a la sesión](/docs/{{version}}/session#flash-data) generalmente se realizan al mismo tiempo. Por lo general, esto se hace después de realizar con éxito una acción cuando muestra un mensaje de éxito en la sesión. Para su comodidad, puede crear una instancia `RedirectResponse` y actualizar datos a la sesión en una única cadena de métodos fluida:
> > Redirecting to a new URL and [flashing data to the session](/docs/{{version}}/session#flash-data) are usually done at the same time. Typically, this is done after successfully performing an action when you flash a success message to the session. For convenience, you may create a `RedirectResponse` instance and flash data to the session in a single, fluent method chain:

    Route::post('user/profile', function () {
        // Update the user's profile...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

Después de redirigir al usuario, puede mostrar el mensaje de la [sesión](/docs/{{version}}/sesión). Por ejemplo, usando [sintaxis Blade](/docs/{{version}}/blade):
> > After the user is redirected, you may display the flashed message from the [session](/docs/{{version}}/session). For example, using [Blade syntax](/docs/{{version}}/blade):

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif
