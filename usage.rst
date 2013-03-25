Использование
=============

Эта глава описывает как использовать.

Установка
---------

Для быстрого начала `скачайте`_ архив Silex и распакуйте его. В результате вы получите слудующую структуру каталогов:

.. code-block:: text

    ├── composer.json
    ├── composer.lock
    ├── vendor
    │   └── ...
    └── web
        └── index.php

Если вы хотите большей гибкости, используйте Composer. Создайте ``composer.json``:

.. code-block:: json

    {
        "require": {
            "silex/silex": "1.0.*@dev"
        }
    }

И запустите Composer для установки Silex и всех зависимостей:

.. code-block:: bash

    $ curl -s http://getcomposer.org/installer | php
    $ php composer.phar install

.. tip::

    По умолчанию, Silex использует стабильные компоненты Symfony. Если вместо этого вы хотите использовать актуальные версии, добавьте ``"minimum-stability": "dev"`` в ваш файл ``composer.json``.

Обновление
----------

Обновление Silex до последней версии легко выполнить, запустив команду ``update``::

    $ php composer.phar update

Загрузка
--------

Для загрузки Silex, всё что вам требуется сделать - это включить файл ``vendor/autoload.php`` и создать экземпляр ``Silex\Application``. После определений ваших контроллеров, вызовете метод ``run`` вашего приложения::

    // web/index.php

    require_once __DIR__.'/../vendor/autoload.php';

    $app = new Silex\Application();

    // definitions

    $app->run();

Затем сконфигурируйте веб-сервер (подробности см. в :doc:`отдельной главе <web_servers>`).

.. tip::

    При разработке веб-сайта, вам может потребоваться включить отладочный режим::

        $app['debug'] = true;

.. tip::

    Если ваше приложение располагается за прокси-сервером, и вы хотите чтобы Silex отслеживал заголовки ``X-Forwarded-For*``, то вым надо запустить ваше приложение следующим образом::

        use Symfony\Component\HttpFoundation\Request;

        Request::trustProxyData();
        $app->run();

Маршрутизация
-------------

В Silex вы определяете маршрут и контроллер, который вызывается при соответствии запроса маршруту.

Шаблон маршрута состоит из:

* *Шаблон*: Шаблон маршрута определяет путь, указывающий на ресурс. Шаблон может включать переменные части и вы можете задать RegExp-требования для них.

* *Метод*: Один из следующих HTTP-методов: ``GET``, ``POST``, ``PUT`` или ``DELETE``. Описывает взаимодействие с ресурсом. Обчыно используются только ``GET`` и ``POST``, однако вы можете использовать и остальные.

Контроллер определяется с помощью замыкания следующего вида::

    function () {
        // что-нибудь делаем
    }

Замыкания - это анонимные функции, которые могут импортировать состояние из-за пределов собственного определения. Например, вы можете определить замыкание в функции и импортировать локальные переменные этой функции.

.. note::

    Замыкания, которые ничего не импортируют, называют лямбдами. Так как в PHP все анонимные функции являются экземплярами класса ``Closure``, мы не будем делать между ними разлиций.

Возвращаемое значение замыкания становится контентом страницы.

Пример GET-маршрута
~~~~~~~~~~~~~~~~~~~

Ниже приведён пример определения ``GET``-маршрута::

    $blogPosts = array(
        1 => array(
            'date'      => '2011-03-29',
            'author'    => 'igorw',
            'title'     => 'Using Silex',
            'body'      => '...',
        ),
    );

    $app->get('/blog', function () use ($blogPosts) {
        $output = '';
        foreach ($blogPosts as $post) {
            $output .= $post['title'];
            $output .= '<br />';
        }

        return $output;
    });

При посещении ``/blog`` будет возвращён список заголовков блога. Выражение ``use``
сообщает о необходимости использования чего-либо не из текущего контекста. Оно импортирует в замыкание переменную $blogPosts из внешней области видимости. Это позволяет вам использовать переменную в замыкании.

Динамическая маршрутизация
~~~~~~~~~~~~~~~~~~~~~~~~~~

Теперь, вы можете создать другой контроллер для просмотра отдельных записей в блоге::

    $app->get('/blog/{id}', function (Silex\Application $app, $id) use ($blogPosts) {
        if (!isset($blogPosts[$id])) {
            $app->abort(404, "Post $id does not exist.");
        }

        $post = $blogPosts[$id];

        return  "<h1>{$post['title']}</h1>".
                "<p>{$post['body']}</p>";
    });

Это определение маршрута имеет переменную часть ``{id}``, которая передаётся в замыкание.

Текушей экземпляр ``Application`` автоматически инжектируется Silex в Замыкание благодаря подсказке по типу.

Когда запись не существует, мы используем метод ``abort()`` для остановки запроса в самом начале. В действительности генерируется исключение, обработку которых мы обсудим позже.

Пример POST-маршрута
~~~~~~~~~~~~~~~~~~~~

POST-маршруты означают создание ресурсов. Примером тому является форма обратной связи. Мы будем использовать функцию ``mail`` для отправки электронной почты::

    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;

    $app->post('/feedback', function (Request $request) {
        $message = $request->get('message');
        mail('feedback@yoursite.com', '[YourSite] Feedback', $message);

        return new Response('Thank you for your feedback!', 201);
    });

Это довольно просто.

.. note::

    В базовый комплект включён :doc:`SwiftmailerServiceProvider <providers/swiftmailer>`, который вы можете использовать вместо ``mail()``.
 н6:нн
Текущий запрос ``request`` автоматически инжектируется Silex в в Замыкание благодаря подсказке по типу. Он представляет собой экземпляр `Request
<http://api.symfony.com/master/Symfony/Component/HttpFoundation/Request.html>`_, поэтому вы пожете получить переменные через метод ``get``.

Вместо возврата строки, мы возвращаем экземпляр `Response
<http://api.symfony.com/master/Symfony/Component/HttpFoundation/Response.html>`_. Тем самым мы можем установить код состояния HTTP, в нашем случае ``201 Created``.

.. note::

    Silex всегда внутри использует ``Response``, строки конвертируются в ответы с кодом состояния ``200 Ok``.

Другие методы
~~~~~~~~~~~~~

Вы можете создавать контроллеры для большинства других методов HTTP. Просто вызовите один из этих методов в своём приложении: ``get``, ``post``, ``put`` или ``delete``::

    $app->put('/blog/{id}', function ($id) {
        ...
    });

    $app->delete('/blog/{id}', function ($id) {
        ...
    });

.. tip::

    Формы в большинстве браузеров напрямую не поддерживают использование других HTTP-методов. Для того, чтобы использовать методы отличные от GET и POST, вы можете вставить в форму специальное поле с именем ``_method``. При использовании этого поля атрибут ``method`` формы должен содержать значение POST::

        <form action="/my/target/route/" method="post">
            ...
            <input type="hidden" id="_method" name="_method" value="PUT" />
        </form>

    Если вы используете компоненты Symfony 2.2+, вам необходивно явно включить переопределение метода::

        use Symfony\Component\HttpFoundation\Request;

        Request::enableHttpMethodParameterOverride();
        $app->run();

Вы также можете вызвать ``match``, что будет соответствовать всем методам. Ограничение можно наложить методом ``method``::

    $app->match('/blog', function () {
        ...
    });

    $app->match('/blog', function () {
        ...
    })
    ->method('PATCH');

    $app->match('/blog', function () {
        ...
    })
    ->method('PUT|POST');

.. note::

    Порядок, в котором определяются маршруты очень важен. Используется первый подходящий маршрут, поэтому располагайте более общие маршруты внизу.


Переменные маршрутов
~~~~~~~~~~~~~~~~~~~~

Как уже было показано, вы можете использовать в маршрутах переменные части::

    $app->get('/blog/{id}', function ($id) {
        ...
    });

Также возможно использовать более одной переменной части, просто убедитесь что аргументы замыкания соответствуют именам переменных частей::

    $app->get('/blog/{postId}/{commentId}', function ($postId, $commentId) {
        ...
    });

Так как обязанности соблюдать порядок нет, вы можете менять аргументы местами::

    $app->get('/blog/{postId}/{commentId}', function ($commentId, $postId) {
        ...
    });

Вы также можете запросить текущие объекты Request и Application::

    $app->get('/blog/{id}', function (Application $app, Request $request, $id) {
        ...
    });

.. note::

    Обратите внимание, что для объектов Application и Request, Silex делает инжектирование основываясь на подсказке о типе переменной, а не на имени::

        $app->get('/blog/{id}', function (Application $foo, Request $bar, $id) {
            ...
        });

Конвертеры переменных маршрута
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Перед инжектированием переменных маршрута в контроллер, вы можете применять конвертеры::

    $app->get('/user/{id}', function ($id) {
        // ...
    })->convert('id', function ($id) { return (int) $id; });

Это полезно, когда вы хотите конвертировать переменные маршрута в объекты, так как можно использовать код конвертера в различных контроллерах::

    $userProvider = function ($id) {
        return new User($id);
    };

    $app->get('/user/{user}', function (User $user) {
        // ...
    })->convert('user', $userProvider);

    $app->get('/user/{user}/edit', function (User $user) {
        // ...
    })->convert('user', $userProvider);

Вторым аргументом конвертер также получает ``Request``::

    $callback = function ($post, Request $request) {
        return new Post($request->attributes->get('slug'));
    };

    $app->get('/blog/{id}/{slug}', function (Post $post) {
        // ...
    })->convert('post', $callback);

Требования
~~~~~~~~~~

В некоторых случаях вам может потребоваться соответствие определённым выражениям. Вы можете определить требования используя регулярние выражения в методе ``assert`` объекта ``Controller``, который возвращается маршрутными методами.

Следующий код проверяет является ли ``id`` числом, так как ``\d+`` соответствует любому количеству цифр::

    $app->get('/blog/{id}', function ($id) {
        ...
    })
    ->assert('id', '\d+');

Также вы можете строить цепочки из этих вызовов::

    $app->get('/blog/{postId}/{commentId}', function ($postId, $commentId) {
        ...
    })
    ->assert('postId', '\d+')
    ->assert('commentId', '\d+');

Значения по умолчанию
~~~~~~~~~~~~~~~~~~~~~

Вы можете определить значение по умолчанию для любой переменной маршрута, вызвав ``value`` объекта ``Controller``::

    $app->get('/{pageName}', function ($pageName) {
        ...
    })
    ->value('pageName', 'index');

Это позволит сделать соостветствующим маршрут ``/``, в данном случае переменная ``pageName`` будет иметь значение ``index``.

Именованные маршруты
~~~~~~~~~~~~~~~~~~~~

Некоторые провайдеры (такие как ``UrlGeneratorProvider``) могут использовать именованные маршруты. По умолчанию, Silex генерирует имена маршрутов, которые в реальности не могут использоваться. Вы можете дать маршруту имя, вызвав ``bind`` объекта ``Controller``, который возвращается маршрутными методами::

    $app->get('/', function () {
        ...
    })
    ->bind('homepage');

    $app->get('/blog/{id}', function ($id) {
        ...
    })
    ->bind('blog_post');


.. note::

    Смысл в именовании маршрутов появляется лишь тогда, когда вы задействуете провайдеров, использующих ``RouteCollection``.

Контроллеры в классах
~~~~~~~~~~~~~~~~~~~~~

Если вы не хотите использовать анонимные функции, то вы можете определить контроллеры как методы. Используя синтаксис ``КлассКонтроллера::имяМетода``, вы можете сообщить Silex to lazily create the controller object for you::

    $app->get('/', 'Igorw\Foo::bar');

    use Silex\Application;
    use Symfony\Component\HttpFoundation\Request;

    namespace Igorw
    {
        class Foo
        {
            public function bar(Request $request, Application $app)
            {
                ...
            }
        }
    }

This will load the ``Igorw\Foo`` class on demand, create an instance and call
the ``bar`` method to get the response. You can use ``Request`` and
``Silex\Application`` type hints to get ``$request`` and ``$app`` injected.

For an even stronger separation between Silex and your controllers, you can
:doc:`define your controllers as services <providers/service_controller>`.

Global Configuration
--------------------

If a controller setting must be applied to all controllers (a converter, a
middleware, a requirement, or a default value), you can configure it on
``$app['controllers']``, which holds all application controllers::

    $app['controllers']
        ->value('id', '1')
        ->assert('id', '\d+')
        ->requireHttps()
        ->method('get')
        ->convert('id', function () { /* ... */ })
        ->before(function () { /* ... */ })
    ;

These settings are applied to already registered controllers and they become
the defaults for new controllers.

.. note::

    The global configuration does not apply to controller providers you might
    mount as they have their own global configuration (see the Modularity
    paragraph below).

Error handlers
--------------

If some part of your code throws an exception you will want to display some
kind of error page to the user. This is what error handlers do. You can also
use them to do additional things, such as logging.

To register an error handler, pass a closure to the ``error`` method which
takes an ``Exception`` argument and returns a response::

    use Symfony\Component\HttpFoundation\Response;

    $app->error(function (\Exception $e, $code) {
        return new Response('We are sorry, but something went terribly wrong.');
    });

You can also check for specific errors by using the ``$code`` argument, and
handle them differently::

    use Symfony\Component\HttpFoundation\Response;

    $app->error(function (\Exception $e, $code) {
        switch ($code) {
            case 404:
                $message = 'The requested page could not be found.';
                break;
            default:
                $message = 'We are sorry, but something went terribly wrong.';
        }

        return new Response($message);
    });

.. note::

    As Silex ensures that the Response status code is set to the most
    appropriate one depending on the exception, setting the status on the
    response won't work. If you want to overwrite the status code (which you
    should not without a good reason), set the ``X-Status-Code`` header::

        return new Response('Error', 404 /* ignored */, array('X-Status-Code' => 200));

You can restrict an error handler to only handle some Exception classes by
setting a more specific type hint for the Closure argument::

    $app->error(function (\LogicException $e, $code) {
        // this handler will only \LogicException exceptions
        // and exceptions that extends \LogicException
    });

If you want to set up logging you can use a separate error handler for that.
Just make sure you register it before the response error handlers, because
once a response is returned, the following handlers are ignored.

.. note::

    Silex ships with a provider for `Monolog
    <https://github.com/Seldaek/monolog>`_ which handles logging of errors.
    Check out the *Providers* chapter for details.

.. tip::

    Silex comes with a default error handler that displays a detailed error
    message with the stack trace when **debug** is true, and a simple error
    message otherwise. Error handlers registered via the ``error()`` method
    always take precedence but you can keep the nice error messages when debug
    is turned on like this::

        use Symfony\Component\HttpFoundation\Response;

        $app->error(function (\Exception $e, $code) use ($app) {
            if ($app['debug']) {
                return;
            }

            // logic to handle the error and return a Response
        });

The error handlers are also called when you use ``abort`` to abort a request
early::

    $app->get('/blog/{id}', function (Silex\Application $app, $id) use ($blogPosts) {
        if (!isset($blogPosts[$id])) {
            $app->abort(404, "Post $id does not exist.");
        }

        return new Response(...);
    });

Redirects
---------

You can redirect to another page by returning a redirect response, which you
can create by calling the ``redirect`` method::

    $app->get('/', function () use ($app) {
        return $app->redirect('/hello');
    });

This will redirect from ``/`` to ``/hello``.

Forwards
--------

When you want to delegate the rendering to another controller, without a
round-trip to the browser (as for a redirect), use an internal sub-request::

    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpKernel\HttpKernelInterface;

    $app->get('/', function () use ($app) {
        // redirect to /hello
        $subRequest = Request::create('/hello', 'GET');

        return $app->handle($subRequest, HttpKernelInterface::SUB_REQUEST);
    });

.. tip::

    If you are using ``UrlGeneratorProvider``, you can also generate the URI::

        $request = Request::create($app['url_generator']->generate('hello'), 'GET');

There's some more things that you need to keep in mind though. In most cases you
will want to forward some parts of the current master request to the sub-request.
That includes: Cookies, server information, session.
Read more on :doc:`how to make sub-requests <cookbook/sub_requests>`.

JSON
----

If you want to return JSON data, you can use the ``json`` helper method.
Simply pass it your data, status code and headers, and it will create a JSON
response for you::

    $app->get('/users/{id}', function ($id) use ($app) {
        $user = getUser($id);

        if (!$user) {
            $error = array('message' => 'The user was not found.');
            return $app->json($error, 404);
        }

        return $app->json($user);
    });

Streaming
---------

It's possible to create a streaming response, which is important in cases when
you cannot buffer the data being sent::

    $app->get('/images/{file}', function ($file) use ($app) {
        if (!file_exists(__DIR__.'/images/'.$file)) {
            return $app->abort(404, 'The image was not found.');
        }

        $stream = function () use ($file) {
            readfile($file);
        };

        return $app->stream($stream, 200, array('Content-Type' => 'image/png'));
    });

If you need to send chunks, make sure you call ``ob_flush`` and ``flush``
after every chunk::

    $stream = function () {
        $fh = fopen('http://www.example.com/', 'rb');
        while (!feof($fh)) {
          echo fread($fh, 1024);
          ob_flush();
          flush();
        }
        fclose($fh);
    };

Sending a file
--------------

If you want to return a file, you can use the ``sendFile`` helper method.
It eases returning files that would otherwise not be publicly available. Simply
pass it your file path, status code, headers and the content disposition and it
will create a ``BinaryFileResponse`` based response for you::

    $app->get('/files/{path}', function ($path) use ($app) {
        if (!file_exists('/base/path/' . $path)) {
            $app->abort(404);
        }

        return $app->sendFile('/base/path/' . $path);
    });

To further customize the response before returning it, check the API doc for
`Symfony\Component\HttpFoundation\BinaryFileResponse
<http://api.symfony.com/master/Symfony/Component/HttpFoundation/BinaryFileResponse.html>`_::

    return $app
        ->sendFile('/base/path/' . $path)
        ->setContentDisposition(ResponseHeaderBag::DISPOSITION_ATTACHMENT, 'pic.jpg')
    ;

.. note::

    HttpFoundation 2.2 or greater is required for this feature to be available.

Traits
------

Silex comes with PHP traits that define shortcut methods.

.. caution::

    You need to use PHP 5.4 or later to benefit from this feature.

Almost all built-in service providers have some corresponding PHP traits. To
use them, define your own Application class and include the traits you want::

    use Silex\Application;

    class MyApplication extends Application
    {
        use Application\TwigTrait;
        use Application\SecurityTrait;
        use Application\FormTrait;
        use Application\UrlGeneratorTrait;
        use Application\SwiftmailerTrait;
        use Application\MonologTrait;
        use Application\TranslationTrait;
    }

You can also define your own Route class and use some traits::

    use Silex\Route;

    class MyRoute extends Route
    {
        use Route\SecurityTrait;
    }

To use your newly defined route, override the ``$app['route_class']``
setting::

    $app['route_class'] = 'MyRoute';

Read each provider chapter to learn more about the added methods.

Безопасность
------------

Убедитесь в том, что ваши приложения защищены от атак.

Экранирование
~~~~~~~~~~~~~

При выводе любого пользовательского ввода (переменных маршрута, или переменных, полученных из запроса), вы должны убедиться в корректном экранировании для предотвращения XSS-атак.

* **Экранирование HTML**: PHP для этого предлагает функцию ``htmlspecialchars``. В Silex есть более краткий метод ``escape``::

      $app->get('/name', function (Silex\Application $app) {
          $name = $app['request']->get('name');
          return "You provided the name {$app->escape($name)}.";
      });

  Если вы используете движок шаблонов Twig, вы должны использовать его экранирование или даже механизмы авто-экранирования.

* **Экранирование JSON**: Если вы хотите передать данные в формате JSON, вы должны использовать Silex-функцию ``json``::

      $app->get('/name.json', function (Silex\Application $app) {
          $name = $app['request']->get('name');
          return $app->json(array('name' => $name));
      });

.. _download: http://silex.sensiolabs.org/download
