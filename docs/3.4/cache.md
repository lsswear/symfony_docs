## 基础使用

```
if (!$cache->has('my_cache_key')) {
    // ... do some HTTP request or heavy computations
    $cache->set('my_cache_key', 'foobar', 3600);
}

echo $cache->get('my_cache_key'); // 'foobar'

// ... and to remove the cache key
$cache->delete('my_cache_key');
```

Symfony支持PSR-6和PSR-16缓存接口。

## 安装
 
composer require symfony/cache

## 缓存关于FrameworkBundle配置

**Pool**
    
服务引入配置。每个pool有自己的命名空间和缓存条目。pool之间不会有冲突。

**Adapter**

一个适配就是一个模板，用于创建pool。

**Provider**

提供者是一些适配器用于连接到存储的服务。Redis和Memcached就是这样的适配器。如果使用DSN作为提供者，则会自动创建服务。

以下两个pool定义。cache.app和cache.system。System缓存用于注释、序列化、验证。可对其进行配置。

*app/config/config.yml*

```
framework:
    cache:
        app: cache.adapter.filesystem
        system: cache.adapter.system
```

*app/config/config.php*

```
$container->loadFromExtension('framework', [
    'cache' => [
        'app' => 'cache.adapter.filesystem',
        'system' => 'cache.adapter.system',
    ],
]);
```

已存在缓存：

+ [cache.adapter.apcu][#cache.adapter.apcu]
+ cache.adapter.doctrine
+ cache.adapter.filesystem
+ cache.adapter.memcached
+ cache.adapter.pdo
+ cache.adapter.redis
+ PHPFileAdapter
+ PHPArrayAdapter
+ ChainAdapter
+ ProxyAdapter
+ cache.adapter.psr6
+ cache.adapter.system
+ NullAdapter

### cache.adapter.apcu

该适配器是高性能的，共享高速缓存。可明显提高应用性能，因为它缓存放于共享内存中，比其他组件快，比如文件系统

APCu使用必须安装并使用该适配。

ApcuAdapter可配置namespace、defaultLifetime、version：

```
use Symfony\Component\Cache\Adapter\ApcuAdapter;

$cache = new ApcuAdapter(

    // 缓存存储关键字的前缀
    $namespace = '',

    // 没有缓存条目默认生存时间（秒）值为0，为无限期生存时间，直到APCu缓存被清除
    $defaultLifetime = 0,

    // 该值设置后$namespace的设置可能无效
    // this $version string
    $version = null
);
```

在写/删除繁重的工作负载时，不建议使用此适配器，因为这些操作会导致内存碎片，从而导致性能显著下降

这个适配器的CRUD操作特定于它所运行的PHP SAPI。这意味着使用CLI的缓存操作(如添加、删除等)在FPM或CGI SAPIs下不可用。

### cache.adapter.doctrine

该适配器包装提供Doctrine缓存扩展抽象类，可以使用这些类在symfony的缓存适配器中。

该适配器第一个参数为\Doctrine\Common\Cache\CacheProvider类的实例，和可选参数namespace以及defaultLifetime。

```
use Doctrine\Common\Cache\CacheProvider;
use Doctrine\Common\Cache\SQLite3Cache;
use Symfony\Component\Cache\Adapter\DoctrineAdapter;

$provider = new SQLite3Cache(new \SQLite3(__DIR__.'/cache/data.sqlite'), 'youTableName');

$cache = new DoctrineAdapter(

    // 缓存提供实例
    CacheProvider $provider,

    // a string prefixed to the keys of the items stored in this cache
    $namespace = '',

    // the default lifetime (in seconds) for cache items that do not define their
    // own lifetime, with a value 0 causing items to be stored indefinitely (i.e.
    // until the database table is truncated or its rows are otherwise deleted)
    $defaultLifetime = 0
);
```

应该是用于自定义缓存适配器

### cache.adapter.filesystem

该适配器为不能引入APCu或者Redis环境应用提高性能。

它将缓存项过期和内容作为常规文件存储在本地挂载的文件系统的目录集合中。

通过使用临时内存文件系统，例如Linux的tmpfs，或其他可用RAM磁盘，可大幅提高此适配器性能。

可使用参数：namespace、defaultLifetime、directory

```
use Symfony\Component\Cache\Adapter\FilesystemAdapter;

$cache = new FilesystemAdapter(

    // a string used as the subdirectory of the root cache directory, where cache
    // items will be stored
    $namespace = '',

    // the default lifetime (in seconds) for cache items that do not define their
    // own lifetime, with a value 0 causing items to be stored indefinitely (i.e.
    // until the files are deleted)
    $defaultLifetime = 0,

    // 主缓存文件，系统需要开启read-write权限，若未指定，则会创建临时文件夹
    $directory = null
);
```

文件系统IO的开销常常使这个适配器成为较慢的选择之一。如果吞吐量是最重要的，那么建议使用内存适配器(Apcu、Memcached和Redis)或数据库适配器(Doctrine和PDO)。

### cache.adapter.memcached

Memcached适配器在Symfony3.3中引入。

该适配器允许使用多个Memcached服务实例，不限于当前服务器共享内存，可独立于php环境存储，还可以利用服务器集群提供冗余和/或故障转移。

Memcached服务使用php环境Memcached扩展必须安装并运行，这个适配器需要Memcached PHP扩展的版本2.2或更高版本。

可使用参数：Memcached实例、namespace、defaultLifetime

```
use Symfony\Component\Cache\Adapter\MemcachedAdapter;

$cache = new MemcachedAdapter(
    // 设置选项并添加服务器实例的客户端对象
    \Memcached $client,

    // a string prefixed to the keys of the items stored in this cache
    $namespace = '',

    // the default lifetime (in seconds) for cache items that do not define their
    // own lifetime, with a value 0 causing items to be stored indefinitely (i.e.
    // until MemcachedAdapter::clear() is invoked or the server(s) are restarted)
    $defaultLifetime = 0
);
```
 
**配置和连接**
 
createConnection()方法使用数据源名称(DSN)或DSN数组创建Memcached类实例配置。

```
use Symfony\Component\Cache\Adapter\MemcachedAdapter;

// pass a single DSN string to register a single server with the client
$client = MemcachedAdapter::createConnection(
    'memcached://localhost'
    // the DSN can include config options (pass them as a query string):
    // 'memcached://localhost:11222?retry_timeout=10'
    // 'memcached://localhost:11222?socket_recv_size=1&socket_send_size=2'
);

// pass an array of DSN strings to register multiple servers with the client
$client = MemcachedAdapter::createConnection([
    'memcached://10.0.0.100',
    'memcached://10.0.0.101',
    'memcached://10.0.0.102',
    // etc...
]);
```

在memcached DSN中传递配置选项的功能是在Symfony 3.4中引入的。

DSN格式：

```
    memcached://[user:pass@][ip|host|socket[:port]][?weight=int]
```

DSN必须包含IP/HOST、可选参数port或套接字路径、用户名和密码、weight

密码为SASL验证；要求memcached扩展在php环境下用--enable-memcached-sasl编译

weight为在集群中服务的优先级；在0到100之间默认为null；值越高优先级越高。

以下为常用DSN示例，显示可用值的组合：

```
use Symfony\Component\Cache\Adapter\MemcachedAdapter;

$client = MemcachedAdapter::createConnection([
    // hostname + port
    'memcached://my.server.com:11211'

    // hostname without port + SASL username and password
    'memcached://rmf:abcdef@localhost'

    // IP address instead of hostname + weight
    'memcached://127.0.0.1?weight=50'

    // socket instead of hostname/IP + SASL username and password
    'memcached://janesmith:mypassword@/var/run/memcached.sock'

    // socket instead of hostname/IP + weight
    'memcached:///var/run/memcached.sock?weight=20'
]);
```

**配置参数**

createConnection()函数第二个参数接收配置项数组。

预期的格式是key => value 的关联数组，表示选项名及其各自的值:

```
use Symfony\Component\Cache\Adapter\MemcachedAdapter;

$client = MemcachedAdapter::createConnection(
    // a DSN string or an array of DSN strings
    [],

    // associative array of configuration options
    [
        'compression' => true,
        'libketama_compatible' => true,
        'serializer' => 'igbinary',
    ]
);
```

**可用配置**

+ auto_eject_hosts
    
    类型：bool 默认：false
    
    通过自动弹出超过配置的server_failure_limit的主机，启用或禁用集群的常量、自动重新平衡。

+ buffer_writes

    类型：bool 默认：false
    
    
+ compression
+ compression_type
+ connect_timeout
+ distribution
+ hash
+ libketama_compatible
+ no_block
+ number_of_replicas
+ prefix_key
+ poll_timeout
+ randomize_replica_read
+ recv_timeout
+ retry_timeout
+ send_timeout
+ serializer
+ server_failure_limit
+ socket_recv_size
+ socket_send_size
+ tcp_keepalive
+ tcp_nodelay
+ use_udp
+ verify_key



### cache.adapter.apcu




有些是配置可通过快捷方式视配置。使用这些快捷方式可用cache.[type]格式的服务id创建pools。

*app/config/config.yml*

```
framework:
    cache:
        directory: '%kernel.cache_dir%/pools' # Only used with cache.adapter.filesystem

        # service: cache.doctrine
        default_doctrine_provider: 'app.doctrine_cache'
        # service: cache.psr6
        default_psr6_provider: 'app.my_psr6_service'
        # service: cache.redis
        default_redis_provider: 'redis://localhost'
        # service: cache.memcached
        default_memcached_provider: 'memcached://localhost'
        # service: cache.pdo
        default_pdo_provider: 'doctrine.dbal.default_connection'
```

*app/config/config.php*

```
 $container->loadFromExtension('framework', [
     'cache' => [
         // Only used with cache.adapter.filesystem
         'directory' => '%kernel.cache_dir%/pools',
         // Service: cache.doctrine
         'default_doctrine_provider' => 'app.doctrine_cache',
         // Service: cache.psr6
         'default_psr6_provider' => 'app.my_psr6_service',
         // Service: cache.redis
         'default_redis_provider' => 'redis://localhost',
         // Service: cache.memcached
         'default_memcached_provider' => 'memcached://localhost',
         // Service: cache.pdo
         'default_pdo_provider' => 'doctrine.dbal.default_connection',
     ],
 ]);
```

## 创建自动以pools

根据适配器创建服务的pools:

*app/config/config.yml*

```
framework:
    cache:
        default_memcached_provider: 'memcached://localhost'
        pools:
            my_cache_pool:
                adapter: cache.adapter.filesystem
            cache.acme:
                adapter: cache.adapter.memcached
            cache.foobar:
                adapter: cache.adapter.memcached
                provider: 'memcached://user:password@example.com'
```