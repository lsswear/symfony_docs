# 根据已存在数据库创建实体类

## Doctrine Associations / Relations

两种关联类型：

+ ManyToOne/OneToMany
    大多数关联基于外键，这实际上只是一种关联类型，但是从关系的两个不同方面来看。

+ ManyToMany
    使用联接表，当关系的双方都可以拥有许多另一方时，需要使用联接表。

### ManyToOne OneToMany 关系

应用中每个产品都属于一个类别。

需要一个Category类关联Product表。

创建Category实体类和name字段：

```
> php bin/console make:entity Category

 to stop adding fields):
> name

Field type (enter ? to see all types) [string]:
> string

Field length [255]:
> 255

Can this field be null in the database (nullable) (yes/no) [no]:
> no

 to stop adding fields):
>
(press enter again to finish)
```

生成实体类：

*src/Entity/Category.php*

```
class Category
{
    /**
     * @ORM\Id
     * @ORM\GeneratedValue
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(type="string")
     */
    private $name;

    // ... getters and setters
}
```

### 映射多对一的关系

例如每个类型可以关联多个产品，但一个产品只能对应一个类型。可以总结为：多个产品对应一个类。

从产品实体类的角度看是many-to-one关系。从类的实体类角度看是one-to-many关系。

要映射它，首先在Product类上创建一个带有ManyToOne注释的category属性。

可以手动设置，或使用 make:entity命令。make:entity可修改设置：

```
> php bin/console make:entity

Class name of the entity to create or update (e.g. BraveChef):
> Product

 to stop adding fields):
> category

Field type (enter ? to see all types) [string]:
> relation

What class should this entity be related to?:
> Category

Relation type? [ManyToOne, OneToMany, ManyToMany, OneToOne]:
> ManyToOne

Is the Product.category property allowed to be null (nullable)? (yes/no) [yes]:
> no

Do you want to add a new property to Category so that you can access/update
getProducts()? (yes/no) [yes]:
> yes

New field name inside Category [products]:
> products

Do you want to automatically delete orphaned App\Entity\Product objects
(orphanRemoval)? (yes/no) [no]:
> no

 to stop adding fields):
>
(press enter again to finish)
```

需要改两个实体类。需要在product类添加category属性：

*src/Entity/Product.php*

```
class Product
{
    // ...

    /**
     * @ORM\ManyToOne(targetEntity="App\Entity\Category", inversedBy="products")
     */
    private $category;

    public function getCategory(): ?Category
    {
        return $this->category;
    }

    public function setCategory(?Category $category): self
    {
        $this->category = $category;

        return $this;
    }
}

```

这是多对一映射必须的。它告诉Doctrine使用product表上的category_id列将该表中的每个记录与category表中的一个记录关联起来。

一个Category对象关联多个Product对象，make:entity命令也在Category类添加products属性：

*src/Entity/Category.php*

```
use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\Common\Collections\Collection;

class Category
{
    // ...

    /**
     * @ORM\OneToMany(targetEntity="App\Entity\Product", mappedBy="category")
     */
    private $products;

    public function __construct()
    {
        $this->products = new ArrayCollection();
    }

    /**
     * @return Collection|Product[]
     */
    public function getProducts(): Collection
    {
        return $this->products;
    }

    // addProduct() and removeProduct() were also added
}
```

多对一关系必须写在开始位置，但一对多关系是可选的：若想通过与某个类相关的产品时才能访问某个类。

ArrayCollection使对象属性像数组一样使用。

数据库迁移：

```
 php bin/console doctrine:migrations:diff
 php bin/console doctrine:migrations:migrate
```

由于关系的设置，会在product表创建category_id 外键。

### 保存相关的实体类

```
use App\Entity\Category;
use App\Entity\Product;
use Symfony\Component\HttpFoundation\Response;

class ProductController extends AbstractController
{
    /**
     * @Route("/product", name="product")
     */
    public function index()
    {
        $category = new Category();
        $category->setName('Computer Peripherals');

        $product = new Product();
        $product->setName('Keyboard');
        $product->setPrice(19.99);
        $product->setDescription('Ergonomic and stylish!');

        // relates this product to the category
        $product->setCategory($category);

        $entityManager = $this->getDoctrine()->getManager();
        $entityManager->persist($category);
        $entityManager->persist($product);
        $entityManager->flush();

        return new Response(
            'Saved new product with id: '.$product->getId()
            .' and new category with id: '.$category->getId()
        );
    }
}
```

使ORM可以仅考虑数据实体对象而不考虑数据库。

代替向Product对象设置category_id，而是设置Category对象。

改变关系从关系被维护端：使用$category->addProduct()。

### 获取关系对象

获取对象流程：获取Product对象再访问Category对象：

```
use App\Entity\Product;
public function show($id)
{
    $product = $this->getDoctrine()
        ->getRepository(Product::class)
        ->find($id);
    $categoryName = $product->getCategory()->getName();
}
```

先根据product_id查询Product对象，使用方法$product->getCategory()->getName()，Doctrine第二次查询Product对象对应Category对象。

可以通过产品对象访问产品类对象，但在仅向产品对象调用产品类对象时才会实际查询。

因为实现了OneToMany，可以采用另一种方法实现：

````
public function showProducts($id)
{
    $category = $this->getDoctrine()
        ->getRepository(Category::class)
        ->find($id);

    $products = $category->getProducts();
}
````

#### 关系和代理类

懒加载实现Doctrine返回实际“代理”对象：

```
$product = $this->getDoctrine()
    ->getRepository(Product::class)
    ->find($id);

$category = $product->getCategory();

// prints "Proxies\AppEntityCategoryProxy"
dump(get_class($category));
die();
```

使用代理对象，Doctrine可以延迟查询Category数据，直到真正调用查询语句。

代理类由Doctrine生成，并储存在缓存文件。使用join语句则不使用懒加载。

### 加入相关记录

如果预先知道需要访问两个对象，可以通过在原始查询中发出联接来避免第二个查询。将以下方法添加到ProductRepository类：

*src/Repository/ProductRepository.php*

```
public function findOneByIdJoinedToCategory($productId)
{
    $entityManager = $this->getEntityManager();

    $query = $entityManager->createQuery(
        'SELECT p, c
        FROM App\Entity\Product p
        INNER JOIN p.category c
        WHERE p.id = :id'
    )->setParameter('id', $productId);

    return $query->getOneOrNullResult();
}
```

会返回Product对象，想在可以使用$product->getCategory()并使用数据，没有第二次查询。

现在可以在控制器中使用查询方法一次查询获取Product对象和与之关联的Category对象：

```
public function show($id)
{
    $product = $this->getDoctrine()
        ->getRepository(Product::class)
        ->findOneByIdJoinedToCategory($id);

    $category = $product->getCategory();
}
```

### 从反方向设置信息

在数据库中修改关系，必须在拥有方设置关系。

多对一关系映射设置总是在拥有方，例如多对多关系可以选择拥有方。

*src/Entity/Category.php*

```
class Category
{
    public function addProduct(Product $product): self
    {
        if (!$this->products->contains($product)) {
            $this->products[] = $product;
            $product->setCategory($this);
        }
        return $this;
    }
}
```

若想执行关联删除：

*src/Entity/Category.php*

```
class Category
{
    public function removeProduct(Product $product): self
    {
        if ($this->products->contains($product)) {
            $this->products->removeElement($product);
            // set the owning side to null (unless already changed)
            if ($product->getCategory() === $this) {
                $product->setCategory(null);
            }
        }
        return $this;
    }
}
```

若调用$category->removeProduct($product)，则Product中对应category_id会被设置为null。

若想删除对应对象，需设置orphanRemoval参数：

*src/Entity/Category.php*

```
   /**
    * @ORM\OneToMany(targetEntity="App\Entity\Product", mappedBy="category", orphanRemoval=true)
    */
   private $products; 
```

***

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

        // ... get the entity information and log it somehow
    }
}
```
为Doctrine