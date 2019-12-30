# Doctrine DBAL 使用

Doctrine抽象层DBAL(Doctrine Database Abstraction Layer)在PDO顶部提供直观和灵活的api与最流行的关系数据库进行通信。
 
DBAL库允许独立于ORM模型编写查询，例如创建报告或直接数据库操作。

[DBAL文档](https://www.doctrine-project.org/projects/doctrine-dbal/en/latest/index.html)

安装Doctrine orm Symfony包：composer require symfony/orm-pack

.env环境变量配置DATABASE_URL:

```
# .env (or override DATABASE_URL in .env.local to avoid committing your changes)

# customize this line!
DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/db_name"
```

在config/packages/doctrine.yaml中配置，具体查看[Doctrine DBAL 配置](https://symfony.com/doc/4.3/reference/configuration/doctrine.html#reference-dbal-configuration)。

不使用Doctrine orm 删除文件中orm键。

可以访问Doctrine DBAL通过Connection对象：

```
use Doctrine\DBAL\Driver\Connection;

class UserController extends AbstractController
{
    public function index(Connection $connection)
    {
        $users = $connection->fetchAll('SELECT * FROM users');
    }
}
```

会通过database_connection服务。

## 注册自定义映射类型

可以自定义映射类型通过Symfony配置。可以添加到所有配置的连接中。

所有映射类型信息，查看详细[自定义映射类型](https://www.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/types.html#custom-mapping-types)。

*config/packages/doctrine.yaml*

```
doctrine:
    dbal:
        types:
            custom_first:  App\Type\CustomFirst
            custom_second: App\Type\CustomSecond
```

*config/packages/doctrine.php*

```
use App\Type\CustomFirst;
use App\Type\CustomSecond;

$container->loadFromExtension('doctrine', [
    'dbal' => [
        'types' => [
            'custom_first'  => CustomFirst::class,
            'custom_second' => CustomSecond::class,
        ],
    ],
]);
```

## 在SchemaTool中注册自定义映射类型

SchemaTool用于检查比较模式的数据库。需要知道所有数据库类型需要的映射类型。

可以通过配置类注册类型。将ENUM类型(默认情况下不支持DBAL)映射到字符串映射类型。

*config/packages/doctrine.yaml*

```
doctrine:
    dbal:
        mapping_types:
            enum: string
```

*config/packages/doctrine.php*

```
$container->loadFromExtension('doctrine', [
    'dbal' => [
        'mapping_types' => [
            'enum'  => 'string',
        ],
    ],
]);
```