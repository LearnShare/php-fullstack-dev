# 7.5.4 六边形架构实践

## 概述

六边形架构（Hexagonal Architecture）是一种追求应用核心与外部技术完全解耦的软件架构模式。在前面的章节中，我们已经学习了六边形架构的核心概念、端口设计和适配器实现。本节将综合运用这些知识，通过完整的实践示例，帮助零基础学员掌握六边形架构的实际应用。

六边形架构的核心理念是"端口与适配器"模式，应用核心（业务逻辑）位于架构的中心，不依赖于任何外部技术。外部世界通过适配器与端口进行交互，适配器负责技术实现，端口定义交互规范。这种设计使得应用核心可以独立于技术细节进行开发和测试。

本节将通过一个完整的订单管理系统示例，展示如何从零开始构建六边形架构的应用。我们将涵盖目录结构设计、依赖管理、完整代码实现、测试策略等内容。

**主要内容**：
- 目录结构设计
- 依赖管理策略
- 完整代码实现
- 测试策略
- 架构演进方法

## 特性

- **技术无关性**：业务逻辑不依赖任何特定技术，可以轻松切换数据库、缓存、消息队列等
- **高度可测试**：业务逻辑可以通过单元测试完全覆盖，不依赖外部系统
- **清晰的分层**：通过端口和适配器的分离，代码结构清晰，职责明确
- **可替换性**：任何外部技术都可以在不修改业务逻辑的情况下替换
- **团队协作友好**：不同团队可以并行开发不同的适配器

## 核心概念

### 目录结构设计

六边形架构的目录结构应该清晰地反映架构的分层思想。以下是一个推荐的目录结构：

```
src/
├── Domain/                    # 领域层 - 核心业务逻辑
│   ├── Entity/               # 领域实体
│   │   └── Order.php
│   ├── ValueObject/         # 值对象
│   │   └── Money.php
│   ├── Service/             # 领域服务
│   │   └── OrderDomainService.php
│   └── Event/               # 领域事件
│       └── OrderCreatedEvent.php
│
├── Application/              # 应用层 - 用例编排
│   ├── Command/             # 命令对象
│   │   └── CreateOrderCommand.php
│   ├── Query/               # 查询对象
│   │   └── GetOrderQuery.php
│   ├── Dto/                 # 数据传输对象
│   │   └── OrderDto.php
│   └── UseCase/             # 用例实现
│       ├── CreateOrderUseCase.php
│       └── GetOrderUseCase.php
│
├── Port/                     # 端口层 - 接口定义
│   ├── Input/               # 输入端口（Driving Ports）
│   │   ├── CreateOrderPort.php
│   │   └── GetOrderPort.php
│   └── Output/              # 输出端口（Driven Ports）
│       ├── OrderRepositoryPort.php
│       ├── PaymentGatewayPort.php
│       └── NotificationPort.php
│
├── Adapter/                  # 适配器层 - 技术实现
│   ├── Input/               # 输入适配器
│   │   ├── Http/
│   │   │   └── OrderController.php
│   │   └── Cli/
│   │       └── OrderCommand.php
│   └── Output/              # 输出适配器
│       ├── Database/
│       │   └── MysqlOrderRepository.php
│       ├── Api/
│       │   └── StripePaymentAdapter.php
│       └── Notification/
│           └── EmailAdapter.php
│
└── Infrastructure/           # 基础设施层
    ├── Bootstrap/
    │   └── Application.php
    └── Config/
        └── config.php
```

这种目录结构清晰地分离了不同层次的代码，使得代码的职责边界非常明确。新加入项目的开发者可以通过目录结构快速理解整个应用的架构。

### 依赖管理

在六边形架构中，依赖方向必须严格控制：适配器依赖端口，端口位于应用层，应用层依赖领域层。这种依赖方向确保了领域层的纯净性，不受外部技术的影响。

```
┌────────────────────────────────────────────────────────────┐
│                        适配器层                             │
│  (Http/Controller, Database/Repository, Api/Client)       │
└──────────────────────────┬─────────────────────────────────┘
                           │ 依赖
                           ▼
┌────────────────────────────────────────────────────────────┐
│                        端口层                               │
│          (Input/Output Interfaces)                          │
└──────────────────────────┬─────────────────────────────────┘
                           │ 依赖
                           ▼
┌────────────────────────────────────────────────────────────┐
│                        应用层                               │
│                  (UseCases, Commands)                       │
└──────────────────────────┬─────────────────────────────────┘
                           │ 依赖
                           ▼
┌────────────────────────────────────────────────────────────┐
│                        领域层                               │
│               (Entities, Value Objects)                    │
└────────────────────────────────────────────────────────────┘
```

## 基本用法

### 1. 领域层实现

领域层包含核心的业务实体和业务规则，是整个应用的核心。

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

    public function calculateTotal(): Money
    {
        $total = 0;
        foreach ($this->items as $item) {
            $total += $item->getSubtotal()->getAmount();
        }
        return new Money($total, $this->totalAmount->getCurrency());
    }

    private function recalculateTotal(): void
    {
        $this->totalAmount = $this->calculateTotal();
    }

    public function confirm(): void
    {
        if ($this->status !== OrderStatus::PENDING) {
            throw new \InvalidArgumentException('Only pending orders can be confirmed');
        }
        $this->status = OrderStatus::CONFIRMED;
    }

    public function cancel(): void
    {
        if (!in_array($this->status, [OrderStatus::PENDING, OrderStatus::CONFIRMED])) {
            throw new \InvalidArgumentException('Order cannot be cancelled in current status');
        }
        $this->status = OrderStatus::CANCELLED;
    }

    public function complete(): void
    {
        if ($this->status !== OrderStatus::CONFIRMED) {
            throw new \InvalidArgumentException('Only confirmed orders can be completed');
        }
        $this->status = OrderStatus::COMPLETED;
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

领域实体封装了核心的业务规则，如订单状态的转换逻辑、订单金额的计算逻辑等。这些业务规则不应该依赖于任何外部技术，只依赖于领域内部的值对象。

### 2. 值对象实现

值对象是不可变的、用来描述领域概念的对象。

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
            throw new \InvalidArgumentException('Cannot add money with different currencies');
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
}
```

```php
<?php
declare(strict_types=1);

namespace App\Domain\ValueObject;

enum OrderStatus: string
{
    case PENDING = 'pending';
    case CONFIRMED = 'confirmed';
    case CANCELLED = 'cancelled';
    case COMPLETED = 'completed';
}
```

### 3. 端口层实现

端口层定义应用与外部世界交互的接口规范。

```php
<?php
declare(strict_types=1);

namespace App\Port\Input;

use App\Application\Dto\CreateOrderDto;

interface CreateOrderPort
{
    public function execute(CreateOrderDto $dto): OrderResultDto;
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

端口接口的定义应该使用业务语言，避免暴露技术实现细节。例如，`OrderRepositoryPort` 定义了 `findByCustomerId` 这样的业务方法名，而不是 `selectByCustomerId` 这样的技术方法名。

### 4. 应用层实现

应用层负责用例的编排，调用领域服务并通过端口与外部世界交互。

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
        public readonly string $currency,
        public readonly string $createdAt
    ) {}
}
```

```php
<?php
declare(strict_types=1);

namespace App\Application\UseCase;

use App\Port\Input\CreateOrderPort;
use App\Port\Output\OrderRepositoryPort;
use App\Port\Output\PaymentGatewayPort;
use App\Port\Output\NotificationPort;
use App\Application\Dto\CreateOrderDto;
use App\Application\Dto\OrderResultDto;
use App\Domain\Entity\Order;
use App\Domain\Entity\OrderItem;
use App\Domain\ValueObject\Money;

class CreateOrderUseCase implements CreateOrderPort
{
    public function __construct(
        private readonly OrderRepositoryPort $orderRepository,
        private readonly PaymentGatewayPort $paymentGateway,
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
        $order->setId($orderId);

        $this->notification->sendOrderConfirmation($order);

        return new OrderResultDto(
            orderId: $orderId,
            status: $order->getStatus()->value,
            totalAmount: $order->getTotalAmount()->getAmount(),
            currency: $order->getTotalAmount()->getCurrency(),
            createdAt: (new \DateTimeImmutable())->format('Y-m-d H:i:s')
        );
    }
}
```

用例的实现完全依赖于端口接口，不直接依赖任何适配器实现。这种设计使得用例可以在不同的环境中使用不同的适配器实现。

### 5. 适配器层实现

适配器层实现端口接口，处理具体的技术细节。

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\Database;

use App\Domain\Entity\Order;
use App\Domain\ValueObject\OrderStatus;
use App\Port\Output\OrderRepositoryPort;
use PDO;

class MysqlOrderRepository implements OrderRepositoryPort
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

    public function findByCustomerId(int $customerId): array
    {
        $stmt = $this->pdo->prepare('SELECT * FROM orders WHERE customer_id = ?');
        $stmt->execute([$customerId]);
        $rows = $stmt->fetchAll(PDO::FETCH_ASSOC);

        return array_map([$this, 'mapToOrder'], $rows);
    }

    public function save(Order $order): int
    {
        $stmt = $this->pdo->prepare(
            'INSERT INTO orders (customer_id, customer_email, status, total_amount, currency, created_at, updated_at) 
             VALUES (?, ?, ?, ?, ?, ?, ?)'
        );

        $now = (new \DateTimeImmutable())->format('Y-m-d H:i:s');
        $stmt->execute([
            $order->getCustomerId(),
            $order->getCustomerEmail(),
            $order->getStatus()->value,
            $order->getTotalAmount()->getAmount(),
            $order->getTotalAmount()->getCurrency(),
            $now,
            $now
        ]);

        return (int) $this->pdo->lastInsertId();
    }

    public function update(Order $order): void
    {
        $stmt = $this->pdo->prepare(
            'UPDATE orders SET status = ?, total_amount = ?, updated_at = ? WHERE id = ?'
        );

        $stmt->execute([
            $order->getStatus()->value,
            $order->getTotalAmount()->getAmount(),
            (new \DateTimeImmutable())->format('Y-m-d H:i:s'),
            $order->getId()
        ]);
    }

    public function delete(int $id): void
    {
        $stmt = $this->pdo->prepare('DELETE FROM orders WHERE id = ?');
        $stmt->execute([$id]);
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

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Input\Http;

use App\Port\Input\CreateOrderPort;
use App\Application\Dto\CreateOrderDto;
use App\Application\Dto\OrderItemDto;
use App\Application\Dto\OrderResultDto;

class OrderController
{
    public function __construct(private readonly CreateOrderPort $createOrderUseCase) {}

    public function create(array $requestData): array
    {
        $items = array_map(fn($item) => new OrderItemDto(
            productId: $item['product_id'],
            productName: $item['product_name'],
            quantity: (int) $item['quantity'],
            unitPrice: (int) $item['unit_price']
        ), $requestData['items'] ?? []);

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
                'currency' => $result->currency,
                'created_at' => $result->createdAt
            ]
        ];
    }
}
```

### 6. 依赖注入配置

最后，通过依赖注入容器将所有组件组装在一起。

```php
<?php
declare(strict_types=1);

namespace App\Infrastructure\Bootstrap;

use App\Adapter\Input\Http\OrderController;
use App\Adapter\Output\Database\MysqlOrderRepository;
use App\Application\UseCase\CreateOrderUseCase;
use App\Port\Output\OrderRepositoryPort;
use App\Port\Output\PaymentGatewayPort;
use App\Port\Output\NotificationPort;
use PDO;

class Application
{
    private array $config;
    private array $services = [];

    public function __construct(array $config)
    {
        $this->config = $config;
    }

    public function getOrderController(): OrderController
    {
        return new OrderController(
            $this->getCreateOrderUseCase()
        );
    }

    private function getCreateOrderUseCase(): CreateOrderUseCase
    {
        return new CreateOrderUseCase(
            $this->getOrderRepository(),
            $this->getPaymentGateway(),
            $this->getNotificationService()
        );
    }

    private function getOrderRepository(): OrderRepositoryPort
    {
        if (!isset($this->services['orderRepository'])) {
            $pdo = new PDO(
                $this->config['database']['dsn'],
                $this->config['database']['username'],
                $this->config['database']['password']
            );
            $this->services['orderRepository'] = new MysqlOrderRepository($pdo);
        }
        return $this->services['orderRepository'];
    }

    private function getPaymentGateway(): PaymentGatewayPort
    {
        return $this->services['paymentGateway'] ??= new \App\Adapter\Output\Api\StripePaymentAdapter(
            $this->config['payment']['api_key']
        );
    }

    private function getNotificationService(): NotificationPort
    {
        return $this->services['notification'] ??= new \App\Adapter\Output\Notification\EmailAdapter(
            $this->config['mail']['smtp_host']
        );
    }
}
```

## 测试策略

### 1. 单元测试

单元测试针对业务逻辑进行测试，不依赖任何外部系统。使用 mock 对象模拟端口的实现。

```php
<?php
declare(strict_types=1);

namespace Tests\Unit\Application\UseCase;

use PHPUnit\Framework\TestCase;
use App\Application\UseCase\CreateOrderUseCase;
use App\Application\Dto\CreateOrderDto;
use App\Application\Dto\OrderItemDto;
use App\Port\Output\OrderRepositoryPort;
use App\Port\Output\PaymentGatewayPort;
use App\Port\Output\NotificationPort;
use App\Domain\Entity\Order;

class CreateOrderUseCaseTest extends TestCase
{
    private CreateOrderUseCase $useCase;
    private $orderRepository;
    private $paymentGateway;
    private $notification;

    protected function setUp(): void
    {
        $this->orderRepository = $this->createMock(OrderRepositoryPort::class);
        $this->paymentGateway = $this->createMock(PaymentGatewayPort::class);
        $this->notification = $this->createMock(NotificationPort::class);

        $this->useCase = new CreateOrderUseCase(
            $this->orderRepository,
            $this->paymentGateway,
            $this->notification
        );
    }

    public function testCreateOrderSuccessfully(): void
    {
        $dto = new CreateOrderDto(
            customerId: 1,
            customerEmail: 'test@example.com',
            items: [
                new OrderItemDto('P001', 'Product 1', 2, 1000)
            ]
        );

        $this->orderRepository
            ->expects($this->once())
            ->method('save')
            ->willReturn(1);

        $this->notification
            ->expects($this->once())
            ->method('sendOrderConfirmation');

        $result = $this->useCase->execute($dto);

        $this->assertEquals(1, $result->orderId);
        $this->assertEquals('pending', $result->status);
    }
}
```

### 2. 集成测试

集成测试针对适配器进行测试，测试适配器与外部系统的交互是否正确。

```php
<?php
declare(strict_types=1);

namespace Tests\Integration\Adapter\Database;

use PHPUnit\Framework\TestCase;
use App\Adapter\Output\Database\MysqlOrderRepository;
use App\Domain\Entity\Order;
use PDO;

class MysqlOrderRepositoryTest extends TestCase
{
    private MysqlOrderRepository $repository;
    private PDO $pdo;

    protected function setUp(): void
    {
        $this->pdo = new PDO('mysql:host=localhost;dbname=test', 'root', '');
        $this->repository = new MysqlOrderRepository($this->pdo);
        
        $this->pdo->exec('DELETE FROM orders');
    }

    public function testSaveAndFindOrder(): void
    {
        $order = new Order(
            customerId: 1,
            customerEmail: 'test@example.com'
        );

        $id = $this->repository->save($order);
        $foundOrder = $this->repository->findById($id);

        $this->assertNotNull($foundOrder);
        $this->assertEquals(1, $foundOrder->getCustomerId());
    }
}
```

## 使用场景

### 1. 新项目启动

对于新启动的项目，从一开始就采用六边形架构可以避免后期的重构成本。建议在项目初期就定义好端口接口，然后逐步实现适配器。

### 2. 遗留系统改造

对于已有的遗留系统，可以逐步引入六边形架构。先为现有系统定义端口接口，然后创建适配器，最后将业务逻辑迁移到用例中。

### 3. 第三方服务集成

当需要集成多个第三方服务时，六边形架构可以帮助统一管理外部依赖，降低服务变更对核心业务的影响。

## 注意事项

### 1. 避免过度工程

六边形架构是一种相对复杂的架构模式，对于小型项目可能过于笨重。应该根据项目的实际规模和复杂度选择合适的架构。

### 2. 依赖方向必须严格

确保适配器依赖端口，而不是端口依赖适配器。依赖注入是实现这一原则的关键技术。

### 3. 保持领域层纯净

领域层不应该依赖应用层或端口层。领域实体、值对象、领域服务等应该只依赖于领域内部的概念。

### 4. 接口稳定性

端口接口一旦发布，应该尽量保持稳定。如果需要修改接口，应该通过版本控制或向后兼容的方式进行。

## 最佳实践

### 1. 渐进式采用

不需要一次性将整个应用重构为六边形架构。可以从新功能开始，逐步改造遗留代码。

### 2. 使用依赖注入容器

使用依赖注入容器管理适配器和用例的依赖关系，使得依赖的切换更加灵活。

### 3. 编写集成测试

为每个适配器编写集成测试，确保适配器与外部系统的交互正确。

### 4. 文档化端口接口

为每个端口接口编写清晰的文档，说明接口的职责、参数、返回值等。

## 练习任务

### 练习 1：实现产品管理功能

使用六边形架构实现产品管理功能，包括产品的增删改查。创建 `Product` 实体、`ProductRepositoryPort` 端口、`MysqlProductRepository` 适配器和 `ProductController`。

### 练习 2：实现用户认证功能

实现用户注册和登录功能。创建 `User` 实体、`UserRepositoryPort` 端口、`PasswordHasherPort` 端口，以及相应的适配器实现。

### 练习 3：编写完整的测试套件

为订单管理功能编写完整的测试套件，包括用例的单元测试、仓储的集成测试、控制器的功能测试。

### 练习 4：切换数据库后端

将订单管理的数据库适配器从 MySQL 切换到 PostgreSQL，体验适配器可替换性的优势。

### 练习 5：实现缓存适配器

为订单管理添加缓存功能。创建 `CachePort` 接口，实现 Redis 缓存适配器，并在用例中使用缓存。
