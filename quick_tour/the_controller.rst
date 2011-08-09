Контроллер
==========

Всё ещё с нами после первых двух частей? Вы становитесь ярым приверженцем Symfony2!
Давайте, без лишней суеты, узнаем что контроллеры могут сделать для вас.

Использование форматов
----------------------

В наши дни, web приложение должно уметь выдавать не только HTML страницы.
Начиная с XML для RSS каналов и Web служб, заканчивая JSON для Ajax запросов,
существует множество различных форматов. Поддержка этих форматов в Symfony2
проста. Измените маршруты добавив знаначение по-умолчанию
``xml`` для переменной ``_format``::

    // src/Acme/DemoBundle/Controller/DemoController.php
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

    /**
     * @Route("/hello/{name}", defaults={"_format"="xml"}, name="_demo_hello")
     * @Template()
     */
    public function helloAction($name)
    {
        return array('name' => $name);
    }

By using the request format (as defined by the ``_format`` value), Symfony2
automatically selects the right template, here ``hello.xml.twig``:

.. code-block:: xml+php

    <!-- src/Acme/DemoBundle/Resources/views/Demo/hello.xml.twig -->
    <hello>
        <name>{{ name }}</name>
    </hello>

Вот и всё что для этого нужно. Для стандартных
форматов Symfony2 автоматически подбирает заголовок ``Content-Type`` для ответа.
Если хотите поддержку форматов лишь для одного действия, тогда используйте
заполнитель ``{_format}`` в паттерне::

    // src/Acme/DemoBundle/Controller/DemoController.php
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;

    /**
     * @Route("/hello/{name}.{_format}", defaults={"_format"="html"}, requirements={"_format"="html|xml|json"}, name="_demo_hello")
     * @Template()
     */
    public function helloAction($name)
    {
        return array('name' => $name);
    }

The controller will now be called for URLs like ``/demo/hello/Fabien.xml`` or
``/demo/hello/Fabien.json``.

The ``requirements`` entry defines regular expressions that placeholders must
match. In this example, if you try to request the ``/demo/hello/Fabien.js``
resource, you will get a 404 HTTP error, as it does not match the ``_format``
requirement.

Перемещения и перенаправления
-----------------------------

Если вы хотите переместить пользователя на другую страницу, используйте метод
``redirect()``::

    return $this->redirect($this->generateUrl('_demo_hello', array('name' => 'Lucas')));

The ``generateUrl()`` is the same method as the ``path()`` function we used in
templates. It takes the route name and an array of parameters as arguments and
returns the associated friendly URL.

Также вы можете легко переместить одно действие на другое с помощью метода
``forward()``. Как и для хелпера ``actions``, он применяет внутренний подзапрос,
но возвращает объект ``Response``, что позволяет в дальнейшем его изменить::

    $response = $this->forward('AcmeDemoBundle:Hello:fancy', array('name' => $name, 'color' => 'green'));

    // do something with the response or return it directly

Получение информации о запросе
------------------------------

Помимо значений заполнителей для маршрутизации, контроллер имеет доступ к
объекту ``Request``::

    $request = $this->getRequest();

    $request->isXmlHttpRequest(); // is it an Ajax request?

    $request->getPreferredLanguage(array('en', 'fr'));

    $request->query->get('page'); // get a $_GET parameter

    $request->request->get('page'); // get a $_POST parameter

В шаблоне получить доступ к объекту ``Request`` можно через переменную ``app.request``:

.. code-block:: html+jinja

    {{ app.request.query.get('page') }}

    {{ app.request.parameter('page') }}

Сохранение и получение информации из сессии
-------------------------------------------

Протокол HTTP не имеет состояний, но Symfony2 предоставляет удобный объект
сиссии, который представляет клиента (будь он человеком, использующим браузер,
ботом или web службой). Между двумя запросами Symfony2 хранит атрибуты в cookie,
используя родные сессии из PHP.

Сохранение и получение информации из сессии легко выполняется из любого
контроллера::

    $session = $this->getRequest()->getSession();

    // store an attribute for reuse during a later user request
    $session->set('foo', 'bar');

    // in another controller for another request
    $foo = $session->get('foo');

    // set the user locale
    $session->setLocale('fr');

Также можно хранить небольшие сообщения, которые будут доступны для следующего
запроса::

    // store a message for the very next request (in a controller)
    $session->setFlash('notice', 'Congratulations, your action succeeded!');

    // display the message back in the next request (in a template)
    {{ app.session.flash('notice') }}

This is useful when you need to set a success message before redirecting
the user to another page (which will then show the message).

Securing Resources
------------------

The Symfony Standard Edition comes with a simple security configuration that
fits most common needs:

.. code-block:: yaml

    # app/config/security.yml
    security:
        encoders:
            Symfony\Component\Security\Core\User\User: plaintext

        role_hierarchy:
            ROLE_ADMIN:       ROLE_USER
            ROLE_SUPER_ADMIN: [ROLE_USER, ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH]

        providers:
            in_memory:
                users:
                    user:  { password: userpass, roles: [ 'ROLE_USER' ] }
                    admin: { password: adminpass, roles: [ 'ROLE_ADMIN' ] }

        firewalls:
            dev:
                pattern:  ^/(_(profiler|wdt)|css|images|js)/
                security: false

            login:
                pattern:  ^/demo/secured/login$
                security: false

            secured_area:
                pattern:    ^/demo/secured/
                form_login:
                    check_path: /demo/secured/login_check
                    login_path: /demo/secured/login
                logout:
                    path:   /demo/secured/logout
                    target: /demo/

This configuration requires users to log in for any URL starting with
``/demo/secured/`` and defines two valid users: ``user`` and ``admin``.
Moreover, the ``admin`` user has a ``ROLE_ADMIN`` role, which includes the
``ROLE_USER`` role as well (see the ``role_hierarchy`` setting).

.. tip::

    For readability, passwords are stored in clear text in this simple
    configuration, but you can use any hashing algorithm by tweaking the
    ``encoders`` section.

Going to the ``http://localhost/Symfony/web/app_dev.php/demo/secured/hello``
URL will automatically redirect you to the login form because this resource is
protected by a ``firewall``.

You can also force the action to require a given role by using the ``@Secure``
annotation on the controller::

    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;
    use JMS\SecurityExtraBundle\Annotation\Secure;

    /**
     * @Route("/hello/admin/{name}", name="_demo_secured_hello_admin")
     * @Secure(roles="ROLE_ADMIN")
     * @Template()
     */
    public function helloAdminAction($name)
    {
        return array('name' => $name);
    }

Now, log in as ``user`` (who does *not* have the ``ROLE_ADMIN`` role) and
from the secured hello page, click on the "Hello resource secured" link.
Symfony2 should return a 403 HTTP status code, indicating that the user
is "forbidden" from accessing that resource.

.. note::

    The Symfony2 security layer is very flexible and comes with many different
    user providers (like one for the Doctrine ORM) and authentication providers
    (like HTTP basic, HTTP digest, or X509 certificates). Read the
    ":doc:`/book/security`" chapter of the book for more information
    on how to use and configure them.

Caching Resources
-----------------

As soon as your website starts to generate more traffic, you will want to
avoid generating the same resource again and again. Symfony2 uses HTTP cache
headers to manage resources cache. For simple caching strategies, use the
convenient ``@Cache()`` annotation::

    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Cache;

    /**
     * @Route("/hello/{name}", name="_demo_hello")
     * @Template()
     * @Cache(maxage="86400")
     */
    public function helloAction($name)
    {
        return array('name' => $name);
    }

In this example, the resource will be cached for a day. But you can also use
validation instead of expiration or a combination of both if that fits your
needs better.

Resource caching is managed by the Symfony2 built-in reverse proxy. But because 
caching is managed using regular HTTP cache headers, you can replace the 
built-in reverse proxy with Varnish or Squid and easily scale your application.

.. note::

    But what if you cannot cache whole pages? Symfony2 still has the solution
    via Edge Side Includes (ESI), which are supported natively. Learn more by
    reading the ":doc:`/book/http_cache`" chapter of the book.

Заключительное слово
--------------------

Вот и всё что хотелось рассказать, и я даже уверен, что мы не использовали все
отведённые 10 минут. Мы коротко рассмотрели бандлы в первой части, и все 
особенности о которых мы узнали являются частью бандлов ядра фреймворка.
Но благодаря бандлам, в Symfony2 всё может быть
расширено или заменено. Это и есть тема :doc:`следующей части руководства<the_architecture>`.
