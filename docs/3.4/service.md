## 加载
Symfony的服务在控制器中懒加载，即调用时为对应的服务进行实例化。


## services设置

*app/config/services.yml*

```php
services:
    # default configuration for services in *this* file
    _defaults:
        autowire: true
        autoconfigure: true
        public: false

    # makes classes in src/AppBundle available to be used as services
    AppBundle\:
        resource: '../../src/AppBundle/*'
        # you can exclude directories or files
        # but if a service is unused, it's removed anyway
        exclude: '../../src/AppBundle/{Entity,Repository}'
    command_handlers:
            namespace: App\Domain\
            resource: '../../src/Domain/*/CommandHandler'
            tags: [command_handler]
            
    event_subscribers:
            namespace: App\Domain\
            resource: '../../src/Domain/*/EventSubscriber'
            tags: [event_subscriber]
        
    # explicitly configure the service
    AppBundle\Controller\LuckyController:
        public: true
        bind:
            # for any $logger argument, pass this specific service
            $logger: '@monolog.logger.doctrine'
     # version 3.3
     AppBundle\Controller\:
             resource: '../../src/AppBundle/Controller'
             public: true
             tags: ['controller.service_arguments']
    site_update_manager.superadmin:
            class: AppBundle\Updates\SiteUpdateManager
            # you CAN still use autowiring: we just want to show what it looks like without
            autowire: false
            # manually wire all arguments
            arguments:
                - '@AppBundle\Service\MessageGenerator'
                - '@mailer'
                - 'superadmin@example.com'
    
    site_update_manager.normal_users:
        class: AppBundle\Updates\SiteUpdateManager
        autowire: false
        arguments:
            - '@AppBundle\Service\MessageGenerator'
            - '@mailer'
            - 'contact@example.com'
```

_defaults中的设置对文件中定义的所有服务都有用，autowire: true根据服务中的构造函数定义的依赖注入自动传入正确的参数。其根据自动配置定义

resource为资源内容，exclude为排除项。

服务的id为配置项中的键值。

通过资源加载可以将资源中对应的类作为服务使用，仅注册其中显示定义为服务的类。

服务id覆盖，两个服务的配置没有继承关系，但都继承_defaults。

exclude会提高开发性能，但修改其中对应文件内容不影响容器，即不会导致容器重新构建。

配置文件中public为false时，意思是注册的服务为私有。

默认情况下服务的命名空间是其对应配置的建值。 不同资源建值但命名空间相同的情况下，用tags区别。

可以为服务配置常量，相同的类需设置不同的id。为确保服务的正确调用，尽量确保服务id唯一。

bind可为依赖注入的接口选定特殊的实现类，也可为构造函数设置值。

arguments为控制器构造中不能自动传值的类设置参数，比如：

*app/config/services.yml*

```php
services:
    # same as before
    AppBundle\:
        resource: '../../src/AppBundle/*'
        exclude: '../../src/AppBundle/{Entity,Repository}'

    # explicitly configure the service
    AppBundle\Updates\SiteUpdateManager:
        arguments:
            $adminEmail: 'manager@example.com'
```

*src/AppBundle/Updates/SiteUpdateManager.php*

```php
class SiteUpdateManager
{
    private $adminEmail;

    public function __construct(MessageGenerator $messageGenerator, \Swift_Mailer $mailer, $adminEmail)
    {
        $this->adminEmail = $adminEmail;
    }

    public function notifyOfSiteUpdate()
    {
        $message = \Swift_Message::newInstance()
            ->setTo($this->adminEmail)
    }
}
```
*app/config/services.php*

```php
use AppBundle\Controller\LuckyController;
use Symfony\Component\DependencyInjection\Reference;

// To use as default template
$definition = new Definition();

$definition
    ->setAutowired(true)
    ->setAutoconfigured(true)
    ->setPublic(false);

// $this is a reference to the current loader
$this->registerClasses($definition, 'AppBundle\\', '../../src/AppBundle/*', '../../src/AppBundle/{Entity,Repository}');

$container->register(LuckyController::class)
    ->setPublic(true)
    ->setBindings([
        '$logger' => new Reference('monolog.logger.doctrine'),
    ])
```

## 创建服务

创建位置：src/AppBundle/Service/

*src/AppBundle/Service/MessageGenerator.php*

```php
namespace AppBundle\Service;

class MessageGenerator
{
    public function getHappyMessage()
    {
        $messages = [
            'You did it! You updated the system! Amazing!',
            'That was one of the coolest updates I\'ve seen all day!',
            'Great work! Keep going!',
        ];

        $index = array_rand($messages);

        return $messages[$index];
    }
}
```

## 使用

可以作为注入使用，也可以最为参数使用。普通控制器中还可以使用服务名或服务id。

注入：

```php
namespace AppBundle\Controller;

use Symfony\Component\HttpFoundation\Response;
use Twig\Environment;

class HelloController
{
    private $twig;

    public function __construct(Environment $twig)
    {
        $this->twig = $twig;
    }

    public function indexAction($name)
    {
        $content = $this->twig->render(
            'hello/index.html.twig',
            ['name' => $name]
        );

        return new Response($content);
    }
}
```

参数：

```php
use Psr\Log\LoggerInterface;
// ...

/**
 * @Route("/lucky/number/{max}")
 */
public function numberAction($max, LoggerInterface $logger)
{
    $logger->info('We are logging!');
    // ...
}
```

使用服务id，并用其获取参数：

```php
$logger = $container->get('logger');
$entityManager = $container->get('doctrine.orm.entity_manager');
$someParameter = $this->getParameter('kernel.debug');
```

每个服务的id是其限定的类名。

Symgony3.0中服务id不区分大小写，但不建议这样书写，4.0版本中区分大小写。

查看可使用的服务：

```php
php bin/console debug:autowiring
```

## 服务参数

*app/config/services.yml*

```
parameters:
    admin_email: manager@example.com
services:
    AppBundle\Updates\SiteUpdateManager:
        arguments:
            $adminEmail: '%admin_email%'
```

parameters 中设置参数名和参数值，使用是可以用参数名加%号，如%admin_email%，可以在其他配置文件中这样使用。

也可以定义在parameters.yml文件中。

*从依赖注入中获取*

```php
    class SiteUpdateManager
    {
        private $adminEmail;
        public function __construct($adminEmail)
        {
            $this->adminEmail = $adminEmail;
        }
    }
```

*从服务中获取参数*

```php
public function newAction()
{
    // this ONLY works if you extend Controller
    $adminEmail = $this->container->getParameter('admin_email');

    // or a shorter way!
    $adminEmail = $this->getParameter('admin_email');
}
```

## 选择特定类型

依赖注入的类为接口，被某个类实现，可选择其中的某个实现类作为绑定。

*app/config/services.yml*

```php
services:
    # explicitly configure the service
    AppBundle\Service\MessageGenerator:
        arguments:
            $logger: '@monolog.logger.request'
```

*app/config/services.php*

```php
use AppBundle\Service\MessageGenerator;
use Symfony\Component\DependencyInjection\Reference;

$container->autowire(MessageGenerator::class)
    ->setAutoconfigured(true)
    ->setPublic(false)
    ->setArgument('$logger', new Reference('monolog.logger.request'));
```

*src/AppBundle/Service/MessageGenerator.php*

```php
use Psr\Log\LoggerInterface;

class MessageGenerator
{
    private $logger;

    public function __construct(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }
    // ...
}
```

可以通过名字和类型绑定。

*config/services.yaml*

```php
services:
    _defaults:
        bind:
            # pass this value to any $adminEmail argument for any service
            # that's defined in this file (including controller arguments)
            $adminEmail: 'manager@example.com'

            # pass this service to any $requestLogger argument for any
            # service that's defined in this file
            $requestLogger: '@monolog.logger.request'

            # pass this service for any LoggerInterface type-hint for any
            # service that's defined in this file
            Psr\Log\LoggerInterface: '@monolog.logger.request'
```

*config/services.php*

```php
use App\Controller\LuckyController;
use Psr\Log\LoggerInterface;
use Symfony\Component\DependencyInjection\Reference;

$container->register(LuckyController::class)
    ->setPublic(true)
    ->setBindings([
        '$adminEmail' => 'manager@example.com',
        '$requestLogger' => new Reference('monolog.logger.request'),
        LoggerInterface::class => new Reference('monolog.logger.request'),
    ])
;
```
## 服务类型

AbstractController