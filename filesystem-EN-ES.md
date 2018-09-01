# Almacenamiento de archivos : File Storage

- [Introduction](#introduction)
- [Configuration](#configuration)
    - [The Public Disk](#the-public-disk)
    - [The Local Driver](#the-local-driver)
    - [Driver Prerequisites](#driver-prerequisites)
    - [Caching](#caching)
- [Obtaining Disk Instances](#obtaining-disk-instances)
- [Retrieving Files](#retrieving-files)
    - [Downloading Files](#downloading-files)
    - [File URLs](#file-urls)
    - [File Metadata](#file-metadata)
- [Storing Files](#storing-files)
    - [File Uploads](#file-uploads)
    - [File Visibility](#file-visibility)
- [Deleting Files](#deleting-files)
- [Directories](#directories)
- [Custom Filesystems](#custom-filesystems)

<a name="introduction"></a>
## Introducción : Introduction

Laravel proporciona una poderosa abstracción del sistema de archivos gracias al maravilloso paquete de PHP [Flysystem](https://github.com/thephpleague/flysystem) de Frank de Jonge. La integración de Laravel Flysystem proporciona controladores fáciles de usar para trabajar con sistemas de archivos locales, Amazon S3 y Rackspace Cloud Storage. Aún mejor, es increíblemente simple cambiar entre estas opciones de almacenamiento ya que la API sigue siendo la misma para cada sistema.
> > Laravel provides a powerful filesystem abstraction thanks to the wonderful [Flysystem](https://github.com/thephpleague/flysystem) PHP package by Frank de Jonge. The Laravel Flysystem integration provides simple to use drivers for working with local filesystems, Amazon S3, and Rackspace Cloud Storage. Even better, it's amazingly simple to switch between these storage options as the API remains the same for each system.

<a name="configuration"></a>
## Configuración : Configuration

El archivo de configuración del sistema de archivos se encuentra en `config/filesystems.php`. Dentro de este archivo puede configurar todos sus "discos". Cada disco representa un controlador de almacenamiento y una ubicación de almacenamiento en particular. Las configuraciones de ejemplo para cada controlador compatible se incluyen en el archivo de configuración. Por lo tanto, modifique la configuración para reflejar sus preferencias y credenciales de almacenamiento.
> > The filesystem configuration file is located at `config/filesystems.php`. Within this file you may configure all of your "disks". Each disk represents a particular storage driver and storage location. Example configurations for each supported driver are included in the configuration file. So, modify the configuration to reflect your storage preferences and credentials.

Por supuesto, puede configurar tantos discos como desee e incluso puede tener varios discos que usen el mismo controlador.
> > Of course, you may configure as many disks as you like, and may even have multiple disks that use the same driver.

<a name="the-public-disk"></a>
### El disco público : The Public Disk

El disco `public` está destinado a archivos que serán de acceso público. Por defecto, el disco `public` usa el controlador `local` y almacena estos archivos en `storage/app/public`. Para hacerlos accesibles desde la web, debe crear un enlace simbólico de `public/storage` a `storage/app/public`. Esta convención mantendrá sus archivos de acceso público en un directorio que se puede compartir fácilmente en las implementaciones cuando se utilizan sistemas de implementación sin tiempo de inactividad como [Envoyer](https://envoyer.io).
> > The `public` disk is intended for files that are going to be publicly accessible. By default, the `public` disk uses the `local` driver and stores these files in `storage/app/public`. To make them accessible from the web, you should create a symbolic link from `public/storage` to `storage/app/public`. This convention will keep your publicly accessible files in one directory that can be easily shared across deployments when using zero down-time deployment systems like [Envoyer](https://envoyer.io).

Para crear el enlace simbólico, puede usar el comando Artisan `storage:link`:
> > To create the symbolic link, you may use the `storage:link` Artisan command:

    php artisan storage:link

Por supuesto, una vez que se ha almacenado un archivo y se ha creado el enlace simbólico, puede crear una URL para los archivos usando el helper `asset`:
> > Of course, once a file has been stored and the symbolic link has been created, you can create a URL to the files using the `asset` helper:

    echo asset('storage/file.txt');

<a name="the-local-driver"></a>
### El controlador local : The Local Driver

Cuando se utiliza el controlador `local`, todas las operaciones de archivo son relativas al directorio `root` definido en su archivo de configuración. Por defecto, este valor se establece en el directorio `storage/app`. Por lo tanto, el siguiente método almacenaría un archivo en `storage/app/file.txt`:
> > When using the `local` driver, all file operations are relative to the `root` directory defined in your configuration file. By default, this value is set to the `storage/app` directory. Therefore, the following method would store a file in `storage/app/file.txt`:

    Storage::disk('local')->put('file.txt', 'Contents');

<a name="driver-prerequisites"></a>
### Prerrequisitos del Driver : Driver Prerequisites

#### Paquetes Composer : Composer Packages

Antes de usar los controladores SFTP, S3 o Rackspace, deberá instalar el paquete apropiado a través de Composer:
> > Before using the SFTP, S3, or Rackspace drivers, you will need to install the appropriate package via Composer:

- SFTP: `league/flysystem-sftp ~1.0`
- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

Una necesidad absoluta para el rendimiento es usar un adaptador en caché. Necesitará un paquete adicional para esto:
> > An absolute must for performance is to use a cached adapter. You will need an additional package for this:

- CachedAdapter: `league/flysystem-cached-adapter ~1.0`

#### Configuración del Driver S3 : S3 Driver Configuration

La información de configuración del controlador S3 se encuentra en su archivo de configuración `config/filesystems.php`. Este archivo contiene un array de configuración de ejemplo para un controlador S3. Usted es libre de modificar esta matriz con su propia configuración y credenciales de S3. Por conveniencia, estas variables de entorno coinciden con la convención de nomenclatura utilizada por AWS CLI.
> > The S3 driver configuration information is located in your `config/filesystems.php` configuration file. This file contains an example configuration array for an S3 driver. You are free to modify this array with your own S3 configuration and credentials. For convenience, these environment variables match the naming convention used by the AWS CLI.

#### Configuración del Driver FTP : FTP Driver Configuration

Las integraciones de Laravel en Flysystem funcionan muy bien con FTP; sin embargo, no se incluye una configuración de muestra con el archivo de configuración `filesystems.php` predeterminado de la estructura. Si necesita configurar un sistema de archivos FTP, puede usar la siguiente configuración de ejemplo:
> > Laravel's Flysystem integrations works great with FTP; however, a sample configuration is not included with the framework's default `filesystems.php` configuration file. If you need to configure a FTP filesystem, you may use the example configuration below:

    'ftp' => [
        'driver'   => 'ftp',
        'host'     => 'ftp.example.com',
        'username' => 'your-username',
        'password' => 'your-password',

        // Optional FTP Settings...
        // 'port'     => 21,
        // 'root'     => '',
        // 'passive'  => true,
        // 'ssl'      => true,
        // 'timeout'  => 30,
    ],

#### Configuración del Driver SFTP : SFTP Driver Configuration

Las integraciones Flysystem de Laravel funcionan muy bien con SFTP; sin embargo, no se incluye una configuración de muestra con el archivo de configuración `filesystems.php` predeterminado de la estructura. Si necesita configurar un sistema de archivos SFTP, puede usar la siguiente configuración de ejemplo:
> > Laravel's Flysystem integrations works great with SFTP; however, a sample configuration is not included with the framework's default `filesystems.php` configuration file. If you need to configure a SFTP filesystem, you may use the example configuration below:

    'sftp' => [
        'driver' => 'sftp',
        'host' => 'example.com',
        'username' => 'your-username',
        'password' => 'your-password',

        // Settings for SSH key based authentication...
        // 'privateKey' => '/path/to/privateKey',
        // 'password' => 'encryption-password',

        // Optional SFTP Settings...
        // 'port' => 22,
        // 'root' => '',
        // 'timeout' => 30,
    ],

#### Configuración del Driver Rackspace : Rackspace Driver Configuration

Las integraciones Flysystem de Laravel funcionan muy bien con Rackspace; sin embargo, no se incluye una configuración de muestra con el archivo de configuración `filesystems.php` predeterminado de la estructura. Si necesita configurar un sistema de archivos Rackspace, puede usar la siguiente configuración de ejemplo:
> > Laravel's Flysystem integrations works great with Rackspace; however, a sample configuration is not included with the framework's default `filesystems.php` configuration file. If you need to configure a Rackspace filesystem, you may use the example configuration below:

    'rackspace' => [
        'driver'    => 'rackspace',
        'username'  => 'your-username',
        'key'       => 'your-key',
        'container' => 'your-container',
        'endpoint'  => 'https://identity.api.rackspacecloud.com/v2.0/',
        'region'    => 'IAD',
        'url_type'  => 'publicURL',
    ],

<a name="caching"></a>
### Almacenamiento en caché : Caching

Para habilitar el almacenamiento en caché de un disco determinado, puede agregar una directiva `cache` a las opciones de configuración del disco. La opción `caché` debe ser un array de opciones de almacenamiento en caché que contenga el nombre `disk`, el tiempo `expire` en segundos y el `prefix` de caché:
> > To enable caching for a given disk, you may add a `cache` directive to the disk's configuration options. The `cache` option should be an array of caching options containing the `disk` name, the `expire` time in seconds, and the cache `prefix`:

    's3' => [
        'driver' => 's3',

        // Other Disk Options...

        'cache' => [
            'store' => 'memcached',
            'expire' => 600,
            'prefix' => 'cache-prefix',
        ],
    ],

<a name="obtaining-disk-instances"></a>
## Obtención de instancias de disco : Obtaining Disk Instances

La fachada `Storage` se puede usar para interactuar con cualquiera de sus discos configurados. Por ejemplo, puede usar el método `put` en la fachada para almacenar un avatar en el disco predeterminado. Si llama a métodos en la fachada `Storage` sin llamar primero al método `disk`, la llamada al método se pasará automáticamente al disco predeterminado:
> > The `Storage` facade may be used to interact with any of your configured disks. For example, you may use the `put` method on the facade to store an avatar on the default disk. If you call methods on the `Storage` facade without first calling the `disk` method, the method call will automatically be passed to the default disk:

    use Illuminate\Support\Facades\Storage;

    Storage::put('avatars/1', $fileContents);

Si sus aplicaciones interactúan con múltiples discos, puede usar el método `disk` en la fachada `Storage` para trabajar con archivos en un disco particular:
> > If your applications interacts with multiple disks, you may use the `disk` method on the `Storage` facade to work with files on a particular disk:

    Storage::disk('s3')->put('avatars/1', $fileContents);

<a name="retrieving-files"></a>
## Recuperando archivos : Retrieving Files

El método `get` puede usarse para recuperar el contenido de un archivo. El contenido de la cadena sin formato del archivo será devuelto por el método. Recuerde, todas las rutas de archivos deben especificarse en relación con la ubicación "raíz" configurada para el disco:
> > The `get` method may be used to retrieve the contents of a file. The raw string contents of the file will be returned by the method. Remember, all file paths should be specified relative to the "root" location configured for the disk:

    $contents = Storage::get('file.jpg');

El método `exists` se puede usar para determinar si existe un archivo en el disco:
> > The `exists` method may be used to determine if a file exists on the disk:

    $exists = Storage::disk('s3')->exists('file.jpg');

<a name="downloading-files"></a>
### Descargando archivos : Downloading Files

El método `download` se puede usar para generar una respuesta que obligue al navegador del usuario a descargar el archivo en la ruta determinada. El método `download` acepta un nombre de archivo como segundo argumento para el método, que determinará el nombre de archivo que ve el usuario que descarga el archivo. Finalmente, puede pasar un array de encabezados HTTP como tercer argumento del método:
> > The `download` method may be used to generate a response that forces the user's browser to download the file at the given path. The `download` method accepts a file name as the second argument to the method, which will determine the file name that is seen by the user downloading the file. Finally, you may pass an array of HTTP headers as the third argument to the method:

    return Storage::download('file.jpg');

    return Storage::download('file.jpg', $name, $headers);

<a name="file-urls"></a>
### URLs de archivos : File URLs

Puede usar el método `url` para obtener la URL del archivo dado. Si está utilizando el controlador `local`, esto normalmente solo añadirá `/storage` a la ruta determinada y devolverá una URL relativa al archivo. Si está utilizando el controlador `s3` o `rackspace`, se devolverá la URL remota totalmente calificada:
> > You may use the `url` method to get the URL for the given file. If you are using the `local` driver, this will typically just prepend `/storage` to the given path and return a relative URL to the file. If you are using the `s3` or `rackspace` driver, the fully qualified remote URL will be returned:

    use Illuminate\Support\Facades\Storage;

    $url = Storage::url('file.jpg');

> {note} Recuerde, si está usando el controlador `local`, todos los archivos que deberían ser de acceso público deben colocarse en el directorio `storage/app/public`. Además, debe [crear un enlace simbólico](#the-public-disk) en `public/storage` que apunta al directorio `storage/app/public`.
> > > {note} Remember, if you are using the `local` driver, all files that should be publicly accessible should be placed in the `storage/app/public` directory. Furthermore, you should [create a symbolic link](#the-public-disk) at `public/storage` which points to the `storage/app/public` directory.

#### URLs temporales : Temporary URLs

Para los archivos almacenados utilizando el controlador `s3` o `rackspace`, puede crear una URL temporal para un archivo determinado utilizando el método `temporaryUrl`. Este método acepta una ruta y una instancia `DateTime` que especifica cuándo caducará la URL:
> > For files stored using the `s3` or `rackspace` driver, you may create a temporary URL to a given file using the `temporaryUrl` method. This methods accepts a path and a `DateTime` instance specifying when the URL should expire:

    $url = Storage::temporaryUrl(
        'file.jpg', now()->addMinutes(5)
    );

#### Personalización del host de URL local : Local URL Host Customization

Si desea predefinir el host para los archivos almacenados en un disco utilizando el controlador `local`, puede agregar una opción `url` al array de configuración del disco:
> > If you would like to pre-define the host for files stored on a disk using the `local` driver, you may add a `url` option to the disk's configuration array:

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],

<a name="file-metadata"></a>
### Metadatos de archivos : File Metadata

Además de leer y escribir archivos, Laravel también puede proporcionar información sobre los archivos en sí. Por ejemplo, el método `size` se puede usar para obtener el tamaño del archivo en bytes:
> > In addition to reading and writing files, Laravel can also provide information about the files themselves. For example, the `size` method may be used to get the size of the file in bytes:

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file.jpg');

El método `lastModified` devuelve la marca de tiempo UNIX de la última vez que se modificó el archivo:
> > The `lastModified` method returns the UNIX timestamp of the last time the file was modified:

    $time = Storage::lastModified('file.jpg');

<a name="storing-files"></a>
## Almacenamiento de archivos : Storing Files

El método `put` se puede usar para almacenar contenido de archivos sin formato en un disco. También puede pasar un `recurso` de PHP al método `put`, que usará el soporte de flujo subyacente de Flysystem. El uso de transmisiones es muy recomendable cuando se trata de archivos grandes:
> > The `put` method may be used to store raw file contents on a disk. You may also pass a PHP `resource` to the `put` method, which will use Flysystem's underlying stream support. Using streams is greatly recommended when dealing with large files:

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

#### Transmisión automática : Automatic Streaming

Si desea que Laravel gestione automáticamente la transmisión de un archivo determinado a su ubicación de almacenamiento, puede utilizar el método `putFile` o `putFileAs`. Este método acepta una instancia `Illuminate\Http\File` o `Illuminate\Http\UploadedFile` y automáticamente transmitirá el archivo a la ubicación deseada:
> > If you would like Laravel to automatically manage streaming a given file to your storage location, you may use the `putFile` or `putFileAs` method. This method accepts either a `Illuminate\Http\File` or `Illuminate\Http\UploadedFile` instance and will automatically stream the file to your desired location:

    use Illuminate\Http\File;
    use Illuminate\Support\Facades\Storage;

    // Automatically generate a unique ID for file name...
    Storage::putFile('photos', new File('/path/to/photo'));

    // Manually specify a file name...
    Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');

Hay algunas cosas importantes a tener en cuenta sobre el método `putFile`. Tenga en cuenta que solo especificamos un nombre de directorio, no un nombre de archivo. Por defecto, el método `putFile` generará una identificación única para servir como nombre de archivo. La extensión del archivo se determinará al examinar el tipo MIME del archivo. La ruta al archivo será devuelta por el método `putFile` para que pueda almacenar la ruta, incluido el nombre del archivo generado, en su base de datos.
> > There are a few important things to note about the `putFile` method. Note that we only specified a directory name, not a file name. By default, the `putFile` method will generate a unique ID to serve as the file name. The file's extension will be determined by examining the file's MIME type. The path to the file will be returned by the `putFile` method so you can store the path, including the generated file name, in your database.

Los métodos `putFile` y `putFileAs` también aceptan un argumento para especificar la "visibilidad" del archivo almacenado. Esto es particularmente útil si está almacenando el archivo en un disco de la nube como S3 y desea que el archivo sea de acceso público:
> > The `putFile` and `putFileAs` methods also accept an argument to specify the "visibility" of the stored file. This is particularly useful if you are storing the file on a cloud disk such as S3 and would like the file to be publicly accessible:

    Storage::putFile('photos', new File('/path/to/photo'), 'public');

#### Prepending & Appending To Files

Los métodos `prepend` y `append` le permiten escribir al principio o al final de un archivo:
> > The `prepend` and `append` methods allow you to write to the beginning or end of a file:

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

#### Copying & Moving Files : Copying & Moving Files

El método `copy` se puede usar para copiar un archivo existente a una nueva ubicación en el disco, mientras que el método `move` se puede usar para cambiar el nombre o mover un archivo existente a una nueva ubicación:
> > The `copy` method may be used to copy an existing file to a new location on the disk, while the `move` method may be used to rename or move an existing file to a new location:

    Storage::copy('old/file.jpg', 'new/file.jpg');

    Storage::move('old/file.jpg', 'new/file.jpg');

<a name="file-uploads"></a>
### Subidas de archivos : File Uploads

En las aplicaciones web, uno de los casos de uso más comunes para almacenar archivos es almacenar archivos cargados por el usuario, como fotos de perfil, fotos y documentos. Laravel hace que sea muy fácil almacenar archivos cargados usando el método `store` en una instancia de archivo cargada. Llame al método `store` con la ruta en la que desea almacenar el archivo cargado:
> > In web applications, one of the most common use-cases for storing files is storing user uploaded files such as profile pictures, photos, and documents. Laravel makes it very easy to store uploaded files using the `store` method on an uploaded file instance. Call the `store` method with the path at which you wish to store the uploaded file:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserAvatarController extends Controller
    {
        /**
         * Update the avatar for the user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            $path = $request->file('avatar')->store('avatars');

            return $path;
        }
    }

Hay algunas cosas importantes a tener en cuenta sobre este ejemplo. Tenga en cuenta que solo especificamos un nombre de directorio, no un nombre de archivo. Por defecto, el método `store` generará una identificación única para servir como nombre de archivo. La extensión del archivo se determinará al examinar el tipo MIME del archivo. La ruta al archivo será devuelta por el método `store` para que pueda almacenar la ruta, incluido el nombre del archivo generado, en su base de datos.
> > There are a few important things to note about this example. Note that we only specified a directory name, not a file name. By default, the `store` method will generate a unique ID to serve as the file name. The file's extension will be determined by examining the file's MIME type. The path to the file will be returned by the `store` method so you can store the path, including the generated file name, in your database.

También puede llamar al método `putFile` en la fachada `Storage` para realizar la misma manipulación de archivos que en el ejemplo anterior:
> > You may also call the `putFile` method on the `Storage` facade to perform the same file manipulation as the example above:

    $path = Storage::putFile('avatars', $request->file('avatar'));

#### Especificación de un nombre de archivo : Specifying A File Name

Si no desea que un nombre de archivo se asigne automáticamente a su archivo almacenado, puede usar el método `storeAs`, que recibe la ruta, el nombre del archivo y el disco (opcional) como sus argumentos:
> > If you would not like a file name to be automatically assigned to your stored file, you may use the `storeAs` method, which receives the path, the file name, and the (optional) disk as its arguments:

    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );

Por supuesto, también puede usar el método `putFileAs` en la fachada `Storage`, que realizará la misma manipulación de archivos que en el ejemplo anterior:
> > Of course, you may also use the `putFileAs` method on the `Storage` facade, which will perform the same file manipulation as the example above:

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );

#### Especificación de un disco : Specifying A Disk

Por defecto, este método usará su disco predeterminado. Si desea especificar otro disco, pase el nombre del disco como el segundo argumento al método `store`:
> > By default, this method will use your default disk. If you would like to specify another disk, pass the disk name as the second argument to the `store` method:

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );

<a name="file-visibility"></a>
### Visibilidad del archivo : File Visibility

En la integración Flysystem de Laravel, "visibilidad" es una abstracción de permisos de archivos en múltiples plataformas. Los archivos pueden ser declarados `public` o `private`. Cuando un archivo se declara como `public`, está indicando que el archivo generalmente debería ser accesible para otros. Por ejemplo, al usar el controlador S3, puede recuperar URLs para archivos `public`.
> > In Laravel's Flysystem integration, "visibility" is an abstraction of file permissions across multiple platforms. Files may either be declared `public` or `private`. When a file is declared `public`, you are indicating that the file should generally be accessible to others. For example, when using the S3 driver, you may retrieve URLs for `public` files.

Puede establecer la visibilidad al configurar el archivo mediante el método `put`:
> > You can set the visibility when setting the file via the `put` method:

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents, 'public');

Si el archivo ya se ha almacenado, su visibilidad se puede recuperar y establecer a través de los métodos `getVisibility` y `setVisibility`:
> > If the file has already been stored, its visibility can be retrieved and set via the `getVisibility` and `setVisibility` methods:

    $visibility = Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public')

<a name="deleting-files"></a>
## Eliminación de archivos : Deleting Files

El método `delete` acepta un único nombre de archivo o un array de archivos para eliminar del disco:
> > The `delete` method accepts a single filename or an array of files to remove from the disk:

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file.jpg', 'file2.jpg']);

Si es necesario, puede especificar el disco del que debe eliminarse el archivo:
> > If necessary, you may specify the disk that the file should be deleted from:

    use Illuminate\Support\Facades\Storage;

    Storage::disk('s3')->delete('folder_path/file_name.jpg');

<a name="directories"></a>
## Directories : Directories

#### Obtener todos los archivos de un directorio : Get All Files Within A Directory

El método `files` devuelve una matriz de todos los archivos en un directorio determinado. Si desea recuperar una lista de todos los archivos dentro de un directorio determinado, incluidos todos los subdirectorios, puede usar el método `allFiles`:
> > The `files` method returns an array of all of the files in a given directory. If you would like to retrieve a list of all files within a given directory including all sub-directories, you may use the `allFiles` method:

    use Illuminate\Support\Facades\Storage;

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

#### Obtener todos los directorios dentro de un directorio : Get All Directories Within A Directory

El método `directories` devuelve una matriz de todos los directorios dentro de un directorio determinado. Además, puede usar el método `allDirectories` para obtener una lista de todos los directorios dentro de un directorio determinado y todos sus subdirectorios:
> > The `directories` method returns an array of all the directories within a given directory. Additionally, you may use the `allDirectories` method to get a list of all directories within a given directory and all of its sub-directories:

    $directories = Storage::directories($directory);

    // Recursive...
    $directories = Storage::allDirectories($directory);

#### Crear un directorio : Create A Directory

El método `makeDirectory` creará el directorio dado, incluyendo cualquier subdirectorio necesario:
> > The `makeDirectory` method will create the given directory, including any needed sub-directories:

    Storage::makeDirectory($directory);

#### Eliminar un directorio : Delete A Directory

Finalmente, el `deleteDirectory` se puede usar para eliminar un directorio y todos sus archivos:
> > Finally, the `deleteDirectory` may be used to remove a directory and all of its files:

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## Sistemas de archivos personalizados : Custom Filesystems

La integración con Flysystem de Laravel proporciona controladores para varios "controladores" listos para usar; sin embargo, Flysystem no está limitado a estos y tiene adaptadores para muchos otros sistemas de almacenamiento. Puede crear un controlador personalizado si desea utilizar uno de estos adaptadores adicionales en su aplicación Laravel.
> > Laravel's Flysystem integration provides drivers for several "drivers" out of the box; however, Flysystem is not limited to these and has adapters for many other storage systems. You can create a custom driver if you want to use one of these additional adapters in your Laravel application.

Para configurar el sistema de archivos personalizado, necesitará un adaptador Flysystem. Agreguemos un adaptador de Dropbox mantenido por la comunidad a nuestro proyecto:
> > In order to set up the custom filesystem you will need a Flysystem adapter. Let's add a community maintained Dropbox adapter to our project:

    composer require spatie/flysystem-dropbox

A continuación, debe crear un [proveedor de servicios](/docs/{{version}}/providers) como `DropboxServiceProvider`. En el método `boot` del proveedor, puede usar el método `extend` de la fachada `Storage` para definir el controlador personalizado:
> > Next, you should create a [service provider](/docs/{{version}}/providers) such as `DropboxServiceProvider`. In the provider's `boot` method, you may use the `Storage` facade's `extend` method to define the custom driver:

    <?php

    namespace App\Providers;

    use Storage;
    use League\Flysystem\Filesystem;
    use Illuminate\Support\ServiceProvider;
    use Spatie\Dropbox\Client as DropboxClient;
    use Spatie\FlysystemDropbox\DropboxAdapter;

    class DropboxServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Storage::extend('dropbox', function ($app, $config) {
                $client = new DropboxClient(
                    $config['authorization_token']
                );

                return new Filesystem(new DropboxAdapter($client));
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

El primer argumento del método `extend` es el nombre del controlador y el segundo es un Cierre que recibe las variables `$app` y `$config`. El cierre de resolver debe devolver una instancia de `League\Flysystem\Filesystem`. La variable `$config` contiene los valores definidos en `config/filesystems.php` para el disco especificado.
> > The first argument of the `extend` method is the name of the driver and the second is a Closure that receives the `$app` and `$config` variables. The resolver Closure must return an instance of `League\Flysystem\Filesystem`. The `$config` variable contains the values defined in `config/filesystems.php` for the specified disk.

Una vez que haya creado el proveedor de servicios para registrar la extensión, puede usar el controlador `dropbox` en su archivo de configuración `config/filesystems.php`.
> > Once you have created the service provider to register the extension, you may use the `dropbox` driver in your `config/filesystems.php` configuration file.
