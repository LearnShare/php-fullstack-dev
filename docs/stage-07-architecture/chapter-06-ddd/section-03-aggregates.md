# 7.6.3 聚合根（Aggregate Root）

## 概述

聚合（Aggregate）是领域驱动设计中用于组织相关领域对象的模式，它定义了一组具有一致性边界的领域对象集合。聚合根（Aggregate Root）是聚合的入口点，是唯一一个外部可以直接引用的对象。所有的对聚合内部对象的修改都必须通过聚合根来进行，这种设计确保了聚合内部的一致性不变性。

聚合的概念源于对复杂业务场景的建模需求。在现实业务中，很多领域对象之间存在紧密的关联关系，它们共同构成一个整体，需要保持某些业务规则的一致性。例如，一个订单及其订单项就是一个典型的聚合：订单项不能独立于订单存在，订单总金额需要根据订单项计算，所有订单项的状态变化会影响订单的状态。

正确设计聚合是 DDD 建模的核心技能之一。好的聚合设计能够简化业务逻辑的实现，降低并发问题的复杂性，提高系统的性能。设计不当的聚合则可能导致业务逻辑分散、事务边界模糊、性能问题等。

**主要内容**：
- 聚合的概念和作用
- 聚合根的定义和职责
- 聚合边界的设计原则
- 聚合内部的一致性管理
- 聚合间的引用方式
- 完整的代码示例

## 特性

- **一致性边界**：聚合定义了一组需要保持一致性的对象边界
- **聚合根入口**：外部只能通过聚合根访问聚合内部的对象
- **不变式保护**：聚合根负责保护聚合内部的不变式规则
- **事务边界**：聚合通常作为事务的边界，一个事务只操作一个聚合
- **独立持久化**：每个聚合有独立的仓储进行持久化

## 核心概念

### 聚合的定义

聚合是一组相关联的领域对象，它们共同构成一个整体，有一个聚合根作为外部访问的入口。聚合内的对象可以相互引用，但外部世界只能看到聚合根，聚合内部的对象不能被外部直接引用或修改。

```
┌─────────────────────────────────────┐
│           Order (聚合根)             │
│  ┌─────────────────────────────┐   │
│  │ - id: OrderId               │   │
│  │ - customerId: int           │   │
│  │ - status: OrderStatus       │   │
│  │ - items: OrderItem[]        │◄──┼── 聚合内部实体
│  │ - totalAmount: Money        │   │
│  └─────────────────────────────┘   │
│         │                   │       │
│         │                   │       │
│    引用关系            引用关系     │
│         │                   │       │
│         ▼                   ▼       │
│  ┌────────────┐     ┌────────────┐   │
│  │OrderItem 1 │     │OrderItem 2 │   │
│  │- productId │     │- productId │   │
│  │- quantity  │     │- quantity  │   │
│  │- price     │     │- price     │   │
│  └────────────┘     └────────────┘   │
└─────────────────────────────────────┘

外部只能通过 Order 访问 OrderItem
OrderItem 不能被外部直接引用
```

### 聚合根的职责

聚合根作为聚合的守门人，承担着以下重要职责：

首先，聚合根负责维护聚合内部的不变式规则。不变式是指在聚合的整个生命周期中必须保持为真的业务规则。例如，订单的总金额必须等于所有订单项金额的总和，这个规则需要由聚合根来维护。

其次，聚合根控制对聚合内部对象的访问。外部代码不能直接创建或修改聚合内部的对象，只能通过聚合根提供的方法来操作聚合内部的状态。这种控制确保了所有对聚合的修改都经过聚合根的验证。

第三，聚合根负责协调聚合内部对象的业务逻辑。当需要修改聚合内部对象的状态时，聚合根应该提供相应的方法，而不是让外部代码直接修改内部对象的状态。

### 聚合边界的设计

确定聚合的边界是聚合设计的核心问题。一个好的聚合边界应该满足以下原则：

第一，聚合应该包含在业务操作中经常一起修改的对象。如果某些对象总是需要同时被修改，它们应该属于同一个聚合。

第二，聚合应该保护重要的业务不变式。将需要保持一致性的对象放在同一个聚合中，可以更容易地保护业务规则。

第三，聚合不宜过大。大的聚合会导致性能问题和并发控制的复杂性。建议将聚合设计得小而专注。

第四，考虑查询需求。频繁一起查询的对象通常应该放在同一个聚合中，但也要平衡修改需求和查询需求。

## 语法/定义

### 聚合根类定义

```php
<?php
declare(strict_types=1);

// 聚合根特征：
// - 有唯一标识
// - 包含对聚合内部对象的引用
// - 提供修改聚合状态的方法
// - 负责保护不变式
```

### 聚合内实体定义

```php
<?php
declare(strict_types=1);

// 聚合内实体特征：
// - 无独立对外的标识
// - 只能通过聚合根访问
// - 状态由聚合根管理
```

## 基本用法

### 1. 订单聚合实现

以下是一个完整的订单聚合实现，展示了聚合根如何保护聚合内部的一致性：

```php
<?php
declare(strict_types=1);

namespace App\Domain\Entity;

use App\Domain\ValueObject\Money;
use App\Domain\ValueObject\OrderStatus;
use App\Domain\Exception\OrderException;

class Order
{
    private ?int $id = null;
    private OrderStatus $status;
    private array $items = [];
    private Money $totalAmount;
    private \DateTimeImmutable $createdAt;
    private \DateTimeImmutable $updatedAt;

    public function __construct(
        private readonly int $customerId,
        private readonly string $customerEmail
    ) {
        $this->status = OrderStatus::PENDING;
        $this->totalAmount = new Money(0, 'CNY');
        $this->createdAt = new \DateTimeImmutable();
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function addItem(OrderItem $item): void
    {
        if ($this->status !== OrderStatus::PENDING) {
            throw new OrderException('Cannot add items to a non-pending order');
        }

        $this->items[] = $item;
        $this->recalculateTotal();
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function removeItem(int $itemIndex): void
    {
        if ($this->status !== OrderStatus::PENDING) {
            throw new OrderException('Cannot remove items from a non-pending order');
        }

        if (!isset($this->items[$itemIndex])) {
            throw new OrderException('Item not found');
        }

        unset($this->items[$itemIndex]);
        $this->items = array_values($this->items);
        $this->recalculateTotal();
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function confirm(): void
    {
        if (empty($this->items)) {
            throw new OrderException('Cannot confirm an empty order');
        }

        if ($this->status !== OrderStatus::PENDING) {
            throw new OrderException('Only pending orders can be confirmed');
        }

        $this->status = OrderStatus::CONFIRMED;
        $this->updatedAt = new \DateTimeImmutable();

        foreach ($this->items as $item) {
            $item->confirm();
        }
    }

    public function cancel(): void
    {
        if (!in_array($this->status, [OrderStatus::PENDING, OrderStatus::CONFIRMED])) {
            throw new OrderException('Order cannot be cancelled in current status');
        }

        $this->status = OrderStatus::CANCELLED;
        $this->updatedAt = new \DateTimeImmutable();

        foreach ($this->items as $item) {
            $item->cancel();
        }
    }

    private function recalculateTotal(): void
    {
        $total = 0;
        foreach ($this->items as $item) {
            $total += $item->getSubtotal()->getAmount();
        }
        $this->totalAmount = new Money($total, $this->totalAmount->getCurrency());
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

    public function getCustomerEmail(): string
    {
        return $this->customerEmail;
    }

    public function getStatus(): OrderStatus
    {
        return $this->status;
    }

    public function getItems(): array
    {
        return $this->items;
    }

    public function getTotalAmount(): Money
    {
        return $this->totalAmount;
    }

    public function getCreatedAt(): \DateTimeImmutable
    {
        return $this->createdAt;
    }

    public function getUpdatedAt(): \DateTimeImmutable
    {
        return $this->updatedAt;
    }
}
```

订单聚合根展示了多个重要的设计模式。首先，所有对订单项的添加和删除都通过聚合根的方法进行，这确保了订单状态的一致性。其次，聚合根在确认订单时会同步更新所有订单项的状态，这保持了聚合内部的一致性。第三，所有的业务规则验证都封装在聚合根的方法中，外部代码不能绕过这些规则。

### 2. 订单项实现

订单项作为聚合内的实体，其设计应该简单，不能独立存在：

```php
<?php
declare(strict_types=1);

namespace App\Domain\Entity;

use App\Domain\ValueObject\Money;
use App\Domain\ValueObject\OrderItemStatus;

class OrderItem
{
    private OrderItemStatus $status;

    public function __construct(
        private readonly string $productId,
        private readonly string $productName,
        private readonly int $quantity,
        private readonly Money $unitPrice
    ) {
        $this->status = OrderItemStatus::PENDING;
    }

    public function confirm(): void
    {
        if ($this->status !== OrderItemStatus::PENDING) {
            throw new \DomainException('Cannot confirm a non-pending item');
        }
        $this->status = OrderItemStatus::CONFIRMED;
    }

    public function cancel(): void
    {
        if ($this->status === OrderItemStatus::CANCELLED) {
            throw new \DomainException('Item is already cancelled');
        }
        $this->status = OrderItemStatus::CANCELLED;
    }

    public function getSubtotal(): Money
    {
        return $this->unitPrice->multiply($this->quantity);
    }

    public function getProductId(): string
    {
        return $this->productId;
    }

    public function getProductName(): string
    {
        return $this->productName;
    }

    public function getQuantity(): int
    {
        return $this->quantity;
    }

    public function getUnitPrice(): Money
    {
        return $this->unitPrice;
    }

    public function getStatus(): OrderItemStatus
    {
        return $this->status;
    }
}
```

订单项作为聚合内的实体，其状态由聚合根管理。外部代码不能直接创建订单项，必须通过订单聚合根的 `addItem` 方法添加。这种设计确保了订单项始终属于一个有效的订单。

### 3. 聚合引用设计

聚合之间的引用应该通过聚合根的标识来实现，而不是直接引用聚合根对象：

```php
<?php
declare(strict_types=1);

namespace App\Domain\ValueObject;

class CustomerId
{
    public function __construct(private readonly int $value) {}

    public function getValue(): int
    {
        return $this->value;
    }

    public function equals(CustomerId $other): bool
    {
        return $this->value === $other->value;
    }
}
```

```php
<?php
declare(strict_types=1);

namespace App\Domain\Entity;

use App\Domain\ValueObject\CustomerId;

class Order
{
    public function __construct(
        private readonly CustomerId $customerId,
        // 其他属性...
    ) {}

    public function getCustomerId(): CustomerId
    {
        return $this->customerId;
    }
}
```

通过值对象包装标识符而不是直接使用整数或字符串，可以提供更好的类型安全性，并且可以在标识符层面添加验证逻辑。

## 使用场景

### 1. 订单管理系统

订单及其订单项是最经典的聚合示例。订单作为聚合根，订单项作为聚合内的实体。订单的状态变化需要同步到所有订单项，这正体现了聚合保护不变式的价值。

### 2. 购物车

购物车及其购物车项构成另一个典型的聚合。购物车项不能独立于购物车存在，购物车的总金额需要根据所有购物车项计算。

### 3. 文档编辑

一个文档及其段落、格式信息可以构成一个聚合。文档的发布状态需要检查所有段落的状态。

## 注意事项

### 1. 聚合大小控制

聚合不宜过大。过大的聚合会导致性能问题和并发控制的复杂性。建议将聚合设计得小而专注，每个聚合只包含在业务上必须一起修改的对象。

### 2. 事务边界

每个聚合应该作为独立的事务边界。在一个事务中只修改一个聚合，避免跨聚合的分布式事务。如果业务需要修改多个聚合，考虑使用事件驱动的最终一致性。

### 3. 聚合内部引用

聚合内部的对象可以相互引用，但这种引用应该谨慎使用。避免形成循环引用，也避免让聚合内部对象持有聚合根的引用。

### 4. 聚合与仓储

每个聚合有独立的仓储进行持久化。仓储的接口应该以聚合根为参数，这样简化了仓储的实现，也明确了操作的边界。

## 常见问题

### Q1: 聚合和实体有什么区别？

实体是具有唯一标识的对象，聚合是由多个相关对象组成的整体，聚合根是聚合中唯一对外可见的实体。聚合是一种组织实体的模式，而实体是领域模型的基本构建块。

### Q2: 一个聚合可以包含多个实体吗？

可以。一个聚合可以包含一个或多个实体，以及多个值对象。关键是要确定哪些对象必须作为整体一起修改，它们就属于同一个聚合。

### Q3: 如何处理跨聚合的引用？

跨聚合的引用应该通过标识来实现，而不是直接的对象引用。如果需要获取另一个聚合的信息，可以通过查询服务或事件机制来实现。

### Q4: 聚合内部的对象可以直接被外部访问吗？

不应该。聚合内部的对象只能通过聚合根暴露的方法来访问，外部代码不应该直接访问聚合内部的实体或值对象。

## 最佳实践

### 1. 设计小聚合

尽量设计小的聚合，只包含在业务上必须一起修改的对象。小的聚合更容易维护，也更容易实现并发控制。

### 2. 通过标识引用其他聚合

使用聚合根的标识来引用其他聚合，而不是直接引用聚合根对象。这样可以降低聚合之间的耦合度。

### 3. 在聚合根中保护不变式

将所有需要保护的不变式规则放在聚合根的方法中实现。确保所有的状态修改都通过聚合根的方法进行。

### 4. 使用领域事件记录状态变化

使用领域事件来记录聚合内部的状态变化，这样可以解耦聚合之间的依赖，实现事件驱动的架构。

### 5. 为每个聚合创建独立的仓储

每个聚合应该有独立的仓储接口和实现。仓储的接口应该以聚合根为参数，这样使得接口更加清晰。

## 练习任务

### 练习 1：设计产品聚合

为一个电商系统设计产品聚合。考虑产品的主要属性、产品的价格策略、产品与分类的关系。产品聚合应该包含哪些实体和值对象？

### 练习 2：实现购物车聚合

实现一个购物车聚合，包括购物车根实体和购物车项实体。实现添加商品、移除商品、清空购物车等功能。

### 练习 3：重构现有代码

分析一个现有的订单处理代码，识别其中的聚合。将代码重构为遵循聚合设计原则的形式。

### 练习 4：设计聚合边界

为一个学生管理系统设计聚合。识别学生、课程、成绩、班级等实体之间的关系，确定聚合边界。解释为什么这样划分。

### 练习 5：处理跨聚合引用

设计一个订单和客户之间的关系。订单需要引用客户，但订单和客户应该是两个独立的聚合。说明如何实现跨聚合引用。
