# Laravel Homestead

- [Introduction](#introduction)
- [Installation & Setup](#installation-and-setup)
    - [First Steps](#first-steps)
    - [Configuring Homestead](#configuring-homestead)
    - [Launching The Vagrant Box](#launching-the-vagrant-box)
    - [Per Project Installation](#per-project-installation)
    - [Installing MariaDB](#installing-mariadb)
    - [Installing MongoDB](#installing-mongodb)
    - [Installing Elasticsearch](#installing-elasticsearch)
    - [Installing Neo4j](#installing-neo4j)
    - [Aliases](#aliases)
- [Daily Usage](#daily-usage)
    - [Accessing Homestead Globally](#accessing-homestead-globally)
    - [Connecting Via SSH](#connecting-via-ssh)
    - [Connecting To Databases](#connecting-to-databases)
    - [Database Backups](#database-backups)
    - [Adding Additional Sites](#adding-additional-sites)
    - [Environment Variables](#environment-variables)
    - [Configuring Cron Schedules](#configuring-cron-schedules)
    - [Configuring Mailhog](#configuring-mailhog)
    - [Configuring Minio](#configuring-minio)
    - [Ports](#ports)
    - [Sharing Your Environment](#sharing-your-environment)
    - [Multiple PHP Versions](#multiple-php-versions)
    - [Web Servers](#web-servers)
    - [Mail](#mail)
- [Network Interfaces](#network-interfaces)
- [Updating Homestead](#updating-homestead)
- [Provider Specific Settings](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## Introducción : Introduction

Laravel se esfuerza por hacer que toda la experiencia de desarrollo de PHP sea deliciosa, incluido el entorno de desarrollo local. [Vagrant](https://www.vagrantup.com) proporciona una forma simple y elegante de administrar y aprovisionar máquinas virtuales.
> > Laravel strives to make the entire PHP development experience delightful, including your local development environment. [Vagrant](https://www.vagrantup.com) provides a simple, elegant way to manage and provision Virtual Machines.

Laravel Homestead es una caja de Vagrant oficial, preempaquetada, que le proporciona un maravilloso entorno de desarrollo sin necesidad de instalar PHP, un servidor web y cualquier otro software de servidor en su máquina local. ¡No más preocuparse por estropear su sistema operativo! Las cajas de Vagrant son completamente desechables. Si algo sale mal, ¡puedes destruir y volver a crear el cuadro en minutos!
> > Laravel Homestead is an official, pre-packaged Vagrant box that provides you a wonderful development environment without requiring you to install PHP, a web server, and any other server software on your local machine. No more worrying about messing up your operating system! Vagrant boxes are completely disposable. If something goes wrong, you can destroy and re-create the box in minutes!

Homestead se ejecuta en cualquier sistema Windows, Mac o Linux, e incluye el servidor web Nginx, PHP 7.2, PHP 7.1, PHP 7.0, PHP 5.6, MySQL, PostgreSQL, Redis, Memcached, Node y todas las otras cosas que necesita para Desarrollar sorprendentes aplicaciones de Laravel.
> > Homestead runs on any Windows, Mac, or Linux system, and includes the Nginx web server, PHP 7.2, PHP 7.1, PHP 7.0, PHP 5.6, MySQL, PostgreSQL, Redis, Memcached, Node, and all of the other goodies you need to develop amazing Laravel applications.

> {note} Si está usando Windows, es posible que necesite habilitar la virtualización de hardware (VT-x). Por lo general, se puede habilitar a través de su BIOS. Si está utilizando Hyper-V en un sistema UEFI, también puede necesitar deshabilitar Hyper-V para acceder a VT-x.
> > > {note} If you are using Windows, you may need to enable hardware virtualization (VT-x). It can usually be enabled via your BIOS. If you are using Hyper-V on a UEFI system you may additionally need to disable Hyper-V in order to access VT-x.

<a name="included-software"></a>
### Software incluido : Included Software

<div class="content-list" markdown="1">
- Ubuntu 18.04
- Git
- PHP 7.2
- PHP 7.1
- PHP 7.0
- PHP 5.6
- Nginx
- Apache (Optional)
- MySQL
- MariaDB (Optional)
- Sqlite3
- PostgreSQL
- Composer
- Node (With Yarn, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd
- Mailhog
- Neo4j (Optional)
- MongoDB (Optional)
- Elasticsearch (Optional)
- ngrok
- wp-cli
- Zend Z-Ray
- Go
- Minio
</div>

<a name="installation-and-setup"></a>
## Instalación y configuración : Installation & Setup

<a name="first-steps"></a>
### Primeros pasos : First Steps

Antes de iniciar su entorno Homestead, debe instalar [VirtualBox 5.2](https://www.virtualbox.org/wiki/Downloads), [VMWare](https://www.vmware.com), [Parallels](https://www.parallels.com/products/desktop/) o [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) así como también [Vagrant](https://www.vagrantup.com/downloads.html). Todos estos paquetes de software proporcionan instaladores visuales fáciles de usar para todos los sistemas operativos populares.
> > Before launching your Homestead environment, you must install [VirtualBox 5.2](https://www.virtualbox.org/wiki/Downloads), [VMWare](https://www.vmware.com), [Parallels](https://www.parallels.com/products/desktop/) or [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) as well as [Vagrant](https://www.vagrantup.com/downloads.html). All of these software packages provide easy-to-use visual installers for all popular operating systems.

Para usar el proveedor de VMware, deberá comprar VMware Fusion / Workstation y el [VMware Vagrant plug-in](https://www.vagrantup.com/vmware). Si bien no es gratuito, VMware puede proporcionar un rendimiento de carpetas compartidas más rápido de manera inmediata.
> > To use the VMware provider, you will need to purchase both VMware Fusion / Workstation and the [VMware Vagrant plug-in](https://www.vagrantup.com/vmware). Though it is not free, VMware can provide faster shared folder performance out of the box.

Para usar el proveedor de Parallels, deberá instalar [el complemento Parallels Vagrant](https://github.com/Parallels/vagrant-parallels). Es gratis.
> > To use the Parallels provider, you will need to install [Parallels Vagrant plug-in](https://github.com/Parallels/vagrant-parallels). It is free of charge.

Debido a [limitaciones de Vagrant](https://www.vagrantup.com/docs/hyperv/limitations.html), el proveedor de Hyper-V ignora todas las configuraciones de red.
> > Because of [Vagrant limitations](https://www.vagrantup.com/docs/hyperv/limitations.html), The Hyper-V provider ignores all networking settings.

#### Instalación de Homestead Vagrant Box : Installing The Homestead Vagrant Box

Una vez que se hayan instalado VirtualBox / VMware y Vagrant, debe agregar el cuadro `laravel/homestead` a su instalación de Vagrant usando el siguiente comando en su terminal. La descarga de la caja demorará unos minutos, dependiendo de la velocidad de su conexión a Internet:
> > Once VirtualBox / VMware and Vagrant have been installed, you should add the `laravel/homestead` box to your Vagrant installation using the following command in your terminal. It will take a few minutes to download the box, depending on your Internet connection speed:

    vagrant box add laravel/homestead

Si este comando falla, asegúrese de que su instalación de Vagrant esté actualizada.
> > If this command fails, make sure your Vagrant installation is up to date.

#### Instalación de Homestead : Installing Homestead

Puede instalar Homestead clonando el repositorio. Considere clonar el repositorio en una carpeta `Homestead` dentro de su directorio "home", ya que el cuadro de Homestead servirá como anfitrión para todos sus proyectos de Laravel:
> > You may install Homestead by cloning the repository. Consider cloning the repository into a `Homestead` folder within your "home" directory, as the Homestead box will serve as the host to all of your Laravel projects:

    git clone https://github.com/laravel/homestead.git ~/Homestead

Debería verificar una versión etiquetada de Homestead ya que la rama `master` no siempre es estable. Puede encontrar la última versión estable en la [Página de publicación de GitHub](https://github.com/laravel/homestead/releases):
> > You should check out a tagged version of Homestead since the `master` branch may not always be stable. You can find the latest stable version on the [GitHub Release Page](https://github.com/laravel/homestead/releases):

    cd ~/Homestead

    // Clone the desired release...
    git checkout v7.14.2

Una vez que haya clonado el repositorio de Homestead, ejecute el comando `bash init.sh` desde el directorio de Homestead para crear el archivo de configuración` Homestead.yaml`. El archivo `Homestead.yaml` se colocará en el directorio de Homestead:
> > Once you have cloned the Homestead repository, run the `bash init.sh` command from the Homestead directory to create the `Homestead.yaml` configuration file. The `Homestead.yaml` file will be placed in the Homestead directory:

    // Mac / Linux...
    bash init.sh

    // Windows...
    init.bat

<a name="configuring-homestead"></a>
### Configuración de Homestead : Configuring Homestead

#### Configuración de su proveedor : Setting Your Provider

La clave `provider` en su archivo `Homestead.yaml` indica qué proveedor Vagrant debe usarse: `virtualbox`,` vmware_fusion`, `vmware_workstation`,` parallels` o `hyperv`. Puede configurar esto para el proveedor que prefiera:
> > The `provider` key in your `Homestead.yaml` file indicates which Vagrant provider should be used: `virtualbox`, `vmware_fusion`, `vmware_workstation`, `parallels` or `hyperv`. You may set this to the provider you prefer:

    provider: virtualbox

#### Configuración de carpetas compartidas : Configuring Shared Folders

La propiedad `folders` del archivo `Homestead.yaml` lista todas las carpetas que desea compartir con su entorno de Homestead. A medida que cambian los archivos dentro de estas carpetas, se mantendrán sincronizados entre su máquina local y el entorno de Homestead. Puede configurar tantas carpetas compartidas como sea necesario:
> > The `folders` property of the `Homestead.yaml` file lists all of the folders you wish to share with your Homestead environment. As files within these folders are changed, they will be kept in sync between your local machine and the Homestead environment. You may configure as many shared folders as necessary:

    folders:
        - map: ~/code
          to: /home/vagrant/code

Si solo está creando algunos sitios, este mapeo genérico funcionará perfectamente. Sin embargo, a medida que la cantidad de sitios continúa creciendo, puede comenzar a experimentar problemas de rendimiento. Este problema puede ser dolorosamente evidente en máquinas de bajo costo o proyectos que contienen una gran cantidad de archivos. Si tiene este problema, intente asignar cada proyecto a su propia carpeta Vagrant:
> > If you are only creating a few sites, this generic mapping will work just fine. However, as the number of sites continue to grow, you may begin to experience performance problems. This problem can be painfully apparent on low-end machines or projects that contain a very large number of files. If you are experiencing this issue, try mapping every project to its own Vagrant folder:

    folders:
        - map: ~/code/project1
          to: /home/vagrant/code/project1

        - map: ~/code/project2
          to: /home/vagrant/code/project2

Para habilitar [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html), solo necesita agregar un indicador simple a la configuración de su carpeta sincronizada:
> > To enable [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html), you only need to add a simple flag to your synced folder configuration:

    folders:
        - map: ~/code
          to: /home/vagrant/code
          type: "nfs"

> {note} Cuando use NFS, debería considerar instalar el complemento [vagrant-bindfs](https://github.com/gael-ian/vagrant-bindfs). Este complemento mantendrá los permisos correctos de usuario / grupo para los archivos y directorios dentro del cuadro de Homestead.
> > > {note} When using NFS, you should consider installing the [vagrant-bindfs](https://github.com/gael-ian/vagrant-bindfs) plug-in. This plug-in will maintain the correct user / group permissions for files and directories within the Homestead box.

También puede pasar cualquier opción compatible con las [Carpetas sincronizadas](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) de Vagrant listándolas con la tecla `options`:
> > You may also pass any options supported by Vagrant's [Synced Folders](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) by listing them under the `options` key:

    folders:
        - map: ~/code
          to: /home/vagrant/code
          type: "rsync"
          options:
              rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
              rsync__exclude: ["node_modules"]

#### Configuración de sitios Nginx : Configuring Nginx Sites

¿No estás familiarizado con Nginx? No hay problema. La propiedad `sites` le permite mapear fácilmente un "dominio" a una carpeta en su entorno Homestead. Se incluye una configuración de muestra del sitio en el archivo `Homestead.yaml`. De nuevo, puede agregar tantos sitios a su entorno de Homestead como sea necesario. Homestead puede servir como un entorno conveniente y virtualizado para cada proyecto de Laravel en el que esté trabajando:
> > Not familiar with Nginx? No problem. The `sites` property allows you to easily map a "domain" to a folder on your Homestead environment. A sample site configuration is included in the `Homestead.yaml` file. Again, you may add as many sites to your Homestead environment as necessary. Homestead can serve as a convenient, virtualized environment for every Laravel project you are working on:

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public

Si cambia la propiedad `sites` después de aprovisionar el cuadro de Homestead, debe volver a ejecutar` vagrant reload --provision `para actualizar la configuración de Nginx en la máquina virtual.
> > If you change the `sites` property after provisioning the Homestead box, you should re-run `vagrant reload --provision` to update the Nginx configuration on the virtual machine.

#### El archivo de hosts : The Hosts File

Debe agregar los "dominios" para sus sitios Nginx al archivo `hosts` en su máquina. El archivo `hosts` redireccionará las solicitudes de sus sitios Homestead a su máquina Homestead. En Mac y Linux, este archivo se encuentra en `/etc/hosts`. En Windows, se encuentra en `C:\Windows\System32\drivers\etc\hosts`. Las líneas que agregue a este archivo se verán así:
> > You must add the "domains" for your Nginx sites to the `hosts` file on your machine. The `hosts` file will redirect requests for your Homestead sites into your Homestead machine. On Mac and Linux, this file is located at `/etc/hosts`. On Windows, it is located at `C:\Windows\System32\drivers\etc\hosts`. The lines you add to this file will look like the following:

    192.168.10.10  homestead.test

Asegúrese de que la dirección IP indicada sea la que se encuentra en su archivo `Homestead.yaml`. Una vez que haya agregado el dominio a su archivo `hosts` y haya lanzado el cuadro Vagrant, podrá acceder al sitio a través de su navegador web:
> > Make sure the IP address listed is the one set in your `Homestead.yaml` file. Once you have added the domain to your `hosts` file and launched the Vagrant box you will be able to access the site via your web browser:

    http://homestead.test

<a name="launching-the-vagrant-box"></a>
### Lanzando The Vagrant Box : Launching The Vagrant Box

Una vez que hayas editado `Homestead.yaml` a tu gusto, ejecuta el comando `vagrant up` desde tu directorio de Homestead. Vagrant arrancará la máquina virtual y configurará automáticamente sus carpetas compartidas y sitios Nginx.
> > Once you have edited the `Homestead.yaml` to your liking, run the `vagrant up` command from your Homestead directory. Vagrant will boot the virtual machine and automatically configure your shared folders and Nginx sites.

Para destruir la máquina, puedes usar el comando `vagrant destroy --force`.
> > To destroy the machine, you may use the `vagrant destroy --force` command.

<a name="per-project-installation"></a>
### Por instalación del proyecto : Per Project Installation

En lugar de instalar Homestead globalmente y compartir el mismo cuadro de Homestead en todos sus proyectos, en su lugar puede configurar una instancia de Homestead para cada proyecto que administre. Instalar Homestead por proyecto puede ser beneficioso si desea enviar un `Vagrantfile` con su proyecto, permitiendo que otros que trabajan en el proyecto `vagrant up`.
> > Instead of installing Homestead globally and sharing the same Homestead box across all of your projects, you may instead configure a Homestead instance for each project you manage. Installing Homestead per project may be beneficial if you wish to ship a `Vagrantfile` with your project, allowing others working on the project to `vagrant up`.

Para instalar Homestead directamente en su proyecto, solicítelo usando Composer:
> > To install Homestead directly into your project, require it using Composer:

    composer require laravel/homestead --dev

Una vez que se haya instalado Homestead, use el comando `make` para generar el archivo `Vagrantfile` y `Homestead.yaml` en la raíz del proyecto. El comando `make` configurará automáticamente las directivas `sites` y `folders` en el archivo `Homestead.yaml`.
> > Once Homestead has been installed, use the `make` command to generate the `Vagrantfile` and `Homestead.yaml` file in your project root. The `make` command will automatically configure the `sites` and `folders` directives in the `Homestead.yaml` file.

Mac / Linux:

    php vendor/bin/homestead make

Windows:

    vendor\\bin\\homestead make

A continuación, ejecute el comando `vagrant up` en su terminal y acceda a su proyecto en `http://homestead.test` en su navegador. Recuerde, aún necesitará agregar una entrada de archivo `/etc/hosts` para `homestead.test` o el dominio de su elección.
> > Next, run the `vagrant up` command in your terminal and access your project at `http://homestead.test` in your browser. Remember, you will still need to add an `/etc/hosts` file entry for `homestead.test` or the domain of your choice.

<a name="installing-mariadb"></a>
### Instalación de MariaDB : Installing MariaDB

Si prefiere usar MariaDB en lugar de MySQL, puede agregar la opción `mariadb` a su archivo `Homestead.yaml`. Esta opción eliminará MySQL e instalará MariaDB. MariaDB sirve como un reemplazo directo para MySQL, por lo que aún debe usar el controlador de base de datos `mysql` en la configuración de la base de datos de su aplicación:
> > If you prefer to use MariaDB instead of MySQL, you may add the `mariadb` option to your `Homestead.yaml` file. This option will remove MySQL and install MariaDB. MariaDB serves as a drop-in replacement for MySQL so you should still use the `mysql` database driver in your application's database configuration:

    box: laravel/homestead
    ip: "192.168.10.10"
    memory: 2048
    cpus: 4
    provider: virtualbox
    mariadb: true

<a name="installing-mongodb"></a>
### Instalación de MongoDB : Installing MongoDB

Para instalar MongoDB Community Edition, actualice su archivo `Homestead.yaml` con la siguiente opción de configuración:
> > To install MongoDB Community Edition, update your `Homestead.yaml` file with the following configuration option:

    mongodb: true

La instalación predeterminada de MongoDB establecerá el nombre de usuario de la base de datos en `homestead` y la contraseña correspondiente en `secret`.
> > The default MongoDB installation will set the database username to `homestead` and the corresponding password to `secret`.

<a name="installing-elasticsearch"></a>
### Instalación de Elasticsearch : Installing Elasticsearch

Para instalar Elasticsearch, agregue la opción `elasticsearch` al archivo `Homestead.yaml` y especifique una versión compatible, que puede ser una versión principal o un número de versión exacto (major.minor.patch). La instalación predeterminada creará un clúster llamado 'homestead'. Nunca le de a Elasticsearch más de la mitad de la memoria del sistema operativo, así que asegúrese de que su máquina Homestead tenga al menos el doble de la asignación de Elasticsearch:
> > To install Elasticsearch, add the `elasticsearch` option to your `Homestead.yaml` file and specify a supported version, which may be a major version or an exact version number (major.minor.patch). The default installation will create a cluster named 'homestead'. You should never give Elasticsearch more than half of the operating system's memory, so make sure your Homestead machine has at least twice the Elasticsearch allocation:

    box: laravel/homestead
    ip: "192.168.10.10"
    memory: 4096
    cpus: 4
    provider: virtualbox
    elasticsearch: 6

> {tip} Consulte la [documentación de Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current) para aprender a personalizar su configuración.
> > > {tip} Check out the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current) to learn how to customize your configuration.

<a name="installing-neo4j"></a>
### Instalación de Neo4j : Installing Neo4j

[Neo4j](https://neo4j.com/) es un sistema de administración de base de datos de gráficos. Para instalar Neo4j Community Edition, actualice su archivo `Homestead.yaml` con la siguiente opción de configuración:
> > [Neo4j](https://neo4j.com/) is a graph database management system. To install Neo4j Community Edition, update your `Homestead.yaml` file with the following configuration option:

    neo4j: true

La instalación predeterminada de Neo4j establecerá el nombre de usuario de la base de datos en `homestead` y la contraseña correspondiente en `secret`. Para acceder al navegador Neo4j, visite `http://homestead.test:7474` a través de su navegador web. Los puertos `7687` (Bolt), `7474` (HTTP) y `7473` (HTTPS) están listos para atender las solicitudes del cliente Neo4j.
> > The default Neo4j installation will set the database username to `homestead` and corresponding password to `secret`. To access the Neo4j browser, visit `http://homestead.test:7474` via your web browser. The ports `7687` (Bolt), `7474` (HTTP), and `7473` (HTTPS) are ready to serve requests from the Neo4j client.

<a name="aliases"></a>
### Aliases

Puede agregar alias Bash a su máquina Homestead modificando el archivo `aliases` dentro de su directorio Homestead:
> > You may add Bash aliases to your Homestead machine by modifying the `aliases` file within your Homestead directory:

    alias c='clear'
    alias ..='cd ..'

Después de haber actualizado el archivo `aliases`, debe volver a aprovisionar la máquina Homestead utilizando el comando `vagrant reload --provision`. Esto asegurará que sus nuevos alias estén disponibles en la máquina.
> >  After you have updated the `aliases` file, you should re-provision the Homestead machine using the `vagrant reload --provision` command. This will ensure that your new aliases are available on the machine.

<a name="daily-usage"></a>
## Uso diario : Daily Usage

<a name="accessing-homestead-globally"></a>
### Acceso global a Homestead : Accessing Homestead Globally

A veces es posible que desee `vagrant up` su máquina Homestead desde cualquier lugar de su sistema de archivos. Puede hacer esto en sistemas Mac / Linux agregando una función Bash a su perfil Bash. En Windows, puede lograr esto agregando un archivo "por lotes" a su `PATH`. Estas secuencias de comandos le permitirán ejecutar cualquier comando de Vagrant desde cualquier lugar de su sistema y automáticamente apuntarán ese comando a su instalación de Homestead:
> > Sometimes you may want to `vagrant up` your Homestead machine from anywhere on your filesystem. You can do this on Mac / Linux systems by adding a Bash function to your Bash profile. On Windows, you may accomplish this by adding a "batch" file to your `PATH`. These scripts will allow you to run any Vagrant command from anywhere on your system and will automatically point that command to your Homestead installation:

#### Mac / Linux

    function homestead() {
        ( cd ~/Homestead && vagrant $* )
    }

Asegúrese de ajustar la ruta `~/Homestead` en la función a la ubicación de su instalación real de Homestead. Una vez que la función está instalada, puede ejecutar comandos como `homestead up` o `homestead ssh` desde cualquier lugar de su sistema.
> > Make sure to tweak the `~/Homestead` path in the function to the location of your actual Homestead installation. Once the function is installed, you may run commands like `homestead up` or `homestead ssh` from anywhere on your system.

#### Windows

Cree un archivo por lotes `homestead.bat` en cualquier lugar de su máquina con los siguientes contenidos:
> > Create a `homestead.bat` batch file anywhere on your machine with the following contents:

    @echo off

    set cwd=%cd%
    set homesteadVagrant=C:\Homestead

    cd /d %homesteadVagrant% && vagrant %*
    cd /d %cwd%

    set cwd=
    set homesteadVagrant=

Asegúrese de ajustar el ejemplo de la ruta `C:\Homestead` en el script a la ubicación real de su instalación de Homestead. Después de crear el archivo, agregue la ubicación del archivo a su `PATH`. A continuación, puede ejecutar comandos como `homestead up` o `homestead ssh` desde cualquier lugar de su sistema.
> > Make sure to tweak the example `C:\Homestead` path in the script to the actual location of your Homestead installation. After creating the file, add the file location to your `PATH`. You may then run commands like `homestead up` or `homestead ssh` from anywhere on your system.

<a name="connecting-via-ssh"></a>
### Conectando a través de SSH : Connecting Via SSH

Puede SSH en su máquina virtual emitiendo el comando de la terminal `vagrant ssh` desde su directorio de Homestead.
> > You can SSH into your virtual machine by issuing the `vagrant ssh` terminal command from your Homestead directory.

Pero, dado que es probable que necesite SSH en su máquina de Homestead con frecuencia, considere agregar la "función" descrita anteriormente a su equipo host para rápidamente SSH en la casilla de Homestead.
> > But, since you will probably need to SSH into your Homestead machine frequently, consider adding the "function" described above to your host machine to quickly SSH into the Homestead box.

<a name="connecting-to-databases"></a>
### Conectando a Bases de Datos : Connecting To Databases

Una base de datos `homestead` está configurada para MySQL y PostgreSQL fuera de la caja. Para una mayor comodidad, el archivo `.env` de Laravel configura el marco para utilizar esta base de datos de manera inmediata.
> > A `homestead` database is configured for both MySQL and PostgreSQL out of the box. For even more convenience, Laravel's `.env` file configures the framework to use this database out of the box.

Para conectarse a su base de datos MySQL o PostgreSQL desde el cliente de la base de datos de su máquina host, debe conectarse a `127.0.0.1` y al puerto` 33060` (MySQL) o `54320` (PostgreSQL). El nombre de usuario y la contraseña para ambas bases de datos son `homestead` / `secret`.
> > To connect to your MySQL or PostgreSQL database from your host machine's database client, you should connect to `127.0.0.1` and port `33060` (MySQL) or `54320` (PostgreSQL). The username and password for both databases is `homestead` / `secret`.

> {note} Solo debe usar estos puertos no estándar cuando se conecte a las bases de datos desde su equipo host. Utilizará los puertos predeterminados 3306 y 5432 en su archivo de configuración de base de datos de Laravel, ya que Laravel se está ejecutando dentro de la máquina virtual.
> > > {note} You should only use these non-standard ports when connecting to the databases from your host machine. You will use the default 3306 and 5432 ports in your Laravel database configuration file since Laravel is running _within_ the virtual machine.

<a name="database-backups"></a>
### Copias de seguridad de la base de datos : Database Backups

Homestead puede respaldar automáticamente su base de datos cuando se destruye su caja Vagrant. Para utilizar esta característica, debe usar Vagrant 2.1.0 o superior. O bien, si está utilizando una versión anterior de Vagrant, debe instalar el complemento `vagrant-triggers`. Para habilitar las copias de seguridad automáticas de la base de datos, agregue la siguiente línea a su archivo `Homestead.yaml`:
> > Homestead can automatically backup your database when your Vagrant box is destroyed. To utilize this feature, you must be using Vagrant 2.1.0 or greater. Or, if you are using an older version of Vagrant, you must install the `vagrant-triggers` plug-in. To enable automatic database backups, add the following line to your `Homestead.yaml` file:

    backup: true

Una vez configurado, Homestead exportará sus bases de datos a los directorios `mysql_backup` y `postgres_backup` cuando se ejecute el comando `vagrant destroy`. Estos directorios se pueden encontrar en la carpeta donde clonó Homestead o en la raíz de su proyecto si está utilizando el método [por instalación de proyecto](#per-project-installation).
> > Once configured, Homestead will export your databases to `mysql_backup` and `postgres_backup` directories when the `vagrant destroy` command is executed. These directories can be found in the folder where you cloned Homestead or in the root of your project if you are using the [per project installation](#per-project-installation) method.

<a name="adding-additional-sites"></a>
### Agregar sitios adicionales : Adding Additional Sites

Una vez que se haya aprovisionado y ejecutado su entorno Homestead, es posible que desee agregar sitios Nginx adicionales para sus aplicaciones Laravel. Puede ejecutar tantas instalaciones de Laravel como desee en un solo entorno de Homestead. Para agregar un sitio adicional, agregue el sitio a su archivo `Homestead.yaml`:
> > Once your Homestead environment is provisioned and running, you may want to add additional Nginx sites for your Laravel applications. You can run as many Laravel installations as you wish on a single Homestead environment. To add an additional site, add the site to your `Homestead.yaml` file:

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
        - map: another.test
          to: /home/vagrant/code/another/public

Si Vagrant no está administrando automáticamente su archivo de "hosts", es posible que deba agregar el nuevo sitio a ese archivo también:
> > If Vagrant is not automatically managing your "hosts" file, you may need to add the new site to that file as well:

    192.168.10.10  homestead.test
    192.168.10.10  another.test

Una vez que se haya agregado el sitio, ejecute el comando `vagrant reload --provision` desde su directorio de Homestead.
> > Once the site has been added, run the `vagrant reload --provision` command from your Homestead directory.

<a name="site-types"></a>
#### Tipos de sitios : Site Types

Homestead admite varios tipos de sitios que le permiten ejecutar fácilmente proyectos que no están basados ​​en Laravel. Por ejemplo, podemos agregar fácilmente una aplicación Symfony a Homestead usando el tipo de sitio `symfony2`:
> > Homestead supports several types of sites which allow you to easily run projects that are not based on Laravel. For example, we may easily add a Symfony application to Homestead using the `symfony2` site type:

    sites:
        - map: symfony2.test
          to: /home/vagrant/code/Symfony/web
          type: "symfony2"

Los tipos de sitios disponibles son: `apache`, `apigility`, `expressive`, `laravel` (predeterminado), `proxy`, `silverstripe`, `statamic`,` symfony2`, `symfony4`, y `zf`.
> > The available site types are: `apache`, `apigility`, `expressive`, `laravel` (the default), `proxy`, `silverstripe`, `statamic`, `symfony2`, `symfony4`, and `zf`.

<a name="site-parameters"></a>
#### Parámetros del sitio : Site Parameters

Puede agregar valores Nginx `fastcgi_param` adicionales a su sitio a través de la directiva del sitio `params`. Por ejemplo, agregaremos un parámetro `FOO` con un valor `BAR`:
> > You may add additional Nginx `fastcgi_param` values to your site via the `params` site directive. For example, we'll add a `FOO` parameter with a value of `BAR`:

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          params:
              - key: FOO
                value: BAR

<a name="environment-variables"></a>
### Variables de entorno : Environment Variables

Puede establecer variables de entorno globales agregándolas a su archivo `Homestead.yaml`:
> > You can set global environment variables by adding them to your `Homestead.yaml` file:

    variables:
        - key: APP_ENV
          value: local
        - key: FOO
          value: bar

Después de actualizar `Homestead.yaml`, asegúrese de volver a aprovisionar la máquina ejecutando` vagrant reload --provision`. Esto actualizará la configuración de PHP-FPM para todas las versiones de PHP instaladas y también actualizará el entorno para el usuario `vagrant`.
> > After updating the `Homestead.yaml`, be sure to re-provision the machine by running `vagrant reload --provision`. This will update the PHP-FPM configuration for all of the installed PHP versions and also update the environment for the `vagrant` user.

<a name="configuring-cron-schedules"></a>
### Configuración de cronogramas : Configuring Cron Schedules

Laravel proporciona una forma conveniente de [programar trabajos Cron](/docs/{{version}}/scheduling) programando un único comando Artisan `schedule:run` para ejecutarse cada minuto. El comando `schedule:run` examinará la programación de trabajos definida en la clase `App\Console\Kernel` para determinar qué trabajos deben ejecutarse.
> > Laravel provides a convenient way to [schedule Cron jobs](/docs/{{version}}/scheduling) by scheduling a single `schedule:run` Artisan command to be run every minute. The `schedule:run` command will examine the job schedule defined in your `App\Console\Kernel` class to determine which jobs should be run.

Si desea que el comando `schedule:run` se ejecute para un sitio de Homestead, puede establecer la opción `schedule` en `true` al definir el sitio:
> > If you would like the `schedule:run` command to be run for a Homestead site, you may set the `schedule` option to `true` when defining the site:

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          schedule: true

El trabajo de Cron para el sitio se definirá en la carpeta `/etc/cron.d` de la máquina virtual.
> > The Cron job for the site will be defined in the `/etc/cron.d` folder of the virtual machine.

<a name="configuring-mailhog"></a>
### Configuración de Mailhog : Configuring Mailhog

Mailhog le permite capturar fácilmente su correo electrónico saliente y examinarlo sin enviar el correo a sus destinatarios. Para comenzar, actualice su archivo `.env` para usar la siguiente configuración de correo:
> > Mailhog allows you to easily catch your outgoing email and examine it without actually sending the mail to its recipients. To get started, update your `.env` file to use the following mail settings:

    MAIL_DRIVER=smtp
    MAIL_HOST=localhost
    MAIL_PORT=1025
    MAIL_USERNAME=null
    MAIL_PASSWORD=null
    MAIL_ENCRYPTION=null

<a name="configuring-minio"></a>
### Configurando Minio : Configuring Minio

Minio es un servidor de almacenamiento de objetos de código abierto con una API compatible con Amazon S3. Para instalar Minio, actualice su archivo `Homestead.yaml` con la siguiente opción de configuración:
> > Minio is an open source object storage server with an Amazon S3 compatible API. To install Minio, update your `Homestead.yaml` file with the following configuration option:

    minio: true

Por defecto, Minio está disponible en el puerto 9600. Puede acceder al panel de control de Minio visitando `http: // homestead: 9600 /`. La clave de acceso predeterminada es `homestead`, mientras que la clave secreta predeterminada es` secretkey`. Al acceder a Minio, siempre debe usar la región `us-east-1`.
> > By default, Minio is available on port 9600. You may access the Minio control panel by visiting `http://homestead:9600/`. The default access key is `homestead`, while the default secret key is `secretkey`. When accessing Minio, you should always use region `us-east-1`.

Para usar Minio, necesitará ajustar la configuración del disco S3 en su archivo de configuración `config/filesystems.php`. Necesitará agregar la opción `use_path_style_endpoint` a la configuración del disco, así como también cambiar la clave `url` a `endpoint`:
> > In order to use Minio you will need to adjust the S3 disk configuration in your `config/filesystems.php` configuration file. You will need to add the `use_path_style_endpoint` option to the disk configuration, as well as change the `url` key to `endpoint`:

    's3' => [
        'driver' => 's3',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION'),
        'bucket' => env('AWS_BUCKET'),
        'endpoint' => env('AWS_URL'),
        'use_path_style_endpoint' => true
    ]

Finalmente, asegúrese de que su archivo `.env` tenga las siguientes opciones:
> > Finally, ensure your `.env` file has the following options:

    AWS_ACCESS_KEY_ID=homestead
    AWS_SECRET_ACCESS_KEY=secretkey
    AWS_DEFAULT_REGION=us-east-1
    AWS_URL=http://homestead:9600

<a name="ports"></a>
### Puertos : Ports

Por defecto, los siguientes puertos se envían a su entorno de Homestead:
> > By default, the following ports are forwarded to your Homestead environment:

<div class="content-list" markdown="1">
- **SSH:** 2222 &rarr; Forwards To 22
- **ngrok UI:** 4040 &rarr; Forwards To 4040
- **HTTP:** 8000 &rarr; Forwards To 80
- **HTTPS:** 44300 &rarr; Forwards To 443
- **MySQL:** 33060 &rarr; Forwards To 3306
- **PostgreSQL:** 54320 &rarr; Forwards To 5432
- **MongoDB:** 27017 &rarr; Forwards To 27017
- **Mailhog:** 8025 &rarr; Forwards To 8025
- **Minio:** 9600 &rarr; Forwards To 9600
</div>

#### Reenvío de puertos adicionales : Forwarding Additional Ports

Si lo desea, puede reenviar puertos adicionales al cuadro Vagrant, así como especificar su protocolo:
> > If you wish, you may forward additional ports to the Vagrant box, as well as specify their protocol:

    ports:
        - send: 50000
          to: 5000
        - send: 7777
          to: 777
          protocol: udp

<a name="sharing-your-environment"></a>
### Compartiendo su entorno : Sharing Your Environment

A veces puede desear compartir en qué está trabajando actualmente con sus compañeros de trabajo o un cliente. Vagrant tiene una forma incorporada de soportar esto a través de `vagrant share`; sin embargo, esto no funcionará si tiene varios sitios configurados en su archivo `Homestead.yaml`.
> > Sometimes you may wish to share what you're currently working on with coworkers or a  client. Vagrant has a built-in way to support this via `vagrant share`; however, this will not work if you have multiple sites configured in your `Homestead.yaml` file.

Para resolver este problema, Homestead incluye su propio comando `share`. Para comenzar, SSH en su máquina de Homestead a través de `vagrant ssh` y ejecute `share homestead.test`. Esto compartirá el sitio `homestead.test` de su archivo de configuración `Homestead.yaml`. Por supuesto, puede sustituir cualquiera de sus otros sitios configurados por `homestead.test`:
> > To solve this problem, Homestead includes its own `share` command. To get started, SSH into your Homestead machine via `vagrant ssh` and run `share homestead.test`. This will share the `homestead.test` site from your `Homestead.yaml` configuration file. Of course, you may substitute any of your other configured sites for `homestead.test`:

    share homestead.test

Después de ejecutar el comando, verá aparecer una pantalla Ngrok que contiene el registro de actividad y las URL de acceso público para el sitio compartido. Si desea especificar una región personalizada, un subdominio u otra opción de tiempo de ejecución de Ngrok, puede agregarlos a su comando `share`:
> > After running the command, you will see an Ngrok screen appear which contains the activity log and the publicly accessible URLs for the shared site. If you would like to specify a custom region, subdomain, or other Ngrok runtime option, you may add them to your `share` command:

    share homestead.test -region=eu -subdomain=laravel

> {note} Recuerde, Vagrant es intrínsecamente inseguro y está exponiendo su máquina virtual a Internet cuando ejecuta el comando `share`.
> > > {note} Remember, Vagrant is inherently insecure and you are exposing your virtual machine to the Internet when running the `share` command.

<a name="multiple-php-versions"></a>
### Varias versiones de PHP : Multiple PHP Versions

> {note} Esta característica solo es compatible con Nginx.
> > > {note} This feature is only compatible with Nginx.

Homestead 6 presentó soporte para múltiples versiones de PHP en la misma máquina virtual. Puede especificar qué versión de PHP usar para un sitio determinado dentro de su archivo `Homestead.yaml`. Las versiones de PHP disponibles son: "5.6", "7.0", "7.1" y "7.2" (valor predeterminado):
> > Homestead 6 introduced support for multiple versions of PHP on the same virtual machine. You may specify which version of PHP to use for a given site within your `Homestead.yaml` file. The available PHP versions are: "5.6", "7.0", "7.1" and "7.2" (the default):

    sites:
        - map: homestead.test
          to: /home/vagrant/code/Laravel/public
          php: "5.6"

Además, puede usar cualquiera de las versiones de PHP compatibles a través de la CLI:
> > In addition, you may use any of the supported PHP versions via the CLI:

    php5.6 artisan list
    php7.0 artisan list
    php7.1 artisan list
    php7.2 artisan list

<a name="web-servers"></a>
### Servidores web : Web Servers

Homestead utiliza el servidor web Nginx de forma predeterminada. Sin embargo, puede instalar Apache si se especifica `apache` como un tipo de sitio. Si bien ambos servidores web se pueden instalar al mismo tiempo, no se pueden *ejecutar* al mismo tiempo. El comando `flip` shell está disponible para facilitar el proceso de cambio entre servidores web. El comando `flip` determina automáticamente qué servidor web se está ejecutando, lo apaga y luego inicia el otro servidor. Para usar este comando, SSH en su máquina Homestead y ejecute el comando en su terminal:
> > Homestead uses the Nginx web server by default. However, it can install Apache if `apache` is specified as a site type. While both web servers can be installed at the same time, they cannot both be *running* at the same time. The `flip` shell command is available to ease the process of switching between web servers. The `flip` command automatically determines which web server is running, shuts it off, and then starts the other server. To use this command, SSH into your Homestead machine and run the command in your terminal:

    flip

<a name="mail"></a>
### Mail

Homestead incluye el agente de transferencia de correo Postfix, que está escuchando en el puerto `1025` por defecto. Por lo tanto, puede indicar a su aplicación que use el controlador de correo `smtp` en `localhost` puerto `1025`. Entonces, todo el correo enviado será manejado por Postfix y atrapado por Mailhog. Para ver sus correos electrónicos enviados, abra [http://localhost:8025](http://localhost:8025) en su navegador web.
> > Homestead includes the Postfix mail transfer agent, which is listening on port `1025` by default. So, you may instruct your application to use the `smtp` mail driver on `localhost` port `1025`. Then, all sent mail will be handled by Postfix and caught by Mailhog. To view your sent emails, open [http://localhost:8025](http://localhost:8025) in your web browser.

<a name="network-interfaces"></a>
## Interfaces de red : Network Interfaces

La propiedad `networks` de `Homestead.yaml` configura las interfaces de red para su entorno Homestead. Puede configurar tantas interfaces como sea necesario:
> > The `networks` property of the `Homestead.yaml` configures network interfaces for your Homestead environment. You may configure as many interfaces as necessary:

    networks:
        - type: "private_network"
          ip: "192.168.10.20"

Para habilitar una interfaz [puenteada](https://www.vagrantup.com/docs/networking/public_network.html), configure una configuración `bridge` y cambie el tipo de red a `public_network`:
> > To enable a [bridged](https://www.vagrantup.com/docs/networking/public_network.html) interface, configure a `bridge` setting and change the network type to `public_network`:

    networks:
        - type: "public_network"
          ip: "192.168.10.20"
          bridge: "en1: Wi-Fi (AirPort)"

Para habilitar [DHCP](https://www.vagrantup.com/docs/networking/public_network.html), simplemente elimine la opción `ip` de su configuración:
> > To enable [DHCP](https://www.vagrantup.com/docs/networking/public_network.html), just remove the `ip` option from your configuration:

    networks:
        - type: "public_network"
          bridge: "en1: Wi-Fi (AirPort)"

<a name="updating-homestead"></a>
## Actualización de Homestead : Updating Homestead

Puede actualizar Homestead en dos simples pasos. Primero, debe actualizar el cuadro Vagrant usando el comando `vagrant box update`:
> > You can update Homestead in two simple steps. First, you should update the Vagrant box using the `vagrant box update` command:

    vagrant box update

A continuación, debe actualizar el código fuente de Homestead. Si clonó el repositorio, puede `git pull origin master` en la ubicación donde originalmente clonó el repositorio.
> > Next, you need to update the Homestead source code. If you cloned the repository you can `git pull origin master` at the location you originally cloned the repository.

Si ha instalado Homestead a través del archivo `composer.json` de su proyecto, debe asegurarse de que su archivo `composer.json` contiene `"laravel/homestead": "^7"` y actualice sus dependencias:
> > If you have installed Homestead via your project's `composer.json` file, you should ensure your `composer.json` file contains `"laravel/homestead": "^7"` and update your dependencies:

    composer update

<a name="provider-specific-settings"></a>
## Configuración específica del proveedor : Provider Specific Settings

<a name="provider-specific-virtualbox"></a>
### VirtualBox

#### `natdnshostresolver`

Por defecto, Homestead configura la configuración `natdnshostresolver` en `on`. Esto le permite a Homestead usar la configuración DNS de su sistema operativo host. Si desea anular este comportamiento, agregue las siguientes líneas a su archivo `Homestead.yaml`:
> > By default, Homestead configures the `natdnshostresolver` setting to `on`. This allows Homestead to use your host operating system's DNS settings. If you would like to override this behavior, add the following lines to your `Homestead.yaml` file:

    provider: virtualbox
    natdnshostresolver: off

#### Enlaces simbólicos en Windows : Symbolic Links On Windows

Si los enlaces simbólicos no funcionan correctamente en su máquina con Windows, es posible que deba agregar el siguiente bloque a su `Vagrantfile`:
> > If symbolic links are not working properly on your Windows machine, you may need to add the following block to your `Vagrantfile`:

    config.vm.provider "virtualbox" do |v|
        v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
    end
