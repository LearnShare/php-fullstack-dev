# 3.9 DDD 领域驱动设计初级入门

## 目标

- 理解 DDD（Domain-Driven Design）的核心概念与价值。
- 掌握实体（Entity）与值对象（Value Object）的区别与使用场景。
- 理解聚合根（Aggregate Root）的概念，掌握如何设计聚合边界。
- 熟悉仓储（Repository）模式，理解其在 DDD 中的作用。

## DDD 概述

### 什么是 DDD

领域驱动设计（Domain-Driven Design）是一种软件开发方法，强调：

- **以领域为中心**：业务逻辑是核心，技术实现是细节。
- **通用语言（Ubiquitous Language）**：开发团队与业务专家使用相同的术语。
- **模型驱动**：代码直接反映业务模型。

### 为什么需要 DDD

**问题**：传统开发方式的问题
- 业务逻辑分散在 Controller、Service、Repository 中
- 难以理解业务规则和约束
- 代码难以维护和扩展

**解决方案**：DDD 将业务逻辑集中在领域层
- 业务规则清晰可见
- 代码直接反映业务模型
- 易于理解和维护

### DDD 的分层（渐进式理解）

#### 第一步：理解分层架构

```
┌─────────────────────────┐
│  User Interface Layer   │ 表现层（HTTP、CLI、WebSocket）
│  - 接收用户输入          │
│  - 返回响应              │
├─────────────────────────┤
│  Application Layer       │ 应用层（用例编排）
│  - 协调领域对象          │
│  - 处理事务              │
│  - 调用基础设施服务      │
├─────────────────────────┤
│  Domain Layer           │ 领域层（核心业务逻辑）
│  - 实体（Entity）        │
│  - 值对象（Value Object）│
│  - 领域服务（Domain Service）│
│  - 聚合根（Aggregate Root）│
├─────────────────────────┤
│  Infrastructure Layer   │ 基础设施层
│  - 数据库访问            │
│  - 外部 API 调用        │
│  - 文件系统操作          │
└─────────────────────────┘
```

#### 第二步：理解依赖方向

```
依赖方向：从上到下
- 表现层 → 应用层 → 领域层
- 应用层 → 基础设施层
- 领域层 ← 基础设施层（通过接口）

关键原则：
- 领域层不依赖任何其他层
- 基础设施层实现领域层定义的接口
```

#### 第三步：实际应用示例

**简单示例：用户注册**

```php
<?php
declare(strict_types=1);

// 第一步：领域层定义实体和业务规则
namespace App\Domain\User;

/**
 * 用户实体（领域层）
 * 
 * 包含业务逻辑和业务规则
 */
class User
{
    public function __construct(
        private UserId $id,
        private Email $email,
        private string $name,
        private PasswordHash $passwordHash
    ) {
        // 业务规则：邮箱必须有效
        if (!$email->isValid()) {
            throw new InvalidEmailException("Invalid email: {$email->getValue()}");
        }
        
        // 业务规则：名称不能为空
        if (empty(trim($name))) {
            throw new InvalidNameException('Name cannot be empty');
        }
    }
    
    /**
     * 更改邮箱（业务操作）
     */
    public function changeEmail(Email $newEmail): void
    {
        // 业务规则：新邮箱必须与当前邮箱不同
        if ($newEmail->equals($this->email)) {
            throw new \DomainException('New email must be different from current email');
        }
        
        // 业务规则：新邮箱必须有效
        if (!$newEmail->isValid()) {
            throw new InvalidEmailException("Invalid email: {$newEmail->getValue()}");
        }
        
        $this->email = $newEmail;
    }
}

// 第二步：应用层编排用例
namespace App\Application;

use App\Domain\User\User;
use App\Domain\User\UserRepository;
use App\Domain\User\Email;

/**
 * 用户应用服务（应用层）
 * 
 * 编排领域对象，处理用例
 */
class UserApplicationService
{
    public function __construct(
        private UserRepository $userRepository
    ) {}
    
    /**
     * 注册用户（用例）
     */
    public function registerUser(string $email, string $name, string $password): User
    {
        // 1. 验证邮箱是否已存在（业务规则）
        $existingUser = $this->userRepository->findByEmail(new Email($email));
        if ($existingUser !== null) {
            throw new EmailAlreadyExistsException("Email already exists: {$email}");
        }
        
        // 2. 创建用户实体（领域对象）
        $user = new User(
            UserId::generate(),
            new Email($email),
            $name,
            PasswordHash::fromPlainText($password)
        );
        
        // 3. 保存用户（通过仓储）
        $this->userRepository->save($user);
        
        return $user;
    }
}

// 第三步：基础设施层实现仓储
namespace App\Infrastructure\Persistence;

use App\Domain\User\User;
use App\Domain\User\UserRepository;
use App\Domain\User\Email;
use App\Domain\User\UserId;
use PDO;

/**
 * 用户仓储实现（基础设施层）
 * 
 * 实现领域层定义的仓储接口
 */
class DatabaseUserRepository implements UserRepository
{
    public function __construct(
        private PDO $pdo
    ) {}
    
    public function findByEmail(Email $email): ?User
    {
        $stmt = $this->pdo->prepare(
            'SELECT id, email, name, password_hash FROM users WHERE email = ?'
        );
        $stmt->execute([$email->getValue()]);
        $data = $stmt->fetch(PDO::FETCH_ASSOC);
        
        if ($data === false) {
            return null;
        }
        
        return $this->mapToUser($data);
    }
    
    public function save(User $user): void
    {
        $stmt = $this->pdo->prepare(
            'INSERT INTO users (id, email, name, password_hash) 
             VALUES (?, ?, ?, ?)
             ON DUPLICATE KEY UPDATE 
                email = VALUES(email),
                name = VALUES(name),
                password_hash = VALUES(password_hash)'
        );
        
        $stmt->execute([
            $user->getId()->getValue(),
            $user->getEmail()->getValue(),
            $user->getName(),
            $user->getPasswordHash()->getValue(),
        ]);
    }
    
    private function mapToUser(array $data): User
    {
        return new User(
            new UserId($data['id']),
            new Email($data['email']),
            $data['name'],
            new PasswordHash($data['password_hash'])
        );
    }
}

// 第四步：表现层处理 HTTP 请求
namespace App\Http\Controllers;

use App\Application\UserApplicationService;

/**
 * 用户控制器（表现层）
 * 
 * 处理 HTTP 请求，调用应用服务
 */
class UserController
{
    public function __construct(
        private UserApplicationService $userService
    ) {}
    
    public function register(array $request): void
    {
        try {
            $input = json_decode($request['body'] ?? '{}', true);
            
            // 调用应用服务
            $user = $this->userService->registerUser(
                $input['email'] ?? '',
                $input['name'] ?? '',
                $input['password'] ?? ''
            );
            
            // 返回响应
            http_response_code(201);
            header('Content-Type: application/json');
            echo json_encode([
                'id' => $user->getId()->getValue(),
                'email' => $user->getEmail()->getValue(),
                'name' => $user->getName(),
            ]);
            
        } catch (EmailAlreadyExistsException $e) {
            http_response_code(409);
            echo json_encode(['error' => $e->getMessage()]);
        } catch (\Exception $e) {
            http_response_code(400);
            echo json_encode(['error' => $e->getMessage()]);
        }
    }
}
```

**完整示例说明**：

1. **领域层**：定义 `User` 实体和业务规则（邮箱验证、名称验证等）
2. **应用层**：`UserApplicationService` 编排用例（注册用户）
3. **基础设施层**：`DatabaseUserRepository` 实现数据访问
4. **表现层**：`UserController` 处理 HTTP 请求

**优势**：
- 业务逻辑集中在领域层，易于理解和维护
- 各层职责清晰，易于测试
- 可以轻松替换基础设施层（如从 MySQL 切换到 PostgreSQL）

## 实体（Entity）

### 实体特征

- **有唯一标识**：通过 ID 区分不同的实体实例。
- **可变**：实体的状态可以改变。
- **有生命周期**：可以被创建、修改、删除。

```php
namespace App\Domain\User;

class User
{
    public function __construct(
        private UserId $id,
        private string $name,
        private Email $email
    ) {
    }

    public function getId(): UserId
    {
        return $this->id;
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function changeName(string $newName): void
    {
        if (empty($newName)) {
            throw new InvalidArgumentException('Name cannot be empty');
        }
        $this->name = $newName;
    }

    public function changeEmail(Email $newEmail): void
    {
        $this->email = $newEmail;
    }
}
```

### 实体相等性

- 实体通过 ID 判断相等性，而非属性值。

```php
class User
{
    public function equals(User $other): bool
    {
        return $this->id->equals($other->id);
    }
}

$user1 = new User(new UserId(1), 'Alice', new Email('alice@example.com'));
$user2 = new User(new UserId(1), 'Bob', new Email('bob@example.com'));

// 即使属性不同，ID 相同就是同一个实体
$user1->equals($user2); // true
```

## 值对象（Value Object）

### 值对象特征

- **无标识**：通过属性值判断相等性。
- **不可变（Immutable）**：创建后不能修改。
- **可替换**：可以整体替换。

```php
namespace App\Domain\User;

readonly class Email
{
    public function __construct(private string $value)
    {
        if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidEmailException($value);
        }
    }

    public function getValue(): string
    {
        return $this->value;
    }

    public function equals(Email $other): bool
    {
        return $this->value === $other->value;
    }
}

readonly class Money
{
    public function __construct(
        public float $amount,
        public string $currency
    ) {
        if ($amount < 0) {
            throw new InvalidArgumentException('Amount cannot be negative');
        }
    }

    public function add(Money $other): Money
    {
        if ($this->currency !== $other->currency) {
            throw new CurrencyMismatchException();
        }
        return new Money($this->amount + $other->amount, $this->currency);
    }

    public function equals(Money $other): bool
    {
        return $this->amount === $other->amount 
            && $this->currency === $other->currency;
    }
}
```

### 值对象 vs 实体

| 特性         | 实体（Entity）           | 值对象（Value Object）     |
| :----------- | :----------------------- | :------------------------- |
| 标识         | 有唯一 ID                | 无标识                     |
| 相等性       | 通过 ID 判断             | 通过属性值判断             |
| 可变性       | 可变                     | 不可变                     |
| 生命周期     | 有生命周期               | 无生命周期                 |
| 示例         | User、Order、Product     | Email、Money、Address      |

## 聚合（Aggregate）

### 聚合概念

- **聚合**：一组相关对象的集合，作为一个整体进行管理。
- **聚合根（Aggregate Root）**：聚合的入口点，外部只能通过聚合根访问聚合内的对象。

```php
namespace App\Domain\Order;

class Order // 聚合根
{
    private array $items = [];

    public function __construct(
        private OrderId $id,
        private CustomerId $customerId,
        private Money $total
    ) {
    }

    public function addItem(ProductId $productId, int $quantity, Money $price): void
    {
        // 业务规则：不能添加已存在的商品
        foreach ($this->items as $item) {
            if ($item->getProductId()->equals($productId)) {
                throw new ProductAlreadyInOrderException();
            }
        }

        $item = new OrderItem($productId, $quantity, $price);
        $this->items[] = $item;
        $this->recalculateTotal();
    }

    public function removeItem(ProductId $productId): void
    {
        $this->items = array_filter(
            $this->items,
            fn($item) => !$item->getProductId()->equals($productId)
        );
        $this->recalculateTotal();
    }

    private function recalculateTotal(): void
    {
        $this->total = new Money(0, 'USD');
        foreach ($this->items as $item) {
            $itemTotal = new Money(
                $item->getPrice()->amount * $item->getQuantity(),
                $item->getPrice()->currency
            );
            $this->total = $this->total->add($itemTotal);
        }
    }

    public function getItems(): array
    {
        return $this->items; // 返回副本，保护内部状态
    }
}

class OrderItem // 聚合内的实体
{
    public function __construct(
        private ProductId $productId,
        private int $quantity,
        private Money $price
    ) {
        if ($quantity <= 0) {
            throw new InvalidArgumentException('Quantity must be positive');
        }
    }

    public function getProductId(): ProductId
    {
        return $this->productId;
    }

    public function getQuantity(): int
    {
        return $this->quantity;
    }

    public function getPrice(): Money
    {
        return $this->price;
    }
}
```

### 聚合设计原则

1. **通过聚合根访问**：外部只能通过聚合根访问聚合内的对象。
2. **事务边界**：一个事务只能修改一个聚合。
3. **保持一致性**：聚合内的对象保持一致状态。

## 仓储（Repository）

### 仓储模式

- **仓储**：封装数据访问逻辑，提供领域对象的持久化接口。
- 仓储接口定义在领域层，实现放在基础设施层。

```php
namespace App\Domain\Order;

interface OrderRepository
{
    public function save(Order $order): void;
    public function findById(OrderId $id): ?Order;
    public function findByCustomerId(CustomerId $customerId): array;
}
```

### 仓储实现

```php
namespace App\Infrastructure\Persistence;

use App\Domain\Order\Order;
use App\Domain\Order\OrderId;
use App\Domain\Order\OrderRepository;
use App\Domain\Order\CustomerId;
use PDO;

class DatabaseOrderRepository implements OrderRepository
{
    public function __construct(private PDO $db)
    {
    }

    public function save(Order $order): void
    {
        $this->db->beginTransaction();
        try {
            // 保存订单
            $stmt = $this->db->prepare(
                'INSERT INTO orders (id, customer_id, total_amount, total_currency) 
                 VALUES (?, ?, ?, ?)
                 ON DUPLICATE KEY UPDATE 
                 customer_id = ?, total_amount = ?, total_currency = ?'
            );
            $stmt->execute([
                $order->getId()->getValue(),
                $order->getCustomerId()->getValue(),
                $order->getTotal()->amount,
                $order->getTotal()->currency,
                $order->getCustomerId()->getValue(),
                $order->getTotal()->amount,
                $order->getTotal()->currency,
            ]);

            // 删除旧订单项
            $stmt = $this->db->prepare('DELETE FROM order_items WHERE order_id = ?');
            $stmt->execute([$order->getId()->getValue()]);

            // 保存订单项
            foreach ($order->getItems() as $item) {
                $stmt = $this->db->prepare(
                    'INSERT INTO order_items (order_id, product_id, quantity, price_amount, price_currency)
                     VALUES (?, ?, ?, ?, ?)'
                );
                $stmt->execute([
                    $order->getId()->getValue(),
                    $item->getProductId()->getValue(),
                    $item->getQuantity(),
                    $item->getPrice()->amount,
                    $item->getPrice()->currency,
                ]);
            }

            $this->db->commit();
        } catch (\Exception $e) {
            $this->db->rollBack();
            throw $e;
        }
    }

    public function findById(OrderId $id): ?Order
    {
        // 查询订单和订单项，重建聚合
        // ...
    }

    public function findByCustomerId(CustomerId $customerId): array
    {
        // 查询特定客户的所有订单
        // ...
    }
}
```

## 领域服务（Domain Service）

### 何时使用领域服务

- 操作涉及多个实体。
- 操作不属于任何单个实体。
- 需要访问仓储或其他领域服务。

```php
namespace App\Domain\Order;

class OrderPricingService
{
    public function __construct(
        private DiscountRepository $discountRepository
    ) {
    }

    public function calculateTotal(Order $order): Money
    {
        $total = new Money(0, 'USD');
        
        foreach ($order->getItems() as $item) {
            $itemTotal = new Money(
                $item->getPrice()->amount * $item->getQuantity(),
                $item->getPrice()->currency
            );
            $total = $total->add($itemTotal);
        }

        // 应用折扣
        $discount = $this->discountRepository->findApplicableDiscount($order);
        if ($discount !== null) {
            $discountAmount = new Money(
                $total->amount * $discount->getRate(),
                $total->currency
            );
            $total = $total->add($discountAmount->multiply(-1));
        }

        return $total;
    }
}
```

## 完整示例：订单系统

### 领域模型

```php
namespace App\Domain\Order;

// 聚合根
class Order
{
    private array $items = [];
    private OrderStatus $status;

    public function __construct(
        private OrderId $id,
        private CustomerId $customerId,
        private DateTimeImmutable $createdAt
    ) {
        $this->status = OrderStatus::PENDING;
    }

    public function addItem(ProductId $productId, int $quantity, Money $price): void
    {
        if ($this->status !== OrderStatus::PENDING) {
            throw new OrderCannotBeModifiedException();
        }

        $item = new OrderItem($productId, $quantity, $price);
        $this->items[] = $item;
    }

    public function confirm(): void
    {
        if ($this->status !== OrderStatus::PENDING) {
            throw new InvalidOrderStatusException();
        }

        if (empty($this->items)) {
            throw new EmptyOrderException();
        }

        $this->status = OrderStatus::CONFIRMED;
    }

    public function cancel(): void
    {
        if (!in_array($this->status, [OrderStatus::PENDING, OrderStatus::CONFIRMED])) {
            throw new OrderCannotBeCancelledException();
        }

        $this->status = OrderStatus::CANCELLED;
    }
}

// 值对象
readonly class OrderId
{
    public function __construct(private string $value)
    {
    }

    public function getValue(): string
    {
        return $this->value;
    }

    public function equals(OrderId $other): bool
    {
        return $this->value === $other->value;
    }
}
```

## DDD 最佳实践

### 1. 保持领域模型纯净

- 领域层不依赖基础设施层。
- 使用接口定义依赖。

### 2. 使用值对象封装概念

- 不要使用原始类型（string、int）表示领域概念。
- 使用值对象（Email、Money、Address）提高表达力。

### 3. 聚合设计要小

- 聚合应该尽可能小，只包含必须保持一致的实体。
- 大聚合会导致性能问题和并发冲突。

### 4. 通过 ID 引用其他聚合

- 聚合之间通过 ID 引用，不直接持有对象引用。

## 练习

1. 设计一个用户管理系统，定义 `User` 实体和 `Email`、`UserId` 值对象。

2. 创建一个订单系统，`Order` 作为聚合根，包含多个 `OrderItem`，实现添加、删除商品和计算总价的功能。

3. 实现一个 `Product` 聚合，包含价格、库存等属性，定义业务规则（如价格不能为负、库存不足时不能销售）。

4. 创建一个 `Address` 值对象，包含街道、城市、邮编等属性，实现不可变性和相等性判断。

5. 设计一个 `OrderRepository` 接口在领域层，然后在基础设施层实现数据库版本和内存版本。

6. 实现一个领域服务 `ShippingCalculator`，根据订单重量和目的地计算运费，涉及多个领域对象。
