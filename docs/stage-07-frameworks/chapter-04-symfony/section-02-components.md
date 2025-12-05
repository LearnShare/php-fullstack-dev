# 7.4.2 组件系统

## 概述

Symfony 提供了丰富的组件系统。本节介绍 Messenger（消息队列和总线）、Workflow（业务流程状态机）、API Platform 等高级组件。

## Messenger

### 消息定义

```php
<?php
declare(strict_types=1);

namespace App\Message;

class SendEmailMessage
{
    public function __construct(
        public string $to,
        public string $subject,
        public string $body
    ) {}
}
```

### 消息处理器

```php
<?php
declare(strict_types=1);

namespace App\MessageHandler;

use App\Message\SendEmailMessage;
use Symfony\Component\Messenger\Handler\MessageHandlerInterface;

class SendEmailHandler implements MessageHandlerInterface
{
    public function __invoke(SendEmailMessage $message): void
    {
        mail($message->to, $message->subject, $message->body);
    }
}
```

### 发送消息

```php
<?php
declare(strict_types=1);

use Symfony\Component\Messenger\MessageBusInterface;

class UserService
{
    public function __construct(
        private MessageBusInterface $bus
    ) {}
    
    public function createUser(array $data): void
    {
        // 创建用户
        $user = new User($data);
        
        // 发送消息
        $this->bus->dispatch(new SendEmailMessage(
            $user->getEmail(),
            'Welcome',
            'Welcome message'
        ));
    }
}
```

## Workflow

### 定义工作流

```yaml
# config/packages/workflow.yaml
framework:
    workflows:
        order:
            type: state_machine
            marking_store:
                type: method
                property: 'state'
            supports:
                - App\Entity\Order
            initial_marking: new
            places:
                - new
                - paid
                - shipped
                - delivered
            transitions:
                pay:
                    from: new
                    to: paid
                ship:
                    from: paid
                    to: shipped
                deliver:
                    from: shipped
                    to: delivered
```

### 使用工作流

```php
<?php
declare(strict_types=1);

use Symfony\Component\Workflow\WorkflowInterface;

class OrderService
{
    public function __construct(
        private WorkflowInterface $orderWorkflow
    ) {}
    
    public function payOrder(Order $order): void
    {
        if ($this->orderWorkflow->can($order, 'pay')) {
            $this->orderWorkflow->apply($order, 'pay');
        }
    }
}
```

## API Platform

### 实体定义

```php
<?php
declare(strict_types=1);

namespace App\Entity;

use ApiPlatform\Metadata\ApiResource;
use ApiPlatform\Metadata\Get;
use ApiPlatform\Metadata\Post;
use Doctrine\ORM\Mapping as ORM;

#[ApiResource(
    operations: [
        new Get(),
        new Post(),
    ]
)]
#[ORM\Entity]
class User
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    public ?int $id = null;
    
    #[ORM\Column]
    public string $name;
    
    #[ORM\Column]
    public string $email;
}
```

### 自动生成 API

```yaml
# config/packages/api_platform.yaml
api_platform:
    title: My API
    version: 1.0.0
    formats:
        jsonld: ['application/ld+json']
        json: ['application/json']
```

## 完整示例

```php
<?php
declare(strict_types=1);

namespace App\Controller;

use App\Message\SendEmailMessage;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Messenger\MessageBusInterface;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\Workflow\WorkflowInterface;

class OrderController extends AbstractController
{
    public function __construct(
        private MessageBusInterface $bus,
        private WorkflowInterface $orderWorkflow
    ) {}
    
    #[Route('/orders', name: 'order_create', methods: ['POST'])]
    public function create(Request $request): JsonResponse
    {
        $order = new Order();
        // ... 设置订单数据
        
        // 支付订单
        $this->orderWorkflow->apply($order, 'pay');
        
        // 发送邮件
        $this->bus->dispatch(new SendEmailMessage(
            $order->getUserEmail(),
            'Order Paid',
            'Your order has been paid'
        ));
        
        return $this->json($order, 201);
    }
}
```

## 最佳实践

1. **Messenger**：异步处理耗时操作
2. **Workflow**：管理复杂业务流程
3. **API Platform**：快速构建 REST API
4. **组件复用**：利用 Symfony 组件

## 注意事项

1. Messenger 需要配置消息传输
2. Workflow 需要定义状态和转换
3. API Platform 需要 Doctrine
4. 注意组件版本兼容性

## 练习

1. 使用 Messenger 实现异步任务处理。

2. 创建 Workflow，管理订单状态。

3. 使用 API Platform 快速构建 REST API。

4. 集成多个组件，构建完整应用。
