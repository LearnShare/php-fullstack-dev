# 7.7.4 数据流设计实践

## 概述

数据流设计模式（ CQRS、事件溯源、事件总线）是现代分布式系统中最重要的架构模式之一。这些模式可以单独使用，也可以组合使用，形成强大的事件驱动架构。本节将总结这些模式的组合使用策略、实践方法和注意事项，帮助零基础学员在实际项目中应用这些模式。

CQRS、事件溯源和事件总线这三个模式相互补充：CQRS 将读操作和写操作分离，允许分别优化；事件溯源提供了完整的状态历史记录，支持审计和时间回溯；事件总线实现了组件间的松耦合通信。当这三个模式组合使用时，可以构建一个高度可扩展、可维护的事件驱动系统。

然而，这些模式的组合也带来了额外的复杂性。在决定使用这些模式之前，需要仔细评估项目的实际需求，确保这些复杂性是值得的。对于小型项目或简单场景，使用传统的 CRUD 方式可能更加合适。

**主要内容**：
- CQRS 与事件溯源的组合使用
- 事件总线集成策略
- 渐进式实施方法
- 模式选择指南
- 完整示例

## 特性

### 模式协同

- **CQRS + 事件溯源**：写操作产生事件，存储到事件存储；读操作通过投影事件构建视图
- **事件总线集成**：事件溯源产生的事件通过事件总线分发给各个处理器
- **完整的数据流**：从命令到事件，从事件到投影，形成完整的数据流

### 组合优势

- **可扩展性**：读模型和写模型可以独立扩展
- **可维护性**：清晰的职责分离，代码更易维护
- **可审计性**：完整的事件历史，支持任意时间点的状态回溯
- **灵活性**：通过事件可以轻松添加新的功能

## 核心概念

### 完整的事件驱动架构

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                              客户端请求                                       │
└────────────────────────────────┬─────────────────────────────────────────────┘
                                 │
                                 ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                           命令端 (Command Side)                               │
│  ┌────────────────┐    ┌────────────────┐    ┌────────────────┐            │
│  │   Command      │───▶│   Command      │───▶│    Aggregate   │            │
│  │   Handler      │    │   Validator   │    │   (Entity)    │            │
│  └────────────────┘    └────────────────┘    └───────┬────────┘            │
│                                                       │                     │
│                                                       ▼                     │
│                                                ┌────────────────┐            │
│                                                │    Domain      │            │
│                                                │    Events      │            │
│                                                └───────┬────────┘            │
└───────────────────────────────────────────────────────┼─────────────────────┘
                                                        │
                                                        ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                          事件存储 (Event Store)                              │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │  EventStore: append(), getEvents(), getEventsFromVersion()        │     │
│  └────────────────────────────────────────────────────────────────────┘     │
└───────────────────────────────────────────────────────┬─────────────────────┘
                                                        │
                                                        ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                           事件总线 (Event Bus)                               │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │  EventBus: publish(), subscribe(), handlers                       │     │
│  └────────────────────────────────────────────────────────────────────┘     │
└────────────────────────────────┬─────────────────────────────────────────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   读模型投影     │    │   通知服务       │    │   审计日志       │
│ (Projection 1)  │    │ (Notification)  │    │  (Audit Log)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │
         ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                           查询端 (Query Side)                                │
│  ┌────────────────┐    ┌────────────────┐    ┌────────────────┐            │
│  │   Query        │───▶│   Query        │───▶│    Read         │            │
│  │   Handler      │    │   Validator   │    │    Model        │            │
│  └────────────────┘    └────────────────┘    └────────────────┘            │
└──────────────────────────────────────────────────────────────────────────────┘
```

## 基本用法

### 1. 完整的命令处理流程

```php
<?php
declare(strict_types=1);

namespace App\Application\Command;

use App\Application\Command\CreateOrderCommand;
use App\Domain\Entity\Order;
use App\Domain\ValueObject\Money;
use App\Port\Output\EventStorePort;
use App\Port\Output\EventBusPort;
use App\Port\Output\OrderRepositoryPort;

class CreateOrderHandler
{
    public function __construct(
        private readonly OrderRepositoryPort $orderRepository,
        private readonly EventStorePort $eventStore,
        private readonly EventBusPort $eventBus
    ) {}

    public function handle(CreateOrderCommand $command): int
    {
        $order = Order::create(
            customerId: $command->customerId,
            customerEmail: $command->customerEmail
        );

        foreach ($command->items as $itemData) {
            $order->addItem(new OrderItem(
                productId: $itemData['product_id'],
                productName: $itemData['product_name'],
                quantity: $itemData['quantity'],
                unitPrice: new Money($itemData['unit_price'], 'CNY')
            ));
        }

        $events = $order->pullDomainEvents();
        
        foreach ($events as $event) {
            $this->eventStore->append($order->getId(), $event);
        }

        $orderId = $this->orderRepository->save($order);

        foreach ($events as $event) {
            $this->eventBus->publish($event);
        }

        return $orderId;
    }
}
```

### 2. 读模型投影实现

```php
<?php
declare(strict_types=1);

namespace App\Infrastructure\Projection;

use App\Domain\Event\OrderCreatedEvent;
use App\Domain\Event\OrderConfirmedEvent;
use App\Domain\Event\OrderCancelledEvent;
use App\Port\Output\EventHandlerPort;
use App\Adapter\Output\Projection\OrderReadModel;

class OrderProjection implements EventHandlerPort
{
    public function __construct(
        private readonly OrderReadModel $readModel
    ) {}

    public function handle(object $event): void
    {
        match (true) {
            $event instanceof OrderCreatedEvent => $this->handleOrderCreated($event),
            $event instanceof OrderConfirmedEvent => $this->handleOrderConfirmed($event),
            $event instanceof OrderCancelledEvent => $this->handleOrderCancelled($event),
            default => null
        };
    }

    private function handleOrderCreated(OrderCreatedEvent $event): void
    {
        $this->readModel->create([
            'id' => $event->orderId,
            'customer_id' => $event->customerId,
            'customer_email' => $event->customerEmail,
            'status' => 'pending',
            'total_amount' => 0,
            'created_at' => $event->getOccurredAt()->format('Y-m-d H:i:s')
        ]);
    }

    private function handleOrderConfirmed(OrderConfirmedEvent $event): void
    {
        $this->readModel->update($event->orderId, [
            'status' => 'confirmed',
            'confirmed_at' => $event->getOccurredAt()->format('Y-m-d H:i:s')
        ]);
    }

    private function handleOrderCancelled(OrderCancelledEvent $event): void
    {
        $this->readModel->update($event->orderId, [
            'status' => 'cancelled',
            'cancelled_at' => $event->getOccurredAt()->format('Y-m-d H:i:s')
        ]);
    }
}
```

### 3. 事件溯源仓储

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\Repository;

use App\Domain\Entity\Order;
use App\Port\Output\OrderRepositoryPort;
use App\Port\Output\EventStorePort;
use App\Port\Output\SnapshotStorePort;

class EventSourcingOrderRepository implements OrderRepositoryPort
{
    private const SNAPSHOT_INTERVAL = 50;

    public function __construct(
        private readonly EventStorePort $eventStore,
        private readonly SnapshotStorePort $snapshotStore
    ) {}

    public function save(Order $order): int
    {
        $events = $order->pullDomainEvents();
        
        foreach ($events as $event) {
            $this->eventStore->append($order->getId(), $event);
        }

        $version = $order->getVersion();
        if ($version % self::SNAPSHOT_INTERVAL === 0) {
            $this->snapshotStore->save($order);
        }

        return $order->getId();
    }

    public function findById(int $id): ?Order
    {
        $snapshot = $this->snapshotStore->find($id);
        
        if ($snapshot !== null) {
            $events = $this->eventStore->getEventsFromVersion($id, $snapshot->getVersion());
            foreach ($events as $event) {
                $snapshot->apply($event);
            }
            return $snapshot;
        }

        $events = $this->eventStore->getEvents($id);
        
        if (empty($events)) {
            return null;
        }

        return Order::fromHistory($events);
    }

    public function findByCustomerId(int $customerId): array
    {
        $eventStream = $this->eventStore->getEventsByAggregateType(Order::class);
        
        $orders = [];
        $currentOrder = null;
        
        foreach ($eventStream as $event) {
            if ($event instanceof OrderCreatedEvent && $event->customerId === $customerId) {
                $currentOrder = Order::fromHistory([$event]);
            } elseif ($currentOrder !== null && $event->orderId === $currentOrder->getId()) {
                $currentOrder->apply($event);
            }
            
            if ($currentOrder !== null && $currentOrder->getId() !== null) {
                $orders[$currentOrder->getId()] = $currentOrder;
                $currentOrder = null;
            }
        }

        return array_values($orders);
    }
}
```

### 4. 应用启动配置

```php
<?php
declare(strict_types=1);

namespace App\Infrastructure\Bootstrap;

use App\Adapter\Output\EventBus\SynchronousEventBus;
use App\Adapter\Output\EventStore\MysqlEventStore;
use App\Adapter\Output\Snapshot\MysqlSnapshotStore;
use App\Adapter\Output\Repository\EventSourcingOrderRepository;
use App\Application\Command\CreateOrderHandler;
use App\Application\Handler\OrderEventHandler;
use App\Infrastructure\Projection\OrderProjection;
use App\Domain\Event\OrderCreatedEvent;
use App\Domain\Event\OrderConfirmedEvent;
use App\Domain\Event\OrderCancelledEvent;

class ApplicationBootstrap
{
    public static function bootstrap(array $config): array
    {
        $pdo = new \PDO(
            $config['database']['dsn'],
            $config['database']['username'],
            $config['database']['password']
        );

        $eventStore = new MysqlEventStore($pdo);
        $snapshotStore = new MysqlSnapshotStore($pdo, $eventStore);
        $orderRepository = new EventSourcingOrderRepository($eventStore, $snapshotStore);
        $eventBus = new SynchronousEventBus(new \App\Infrastructure\Logger\SimpleLogger());

        $createOrderHandler = new CreateOrderHandler(
            $orderRepository,
            $eventStore,
            $eventBus
        );

        $orderProjection = new OrderProjection(
            new \App\Adapter\Output\Projection\MysqlOrderReadModel($pdo)
        );

        $eventBus->subscribe(OrderCreatedEvent::class, [$orderProjection, 'handle']);
        $eventBus->subscribe(OrderConfirmedEvent::class, [$orderProjection, 'handle']);
        $eventBus->subscribe(OrderCancelledEvent::class, [$orderProjection, 'handle']);

        $notificationHandler = new OrderEventHandler(
            new \App\Adapter\Output\Notification\EmailAdapter($config['mail']),
            new \App\Adapter\Output\Inventory\InventoryAdapter(),
            new \App\Adapter\Output\Audit\MysqlAuditLog($pdo)
        );

        $eventBus->subscribe(OrderCreatedEvent::class, [$notificationHandler, 'handleOrderCreated']);
        $eventBus->subscribe(OrderConfirmedEvent::class, [$notificationHandler, 'handleOrderConfirmed']);

        return [
            'orderRepository' => $orderRepository,
            'createOrderHandler' => $createOrderHandler,
            'eventBus' => $eventBus
        ];
    }
}
```

## 使用场景

### 1. 复杂业务系统

当业务逻辑复杂、需要维护完整的历史记录时，事件溯源是很好的选择。

### 2. 高性能系统

当读操作和写操作性能需求差异大时，CQRS 可以分别优化读写模型。

### 3. 审计需求

当需要完整审计追踪时，事件溯源提供了天然的支持。

### 4. 事件驱动架构

当系统需要响应各种业务事件时，事件总线提供了松耦合的通信方式。

## 注意事项

### 1. 渐进式采用

不需要一开始就使用所有模式。可以从 CQRS 开始，然后逐步引入事件溯源。

### 2. 性能考虑

事件溯源会产生大量事件，需要考虑存储和性能优化。

### 3. 学习曲线

这些模式需要团队投入时间学习。确保团队理解这些模式的工作原理。

## 常见问题

### Q1: 何时使用完整的模式组合？

当系统需要完整的审计追踪、复杂的状态逻辑、高性能读写分离时，可以使用完整组合。

### Q2: 如何处理事件版本升级？

设计事件时考虑版本，使用升级器处理不同版本的事件。

### Q3: 如何保证最终一致性？

使用事件驱动的投影机制，确保读模型最终与写模型一致。

## 最佳实践

### 1. 从简单开始

先使用传统方式，只在需要时逐步引入这些模式。

### 2. 设计清晰的事件

事件应该简洁、易于理解，包含足够的信息。

### 3. 优化投影性能

为常用的查询场景设计专门的投影，定期更新。

### 4. 监控和日志

添加适当的监控和日志，便于问题排查。

## 练习任务

### 练习 1：设计完整的数据流

为一个订单系统设计完整的数据流，包括命令端、事件存储、事件总线、读模型投影。

### 2：实现事件投影

为订单系统实现一个订单列表的读模型投影，优化查询性能。

### 3：设计事件升级器

设计一个事件升级器，处理不同版本事件的兼容性问题。

### 4：评估模式选择

分析一个现有系统，评估是否适合使用 CQRS 和事件溯源，并说明理由。

### 5：实施渐进式迁移

设计一个渐进式迁移方案，将现有系统迁移到事件驱动架构。
