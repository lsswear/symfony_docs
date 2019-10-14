## console命令

symfony框架通过bin/console脚本提供一些命令。这些命令由 Console component创建。

### 创建console命令

命令定义在用户自定义包中的Command类中，类名后缀名为Command。

*src/AppBundle/Command/CreateUserCommand.php*

```
namespace AppBundle\Command;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

class CreateUserCommand extends Command
{
    // bin/comsole 脚本中命令的名字 
    protected static $defaultName = 'app:create-user';

    protected function configure()
    {
        // ...
    }

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        // ...
    }
}
```

### 命令配置

必须在configure()方法中配置

### 命令参数

命令参数为字符串，以空格分割，在命令之后设置。他们是有序的、可选的、也是必须的。


```
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputArgument;

class GreetCommand extends Command
{
    // ...

    protected function configure()
    {
        //接收参数
        $this
            // ...
            ->addArgument('name', InputArgument::REQUIRED, 'Who do you want to greet?')
            ->addArgument('last_name', InputArgument::OPTIONAL, 'Your last name?')
    }
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        //参数使用
        $text = 'Hi '.$input->getArgument('name');
        $lastName = $input->getArgument('last_name');
        if ($lastName) {
            $text .= ' '.$lastName;
        }
        $output->writeln($text.'!');
    }
}

```

命令使用：

```
php bin/console app:greet Fabien
Hi Fabien!

 php bin/console app:greet Fabien Potencier
Hi Fabien Potencier!
```

数组参数接收和使用

```
命令行请求：
php bin/console app:greet Fabien Ryan Bernhard
接收数组参数：
$this
    // ...
    ->addArgument(
        'names',
        InputArgument::IS_ARRAY,
        'Who do you want to greet (separate multiple names with a space)?'
    );
使用数组参数：
$names = $input->getArgument('names');
if (count($names) > 0) {
    $text .= ' '.implode(', ', $names);
}
```

addArgument()方法第二个参数有三个值可供使用：

+ InputArgument::REQUIRED

    必填参数，参数不存在则命令不运行。

+ InputArgument::OPTIONAL
        
    可选参数，默认值。

+ InputArgument::IS_ARRAY
    
    数组参数，将用空格分隔的参数作为数组获取，设置时在最后。
    
设置数组参数必填：

```
$this
    // ...
    ->addArgument(
        'names',
        InputArgument::IS_ARRAY | InputArgument::REQUIRED,
        'Who do you want to greet (separate multiple names with a space)?'
    );
```

### 使用命令参数



