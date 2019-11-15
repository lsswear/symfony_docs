Symfony数据库连接用第三方组件Doctrine，Doctrine提供工具使数据库交互更容易。

数据库连接参数配置：

*app/config/parameters.yml*

```
parameters:
    database_host:     localhost
    database_name:     test_project
    database_user:     root
    database_password: password
```

也可以在config中设置：

*app/config/config.yml*

```
doctrine:
    dbal:
        driver:   pdo_mysql
        dbname:   Symfony
        user:     root
        password: null
        charset:  UTF8
        server_version: 5.6
```

或者：

```
doctrine:
    dbal:
        driver:   pdo_mysql
        host:     '%database_host%'
        dbname:   '%database_name%'
        user:     '%database_user%'
        password: '%database_password%'
```
*app/config/config.php*

```
$container->loadFromExtension('doctrine', [
    'dbal' => [
        'driver'    => 'pdo_mysql',
        'dbname'    => 'Symfony',
        'user'      => 'root',
        'password'  => null,
        'charset'   => 'UTF8',
        'server_version' => '5.6',
    ],
]);
```

创建测试数据库test_project：php bin/console doctrine:database:create

有时忘记设置数据库字符，默认字符为latin，可修改mysql配置my.cnf文件：

```
[mysqld]
# Version 5.5.3 introduced "utf8mb4", which is recommended
collation-server     = utf8mb4_unicode_ci # Replaces utf8_unicode_ci
character-set-server = utf8mb4            # Replaces utf8
```

或者修改连接配置的字符设置：

*app/config/config.yml*

```
doctrine:
    dbal:
        charset: utf8mb4
        default_table_options:
            charset: utf8mb4
            collate: utf8mb4_unicode_ci
```

若使用SQLite数据库连接文件必须存在：

*app/config/config.yml*

```
doctrine:
    dbal:
        driver: pdo_sqlite
        path: '%kernel.project_dir%/app/sqlite.db'
        charset: UTF8
```

## 创建实体类

*src/AppBundle/Entity/Product.php*

```
namespace AppBundle\Entity;

class Product
{
    private $name;
    private $price;
    private $description;
}
```

这钟类通常称为“实体”，意思为一个保存数据的基类。这个类还不能持久化到数据库中——它只是一个简单的PHP类。

命令行：php bin/console doctrine:generate:entity

Doctrine允许您从数据库中获取整个对象，并将整个对象持久化到数据库中。Doctrine要做到这一点，必须将数据库表映射到特定的PHP类，并且这些表上的列必须映射到相应PHP类上的特定属性。

从"metadata"提供映射关系信息，定义一组规则，准确地告诉Doctrine如何将Product类及其属性映射到特定的数据库表。

备注：

*src/AppBundle/Entity/Product.php*

```
namespace AppBundle\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 * @ORM\Table(name="product")
 */
class Product
{
    /**
     * @ORM\Column(type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    private $id;

    /**
     * @ORM\Column(type="string", length=100)
     */
    private $name;

    /**
     * @ORM\Column(type="decimal", scale=2)
     */
    private $price;

    /**
     * @ORM\Column(type="text")
     */
    private $description;
}
```

YAML:

*src/AppBundle/Resources/config/doctrine/Product.orm.yml*

```
AppBundle\Entity\Product:
    type: entity
    table: product
    id:
        id:
            type: integer
            generator: { strategy: AUTO }
    fields:
        name:
            type: string
            length: 100
        price:
            type: decimal
            scale: 2
        description:
            type: text
```

备注详细内容：https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html#property-mapping

Bundle只能接受一种元数据定义格式。例如，不可能将YAML元数据定义与带注释的PHP实体类定义混合在一起。

表名是可选的，如果省略，将根据实体类的名称自动确定。

当使用另一个使用注释的库或程序(例如Doxygen)时，应该在类上放置@IgnoreAnnotation，以指示Symfony应该忽略哪些注释。表名是可选的，如果省略，将根据实体类的名称自动确定。

````
/**
 * @IgnoreAnnotation("fn")
 */
class Product
````

创建实体之后验证映射关系：php bin/console doctrine:schema:validate

实体类中属性为私有访问，需要设置公有访问get和set方法。

Doctrine持久化特性，可以根据设置的映射关系反向修改数据库表结构：php bin/console doctrine:schema:update --force

迁移可以生成sql语句并将它们存储在迁移类,可以生产服务器上运行系统,以更新和安全可靠地跟踪更改数据库模式。

无论您是否利用了迁移，doctrine:schema:update命令都应该只在开发期间使用。不应该在生产环境中使用它。

## 持久化对象到数据库中

在控制器中使用数据对象：

*src/AppBundle/Controller/DefaultController.php*

```
use AppBundle\Entity\Product;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Component\HttpFoundation\Response;

public function createAction()
{
    // you can fetch the EntityManager via $this->getDoctrine()
    // or you can add an argument to your action: createAction(EntityManagerInterface $entityManager)
    $entityManager = $this->getDoctrine()->getManager();

    $product = new Product();
    $product->setName('Keyboard');
    $product->setPrice(19.99);
    $product->setDescription('Ergonomic and stylish!');

    // tells Doctrine you want to (eventually) save the Product (no queries yet)
    $entityManager->persist($product);

    // actually executes the queries (i.e. the INSERT query)
    $entityManager->flush();

    return new Response('Saved new product with id '.$product->getId());
}

// if you have multiple entity managers, use the registry to fetch them
public function editAction()
{
    $doctrine = $this->getDoctrine();
    $entityManager = $doctrine->getManager();
    $otherEntityManager = $doctrine->getManager('other_connection');
}
```

过程：
+ $this->getDoctrine()->getManager()方法获取Doctrine的实体管理器对象。
+ $product = new Product();获取实体对象，并设置数据
+ $entityManager->persist($product);向实体管理对象设置管理的实体类
+ $entityManager->flush();执行语句，不存在的单数据执行INSERT

flush()失败会抛出Doctrine\ORM\ORMException异常。

## 获取数据

通过id获取数据：

```
public function showAction($productId)
{
    $product = $this->getDoctrine()
        ->getRepository(Product::class)
        ->find($productId);

    if (!$product) {
        throw $this->createNotFoundException(
            'No product found for id '.$productId
        );
    }
}
```

可以使用快捷方式@ParamConverter，根据[FrameworkExtraBundle](https://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/annotations/converters.html)文档。

获取资源对象：$repository = $this->getDoctrine()->getRepository(Product::class);

也可以使用AppBundle:Product语法。可以在任何Doctrine对象中使用，而不是实体类中，如AppBundle\Entity\Product。只要实体类在命名空间在对应的bundle中就可以使用。

Doctrine帮助方法：

```
$repository = $this->getDoctrine()->getRepository(Product::class);

// 通过id找单行数据
$product = $repository->find($productId);

// 根据列值查找单个产品的动态方法名称
$product = $repository->findOneById($productId);
$product = $repository->findOneByName('Keyboard');

// 根据列值查找一组产品的动态方法名称
$products = $repository->findByPrice(19.99);

// 查找所有数据
$products = $repository->findAll();
```

用findBy()和findOneBy()多条件查询：

```
$repository = $this->getDoctrine()->getRepository(Product::class);

// looks for a single product matching the given name and price
$product = $repository->findOneBy([
    'name' => 'Keyboard',
    'price' => 19.99
]);

// looks for multiple products matching the given name, ordered by price
$products = $repository->findBy(
    ['name' => 'Keyboard'],
    ['price' => 'ASC']
);
```

## 修改对象

在控制器中先查询再修改数据：

```
use AppBundle\Entity\Product;

public function updateAction($productId)
{
    $entityManager = $this->getDoctrine()->getManager();
    $product = $entityManager->getRepository(Product::class)->find($productId);

    if (!$product) {
        throw $this->createNotFoundException(
            'No product found for id '.$productId
        );
    }

    $product->setName('New product name!');
    $entityManager->flush();

    return $this->redirectToRoute('homepage');
}
```

修改步骤：
+ 用Doctrine查询数据
+ 修改对象属性
+ 使用实例管理器flush()，更新对象数据

## 删除对象

用实例管理器remove()方法删除数据：

```
$entityManager->remove($product);
$entityManager->flush();
```

## 查询对象

使用repository对象调用基本查询：

```
$repository = $this->getDoctrine()->getRepository(Product::class);

$product = $repository->find($productId);
$product = $repository->findOneByName('Keyboard');
```

Doctrine Query Language (DQL)可以使用复杂查询。DQL类似于SQL，但查询的是实体类的一个或多个对象例如Product，不是表中的行.

## 使用DQL查询对象

```
$query = $entityManager->createQuery(
    'SELECT p
    FROM AppBundle:Product p
    WHERE p.price > :price
    ORDER BY p.price ASC'
)->setParameter('price', 19.99);

$product = $query->setMaxResults(1)->getOneOrNullResult();

$products = $query->getResult();
```

从AppBundle实体中进行查询，命名别名为p。

使用setParameter()设置查询参数，查询参数占位符设置，防止sql注入。

$query->setMaxResults(1)->getOneOrNullResult()仅获取一行。 $query->getResult()获取多行。

[Doctrine Query Language文档](https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/dql-doctrine-query-language.html)

## 使用Doctrine查询生成器查询对象

可以用帮助方法QueryBuilder动态创建查询语句，比DQL更容易阅读。

```
$repository = $this->getDoctrine()->getRepository(Product::class);

// $repository对象使用createQueryBuilder()方法自动从AppBundle:Product查询
// 设置别名为p
$query = $repository->createQueryBuilder('p')
    ->where('p.price > :price')
    ->setParameter('price', '19.99')
    ->orderBy('p.price', 'ASC')
    ->getQuery();
// 多行查询
$products = $query->getResult();
// 单行查询
$product = $query->setMaxResults(1)->getOneOrNullResult();
```


QueryBuilder对象包含构建查询所需的每个方法。通过调用getQuery()方法，查询生成器返回一个普通的查询对象，该对象可用于获取查询的结果。

更多Doctrine Query Builder 文档查询 [Query Builder](https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/query-builder.html)


## 将自定义查询组织到存储库类中

https://symfony.com/doc/3.4/doctrine/repository.html

## 配置

https://symfony.com/doc/3.4/reference/configuration/doctrine.html

## 字段类型推荐

https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html#property-mapping

## 关系和关联

https://symfony.com/doc/3.4/doctrine/associations.html


