# Compilación de Assets (Laravel Mix) : Compiling Assets (Laravel Mix)

- [Introduction](#introduction)
- [Installation & Setup](#installation)
- [Running Mix](#running-mix)
- [Working With Stylesheets](#working-with-stylesheets)
    - [Less](#less)
    - [Sass](#sass)
    - [Stylus](#stylus)
    - [PostCSS](#postcss)
    - [Plain CSS](#plain-css)
    - [URL Processing](#url-processing)
    - [Source Maps](#css-source-maps)
- [Working With JavaScript](#working-with-scripts)
    - [Vendor Extraction](#vendor-extraction)
    - [React](#react)
    - [Vanilla JS](#vanilla-js)
    - [Custom Webpack Configuration](#custom-webpack-configuration)
- [Copying Files & Directories](#copying-files-and-directories)
- [Versioning / Cache Busting](#versioning-and-cache-busting)
- [Browsersync Reloading](#browsersync-reloading)
- [Environment Variables](#environment-variables)
- [Notifications](#notifications)

<a name="introduction"></a>
## Introducción : Introduction

[Laravel Mix] (https://github.com/JeffreyWay/laravel-mix) proporciona una API fluida para definir los pasos de compilación de Webpack para su aplicación Laravel utilizando varios preprocesadores comunes de CSS y JavaScript. Mediante el encadenamiento de métodos simple, puede definir con fluidez su cartera de activos. Por ejemplo:
> > [Laravel Mix](https://github.com/JeffreyWay/laravel-mix) provides a fluent API for defining Webpack build steps for your Laravel application using several common CSS and JavaScript pre-processors. Through simple method chaining, you can fluently define your asset pipeline. For example:

    mix.js('resources/assets/js/app.js', 'public/js')
       .sass('resources/assets/sass/app.scss', 'public/css');

Si alguna vez ha estado confundido y abrumado sobre cómo comenzar con la compilación de elementos Webpack, le encantará Laravel Mix. Sin embargo, no está obligado a usarlo mientras desarrolla su aplicación. Por supuesto, puede usar cualquier herramienta de canalización de asset que desee, o incluso ninguna.
> > If you've ever been confused and overwhelmed about getting started with Webpack and asset compilation, you will love Laravel Mix. However, you are not required to use it while developing your application. Of course, you are free to use any asset pipeline tool you wish, or even none at all.

<a name="installation"></a>
## Instalación y configuración : Installation & Setup

#### Instalación del nodo : Installing Node

Antes de activar Mix, primero debe asegurarse de que Node.js y NPM estén instalados en su máquina.
> > Before triggering Mix, you must first ensure that Node.js and NPM are installed on your machine.

    node -v
    npm -v

Por defecto, Laravel Homestead incluye todo lo que necesita; sin embargo, si no está utilizando Vagrant, puede instalar fácilmente la última versión de Node y NPM usando instaladores gráficos simples desde [su página de descarga](https://nodejs.org/en/download/).
> > By default, Laravel Homestead includes everything you need; however, if you aren't using Vagrant, then you can easily install the latest version of Node and NPM using simple graphical installers from [their download page](https://nodejs.org/en/download/).

#### Laravel Mix

El único paso restante es instalar Laravel Mix. Dentro de una nueva instalación de Laravel, encontrará un archivo `package.json` en la raíz de la estructura de su directorio. El archivo `package.json` predeterminado incluye todo lo que necesita para comenzar. Piense en esto como su archivo `composer.json`, excepto que define dependencias de Nodo en lugar de PHP. Puede instalar las dependencias a las que hace referencia ejecutando:
> > The only remaining step is to install Laravel Mix. Within a fresh installation of Laravel, you'll find a `package.json` file in the root of your directory structure. The default `package.json` file includes everything you need to get started. Think of this like your `composer.json` file, except it defines Node dependencies instead of PHP. You may install the dependencies it references by running:

    npm install

<a name="running-mix"></a>
## Ejecutando Mix : Running Mix

Mix es una capa de configuración en la parte superior de [Webpack](https://webpack.js.org), por lo que para ejecutar las tareas de Mix solo necesita ejecutar uno de los scripts de NPM que se incluye por defecto con el archivo `package.json` de Laravel:
> > Mix is a configuration layer on top of [Webpack](https://webpack.js.org), so to run your Mix tasks you only need to execute one of the NPM scripts that is included with the default Laravel `package.json` file:

    // Run all Mix tasks...
    npm run dev

    // Run all Mix tasks and minify output...
    npm run production

#### Watching Assets For Changes

El comando `npm run watch` continuará ejecutándose en su terminal y mirará todos los archivos relevantes para ver los cambios. Webpack luego recompilará automáticamente sus assets cuando detecte un cambio:
> > The `npm run watch` command will continue running in your terminal and watch all relevant files for changes. Webpack will then automatically recompile your assets when it detects a change:

    npm run watch

Puede encontrar que en ciertos entornos, Webpack no se actualiza cuando cambian sus archivos. Si este es el caso en su sistema, considere usar el comando `watch-poll`:
> > You may find that in certain environments Webpack isn't updating when your files change. If this is the case on your system, consider using the `watch-poll` command:

    npm run watch-poll

<a name="working-with-stylesheets"></a>
## Trabajar con hojas de estilo : Working With Stylesheets

El archivo `webpack.mix.js` es su punto de entrada para toda la compilación de assets. Piense en ello como un contenedor de configuración ligera en Webpack. Las tareas de mezcla se pueden encadenar juntas para definir exactamente cómo deben compilarse sus assets.
> > The `webpack.mix.js` file is your entry point for all asset compilation. Think of it as a light configuration wrapper around Webpack. Mix tasks can be chained together to define exactly how your assets should be compiled.

<a name="less"></a>
### Less

El método `less` se puede usar para compilar [Less](http://lesscss.org/) en CSS. Compilemos nuestro archivo `app.less` primario en `public/css/app.css`.
> > The `less` method may be used to compile [Less](http://lesscss.org/) into CSS. Let's compile our primary `app.less` file to `public/css/app.css`.

    mix.less('resources/assets/less/app.less', 'public/css');

Se pueden usar múltiples llamadas al método `less` para compilar varios archivos:
> > Multiple calls to the `less` method may be used to compile multiple files:

    mix.less('resources/assets/less/app.less', 'public/css')
       .less('resources/assets/less/admin.less', 'public/css');

Si desea personalizar el nombre de archivo de la CSS compilada, puede pasar una ruta de archivo completa como segundo argumento para el método `less`:
> > If you wish to customize the file name of the compiled CSS, you may pass a full file path as the second argument to the `less` method:

    mix.less('resources/assets/less/app.less', 'public/stylesheets/styles.css');

Si necesita anular las [opciones de complemento Less subyacentes](https://github.com/webpack-contrib/less-loader#options), puede pasar un objeto como tercer argumento a `mix.less()`:
> > If you need to override the [underlying Less plug-in options](https://github.com/webpack-contrib/less-loader#options), you may pass an object as the third argument to `mix.less()`:

    mix.less('resources/assets/less/app.less', 'public/css', {
        strictMath: true
    });

<a name="sass"></a>
### Sass

El método `sass` le permite compilar [Sass](http://sass-lang.com/) en CSS. Puede usar el método de esta manera:
> > The `sass` method allows you to compile [Sass](http://sass-lang.com/) into CSS. You may use the method like so:

    mix.sass('resources/assets/sass/app.scss', 'public/css');

Nuevamente, al igual que el método `less`, puede compilar varios archivos Sass en sus respectivos archivos CSS e incluso personalizar el directorio de salida del CSS resultante:
> > Again, like the `less` method, you may compile multiple Sass files into their own respective CSS files and even customize the output directory of the resulting CSS:

    mix.sass('resources/assets/sass/app.sass', 'public/css')
       .sass('resources/assets/sass/admin.sass', 'public/css/admin');

Se pueden proporcionar opciones adicionales [Node-Sass plug-in](https://github.com/sass/node-sass#options) como tercer argumento:
> > Additional [Node-Sass plug-in options](https://github.com/sass/node-sass#options) may be provided as the third argument:

    mix.sass('resources/assets/sass/app.sass', 'public/css', {
        precision: 5
    });

<a name="stylus"></a>
### Stylus

Al igual que Less y Sass, el método `stylus` le permite compilar [Stylus](http://stylus-lang.com/) en CSS:
> > Similar to Less and Sass, the `stylus` method allows you to compile [Stylus](http://stylus-lang.com/) into CSS:

    mix.stylus('resources/assets/stylus/app.styl', 'public/css');

También puede instalar complementos de Stylus adicionales, como [Rupture](https://github.com/jescalan/rupture). Primero, instale el complemento en cuestión a través de NPM (`npm install rupture`) y luego solicítelo llamando a `mix.stylus()`:
> > You may also install additional Stylus plug-ins, such as [Rupture](https://github.com/jescalan/rupture). First, install the plug-in in question through NPM (`npm install rupture`) and then require it in your call to `mix.stylus()`:

    mix.stylus('resources/assets/stylus/app.styl', 'public/css', {
        use: [
            require('rupture')()
        ]
    });

<a name="postcss"></a>
### PostCSS

[PostCSS](http://postcss.org/), una poderosa herramienta para transformar su CSS, se incluye con Laravel Mix de fábrica. Por defecto, Mix aprovecha el popular plugin [Autoprefixer](https://github.com/postcss/autoprefixer) para aplicar automáticamente todos los prefijos de proveedor de CSS3 necesarios. Sin embargo, puede agregar complementos adicionales que sean apropiados para su aplicación. Primero, instale el complemento deseado a través de NPM y luego haga referencia a él en su archivo `webpack.mix.js`:
> > [PostCSS](http://postcss.org/), a powerful tool for transforming your CSS, is included with Laravel Mix out of the box. By default, Mix leverages the popular [Autoprefixer](https://github.com/postcss/autoprefixer) plug-in to automatically apply all necessary CSS3 vendor prefixes. However, you're free to add any additional plug-ins that are appropriate for your application. First, install the desired plug-in through NPM and then reference it in your `webpack.mix.js` file:

    mix.sass('resources/assets/sass/app.scss', 'public/css')
       .options({
            postCss: [
                require('postcss-css-variables')()
            ]
       });

<a name="plain-css"></a>
### CSS plano : Plain CSS

Si simplemente desea concatenar algunas hojas de estilos CSS simples en un solo archivo, puede usar el método `styles`.
> > If you would just like to concatenate some plain CSS stylesheets into a single file, you may use the `styles` method.

    mix.styles([
        'public/css/vendor/normalize.css',
        'public/css/vendor/videojs.css'
    ], 'public/css/all.css');

<a name="url-processing"></a>
### URL Processing

Debido a que Laravel Mix se basa en Webpack, es importante comprender algunos conceptos de Webpack. Para la compilación de CSS, Webpack reescribirá y optimizará cualquier llamada `url()` dentro de sus hojas de estilo. Si bien esto inicialmente puede sonar extraño, es una funcionalidad increíblemente poderosa. Imagine que queremos compilar Sass que incluye una URL relativa a una imagen:
> > Because Laravel Mix is built on top of Webpack, it's important to understand a few Webpack concepts. For CSS compilation, Webpack will rewrite and optimize any `url()` calls within your stylesheets. While this might initially sound strange, it's an incredibly powerful piece of functionality. Imagine that we want to compile Sass that includes a relative URL to an image:

    .example {
        background: url('../images/example.png');
    }

> {note} Las rutas absolutas para cualquier `url()` dado se excluirán de la reescritura de URL. Por ejemplo, `url('/images/thing.png')` o `url('http://example.com/images/thing.png')` no se modificarán.
> > > {note} Absolute paths for any given `url()` will be excluded from URL-rewriting. For example, `url('/images/thing.png')` or `url('http://example.com/images/thing.png')` won't be modified.

Por defecto, Laravel Mix y Webpack encontrarán `example.png`, lo copiarán en su carpeta `public/images`, y luego reescribirán `url()` dentro de su hoja de estilos generada. Como tal, su CSS compilado será:
> > By default, Laravel Mix and Webpack will find `example.png`, copy it to your `public/images` folder, and then rewrite the `url()` within your generated stylesheet. As such, your compiled CSS will be:

    .example {
      background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
    }

Tan útil como puede ser esta característica, es posible que su estructura de carpetas existente ya esté configurada de la manera que desee. Si este es el caso, puede desactivar la reescritura de `url()` como sigue:
> > As useful as this feature may be, it's possible that your existing folder structure is already configured in a way you like. If this is the case, you may disable `url()` rewriting like so:

    mix.sass('resources/assets/app/app.scss', 'public/css')
       .options({
          processCssUrls: false
       });

Con esta adición a su archivo `webpack.mix.js`, Mix ya no coincidirá con `url()` ni copiará activos en su directorio público. En otras palabras, el CSS compilado se verá exactamente como lo escribiste originalmente:
> > With this addition to your `webpack.mix.js` file, Mix will no longer match any `url()` or copy assets to your public directory. In other words, the compiled CSS will look just like how you originally typed it:

    .example {
        background: url("../images/thing.png");
    }

<a name="css-source-maps"></a>
### Mapas de origen : Source Maps

Aunque está deshabilitado por defecto, los mapas fuente pueden activarse llamando al método `mix.sourceMaps()` en su archivo `webpack.mix.js`. A pesar de que viene con un costo de compilación/rendimiento, esto proporcionará información de depuración adicional a las herramientas de desarrollo de su navegador al usar recursos compilados.
> > Though disabled by default, source maps may be activated by calling the `mix.sourceMaps()` method in your `webpack.mix.js` file. Though it comes with a compile/performance cost, this will provide extra debugging information to your browser's developer tools when using compiled assets.

    mix.js('resources/assets/js/app.js', 'public/js')
       .sourceMaps();

<a name="working-with-scripts"></a>
## Trabajar con JavaScript : Working With JavaScript

Mix proporciona varias características para ayudarlo a trabajar con sus archivos JavaScript, como la compilación de ECMAScript 2015, la agrupación de módulos, la minificación y la concatenación de archivos JavaScript simples. Aún mejor, todo esto funciona a la perfección, sin requerir una onza de configuración personalizada:
> > Mix provides several features to help you work with your JavaScript files, such as compiling ECMAScript 2015, module bundling, minification, and concatenating plain JavaScript files. Even better, this all works seamlessly, without requiring an ounce of custom configuration:

    mix.js('resources/assets/js/app.js', 'public/js');

Con esta única línea de código, ahora puede aprovechar:
> > With this single line of code, you may now take advantage of:

<div class="content-list" markdown="1">
- ES2015 syntax.
- Modules
- Compilation of `.vue` files.
- Minification for production environments.
</div>

<a name="vendor-extraction"></a>
### Vendor Extraction

Una posible desventaja de agrupar todo el JavaScript específico de la aplicación con las bibliotecas de sus proveedores es que dificulta el almacenamiento en caché a largo plazo. Por ejemplo, una sola actualización de su código de aplicación obligará al navegador a volver a descargar todas las bibliotecas de sus proveedores, incluso si no han cambiado.
> > One potential downside to bundling all application-specific JavaScript with your vendor libraries is that it makes long-term caching more difficult. For example, a single update to your application code will force the browser to re-download all of your vendor libraries even if they haven't changed.

Si tiene la intención de realizar actualizaciones frecuentes del JavaScript de su aplicación, debería considerar extraer todas sus bibliotecas de proveedores en su propio archivo. De esta forma, un cambio en el código de su aplicación no afectará el almacenamiento en caché de su gran archivo `vendor.js`. El método `extract` de Mix hace que esto sea muy sencillo:
> > If you intend to make frequent updates to your application's JavaScript, you should consider extracting all of your vendor libraries into their own file. This way, a change to your application code will not affect the caching of your large `vendor.js` file. Mix's `extract` method makes this a breeze:

    mix.js('resources/assets/js/app.js', 'public/js')
       .extract(['vue'])

El método `extract` acepta un array de todas las bibliotecas o módulos que desea extraer en un archivo `vendor.js`. Usando el fragmento de arriba como ejemplo, Mix generará los siguientes archivos:
> > The `extract` method accepts an array of all libraries or modules that you wish to extract into a `vendor.js` file. Using the above snippet as an example, Mix will generate the following files:

<div class="content-list" markdown="1">
- `public/js/manifest.js`: *The Webpack manifest runtime*
- `public/js/vendor.js`: *Your vendor libraries*
- `public/js/app.js`: *Your application code*
</div>

Para evitar errores de JavaScript, asegúrese de cargar estos archivos en el orden correcto:
> > To avoid JavaScript errors, be sure to load these files in the proper order:

    <script src="/js/manifest.js"></script>
    <script src="/js/vendor.js"></script>
    <script src="/js/app.js"></script>

<a name="react"></a>
### React

Mix puede instalar automáticamente los plug-ins de Babel necesarios para la compatibilidad con React. Para comenzar, reemplace su llamada `mix.js()` con `mix.react()`:
> > Mix can automatically install the Babel plug-ins necessary for React support. To get started, replace your `mix.js()` call with `mix.react()`:

    mix.react('resources/assets/js/app.jsx', 'public/js');

Detrás de escena, Mix descargará e incluirá el complemento `babel-preset-react` Babel apropiado.
> > Behind the scenes, Mix will download and include the appropriate `babel-preset-react` Babel plug-in.

<a name="vanilla-js"></a>
### Vanilla JS

Similar a la combinación de hojas de estilo con `mix.styles()`, también puedes combinar y minificar cualquier cantidad de archivos JavaScript con el método `scripts()`:
> > Similar to combining stylesheets with `mix.styles()`, you may also combine and minify any number of JavaScript files with the `scripts()` method:

    mix.scripts([
        'public/js/admin.js',
        'public/js/dashboard.js'
    ], 'public/js/all.js');

Esta opción es particularmente útil para proyectos heredados donde no se requiere compilación de Webpack para su JavaScript.
> > This option is particularly useful for legacy projects where you don't require Webpack compilation for your JavaScript.

> {tip} Una ligera variación de `mix.scripts()` es `mix.babel()`. Su firma de método es idéntica a `scripts`; sin embargo, el archivo concatenado recibirá la compilación de Babel, que traduce cualquier código de ES2015 a JavaScript de vanilla que todos los navegadores entenderán.
> > > {tip} A slight variation of `mix.scripts()` is `mix.babel()`. Its method signature is identical to `scripts`; however, the concatenated file will receive Babel compilation, which translates any ES2015 code to vanilla JavaScript that all browsers will understand.

<a name="custom-webpack-configuration"></a>
### Configuración de paquete web personalizado : Custom Webpack Configuration

Detrás de escena, Laravel Mix hace referencia a un archivo `webpack.config.js` preconfigurado para que pueda comenzar a trabajar lo más rápido posible. Ocasionalmente, puede necesitar modificar manualmente este archivo. Es posible que tenga un cargador o complemento especial al que se deba hacer referencia, o tal vez prefiera usar Stylus en lugar de Sass. En tales casos, tiene dos opciones:
> > Behind the scenes, Laravel Mix references a pre-configured `webpack.config.js` file to get you up and running as quickly as possible. Occasionally, you may need to manually modify this file. You might have a special loader or plug-in that needs to be referenced, or maybe you prefer to use Stylus instead of Sass. In such instances, you have two choices:

#### Merging Custom Configuration

Mix proporciona un método útil `webpackConfig` que le permite fusionar cualquier configuración corta de configuración de Webpack. Esta es una opción particularmente atractiva, ya que no requiere que copie y mantenga su propia copia del archivo `webpack.config.js`. El método `webpackConfig` acepta un objeto, que debe contener cualquier [configuración específica de Webpack](https://webpack.js.org/configuration/) que desee aplicar.
> > Mix provides a useful `webpackConfig` method that allows you to merge any short Webpack configuration overrides. This is a particularly appealing choice, as it doesn't require you to copy and maintain your own copy of the `webpack.config.js` file. The `webpackConfig` method accepts an object, which should contain any [Webpack-specific configuration](https://webpack.js.org/configuration/) that you wish to apply.

    mix.webpackConfig({
        resolve: {
            modules: [
                path.resolve(__dirname, 'vendor/laravel/spark/resources/assets/js')
            ]
        }
    });

#### Custom Configuration Files

Si desea personalizar completamente la configuración de su Webpack, copie el archivo `node_modules/laravel-mix/setup/webpack.config.js` en el directorio raíz de su proyecto. Luego, apunte todas las referencias `--config` en su archivo `package.json` al archivo de configuración recién copiado. Si opta por llevar este enfoque a la personalización, todas las actualizaciones futuras de upstream para `webpack.config.js` de Mix se deben fusionar manualmente en su archivo personalizado.
> > If you would like to completely customize your Webpack configuration, copy the `node_modules/laravel-mix/setup/webpack.config.js` file to your project's root directory. Next, point all of the `--config` references in your `package.json` file to the newly copied configuration file. If you choose to take this approach to customization, any future upstream updates to Mix's `webpack.config.js` must be manually merged into your customized file.

<a name="copying-files-and-directories"></a>
## Copiar archivos y directorios : Copying Files & Directories

El método `copy` se puede usar para copiar archivos y directorios a nuevas ubicaciones. Esto puede ser útil cuando un recurso particular dentro de su directorio `node_modules` necesita ser reubicado en su carpeta `public`.
> > The `copy` method may be used to copy files and directories to new locations. This can be useful when a particular asset within your `node_modules` directory needs to be relocated to your `public` folder.

    mix.copy('node_modules/foo/bar.css', 'public/css/bar.css');

Al copiar un directorio, el método `copy` aplanará la estructura del directorio. Para mantener la estructura original del directorio, debe usar el método `copyDirectory` en su lugar:
> > When copying a directory, the `copy` method will flatten the directory's structure. To maintain the directory's original structure, you should use the `copyDirectory` method instead:

    mix.copyDirectory('assets/img', 'public/img');

<a name="versioning-and-cache-busting"></a>
## Versioning / Cache Busting

Muchos desarrolladores sufilizan sus assets compilados con una marca de tiempo o un token único para obligar a los navegadores a cargar los activos nuevos en el lugar de publicación copias obsoletas del código. La mezcla puede manejar esto por ti usando el método `version`.
> > Many developers suffix their compiled assets with a timestamp or unique token to force browsers to load the fresh assets instead of serving stale copies of the code. Mix can handle this for you using the `version` method.

El método `version` agregará automáticamente un hash único a los nombres de archivo de todos los archivos compilados, lo que permite un almacenamiento en memoria caché más conveniente:
> > The `version` method will automatically append a unique hash to the filenames of all compiled files, allowing for more convenient cache busting:

    mix.js('resources/assets/js/app.js', 'public/js')
       .version();

Después de generar el archivo versionado, no sabrá el nombre exacto del archivo. Entonces, debes usar la función global `mix` de Laravel dentro de tus [views](/docs/{{version}}/views) para cargar el activo hash adecuado. La función `mix` determinará automáticamente el nombre actual del archivo hash:
> > After generating the versioned file, you won't know the exact file name. So, you should use Laravel's global `mix` function within your [views](/docs/{{version}}/views) to load the appropriately hashed asset. The `mix` function will automatically determine the current name of the hashed file:

    <link rel="stylesheet" href="{{ mix('/css/app.css') }}">

Debido a que los archivos versionados generalmente no son necesarios en el desarrollo, puede indicarle al proceso de versiones que solo se ejecute durante `npm run production`:
> > Because versioned files are usually unnecessary in development, you may instruct the versioning process to only run during `npm run production`:

    mix.js('resources/assets/js/app.js', 'public/js');

    if (mix.inProduction()) {
        mix.version();
    }

<a name="browsersync-reloading"></a>
## Recarga Browsersync : Browsersync Reloading

[BrowserSync](https://browsersync.io/) puede supervisar automáticamente sus archivos en busca de cambios e inyectar sus cambios en el navegador sin necesidad de una actualización manual. Puede habilitar el soporte llamando al método `mix.browserSync()`:
> > [BrowserSync](https://browsersync.io/) can automatically monitor your files for changes, and inject your changes into the browser without requiring a manual refresh. You may enable support by calling the `mix.browserSync()` method:

    mix.browserSync('my-domain.test');

    // Or...

    // https://browsersync.io/docs/options
    mix.browserSync({
        proxy: 'my-domain.test'
    });

Puede pasar un string (proxy) u objeto (configuración BrowserSync) a este método. Luego, inicie el servidor de desarrollo de Webpack usando el comando `npm run watch`. Ahora, cuando modifique una secuencia de comandos o un archivo PHP, observe cómo el navegador actualiza al instante la página para reflejar sus cambios.
> > You may pass either a string (proxy) or object (BrowserSync settings) to this method. Next, start Webpack's dev server using the `npm run watch` command. Now, when you modify a script or PHP file, watch as the browser instantly refreshes the page to reflect your changes.

<a name="environment-variables"></a>
## Variables de entorno : Environment Variables

Puede inyectar variables de entorno en Mix prefijando una clave en su archivo `.env` con` MIX_`:
> > You may inject environment variables into Mix by prefixing a key in your `.env` file with `MIX_`:

    MIX_SENTRY_DSN_PUBLIC=http://example.com

Después de que la variable se haya definido en su archivo `.env`, puede acceder a través del objeto `process.env`. Si el valor cambia mientras ejecuta una tarea `watch`, deberá reiniciar la tarea:
> > After the variable has been defined in your `.env` file, you may access via the `process.env` object. If the value changes while you are running a `watch` task, you will need to restart the task:

    process.env.MIX_SENTRY_DSN_PUBLIC

<a name="notifications"></a>
## Notificaciones : Notifications

Cuando esté disponible, Mix mostrará automáticamente las notificaciones del sistema operativo para cada paquete. Esto le dará una respuesta instantánea, si la compilación fue exitosa o no. Sin embargo, puede haber instancias en las que prefiera inhabilitar estas notificaciones. Uno de esos ejemplos podría ser activar Mix en su servidor de producción. Las notificaciones pueden ser desactivadas, a través del método `disableNotifications`.
> > When available, Mix will automatically display OS notifications for each bundle. This will give you instant feedback, as to whether the compilation was successful or not. However, there may be instances when you'd prefer to disable these notifications. One such example might be triggering Mix on your production server. Notifications may be deactivated, via the `disableNotifications` method.

    mix.disableNotifications();
