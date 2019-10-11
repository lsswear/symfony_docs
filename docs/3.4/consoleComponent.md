## console 组件

控制台组件简化了可测试的命令行接口的创建。

console组件允许创建命令行命令。创建的命令可使用在重复任务中，比如计划、引入或其他批处理操作。

### 安装

composer require symfony/console:^3.4

若在symfony应用外安装，必须自动加载composer提供的vendor/autoload.php文件。

## 创建console应用

php脚本定义console应用：

*application.php*

```
#!/usr/bin/env php
<?php
require __DIR__.'/vendor/autoload.php';

use Symfony\Component\Console\Application;

$application = new Application();

// ... register commands

$application->run();
```

用add()方法注册：

```
$application->add(new GenerateAdminCommand());
```

