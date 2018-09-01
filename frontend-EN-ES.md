# Andamios JavaScript y CSS : JavaScript & CSS Scaffolding

- [Introduction](#introduction)
- [Writing CSS](#writing-css)
- [Writing JavaScript](#writing-javascript)
    - [Writing Vue Components](#writing-vue-components)
    - [Using React](#using-react)

<a name="introduction"></a>
## Introducción : Introduction

Si bien Laravel no dicta qué preprocesadores de JavaScript o CSS utiliza, proporciona un punto de partida básico con [Bootstrap](https://getbootstrap.com/) y [Vue](https://vuejs.org) que será útil para muchas aplicaciones. De forma predeterminada, Laravel usa [NPM](https://www.npmjs.org) para instalar ambos paquetes frontend.
> > While Laravel does not dictate which JavaScript or CSS pre-processors you use, it does provide a basic starting point using [Bootstrap](https://getbootstrap.com/) and [Vue](https://vuejs.org) that will be helpful for many applications. By default, Laravel uses [NPM](https://www.npmjs.org) to install both of these frontend packages.

#### CSS

[Laravel Mix](/docs/{{version}}/mix) proporciona una API limpia y expresiva sobre la compilación de SASS o Less, que son extensiones de CSS simples que agregan variables, mixins y otras poderosas funciones que hacen que trabajar con CSS sea mucho más agradable. En este documento, discutiremos brevemente la compilación de CSS en general; sin embargo, debe consultar la documentación completa de [Laravel Mix](/docs/{{version}}/mix) para obtener más información sobre cómo compilar SASS o Less.
> > [Laravel Mix](/docs/{{version}}/mix) provides a clean, expressive API over compiling SASS or Less, which are extensions of plain CSS that add variables, mixins, and other powerful features that make working with CSS much more enjoyable. In this document, we will briefly discuss CSS compilation in general; however, you should consult the full [Laravel Mix documentation](/docs/{{version}}/mix) for more information on compiling SASS or Less.

#### JavaScript

Laravel no requiere que use un framework de JavaScript específico o una librería para construir sus aplicaciones. De hecho, no tiene que usar JavaScript en absoluto. Sin embargo, Laravel sí incluye algunos andamios básicos para que sea más fácil comenzar a escribir JavaScript moderno usando la biblioteca [Vue](https://vuejs.org). Vue proporciona una API expresiva para crear aplicaciones robustas de JavaScript utilizando componentes. Al igual que con CSS, podemos usar Laravel Mix para compilar fácilmente componentes JavaScript en un solo archivo JavaScript listo para el navegador.
> > Laravel does not require you to use a specific JavaScript framework or library to build your applications. In fact, you don't have to use JavaScript at all. However, Laravel does include some basic scaffolding to make it easier to get started writing modern JavaScript using the [Vue](https://vuejs.org) library. Vue provides an expressive API for building robust JavaScript applications using components. As with CSS, we may use Laravel Mix to easily compile JavaScript components into a single, browser-ready JavaScript file.

#### Extracción del andamio frontend : Removing The Frontend Scaffolding

Si desea eliminar el andamio frontend de su aplicación, puede usar el comando Artisan `preset`. Este comando, cuando se combina con la opción `none`, eliminará los andamios Bootstrap y Vue de su aplicación, dejando solo un archivo SASS en blanco y algunas librerías comunes de utilidades de JavaScript:
> > If you would like to remove the frontend scaffolding from your application, you may use the `preset` Artisan command. This command, when combined with the `none` option, will remove the Bootstrap and Vue scaffolding from your application, leaving only a blank SASS file and a few common JavaScript utility libraries:

    php artisan preset none

<a name="writing-css"></a>
## Escritura de CSS : Writing CSS

El archivo `package.json` de Laravel incluye el paquete `bootstrap` para ayudarlo a comenzar a crear prototipos de la interfaz de su aplicación usando Bootstrap. Sin embargo, siéntase libre de agregar o eliminar paquetes del archivo `package.json` según sea necesario para su propia aplicación. No es necesario que utilice el marco Bootstrap para construir su aplicación Laravel, sino que se proporciona como un buen punto de partida para quienes eligen usarlo.
> > Laravel's `package.json` file includes the `bootstrap` package to help you get started prototyping your application's frontend using Bootstrap. However, feel free to add or remove packages from the `package.json` file as needed for your own application. You are not required to use the Bootstrap framework to build your Laravel application - it is provided as a good starting point for those who choose to use it.

Antes de compilar su CSS, instale las dependencias frontend de su proyecto usando [Node package manager (NPM)](https://www.npmjs.org):
> > Before compiling your CSS, install your project's frontend dependencies using the [Node package manager (NPM)](https://www.npmjs.org):

    npm install

Una vez que las dependencias se han instalado usando `npm install`, puede compilar sus archivos SASS en CSS simple usando [Laravel Mix](/docs/{{version}}/mix#working-with-stylesheets). El comando `npm run dev` procesará las instrucciones en su archivo `webpack.mix.js`. Normalmente, su CSS compilado se colocará en el directorio `public/css`:
> > Once the dependencies have been installed using `npm install`, you can compile your SASS files to plain CSS using [Laravel Mix](/docs/{{version}}/mix#working-with-stylesheets). The `npm run dev` command will process the instructions in your `webpack.mix.js` file. Typically, your compiled CSS will be placed in the `public/css` directory:

    npm run dev

El `webpack.mix.js` predeterminado que se incluye con Laravel compilará el archivo SASS `resources/assets/sass/app.scss`. Este archivo `app.scss` importa un archivo de variables SASS y carga Bootstrap, que proporciona un buen punto de partida para la mayoría de las aplicaciones. Siéntase libre de personalizar el archivo `app.scss` como lo desee o incluso usar un pre-procesador completamente diferente al [configurar Laravel Mix](/docs/{{version}}/mix).
> > The default `webpack.mix.js` included with Laravel will compile the `resources/assets/sass/app.scss` SASS file. This `app.scss` file imports a file of SASS variables and loads Bootstrap, which provides a good starting point for most applications. Feel free to customize the `app.scss` file however you wish or even use an entirely different pre-processor by [configuring Laravel Mix](/docs/{{version}}/mix).

<a name="writing-javascript"></a>
## Writing JavaScript

Todas las dependencias de JavaScript requeridas por su aplicación se pueden encontrar en el archivo `package.json` en el directorio raíz del proyecto. Este archivo es similar a un archivo `composer.json`, excepto que especifica dependencias de JavaScript en lugar de dependencias de PHP. Puede instalar estas dependencias usando [Node package manager (NPM)](https://www.npmjs.org):
> > All of the JavaScript dependencies required by your application can be found in the `package.json` file in the project's root directory. This file is similar to a `composer.json` file except it specifies JavaScript dependencies instead of PHP dependencies. You can install these dependencies using the [Node package manager (NPM)](https://www.npmjs.org):

    npm install

> {tip} Por defecto, el archivo Laravel `package.json` incluye algunos paquetes como `vue` y `axios` para ayudarlo a comenzar a construir su aplicación JavaScript. Siéntase libre de agregar o eliminar del archivo `package.json` según sea necesario para su propia aplicación.
> > > {tip} By default, the Laravel `package.json` file includes a few packages such as `vue` and `axios` to help you get started building your JavaScript application. Feel free to add or remove from the `package.json` file as needed for your own application.

Una vez que los paquetes están instalados, puede usar el comando `npm run dev` para [compilar sus activos](/docs/{{version}}/mix). Webpack es un paquete de módulos para aplicaciones modernas de JavaScript. Cuando ejecuta el comando `npm run dev`, Webpack ejecutará las instrucciones en su archivo `webpack.mix.js`:
> > Once the packages are installed, you can use the `npm run dev` command to [compile your assets](/docs/{{version}}/mix). Webpack is a module bundler for modern JavaScript applications. When you run the `npm run dev` command, Webpack will execute the instructions in your `webpack.mix.js` file:

    npm run dev

Por defecto, el archivo `webpack.mix.js` de Laravel compila su SASS y el archivo `resources/assets/js/app.js`. Dentro del archivo `app.js` puede registrar sus componentes Vue o, si prefiere un marco diferente, configure su propia aplicación JavaScript. Su JavaScript compilado generalmente se colocará en el directorio `public/js`.
> > By default, the Laravel `webpack.mix.js` file compiles your SASS and the `resources/assets/js/app.js` file. Within the `app.js` file you may register your Vue components or, if you prefer a different framework, configure your own JavaScript application. Your compiled JavaScript will typically be placed in the `public/js` directory.

> {tip} El archivo `app.js` cargará el archivo `resources/assets/js/bootstrap.js` que inicia y configura Vue, Axios, jQuery y todas las demás dependencias de JavaScript. Si tiene dependencias de JavaScript adicionales para configurar, puede hacerlo en este archivo.
> > > {tip} The `app.js` file will load the `resources/assets/js/bootstrap.js` file which bootstraps and configures Vue, Axios, jQuery, and all other JavaScript dependencies. If you have additional JavaScript dependencies to configure, you may do so in this file.

<a name="writing-vue-components"></a>
### Escritura de componentes Vue : Writing Vue Components

Por defecto, las aplicaciones Laravel nuevas contienen un componente Vue `ExampleComponent.vue` ubicado en el directorio `resources/assets/js/components`. El archivo `ExampleComponent.vue` es un ejemplo de un [componente Vue de un único archivo](https://vuejs.org/guide/single-file-components) que define su plantilla de JavaScript y HTML en el mismo archivo. Los componentes de un solo archivo proporcionan un enfoque muy conveniente para crear aplicaciones basadas en JavaScript. El componente de ejemplo está registrado en su archivo `app.js`:
> > By default, fresh Laravel applications contain an `ExampleComponent.vue` Vue component located in the `resources/assets/js/components` directory. The `ExampleComponent.vue` file is an example of a [single file Vue component](https://vuejs.org/guide/single-file-components) which defines its JavaScript and HTML template in the same file. Single file components provide a very convenient approach to building JavaScript driven applications. The example component is registered in your `app.js` file:

    Vue.component(
        'example-component',
        require('./components/ExampleComponent.vue')
    );

Para usar el componente en su aplicación, puede colocarlo en una de sus plantillas HTML. Por ejemplo, después de ejecutar el comando Artisan `make:auth` para andamiar las pantallas de autenticación y registro de su aplicación, puede colocar el componente en la plantilla Blade `home.blade.php`:
> > To use the component in your application, you may drop it into one of your HTML templates. For example, after running the `make:auth` Artisan command to scaffold your application's authentication and registration screens, you could drop the component into the `home.blade.php` Blade template:

    @extends('layouts.app')

    @section('content')
        <example-component></example-component>
    @endsection

> {tip} Recuerde, debe ejecutar el comando `npm run dev` cada vez que cambie un componente Vue. O bien, puede ejecutar el comando `npm run watch` para supervisar y volver a compilar automáticamente sus componentes cada vez que se modifiquen.
> > > {tip} Remember, you should run the `npm run dev` command each time you change a Vue component. Or, you may run the `npm run watch` command to monitor and automatically recompile your components each time they are modified.

Por supuesto, si está interesado en obtener más información sobre la redacción de los componentes de Vue, debe leer la [documentación de Vue](https://vuejs.org/guide/), que proporciona una visión general completa y fácil de leer de toda la documentación. Vue framework.
> > Of course, if you are interested in learning more about writing Vue components, you should read the [Vue documentation](https://vuejs.org/guide/), which provides a thorough, easy-to-read overview of the entire Vue framework.

<a name="using-react"></a>
### Uso de React : Using React

Si prefiere usar React para construir su aplicación JavaScript, Laravel hace que sea muy fácil cambiar el andamio Vue con el andamio React. En cualquier aplicación nueva de Laravel, puede usar el comando `preset` con la opción `react`:
> > If you prefer to use React to build your JavaScript application, Laravel makes it a cinch to swap the Vue scaffolding with React scaffolding. On any fresh Laravel application, you may use the `preset` command with the `react` option:

    php artisan preset react

Este comando único eliminará el andamio Vue y lo reemplazará con un andamio React, que incluye un componente de ejemplo.
> > This single command will remove the Vue scaffolding and replace it with React scaffolding, including an example component.
