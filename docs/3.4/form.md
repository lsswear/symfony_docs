https://symfony.com/doc/3.4/forms.html

## 改提交方法

### 控制器中修改

使用 FormBuilder 中的 setAction()和setMethod()。

*AppBundle/Controller/DefaultController.php*

```php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\Form\Extension\Core\Type\DateType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Component\Form\Extension\Core\Type\TextType;

class DefaultController extends Controller
{
    public function newAction()
    {
        $form = $this->createFormBuilder($task)
            ->setAction($this->generateUrl('target_route'))
            ->setMethod('GET')
            ->add('task', TextType::class)
            ->add('dueDate', DateType::class)
            ->add('save', SubmitType::class)
            ->getForm();
    }
}

```

在参数中修改

*AppBundle/Controller/DefaultController.php*

```php
namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use AppBundle\Form\TaskType;

class DefaultController extends Controller
{
    public function newAction()
    {
        $form = $this->createForm(TaskType::class, $task, [
            'action' => $this->generateUrl('target_route'),
            'method' => 'GET',
        ]);
    }
}
```

### 独立使用

```php
use Symfony\Component\Form\Forms;
use Symfony\Component\Form\Extension\Core\Type\DateType;
use Symfony\Component\Form\Extension\Core\Type\FormType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use AppBundle\Form\TaskType;

$formFactoryBuilder = Forms::createFormFactoryBuilder();

$formFactory = $formFactoryBuilder->getFormFactory();

//使用 FormBuilder 中的 setAction()和setMethod()。
$form = $formFactory->createBuilder(FormType::class, $task)
    ->setAction('...')
    ->setMethod('GET')
    ->add('task', TextType::class)
    ->add('dueDate', DateType::class)
    ->add('save', SubmitType::class)
    ->getForm();
    
//在参数中修改
$form = $formFactory->create(TaskType::class, $task, [
    'action' => '...',
    'method' => 'GET',
]);
```

### 页面中修改

*app/Resources/views/default/new.html.twig*

```php
{{ form_start(form, {'action': path('target_route'), 'method': 'GET'}) }}
```

若页面中method非post或get，而是put、patch、DELETE则加隐藏字段_method，用于设置请求方法，配合http_method_override选项，程序和解析form请求方法类型。