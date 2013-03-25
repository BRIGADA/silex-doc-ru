FormServiceProvider
===================

*FormServiceProvider* представляет службу для построения форм в вашем приложении с помощью компонента Symfony2 Form.

Параметры
---------

* **form.secret**: Это значение секрета, которое используется для генерации и валидации токена CSRF. Для вас очень важно задать здесь статическое случайно сгенерированное значение, чтобы предотвратить "угон" форм. Значение по умолчанию -- ``md5(__DIR__)``.

Службы
------

* **form.factory**: Экземпляр `FormFactory <http://api.symfony.com/master/Symfony/Component/Form/FormFactory.html>`_, который используется для построения форм.

* **form.csrf_provider**: Экземпляр реализации `CsrfProviderInterface <http://api.symfony.com/master/Symfony/Component/Form/Extension/Csrf/CsrfProvider/CsrfProviderInterface.html>`_, по умолчанию `DefaultCsrfProvider <http://api.symfony.com/master/Symfony/Component/Form/Extension/Csrf/CsrfProvider/DefaultCsrfProvider.html>`_.

Регистрация
-----------

.. code-block:: php

    use Silex\Provider\FormServiceProvider;

    $app->register(new FormServiceProvider());

.. note::

    Если вы не хотите создавать собственную разметку для формы, то используется заданный по умолчанию.
    В этом случае вам необходимо зарегистрировать :doc:`провайдер Translation <providers/translation>`.

    Если вы хотите использовать валидацию, то не забудте зарегистрировать :doc:`провайдер Validator <providers/validator>`.

.. note::

    Компонент Symfony Form и все его зависимости (опциональные и нет) поставляются только в "большом" архиве Silex.

    Если вы используете Composer, добавьте зависимость в файл ``composer.json``:

    .. code-block:: json

        "require": {
            "symfony/form": "~2.1.4"
        }

    Если вы будете использовать расширение validation, вы должны также добавить зависимости для ``symfony/config`` и ```symfony/translation``:

    .. code-block:: json

        "require": {
            "symfony/validator": "~2.1",
            "symfony/config": "~2.1",
            "symfony/translation": "~2.1"
        }

    Компонент Symfony Form зависит от PHP-расширения intl. Если у вас его нет, то в качестве замены вы можете установить компонент Symfony Locale:

    .. code-block:: json

        "require": {
            "symfony/locale": "~2.1"
        }

    Если вы хотите использовать формы в шаблонах Twig, убедитесь что установили Symfony Twig Bridge:

    .. code-block:: json

        "require": {
            "symfony/twig-bridge": "~2.1"
        }

Использование
-------------

FormServiceProvider предоставляет службу ``form.factory``. Вот пример использования::

    $app->match('/form', function (Request $request) use ($app) {
        // некоторые начальные данные при отображении формы первый раз
        $data = array(
            'name' => 'Your name',
            'email' => 'Your email',
        );

        $form = $app['form.factory']->createBuilder('form', $data)
            ->add('name')
            ->add('email')
            ->add('gender', 'choice', array(
                'choices' => array(1 => 'male', 2 => 'female'),
                'expanded' => true,
            ))
            ->getForm();

        if ('POST' == $request->getMethod()) {
            $form->bind($request);

            if ($form->isValid()) {
                $data = $form->getData();

                // делаем что-нибудь с данными

                // редирект куда-нибудь
                return $app->redirect('...');
            }
        }

        // отображение формы
        return $app['twig']->render('index.twig', array('form' => $form->createView()));
    });

Шаблон формы ``index.twig`` (требуется ``symfony/twig-bridge``):

.. code-block:: jinja

    <form action="#" method="post">
        {{ form_widget(form) }}

        <input type="submit" name="submit" />
    </form>

Если вы используете провайдер валидации, вы также можете добавить его к форме, указав ограничения для полей::

    use Symfony\Component\Validator\Constraints as Assert;

    $app->register(new Silex\Provider\ValidatorServiceProvider());
    $app->register(new Silex\Provider\TranslationServiceProvider(), array(
        'translator.messages' => array(),
    ));

    $form = $app['form.factory']->createBuilder('form')
        ->add('name', 'text', array(
            'constraints' => array(new Assert\NotBlank(), new Assert\Length(array('min' => 5)))
        ))
        ->add('email', 'text', array(
            'constraints' => new Assert\Email()
        ))
        ->add('gender', 'choice', array(
            'choices' => array(1 => 'male', 2 => 'female'),
            'expanded' => true,
            'constraints' => new Assert\Choice(array(1, 2)),
        ))
        ->getForm();

Вы можете зарегистрировать расширение формы, расширив ``form.extensions``::

    $app['form.extensions'] = $app->share($app->extend('form.extensions', function ($extensions) use ($app) {
        $extensions[] = new YourTopFormExtension();

        return $extensions;
    }));


Вы можете зарегистрировать расширение типа формы, расширив ``form.type.extensions``::

    $app['form.type.extensions'] = $app->share($app->extend('form.type.extensions', function ($extensions) use ($app) {
        $extensions[] = new YourFormTypeExtension();

        return $extensions;
    }));

Вы можете зарегистрировать "отгадыватель" типа формы, расширив ``form.type.guessers``::

    $app['form.type.guessers'] = $app->share($app->extend('form.type.guessers', function ($guessers) use ($app) {
        $guessers[] = new YourFormTypeGuesser();

        return $guessers;
    }));

Особенности
-----------

``Silex\Application\FormTrait`` добавляет следующие ярлыки:

* **form**: Создаёт экземпляр FormBuilder.

.. code-block:: php

    $app->form($data);

Больше информации содержится в `документации Symfony2 Forms <http://symfony.com/doc/2.1/book/forms.html>`_.
