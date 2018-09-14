# Pruebas de consola : Console Tests

- [Introduction](#introduction)
- [Expecting Input / Output](#expecting-input-and-output)

## Introducción : Introduction

Además de simplificar las pruebas HTTP, Laravel proporciona una API simple para probar aplicaciones de consola que solicitan la entrada del usuario.
> > In addition to simplifying HTTP testing, Laravel provides a simple API for testing console applications that ask for user input.

<a name="expecting-input-and-output"></a>
## Esperando entrada / salida : Expecting Input / Output

Laravel le permite "simular" fácilmente la entrada del usuario para los comandos de su consola utilizando el método `expectsQuestion`. Además, puede especificar el código de salida y el texto que espera que el comando de la consola genere utilizando los métodos `assertExitCode` y `expectsOutput`. Por ejemplo, considere el siguiente comando de consola:
> > Laravel allows you to easily "mock" user input for your console commands using the `expectsQuestion` method. In addition, you may specify the exit code and text that you expect to be output by the console command using the `assertExitCode` and `expectsOutput` methods. For example, consider the following console command:

    Artisan::command('question', function () {
        $name = $this->ask('What is your name?');

        $language = $this->choice('Which language do you program in?', [
            'PHP',
            'Ruby',
            'Python',
        ]);

        $this->line('Your name is '.$name.' and you program in '.$language.'.');
    });

Puede probar este comando con la siguiente prueba que utiliza los métodos `expectsQuestion`, `expectsOutput` y `assertExitCode`:
> > You may test this command with the following test which utilizes the `expectsQuestion`, `expectsOutput`, and `assertExitCode` methods:

    /**
     * Test a console command.
     *
     * @return void
     */
    public function test_console_command()
    {
        $this->artisan('question')
             ->expectsQuestion('What is your name?', 'Taylor Otwell')
             ->expectsQuestion('Which language do you program in?', 'PHP')
             ->expectsOutput('Your name is Taylor Otwell and you program in PHP.')
             ->assertExitCode(0);
    }

