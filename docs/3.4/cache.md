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