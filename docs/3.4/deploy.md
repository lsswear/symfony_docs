## Symfony开发基础

部署步骤：
+ 1、将代码上传到服务器
+ 2、添加依赖扩展，composer更新
+ 3、运行数据库迁移任务
+ 4、清除缓存

开发步骤：
+ 1、代码下载
+ 2、测试运行
+ 3、清除不必要文件
+ 4、清除缓存

## 开发应用

### 基本文件传输

可以通过ftp进行文件传输，但随着文件的改变，不容易对修改版本进行控制，可以在修改后进行其他操作。

### 使用版本控制

若使用Git或SVN,服务器涉及git和svn搭建，git和svn均有可视化工具。
git:https://git-scm.com/
svn:https://www.visualsvn.com/

### 使用构建脚本和其他工具

+ EasyDeployBundle
    向应用程序添加部署工具的symfony的包
+ Deployer
    这是Capistrano的另一个本地PHP重写，有一些Symfony的现成系统。
+ Ansistrano
    Ansible角色，可以通过YAML文件配置一个强大的部署配置。
+ Magallanes
    这个类似capistrano的部署工具是在PHP中构建的，PHP开发人员可以更容易地扩展它以满足他们的需要。
+ Fabric
    这个基于python的库提供了一套基本的操作，用于执行本地或远程shell命令和上传/下载文件。
+ Capistrano with Symfony插件
    Capistrano是一个用Ruby编写的远程服务器自动化和部署工具。Symfony插件是一个插件，以方便Symfony相关的任务，启发Capifony(它只与Capistrano 2)。这个基于python的库提供了一套基本的操作，用于执行本地或远程shell命令和上传/下载文件。
+ sf2debpkg：
    帮助您为Symfony项目构建一个本地的Debian包。
+ 基本脚本
    您可以使用shell脚本、Ant或任何其他构建工具为项目的部署编写脚本。
    
### 常见的部署后任务

+ A) 检查要求
    安装symfony时创建symfony二进制文件提供命令检查服务器环境是否满足symfony要求。
+ B) 配置换将 Configure your Environment Variables
    
    大多数symfony应用定义配置参数在app/config/parameter.yml，通过app/config/parameters.yml.dist自动生成，可以不用部署这个文件，parameters.yml.dist文件必须部署。
    
    如果应用程序使用环境变量而不是这些参数，则必须使用托管服务提供的工具在生产服务器中定义这些env变量。
    
    需要定义SYMFONY_ENV=prod，若使用Symfony Flex定义APP_ENV=prod，使应用在prod模式下运行时，但根据应用可能需要定义其他env变量。

+ C) 安装、更新供应商
    
    供应商可以在传输代码之前修改，例如 vendor/文件夹，然后将其与源代码一起传输，或者在之后在服务器上处理。
    
    命令： composer install --no-dev --optimize-autoloader
    
    —optimize-autoloader标志通过构建“类映射”来显著提高编写器的autoloader性能。-no-dev标志确保没有在生产环境中安装开发包
    
    如果在此步骤中出现“class not found”错误，则可能需要在运行此命令之前运行export SYMFONY_ENV=prod，如果使用Symfony Flex，则需要设置APP_ENV=prod，以便安装后的cmd脚本在prod环境中运行。
    
+ D) 清除Symfony缓存
    命令：php bin/console cache:clear --env=prod --no-debug
+ E) 其他处理
    - 运行数据库迁移
    - 清理APC缓存
    - 添加或修改CRON任务
    - 用Webpack Encore 构建或缩减assets资源
    - 将assets传到CDN

## 故障排除

**不使用composer.json文件**

symfony提供kernel.project_dir参数和相关getProjectDir()方法。

可以使用此方法对相对于项目根目录的文件执行操作操作。根据本地composer.json查找项目根目录。

若未使用Composer，需移除composer.json，应用不工作在生产服务器上。

这种情况应在应用核心文件重写getProjectDir()方法，并返回项目根目录。

*app/AppKernel.php*

```
class AppKernel extends Kernel
{
    public function getProjectDir()
    {
        return dirname(__DIR__);
    }
}
```

 