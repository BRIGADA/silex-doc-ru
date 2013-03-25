Как использовать PdoSessionStorage для хранения сессий в базе данных
====================================================================

По умолчанию, :doc:`SessionServiceProvider </providers/session>` записывает сессионную информацию в файлы используя NativeFileSessionStorage из Symfony2. Большинство средних и крупных веб-сайтов вместо фалов используют базу данных, так как базы данных проще использовать и масштабировать для среды с несколькими веб-серверами.

В Symfony2 есть несколько обработчиков для хранения сессий и один из них использует PDO: `PdoSessionHandler <http://api.symfony.com/master/Symfony/Component/HttpFoundation/Session/Storage/Handler/PdoSessionHandler.html>`_. Для его задействования, замените службу ``session.storage.handler`` вашего приложения так, как показано ниже.

С выделенной PDO-службой 
------------------------

.. code-block:: php

    use Symfony\Component\HttpFoundation\Session\Storage\Handler\PdoSessionHandler;

    $app->register(new Silex\Provider\SessionServiceProvider());

    $app['pdo.dsn'] = 'mysql:dbname=mydatabase';
    $app['pdo.user'] = 'myuser';
    $app['pdo.password'] = 'mypassword';

    $app['session.db_options'] = array(
        'db_table'      => 'session',
        'db_id_col'     => 'session_id',
        'db_data_col'   => 'session_value',
        'db_time_col'   => 'session_time',
    );

    $app['pdo'] = $app->share(function () use ($app) {
        return new PDO(
            $app['pdo.dsn'],
            $app['pdo.user'],
            $app['pdo.password']
        );
    });

    $app['session.storage.handler'] = $app->share(function () use ($app) {
        return new PdoSessionHandler(
            $app['pdo'],
            $app['session.db_options'],
            $app['session.storage.options']
        );
    });

Использование DoctrineServiceProvider
-------------------------------------

При использовании :doc:`DoctrineServiceProvider </providers/doctrine>` вам не требуется иметь отдельное соединение с базой данных, просто передайте метод ``getWrappedConnection``.

.. code-block:: php

    use Symfony\Component\HttpFoundation\Session\Storage\Handler\PdoSessionHandler;

    $app->register(new Silex\Provider\SessionServiceProvider());

    $app['session.db_options'] = array(
        'db_table'      => 'session',
        'db_id_col'     => 'session_id',
        'db_data_col'   => 'session_value',
        'db_time_col'   => 'session_time',
    );

    $app['session.storage.handler'] = $app->share(function () use ($app) {
        return new PdoSessionHandler(
            $app['db']->getWrappedConnection(),
            $app['session.db_options'],
            $app['session.storage.options']
        );
    });

Структура базы данных
---------------------

Для PdoSessionStorage требуется таблица базы данных с 3 столбцами:

* ``session_id``: Столбец ID (VARCHAR(255) или больше)
* ``session_value``: Столбец значений (TEXT или CLOB)
* ``session_time``: Столбец времени (INTEGER)

Примеры SQL-выражений для создания таблицы сессий можно найти в `Symfony2 cookbook <http://symfony.com/doc/current/cookbook/configuration/pdo_session_storage.html#example-sql-statements>`_
