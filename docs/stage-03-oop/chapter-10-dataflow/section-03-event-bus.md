# 3.10.3 事件总线（Event Bus）

## 概述

事件总线（Event Bus）用于在应用的不同部分之间传递领域事件，实现解耦和异步处理。理解事件总线的设计对于构建可扩展的应用至关重要。

## 事件总线概念

### 基本定义

- **事件总线**：用于发布和订阅领域事件的机制。
- **发布者**：发布事件的对象。
- **订阅者**：处理事件的对象。

```
发布者 → 事件总线 → 订阅者1
                  → 订阅者2
                  → 订阅者3
```

## 事件总线实现

### 基本实现

```php
namespace App\Application\Events;

interface EventBus
{
    public function publish(DomainEvent $event): void;
    public function subscribe(string $eventClass, callable $handler): void;
}

class SimpleEventBus implements EventBus
{
    private array $subscribers = [];

    public function subscribe(string $eventClass, callable $handler): void
    {
        if (!isset($this->subscribers[$eventClass])) {
            $this->subscribers[$eventClass] = [];
        }
        $this->subscribers[$eventClass][] = $handler;
    }

    public function publish(DomainEvent $event): void
    {
        $eventClass = get_class($event);
        
        if (!isset($this->subscribers[$eventClass])) {
            return;
        }

        foreach ($this->subscribers[$eventClass] as $handler) {
            $handler($event);
        }
    }
}
```

## 事件处理

### 同步处理

```php
class OrderService
{
    public function __construct(
        private OrderRepository $orderRepository,
        private EventBus $eventBus
    ) {
    }

    public function createOrder(CustomerId $customerId): Order
    {
        $order = new Order(OrderId::generate(), $customerId);
        $this->orderRepository->save($order);

        // 发布事件
        $this->eventBus->publish(new OrderCreated(
            $order->getId()->getValue(),
            $customerId->getValue(),
            new DateTimeImmutable()
        ));

        return $order;
    }
}

// 订阅事件
$eventBus->subscribe(OrderCreated::class, function (OrderCreated $event) {
    // 发送欢迎邮件
    $emailService->sendWelcomeEmail($event->customerId);
});

$eventBus->subscribe(OrderCreated::class, function (OrderCreated $event) {
    // 更新统计
    $statsService->incrementOrderCount();
});
```

### 异步处理

```php
class AsyncEventBus implements EventBus
{
    private array $queue = [];

    public function publish(DomainEvent $event): void
    {
        // 将事件加入队列，异步处理
        $this->queue[] = $event;
    }

    public function processQueue(): void
    {
        while (!empty($this->queue)) {
            $event = array_shift($this->queue);
            $this->handleEvent($event);
        }
    }

    private function handleEvent(DomainEvent $event): void
    {
        $eventClass = get_class($event);
        $handlers = $this->subscribers[$eventClass] ?? [];
        
        foreach ($handlers as $handler) {
            $handler($event);
        }
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

namespace App\Application\Events;

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

class EventBus
{
    private array $subscribers = [];

    public function subscribe(string $eventClass, callable $handler): void
    {
        if (!isset($this->subscribers[$eventClass])) {
            $this->subscribers[$eventClass] = [];
        }
        $this->subscribers[$eventClass][] = $handler;
    }

    public function publish(DomainEvent $event): void
    {
        $eventClass = get_class($event);
        
        if (!isset($this->subscribers[$eventClass])) {
            return;
        }

        foreach ($this->subscribers[$eventClass] as $handler) {
            try {
                $handler($event);
            } catch (\Exception $e) {
                // 记录错误，但不中断其他处理器
                error_log("Event handler failed: {$e->getMessage()}");
            }
        }
    }
}

// 使用
$eventBus = new EventBus();

// 订阅事件
$eventBus->subscribe(OrderCreated::class, function (OrderCreated $event) {
    echo "Order created: {$event->orderId}\n";
});

$eventBus->subscribe(OrderCreated::class, function (OrderCreated $event) {
    echo "Sending notification for order: {$event->orderId}\n";
});

// 发布事件
$eventBus->publish(new OrderCreated('123', '456', new DateTimeImmutable()));
```

## 注意事项

1. **事件顺序**：确保事件处理的顺序性（如果需要）。

2. **错误处理**：一个处理器的错误不应该影响其他处理器。

3. **异步处理**：对于耗时操作，使用异步事件处理。

4. **事件持久化**：重要事件应该持久化，确保不丢失。

5. **解耦**：事件总线实现发布者和订阅者的解耦。

## 练习

1. 实现一个事件总线，支持事件的发布和订阅。

2. 创建一个订单服务，在创建订单时发布事件。

3. 实现多个事件订阅者，处理同一个事件。

4. 设计一个异步事件处理系统。

5. 创建一个支持事件持久化的事件总线。
