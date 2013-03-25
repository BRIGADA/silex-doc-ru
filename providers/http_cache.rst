HttpCacheServiceProvider
========================

*HttpCacheProvider* обеспечивает поддержку Symfony2 Reverse Proxy.

Параметры
---------

* **http_cache.cache_dir**: Каталог, используемый для хранения кешировнных данных HTTP.

* **http_cache.options** (опциональный): Массив опций для конструктора `HttpCache <http://api.symfony.com/master/Symfony/Component/HttpKernel/HttpCache/HttpCache.html>`_.

Службы
------

* **http_cache**: Экземпляр `HttpCache <http://api.symfony.com/master/Symfony/Component/HttpKernel/HttpCache/HttpCache.html>`_.

* **http_cache.esi**: Экземпляр `Esi <http://api.symfony.com/master/Symfony/Component/HttpKernel/HttpCache/Esi.html>`_, который реализует ESI-возможности для экземпляров Request и Response.

* **http_cache.store**: Экземпляр `Store <http://api.symfony.com/master/Symfony/Component/HttpKernel/HttpCache/Store.html>`_,
  который реализует всю логику сохранения метаданных кеша (заголовки Request и Response).

Регистрация
-----------

.. code-block:: php

    $app->register(new Silex\Provider\HttpCacheServiceProvider(), array(
        'http_cache.cache_dir' => __DIR__.'/cache/',
    ));

Использование
-------------

Silex уже поддерживает обратные прокси, такие как Varnish, установкой в Response специальных заголовков HTTP::

    use Symfony\Component\HttpFoundation\Response;

    $app->get('/', function() {
        return new Response('Foo', 200, array(
            'Cache-Control' => 's-maxage=5',
        ));
    });

.. tip::

    If you want Silex to trust the ``X-Forwarded-For*`` headers from your
    reverse proxy, you will need to run your application like this::

        use Symfony\Component\HttpFoundation\Request;

        Request::trustProxyData();
        $app->run();

This provider allows you to use the Symfony2 reverse proxy natively with
Silex applications by using the ``http_cache`` service::

    $app['http_cache']->run();

The provider also provides ESI support::

    $app->get('/', function() {
        $response = new Response(<<<EOF
    <html>
        <body>
            Hello
            <esi:include src="/included" />
        </body>
    </html>

    EOF
        , 200, array(
            'Surrogate-Control' => 'content="ESI/1.0"',
        ));

        $response->setTtl(20);

        return $response;
    });

    $app->get('/included', function() {
        $response = new Response('Foo');
        $response->setTtl(5);

        return $response;
    });

    $app['http_cache']->run();

If your application doesn't use ESI, you can disable it to slightly improve the
overall performance::

    $app->register(new Silex\Provider\HttpCacheServiceProvider(), array(
       'http_cache.cache_dir' => __DIR__.'/cache/',
       'http_cache.esi'       => null,
    ));

.. tip::

    To help you debug caching issues, set your application ``debug`` to true.
    Symfony automatically adds a ``X-Symfony-Cache`` header to each response
    with useful information about cache hits and misses.

    If you are *not* using the Symfony Session provider, you might want to set
    the PHP ``session.cache_limiter`` setting to an empty value to avoid the
    default PHP behavior.

    Finally, check that your Web server does not override your caching strategy.

For more information, consult the `Symfony2 HTTP Cache documentation
<http://symfony.com/doc/current/book/http_cache.html>`_.
