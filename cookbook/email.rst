.. index::
   single: Emails

How to send an Email
====================

One solution to send emails is to use the ``SwiftmailerBundle``, which
leverages the power of the `Swiftmailer`_ library.

.. note::

    Don't forget to enable the bundle in your kernel before using it::

        public function registerBundles()
        {
            $bundles = array(
                // ...
                new Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle(),
            );

            // ...
        }

Configuration
-------------

The only mandatory configuration parameter is ``transport``:

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
        xmlns:swiftmailer="http://www.symfony-project.org/schema/dic/swiftmailer"
        http://www.symfony-project.org/schema/dic/swiftmailer http://www.symfony-project.org/schema/dic/swiftmailer/swiftmailer-1.0.xsd
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

The following configuration attribute are available:

* ``transport`` (``smtp``, ``mail``, ``sendmail``, or ``gmail``)
* ``username``
* ``password``
* ``host``
* ``port``
* ``encryption`` (``tls``, or ``ssl``)
* ``auth_mode`` (``plain``, ``login``, or ``cram-md5``)
* ``spool``
    
    * ``type`` (only ``file`` is supported currently)
    * ``path``
* ``delivery_address`` (an email address where to send ALL emails)
* ``disable_delivery``

Sending Emails
--------------

The mailer is accessible via the ``mailer`` service; from an action::

    public function indexAction($name)
    {
        // get the mailer first (mandatory to initialize Swift Mailer)
        $mailer = $this->get('mailer');

        $message = \Swift_Message::newInstance()
            ->setSubject('Hello Email')
            ->setFrom('send@example.com')
            ->setTo('recipient@example.com')
            ->setBody($this->renderView('HelloBundle:Hello:email', array('name' => $name)))
        ;
        $mailer->send($message);

        return $this->render(...);
    }

.. note::

    To keep things decoupled, the email body has been stored in a template,
    rendered with the ``renderView()`` method.

.. tip::

    Read the ":doc:`gmail`" recipe if you want to use Gmail as a transport in
    the development environment.

.. _`Swiftmailer`: http://www.swiftmailer.org/