# 7.7.1 CQRS 模式

## 概述

CQRS（Command Query Responsibility Segregation，命令查询职责分离）是一种将系统的读操作和写操作分离的架构模式。CQRS 的核心思想是：命令（写入数据）和查询（读取数据）是两种不同性质的操作，它们有不同的性能需求、数据表示方式和扩展策略，因此应该使用不同的模型来处理。

传统的 CRUD 应用通常使用统一的模型来处理读和写。例如，一个 `UserService` 可能同时包含 `createUser`、`updateUser`、`deleteUser`（命令）和 `getUser`、`listUsers`（查询）方法。这种方式简单直观，但在复杂场景下可能会遇到问题：读操作和写操作有不同的性能特征，读操作可能需要特殊的数据格式来提高查询效率，而写操作需要完整的业务验证逻辑。

CQRS 模式将这种混合的职责分离，允许分别优化读模型和写模型。写模型可以专注于业务规则的完整性和数据的一致性，而读模型可以针对查询性能进行优化，使用反规范化的设计来减少关联查询。这种分离在大型系统中特别有价值，可以显著提高系统的整体性能。

**主要内容**：
- CQRS 的基本概念
- 命令与查询的分离原则
- 写模型的设计
- 读模型的设计
- CQRS 的实现方法
- 完整的代码示例

## 特性

- **读写分离**：命令和查询使用不同的处理逻辑
- **独立优化**：可以分别优化读模型和写模型的性能
- **不同数据表示**：读模型和写模型可以使用不同的数据结构
- **事件驱动**：CQRS 通常与事件溯源结合使用
- **最终一致性**：读模型可能与写模型存在短暂的不一致

## 核心概念

### 命令（Command）

命令是引起数据变化的操作。命令应该表达意图，而不是携带数据。例如，"创建用户"是一个命令，而不是"保存用户数据"。命令应该是幂等的，即多次执行相同的命令应该得到相同的结果。

命令的特点：
- 执行后会导致状态变化
- 可能包含复杂的业务验证逻辑
- 通常返回执行结果（成功或失败）
- 命名应该使用动词，如 CreateOrder、CancelOrder

### 查询（Query）

查询是读取数据而不改变状态的操作。查询应该是纯函数，多次执行相同查询应该返回相同的结果。

查询的特点：
- 不改变系统状态
- 专注于数据的检索和展示
- 可以针对性能进行优化
- 命名应该使用动词，如 GetOrder、ListOrders

### 读写模型分离

```
┌─────────────────────────────────────────────────────────────────┐
│                         应用层                                  │
│  ┌─────────────────────────┐    ┌─────────────────────────┐   │
│  │     命令端 (Command)    │    │      查询端 (Query)      │   │
│  │  ┌─────────────────┐   │    │  ┌─────────────────┐   │   │
│  │  │ CreateOrder    │   │    │  │ GetOrder        │   │   │
│  │  │ UpdateOrder    │   │    │  │ ListOrders      │   │   │
│  │  │ CancelOrder    │   │    │  │ GetOrderSummary │   │   │
│  │  └────────┬────────┘   │    │  └────────┬────────┘   │   │
│  └───────────│────────────┘    └───────────│────────────┘   │
│              │                              │                │
│              ▼                              ▼                │
│  ┌─────────────────────────┐    ┌─────────────────────────┐ │
│  │        写模型           │    │         读模型           │ │
│  │  (Domain Model)         │    │  (Read Model/Projection) │ │
│  │  - 完整的业务逻辑       │    │  - 反规范化设计          │ │
│  │  - 实体和聚合          │    │  - 专用视图              │ │
│  │  - 验证规则            │    │  - 缓存优化              │ │
│  └────────────┬────────────┘    └────────────┬────────────┘ │
│               │                                │               │
│               ▼                                ▼               │
│  ┌─────────────────────────┐    ┌─────────────────────────┐ │
│  │        命令端数据库     │    │        查询端数据库     │ │
│  │     (写入数据库)        │    │     (读取数据库)        │ │
│  └─────────────────────────┘    └─────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## 语法/定义

### 命令定义

```php
<?php
declare(strict_types=1);

// 语法：class CommandName { ... }

// 特征：
// - 使用动词命名
// - 表达业务意图
// - 不可变对象
```

### 命令处理器定义

```php
<?php
declare(strict_types=1);

// 语法：class CommandNameHandler { ... }

// 特征：
// - 处理命令执行
// - 调用领域逻辑
// - 返回执行结果
```

### 查询定义

```php
<?php
declare(strict_types=1);

// 语法：class QueryName { ... }

// 特征：
// - 使用动词命名
// - 表达数据需求
// - 不可变对象
```

## 基本用法

### 1. 命令定义和实现

以下是订单相关命令的定义：

```php
<?php
declare(strict_types=1);

namespace App\Application\Command;

class CreateOrderCommand
{
    public function __construct(
        public readonly int $customerId,
        public readonly string $customerEmail,
        public readonly array $items
    ) {}
}

class UpdateOrderCommand
{
    public function __construct(
        public readonly int $orderId,
        public readonly ?string $shippingAddress = null,
        public readonly ?string $notes = null
    ) {}
}

class ConfirmOrderCommand
{
    public function __construct(
        public readonly int $orderId
    ) {}
}

class CancelOrderCommand
{
    public function __construct(
        public readonly int $orderId,
        public readonly string $reason
    ) {}
}
```

### 2. 命令处理器实现

```php
<?php
declare(strict_types=1);

namespace App\Application\Handler;

use App\Application\Command\CreateOrderCommand;
use App\Domain\Entity\Order;
use App\Domain\Entity\OrderItem;
use App\Domain\ValueObject\Money;
use App\Port\Output\OrderRepositoryPort;
use App\Port\Output\DomainEventPublisherPort;

class CreateOrderHandler
{
    public function __construct(
        private readonly OrderRepositoryPort $orderRepository,
        private readonly DomainEventPublisherPort $eventPublisher
    ) {}

    public function handle(CreateOrderCommand $command): int
    {
        $order = new Order(
            customerId: $command->customerId,
            customerEmail: $command->customerEmail
        );

        foreach ($command->items as $itemData) {
            $item = new OrderItem(
                productId: $itemData['product_id'],
                productName: $itemData['product_name'],
                quantity: $itemData['quantity'],
                unitPrice: new Money($itemData['unit_price'], 'CNY')
            );
            $order->addItem($item);
        }

        $orderId = $this->orderRepository->save($order);

        $events = $order->pullDomainEvents();
        foreach ($events as $event) {
            $this->eventPublisher->publish($event);
        }

        return $orderId;
    }
}
```

### 3. 查询定义和实现

```php
<?php
declare(strict_types=1);

namespace App\Application\Query;

class GetOrderQuery
{
    public function __construct(
        public readonly int $orderId
    ) {}
}

class ListOrdersQuery
{
    public function __construct(
        public readonly int $customerId,
        public readonly int $page = 1,
        public readonly int $limit = 20
    ) {}
}

class GetOrderSummaryQuery
{
    public function __construct(
        public readonly int $orderId
    ) {}
}
```

### 4. 查询处理器实现

```php
<?php
declare(strict_types=1);

namespace App\Application\Handler;

use App\Application\Query\GetOrderQuery;
use App\Application\Query\ListOrdersQuery;
use App\Application\Query\GetOrderSummaryQuery;
use App\Application\Dto\OrderDto;
use App\Application\Dto\OrderSummaryDto;
use App\Port\Output\OrderQueryPort;

class OrderQueryHandler
{
    public function __construct(
        private readonly OrderQueryPort $queryPort
    ) {}

    public function handle(GetOrderQuery $query): ?OrderDto
    {
        $order = $this->queryPort->findById($query->orderId);
        
        if ($order === null) {
            return null;
        }

        return new OrderDto(
            id: $order['id'],
            customerId: $order['customer_id'],
            customerEmail: $order['customer_email'],
            status: $order['status'],
            items: $order['items'],
            totalAmount: $order['total_amount'],
            currency: $order['currency'],
            createdAt: $order['created_at']
        );
    }

    public function handle(ListOrdersQuery $query): array
    {
        $orders = $this->queryPort->findByCustomerId(
            $query->customerId,
            $query->page,
            $query->limit
        );

        return array_map(
            fn($order) => new OrderDto(
                id: $order['id'],
                customerId: $order['customer_id'],
                customerEmail: $order['customer_email'],
                status: $order['status'],
                items: $order['items'],
                totalAmount: $order['total_amount'],
                currency: $order['currency'],
                createdAt: $order['created_at']
            ),
            $orders
        );
    }

    public function handle(GetOrderSummaryQuery $query): ?OrderSummaryDto
    {
        $summary = $this->queryPort->findSummaryById($query->orderId);
        
        if ($summary === null) {
            return null;
        }

        return new OrderSummaryDto(
            id: $summary['id'],
            status: $summary['status'],
            totalAmount: $summary['total_amount'],
            itemCount: $summary['item_count'],
            customerName: $summary['customer_name']
        );
    }
}
```

### 5. 读模型实现（专用视图）

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\Query;

use App\Port\Output\OrderQueryPort;

class MysqlOrderQueryModel implements OrderQueryPort
{
    public function __construct(private readonly \PDO $pdo) {}

    public function findById(int $id): ?array
    {
        $stmt = $this->pdo->prepare(
            'SELECT o.*, 
                    (SELECT COUNT(*) FROM order_items WHERE order_id = o.id) as item_count
             FROM orders o 
             WHERE o.id = ?'
        );
        $stmt->execute([$id]);
        $order = $stmt->fetch(\PDO::FETCH_ASSOC);

        if ($order === false) {
            return null;
        }

        $items = $this->findOrderItems($id);
        
        return [
            'id' => (int) $order['id'],
            'customer_id' => (int) $order['customer_id'],
            'customer_email' => $order['customer_email'],
            'status' => $order['status'],
            'items' => $items,
            'total_amount' => (int) $order['total_amount'],
            'currency' => $order['currency'],
            'created_at' => $order['created_at']
        ];
    }

    public function findByCustomerId(int $customerId, int $page, int $limit): array
    {
        $offset = ($page - 1) * $limit;
        
        $stmt = $this->pdo->prepare(
            'SELECT * FROM orders 
             WHERE customer_id = ? 
             ORDER BY created_at DESC 
             LIMIT ? OFFSET ?'
        );
        $stmt->bindValue(1, $customerId, \PDO::PARAM_INT);
        $stmt->bindValue(2, $limit, \PDO::PARAM_INT);
        $stmt->bindValue(3, $offset, \PDO::PARAM_INT);
        $stmt->execute();

        return $stmt->fetchAll(\PDO::FETCH_ASSOC);
    }

    public function findSummaryById(int $id): ?array
    {
        $stmt = $this->pdo->prepare(
            'SELECT o.id, o.status, o.total_amount, o.currency,
                    COUNT(oi.id) as item_count,
                    u.name as customer_name
             FROM orders o
             LEFT JOIN order_items oi ON o.id = oi.order_id
             LEFT JOIN users u ON o.customer_id = u.id
             WHERE o.id = ?
             GROUP BY o.id'
        );
        $stmt->execute([$id]);
        $result = $stmt->fetch(\PDO::FETCH_ASSOC);

        if ($result === false) {
            return null;
        }

        return [
            'id' => (int) $result['id'],
            'status' => $result['status'],
            'total_amount' => (int) $result['total_amount'],
            'item_count' => (int) $result['item_count'],
            'customer_name' => $result['customer_name']
        ];
    }

    private function findOrderItems(int $orderId): array
    {
        $stmt = $this->pdo->prepare(
            'SELECT * FROM order_items WHERE order_id = ?'
        );
        $stmt->execute([$orderId]);
        return $stmt->fetchAll(\PDO::FETCH_ASSOC);
    }
}
```

### 6. 命令总线和查询总线

```php
<?php
declare(strict_types=1);

namespace App\Infrastructure\Bus;

class CommandBus
{
    private array $handlers = [];

    public function register(string $commandClass, callable $handler): void
    {
        $this->handlers[$commandClass] = $handler;
    }

    public function dispatch(object $command): mixed
    {
        $commandClass = get_class($command);
        
        if (!isset($this->handlers[$commandClass])) {
            throw new \RuntimeException("No handler registered for {$commandClass}");
        }

        $handler = $this->handlers[$commandClass];
        return $handler($command);
    }
}
```

```php
<?php
declare(strict_types=1);

namespace App\Infrastructure\Bus;

class QueryBus
{
    private array $handlers = [];

    public function register(string $queryClass, callable $handler): void
    {
        $this->handlers[$queryClass] = $handler;
    }

    public function ask(object $query): mixed
    {
        $queryClass = get_class($query);
        
        if (!isset($this->handlers[$queryClass])) {
            throw new \RuntimeException("No handler registered for {$queryClass}");
        }

        $handler = $this->handlers[$queryClass];
        return $handler($query);
    }
}
```

## 使用场景

### 1. 复杂查询场景

当系统需要支持复杂的查询操作时，CQRS 允许为读操作设计专门的模型。可以创建反规范化的视图、索引表等来优化查询性能。

### 2. 读写性能差异大

当读操作和写操作的数量差异很大时，可以分别扩展读模型和写模型。例如，在电商系统中，读操作（浏览商品）远多于写操作（下单），可以为读操作配置更多的资源。

### 3. 多端显示

当同一个数据需要以不同方式显示给不同用户时，CQRS 允许为每种显示方式创建专门的读模型。

### 4. 事件驱动系统

CQRS 与事件溯源结合使用时，可以实现完全的事件驱动架构。写操作产生事件，读模型通过投影事件来构建。

## 注意事项

### 1. 复杂度增加

CQRS 会增加系统的复杂度，特别是需要维护多个数据模型。应该在确实需要时才使用 CQRS。

### 2. 最终一致性

读模型和写模型之间可能存在短暂的不一致。需要考虑如何处理这种不一致，以及如何在界面上展示。

### 3. 避免过早优化

不要在项目初期就使用 CQRS。先使用简单的 CRUD，当性能或复杂度需求出现时再考虑 CQRS。

## 常见问题

### Q1: CQRS 与 CRUD 有什么区别？

CQRS 将读和写分离，可以使用不同的模型和处理逻辑。CRUD 使用统一的模型处理读和写。在简单场景下，CRUD 更合适；在复杂场景下，CQRS 可以提供更好的性能和灵活性。

### Q2: CQRS 必须使用事件溯源吗？

不一定。CQRS 可以与传统的持久化方式结合使用。但 CQRS 与事件溯源是天然的组合，很多 CQRS 实现都使用事件溯源。

### Q3: 如何处理读模型和写模型的数据同步？

可以通过同步机制（如数据库复制）或异步机制（如事件驱动）来同步数据。同步方式延迟低，但耦合度高；异步方式延迟高，但解耦更好。

## 最佳实践

### 1. 从简单开始

先使用统一的模型，只在需要时才分离读和写。

### 2. 清晰的命名

命令和查询的命名应该清晰表达意图，使用动词开头。

### 3. 优化读模型

根据实际的查询需求设计和优化读模型，使用反规范化、缓存等手段提高性能。

### 4. 考虑一致性

明确读模型和写模型之间的同步策略，考虑界面上如何处理不一致的情况。

## 练习任务

### 练习 1：设计命令和查询

为一个用户管理系统设计 CQRS 模式的命令和查询。包括创建用户、更新用户、删除用户等命令，以及获取用户、列表用户等查询。

### 2：实现读模型

为订单系统实现一个优化的读模型，设计专门用于查询的数据表或视图。

### 3：实现命令处理器

实现订单系统的命令处理器，包括创建订单、确认订单、取消订单等。

### 4：设计事件同步机制

设计一个机制，通过事件将写模型的数据同步到读模型。

### 5：评估 CQRS 适用性

分析一个现有的电商系统，评估是否适合使用 CQRS，并说明理由。
