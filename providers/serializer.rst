SerializerServiceProvider
===========================

*SerializerServiceProvider* предоставляет службу для сериализации объектов.

Параметры
---------

Нет.

Службы
------

* **serializer**: Экземпляр `Symfony\Component\Serializer\Serializer
  <http://api.symfony.com/master/Symfony/Component/Serializer/Serializer.html>`_.

* **serializer.encoders**: `Symfony\Component\Serializer\Encoder\JsonEncoder
  <http://api.symfony.com/master/Symfony/Component/Serializer/Encoder/JsonEncoder.html>`_
  и `Symfony\Component\Serializer\Encoder\XmlEncoder
  <http://api.symfony.com/master/Symfony/Component/Serializer/Encoder/XmlEncoder.html>`_.

* **serializer.normalizers**: `Symfony\Component\Serializer\Normalizer\CustomNormalizer
  <http://api.symfony.com/master/Symfony/Component/Serializer/Normalizer/CustomNormalizer.html>`_
  и `Symfony\Component\Serializer\Normalizer\GetSetMethodNormalizer
  <http://api.symfony.com/master/Symfony/Component/Serializer/Normalizer/GetSetMethodNormalizer.html>`_.

Регистрация
-----------

.. code-block:: php

    $app->register(new Silex\Provider\SerializerServiceProvider());

Использование
-------------

Провайдер ``SerializerServiceProvider`` предоставляет службу ``serializer``:

.. code-block:: php

    use Silex\Application;
    use Silex\Provider\SerializerServiceProvider;
    use Symfony\Component\HttpFoundation\Response;
    
    $app = new Application();
    
    $app->register(new SerializerServiceProvider());
    
    // поддерживаемые сериализатором типы содержимого задаём через метод assert.
    $app->get("/pages/{id}.{_format}", function ($id) use ($app) {
        // Предположим существование службы page_repository, которая возвращает объекты Page.
        // Эти объекты имею геггеры и сеттеры, позволяющие получать и изменять их состояние.
        $page = $app['page_repository']->find($id);
        $format = $app['request']->getRequestFormat();
    
        if (!$page instanceof Page) {
            $app->abort("No page found for id: $id");
        }
    
        return new Response($app['serializer']->serialize($page, $format), 200, array(
            "Content-Type" => $app['request']->getMimeType($format)
        ));
    })->assert("_format", "xml|json")
      ->assert("id", "\d+");

