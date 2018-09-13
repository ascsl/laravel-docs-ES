# Ciclo de vida de la solicitud: Request Lifecycle

- [Introduction](#introduction)
- [Lifecycle Overview](#lifecycle-overview)
- [Focus On Service Providers](#focus-on-service-providers)

<a name="introduction"></a>
## Introducción : Introduction

Al usar cualquier herramienta en el "mundo real", se siente más seguro si comprende cómo funciona esa herramienta. El desarrollo de aplicaciones no es diferente. Cuando comprenda cómo funcionan sus herramientas de desarrollo, se sentirá más cómodo y seguro usándolas.
> > When using any tool in the "real world", you feel more confident if you understand how that tool works. Application development is no different. When you understand how your development tools function, you feel more comfortable and confident using them.

El objetivo de este documento es brindarle una buena descripción general de alto nivel de cómo funciona el framework de trabajo de Laravel. Al conocer mejor el framework general, todo se siente menos "mágico" y tendrá más confianza en la construcción de sus aplicaciones. Si no comprende todos los términos de inmediato, ¡no se desanime! Intente obtener una comprensión básica de lo que está sucediendo y su conocimiento crecerá a medida que explore otras secciones de la documentación.
> > The goal of this document is to give you a good, high-level overview of how the Laravel framework works. By getting to know the overall framework better, everything feels less "magical" and you will be more confident building your applications. If you don't understand all of the terms right away, don't lose heart! Just try to get a basic grasp of what is going on, and your knowledge will grow as you explore other sections of the documentation.

<a name="lifecycle-overview"></a>
## Resumen del ciclo de vida : Lifecycle Overview

### Primeras cosas : First Things

El punto de entrada para todas las solicitudes a una aplicación Laravel es el archivo `public/index.php`. Todas las solicitudes se dirigen a este archivo mediante la configuración de su servidor web (Apache / Nginx). El archivo `index.php` no contiene mucho código. Más bien, es un punto de partida para cargar el resto del framework.
> > The entry point for all requests to a Laravel application is the `public/index.php` file. All requests are directed to this file by your web server (Apache / Nginx) configuration. The `index.php` file doesn't contain much code. Rather, it is a starting point for loading the rest of the framework.

El archivo `index.php` carga la definición del autocargador generado por Composer y luego recupera una instancia de la aplicación Laravel del script `bootstrap/app.php`. La primera acción realizada por Laravel en sí es crear una instancia de la aplicación / [contenedor de servicios](/docs/{{version}}/container).
> > The `index.php` file loads the Composer generated autoloader definition, and then retrieves an instance of the Laravel application from `bootstrap/app.php` script. The first action taken by Laravel itself is to create an instance of the application / [service container](/docs/{{version}}/container).

### Núcleos HTTP / Consola : HTTP / Console Kernels

A continuación, la solicitud entrante se envía al kernel HTTP o al kernel de la consola, según el tipo de solicitud que ingrese a la aplicación. Estos dos núcleos sirven como la ubicación central a través de la cual fluyen todas las solicitudes. Por ahora, centrémonos en el kernel HTTP, que se encuentra en `app/Http/Kernel.php`.
> > Next, the incoming request is sent to either the HTTP kernel or the console kernel, depending on the type of request that is entering the application. These two kernels serve as the central location that all requests flow through. For now, let's just focus on the HTTP kernel, which is located in `app/Http/Kernel.php`.

El kernel HTTP amplía la clase `Illuminate\Foundation\Http\Kernel`, que define un array de `bootstrappers` que se ejecutará antes de que se ejecute la solicitud. Estos arrancadores configuran el manejo de errores, configuran el registro, [detectan el entorno de la aplicación](/docs/{{version}}/configuration#environment-configuration) y realizan otras tareas que deben realizarse antes de que la solicitud sea realmente manejada.
> > The HTTP kernel extends the `Illuminate\Foundation\Http\Kernel` class, which defines an array of `bootstrappers` that will be run before the request is executed. These bootstrappers configure error handling, configure logging, [detect the application environment](/docs/{{version}}/configuration#environment-configuration), and perform other tasks that need to be done before the request is actually handled.

El kernel de HTTP también define una lista de HTTP [middleware](/docs/{{version}}/middleware) que todas las solicitudes deben pasar antes de ser manejadas por la aplicación. Estos middleware manejan la lectura y escritura de la [sesión HTTP](/docs/{{version}}/session), determinando si la aplicación está en modo de mantenimiento, [verificando el token CSRF](/docs/{{version}}/csrf), y más.
> > The HTTP kernel also defines a list of HTTP [middleware](/docs/{{version}}/middleware) that all requests must pass through before being handled by the application. These middleware handle reading and writing the [HTTP session](/docs/{{version}}/session), determining if the application is in maintenance mode, [verifying the CSRF token](/docs/{{version}}/csrf), and more.

La firma del método para el método `handle` del núcleo HTTP es bastante simple: recibe una `Request` y devuelve una `Response`. Piense en Kernel como una gran caja negra que representa toda su aplicación. Alimenta las solicitudes HTTP y devolverá las respuestas HTTP.
> > The method signature for the HTTP kernel's `handle` method is quite simple: receive a `Request` and return a `Response`. Think of the Kernel as being a big black box that represents your entire application. Feed it HTTP requests and it will return HTTP responses.

#### Proveedores de servicio : Service Providers

Una de las acciones de arranque más importantes de Kernel es cargar los [proveedores de servicios](/docs/{{version}}/providers) para su aplicación. Todos los proveedores de servicios para la aplicación se configuran en el array `providers` del archivo de configuración `config/app.php`. Primero, se llamará al método `register` en todos los proveedores, luego, una vez que todos los proveedores hayan sido registrados, se llamará al método `boot`.
> > One of the most important Kernel bootstrapping actions is loading the [service providers](/docs/{{version}}/providers) for your application. All of the service providers for the application are configured in the `config/app.php` configuration file's `providers` array. First, the `register` method will be called on all providers, then, once all providers have been registered, the `boot` method will be called.

Los proveedores de servicios son responsables de iniciar todos los diversos componentes del framework, como la base de datos, la cola, la validación y los componentes de enrutamiento. Desde que inician y configuran todas las características ofrecidas por el framework, los proveedores de servicios son el aspecto más importante de todo el proceso de arranque de Laravel.
> > Service providers are responsible for bootstrapping all of the framework's various components, such as the database, queue, validation, and routing components. Since they bootstrap and configure every feature offered by the framework, service providers are the most important aspect of the entire Laravel bootstrap process.

#### Solicitud de envío : Dispatch Request

Una vez que la aplicación se ha iniciado y todos los proveedores de servicios se han registrado, la `Request` se transferirá al enrutador para su envío. El enrutador enviará la solicitud a una ruta o controlador, así como también ejecutará cualquier middleware específico de la ruta.
> > Once the application has been bootstrapped and all service providers have been registered, the `Request` will be handed off to the router for dispatching. The router will dispatch the request to a route or controller, as well as run any route specific middleware.

<a name="focus-on-service-providers"></a>
## Foco en los proveedores de servicios : Focus On Service Providers

Los proveedores de servicios son realmente la clave para iniciar una aplicación Laravel. Se crea la instancia de la aplicación, los proveedores del servicio están registrados y la solicitud se entrega a la aplicación de arranque. ¡Es así de simple!
> > Service providers are truly the key to bootstrapping a Laravel application. The application instance is created, the service providers are registered, and the request is handed to the bootstrapped application. It's really that simple!

Tener una comprensión firme de cómo se construye una aplicación Laravel y se inicia a través de proveedores de servicios es muy valiosa. Por supuesto, los proveedores de servicios predeterminados de su aplicación se almacenan en el directorio `app/Providers`.
> > Having a firm grasp of how a Laravel application is built and bootstrapped via service providers is very valuable. Of course, your application's default service providers are stored in the `app/Providers` directory.

Por defecto, `AppServiceProvider` está bastante vacío. Este proveedor es un gran lugar para agregar los enlaces de carga y de contenedor de servicio propios de su aplicación. Por supuesto, para aplicaciones grandes, es posible que desee crear varios proveedores de servicios, cada uno con un tipo más detallado de bootstrapping.
> > By default, the `AppServiceProvider` is fairly empty. This provider is a great place to add your application's own bootstrapping and service container bindings. Of course, for large applications, you may wish to create several service providers, each with a more granular type of bootstrapping.
