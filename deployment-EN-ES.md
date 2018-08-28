# Despliegue : Deployment

- [Introduction](#introduction)
- [Server Configuration](#server-configuration)
    - [Nginx](#nginx)
- [Optimization](#optimization)
    - [Autoloader Optimization](#autoloader-optimization)
    - [Optimizing Configuration Loading](#optimizing-configuration-loading)
    - [Optimizing Route Loading](#optimizing-route-loading)
- [Deploying With Forge](#deploying-with-forge)

<a name="introduction"></a>
## Introducción : Introduction

Cuando esté listo para implementar su aplicación Laravel en producción, hay algunas cosas importantes que puede hacer para asegurarse de que su aplicación se ejecute de la manera más eficiente posible. En este documento, cubriremos algunos puntos de partida para asegurarnos de que su aplicación Laravel se implemente correctamente.
> > When you're ready to deploy your Laravel application to production, there are some important things you can do to make sure your application is running as efficiently as possible. In this document, we'll cover some great starting points for making sure your Laravel application is deployed properly.

<a name="server-configuration"></a>
## Configuración del servidor : Server Configuration

<a name="nginx"></a>
### Nginx

Si está implementando su aplicación en un servidor que está ejecutando Nginx, puede usar el siguiente archivo de configuración como punto de partida para configurar su servidor web. Lo más probable es que este archivo deba personalizarse según la configuración de su servidor. Si necesita ayuda para administrar su servidor, considere usar un servicio como [Laravel Forge](https://forge.laravel.com):
> > If you are deploying your application to a server that is running Nginx, you may use the following configuration file as a starting point for configuring your web server. Most likely, this file will need to be customized depending on your server's configuration. If you would like assistance in managing your server, consider using a service such as [Laravel Forge](https://forge.laravel.com):

    server {
        listen 80;
        server_name example.com;
        root /example.com/public;

        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        index index.html index.htm index.php;

        charset utf-8;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        error_page 404 /index.php;

        location ~ \.php$ {
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php/php7.1-fpm.sock;
            fastcgi_index index.php;
            include fastcgi_params;
        }

        location ~ /\.(?!well-known).* {
            deny all;
        }
    }

<a name="optimization"></a>
## Optimización : Optimization

<a name="autoloader-optimization"></a>
### Optimización de autocarga : Autoloader Optimization

Al implementar en producción, asegúrese de estar optimizando el mapa del autocargador de clases de Composer para que Composer pueda encontrar rápidamente el archivo adecuado para cargar para una clase determinada:
> > When deploying to production, make sure that you are optimizing Composer's class autoloader map so Composer can quickly find the proper file to load for a given class:

    composer install --optimize-autoloader --no-dev

> {tip} Además de optimizar el autocargador, siempre debe asegurarse de incluir un archivo `composer.lock` en el repositorio de control de origen de su proyecto. Las dependencias de su proyecto se pueden instalar mucho más rápido cuando está presente un archivo `composer.lock`.
> > > {tip} In addition to optimizing the autoloader, you should always be sure to include a `composer.lock` file in your project's source control repository. Your project's dependencies can be installed much faster when a `composer.lock` file is present.

<a name="optimizing-configuration-loading"></a>
### Optimización de la configuración de carga : Optimizing Configuration Loading

Al implementar su aplicación en producción, debe asegurarse de ejecutar el comando Artisan `config:cache` durante su proceso de implementación:
> > When deploying your application to production, you should make sure that you run the `config:cache` Artisan command during your deployment process:

    php artisan config:cache

Este comando combinará todos los archivos de configuración de Laravel en un único archivo en caché, lo que reduce en gran medida la cantidad de viajes que el marco debe realizar al sistema de archivos al cargar los valores de configuración.
> > This command will combine all of Laravel's configuration files into a single, cached file, which greatly reduces the number of trips the framework must make to the filesystem when loading your configuration values.

> {note} Si ejecuta el comando `config:cache` durante su proceso de implementación, debe asegurarse de llamar solo a la función `env` desde sus archivos de configuración. Una vez que la configuración ha sido almacenada en caché, el archivo `.env` no se cargará y todas las llamadas a la función `env` devolverán `null`.
> > > {note} If you execute the `config:cache` command during your deployment process, you should be sure that you are only calling the `env` function from within your configuration files. Once the configuration has been cached, the `.env` file will not be loaded and all calls to the `env` function will return `null`.

<a name="optimizing-route-loading"></a>
### Optimizar la ruta de carga : Optimizing Route Loading

Si está compilando una aplicación grande con muchas rutas, debe asegurarse de que está ejecutando el comando `route:cache` Artisan durante su proceso de implementación:
> > If you are building a large application with many routes, you should make sure that you are running the `route:cache` Artisan command during your deployment process:

    php artisan route:cache

Este comando reduce todos los registros de ruta en una única llamada al método dentro de un archivo en caché, lo que mejora el rendimiento del registro de ruta al registrar cientos de rutas.
> > This command reduces all of your route registrations into a single method call within a cached file, improving the performance of route registration when registering hundreds of routes.

> {note} Como esta característica usa la serialización PHP, solo puede almacenar en caché las rutas para las aplicaciones que usan exclusivamente rutas basadas en controladores. PHP no puede serializar cierres.
> > > {note} Since this feature uses PHP serialization, you may only cache the routes for applications that exclusively use controller based routes. PHP is not able to serialize Closures.

<a name="deploying-with-forge"></a>
## Desplegando con Forge : Deploying With Forge

Si no está listo para administrar su propia configuración de servidor o no se siente cómodo configurando todos los diversos servicios necesarios para ejecutar una aplicación robusta de Laravel, [Laravel Forge](https://forge.laravel.com) es una maravillosa alternativa.
> > If you aren't quite ready to manage your own server configuration or aren't comfortable configuring all of the various services needed to run a robust Laravel application, [Laravel Forge](https://forge.laravel.com) is a wonderful alternative.

Laravel Forge puede crear servidores en varios proveedores de infraestructura como DigitalOcean, Linode, AWS y más. Además, Forge instala y administra todas las herramientas necesarias para construir aplicaciones Laravel robustas, como Nginx, MySQL, Redis, Memcached, Beanstalk y más.
> > Laravel Forge can create servers on various infrastructure providers such as DigitalOcean, Linode, AWS, and more. In addition, Forge installs and manages all of the tools needed to build robust Laravel applications, such as Nginx, MySQL, Redis, Memcached, Beanstalk, and more.
