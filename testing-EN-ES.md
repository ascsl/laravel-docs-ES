# Pruebas: Primeros pasos : Testing: Getting Started

- [Introduction](#introduction)
- [Environment](#environment)
- [Creating & Running Tests](#creating-and-running-tests)

<a name="introduction"></a>
## Introducción : Introduction

Laravel está construido con pruebas en mente. De hecho, el soporte para probar con PHPUnit se incluye de fábrica y ya está configurado un archivo `phpunit.xml` para su aplicación. El framework también incluye métodos convenientes de ayuda que le permiten probar expresivamente sus aplicaciones.
> > Laravel is built with testing in mind. In fact, support for testing with PHPUnit is included out of the box and a `phpunit.xml` file is already set up for your application. The framework also ships with convenient helper methods that allow you to expressively test your applications.

Por defecto, el directorio `tests` de su aplicación contiene dos directorios: `Feature` y `Unit`. Las pruebas unitarias son pruebas que se enfocan en una porción muy pequeña y aislada de su código. De hecho, la mayoría de las pruebas unitarias probablemente se centren en un solo método. Las pruebas de características pueden probar una porción más grande de su código, incluyendo cómo varios objetos interactúan entre sí o incluso una solicitud HTTP completa a un punto final JSON.
> > By default, your application's `tests` directory contains two directories: `Feature` and `Unit`. Unit tests are tests that focus on a very small, isolated portion of your code. In fact, most unit tests probably focus on a single method. Feature tests may test a larger portion of your code, including how several objects interact with each other or even a full HTTP request to a JSON endpoint.

Se proporciona un archivo `ExampleTest.php` en los directorios de prueba `Feature` y `Unit`. Después de instalar una nueva aplicación Laravel, ejecute `phpunit` en la línea de comandos para ejecutar sus pruebas.
> > An `ExampleTest.php` file is provided in both the `Feature` and `Unit` test directories. After installing a new Laravel application, run `phpunit` on the command line to run your tests.

<a name="environment"></a>
## Entorno : Environment

Al ejecutar pruebas a través de `phpunit`, Laravel configurará automáticamente el entorno de configuración en `testing` debido a las variables de entorno definidas en el archivo `phpunit.xml`. Laravel también configura automáticamente la sesión y la memoria caché en el controlador `array` durante la prueba, lo que significa que no se conservarán datos de sesión o de caché durante la prueba.
> > When running tests via `phpunit`, Laravel will automatically set the configuration environment to `testing` because of the environment variables defined in the `phpunit.xml` file. Laravel also automatically configures the session and cache to the `array` driver while testing, meaning no session or cache data will be persisted while testing.

Puede definir otros valores de configuración del entorno de prueba según sea necesario. Las variables de entorno `testing` se pueden configurar en el archivo `phpunit.xml`, pero asegúrese de borrar su caché de configuración con el comando Artisan `config:clear` antes de ejecutar sus pruebas.
> > You are free to define other testing environment configuration values as necessary. The `testing` environment variables may be configured in the `phpunit.xml` file, but make sure to clear your configuration cache using the `config:clear` Artisan command before running your tests!

Además, puede crear un archivo `.env.testing` en la raíz de su proyecto. Este archivo anulará el archivo `.env` cuando ejecute las pruebas de PHPUnit o ejecute los comandos de Artisan con la opción `--env=testing`.
> > In addition, you may create a `.env.testing` file in the root of your project. This file will override the `.env` file when running PHPUnit tests or executing Artisan commands with the `--env=testing` option.

<a name="creating-and-running-tests"></a>
## Creaando y ejecutando pruebas : Creating & Running Tests

Para crear un nuevo caso de prueba, use el comando Artisan `make:test`:
> > To create a new test case, use the `make:test` Artisan command:

    // Create a test in the Feature directory...
    php artisan make:test UserTest

    // Create a test in the Unit directory...
    php artisan make:test UserTest --unit

Una vez que se haya generado la prueba, puede definir los métodos de prueba como lo haría normalmente usando PHPUnit. Para ejecutar tus pruebas, ejecuta el comando `phpunit` desde tu terminal:
> > Once the test has been generated, you may define test methods as you normally would using PHPUnit. To run your tests, execute the `phpunit` command from your terminal:

    <?php

    namespace Tests\Unit;

    use Tests\TestCase;
    use Illuminate\Foundation\Testing\RefreshDatabase;

    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testBasicTest()
        {
            $this->assertTrue(true);
        }
    }

> {note} Si defines tu propio método `setUp` dentro de una clase de prueba, asegúrate de llamar a `parent::setUp()`.
> > > {note} If you define your own `setUp` method within a test class, be sure to call `parent::setUp()`.
