Тестирование
============

Так как Silex построен из компонентов Symfony2, в нём очень легко писать функциональные тесты для ваших приложений.
Функциональные тесты -- это автоматизированные тесты программного обеспечения, которые гарантируют, что ваш код работает правильно. Они выполняются в пользовательском интерфейсе через фальшивый браузер и имитируют действия пользователя.

Почему
------

Если вы не знакомы с тестирование программного обеспечения, вас может удивить необходимость этого. Каждый раз, когда вы вносите изменения в своё приложение, вам необходимо их проверять. Это означает, что вам нужно пройтись по всем возможным страницам и убедиться в том, что они всё ещё работают. Функциональные тесты экономят кучу вашего времени, поскольку позволяют протестировать всё приложение за несколько секунд с помощью запуска одной команды.

Более подробно о функциональных и модульных тестах, а также об автоматическом тестировании программного обеспечения в целом смотри `PHPUnit <https://github.com/sebastianbergmann/phpunit>`_ и `Bulat Shakirzyanov's talk on Clean Code <http://www.slideshare.net/avalanche123/clean-code-5609451>`_.

PHPUnit
-------

`PHPUnit <https://github.com/sebastianbergmann/phpunit>`_ де-факто является стандартной средой тестирования для PHP. Она создана для модульного тестирования, однако её также можно использовать и для функциональных тестов. Вы пишите тесты создавая новый класс, который расширяет ``PHPUnit_Framework_TestCase``. Сами же тесты -- это методы, которые начинаются с ``test``::

    class ContactFormTest extends PHPUnit_Framework_TestCase
    {
        public function testInitialPage()
        {
            ...
        }
    }

В тестах вы описываете ожидаемое состояние. В следующем тесте мы проверяем форму контактов, поэтому мы должны убедиться что страница загружена успешно и она содержит нашу форму::

        public function testInitialPage()
        {
            $statusCode = ...
            $pageContent = ...

            $this->assertEquals(200, $statusCode);
            $this->assertContains('Contact us', $pageContent);
            $this->assertContains('<form', $pageContent);
        }

Здесь вы видите некоторые из возможных проверок. Полный список содержится в главе `Writing Tests for PHPUnit
<http://www.phpunit.de/manual/current/en/writing-tests-for-phpunit.html>`_ документации PHPUnit.

WebTestCase
-----------

В Symfony2 есть класс WebTestCase, который используется для написания функциональных тестов. Silex также содержит этот класс: ``Silex\WebTestCase``, и вы можете использовать его::

    use Silex\WebTestCase;

    class ContactFormTest extends WebTestCase
    {
        ...
    }

.. note::

    Чтобы сделать своё приложение тестируемым, вам необходимо следовать инструкциям из раздела "Повторное использование приложений" главы :doc:`usage`.

Для вашего WebTestCase, вам необходимо реализовать метод ``createApplication``, который вернёт ваше приложение. Обычно это выглядит так::

        public function createApplication()
        {
            return require __DIR__.'/path/to/app.php';
        }

Убедитесь, что вы **не используете** здесь ``require_once``, так как этот метод вызывается перед каждым тестом.

.. tip::

    По умолчанию, приложение ведёт себя также, как и при обращении к нему через браузер. Однако при возникновении ошибки иногда проще получить простое исключание, а не HTML-страницу. Добиться этого достаточно просто, задав соответствующую конфигурацию приложения в методе ``createApplication()``::

        public function createApplication()
        {
            $app = require __DIR__.'/path/to/app.php';
            $app['debug'] = true;
            $app['exception_handler']->disable();

            return $app;
        }

.. tip::

    Если ваше приложение использует сессии, установите для ``session.test`` значение ``true`` для симуляции сессий::

        public function createApplication()
        {
            // ...

            $app['session.test'] = true;

            // ...
        }

В WebTestCase имеется метод ``createClient``, который создаёт экземпляр клиента. Клиент действует как браузер и позволяет вам взаимодействовать с приложением. Работает это следующим образом::

        public function testInitialPage()
        {
            $client = $this->createClient();
            $crawler = $client->request('GET', '/');

            $this->assertTrue($client->getResponse()->isOk());
            $this->assertCount(1, $crawler->filter('h1:contains("Contact us")'));
            $this->assertCount(1, $crawler->filter('form'));
            ...
        }

Здесь появляются некоторые любопытные вещи: ``Client`` и ``Crawler``.

Также вы можете получить доступ к приложению через ``$this->app``.

Client
------

Этот класс представляет браузер. Он ведёт историю, сохраняет печеньки (куки) и многое другое. Метод ``request`` позволяет запросить страницу у вашего приложения.

.. note::

    Более подробное описание приводится в соответствующем разделе `документации Symfony2 <http://symfony.com/doc/current/book/testing.html#the-test-client>`_.

Crawler
-------

Экземпляры этого класса позволяют вам инспектировать содержимое страницы. Например вы можете фильтровать его при помощи CSS-выражений.

.. note::

    Более подробное описание приводится в соответствующем разделе `документации Symfony2 <http://symfony.com/doc/current/book/testing.html#the-test-client>`_.

Конфигурация
------------

Предлагаемый путь конфигурирования PHPUnit заключается в создании файла ``phpunit.xml.dist``, папки ``tests`` и помещение ваших тестов в ``tests/YourApp/Tests/YourTest.php``. Файл ``phpunit.xml.dist`` должен выглядеть так:

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <phpunit backupGlobals="false"
             backupStaticAttributes="false"
             colors="true"
             convertErrorsToExceptions="true"
             convertNoticesToExceptions="true"
             convertWarningsToExceptions="true"
             processIsolation="false"
             stopOnFailure="false"
             syntaxCheck="false"
    >
        <testsuites>
            <testsuite name="YourApp Test Suite">
                <directory>./tests/</directory>
            </testsuite>
        </testsuites>
    </phpunit>

Файл ``tests/YourApp/Tests/YourTest.php`` должен выглядеть следующим образом::

    namespace YourApp\Tests;

    use Silex\WebTestCase;

    class YourTest extends WebTestCase
    {
        public function createApplication()
        {
            return require __DIR__.'/../../../app.php';
        }

        public function testFooBar()
        {
            ...
        }
    }

Теперь, если вы в командной строке запустите ``phpunit``, то ваши тесты будут выполнены.
