# 3.2.3 枚举（Enums）

## 概述

枚举（Enums）是 PHP 8.1+ 引入的类型安全常量集合。它们提供了比传统常量类更好的类型安全性和功能。

## 基础枚举

### 基本语法

- **语法**：`enum EnumName { case CaseName; }`
- 枚举是类型安全的常量集合。

```php
enum Status
{
    case PENDING;
    case APPROVED;
    case REJECTED;
}

class Order
{
    public function __construct(
        public int $id,
        public Status $status
    ) {
    }
}

$order = new Order(1, Status::PENDING);
```

### 枚举值访问

```php
$status = Status::PENDING;
echo $status->name; // PENDING

// 枚举值比较
if ($status === Status::PENDING) {
    echo "Order is pending";
}
```

## Backed Enums（关联值枚举）

### 整数枚举

```php
enum HttpStatus: int
{
    case OK = 200;
    case NOT_FOUND = 404;
    case SERVER_ERROR = 500;
}

$status = HttpStatus::OK;
echo $status->value; // 200
echo $status->name;  // OK
```

### 字符串枚举

```php
enum Color: string
{
    case RED = 'red';
    case GREEN = 'green';
    case BLUE = 'blue';
}

$color = Color::RED;
echo $color->value; // 'red'
```

### 从值创建枚举

```php
// 从值创建枚举
$status = HttpStatus::from(200); // HttpStatus::OK

// 尝试从值创建，不存在时抛出异常
try {
    $status = HttpStatus::from(999); // 抛出 ValueError
} catch (ValueError $e) {
    echo "Invalid status code";
}

// 安全地从值创建，不存在时返回 null
$status = HttpStatus::tryFrom(999); // null
```

## 枚举方法

### 实例方法

```php
enum Priority: int
{
    case LOW = 1;
    case MEDIUM = 2;
    case HIGH = 3;
    case URGENT = 4;

    public function isHigh(): bool
    {
        return $this->value >= self::HIGH->value;
    }

    public function getLabel(): string
    {
        return match ($this) {
            self::LOW => 'Low Priority',
            self::MEDIUM => 'Medium Priority',
            self::HIGH => 'High Priority',
            self::URGENT => 'Urgent',
        };
    }
}

$priority = Priority::HIGH;
echo $priority->isHigh(); // true
echo $priority->getLabel(); // High Priority
```

### 静态方法

```php
enum Priority: int
{
    case LOW = 1;
    case MEDIUM = 2;
    case HIGH = 3;
    case URGENT = 4;

    public static function fromString(string $name): self
    {
        return match (strtoupper($name)) {
            'LOW' => self::LOW,
            'MEDIUM' => self::MEDIUM,
            'HIGH' => self::HIGH,
            'URGENT' => self::URGENT,
            default => throw new ValueError("Invalid priority: {$name}"),
        };
    }

    public static function getAll(): array
    {
        return [self::LOW, self::MEDIUM, self::HIGH, self::URGENT];
    }
}

$priority = Priority::fromString('urgent');
$all = Priority::getAll();
```

## 枚举实现接口

### 接口实现

```php
interface HasDescription
{
    public function getDescription(): string;
}

enum UserRole: string implements HasDescription
{
    case ADMIN = 'admin';
    case EDITOR = 'editor';
    case VIEWER = 'viewer';

    public function getDescription(): string
    {
        return match ($this) {
            self::ADMIN => 'Full system access',
            self::EDITOR => 'Can create and edit content',
            self::VIEWER => 'Read-only access',
        };
    }
}

$role = UserRole::ADMIN;
echo $role->getDescription(); // Full system access
```

## 枚举与 match 表达式

### 类型安全的状态处理

```php
enum OrderStatus: string
{
    case PENDING = 'pending';
    case PROCESSING = 'processing';
    case SHIPPED = 'shipped';
    case DELIVERED = 'delivered';
    case CANCELLED = 'cancelled';

    public function canCancel(): bool
    {
        return match ($this) {
            self::PENDING, self::PROCESSING => true,
            default => false,
        };
    }

    public function getNextStatus(): ?self
    {
        return match ($this) {
            self::PENDING => self::PROCESSING,
            self::PROCESSING => self::SHIPPED,
            self::SHIPPED => self::DELIVERED,
            default => null,
        };
    }

    public function getColor(): string
    {
        return match ($this) {
            self::PENDING => 'yellow',
            self::PROCESSING => 'blue',
            self::SHIPPED => 'purple',
            self::DELIVERED => 'green',
            self::CANCELLED => 'red',
        };
    }
}
```

## 枚举与数据库

### 存储和恢复

```php
enum UserType: string
{
    case CUSTOMER = 'customer';
    case VENDOR = 'vendor';
    case ADMIN = 'admin';
}

class User
{
    public function __construct(
        public int $id,
        public string $name,
        public UserType $type
    ) {
    }

    public function toArray(): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'type' => $this->type->value, // 存储到数据库
        ];
    }

    public static function fromArray(array $data): self
    {
        return new self(
            $data['id'],
            $data['name'],
            UserType::from($data['type']) // 从数据库恢复
        );
    }
}
```

### 数据库查询示例

```php
// 存储
$user = new User(1, 'Alice', UserType::ADMIN);
$data = $user->toArray();
// INSERT INTO users (id, name, type) VALUES (1, 'Alice', 'admin')

// 恢复
$row = ['id' => 1, 'name' => 'Alice', 'type' => 'admin'];
$user = User::fromArray($row);
```

## 完整示例

```php
<?php
declare(strict_types=1);

enum OrderStatus: string
{
    case PENDING = 'pending';
    case PROCESSING = 'processing';
    case SHIPPED = 'shipped';
    case DELIVERED = 'delivered';
    case CANCELLED = 'cancelled';

    public function canCancel(): bool
    {
        return $this === self::PENDING || $this === self::PROCESSING;
    }

    public function getNextStatus(): ?self
    {
        return match ($this) {
            self::PENDING => self::PROCESSING,
            self::PROCESSING => self::SHIPPED,
            self::SHIPPED => self::DELIVERED,
            default => null,
        };
    }
}

readonly class Order
{
    public function __construct(
        public int $id,
        public string $customerName,
        public OrderStatus $status,
        public Money $total,
        public DateTimeImmutable $createdAt
    ) {
    }

    public function canCancel(): bool
    {
        return $this->status->canCancel();
    }

    public function markAsShipped(): self
    {
        $nextStatus = $this->status->getNextStatus();
        if ($nextStatus === null) {
            throw new RuntimeException('Cannot ship order in current status');
        }
        return new self(
            $this->id,
            $this->customerName,
            $nextStatus,
            $this->total,
            $this->createdAt
        );
    }
}
```

## 注意事项

1. **类型安全**：枚举提供类型安全，避免使用字符串或整数常量。

2. **值唯一性**：枚举值必须唯一，不能重复。

3. **数据库存储**：使用 Backed Enums 存储到数据库，通过 `value` 属性获取值。

4. **方法定义**：枚举可以定义方法和静态方法，提供丰富的功能。

5. **接口实现**：枚举可以实现接口，提供更灵活的扩展。

## 练习

1. 定义一个 `PaymentMethod` 枚举，包含 `CREDIT_CARD`、`PAYPAL`、`BANK_TRANSFER`，每个枚举值关联一个字符串描述。

2. 创建一个 `Status` 枚举，包含 `DRAFT`、`PUBLISHED`、`ARCHIVED`，提供 `canEdit()` 和 `canDelete()` 方法。

3. 实现一个 `Priority` 枚举，包含 `LOW`、`MEDIUM`、`HIGH`、`URGENT`，提供 `isHigh()` 和 `getLabel()` 方法。

4. 创建一个 `UserRole` 枚举，实现 `HasDescription` 接口，提供角色描述。

5. 实现一个 `OrderStatus` 枚举，与 `Order` 类配合使用，提供状态转换方法。
