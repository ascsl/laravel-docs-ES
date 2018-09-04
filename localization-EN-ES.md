# Localización : Localization

- [Introduction](#introduction)
- [Defining Translation Strings](#defining-translation-strings)
    - [Using Short Keys](#using-short-keys)
    - [Using Translation Strings As Keys](#using-translation-strings-as-keys)
- [Retrieving Translation Strings](#retrieving-translation-strings)
    - [Replacing Parameters In Translation Strings](#replacing-parameters-in-translation-strings)
    - [Pluralization](#pluralization)
- [Overriding Package Language Files](#overriding-package-language-files)

<a name="introduction"></a>
## Introducción : Introduction

Las funciones de localización de Laravel proporcionan una forma conveniente de recuperar cadenas en varios idiomas, lo que le permite admitir fácilmente varios idiomas dentro de su aplicación. Las cadenas de idioma se almacenan en archivos dentro del directorio `resources/lang`. Dentro de este directorio debe haber un subdirectorio para cada idioma admitido por la aplicación:
> > Laravel's localization features provide a convenient way to retrieve strings in various languages, allowing you to easily support multiple languages within your application. Language strings are stored in files within the `resources/lang` directory. Within this directory there should be a subdirectory for each language supported by the application:

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

Todos los archivos de idioma devuelven un array de cadenas con clave. Por ejemplo:
> > All language files return an array of keyed strings. For example:

    <?php

    return [
        'welcome' => 'Welcome to our application'
    ];

### Configuración regional : Configuring The Locale

El idioma predeterminado para su aplicación se almacena en el archivo de configuración `config/app.php`. Por supuesto, puede modificar este valor para satisfacer las necesidades de su aplicación. También puede cambiar el idioma activo en tiempo de ejecución utilizando el método `setLocale` en la fachada `App`:
> > The default language for your application is stored in the `config/app.php` configuration file. Of course, you may modify this value to suit the needs of your application. You may also change the active language at runtime using the `setLocale` method on the `App` facade:

    Route::get('welcome/{locale}', function ($locale) {
        App::setLocale($locale);

        //
    });

Puede configurar un "lenguaje alternativo", que se usará cuando el idioma activo no contenga una cadena de traducción dada. Al igual que el idioma predeterminado, el idioma alternativo también se configura en el archivo de configuración `config/app.php`:
> > You may configure a "fallback language", which will be used when the active language does not contain a given translation string. Like the default language, the fallback language is also configured in the `config/app.php` configuration file:

    'fallback_locale' => 'en',

#### Determinación de la configuración regional actual : Determining The Current Locale

Puede usar los métodos `getLocale` y `isLocale` en la fachada `App` para determinar la configuración regional actual o verificar si la configuración regional es un valor dado:
> > You may use the `getLocale` and `isLocale` methods on the `App` facade to determine the current locale or check if the locale is a given value:

    $locale = App::getLocale();

    if (App::isLocale('en')) {
        //
    }

<a name="defining-translation-strings"></a>
## Definición de cadenas de traducción : Defining Translation Strings

<a name="using-short-keys"></a>
### Uso de claves cortas : Using Short Keys

Normalmente, las cadenas de traducción se almacenan en archivos dentro del directorio `resources/lang`. Dentro de este directorio debe haber un subdirectorio para cada idioma admitido por la aplicación:
> > Typically, translation strings are stored in files within the `resources/lang` directory. Within this directory there should be a subdirectory for each language supported by the application:

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

Todos los archivos de idioma devuelven una matriz de cadenas con clave. Por ejemplo:
> > All language files return an array of keyed strings. For example:

    <?php

    // resources/lang/en/messages.php

    return [
        'welcome' => 'Welcome to our application'
    ];

<a name="using-translation-strings-as-keys"></a>
### Uso de cadenas de traducción como claves : Using Translation Strings As Keys

Para aplicaciones con grandes requisitos de traducción, definir cada cadena con una "clave corta" puede volverse rápidamente confusa al hacer referencia a ellas en sus vistas. Por esta razón, Laravel también proporciona soporte para definir cadenas de traducción utilizando la traducción "predeterminada" de la cadena como la clave.
> > For applications with heavy translation requirements, defining every string with a "short key" can become quickly confusing when referencing them in your views. For this reason, Laravel also provides support for defining translation strings using the "default" translation of the string as the key.

Los archivos de traducción que usan cadenas de traducción como claves se almacenan como archivos JSON en el directorio `resources/lang`. Por ejemplo, si su aplicación tiene una traducción al español, debe crear un archivo `resources/lang/es.json`:
> > Translation files that use translation strings as keys are stored as JSON files in the `resources/lang` directory. For example, if your application has a Spanish translation, you should create a `resources/lang/es.json` file:

    {
        "I love programming.": "Me encanta programar."
    }

<a name="retrieving-translation-strings"></a>
## Recuperando cadenas de traducción : Retrieving Translation Strings

Puede recuperar líneas de archivos de idioma utilizando la función de ayuda `__`. El método `__` acepta el archivo y la clave de la cadena de traducción como su primer argumento. Por ejemplo, recuperemos la cadena de traducción `welcome` del archivo de idioma `resources/lang/messages.php`:
> > You may retrieve lines from language files using the `__` helper function. The `__` method accepts the file and key of the translation string as its first argument. For example, let's retrieve the `welcome` translation string from the `resources/lang/messages.php` language file:

    echo __('messages.welcome');

    echo __('I love programming.');

Por supuesto, si está utilizando el [motor de plantillas Blade](/docs/{{version}}/blade), puede usar la sintaxis `{{ }}` para hacer eco de la cadena de traducción o usar la directiva `@lang`:
> > Of course if you are using the [Blade templating engine](/docs/{{version}}/blade), you may use the `{{ }}` syntax to echo the translation string or use the `@lang` directive:

    {{ __('messages.welcome') }}

    @lang('messages.welcome')

Si la cadena de traducción especificada no existe, la función `__` devolverá la clave de cadena de traducción. Entonces, usando el ejemplo anterior, la función `__` devolvería `messages.welcome` si la cadena de traducción no existe.
> > If the specified translation string does not exist, the `__` function will return the translation string key. So, using the example above, the `__` function would return `messages.welcome` if the translation string does not exist.

<a name="replacing-parameters-in-translation-strings"></a>
### Reemplazar parámetros en cadenas de traducción : Replacing Parameters In Translation Strings

Si lo desea, puede definir marcadores en sus cadenas de traducción. Todos los marcadores de posición tienen el prefijo `:`. Por ejemplo, puede definir un mensaje de bienvenida con un marcador:
> > If you wish, you may define place-holders in your translation strings. All place-holders are prefixed with a `:`. For example, you may define a welcome message with a place-holder name:

    'welcome' => 'Welcome, :name',

Para reemplazar los marcadores de posición al recuperar una cadena de traducción, pase un array de reemplazos como segundo argumento a la función `__`:
> > To replace the place-holders when retrieving a translation string, pass an array of replacements as the second argument to the `__` function:

    echo __('messages.welcome', ['name' => 'dayle']);

Si su marcador de posición contiene todas las letras mayúsculas, o solo tiene su primera letra en mayúscula, el valor traducido se escribirá en mayúscula en consecuencia:
> > If your place-holder contains all capital letters, or only has its first letter capitalized, the translated value will be capitalized accordingly:

    'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
    'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle

<a name="pluralization"></a>
### Pluralización : Pluralization

La pluralización es un problema complejo, ya que los diferentes idiomas tienen una variedad de reglas complejas para la pluralización. Al usar un carácter "pipa", puedes distinguir las formas en singular y plural de una cadena:
> > Pluralization is a complex problem, as different languages have a variety of complex rules for pluralization. By using a "pipe" character, you may distinguish singular and plural forms of a string:

    'apples' => 'There is one apple|There are many apples',

Incluso puede crear reglas de pluralización más complejas que especifiquen cadenas de traducción para múltiples rangos de números:
> > You may even create more complex pluralization rules which specify translation strings for multiple number ranges:

    'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',

Después de definir una cadena de traducción que tenga opciones de pluralización, puede usar la función `trans_choice` para recuperar la línea para un "conteo" dado. En este ejemplo, dado que el recuento es mayor que uno, se devuelve la forma plural de la cadena de traducción:
> > After defining a translation string that has pluralization options, you may use the `trans_choice` function to retrieve the line for a given "count". In this example, since the count is greater than one, the plural form of the translation string is returned:

    echo trans_choice('messages.apples', 10);

También puede definir atributos de marcador de posición en cadenas de pluralización. Estos marcadores se pueden reemplazar pasando un arraç como tercer argumento de la función `trans_choice`:
> > You may also define place-holder attributes in pluralization strings. These place-holders may be replaced by passing an array as the third argument to the `trans_choice` function:

    'minutes_ago' => '{1} :value minute ago|[2,*] :value minutes ago',

    echo trans_choice('time.minutes_ago', 5, ['value' => 5]);

<a name="overriding-package-language-files"></a>
## Overriding Package Language Files

Algunos paquetes pueden enviarse con sus propios archivos de idioma. En lugar de cambiar los archivos principales del paquete para ajustar estas líneas, puede anularlas colocando archivos en el directorio `resources/lang/vendor/{package}/{locale}`.
> > Some packages may ship with their own language files. Instead of changing the package's core files to tweak these lines, you may override them by placing files in the `resources/lang/vendor/{package}/{locale}` directory.

Entonces, por ejemplo, si necesita anular las cadenas de traducción en inglés en `messages.php` para un paquete llamado `skyrim/hearthfire`, debe colocar un archivo de idioma en: `resources/lang/vendor/hearthfire/en/messages.php`. Dentro de este archivo, solo debe definir las cadenas de traducción que desea sobrescribir. Cualquier cadena de traducción que no anule se cargará desde los archivos de idioma originales del paquete.
> > So, for example, if you need to override the English translation strings in `messages.php` for a package named `skyrim/hearthfire`, you should place a language file at: `resources/lang/vendor/hearthfire/en/messages.php`. Within this file, you should only define the translation strings you wish to override. Any translation strings you don't override will still be loaded from the package's original language files.
