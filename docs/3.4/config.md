## 配置参数

### 查看命令

php bin/console config:dump-reference framework

php bin/console debug:config framework

### 解释

**参数列表**

[secret](#secret)    [http_method_override](#http_method_override)    [ide](#ide)

* * *

#### secret

参数名：kernel.secret 类型：string

默认：     必填：是

解释：*应用中唯一，由字符、数字、符号随机组成，32位左右。应用于记住登录时长、Edge Side Includes/数据缓冲服务器。
   应用中使用不可变随机字符串、为安全相关的操作增加熵时使用。
   更改此值将使所有签名的uri和cookie无效,故改后应重新生成应用程序缓存并注销所有应用程序用户。*

#### http_method_override

参数名：kernel.http_method_override 类型：boolean

默认：true     必填：是

解释：*是否将post默认为表单提交方法，是则自动调用Request::enableHttpMethodParameterOverride方法。从字面意义上看是_method重新设置。*

#### trusted_proxies

 在Symfony 3.3 版本删除。

#### ide

