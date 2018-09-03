# Hashing

- [Introduction](#introduction)
- [Configuration](#configuration)
- [Basic Usage](#basic-usage)

<a name="introduction"></a>
## Introducción : Introduction

Laravel `Hash` [facade](/docs/{{version}}/facades) proporciona un cifrado seguro de Bcrypt y Argon2 para almacenar las contraseñas de los usuarios. Si está utilizando las clases incorporadas `LoginController` y `RegisterController` que se incluyen con su aplicación Laravel, usarán Bcrypt para el registro y la autenticación de forma predeterminada.
> > The Laravel `Hash` [facade](/docs/{{version}}/facades) provides secure Bcrypt and Argon2 hashing for storing user passwords. If you are using the built-in `LoginController` and `RegisterController` classes that are included with your Laravel application, they will use Bcrypt for registration and authentication by default.

> {tip} Bcrypt es una gran opción para contraseñas hash porque su "factor de trabajo" es ajustable, lo que significa que el tiempo que se tarda en generar un hash puede aumentar a medida que aumenta la potencia del hardware.
> > > {tip} Bcrypt is a great choice for hashing passwords because its "work factor" is adjustable, which means that the time it takes to generate a hash can be increased as hardware power increases.

<a name="configuration"></a>
## Configuración : Configuration

El controlador hash predeterminado para su aplicación se configura en el archivo de configuración `config/hashing.php`. Actualmente hay tres controladores admitidos: [Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) y [Argon2](https://en.wikipedia.org/wiki/Argon2) (variantes de Argon2i y Argon2id) .
> > The default hashing driver for your application is configured in the `config/hashing.php` configuration file. There are currently three supported drivers: [Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) and [Argon2](https://en.wikipedia.org/wiki/Argon2) (Argon2i and Argon2id variants).

> {note} El controlador Argon2i requiere PHP 7.2.0 o superior y el controlador Argon2id requiere PHP 7.3.0 o superior.
> > > {note} The Argon2i driver requires PHP 7.2.0 or greater and the Argon2id driver requires PHP 7.3.0 or greater.

<a name="basic-usage"></a>
## Uso básico : Basic Usage

Puede "hash" una contraseña llamando al método `make` en la fachada `Hash`:
> > You may hash a password by calling the `make` method on the `Hash` facade:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use App\Http\Controllers\Controller;

    class UpdatePasswordController extends Controller
    {
        /**
         * Update the password for the user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // Validate the new password length...

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

#### Ajuste del factor de trabajo Bcrypt : Adjusting The Bcrypt Work Factor

Si está utilizando el algoritmo Bcrypt, el método `make` le permite administrar el factor de trabajo del algoritmo utilizando la opción `rounds`; sin embargo, el valor predeterminado es aceptable para la mayoría de las aplicaciones:
> > If you are using the Bcrypt algorithm, the `make` method allows you to manage the work factor of the algorithm using the `rounds` option; however, the default is acceptable for most applications:

    $hashed = Hash::make('password', [
        'rounds' => 12
    ]);

#### Ajuste del factor de trabajo Argon2 : Adjusting The Argon2 Work Factor

Si está utilizando el algoritmo Argon2, el método `make` le permite administrar el factor de trabajo del algoritmo utilizando las opciones `memory`, `time`, y `threads`; sin embargo, los valores predeterminados son aceptables para la mayoría de las aplicaciones:
> > If you are using the Argon2 algorithm, the `make` method allows you to manage the work factor of the algorithm using the `memory`, `time`, and `threads` options; however, the defaults are acceptable for most applications:

    $hashed = Hash::make('password', [
        'memory' => 1024,
        'time' => 2,
        'threads' => 2,
    ]);

> {tip} Para obtener más información sobre estas opciones, consulte la [documentación oficial de PHP](http://php.net/manual/en/function.password-hash.php).
> > > {tip} For more information on these options, check out the [official PHP documentation](http://php.net/manual/en/function.password-hash.php).

#### Verificación de una contraseña contra un hash : Verifying A Password Against A Hash

El método `check` le permite verificar que una cadena dada de texto llano corresponde a un hash dado. Sin embargo, si está utilizando `LoginController` [incluido con Laravel](/docs/{{version}}/authentication), probablemente no necesite usar esto directamente, ya que este controlador llama automáticamente a este método:
> > The `check` method allows you to verify that a given plain-text string corresponds to a given hash. However, if you are using the `LoginController` [included with Laravel](/docs/{{version}}/authentication), you will probably not need to use this directly, as this controller automatically calls this method:

    if (Hash::check('plain-text', $hashedPassword)) {
        // The passwords match...
    }

#### Comprobando si una contraseña necesita ser actualizada : Checking If A Password Needs To Be Rehashed

La función `needsRehash` le permite determinar si el factor de trabajo utilizado por hasher ha cambiado desde que la contraseña fue hash:
> > The `needsRehash` function allows you to determine if the work factor used by the hasher has changed since the password was hashed:

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
