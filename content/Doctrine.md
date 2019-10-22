# Doctrine

官网：https://www.doctrine-project.org/

## Doctrine DBAL使用

Doctrine数据库抽象层(DBAL)是一个位于PDO之上的抽象层，它提供了一个直观而灵活的API，用于与最流行的关系数据库通信。换句话说，DBAL库使执行查询和执行其他数据库操作变得很容易。

### 配置

#### 获取连接

可通过Doctrine\DBAL\DriverManager类获取DBAL连接：

```
$config = new \Doctrine\DBAL\Configuration();
$connectionParams = array(
    'dbname' => 'mydb',
    'user' => 'user',
    'password' => 'secret',
    'host' => 'localhost',
    'driver' => 'pdo_mysql',
);
$conn = \Doctrine\DBAL\DriverManager::getConnection($connectionParams, $config);
```

或者使用简化URL:

```
$config = new \Doctrine\DBAL\Configuration();
//..
$connectionParams = array(
    'url' => 'mysql://user:secret@localhost/mydb',
);
$conn = \Doctrine\DBAL\DriverManager::getConnection($connectionParams, $config);
```

DriverManager返回Doctrine\DBAL\Connection实例，该类为底层驱动连接的包容器，通常为PDO实例。

以下内容描述可用的连接参数详情。

**连接用URL**:

最容易的方式为使用数据库URL指定连接参数。指定驱动、用户、密码，在域名和端口之后。

中心部分后面的路径表示数据库的名称，不带前导斜杠。任何查询参数都用作附加的连接参数。

代表驱动程序的方案名称是常规的驱动程序名称(参见下面)，其中的下划线用连字符替换(使它们在URL方案名称中合法)，或者作为别名的下列简化驱动程序名称之一:

格式：驱动名://用户名:密码@域名:端口/数据库名?参数字符串

sqlite连接时数据库名可以是文件路径，也用/开头。


驱动名:
+ db2: ibm_db2别名 
+ mssql: pdo_sqlsrv别名
+ mysql/mysql2: pdo_mysql别名
+ pgsql/postgres/postgresql: pdo_pgsql别名
+ sqlite/sqlite3: pdo_sqlite别名

DoctrineBundle在Symfony应用程序中集成了DBAL和ORM Doctrine项目。

所有配置单数在应用配置文件doctrine键值下：

显示默认配置：> php bin/console config:dump-reference doctrine

显示实际配置：> php bin/console debug:config doctrine

使用xml配置必须使用http://symfony.com/schema/dic/doctrine命名空间，并且相关的XSD模式可从以下地址获得:https://symfony.com/schema/dic/doctrine/doctring-1.0.xsdthe DoctrineBundle在Symfony应用程序中集成了DBAL和ORM Doctrine项目。

### Doctrine DBAL 配置内容

DoctrineBundle支持默认Doctrine驱动接受所有参数，Symfony强制执行的XML或YAML命名标准。
