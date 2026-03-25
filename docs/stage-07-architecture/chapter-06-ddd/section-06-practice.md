# 7.6.6 DDD 实践指南

## 概述

领域驱动设计（DDD）不仅仅是一套技术模式，更是一种软件开发方法论。成功实施 DDD 需要团队在思维方式、工作流程、组织结构等方面做出改变。本节将总结 DDD 的实施步骤、领域建模方法、代码组织实践，以及团队协作模式，帮助零基础学员将 DDD 真正应用到实际项目中。

DDD 的实施是一个渐进的过程，不可能一蹴而就。团队需要从理解 DDD 的核心概念开始，然后在项目中逐步实践，从简单场景开始，逐步应用到更复杂的领域。在实施过程中，团队会不断深化对业务的理解，领域模型也会随之演进。

成功的 DDD 实施需要技术团队与业务团队的紧密合作。技术人员需要深入理解业务，业务人员需要理解技术实现的约束。通过建立通用语言，团队成员可以使用共同的术语进行沟通，减少误解和沟通成本。

**主要内容**：
- DDD 实施步骤
- 领域建模方法
- 代码组织实践
- 团队协作模式
- 常见陷阱和解决方案
- 完整示例

## 特性

- **渐进式实施**：从简单场景开始，逐步推进
- **业务导向**：以业务需求为核心
- **团队协作**：技术团队与业务团队紧密合作
- **持续演进**：领域模型随着理解深入而演进
- **实践验证**：通过代码和测试验证模型

## 核心概念

### DDD 实施步骤

DDD 的实施可以分为以下几个步骤：

**第一步：了解业务领域**

在开始建模之前，团队需要深入了解业务领域。这包括与业务专家交流、阅读业务文档、了解业务流程、识别业务痛点等。团队应该花时间真正理解业务，而不是急于开始写代码。

**第二步：识别有界上下文**

识别系统中的有界上下文是战略设计的关键一步。有界上下文划定了领域模型的边界，不同的上下文可能有不同的领域模型。识别有界上下文的方法包括：分析业务能力、识别不同的用户群体、观察不同的数据变更频率等。

**第三步：建立通用语言**

在每个有界上下文中，团队需要建立通用语言。通用语言是团队成员（开发人员、产品经理、业务专家）共同使用的术语体系。所有的代码、文档、沟通都应该使用通用语言。

**第四步：进行领域建模**

在通用语言的基础上，团队开始进行领域建模。领域建模包括：识别实体和值对象、设计聚合、定义领域服务、规划领域事件等。领域模型应该准确反映业务概念和业务规则。

**第五步：实现代码**

根据领域模型编写代码。代码应该遵循 DDD 的分层架构：领域层包含领域模型，应用层包含用例，基础设施层包含技术实现。

**第六步：验证和演进**

通过测试和实际使用验证领域模型是否正确反映业务需求。根据反馈不断调整和改进领域模型。领域模型不是一次性设计完成的，而是持续演进的过程。

### 领域建模方法

领域建模是 DDD 的核心活动。以下是一些常用的领域建模方法：

**事件风暴（Event Storming）**

事件风暴是一种快速识别领域事件的建模方法。参与者在白板上贴上代表事件的卡片，按照时间顺序排列。通过识别领域事件，可以发现实体、聚合和边界。

**四色建模（Color Modeling）**

四色建模使用四种不同颜色的贴纸表示不同的建模元素：
- 红色（Moment-Interval）：表示时刻或时间段，如订单、合同
- 黄色（Role）：表示角色，如用户、员工
- 绿色（Description）：表示描述，如产品分类
- 蓝色（Place）：表示地点，如仓库、门店

**用例分析**

通过分析系统的用例，识别每个用例涉及的实体和业务规则。用例提供了业务需求的清晰描述，是领域建模的重要输入。

### 代码组织实践

以下是推荐的 DDD 项目目录结构：

```
src/
├── Domain/                          # 领域层
│   ├── Entity/                      # 实体
│   │   └── Order.php
│   ├── ValueObject/                 # 值对象
│   │   └── Money.php
│   ├── Aggregate/                   # 聚合
│   │   └── OrderAggregate.php
│   ├── Service/                     # 领域服务
│   │   └── TransferService.php
│   ├── Event/                       # 领域事件
│   │   └── OrderCreatedEvent.php
│   ├── Repository/                  # 仓储接口
│   │   └── OrderRepositoryPort.php
│   └── Exception/                   # 领域异常
│       └── OrderException.php
│
├── Application/                     # 应用层
│   ├── Command/                     # 命令对象
│   │   └── CreateOrderCommand.php
│   ├── Query/                       # 查询对象
│   │   └── GetOrderQuery.php
│   ├── Dto/                         # 数据传输对象
│   │   └── OrderDto.php
│   ├── UseCase/                     # 用例
│   │   └── CreateOrderUseCase.php
│   ├── Handler/                     # 事件处理器
│   │   └── OrderEventHandler.php
│   └── Service/                     # 应用服务
│       └── OrderApplicationService.php
│
├── Port/                            # 端口层
│   ├── Input/                       # 输入端口
│   │   └── OrderInputPort.php
│   └── Output/                      # 输出端口
│       └── OrderOutputPort.php
│
├── Adapter/                         # 适配器层
│   ├── Input/                       # 输入适配器
│   │   ├── Http/
│   │   │   └── OrderController.php
│   │   └── Cli/
│   │       └── OrderCommand.php
│   └── Output/                      # 输出适配器
│       ├── Database/
│       │   └── MySqlOrderRepository.php
│       ├── Cache/
│       │   └── RedisOrderRepository.php
│       └── Notification/
│           └── EmailAdapter.php
│
└── Infrastructure/                 # 基础设施层
    ├── Bootstrap/
    │   └── Application.php
    ├── Config/
    │   └── config.php
    └── Logger/
        └── ApplicationLogger.php
```

这种目录结构清晰地分离了不同层次的代码，便于团队成员理解代码结构，也便于代码的维护和演进。

## 基本用法

### 完整的 DDD 项目示例

以下是一个完整的 DDD 订单管理系统的实现示例：

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
    private \DateTimeImmutable $createdAt;
    private \DateTimeImmutable $updatedAt;
    private array $domainEvents = [];

    public function __construct(
        private readonly int $customerId,
        private readonly string $customerEmail
    ) {
        $this->status = OrderStatus::PENDING;
        $this->totalAmount = new Money(0, 'CNY');
        $this->createdAt = new \DateTimeImmutable();
        $this->updatedAt = new \DateTimeImmutable();
        
        $this->addDomainEvent(new \App\Domain\Event\OrderCreatedEvent(
            orderId: 0,
            customerId: $this->customerId,
            customerEmail: $this->customerEmail,
            createdAt: $this->createdAt
        ));
    }

    public function addItem(OrderItem $item): void
    {
        if ($this->status !== OrderStatus::PENDING) {
            throw new \DomainException('Cannot add items to a non-pending order');
        }

        $this->items[] = $item;
        $this->recalculateTotal();
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function confirm(): void
    {
        if ($this->status !== OrderStatus::PENDING) {
            throw new \DomainException('Only pending orders can be confirmed');
        }

        $this->status = OrderStatus::CONFIRMED;
        $this->updatedAt = new \DateTimeImmutable();

        $this->addDomainEvent(new \App\Domain\Event\OrderConfirmedEvent(
            orderId: $this->id,
            customerId: $this->customerId,
            confirmedAt: $this->updatedAt
        ));
    }

    public function cancel(): void
    {
        if (!in_array($this->status, [OrderStatus::PENDING, OrderStatus::CONFIRMED])) {
            throw new \DomainException('Order cannot be cancelled');
        }

        $this->status = OrderStatus::CANCELLED;
        $this->updatedAt = new \DateTimeImmutable();

        $this->addDomainEvent(new \App\Domain\Event\OrderCancelledEvent(
            orderId: $this->id,
            customerId: $this->customerId,
            cancelledAt: $this->updatedAt
        ));
    }

    private function recalculateTotal(): void
    {
        $total = 0;
        foreach ($this->items as $item) {
            $total += $item->getSubtotal()->getAmount();
        }
        $this->totalAmount = new Money($total, $this->totalAmount->getCurrency());
    }

    public function pullDomainEvents(): array
    {
        $events = $this->domainEvents;
        $this->domainEvents = [];
        return $events;
    }

    private function addDomainEvent(\App\Domain\Event\DomainEvent $event): void
    {
        $this->domainEvents[] = $event;
    }

    public function getId(): ?int { return $this->id; }
    public function setId(int $id): void { $this->id = $id; }
    public function getCustomerId(): int { return $this->customerId; }
    public function getCustomerEmail(): string { return $this->customerEmail; }
    public function getStatus(): OrderStatus { return $this->status; }
    public function getItems(): array { return $this->items; }
    public function getTotalAmount(): Money { return $this->totalAmount; }
    public function getCreatedAt(): \DateTimeImmutable { return $this->createdAt; }
}
```

```php
<?php
declare(strict_types=1);

namespace App\Port\Output;

use App\Domain\Entity\Order;

interface OrderRepositoryPort
{
    public function findById(int $id): ?Order;
    public function findByCustomerId(int $customerId): array;
    public function save(Order $order): int;
    public function update(Order $order): void;
    public function delete(int $id): void;
}
```

```php
<?php
declare(strict_types=1);

namespace App\Application\UseCase;

use App\Port\Output\OrderRepositoryPort;
use App\Port\Output\DomainEventPublisherPort;
use App\Application\Dto\CreateOrderDto;
use App\Domain\Entity\Order;
use App\Domain\Entity\OrderItem;
use App\Domain\ValueObject\Money;

class CreateOrderUseCase
{
    public function __construct(
        private readonly OrderRepositoryPort $orderRepository,
        private readonly DomainEventPublisherPort $eventPublisher
    ) {}

    public function execute(CreateOrderDto $dto): Order
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

        $events = $order->pullDomainEvents();
        foreach ($events as $event) {
            $this->eventPublisher->publish($event);
        }

        return $this->orderRepository->findById($orderId);
    }
}
```

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Input\Http;

use App\Application\UseCase\CreateOrderUseCase;
use App\Application\Dto\CreateOrderDto;
use App\Application\Dto\OrderItemDto;

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

        $order = $this->createOrderUseCase->execute($dto);

        return [
            'status' => 'success',
            'data' => [
                'order_id' => $order->getId(),
                'status' => $order->getStatus()->value,
                'total_amount' => $order->getTotalAmount()->getAmount()
            ]
        ];
    }
}
```

## 使用场景

### 何时使用 DDD

- 复杂业务逻辑的企业应用
- 需要长期维护和演进的项目
- 多个团队协作开发的大型系统
- 需要频繁适应业务变化的系统

### 何时不使用 DDD

- 简单的 CRUD 应用
- 生命周期较短的项目
- 业务逻辑简单的系统
- 团队对 DDD 缺乏经验的初期

## 注意事项

### 1. 避免过度工程

DDD 是一种复杂的架构方法论，不适用于所有项目。对于简单的项目，使用 DDD 可能带来不必要的复杂性。应该根据项目的实际需求选择架构。

### 2. 渐进式采用

不需要一次性将整个系统重构为 DDD。可以从新功能开始，逐步引入 DDD 的实践方法。

### 3. 领域模型不是数据库模型

领域模型反映业务概念，数据库模型反映存储需求。两者可能有很大的不同，不要将领域模型和数据库模型混为一谈。

### 4. 持续学习

DDD 需要团队持续学习和实践。可以通过阅读书籍、参加研讨会、在项目中实践来提高团队能力。

## 常见问题

### Q1: DDD 需要多长时间才能掌握？

DDD 的学习曲线比较陡峭，团队通常需要几个月的时间才能基本掌握。精通 DDD 需要多年的实践经验。

### Q2: DDD 与敏捷开发冲突吗？

不冲突。DDD 与敏捷开发方法是互补的。两者都强调迭代、反馈和持续改进。

### Q3: 如何说服团队采用 DDD？

可以从一个小的试点项目开始，展示 DDD 的价值。也可以先在团队中培养 DDD 的学习氛围，让成员自愿尝试。

### Q4: DDD 适合微服务吗？

是的，DDD 与微服务架构是天然的组合。每个微服务可以作为一个有界上下文，独立开发、部署和扩展。

## 最佳实践

### 1. 从核心领域开始

不要试图一次性建模整个系统。从最核心、最复杂的业务领域开始，逐步扩展。

### 2. 建立通用语言

在团队中建立并使用通用语言。确保代码、文档、沟通都使用一致的术语。

### 3. 保持领域层纯净

领域层应该只包含业务逻辑，不应该依赖应用层或基础设施层。使用依赖注入来管理依赖。

### 4. 通过测试验证模型

为领域模型编写单元测试。测试可以验证业务规则的正确性，也可以帮助发现模型的问题。

### 5. 持续重构

领域模型需要持续重构和优化。随着对业务的深入理解，不断改进模型的结构和命名。

## 练习任务

### 练习 1：分析业务领域

选择一个熟悉的业务系统（如在线教育、图书管理），进行领域分析，识别有界上下文，建立通用语言。

### 练习 2：设计领域模型

为练习 1 中的核心领域设计完整的领域模型，包括实体、值对象、聚合、领域服务、领域事件。

### 练习 3：实现完整项目

使用 DDD 架构实现一个简单的订单管理系统，包括完整的代码实现和测试。

### 练习 4：组织代码审查

组织一次代码审查，检查代码是否符合 DDD 的原则，找出改进点。

### 练习 5：评估架构选择

分析一个现有项目，评估是否适合使用 DDD，并提出重构建议。
