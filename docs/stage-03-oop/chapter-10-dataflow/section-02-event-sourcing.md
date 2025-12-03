# 3.10.2 事件溯源（Event Sourcing）

## 概述

事件溯源（Event Sourcing）不存储当前状态，而是存储所有事件。通过重放事件重建当前状态，提供完整的历史记录和审计能力。

## 事件溯源概念

### 基本思想

- **事件溯源**：不存储当前状态，而是存储所有事件。
- 通过重放事件重建当前状态。

```
传统方式：
Order { id: 1, status: 'shipped', total: 100 }

事件溯源：
- OrderCreated { id: 1, customerId: 123 }
- ItemAdded { orderId: 1, productId: 456, quantity: 2 }
- OrderConfirmed { orderId: 1 }
- OrderShipped { orderId: 1 }
```

### 优势

- **完整历史**：保留所有状态变更历史。
- **审计能力**：可以追踪所有操作。
- **时间旅行**：可以重建任意时间点的状态。
- **调试友好**：可以重放事件定位问题。

## 事件定义

### 领域事件

```php
namespace App\Domain\Events;

interface DomainEvent
{
    public function occurredOn(): DateTimeImmutable;
}

class OrderCreated implements DomainEvent
{
    public function __construct(
        public readonly string $orderId,
        public readonly string $customerId,
        private DateTimeImmutable $occurredOn
    ) {
    }

    public function occurredOn(): DateTimeImmutable
    {
        return $this->occurredOn;
    }
}

class ItemAdded implements DomainEvent
{
    public function __construct(
        public readonly string $orderId,
        public readonly string $productId,
        public readonly int $quantity,
        private DateTimeImmutable $occurredOn
    ) {
    }

    public function occurredOn(): DateTimeImmutable
    {
        return $this->occurredOn;
    }
}
```

## 事件存储

### EventStore 接口

```php
namespace App\Infrastructure\EventStore;

interface EventStore
{
    public function append(string $aggregateId, DomainEvent $event): void;
    public function getEvents(string $aggregateId): array;
}

class DatabaseEventStore implements EventStore
{
    public function __construct(private PDO $db)
    {
    }

    public function append(string $aggregateId, DomainEvent $event): void
    {
        $stmt = $this->db->prepare(
            'INSERT INTO events (aggregate_id, event_type, event_data, occurred_at)
             VALUES (?, ?, ?, ?)'
        );
        $stmt->execute([
            $aggregateId,
            get_class($event),
            json_encode($event),
            $event->occurredOn()->format('Y-m-d H:i:s'),
        ]);
    }

    public function getEvents(string $aggregateId): array
    {
        $stmt = $this->db->prepare(
            'SELECT event_type, event_data FROM events 
             WHERE aggregate_id = ? 
             ORDER BY occurred_at ASC'
        );
        $stmt->execute([$aggregateId]);
        $events = [];

        while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
            $eventClass = $row['event_type'];
            $eventData = json_decode($row['event_data'], true);
            $events[] = $this->deserializeEvent($eventClass, $eventData);
        }

        return $events;
    }

    private function deserializeEvent(string $eventClass, array $data): DomainEvent
    {
        // 反序列化事件
        return new $eventClass(...$data);
    }
}
```

## 聚合重建

### 从事件重建状态

```php
class Order
{
    private array $items = [];
    private OrderStatus $status = OrderStatus::PENDING;
    private Money $total;

    public static function fromEvents(array $events): self
    {
        $order = new self();
        foreach ($events as $event) {
            $order->apply($event);
        }
        return $order;
    }

    private function apply(DomainEvent $event): void
    {
        match (get_class($event)) {
            OrderCreated::class => $this->handleOrderCreated($event),
            ItemAdded::class => $this->handleItemAdded($event),
            OrderShipped::class => $this->handleOrderShipped($event),
            default => throw new UnknownEventException(get_class($event)),
        };
    }

    private function handleOrderCreated(OrderCreated $event): void
    {
        $this->id = new OrderId($event->orderId);
        $this->customerId = new CustomerId($event->customerId);
        $this->total = new Money(0, 'USD');
    }

    private function handleItemAdded(ItemAdded $event): void
    {
        $item = new OrderItem(
            new ProductId($event->productId),
            $event->quantity,
            new Money(10.0, 'USD') // 从事件或查询获取价格
        );
        $this->items[] = $item;
        $this->recalculateTotal();
    }

    private function handleOrderShipped(OrderShipped $event): void
    {
        $this->status = OrderStatus::SHIPPED;
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

namespace App\Domain\Order;

class OrderService
{
    public function __construct(
        private EventStore $eventStore
    ) {
    }

    public function getOrder(OrderId $id): Order
    {
        $events = $this->eventStore->getEvents($id->getValue());
        return Order::fromEvents($events);
    }

    public function createOrder(CustomerId $customerId): Order
    {
        $orderId = OrderId::generate();
        $event = new OrderCreated($orderId->getValue(), $customerId->getValue(), new DateTimeImmutable());
        
        $this->eventStore->append($orderId->getValue(), $event);
        
        return $this->getOrder($orderId);
    }

    public function addItem(OrderId $orderId, ProductId $productId, int $quantity): void
    {
        $event = new ItemAdded(
            $orderId->getValue(),
            $productId->getValue(),
            $quantity,
            new DateTimeImmutable()
        );
        
        $this->eventStore->append($orderId->getValue(), $event);
    }
}
```

## 注意事项

1. **事件版本**：事件结构可能变化，需要处理版本兼容性。

2. **性能考虑**：重建聚合需要重放所有事件，可能影响性能。

3. **快照**：对于大型聚合，可以使用快照优化性能。

4. **事件不可变**：事件一旦存储，不应该修改。

5. **适用场景**：需要完整历史记录、审计能力的场景。

## 练习

1. 实现一个事件存储系统，支持事件的追加和查询。

2. 创建一个订单聚合，使用事件溯源存储状态变更。

3. 实现从事件重建聚合状态的逻辑。

4. 设计一个支持事件版本升级的系统。

5. 创建一个支持快照的事件溯源系统。
