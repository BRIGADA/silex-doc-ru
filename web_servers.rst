Конфигурирование веб-сервера
============================

Apache
------

Если вы используете Apache, то задействуйте следующий файл ``.htaccess``:

.. code-block:: apache

    <IfModule mod_rewrite.c>
        Options -MultiViews

        RewriteEngine On
        #RewriteBase /path/to/app
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^ index.php [L]
    </IfModule>

.. note::

    Если ваш сайт располагается не в корне, вам необходимо раскоментировать выражение ``RewriteBase`` и указать путь, соответствующий вашему каталогу, относительно корня сайта.

Если вы используете Apache 2.2.16 или выше, то вы можете использовать директиву `FallbackResource`_ для упрощения .htaccess:

.. code-block:: apache

    FallbackResource /index.php

.. note::

    Если ваш сайт не располагается в корне, вам необходимо скорректировать путь до вашего каталога относительно корня.

nginx
-----

Если вы используете nginx, сконфигурируйте ваш vhost таким образом, чтобы все несуществующие ресурсы направлялись в ``index.php``:

.. code-block:: nginx

    server {
        #корень сайта: перенаправляется в загрузочный скрипт приложения
        location = / {
            try_files @site @site;
        }

        #все прочие расположения: вначале проверяется существование файлов, а затем идём в наш фронт-контроллер
        location / {
            try_files $uri $uri/ @site;
        }

        #возвращаем 404 для всех php-файлов, т.к. у нас есть фронт-контроллер
        location ~ \.php$ {
            return 404;
        }

        location @site {
            fastcgi_pass   unix:/var/run/php-fpm/www.sock;
            include fastcgi_params;
            fastcgi_param  SCRIPT_FILENAME $document_root/index.php;
            #раскоментировать при использовании https
            #fastcgi_param HTTPS on;
        }
    }

IIS
---

Если вы используете Internet Information Services из Windows, вы можете использовать следующий пример файла ``web.config``:

.. code-block:: xml

    <?xml version="1.0"?>
    <configuration>
        <system.webServer>
            <defaultDocument>
                <files>
                    <clear />
                    <add value="index.php" />
                </files>
            </defaultDocument>
            <rewrite>
                <rules>
                    <rule name="Silex Front Controller" stopProcessing="true">
                        <match url="^(.*)$" ignoreCase="false" />
                        <conditions logicalGrouping="MatchAll">
                            <add input="{REQUEST_FILENAME}" matchType="IsFile" ignoreCase="false" negate="true" />
                        </conditions>
                        <action type="Rewrite" url="index.php" appendQueryString="true" />
                    </rule>
                </rules>
            </rewrite>
        </system.webServer>
    </configuration>

Lighttpd
--------

Если вы используете lighttpd, воспользуйтесь примером ``simple-vhost`` в качестве отправной точки:

.. code-block:: lighttpd

    server.document-root = "/path/to/app"

    url.rewrite-once = (
        # configure some static files
        "^/assets/.+" => "$0",
        "^/favicon\.ico$" => "$0",

        "^(/[^\?]*)(\?.*)?" => "/index.php$1$2"
    )

.. _FallbackResource: http://www.adayinthelifeof.nl/2012/01/21/apaches-fallbackresource-your-new-htaccess-command/

PHP 5.4
-------

PHP 5.4 поставляется вместе со встроенным веб-сервером для разработки. Этот сервер позволяет вам запускать Silex вообще без конфигурирования. Однако, для обслуживания статичных файлов вам нужно убедиться, что ваш фронт-контроллер возвращает ``false`` в этом случае::

    // web/index.php

    $filename = __DIR__.preg_replace('#(\?.*)$#', '', $_SERVER['REQUEST_URI']);
    if (php_sapi_name() === 'cli-server' && is_file($filename)) {
        return false;
    }

    $app = require __DIR__.'/../src/app.php';
    $app->run();


Если ваш фронт-контроллер располагается в файле ``web/index.php``, вы можете запустить сервер из командной строки:

.. code-block:: text

    $ php -S localhost:8080 -t web web/index.php

Теперь приложение должно быть доступно по адресу ``http://localhost:8080``.

.. note::

    Это сервер только для разработки. Его использование для реальных проектов **НЕ РЕКОМЕНДУЕТСЯ**.
