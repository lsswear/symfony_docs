## 请求异常处理

控制器继承的Controller类设置createNotFoundException函数调用NotFoundHttpException抛出404异常。

NotFoundHttpException继承HttpException类。HttpException实现HttpExceptionInterface规范接口定义必要函数，继承RuntimeException而RuntimeException仅仅是继承了Exception。

也就是说HttpException仅用接口定义了规范和继承了Exception，接口定义的函数包括getStatusCode获取状态编码、getHeaders获取请求头部。

## 自定义异常

## 异常页面修改

异常页面加载时使用内部ExceptionController通过Twig模板显示页面,该控制器使用HTTP状态码、请求格式和以下逻辑定义模板文件名：

+ 1、根据给定格式和状态码查找模板。例如error404.json.twig 或 error500.html.twig。

+ 2、如果之前的模板不存在，则丢弃状态代码，并为给定的格式寻找通用模板，如:error.json.twig 或 error.xml.twig.

+ 3、如果前面的模板都不存在，则返回到通用HTML模板，error.html.twig。

页面存放位置：app/Resources/TwigBundle/views/Exception/页面

页面样例：

*app/Resources/TwigBundle/views/Exception/error404.html.twig*

```php
{% extends 'base.html.twig' %}

{% block body %}
    <h1>Page not found</h1>

    <p>
        The requested page couldn't be located. Checkout for any URL
        misspelling or <a href="{{ path('homepage') }}">return to the homepage</a>.
    </p>
{% endblock %}
```

ExceptionController中status_code存储HTTP状态码，status_text存储错误信息。

实现HttpExceptionInterface接口可以自定义状态码，status_code默认500。

**在开发过程中测试错误页面**

*app/config/routing_dev.yml*

```
_errors:
    resource: "@TwigBundle/Resources/config/routing/errors.xml"
    prefix:   /_error
```

*app/config/routing_dev.php*

```
use Symfony\Component\Routing\RouteCollection;

$routes = new RouteCollection();
$routes->addCollection(
    $loader->import('@TwigBundle/Resources/config/routing/errors.xml')
);
$routes->addPrefix("/_error");

return $routes;
```

测试路径

```
http://localhost/app_dev.php/_error/{statusCode}
http://localhost/app_dev.php/_error/{statusCode}.{format}
```

## 异常控制器修改

*app/config/config.yml*

```
twig:
    exception_controller: AppBundle:Exception:showAction
```

*app/config/config.php*

```
$container->loadFromExtension('twig', [
    'exception_controller' => 'AppBundle:Exception:showAction',
]);
```

ExceptionListener类作为TwigBundle的监听器，通过kernel.exception事件向控制器传递参数。

传递参数如下：

**exception**

从异常处理创建的FlattenException实例。

**logger**

DebugLoggerInterface实例，某些情况下可为空。

扩展异常控制器，覆盖showAction()和findTemplate(),后者定义要使用的模板。

*app/config/services.yml*

```
services:
    _defaults:
        # ... be sure autowiring is enabled
        autowire: true
    # ...

    AppBundle\Controller\CustomExceptionController:
        public: true
        arguments:
            $debug: '%kernel.debug%'
```

*app/config/services.php*

```
use AppBundle\Controller\CustomExceptionController;

$container->autowire(CustomExceptionController::class)
    ->setArgument('$debug', '%kernel.debug%');
```
    
一般控制器中也可以使用。
    
## 使用kernel.exception事件

当异常抛出时，HttpKernel捕获并传递一个kernel.exception事件，使异常信息通过不同方法转换成Response。

为kernel.exception事件自定义监听，可以仔细的检查异常并对异常进行处理，可以记录异常日志或者重定向到其他页面或者异常页面。

若监听在GetResponseForExceptionEvent上调用setResponse()，时间和传播将停止，并且响应代码会传给客户端。

通过这些方法可以创建集中的和分层的错误处理：您可以让一个(或多个)侦听器来处理它们，而不是一次又一次地在不同的控制器中捕获(并处理)相同的异常。

有关此类高级侦听器的实际示例，请参见ExceptionListener类代码。这个侦听器处理应用程序中抛出的各种与安全相关的异常(如AccessDeniedException)，并采取一些措施，如将用户重定向到登录页面、将其注销等。

symfony2\vendor\symfony\symfony\src\Symfony\Component\HttpKernel\EventListener\ExceptionListener.php

symfony2\vendor\symfony\symfony\src\Symfony\Component\HttpKernel\Tests\EventListener\ExceptionListenerTest.php

