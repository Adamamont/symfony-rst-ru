.. index::
   single: Валидация

Валидация
=========

Валидация очень частая задача в веб приложениях. Данные введеные в форму 
должны быть валидированы. Данные также должны пройти валидацию перед 
записью в базу данных или передачи в web-службу.

Symfony2 поставляется с компонентом `Validator`_, который выполняет эту задачу легко и прозрачно.
Этот компонент основан на спецификации `JSR303 Bean Validation specification`_. Что?
Спецификация Java в PHP? Вы не ослышались, но это не так плохо, как кажется.
Давайте посмотрим, как это можно использовать в PHP.

.. index:
   single: Валидация; Основы

Основы валидации
----------------

Лучший способ понять валидацию - это увидеть ее в действии. Для начала 
предположим, что вы создали старый-добрый PHP объект, который необходимо 
использовать где-нибудь в вашем приложении:

.. code-block:: php

    // src/Acme/BlogBundle/Entity/Author.php
    namespace Acme\BlogBundle\Entity;

    class Author
    {
        public $name;
    }

Пока это всего лишь обычный класс, созданный с какой-то целью. Цель 
валидации в том, чтобы сообщить вам, являются ли данные объекта валидными 
или же нет. Чтобы это заработало, вы должны сконфигурировать список 
правил (называемых :ref:`ограничениями<validation-constraints>` 
(constraints)) которым должен следовать объект, что бы быть валидным. Эти 
правила могут быть определены с помощью различных форматов (YML, XML, 
аннотации или PHP).

Чтобы гарантировать, что свойство ``$name`` не пустое, добавьте следующее:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            properties:
                name:
                    - NotBlank: ~

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\NotBlank()
             */
            public $name;
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <constraint-mapping xmlns="http://symfony.com/schema/dic/constraint-mapping"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/constraint-mapping http://symfony.com/schema/dic/services/constraint-mapping-1.0.xsd">

            <class name="Acme\BlogBundle\Entity\Author">
                <property name="name">
                    <constraint name="NotBlank" />
                </property>
            </class>
        </constraint-mapping>

    .. code-block:: php

        // src/Acme/BlogBundle/Entity/Author.php

        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\NotBlank;

        class Author
        {
            public $name;

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('name', new NotBlank());
            }
        }

.. tip::

    Protected и private свойства также могут быть валидированы, как
    геттеры (см. `Цели ограничений`_).

.. index::
   single: Валидация; Использование validator

Использование ``validator`` Service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Чтобы на самом деле проверить объект ``Author`` используется метод  ``validate`` в сервисе ``validator`` (класс :class:`Symfony\\Component\\Validator\\Validator`).
Работа ``validator`` iпроста: прочесть ограничения (т.е. правила)
класса и проверить удовлетвореют ли данные этим правилам или нет. Если 
валадация не пройдена, возвращается массив ошибок. Рассмотрим этот 
простой пример контроллера:

.. code-block:: php

    use Symfony\Component\HttpFoundation\Response;
    use Acme\BlogBundle\Entity\Author;
    // ...

    public function indexAction()
    {
        $author = new Author();
        // ... do something to the $author object

        $validator = $this->get('validator');
        $errors = $validator->validate($author);

        if (count($errors) > 0) {
            return new Response(print_r($errors, true));
        } else {
            return new Response('The author is valid! Yes!');
        }
    }

Если свойство ``$name`` пусто, вы увидите следующее сообщение об ошибке:

.. code-block:: text

    Acme\BlogBundle\Author.name:
        This value should not be blank

Если в это свойство ``name`` вставить значение, то вернется сообщение об успехе.

.. tip::

    Most of the time, you won't interact directly with the ``validator``
    service or need to worry about printing out the errors. Most of the time,
    you'll use validation indirectly when handling submitted form data. For
    more information, see the :ref:`book-validation-forms`.

You could also pass the collection of errors into a template.

.. code-block:: php

    if (count($errors) > 0) {
        return $this->render('AcmeBlogBundle:Author:validate.html.twig', array(
            'errors' => $errors,
        ));
    } else {
        // ...
    }

В шаблоне вы можете вывести ошибки так, как хотите:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/BlogBundle/Resources/views/Author/validate.html.twig #}

        <h3>The author has the following errors</h3>
        <ul>
        {% for error in errors %}
            <li>{{ error.message }}</li>
        {% endfor %}
        </ul>

    .. code-block:: html+php

        <!-- src/Acme/BlogBundle/Resources/views/Author/validate.html.php -->

        <h3>The author has the following errors</h3>
        <ul>
        <?php foreach ($errors as $error): ?>
            <li><?php echo $error->getMessage() ?></li>
        <?php endforeach; ?>
        </ul>

.. note::

    Каждая ошибка валидации (называющаяся "нарушение ограничения" ("constraint violation")), представлена объектом :class:`Symfony\\Component\\Validator\\ConstraintViolation`.

.. index::
   single: Валидация; Валидация и формы

.. _book-validation-forms:

Валидация и формы
~~~~~~~~~~~~~~~~~

Сервис ``validator`` может быть использован в любое время для проверки любого объекта.
Однако в действительности, вы обычно будете работать с ``validator`` при
работе с формами. Бибилиотека Symfony для работы с формами ``validator`` использует сервис валидации внутренне для проверки объекта после того, как данные были отправлены и связаны. Нарушения ограничений объекта преобразуются в объекты ``FieldError``, которые затем могут отображаться с вашей формой. The typical form submission
workflow looks like the following from inside a controller::

    use Acme\BlogBundle\Entity\Author;
    use Acme\BlogBundle\Form\AuthorType;
    use Symfony\Component\HttpFoundation\Request;
    // ...

    public function updateAction(Request $request)
    {
        $author = new Acme\BlogBundle\Entity\Author();
        $form = $this->createForm(new AuthorType(), $author);

        if ($request->getMethod() == 'POST') {
            $form->bindRequest($request);

            if ($form->isValid()) {
                // the validation passed, do something with the $author object

                $this->redirect($this->generateUrl('...'));
            }
        }

        return $this->render('BlogBundle:Author:form.html.twig', array(
            'form' => $form->createView(),
        ));
    }

.. note::

    This example uses an ``AuthorType`` form class, which is not shown here.

Для большей информации, смотрите главу :doc:`Forms</book/forms>`.

.. index::
   pair: Валидация; Настройка

.. _book-validation-configuration:

Конфигурация
-------------

The Symfony2 validator is enabled by default, but you must explicitly enable
annotations if you're using the annotation method to specify your constraints:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            validation: { enable_annotations: true }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config>
            <framework:validation enable_annotations="true" />
        </framework:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('framework', array('validation' => array(
            'enable_annotations' => true,
        )));

.. index::
   single: Валидация; Ограничения

.. _validation-constraints:

Ограничения
-----------

``validator`` разработан для проверки объектов на соответствия 
*ограничениям* (т.е. правилам). Для валидации объекта, просто представьте 
одно или более ограничений в своем классе, а затем передайте их сервису ``validator``.

Ограничение это просто PHP объект, которое представляется в виде жесткого 
заявления. В реальной жизни, ограничение может быть представлено в виде: 
"Пирог не должен быть подгорелым". В Symfony2 ограничения похожи: они 
являются утверждениями, что условие истинно. Получив значение, 
ограничение сообщает сообщит вам, придерживается ли значение правилам 
ограничений.

Поддерживаемые ограничения
~~~~~~~~~~~~~~~~~~~~~~~~~~

Пакеты Symfony2 содержат большое число наиболее часто необходимых ограничений.
Полный список ограничений с различными деталями доступен в
:doc:`справочном разделе ограничений</reference/constraints>`.

.. index::
   single: Валидация; Конфигурация ограничений

.. _book-validation-constraint-configuration:

Конфигурация ограничений
~~~~~~~~~~~~~~~~~~~~~~~~

Some constraints, like :doc:`NotBlank</reference/constraints/NotBlank>`,
are simple whereas others, like the :doc:`Choice</reference/constraints/Choice>`
constraint, have several configuration options available. Suppose that the
``Author`` class has another property, ``gender`` that can be set to either
"male" or "female":

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            properties:
                gender:
                    - Choice: { choices: [male, female], message: Choose a valid gender. }

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\Choice(
             *     choices = { "male", "female" },
             *     message = "Choose a valid gender."
             * )
             */
            public $gender;
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <constraint-mapping xmlns="http://symfony.com/schema/dic/constraint-mapping"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/constraint-mapping http://symfony.com/schema/dic/services/constraint-mapping-1.0.xsd">

            <class name="Acme\BlogBundle\Entity\Author">
                <property name="gender">
                    <constraint name="Choice">
                        <option name="choices">
                            <value>male</value>
                            <value>female</value>
                        </option>
                        <option name="message">Choose a valid gender.</option>
                    </constraint>
                </property>
            </class>
        </constraint-mapping>

    .. code-block:: php

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\NotBlank;

        class Author
        {
            public $gender;

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('gender', new Choice(array(
                    'choices' => array('male', 'female'),
                    'message' => 'Choose a valid gender.',
                )));
            }
        }

.. _validation-default-option:

The options of a constraint can always be passed in as an array. Some constraints,
however, also allow you to pass the value of one, "*default*", option in place
of the array. In the case of the ``Choice`` constraint, the ``choices``
options can be specified in this way.

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            properties:
                gender:
                    - Choice: [male, female]

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\Choice({"male", "female"})
             */
            protected $gender;
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <constraint-mapping xmlns="http://symfony.com/schema/dic/constraint-mapping"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/constraint-mapping http://symfony.com/schema/dic/services/constraint-mapping-1.0.xsd">

            <class name="Acme\BlogBundle\Entity\Author">
                <property name="gender">
                    <constraint name="Choice">
                        <value>male</value>
                        <value>female</value>
                    </constraint>
                </property>
            </class>
        </constraint-mapping>

    .. code-block:: php

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\Choice;

        class Author
        {
            protected $gender;

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('gender', new Choice(array('male', 'female')));
            }
        }

This is purely meant to make the configuration of the most common option of
a constraint shorter and quicker.

If you're ever unsure of how to specify an option, either check the API documentation
for the constraint or play it safe by always passing in an array of options
(the first method shown above).

.. index::
   single: Validation; Constraint targets

.. _validator-constraint-targets:

Цели ограничений
-----------------

Ограничения могут быть применены к свойству класса (например ``name``) 
или к открытому геттер-методу (например ``getFullName``). The first is 
the most common and easy
to use, but the second allows you to specify more complex validation rules.

.. index::
   single: Validation; Property constraints

.. _validation-property-target:

Свойства
~~~~~~~~~

Проверка свойств класса является самой основной техникой валидации. 
Symfony2 позволяет вам проверять private, protected или public свойства. 
Следующий листинг показывает вам, как конфигурировать свойства ``$firstName`` класса ``Author``
чтобы иметь по крайней-мере 3 символа.

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            properties:
                firstName:
                    - NotBlank: ~
                    - MinLength: 3

    .. code-block:: php-annotations

        // Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\NotBlank()
             * @Assert\MinLength(3)
             */
            private $firstName;
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <class name="Acme\BlogBundle\Entity\Author">
            <property name="firstName">
                <constraint name="NotBlank" />
                <constraint name="MinLength">3</constraint>
            </property>
        </class>

    .. code-block:: php

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\NotBlank;
        use Symfony\Component\Validator\Constraints\MinLength;

        class Author
        {
            private $firstName;

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('firstName', new NotBlank());
                $metadata->addPropertyConstraint('firstName', new MinLength(3));
            }
        }

.. index::
   single: Validation; Getter constraints

Геттеры
~~~~~~~

Ограничение также может применено для возвращения значения метода. 
Symfony2 позволяет вам добавлять ограничение public методам, котрые 
начинаются с "get" или "is". В этом руководстве, оба этих методов 
называются "геттерами".

Преимущество этой техники в том, что она позоволяет вам проверить ваш 
объект динамически. For example, suppose you want to make sure that a 
password field
doesn't match the first name of the user (for security reasons). You can
do this by creating an ``isPasswordLegal`` method, and then asserting that
this method must return ``true``:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            getters:
                passwordLegal:
                    - "True": { message: "The password cannot match your first name" }

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\True(message = "The password cannot match your first name")
             */
            public function isPasswordLegal()
            {
                // return true or false
            }
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <class name="Acme\BlogBundle\Entity\Author">
            <getter property="passwordLegal">
                <constraint name="True">
                    <option name="message">The password cannot match your first name</option>
                </constraint>
            </getter>
        </class>

    .. code-block:: php

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\True;

        class Author
        {
            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addGetterConstraint('passwordLegal', new True(array(
                    'message' => 'The password cannot match your first name',
                )));
            }
        }

Now, create the ``isPasswordLegal()`` method, and include the logic you need::

    public function isPasswordLegal()
    {
        return ($this->firstName != $this->password);
    }

.. note::

    Внимательные из вас заметят, что префикс геттера ("get" или "is")      
    опущен в отображении (mapping). Это позволяет вам перемещать 
    ограничение свойства с тем же именем позже (или наоборот) без 
    изменения логики валидации.

.. _validation-class-target:

Classes
~~~~~~~

Some constraints apply to the entire class being validated. For example,
the :doc:`Callback</reference/constraints/Callback>` constraint is a generic
constraint that's applied to the class itself. When that class is validated,
methods specified by that constraint are simply executed so that each can
provide more custom validation.

.. _book-validation-validation-groups:

Validation Groups
-----------------

So far, you've been able to add constraints to a class and ask whether or
not that class passes all of the defined constraints. In some cases, however,
you'll need to validate an object against only *some* of the constraints
on that class. To do this, you can organize each constraint into one or more
"validation groups", and then apply validation against just one group of
constraints.

For example, suppose you have a ``User`` class, which is used both when a
user registers and when a user updates his/her contact information later:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\User:
            properties:
                email:
                    - Email: { groups: [registration] }
                password:
                    - NotBlank: { groups: [registration] }
                    - MinLength: { limit: 7, groups: [registration] }
                city:
                    - MinLength: 2

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/User.php
        namespace Acme\BlogBundle\Entity;

        use Symfony\Component\Security\Core\User\UserInterface
        use Symfony\Component\Validator\Constraints as Assert;

        class User implements UserInterface
        {
            /**
            * @Assert\Email(groups={"registration"})
            */
            private $email;

            /**
            * @Assert\NotBlank(groups={"registration"})
            * @Assert\MinLength(limit=7, groups={"registration"})
            */
            private $password;

            /**
            * @Assert\MinLength(2)
            */
            private $city;
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <class name="Acme\BlogBundle\Entity\User">
            <property name="email">
                <constraint name="Email">
                    <option name="groups">
                        <value>registration</value>
                    </option>
                </constraint>
            </property>
            <property name="password">
                <constraint name="NotBlank">
                    <option name="groups">
                        <value>registration</value>
                    </option>
                </constraint>
                <constraint name="MinLength">
                    <option name="limit">7</option>
                    <option name="groups">
                        <value>registration</value>
                    </option>
                </constraint>
            </property>
            <property name="city">
                <constraint name="MinLength">7</constraint>
            </property>
        </class>

    .. code-block:: php

        // src/Acme/BlogBundle/Entity/User.php
        namespace Acme\BlogBundle\Entity;

        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\Email;
        use Symfony\Component\Validator\Constraints\NotBlank;
        use Symfony\Component\Validator\Constraints\MinLength;

        class User
        {
            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('email', new Email(array(
                    'groups' => array('registration')
                )));

                $metadata->addPropertyConstraint('password', new NotBlank(array(
                    'groups' => array('registration')
                )));
                $metadata->addPropertyConstraint('password', new MinLength(array(
                    'limit'  => 7,
                    'groups' => array('registration')
                )));

                $metadata->addPropertyConstraint('city', new MinLength(3));
            }
        }

With this configuration, there are two validation groups:

* ``Default`` - contains the constraints not assigned to any other group;

* ``registration`` - contains the constraints on the ``email`` and ``password``
  fields only.

To tell the validator to use a specific group, pass one or more group names
as the second argument to the ``validate()`` method::

    $errors = $validator->validate($author, array('registration'));

Of course, you'll usually work with validation indirectly through the form
library. For information on how to use validation groups inside forms, see
:ref:`book-forms-validation-groups`.

Заключительные мысли
--------------------

В Symfony2 ``validator`` мощный инструмент, который может быть 
использован для гарантирования, что данные любого объекта валидны. Мощь 
валидации заключается в "ограничениях", представляющие собой правила, 
которые вы можете применить к свойствам или геттер-методам вашего 
объекта. И пока вы будете использовать фреймворк валидации вместе с 
формами, помните, что он может быть использован в любом месте для 
проверки любого объекта.

Узнайте больше из книги рецептов
--------------------------------

* :doc:`/cookbook/validation/custom_constraint`

.. _Validator: https://github.com/symfony/Validator
.. _JSR303 Bean Validation specification: http://jcp.org/en/jsr/detail?id=303
