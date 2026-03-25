# 7.6.5 领域服务与领域事件

## 概述

领域服务（Domain Service）和领域事件（Domain Event）是领域驱动设计中两个重要的战术设计工具。领域服务用于封装不属于单个实体或值对象的业务逻辑，当某个业务操作需要多个领域对象协作完成时，领域服务提供了最佳的组织方式。领域事件用于表达领域中发生的业务事件，它是一种解耦系统组件的强大机制，使得系统可以采用事件驱动的架构模式。

领域服务的出现是为了解决这样一个问题：不是所有的业务逻辑都可以放在实体或值对象中。当一个操作涉及多个实体或聚合的协作，或者业务逻辑本身不适合放在任何一个实体中时，就应该使用领域服务。例如，转账业务涉及两个账户的操作，既不能放在转出账户中，也不能放在转入账户中，这时就需要一个转账领域服务来处理。

领域事件是领域模型对业务活动中发生的事件的建模。当领域中发生重要的事情时，可以发布一个领域事件，其他部分的应用可以通过订阅事件来做出响应。例如，当订单创建时，可以发布一个"订单已创建"的事件，通知库存系统扣减库存、通知物流系统准备发货、发送邮件通知用户等。

**主要内容**：
- 领域服务的概念和使用场景
- 领域服务的设计原则
- 领域事件的概念和建模
- 事件发布和处理机制
- 事件总线的实现
- 完整的代码示例

## 特性

### 领域服务的特性

- **无状态**：领域服务通常不持有状态，其操作都是幂等的
- **协调多个实体**：处理需要多个实体协作的业务逻辑
- **领域语言**：使用领域语言命名，反映业务概念
- **无标识**：领域服务不是实体，不需要唯一标识

### 领域事件的特性

- **过去式命名**：事件名应该使用过去式，表示已发生的事情
- **不可变性**：事件一旦发布就不应该修改
- **携带数据**：事件应该包含与事件相关的所有数据
- **时间戳**：事件应该包含发生的时间

## 核心概念

### 领域服务的设计原则

领域服务应该遵循以下设计原则：

第一，领域服务应该只包含不适合放在实体或值对象中的业务逻辑。如果业务逻辑可以放在实体或值对象中，应该优先放在那里。领域服务应该是补充性的，而不是替代实体方法。

第二，领域服务应该使用领域语言。服务的方法名和参数名应该反映业务概念，而不是技术细节。

第三，领域服务应该是无状态的。服务不应该持有任何会影响其行为的状态，所有的操作应该只依赖于传入的参数。

第四，领域服务不应该调用应用层或基础设施层的组件。领域服务应该只依赖于领域层的组件。

### 领域事件的建模

领域事件是对领域中发生的重要事实的建模。一个好的领域事件应该满足以下要求：

第一，事件名应该使用过去式，表示已经发生的事情。例如，`OrderCreatedEvent` 表示订单已创建，`PaymentProcessedEvent` 表示支付已处理。

第二，事件应该包含足够的信息，使得事件的订阅者不需要查询其他系统就能做出响应。事件应该自包含。

第三，事件应该是不可变的。事件一旦发布就不应该被修改。

第四，事件应该包含发生的时间戳，便于事件的排序和追踪。

## 语法/定义

### 领域服务定义

```php
<?php
declare(strict_types=1);

// 语法：class DomainServiceName { ... }

// 特征：
// - 无状态
// - 协调多个领域对象
// - 使用领域语言命名
```

### 领域事件定义

```php
<?php
declare(strict_types=1);

// 语法：class DomainEventName { ... }

// 特征：
// - 过去式命名
// - 不可变
// - 自包含
// - 包含时间戳
```

## 基本用法

### 1. 领域服务实现

以下是一个完整的转账领域服务实现：

```php
<?php
declare(strict_types=1);

namespace App\Domain\Service;

use App\Domain\Entity\Account;
use App\Domain\ValueObject\Money;
use App\Domain\Exception\InsufficientFundsException;
use App\Domain\Exception\AccountNotFoundException;

class TransferService
{
    public function __construct(
        private readonly AccountRepositoryPort $accountRepository,
        private readonly DomainEventPublisher $eventPublisher
    ) {}

    public function transfer(
        int $fromAccountId,
        int $toAccountId,
        Money $amount
    ): void {
        if ($amount->getAmount() <= 0) {
            throw new \InvalidArgumentException('Transfer amount must be positive');
        }

        if ($fromAccountId === $toAccountId) {
            throw new \InvalidArgumentException('Cannot transfer to the same account');
        }

        $fromAccount = $this->accountRepository->findById($fromAccountId);
        if ($fromAccount === null) {
            throw new AccountNotFoundException("Source account not found: {$fromAccountId}");
        }

        $toAccount = $this->accountRepository->findById($toAccountId);
        if ($toAccount === null) {
            throw new AccountNotFoundException("Target account not found: {$toAccountId}");
        }

        $fromAccount->withdraw($amount);
        $toAccount->deposit($amount);

        $this->accountRepository->save($fromAccount);
        $this->accountRepository->save($toAccount);

        $this->eventPublisher->publish(new MoneyTransferredEvent(
            fromAccountId: $fromAccountId,
            toAccountId: $toAccountId,
            amount: $amount,
            transferredAt: new \DateTimeImmutable()
        ));
    }
}
```

转账业务涉及两个账户的操作，不能放在任何一个账户中，所以创建了转账领域服务来处理。服务调用账户的 withdraw 和 deposit 方法来完成实际的转账操作，然后发布领域事件通知其他系统。

### 2. 领域事件实现

以下是转账相关的事件定义：

```php
<?php
declare(strict_types=1);

namespace App\Domain\Event;

use App\Domain\ValueObject\Money;

class MoneyTransferredEvent
{
    public function __construct(
        public readonly int $fromAccountId,
        public readonly int $toAccountId,
        public readonly Money $amount,
        public readonly \DateTimeImmutable $transferredAt
    ) {}

    public function getEventType(): string
    {
        return 'money.transferred';
    }
}
```

```php
<?php
declare(strict_types=1);

namespace App\Domain\Event;

class AccountCreatedEvent
{
    public function __construct(
        public readonly int $accountId,
        public readonly string $accountNumber,
        public readonly string $accountHolderName,
        public readonly \DateTimeImmutable $createdAt
    ) {}

    public function getEventType(): string
    {
        return 'account.created';
    }
}
```

```php
<?php
declare(strict_types=1);

namespace App\Domain\Event;

use App\Domain\ValueObject\Money;

class AccountCreditedEvent
{
    public function __construct(
        public readonly int $accountId,
        public readonly Money $amount,
        public readonly string $reference,
        public readonly \DateTimeImmutable $creditedAt
    ) {}

    public function getEventType(): string
    {
        return 'account.credited';
    }
}
```

### 3. 事件发布机制

以下是事件发布器的实现：

```php
<?php
declare(strict_types=1);

namespace App\Domain\Service;

use App\Domain\Event\DomainEvent;

interface DomainEventPublisher
{
    public function publish(DomainEvent $event): void;
    public function publishBatch(array $events): void;
}
```

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\Event;

use App\Domain\Service\DomainEventPublisher;
use App\Domain\Event\DomainEvent;
use Psr\Log\LoggerInterface;

class SimpleDomainEventPublisher implements DomainEventPublisher
{
    private array $handlers = [];

    public function __construct(
        private readonly LoggerInterface $logger
    ) {}

    public function publish(DomainEvent $event): void
    {
        $eventType = $event->getEventType();
        
        $this->logger->info('Event published', [
            'event_type' => $eventType,
            'event_data' => get_object_vars($event)
        ]);

        if (isset($this->handlers[$eventType])) {
            foreach ($this->handlers[$eventType] as $handler) {
                $handler($event);
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
    }
}
```

### 4. 事件处理器

以下是事件处理器的实现：

```php
<?php
declare(strict_types=1);

namespace App\Application\Handler;

use App\Domain\Event\MoneyTransferredEvent;
use App\Port\Output\NotificationPort;

class SendTransferNotificationHandler
{
    public function __construct(
        private readonly NotificationPort $notification
    ) {}

    public function handle(MoneyTransferredEvent $event): void
    {
        $message = sprintf(
            '您已成功转账 %s 元到账户 %d',
            $event->amount->getAmount() / 100,
            $event->toAccountId
        );

        $this->notification->send([
            'account_id' => $event->fromAccountId,
            'message' => $message,
            'type' => 'transfer_notification'
        ]);
    }
}
```

```php
<?php
declare(strict_types=1);

namespace App\Application\Handler;

use App\Domain\Event\MoneyTransferredEvent;
use App\Port\Output\AuditLogPort;

class LogTransferHandler
{
    public function __construct(
        private readonly AuditLogPort $auditLog
    ) {}

    public function handle(MoneyTransferredEvent $event): void
    {
        $this->auditLog->log([
            'event' => 'money_transfer',
            'from_account' => $event->fromAccountId,
            'to_account' => $event->toAccountId,
            'amount' => $event->amount->getAmount(),
            'timestamp' => $event->transferredAt->format('Y-m-d H:i:s')
        ]);
    }
}
```

### 5. 聚合中的事件发布

在聚合中发布领域事件：

```php
<?php
declare(strict_types=1);

namespace App\Domain\Entity;

use App\Domain\Event\DomainEvent;
use App\Domain\Event\OrderCreatedEvent;
use App\Domain\Event\OrderConfirmedEvent;
use App\Domain\Event\OrderCancelledEvent;

class Order
{
    private array $domainEvents = [];

    public function __construct(
        private readonly int $customerId,
        private readonly string $customerEmail
    ) {
        $this->addDomainEvent(new OrderCreatedEvent(
            orderId: 0,
            customerId: $customerId,
            customerEmail: $customerEmail,
            createdAt: new \DateTimeImmutable()
        ));
    }

    public function confirm(): void
    {
        $this->status = OrderStatus::CONFIRMED;
        
        $this->addDomainEvent(new OrderConfirmedEvent(
            orderId: $this->id,
            customerId: $this->customerId,
            confirmedAt: new \DateTimeImmutable()
        ));
    }

    public function cancel(): void
    {
        $this->status = OrderStatus::CANCELLED;
        
        $this->addDomainEvent(new OrderCancelledEvent(
            orderId: $this->id,
            customerId: $this->customerId,
            reason: 'User cancelled',
            cancelledAt: new \DateTimeImmutable()
        ));
    }

    public function pullDomainEvents(): array
    {
        $events = $this->domainEvents;
        $this->domainEvents = [];
        return $events;
    }

    private function addDomainEvent(DomainEvent $event): void
    {
        $this->domainEvents[] = $event;
    }
}
```

聚合根可以维护一个事件队列，在状态变化时添加事件，然后用 pullDomainEvents 方法取出所有事件进行发布。

### 6. 完整的事件驱动流程

以下是完整的事件驱动流程实现：

```php
<?php
declare(strict_types=1);

namespace App\Application\UseCase;

use App\Application\Dto\CreateOrderDto;
use App\Domain\Entity\Order;
use App\Domain\Service\OrderDomainService;
use App\Port\Output\OrderRepositoryPort;
use App\Port\Output\DomainEventPublisherPort;

class CreateOrderUseCase
{
    public function __construct(
        private readonly OrderRepositoryPort $orderRepository,
        private readonly OrderDomainService $orderDomainService,
        private readonly DomainEventPublisherPort $eventPublisher
    ) {}

    public function execute(CreateOrderDto $dto): Order
    {
        $order = $this->orderDomainService->createOrder(
            customerId: $dto->customerId,
            customerEmail: $dto->customerEmail,
            items: $dto->items
        );

        $orderId = $this->orderRepository->save($order);

        $events = $order->pullDomainEvents();
        foreach ($events as $event) {
            $this->eventPublisher->publish($event);
        }

        return $this->orderRepository->findById($orderId);
    }
}
```

## 使用场景

### 领域服务使用场景

- **跨聚合操作**：当业务操作涉及多个聚合的协作时，使用领域服务协调
- **计算密集型逻辑**：某些复杂的计算逻辑不适合放在实体中
- **业务规则验证**：需要访问多个实体才能完成的业务规则验证

### 领域事件使用场景

- **解耦系统组件**：当需要解耦系统内部的不同模块时
- **跨系统集成**：当需要与其他系统集成时，事件是很好的桥梁
- **审计日志**：记录所有重要业务操作的历史
- **异步处理**：某些操作可以异步执行，提高系统响应速度

## 注意事项

### 1. 谨慎使用领域服务

领域服务应该是补充性的。大多数业务逻辑应该放在实体或值对象中。只在不适合放在实体中的情况下才使用领域服务。

### 2. 事件命名规范

事件名应该使用过去式，表示已经发生的事情。事件名应该清晰表达事件的内容。

### 3. 事件幂等性

事件处理应该是幂等的，即多次处理同一个事件应该得到相同的结果。这对于处理重复消息非常重要。

### 4. 事件顺序

在某些场景下，事件的顺序很重要。确保事件按照正确的顺序处理，或者使用消息队列保证顺序。

## 常见问题

### Q1: 领域服务与应用服务有什么区别？

领域服务处理纯业务逻辑，不依赖应用层或基础设施层。应用服务处理用例的编排，依赖于领域服务和仓储。领域服务是领域层的一部分，应用服务是应用层的一部分。

### Q2: 领域事件与应用事件有什么区别？

领域事件表达领域概念中的重要事件，与业务紧密相关。应用事件更偏向技术层面，如请求开始、请求结束等。领域事件是领域模型的一部分，应用事件是应用层面的实现细节。

### Q3: 事件驱动与直接调用如何选择？

当模块之间需要紧密协作、实时响应时，使用直接调用更简单。当需要解耦、异步处理、或需要多系统响应时，事件驱动更合适。

### Q4: 如何保证事件不丢失？

可以使用可靠的消息队列来保证事件的传递。也可以将事件持久化到数据库，然后异步处理。

## 最佳实践

### 1. 优先使用实体方法

将业务逻辑优先放在实体或值对象中，只在不适合的情况下使用领域服务。

### 2. 事件自包含

确保事件包含足够的信息，使订阅者不需要查询其他系统。

### 3. 事件溯源

考虑使用事件溯源（Event Sourcing）模式，存储所有领域事件而不是当前状态。

### 4. 异步处理

对于不需要同步处理的副作用，使用异步事件处理。

## 练习任务

### 练习 1：设计支付领域服务

为一个电商系统设计支付领域服务。支付涉及订单、支付渠道、退款等复杂逻辑，使用领域服务来协调这些操作。

### 练习 2：实现订单事件

为订单聚合设计领域事件，包括订单创建、订单确认、订单取消、订单完成等事件。

### 练习 3：实现事件总线

实现一个简单的事件总线，支持事件的发布和订阅。

### 练习 4：处理重复事件

设计一个机制来处理重复的事件，确保事件处理的幂等性。

### 练习 5：实现事件溯源

使用事件溯源模式实现一个简单的账户聚合，账户的所有操作都通过事件来记录和回放。
