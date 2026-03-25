# 7.6.1 DDD 概述与分层

## 概述

领域驱动设计（Domain-Driven Design，简称 DDD）是一种以业务领域为核心的软件设计方法论。DDD 强调软件开发应该围绕业务领域的核心概念展开，通过建立通用语言（Ubiquitous Language）来连接技术团队与业务专家，使软件模型能够准确反映业务需求。

DDD 的核心思想是将业务领域的专业知识和规则融入到软件设计中，而不是让业务适应技术的限制。这种方法特别适用于复杂业务逻辑的企业级应用，能够有效降低业务需求与技术实现之间的鸿沟，提高软件的可维护性和可扩展性。

Eric Evans 在 2003 年出版的《领域驱动设计：软件核心复杂性应对之道》一书中首次系统性地提出了 DDD 的概念和实践方法。如今，DDD 已成为企业级软件开发中最具影响力的架构方法论之一，被广泛应用于金融、电商、物流等行业的大型系统中。

**主要内容**：
- DDD 的基本概念和核心思想
- 通用语言的建立
- 领域模型的设计原则
- DDD 分层架构详解
- 各层之间的依赖关系
- 完整的代码示例

## 特性

- **业务导向**：以业务需求为核心，而非技术实现
- **通用语言**：建立开发团队与业务专家共同使用的语言
- **领域模型**：用软件模型准确表达业务概念
- **边界清晰**：通过有界上下文划分领域边界
- **持续演进**：领域模型随着对业务的深入理解而不断演进

## 核心概念

### 领域驱动设计的基本概念

领域驱动设计的核心目标是将软件系统的核心业务逻辑与底层技术实现解耦，使软件能够准确反映业务领域的复杂性和规则。DDD 不是一种具体的技术或框架，而是一种设计思想和方法论，需要开发团队在项目实践中不断深化理解。

在 DDD 中，"领域"指的是软件所解决的业务问题域。例如，一个电子商务系统的领域包括产品目录、订单管理、用户账户、物流配送等子领域。每个子领域都有自己的业务规则和概念，需要用专门的领域模型来表达。

通用语言是 DDD 的另一个核心概念。在软件开发过程中，开发人员与业务专家通常使用不同的术语和表达方式，这种沟通障碍往往导致需求的误解和软件的缺陷。DDD 强调建立一套团队成员共同理解的术语体系，确保每个人谈论的都是同一回事。

### 战略设计： bounded context（有界上下文）

有界上下文是 DDD 战略设计的核心概念，用于划分系统的业务边界。在大型系统中，不同的业务领域可能有不同的模型和规则，有界上下文帮助我们明确每个模型的适用范围，避免概念的混淆和模型的膨胀。

例如，在一个电商系统中，用户模块和产品模块可能由不同的团队负责，它们各自有自己的用户模型和产品模型。虽然都涉及"用户"概念，但在用户模块中，用户是一个实体，包含登录、权限等属性；在产品模块中，用户可能是购买者的角色。这种情况下，我们应该为两个模块分别建立有界上下文，在各自的上下文中定义清晰的领域模型。

有界上下文之间通过上下文映射（Context Mapping）来建立联系。常见的映射关系包括：共享内核（Shared Kernel）、客户-供应商（Customer-Supplier）、防腐层（Anti-Corruption Layer）等。这些映射关系帮助我们在保持模型独立性的同时，实现上下文之间的协作。

### 战术设计： building blocks（构造块）

DDD 提供了丰富的战术设计工具，帮助我们在有界上下文内部建立领域模型。这些构造块包括：

实体（Entity）是指具有唯一标识的对象，其身份在生命周期内保持不变。例如，用户订单是一个实体，即使订单的所有属性都发生了变化，它仍然是同一个订单。实体通常有自己的一套业务规则和行为方法。

值对象（Value Object）是指没有唯一标识、仅通过属性值来识别的对象。例如，地址是一个值对象，如果两个地址的省市区、街道门牌号都相同，它们就是相等的。值对象通常是不可变的，用于描述实体的某些特征。

聚合（Aggregate）是一组相关联的实体和值对象的组合，作为数据修改的单元。聚合根（Aggregate Root）是聚合的入口点，所有对聚合内部对象的修改都必须通过聚合根来进行，确保聚合内部的一致性不变性。

领域服务（Domain Service）用于封装不属于单个实体或值对象的业务逻辑。当某个业务操作需要多个领域对象协作完成时，可以将其放在领域服务中。

仓储（Repository）提供领域模型的持久化机制，封装了对数据库等存储介质的访问细节。仓储接口定义在领域层，具体实现放在基础设施层。

领域事件（Domain Event）用于表达领域中发生的业务事件，如订单创建、用户注册等。领域事件可以帮助解耦系统内部的不同模块，实现事件驱动的架构。

## 语法/定义

### 分层架构

DDD 推荐的经典分层架构将系统分为四个主要层次：表示层、应用层、领域层和基础设施层。这种分层方式遵循了依赖倒置原则，使得核心业务逻辑不依赖于外部技术实现。

```
┌─────────────────────────────────────────────────────────────┐
│                      表示层 (Presentation)                    │
│    HTTP Controllers, CLI Commands, Event Handlers          │
└────────────────────────────┬────────────────────────────────┘
                             │ 依赖
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                      应用层 (Application)                     │
│    Use Cases, Application Services, DTOs                    │
└────────────────────────────┬────────────────────────────────┘
                             │ 依赖
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                      领域层 (Domain)                          │
│    Entities, Value Objects, Domain Services, Events         │
└────────────────────────────┬────────────────────────────────┘
                             │ 被依赖（依赖倒置）
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                    基础设施层 (Infrastructure)                 │
│    Repositories, External Services, Frameworks              │
└─────────────────────────────────────────────────────────────┘
```

## 基本用法

### 1. 领域层实现

领域层是整个系统的核心，包含所有的业务逻辑和领域模型。领域层的代码应该独立于任何技术框架，只依赖于 PHP 语言本身。

```php
<?php
declare(strict_types=1);

namespace App\Domain\Entity;

use App\Domain\ValueObject\Money;
use App\Domain\ValueObject\OrderStatus;

class Order
{
    private ?int $id = null;
    private OrderStatus $status;
    private array $items = [];
    private Money $totalAmount;

    public function __construct(
        private readonly int $customerId,
        private readonly string $customerEmail
    ) {
        $this->status = OrderStatus::PENDING;
        $this->totalAmount = new Money(0, 'CNY');
    }

    public function addItem(OrderItem $item): void
    {
        $this->items[] = $item;
        $this->recalculateTotal();
    }

    private function recalculateTotal(): void
    {
        $total = 0;
        foreach ($this->items as $item) {
            $total += $item->getSubtotal()->getAmount();
        }
        $this->totalAmount = new Money($total, $this->totalAmount->getCurrency());
    }

    public function confirm(): void
    {
        if ($this->status !== OrderStatus::PENDING) {
            throw new \DomainException('Only pending orders can be confirmed');
        }
        $this->status = OrderStatus::CONFIRMED;
    }

    public function cancel(): void
    {
        if (!in_array($this->status, [OrderStatus::PENDING, OrderStatus::CONFIRMED])) {
            throw new \DomainException('Order cannot be cancelled in current status');
        }
        $this->status = OrderStatus::CANCELLED;
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
}
```

领域实体封装了核心的业务规则。`Order` 实体的 `confirm()` 和 `cancel()` 方法不仅修改状态，还包含业务逻辑验证：只有待处理状态的订单才能确认，已确认和已完成的订单不能取消。这种将业务规则封装在领域实体中的做法，正是 DDD 的核心实践之一。

### 2. 值对象实现

值对象是 DDD 中用于表示没有唯一标识的概念的对象。值对象应该是不可变的，并且通过属性值来判定相等性。

```php
<?php
declare(strict_types=1);

namespace App\Domain\ValueObject;

class Money
{
    public function __construct(
        private readonly int $amount,
        private readonly string $currency
    ) {
        if ($amount < 0) {
            throw new \InvalidArgumentException('Amount cannot be negative');
        }
    }

    public function add(Money $other): Money
    {
        if ($this->currency !== $other->currency) {
            throw new \InvalidArgumentException('Currency mismatch');
        }
        return new Money($this->amount + $other->amount, $this->currency);
    }

    public function multiply(int $multiplier): Money
    {
        return new Money($this->amount * $multiplier, $this->currency);
    }

    public function getAmount(): int
    {
        return $this->amount;
    }

    public function getCurrency(): string
    {
        return $this->currency;
    }

    public function equals(Money $other): bool
    {
        return $this->amount === $other->amount && $this->currency === $other->currency;
    }

    public function __toString(): string
    {
        return sprintf('%s %d', $this->currency, $this->amount);
    }
}
```

`Money` 值对象封装了货币金额的概念，提供了加法、乘法等运算方法。值对象的不可变性通过 `readonly` 属性来实现，确保对象创建后其状态不可改变。

### 3. 应用层实现

应用层位于领域层之上，负责用例的编排和协调。应用层调用领域层的服务来完成具体的业务用例，但不包含业务逻辑。

```php
<?php
declare(strict_types=1);

namespace App\Application\Dto;

class CreateOrderDto
{
    public function __construct(
        public readonly int $customerId,
        public readonly string $customerEmail,
        public readonly array $items
    ) {}
}

class OrderItemDto
{
    public function __construct(
        public readonly string $productId,
        public readonly string $productName,
        public readonly int $quantity,
        public readonly int $unitPrice
    ) {}
}

class OrderResultDto
{
    public function __construct(
        public readonly int $orderId,
        public readonly string $status,
        public readonly int $totalAmount,
        public readonly string $currency
    ) {}
}
```

```php
<?php
declare(strict_types=1);

namespace App\Application\UseCase;

use App\Application\Dto\CreateOrderDto;
use App\Application\Dto\OrderResultDto;
use App\Domain\Entity\Order;
use App\Domain\Entity\OrderItem;
use App\Domain\ValueObject\Money;
use App\Port\Output\OrderRepositoryPort;
use App\Port\Output\NotificationPort;

class CreateOrderUseCase
{
    public function __construct(
        private readonly OrderRepositoryPort $orderRepository,
        private readonly NotificationPort $notification
    ) {}

    public function execute(CreateOrderDto $dto): OrderResultDto
    {
        $order = new Order(
            customerId: $dto->customerId,
            customerEmail: $dto->customerEmail
        );

        foreach ($dto->items as $itemDto) {
            $item = new OrderItem(
                productId: $itemDto->productId,
                productName: $itemDto->productName,
                quantity: $itemDto->quantity,
                unitPrice: new Money($itemDto->unitPrice, 'CNY')
            );
            $order->addItem($item);
        }

        $orderId = $this->orderRepository->save($order);
        
        $this->notification->sendOrderConfirmation($order);

        return new OrderResultDto(
            orderId: $orderId,
            status: $order->getStatus()->value,
            totalAmount: $order->getTotalAmount()->getAmount(),
            currency: $order->getTotalAmount()->getCurrency()
        );
    }
}
```

应用层用例的职责是：创建领域对象、调用领域服务完成业务操作、调用仓储持久化数据、触发后续通知等。应用层本身不包含业务逻辑，业务逻辑都封装在领域实体和领域服务中。

### 4. 基础设施层实现

基础设施层负责处理技术细节，如数据库访问、外部 API 调用、消息队列等。基础设施层的代码依赖于领域层定义的接口（端口）。

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\Database;

use App\Domain\Entity\Order;
use App\Port\Output\OrderRepositoryPort;
use PDO;

class OrderRepository implements OrderRepositoryPort
{
    public function __construct(private readonly PDO $pdo) {}

    public function findById(int $id): ?Order
    {
        $stmt = $this->pdo->prepare('SELECT * FROM orders WHERE id = ?');
        $stmt->execute([$id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);

        if ($row === false) {
            return null;
        }

        return $this->mapToOrder($row);
    }

    public function save(Order $order): int
    {
        $stmt = $this->pdo->prepare(
            'INSERT INTO orders (customer_id, customer_email, status, total_amount, currency, created_at) 
             VALUES (?, ?, ?, ?, ?, ?)'
        );

        $now = date('Y-m-d H:i:s');
        $stmt->execute([
            $order->getCustomerId(),
            $order->getCustomerEmail(),
            $order->getStatus()->value,
            $order->getTotalAmount()->getAmount(),
            $order->getTotalAmount()->getCurrency(),
            $now
        ]);

        return (int) $this->pdo->lastInsertId();
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

仓储实现位于基础设施层，实现了领域层定义的 `OrderRepositoryPort` 接口。这种设计使得领域层不依赖于具体的数据库技术，可以轻松切换不同的存储后端。

### 5. 表示层实现

表示层负责处理用户交互，将用户的请求传递给应用层，并将处理结果返回给用户。表示层可以是 HTTP 控制器、CLI 命令或消息队列消费者。

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Input\Http;

use App\Application\Dto\CreateOrderDto;
use App\Application\Dto\OrderItemDto;
use App\Application\UseCase\CreateOrderUseCase;

class OrderController
{
    public function __construct(
        private readonly CreateOrderUseCase $createOrderUseCase
    ) {}

    public function create(array $requestData): array
    {
        $items = array_map(
            fn($item) => new OrderItemDto(
                productId: $item['product_id'],
                productName: $item['product_name'],
                quantity: (int) $item['quantity'],
                unitPrice: (int) $item['unit_price']
            ),
            $requestData['items'] ?? []
        );

        $dto = new CreateOrderDto(
            customerId: (int) $requestData['customer_id'],
            customerEmail: $requestData['customer_email'],
            items: $items
        );

        $result = $this->createOrderUseCase->execute($dto);

        return [
            'status' => 'success',
            'data' => [
                'order_id' => $result->orderId,
                'status' => $result->status,
                'total_amount' => $result->totalAmount,
                'currency' => $result->currency
            ]
        ];
    }
}
```

表示层的控制器只负责处理请求数据的解析和响应数据的格式化，不包含任何业务逻辑。所有的业务处理都委托给应用层的用例来完成。

## 使用场景

### 1. 复杂业务逻辑系统

DDD 特别适用于业务逻辑复杂的系统，如金融交易系统、保险理赔系统、ERP 系统等。这些系统的业务规则复杂多变，需要用领域模型来准确表达业务概念。

### 2. 长期维护项目

对于需要长期维护和迭代的项目，DDD 可以帮助建立清晰的代码结构，使得后续的维护和扩展更加容易。领域模型的可测试性也降低了修改代码引入 bug 的风险。

### 3. 大型团队协作

在大型团队中，不同的团队负责不同的业务领域。有界上下文帮助团队划定工作边界，通用语言帮助团队成员高效沟通。

### 4. 微服务架构

微服务架构与 DDD 有天然的契合度。每个微服务可以作为一个有界上下文，独立开发、部署和扩展。DDD 提供了识别服务边界的方法论。

## 注意事项

### 1. 学习曲线

DDD 是一种相对复杂的设计方法论，需要开发团队投入时间学习相关概念和实践方法。在项目初期，可能需要额外的培训和学习成本。

### 2. 避免过度设计

DDD 不是万能的，对于简单的 CRUD 应用，使用 DDD 可能带来不必要的复杂性。应该根据项目的实际需求选择合适的架构。

### 3. 领域建模的重要性

DDD 的核心是领域模型，一个好的领域模型是 DDD 成功的关键。领域建模需要开发人员与业务专家的深入合作，需要多次迭代和完善。

### 4. 持续演进

领域模型不是一次性设计完成的，而是随着对业务的深入理解不断演进的。团队需要建立持续改进的机制，不断优化领域模型。

## 常见问题

### Q1: DDD 与 MVC 有什么区别？

MVC 是一种表现层架构模式，关注的是用户界面的组织方式。DDD 是一种整体的设计方法论，关注的是整个软件系统的设计和组织方式。两者可以结合使用：DDD 负责整体的领域建模和分层架构，MVC 负责表示层的组织。

### Q2: DDD 适合小项目吗？

DDD 带来的架构复杂度对于小项目可能过高。如果项目规模较小、业务逻辑简单，使用简单的三层架构可能更加合适。建议在项目规模较大、业务复杂时考虑使用 DDD。

### Q3: 如何开始 DDD 实践？

可以从以下步骤开始：与业务专家建立联系，了解核心业务概念；识别有界上下文，划定领域边界；在每个上下文中建立通用语言；逐步构建领域模型，从核心领域开始。

### Q4: DDD 与六边形架构的关系？

DDD 和六边形架构是互补的关系。六边形架构提供了一种实现 DDD 分层架构的技术方案，使得领域层可以独立于技术细节。两者都强调业务核心的重要性，强调依赖方向的控制。

## 最佳实践

### 1. 深入理解业务

DDD 的成功建立在对业务的深入理解之上。花时间与业务专家交流，阅读业务文档，参与业务讨论，建立对业务的全面认知。

### 2. 建立通用语言

在团队中建立通用语言，确保开发人员、产品经理、业务专家使用一致的术语。将通用语言体现在代码、文档、沟通的各个方面。

### 3. 从核心领域开始

不要试图一次性建立整个系统的领域模型。从最核心、最复杂的业务领域开始，逐步扩展到其他领域。

### 4. 保持领域层纯净

领域层应该只包含业务逻辑，不应该依赖于应用层或基础设施层。使用依赖注入和依赖倒置来保持领域层的独立性。

### 5. 持续重构

领域模型需要持续重构和优化。随着对业务的深入理解，不断改进领域模型的结构和命名，使其更加准确地表达业务概念。

## 练习任务

### 练习 1：分析业务领域

选择一个熟悉的业务系统（如在线图书馆、学生管理系统），分析其业务领域，识别核心领域和支持领域，绘制领域地图。

### 练习 2：设计领域模型

为练习 1 中的核心领域设计领域模型，包括实体、值对象、聚合的识别和定义。

### 练习 3：实现分层架构

使用 DDD 分层架构实现一个简单的订单管理模块，包括领域层、应用层、基础设施层和表示层的代码。

### 练习 4：识别有界上下文

分析一个电商系统，识别其有界上下文，定义各上下文之间的映射关系。

### 练习 5：编写领域测试

为实现的领域实体编写单元测试，验证业务规则的正确性。
