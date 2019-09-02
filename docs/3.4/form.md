## 改提交方法



*AppBundle/Controller/DefaultController.php*

使用 FormBuilder 中的 setAction()和setMethod()。

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
