DoctrineServiceProvider
=======================

*DoctrineServiceProvider* обеспечивает интеграцию с `Doctrine DBAL <http://www.doctrine-project.org/projects/dbal>`_ для более лёгкого доступа к базе данных.

.. note::

    Здесь только Doctrine DBAL. Служба ORM **не поставляется**.

Параметры
---------

* **db.options**: Массив опций Doctrine DBAL.

  Доступные опции:

  * **driver**: Используемый драйвер БД, по умолчанию ``pdo_mysql``.
    Возможные варианты: ``pdo_mysql``, ``pdo_sqlite``, ``pdo_pgsql``, ``pdo_oci``, ``oci8``, ``ibm_db2``, ``pdo_ibm``, ``pdo_sqlsrv``.

  * **dbname**: Имя используемой БД.

  * **host**: Хост сервера БД. По умолчанию ``localhost``.

  * **user**: Имя пользователя БД. По умолчанию ``root``.

  * **password**: Пароль пользователя БД.

  * **charset**: Действительно только для ``pdo_mysql``, ``pdo_oci`` и ``oci8``; указывает используемый набор символов.
    
  * **path**: Действительно только для ``pdo_sqlite``, указывает путь к файлу БД.

  Эти и другие опции более подробно описаны в `документации Doctrine DBAL <http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/configuration.html>`_.

Службы
------

* **db**: Соединение с БД, экземпляр ``Doctrine\DBAL\Connection``.

* **db.config**: Конфигурационный объект Doctrine. По умолчанию пустой ``Doctrine\DBAL\Configuration``.

* **db.event_manager**: Диспетчер событий Doctrine.

Регистрация
-----------

.. code-block:: php

    $app->register(new Silex\Provider\DoctrineServiceProvider(), array(
        'db.options' => array(
            'driver'   => 'pdo_sqlite',
            'path'     => __DIR__.'/app.db',
        ),
    ));

.. note::

    Doctrine DBAL поставляется только в "большом" архиве Silex, а не в обычном. Если вы используете Composer, добавьте зависимость в файл ``composer.json``:

    .. code-block:: json

        "require": {
            "doctrine/dbal": "2.2.*",
         }

Использование
-------------

Этот провайдер предоставляет службу ``db``. Вот пример использования::

    $app->get('/blog/{id}', function ($id) use ($app) {
        $sql = "SELECT * FROM posts WHERE id = ?";
        $post = $app['db']->fetchAssoc($sql, array((int) $id));

        return  "<h1>{$post['title']}</h1>".
                "<p>{$post['body']}</p>";
    });

Использование нескольких БД
---------------------------

Этот провайдер допускает доступ к нескольким БД одновременно. Чтобы сконфигурировать источники данных, замените **db.options** на **dbs.options**.
**dbs.options** -- это массив конфигураций, где ключами являются имена соединений, а значения -- опциями::

    $app->register(new Silex\Provider\DoctrineServiceProvider(), array(
        'dbs.options' => array (
            'mysql_read' => array(
                'driver'    => 'pdo_mysql',
                'host'      => 'mysql_read.someplace.tld',
                'dbname'    => 'my_database',
                'user'      => 'my_username',
                'password'  => 'my_password',
                'charset'   => 'utf8',
            ),
            'mysql_write' => array(
                'driver'    => 'pdo_mysql',
                'host'      => 'mysql_write.someplace.tld',
                'dbname'    => 'my_database',
                'user'      => 'my_username',
                'password'  => 'my_password',
                'charset'   => 'utf8',
            ),
        ),
    ));

Первое зарегистрированное соединение будет соединением по умолчанию и вы можете получить к нему доступ также, как и в случае с одной БД. Применительно к указанной выше конфигурации следующие две строки эквивалентны::

    $app['db']->fetchAssoc('SELECT * FROM table');

    $app['dbs']['mysql_read']->fetchAssoc('SELECT * FROM table');

Использование нескольких соединений::

    $app->get('/blog/{id}', function ($id) use ($app) {
        $sql = "SELECT * FROM posts WHERE id = ?";
        $post = $app['dbs']['mysql_read']->fetchAssoc($sql, array((int) $id));

        $sql = "UPDATE posts SET value = ? WHERE id = ?";
        $app['dbs']['mysql_write']->executeUpdate($sql, array('newValue', (int) $id));

        return  "<h1>{$post['title']}</h1>".
                "<p>{$post['body']}</p>";
    });

Более подробная информация содержится в `документации Doctrine DBAL <http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/>`_.
