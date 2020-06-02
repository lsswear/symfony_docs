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

无论是否本地开发，运行symfony最简单的方法是通过使用本地网络服务提供的symfony二进制命令。本地服务提供