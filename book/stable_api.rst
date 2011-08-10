Стабильный API для Symfony2
===========================

Стабильный API это набор всех public методов Symfony2 (компонентов и бандлов
ядра), которые соотвествуют следующим критериям:

* пространство имён и имя класса не изменятся;
* названия метода не изменится;
* сигнатура (аргументы и возвращаемое значение) метода не изменится;
* семантика того, что делает метод не изменится.

Хотя реализация может изменится. Единственный обоснованный случай для
изменения в стабильном API это исправление проблемы безопасности.

The stable API is based on a whitelist, tagged with `@api`. Therefore,
everything not tagged explicitly is not part of the stable API.

.. tip::

    Любой сторонний бандл должен публиковать свой стабильный API.

As of Symfony 2.0, the following components have a public tagged API:

* BrowserKit
* ClassLoader
* Console
* CssSelector
* DependencyInjection
* DomCrawler
* EventDispatcher
* Finder
* HttpFoundation
* HttpKernel
* Locale
* Process
* Routing
* Templating
* Translation
* Validator
* Yaml
