## 基础控制类和服务

可选基础控制器类AbstractController，可通过继承访问帮助方法。

*src/Controller/LuckyController.php*

```
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class LuckyController extends AbstractController
{
    public function test(){
        //返回 templates/lucky/number.html.twig
       return $this->render('lucky/number.html.twig', ['number' => $number]); 
    }
    public function test1(){
        $url = $this->generateUrl('app_lucky_number', ['max' => 10]);
        return $this->redirect($url);
        
        // 重定向到"homepage"路由
        return $this->redirectToRoute('homepage');
    
        // redirectToRoute is a shortcut for:
        // return new RedirectResponse($this->generateUrl('homepage'));
    
        //301跳转
        return $this->redirectToRoute('homepage', [], 301);
    
        // 带参数跳转到路由
        return $this->redirectToRoute('app_lucky_number', ['max' => 10]);
    
        // 带原始参数跳转到路由
        return $this->redirectToRoute('blog_show', $request->query->all());
    
        // 跳转到外部
        return $this->redirect('http://symfony.com/doc');
    }
    
    //获取服务
    public function number($max, LoggerInterface $logger)
    {
        $logger->info('We are logging!');
        // ...
    }
}
```

$this->render()获取模板数据。

$this->generateUrl()生成路径。

$this->redirect()用于使用路径重定向，$this->redirectToRoute()为 return new RedirectResponse($this->generateUrl('homepage'))简单方法。

可用依赖注入的方法获取参数，获取服务的相关配置：

*config/services.yaml*

```
services:
    # 显示配置服务
    App\Controller\LuckyController:
        public: true
        bind:
            # 对$logger参数设置特殊服务
            $logger: '@monolog.logger.doctrine'
            # 对$projectDir参数设置值
            $projectDir: '%kernel.project_dir%'
```

## 生成控制器

命令
+ bin/console make:controller BrandNewController

创建：

+ src/Controller/BrandNewController.php
+ templates/brandnew/index.html.twig

为实例生成CRUD,从Doctrine的entity：

+ php bin/console make:crud Product

创建：

+ src/Controller/ProductController.php
+ src/Form/ProductType.php
+ templates/product/_delete_form.html.twig
+ templates/product/_form.html.twig
+ templates/product/edit.html.twig
+ templates/product/index.html.twig
+ templates/product/new.html.twig
+ templates/product/show.html.twig

## 404页面

```
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

public function index()
{
    // retrieve the object from database
    $product = ...;
    if (!$product) {
        throw $this->createNotFoundException('The product does not exist');

        // 上述语句为其简单使用
        // throw new NotFoundHttpException('The product does not exist');
    }
    return $this->render(...);
}
```

$this->createNotFoundException()为throw new NotFoundHttpException()简单使用，最终都调用symfony内的404HTTP响应。

若抛出异常扩展或HttpException实例，symfony会抛出适当的异常代码。

```
//返回 500 错误代码
throw new \Exception('Something went wrong!');
```

## Request对象作为控制器参数

Request对象可实现读取请求参数、抓取请求头、访问上传文件等。

```
use Symfony\Component\HttpFoundation\Request;

public function index(Request $request, $firstName, $lastName)
{
    $page = $request->query->get('page', 1);
}
```

## session使用

config/packages/framework.yaml文件的[session配置](https://symfony.com/doc/current/reference/configuration/framework.html#config-framework-session) 。

controller加SessionInterface类型参数：

```
use Symfony\Component\HttpFoundation\Session\SessionInterface;

public function index(SessionInterface $session)
{
    // stores an attribute for reuse during a later user request
    $session->set('foo', 'bar');

    // gets the attribute set by another controller in another request
    $foobar = $session->get('foobar');

    // uses a default value if the attribute doesn't exist
    $filters = $session->get('filters', []);
}
```

[session详细内容](https://symfony.com/doc/current/session.html)

## 访问Messages:

flash信息仅执行一次，适合存储用户通知：

```
use Symfony\Component\HttpFoundation\Request;

public function update(Request $request)
{
    //$form为处理提交请求对象
    if ($form->isSubmitted() && $form->isValid()) {
        $this->addFlash(
            'notice',
            'Your changes were saved!'
        );
        // $this->addFlash() is equivalent to $request->getSession()->getFlashBag()->add()
        return $this->redirectToRoute(...);
    }
    return $this->render(...);
}
```

提交请求后，控制器再session中设置flash信息然后跳转。信息的key值可为任何值，例子中为“notice”，之后可用key值搜索信息。

模板使用Twig全局变量app对象提供的flashes()方法从session中读取flash信息。

```
{# templates/base.html.twig #}

{# read and display just one flash message type #}
{% for message in app.flashes('notice') %}
    <div class="flash-notice">
        {{ message }}
    </div>
{% endfor %}

{# read and display several types of flash messages #}
{% for label, messages in app.flashes(['success', 'warning']) %}
    {% for message in messages %}
        <div class="flash-{{ label }}">
            {{ message }}
        </div>
    {% endfor %}
{% endfor %}

{# read and display all flash messages #}
{% for label, messages in app.flashes %}
    {% for message in messages %}
        <div class="flash-{{ label }}">
            {{ message }}
        </div>
    {% endfor %}
{% endfor %}
```

以上例子使用notice、waring、error、作为flash信息不同类型的key值。

您可以使用peek()方法来检索消息，同时将消息保存在包中。

## 请求和响应对象

前面提到的请求对象作为控制器成员方法的参数：

```
use Symfony\Component\HttpFoundation\Request;

public function index(Request $request)
{
    $request->isXmlHttpRequest(); // is it an Ajax request?

    $request->getPreferredLanguage(['en', 'fr']);

    // retrieves GET and POST variables respectively
    $request->query->get('page');
    $request->request->get('page');

    // retrieves SERVER variables
    $request->server->get('HTTP_HOST');

    // retrieves an instance of UploadedFile identified by foo
    $request->files->get('foo');

    // retrieves a COOKIE value
    $request->cookies->get('PHPSESSID');

    // retrieves an HTTP request header, with normalized, lowercase keys
    $request->headers->get('host');
    $request->headers->get('content-type');
}
```

Response类对象有公共变量headers，该对象为ResponseHeaderBag类，并提供获取和设置响应头。

标头名称是标准化的。因此，名称Content-Type等价于名称Content-Type或content_type。

```
use Symfony\Component\HttpFoundation\Response;

// 创建简单的200状态码响应的Response对象，200是默认响应状态码
$response = new Response('Hello '.$name, Response::HTTP_OK);

// 创建css资源200状态码响应
$response = new Response('<style> ... </style>');
$response->headers->set('Content-Type', 'text/css');
```

不同的响应对象包含不同的响应类型，更多参照[HttpFoundation文档](https://symfony.com/doc/4.4/components/http_foundation.html#component-http-foundation-request)

## 访问配置变量

在控制器中获取配置变量，使用getParameter()方法：

```
public function index()
{
    $contentsDir = $this->getParameter('kernel.project_dir').'/contents';
}
```

## 返回JSON响应

使用jsonResponse对象的json()方法编码数据：

```
public function index()
{
    // returns '{"username":"jane.doe"}' and sets the proper Content-Type header
    return $this->json(['username' => 'jane.doe']);

    // the shortcut defines three optional arguments
    // return $this->json($data, $status = 200, $headers = [], $context = []);
}
```

若序列化服务在应用中可用，会序列化数组为惊悚数据，或者使用json_encode方法。

## 流文件响应

使用file()函数加载文件到控制器：

```
public function download()
{
    // load the file from the filesystem
    $file = new File('/path/to/some_file.pdf');

    return $this->file($file);

    // rename the downloaded file
    return $this->file($file, 'custom_name.pdf');

    // display the file contents in the browser instead of downloading it
    return $this->file('invoice_3241.pdf', 'my_invoice.pdf', ResponseHeaderBag::DISPOSITION_INLINE);
}
```