## 使用Symfony Flex 去管理Symfony应用

Symfony Flex 是新方式安装symfony应用。

Flex 不是一个symfony版本，而代替和提高了Symfony Installer 和Symfony Standard Edition的工具。

Symfony Flex 自动执行大多数symfony应用命令任务，比如安装和卸载bundles和其他Composer依赖。

Symfony Flex 为symfony3.3或更高版本工作。

从symfony4.0开始，Flex应该默认使用，但仍然是可选的。

## Flex 工作原理

Symfony Flex 是Composer 插件，它修改了修改了require、update、remove命令的行为。

当再Flex可使用的应用中安装或移除依赖，Symfony在执行任务之前和之后执行Composer任务。

```
> cd my-project/
> composer require mailer
```

若在不可用Flex的系统中执行命令，可以看Composer失败新系统提示“mailer”非有效包名。

若用Symfony Flex安装，该命令最终安装并启用了SwiftmailerBundle，这是集成Swiftmailer（Symfony应用程序的官方邮件程序）最好的方式。

当在Symfony Flex已安装的应用中执行composer require，在尝试使用Composer安装包之前，应用程序向Symfony Flex服务器发出请求。

+ 若没有这个包信息，Flex不返回信息，包的安装过程根据composer。
+ 若有关于包的特殊的信息，flex在“recipe”文件返回，应用根据信息决定安装哪个包以及安装后运行哪些自动任务。

在之前的例子中，Symonfy Flex 请求mailer包，并且Symonfy Flex服务检测到mailer实际上是SwiftmailerBundle的别名并返回它的 "recipe"。

Flex在symfony.lock中保持安装进度的跟踪，该文件必须提交到代码库中。

## Symfony Flex 配置

Flex配置定义在mainfest.json文件，可包含任何数量的其他文件和目录。

例如manifest.json为的SwiftmailerBundle配置:

```
{
    "bundles": {
        "Symfony\\Bundle\\SwiftmailerBundle\\SwiftmailerBundle": ["all"]
    },
    "copy-from-recipe": {
        "config/": "%CONFIG_DIR%/"
    },
    "env": {
        "MAILER_URL": "smtp://localhost:25?encryption=&auth_mode="
    },
    "aliases": ["mailer", "mail"]
}
```

aliases选项允许安装包时使用容易记住的名称,composer require mailer 和 composer require symfony/swiftmailer-bundle 相比。

bundles选项告诉Flex这个配置项对应的bundle在哪些环境中自动启动，第二个参数all标识全部环境。

env选项使Flex向应用中添加新环境变量。

copy-from-recipe选项允许配置复制文件和文件夹向应用。

结构定义在mainfest.json文件中，并且在Symfony Flex卸载依赖，也会使用改文件撤销所有更改，比如composer remove mailer。

这意味flex从应用中移除SwiftmailerBundle，删除通过配置创建的MAILER_URL环境变量和其他文件和文件夹。

Symfony Flex 配置由社区提供，并存储在两个公有存储库中：

+ 主要库：高质量和维护包的配置列表。默认symfony flex 只从这个存储库读取。

+ 普通库： 包含社区创建的所有配置(包)。所有的配置保证可以工作，但是关联的包无人维护。Symfony Flex 在安装这些包是会询问是否要安装。

