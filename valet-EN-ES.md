# Laravel Valet

- [Introduction](#introduction)
    - [Valet Or Homestead](#valet-or-homestead)
- [Installation](#installation)
    - [Upgrading](#upgrading)
- [Serving Sites](#serving-sites)
    - [The "Park" Command](#the-park-command)
    - [The "Link" Command](#the-link-command)
    - [Securing Sites With TLS](#securing-sites)
- [Sharing Sites](#sharing-sites)
- [Custom Valet Drivers](#custom-valet-drivers)
    - [Local Drivers](#local-drivers)
- [Other Valet Commands](#other-valet-commands)

<a name="introduction"></a>
## Introducción : Introduction

Valet es un entorno de desarrollo de Laravel para Mac minimalistas. Sin Vagrant, no hay archivo `/etc/hosts`. Incluso puedes compartir tus sitios públicamente usando túneles locales. Sí, nos gusta también.
> > Valet is a Laravel development environment for Mac minimalists. No Vagrant, no `/etc/hosts` file. You can even share your sites publicly using local tunnels. _Yeah, we like it too._

Laravel Valet configura su Mac para ejecutar siempre [Nginx](https://www.nginx.com/) en segundo plano cuando se inicia la máquina. Luego, al usar [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq), Valet procesa todas las solicitudes en el dominio `*.test` para que apunten a los sitios instalados en su máquina local.
> > Laravel Valet configures your Mac to always run [Nginx](https://www.nginx.com/) in the background when your machine starts. Then, using [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq), Valet proxies all requests on the `*.test` domain to point to sites installed on your local machine.

En otras palabras, un entorno de desarrollo de Laravel extremadamente rápido que utiliza aproximadamente 7 MB de RAM. Valet no es un reemplazo completo para Vagrant o Homestead, pero ofrece una gran alternativa si desea información básica flexible, prefiere la velocidad extrema o está trabajando en una máquina con una cantidad limitada de RAM.
> > In other words, a blazing fast Laravel development environment that uses roughly 7 MB of RAM. Valet isn't a complete replacement for Vagrant or Homestead, but provides a great alternative if you want flexible basics, prefer extreme speed, or are working on a machine with a limited amount of RAM.

Fuera de la caja, el soporte de Valet incluye, pero no se limita a:
> > Out of the box, Valet support includes, but is not limited to:

<div class="content-list" markdown="1">
- [Laravel](https://laravel.com)
- [Lumen](https://lumen.laravel.com)
- [Bedrock](https://roots.io/bedrock/)
- [CakePHP 3](https://cakephp.org)
- [Concrete5](https://www.concrete5.org/)
- [Contao](https://contao.org/en/)
- [Craft](https://craftcms.com)
- [Drupal](https://www.drupal.org/)
- [Jigsaw](http://jigsaw.tighten.co)
- [Joomla](https://www.joomla.org/)
- [Katana](https://github.com/themsaid/katana)
- [Kirby](https://getkirby.com/)
- [Magento](https://magento.com/)
- [OctoberCMS](https://octobercms.com/)
- [Sculpin](https://sculpin.io/)
- [Slim](https://www.slimframework.com)
- [Statamic](https://statamic.com)
- Static HTML
- [Symfony](https://symfony.com)
- [WordPress](https://wordpress.org)
- [Zend](https://framework.zend.com)
</div>

Sin embargo, puede extender Valet con sus propios [controladores personalizados] (# custom-valet-drivers).
> > However, you may extend Valet with your own [custom drivers](#custom-valet-drivers).

<a name="valet-or-homestead"></a>
### Valet o Homestead : Valet Or Homestead

Como sabrá, Laravel ofrece [Homestead](/docs/{{version}}/homestead), otro entorno de desarrollo local de Laravel. Homestead y Valet difieren en cuanto a su público objetivo y su enfoque del desarrollo local. Homestead ofrece una máquina virtual completa de Ubuntu con configuración automatizada de Nginx. Homestead es una elección maravillosa si desea un entorno de desarrollo de Linux completamente virtualizado o está en Windows / Linux.
> > As you may know, Laravel offers [Homestead](/docs/{{version}}/homestead), another local Laravel development environment. Homestead and Valet differ in regards to their intended audience and their approach to local development. Homestead offers an entire Ubuntu virtual machine with automated Nginx configuration. Homestead is a wonderful choice if you want a fully virtualized Linux development environment or are on Windows / Linux.

Valet solo es compatible con Mac, y requiere que instale PHP y un servidor de base de datos directamente en su máquina local. Esto se logra fácilmente usando [Homebrew](http://brew.sh/) con comandos como `brew install php` y `brew install mysql`. Valet proporciona un entorno de desarrollo local extremadamente rápido con un consumo mínimo de recursos, por lo que es ideal para desarrolladores que solo requieren PHP / MySQL y no necesitan un entorno de desarrollo totalmente virtualizado.
> > Valet only supports Mac, and requires you to install PHP and a database server directly onto your local machine. This is easily achieved by using [Homebrew](http://brew.sh/) with commands like `brew install php` and `brew install mysql`. Valet provides a blazing fast local development environment with minimal resource consumption, so it's great for developers who only require PHP / MySQL and do not need a fully virtualized development environment.

Tanto Valet como Homestead son excelentes opciones para configurar su entorno de desarrollo de Laravel. El que elija dependerá de su gusto personal y las necesidades de su equipo.
> > Both Valet and Homestead are great choices for configuring your Laravel development environment. Which one you choose will depend on your personal taste and your team's needs.

<a name="installation"></a>
## Instalación : Installation

**El valet requiere macOS y [Homebrew](http://brew.sh/). Antes de la instalación, debe asegurarse de que ningún otro programa como Apache o Nginx sea vinculante para el puerto 80 de su máquina local.**
> > **Valet requires macOS and [Homebrew](http://brew.sh/). Before installation, you should make sure that no other programs such as Apache or Nginx are binding to your local machine's port 80.**

<div class="content-list" markdown="1">
- Install or update [Homebrew](http://brew.sh/) to the latest version using `brew update`.
- Install PHP 7.2 using Homebrew via `brew install php@7.2`.
- Install Valet with Composer via `composer global require laravel/valet`. Make sure the `~/.composer/vendor/bin` directory is in your system's "PATH".
- Run the `valet install` command. This will configure and install Valet and DnsMasq, and register Valet's daemon to launch when your system starts.
</div>

Una vez que Valet está instalado, intente hacer ping a cualquier dominio `*.test` en su terminal usando un comando como `ping foobar.test`. Si Valet está instalado correctamente, debería ver este dominio respondiendo `127.0.0.1`.
> > Once Valet is installed, try pinging any `*.test` domain on your terminal using a command such as `ping foobar.test`. If Valet is installed correctly you should see this domain responding on `127.0.0.1`.

Valet iniciará automáticamente su demonio cada vez que arranque la máquina. No es necesario ejecutar `valet start` o `valet install` nunca más una vez que se completa la instalación inicial de Valet.
> > Valet will automatically start its daemon each time your machine boots. There is no need to run `valet start` or `valet install` ever again once the initial Valet installation is complete.

#### Usando otro dominio : Using Another Domain

Por defecto, Valet sirve sus proyectos usando el TLD `.test`. Si desea utilizar otro dominio, puede hacerlo utilizando el comando `valet domain tld-name`.
> > By default, Valet serves your projects using the `.test` TLD. If you'd like to use another domain, you can do so using the `valet domain tld-name` command.

Por ejemplo, si desea utilizar `.app` en lugar de `.test`, ejecute `valet domain app` y Valet comenzará a publicar sus proyectos en `*.app` automáticamente.
> > For example, if you'd like to use `.app` instead of `.test`, run `valet domain app` and Valet will start serving your projects at `*.app` automatically.

#### Base de datos : Database

Si necesita una base de datos, intente con MySQL ejecutando `brew install mysql@5.7` en su línea de comando. Una vez que se haya instalado MySQL, puede iniciarlo utilizando el comando `brew services start mysql`. A continuación, puede conectarse a la base de datos en `127.0.0.1` utilizando el nombre de usuario `root` y una cadena vacía para la contraseña.
> > If you need a database, try MySQL by running `brew install mysql@5.7` on your command line. Once MySQL has been installed, you may start it using the `brew services start mysql` command. You can then connect to the database at `127.0.0.1` using the `root` username and an empty string for the password.

<a name="upgrading"></a>
### Actualización : Upgrading

Puede actualizar su instalación de Valet utilizando el comando `composer global update` en su terminal. Después de la actualización, es una buena práctica ejecutar el comando `valet install` para que Valet pueda realizar actualizaciones adicionales a sus archivos de configuración si es necesario.
> > You may update your Valet installation using the `composer global update` command in your terminal. After upgrading, it is good practice to run the `valet install` command so Valet can make additional upgrades to your configuration files if necessary.

#### Actualización a Valet 2.0 : Upgrading To Valet 2.0

Valet 2.0 transiciona el servidor web subyacente de Valet de Caddy a Nginx. Antes de actualizar a esta versión, debe ejecutar los siguientes comandos para detener y desinstalar el Caddie Daemon existente:
> > Valet 2.0 transitions Valet's underlying web server from Caddy to Nginx. Before upgrading to this version you should run the following commands to stop and uninstall the existing Caddy daemon:

    valet stop
    valet uninstall

A continuación, debe actualizar a la última versión de Valet. Dependiendo de cómo instaló Valet, esto generalmente se hace a través de Git o Composer. Si instaló Valet a través de Composer, debe usar el siguiente comando para actualizar a la última versión principal:
> > Next, you should upgrade to the latest version of Valet. Depending on how you installed Valet, this is typically done through Git or Composer. If you installed Valet via Composer, you should use the following command to update to the latest major version:

    composer global require laravel/valet

Una vez que se haya descargado el código fuente nuevo de Valet, debe ejecutar el comando `install`:
> > Once the fresh Valet source code has been downloaded, you should run the `install` command:

    valet install
    valet restart

Después de la actualización, puede ser necesario volver a aparcar o volver a vincular sus sitios.
> > After upgrading, it may be necessary to re-park or re-link your sites.

<a name="serving-sites"></a>
## Sirviendo sitios : Serving Sites

Una vez que Valet está instalado, está listo para comenzar a servir sitios. Valet proporciona dos comandos para ayudarlo a servir sus sitios de Laravel: `park` y `link`.
> > Once Valet is installed, you're ready to start serving sites. Valet provides two commands to help you serve your Laravel sites: `park` and `link`.

<a name="the-park-command"></a>
**The `park` Command**

<div class="content-list" markdown="1">
- Create a new directory on your Mac by running something like `mkdir ~/Sites`. Next, `cd ~/Sites` and run `valet park`. This command will register your current working directory as a path that Valet should search for sites.
- Next, create a new Laravel site within this directory: `laravel new blog`.
- Open `http://blog.test` in your browser.
</div>

**Eso es todo lo que hay que hacer.** Ahora, cualquier proyecto de Laravel que crees dentro de tu directorio "estacionado" se servirá automáticamente utilizando la convención `http://folder-name.test`.
> > **That's all there is to it.** Now, any Laravel project you create within your "parked" directory will automatically be served using the `http://folder-name.test` convention.

<a name="the-link-command"></a>
**The `link` Command**

El comando `link` también se puede usar para servir a sus sitios Laravel. Este comando es útil si desea servir un solo sitio en un directorio y no en todo el directorio.
> > The `link` command may also be used to serve your Laravel sites. This command is useful if you want to serve a single site in a directory and not the entire directory.

<div class="content-list" markdown="1">
- Para usar el comando, vaya a uno de sus proyectos y ejecute `valet link app-name` en su terminal. Valet creará un enlace simbólico en `~/.valet/Sites` que apunta a su directorio de trabajo actual.
- Después de ejecutar el comando `link`, puede acceder al sitio en su navegador en `http://app-name.test`.
> > - To use the command, navigate to one of your projects and run `valet link app-name` in your terminal. Valet will create a symbolic link in `~/.valet/Sites` which points to your current working directory.
> > - After running the `link` command, you can access the site in your browser at `http://app-name.test`.
</div>

Para ver una lista de todos sus directorios vinculados, ejecute el comando `valet links`. Puede usar `valet unlink app-name` para destruir el enlace simbólico.
> > To see a listing of all of your linked directories, run the `valet links` command. You may use `valet unlink app-name` to destroy the symbolic link.

> {tip} Puede usar `valet link` para servir el mismo proyecto desde múltiples (sub)dominios. Para agregar un subdominio u otro dominio a su proyecto ejecute `valet link subdomain.app-name` desde la carpeta del proyecto.
> > > {tip} You can use `valet link` to serve the same project from multiple (sub)domains. To add a subdomain or another domain to your project run `valet link subdomain.app-name` from the project folder.

<a name="securing-sites"></a>
**Asegurar sitios con TLS**
> > **Securing Sites With TLS**

Por defecto, Valet sirve a los sitios a través de HTTP simple. Sin embargo, si desea servir un sitio sobre TLS encriptado usando HTTP/2, use el comando `secure`. Por ejemplo, si su servidor está siendo servido por Valet en el dominio `laravel.test`, debe ejecutar el siguiente comando para asegurarlo:
> > By default, Valet serves sites over plain HTTP. However, if you would like to serve a site over encrypted TLS using HTTP/2, use the `secure` command. For example, if your site is being served by Valet on the `laravel.test` domain, you should run the following command to secure it:

    valet secure laravel

Para "desproteger" un sitio y volver a servir su tráfico a través de HTTP simple, use el comando `unsecure`. Al igual que el comando `secure`, este comando acepta el nombre de host que desea anular:
> > To "unsecure" a site and revert back to serving its traffic over plain HTTP, use the `unsecure` command. Like the `secure` command, this command accepts the host name that you wish to unsecure:

    valet unsecure laravel

<a name="sharing-sites"></a>
## Compartir sitios : Sharing Sites

Valet incluye un comando para compartir sus sitios locales con el mundo. No se requiere instalación de software adicional una vez que se instala Valet.
> > Valet even includes a command to share your local sites with the world. No additional software installation is required once Valet is installed.

Para compartir un sitio, vaya al directorio del sitio en su terminal y ejecute el comando `valet share`. Se insertará una URL pública en su portapapeles y estará lista para pegar directamente en su navegador. Eso es.
> > To share a site, navigate to the site's directory in your terminal and run the `valet share` command. A publicly accessible URL will be inserted into your clipboard and is ready to paste directly into your browser. That's it.

Para dejar de compartir su sitio, presione `Control + C` para cancelar el proceso.
> > To stop sharing your site, hit `Control + C` to cancel the process.

> {note} `valet share` no admite sitios compartidos que hayan sido asegurados usando el comando `valet secure`.
> > > {note} `valet share` does not currently support sharing sites that have been secured using the `valet secure` command.

<a name="custom-valet-drivers"></a>
## Drivers de Valet personalizados : Custom Valet Drivers

Puede escribir su propio "controlador" de Valet para que sirva las aplicaciones PHP que se ejecutan en otro framework o CMS que no es compatible nativamente con Valet. Cuando instala Valet, se crea un directorio `~/.valet/Drivers` que contiene un archivo `SampleValetDriver.php`. Este archivo contiene una implementación de controlador de muestra para demostrar cómo escribir un controlador personalizado. Escribir un controlador solo requiere que implemente tres métodos: `serve`, `isStaticFile`, y `frontControllerPath`.
> > You can write your own Valet "driver" to serve PHP applications running on another framework or CMS that is not natively supported by Valet. When you install Valet, a `~/.valet/Drivers` directory is created which contains a `SampleValetDriver.php` file. This file contains a sample driver implementation to demonstrate how to write a custom driver. Writing a driver only requires you to implement three methods: `serves`, `isStaticFile`, and `frontControllerPath`.

Los tres métodos reciben los valores `$sitePath`, `$siteName`, y `$uri` como argumentos. El `$sitePath` es la ruta completa al sitio que se está publicando en su máquina, como `/Users/Lisa/Sites/my-project`. El `$siteName` es la porción de "host" / "nombre del sitio" del dominio (`my-project`). La `$uri` es la URI de solicitud entrante (`/foo/bar`).
> > All three methods receive the `$sitePath`, `$siteName`, and `$uri` values as their arguments. The `$sitePath` is the fully qualified path to the site being served on your machine, such as `/Users/Lisa/Sites/my-project`. The `$siteName` is the "host" / "site name" portion of the domain (`my-project`). The `$uri` is the incoming request URI (`/foo/bar`).

Una vez que haya completado su controlador de Valet personalizado, colóquelo en el directorio `~/.valet/Drivers` utilizando la convención de nomenclatura `FrameworkValetDriver.php`. Por ejemplo, si está escribiendo un controlador de Valet personalizado para WordPress, su nombre de archivo debe ser `WordPressValetDriver.php`.
> > Once you have completed your custom Valet driver, place it in the `~/.valet/Drivers` directory using the `FrameworkValetDriver.php` naming convention. For example, if you are writing a custom valet driver for WordPress, your file name should be `WordPressValetDriver.php`.

Echemos un vistazo a una implementación de ejemplo de cada método que su driver Valet personalizado debería implementar.
> > Let's take a look at a sample implementation of each method your custom Valet driver should implement.

#### El método `serves` : The `serves` Method

El método `serves` debe devolver `true` si su driver debe manejar la solicitud entrante. De lo contrario, el método debería devolver `false`. Entonces, dentro de este método, debe intentar determinar si el `$sitePath` dado contiene un proyecto del tipo que está tratando de publicar.
> > The `serves` method should return `true` if your driver should handle the incoming request. Otherwise, the method should return `false`. So, within this method you should attempt to determine if the given `$sitePath` contains a project of the type you are trying to serve.

Por ejemplo, imaginemos que estamos escribiendo un `WordPressValetDriver`. Nuestro método `serves` podría verse más o menos así:
> > For example, let's pretend we are writing a `WordPressValetDriver`. Our `serves` method might look something like this:

    /**
     * Determine if the driver serves the request.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return bool
     */
    public function serves($sitePath, $siteName, $uri)
    {
        return is_dir($sitePath.'/wp-admin');
    }

#### El método `isStaticFile` : The `isStaticFile` Method

El `isStaticFile` debe determinar si la solicitud entrante es para un archivo "estático", como una imagen o una hoja de estilo. Si el archivo es estático, el método debe devolver la ruta completa al archivo estático en el disco. Si la solicitud entrante no es para un archivo estático, el método debe devolver `false`:
> > The `isStaticFile` should determine if the incoming request is for a file that is "static", such as an image or a stylesheet. If the file is static, the method should return the fully qualified path to the static file on disk. If the incoming request is not for a static file, the method should return `false`:

    /**
     * Determine if the incoming request is for a static file.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string|false
     */
    public function isStaticFile($sitePath, $siteName, $uri)
    {
        if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
            return $staticFilePath;
        }

        return false;
    }

> {note} El método `isStaticFile` solo se invocará si el método `serves` devuelve `true` para la solicitud entrante y el URI de solicitud no es `/`.
> > > {note} The `isStaticFile` method will only be called if the `serves` method returns `true` for the incoming request and the request URI is not `/`.

#### El método `frontControllerPath` : The `frontControllerPath` Method

El método `frontControllerPath` debe devolver la ruta completa al "controlador frontal" de su aplicación, que generalmente es su archivo "index.php" o equivalente:
> > The `frontControllerPath` method should return the fully qualified path to your application's "front controller", which is typically your "index.php" file or equivalent:

    /**
     * Get the fully resolved path to the application's front controller.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string
     */
    public function frontControllerPath($sitePath, $siteName, $uri)
    {
        return $sitePath.'/public/index.php';
    }

<a name="local-drivers"></a>
### Drivers locales : Local Drivers

Si desea definir un controlador Valet personalizado para una sola aplicación, cree un `LocalValetDriver.php` en el directorio raíz de la aplicación. Su driver personalizado puede extender la clase base `ValetDriver` o extender un driver existente específico de la aplicación, como `LaravelValetDriver`:
> > If you would like to define a custom Valet driver for a single application, create a `LocalValetDriver.php` in the application's root directory. Your custom driver may extend the base `ValetDriver` class or extend an existing application specific driver such as the `LaravelValetDriver`:

    class LocalValetDriver extends LaravelValetDriver
    {
        /**
         * Determine if the driver serves the request.
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return bool
         */
        public function serves($sitePath, $siteName, $uri)
        {
            return true;
        }

        /**
         * Get the fully resolved path to the application's front controller.
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return string
         */
        public function frontControllerPath($sitePath, $siteName, $uri)
        {
            return $sitePath.'/public_html/index.php';
        }
    }

<a name="other-valet-commands"></a>
## Otros comandos de Valet : Other Valet Commands

Command  | Descripción | Description
------------- | ------------- | -------------
`valet forget` | Ejecute este comando desde un directorio "aparcado" para eliminarlo de la lista de directorios aparcados. | Run this command from a "parked" directory to remove it from the parked directory list.
`valet paths` | Ver todas sus rutas "estacionadas". | View all of your "parked" paths.
`valet restart` | Reiniciar el daemon Valet. | Restart the Valet daemon.
`valet start` | Iniciar el daemon Valet. | Start the Valet daemon.
`valet stop` | Detener el demonio Valet. | Stop the Valet daemon.
`valet uninstall` | Desinstale por completo el demonio de Valet. | Uninstall the Valet daemon entirely.
