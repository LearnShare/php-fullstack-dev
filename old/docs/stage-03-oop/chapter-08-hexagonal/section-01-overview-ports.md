# 3.8.1 六边形架构概述与 Ports

## 概述

六边形架构（Hexagonal Architecture / Ports & Adapters）将应用分为核心（Core）、端口（Ports）和适配器（Adapters），实现业务逻辑与技术实现的解耦。

## 六边形架构概述

### 核心思想

六边形架构（也称为 Ports & Adapters）将应用分为：

- **核心（Core）**：包含业务逻辑，不依赖外部技术。
- **端口（Ports）**：定义接口，描述应用需要什么功能。
- **适配器（Adapters）**：实现端口，连接外部世界。

```
         ┌─────────────┐
         │   Adapters  │
         │  (外部世界)  │
         └──────┬──────┘
                │
         ┌──────▼──────┐
         │   Ports     │
         │  (接口定义)  │
         └──────┬──────┘
                │
         ┌──────▼──────┐
         │    Core     │
         │  (业务逻辑)  │
         └─────────────┘
```

### 为什么需要六边形架构

- **技术无关性**：业务逻辑不依赖数据库、Web 框架等具体技术。
- **可测试性**：可以轻松替换适配器进行测试。
- **可扩展性**：添加新的适配器不影响核心逻辑。

## Ports（端口）

### 输入端口（Inbound Ports）

- 定义应用如何被外部调用。
- 通常表现为接口或抽象类。

```php
namespace App\Domain\Ports\Inbound;

use App\Domain\User;

interface UserServicePort
{
    public function createUser(string $name, string $email): User;
    public function getUserById(int $id): User;
    public function updateUser(int $id, string $name): void;
}
```

### 输出端口（Outbound Ports）

- 定义应用需要什么外部服务。
- 描述应用对外部世界的需求。

```php
namespace App\Domain\Ports\Outbound;

use App\Domain\User;

interface UserRepositoryPort
{
    public function save(User $user): void;
    public function findById(int $id): ?User;
    public function findByEmail(string $email): ?User;
}

interface EmailServicePort
{
    public function send(string $to, string $subject, string $body): void;
}

interface NotificationPort
{
    public function notify(string $message): void;
}
```

## 核心（Core）实现

### 实现输入端口

```php
namespace App\Domain;

use App\Domain\Ports\Inbound\UserServicePort;
use App\Domain\Ports\Outbound\{UserRepositoryPort, EmailServicePort};

class UserService implements UserServicePort
{
    public function __construct(
        private UserRepositoryPort $userRepository,
        private EmailServicePort $emailService
    ) {
    }

    public function createUser(string $name, string $email): User
    {
        if ($this->userRepository->findByEmail($email) !== null) {
            throw new EmailAlreadyExistsException($email);
        }

        $user = new User(null, $name, $email);
        $this->userRepository->save($user);
        
        $this->emailService->send(
            $email,
            'Welcome',
            "Welcome, {$name}!"
        );

        return $user;
    }

    public function getUserById(int $id): User
    {
        $user = $this->userRepository->findById($id);
        if ($user === null) {
            throw new UserNotFoundException($id);
        }
        return $user;
    }

    public function updateUser(int $id, string $name): void
    {
        $user = $this->getUserById($id);
        $user->changeName($name);
        $this->userRepository->save($user);
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

// Ports
namespace App\Domain\Ports\Inbound;

interface OrderServicePort
{
    public function createOrder(array $items): Order;
    public function getOrder(int $id): Order;
}

namespace App\Domain\Ports\Outbound;

interface OrderRepositoryPort
{
    public function save(Order $order): void;
    public function findById(int $id): ?Order;
}

interface PaymentGatewayPort
{
    public function charge(float $amount): bool;
}

// Core
namespace App\Domain;

use App\Domain\Ports\Inbound\OrderServicePort;
use App\Domain\Ports\Outbound\{OrderRepositoryPort, PaymentGatewayPort};

class OrderService implements OrderServicePort
{
    public function __construct(
        private OrderRepositoryPort $orderRepository,
        private PaymentGatewayPort $paymentGateway
    ) {
    }

    public function createOrder(array $items): Order
    {
        $order = new Order($items);
        $total = $order->calculateTotal();

        if (!$this->paymentGateway->charge($total)) {
            throw new PaymentFailedException();
        }

        $this->orderRepository->save($order);
        return $order;
    }

    public function getOrder(int $id): Order
    {
        $order = $this->orderRepository->findById($id);
        if ($order === null) {
            throw new OrderNotFoundException($id);
        }
        return $order;
    }
}
```

## 注意事项

1. **端口定义**：端口应该定义在领域层，描述业务需求。

2. **依赖方向**：核心依赖端口（接口），不依赖适配器（实现）。

3. **技术无关**：核心代码不应该包含任何技术相关的代码。

4. **接口设计**：端口接口应该反映业务需求，而不是技术实现。

5. **测试友好**：使用端口可以轻松创建测试用的适配器。

## 练习

1. 定义一个用户服务的输入端口和输出端口。

2. 实现一个订单服务的核心逻辑，使用端口定义依赖。

3. 创建一个支付服务的端口定义，包含必要的业务方法。

4. 设计一个通知系统的端口，支持多种通知方式。

5. 实现一个完整的六边形架构核心，包含多个端口定义。
