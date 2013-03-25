Провайдеры
==========

Провайдеры позволяют разработчику использовать части одник приложений в других. Silex поддерживает два типа провайдеров, определяемых двумя интерфейсами: ``ServiceProviderInterface`` для служб и ``ControllerProviderInterface`` для контроллеров.

Провайдеры служб
----------------

Загрузка провайдеров
~~~~~~~~~~~~~~~~~~~~

Чтобы загрузить и использовать провайдер службы, вы должны зарегистрировать его в приложении::

    $app = new Silex\Application();

    $app->register(new Acme\DatabaseServiceProvider());

Также вы можете передать некоторые параметры вторым аргументом. Они будут установлены **после** регистрации провайдера, но **перед** его загрузкой::

    $app->register(new Acme\DatabaseServiceProvider(), array(
        'database.dsn'      => 'mysql:host=localhost;dbname=myapp',
        'database.user'     => 'root',
        'database.password' => 'secret_root_password',
    ));

Соглашения
~~~~~~~~~~

Вам необходимо следить за порядком выполнения некоторых вещей при взаимодействии с провайдерами. Просто придерживайтесь этих правил:

* Переопределение существующих служб должно происходить **после** регистрации провайдера.

  *Причина: если служба уже существует, провайдер её перепишет.*

* Вы можете устанавливать параметры в любое время **после** регистрации провайдера, но **перед** обращением к службе.

  *Причина: провайдеры могут устанавливать некоторые значения по умолчанию для параметров. Также, как и в случае со службами, провайдер перезапишет существующие значения.*

Убедитесь в том, что придерживаетесь такого поведения при создании собственных провайдеров.

Включённые провайдеры
~~~~~~~~~~~~~~~~~~~~~

Есть несколько провайдеров, которые вы получаете прямо из коробки. Все они располагаются в пространстве имён ``Silex\Provider``:

* :doc:`DoctrineServiceProvider <providers/doctrine>`
* :doc:`MonologServiceProvider <providers/monolog>`
* :doc:`SessionServiceProvider <providers/session>`
* :doc:`SerializerServiceProvider <providers/serializer>`
* :doc:`SwiftmailerServiceProvider <providers/swiftmailer>`
* :doc:`TwigServiceProvider <providers/twig>`
* :doc:`TranslationServiceProvider <providers/translation>`
* :doc:`UrlGeneratorServiceProvider <providers/url_generator>`
* :doc:`ValidatorServiceProvider <providers/validator>`
* :doc:`HttpCacheServiceProvider <providers/http_cache>`
* :doc:`FormServiceProvider <providers/form>`
* :doc:`SecurityServiceProvider <providers/security>`
* :doc:`ServiceControllerServiceProvider <providers/service_controller>`

Сторонние провайдеры
~~~~~~~~~~~~~~~~~~~~

Некоторые провайдеры служб разрабатываются сообществом. Эти сторонние провайдеры перечислены в `wiki-репозитории Silex <https://github.com/fabpot/Silex/wiki/Third-Party-ServiceProviders>`_.

Вы можете делиться вашими собственными.

Создание провайдера
~~~~~~~~~~~~~~~~~~~

Провайдеры должны реализовывать ``Silex\ServiceProviderInterface``::

    interface ServiceProviderInterface
    {
        function register(Application $app);

        function boot(Application $app);
    }

This is very straight forward, just create a new class that implements the two methods. В методе ``register()`` вы можете определить службы приложения, которые затем могут быть использованы другими службами и параметрами. В методе ``boot()`` вы можете конфигурировать приложение перед обработкой им запроса.

Ниже приведён пример такого провайдера::

    namespace Acme;

    use Silex\Application;
    use Silex\ServiceProviderInterface;

    class HelloServiceProvider implements ServiceProviderInterface
    {
        public function register(Application $app)
        {
            $app['hello'] = $app->protect(function ($name) use ($app) {
                $default = $app['hello.default_name'] ? $app['hello.default_name'] : '';
                $name = $name ?: $default;

                return 'Hello '.$app->escape($name);
            });
        }

        public function boot(Application $app)
        {
        }
    }

Этот класс реализует службу ``hello`` как защищённое замыкание. Он принимает аргумент ``name`` и возвращает ``hello.default_name`` если никакое имя не указано. Если имя по умолчанию также не задано, возвращается пустая строка.

Теперь вы можете использовать этот провайдер следующим образом::

    $app = new Silex\Application();

    $app->register(new Acme\HelloServiceProvider(), array(
        'hello.default_name' => 'Igor',
    ));

    $app->get('/hello', function () use ($app) {
        $name = $app['request']->get('name');

        return $app['hello']($name);
    });

В этом примере мы получаем параметр ``name`` из строки запроса, поэтому путь запроса должен иметь вид ``/hello?name=Fabien``.

Провайдеры контроллеров
-----------------------

Загрузка провайдеров
~~~~~~~~~~~~~~~~~~~~

Для загрузки и использования провайдера контроллера вы должны "смонтировать" его контроллеры по определённому пути::

    $app = new Silex\Application();

    $app->mount('/blog', new Acme\BlogControllerProvider());

Теперь все определённые провайдером контроллеры доступны по пути ``/blog``.

Создание провайдера
~~~~~~~~~~~~~~~~~~~

Провайдеры должны реализовывать ``Silex\ControllerProviderInterface``::

    interface ControllerProviderInterface
    {
        function connect(Application $app);
    }

Ниже приведён пример такого провайдера::

    namespace Acme;

    use Silex\Application;
    use Silex\ControllerProviderInterface;

    class HelloControllerProvider implements ControllerProviderInterface
    {
        public function connect(Application $app)
        {
            // создание нового контроллера для маршрута по умолчанию
            $controllers = $app['controllers_factory'];

            $controllers->get('/', function (Application $app) {
                return $app->redirect('/hello');
            });

            return $controllers;
        }
    }

Метод ``connect`` должен возвращать экземпляр ``ControllerCollection``. ``ControllerCollection`` -- это класс, в котором определены все связанные с контроллером методы (такие как ``get``, ``post``, ``match``, ...).

.. tip::

    Класс ``Application`` по факту выступает как прокси для этих методов.

Теперь вы може использовать этот провайдер следующим образом::

    $app = new Silex\Application();

    $app->mount('/blog', new Acme\HelloControllerProvider());

В этом примере, путь ``/blog/`` относится к контроллерам, которые определяются провайдером.

.. tip::

    Вы также можете определить провайдер, который будет реализовывать оба интерфейса одновременно.
