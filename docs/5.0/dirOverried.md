## 默认目录覆盖

https://symfony.com/doc/current/configuration/override_dir_structure.html#override-web-dir

### 目录结构

```
project
|-assets/
|-bin/
	|-console
|- config/
|- public/
	|-index.php
|- src/
	|- ...
|- templates/
|- tests/
|- translations/
|- var/
	|-cache/
	|- log/
	|- ...
|- vendor/
```

### config目录覆盖

配置目录是唯一不可覆盖

### cache目录覆盖

可以改变默认cache目录通过重写应用Kernel类中getCacheDir()方法：

*src/Kernel.php*

```
class Kernel extends BaseKernel
{
	public function getCacheDir()
	{
		return dirname(__DIR__).'/var/'.$this->enviroment.'/cache';
	}
}
```
$this->enviroment是当前环境。上述情况将本地cache目录改为var/{enviroment}/cache/。

应每个应用环境保持不同的cache目录，否则可能发生位置行为。每个环境生成自己的缓存配置文件，并需要分别的文件存储缓存文件。

### log目录覆盖

覆盖var/log/目录和覆盖var/cache/目录相同。唯一的区别是重写getLogDir()方法：

*src/Kernel.php*

```
class Kernel extends Kernel
{
	public function getLogDir()
	{
		return dirname(_DIR_).'/var/'.$this->environment.'/log';
	}
}
```

以上将本地目录改为/var/{enviroment}/log/。

### tempaltes目录覆盖

如果模板不储存在默认template/目录，使用twig.paths配置项去定义自己的模板目录。

*config/packages/twig.yaml*

```
	twig:
		paths:["%kernel.project_dir%/resources/views"]
```

*config/packages/twig.php*

```
$container->loadFromExtension('twig',[
	'paths'=>[
		'%kernel.project_dir%/resources/views',
	]
]);

```

### translations目录覆盖

如果多语言文件未存储在默认translations/目录，使用framework.translator.paths配置定义自己的translations目录。

*config/packages/translation.php*

```
$container->loadFromExtension('framework',[
	'translator'=>[
		'paths'=>[
			'%kernel.project_dir%/i18n',
		],
	]
]);	
```
### public文件目录覆盖

如果需要重命名或移动public/目录，仅需保证在index.php中的var/目录路径正确。重命名对影响不大，但是移动的话需改写文件。

```
require_once __DIR__.'/../path/to/vendor/autoload.php';

#上述内容在config/bootstrap.php文件中存在，bootstrap.php在index.php文件中引入，可能这句并不用改。
```



*composer.json*

```
{
	"extra":{
		"public-dir":"rename_public_dir"
	}
}
```

一些共享主机有public_html/web根目录。一种方法是重命名public/为public_html/,另一种方法是项目部署到web根目录之外，删除自己的public_html/目录，并软连接到项目中的public/文件夹。

### vendor目录覆盖

覆盖vendor/目录，需要在composer.json文件中定义vendor-dir选项。

```
"config":{
	"bin-dir":"bin",
	"vendor-dir":"/some/dir/vendor"
}
```