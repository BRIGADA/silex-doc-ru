Перевод сообщений валидации
===========================

При работе с валидаторами Symfony2, общей задачей будет отображение локализованных сообщений.

Для того, чтобы сделать это, вам необходимо зарегистрировать переводчик указать переведённые ресурсы::

    $app->register(new Silex\Provider\TranslationServiceProvider(), array(
        'locale' => 'sr_Latn',
        'translator.domains' => array(),
    ));

    $app->before(function () use ($app) {
        $app['translator']->addLoader('xlf', new Symfony\Component\Translation\Loader\XliffFileLoader());
        $app['translator']->addResource('xlf', __DIR__.'/vendor/symfony/src/Symfony/Bundle/FrameworkBundle/Resources/translations/validators.sr_Latn.xlf', 'sr_Latn', 'validators');
    });

Этого достаточно для загрузки переводов из файлов ``xlf``.
