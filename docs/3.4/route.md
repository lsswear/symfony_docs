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
@Route 为 annotation 翻译为注释，可设置参数，例：
@Route("/test/{param}",name="test",requirements={"param"=".+"})
name为路由名称,requirement为对应的变量的规则正则定义。
特殊路由变量名包括：
    _locale 本地语言环境
    _format 规则
    _controller 路由对应控制器
    _fragment 苗点定义 以#开始
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
## 使用方法2-写在配置文件中
路由配置写在文件中
```php
   app/config/routing.yml YAML:
   app:
        resource: "@TestBundle/Controller/"
        type: annotation
        prefix: /site
        
   app/config/routing.php php:
   use Symfony\Bundle\FrameworkBundle\Controller\Controller;
   $collection = new RouteCollection();
   $collection->addCollection(
        $loader->import("@AppBundle/Controller/","annotation")
   );
   return $collection;
```
## 定义路由条件 
 