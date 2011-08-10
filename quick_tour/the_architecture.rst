Архитектура
===========

Вы мой герой! Кто бы мог подумать, что вы все еще будете здесь после первых трех частей? Ваши усилия скоро будут вознаграждены. В первых частях мы глубоко не рассматривали архитектуру фреймворка. Поскольку это одна из отличительных особенностей Symfony, давайте-ка остановимся на этом подробнее.

Структура папок
---------------

Структура директорий :term:`приложения<приложение>` очень гибкая,
but the directory structure of the *Standard Edition* distribution reflects
the typical and recommended structure of a Symfony2 application:

* ``app/``:    Эта папка содержит конфигурацию приложения;
* ``src/``:    Весь PHP код хранится здесь;
* ``vendor/``: The third-party dependencies;
* ``web/``:    Эта папка должна быть корневой web директорией.

Каталог ``web/``
~~~~~~~~~~~~~~~~

Корневая web директория - это дом для всех публичных и статичных файлов, таких
как изображения, таблицы стилей и файлы JavaScript. Здесь также обитает
:term:`front controller`::

    // web/app.php
    require_once __DIR__.'/../app/bootstrap.php.cache';
    require_once __DIR__.'/../app/AppKernel.php';

    use Symfony\Component\HttpFoundation\Request;

    $kernel = new AppKernel('prod', false);
    $kernel->loadClassCache();
    $kernel->handle(Request::createFromGlobals())->send();

The kernel first requires the ``bootstrap.php.cache`` file, which bootstraps
the framework and registers the autoloader (see below).

Like any front controller, ``app.php`` uses a Kernel Class, ``AppKernel``, to
bootstrap the application.

.. _the-app-dir:

Папка ``app/``
~~~~~~~~~~~~~~

Класс ``AppKernel`` это главная входная точка конфигурации приложения, поэтому
он содержится в директории ``app/``.

Этот класс должен реализовывать два метода:

* ``registerBundles()`` возвращает массив всех бандлов, необходимых для
  запуска приложения;

* ``registerContainerConfiguration()`` загружает конфигурацию  (об этом чуть позже);

PHP autoloading can be configured via ``app/autoload.php``::

    // app/autoload.php
    use Symfony\Component\ClassLoader\UniversalClassLoader;

    $loader = new UniversalClassLoader();
    $loader->registerNamespaces(array(
        'Symfony'          => array(__DIR__.'/../vendor/symfony/src', __DIR__.'/../vendor/bundles'),
        'Sensio'           => __DIR__.'/../vendor/bundles',
        'JMS'              => __DIR__.'/../vendor/bundles',
        'Doctrine\\Common' => __DIR__.'/../vendor/doctrine-common/lib',
        'Doctrine\\DBAL'   => __DIR__.'/../vendor/doctrine-dbal/lib',
        'Doctrine'         => __DIR__.'/../vendor/doctrine/lib',
        'Monolog'          => __DIR__.'/../vendor/monolog/src',
        'Assetic'          => __DIR__.'/../vendor/assetic/src',
        'Metadata'         => __DIR__.'/../vendor/metadata/src',
    ));
    $loader->registerPrefixes(array(
        'Twig_Extensions_' => __DIR__.'/../vendor/twig-extensions/lib',
        'Twig_'            => __DIR__.'/../vendor/twig/lib',
    ));

    // ...

    $loader->registerNamespaceFallbacks(array(
        __DIR__.'/../src',
    ));
    $loader->register();

The :class:`Symfony\\Component\\ClassLoader\\UniversalClassLoader` is used to
autoload files that respect either the technical interoperability `standards`_
for PHP 5.3 namespaces or the PEAR naming `convention`_ for classes. As you
can see here, all dependencies are stored under the ``vendor/`` directory, but
this is just a convention. You can store them wherever you want, globally on
your server or locally in your projects.

.. note::

    If you want to learn more about the flexibility of the Symfony2
    autoloader, read the ":doc:`/cookbook/tools/autoloader`" recipe in the
    cookbook.

Система бандлов
---------------

Этот раздел кратко поведает вам об одной из существеннейших и наиболее мощных
особенностей Symfony2, о системе :term:`пакетов<пакет>`.

Бандл в некотором роде как плагин в других программах. Почему его назвали
*бандл*, а не *плагин*? Потому что *всё что угодно* в Symfony2 это бандл, от
ключевых особенностей фреймворка до кода, который вы пишете для приложения.
Бандлы это высшая каста в Symfony2. Это даёт вам гибкость в применении как уже
встроенных особенностей сторонних бандлов, так и в написании своих собственных.
Бандл позволяет выбрать необходимые для приложения особенности и оптимизировать
их как вы этого хотите.

Registering a Bundle
~~~~~~~~~~~~~~~~~~~~

An application is made up of bundles as defined in the ``registerBundles()``
method of the ``AppKernel`` class. Each bundle is a directory that contains
a single ``Bundle`` class that describes it::

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
            new Symfony\Bundle\SecurityBundle\SecurityBundle(),
            new Symfony\Bundle\TwigBundle\TwigBundle(),
            new Symfony\Bundle\MonologBundle\MonologBundle(),
            new Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle(),
            new Symfony\Bundle\DoctrineBundle\DoctrineBundle(),
            new Symfony\Bundle\AsseticBundle\AsseticBundle(),
            new Sensio\Bundle\FrameworkExtraBundle\SensioFrameworkExtraBundle(),
            new JMS\SecurityExtraBundle\JMSSecurityExtraBundle(),
        );

        if (in_array($this->getEnvironment(), array('dev', 'test'))) {
            $bundles[] = new Acme\DemoBundle\AcmeDemoBundle();
            $bundles[] = new Symfony\Bundle\WebProfilerBundle\WebProfilerBundle();
            $bundles[] = new Sensio\Bundle\DistributionBundle\SensioDistributionBundle();
            $bundles[] = new Sensio\Bundle\GeneratorBundle\SensioGeneratorBundle();
        }

        return $bundles;
    }

In addition to the ``AcmeDemoBundle`` that we have already talked about, notice
that the kernel also enables other bundles such as the ``FrameworkBundle``,
``DoctrineBundle``, ``SwiftmailerBundle``, and ``AsseticBundle`` bundle.
They are all part of the core framework.

Configuring a Bundle
~~~~~~~~~~~~~~~~~~~~

Каждый бандл может быть настроен при помощи конфигурационных файлов, написанных
на YAML, XML, или PHP. Взгляните на конфигурацию по умолчанию:

.. code-block:: yaml

    # app/config/config.yml
    imports:
        - { resource: parameters.ini }
        - { resource: security.yml }

    framework:
        secret:          %secret%
        charset:         UTF-8
        router:          { resource: "%kernel.root_dir%/config/routing.yml" }
        form:            true
        csrf_protection: true
        validation:      { enable_annotations: true }
        templating:      { engines: ['twig'] } #assets_version: SomeVersionScheme
        session:
            default_locale: %locale%
            auto_start:     true

    # Twig Configuration
    twig:
        debug:            %kernel.debug%
        strict_variables: %kernel.debug%

    # Assetic Configuration
    assetic:
        debug:          %kernel.debug%
        use_controller: false
        filters:
            cssrewrite: ~
            # closure:
            #     jar: %kernel.root_dir%/java/compiler.jar
            # yui_css:
            #     jar: %kernel.root_dir%/java/yuicompressor-2.4.2.jar

    # Doctrine Configuration
    doctrine:
        dbal:
            driver:   %database_driver%
            host:     %database_host%
            dbname:   %database_name%
            user:     %database_user%
            password: %database_password%
            charset:  UTF8

        orm:
            auto_generate_proxy_classes: %kernel.debug%
            auto_mapping: true

    # Swiftmailer Configuration
    swiftmailer:
        transport: %mailer_transport%
        host:      %mailer_host%
        username:  %mailer_user%
        password:  %mailer_password%

    jms_security_extra:
        secure_controllers:  true
        secure_all_services: false

Each entry like ``framework`` defines the configuration for a specific bundle.
For example, ``framework`` configures the ``FrameworkBundle`` while ``swiftmailer``
configures the ``SwiftmailerBundle``.

Каждое `окружение` (:term:`environment`) может переопределять стандартную
конфигурацию, задавая специфичный конфигурационный файл. For example, the ``dev`` environment loads the
``config_dev.yml`` file, which loads the main configuration (i.e. ``config.yml``)
and then modifies it to add some debugging tools:

.. code-block:: yaml

    # app/config/config_dev.yml
    imports:
        - { resource: config.yml }

    framework:
        router:   { resource: "%kernel.root_dir%/config/routing_dev.yml" }
        profiler: { only_exceptions: false }

    web_profiler:
        toolbar: true
        intercept_redirects: false

    monolog:
        handlers:
            main:
                type:  stream
                path:  %kernel.logs_dir%/%kernel.environment%.log
                level: debug
            firephp:
                type:  firephp
                level: info

    assetic:
        use_controller: true

Extending a Bundle
~~~~~~~~~~~~~~~~~~

In addition to being a nice way to organize and configure your code, a bundle
can extend another bundle. Bundle inheritance allows you to override any existing
bundle in order to customize its controllers, templates, or any of its files.
This is where the logical names (e.g. ``@AcmeDemoBundle/Controller/SecuredController.php``)
come in handy: they abstract where the resource is actually stored.

Logical File Names
..................

When you want to reference a file from a bundle, use this notation:
``@BUNDLE_NAME/path/to/file``; Symfony2 will resolve ``@BUNDLE_NAME``
to the real path to the bundle. For instance, the logical path
``@AcmeDemoBundle/Controller/DemoController.php`` would be converted to
``src/Acme/DemoBundle/Controller/DemoController.php``, because Symfony knows
the location of the ``AcmeDemoBundle``.

Logical Controller Names
........................

For controllers, you need to reference method names using the format
``BUNDLE_NAME:CONTROLLER_NAME:ACTION_NAME``. For instance,
``AcmeDemoBundle:Welcome:index`` maps to the ``indexAction`` method from the
``Acme\DemoBundle\Controller\WelcomeController`` class.

Logical Template Names
......................

For templates, the logical name ``AcmeDemoBundle:Welcome:index.html.twig`` is
converted to the file path ``src/Acme/DemoBundle/Resources/views/Welcome/index.html.twig``.
Templates become even more interesting when you realize they don't need to be
stored on the filesystem. You can easily store them in a database table for
instance.

Extending Bundles
.................

If you follow these conventions, then you can use :doc:`bundle inheritance</cookbook/bundles/inheritance>`
to "override" files, controllers or templates. For example, if a new bundle
called ``AcmeNewBundle`` extended the ``AcmeDemoBundle``, then Symfony would
try to load the ``AcmeDemoBundle:Welcome:index`` controller from ``AcmeNewBundle``
first, and then look inside ``AcmeDemoBundle`` second.

Do you understand now why Symfony2 is so flexible? Share your bundles between
applications, store them locally or globally, your choice.

.. _using-vendors:

Using Vendors
-------------

Скорее всего ваше приложение будет зависеть и от сторонних библиотек. Они должны
хранится в папке ``vendor/``. Она уже содержит библиотеки Symfony2,
библиотеку SwiftMailer, Doctrine ORM, систему шаблонизации Twig и
некоторые другие сторонние библиотеки.


Кэширование и Логи
------------------

Symfony2 is probably one of the fastest full-stack frameworks around. But how
can it be so fast if it parses and interprets tens of YAML and XML files for
each request? The speed is partly due to its cache system. The application
configuration is only parsed for the very first request and then compiled down
to plain PHP code stored in the ``app/cache/`` directory. In the development
environment, Symfony2 is smart enough to flush the cache when you change a
file. But in the production environment, it is your responsibility to clear
the cache when you update your code or change its configuration.

When developing a web application, things can go wrong in many ways. The log
files in the ``app/logs/`` directory tell you everything about the requests
and help you fix the problem quickly.

Интерфейс командной строки
--------------------------

Все приложения идут с интерфейсом командной строки (``app/console``),
который
помогает обслуживать приложение. Он предоставляет команды, которые увеличивают
вашу продуктивность, автоматизируя частые и повторяющиеся задачи.

Запустите консоль без агрументов, чтобы получить представление о её возможностях:

.. code-block:: bash

    php app/console

Опция ``--help`` поможет вам уточнить возможности использования команды:

.. code-block:: bash

    php app/console router:debug --help

Заключительное слово
--------------------

Называйте меня сумасшедшим, но после прочтения этой части, вам должно быть
комфортно перемещать любые вещи и при этом заставить Symfony2 работать на вас.
В Symfony2 всё сделано так, чтобы вы смогли настроить его на ваше усмотрение.
Так что, переименовывайте и перемещайте директории как вам угодно.

Для начала этого достаточно. Вам ещё предстоит многому научиться, от
тестирования до отправки почты, чтобы стать мастером Symfony2. Готовы
погрузиться в чтение сейчас? Следуйте на официальную страницу руководств :doc:`/book/index` и выбирайте любую тему.

.. _standards:               http://groups.google.com/group/php-standards/web/psr-0-final-proposal
.. _convention:              http://pear.php.net/
