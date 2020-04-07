# Doctrine 事件

Doctrine是Symfony用于处理数据库的一组php库，提供一个轻量的事件系统，在应用执行期间更改实体对象，这些事件成为生命周期事件。

Doctrine触发事件之前或之后执行最常见的实体操作，例如prePersist/postPersist、preUpdate/postUpdate,和其他常见任务，例如 loadClassMetadata,onClear。

不同的方式学习Doctrine事件：

+ 生命周期回调： 定义为实体类上的方法，并在事件触发时调用。    
+ 生命周期监听和订阅器：有一个或多个事件回调方法的类，可以被所有实体调用。
+ 事件监听：类似于生命周期监听，仅被特定的实体类调用。

缺点和优点如下：

+ 回调有更好的性能，应为只应用于一个实体类，但是不同实体不能重用逻辑，不能访问Symfony服务。
+ 生命周期监听和订阅可以重用逻辑在不同实体之间，并且可以访问Symfony服务，但是性能不好，因为他们被所有实体调用。
+ 事件监听和生命周期监听有相同的优点，有更好的性能，因为他们只被一个实体调用。

文章仅解释当在Symfony应用中使用Doctrine事件的基础内容，可以查看[更多内容](https://www.doctrine-project.org/projects/doctrine-orm/en/2.6/reference/events.html)。

MongoDB使用ODM，阅读[DoctrineMongoDB Bundle文档](https://symfony.com/doc/current/bundles/DoctrineMongoDBBundle/index.html)。

## Doctrine 生命周期回调

生命周期回调定义为要修改的实体的方法，假如想将createdAt设置为当前日期，只有当实体第一次插入是调用。

定义Doctrine prePersist 回调：

*src/Entity/Product.php*

```
use Doctrine\ORM\Mapping as ORM;

// When using annotations, don't forget to add @ORM\HasLifecycleCallbacks()
// to the class of the entity where you define the callback

/**
 * @ORM\Entity()
 * @ORM\HasLifecycleCallbacks()
 */
class Product
{
    /**
     * @ORM\PrePersist
     */
    public function setCreatedAtValue()
    {
        $this->createdAt = new \DateTime();
    }
}
```

一些回调函数接收参数用于提供刚问有用信息，例如当前实体的管理类。

## 生命周期监听器

生命周期监听定义为监听所有应用实体的一个Doctrine事件的php类。

假设持久化实体时要更新摸个搜索索引。

因此为Doctrine postPersist 定义监听器：

*src/EventListener/SearchIndexer.php*

```
namespace App\EventListener;

use App\Entity\Product;
use Doctrine\Common\Persistence\Event\LifecycleEventArgs;

class SearchIndexer
{
    // the listener methods receive an argument which gives you access to
    // both the entity object of the event and the entity manager itself
    public function postPersist(LifecycleEventArgs $args)
    {
        $entity = $args->getObject();

        // if this listener only applies to certain entity types,
        // add some code to check the entity type as early as possible
        if (!$entity instanceof Product) {
            return;
        }

        $entityManager = $args->getObjectManager();
        // ... do something with the Product entity
    }
}
```

通过创建新服务和doctrine.event_listener标签应用监听类：

*config/services.yaml*

```
    services:
        # ...
    
        App\EventListener\SearchIndexer:
            tags:
                -
                    name: 'doctrine.event_listener'
                    # 生命周期监听标签唯一需要的参数
                    event: 'postPersist'
                    
                    # 监听器可以定义优先级，以防多个监听器监听监听同一个事件
                    # 默认default priority = 0; 优先级更高运行更快。
                    priority: 500
    
                    # 可以限制监听器具体的Doctrine内容
                    connection: 'default'
```

*config/services.php*

```
use App\EventListener\SearchIndexer;

// listeners are applied by default to all Doctrine connections
$container->autowire(SearchIndexer::class)
    ->addTag('doctrine.event_listener', [
        // this is the only required option for the lifecycle listener tag
        'event' => 'postPersist',

        // listeners can define their priority in case multiple listeners are associated
        // to the same event (default priority = 0; higher numbers = listener is run earlier)
        'priority' => 500,

        # you can also restrict listeners to a specific Doctrine connection
        'connection' => 'default',
    ]);
```

Symfony只在实际触发相关事件时才加载并实例化Doctrine监听器；而Doctrine订阅者由Symfony加载并实例化，这使它们性能更低。

## 实体监听器

实体监听类定义为一个监听实体类的一个Doctrine事件的php类。

假如，想要所有数据库操作的日志。为Doctrine的 postPersist、postRemove、postUpdate事件设置订阅者：

*src/EventListener/DatabaseActivitySubscriber.php*

```
namespace App\EventListener;

use App\Entity\Product;
use Doctrine\Common\EventSubscriber;
use Doctrine\Common\Persistence\Event\LifecycleEventArgs;
use Doctrine\ORM\Events;

class DatabaseActivitySubscriber implements EventSubscriber
{
    // this method can only return the event names; you cannot define a
    // custom method name to execute when each event triggers
    public function getSubscribedEvents()
    {
        return [
            Events::postPersist,
            Events::postRemove,
            Events::postUpdate,
        ];
    }

    // callback methods must be called exactly like the events they listen to;
    // they receive an argument of type LifecycleEventArgs, which gives you access
    // to both the entity object of the event and the entity manager itself
    public function postPersist(LifecycleEventArgs $args)
    {
        $this->logActivity('persist', $args);
    }

    public function postRemove(LifecycleEventArgs $args)
    {
        $this->logActivity('remove', $args);
    }

    public function postUpdate(LifecycleEventArgs $args)
    {
        $this->logActivity('update', $args);
    }

    private function logActivity(string $action, LifecycleEventArgs $args)
    {
        $entity = $args->getObject();

        // if this subscriber only applies to certain entity types,
        // add some code to check the entity type as early as possible
        if (!$entity instanceof Product) {
            return;
        }
    }
}
```

创建doctrine.event_subscriber标签的新服务使Doctrine订阅器在Symfony应用中可用。

*config/services.yaml*

```
services:
    App\EventListener\DatabaseActivitySubscriber:
        tags:
            - { name: 'doctrine.event_subscriber' }
```

*config/services.php*

```
use App\EventListener\DatabaseActivitySubscriber;

$container->autowire(DatabaseActivitySubscriber::class)
    ->addTag('doctrine.event_subscriber');
```

若想通过特殊的Doctrine容器访问订阅器，可以设置服务配置：

*config/services.yaml*

```
services:
    App\EventListener\DatabaseActivitySubscriber:
        tags:
            - { name: 'doctrine.event_subscriber', connection: 'default' }
```

*config/services.php*

```
use App\EventListener\DatabaseActivitySubscriber;

$container->autowire(DatabaseActivitySubscriber::class)
    ->addTag('doctrine.event_subscriber', ['connection' => 'default']);
```

程序执行时，Symfony加载和初始化Doctrine订阅器；Doctrine监听器仅在相关事件触发时加载，这使它们性能更好。