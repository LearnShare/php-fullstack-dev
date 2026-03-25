# 7.7.2 事件溯源（Event Sourcing）

## 概述

事件溯源（Event Sourcing）是一种以事件为中心来构建系统状态的架构模式。在传统的持久化模式中，我们存储的是系统的当前状态（快照）；而在事件溯源中，我们存储的是导致状态变化的所有事件，通过重放这些事件可以重建系统的任何历史状态。

事件溯源的核心思想可以用一个简单的比喻来理解：把系统状态看作一本账本的当前余额，而事件溯源记录的是所有的交易记录。通过所有的交易记录，我们可以知道当前余额是如何形成的，也可以回溯到任何一个时间点的状态。

事件溯源与 CQRS 是天然的组合。在事件溯源中，写操作产生事件，事件被存储到事件存储中；读操作通过重放事件来重建状态，或者通过预计算的投影来快速获取数据。这种模式在审计追踪、时间旅行查询、业务流程可视化等场景中特别有价值。

**主要内容**：
- 事件溯源的基本概念
- 事件存储的设计
- 事件重放机制
- 快照技术
- 事件版本管理
- 完整的代码示例

## 特性

- **完整的历史记录**：存储所有导致状态变化的事件
- **时间回溯**：可以重建任何历史时间点的状态
- **审计追踪**：天然支持业务操作的审计
- **可重放性**：可以通过重放事件来调试和测试
- **事件驱动**：与 CQRS 和事件驱动架构完美配合

## 核心概念

### 事件溯源 vs 传统持久化

传统持久化方式直接存储对象的当前状态：
```
用户表：id=1, name=张三, email=zhangsan@example.com, status=active
```

事件溯源存储所有导致状态变化的事件：
```
事件表：
- UserCreatedEvent { id: 1, name: 张三, email: zhangsan@example.com }
- UserEmailChangedEvent { id: 1, old: zhangsan@example.com, new: new@example.com }
- UserStatusChangedEvent { id: 1, old: inactive, new: active }
```

通过重放这些事件，可以重建用户的任何历史状态。

### 事件存储

事件存储（Event Store）是事件溯源的核心组件，负责事件的持久化和检索。一个事件存储应该支持以下操作：

- **追加事件**：将新事件追加到事件流中
- **读取事件**：根据聚合 ID 读取所有相关事件
- **读取版本**：读取特定版本的事件
- **事件投影**：创建不同的数据视图

### 事件版本

事件溯源中的版本管理非常重要。当领域模型演进时，新版本的事件可能与旧版本不同。事件存储需要能够处理事件的不同版本，支持事件的升级和迁移。

## 基本用法

### 1. 事件定义

```php
<?php
declare(strict_types=1);

namespace App\Domain\Event;

interface DomainEvent
{
    public function getEventId(): string;
    public function getOccurredAt(): \DateTimeImmutable;
    public function getAggregateId(): string;
    public function getVersion(): int;
}
```

```php
<?php
declare(strict_types=1);

namespace App\Domain\Event;

class AccountCreatedEvent implements DomainEvent
{
    public function __construct(
        public readonly string $accountId,
        public readonly string $accountNumber,
        public readonly string $ownerName,
        public readonly int $initialBalance
    ) {}

    public function getEventId(): string
    {
        return $this->eventId ??= uniqid();
    }

    public function getOccurredAt(): \DateTimeImmutable
    {
        return $this->occurredAt ??= new \DateTimeImmutable();
    }

    public function getAggregateId(): string
    {
        return $this->accountId;
    }

    public function getVersion(): int
    {
        return 1;
    }
}
```

```php
<?php
declare(strict_types=1);

namespace App\Domain\Event;

class MoneyDepositedEvent implements DomainEvent
{
    public function __construct(
        public readonly string $accountId,
        public readonly int $amount,
        public readonly string $description
    ) {}

    public function getEventId(): string
    {
        return $this->eventId ??= uniqid();
    }

    public function getOccurredAt(): \DateTimeImmutable
    {
        return $this->occurredAt ??= new \DateTimeImmutable();
    }

    public function getAggregateId(): string
    {
        return $this->accountId;
    }

    public function getVersion(): int
    {
        return 1;
    }
}
```

```php
<?php
declare(strict_types=1);

namespace App\Domain\Event;

class MoneyWithdrawnEvent implements DomainEvent
{
    public function __construct(
        public readonly string $accountId,
        public readonly int $amount,
        public readonly string $description
    ) {}

    public function getEventId(): string
    {
        return $this->eventId ??= uniqid();
    }

    public function getOccurredAt(): \DateTimeImmutable
    {
        return $this->occurredAt ??= new \DateTimeImmutable();
    }

    public function getAggregateId(): string
    {
        return $this->accountId;
    }

    public function getVersion(): int
    {
        return 1;
    }
}
```

### 2. 聚合根实现

```php
<?php
declare(strict_types=1);

namespace App\Domain\Entity;

use App\Domain\Event\DomainEvent;
use App\Domain\Event\AccountCreatedEvent;
use App\Domain\Event\MoneyDepositedEvent;
use App\Domain\Event\MoneyWithdrawnEvent;

class Account
{
    private string $accountId;
    private string $accountNumber;
    private string $ownerName;
    private int $balance;
    private int $version = 0;
    private array $pendingEvents = [];

    private function __construct() {}

    public static function create(
        string $accountId,
        string $accountNumber,
        string $ownerName,
        int $initialBalance = 0
    ): self {
        $account = new self();
        $account->accountId = $accountId;
        $account->accountNumber = $accountNumber;
        $account->ownerName = $ownerName;
        $account->balance = $initialBalance;
        
        $account->apply(new AccountCreatedEvent(
            accountId: $accountId,
            accountNumber: $accountNumber,
            ownerName: $ownerName,
            initialBalance: $initialBalance
        ));

        return $account;
    }

    public static function fromHistory(array $events): self
    {
        $account = new self();
        foreach ($events as $event) {
            $account->apply($event, false);
        }
        return $account;
    }

    public function deposit(int $amount, string $description): void
    {
        if ($amount <= 0) {
            throw new \InvalidArgumentException('Deposit amount must be positive');
        }

        $this->balance += $amount;
        $this->apply(new MoneyDepositedEvent(
            accountId: $this->accountId,
            amount: $amount,
            description: $description
        ));
    }

    public function withdraw(int $amount, string $description): void
    {
        if ($amount <= 0) {
            throw new \InvalidArgumentException('Withdrawal amount must be positive');
        }

        if ($amount > $this->balance) {
            throw new \DomainException('Insufficient balance');
        }

        $this->balance -= $amount;
        $this->apply(new MoneyWithdrawnEvent(
            accountId: $this->accountId,
            amount: $amount,
            description: $description
        ));
    }

    private function apply(DomainEvent $event, bool $isNew = true): void
    {
        match (true) {
            $event instanceof AccountCreatedEvent => $this->applyAccountCreated($event),
            $event instanceof MoneyDepositedEvent => $this->applyMoneyDeposited($event),
            $event instanceof MoneyWithdrawnEvent => $this->applyMoneyWithdrawn($event),
            default => throw new \RuntimeException('Unknown event type')
        };

        if ($isNew) {
            $this->pendingEvents[] = $event;
        }
        
        $this->version++;
    }

    private function applyAccountCreated(AccountCreatedEvent $event): void
    {
        $this->accountId = $event->accountId;
        $this->accountNumber = $event->accountNumber;
        $this->ownerName = $event->ownerName;
        $this->balance = $event->initialBalance;
    }

    private function applyMoneyDeposited(MoneyDepositedEvent $event): void
    {
        $this->balance += $event->amount;
    }

    private function applyMoneyWithdrawn(MoneyWithdrawnEvent $event): void
    {
        $this->balance -= $event->amount;
    }

    public function pullPendingEvents(): array
    {
        $events = $this->pendingEvents;
        $this->pendingEvents = [];
        return $events;
    }

    public function getAccountId(): string { return $this->accountId; }
    public function getBalance(): int { return $this->balance; }
    public function getVersion(): int { return $this->version; }
}
```

### 3. 事件存储实现

```php
<?php
declare(strict_types=1);

namespace App\Port\Output;

use App\Domain\Event\DomainEvent;

interface EventStorePort
{
    public function append(string $aggregateId, DomainEvent $event): void;
    public function getEvents(string $aggregateId): array;
    public function getEventsFromVersion(string $aggregateId, int $version): array;
}
```

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\EventStore;

use App\Domain\Event\DomainEvent;
use App\Port\Output\EventStorePort;
use PDO;
use JsonSerializable;

class MysqlEventStore implements EventStorePort
{
    public function __construct(private readonly PDO $pdo) {}

    public function append(string $aggregateId, DomainEvent $event): void
    {
        $stmt = $this->pdo->prepare(
            'INSERT INTO events (aggregate_id, event_type, event_data, occurred_at, version) 
             VALUES (:aggregate_id, :event_type, :event_data, :occurred_at, :version)'
        );

        $stmt->execute([
            'aggregate_id' => $aggregateId,
            'event_type' => get_class($event),
            'event_data' => json_encode($event),
            'occurred_at' => $event->getOccurredAt()->format('Y-m-d H:i:s'),
            'version' => $event->getVersion()
        ]);
    }

    public function getEvents(string $aggregateId): array
    {
        $stmt = $this->pdo->prepare(
            'SELECT * FROM events WHERE aggregate_id = :aggregate_id ORDER BY version ASC'
        );
        $stmt->execute(['aggregate_id' => $aggregateId]);
        $rows = $stmt->fetchAll(PDO::FETCH_ASSOC);

        return array_map([$this, 'deserializeEvent'], $rows);
    }

    public function getEventsFromVersion(string $aggregateId, int $version): array
    {
        $stmt = $this->pdo->prepare(
            'SELECT * FROM events WHERE aggregate_id = :aggregate_id AND version > :version ORDER BY version ASC'
        );
        $stmt->execute(['aggregate_id' => $aggregateId, 'version' => $version]);
        $rows = $stmt->fetchAll(PDO::FETCH_ASSOC);

        return array_map([$this, 'deserializeEvent'], $rows);
    }

    private function deserializeEvent(array $row): DomainEvent
    {
        $eventData = json_decode($row['event_data'], true);
        $eventClass = $row['event_type'];
        
        return new $eventClass(...$eventData);
    }
}
```

### 4. 快照机制

```php
<?php
declare(strict_types=1);

namespace App\Port\Output;

use App\Domain\Entity\Account;

interface SnapshotStorePort
{
    public function save(Account $account): void;
    public function find(string $aggregateId): ?Account;
}
```

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\Snapshot;

use App\Domain\Entity\Account;
use App\Port\Output\SnapshotStorePort;
use PDO;

class MysqlSnapshotStore implements SnapshotStorePort
{
    private const SNAPSHOT_INTERVAL = 100;

    public function __construct(
        private readonly PDO $pdo,
        private readonly \App\Port\Output\EventStorePort $eventStore
    ) {}

    public function save(Account $account): void
    {
        if ($account->getVersion() % self::SNAPSHOT_INTERVAL !== 0) {
            return;
        }

        $stmt = $this->pdo->prepare(
            'INSERT INTO snapshots (aggregate_id, aggregate_type, snapshot_data, version) 
             VALUES (:aggregate_id, :aggregate_type, :snapshot_data, :version)
             ON DUPLICATE KEY UPDATE snapshot_data = :snapshot_data, version = :version'
        );

        $stmt->execute([
            'aggregate_id' => $account->getAccountId(),
            'aggregate_type' => Account::class,
            'snapshot_data' => json_encode([
                'accountId' => $account->getAccountId(),
                'balance' => $account->getBalance()
            ]),
            'version' => $account->getVersion()
        ]);
    }

    public function find(string $aggregateId): ?Account
    {
        $stmt = $this->pdo->prepare(
            'SELECT * FROM snapshots WHERE aggregate_id = :aggregate_id ORDER BY version DESC LIMIT 1'
        );
        $stmt->execute(['aggregate_id' => $aggregateId]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);

        if ($row === false) {
            return null;
        }

        $snapshotData = json_decode($row['snapshot_data'], true);
        
        $events = $this->eventStore->getEventsFromVersion(
            $aggregateId,
            $row['version']
        );

        $account = Account::fromHistory($events);
        
        return $account;
    }
}
```

### 5. 仓储实现

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\Repository;

use App\Domain\Entity\Account;
use App\Port\Output\AccountRepositoryPort;
use App\Port\Output\EventStorePort;
use App\Port\Output\SnapshotStorePort;

class EventSourcingAccountRepository implements AccountRepositoryPort
{
    public function __construct(
        private readonly EventStorePort $eventStore,
        private readonly SnapshotStorePort $snapshotStore,
        private readonly \App\Adapter\Output\EventStore\MysqlEventStore $mysqlEventStore
    ) {}

    public function save(Account $account): void
    {
        $events = $account->pullPendingEvents();
        
        foreach ($events as $event) {
            $this->eventStore->append($account->getAccountId(), $event);
        }

        $this->snapshotStore->save($account);
    }

    public function findById(string $accountId): ?Account
    {
        $snapshot = $this->snapshotStore->find($accountId);
        
        if ($snapshot !== null) {
            return $snapshot;
        }

        $events = $this->eventStore->getEvents($accountId);
        
        if (empty($events)) {
            return null;
        }

        return Account::fromHistory($events);
    }
}
```

## 使用场景

### 1. 审计需求

事件溯源天然支持业务操作的审计。每一个操作都被记录为事件，可以追溯任何历史操作。

### 2. 时间旅行

可以回溯到任何一个历史时间点的状态，用于调试、分析或业务决策。

### 3. 业务流程可视化

事件可以用于重建业务流程的状态，用于流程监控和可视化。

### 4. 复杂状态逻辑

当状态变化逻辑复杂时，事件溯源可以将变化逻辑清晰化，便于理解和维护。

## 注意事项

### 1. 事件不可变性

事件一旦存储就不应该修改。这需要确保事件的设计是完整的，不需要后续修改。

### 2. 性能考虑

对于长期运行的应用，事件数量可能非常大。需要使用快照机制来优化状态重建的性能。

### 3. 事件版本管理

领域模型演进时，需要处理不同版本的事件。

## 常见问题

### Q1: 事件溯源与 CQRS 的关系？

事件溯源是 CQRS 的一种实现方式。写操作产生事件，存储到事件存储；读操作通过投影事件来构建不同的数据视图。

### Q2: 事件溯源的缺点？

事件溯源会增加系统的复杂度，特别是需要处理事件版本、快照等。对于简单的 CRUD 应用，使用传统方式更合适。

### Q3: 如何处理大量事件？

使用快照机制，定期创建聚合的快照。只重放快照之后的事件，减少重放的事件数量。

## 最佳实践

### 1. 设计简洁的事件

事件应该包含足够的信息来重建状态，但不要过度设计。

### 2. 实现快照机制

对于长期运行的聚合，实现快照机制来优化性能。

### 3. 处理事件版本

设计事件时考虑版本升级，使用升级器处理不同版本的事件。

### 4. 考虑最终一致性

事件写入和快照创建应该考虑事务边界，确保数据一致性。

## 练习任务

### 练习 1：实现账户事件溯源

使用事件溯源实现一个银行账户系统，支持存款、取款操作，并记录所有历史事件。

### 2：设计快照策略

为账户系统设计快照策略，确定快照的频率和存储方式。

### 3：实现事件查询

实现一个查询接口，可以查询账户在任意时间点的余额。

### 4：实现事件升级器

设计一个事件升级器，处理旧版本事件到新版本事件的转换。

### 5：分析适用场景

分析哪些业务场景适合使用事件溯源，哪些场景不适合。
