# Guía de contribución : Contribution Guide

- [Bug Reports](#bug-reports)
- [Core Development Discussion](#core-development-discussion)
- [Which Branch?](#which-branch)
- [Security Vulnerabilities](#security-vulnerabilities)
- [Coding Style](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)

<a name="bug-reports"></a>
## Informes de errores : Bug Reports

Para alentar la colaboración activa, Laravel recomienda encarecidamente las solicitudes de extracción, no solo los informes de errores. Los "informes de errores" también pueden enviarse en forma de una solicitud de extracción que contiene una prueba fallida.
> > To encourage active collaboration, Laravel strongly encourages pull requests, not just bug reports. "Bug reports" may also be sent in the form of a pull request containing a failing test.

Sin embargo, si presenta un informe de error, su problema debe contener un título y una descripción clara del problema. También debe incluir tanta información relevante como sea posible y una muestra de código que demuestre el problema. El objetivo de un informe de errores es facilitarle a usted, y a los demás, replicar el error y desarrollar una solución.
> > However, if you file a bug report, your issue should contain a title and a clear description of the issue. You should also include as much relevant information as possible and a code sample that demonstrates the issue. The goal of a bug report is to make it easy for yourself - and others - to replicate the bug and develop a fix.

Recuerde, los informes de errores se crean con la esperanza de que otros con el mismo problema puedan colaborar con usted para resolverlo. No espere que el informe de error vea automáticamente ninguna actividad o que otros salten para solucionarlo. Crear un informe de errores sirve para ayudarse a usted y a otros a comenzar el camino de solucionar el problema.
> > Remember, bug reports are created in the hope that others with the same problem will be able to collaborate with you on solving it. Do not expect that the bug report will automatically see any activity or that others will jump to fix it. Creating a bug report serves to help yourself and others start on the path of fixing the problem.

El código fuente de Laravel se gestiona en GitHub, y hay repositorios para cada uno de los proyectos de Laravel:
> > The Laravel source code is managed on GitHub, and there are repositories for each of the Laravel projects:

<div class="content-list" markdown="1">
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Cashier for Braintree](https://github.com/laravel/cashier-braintree)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Horizon](https://github.com/laravel/horizon)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Website](https://github.com/laravel/laravel.com)
</div>

<a name="core-development-discussion"></a>
## Discusión sobre el desarrollo del núcleo : Core Development Discussion

Puede proponer nuevas características o mejoras del comportamiento existente de Laravel en la publicación Laravel Ideas [issue board] (https://github.com/laravel/ideas/issues). Si propone una nueva característica, esté dispuesto a implementar al menos parte del código que se necesitaría para completar la función.
> > You may propose new features or improvements of existing Laravel behavior in the Laravel Ideas [issue board](https://github.com/laravel/ideas/issues). If you propose a new feature, please be willing to implement at least some of the code that would be needed to complete the feature.

La discusión informal con respecto a los errores, las nuevas funciones y la implementación de las funciones existentes se lleva a cabo en el canal `#internals` del equipo de LaraChat (https://larachat.co) Slack. Taylor Otwell, el mantenedor de Laravel, suele estar presente en el canal los días de semana de 8 a. M. A 5 p. M. (UTC-06: 00 o América / Chicago) y esporádicamente presente en el canal en otros momentos.
> > Informal discussion regarding bugs, new features, and implementation of existing features takes place in the `#internals` channel of the [LaraChat](https://larachat.co) Slack team. Taylor Otwell, the maintainer of Laravel, is typically present in the channel on weekdays from 8am-5pm (UTC-06:00 or America/Chicago), and sporadically present in the channel at other times.

<a name="which-branch"></a>
## ¿Qué rama? : Which Branch?

**Todas** las correcciones de errores deben enviarse a la última rama estable o a la rama LTS actual (5.6). Las correcciones de errores **nunca** deben enviarse a la rama `master` a menos que arreglen las características que existen solo en la próxima versión.
> > **All** bug fixes should be sent to the latest stable branch or to the current LTS branch (5.6). Bug fixes should **never** be sent to the `master` branch unless they fix features that exist only in the upcoming release.

Las características **Menores** que son **totalmente compatibles con versiones anteriores** con la versión actual de Laravel pueden enviarse a la última rama estable.
> > **Minor** features that are **fully backwards compatible** with the current Laravel release may be sent to the latest stable branch.

Las características nuevas **Major** siempre deben enviarse a la rama `master`, que contiene la próxima versión de Laravel.
> > **Major** new features should always be sent to the `master` branch, which contains the upcoming Laravel release.

Si no está seguro de si su característica califica como mayor o menor, por favor pregunte a Taylor Otwell en el canal `#internals` del equipo [LaraChat](https://larachat.co) Slack.
> > If you are unsure if your feature qualifies as a major or minor, please ask Taylor Otwell in the `#internals` channel of the [LaraChat](https://larachat.co) Slack team.

<a name="security-vulnerabilities"></a>
## Vulnerabilidades de seguridad : Security Vulnerabilities

Si descubre una vulnerabilidad de seguridad dentro de Laravel, envíe un correo electrónico a Taylor Otwell a <a href="mailto:taylor@laravel.com"> taylor@laravel.com </a>. Todas las vulnerabilidades de seguridad serán atendidas rápidamente.
> > If you discover a security vulnerability within Laravel, please send an email to Taylor Otwell at <a href="mailto:taylor@laravel.com">taylor@laravel.com</a>. All security vulnerabilities will be promptly addressed.

<a name="coding-style"></a>
## Estilo de codificación : Coding Style

Laravel sigue el estándar de codificación [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) y el [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md) estándar de carga automática.
> > Laravel follows the [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) coding standard and the [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md) autoloading standard.

<a name="phpdoc"></a>
### PHPDoc

A continuación se muestra un ejemplo de un bloque de documentación válida de Laravel. Tenga en cuenta que al atributo `@param` le siguen dos espacios, el tipo de argumento, dos espacios más y, finalmente, el nombre de la variable:
> > Below is an example of a valid Laravel documentation block. Note that the `@param` attribute is followed by two spaces, the argument type, two more spaces, and finally the variable name:

    /**
     * Register a binding with the container.
     *
     * @param  string|array  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        //
    }

<a name="styleci"></a>
### StyleCI

¡No se preocupe si el estilo de su código no es perfecto! [StyleCI](https://styleci.io/) fusionará automáticamente cualquier corrección de estilo en el repositorio de Laravel una vez que se hayan fusionado las solicitudes de extracción. Esto nos permite enfocarnos en el contenido de la contribución y no en el estilo del código.
> > Don't worry if your code styling isn't perfect! [StyleCI](https://styleci.io/) will automatically merge any style fixes into the Laravel repository after pull requests are merged. This allows us to focus on the content of the contribution and not the code style.
