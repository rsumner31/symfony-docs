.. index::
   single: Controller; As Services

How to Define Controllers as Services
=====================================

In Symfony, a controller does *not* need to be registered as a service. But if you're
using the :ref:`default services.yaml configuration <service-container-services-load-example>`,
your controllers *are* already registered as services. This means you can use dependency
injection like any other normal service.

Referencing your Service from Routing
-------------------------------------

Registering your controller as a service is great, but you also need to make sure
that your routing references the service properly, so that Symfony knows to use it.

If the service id is the fully-qualified class name (FQCN) of your controller,
you're done! You can use the normal ``App\Controller\HelloController::index``
syntax in your routing and it will find your service.

.. configuration-block::

    .. code-block:: php-annotations

        // src/Controller/HelloController.php

        // You need to use Sensio's annotation to specify a service id
        use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
        // ...

        /**
         * @Route(service="app.hello_controller")
         */
        class HelloController
        {
            // ...
        }

    .. code-block:: yaml

        # config/routes.yaml
        hello:
            path:     /hello
            defaults: { _controller: app.hello_controller:indexAction }

    .. code-block:: xml

        <!-- config/routes.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing
                http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="hello" path="/hello">
                <default key="_controller">app.hello_controller:indexAction</default>
            </route>

        </routes>

    .. code-block:: php

        // config/routes.php
        $collection->add('hello', new Route('/hello', array(
            '_controller' => 'app.hello_controller:indexAction',
        )));

.. note::

    When using the ``service_id:method_name`` syntax, the method name must
    end with the ``Action`` suffix.

.. _controller-service-invoke:

Invokable Controllers
---------------------

If your controller implements the ``__invoke()`` method - popular with the
Action-Domain-Response (ADR) pattern, you can simply refer to the service id
(``App\Controller\HelloController`` or ``app.hello_controller`` for example).

Alternatives to base Controller Methods
---------------------------------------

When using a controller defined as a service, you can still extend any of the
:ref:`normal base controller <the-base-controller-class-services>` classes and
use their shortcuts. But, you don't need to! You can choose to extend *nothing*,
and use dependency injection to access different services.

The base `Controller class source code`_ is a great way to see how to accomplish
common tasks. For example, ``$this->render()`` is usually used to render a Twig
template and return a Response. But, you can also do this directly:

In a controller that's defined as a service, you can instead inject the ``templating``
service and use it directly::

    // src/Controller/HelloController.php
    namespace App\Controller;

    use Symfony\Component\HttpFoundation\Response;

    class HelloController
    {
        private $twig;

        public function __construct(\Twig_Environment $twig)
        {
            $this->twig = $twig;
        }

        public function index($name)
        {
            $content = $this->twig->render(
                'hello/index.html.twig',
                array('name' => $name)
            );

            return new Response($content);
        }
    }

You can also use a special :ref:`action-based dependency injection <controller-accessing-services>`
to receive services as arguments to your controller action methods.

Base Controller Methods and Their Service Replacements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The best way to see how to replace base ``Controller`` convenience methods is to
look at the `ControllerTrait`_ that holds its logic.

If you want to know what type-hints to use for each service, see the
``getSubscribedServices()`` method in `AbstractController`_.

.. _`Controller class source code`: https://github.com/symfony/symfony/blob/master/src/Symfony/Bundle/FrameworkBundle/Controller/ControllerTrait.php
.. _`base Controller class`: https://github.com/symfony/symfony/blob/master/src/Symfony/Bundle/FrameworkBundle/Controller/ControllerTrait.php
.. _`ControllerTrait`: https://github.com/symfony/symfony/blob/master/src/Symfony/Bundle/FrameworkBundle/Controller/ControllerTrait.php
.. _`AbstractController`: https://github.com/symfony/symfony/blob/master/src/Symfony/Bundle/FrameworkBundle/Controller/AbstractController.php
