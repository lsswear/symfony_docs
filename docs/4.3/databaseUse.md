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







