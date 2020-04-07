# 多个实体类管理和连接

可以使用Doctrine管理或连接多个实体。对于使用的是不同的数据库，甚至完全不同实体集的供应商，是必要的。

使用多个实例管理配置不复杂，但是更先进，不需要设置。

配置两个实体管理：

*config/packages/doctrine.yaml*

```
doctrine:
    dbal:
        default_connection: default
        connections:
            default:
                # configure these for your database server
                url: '%env(DATABASE_URL)%'
                driver: 'pdo_mysql'
                server_version: '5.7'
                charset: utf8mb4
            customer:
                # configure these for your database server
                url: '%env(DATABASE_CUSTOMER_URL)%'
                driver: 'pdo_mysql'
                server_version: '5.7'
                charset: utf8mb4

    orm:
        default_entity_manager: default
        entity_managers:
            default:
                connection: default
                mappings:
                    Main:
                        is_bundle: false
                        type: annotation
                        dir: '%kernel.project_dir%/src/Entity/Main'
                        prefix: 'App\Entity\Main'
                        alias: Main
            customer:
                connection: customer
                mappings:
                    Customer:
                        is_bundle: false
                        type: annotation
                        dir: '%kernel.project_dir%/src/Entity/Customer'
                        prefix: 'App\Entity\Customer'
                        alias: Customer
```

*config/packages/doctrine.php*

```
$container->loadFromExtension('doctrine', [
    'dbal' => [
        'default_connection' => 'default',
        'connections' => [
            // configure these for your database server
            'default' => [
                'url'            => '%env(DATABASE_URL)%',
                'driver'         => 'pdo_mysql',
                'server_version' => '5.7',
                'charset'        => 'utf8mb4',
            ],
            // configure these for your database server
            'customer' => [
                'url'            => '%env(DATABASE_CUSTOMER_URL)%',
                'driver'         => 'pdo_mysql',
                'server_version' => '5.7',
                'charset'        => 'utf8mb4',
            ],
        ],
    ],

    'orm' => [
        'default_entity_manager' => 'default',
        'entity_managers' => [
            'default' => [
                'connection' => 'default',
                'mappings'   => [
                    'Main'  => [
                        is_bundle => false,
                        type => 'annotation',
                        dir => '%kernel.project_dir%/src/Entity/Main',
                        prefix => 'App\Entity\Main',
                        alias => 'Main',
                    ]
                ],
            ],
            'customer' => [
                'connection' => 'customer',
                'mappings'   => [
                    'Customer'  => [
                        is_bundle => false,
                        type => 'annotation',
                        dir => '%kernel.project_dir%/src/Entity/Customer',
                        prefix => 'App\Entity\Customer',
                        alias => 'Customer',
                    ]
                ],
            ],
        ],
    ],
]);
```

定义两个实体管理default和customer，并定义了两个连接。 default实体类文件在App\Entity\Main，customer实体类在App\Entity\Customer。

在使用多个连接和实体管理器时，如果确实省略了连接或实体管理器的名称，则使用缺省值(即default)。

如果你使用默认的实体管理器不同于默认的名称，你也需要在prod环境配置中重新定义默认的实体管理器:

*config/packages/prod/doctrine.yaml*

```
doctrine:
    orm:
        default_entity_manager: 'your default entity manager name'
```

使用多个连接创建数据库：

```
> php bin/console doctrine:database:create

> php bin/console doctrine:database:create --connection=customer
```

多个实体管理生成迁移：

```
> php bin/console doctrine:migrations:diff
> php bin/console doctrine:migrations:migrate

> php bin/console doctrine:migrations:diff --em=customer
> php bin/console doctrine:migrations:migrate --em=customer
```

当调用时忽略实例管理名称则返回默认的实体管理。

```
use Doctrine\ORM\EntityManagerInterface;

class UserController extends AbstractController
{
    public function index(EntityManagerInterface $entityManager)
    {
        // These methods also return the default entity manager, but it's preferred
        // to get it by injecting EntityManagerInterface in the action method
        $entityManager = $this->getDoctrine()->getManager();
        $entityManager = $this->getDoctrine()->getManager('default');
        $entityManager = $this->get('doctrine.orm.default_entity_manager');

        // Both of these return the "customer" entity manager
        $customerEntityManager = $this->getDoctrine()->getManager('customer');
        $customerEntityManager = $this->get('doctrine.orm.customer_entity_manager');
    }
}
```

您现在可以像以前一样使用Doctrine—使用默认的实体管理器来持久化和获取它所管理的实体，使用客户实体管理器持久化和获取它的实体。


这同样适用于存储库调用:

```
use AcmeStoreBundle\Entity\Customer;
use AcmeStoreBundle\Entity\Product;
// ...

class UserController extends AbstractController
{
    public function index()
    {
        // Retrieves a repository managed by the "default" em
        $products = $this->getDoctrine()
            ->getRepository(Product::class)
            ->findAll()
        ;

        // Explicit way to deal with the "default" em
        $products = $this->getDoctrine()
            ->getRepository(Product::class, 'default')
            ->findAll()
        ;

        // Retrieves a repository managed by the "customer" em
        $customers = $this->getDoctrine()
            ->getRepository(Customer::class, 'customer')
            ->findAll()
        ;
    }
}
```