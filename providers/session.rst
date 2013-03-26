SessionServiceProvider
======================

*SessionServiceProvider* предоставляет службу для хранения данных между запросами.

Параметры
---------

* **session.storage.save_path** (опциональный): Путь для ``NativeFileSessionHandler``, по умолчанию ``sys_get_temp_dir()``.

* **session.storage.options**: Массив параметров, которые передаются в конструктор службы ``session.storage``.

  В случае используемого по умолчанию класса `NativeSessionStorage
  <http://api.symfony.com/master/Symfony/Component/HttpFoundation/Session/Storage/NativeSessionStorage.html>`_,
  наиболее полезные опции следующие:

  * **name**: Имя печеньки (куки) (по умолчанию ``_SESS``)
  * **id**: Идентификатор сессии (по умолчанию ``null``)
  * **cookie_lifetime**: Время жизни печеньки (куки)
  * **cookie_path**: Путь печеньки (куки)
  * **cookie_domain**: Домен печеньки (куки)
  * **cookie_secure**: Безопасность (HTTPS)
  * **cookie_httponly**: Печенька (кука) только ли для HTTP

  Однако, все они опциональны. Сессия длится до тех пор, пока открыт браузер.
  Для изменения этого поведения установите значение опции ``lifetime``.

  Полный список всех доступных опций приведён в официальной `документации PHP  <http://php.net/session.configuration>`_.

* **session.test**: Требуется ли симуляция сессий или нет (полезно при написании функциональных тестов).

Службы
------

* **session**: Экземпляр класса `Session
  <http://api.symfony.com/master/Symfony/Component/HttpFoundation/Session/Session.html>`_.

* **session.storage**: Служба, используемая для хранения данных сессии.

* **session.storage.handler**: Служба, используемая ``session.storage`` для доступа к данным.
  По умолчанию используется обработчик `NativeFileSessionHandler
  <http://api.symfony.com/master/Symfony/Component/HttpFoundation/Session/Storage/Handler/NativeFileSessionHandler.html>`_.

Регистрация
-----------

.. code-block:: php

    $app->register(new Silex\Provider\SessionServiceProvider());

Использование
-------------

Провайдер сессий предоставляет службу ``session``. Ниже приведён пример аутентификации пользователя и создания сессии для него::

    use Symfony\Component\HttpFoundation\Response;

    $app->get('/login', function () use ($app) {
        $username = $app['request']->server->get('PHP_AUTH_USER', false);
        $password = $app['request']->server->get('PHP_AUTH_PW');

        if ('igor' === $username && 'password' === $password) {
            $app['session']->set('user', array('username' => $username));
            return $app->redirect('/account');
        }

        $response = new Response();
        $response->headers->set('WWW-Authenticate', sprintf('Basic realm="%s"', 'site_login'));
        $response->setStatusCode(401, 'Please sign in.');
        return $response;
    });

    $app->get('/account', function () use ($app) {
        if (null === $user = $app['session']->get('user')) {
            return $app->redirect('/login');
        }

        return "Welcome {$user['username']}!";
    });


Custom Session Configurations
-----------------------------

Если ваша система использует специальную конфигурацию сессий (обработчик redis из расширений PHP),
то вам необходимо отключить NativeFileSessionHandler, установив ``session.storage.handler`` в null,
а также верно указать значение ``session.save_path``.

.. code-block:: php

    $app['session.storage.handler'] = null;

