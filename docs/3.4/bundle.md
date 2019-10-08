*3.4版本之前推荐bundle组织项目，3.4版本之后不再推荐，bundle最为多个应用之间代码分享。*

bundle和其他系统的插件类似，但包括核心框架功能和为应用程序编写的代码都是独立的，更有利于打包和包的分发。

## 注册

*app/AppKernel.php*

```
public function registerBundles()
{
    $bundles = [
        new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
        new Symfony\Bundle\SecurityBundle\SecurityBundle(),
        new Symfony\Bundle\TwigBundle\TwigBundle(),
        new Symfony\Bundle\MonologBundle\MonologBundle(),
        new Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle(),
        new Symfony\Bundle\DoctrineBundle\DoctrineBundle(),
        new Sensio\Bundle\FrameworkExtraBundle\SensioFrameworkExtraBundle(),
        new AppBundle\AppBundle(),
    ];

    if (in_array($this->getEnvironment(), ['dev', 'test'])) {
        $bundles[] = new Symfony\Bundle\WebProfilerBundle\WebProfilerBundle();
        $bundles[] = new Sensio\Bundle\DistributionBundle\SensioDistributionBundle();
        $bundles[] = new Sensio\Bundle\GeneratorBundle\SensioGeneratorBundle();
    }

    return $bundles;
}
```

 registerBundles() 可控制项目使用哪些包，若项目使用app/autoload.php加载bundle，则对应的bundle可以在任何地方使用。
 
## 创建
 
### 具体操作
 
 创建AcmeTestBundle：
 
 *src/Acme/TestBundle/AcmeTestBundle.php*
 
 
 ```
 namespace Acme\TestBundle;
 
 use Symfony\Component\HttpKernel\Bundle\Bundle;
 
 class AcmeTestBundle extends Bundle
 {
 }
 ```
 
 注册：
 
 *app/AppKernel.php*
 
```
public function registerBundles()
{
    $bundles = [
        new Acme\TestBundle\AcmeTestBundle(),
    ];
    return $bundles;
}
```

### 命令行操作

```
php bin/console generate:bundle --namespace=Acme/TestBundle
```

## bundle 文件结构

+ Controller 控制器
+ DependencyInjection 依赖注入，非必要文件夹，用于引入服务配置等
+ Resources
    - config 包括路径配置，routing.yml
    - views 根据控制器名字存放视图页面，比如Random/index.html.twig
    - public 包含外部资源，assets:install可链接到web目录
+ Test 包含测试的Bundle

bundle应尽量小仅包含其自身实现的功能。




              
     

