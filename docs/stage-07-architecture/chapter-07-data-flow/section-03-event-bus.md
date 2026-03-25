# 7.7.3 事件总线（Event Bus）

## 概述

事件总线（Event Bus）是实现事件驱动架构的核心组件，它提供了一种发布-订阅机制，使得系统中的不同组件可以通过事件进行松耦合的通信。在事件总线中，事件的发布者不需要知道谁会处理这个事件，事件的订阅者也不需要知道事件来自哪里。这种松耦合的通信方式使得系统更加灵活，更容易扩展和维护。

事件总线的工作原理类似于现实生活中的广播站：广播站（事件总线）发送广播（事件），听众（事件订阅者）可以选择收听自己感兴趣的节目（事件类型）。听众和广播站之间没有直接的连接，它们都只与广播媒介（事件总线）交互。

在 PHP 应用中，事件总线通常用于解耦业务逻辑。例如，当订单创建时，可能需要发送确认邮件、更新库存、记录日志等多个操作。这些操作不应该直接耦合在订单创建的业务逻辑中，而应该通过事件总线发布订单创建事件，然后由不同的处理器异步处理。

**主要内容**：
- 事件总线的概念和作用
- 事件的发布机制
- 事件的订阅和处理
- 事件路由和分发
- 同步与异步处理
- 完整的代码示例

## 特性

- **松耦合**：发布者和订阅者之间没有直接依赖
- **可扩展性**：可以轻松添加新的订阅者
- **灵活性**：支持同步和异步处理
- **广播**：一个事件可以触发多个处理
- **单点管理**：所有事件流经事件总线，便于监控和管理

## 核心概念

### 发布-订阅模式

```
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│   发布者    │         │  事件总线   │         │   订阅者    │
│             │         │             │         │             │
│  发布事件   │────────▶│  接收事件   │────────▶│  处理事件   │
│             │         │             │         │             │
└─────────────┘         │             │         └─────────────┘
                        │             │         
                        │             │         ┌─────────────┐
                        │             │────────▶│  订阅者 2   │
                        │             │         │             │
                        └─────────────┘         └─────────────┘
```

### 事件总线的组成

- **事件通道（Event Channel）**：事件的分类和路由
- **事件总线（Event Bus）**：事件的分发中心
- **事件处理器（Event Handler）**：处理事件的逻辑

## 基本用法

### 1. 事件总线接口定义

```php
<?php
declare(strict_types=1);

namespace App\Port\Output;

use App\Domain\Event\DomainEvent;

interface EventBusPort
{
    public function publish(DomainEvent $event): void;
    
    public function publishBatch(array $events): void;
    
    public function subscribe(string $eventType, callable $handler): void;
    
    public function unsubscribe(string $eventType, callable $handler): void;
}
```

### 2. 同步事件总线实现

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\EventBus;

use App\Domain\Event\DomainEvent;
use App\Port\Output\EventBusPort;
use Psr\Log\LoggerInterface;

class SynchronousEventBus implements EventBusPort
{
    private array $handlers = [];
    private LoggerInterface $logger;

    public function __construct(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    public function publish(DomainEvent $event): void
    {
        $eventType = get_class($event);
        
        $this->logger->info('Event published', [
            'event_type' => $eventType,
            'event_data' => $this->serializeEvent($event)
        ]);

        if (!isset($this->handlers[$eventType])) {
            return;
        }

        foreach ($this->handlers[$eventType] as $handler) {
            try {
                $handler($event);
            } catch (\Throwable $e) {
                $this->logger->error('Event handler failed', [
                    'event_type' => $eventType,
                    'error' => $e->getMessage()
                ]);
                throw $e;
            }
        }
    }

    public function publishBatch(array $events): void
    {
        foreach ($events as $event) {
            $this->publish($event);
        }
    }

    public function subscribe(string $eventType, callable $handler): void
    {
        if (!isset($this->handlers[$eventType])) {
            $this->handlers[$eventType] = [];
        }
        
        $this->handlers[$eventType][] = $handler;
        
        $this->logger->debug('Handler subscribed', [
            'event_type' => $eventType
        ]);
    }

    public function unsubscribe(string $eventType, callable $handler): void
    {
        if (!isset($this->handlers[$eventType])) {
            return;
        }

        $this->handlers[$eventType] = array_filter(
            $this->handlers[$eventType],
            fn($h) => $h !== $handler
        );
    }

    private function serializeEvent(DomainEvent $event): array
    {
        $data = [];
        foreach (get_object_vars($event) as $key => $value) {
            if (is_object($value)) {
                $data[$key] = method_exists($value, '__toString') 
                    ? (string) $value 
                    : get_class($value);
            } else {
                $data[$key] = $value;
            }
        }
        return $data;
    }
}
```

### 3. 异步事件总线实现

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\EventBus;

use App\Domain\Event\DomainEvent;
use App\Port\Output\EventBusPort;
use Psr\Log\LoggerInterface;

class AsynchronousEventBus implements EventBusPort
{
    private array $handlers = [];
    private LoggerInterface $logger;
    private array $eventQueue = [];

    public function __construct(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }

    public function publish(DomainEvent $event): void
    {
        $eventType = get_class($event);
        
        $this->logger->info('Event queued', [
            'event_type' => $eventType,
            'event_data' => $this->serializeEvent($event)
        ]);

        $this->eventQueue[] = $event;

        if (!isset($this->handlers[$eventType])) {
            return;
        }

        foreach ($this->handlers[$eventType] as $handler) {
            $this->processAsync($handler, $event);
        }
    }

    public function publishBatch(array $events): void
    {
        foreach ($events as $event) {
            $this->publish($event);
        }
    }

    public function subscribe(string $eventType, callable $handler): void
    {
        if (!isset($this->handlers[$eventType])) {
            $this->handlers[$eventType] = [];
        }
        
        $this->handlers[$eventType][] = $handler;
    }

    public function unsubscribe(string $eventType, callable $handler): void
    {
        if (!isset($this->handlers[$eventType])) {
            return;
        }

        $this->handlers[$eventType] = array_filter(
            $this->handlers[$eventType],
            fn($h) => $h !== $handler
        );
    }

    public function processQueue(): void
    {
        while ($event = array_shift($this->eventQueue)) {
            $eventType = get_class($event);
            
            if (!isset($this->handlers[$eventType])) {
                continue;
            }

            foreach ($this->handlers[$eventType] as $handler) {
                try {
                    $handler($event);
                } catch (\Throwable $e) {
                    $this->logger->error('Async event handler failed', [
                        'event_type' => $eventType,
                        'error' => $e->getMessage()
                    ]);
                }
            }
        }
    }

    private function processAsync(callable $handler, DomainEvent $event): void
    {
        $handler = $handler;
        $event = $event;
        
        if (function_exists('proc_open')) {
            $this->logger->info('Async processing not available, using sync');
        }
    }

    private function serializeEvent(DomainEvent $event): array
    {
        $data = [];
        foreach (get_object_vars($event) as $key => $value) {
            if (is_object($value)) {
                $data[$key] = method_exists($value, '__toString') 
                    ? (string) $value 
                    : get_class($value);
            } else {
                $data[$key] = $value;
            }
        }
        return $data;
    }
}
```

### 4. 事件处理器定义

```php
<?php
declare(strict_types=1);

namespace App\Application\Handler;

use App\Domain\Event\OrderCreatedEvent;
use App\Domain\Event\OrderConfirmedEvent;
use App\Port\Output\NotificationPort;
use App\Port\Output\InventoryPort;
use App\Port\Output\AuditLogPort;

class OrderEventHandler
{
    public function __construct(
        private readonly NotificationPort $notification,
        private readonly InventoryPort $inventory,
        private readonly AuditLogPort $auditLog
    ) {}

    public function handleOrderCreated(OrderCreatedEvent $event): void
    {
        $this->notification->sendEmail([
            'to' => $event->customerEmail,
            'subject' => 'Order Confirmation',
            'body' => "Your order #{$event->orderId} has been created."
        ]);

        $this->auditLog->log([
            'event' => 'order_created',
            'order_id' => $event->orderId,
            'customer_id' => $event->customerId,
            'timestamp' => $event->getOccurredAt()->format('Y-m-d H:i:s')
        ]);
    }

    public function handleOrderConfirmed(OrderConfirmedEvent $event): void
    {
        foreach ($event->items as $item) {
            $this->inventory->reserve($item['product_id'], $item['quantity']);
        }

        $this->notification->sendEmail([
            'to' => $event->customerEmail,
            'subject' => 'Order Confirmed',
            'body' => "Your order #{$event->orderId} has been confirmed."
        ]);

        $this->auditLog->log([
            'event' => 'order_confirmed',
            'order_id' => $event->orderId,
            'timestamp' => $event->getOccurredAt()->format('Y-m-d H:i:s')
        ]);
    }
}
```

### 5. 事件总线配置和启动

```php
<?php
declare(strict_types=1);

namespace App\Infrastructure\Bootstrap;

use App\Adapter\Output\EventBus\SynchronousEventBus;
use App\Application\Handler\OrderEventHandler;
use App\Domain\Event\OrderCreatedEvent;
use App\Domain\Event\OrderConfirmedEvent;
use App\Domain\Event\OrderCancelledEvent;

class EventBusBootstrap
{
    public static function bootstrap(SynchronousEventBus $eventBus): void
    {
        $handler = new OrderEventHandler(
            new \App\Adapter\Output\Notification\EmailAdapter(),
            new \App\Adapter\Output\Inventory\InventoryAdapter(),
            new \App\Adapter\Output\Audit\MysqlAuditLog()
        );

        $eventBus->subscribe(OrderCreatedEvent::class, [$handler, 'handleOrderCreated']);
        $eventBus->subscribe(OrderConfirmedEvent::class, [$handler, 'handleOrderConfirmed']);
        $eventBus->subscribe(OrderCancelledEvent::class, [$handler, 'handleOrderCancelled']);
    }
}
```

### 6. 事件总线中间件

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\EventBus;

use App\Domain\Event\DomainEvent;

interface EventBusMiddleware
{
    public function process(DomainEvent $event, callable $next): void;
}
```

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\EventBus;

use App\Domain\Event\DomainEvent;

class LoggingMiddleware implements EventBusMiddleware
{
    public function process(DomainEvent $event, callable $next): void
    {
        $startTime = microtime(true);
        
        try {
            $next($event);
        } finally {
            $duration = microtime(true) - $startTime;
            
            error_log(sprintf(
                'Event %s processed in %.4fms',
                get_class($event),
                $duration * 1000
            ));
        }
    }
}
```

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\EventBus;

use App\Domain\Event\DomainEvent;

class ErrorHandlingMiddleware implements EventBusMiddleware
{
    public function process(DomainEvent $event, callable $next): void
    {
        try {
            $next($event);
        } catch (\Throwable $e) {
            error_log(sprintf(
                'Error processing event %s: %s',
                get_class($event),
                $e->getMessage()
            ));
            
            throw $e;
        }
    }
}
```

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\EventBus;

use App\Domain\Event\DomainEvent;

class EventBusWithMiddleware implements EventBusPort
{
    private array $handlers = [];
    private array $middlewares = [];

    public function __construct(private array $middlewares = [])
    {
        $this->middlewares = $middlewares;
    }

    public function publish(DomainEvent $event): void
    {
        $eventType = get_class($event);
        
        if (!isset($this->handlers[$eventType])) {
            return;
        }

        $handler = function(DomainEvent $event) use ($eventType) {
            foreach ($this->handlers[$eventType] as $handler) {
                $handler($event);
            }
        };

        foreach (array_reverse($this->middlewares) as $middleware) {
            $handler = fn($event) => $middleware->process($event, $handler);
        }

        $handler($event);
    }

    public function publishBatch(array $events): void
    {
        foreach ($events as $event) {
            $this->publish($event);
        }
    }

    public function subscribe(string $eventType, callable $handler): void
    {
        if (!isset($this->handlers[$eventType])) {
            $this->handlers[$eventType] = [];
        }
        $this->handlers[$eventType][] = $handler;
    }

    public function unsubscribe(string $eventType, callable $handler): void
    {
        if (!isset($this->handlers[$eventType])) {
            return;
        }
        $this->handlers[$eventType] = array_filter(
            $this->handlers[$eventType],
            fn($h) => $h !== $handler
        );
    }
}
```

## 使用场景

### 1. 解耦业务逻辑

当业务逻辑的各个部分需要松耦合时，事件总线是一个很好的选择。例如，订单创建后需要发送邮件、更新库存、记录日志等操作，可以通过事件总线解耦。

### 2. 异步处理

对于不需要同步处理的耗时操作，可以通过异步事件处理来提高系统响应速度。

### 3. 系统集成

当需要与多个外部系统集成时，事件总线可以提供一个统一的集成点。

### 4. 审计和监控

所有事件都经过事件总线，可以方便地实现审计日志和系统监控。

## 注意事项

### 1. 事件顺序

在某些场景下，事件的顺序很重要。需要确保事件按照正确的顺序处理，或者在事件中包含足够的信息来处理乱序。

### 2. 错误处理

事件处理可能出现错误。需要设计适当的错误处理机制，如重试、死信队列等。

### 3. 幂等性

事件处理器应该是幂等的，即多次处理同一个事件应该得到相同的结果。

## 常见问题

### Q1: 事件总线与消息队列的区别？

事件总线是进程内的同步或异步通信机制，适合单进程内的组件通信。消息队列是进程间或系统间的异步通信机制，适合分布式系统。

### Q2: 如何处理事件处理的失败？

可以使用重试机制、死信队列等来处理事件处理失败。对于重要的操作，需要确保最终成功处理。

### Q3: 事件总线是否会影响性能？

同步事件总线会增加请求的处理时间。可以使用异步事件处理来优化性能，但会增加系统的复杂性。

## 最佳实践

### 1. 使用事件接口

定义统一的事件接口，确保所有事件都遵循相同的结构。

### 2. 添加日志

在事件总线中添加适当的日志，便于问题排查和监控。

### 3. 处理异常

设计适当的异常处理机制，确保单个事件的失败不会影响其他事件。

### 4. 考虑幂等性

在事件处理器中考虑幂等性，确保重复处理不会造成问题。

## 练习任务

### 练习 1：实现事件总线

实现一个简单的事件总线，支持事件的订阅和发布。

### 2：设计事件处理器

为一个电商系统设计订单事件处理器，包括订单创建、订单确认、订单取消等事件。

### 3：实现事件中间件

实现日志记录和错误处理中间件，添加到事件总线中。

### 4：设计异步处理

设计一个异步事件处理机制，将耗时操作放到后台处理。

### 5：分析解耦效果

分析一个现有系统，识别可以使用事件总线解耦的业务逻辑。
