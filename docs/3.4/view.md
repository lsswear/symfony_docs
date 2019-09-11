## 视图资源位置

### 位置： app/Resources/

```php
//app/Resources/views/lucky/number.html.twig
$this->render('lucky/number.html.twig', array('number' => $number));
//子目录
//app/Resources/views/lottery/lucky/number.html.twig
$this->render('lottery/lucky/number.html.twig', array( 'number' => $number,));
```

### 404页面

```php
     throw $this->createNotFoundException('The product does not exist');
```

## 模板

模板引擎[Twig](https://twig.symfony.com/)，基本语法：
 * {{$value}} 变量输出
 * {%  %} 逻辑方法输出，if、for等
 * {# ... #} 注释
 * {{ title|upper }} Twig附带一长串标签、过滤器和默认可用的函数。您甚至可以通过Twig扩展添加自己的自定义过滤器、函数(以及更多)。
 
php bin/console debug:twig 

列出Twig可用内容。

```
{% for i in 1..10 %}
    <div class="{{ cycle(['even', 'odd'], i) }}">
        <!-- some HTML here -->
    </div>
{% endfor %}
```

## 模板缓存

Twig自动将模板编译成原生php类并缓存，模板修改后重新编译。

## 模板继承和布局

layout文件（母模板）：

*app/Resources/views/base.html.twig*

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Test Application{% endblock %}</title>
    </head>
    <body>
        <div id="sidebar">
            {% block sidebar %}
                <ul>
                    <li><a href="/">Home</a></li>
                    <li><a href="/blog">Blog</a></li>
                </ul>
            {% endblock %}
        </div>

        <div id="content">
            {% block body %}{% endblock %}
        </div>
    </body>
</html>
```

子模板：

母模板的{% block %}可在子模板中重写，或者保留期原来的内容。

*app/Resources/views/blog/index.html.twig*

```
{% extends 'base.html.twig' %}
{% block title %}My cool blog posts{% endblock %}
{% block body %}
    {% for entry in blog_entries %}
        <h2>{{ entry.title }}</h2>
        <p>{{ entry.body }}</p>
    {% endfor %}
{% endblock %}
```

模板可多层继承，假设已存在base.html.twig，layout.html.twig可继承base.html.twig，index.html.twig可继承layout.html.twig。

模板处理注意事项：

* {% extends %}必须是第一行
* 基本模板中的{% block %}标记越多越好，可以使模板更灵活
* 重复的代码应设置在{% block %}中，并在其他模板中引入（include）
* 使用{{parent()}}从父模板中欧获取内容，保留原模块内容使用

```
{% block sidebar %}
    <h3>Table of Contents</h3>
    {# ... #}
    {{ parent() }}
{% endblock %}
```

## 模板名字和存储位置

存贮位置：

app/Resources/views/

包含基本模板，layout、基础页面以及覆盖第三方包模板的模板。

vendor/path/to/CoolBundle/Resources/views/

第三方模板存贮位置。

*大多数页面存放在app/Resources/views/，一般使用相对路径，base.html.twig为app/Resources/views/base.html.twig，blog/index.html.twig为app/Resources/views/blog/index.html.twig。*

## 引用Bundle中的模板

引用Bundle中的模板使用命名空间语法，例如：@BundleName/directory/filename.html.twig、@AcmeBlog/layout.html.twig。

以上例子分别是两种模板类型，位于两个不同位置：

+ @AcmeBlog/Blog/index.html.twig：

  - @AcmeBlog：不包含Bundle的包名，表示模板存在于src/Acme/BlogBundle中
  - Blog：子文件夹名，位置：Resources/views/Blog。
  - index.html.twig： 模板文件名，位置：src/Acme/BlogBundle/Resources/views/Blog/index.html.twig。

+ @AcmeBlog/layout.html.twig：
    - layout.html.twig：基础模板名称，位置src/Acme/BlogBundle/Resources/views/layout.html.twig

## 第三方包模板重写
    
假设情况，业务需求重写src/BundleName/Resources/views/Blog/index.html.twig 复制该文件到 app/Resources/AcmeBlogBundle/views/Blog/index.html.twig，若app/Resources/AcmeBlogBundle不存在则需要新建。

*app/Resources/AcmeBlogBundle/views/Blog/index.html.twig*

模板名前加！，则从原始模板扩展，而不是从覆盖模板扩展，3.4中引入。

```
{% extends "@!AcmeBlog/index.html.twig" %}
{% block some_block %}
{% endblock %}
```

新建模板存储位置，最好清除缓存php bin/console cache:clear，否做可能会有其他bug。

Bundle继承（inheritance）在3.4中废弃。

## 重写核心模板

核心模板指核心包中的模板，例如“exception”、“error”模板。

从Resources/views/Exception文件夹复制出，拷贝到app/Resources/TwigBundle/views/Exception文件夹。

## 模板后缀名

|文件名|格式|模板引擎|
|----|----|----|
|blog/index.html.twig|html|Twig|
|blog/index.html.php|html|PHP|
|blog/index.css.php|css|Twig|

一个文件的两个后缀名表示两个引擎，在项目中都要使用，第一个后缀名为文件最终生成格式。

## 模板中输出不同格式

根据公约，不同格式显示不同格式的内容。

根据$slug请求变量或者请求的格式，显示不同模板的不同格式。

```
use Symfony\Component\Routing\Annotation\Route;

class ArticleController extends Controller
{
    /**
     * @Route("/{slug}")
     */
    public function showAction(Request $request, $slug)
    {
        // retrieve the article based on $slug
        $article = ...;

        $format = $request->getRequestFormat();

        return $this->render('article/show.'.$format.'.twig', [
            'article' => $article,
        ]);
    }
}
```

getRequestFormat()获取Request对象默认格式html，请求格式可根据路由参数判断，可以更改_format过滤请求格式。

```
/**
 * @Route("/{slug}.{_format}", defaults={"_format"="html"}, requirements={"_format"="html|xml"}))
 */
public function showAction(Request $request, $slug)
{
    // ...
}
```

路径生成

```
<a href="{{ path('article_show', {'slug': 'about-us', '_format': 'xml'}) }}">
    View as XML
</a>
```

构建API时，最好不使用文件扩展名。FOSRestBundle提供了一个使用内容协商的请求侦听器。