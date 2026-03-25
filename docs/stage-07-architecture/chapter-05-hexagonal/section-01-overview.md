# 7.5.1 六边形架构概述

## 概述

六边形架构（Hexagonal Architecture），也被称为端口与适配器架构（Ports and Adapters Architecture），由 Alistair Cockburn 在 2005 年提出。这种架构模式的核心思想是将应用的核心业务逻辑与外部依赖（如数据库、UI、外部服务等）完全隔离，使得业务逻辑可以独立于技术实现进行测试和演进。

六边形架构的"六角"并非固定为六个边，而是一个比喻，表示应用可以被多个不同的"驱动者"（如 Web UI、命令行工具、测试脚本）和"驱动者"（如数据库、第三方服务）所围绕。

**主要内容**：
- 六边形架构的定义和核心概念
- 架构的优势
- 核心组成：业务核心、端口、适配器
- 与传统架构的对比

## 六边形架构核心概念

### 架构示意图

```
                    ┌─────────────────────────────────┐
                    │         外部世界              │
                    │  (Web UI, CLI, Tests, etc.)    │
                    └──────────────┬──────────────────┘
                                   │
                    ┌──────────────▼──────────────────┐
                    │      驱动适配器（Driving）       │
                    │   (Controllers, Commands)       │
                    └──────────────┬──────────────────┘
                                   │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
    ┌─────────▼─────────┐  ┌─────────▼─────────┐  ┌─────────▼─────────┐
    │      端口 1       │  │      端口 2       │  │      端口 N       │
    │  (User Service)   │  │ (Payment Service) │  │     (Ports)      │
    └─────────┬─────────┘  └─────────┬─────────┘  └─────────┬─────────┘
              │                     │                     │
              │         ┌───────────┴───────────┐          │
              │         │                     │          │
    ┌─────────▼─────────▼──────┐  ┌────────────▼────────────┐
    │                          │  │                        │
    │      业务核心           │  │      业务核心          │
    │   (Domain Logic)       │  │   (Domain Logic)       │
    │                          │  │                        │
    └────────────┬────────────┘  └───────────┬────────────┘
                  │                             │
    ┌─────────────▼─────────────────────────────▼─────────────┐
    │                  被驱动适配器（Driven）               │
    │      (Database, External Services, etc.)              │
    └─────────────────────────────────────────────────────┘
```

### 核心组成

```php
<?php
declare(strict_types=1);

// 1. 业务核心（Domain/Core）
// 不依赖任何外部技术，只包含业务逻辑

class Order
{
    private int $id;
    private int $customerId;
    private array $items;
    private string $status;
    
    public function __construct(int $customerId, array $items)
    {
        $this->customerId = $customerId;
        $this->items = $items;
        $this->status = 'pending';
    }
    
    public function getTotal(): float
    {
        return array_reduce(
            $this->items,
            fn(float $sum, $item) => $sum + ($item['price'] * $item['quantity']),
            0
        );
    }
    
    public function canCancel(): bool
    {
        return $this->status === 'pending';
    }
    
    public function cancel(): void
    {
        if (!$this->canCancel()) {
            throw new DomainException("Cannot cancel order in status: {$this->status}");
        }
        $this->status = 'cancelled';
    }
}

// 2. 端口（Ports）
// 定义业务核心与外部世界交互的接口

// 输入端口（Driving Port）
interface OrderServiceInterface
{
    public function createOrder(int $customerId, array $items): Order;
    public function cancelOrder(int $orderId): void;
}

// 输出端口（Driven Port）
interface OrderRepositoryInterface
{
    public function save(Order $order): int;
    public function findById(int $id): ?Order;
}

interface PaymentGatewayInterface
{
    public function process(float $amount): bool;
}

interface NotificationServiceInterface
{
    public function sendOrderConfirmation(Order $order): void;
}

// 3. 适配器（Adapters）
// 实现端口接口的具体技术实现

// 驱动适配器（Driving Adapter）
class HttpOrderController implements OrderServiceInterface
{
    public function __construct(
        private OrderServiceInterface $orderService
    ) {}
    
    public function createOrder(int $customerId, array $items): Order
    {
        return $this->orderService->createOrder($customerId, $items);
    }
    
    public function cancelOrder(int $orderId): void
    {
        $this->orderService->cancelOrder($orderId);
    }
}

// 被驱动适配器（Driven Adapter）
class MySQLOrderRepository implements OrderRepositoryInterface
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function save(Order $order): int
    {
        // 数据库实现
    }
    
    public function findById(int $id): ?Order
    {
        // 数据库实现
    }
}

class StripePaymentGateway implements PaymentGatewayInterface
{
    public function process(float $amount): bool
    {
        // Stripe API 实现
    }
}
```

## 架构优势

### 1. 业务核心独立

```php
<?php
declare(strict_types=1);

// 业务核心完全不依赖外部技术
class OrderService
{
    private OrderRepositoryInterface $repository;
    private PaymentGatewayInterface $payment;
    private NotificationServiceInterface $notifier;
    
    public function __construct(
        OrderRepositoryInterface $repository,
        PaymentGatewayInterface $payment,
        NotificationServiceInterface $notifier
    ) {
        $this->repository = $repository;
        $this->payment = $payment;
        $this->notifier = $notifier;
    }
    
    public function createOrder(int $customerId, array $items): Order
    {
        // 业务逻辑完全独立，不依赖任何外部实现
        $order = new Order($customerId, $items);
        
        // 处理支付
        $this->payment->process($order->getTotal());
        
        // 保存订单
        $this->repository->save($order);
        
        // 发送通知
        $this->notifier->sendOrderConfirmation($order);
        
        return $order;
    }
}
```

### 2. 技术无关性

```php
<?php
declare(strict_types=1);

// 同一个业务逻辑可以使用不同的适配器

// MySQL 适配器
class MySQLUserRepository implements UserRepositoryInterface {}

// MongoDB 适配器
class MongoDBUserRepository implements UserRepositoryInterface {}

// Elasticsearch 适配器
class ElasticsearchUserRepository implements UserRepositoryInterface {}

// 内存适配器（用于测试）
class InMemoryUserRepository implements UserRepositoryInterface {}

// 可以轻松切换
$repository = new MySQLUserRepository($pdo);
// 或
$repository = new MongoDBUserRepository($mongoClient);
// 或
$repository = new InMemoryUserRepository();
```

### 3. 测试友好

```php
<?php
declare(strict_types=1);

// 使用内存适配器进行单元测试
class InMemoryUserRepository implements UserRepositoryInterface
{
    private array $users = [];
    
    public function save(User $user): int
    {
        $id = count($this->users) + 1;
        $this->users[$id] = $user;
        return $id;
    }
    
    public function findById(int $id): ?User
    {
        return $this->users[$id] ?? null;
    }
}

// 单元测试
class UserServiceTest
{
    public function testCreateUser(): void
    {
        // 使用内存适配器，不需要真实数据库
        $repository = new InMemoryUserRepository();
        $service = new UserService($repository);
        
        $user = $service->createUser('John', 'john@example.com');
        
        $this->assertEquals('John', $user->getName());
    }
}
```

## 与传统架构对比

| 方面 | 传统分层架构 | 六边形架构 |
|:-----|:------------|:------------|
| 依赖方向 | 表现层 → 业务层 → 数据层 | 外部 → 端口 → 核心 |
| 业务核心 | 依赖数据层 | 独立，不依赖外部 |
| 测试 | 需要模拟数据库 | 可以完全隔离测试 |
| 技术替换 | 困难 | 容易 |
| 代码组织 | 按技术分层 | 按业务功能组织 |

## 使用场景

六边形架构适用于：
- 复杂业务逻辑的企业应用
- 需要集成多个外部系统
- 对测试有高要求
- 需要长期维护和演进
- 需要灵活切换技术实现

## 练习任务

1. **设计六边形架构**：为一个电商系统设计六边形架构

2. **实现端口和适配器**：实现用户管理的端口和适配器

3. **测试业务核心**：使用模拟适配器测试业务逻辑

4. **重构现有代码**：将一个传统分层架构重构为六边形架构
