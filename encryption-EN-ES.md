# Encriptación : Encryption

- [Introduction](#introduction)
- [Configuration](#configuration)
- [Using The Encrypter](#using-the-encrypter)

<a name="introduction"></a>
## Introducción : Introduction

El encriptador de Laravel usa OpenSSL para proporcionar el cifrado AES-256 y AES-128. Le recomendamos encarecidamente que utilice las funciones de cifrado integradas de Laravel y que no intente desplegar sus propios algoritmos de cifrado de "cosecha propia". Todos los valores encriptados de Laravel se firman usando un código de autenticación de mensaje (MAC) para que su valor subyacente no se pueda modificar una vez encriptado.
> > Laravel's encrypter uses OpenSSL to provide AES-256 and AES-128 encryption. You are strongly encouraged to use Laravel's built-in encryption facilities and not attempt to roll your own "home grown" encryption algorithms. All of Laravel's encrypted values are signed using a message authentication code (MAC) so that their underlying value can not be modified once encrypted.

<a name="configuration"></a>
## Configuración : Configuration

Antes de usar el encriptador de Laravel, debe establecer una opción `key` en su archivo de configuración `config/app.php`. Debe usar el comando `php artisan key:generate` para generar esta clave, ya que este comando Artisan utilizará el generador seguro de bytes aleatorios de PHP para construir su clave. Si este valor no está configurado correctamente, todos los valores cifrados por Laravel serán inseguros.
> > Before using Laravel's encrypter, you must set a `key` option in your `config/app.php` configuration file. You should use the `php artisan key:generate` command to generate this key since this Artisan command will use PHP's secure random bytes generator to build your key. If this value is not properly set, all values encrypted by Laravel will be insecure.

<a name="using-the-encrypter"></a>
## Usando The Encrypter : Using The Encrypter

#### Encriptación de un valor : Encrypting A Value

Puedes encriptar un valor usando el helper `encrypt`. Todos los valores cifrados se cifran utilizando OpenSSL y el cifrado `AES-256-CBC`. Además, todos los valores cifrados se firman con un código de autenticación de mensaje (MAC) para detectar cualquier modificación en la cadena cifrada:
> > You may encrypt a value using the `encrypt` helper. All encrypted values are encrypted using OpenSSL and the `AES-256-CBC` cipher. Furthermore, all encrypted values are signed with a message authentication code (MAC) to detect any modifications to the encrypted string:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Store a secret message for the user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $user->fill([
                'secret' => encrypt($request->secret)
            ])->save();
        }
    }

#### Cifrado sin serialización : Encrypting Without Serialization

Los valores cifrados se pasan a través de `serialize` durante el cifrado, lo que permite el cifrado de objetos y arrays. Por lo tanto, los clientes que no sean PHP que reciban valores encriptados necesitarán 'deserializar' los datos. Si desea cifrar y descifrar valores sin serialización, puede usar los métodos `encryptString` y` decryptString` de la fachada `Crypt`:
> > Encrypted values are passed through `serialize` during encryption, which allows for encryption of objects and arrays. Thus, non-PHP clients receiving encrypted values will need to `unserialize` the data. If you would like to encrypt and decrypt values without serialization, you may use the `encryptString` and `decryptString` methods of the `Crypt` facade:

    use Illuminate\Support\Facades\Crypt;

    $encrypted = Crypt::encryptString('Hello world.');

    $decrypted = Crypt::decryptString($encrypted);

#### Descifrar un valor : Decrypting A Value

Puedes descifrar valores usando el helper `decrypt`. Si el valor no se puede descifrar correctamente, como cuando el MAC no es válido, se lanzará un `Illuminate\Contracts\Encryption\DecryptException`:
> > You may decrypt values using the `decrypt` helper. If the value can not be properly decrypted, such as when the MAC is invalid, an `Illuminate\Contracts\Encryption\DecryptException` will be thrown:

    use Illuminate\Contracts\Encryption\DecryptException;

    try {
        $decrypted = decrypt($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
