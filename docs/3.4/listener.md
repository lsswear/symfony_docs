https://symfony.com/doc/current/bundles/FOSRestBundle/3-listener-support.html#priorities
侦听器是连接（hook）到请求处理的一种方法。

+ [body listener](#Body listener)：解码请求内容
+ [format listener](#Format listener)：确定正确的响应格式
+ [parameter fetcher listener](#Param fetcher listener)：获取请求参数
+ [view response listener](#View Response listener)：格式化响应内容，twig模板引擎或者xml、json序列化
+ [accept listener](#)：在响应中自动设置接受的HTTP方法


### Body listener

请求体监听解码请求内容，填充请求参数。

例如接收application/x-www-form-urlencode形式post请求的，应为put请求中的格式的数据，比如application/json。

### Decoders

可以添加解码格式，也可以覆盖默认解码服务，通过第三方包为json、xml设置格式。

xml解码器显式地保留为其默认服务。

*app/config/config.yml*

```
fos_rest:
    body_listener:
        decoders:
            json: acme.decoder.json
            xml: fos_rest.decoder.xml
```

自定义解码服务必须是引入的**FOS\RestBundle\Decoder\DecoderInterface**类。

使用带有复选框的form并具有true和false值，必须使用:fos_rest.decoder.jsontoform，从fosrest 0.8.0开始可用。

监听接收内容，解释失败时抛出BadRequestHttpException异常，包括信息：'Invalid ' .$format . ' message received'。

### Array Normalizer

数组规格化器允许在数据解码后对其进行转换，以便于对其进行处理。



## 优先级

