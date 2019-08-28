## 特殊字符

<b>@<b/>字符告诉容器传递id为某个字符串的服务。


## 设置全局变量

### 文件中设置

*app/config/parameters.yml*

```php
parameters:
    router.request_context.host: 'example.org'
    router.request_context.scheme: 'https'
    router.request_context.base_url: 'my/path'
    asset.request_context.base_path: '%router.request_context.base_url%'
    asset.request_context.secure: true
```

*app/config/parameters.php*

```php
$container->setParameter('router.request_context.host', 'example.org');
$container->setParameter('router.request_context.scheme', 'https');
$container->setParameter('router.request_context.base_url', 'my/path');
$container->setParameter('asset.request_context.base_path', $container->getParameter('router.request_context.base_url'));
$container->setParameter('asset.request_context.secure', true);
```

asset.request_context.* 参数在 Symfony 3.4 引入。

### 控制器中设置

```
class DemoCommand extends ContainerAwareCommand
{
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $router = $this->getContainer()->get('router');
        $context = $router->getContext();
        $context->setHost('example.com');
        $context->setScheme('https');
        $context->setBaseUrl('my/path');

        $url = $router->generate('route-name', ['param-name' => 'param-value']);
        // ...
    }
}
```

### 特殊字符处理

以@或%字符开头的值需要在字符串开始位置加@或者%。

*app/config/parameters.yml*

```php
parameters:
    # This will be parsed as string '@securepass'
    mailer_password: '@@securepass'

    # Parsed as http://symfony.com/?foo=%s&amp;bar=%d
    url_pattern: 'http://symfony.com/?foo=%%s&amp;bar=%%d'
```

### 数组

*app/config/parameters.yml*

```
parameters:
    my_mailer.gateways: [mail1, mail2, mail3]

    my_multilang.language_fallback:
        en:
            - en
            - fr
        fr:
            - fr
            - en
```

*app/config/parameters.php*

```php
$container->setParameter('my_mailer.gateways', ['mail1', 'mail2', 'mail3']);
$container->setParameter('my_multilang.language_fallback', [
    'en' => ['en', 'fr'],
    'fr' => ['fr', 'en'],
]);
```

### php常量

*app/config/parameters.yml*

```php
parameters:
    global.constant.value: !php/const GLOBAL_CONSTANT
    my_class.constant.value: !php/const My_Class::CONSTANT_NAME
```

*app/config/parameters.php*

```
    $container->setParameter('global.constant.value', GLOBAL_CONSTANT);
    $container->setParameter('my_class.constant.value', My_Class::CONSTANT_NAME);
```

## 自动配置
System3.3版本引入。


## 环境设置

环境配置文件：

开发 dev : app/config/config_dev.yml

生产 prod: app/config/config_prod.yml

测试 test: app/config/config_test.yml

环配置文件加载：

*app/AppKernel.php*

```php
class AppKernel extends Kernel
{
    // ...

    public function registerContainerConfiguration(LoaderInterface $loader)
    {
        $loader->load($this->getRootDir().'/config/config_'.$this->getEnvironment().'.yml');
    }
}
```

环境类型定义

*web\app.php*

```php
    //非debug模式
    $kernel = new AppKernel('prod', false);
```

*vendor/symfony/symfony/src/Symfony/Component/HttpKernel\Kernel.php*

```php
abstract class Kernel implements KernelInterface, TerminableInterface
{
    public function __construct($environment, $debug)
    {
        $this->environment = $environment;
        $this->debug = (bool) $debug;
        $this->rootDir = $this->getRootDir();
        $this->name = $this->getName();

        if ($this->debug) {
            $this->startTime = microtime(true);
        }
    }
    public function getEnvironment()
    {
        return $this->environment;
    }
}
```

## 配置文件使用

加载其他配置文件

*yaml*

```php
imports:
    - { resource: config.yml }
    - { resource: "@ManagementBundle/Resources/config/services.yml" }
```

*php*

```php
$loader->import('config.php');
$loader->import('@ManagementBundle/Resources/config/services.yml');
```

根据引入的文件，设置值可以被覆盖。

*app/config/config_dev.yml*

```php
    web_profiler:
        toolbar: true    
```

*app/config/config_dev.php*

```php
$container->loadFromExtension('web_profiler', array(
    'toolbar' => true,
    // ...
));
```




