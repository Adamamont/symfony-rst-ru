.. index::
   single: Emails; Gmail

Как использовать Gmail для отправки писем
===============================

Во время разработки, вместо того чтобы использовать обычный сервер SMTP для 
рассылки электронных писем, вы можете найти использование Gmail простым и более 
практичным. Пакет Swiftmailer очень легко позволяет это.

.. tip::
    Вместо того чтобы использовать ваш обычный Gmail аккаунт, рекомендуется 
    создать специальный аккаунт.

В файле конфигурации разработки, измените ``transport`` настройку на ``gmail``
и задайте ``username`` и ``password`` к учётной записи Google:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_dev.yml
        swiftmailer:
            transport: gmail
            username:  your_gmail_username
            password:  your_gmail_password

    .. code-block:: xml

        <!-- app/config/config_dev.xml -->

        <!--
        xmlns:swiftmailer="http://symfony.com/schema/dic/swiftmailer"
        http://symfony.com/schema/dic/swiftmailer http://symfony.com/schema/dic/swiftmailer/swiftmailer-1.0.xsd
        -->

        <swiftmailer:config
            transport="gmail"
            username="your_gmail_username"
            password="your_gmail_password" />

    .. code-block:: php

        // app/config/config_dev.php
        $container->loadFromExtension('swiftmailer', array(
            'transport' => "gmail",
            'username'  => "your_gmail_username",
            'password'  => "your_gmail_password",
        ));

Вот и все!

.. note::
    Транспорт ``gmail`` просто ярлык, который использует ``smtp`` транспорт
    и устанавливает ``encryption``,  ``auth_mode`` и ``host`` для работы с Gmail.
