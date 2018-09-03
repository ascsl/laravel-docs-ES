# Instalación : Installation

- [Installation](#installation)
    - [Server Requirements](#server-requirements)
    - [Installing Laravel](#installing-laravel)
    - [Configuration](#configuration)
- [Web Server Configuration](#web-server-configuration)
    - [Pretty URLs](#pretty-urls)

<a name="installation"></a>
## Instalación : Installation

> {video} Laracasts brinda una [introducción completa y gratuita a Laravel](http://laravelfromscratch.com) para los recién llegados al framework. Es un excelente lugar para comenzar tu viaje.
> > > {video} Laracasts provides a [free, thorough introduction to Laravel](http://laravelfromscratch.com) for newcomers to the framework. It's a great place to start your journey.

<a name="server-requirements"></a>
### Requisitos del servidor : Server Requirements

El framework de Laravel tiene algunos requisitos del sistema. Por supuesto, todos estos requisitos son satisfechos por la máquina virtual [Laravel Homestead](/docs/{{version}}/homestead), por lo que es muy recomendable que utilice Homestead como su entorno de desarrollo local de Laravel.
> > The Laravel framework has a few system requirements. Of course, all of these requirements are satisfied by the [Laravel Homestead](/docs/{{version}}/homestead) virtual machine, so it's highly recommended that you use Homestead as your local Laravel development environment.

Sin embargo, si no está utilizando Homestead, deberá asegurarse de que su servidor cumpla con los siguientes requisitos:
> > However, if you are not using Homestead, you will need to make sure your server meets the following requirements:

<div class="content-list" markdown="1">
- PHP >= 7.1.3
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension
- Ctype PHP Extension
- JSON PHP Extension
</div>

<a name="installing-laravel"></a>
### Instalación de Laravel : Installing Laravel

Laravel utiliza [Composer](https://getcomposer.org) para administrar sus dependencias. Entonces, antes de usar Laravel, asegúrese de tener Composer instalado en su máquina.
> > Laravel utilizes [Composer](https://getcomposer.org) to manage its dependencies. So, before using Laravel, make sure you have Composer installed on your machine.

#### Instalador de Laravel : Via Laravel Installer

Primero, descargue el instalador de Laravel usando Composer:
> > First, download the Laravel installer using Composer:

    composer global require "laravel/installer"

Asegúrese de ubicar el directorio bin del proveedor del sistema en su `$PATH` para que el ejecutable laravel pueda ser localizado por su sistema. Este directorio existe en diferentes ubicaciones en función de su sistema operativo; Sin embargo, algunas ubicaciones comunes incluyen:
> > Make sure to place composer's system-wide vendor bin directory in your `$PATH` so the laravel executable can be located by your system. This directory exists in different locations based on your operating system; however, some common locations include:

<div class="content-list" markdown="1">
- macOS: `$HOME/.composer/vendor/bin`
- GNU / Linux Distributions: `$HOME/.config/composer/vendor/bin`
</div>

Una vez instalado, el comando `laravel new` creará una nueva instalación de Laravel en el directorio que especifique. Por ejemplo, `laravel new blog` creará un directorio llamado `blog` que contiene una nueva instalación de Laravel con todas las dependencias de Laravel ya instaladas:
> > Once installed, the `laravel new` command will create a fresh Laravel installation in the directory you specify. For instance, `laravel new blog` will create a directory named `blog` containing a fresh Laravel installation with all of Laravel's dependencies already installed:

    laravel new blog

#### A través de Composer Create-Project : Via Composer Create-Project

Alternativamente, también puede instalar Laravel con el comando Composer `create-project` en su terminal:
> > Alternatively, you may also install Laravel by issuing the Composer `create-project` command in your terminal:

    composer create-project --prefer-dist laravel/laravel blog

#### Servidor de desarrollo local : Local Development Server

Si tiene PHP instalado localmente y desea usar el servidor de desarrollo incorporado de PHP para servir su aplicación, puede usar el comando `serve` Artisan. Este comando iniciará un servidor de desarrollo en `http://localhost:8000`:
> > If you have PHP installed locally and you would like to use PHP's built-in development server to serve your application, you may use the `serve` Artisan command. This command will start a development server at `http://localhost:8000`:

    php artisan serve

Por supuesto, hay disponibles opciones de desarrollo local más sólidas a través de [Homestead](/docs/{{version}}/homestead) y [Valet](/docs/{{version}}/valet).
> > Of course, more robust local development options are available via [Homestead](/docs/{{version}}/homestead) and [Valet](/docs/{{version}}/valet).

<a name="configuration"></a>
### Configuración : Configuration

#### Directorio público : Public Directory

Después de instalar Laravel, debe configurar el documento / raíz web del servidor web para que sea el directorio `public`. El `index.php` en este directorio sirve como controlador frontal para todas las solicitudes HTTP que ingresan a su aplicación.
> > After installing Laravel, you should configure your web server's document / web root to be the `public` directory. The `index.php` in this directory serves as the front controller for all HTTP requests entering your application.

#### Archivos de configuración : Configuration Files

Todos los archivos de configuración para el framework de Laravel se almacenan en el directorio `config`. Cada opción está documentada, así que no dude en consultar los archivos y familiarizarse con las opciones disponibles para usted.
> > All of the configuration files for the Laravel framework are stored in the `config` directory. Each option is documented, so feel free to look through the files and get familiar with the options available to you.

#### Permisos de directorio : Directory Permissions

Después de instalar Laravel, es posible que deba configurar algunos permisos. Los directorios dentro de los directorios `storage` y `bootstrap/cache` deben ser editables por su servidor web o Laravel no se ejecutará. Si está utilizando la máquina virtual [Homestead](/docs/{{version}}/homestead), estos permisos ya deberían estar configurados.
> > After installing Laravel, you may need to configure some permissions. Directories within the `storage` and the `bootstrap/cache` directories should be writable by your web server or Laravel will not run. If you are using the [Homestead](/docs/{{version}}/homestead) virtual machine, these permissions should already be set.

#### Clave de aplicación : Application Key

Lo siguiente que debe hacer después de instalar Laravel es establecer su clave de aplicación en una cadena aleatoria. Si instaló Laravel mediante Composer o el instalador de Laravel, esta clave ya se configuró para usted con el comando `php artisan key:generate`.
> > The next thing you should do after installing Laravel is set your application key to a random string. If you installed Laravel via Composer or the Laravel installer, this key has already been set for you by the `php artisan key:generate` command.

Normalmente, esta cadena debe tener 32 caracteres de longitud. La clave se puede establecer en el archivo de entorno `.env`. Si no ha cambiado el nombre del archivo `.env.example` a `.env`, debe hacerlo ahora. **Si la clave de la aplicación no está configurada, ¡sus sesiones de usuario y otros datos encriptados no serán seguros!**
> > Typically, this string should be 32 characters long. The key can be set in the `.env` environment file. If you have not renamed the `.env.example` file to `.env`, you should do that now. **If the application key is not set, your user sessions and other encrypted data will not be secure!**

#### Configuración adicional : Additional Configuration

Laravel casi no necesita otra configuración de la caja. ¡Eres libre de comenzar a desarrollar! Sin embargo, es posible que desee revisar el archivo `config/app.php` y su documentación. Contiene varias opciones, como `timezone` y `locale`, que es posible que desee cambiar de acuerdo con su aplicación.
> > Laravel needs almost no other configuration out of the box. You are free to get started developing! However, you may wish to review the `config/app.php` file and its documentation. It contains several options such as `timezone` and `locale` that you may wish to change according to your application.

Es posible que también desee configurar algunos componentes adicionales de Laravel, como:
> > You may also want to configure a few additional components of Laravel, such as:

<div class="content-list" markdown="1">
- [Cache](/docs/{{version}}/cache#configuration)
- [Database](/docs/{{version}}/database#configuration)
- [Session](/docs/{{version}}/session#configuration)
</div>

<a name="web-server-configuration"></a>
## Configuración del servidor web : Web Server Configuration

<a name="pretty-urls"></a>
### URLs amigables : Pretty URLs

#### Apache

Laravel includes a `public/.htaccess` file that is used to provide URLs without the `index.php` front controller in the path. Before serving Laravel with Apache, be sure to enable the `mod_rewrite` module so the `.htaccess` file will be honored by the server.

Laravel incluye un archivo `public/.htaccess` que se utiliza para proporcionar URL sin el controlador frontal `index.php` en la ruta. Antes de servir a Laravel con Apache, asegúrese de habilitar el módulo `mod_rewrite` para que el servidor cumpla con el archivo `.htaccess`.
> > If the `.htaccess` file that ships with Laravel does not work with your Apache installation, try this alternative:

    Options +FollowSymLinks -Indexes
    RewriteEngine On

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

#### Nginx

Si está utilizando Nginx, la siguiente directiva en la configuración de su sitio dirigirá todas las solicitudes al controlador frontal `index.php`:
> > If you are using Nginx, the following directive in your site configuration will direct all requests to the `index.php` front controller:

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

Por supuesto, al usar [Homestead](/docs/{{version}}/homestead) o [Valet](/docs/{{version}}/valet), las URL amigables se configurarán automáticamente.
> > Of course, when using [Homestead](/docs/{{version}}/homestead) or [Valet](/docs/{{version}}/valet), pretty URLs will be automatically configured.
