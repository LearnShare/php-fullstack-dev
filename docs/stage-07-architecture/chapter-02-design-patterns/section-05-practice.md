# 7.2.5 设计模式实践

## 概述

设计模式是解决软件设计问题的强大工具，但如何正确选择和应用这些模式是每个开发者都需要掌握的技能。本节将通过一个完整的案例，展示如何在实际项目中识别问题、选择合适的模式、组合使用多个模式，以及如何通过重构逐步应用设计模式。

通过本节的学习，你将掌握：
- 如何识别设计问题并选择合适的模式
- 如何组合使用多个设计模式
- 如何通过重构逐步应用设计模式
- 如何评估模式应用的效果

**主要内容**：
- 设计模式选择原则
- 模式组合使用技巧
- 重构实践方法
- 完整案例分析

## 案例：订单处理系统重构

### 原始代码问题分析

让我们先看一个典型的"一锅端"订单处理类：

```php
<?php
declare(strict_types=1);

// 原始订单处理类（违反多种设计原则）
class OrderProcessor
{
    private PDO $pdo;
    private array $payments = [];
    
    public function __construct()
    {
        $this->pdo = new PDO('mysql:host=localhost;dbname=shop', 'root', '');
    }
    
    public function createOrder(array $data): int
    {
        // 直接验证
        if (empty($data['customer_id'])) {
            throw new InvalidArgumentException('Customer ID required');
        }
        
        // 直接创建 SQL
        $stmt = $this->pdo->prepare(
            "INSERT INTO orders (customer_id, total, status) VALUES (?, ?, ?)"
        );
        $stmt->execute([
            $data['customer_id'],
            $data['total'],
            'pending'
        ]);
        
        return (int)$this->pdo->lastInsertId();
    }
    
    public function processPayment(int $orderId, string $type, float $amount): bool
    {
        // 直接处理不同支付方式
        if ($type === 'credit_card') {
            // 处理信用卡
            $this->payments[] = ['type' => 'credit_card', 'amount' => $amount];
            return true;
        } elseif ($type === 'paypal') {
            // 处理 PayPal
            $this->payments[] = ['type' => 'paypal', 'amount' => $amount];
            return true;
        }
        
        return false;
    }
    
    public function sendNotification(int $orderId): void
    {
        // 直接发送邮件
        mail('customer@example.com', 'Order Created', 'Your order has been created');
    }
}
```

**问题分析**：

1. **违反单一职责**：订单处理、支付处理、通知发送都在一个类中
2. **违反开闭原则**：添加新支付方式需要修改代码
3. **违反依赖倒置**：直接依赖 PDO、具体支付处理
4. **难以测试**：无法 mock 依赖

### 重构方案设计

#### 1. 使用工厂模式创建支付处理器

```php
<?php
declare(strict_types=1);

// 支付策略接口
interface PaymentProcessor
{
    public function process(float $amount): bool;
    public function getType(): string;
}

// 信用卡支付
class CreditCardProcessor implements PaymentProcessor
{
    private string $cardNumber;
    
    public function __construct(string $cardNumber)
    {
        $this->cardNumber = $cardNumber;
    }
    
    public function process(float $amount): bool
    {
        echo "Processing credit card: {$this->cardNumber}\n";
        return true;
    }
    
    public function getType(): string
    {
        return 'credit_card';
    }
}

// PayPal 支付
class PayPalProcessor implements PaymentProcessor
{
    private string $email;
    
    public function __construct(string $email)
    {
        $this->email = $email;
    }
    
    public function process(float $amount): bool
    {
        echo "Processing PayPal: {$this->email}\n";
        return true;
    }
    
    public function getType(): string
    {
        return 'paypal';
    }
}

// 工厂模式创建支付处理器
class PaymentProcessorFactory
{
    public static function create(string $type, array $options = []): PaymentProcessor
    {
        return match($type) {
            'credit_card' => new CreditCardProcessor($options['card_number'] ?? ''),
            'paypal' => new PayPalProcessor($options['email'] ?? ''),
            default => throw new InvalidArgumentException("Unknown payment type: {$type}"),
        };
    }
}
```

#### 2. 使用观察者模式实现事件通知

```php
<?php
declare(strict_types=1);

// 事件系统
interface OrderObserver
{
    public function onOrderCreated(Order $order): void;
    public function onOrderPaid(Order $order): void;
}

// 邮件通知观察者
class EmailNotificationObserver implements OrderObserver
{
    public function onOrderCreated(Order $order): void
    {
        echo "Sending order confirmation email to customer\n";
    }
    
    public function onOrderPaid(Order $order): void
    {
        echo "Sending payment confirmation email to customer\n";
    }
}

// 日志观察者
class LoggingObserver implements OrderObserver
{
    public function onOrderCreated(Order $order): void
    {
        echo "Logging: Order #{$order->getId()} created\n";
    }
    
    public function onOrderPaid(Order $order): void
    {
        echo "Logging: Order #{$order->getId()} paid\n";
    }
}

// 订单主题
class Order
{
    private ?int $id;
    private int $customerId;
    private float $total;
    private string $status;
    private array $observers = [];
    
    public function __construct(int $customerId, float $total)
    {
        $this->customerId = $customerId;
        $this->total = $total;
        $this->status = 'pending';
    }
    
    public function attach(OrderObserver $observer): void
    {
        $this->observers[] = $observer;
    }
    
    public function setId(int $id): void
    {
        $this->id = $id;
    }
    
    public function getId(): ?int
    {
        return $this->id;
    }
    
    public function setStatus(string $status): void
    {
        $this->status = $status;
    }
    
    public function notify(string $event): void
    {
        foreach ($this->observers as $observer) {
            match($event) {
                'created' => $observer->onOrderCreated($this),
                'paid' => $observer->onOrderPaid($this),
                default => null,
            };
        }
    }
}
```

#### 3. 使用仓储模式管理数据

```php
<?php
declare(strict_types=1);

// 仓储接口
interface OrderRepository
{
    public function save(Order $order): int;
    public function findById(int $id): ?Order;
    public function update(Order $order): void;
}

// MySQL 仓储实现
class MySQLOrderRepository implements OrderRepository
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function save(Order $order): int
    {
        $stmt = $this->pdo->prepare(
            "INSERT INTO orders (customer_id, total, status) VALUES (?, ?, ?)"
        );
        $stmt->execute([
            $order->getCustomerId(),
            $order->getTotal(),
            $order->getStatus()
        ]);
        
        return (int)$this->pdo->lastInsertId();
    }
    
    public function findById(int $id): ?Order
    {
        $stmt = $this->pdo->prepare("SELECT * FROM orders WHERE id = ?");
        $stmt->execute([$id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);
        
        if ($row === false) {
            return null;
        }
        
        $order = new Order((int)$row['customer_id'], (float)$row['total']);
        $order->setId((int)$row['id']);
        $order->setStatus($row['status']);
        
        return $order;
    }
    
    public function update(Order $order): void
    {
        $stmt = $this->pdo->prepare(
            "UPDATE orders SET status = ? WHERE id = ?"
        );
        $stmt->execute([$order->getStatus(), $order->getId()]);
    }
}
```

#### 4. 组合使用所有模式

```php
<?php
declare(strict_types=1);

// 重构后的订单服务
class OrderService
{
    private OrderRepository $repository;
    private array $observers = [];
    
    public function __construct(OrderRepository $repository)
    {
        $this->repository = $repository;
    }
    
    public function attach(OrderObserver $observer): void
    {
        $this->observers[] = $observer;
    }
    
    public function createOrder(int $customerId, float $total): Order
    {
        $order = new Order($customerId, $total);
        
        // 保存订单
        $orderId = $this->repository->save($order);
        $order->setId($orderId);
        
        // 通知观察者
        foreach ($this->observers as $observer) {
            $observer->onOrderCreated($order);
        }
        
        return $order;
    }
    
    public function processPayment(int $orderId, string $paymentType, array $options): bool
    {
        // 查找订单
        $order = $this->repository->findById($orderId);
        
        if ($order === null) {
            throw new RuntimeException('Order not found');
        }
        
        // 使用工厂创建支付处理器（策略模式）
        $paymentProcessor = PaymentProcessorFactory::create(
            $paymentType,
            $options
        );
        
        // 处理支付
        $success = $paymentProcessor->process($order->getTotal());
        
        if ($success) {
            $order->setStatus('paid');
            $this->repository->update($order);
            
            // 通知观察者
            foreach ($this->observers as $observer) {
                $observer->onOrderPaid($order);
            }
        }
        
        return $success;
    }
}

// 使用示例
$pdo = new PDO('mysql:host=localhost;dbname=shop', 'root', '');
$repository = new MySQLOrderRepository($pdo);

$service = new OrderService($repository);

// 添加观察者
$service->attach(new EmailNotificationObserver());
$service->attach(new LoggingObserver());

// 创建订单（触发观察者）
$order = $service->createOrder(1, 99.99);
echo "Order created: #{$order->getId()}\n";

// 处理支付
$service->processPayment($order->getId(), 'credit_card', [
    'card_number' => '4111111111111111'
]);
```

## 模式选择指南

### 根据问题类型选择

| 问题类型 | 推荐模式 |
|:---------|:---------|
| 对象创建复杂 | 工厂模式、建造者模式 |
| 对象结构复杂 | 适配器模式、装饰器模式 |
| 对象通信复杂 | 观察者模式、中介者模式 |
| 行为变化多 | 策略模式、状态模式 |
| 算法变化多 | 策略模式、模板方法模式 |

### 组合模式使用场景

1. **工厂 + 策略**：工厂创建策略对象
2. **观察者 + 命令**：命令可以作为观察者的回调
3. **装饰器 + 策略**：装饰器包装策略
4. **责任链 + 命令**：责任链处理命令对象

## 重构步骤

### 1. 识别问题

- 代码太大，承担多项职责
- 修改一处影响多处
- 难以测试
- 添加新功能困难

### 2. 分离关注点

- 识别不同的职责
- 定义清晰的接口
- 提取独立类

### 3. 应用模式

- 选择合适的模式
- 小步重构
- 频繁测试

### 4. 验证效果

- 功能测试
- 性能测试
- 代码审查

## 最佳实践

1. **不要过度设计**：只在真正需要时使用模式
2. **保持简单**：优先使用简单的解决方案
3. **渐进式重构**：一步一步改进
4. **测试驱动**：先写测试再重构
5. **持续改进**：代码需要持续维护

## 练习任务

1. **分析现有代码**：选择一个违反设计原则的类，分析问题

2. **模式选择练习**：为一个具体问题选择最合适的模式

3. **组合模式**：设计一个结合工厂模式和观察者模式的系统

4. **重构实践**：将一个单体类重构为使用多个设计模式

5. **模式识别**：分析一个开源项目使用的设计模式
