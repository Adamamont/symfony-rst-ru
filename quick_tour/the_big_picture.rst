Общая картина
=============

Итак вы хотите попробовать Symfony2, но в наличии у вас не более 10 минут?
Первая часть этого учебника написана для вас. Она объяснит как быстро начать
с Symfony2, показав структуру простого готового проекта.

Если вы когда-нибудь использовали какой-либо веб-фреймворк прежде, вы будете
чувствовать себя в Symfony2 как дома.

.. tip::

    Хотите узнать зачем и почему стоит использовать фреймворк?
    Прочтите "`Symfony за 5 минут`_".

Загрузка Symfony2
-----------------

В первую очередь, убедитесь что у вас установлен как минимум PHP 5.3.2 и он
настроен для работы с web сервером, таким как Apache.

Готовы? Давайте начнем с загрузки "`Symfony2 Standard Edition`_", 
дистрибутива (:term:`distribution`) Symfony настроенного для большинства 
потребностей, а так же содержащий код, показывающий как использовать 
Symfony2 (загрузите архив с бибилотеками (*vendors*) чтобы начать 
быстрее).

После распаковки архива в корневую директорию веб-сервера, вы должны 
получить папку ``Symfony/``, в которой содержится следующее:

.. code-block:: text

    www/ <- ваш корневой каталог
        Symfony/ <- распакованных архив
            app/
                cache/
                config/
                logs/
                Resources/
            bin/
            src/
                Acme/
                    DemoBundle/
                        Controller/
                        Resources/
                        ...
            vendor/
                symfony/
                doctrine/
                ...
            web/
                app.php
                ...

.. note::

    Если вы загрузили стандартное издание без бибилотек (*without 
    vendors*), просто запустите команду чтобы получить все библиотеки:

    .. code-block:: bash

        php bin/vendors install

Проверка конфигурации
---------------------

Для того чтобы избежать головной боли в будущем, проверьте, сможет ли ваша
система запустить Symfony2 без проблем – для этого откройте следующий URL:

.. code-block:: text

    http://localhost/Symfony/web/config.php

Если в списке есть ошибки - исправьте их. Вы так же можете настроить 
сервер, следуя рекомендациям. Когда все будет исправлено, кликните на 
"*Bypass configuration and go to the Welcome page*" чтобы перейти на вашу 
первую "реальную" страницу на Symfony2:

.. code-block:: text

    http://localhost/Symfony/web/app_dev.php/

Symfony2 должен поблагодарить вас за приложенные усилия!

.. image:: /images/quick_tour/welcome.jpg
   :align: center

Понимание основ
---------------

Одна из главных целей фреймворка - следовать концепции `разделения ответственности`_.
Это делает ваш код организованным и позволяет эволюционировать 
приложению, избегая смеси запросов к базе, HTML-тэгов и бизнес-логики в 
одном скрипте. Чтобы достичь эту цель с Symfony2 вы должны узнать 
несколько фундаментальных концепций и терминов.

.. tip::

    Хотите доказательств, что использование фреймворка лучше чем смеси 
    всего в одном скрипте? Прочтите главу ":doc:`/book/from_flat_php_to_symfony2`"

Базовое издание идет с примером кода, что позволяет узнать вам больше о 
главных концепциях Symfony2. Перейдите по следующему URL, чтобы Symfony2 
поприветствовала вас (замените *Fabien* своим именем):

.. code-block:: text

    http://localhost/Symfony/web/app_dev.php/demo/hello/Fabien

.. image:: /images/quick_tour/hello_fabien.png
   :align: center

Что происходит в этом месте? Давайте разберём URL:

* ``app_dev.php``: Это :term:`front controller`. Уникальная точка входа для приложения,
  которая отвечает на все запросы пользователя;

* ``/demo/hello/Fabien``: Это *виртуальный путь* ресурса, к которому пользователь
  хочет получить доступ.

От вас как от разработчика требуется написать код, который сопоставит
пользовательский *запрос* (``/hello/Fabien``) и ассоциированный с ним ресурс
(Страница ``Hello Fabien!``).

Маршрутизация
~~~~~~~~~~~~~

Symfony2 направляет запрос на код, который сравнивает текущий URL с 
настроенными шаблонами. По умолчанию шаблоны (называемые маршрутами) 
задаются в файле ``app/config/routing.yml``. Если вы в ``dev``
:ref:`окружении<quick-tour-big-picture-environments>` - на это указывает
front-контроллер app_**dev**.php - файл ``app/config/routing_dev.yml``
так же будет загружен. В стандартной поставке, маршруты "demo"-страниц
указываются в этом файле:

.. code-block:: yaml

    # app/config/routing_dev.yml
    _welcome:
        pattern:  /
        defaults: { _controller: AcmeDemoBundle:Welcome:index }

    _demo:
        resource: "@AcmeDemoBundle/Controller/DemoController.php"
        type:     annotation
        prefix:   /demo

    # ...

Три первые линии (после комментария) задают код, который будет запущен 
при запросе ресурса "``/``" (т.е. страницы приветствия). После запроса 
будет запущен контроллер ``AcmeDemoBundle:Welcome:index``.

.. tip::

    В стандартной поставке Symfony2 использует `YAML`_ для файлов 
    конфигурации, но Symfony2 так же поддерживает XML, PHP и аннотации 
    "из коробки". Различные форматы совместимы и могут быть 
    взаимозаменяемы внутри приложения. Так же быстродействие вашего 
    приложения не зависит от формата конфигурации, который вы выберете - 
    все будет закешировано при первом запросе.

Контроллеры
~~~~~~~~~~~

Контроллер обрабатывает входящий *запрос* и возвращает *ответ* (чаще 
всего HTML-код). Вместо использования глобальных переменных и функций 
(таких как ``$_GET`` или ``header()``) для управления HTTP-сообщениями 
Symfony использует объекты :class:`Symfony\\Component\\HttpFoundation\\Request`
и :class:`Symfony\\Component\\HttpFoundation\\Response`. Простейший 
контроллер, который создает ответ на базе запроса::

    use Symfony\Component\HttpFoundation\Response;

    $name = $request->query->get('name');

    return new Response('Hello '.$name, 200, array('Content-Type' => 'text/plain'));

.. note::

    Symfony2 использует спецефикации протокола HTTP, которые являются
    правилами для всех коммуникаций в Web. Прочтите главу ":doc:`/book/http_fundamentals`"
    чтобы больше узнать об этом.

Symfony2 выбирает контроллер базируясь на значении ``_controller`` из 
конфигурации маршрутизации: ``AcmeDemoBundle:Welcome:index``. Это - 
*логическое* имя и оно указывает на метод ``indexAction`` класса ``Acme\DemoBundle\Controller\WelcomeController``::

    // src/Acme/DemoBundle/Controller/WelcomeController.php
    namespace Acme\DemoBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class WelcomeController extends Controller
    {
        public function indexAction()
        {
            return $this->render('AcmeDemoBundle:Welcome:index.html.twig');
        }
    }

.. tip::

    Вы могли бы использовать ``Acme\DemoBundle\Controller\WelcomeController::indexAction`` для значения ``_controller``, но если 
    следовать простым соглашениям логическое имя может быть более простым и 
    более гибким.

Класс ``WelcomeController`` расширяет встроенный класс ``Controller`` 
который представляет удобные методы, такой как
:method:`Symfony\\Bundle\\FrameworkBundle\\Controller\\Controller::render`
который загружает и отображает шаблон 
(``AcmeDemoBundle:Welcome:index.html.twig``). Возвращаемое значение - это 
объект Response, наполненный отображаемым контентом. Так, если вам нужно, 
Response может быть настроен до отправки браузеру::

    public function indexAction()
    {
        $response = $this->render('AcmeDemoBundle:Welcome:index.txt.twig');
        $response->headers->set('Content-Type', 'text/plain');

        return $response;
    }

Не важно как вы это сделаете, конечная цель в том, что контроллер всегда
возращает объект ``Response``, который должен быть возращен пользователю обратно.
Этот объект может быть наполнен HTML-кодом, представлять из себя 
перенаправление на другую страницу или возращать содержимое JPG-
изображения
с заголовком ``Content-Type`` ``image/jpg``.
.. tip::

    Расширять класс ``Controller`` не обязательно. В сущности, контроллер 
    может быть функцией на плоском PHP или даже PHP-замыканием. Глава 
    ":doc:`Контроллер</book/controller>`" расскажет вам все о 
    контроллерах Symfony2.

Имя шаблона ``AcmeDemoBundle:Welcome:index.html.twig`` - это *логическое* 
имя и оно указывает на файл ``Resources/views/Welcome/index.html.twig`` 
внутри пакета ``AcmeDemoBundle``
(расположенного в ``src/Acme/DemoBundle``). Глава о пакетах расскажет вам 
почему это удобно.

А сейчас, давайте снова взглянем на конфигурацию маршрутизации:

.. code-block:: yaml

    # app/config/routing_dev.yml
    _demo:
        resource: "@AcmeDemoBundle/Controller/DemoController.php"
        type:     annotation
        prefix:   /demo

Symfony2 может читать информацию о маршрутизации в форматах YAML, XML, 
PHP или даже встроенных в PHP аннотаций. Здесь *логическое имя*
``@AcmeDemoBundle/Controller/DemoController.php`` ссылается на файл ``src/
Acme/DemoBundle/Controller/DemoController.php``. В этом файле маршруты 
заданы как аннотации к методам::

    // src/Acme/DemoBundle/Controller/DemoController.php
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

    class DemoController extends Controller
    {
        /**
         * @Route("/hello/{name}", name="_demo_hello")
         * @Template()
         */
        public function helloAction($name)
        {
            return array('name' => $name);
        }

        // ...
    }

Аннотация ``@Route()`` задает новый маршрут с шаблоном ``/hello/{name}``, 
который запускает метод ``helloAction`` при совпадении. Строка, обернутая 
в фигурные скобки, такая как ``{name}`` называется placeholder. Как вы 
можете видеть ее значение доступно через аргумент ``$name``.

.. note::

    Даже если аннотации не поддерживаются PHP, вы можете их широко
    использовать в Symfony2 как удобный способ хранить настройки рядом с кодом.

Если вы внимательно посмотрите на код действия, то сможете увидеть, что 
вместо вывода шаблона как раньше, теперь мы просто возвращаем массив 
параметров. Аннотация ``@Template()`` говорит Symfony2 отобразить шаблон, 
передавая каждый параметр массива в шаблон. Название шаблона следует из 
имени контроллера. Так, в этом примере, отобразится шаблон 
``AcmeDemoBundle:Demo:hello.html.twig`` (расположенный в ``src/Acme/DemoBundle/Resources/views/Demo/hello.html.twig``).

.. tip::

    Аннотации ``@Route()`` и ``@Template()`` более мощные чем показано в 
    этом примере. Узнайте больше о  "`аннотациях в контроллерах`_" в 
    официальной документации.

Шаблоны
~~~~~~~

Контроллер отображает шаблон ``src/Acme/DemoBundle/Resources/views/Demo/hello.html.twig`` (или ``AcmeDemoBundle:Demo:hello.html.twig`` если вы 
предпочитаете логические имена):

.. code-block:: jinja

    {# src/Acme/DemoBundle/Resources/views/Demo/hello.html.twig #}
    {% extends "AcmeDemoBundle::layout.html.twig" %}

    {% block title "Hello " ~ name %}

    {% block content %}
        <h1>Hello {{ name }}!</h1>
    {% endblock %}

По умолчанию, Symfony2 использует `Twig`_ в качестве шаблнизатора, но вы 
так же можете использовать обычный PHP если вам так больше нравится. В 
следующих главах мы поговорим о том как шаблоны работают в Symfony2.

Пакеты (bundles)
~~~~~~~~~~~~~~~~

Вы должно быть удивлены, тем что видите слово :term:`пакет` так часто. 
Весь код вашего приложения находится в пакетах. В терминологии Symfony2 
пакет представляет собой структурированный набор файлов (файлы PHP, 
стили, JavaScript'ы, картинки, ...) которые выполняют одну функцию (блог, 
форум, ...) и которыми можно легко поделиться с другими разработчиками. 
Пока мы работали только с одним пакетом - ``AcmeDemoBundle``. Вы узнаете 
больше о пакетах в последней главе данного урока.

.. _quick-tour-big-picture-environments:

Работа с окружениями
--------------------

Сейчас, когда вы имеете лучшее понимание работы Symfony2, обратите 
внимание на нижнию часть страницы; вы увидите маленькую панель с 
логотипом Symfony2. Она называется "Веб-панелью отладки" ("Web Debug 
Toolbar") и это лучший друг разработчика.

.. image:: /images/quick_tour/web_debug_toolbar.png
   :align: center

Но эта только вершина айсберга; нажмите на странное шестнадцатеричное 
число чтобы открыть еще один очень полезный инструмент отладки: профайлер.

.. image:: /images/quick_tour/profiler.png
   :align: center

Конечно, вы не хотите видеть эти инструменты, когда вы развертываете 
приложение на рабочий сервер. Вот почему вы найдете еще один контроллер 
входа в папке ``web/`` (``app.php``), он оптимизирован для работы в 
рабочем окружении:

.. code-block:: text

    http://localhost/Symfony/web/app.php/demo/hello/Fabien

И если вы используете Apache с включенным ``mod_rewrite``, вы можете 
опустить часть URL с ``app.php``:

.. code-block:: text

    http://localhost/Symfony/web/demo/hello/Fabien

Но гораздо лучше, на рабочем сервере, установить корнем веб-сервера папку 
``web/``, чтобы защитить файлы и сделать более красивый URL:

.. code-block:: text

    http://localhost/demo/hello/Fabien

Чтобы сделать ответ приложение быстрым, Symfony2 сохраняет кеш в папку 
``app/cache/``. В окружении разработки (``app_dev.php``), кэш очищается 
автоматически, когда вы производите какое-либо изменение в коде. Но в 
рабочем окружении (``app.php``), где производительность это самое важное, 
этого не происходит. Вот почему вы всегда должны использовать окружение 
разработки, когда создаете ваше приложение.

Разные :term:`окружения<environment>` одного приложения различаются 
только в своей конфигурации. На самом деле, одна конфигурация может 
наследоваться от другой:

.. code-block:: yaml

    # app/config/config_dev.yml
    imports:
        - { resource: config.yml }

    web_profiler:
        toolbar: true
        intercept_redirects: false

Окружение ``dev`` (заданное в ``config_dev.yml``) наследуется от 
глобального файла ``config.yml`` и расширяет его, включая веб-панель 
отладки.

Заключительное слово
--------------------

Поздравляю! Вы почувствовали вкус кода Symfony2. Это было не тяжело, 
правда? Вы должны узнать еще многое, но вы уже можете видеть как Symfony2 
позволяет делать сайты лучше и быстрее. Если вы хотите узнать больше о 
Symfony2 погрузитесь в следующую главу:
":doc:`Вид<the_view>`".

.. _Symfony2 Standard Edition:      http://symfony.com/download
.. _Symfony за 5 минут:             http://symf.ru/symfony-in-five-minutes
.. _разделения ответственности:         http://en.wikipedia.org/wiki/Separation_of_concerns
.. _YAML:                           http://www.yaml.org/
.. _аннотациях в контроллерах:     http://bundles.symfony-reloaded.org/frameworkextrabundle/
.. _Twig:                           http://www.twig-project.org/
