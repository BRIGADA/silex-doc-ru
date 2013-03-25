How to make sub-requests
========================

Так как Silex основывается на ``HttpKernelInterface``, он позволяет имитировать запросы к самому себе.
Это означает, что вы можете встраивать одни страницы в другие, а также пересылать запросы, выполняя внутренние перенаправление, которое не изменяет URL.

Основы
------

Вы можете делать запросы, вызывая метод ``handle`` из ``Application``. Этот метод принимает три аргумента:

* ``$request``: Экземпляр класса ``Request``, который представляет HTTP-запрос.

* ``$type``: Должен быть или ``HttpKernelInterface::MASTER_REQUEST`` или ``HttpKernelInterface::SUB_REQUEST``. Некоторые листенеры исполняются только для мастер-запросов, поэтому важно указать в этом параметре значение ``SUB_REQUEST``.

* ``$catch``: Отлавливать исключения и возвращать их в ответе с кодом состояния ``500``. Этот аргумент по умолчанию имеет значение ``true``. Для подзапросов вам скорее всего захочется установить ``false``.

Вызывая ``handle``, вы можете вручную делать подзапросы. Вот пример::

    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpKernel\HttpKernelInterface;

    $subRequest = Request::create('/');
    $response = $app->handle($subRequest, HttpKernelInterface::SUB_REQUEST, false);

Однако, есть ещё несколько вещей, которые следует учитывать. В большинстве случаев, вам скорее всего потребуется передать в подзапрос некоторые части мастер-запроса. Имеются ввиду плюшки (куки), информация сервера, сессия.

Вот более сложный пример, который передаёт дополнительную информацию (``$request`` хранит мастер-запрос)::

    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpKernel\HttpKernelInterface;

    $subRequest = Request::create('/', 'GET', array(), $request->cookies->all(), array(), $request->server->all());
    if ($request->getSession()) {
        $subRequest->setSession($request->getSession());
    }

    $response = $app->handle($subRequest, HttpKernelInterface::SUB_REQUEST, false);

Для отправки ответа клиенту, вы можете просто вернуть его из контролллера::

    use Silex\Application;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpKernel\HttpKernelInterface;

    $app->get('/foo', function (Application $app, Request $request) {
        $subRequest = Request::create('/', ...);
        $response = $app->handle($subRequest, HttpKernelInterface::SUB_REQUEST, false);

        return $response;
    });

Если вы хотите встроить ответ в большую страницу, вы можете вызвать ``Response::getContent``::

    $header = ...;
    $footer = ...;
    $body = $response->getContent();

    return $header.$body.$footer;

Вывод страниц в шаблонах Twig
-----------------------------

:doc:`TwigServiceProvider </providers/twig>` содержит функцию ``render``, которая может использоваться вами в шаблонах Twig. Это удобный способ встраивания страниц.

.. code-block:: jinja

    {{ render('/sidebar') }}

Подробности смотри в документации к :doc:`TwigServiceProvider </providers/twig>`.

Edge Side Includes
------------------

You can use ESI either through the :doc:`HttpCacheServiceProvider
</providers/http_cache>` or a reverse proxy cache such as Varnish. This also
allows you to embed pages, however it also gives you the benefit of caching
parts of the page.

Here is an example of how you would embed a page via ESI:

.. code-block:: jinja

    <esi:include src="/sidebar" />

For details, refer to the :doc:`HttpCacheServiceProvider
</providers/http_cache>` docs.

Dealing with the request base URL
---------------------------------

One thing to watch out for is the base URL. If your application is not
hosted at the webroot of your web server, then you may have an URL like
``http://example.org/foo/index.php/articles/42``.

In this case, ``/foo/index.php`` is your request base path. Silex accounts for
this path prefix in the routing process, it reads it from
``$request->server``. In the context of sub-requests this can lead to issues,
because if you do not prepend the base path the request could mistake a part
of the path you want to match as the base path and cut it off.

You can prevent that from happening by always prepending the base path when
constructing a request::

    $url = $request->getUriForPath('/');
    $subRequest = Request::create($url, 'GET', array(), $request->cookies->all(), array(), $request->server->all());

This is something to be aware of when making sub-requests by hand.

Lack of container scopes
------------------------

While the sub-requests available in Silex are quite powerful, they have their
limits. The major limitation/danger that you will run into is the lack of
scopes on the Pimple container.

The container is a concept that is global to a Silex application, since the
application object **is** the container. Any request that is run against an
application will re-use the same set of services. Since these services are
mutable, code in a master request can affect the sub-requests and vice versa.
Any services depending on the ``request`` service will store the first request
that they get (could be master or sub-request), and keep using it, even if
that request is already over.

For example::

    use Symfony\Component\HttpFoundation\Request;

    class ContentFormatNegotiator
    {
        private $request;

        public function __construct(Request $request)
        {
            $this->request = $request;
        }

        public function negotiateFormat(array $serverTypes)
        {
            $clientAcceptType = $this->request->headers->get('Accept');

            ...

            return $format;
        }
    }

This example looks harmless, but it might blow up. You have no way of knowing
what ``$request->headers->get()`` will return, because ``$request`` could be
either the master request or a sub-request. The answer in this case is to pass
the request as an argument to ``negotiateFormat``. Then you can pass it in
from a location where you have safe access to the current request: a listener
or a controller.

Here are a few general approaches to working around this issue:

* Use ESI with Varnish.

* Do not inject the request, ever. Use listeners instead, as they can access
  the request without storing it.

* Inject the Silex Application and fetch the request from it.
