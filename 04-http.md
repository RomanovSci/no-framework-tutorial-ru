[<< Обработчик ошибок](03-error-handler.md) | [Маршрутизатор >>](05-router.md)

### HTTP

Всем известно что PHP предоставляет некоторые средства для работы с HTTP. К примеру [суперглобальные переменные](http://php.net/manual/ru/language.variables.superglobals.php) содержат информацию запроса. Они хороши, если вы разрабатываете небольшой скриптик, который не будет требовать дальнейшей поддержки. Однако, если мы хотим написать чистый, поддерживаемый [SOLID](http://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29) код, тогда нам понадобится класс с хорошим объектно-ориентированным интерфейсом.

И опять же, вам не нужно изобретать велосипед, можно просто установить сторонний модуль для работы с HTTP. Я решил написать свой собственный [HTTP компонент](https://github.com/PatrickLouys/http), так как меня не сильно устраивают существующие аналоги, но вам не обязательно делать то же что и я :)

Как альтернативу можно выбрать один из пакетов: [Symfony HttpFoundation](https://github.com/symfony/HttpFoundation), [Nette HTTP Component](https://github.com/nette/http), [Aura Web](https://github.com/auraphp/Aura.Web), [sabre/http](https://github.com/fruux/sabre-http). В данном руководстве я буду использовать компонент написаный мною, но вы можете использовать любой другой который вам по душе.

Давайте добавим нужный нам пакет в `composer.json` и запустим команду `composer update`:

```json
  "require": {
    "php": ">=7.0.0",
    "filp/whoops": "~2.1",
    "patricklouys/http": "~1.4"
  },
```

После регистрации обработчика ошибок в `Bootstrap.php`, создадим объекты запроса и ответа (и не забудьте удалить исключение, которое мы добавляли в предыдущем разделе).

```php
$request = new \Http\HttpRequest($_GET, $_POST, $_COOKIE, $_FILES, $_SERVER);
$response = new \Http\HttpResponse;
```
Объекты `Request` и `Response` служат для обработки запросов и, как вы поняли, отправки ответа клиенту. Чтобы отправить что-либо обратно в браузер добавим в `Bootstrap.php` следующий код:

```php
foreach ($response->getHeaders() as $header) {
    header($header, false);
}

echo $response->getContent();
```

Второй параметр метода `header()` устанавливаем false чтобы избежать перезаписи существующих заголовков ответа. Сейчас наш код отправляет пустой ответ браузеру со статусом `200`. Давайте изменим это, добавив в `Bootstrap.php` следующий код  (выше инструкции `foreach`):

```php
$content = '<h1>Hello World</h1>';
$response->setContent($content);
```

Если мы хотим вернуть 404:

```php
$response->setContent('404 - Page not found');
$response->setStatusCode(404);
```
Помните что объект `Response` только хранит данные, и если вы установите код ответа множество раз перед отправкой клиенту, приложение вернет тот, который был установлен последним.

[<< Обработчик ошибок](03-error-handler.md) | [Маршрутизатор >>](05-router.md)
