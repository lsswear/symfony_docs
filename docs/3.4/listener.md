https://symfony.com/doc/current/bundles/FOSRestBundle/3-listener-support.html#priorities
侦听器是连接（hook）到请求处理的一种方法。

+ [body listener](#Body listener)：解码请求内容
+ [format listener](#Format listener)：确定正确的响应格式
+ [parameter fetcher listener](#Param fetcher listener)：获取请求参数
+ [view response listener](#View Response listener)：格式化响应内容，twig模板引擎或者xml、json序列化
+ [accept listener](#)：在响应中自动设置接受的HTTP方法

## 优先级

