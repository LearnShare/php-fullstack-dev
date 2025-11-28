# 3.10 数据流设计进阶

## 目标

- 理解 CQRS（Command Query Responsibility Segregation）模式的核心概念。
- 掌握命令（Command）与查询（Query）的分离原则。
- 了解事件溯源（Event Sourcing）的基本概念与应用场景。
- 熟悉事件总线（Event Bus）的设计与实现。

## CQRS 模式

### CQRS 核心概念

CQRS（Command Query Responsibility Segregation）将数据操作分为：

- **Command（命令）**：修改状态的操作，不返回数据。
- **Query（查询）**：读取数据的操作，不修改状态。

```
┌─────────────┐
│   Command   │ → 写入模型 → 数据库（写）
└─────────────┘

┌─────────────┐
│    Query    │ → 读取模型 → 数据库（读）
└─────────────┘
```

### Command 实现

```php
namespace App\Application\Commands;

interface Command
{
}

interface CommandHandler
{
    public function handle(Command $command): void;
}

class CreateUserCommand implements Command
{
    public function __construct(
        public readonly string $name,
        public readonly string $email
    ) {
    }
}

class CreateUserCommandHandler implements CommandHandler
{
    public function __construct(
        private UserRepository $userRepository
    ) {
    }

    public function handle(Command $command): void
    {
        if (!$command instanceof CreateUserCommand) {
            throw new InvalidArgumentException('Invalid command type');
        }

        $email = new Email($command->email);
        
        if ($this->userRepository->existsByEmail($email)) {
            throw new EmailAlreadyExistsException($command->email);
        }

        $user = new User(null, $command->name, $email);
        $this->userRepository->save($user);
    }
}
```

### Query 实现

```php
namespace App\Application\Queries;

interface Query
{
}

interface QueryHandler
{
    public function handle(Query $query): mixed;
}

class GetUserQuery implements Query
{
    public function __construct(
        public readonly int $userId
    ) {
    }
}

class GetUserQueryHandler implements QueryHandler
{
    public function __construct(
        private UserRepository $userRepository
    ) {
    }

    public function handle(Query $query): ?UserDTO
    {
        if (!$query instanceof GetUserQuery) {
            throw new InvalidArgumentException('Invalid query type');
        }

        $user = $this->userRepository->findById($query->userId);
        
        if ($user === null) {
            return null;
        }

        return new UserDTO(
            $user->getId()->getValue(),
            $user->getName(),
            $user->getEmail()->getValue()
        );
    }
}

class UserDTO
{
    public function __construct(
        public readonly int $id,
        public readonly string $name,
        public readonly string $email
    ) {
    }
}
```

### Command/Query 总线

```php
namespace App\Application;

class CommandBus
{
    private array $handlers = [];

    public function register(string $commandClass, CommandHandler $handler): void
    {
        $this->handlers[$commandClass] = $handler;
    }

    public function dispatch(Command $command): void
    {
        $commandClass = get_class($command);
        
        if (!isset($this->handlers[$commandClass])) {
            throw new HandlerNotFoundException("No handler for {$commandClass}");
        }

        $this->handlers[$commandClass]->handle($command);
    }
}

class QueryBus
{
    private array $handlers = [];

    public function register(string $queryClass, QueryHandler $handler): void
    {
        $this->handlers[$queryClass] = $handler;
    }

    public function dispatch(Query $query): mixed
    {
        $queryClass = get_class($query);
        
        if (!isset($this->handlers[$queryClass])) {
            throw new HandlerNotFoundException("No handler for {$queryClass}");
        }

        return $this->handlers[$queryClass]->handle($query);
    }
}
```

### 使用示例

```php
// 注册处理器
$commandBus = new CommandBus();
$commandBus->register(CreateUserCommand::class, new CreateUserCommandHandler($userRepository));

$queryBus = new QueryBus();
$queryBus->register(GetUserQuery::class, new GetUserQueryHandler($userRepository));

// 执行命令
$commandBus->dispatch(new CreateUserCommand('Alice', 'alice@example.com'));

// 执行查询
$user = $queryBus->dispatch(new GetUserQuery(1));
```

## 事件溯源（Event Sourcing）

### 事件溯源概念

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

### 事件定义

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

### 事件存储

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
            $events[] = unserialize($row['event_data']);
        }
        
        return $events;
    }
}
```

### 聚合重建

```php
class Order
{
    private array $items = [];
    private OrderStatus $status;

    /**
     * 从事件流重建聚合
     * 
     * 这是事件溯源的核心方法：通过重放所有历史事件来重建聚合的当前状态
     * 
     * @param array $events 事件数组，按时间顺序排列
     * @return self 重建后的订单聚合
     */
    public static function fromEvents(array $events): self
    {
        // 创建新的订单实例（空状态）
        $order = new self();
        
        // 按顺序应用每个事件，逐步重建聚合状态
        // 这确保了聚合状态与事件历史完全一致
        foreach ($events as $event) {
            $order->apply($event);
        }
        
        return $order;
    }

    /**
     * 应用领域事件到聚合
     * 
     * 使用 match 表达式根据事件类型调用对应的应用方法
     * 每个事件类型都有对应的 apply 方法，用于更新聚合状态
     * 
     * @param DomainEvent $event 要应用的领域事件
     * @throws UnknownEventException 如果事件类型未知
     */
    private function apply(DomainEvent $event): void
    {
        // 根据事件类型分发到对应的处理方法
        match (get_class($event)) {
            OrderCreated::class => $this->applyOrderCreated($event),      // 订单创建事件
            ItemAdded::class => $this->applyItemAdded($event),            // 添加商品事件
            OrderConfirmed::class => $this->applyOrderConfirmed($event), // 订单确认事件
            default => throw new UnknownEventException(get_class($event)), // 未知事件类型
        };
    }

    private function applyOrderCreated(OrderCreated $event): void
    {
        $this->id = new OrderId($event->orderId);
        $this->customerId = new CustomerId($event->customerId);
        $this->status = OrderStatus::PENDING;
    }

    private function applyItemAdded(ItemAdded $event): void
    {
        $this->items[] = new OrderItem(
            new ProductId($event->productId),
            $event->quantity
        );
    }

    private function applyOrderConfirmed(OrderConfirmed $event): void
    {
        $this->status = OrderStatus::CONFIRMED;
    }
}
```

## 事件总线（Event Bus）

### 事件总线设计

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

### 事件处理器

```php
class OrderCreatedHandler
{
    public function __construct(
        private EmailService $emailService,
        private NotificationService $notificationService
    ) {
    }

    public function handle(OrderCreated $event): void
    {
        // 发送确认邮件
        $this->emailService->send(
            $event->customerId,
            'Order Created',
            "Your order {$event->orderId} has been created"
        );

        // 发送通知
        $this->notificationService->notify(
            $event->customerId,
            "Order {$event->orderId} created"
        );
    }
}

// 注册处理器
$eventBus = new SimpleEventBus();
$eventBus->subscribe(
    OrderCreated::class,
    [new OrderCreatedHandler($emailService, $notificationService), 'handle']
);
```

### 领域事件发布

```php
class Order
{
    private array $domainEvents = [];

    public function create(CustomerId $customerId): void
    {
        $this->id = OrderId::generate();
        $this->customerId = $customerId;
        $this->status = OrderStatus::PENDING;
        
        $this->recordEvent(new OrderCreated(
            $this->id->getValue(),
            $customerId->getValue()
        ));
    }

    private function recordEvent(DomainEvent $event): void
    {
        $this->domainEvents[] = $event;
    }

    public function getDomainEvents(): array
    {
        return $this->domainEvents;
    }

    public function clearDomainEvents(): void
    {
        $this->domainEvents = [];
    }
}

// 在应用服务中发布事件
class OrderService
{
    public function __construct(
        private OrderRepository $repository,
        private EventBus $eventBus
    ) {
    }

    public function createOrder(CustomerId $customerId): Order
    {
        $order = new Order();
        $order->create($customerId);
        
        $this->repository->save($order);
        
        // 发布领域事件
        foreach ($order->getDomainEvents() as $event) {
            $this->eventBus->publish($event);
        }
        
        $order->clearDomainEvents();
        
        return $order;
    }
}
```

## CQRS + 事件溯源组合

### 写入端（Command Side）

```php
class CreateOrderCommandHandler
{
    public function __construct(
        private EventStore $eventStore
    ) {
    }

    public function handle(CreateOrderCommand $command): void
    {
        $order = Order::create($command->customerId);
        
        foreach ($order->getDomainEvents() as $event) {
            $this->eventStore->append($order->getId()->getValue(), $event);
        }
    }
}
```

### 读取端（Query Side）

```php
class OrderReadModel
{
    public function __construct(
        private PDO $db
    ) {
    }

    public function getOrder(string $orderId): ?OrderView
    {
        $stmt = $this->db->prepare('SELECT * FROM order_views WHERE id = ?');
        $stmt->execute([$orderId]);
        $data = $stmt->fetch(PDO::FETCH_ASSOC);
        
        return $data ? new OrderView($data) : null;
    }
}

// 事件处理器更新读模型
class OrderCreatedProjection
{
    public function __construct(private PDO $db)
    {
    }

    public function handle(OrderCreated $event): void
    {
        $stmt = $this->db->prepare(
            'INSERT INTO order_views (id, customer_id, status, created_at)
             VALUES (?, ?, ?, ?)'
        );
        $stmt->execute([
            $event->orderId,
            $event->customerId,
            'pending',
            $event->occurredOn()->format('Y-m-d H:i:s'),
        ]);
    }
}
```

## 最佳实践

### 1. 命令应该是意图

- 命令名称应该表达业务意图，而非技术操作。

```php
// 不推荐
class UpdateUserEmailCommand

// 推荐
class ChangeUserEmailCommand
```

### 2. 查询返回 DTO

- 查询应该返回数据传输对象（DTO），而非领域实体。

### 3. 事件应该是过去时

- 事件名称使用过去时，表示已发生的事实。

```php
OrderCreated
ItemAdded
OrderShipped
```

### 4. 保持事件简单

- 事件应该包含必要的数据，避免过度复杂。

## 练习

1. 实现一个简单的 CQRS 系统，包含 `CreateUserCommand`、`GetUserQuery` 和对应的处理器。

2. 创建一个事件溯源系统，实现 `Order` 聚合的事件存储和重建功能。

3. 设计一个事件总线，支持同步和异步事件处理。

4. 实现一个读模型投影（Projection），监听领域事件并更新读模型。

5. 创建一个命令验证器，在命令处理前验证命令的有效性。

6. 设计一个事件版本控制系统，支持事件结构的演进和迁移。
