## 创建

Bundle注册

*app/AppKernel.php*

```
class AppKernel extends Kernel
{
    public function registerBundles()
    {
        $bundles = [
            new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
            new Symfony\Bundle\TwigBundle\TwigBundle(),
            // ...
        ];
        // ...

        return $bundles;
    }

    // ...
}
```

控制器有两个基类Controller和AbstractController,两者区别在于AbstractController不能用$this->get()或者$this->controller->get()获取服务，继承AbstractController的控制器为一个服务。

Symfony4.2中Controller被废用，仅保留AbstractController。AbstractController在3.3版本中引入。

命令行创建

### 控制器创建

php bin/console make:controller BrandNewController

创建：

* src/Controller/BrandNewController.php
* templates/brandnew/index.html.twig
    
php bin/console make:crud Product

创建：

* src/Controller/ProductController.php
* src/Form/ProductType.php
* templates/product/_delete_form.html.twig
* templates/product/_form.html.twig
* templates/product/edit.html.twig
* templates/product/index.html.twig
* templates/product/new.html.twig
* templates/product/show.html.twig

## 响应类型

资源

```php
namespace AppBundle\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class LuckyController
{
    /**
     * @Route("/lucky/number")
     */
    public function numberAction()
    {
        $response = new Response('Hello '.$name, Response::HTTP_OK);
        // creates a CSS-response with a 200 status code
        $response = new Response('<style> ... </style>');
        $response->headers->set('Content-Type', 'text/css');
       
        $number = random_int(0, 100);
        $response2 = new Response( '<html><body>Lucky number: '.$number.'</body></html>');
        
        return $response;
        //return $response2;
    }
}
```

路径

```
$url = $this->generateUrl('app_lucky_number', ['max' => 10]);
```

页面

```php
class LuckyController extends Controller
{
    /**
     * @Route("/lucky/number")
     */
    public function numberAction()
    {
        $number = random_int(0, 100);

        return $this->render('lucky/number.html.twig', [
            'number' => $number,
        ]);
        
        // redirects to the "homepage" route
        return $this->redirectToRoute('homepage');
    
        // does a permanent - 301 redirect
        return $this->redirectToRoute('homepage', [], 301);
    
        // redirects to a route with parameters
        return $this->redirectToRoute('blog_show', ['slug' => 'my-page']);
    
        // redirects to a route and maintains the original query string parameters
        return $this->redirectToRoute('blog_show', $request->query->all());
    
        // redirects externally
        return $this->redirect('http://symfony.com/doc');
    }
}
```

redirect()不验证跳转路径的可靠性。

redirectToRoute()用RedirectResponse对象跳转，相当于：

```php
use Symfony\Component\HttpFoundation\RedirectResponse;

public function indexAction()
{
    return new RedirectResponse($this->generateUrl('homepage'));
}
```

##  获取服务

继承Controller的控制器

```php
$route = $this->container->get('route');
//or
//$route = $this->get('router');
$route->generate('blog', [
    'page' => 2,
    'category' => 'Symfony',
]);
```

## 控制器定义为服务

Symfony中控制器不必通过注册变成服务，使用默认服务即配置文件services.yml中设置内容，使用时对应控制器已经变成服务，可以和其他服务一样使用。

通过继承AbstractController，控制器定义为服务。

### 使用

作为服务时，只有配置文件中对应的public为true才能使用。

*注释*

指定服务id

```php
 use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
 
 /**
  * @Route(service="app.hello_controller")
  */
 class HelloController
 {
     // ...
 }
```

使用Controller中__invoke()方法

```php
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

/**
 * @Route("/hello/{name}", name="hello")
 */
class Hello
{
    public function __invoke($name = 'World')
    {
        return new Response(sprintf('Hello %s!', $name));
    }
}
```

*app/config/routing.yml*

```php
hello:
    path:     /hello
    defaults: { _controller: app.hello_controller:indexAction }
```

*app/config/routing.php*

```php
$collection->add('hello', new Route('/hello', [
    '_controller' => 'app.hello_controller:indexAction',
]));
```

使用id访问服务：

```php
$logger = $container->get('hello');
$entityManager = $container->get('app.hello_controller');
```

使用依赖注入访问服务。服务也可以作为成员方法中的参数使用。

```php
namespace AppBundle\Controller;

use Symfony\Component\HttpFoundation\Response;
use Twig\Environment;

class HelloController
{
    private $twig;

    public function __construct(Environment $twig)
    {
        $this->twig = $twig;
    }

    public function indexAction($name)
    {
        $content = $this->twig->render(
            'hello/index.html.twig',
            ['name' => $name]
        );

        return new Response($content);
    }
}
```

## 请求异常处理

使用createNotFoundException()方法跳转404界面。

```
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
public function index()
{
    // retrieve the object from database
    $product = ...;
    if (!$product) {
        throw $this->createNotFoundException('The product does not exist');
    }
    return $this->render(...);
}
```

HttpException类提供异常处理方法，抛出基本异常使用 throw new \Exception('Something went wrong!');

## Request对象使用

将Request对象作为控制器方法参数使用。

```
use Symfony\Component\HttpFoundation\Request;
public function index(Request $request, $firstName, $lastName)
{
    $page = $request->query->get('page', 1);
    $request->isXmlHttpRequest(); // is it an Ajax request?
    $request->getPreferredLanguage(['en', 'fr']);
    $request->query->get('page');
    $request->request->get('page');
    $request->server->get('HTTP_HOST');
    $request->files->get('foo');
    $request->cookies->get('PHPSESSID');
    $request->headers->get('host');
    $request->headers->get('content-type');
}
```

## 返回JSON对象

使用json()帮助方法，返回自动编码的JsonResponse对象。

```
public function index()
{
    return $this->json(['username' => 'jane.doe']);
}
```

如果您的应用程序中启用了序列化器服务，它将用于将数据序列化为JSON。否则，将使用json_encode函数。

## 流媒体文件响应

可以使用file()帮助器从控制器内部提供文件，并使用参数控制行为。

```
use Symfony\Component\HttpFoundation\File\File;
use Symfony\Component\HttpFoundation\ResponseHeaderBag;

public function download()
{
    // load the file from the filesystem
    $file = new File('/path/to/some_file.pdf');

    return $this->file($file);

    // rename the downloaded file
    return $this->file($file, 'custom_name.pdf');

    // 显示内容，非下载
    return $this->file('invoice_3241.pdf', 'my_invoice.pdf', ResponseHeaderBag::DISPOSITION_INLINE);
}
```

## 总结

