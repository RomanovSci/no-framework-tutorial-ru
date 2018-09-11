[<< previous](07-inversion-of-control.md) | [next >>](09-templating.md)

### Инъектор зависимостей

Инъектор зависимостей определяет зависимости класса и гарантирует их корректное внедрение в момент создания нового экземпляра.

Это единственный инъектор который я мог бы порекомендовать: [Auryn](https://github.com/rdlowrey/Auryn). К сожалению все альтернативы, о которых мне известно, используют [антипатерн локатор служб](http://blog.ploeh.dk/2010/02/03/ServiceLocatorisanAnti-Pattern/) в своей документации и примерах.

Давайте установим пакет `Auryn`, в директории `src/` создадим файл с названием `Dependencies.php` и добавим в него следующий код:

```php
<?php declare(strict_types = 1);

$injector = new \Auryn\Injector;

$injector->alias('Http\Request', 'Http\HttpRequest');
$injector->share('Http\HttpRequest');
$injector->define('Http\HttpRequest', [
    ':get' => $_GET,
    ':post' => $_POST,
    ':cookies' => $_COOKIE,
    ':files' => $_FILES,
    ':server' => $_SERVER,
]);

$injector->alias('Http\Response', 'Http\HttpResponse');
$injector->share('Http\HttpResponse');

return $injector;
```

Перед тем как продолжить, вы должны понимать что такое `alias`, `share` и `define`. Об этом можно узнать на [странице документации пакета](https://github.com/rdlowrey/Auryn).

You are sharing the HTTP objects because there would not be much point in adding content to one object and then returning another one. So by sharing it you always get the same instance.

The alias allows you to type hint the interface instead of the class name. This makes it easy to switch the implementation without having to go back and edit all your classes that use the old implementation.

Of course your `Bootstrap.php` will also need to be changed. Before you were setting up `$request` and `$response` with `new` calls. Switch that to the injector now so that we are using the same instance of those objects everywhere.

```php
$injector = include('Dependencies.php');

$request = $injector->make('Http\HttpRequest');
$response = $injector->make('Http\HttpResponse');
```

The other part that has to be changed is the dispatching of the route. Before you had the following code:

```php
$class = new $className($response);
$class->$method($vars);
```

Change that to the following:

```php
$class = $injector->make($className);
$class->$method($vars);
```

Now all your controller constructor dependencies will be automatically resolved with Auryn.

Go back to your `Homepage` controller and change it to the following:

```php
<?php declare(strict_types = 1);

namespace Example\Controllers;

use Http\Request;
use Http\Response;

class Homepage
{
    private $request;
    private $response;

    public function __construct(Request $request, Response $response)
    {
        $this->request = $request;
        $this->response = $response;
    }

    public function show()
    {
        $content = '<h1>Hello World</h1>';
        $content .= 'Hello ' . $this->request->getParameter('name', 'stranger');
        $this->response->setContent($content);
    }
}
```

As you can see now the class has two dependencies. Try to access the page with a GET parameter like this `http://localhost:8000/?name=Arthur%20Dent`.

Congratulations, you have now successfully laid the groundwork for your application. 

[<< previous](07-inversion-of-control.md) | [next >>](09-templating.md)
