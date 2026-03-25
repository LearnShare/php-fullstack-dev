# 7.1.7 SOLID 原则实践

## 概述

SOLID 原则不是孤立存在的，它们相互关联、相互支持。在实际开发中，我们需要综合应用这些原则，而不是单独考虑每一个原则。本节将通过一个完整的案例，展示如何在实际项目中应用 SOLID 原则，包括从需求分析到代码实现的完整过程。

通过本节的学习，你将掌握：
- 如何综合应用 SOLID 五个原则
- 如何识别代码中违反原则的地方
- 如何进行有计划的重构
- 如何在原则之间进行权衡
- 如何建立团队的设计规范

**主要内容**：
- 原则综合应用的方法
- 识别违反原则的代码
- 重构的步骤和技巧
- 原则之间的权衡
- 完整的实践案例

## 案例：订单处理系统

### 需求概述

假设我们需要开发一个订单处理系统，包含以下功能：
1. 创建订单
2. 处理支付
3. 发送通知
4. 记录日志

### 初步设计（违反 SOLID 原则）

让我们先看一下常见的"一锅端"实现方式：

```php
<?php
declare(strict_types=1);

// 违反 SOLID 原则的订单处理器
class OrderProcessor
{
    private PDO $pdo;
    private string $smtpHost;
    private int $smtpPort;
    
    public function __construct()
    {
        $this->pdo = new PDO('mysql:host=localhost;dbname=shop', 'root', '');
        $this->smtpHost = 'smtp.example.com';
        $this->smtpPort = 587;
    }
    
    public function createOrder(array $orderData): int
    {
        // 验证数据
        if (empty($orderData['customer_id'])) {
            throw new InvalidArgumentException('Customer ID is required');
        }
        if (empty($orderData['items'])) {
            throw new InvalidArgumentException('Order must have items');
        }
        
        // 计算总价
        $total = 0;
        foreach ($orderData['items'] as $item) {
            $total += $item['price'] * $item['quantity'];
        }
        
        // 保存到数据库
        $stmt = $this->pdo->prepare(
            "INSERT INTO orders (customer_id, total, status, created_at) VALUES (?, ?, ?, ?)"
        );
        $stmt->execute([
            $orderData['customer_id'],
            $total,
            'pending',
            date('Y-m-d H:i:s')
        ]);
        
        $orderId = (int)$this->pdo->lastInsertId();
        
        // 保存订单项
        foreach ($orderData['items'] as $item) {
            $stmt = $this->pdo->prepare(
                "INSERT INTO order_items (order_id, product_id, quantity, price) VALUES (?, ?, ?, ?)"
            );
            $stmt->execute([
                $orderId,
                $item['product_id'],
                $item['quantity'],
                $item['price']
            ]);
        }
        
        // 发送邮件通知
        $this->sendEmail($orderData['customer_id'], 'Order Created', 'Your order has been created');
        
        // 记录日志
        file_put_contents(
            'order.log',
            sprintf("[%s] Order created: %d\n", date('Y-m-d H:i:s'), $orderId),
            FILE_APPEND
        );
        
        return $orderId;
    }
    
    public function processPayment(int $orderId, string $paymentMethod): bool
    {
        // 获取订单
        $stmt = $this->pdo->prepare("SELECT * FROM orders WHERE id = ?");
        $stmt->execute([$orderId]);
        $order = $stmt->fetch(PDO::FETCH_ASSOC);
        
        if (!$order) {
            throw new RuntimeException('Order not found');
        }
        
        // 处理不同支付方式
        if ($paymentMethod === 'credit_card') {
            // 信用卡支付逻辑
            $this->processCreditCard($order['total']);
        } elseif ($paymentMethod === 'paypal') {
            // PayPal 支付逻辑
            $this->processPayPal($order['total']);
        } elseif ($paymentMethod === 'bank_transfer') {
            // 银行转账逻辑
            $this->processBankTransfer($order['total']);
        } else {
            throw new InvalidArgumentException('Invalid payment method');
        }
        
        // 更新订单状态
        $stmt = $this->pdo->prepare("UPDATE orders SET status = ? WHERE id = ?");
        $stmt->execute(['paid', $orderId]);
        
        return true;
    }
    
    private function processCreditCard(float $amount): bool
    {
        // 信用卡处理逻辑
        return true;
    }
    
    private function processPayPal(float $amount): bool
    {
        // PayPal 处理逻辑
        return true;
    }
    
    private function processBankTransfer(float $amount): bool
    {
        // 银行转账处理逻辑
        return true;
    }
    
    private function sendEmail(string $to, string $subject, string $body): void
    {
        // 发送邮件逻辑
    }
}
```

**问题分析**：

1. **违反单一职责原则（SRP）**：
   - `OrderProcessor` 负责数据验证、数据库操作、支付处理、邮件发送、日志记录
   - 任何一方面需求变化都需要修改这个类

2. **违反开闭原则（OCP）**：
   - 添加新的支付方式需要修改 `processPayment` 方法
   - 添加新的通知方式需要修改代码

3. **违反依赖倒置原则（DIP）**：
   - 直接依赖具体的 `PDO` 类
   - 直接在类内部创建邮件发送、日志记录的具体实现

4. **违反接口隔离原则（ISP）**：
   - 没有定义清晰的接口
   - 所有功能都集中在一个类中

### 重构设计（遵循 SOLID 原则）

让我们按照 SOLID 原则重构这个系统：

```php
<?php
declare(strict_types=1);

// ============================================
// 1. 定义领域实体
// ============================================

class Order
{
    private ?int $id;
    private int $customerId;
    private array $items;
    private float $total;
    private string $status;
    private DateTime $createdAt;
    
    public function __construct(int $customerId, array $items)
    {
        $this->customerId = $customerId;
        $this->items = $items;
        $this->total = $this->calculateTotal();
        $this->status = 'pending';
        $this->createdAt = new DateTime();
    }
    
    public function calculateTotal(): float
    {
        return array_reduce(
            $this->items,
            fn(float $sum, array $item) => $sum + ($item['price'] * $item['quantity']),
            0
        );
    }
    
    public function getId(): ?int
    {
        return $this->id;
    }
    
    public function setId(int $id): void
    {
        $this->id = $id;
    }
    
    public function getCustomerId(): int
    {
        return $this->customerId;
    }
    
    public function getItems(): array
    {
        return $this->items;
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

// ============================================
// 2. 定义仓储接口（遵循依赖倒置原则）
// ============================================

interface OrderRepositoryInterface
{
    public function save(Order $order): int;
    public function findById(int $id): ?Order;
    public function update(Order $order): void;
}

interface OrderItemRepositoryInterface
{
    public function save(int $orderId, array $items): void;
}

// ============================================
// 3. 定义支付接口（遵循开闭原则和接口隔离原则）
// ============================================

interface PaymentGatewayInterface
{
    public function process(float $amount): bool;
    public function getName(): string;
}

// ============================================
// 4. 定义通知接口（遵循接口隔离原则）
// ============================================

interface NotificationServiceInterface
{
    public function send(string $to, string $subject, string $body): void;
}

// ============================================
// 5. 定义日志接口（遵循依赖倒置原则）
// ============================================

interface LoggerInterface
{
    public function info(string $message): void;
    public function error(string $message): void;
}

// ============================================
// 6. 实现仓储
// ============================================

class MySQLOrderRepository implements OrderRepositoryInterface
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function save(Order $order): int
    {
        $stmt = $this->pdo->prepare(
            "INSERT INTO orders (customer_id, total, status, created_at) VALUES (?, ?, ?, ?)"
        );
        $stmt->execute([
            $order->getCustomerId(),
            $order->getTotal(),
            $order->getStatus(),
            $order->getCreatedAt()->format('Y-m-d H:i:s')
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
        
        // 实际项目中需要加载 items
        return new Order($row['customer_id'], []);
    }
    
    public function update(Order $order): void
    {
        $stmt = $this->pdo->prepare(
            "UPDATE orders SET status = ? WHERE id = ?"
        );
        $stmt->execute([$order->getStatus(), $order->getId()]);
    }
}

// ============================================
// 7. 实现支付网关（遵循开闭原则）
// ============================================

class CreditCardPaymentGateway implements PaymentGatewayInterface
{
    public function process(float $amount): bool
    {
        // 信用卡处理逻辑
        echo "Processing credit card payment: \${$amount}\n";
        return true;
    }
    
    public function getName(): string
    {
        return 'credit_card';
    }
}

class PayPalPaymentGateway implements PaymentGatewayInterface
{
    public function process(float $amount): bool
    {
        // PayPal 处理逻辑
        echo "Processing PayPal payment: \${$amount}\n";
        return true;
    }
    
    public function getName(): string
    {
        return 'paypal';
    }
}

class BankTransferPaymentGateway implements PaymentGatewayInterface
{
    public function process(float $amount): bool
    {
        // 银行转账处理逻辑
        echo "Processing bank transfer: \${$amount}\n";
        return true;
    }
    
    public function getName(): string
    {
        return 'bank_transfer';
    }
}

// 支付网关管理器（可以动态添加新支付方式）
class PaymentGatewayManager
{
    private array $gateways = [];
    
    public function register(PaymentGatewayInterface $gateway): void
    {
        $this->gateways[$gateway->getName()] = $gateway;
    }
    
    public function process(string $gatewayName, float $amount): bool
    {
        if (!isset($this->gateways[$gatewayName])) {
            throw new InvalidArgumentException(
                "Payment gateway not found: {$gatewayName}"
            );
        }
        
        return $this->gateways[$gatewayName]->process($amount);
    }
}

// ============================================
// 8. 实现通知服务
// ============================================

class EmailNotificationService implements NotificationServiceInterface
{
    private string $fromEmail;
    private string $smtpHost;
    private int $smtpPort;
    
    public function __construct(string $fromEmail, string $smtpHost, int $smtpPort)
    {
        $this->fromEmail = $fromEmail;
        $this->smtpHost = $smtpHost;
        $this->smtpPort = $smtpPort;
    }
    
    public function send(string $to, string $subject, string $body): void
    {
        echo "Sending email to: {$to}\n";
        echo "Subject: {$subject}\n";
    }
}

// ============================================
// 9. 实现日志服务
// ============================================

class FileLogger implements LoggerInterface
{
    private string $filename;
    
    public function __construct(string $filename)
    {
        $this->filename = $filename;
    }
    
    public function info(string $message): void
    {
        $this->log('INFO', $message);
    }
    
    public function error(string $message): void
    {
        $this->log('ERROR', $message);
    }
    
    private function log(string $level, string $message): void
    {
        $line = sprintf(
            "[%s] [%s] %s\n",
            date('Y-m-d H:i:s'),
            $level,
            $message
        );
        file_put_contents($this->filename, $line, FILE_APPEND);
    }
}

// ============================================
// 10. 实现订单验证器（单一职责）
// ============================================

class OrderValidator
{
    public function validate(Order $order): ValidationResult
    {
        $errors = [];
        
        if ($order->getCustomerId() <= 0) {
            $errors[] = 'Customer ID is required';
        }
        
        if (empty($order->getItems())) {
            $errors[] = 'Order must have items';
        }
        
        if ($order->getTotal() <= 0) {
            $errors[] = 'Order total must be greater than zero';
        }
        
        return new ValidationResult(empty($errors), $errors);
    }
}

class ValidationResult
{
    public function __construct(
        public readonly bool $isValid,
        public readonly array $errors
    ) {}
}

// ============================================
// 11. 实现订单服务（高层模块，依赖抽象）
// ============================================

class OrderService
{
    private OrderRepositoryInterface $orderRepository;
    private OrderItemRepositoryInterface $orderItemRepository;
    private PaymentGatewayManager $paymentManager;
    private NotificationServiceInterface $notificationService;
    private LoggerInterface $logger;
    private OrderValidator $validator;
    
    public function __construct(
        OrderRepositoryInterface $orderRepository,
        OrderItemRepositoryInterface $orderItemRepository,
        PaymentGatewayManager $paymentManager,
        NotificationServiceInterface $notificationService,
        LoggerInterface $logger
    ) {
        $this->orderRepository = $orderRepository;
        $this->orderItemRepository = $orderItemRepository;
        $this->paymentManager = $paymentManager;
        $this->notificationService = $notificationService;
        $this->logger = $logger;
        $this->validator = new OrderValidator();
    }
    
    public function createOrder(int $customerId, array $items): Order
    {
        // 1. 创建订单对象
        $order = new Order($customerId, $items);
        
        // 2. 验证订单
        $result = $this->validator->validate($order);
        if (!$result->isValid) {
            throw new InvalidArgumentException(
                implode(', ', $result->errors)
            );
        }
        
        // 3. 保存订单
        $orderId = $this->orderRepository->save($order);
        $order->setId($orderId);
        
        // 4. 保存订单项
        $this->orderItemRepository->save($orderId, $items);
        
        // 5. 发送通知
        $this->notificationService->send(
            "customer@example.com",
            'Order Created',
            "Your order #{$orderId} has been created"
        );
        
        // 6. 记录日志
        $this->logger->info("Order created: {$orderId}");
        
        return $order;
    }
    
    public function processPayment(int $orderId, string $paymentMethod): bool
    {
        // 1. 获取订单
        $order = $this->orderRepository->findById($orderId);
        
        if ($order === null) {
            throw new RuntimeException('Order not found');
        }
        
        // 2. 处理支付（通过支付管理器，支持多种支付方式）
        $success = $this->paymentManager->process(
            $paymentMethod,
            $order->getTotal()
        );
        
        // 3. 更新订单状态
        if ($success) {
            $order->setStatus('paid');
            $this->orderRepository->update($order);
            $this->logger->info("Payment processed for order: {$orderId}");
        } else {
            $this->logger->error("Payment failed for order: {$orderId}");
        }
        
        return $success;
    }
}
```

### 重构效果分析

通过重构，我们实现了以下改进：

1. **单一职责原则（SRP）**：
   - `Order` 负责数据
   - `OrderRepository` 负责数据持久化
   - `PaymentGateway` 负责支付处理
   - `NotificationService` 负责通知
   - `Logger` 负责日志
   - `OrderValidator` 负责验证

2. **开闭原则（OCP）**：
   - 添加新的支付方式只需创建新的 `PaymentGatewayInterface` 实现类
   - 不需要修改现有代码

3. **里氏替换原则（LSP）**：
   - 所有实现类都可以替换接口使用

4. **接口隔离原则（ISP）**：
   - 每个接口都小而专注
   - 客户端只依赖需要的接口

5. **依赖倒置原则（DIP）**：
   - `OrderService` 依赖接口，不依赖具体实现
   - 可以轻松切换数据库、支付网关等实现

## 重构步骤和技巧

### 识别需要重构的代码

信号包括：
- 类太大，承担多项职责
- 修改一个功能会影响其他功能
- 难以对代码进行单元测试
- 添加新功能需要修改大量现有代码

### 重构步骤

1. **识别职责**：分析类承担了多少职责
2. **提取接口**：为可替换的依赖定义接口
3. **分离职责**：将不同职责拆分成独立的类
4. **注入依赖**：使用依赖注入替代内部创建
5. **测试验证**：确保重构后功能正常

### 重构技巧

1. **逐步重构**：不要一次性重写整个系统
2. **保持测试**：先写测试，再重构
3. **小步前进**：每次只改一点
4. **频繁验证**：每次修改后运行测试
5. **文档记录**：记录重构的原因和变化

## 原则之间的权衡

### 常见的权衡场景

1. **SRP vs 复杂性**：
   - 过度分离会导致类数量爆炸
   - 需要在职责单一和代码简洁之间找到平衡

2. **OCP vs 过度设计**：
   - 不是所有地方都需要抽象
   - 只为可能变化的地方创建扩展点

3. **ISP vs 接口数量**：
   - 接口过多会导致依赖复杂
   - 可以合并经常一起使用的接口

4. **DIP vs 性能**：
   - 过度抽象可能影响性能
   - 关键路径可以直接依赖具体实现

### 决策建议

1. **根据需求变化频率**：
   - 变化频繁的地方优先应用原则
   - 稳定的地方可以简化

2. **根据项目规模**：
   - 小项目可以简化
   - 大项目需要严格遵循

3. **根据团队经验**：
   - 新手可以先从简单开始
   - 有经验的团队可以深入应用

## 最佳实践总结

1. **从简单开始**：不要一开始就追求完美的设计
2. **持续重构**：代码会随着时间腐化，需要持续改进
3. **测试驱动**：先写测试，确保重构不破坏功能
4. **团队共识**：确保团队成员都理解设计原则
5. **权衡利弊**：根据实际情况灵活应用原则

## 练习任务

1. **分析现有代码**：选择一个你过去的项目，分析违反 SOLID 原则的地方

2. **重构一个类**：选择一个违反 SRP 的类，将其拆分成多个单一职责的类

3. **设计一个系统**：使用 SOLID 原则设计一个用户管理系统

4. **权衡练习**：找一个需要权衡多个原则的场景，分析利弊并做出决策

5. **团队讨论**：与团队成员讨论 SOLID 原则的应用，达成共识
