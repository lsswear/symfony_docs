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