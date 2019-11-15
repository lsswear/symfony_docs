## 安装Doctrine

安装Doctrine支持通过Symfony Packs，以及MakerBundle，会生成一些有用代码：

```
> composer require symfony/orm-pack
> composer require --dev symfony/maker-bundle
```

### Symfony Packs

Symfony提供安装包含多个依赖composer包的包。

例如添加应用测试，可以运行：composer require --dev debug 命令。

运行symfony/debug-pack，会安装symfony/debug-bundle、symfony/monolog-bundle、symfony/var-dumper。

默认情况下，安装Symfony包，composer.json会显示包依赖项，而不是实际安装的软件包。

要显示包，在安装时添加 --unpack 选项，例如composer需要debug --dev --unpack。或者运行命令去unpack已经存在的包：composer unpack PACK_NAME,例如composer unpack debug。


## 配置数据库


数据库连接数据存储在环境变量中，叫做DATABASE_URL。

对于开发环境，可以在.env中找到并自定义它:

```
# .env (or override DATABASE_URL in .env.local to avoid committing your changes)

# customize this line!
DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/db_name"

# to use sqlite:
# DATABASE_URL="sqlite:///%kernel.project_dir%/var/app.db"
```

若用户名、密码、主机名、数据库名包含特殊字符，比如“+, @, $, #, /, :, *, !”，必须进行编码。

根据[URI通用语法](https://www.ietf.org/rfc/rfc3986.txt)中用于保留字符的完整列表，或者用urlencode方法编码。

在这种情况下，您需要在config/packages/doctrine中删除resolve:前缀。避免错误:url: '%env(resolve:DATABASE_URL)%'。

现在已经设置了连接参数，Doctrine可以为您创建db_name数据库：

```
> php bin/console doctrine:database:create
```

更多配置选项在config/packages/doctrine.yaml文件中，包含server_version,例如若mysql版本为5.7其值为5.7，版本可能会影响Doctrine运行。

其他Doctrine命令查看： bin/console list doctrine。

## 创建实体类

命令：make:entity 创建需要的实体类和文件。

```
> php bin/console make:entity

Class name of the entity to create or update:
> Product

 to stop adding fields):
> name

Field type (enter ? to see all types) [string]:
> string

Field length [255]:
> 255

Can this field be null in the database (nullable) (yes/no) [no]:
> no

 to stop adding fields):
> price

Field type (enter ? to see all types) [string]:
> integer

Can this field be null in the database (nullable) (yes/no) [no]:
> no

 to stop adding fields):
>
(press enter again to finish)
```

*src/Entity/Product.php*

```
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity(repositoryClass="App\Repository\ProductRepository")
 */
class Product
{
    /**
     * @ORM\Id
     * @ORM\GeneratedValue
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(type="string", length=255)
     */
    private $name;

    /**
     * @ORM\Column(type="integer")
     */
    private $price;

    public function getId()
    {
        return $this->id;
    }

    // ... getter and setter methods
}
```
    
若使用SQLite报错PDOException: SQLSTATE[HY000]: General error: 1 Cannot add a NOT NULL column with default value NULL，向description添加nullable=true解决问题。

## 迁移:创建数据库表/模式

数据库迁移：

```
> php bin/console make:migration
```

回执信息：

```
    SUCCESS!

    Next: Review the new migration "src/Migrations/Version20180207231217.php" Then: Run the migration with php bin/console doctrine:migrations:migrate
```

数据库更新：

```
> php bin/console doctrine:migrations:migrate
```

命令执行在数据库上所有未运行的迁移文件。

在部署生产数据库以使其保持最新时，应该在生产环境中运行此命令。

## 迁移或添加更多字段

可以编辑实体类添加新字段，也可以使用make:entity:

```
> php bin/console make:entity

Class name of the entity to create or update
> Product

 to stop adding fields):
> description

Field type (enter ? to see all types) [string]:
> text

Can this field be null in the database (nullable) (yes/no) [no]:
> no

 to stop adding fields):
>
(press enter again to finish)
```

添加description字段属性和getDescription()和setDescription()方法：

*src/Entity/Product.php*

```
class Product
{
    // ...

+     /**
+      * @ORM\Column(type="text")
+      */
+     private $description;

    // getDescription() & setDescription() were also added
}
```

这是新加的字段并未创建到数据库中，需执行迁移命令：

```
> php bin/console make:migration
```

执行的sql为：

```
ALTER TABLE product ADD description LONGTEXT NOT NULL
```

或执行

```
> php bin/console doctrine:migrations:migrate
```

将执行一个新的迁移文件，因为DoctrineMigrationsBundle知道第一次迁移已经提前执行，它用migration_versions管理迁移过的文件。

每次对模式进行更改时，运行这两个命令来生成迁移，然后执行迁移。请确保提交迁移文件并在部署时执行它们。

若手动添加新属性，可执行make:entity命令生成get和set方法。

```
> php bin/console make:entity --regenerate
```

如果您做了一些更改，并且想要重新生成所有的getter/setter方法，那么还要传递 --overwrite。

## 解析对象到数据库

创建编辑对象的控制器：

```
> php bin/console make:controller ProductController
```

*src/Controller/ProductController.php*

```
namespace App\Controller;

use App\Entity\Product;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Component\HttpFoundation\Response;

class ProductController extends AbstractController
{
    /**
     * @Route("/product", name="create_product")
     */
    public function createProduct(): Response
    {
        // you can fetch the EntityManager via $this->getDoctrine()
        // or you can add an argument to the action: createProduct(EntityManagerInterface $entityManager)
        $entityManager = $this->getDoctrine()->getManager();

        $product = new Product();
        $product->setName('Keyboard');
        $product->setPrice(1999);
        $product->setDescription('Ergonomic and stylish!');

        // tell Doctrine you want to (eventually) save the Product (no queries yet)
        $entityManager->persist($product);

        // actually executes the queries (i.e. the INSERT query)
        $entityManager->flush();

        return new Response('Saved new product with id '.$product->getId());
    }
}
```

根据以上内容，访问"http://localhost:8000/product"可创建一行数据。

在命令行中执行查询：

```
> php bin/console doctrine:query:sql 'SELECT * FROM product'
```

上述例子代码解析：

$this->getDoctrine()->getManager()：获取Doctrine管理实例对象，这是Doctrine中最重要的对象。可用于保存、更新对象到数据库。

product对象和其他对象一样使用。

persist($product) 使用Doctrine管理product对象，不会导致数据库进行查询。

flush()检查所有管理对象是否需要持久化到数据库。对象不在数据库中则管理实例执行INSERT。

若flush()失败，会抛出Doctrine\ORM\ORMException异常，查看[事务和并发](https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/transactions-and-concurrency.html)。

## 校验对象

[symfony验证器](https://symfony.com/doc/current/validation.html)重用Document元数据执行基本验证任务：

*src/Controller/ProductController.php*

```
namespace App\Controller;

use App\Entity\Product;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Validator\Validator\ValidatorInterface;
// ...

class ProductController extends AbstractController
{
    /**
     * @Route("/product", name="create_product")
     */
    public function createProduct(ValidatorInterface $validator): Response
    {
        $product = new Product();
        // This will trigger an error: the column isn't nullable in the database
        $product->setName(null);
        // This will trigger a type mismatch error: an integer is expected
        $product->setPrice('1999');

        // ...

        $errors = $validator->validate($product);
        if (count($errors) > 0) {
            return new Response((string) $errors, 400);
        }

        // ...
    }
}
```

虽然Product实例未显示定义在[验证器配置](https://symfony.com/doc/current/validation.html)中,symfony会检查Doctrine映射配置推断一些验证规则。

例如，数据库设置name字段属性不可为null，NotNull约束自动添加到属性，若它还没包含那个约束。

根据Doctrine元数据和相应验证约束通过symfony自动添加：

**nullable** 

默认：false

验证约束：[NotNull](https://symfony.com/doc/current/reference/constraints/NotNull.html)

备注：需要引入[PropertyInfo component](https://symfony.com/doc/current/components/property_info.html)

**type**

验证约束：[type](https://symfony.com/doc/current/reference/constraints/Type.html)

备注：需要引入[PropertyInfo component](https://symfony.com/doc/current/components/property_info.html)

**unique**

验证约束：[UniqueEntity](https://symfony.com/doc/current/reference/constraints/UniqueEntity.html)

**length**

验证约束：[Length](https://symfony.com/doc/current/reference/constraints/Length.html)

因为[表单组件](https://symfony.com/doc/current/forms.html)和[API平台](https://api-platform.com/docs/core/validation/)内部使用验证组件， 所有表单和wepAPI都会自动收益于这些验证约束。

4.3版本已加入自动验证。

## 从数据库更新对象

*src/Controller/ProductController.php*

```
/**
 * @Route("/product/{id}", name="product_show")
 */
public function show($id)
{
    $product = $this->getDoctrine()
        ->getRepository(Product::class)
        ->find($id);

    if (!$product) {
        throw $this->createNotFoundException(
            'No product found for id '.$id
        );
    }

    return new Response('Check out this great product: '.$product->getName());

    // or render a template
    // in the template, print things with {{ product.name }}
    // return $this->render('product/show.html.twig', ['product' => $product]);
}
```

```
$repository = $this->getDoctrine()->getRepository(Product::class);

// look for a single Product by its primary key (usually "id")
$product = $repository->find($id);

// look for a single Product by name
$product = $repository->findOneBy(['name' => 'Keyboard']);
// or find by name and price
$product = $repository->findOneBy([
    'name' => 'Keyboard',
    'price' => 1999,
]);

// look for multiple Product objects matching the name, ordered by price
$products = $repository->findBy(
    ['name' => 'Keyboard'],
    ['price' => 'ASC']
);

// look for *all* Product objects
$products = $repository->findAll();

```

可以使用自定义方法处理复杂查询，参照内容：Repository查询对象。
 
## 自动获取对象，参数转换

可以使用[SensioFrameworkExtraBundle](https://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/index.html)自动查询。

引入 SensioFrameworkExtraBundle：

```
> composer require sensio/framework-extra-bundle
```

*src/Controller/ProductController.php*

```
use App\Entity\Product;

/**
 * @Route("/product/{id}", name="product_show")
 */
public function show(Product $product)
{
    // use the Product
}
```

使用{id}查询Product,若，若没找到生成404页面。

其他参数设置参照：[ParamConverter](https://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/annotations/converters.html)

## 更新对象

已存在获取对象的Doctrine实例，交互使用PHP模型：

```
/**
 * @Route("/product/edit/{id}")
 */
public function update($id)
{
    $entityManager = $this->getDoctrine()->getManager();
    $product = $entityManager->getRepository(Product::class)->find($id);

    if (!$product) {
        throw $this->createNotFoundException(
            'No product found for id '.$id
        );
    }

    $product->setName('New product name!');
    $entityManager->flush();

    return $this->redirectToRoute('product_show', [
        'id' => $product->getId()
    ]);
}
```

使用Doctrine编辑和执行：

+ 从Doctrine获取对象
+ 修改对象
+ 实例对象使用flush()方法

可以使用$entityManager->persist($product)，但没必要，Doctrine已经查看了对象的修改内容。

## 删除对象

```
$entityManager->remove($product);
$entityManager->flush();
```

使用remove()仅通知Doctrine，并非实际操作，直到使用flush()方法DELETE才会执行。

