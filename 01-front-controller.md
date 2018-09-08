[>>](02-autoload.md)

### Контроллер запросов

[Контроллер запросов](https://ru.wikipedia.org/wiki/%D0%95%D0%B4%D0%B8%D0%BD%D0%B0%D1%8F_%D1%82%D0%BE%D1%87%D0%BA%D0%B0_%D0%B2%D1%85%D0%BE%D0%B4%D0%B0_(%D1%88%D0%B0%D0%B1%D0%BB%D0%BE%D0%BD_%D0%BF%D1%80%D0%BE%D0%B5%D0%BA%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F)) это единая точка входа в ваше приложение.

Создадим пустую дерикторию в папке вашего проекта. Также нам нужна точка входа, которая будет принимать все запросы. Это означает что нам нужно создать файл, который мы назовем`index.php`.

Некоторые разработчики создают файл `index.php` в корневой папке проекта. Данный подход также используется некоторых фреймворках. Давайте разберемся почему так делать не нужно.

Так как `index.php` является точкой входа в наше веб приложение, то данный файл должен находиться в каталоге веб сервера. Веб сервер имеет доступ ко всем подкаталогам которые расположены в нем, в частности и к файлам в которых сохранен исходный код приложения. Можно ограничить доступ к подкаталогам, путем правильной настройки веб сервера.  

Но иногда не все идет согласно нашему плану. И в случае неправильной настройки доступа к поддерикториям приложения, нашему пользователю может быть доступен исходный код. Думаю вы сами понимаете чем это может быть чревато.

Так что вместо того чтобы создавать `index.php` в корне проекта, давайте создадим дерикторию `public`. Также создадим дерикторию `src` в корневой папке проекта.

Теперь мы можем создать `index.php` внутри дериктории `public`. И помните, что мы здесь ничего не хотим раскрывать, поэтому вставляем данный код в файл `index.php`:

```php
<?php declare(strict_types = 1); 

require __DIR__ . '/../src/Bootstrap.php';
```

`__DIR__` э [волшебная константа](http://php.net/manual/en/language.constants.predefined.php) которая содержит путь к текущей дериктории. By using it, you can make sure that the `require` always uses the same relative path to the file it is used in. Otherwise, if you call the `index.php` from a different folder it will not find the file.

`declare(strict_types = 1);` sets the current file to [strict typing](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.strict). In this tutorial we are going to use this for all PHP files. This means that you can't just pass an integer as a parameter to a method that requires a string. If you don't use strict mode, it would be automatically casted to the required type. With strict mode, it will throw an Exception if it is the wrong type.

The `Bootstrap.php` will be the file that wires your application together. We will get to it shortly.

The rest of the public folder is reserved for your public asset files (like JavaScript files and stylesheets).

Now navigate inside your `src` folder and create a new `Bootstrap.php` file with the following content:

```php
<?php declare(strict_types = 1);

echo 'Hello World!';
```

Now let's see if everything is set up correctly. Open up a console and navigate into your projects `public` folder. In there type `php -S localhost:8000` and press enter. This will start the built-in webserver and you can access your page in a browser with `http://localhost:8000`. You should now see the 'hello world' message.

If there is an error, go back and try to fix it. If you only see a blank page, check the console window where the server is running for errors.

Now would be a good time to commit your progress. If you are not already using Git, set up a repository now. This is not a Git tutorial so I won't go over the details. But using version control should be a habit, even if it is just for a tutorial project like this.

Some editors and IDE's put their own files into your project folders. If that is the case, create a `.gitignore` file in your project root and exclude the files/directories. Below is an example for PHPStorm:

```
.idea/
```

[next >>](02-composer.md)
