.. index::
   single: Emails

Как отправлять письма
====================

Отправка писем является классической задачей для любого веб-приложения которая 
несёт особые осложнения и подводные камни. Вместо того чтобы создавать колесо,
одним из решений для рассылки писем является использование ``SwiftmailerBundle``,
который использует всю силу `Swiftmailer`_ библиотеки.

.. note::

    Не забудьте включить пакет в вашем ядре перед его использованием::

        public function registerBundles()
        {
            $bundles = array(
                // ...
                new Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle(),
            );

            // ...
        }

.. _swift-mailer-configuration:

Конфигурация
-------------

Перед использованием Swiftmailer, не забудьте сконфигурировать его.
Единственный обязательный параметр - ``transport``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        swiftmailer:
            transport:  smtp
            encryption: ssl
            auth_mode:  login
            host:       smtp.gmail.com
            username:   your_username
            password:   your_password

    .. code-block:: xml

        <!-- app/config/config.xml -->

        <!--
        xmlns:swiftmailer="http://symfony.com/schema/dic/swiftmailer"
        http://symfony.com/schema/dic/swiftmailer http://symfony.com/schema/dic/swiftmailer/swiftmailer-1.0.xsd
        -->

        <swiftmailer:config
            transport="smtp"
            encryption="ssl"
            auth-mode="login"
            host="smtp.gmail.com"
            username="your_username"
            password="your_password" />

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('swiftmailer', array(
            'transport'  => "smtp",
            'encryption' => "ssl",
            'auth_mode'  => "login",
            'host'       => "smtp.gmail.com",
            'username'   => "your_username",
            'password'   => "your_password",
        ));

Большинство Swiftmailer конфигураций относится к тому, как сообщения
будут доставляться.

Доступны следующие атрибуты конфигурации:

* ``transport``         (``smtp``, ``mail``, ``sendmail``, or ``gmail``)
* ``username``
* ``password``
* ``host``
* ``port``
* ``encryption``        (``tls``, or ``ssl``)
* ``auth_mode``         (``plain``, ``login``, or ``cram-md5``)
* ``spool``

  * ``type`` (как сохранять сообщения, только ``file`` поддерживается в настоящее время)
  * ``path`` (где сохраняють сообщения)
* ``delivery_address``  (почтовый адрес куда отправлять ВСЕ письма)
* ``disable_delivery``  (установите true для отключения доставки писем полностью)

Отправка писем
--------------

Библиотека Swiftmailer работает путём создания, настройки, а затем отправки 
``Swift_Message`` объектов. "Почтовик" отвечает за фактическую отправку сообщений
и доступен через службу ``mailer``. В целом, отправка писем достаточно проста::

    public function indexAction($name)
    {
        $message = \Swift_Message::newInstance()
            ->setSubject('Hello Email')
            ->setFrom('send@example.com')
            ->setTo('recipient@example.com')
            ->setBody($this->renderView('HelloBundle:Hello:email.txt.twig', array('name' => $name)))
        ;
        $this->get('mailer')->send($message);

        return $this->render(...);
    }

Чтобы не усложнять, тело письма было сохранено в шаблоне и отрисовано методом ``RenderView()``.

Объект ``$message`` поддерживает многие другие параметры, например, прикрепление 
вложений, добавление HTML контента, и многое другое. К счастью, Swiftmailer 
охватывает тему `Создания сообщений`_ очень подробно в своей документации.

.. tip::

    Доступно несколько других рецептов, связанных с отправкой писем в Symfony2:  

    * :doc:`gmail`
    * :doc:`email/dev_environment`
    * :doc:`email/spool`

.. _`Swiftmailer`: http://www.swiftmailer.org/
.. _`Создания сообщений`: http://swiftmailer.org/docs/messages
