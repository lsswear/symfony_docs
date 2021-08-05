# symfony 配置包创建

https://github.com/symfony/recipes/blob/master/README.rst

Symfony配方允许通过Symfony Flex Composer 插件自动化Composer包配置。

这个存储库包含由Symfony核心团队认可的Composer包的“官方”包。

## 创建包

Symfony 包由manifest.json配置文件和可选参数和任何数量的文件和文件夹。包必须存储在他们自己的存储库中，在Composer包存储库之外。他们必须遵循的目录结构为：vendor/package/version，version是包里支持的最小版本。

```
例：
symfony/
	console/
		3.3/
			bin/
			manifest.json
	framework-bundle/
		3.3/
			config/
			public/
			src/
			manifest.json
	requirements-checker/
		1.0/
			manifest.json
```

所有的