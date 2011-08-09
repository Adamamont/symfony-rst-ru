Вид
====

Прочитав первую часть, вы решили что Symfony2 заслуживает ещё 10 минут. Хорошо.
Во второй части вы узнаете больше о движке шаблонов `Twig`_ в Symfony2. Twig
это гибкий, быстрый и безопасный шаблонизатор для PHP. Он делает шаблоны
удобочитаемыми и выразительными, а также более дружественными для web дизайнеров.

.. note::

    Вместо Twig, можете использовать для шаблонов :doc:`PHP </cookbook/templating/PHP>`.Оба шаблонных движка поддерживаются Symfony2.

Twig, краткий обзор
--------------------

.. tip::

    Если хотите изучить Twig, мы настоятельно рекомендуем прочесть эту официальную
    `документацию <documentation>`_. Этот раздел лишь кратко описывает основные концепции.

Шаблон Twig это текстовый файл, который может генерировать любой формат,
основанный на тексте (HTML, XML, CSV, LaTeX, ...). Twig устанавливает два вида
разделителей:

* ``{{ ... }}``: Выводит переменную или результат выражения в шаблон;

* ``{% ... %}``: Тег, управляющий логикой шаблона; например, используется для
  выполнения ``for`` циклов или ``if`` условий.

Ниже приведен минимальный шаблон, иллюстрирующий основы, использующий две
переменные ``page_title`` и ``navigation``, которые переданы в шаблон:

.. code-block:: html+jinja

    <!DOCTYPE html>
    <html>
        <head>
            <title>My Webpage</title>
        </head>
        <body>
            <h1>{{ page_title }}</h1>

            <ul id="navigation">
                {% for item in navigation %}
                    <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
                {% endfor %}
            </ul>
        </body>
    </html>


.. tip::

   Комментарии могут быть включены в шаблон используя разделитель ``{# ... #}``.

Для того, чтобы отобразить шаблон в Symfony используйте метод ``render``
из контроллера и передайте любые переменные, нужные вам в шаблоне::

    $this->render('AcmeDemoBundle:Demo:hello.html.twig', array(
        'name' => $name,
    ));

Переменные, переданные в шаблон, могут быть строками, массивами или даже
объектами. Twig абстрагирует разницу между ними и даёт вам доступ к "атрибутам"
переменной, обозначенным через точку (``.``):

.. code-block:: jinja

    {# array('name' => 'Fabien') #}
    {{ name }}

    {# array('user' => array('name' => 'Fabien')) #}
    {{ user.name }}

    {# force array lookup #}
    {{ user['name'] }}

    {# array('user' => new User('Fabien')) #}
    {{ user.name }}
    {{ user.getName }}

    {# force method name lookup #}
    {{ user.name() }}
    {{ user.getName() }}

    {# pass arguments to a method #}
    {{ user.date('Y-m-d') }}

.. note::

    Важно знать что фигурные скобки это не часть переменной, а оператор печати.
    Если вы используете переменные внутри тегов, не ставьте скобки вокруг них.

Декорирование шаблонов
----------------------

Часто шаблоны в проекте разделяют общие элементы, такие как всем известные
header и footer. В Symfony2, мы смотрим на эту проблему иначе: один шаблон
может быть декорирован другим. Это похоже на классы в PHP: наследование шаблона
позволяет создать его базовый "макет", содержащий общие элементы вашего сайта и
устанавливающий "блоки", которые могут быть переопределены дочерними шаблонами.

Шаблон ``hello.html.twig`` наследуется от ``layout.html.twig`` благодаря  тегу ``extends``:

.. code-block:: html+jinja

    {# src/Acme/DemoBundle/Resources/views/Demo/hello.html.twig #}
    {% extends "AcmeDemoBundle::layout.html.twig" %}

    {% block title "Hello " ~ name %}

    {% block content %}
        <h1>Hello {{ name }}!</h1>
    {% endblock %}

Обозначение ``AcmeDemoBundle::layout.html.twig`` выглядит знакомо, не так ли?
Обозначается так же как ссылка на обычный шаблон. Эта часть ``::`` всего лишь обозначает
что контроллер не указан, т.о. соотвествующий файл хранится прямо в ``Resources/views/``.

Рассмотрим файл ``layout.html.twig``:

.. code-block:: jinja

    {# src/Acme/DemoBundle/Resources/views/layout.html.twig #}
    <div class="symfony-content">
        {% block content %}
        {% endblock %}
    </div>

Тег ``{% block %}`` устанавливает два блока (``body`` и ``content``), которые
дочерние шаблоны смогут заполнить. Всё что делает этот тег, это сообщает движку
шаблонов, что дочерний шаблон может переопределить эти участки.

Шаблон ``hello.html.twig`` переопределяет блок ``content``, 
это значит, что текст "Hello Fabien" будет отображен внутри элемента ``div.symfony-content``.

Теги, фильтры и функции
-----------------------

Одна из лучших особенностей Twig его расширяемость через теги, фильтры и функции;
Многие из них поставляется вместе с Symfony2, облегчая работу web дизайнера.

Включение других шаблонов
~~~~~~~~~~~~~~~~~~~~~~~~~

Лучший способ распределить фрагмент кода между несколькими различными шаблонами
это определить шаблон, подключаемый в другие.

Создайте шаблон ``embedded.html.twig``:

.. code-block:: jinja

    {# src/Acme/DemoBundle/Resources/views/Demo/embedded.html.twig #}
    Hello {{ name }}

Измените шаблон ``index.html.twig`` таким образом, чтобы подключить его:

.. code-block:: jinja

    {# src/Acme/DemoBundle/Resources/views/Demo/hello.html.twig #}
    {% extends "AcmeDemoBundle::layout.html.twig" %}

    {# override the body block from embedded.html.twig #}
    {% block content %}
        {% include "AcmeDemoBundle:Demo:embedded.html.twig" %}
    {% endblock %}

Вложение других контроллеров
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Что если вы захотите вложить результат другого контроллера в шаблон? Это очень
удобно когда работаешь с Ajax или когда встроенному шаблону необходимы
переменные, которые не доступны в главном шаблоне.

Если вы создали действие ``fancy`` и хотите включить его в шаблон ``index``,
используйте тег ``render``:

.. code-block:: jinja

    {# src/Acme/DemoBundle/Resources/views/Demo/index.html.twig #}
    {% render "AcmeDemoBundle:Demo:fancy" with { 'name': name, 'color': 'green' } %}

Имеем строку ``HelloBundle:Hello:fancy``, обращающуюся к действию ``fancy``
контроллера ``Hello`` и аргумент, используемый для имитирования запроса для
заданного пути::

    // src/Acme/DemoBundle/Controller/DemoController.php

    class DemoController extends Controller
    {
        public function fancyAction($name, $color)
        {
            // create some object, based on the $color variable
            $object = ...;

            return $this->render('AcmeDemoBundle:Demo:fancy.html.twig', array('name' => $name, 'object' => $object));
        }

        // ...
    }

Создание ссылок между страницами
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Говоря о web приложениях, нельзя не упомянуть о ссылках. Вместо жёстких URL-ов
в шаблонах, функция ``path`` поможет сделать URL-ы, основанные на конфигурации
маршрутизатора. Таким образом URL-ы могут быть легко обновлены, если изменить
конфигурацию:

.. code-block:: html+jinja

    <a href="{{ path('_demo_hello', { 'name': 'Thomas' }) }}">Greet Thomas!</a>

Функция ``path`` использует имя маршрута и массив параметров как аргументы.
Имя маршрута это основа, в соотвествии с которой выбираются маршруты, а
параметры это значения заполнителей, объявленных в паттерне маршрута::

    // src/Acme/DemoBundle/Controller/DemoController.php
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

    /**
     * @Route("/hello/{name}", name="_demo_hello")
     * @Template()
     */
    public function helloAction($name)
    {
        return array('name' => $name);
    }

.. tip::

    Функция ``url`` создает *абсолютные* URL-ы: ``{{ url('_demo_hello', {
    'name': 'Thomas' }) }}``.

Подключение активов: изображений, JavaScript-ов и таблиц стилей
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Как выглядел бы интернет без изображений, JavaScript-ов и таблиц стилей?
Symfony2 предлагает функцию ``asset`` для работы с ними:

.. code-block:: jinja

    <link href="{{ asset('css/blog.css') }}" rel="stylesheet" type="text/css" />

    <img src="{{ asset('images/logo.png') }}" />

Основная цель функции ``asset`` сделать приложение более переносимым. Благодаря
ей, можно переместить корневую папку приложения куда угодно внутри вашей
корневой web директории без изменения шаблона.

Экранирование переменных
------------------------

Изначально Twig настроен экранировать весь вывод. Прочтите Twig
`documentation`_ чтобы узнать больше об экранировании и расширении Escaper.

Заключительное слово
--------------------

Twig простой и мощный. Благодаря макетам, блокам, шаблонам и внедрениям действий,
становится действительно просто организовать ваши шаблоны логически и сделать их
расширяемыми.

Проработав с Symfony2 около 20 минут, вы уже можете делать удивительные вещи.
В этом сила Symfony2. Изучать основы легко, вскоре вы узнаете что эта простота
скрыта в очень гибкой архитектуре.

Я немного поспешил. Во-первых, вы должны узнать больше о контроллере, именно он
станет темой :doc:`следующей части учебника<the_controller>`. Готовы к следующим 10 минутам с Symfony2?

.. _Twig:          http://www.twig-project.org/
.. _documentation: http://www.twig-project.org/documentation
