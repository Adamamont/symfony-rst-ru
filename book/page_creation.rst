.. index::
   single: Создание страниц

Создание страниц в Symfony2
=============================

Создание новой страницы в Symfony2 это простой процесс, состоящий из 2 шагов:

* *Создание маршрута*: Маршрут определяет URI (например ``/about``) для вашей
  страницы, а также контроллер (PHP функция), который Symfony2 должен выполнить,
  когда URI входящего запроса совпадет шаблоном маршрута;

* *Создание контроллера*: Контроллер – это PHP функция, которая принимает входящий запрос и трансформирует его в объект ``Response``.

Нам нравится такой подход, потому что он соответствует тому как работает Web. Каждое взаимодействие в Web инициализируется HTTP запросом. Забота вашего приложения – интерпретировать запрос и вернуть соответствующий ответ.

Symfony2 следует этой философии и предлагает вам инструменты и соглашения, для того чтобы ваше приложение оставалось структурированным при росте его сложности.

Звучит просто? Давайте попробуем!

.. index::
   single: Создание страниц; Пример

Страница "Hello Symfony!"
----------------------------

Давайте начнем с классического приложения “Hello World!”. Когда мы закончим, пользователь будет иметь возможность получить персональное приветствие, перейдя по следующему URL:

.. code-block:: text

    http://localhost/app_dev.php/hello/Symfony

Вы также сможете заменить ``Symfony`` на другое имя и получить новое приветствие. Для создания этой страницы мы пройдем простой путь из двух шагов.

.. note::

    Данное руководство подразумевает, что вы уже скачали Symfony2 и настроили ваш
    вебсервер. URL, указанный выше, подразумевает, что ``localhost`` указывает на
    ``web``-директорию вашего нового Symfony2 проекта. Если же вы ещё не выполнили
    этих шагов, рекомендуется их выполнить, прежде чем вы продолжите читать. Для
    подробной информации прочтите :doc:`Установка Symfony2</book/installation>`.

Прежде чем вы начнете: Создание пакета (bundle)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Прежде чем начать, вам необходимо создать пакет (*bundle*). В
Symfony2 :term:`пакет` напоминает плагин, за исключением того, что весь
код вашего приложения будет расположен внутри такого пакета.

Вообще говоря, пакет – это не более чем директория (соответствующая тем не
менее пространству имен PHP), которая содержит все что относится к какой-то
специфической функции, включая PHP-классы, конфигурацию и даже стили и
Javascript-файлы (см. :ref:`page-creation-bundles`).

Для создания пакета с именем ``AcmeHelloBundle`` (демо-пакет, который мы
создадим в ходе прочтения данной статьи), выполните следующую команду и
следуйте инструкциям на экране (используйте настройки по-умолчанию):

.. code-block:: bash

    php app/console generate:bundle --namespace=Acme/HelloBundle --format=yml

Behind the scenes, a directory is created for the bundle at ``src/Acme/HelloBundle``.
A line is also automatically added to the ``app/AppKernel.php`` file so that
the bundle is registered with the kernel::

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            // ...
            new Acme\HelloBundle\AcmeHelloBundle(),
        );
        // ...

        return $bundles;
    }

Теперь, когда мы создали и подключили пакет, мы можем начать создание нашего приложения в нём.

Шаг 1: Создание маршрута
~~~~~~~~~~~~~~~~~~~~~~~~~~

По умолчанию, конфигурационный файл маршрутизатора в приложении Symfony2,
располагается в ``app/config/routing.yml``. Для конфигурирования
маршрутизатора, а также любых прочих конфигураций Symfony2, вы можете также
использовать XML или PHP формат.

If you look at the main routing file, you'll see that Symfony already added
an entry when you generated the ``AcmeHelloBundle``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        AcmeHelloBundle:
            resource: "@AcmeHelloBundle/Resources/config/routing.yml"
            prefix:   /

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <import resource="@AcmeHelloBundle/Resources/config/routing.xml" prefix="/" />
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->addCollection(
            $loader->import('@AcmeHelloBundle/Resources/config/routing.php'),
            '/',
        );

        return $collection;

This entry is pretty basic: it tells Symfony to load routing configuration
from the ``Resources/config/routing.yml`` file that lives inside the ``AcmeHelloBundle``.
This means that you place routing configuration directly in ``app/config/routing.yml``
or organize your routes throughout your application, and import them from here.

Now that the ``routing.yml`` file from the bundle is being imported, add
the new route that defines the URL of the page that you're about to create:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/routing.yml
        hello:
            pattern:  /hello/{name}
            defaults: { _controller: AcmeHelloBundle:Hello:index }

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="hello" pattern="/hello/{name}">
                <default key="_controller">AcmeHelloBundle:Hello:index</default>
            </route>
        </routes>

    .. code-block:: php

        // src/Acme/HelloBundle/Resources/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('hello', new Route('/hello/{name}', array(
            '_controller' => 'AcmeHelloBundle:Hello:index',
        )));

        return $collection;

Маршрут состоит из двух основных частей: ``pattern``, с которым сравнивается
URI, а также массив ``defaults`` в котором указывается контроллер, который
необходимо выполнить. Синтаксис указателя места заполнения (placeholder) в
шаблоне (``{name}``) – это групповой символ (wildcard). Он означает, что URI
``/hello/Ryan``, ``/hello/Fabien``, а также прочие, походие на них, будут
соответствовать этому маршруту. Параметр, определённый указателем ``{name}``
также будет передан в наш контроллер, так что мы сможем использовать его,
чтобы поприветствовать пользователя.

.. note::

  Система маршрутизации имеет еще множество замечательных функций для создания
  гибких и функциональных структур URI в нашем приложении. За дополнительной
  информацией вы можете обратиться к главе :doc:`Маршрутизация </book/routing>`.

Шаг 2: Создание Контроллера
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Когда URI вида ``/hello/Ryan`` обнаруживается приложением в запросе, маршрут
``hello`` сработает и будет вызван контроллер ``AcmeHelloBundle:Hello:index``.
Следующим нашим шагом будет создание этого контроллера.

The controller - ``AcmeHelloBundle:Hello:index`` is the *logical* name of
the controller, and it maps to the ``indexAction`` method of a PHP class
called ``Acme\HelloBundle\Controller\Hello``. Start by creating this file
inside your ``AcmeHelloBundle``::

    // src/Acme/HelloBundle/Controller/HelloController.php
    namespace Acme\HelloBundle\Controller;

    use Symfony\Component\HttpFoundation\Response;

    class HelloController
    {
    }

В действительности, контроллер – это не что иное, как метод PHP класса,
который мы создаем, а Symfony выполняет. Это то место, где приложение,
используя информацию из запроса, создает запрошенный ресурс. За исключением
некоторых особых случаев, результатом работы контроллера всегда является
объект Symfony2 ``Response``.

Create the ``indexAction`` method that Symfony will execute when the ``hello``
route is matched::

    // src/Acme/HelloBundle/Controller/HelloController.php

    // ...
    class HelloController
    {
        public function indexAction($name)
        {
            return new Response('<html><body>Hello '.$name.'!</body></html>');
        }
    }

Этот контроллер предельно прост: он создает новый объект ``Response`` чьим
первым аргументом является контент, который будет использован для создания
ответа (в нашем случае это маленькая HTML-страница, код которой мы указали
прямо в контроллере).

Примите мои поздравления! После создания маршрута и контроллера, вы уже имеете
полноценную страницу! Если вы все настроили корректно, ваше приложение должно
поприветствовать вас:

.. code-block:: text

    http://localhost/app_dev.php/hello/Ryan

Опциональным (но как правило востребованным) третьим шагом является создание
шаблона.

.. note::

   Контроллер – это главная точка входа для вашего кода и ключевой ингридиет
   при создании страниц. Больше информации о контроллерах вы можете найти в
   главе :doc:`Контроллеры</book/controller>`.

Необязательный шаг 3: Создание шаблона
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Шаблоны позволяют нам вынести разметку страниц (HTML код как вравило) в
отдельный файл и повторно использовать различные части шаблона страницы.
Вместо того чтобы писать код внутри контроллера, воспользуемся шаблоном:

.. code-block:: php
    :linenos:

    // src/Acme/HelloBundle/Controller/HelloController.php
    namespace Acme\HelloBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class HelloController extends Controller
    {
        public function indexAction($name)
        {
            return $this->render('AcmeHelloBundle:Hello:index.html.twig', array('name' => $name));

            // render a PHP template instead
            // return $this->render('AcmeHelloBundle:Hello:index.html.php', array('name' => $name));
        }
    }

.. note::

   Для того, чтобы использовать метод ``render()`` необходимо отнаследоваться
   от класса ``Symfony\Bundle\FrameworkBundle\Controller\Controller``
   (API :class:`Symfony\\Bundle\\FrameworkBundle\\Controller\\Controller`),
   который добавляет несколько методов для быстрого вызова для часто
   употребляемых функций в контроллере. В примере выше это сделано добавлением
   ``use`` на линии 4 и расширением класса ``Controller`` на линии 6.

Метод ``render()`` создает объект ``Response`` заполненный содержанием
обработанного (рендереного) шаблона. Как и любой другой контроллер, вы в конце
концов вернете объект ``Response``.

Обратите внимание, что есть две различные возможности рендеринга шаблонов.
Symfony2 по-умолчанию, поддерживает 2 языка шаблонов: классические PHP-шаблоны
и простой, но мощный язык шаблонов `Twig`_. Но не пугайтесь, вы свободны в
выборе того или иного из них, кроме того вы можете использовать оба в рамках
одного проекта.

Контроллер отображает шаблон ``AcmeHelloBundle:Hello:index.html.twig``,
который использует следующие соглашения:

    **BundleName**:**ControllerName**:**TemplateName**

This is the *logical* name of the template, which is mapped to a physical
location using the following convention.

    **/path/to/BundleName**/Resources/views/**ControllerName**/**TemplateName**

Таким образом, ``AcmeHelloBundle`` – это имя пакета, ``Hello`` – это контроллер и ``index.html.twig`` это шаблон:

.. configuration-block::

    .. code-block:: jinja
       :linenos:

        {# src/Acme/HelloBundle/Resources/views/Hello/index.html.twig #}
        {% extends '::base.html.twig' %}

        {% block body %}
            Hello {{ name }}!
        {% endblock %}

    .. code-block:: php

        <!-- src/Acme/HelloBundle/Resources/views/Hello/index.html.php -->
        <?php $view->extend('::base.html.php') ?>

        Hello <?php echo $view->escape($name) ?>!

Давайте рассмотрим подробнее шаблон Twig:

* *Строка 2*: Токен ``extends`` определяет родительский шаблон. Таким образом
  сам шаблон однозначным образом определяет родителя (layout) внутрь которого
  он будет помещен.

* *Строка 4*: Токен ``block`` означает, что все внутри него будет помещено в
  блок с именем ``body``. Как мы увидим ниже, это уже обязанность
  родительского шаблона (``base.html.twig``) полностью отобразить блок
  ``body``.

The parent template, ``::base.html.twig``, is missing both the **BundleName**
and **ControllerName** portions of its name (hence the double colon (``::``)
at the beginning). This means that the template lives outside of the bundles
and in the ``app`` directory:

.. configuration-block::

    .. code-block:: html+jinja

        {# app/Resources/views/base.html.twig #}
        <!DOCTYPE html>
        <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
                <title>{% block title %}Welcome!{% endblock %}</title>
                {% block stylesheets %}{% endblock %}
                <link rel="shortcut icon" href="{{ asset('favicon.ico') }}" />
            </head>
            <body>
                {% block body %}{% endblock %}
                {% block javascripts %}{% endblock %}
            </body>
        </html>

    .. code-block:: php

        <!-- app/Resources/views/base.html.php -->
        <!DOCTYPE html>
        <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
                <title><?php $view['slots']->output('title', 'Welcome!') ?></title>
                <?php $view['slots']->output('stylesheets') ?>
                <link rel="shortcut icon" href="<?php echo $view['assets']->getUrl('favicon.ico') ?>" />
            </head>
            <body>
                <?php $view['slots']->output('_content') ?>
                <?php $view['slots']->output('stylesheets') ?>
            </body>
        </html>

Базовый шаблон определяет HTML разметку блока ``body`` который мы определили
в шаблоне ``index.html.twig``. Он также отображает блок ``title``
который мы также можем определить в ``index.html.twig``. Так как мы не
определили блок ``title`` в дочернем шаблоне, он примет значение по умолчанию
– “Hello Application”.

Шаблоны являются мощным инструментом по организации и отображению контента
ваших страниц – HTML разметки, CSS стилей, а также всего прочего, что может
потребоваться вернуть контроллеру.

Но шаблонизатор – это просто средство для достижения цели. А цель состоит в
том, чтобы каждый контроллер возвращал объект ``Response``. Таким образом,
шаблоны мощный, но опциональный инструмент для создания контента для объекта
``Response``.

.. index::
   single: Структура директорий

Структура директорий
-----------------------

Мы прочитали всего лишь после нескольких коротких секций, а вы уже уяснили
философию создания и отображения страниц в Symfony2. Поэтому без лишних слов
мы приступим к изучению того, как организованы и структурированы проекты
Symfony2. К концу этой секции вы будете знать где найти и куда поместить
различные типы файлов. И более того, будет понимать – почему!

Изначально созданный очень гибким, по умолчанию каждое
Symfony :term:`приложение` имеет одну и ту же базовую (и рекомендуемую)
структуру директорий:

* ``app/``: This directory contains the application configuration;

* ``src/``: All the project PHP code is stored under this directory;

* ``vendor/``: Any vendor libraries are placed here by convention;

* ``web/``: This is the web root directory and contains any publicly accessible files;

The Web Directory
~~~~~~~~~~~~~~~~~

The web root directory is the home of all public and static files including
images, stylesheets, and JavaScript files. It is also where each
:term:`front controller` lives::

    // web/app.php
    require_once __DIR__.'/../app/bootstrap.php.cache';
    require_once __DIR__.'/../app/AppKernel.php';

    use Symfony\Component\HttpFoundation\Request;

    $kernel = new AppKernel('prod', false);
    $kernel->loadClassCache();
    $kernel->handle(Request::createFromGlobals())->send();

The front controller file (``app.php`` in this example) is the actual PHP
file that's executed when using a Symfony2 application and its job is to
use a Kernel class, ``AppKernel``, to bootstrap the application.

.. tip::

    Having a front controller means different and more flexible URLs than
    are used in a typical flat PHP application. When using a front controller,
    URLs are formatted in the following way:

    .. code-block:: text

        http://localhost/app.php/hello/Ryan

    The front controller, ``app.php``, is executed and the "internal:" URL
    ``/hello/Ryan`` is routed internally using the routing configuration.
    By using Apache ``mod_rewrite`` rules, you can force the ``app.php`` file
    to be executed without needing to specify it in the URL:

    .. code-block:: text

        http://localhost/hello/Ryan

Though front controllers are essential in handling every request, you'll
rarely need to modify or even think about them. We'll mention them again
briefly in the `Environments`_ section.

The Application (``app``) Directory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As you saw in the front controller, the ``AppKernel`` class is the main entry
point of the application and is responsible for all configuration. As such,
it is stored in the ``app/`` directory.

This class must implement two methods that define everything that Symfony
needs to know about your application. You don't even need to worry about
these methods when starting - Symfony fills them in for you with sensible
defaults.

* ``registerBundles()``: Returns an array of all bundles needed to run the
  application (see :ref:`page-creation-bundles`);

* ``registerContainerConfiguration()``: Loads the main application configuration
  resource file (see the `Application Configuration`_ section).

In day-to-day development, you'll mostly use the ``app/`` directory to modify
configuration and routing files in the ``app/config/`` directory (see
`Application Configuration`_). It also contains the application cache
directory (``app/cache``), a log directory (``app/logs``) and a directory
for application-level resource files, such as templates (``app/Resources``).
You'll learn more about each of these directories in later chapters.

.. _autoloading-introduction-sidebar:

.. sidebar:: Autoloading

    When Symfony is loading, a special file - ``app/autoload.php`` - is included.
    This file is responsible for configuring the autoloader, which will autoload
    your application files from the ``src/`` directory and third-party libraries
    from the ``vendor/`` directory.

    Because of the autoloader, you never need to worry about using ``include``
    or ``require`` statements. Instead, Symfony2 uses the namespace of a class
    to determine its location and automatically includes the file on your
    behalf the instant you need a class.

    The autoloader is already configured to look in the ``src/`` directory
    for any of your PHP classes. For autoloading to work, the class name and
    path to the file have to follow the same pattern:

    .. code-block:: text

        Class Name:
            Acme\HelloBundle\Controller\HelloController
        Path:
            src/Acme/HelloBundle/Controller/HelloController.php

    Typically, the only time you'll need to worry about the ``app/autoload.php``
    file is when you're including a new third-party library in the ``vendor/``
    directory. For more information on autoloading, see
    :doc:`How to autoload Classes</cookbook/tools/autoloader>`.

The Source (``src``) Directory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Put simply, the ``src/`` directory contains all of the actual code (PHP code,
templates, configuration files, stylesheets, etc) that drives *your* application.
When developing, the vast majority of your work will be done inside one or
more bundles that you create in this directory.

But what exactly is a :term:`пакет`?

.. _page-creation-bundles:

Система пакетов
-----------------

Пакет чем-то схож с плагином, но он ещё лучше. Ключевое отличие состоит в
том, что *все* есть пакет в Symfony2, включая функционал ядра и код вашего
приложения. Пакеты – это граждане высшего сорта в Symfony2. Они дают вам
возможность использовать уже готовые пакеты, которые вы можете найти по адресу
`third-party bundles`_.Вы также можете там выкладывать свои пакеты. Они также
дают возможность легко и просто выбрать, какие именно функции подключить в
вашем приложении.

.. note::

   Здесь мы рассмотрим лишь основы, более детальную информацию по пакетам вы
   можете найти в главе :doc:`пакеты</cookbook/bundles/best_practices>`.

A bundle is simply a structured set of files within a directory that implement
a single feature. You might create a ``BlogBundle``, a ``ForumBundle`` or
a bundle for user management (many of these exist already as open source
bundles). Each directory contains everything related to that feature, including
PHP files, templates, stylesheets, JavaScripts, tests and anything else.
Every aspect of a feature exists in a bundle and every feature lives in a
bundle.

An application is made up of bundles as defined in the ``registerBundles()``
method of the ``AppKernel`` class::

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

With the ``registerBundles()`` method, you have total control over which bundles
are used by your application (including the core Symfony bundles).

.. tip::

   A bundle can live *anywhere* as long as it can be autoloaded (via the
   autoloader configured at ``app/autoload.php``).

Creating a Bundle
~~~~~~~~~~~~~~~~~

The Symfony Standard Edition comes with a handy task that creates a fully-functional
bundle for you. Of course, creating a bundle by hand is pretty easy as well.

To show you how simple the bundle system is, create a new bundle called
``AcmeTestBundle`` and enable it.

.. tip::

    The ``Acme`` portion is just a dummy name that should be replaced by
    some "vendor" name that represents you or your organization (e.g. ``ABCTestBundle``
    for some company named ``ABC``).

Start by creating a ``src/Acme/TestBundle/`` directory and adding a new file
called ``AcmeTestBundle.php``::

    // src/Acme/TestBundle/AcmeTestBundle.php
    namespace Acme\TestBundle;

    use Symfony\Component\HttpKernel\Bundle\Bundle;

    class AcmeTestBundle extends Bundle
    {
    }

.. tip::

   The name ``AcmeTestBundle`` follows the standard :ref:`Bundle naming conventions<bundles-naming-conventions>`.
   You could also choose to shorten the name of the bundle to simply ``TestBundle``
   by naming this class ``TestBundle`` (and naming the file ``TestBundle.php``).

This empty class is the only piece you need to create the new bundle. Though
commonly empty, this class is powerful and can be used to customize the behavior
of the bundle.

Now that you've created the bundle, enable it via the ``AppKernel`` class::

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            // ...

            // register your bundles
            new Acme\TestBundle\AcmeTestBundle(),
        );
        // ...

        return $bundles;
    }

And while it doesn't do anything yet, ``AcmeTestBundle`` is now ready to
be used.

And as easy as this is, Symfony also provides a command-line interface for
generating a basic bundle skeleton:

.. code-block:: bash

    php app/console generate:bundle --namespace=Acme/TestBundle

The bundle skeleton generates with a basic controller, template and routing
resource that can be customized. You'll learn more about Symfony2's command-line
tools later.

.. tip::

   Whenever creating a new bundle or using a third-party bundle, always make
   sure the bundle has been enabled in ``registerBundles()``. When using
   the ``generate:bundle`` command, this is done for you.

Bundle Directory Structure
~~~~~~~~~~~~~~~~~~~~~~~~~~

The directory structure of a bundle is simple and flexible. By default, the
bundle system follows a set of conventions that help to keep code consistent
between all Symfony2 bundles. Take a look at ``AcmeHelloBundle``, as it contains
some of the most common elements of a bundle:

* ``Controller/`` contains the controllers of the bundle (e.g. ``HelloController.php``);

* ``Resources/config/`` houses configuration, including routing configuration
  (e.g. ``routing.yml``);

* ``Resources/views/`` holds templates organized by controller name (e.g.
  ``Hello/index.html.twig``);

* ``Resources/public/`` contains web assets (images, stylesheets, etc) and is
  copied or symbolically linked into the project ``web/`` directory via
  the ``assets:install`` console command;

* ``Tests/`` holds all tests for the bundle.

A bundle can be as small or large as the feature it implements. It contains
only the files you need and nothing else.

As you move through the book, you'll learn how to persist objects to a database,
create and validate forms, create translations for your application, write
tests and much more. Each of these has their own place and role within the
bundle.

Application Configuration
-------------------------

An application consists of a collection of bundles representing all of the
features and capabilities of your application. Each bundle can be customized
via configuration files written in YAML, XML or PHP. By default, the main
configuration file lives in the ``app/config/`` directory and is called
either ``config.yml``, ``config.xml`` or ``config.php`` depending on which
format you prefer:

.. configuration-block::

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

        # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <imports>
            <import resource="parameters.ini" />
            <import resource="security.yml" />
        </imports>
        
        <framework:config charset="UTF-8" secret="%secret%">
            <framework:router resource="%kernel.root_dir%/config/routing.xml" />
            <framework:form />
            <framework:csrf-protection />
            <framework:validation annotations="true" />
            <framework:templating assets-version="SomeVersionScheme">
                <framework:engine id="twig" />
            </framework:templating>
            <framework:session default-locale="%locale%" auto-start="true" />
        </framework:config>

        <!-- Twig Configuration -->
        <twig:config debug="%kernel.debug%" strict-variables="%kernel.debug%" />

        <!-- ... -->

    .. code-block:: php

        $this->import('parameters.ini');
        $this->import('security.yml');

        $container->loadFromExtension('framework', array(
            'secret'          => '%secret%',
            'charset'         => 'UTF-8',
            'router'          => array('resource' => '%kernel.root_dir%/config/routing.php'),
            'form'            => array(),
            'csrf-protection' => array(),
            'validation'      => array('annotations' => true),
            'templating'      => array(
                'engines' => array('twig'),
                #'assets_version' => "SomeVersionScheme",
            ),
            'session' => array(
                'default_locale' => "%locale%",
                'auto_start'     => true,
            ),
        ));

        // Twig Configuration
        $container->loadFromExtension('twig', array(
            'debug'            => '%kernel.debug%',
            'strict_variables' => '%kernel.debug%',
        ));

        // ...

.. note::

   You'll learn exactly how to load each file/format in the next section
   `Environments`_.

Each top-level entry like ``framework`` or ``twig`` defines the configuration
for a particular bundle. For example, the ``framework`` key defines the configuration
for the core Symfony ``FrameworkBundle`` and includes configuration for the
routing, templating, and other core systems.

For now, don't worry about the specific configuration options in each section.
The configuration file ships with sensible defaults. As you read more and
explore each part of Symfony2, you'll learn about the specific configuration
options of each feature.

.. sidebar:: Configuration Formats

    Throughout the chapters, all configuration examples will be shown in all
    three formats (YAML, XML and PHP). Each has its own advantages and
    disadvantages. The choice of which to use is up to you:

    * *YAML*: Simple, clean and readable;

    * *XML*: More powerful than YAML at times and supports IDE autocompletion;

    * *PHP*: Very powerful but less readable than standard configuration formats.

.. index::
   single: Environments; Introduction

.. _environments-summary:

Environments
------------

An application can run in various environments. The different environments
share the same PHP code (apart from the front controller), but use different
configuration. For instance, a ``dev`` environment will log warnings and
errors, while a ``prod`` environment will only log errors. Some files are
rebuilt on each request in the ``dev`` environment (for the developer's convenience),
but cached in the ``prod`` environment. All environments live together on
the same machine and execute the same application.

A Symfony2 project generally begins with three environments (``dev``, ``test``
and ``prod``), though creating new environments is easy. You can view your
application in different environments simply by changing the front controller
in your browser. To see the application in the ``dev`` environment, access
the application via the development front controller:

.. code-block:: text

    http://localhost/app_dev.php/hello/Ryan

If you'd like to see how your application will behave in the production environment,
call the ``prod`` front controller instead:

.. code-block:: text

    http://localhost/app.php/hello/Ryan

.. note::

   If you open the ``web/app.php`` file, you'll find that it's configured explicitly
   to use the ``prod`` environment::

       $kernel = new AppKernel('prod', false);

   You can create a new front controller for a new environment by copying
   this file and changing ``prod`` to some other value.

Since the ``prod`` environment is optimized for speed; the configuration,
routing and Twig templates are compiled into flat PHP classes and cached.
When viewing changes in the ``prod`` environment, you'll need to clear these
cached files and allow them to rebuild::

    php app/console cache:clear --env=prod

.. note::

    The ``test`` environment is used when running automated tests and cannot
    be accessed directly through the browser. See the :doc:`testing chapter</book/testing>`
    for more details.

.. index::
   single: Environments; Configuration

Environment Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~

The ``AppKernel`` class is responsible for actually loading the configuration
file of your choice::

    // app/AppKernel.php
    public function registerContainerConfiguration(LoaderInterface $loader)
    {
        $loader->load(__DIR__.'/config/config_'.$this->getEnvironment().'.yml');
    }

You already know that the ``.yml`` extension can be changed to ``.xml`` or
``.php`` if you prefer to use either XML or PHP to write your configuration.
Notice also that each environment loads its own configuration file. Consider
the configuration file for the ``dev`` environment.

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_dev.yml
        imports:
            - { resource: config.yml }

        framework:
            router:   { resource: "%kernel.root_dir%/config/routing_dev.yml" }
            profiler: { only_exceptions: false }

        # ...

    .. code-block:: xml

        <!-- app/config/config_dev.xml -->
        <imports>
            <import resource="config.xml" />
        </imports>

        <framework:config>
            <framework:router resource="%kernel.root_dir%/config/routing_dev.xml" />
            <framework:profiler only-exceptions="false" />
        </framework:config>

        <!-- ... -->

    .. code-block:: php

        // app/config/config_dev.php
        $loader->import('config.php');

        $container->loadFromExtension('framework', array(
            'router'   => array('resource' => '%kernel.root_dir%/config/routing_dev.php'),
            'profiler' => array('only-exceptions' => false),
        ));

        // ...

The ``imports`` key is similar to a PHP ``include`` statement and guarantees
that the main configuration file (``config.yml``) is loaded first. The rest
of the file tweaks the default configuration for increased logging and other
settings conducive to a development environment.

Both the ``prod`` and ``test`` environments follow the same model: each environment
imports the base configuration file and then modifies its configuration values
to fit the needs of the specific environment. This is just a convention,
but one that allows you to reuse most of your configuration and customize
just pieces of it between environments.

Заключение
------------

Поздравляем! Вы усвоили все фундаментальные аспекты Symfony2 и обнаружили, какими лёгкими и в то же время гибкими они могут быть. И, поскольку на подходе ещё *много* интересного, обязательно запомните следующие положения:

* Создание страниц – это три простых шага, включающих **маршрут**,
  **контроллер** и (опционально) **шаблон**.

* Каждое приложение должно состоять только из 4х директорий: ``web/`` (web
  assets и front controllers), ``app/`` (настройки), ``src/`` (ваши пакеты),
  и ``vendor/`` (сторонние библиотеки);

* Каждая функция в Symfony2 (включая ядро фреймворка) должна располагаться
  внутри *пакета*, который представляет собой структурированный набор файлов,
  реализующих эту функцию;

* **настройки** каждого пакета располагаются в директории ``app/config`` и могут
  быть записаны в формате YAML, XML или PHP;

* каждое **окружение** доступно через свой отдельный фронт-контроллер
  (например ``app.php`` и ``app_dev.php``) и загружает отдельный файл настроек.

Далее, каждая глава книги познакомит вас с все более и более мощными инструментами и более глубокими концепциями. Чем больше вы знаете о Symfony2, тем больше вы будете ценить гибкость его архитектуры и его обширные возможности для быстрой разработки приложений.

.. _`Twig`: http://www.twig-project.org
.. _`third-party bundles`: http://symfony2bundles.org/
.. _`Symfony Standard Edition`: http://symfony.com/download
