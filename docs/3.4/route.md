## 使用方法1-写在控制器中

 控制器里使用

```php
 use Symfony\Bundle\FrameworkBundle\Controller\Controller;
 use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
```

  在方法定义上一行,写注释

```php
 /**
  * Matches /blog exactly / 精确匹配了/blog
  *
  * @Route("/blog", name="blog_list")
  */
```

### 样例

```php
    namespace AppBundle\Controller    
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    class TestController extends Controller
    {
        /**
         * @Route("/test",name="test_index")
         */
         public function indexAction(){
           //edit 
         }
         /**
          * @Route("/test/{param}",name="test_list")
          */
          public function listAction($perem){
            //edit 
          }
    }
```

@Route 为 annotation 翻译为注释，可设置参数，例：
@Route("/test/{param}",name="test",requirements={"param"=".+"})

name为路由名称,requirement为对应的变量的规则正则定义。

### 路由中使用变量

*app/config/config.yml*

```php
parameters:
    app.locales: en|es
    app.route_prefix: 'foo'
```

*app/config/routing.yml*

```php
contact:
    path:     /{_locale}/contact
    defaults: { _controller: AppBundle:Main:contact }
    requirements:
        _locale: '%app.locales%'
some_route:
    path:     /%app.route_prefix%/contact
    defaults: { _controller: AppBundle:Main:contact }        
```

*app/config/config.php*

```php
$container->setParameter('app.locales', 'en|es');
$container->setParameter('app.route_prefix', 'foo');
```

*app/config/routing.php*

```php
    use Symfony\Component\Routing\Route;
    use Symfony\Component\Routing\RouteCollection;
    
    $routes = new RouteCollection();
    $routes->add('contact', 
                new Route('/{_locale}/contact', 
                        ['_controller' => 'AppBundle:Main:contact',], 
                        [ '_locale' => '%app.locales%',]
                        )
                );
    $routes->add('some_route', 
                new Route('/%app.route_prefix%/contact', 
                        ['_controller' => 'AppBundle:Main:contact',]
                        )
                );
    return $routes;
```

特殊路由变量名包括：

+  _locale 本地语言环境
+ _format 规则
+ _controller 路由对应控制器
+ _fragment 苗点定义 以#开始

### 匹配服务器请求

### 设置前缀名

```php
    /**
     * @Route(name="blog_")
     */
    class TestController extends Controller{
        //
    }
```


* * *

## 使用方法2-写在配置文件中

路由配置写在文件中,可以为参数设置默认值及正则规则。

*app/config/routing.yml*

```php
   homepage:
        path:/
        default:{_controller:AppBundle:Main:homepage}
   article_show:
       path:     /articles/{_locale}/{year}/{slug}.{_format}
       defaults: { _controller: AppBundle:Article:show,year:1, _format: html }
       requirements:
           _locale:  en|fr
           _format:  html|rss
           year:     \d+
```           

*app/config/routing.php*

```php
   use Symfony\Component\Routing\Route;
   use Symfony\Component\Routing\RouteCollection;
   
   $routes = new RouteCollection();
   
   $routes->add('homepage', 
                new Route('/', 
                    ['_controller' => 'AppBundle:Main:homepage',]
                    )
                );
   
   $routes->add('article_show',
                new Route('/articles/{_locale}/{year}/{slug}.{_format}', 
                    [
                       '_controller' => 'AppBundle:Article:show',
                       '_format'     => 'html',
                       'year'        => 1
                   ], 
                   [
                       '_locale' => 'en|fr',
                       '_format' => 'html|rss',
                       'year'    => '\d+',
                   ]
               )
            );
   
   return $routes;
```

### 匹配服务器请求

*app/config/routing.yml*

```php
api_post_show:
    host:     m.example.com
    path:     /api/posts/{id}
    defaults: { _controller: AppBundle:BlogApi:show }
    methods:  [GET,HEAD,PUT]
mobile_homepage:
    path:     /
    host:     "{subdomain}.example.com"
    defaults:
        _controller: AppBundle:Main:mobileHomepage
        subdomain: m
    requirements:
        subdomain: m|mobile
projects_homepage:
    path:     /
    host:     "{project_name}.example.com"
    defaults: { _controller: AppBundle:Main:projectsHomepage }
mobile_homepage:
    path:     /
    host:     "m.{domain}"
    defaults:
        _controller: AppBundle:Main:mobileHomepage
        domain: '%domain%'
    requirements:
        domain: '%domain%'
        
```

*app/config/routing.php*

```php       
use Symfony\Component\Routing\Route;
use Symfony\Component\Routing\RouteCollection;

$routes = new RouteCollection();
$routes->add('mobile_homepage', 
            new Route(
                '/', 
                ['_controller' => 'AppBundle:Main:mobileHomepage',], 
                [], 
                [], 
                'm.example.com')
            );
$routes->add('project_homepage', new Route('/', [
    '_controller' => 'AppBundle:Main:projectsHomepage',
], [], [], '{project_name}.example.com'));

$routes->add('homepage', new Route('/', [
    '_controller' => 'AppBundle:Main:homepage',
]));

$routes = new RouteCollection();
$routes->add('mobile_homepage', new Route('/', [
    '_controller' => 'AppBundle:Main:mobileHomepage',
    'subdomain'   => 'm',
], [
    'subdomain' => 'm|mobile',
], [], '{subdomain}.example.com'));

$routes->add('homepage', new Route('/', [
    '_controller' => 'AppBundle:Main:homepage',
]));
return $routes;
       
```

### 加载资源

可根据Bundle、配置文件、包含配置文件的文件夹中加载资源。根据Bundle加载资源类型为annotation，根据文件夹加载资源类型为directory。

还可以设置前缀名，比如原路径为test/test1,前缀名为TEST,则路径为/TEST/test/test1。根据php代码，之前的内容应该也能使用前缀名。

同样也能使用之前提到的内容，比如host等。

*app/config/routing.yml*

```php
   app_file:
       # loads routes from the given routing file stored in some bundle
       resource: '@AcmeBundle/Resources/config/routing.yml'
   
   app_annotations:
       # loads routes from the PHP annotations of the controllers found in that directory
       resource: '@AppBundle/Controller/'
       type:     annotation
   
   app_directory:
       # loads routes from the YAML, XML or PHP files found in that directory
       resource: '../legacy/routing/'
       type:     directory
   
   app_bundle:
       # loads routes from the YAML, XML or PHP files found in some bundle directory
       resource: '@AcmeOtherBundle/Resources/config/routing/'
       type:     directory
    app:
        resource: "@TestBundle/Controller/"
        type: annotation
        host: "hello.example.com"
        prefix: /site
```

*app/config/routing.php*

```php
use Symfony\Component\Routing\RouteCollection;

$routes = new RouteCollection();

// 资源路径,资源类型
$app = $loader->import('@AppBundle/Controller/', 'annotation');
$app->addPrefix('/site');
$routes->addCollection($app);
$routes->addCollection( 
    // loads routes from the given routing file stored in some bundle
    $loader->import("@AcmeBundle/Resources/config/routing.yml")
};

// loads routes from the PHP annotations of the controllers found in that directory
$app = $loader->import("@AppBundle/Controller/", "annotation")
$app->setHost('hello.example.com');
$routes->addCollection($app);

$routes->addCollection(
    // loads routes from the YAML or XML files found in that directory
    $loader->import("../legacy/routing/", "directory")
);
$routes->addCollection(
    // loads routes from the YAML or XML files found in some bundle directory
    $routes->import('@AcmeOtherBundle/Resources/config/routing/public/', 'directory');
);
return $routes;
```

* * *

## 控制器命名模式

Symfony取控制器名中Controller之前的字符为控制器名，去方法名中Action之前的字符为方法名，如Test=>TestController test=>testAction。

控制器命名模式 格式为 Bundle名:控制器名:方法名 即bundle:controller:action。

例如其全名为:AppBundle\Controller\BlogController::showAction，控制器命名模式就为:AppBundle:Blog:show。

由于控制中实现了__invoke()方法，所以控制器类可以作为方法使用。
  
## 页面中创建路径

Twig页面引擎使用path()设置路径参数，escape()保证参数值正确。FOSJsRoutingBundle提供对象在页面中使用generate()。

```
<script>
    let route = "{{ path('blog_show', {'slug': 'my-blog-post'})|escape('js') }}";
</script>
<script>
    let url = Routing.generate('blog_show', {
        'slug': 'my-blog-post'
    });
</script>
```

## 控制器中使用路由

使用router服务的generate()生成相对路由，使用generateUrl()生成绝对路由，默认为相对路由。

*相对路由:/blog/2?category=Symfony*

```php
$this->get('router')->generate('blog', [
    'page' => 2,
    'category' => 'Symfony',
]);
```

*绝对路由:http://www.example.com/blog/my-blog-post*

```php
use Symfony\Component\Routing\Generator\UrlGeneratorInterface;

$this->generateUrl('blog_show', ['slug' => 'my-blog-post'], UrlGeneratorInterface::ABSOLUTE_URL);
```

* * *

Symfony不支持根据用户语言定义不同内容的路由。

在这些情况下，可以为每个控制器定义多条路由，每条路由对应一种受支持的语言，或者使用社区创建的任何包来实现该特性。

比如JMSI18nRoutingBundle和BeSimpleI18nRoutingBundle。