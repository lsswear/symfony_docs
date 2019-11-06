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