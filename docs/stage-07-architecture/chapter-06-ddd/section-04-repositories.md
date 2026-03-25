# 7.6.4 仓储模式

## 概述

仓储模式（Repository Pattern）是领域驱动设计中用于管理领域模型持久化的核心模式。仓储扮演着领域模型与数据存储之间的中介角色，它封装了数据访问的技术细节，使得领域层可以独立于具体的存储技术。仓储的核心理念是模拟一个内存中的对象集合，提供添加、删除、查询等操作，让领域代码像操作内存对象一样操作持久化数据。

在传统的多层架构中，数据访问层（DAO）往往直接暴露数据库表结构，领域层需要了解数据库的表结构、字段类型等细节。这种设计导致领域层与数据存储技术紧密耦合，当需要更换存储技术（如从 MySQL 切换到 MongoDB）或改变数据模型时，需要大量修改领域层代码。仓储模式通过在领域层定义仓储接口，将数据访问的实现细节隔离在基础设施层，从而实现了领域层与技术的解耦。

仓储模式与数据访问对象（DAO）模式有相似之处，但也有本质区别。DAO 关注的是数据库表的 CRUD 操作，而仓储关注的是领域模型的持久化。仓储接口应该使用业务语言命名，反映领域概念，而不是暴露数据库细节。

**主要内容**：
- 仓储模式的概念和作用
- 仓储接口的设计原则
- 仓储的实现方法
- 仓储查询技术
- 仓储与聚合的关系
- 完整的代码示例

## 特性

- **领域导向**：仓储接口使用领域语言而非技术语言
- **集合抽象**：仓储模拟内存中的对象集合
- **技术隔离**：领域层不依赖具体的存储技术
- **可测试性**：可以使用内存实现进行单元测试
- **单一职责**：每个聚合有独立的仓储

## 核心概念

### 仓储的工作原理

仓储位于领域层和基础设施层之间。领域层通过仓储接口与数据存储交互，具体的实现位于基础设施层。这种分层遵循了依赖倒置原则：领域层定义了接口，基础设施层实现了接口。

```
┌─────────────────────────────────────────────────────┐
│                      领域层                          │
│                                                     │
│  ┌───────────────────────────────────────────────┐ │
│  │              聚合根 (Order)                   │ │
│  └───────────────────────────────────────────────┘ │
│                         │                          │
│                         ▼                          │
│  ┌───────────────────────────────────────────────┐ │
│  │         OrderRepositoryPort (接口)            │ │
│  └───────────────────────────────────────────────┘ │
└────────────────────────┬────────────────────────────┘
                         │ 依赖
                         ▼
┌─────────────────────────────────────────────────────┐
│                   基础设施层                        │
│                                                     │
│  ┌───────────────────────────────────────────────┐ │
│  │    MySqlOrderRepository (实现)                │ │
│  └───────────────────────────────────────────────┘ │
│                         │                          │
│                         ▼                          │
│  ┌───────────────────────────────────────────────┐ │
│  │              MySQL 数据库                      │ │
│  └───────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

### 仓储接口设计原则

仓储接口的设计应该遵循以下原则：

第一，接口应该使用领域语言。方法名应该反映领域概念，如 `findByCustomerId` 而不是 `selectByCustomerId`。这样可以使领域代码更加清晰，也便于与业务专家沟通。

第二，接口应该以聚合根为参数。仓储方法应该接收和返回聚合根对象，而不是聚合内部的实体或值对象。这保持了聚合的完整性。

第三，接口应该保持简洁。避免在仓储接口中暴露过多的查询方法。如果需要复杂的查询，可以考虑使用查询对象（Query Object）或规范对象（Specification）。

第四，接口应该是稳定的。一旦接口发布，应该尽量保持不变。如果需要添加方法，应该通过向后兼容的方式进行。

## 语法/定义

### 仓储接口定义

```php
<?php
declare(strict_types=1);

// 语法：interface RepositoryNamePort { ... }

// 特征：
// - 使用领域语言命名
// - 以聚合根为参数
// - 包含基本的增删改查方法
```

### 仓储实现定义

```php
<?php
declare(strict_types=1);

// 语法：class RepositoryName implements RepositoryNamePort { ... }

// 特征：
// - 实现领域层定义的接口
// - 处理技术细节
// - 不包含业务逻辑
```

## 基本用法

### 1. 仓储接口定义

以下是订单仓储的接口定义：

```php
<?php
declare(strict_types=1);

namespace App\Port\Output;

use App\Domain\Entity\Order;

interface OrderRepositoryPort
{
    public function findById(int $id): ?Order;
    
    public function findByCustomerId(int $customerId): array;
    
    public function findPending(): array;
    
    public function findByStatus(string $status, int $page = 1, int $limit = 20): array;
    
    public function save(Order $order): int;
    
    public function update(Order $order): void;
    
    public function delete(int $id): void;
    
    public function count(): int;
    
    public function countByStatus(string $status): int;
}
```

这个接口定义了订单聚合的持久化操作。方法名使用领域语言，如 `findPending` 表示查询待处理的订单。接口接收和返回的都是 `Order` 聚合根对象。

### 2. 仓储实现

以下是 MySQL 数据库的仓储实现：

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\Database;

use App\Port\Output\OrderRepositoryPort;
use App\Domain\Entity\Order;
use App\Domain\ValueObject\OrderStatus;
use PDO;
use PDOException;

class MySqlOrderRepository implements OrderRepositoryPort
{
    public function __construct(private readonly PDO $pdo) {}

    public function findById(int $id): ?Order
    {
        $stmt = $this->pdo->prepare(
            'SELECT * FROM orders WHERE id = :id'
        );
        $stmt->execute(['id' => $id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);

        if ($row === false) {
            return null;
        }

        return $this->mapToOrder($row);
    }

    public function findByCustomerId(int $customerId): array
    {
        $stmt = $this->pdo->prepare(
            'SELECT * FROM orders WHERE customer_id = :customer_id ORDER BY created_at DESC'
        );
        $stmt->execute(['customer_id' => $customerId]);
        $rows = $stmt->fetchAll(PDO::FETCH_ASSOC);

        return array_map([$this, 'mapToOrder'], $rows);
    }

    public function findPending(): array
    {
        $stmt = $this->pdo->prepare(
            "SELECT * FROM orders WHERE status = 'pending' ORDER BY created_at ASC"
        );
        $stmt->execute();
        $rows = $stmt->fetchAll(PDO::FETCH_ASSOC);

        return array_map([$this, 'mapToOrder'], $rows);
    }

    public function findByStatus(string $status, int $page = 1, int $limit = 20): array
    {
        $offset = ($page - 1) * $limit;
        
        $stmt = $this->pdo->prepare(
            'SELECT * FROM orders WHERE status = :status ORDER BY created_at DESC LIMIT :limit OFFSET :offset'
        );
        $stmt->bindValue('status', $status);
        $stmt->bindValue('limit', $limit, PDO::PARAM_INT);
        $stmt->bindValue('offset', $offset, PDO::PARAM_INT);
        $stmt->execute();
        
        $rows = $stmt->fetchAll(PDO::FETCH_ASSOC);

        return array_map([$this, 'mapToOrder'], $rows);
    }

    public function save(Order $order): int
    {
        $this->pdo->beginTransaction();

        try {
            $stmt = $this->pdo->prepare(
                'INSERT INTO orders (customer_id, customer_email, status, total_amount, currency, created_at, updated_at) 
                 VALUES (:customer_id, :customer_email, :status, :total_amount, :currency, :created_at, :updated_at)'
            );

            $now = date('Y-m-d H:i:s');
            $stmt->execute([
                'customer_id' => $order->getCustomerId(),
                'customer_email' => $order->getCustomerEmail(),
                'status' => $order->getStatus()->value,
                'total_amount' => $order->getTotalAmount()->getAmount(),
                'currency' => $order->getTotalAmount()->getCurrency(),
                'created_at' => $now,
                'updated_at' => $now
            ]);

            $orderId = (int) $this->pdo->lastInsertId();
            $order->setId($orderId);

            $this->saveOrderItems($order);

            $this->pdo->commit();
            return $orderId;
        } catch (PDOException $e) {
            $this->pdo->rollBack();
            throw $e;
        }
    }

    public function update(Order $order): void
    {
        $this->pdo->beginTransaction();

        try {
            $stmt = $this->pdo->prepare(
                'UPDATE orders 
                 SET status = :status, total_amount = :total_amount, updated_at = :updated_at 
                 WHERE id = :id'
            );

            $stmt->execute([
                'id' => $order->getId(),
                'status' => $order->getStatus()->value,
                'total_amount' => $order->getTotalAmount()->getAmount(),
                'updated_at' => date('Y-m-d H:i:s')
            ]);

            $this->updateOrderItems($order);

            $this->pdo->commit();
        } catch (PDOException $e) {
            $this->pdo->rollBack();
            throw $e;
        }
    }

    public function delete(int $id): void
    {
        $stmt = $this->pdo->prepare('DELETE FROM orders WHERE id = :id');
        $stmt->execute(['id' => $id]);
    }

    public function count(): int
    {
        $stmt = $this->pdo->query('SELECT COUNT(*) FROM orders');
        return (int) $stmt->fetchColumn();
    }

    public function countByStatus(string $status): int
    {
        $stmt = $this->pdo->prepare('SELECT COUNT(*) FROM orders WHERE status = :status');
        $stmt->execute(['status' => $status]);
        return (int) $stmt->fetchColumn();
    }

    private function saveOrderItems(Order $order): void
    {
        $stmt = $this->pdo->prepare(
            'INSERT INTO order_items (order_id, product_id, product_name, quantity, unit_price, subtotal) 
             VALUES (:order_id, :product_id, :product_name, :quantity, :unit_price, :subtotal)'
        );

        foreach ($order->getItems() as $item) {
            $stmt->execute([
                'order_id' => $order->getId(),
                'product_id' => $item->getProductId(),
                'product_name' => $item->getProductName(),
                'quantity' => $item->getQuantity(),
                'unit_price' => $item->getUnitPrice()->getAmount(),
                'subtotal' => $item->getSubtotal()->getAmount()
            ]);
        }
    }

    private function updateOrderItems(Order $order): void
    {
        $stmt = $this->pdo->prepare('DELETE FROM order_items WHERE order_id = :order_id');
        $stmt->execute(['order_id' => $order->getId()]);

        $this->saveOrderItems($order);
    }

    private function mapToOrder(array $row): Order
    {
        $order = new Order(
            customerId: (int) $row['customer_id'],
            customerEmail: $row['customer_email']
        );
        $order->setId((int) $row['id']);

        return $order;
    }
}
```

### 3. 内存实现（用于测试）

仓储的内存实现可以在单元测试中替代真实的数据库实现，加快测试速度：

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\Memory;

use App\Port\Output\OrderRepositoryPort;
use App\Domain\Entity\Order;

class InMemoryOrderRepository implements OrderRepositoryPort
{
    private array $orders = [];
    private int $nextId = 1;

    public function findById(int $id): ?Order
    {
        return $this->orders[$id] ?? null;
    }

    public function findByCustomerId(int $customerId): array
    {
        return array_filter(
            $this->orders,
            fn(Order $order) => $order->getCustomerId() === $customerId
        );
    }

    public function findPending(): array
    {
        return array_filter(
            $this->orders,
            fn(Order $order) => $order->getStatus()->value === 'pending'
        );
    }

    public function findByStatus(string $status, int $page = 1, int $limit = 20): array
    {
        $filtered = array_filter(
            $this->orders,
            fn(Order $order) => $order->getStatus()->value === $status
        );

        $offset = ($page - 1) * $limit;
        return array_slice($filtered, $offset, $limit);
    }

    public function save(Order $order): int
    {
        $id = $this->nextId++;
        $order->setId($id);
        $this->orders[$id] = $order;
        return $id;
    }

    public function update(Order $order): void
    {
        $id = $order->getId();
        if (isset($this->orders[$id])) {
            $this->orders[$id] = $order;
        }
    }

    public function delete(int $id): void
    {
        unset($this->orders[$id]);
    }

    public function count(): int
    {
        return count($this->orders);
    }

    public function countByStatus(string $status): int
    {
        return count(array_filter(
            $this->orders,
            fn(Order $order) => $order->getStatus()->value === $status
        ));
    }

    public function clear(): void
    {
        $this->orders = [];
        $this->nextId = 1;
    }
}
```

### 4. 仓储在用例中的使用

```php
<?php
declare(strict_types=1);

namespace App\Application\UseCase;

use App\Port\Output\OrderRepositoryPort;
use App\Application\Dto\CreateOrderDto;
use App\Domain\Entity\Order;
use App\Domain\ValueObject\OrderStatus;

class CreateOrderUseCase
{
    public function __construct(
        private readonly OrderRepositoryPort $orderRepository
    ) {}

    public function execute(CreateOrderDto $dto): Order
    {
        $order = new Order(
            customerId: $dto->customerId,
            customerEmail: $dto->customerEmail
        );

        foreach ($dto->items as $itemDto) {
            $item = new \App\Domain\Entity\OrderItem(
                productId: $itemDto->productId,
                productName: $itemDto->productName,
                quantity: $itemDto->quantity,
                unitPrice: new \App\Domain\ValueObject\Money($itemDto->unitPrice, 'CNY')
            );
            $order->addItem($item);
        }

        $orderId = $this->orderRepository->save($order);
        
        return $this->orderRepository->findById($orderId);
    }
}
```

用例完全不关心仓储的具体实现，只通过接口与仓储交互。这种设计使得可以在不修改用例代码的情况下切换不同的仓储实现。

## 使用场景

### 1. 数据持久化

仓储最基本的使用场景是将领域模型持久化到数据库或其他存储介质。仓储隐藏了数据存储的技术细节，使得领域代码可以专注于业务逻辑。

### 2. 测试隔离

在单元测试中，可以使用内存实现替代真实的数据库实现，实现测试的快速执行和环境的隔离。

### 3. 技术切换

当需要更换数据存储技术时，只需要实现新的仓储接口，不需要修改领域代码。

### 4. 读写分离

可以实现多个仓储实现，分别处理读操作和写操作，实现读写分离的架构。

## 注意事项

### 1. 仓储不是 DAO

仓储接口应该使用领域语言，方法名应该反映领域概念。避免将数据库表结构暴露在仓储接口中。

### 2. 事务管理

仓储实现需要处理事务。可以在仓储方法内部管理事务，也可以在用例层面管理事务。根据业务需求选择合适的方式。

### 3. 性能考虑

对于复杂的查询，可以考虑使用查询对象（Query Object）来封装查询逻辑，避免在仓储接口中添加过多方法。

### 4. 聚合完整性

仓储方法应该接收和返回聚合根对象，保持聚合的完整性。避免在仓储接口中暴露聚合内部的实体。

## 常见问题

### Q1: 仓储与 DAO 有什么区别？

DAO（Data Access Object）关注的是数据表的 CRUD 操作，接口通常反映数据库表结构。仓储（Repository）关注的是领域模型的持久化，接口使用领域语言。仓储是更高层的抽象，更符合 DDD 的思想。

### Q2: 一个仓储可以管理多个聚合吗？

通常每个聚合有独立的仓储。这样可以保持仓储接口的简洁，也便于管理聚合的生命周期。但在某些简单场景下，也可以多个聚合共用一个仓储。

### Q3: 仓储需要处理聚合内部的实体吗？

仓储的职责是持久化整个聚合。聚合内部实体的持久化应该在聚合的仓储实现中处理，用户不需要关心聚合内部的细节。

### Q4: 如何处理复杂的查询？

对于复杂的查询，可以创建查询对象（Query Object）来封装查询逻辑。查询对象可以在用例中使用，返回扁平化的数据传输对象（DTO）。

## 最佳实践

### 1. 一个聚合一个仓储

为每个聚合创建独立的仓储，保持仓储接口的简洁和专注。

### 2. 使用领域语言命名

方法名应该反映领域概念，使用业务语言而非技术语言。

### 3. 返回聚合根对象

仓储方法应该返回聚合根对象，保持聚合的完整性。

### 4. 实现内存版本

为每个仓储实现内存版本，用于单元测试。

### 5. 保持接口稳定

接口一旦发布，应该尽量保持不变。使用向后兼容的方式进行扩展。

## 练习任务

### 练习 1：设计产品仓储

为产品聚合设计仓储接口。考虑产品的增删改查操作，以及常见的查询场景，如按分类查询、按名称搜索等。

### 练习 2：实现用户仓储

实现一个用户仓储，包括数据库实现和内存实现。比较两种实现的异同。

### 练习 3：实现分页查询

为订单仓储实现分页查询功能，支持按页码查询和游标查询两种方式。

### 练习 4：使用内存仓储测试

使用内存版本的订单仓储编写单元测试，验证用例的业务逻辑。

### 练习 5：实现仓储工厂

创建一个仓储工厂，根据配置动态创建不同实现的仓储实例。
