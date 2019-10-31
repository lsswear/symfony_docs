## bundle

从4.3发布信息看，并没有基本功能的改变，但3.4文档中提到过4.3中bundle应该像“插件”一样。

对于系统的插件，应该为代码逻辑、页面资源、数据库实体都是独立的，可以安装和删除。

3.4也解释过，bundle安装和删除命令都会执行对应必须设置或创建的内容。

### 文件目录结构修改

+ bin
+ config
+ public
+ src
    - Controller
        - .gitignore
    - Kernel.php

既然bundle为插件，系统就应该有不为插件的内容，controller存放对应文件。

### 英文翻译

在4.0版本中，建议使用bundle组织自己的应用代码。不再建议bundle仅使用于多个应用之间分享代码和特性。

symfony代码特性用bundle实现，例如FrameworkBundle, SecurityBundle, DebugBundle。

bundle也使用于用第三方bundle在应用中添加特性。

第三方bundle列表：[github.com](https://github.com/search?q=topic%3Asymfony-bundle&type=Repositories)

bundle在应用中使用必须在每个环境的config/bundle。

*config/bundles.php*

```
return [
    // 'all' means that the bundle is enabled for any Symfony environment
    Symfony\Bundle\FrameworkBundle\FrameworkBundle::class => ['all' => true],
    Symfony\Bundle\SecurityBundle\SecurityBundle::class => ['all' => true],
    Symfony\Bundle\TwigBundle\TwigBundle::class => ['all' => true],
    Symfony\Bundle\MonologBundle\MonologBundle::class => ['all' => true],
    Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle::class => ['all' => true],
    Doctrine\Bundle\DoctrineBundle\DoctrineBundle::class => ['all' => true],
    Sensio\Bundle\FrameworkExtraBundle\SensioFrameworkExtraBundle::class => ['all' => true],
    // this bundle is enabled only in 'dev'  and 'test', so you can't use it in 'prod'
    Symfony\Bundle\WebProfilerBundle\WebProfilerBundle::class => ['dev' => true, 'test' => true],
];
```

默认symfony应用使用symfony flex，bundle安装或卸载时会自动设置可用或不可用，不必编辑bundles.php文件。

### 创建Bundle

例如创建Acme Bundle,创建src/Acme/TestNundle/ 文件夹并添加文件AcmeTestBundle.php:

*src/Acme/TestBundle/AcmeTestBundle.php*

```
namespace App\Acme\TestBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;

class AcmeTestBundle extends Bundle
{
}
```

AcmeTestBundle名字根据Bundle命名规则。您还可以通过命名这个类TestBundle为，以及命名文件TestBundle.php，来选择将包的名称缩短为简单的TestBundle。

*config/bundles.php*

```
return [
    // ...
    App\Acme\TestBundle\AcmeTestBundle::class => ['all' => true],
];
```

### Bundle 文件夹结构

bundle的文件夹结构帮助保持不同bundle之间的代码一致。它遵循一套惯例，但在需要时可以灵活地进行调整。

+ Controller/
+ DependencyInjection/
+ Resources/config/
+ Resources/views/
+ Resources/public/
+ Tests/

一个bundle或大或小根据其特性的实现，仅包含需要的文件。

在浏览指南时，您将了解如何将对象持久化到数据库、创建和验证表单、为应用程序创建翻译、编写测试等等，它们在包中都是独立的。

## 其他

指南的内容基本上都是描述第三插件的使用，故不再翻译。



