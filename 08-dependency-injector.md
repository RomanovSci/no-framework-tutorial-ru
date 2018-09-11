[<< Инверсия управления](07-inversion-of-control.md) | [Шаблонизатор >>](09-templating.md)

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

Мы шарим HTTP объекты, так как нету смысла добавлять контент к одному объекту и потом возвращать этот же контент к другому. С помощью шаринга мы работаем с одним и тем же объектом на протяжении всего сеанса.

Алиасы позволяют нам работать с интерфейсами вместо классов. Теперь все классы которые требуют объект имплементирующий интерфейс `Http\Response` будут получать на вход объект типа `Http\HttpResponse` и если вы захотите изменить конкретную реализацию объекта респонса, вы смомоежете сделать это в одном месте, вместо того чтобы изменять код в каждом классе.

Конечно же нам нужно подправить `Bootstrap.php`. Теперь вместо того, чтобы создавать объекты `$request` и `$response` с помощью оператора `new`, за нас это будет делать инъектор. Теперь мы уверен в том, что каждый класс будет использовать один и тот же экземпляр класса запроса и ответа:

```php
$injector = include('Dependencies.php');

$request = $injector->make('Http\HttpRequest');
$response = $injector->make('Http\HttpResponse');
```

Также изменим часть создания класса контроллера с:

```php
$class = new $className($response);
$class->$method($vars);
```
На:

```php
$class = $injector->make($className);
$class->$method($vars);
```

Теперь все зависимости контроллеров будут резолвится автоматически с помощью пакета Auryn.

Давайте немного порефакторим `Homepage`:

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
Теперь, как вы заметили, класс контроллера имеет две зависимости. Давайте перейдем по ссылке и посмотрим что у нас получилось: `http://localhost:8000/?name=Arthur%20Dent`.

Поздравляю, мы полностью заложили фундамент для нашего приложения.

[<< Инверсия управления](07-inversion-of-control.md) | [Шаблонизатор >>](09-templating.md)
