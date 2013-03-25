Как использовать YAML для настройки валидации
=============================================

Простота является одним из базовых принципов Silex, поэтому в комплекте поставки нет ничего, что могло бы использовать YAML-файлы для валидации. Но это отнюдь не означает, что это невозможно в принципе. Давайте посмотрим как это можно сделать.

Во-первых, вам необходимо установить компонент YAML. Определите его как зависимость в вашем файле ``composer.json``:

.. code-block:: json

    "require": {
        "symfony/yaml": "~2.1"
    }

Затем, вам необходимо сообщить службе валидации, что вы хотите использовать YAML-файл вместо загрузки класса метаданных через ``StaticMethodLoader``::

    $app->register(new ValidatorServiceProvider());

    $app['validator.mapping.class_metadata_factory'] = new Symfony\Component\Validator\Mapping\ClassMetadataFactory(
        new Symfony\Component\Validator\Mapping\Loader\YamlFileLoader(__DIR__.'/validation.yml')
    );

Теперь, мы можем заменить использование статического метода и переместить все правила валидации в ``validation.yml``:

.. code-block:: yaml

    # validation.yml
    Post:
      properties:
        title:
          - NotNull: ~
          - NotBlank: ~
        body:
          - Min: 100
