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