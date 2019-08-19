### 安装

## linux & mac

$ sudo curl -LsS https://symfony.com/installer -o /usr/local/bin/symfony

$ sudo chmod a+x /usr/local/bin/symfony

## windows 

composer create-project symfony/framework-standard-edition project_name project_version

例： composer create-project symfony/framework-standard-edition blog "3.4"

### 目录结构

- app 主要编写代码文件
    - config 配置文件
    	- config.yml 配置
		- config_dev.yml 开发配置
		- config_prod.yml 正式配置
		- config_test.yml 测试配置
		- parameters.yml 数据库配置
		- routing.yml 路径配置
		- routing_dev.yml 开发用路径配置
		- security.yml 安全配置
		- services.yml 服务配置
    - Resourses 资源文件夹
    	- views 页面文件夹 
    		- admin 模块名
    		    - index.html.twig 页面
    		- blog
    		- data_collector
    		    - test.html.twig
    		- default 默认模块
    		- layout 布局文件夹
    		    - base.html.twig 布局文件
    		- user
    - AppCache.php app缓存
    - AppKernel.php app核心包初始化
    - autoload.php 自动加载composer包
- bin
    - console 命令号执行文件
    - symfony_requirements 自动检查需求工具
- src
	- AppBundle
	    - Controller 控制器类
	        - DefaultController.php
	    - DataController 
	        - TestController.php
	    - DependencyInjection 依赖注入
	    - Entity 数据表实体
	        - TableName.php 
	    - Repository 实体类扩展查询部分
	    - Resources 
	        - config
	    - Service 服务
	        - CommonService.php
	    - Tests 测试
	        - Controller
	            - IndexController.php
	    - Twig
	        - Paging.php
	    - AppBundle.php
- tests 测试
	- AppBundle
	- PermissionBundle
- var
	- cache 缓存文件
	- logs 日志文件
	- sessions session文件
	- SymfonyRequirements.php 需求校检查脚本
- web 前端入口文件夹
	- admin 
	- blog
	- bundles
	- app.php 正式入口文件
	- app_dev.php 开发入口文件
	- apple-touch-icon.png
	- config.php 加载var/SymfonyRequirements.php并显示信息
	- favicon.ico
	- robots.txt
    - .htaccess 跳转文件
 - phpunit.xml.dist phpunit 配置文件
