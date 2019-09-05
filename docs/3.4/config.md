https://symfony.com/doc/3.4/reference/configuration/framework.html#a

## 配置参数

### 查看命令

php bin/console config:dump-reference framework

php bin/console debug:config framework

### 解释

#### 参数列表

[secret](#secret) | [http_method_override](#http_method_override) | [ide](#ide) | [test](#test)|[default_locale](#default_locale)|[trusted_hosts](#trusted_hosts)|
[form](#form) | [csrf_protection](#csrf_protection) | [esi](#esi) | [fragments](#fragments) | 

* * *

#### **secret**

参数名：*kernel.secret*   类型：*string*   必填：*是*

解释：*应用中唯一，由字符、数字、符号随机组成，32位左右。应用于记住登录时长、Edge Side Includes/数据缓冲服务器。
   应用中使用不可变随机字符串、为安全相关的操作增加熵时使用。
   更改此值将使所有签名的uri和cookie无效,故改后应重新生成应用程序缓存并注销所有应用程序用户。*


#### **http_method_override**

参数名：*kernel.http_method_override* 类型：*boolean* 默认：*true*     必填：*是*   $\color{red}{红色字}$

解释：*是否将post默认为表单提交方法，是则自动调用Request::enableHttpMethodParameterOverride方法。从字面意义上看是_method重新设置。*


#### **trusted_proxies**

 在Symfony 3.3 版本删除。

#### **ide**

参数名：framework.ide   类型：string

解释：

Symfony将变量转储和异常消息中的文件路径转换为链接，这些链接将在浏览器中打开这些文件。

如果您希望在您喜欢的IDE或文本编辑器中打开这些文件，请将此选项设置为以下值之一:phpstorm、sublime、textmate、macvim和emacs。

若使用其他ide则使用url，%f为中文路径，%l为行号，需用%转义防止Symfony解析。

*app/config/config.yml*

```php
framework:
    ide: 'myide://open?url=file://%%f&line=%%l'
```

*app/config/config.php*

```php
$container->loadFromExtension('framework', [
    'ide' => 'myide://open?url=file://%%f&line=%%l',
]);
```

由于每个开发人员都使用不同的IDE，因此推荐的启用该特性的方法是在系统级别上配置它。

可以通过设置xdebug来实现。php.ini配置文件中的file_link_format选项，使用的格式与框架相同，但不需要通过加%防止解析。

若framework.ide和xdebug.file_link_format同时设置，Symfony使用xdebug.file_link_format。

xdebug.file_link_format设置后，即使没有启用Xdebug扩展，也可以使用。

在容器或虚拟机中运行应用程序时，您可以告诉Symfony通过更改前缀将文件从客户机映射到主机。客户到主机的映射是在Symfony 3.2中引入的。

这个映射应该在URL模板的末尾指定，使用&和>作为来宾到主机的分隔符:

```php
// /path/to/guest/.../file will be opened
// as /path/to/host/.../file on the host
// and /foo/.../file as /bar/.../file also
'myide://%f:%l&/path/to/guest/>/path/to/host/&/foo/>/bar/&...'

```

[返回列表](#参数列表)

### **test**

参数名：*framework.ide*   类型：*boolean*

解释：

测试环境中使用，使用配置文件app/config/config_test.yml，为true则使用相关服务，如test.client等。

### **default_locale**

类型：*boolean*   默认：*en* 

解释：

未设置_local路由参数时使用，通过Request::getDefaultLocale方法使用。

### **trusted_hosts**
 
参数名:*framework.trusted_hosts* 类型：*array|string*   默认：*[]*

各种软件(web服务器、反向代理、web框架等)处理主机报头时的不一致性导致了许多不同的攻击。

基本上，每次框架生成一个绝对URL(例如发送电子邮件重置密码时)，主机都可能被攻击者操纵。

Symfony Request::getHost()方法可能容易受到这些攻击，因为它取决于web服务器的配置。

避免这些攻击的一个简单解决方案是将Symfony应用程序可以响应的主机列入白名单。

这就是trusted_hosts选项的目的。如果传入请求的主机名与此列表中的正则表达式不匹配，应用程序将不响应，用户将收到响应代码400。

默认空数组，可以匹配任何主机域名。

*app/config/config.yml*

```php
framework:
    trusted_hosts:  ['^example\.com$', '^example\.org$']
```

*app/config/config.php*

```php
$container->loadFromExtension('framework', [
    'trusted_hosts' => ['^example\.com$', '^example\.org$'],
]);
```

主机也可以配置为响应任何子域，例如通过^(.+\.)?example\.com$。

此外，还可以使用Request::setTrustedHosts()方法在前端控制器中设置可信主机:

*web/app.php*

```php
Request::setTrustedHosts(['^(.+\.)?example\.com$', '^(.+\.)?example\.org$']);
```

[返回列表](#参数列表)

### **form**

#### enabled

类型：*boolean* 默认：*false*

解释：

是否在服务容器中启用表单服务。如果不使用表单，将其设置为false可能会提高应用程序的性能，因为将加载到容器中的服务更少。

当配置了其中一个子设置时，该选项将自动设置为true。为true时自动启动验证。

### **csrf_protection**

#### enabled

类型: *boolean*  默认: *true* 

表单中csrf保护，若表单支持则启用，反之则为false。

https://symfony.com/doc/3.4/form/csrf_protection.html

此选项可用于在所有表单上禁用CSRF保护。但是您也可以在个别表单上禁用CSRF保护。

如果您正在使用表单，但希望避免启动会话(例如在API-only网站中使用表单)，则需要将csrf_protection设置为false。


### **esi**

#### enabled

类型：*boolean*   默认:*false*

解释：

是否在框架中启用edge side includes的支持。

### **fragments**

#### enabled

类型：*boolean*   默认:*false*

解释：

是否启用片段侦听器。片段侦听器用于独立于页面的其余部分呈现ESI片段。

当配置了其中一个子设置时，此设置将自动设置为true。

#### path

类型：*string*   默认:*'/_fragment'*

解释：

片段的路径前缀。片段侦听器只会在请求以该路径开始时执行。

### **profiler**

* #### enabled

类型：*boolean*   默认:*false*

解释：

为true时启用，当使用Symfony标准版时，在开发和测试环境中启用了分析器。

* #### collect

类型：*boolean*   默认:*true*

解释：

此选项配置启用分析器时的行为方式。如果设置为true，分析器将为所有请求收集数据(除非您进行其他配置，比如自定义匹配器)。
如果你只想按需收集资料，你可以将collect标志设为false，然后手动启动数据收集程序:$profiler->enable();

* #### only_exceptions

类型：*boolean*   默认:*false*

解释：

当该值设置为true时，只有在处理请求抛出异常时才启用分析器。

* #### only_master_requests

类型：*boolean*   默认:*false*

解释：

当这个值设置为true时，剖析器只会在主请求上启用(而不会在子请求上启用)。

* #### dsn


[返回列表](#参数列表)


