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

+ cache.adapter.apcu
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



