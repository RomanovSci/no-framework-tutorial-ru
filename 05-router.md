[<< HTTP](04-http.md) | [Контроллер >>](06-controller.md)

### Маршрутизатор

Маршрутизатор определяет и вызывает обработчик запроса в зависимости от заданных нами правил. С нашими текущими настройками не важно по какому URL перешел пользователь, приложение всегда будет возвращать один и тот же ответ. Давайте исправим эту  ситуацию.

В этом руководстве я буду использовать [FastRoute](https://github.com/nikic/FastRoute). Но вы можете выбрать любою другую библиотеку. Вот несколько альтернативных вариантов: [symfony/Routing](https://github.com/symfony/Routing), [Aura.Router](https://github.com/auraphp/Aura.Router), [fuelphp/routing](https://github.com/fuelphp/routing), [Klein](https://github.com/chriso/klein.php).

Вы уже знаете, как устанавливать сторонние пакеты с помощью Composer, поэтому я опущу подробное описание данного шага. Теперь добавьте данный блок кода в `Bootstrap.php`, вместо сообщения 'hello world' которое мы добавляли в предыдущем разделе.

```php
$dispatcher = \FastRoute\simpleDispatcher(function (\FastRoute\RouteCollector $r) {
    $r->addRoute('GET', '/hello-world', function () {
        echo 'Hello World';
    });
    $r->addRoute('GET', '/another-route', function () {
        echo 'This works too';
    });
});

$routeInfo = $dispatcher->dispatch($request->getMethod(), $request->getPath());
switch ($routeInfo[0]) {
    case \FastRoute\Dispatcher::NOT_FOUND:
        $response->setContent('404 - Page not found');
        $response->setStatusCode(404);
        break;
    case \FastRoute\Dispatcher::METHOD_NOT_ALLOWED:
        $response->setContent('405 - Method not allowed');
        $response->setStatusCode(405);
        break;
    case \FastRoute\Dispatcher::FOUND:
        $handler = $routeInfo[1];
        $vars = $routeInfo[2];
        call_user_func($handler, $vars);
        break;
}
```
Сначала мы регистрируем доступные маршруты для нашего приложения. После чего диспетчер получает информацию о маршруте и выполняется определенный блок кода в зависимости от полученой информации. Такая настройка пригодная для небольших приложений. Если ваше приложение начнет разрастаться, файл загрузки может быть загроможден маршрутами. Давайте вынесем их в отдельный файл. Создайте файл `Routes.php` в директории `src/` и вставьте в него следующий код:

```php
<?php declare(strict_types = 1);

return [
    ['GET', '/hello-world', function () {
        echo 'Hello World';
    }],
    ['GET', '/another-route', function () {
        echo 'This works too';
    }],
];
```

Теперь давайте перепишем блок создания маршрутов:

```php
$routeDefinitionCallback = function (\FastRoute\RouteCollector $r) {
    $routes = include('Routes.php');
    foreach ($routes as $route) {
        $r->addRoute($route[0], $route[1], $route[2]);
    }
};

$dispatcher = \FastRoute\simpleDispatcher($routeDefinitionCallback);
```
Так выглядит получше, но теперь обработчики запросов находятся в файле `Routes.php`. Давайте исправим этот момент в следующем разделе. 

[<< HTTP](04-http.md) | [Контроллер >>](06-controller.md)
