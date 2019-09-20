## asset

```
asset 用于管理URL，网站css、script、图片等。
```

常见url硬编码：

```
<link rel="stylesheet" type="text/css" href="/css/main.css">
<a href="/"><img src="/images/logo.png"></a>
```

除非项目简单，否则不再推荐使用上述写法，原因如下：

+ **得到详细模板** ：必须为每个资源填写完成路径。使用Asset组件后可对资源进行分组，避免重复他们路径的公共部分。
+ **版本控制的困难** ：必须为每个应用进项自定义管理。一些应用必须为资源路径加版本，可控制缓存资源。Asset组件为每个包提供不同版本控制策略。
+ **移动资源位置** ：麻烦且容易犯错：需要在所有页面中小心改变资源的路径。Asset组件仅改变资源包相关的基本路径值，就可实现对应功能。
+ **几乎不可能使用多个CDN**：这种技术要求您为每个请求随机更改资产的URL。Asset组件为多个CDN提供out-of-the-box支持，包括http://和https://。

### 安装

```
composer require symfony/asset
```

在symfony外安装Asset组件，项目中必须包含vendor/autoload.php文件，以便自动加载所用组件。

### 使用

#### 资源包

Asset组件通过包管理资源，所有资源的分组分享相同的属性：版本控制、基本路径、CDN主机。

未设置版本的Asset包：

```
use Symfony\Component\Asset\Package;
use Symfony\Component\Asset\VersionStrategy\EmptyVersionStrategy;

$package = new Package(new EmptyVersionStrategy());

// Absolute path
echo $package->getUrl('/image.png');
// result: /image.png

// Relative path
echo $package->getUrl('image.png');
// result: image.png
```
Package类继承PackageInterface接口，两个默认方法：

getVersion()：返回资源版本

getUrl()：返回公共资源的绝对路径

#### 资源版本化