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




