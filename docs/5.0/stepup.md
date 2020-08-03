## symfony安装

https://symfony.com/doc/current/setup.html

symonfy工具检测安装环境

```
symfony check:requirements
```

应用创建

```
# web应用创建
symfony new project_name --full

# 微服务、控制台、api应用创建
symfony new  project_name
```

## composer安装

应用创建

```
# web应用创建
composer create-project symfony/website-skeleton my_project_name

# 微服务、控制台、api应用创建
composer create-project symfony/skeleton my_project_name
```

## 应用运行

在生产环境中，应该安装如nginx或apache的网络服务器，并配置。这总方法也适用于symfony本地服务或开发环境。

无论是否本地开发，运行symfony最简单的方法是通过使用本地网络服务提供的symfony二进制命令。本地服务提供对HTTP/2、并发请求、TLS/SSL和自动生成安全证书的支持。

打开控制台终端，移动到新项目目录，并启动本地web服务：

```
	cd my-project/
	symfony server:start
```
打开浏览器或导航到 http://localhost:8000。如果顺利运行，讲辉看到欢迎页面。之后，当要结束工作，停止运行服务，可通过在终端运行Ctrl+c。

php应用的网络服务，不只是Symfony项目，所以它是通用开发工具。

## 设置现有Symfony项目

添加Symfony新项目时，可能是其他开发者创见的项目。这种情况下，需要获取项目源码并通过Composer安装依赖。假设团队使用git，设置醒目通过以下命令：

```
cd projects/
git clone ***

cd my-project/
composer install
```

可能还需要定制.env文件并执行项目具体任务(例如数据库创建)。第一次执行Symfony应用，运行这个显示项目信息的命令可能有用：

```
> php bin/console about
```
##安装包

一般情况当Symfony应用处于开发环境安装包（symfony种称为bundles）时提供随时可用的特性。包通常需要使用他们之前需要一些设置（编辑一些文件使bundle可使用，创建一些文件添加一些初始配置）。

大多数情况，设置可以自动化，这是Symfony引入Symfony Flex的原因，这个工具可以在Symfony项目中简单安装或删除包。从技术上说，Symfony Flex时一个Composer插件，在symfony应用创建时默认安装，可自动完成常见任务。

已存在的项目也可添加Symfony Flex。

Symfony Flex修改require、update、remove和Composer命令的行为，以提供先进的特征。
考虑以下示例：

```
> cd my-project/
> composer require logger
```
若在未安装Flex的Symfony应用中执行命令，将会看见Composer错误显示“logger is not a valid package name”。如果应用已经安装Symfony Flex,命令可以安
装并在所有包中