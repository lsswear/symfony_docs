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

resource为资源内容，exclude为排除项。

服务的id为配置项中的键值。

通过资源加载可以将资源中对应的类作为服务使用，仅注册其中显示定义为服务的类。

服务id覆盖，两个服务的配置没有继承关系，但都继承_defaults。

exclude会提高开发性能，但修改其中对应文件内容不影响容器，即不会导致容器重新构建。

配置文件中public为false时，意思是注册的服务为私有。

默认情况下服务的命名空间是其对应配置的建值。 不同资源建值但命名空间相同的情况下，用tags区别。

可以为服务配置常量，相同的类需设置不同的id。为确保服务的正确调用，尽量确保服务id唯一。

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

服务id：

```php
$logger = $container->get('logger');
$entityManager = $container->get('doctrine.orm.entity_manager');
```

每个服务的id是其限定的类名。

Symgony3.0中服务id不区分大小写，但不建议这样书写，4.0版本中区分大小写。

查看可使用的服务：

```php
php bin/console debug:autowiring
```

## 服务类型

AbstractController