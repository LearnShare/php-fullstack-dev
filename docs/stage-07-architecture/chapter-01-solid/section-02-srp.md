# 7.1.2 单一职责原则（SRP）

## 概述

单一职责原则（Single Responsibility Principle，简称 SRP）是 SOLID 原则中的第一个原则，也是最基础的原则之一。该原则的核心思想是：一个类应该只有一个改变的理由。换句话说，每个类应该只负责一项功能或职责。

在软件开发中，变化是永恒的主题。业务需求会变化，技术环境会变化，用户需求会变化。当变化发生时，我们希望变化的范围尽可能小，最好只影响一个类。如果一个类承担了多项职责，那么任何一项职责的变化都可能导致这个类需要修改，这就增加了代码维护的难度和引入 bug 的风险。

理解并应用单一职责原则是编写高质量代码的第一步。本节将详细介绍单一职责原则的概念、职责的识别方法、职责分离的策略，以及如何在实际开发中应用这一原则。

**主要内容**：
- 单一职责原则的定义和重要性
- 职责的识别方法
- 违反 SRP 的常见情况
- 职责分离的策略和技巧
- 代码重构示例
- SRP 与其他设计原则的关系

## 特性

单一职责原则具有以下核心特性：

- **职责明确性**：每个类有清晰定义的职责，职责边界明确
- **变化隔离性**：某一方面的变化不会影响其他方面的功能
- **可测试性**：由于职责单一，类更容易进行单元测试
- **可维护性**：修改一个职责的代码不会影响其他职责
- **高内聚性**：类内部的元素紧密相关，共同完成单一职责

## 核心概念

### 什么是职责

在单一职责原则中，"职责"指的是"改变的理由"。当你因为某个原因需要修改一个类时，这个原因就是一个职责。这个定义的关键在于理解"改变的理由"——即推动代码变化的力量。

例如，考虑一个处理用户订单的类：
- 如果业务规则变化（如折扣计算方式改变），需要修改这个类
- 如果数据库表结构变化（如添加新字段），需要修改这个类
- 如果邮件通知逻辑变化（如更改邮件模板），需要修改这个类

这个类就有三个改变的理由，违反了单一职责原则。

### 职责与功能的关系

理解职责与功能的区别很重要。一个类可以有多个功能，但每个功能应该对应一个职责。功能是类做什么，职责是类为什么会变化。

在设计类时，应该问自己："这个类会因为哪些原因而改变？"如果答案是多个不同的原因，那么这个类可能承担了多项职责。

### 职责的粒度

职责的粒度是指职责的粗细程度。过于粗粒度的职责会导致类承担过多功能，过于细粒度则会导致类的数量过多，增加系统复杂性。

确定合适的职责粒度需要考虑以下因素：
- 业务需求的稳定性：变化频繁的职责应该独立出来
- 团队规模：小团队可能更适合较粗粒度的设计
- 项目阶段：原型阶段可以适当放宽，长远项目应更精细

## 职责识别方法

### 识别变化点

识别职责的一种方法是找出代码中所有可能变化的地方。对于每个可能的变化点，考虑它是否应该是独立类。

以下信号表明可能存在多个职责：
- 类的实例变量中有两组或更多组不相关的变量
- 类的方法处理不相关的任务
- 类在不同的场景下有不同的行为
- 修改一个方法会影响其他不相关的方法

### 使用业务维度分析

从业务角度分析类应该承担的职责：
- 这个类代表业务中的什么概念？
- 这个类应该为系统提供什么行为？
- 这个类的行为会因为什么业务原因而改变？

### 重构检查清单

当发现一个类可能违反 SRP 时，可以尝试以下检查：
- 能否用一句话描述这个类的职责？
- 如果要修改这个类，是因为几件事？
- 类中的方法是否都服务于同一个目标？

## 违反 SRP 的示例

### 示例 1：多功能用户类

以下是一个违反单一职责原则的典型示例：

```php
<?php
declare(strict_types=1);

// 违反 SRP：User 类承担了多项职责
class User
{
    private int $id;
    private string $name;
    private string $email;
    
    // 职责1：数据存储
    public function save(): void
    {
        // 保存到数据库
        $pdo = new PDO('mysql:host=localhost;dbname=test', 'root', '');
        $stmt = $pdo->prepare("INSERT INTO users (name, email) VALUES (?, ?)");
        $stmt->execute([$this->name, $this->email]);
    }
    
    // 职责2：数据验证
    public function validate(): bool
    {
        if (empty($this->name)) {
            return false;
        }
        if (!filter_var($this->email, FILTER_VALIDATE_EMAIL)) {
            return false;
        }
        return true;
    }
    
    // 职责3：发送邮件
    public function sendWelcomeEmail(): void
    {
        // 发送欢迎邮件
        $subject = "Welcome to our platform!";
        $body = "Hello {$this->name}, welcome!";
        mail($this->email, $subject, $body);
    }
    
    // 职责4：生成报告
    public function generateProfileReport(): string
    {
        return "User Report: {$this->name} ({$this->email})";
    }
}
```

**问题分析**：
- `save()` 方法负责数据持久化
- `validate()` 方法负责数据验证
- `sendWelcomeEmail()` 方法负责邮件通知
- `generateProfileReport()` 方法负责报告生成

这个类会因为四个不同的原因而改变：
1. 数据库结构变化
2. 验证规则变化
3. 邮件发送逻辑变化
4. 报告格式变化

### 示例 2：违反 SRP 的订单处理

```php
<?php
declare(strict_types=1);

// 违反 SRP：订单类包含太多职责
class Order
{
    private array $items;
    private float $total;
    private string $status;
    
    public function __construct()
    {
        $this->items = [];
        $this->total = 0;
        $this->status = 'pending';
    }
    
    // 职责1：添加商品
    public function addItem(string $productId, int $quantity, float $price): void
    {
        $this->items[] = [
            'product_id' => $productId,
            'quantity' => $quantity,
            'price' => $price
        ];
        $this->calculateTotal();
    }
    
    // 职责2：计算总价
    private function calculateTotal(): void
    {
        $this->total = array_reduce(
            $this->items,
            fn($sum, $item) => $sum + ($item['quantity'] * $item['price']),
            0
        );
    }
    
    // 职责3：处理支付
    public function processPayment(string $paymentMethod): bool
    {
        // 直接处理支付逻辑
        if ($paymentMethod === 'credit_card') {
            // 处理信用卡支付
            return true;
        } elseif ($paymentMethod === 'paypal') {
            // 处理 PayPal 支付
            return true;
        }
        return false;
    }
    
    // 职责4：发送通知
    public function sendNotification(): void
    {
        // 发送订单确认邮件
        echo "Order notification sent\n";
    }
    
    // 职责5：记录日志
    public function logOrder(): void
    {
        // 记录订单日志
        file_put_contents(
            'order.log',
            json_encode($this->items) . "\n",
            FILE_APPEND
        );
    }
    
    // 职责6：导出数据
    public function exportToPdf(): string
    {
        // 生成 PDF
        return "PDF content";
    }
}
```

这个类的问题更加明显，它承担了六项不同的职责，任何一项的变化都需要修改这个类。

## 符合 SRP 的重构示例

### 重构示例 1：职责分离

将违反 SRP 的 User 类重构为多个单一职责的类：

```php
<?php
declare(strict_types=1);

// 单一职责：只负责用户数据
class User
{
    private ?int $id;
    private string $name;
    private string $email;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }

    public function getId(): ?int
    {
        return $this->id;
    }

    public function setId(int $id): void
    {
        $this->id = $id;
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function getEmail(): string
    {
        return $this->email;
    }
}

// 单一职责：只负责数据验证
class UserValidator
{
    public function validate(User $user): bool
    {
        if (empty($user->getName())) {
            return false;
        }
        if (!filter_var($user->getEmail(), FILTER_VALIDATE_EMAIL)) {
            return false;
        }
        return true;
    }

    public function getValidationErrors(User $user): array
    {
        $errors = [];
        
        if (empty($user->getName())) {
            $errors[] = "Name is required";
        }
        
        if (empty($user->getEmail())) {
            $errors[] = "Email is required";
        } elseif (!filter_var($user->getEmail(), FILTER_VALIDATE_EMAIL)) {
            $errors[] = "Invalid email format";
        }
        
        return $errors;
    }
}

// 单一职责：只负责数据持久化
class UserRepository
{
    private PDO $pdo;

    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }

    public function save(User $user): void
    {
        $stmt = $this->pdo->prepare(
            "INSERT INTO users (name, email) VALUES (?, ?)"
        );
        $stmt->execute([
            $user->getName(),
            $user->getEmail()
        ]);
        $user->setId((int)$this->pdo->lastInsertId());
    }

    public function findById(int $id): ?User
    {
        $stmt = $this->pdo->prepare("SELECT * FROM users WHERE id = ?");
        $stmt->execute([$id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);

        if ($row === false) {
            return null;
        }

        $user = new User($row['name'], $row['email']);
        $user->setId((int)$row['id']);
        return $user;
    }
}

// 单一职责：只负责邮件发送
class UserNotifier
{
    private Mailer $mailer;

    public function __construct(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    public function sendWelcomeEmail(User $user): void
    {
        $subject = "Welcome to our platform!";
        $body = "Hello {$user->getName()}, welcome!";
        $this->mail->send($user->getEmail(), $subject, $body);
    }
}

// 单一职责：只负责报告生成
class UserReportGenerator
{
    public function generateProfileReport(User $user): string
    {
        return sprintf(
            "User Report\nName: %s\nEmail: %s\nRegistered: %s",
            $user->getName(),
            $user->getEmail(),
            date('Y-m-d H:i:s')
        );
    }
}

// 协调各职责的服务类
class UserService
{
    private UserRepository $repository;
    private UserValidator $validator;
    private UserNotifier $notifier;
    private UserReportGenerator $reportGenerator;

    public function __construct(
        UserRepository $repository,
        UserValidator $validator,
        UserNotifier $notifier,
        UserReportGenerator $reportGenerator
    ) {
        $this->repository = $repository;
        $this->validator = $validator;
        $this->notifier = $notifier;
        $this->reportGenerator = $reportGenerator;
    }

    public function register(string $name, string $email): User
    {
        $user = new User($name, $email);
        
        if (!$this->validator->validate($user)) {
            throw new InvalidArgumentException("Invalid user data");
        }
        
        $this->repository->save($user);
        $this->notifier->sendWelcomeEmail($user);
        
        return $user;
    }

    public function getProfileReport(int $userId): string
    {
        $user = $this->repository->findById($userId);
        
        if ($user === null) {
            throw new RuntimeException("User not found");
        }
        
        return $this->reportGenerator->generateProfileReport($user);
    }
}

// 简单的邮件类
class Mailer
{
    public function send(string $to, string $subject, string $body): void
    {
        mail($to, $subject, $body);
    }
}
```

**重构后的改进**：
- 每个类只有一个改变的理由
- 易于测试：每个类可以独立进行单元测试
- 易于维护：修改验证规则只需修改 `UserValidator`
- 易于扩展：添加新的验证规则只需扩展 `UserValidator`

### 重构示例 2：订单处理职责分离

```php
<?php
declare(strict_types=1);

// 订单实体：只负责数据
class Order
{
    private ?int $id;
    private array $items;
    private float $total;
    private string $status;

    public function __construct()
    {
        $this->items = [];
        $this->total = 0;
        $this->status = 'pending';
    }

    public function addItem(string $productId, int $quantity, float $price): void
    {
        $this->items[] = [
            'product_id' => $productId,
            'quantity' => $quantity,
            'price' => $price
        ];
        $this->recalculateTotal();
    }

    private function recalculateTotal(): void
    {
        $this->total = array_reduce(
            $this->items,
            fn($sum, $item) => $sum + ($item['quantity'] * $item['price']),
            0
        );
    }

    public function getTotal(): float
    {
        return $this->total;
    }

    public function getStatus(): string
    {
        return $this->status;
    }

    public function setStatus(string $status): void
    {
        $this->status = $status;
    }
}

// 支付处理：单一职责
interface PaymentGateway
{
    public function process(float $amount): bool;
}

class CreditCardPayment implements PaymentGateway
{
    public function process(float $amount): bool
    {
        // 信用卡支付处理逻辑
        echo "Processing credit card payment: \${$amount}\n";
        return true;
    }
}

class PayPalPayment implements PaymentGateway
{
    public function process(float $amount): bool
    {
        // PayPal 支付处理逻辑
        echo "Processing PayPal payment: \${$amount}\n";
        return true;
    }
}

// 通知服务：单一职责
class OrderNotifier
{
    public function sendConfirmation(Order $order): void
    {
        echo "Order confirmation sent for order #{$order->getId()}\n";
    }
}

// 日志记录：单一职责
class OrderLogger
{
    public function logOrder(Order $order): void
    {
        $logData = [
            'id' => $order->getId(),
            'total' => $order->getTotal(),
            'status' => $order->getStatus(),
            'timestamp' => date('Y-m-d H:i:s')
        ];
        file_put_contents(
            'order.log',
            json_encode($logData) . "\n",
            FILE_APPEND
        );
    }
}

// 订单导出：单一职责
interface OrderExporter
{
    public function export(Order $order): string;
}

class PdfOrderExporter implements OrderExporter
{
    public function export(Order $order): string
    {
        return "PDF export for order #{$order->getId()}";
    }
}

// 订单服务：协调各职责
class OrderService
{
    private PaymentGateway $paymentGateway;
    private OrderNotifier $notifier;
    private OrderLogger $logger;
    private OrderExporter $exporter;

    public function __construct(
        PaymentGateway $paymentGateway,
        OrderNotifier $notifier,
        OrderLogger $logger,
        OrderExporter $exporter
    ) {
        $this->paymentGateway = $paymentGateway;
        $this->notifier = $notifier;
        $this->logger = $logger;
        $this->exporter = $exporter;
    }

    public function processOrder(Order $order): bool
    {
        // 处理支付
        $success = $this->paymentGateway->process($order->getTotal());
        
        if ($success) {
            $order->setStatus('completed');
            $this->notifier->sendConfirmation($order);
            $this->logger->logOrder($order);
        }
        
        return $success;
    }

    public function exportOrder(Order $order): string
    {
        return $this->exporter->export($order);
    }
}
```

## 基本用法

### 在实际项目中的应用

以下示例展示如何在实际 PHP 项目中应用单一职责原则：

```php
<?php
declare(strict_types=1);

// 数据传输对象：纯数据容器
class CreateUserRequest
{
    public function __construct(
        public readonly string $name,
        public readonly string $email,
        public readonly ?string $phone = null
    ) {}
}

// 验证器：单一职责
class CreateUserRequestValidator
{
    public function validate(CreateUserRequest $request): ValidationResult
    {
        $errors = [];
        
        if (strlen($request->name) < 2) {
            $errors['name'] = 'Name must be at least 2 characters';
        }
        
        if (!filter_var($request->email, FILTER_VALIDATE_EMAIL)) {
            $errors['email'] = 'Invalid email format';
        }
        
        return new ValidationResult(empty($errors), $errors);
    }
}

// 验证结果
class ValidationResult
{
    public function __construct(
        public readonly bool $isValid,
        public readonly array $errors
    ) {}
}

// 仓储：单一职责 - 数据持久化
class UserRepository
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function create(CreateUserRequest $request): User
    {
        $stmt = $this->pdo->prepare(
            "INSERT INTO users (name, email, phone) VALUES (?, ?, ?)"
        );
        $stmt->execute([
            $request->name,
            $request->email,
            $request->phone
        ]);
        
        return new User(
            (int)$this->pdo->lastInsertId(),
            $request->name,
            $request->email,
            $request->phone
        );
    }
}

// 用户实体
class User
{
    public function __construct(
        public readonly int $id,
        public readonly string $name,
        public readonly string $email,
        public readonly ?string $phone
    ) {}
}

// 邮件服务：单一职责
class MailService
{
    public function sendWelcome(User $user): void
    {
        echo "Sending welcome email to {$user->email}\n";
    }
}

// 事件：单一职责 - 解耦
class UserCreatedEvent
{
    public function __construct(public readonly User $user) {}
}

// 事件处理器
class UserCreatedEventHandler
{
    private MailService $mailService;
    
    public function __construct(MailService $mailService)
    {
        $this->mailService = $mailService;
    }
    
    public function handle(UserCreatedEvent $event): void
    {
        $this->mailService->sendWelcome($event->user);
    }
}

// 主服务：协调各组件
class CreateUserService
{
    private CreateUserRequestValidator $validator;
    private UserRepository $repository;
    private UserCreatedEventHandler $eventHandler;
    
    public function __construct(
        CreateUserRequestValidator $validator,
        UserRepository $repository,
        UserCreatedEventHandler $eventHandler
    ) {
        $this->validator = $validator;
        $this->repository = $repository;
        $this->eventHandler = $eventHandler;
    }
    
    public function execute(CreateUserRequest $request): User
    {
        // 验证
        $result = $this->validator->validate($request);
        if (!$result->isValid) {
            throw new InvalidArgumentException(
                json_encode($result->errors)
            );
        }
        
        // 创建用户
        $user = $this->repository->create($request);
        
        // 触发事件
        $this->eventHandler->handle(new UserCreatedEvent($user));
        
        return $user;
    }
}

// 使用示例
$pdo = new PDO('mysql:host=localhost;dbname=test', 'root', '');
$validator = new CreateUserRequestValidator();
$repository = new UserRepository($pdo);
$mailService = new MailService();
$eventHandler = new UserCreatedEventHandler($mailService);

$service = new CreateUserService(
    $validator,
    $repository,
    $eventHandler
);

$request = new CreateUserRequest('John Doe', 'john@example.com');
$user = $service->execute($request);

echo "User created with ID: {$user->id}\n";
```

## 使用场景

单一职责原则适用于以下场景：

- **复杂业务逻辑**：业务逻辑复杂的系统更需要职责分离
- **大型代码库**：代码量越大，职责分离的收益越高
- **需要频繁修改的系统**：职责分离使修改的影响范围最小化
- **需要测试的代码**：单一职责的类更容易进行单元测试
- **团队协作项目**：明确的职责边界便于团队分工

## 注意事项

### 避免过度拆分

单一职责原则并不意味着类越小越好。过度拆分会导致：
- 类的数量急剧增加
- 系统复杂性增加
- 理解和导航代码变得困难
- 可能会引入更多的依赖关系

应该在保持类职责单一的前提下，考虑类的实际使用场景和团队的理解能力。

### 职责边界的主观性

职责的定义有时是主观的。不同的开发者可能对"职责"有不同的理解。例如，"用户管理"可以是一个职责，也可以细分为"用户数据管理"和"用户认证"。

关键是要有一个清晰的标准：变化的原因。如果一个类会因为多个不同的原因而改变，那么它就可能违反了 SRP。

### 权衡与实际

在实际项目中，需要权衡各种因素：
- 项目时间和资源
- 团队的技术水平
- 项目的预期生命周期
- 需求的稳定性

对于小型项目或一次性脚本，严格遵循 SRP 可能是过度设计。

## 常见问题

### 如何判断一个类是否违反 SRP？

问自己一个问题："这个类会因为几个不同的原因而改变？"如果答案是两个或更多，可能就违反了 SRP。

### 职责应该如何划分？

职责的划分应该基于业务需求和技术考量。考虑：
- 变化的频率：经常变化的功能应该独立
- 变化的来源：来自不同业务方的需求应该分离
- 依赖关系：相互依赖的功能应该在一起

### 遵循 SRP 后类太多怎么办？

这可能是没有正确识别职责边界。可以：
- 重新审视类的职责定义
- 考虑是否可以将相关类组合成模块
- 使用命名空间组织类

### SRP 与其他原则冲突怎么办？

原则之间有时会冲突。例如，过度追求单一职责可能导致过多依赖。需要在实际场景中权衡，选择最适合当前情况的方案。

## 最佳实践

1. **从一开始就设计良好的结构**：在项目开始时花时间思考类的职责，可以避免未来的大规模重构

2. **使用有意义的类名**：类的名称应该清晰表达其职责。如果类名包含"和"、"或"等词，可能意味着承担多项职责

3. **保持方法与类职责一致**：类的方法应该服务于类的单一职责。如果一个方法与类的核心职责无关，考虑移到其他类

4. **定期重构**：代码会随着时间腐化。定期审查和重构可以保持代码结构良好

5. **编写单元测试**：单一职责的类更容易编写测试。如果测试一个类很困难，可能说明类的职责过多

## 练习任务

1. **分析现有代码**：选择一个你过去的 PHP 项目，找出一个违反 SRP 的类，分析它承担了哪些职责，尝试进行重构

2. **设计一个博客系统**：使用 SRP 原则设计博客系统，包括文章、评论、用户等模块，画出类图并实现核心代码

3. **重构订单处理**：假设有一个处理订单的类，包含验证、存储、支付、通知等功能，按照 SRP 进行职责分离

4. **测试驱动重构**：选择一个违反 SRP 的类，先为其编写测试，然后进行重构，验证重构后功能仍然正常

5. **思考职责边界**：分析一个电商购物车类，列出它可能承担的职责，讨论如何进行职责分离
