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

### 使用命令选项

选项不排序，前缀有两个"-"英文符号，比如"--yell"。选项总是非必须的，可以设置值，如 --dir=src,值为布尔值则不用设置，如 --yell。

例如，加一个命令参数，可以指定一行信息被打印多少次：

```
use Symfony\Component\Console\Input\InputOption;
protected function configure()
{
   $this->addOption(
       'iterations',
       null,
       InputOption::VALUE_REQUIRED,
       'How many times should the message be printed?',
       1
   );  
}
protected function execute(InputInterface $input, OutputInterface $output)
{
    for ($i = 0; $i < $input->getOption('iterations'); $i++) {
        $output->writeln($text);
    }
}
```

指定 --iterations标签使用：

```
//--iterations未设置默认为1
> php bin/console app:greet Fabien
Hi Fabien!

> php bin/console app:greet Fabien --iterations=5
Hi Fabien
Hi Fabien
Hi Fabien
Hi Fabien
Hi Fabien

//包含未设置标签 则命令行不起作用
> php bin/console app:greet Fabien --iterations=5 --yell
> php bin/console app:greet Fabien --yell --iterations=5
> php bin/console app:greet --yell --iterations=5 Fabien
```

可以定义单个"-"英文字符结构：

```
$this->addOption(
        'iterations',
        'i',
        InputOption::VALUE_REQUIRED,
        'How many times should the message be printed?',
        1
    );
```

第二个参数i为iterations的简写，命令号显示 -i。

双"--"可以用空格和等号设置值，如 --iterations 5 或 --iterations=5。单"-"仅可以用空格和无字符设置，如 -i 5 或 -i5。

对于选项的参数，可用有四个：

+ InputOption::VALUE_IS_ARRAY
    可接收多个参数，如 --dir=/foo -dir=/bar。
+ InputOption::VALUE_NONE
    不接收值，使用默认值。
+ InputOption::VALUE_REQUIRED
    选项都必填。
+ InputOption::VALUE_OPTIONAL
    选项都非必填。
    
VALUE_IS_ARRAY可以和VALUE_REQUIRED或VALUE_OPTIONAL使用：

```
$this->addOption(
        'colors',
        null,
        InputOption::VALUE_REQUIRED | InputOption::VALUE_IS_ARRAY,
        'Which colors do you like?',
        ['blue', 'red']
    );
```

### 选项和选项数组

比较麻烦的情况比如：

```
use Symfony\Component\Console\Input\InputOption;
$this->addOption(
        'yell',
        null,
        InputOption::VALUE_OPTIONAL,
        'Should I yell while greeting?'
    );
```

以上定义有三种方式设值，--yell 或 --yell=value 或不传值, 很难区分传递没有值和不传递的情况。

为解决这种方法需要设置默认值：

```
use Symfony\Component\Console\Input\InputOption;
$this->addOption(
        'yell',
        null,
        InputOption::VALUE_OPTIONAL,
        'Should I yell while greeting?',
        //用默认值代替null
        false 
    );
```

检查yell值：

```
$optionValue = $input->getOption('yell');
$yell = ($optionValue !== false);
$yellLouder = ($optionValue === 'louder');
```

### 命令行显示配置

```
protected function configure()
{
    $this
        // 命令行描述 php bin/console list  运行时显示
        ->setDescription('Creates a new user.')

        // 命令号详细描述 运行--help时显示
        ->setHelp('This command allows you to create a user...')
    ;
}
```

configure()方法自动在command类的构造函数之后。若自定义command构造，先设置属性再调用父类构造函数。

```
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputArgument;

class CreateUserCommand extends Command
{
    public function __construct(bool $requirePassword = false)
    {
        // best practices recommend to call the parent constructor first and
        // then set your own properties. That wouldn't work in this case
        // because configure() needs the properties set in this constructor
        $this->requirePassword = $requirePassword;
        parent::__construct();
    }
    protected function configure()
    {
        $this->addArgument(
                'password', 
                $this->requirePassword ? 
                    InputArgument::REQUIRED : 
                    InputArgument::OPTIONAL, 
                 'User password'
        );
    }
}
```

## 注册命令

命令必须先注册后使用。为了命令自动注册，一个命令必须：
+ 在一个文件夹下存储Command文件夹
+ 在Command文件夹下定义Command类
+ 定义的类继承Command类

若不满足以上条件可以手动执行。

### 命令作为服务注册
 
若使用services.yml配置文件，你的命令已定义为服务，此为推荐设置。
 
Symfony也会查找每个bundle中的Command文件为了未注册为服务和未自动注册的command类。这种自动注册的方法在3.4版本中建议使用。4.0版本中命令不可被任何方式自动注册。

若继承ContainerAwareCommand类可实现公共服务功能通过$this->getContainer()->get('SERVICE_ID')。

注册为服务，通过依赖注射访问服务。

```
namespace AppBundle\Command;

use Psr\Log\LoggerInterface;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

class SunshineCommand extends Command
{
    protected static $defaultName = 'app:sunshine';
    private $logger;

    public function __construct(LoggerInterface $logger)
    {
        $this->logger = $logger;

        // you *must* call the parent constructor
        parent::__construct();
    }

    protected function configure()
    {
        $this
            ->setDescription('Good morning!');
    }

    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $this->logger->info('Waking up the sun');
        // ...
    }
}
```

没必要通过configure()访问服务。若命令非懒加载，避免做任何工作，如数据库访问，会在在加载时运行，即使是在控制台使用不同命令。

### 懒加载

3.4版本引入支持懒加载。

设置静态变量$defaultName：

```
class SunshineCommand extends Command
{
    protected static $defaultName = 'app:sunshine';
}
```

或者在配置文件中定义：

*app/config/services.yml*

```
services:
    AppBundle\Command\SunshineCommand:
        tags:
            - { name: 'console.command', command: 'app:sunshine' }
```

*app/config/services.php*

```
use AppBundle\Command\SunshineCommand;
$container
    ->register(SunshineCommand::class)
    ->addTag('console.command', ['command' => 'app:sunshine']);
```

若使用getAliases()方法设置别名，则必须为每个别名添加一个console.command标记。

根据以上代码，SunshineCommand只有运行app:sunshine命令才会实例化。

list命令会列出所有命令，包括懒加载命令。

## 执行命令

配置和注册命令之后可执行命令：

```
php bin/console app:create-user
```

编写execute访问输入流和输出流：

```
protected function execute(InputInterface $input, OutputInterface $output)
{
    // 输出多行到控制台，每行加换行
    $output->writeln([
        'User Creator',
        '============',
        '',
    ]);
    // outputs a message followed by a "\n"
    $output->writeln('Whoa!');
    // 每行不输出换行
    $output->write('You are about to ');
    $output->write('create a user.');
}
```
执行效果：

```
> php bin/console app:create-user
User Creator
============

Whoa!
You are about to create a user.
```

## 控制台输入

通过选项或数组向命令输入信息：

```
use Symfony\Component\Console\Input\InputArgument;

// ...
protected function configure()
{
    $this
        // configure an argument
        ->addArgument('username', InputArgument::REQUIRED, 'The username of the user.');
}
public function execute(InputInterface $input, OutputInterface $output)
{
    $output->writeln([
        'User Creator',
        '============',
        '',
    ]);
    // retrieve the argument value using getArgument()
    $output->writeln('Username: '.$input->getArgument('username'));
}
```

执行效果：

```
> php bin/console app:create-user Wouter
User Creator
============

Username: Wouter
```

## 从服务容器中获取服务

只要命令注册为服务，就可以通过依赖注射访问服务：

```
use AppBundle\Service\UserManager;
use Symfony\Component\Console\Command\Command;

class CreateUserCommand extends Command
{
    private $userManager;

    public function __construct(UserManager $userManager)
    {
        $this->userManager = $userManager;
        parent::__construct();
    }
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $this->userManager->create($input->getArgument('username'));
        $output->writeln('User successfully generated!');
    }
}
```

## 命令声明周期

命令运行时有三个生命周期方法：

+ initialize() 可选
    执行在interact()和execute()之前，初始化其余命令中使用的变量。
+ interact() 可选
    执行在interact()之后，execute()之前，检查选项或参数是否有缺失，交互请求式询用户这些值。
    这是最后询问缺失选项或参数第地方。命令执行之后，缺少选项或参数会导致失败。
+ execute() 必填
    执行在interact()和initialize()之后 ，包含命令执行逻辑。
     
## 命令测试

symfony提供几个工具帮助测试命令。最有用的是CommandTester类。

它使用特殊的输入和输出简化测试，并不需要真正的控制台：

*tests/AppBundle/Command/CreateUserCommandTest.php*

```
namespace Tests\AppBundle\Command;

use AppBundle\Command\CreateUserCommand;
use Symfony\Bundle\FrameworkBundle\Console\Application;
use Symfony\Bundle\FrameworkBundle\Test\KernelTestCase;
use Symfony\Component\Console\Tester\CommandTester;

class CreateUserCommandTest extends KernelTestCase
{
    public function testExecute()
    {
        $kernel = static::createKernel();
        $application = new Application($kernel);
        $command = $application->find('app:create-user');
        $commandTester = new CommandTester($command);
        $commandTester->execute([
            'command'  => $command->getName(),
            // pass arguments to the helper
            'username' => 'Wouter',
            // prefix the key with two dashes when passing options,
            //执行效果： '--some-option' => 'option_value',
        ]);
        // the output of the command in the console
        $output = $commandTester->getDisplay();
        $this->assertContains('Username: Wouter', $output);
    }
}
```

还可以使用ApplicationTester测试整个控制台应用程序。

独立项目使用控制台组件，使用Symfony\Component\Console\Application并且继承\PHPUnit\Framework\TestCase。

## 命令行错误日志

命令运行时任何错误输出，symfony为它添加一条日志包括整个失败的命令。

Symfony注册一个事件订阅者来监听ConsoleEvents::TERMINATE事件，并在命令没有以0退出状态结束时添加一个日志消息。
 






