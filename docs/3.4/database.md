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

