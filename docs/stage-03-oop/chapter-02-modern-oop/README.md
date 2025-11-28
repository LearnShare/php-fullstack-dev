# 3.2 现代 PHP 8+ OOP 能力

## 目标

- 掌握构造器属性提升（Constructor Property Promotion）语法，减少重复代码。
- 理解 `readonly` 属性的作用与使用场景，创建不可变对象。
- 熟悉枚举（Enums）的定义与使用，替代传统常量类。
- 了解静态属性的滥用风险，掌握正确的使用方式。

## 构造器属性提升（PHP 8.0+）

### 基础语法

- **语法**：在构造函数参数前添加可见性修饰符，PHP 会自动创建属性并赋值。
- 减少样板代码，提升可读性。

```php
// 传统写法
class User
{
    public int $id;
    public string $name;
    public string $email;

    public function __construct(int $id, string $name, string $email)
    {
        $this->id = $id;
        $this->name = $name;
        $this->email = $email;
    }
}

// PHP 8.0+ 构造器属性提升
class User
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email
    ) {
    }
}
```

### 混合使用

- 可以混合使用提升属性和普通属性。

```php
class Product
{
    private float $discount = 0.0;

    public function __construct(
        public string $name,
        public float $price,
        public int $stock = 0
    ) {
        // 可以在这里执行额外逻辑
        if ($this->price < 0) {
            throw new InvalidArgumentException('Price cannot be negative');
        }
    }

    public function applyDiscount(float $rate): void
    {
        $this->discount = $rate;
    }

    public function getFinalPrice(): float
    {
        return $this->price * (1 - $this->discount);
    }
}
```

### 可见性修饰符

- 支持 `public`、`protected`、`private`。

```php
class BankAccount
{
    public function __construct(
        public int $accountNumber,
        protected float $balance,
        private string $pin
    ) {
    }

    public function getBalance(): float
    {
        return $this->balance;
    }

    public function verifyPin(string $input): bool
    {
        return $this->pin === $input;
    }
}
```

## Readonly 属性（PHP 8.1+）

### 基础语法

- **语法**：`public readonly 类型 $property;`
- **说明**：属性只能在声明时或构造函数中初始化，之后不可修改。
- **适用场景**：创建不可变对象（Immutable Objects）、值对象（Value Objects）、DTO（Data Transfer Objects）。
- **限制**：
  - 不能有默认值（除非是常量表达式）
  - 不能是 `static`
  - 不能是 `unset`
- 适用于创建不可变对象（Immutable Objects）。

```php
class Point
{
    public function __construct(
        public readonly int $x,
        public readonly int $y
    ) {
    }
}

$point = new Point(10, 20);
echo $point->x; // 10
// $point->x = 30; // 错误：Cannot modify readonly property
```

### Readonly 类（PHP 8.2+）

- **语法**：`readonly class ClassName { ... }`
- 类的所有属性自动为 `readonly`。

```php
readonly class User
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email
    ) {
    }
}

$user = new User(1, 'Alice', 'alice@example.com');
// $user->name = 'Bob'; // 错误：Cannot modify readonly property
```

### Readonly 与克隆

- `readonly` 属性在克隆时可以被重新初始化。

```php
class Config
{
    public function __construct(
        public readonly string $environment,
        public readonly array $settings
    ) {
    }

    public function withEnvironment(string $env): self
    {
        $clone = clone $this;
        // 在克隆后无法修改 readonly 属性
        // 需要重新构造对象
        return new self($env, $this->settings);
    }
}
```

### Readonly 最佳实践

- 适用于值对象（Value Objects）、DTO（Data Transfer Objects）、配置对象。
- 避免在需要频繁修改的对象上使用 `readonly`。

```php
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
            throw new InvalidArgumentException('Currency mismatch');
        }
        return new Money($this->amount + $other->amount, $this->currency);
    }
}
```

## 枚举（Enums）（PHP 8.1+）

### 基础枚举

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

### 枚举与字符串/整数

- 枚举可以关联值（Backed Enums）。

```php
enum HttpStatus: int
{
    case OK = 200;
    case NOT_FOUND = 404;
    case SERVER_ERROR = 500;
}

enum Color: string
{
    case RED = 'red';
    case GREEN = 'green';
    case BLUE = 'blue';
}

$status = HttpStatus::OK;
echo $status->value; // 200
echo $status->name;  // OK
```

### 枚举方法

- 枚举可以定义方法，包括静态方法。

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
}

$priority = Priority::HIGH;
echo $priority->isHigh(); // true
$p = Priority::fromString('urgent');
```

### 枚举实现接口

- 枚举可以实现接口。

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

### 枚举与 `match` 表达式

- 枚举与 `match` 表达式配合使用，提供类型安全的状态处理。

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
}
```

### 枚举与数据库

- 枚举值可以存储在数据库中，通过 `value` 属性获取。

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

## 静态属性与方法（深入）

### 静态属性滥用风险

- 静态属性是全局状态，可能导致：
  - 测试困难（难以隔离状态）
  - 并发问题（多线程/多进程环境）
  - 隐藏依赖（难以追踪数据流）

```php
// 不推荐：使用静态属性存储全局状态
class Config
{
    public static string $databaseHost = 'localhost';
    public static string $databaseName = 'app';
}

// 推荐：使用依赖注入
class DatabaseConnection
{
    public function __construct(
        private string $host,
        private string $database
    ) {
    }
}
```

### 静态方法的合理使用

- 适用于：
  - 工具类方法（如 `Math::add()`）
  - 工厂方法（`User::create()`）
  - 单例模式（需谨慎使用）

```php
class DateTimeHelper
{
    public static function formatTimestamp(int $timestamp, string $format = 'Y-m-d H:i:s'): string
    {
        return date($format, $timestamp);
    }

    public static function isWeekend(int $timestamp): bool
    {
        $dayOfWeek = (int) date('w', $timestamp);
        return $dayOfWeek === 0 || $dayOfWeek === 6;
    }
}

echo DateTimeHelper::formatTimestamp(time());
```

### 静态工厂方法

- 使用静态方法创建对象，提供更灵活的构造方式。

```php
class Email
{
    private function __construct(
        public readonly string $address
    ) {
        if (!filter_var($address, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email: {$address}");
        }
    }

    public static function fromString(string $address): self
    {
        return new self($address);
    }

    public static function fromArray(array $data): self
    {
        return new self($data['email'] ?? throw new InvalidArgumentException('Email required'));
    }
}

$email = Email::fromString('user@example.com');
```

## 综合示例

```php
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

enum OrderStatus: string
{
    case PENDING = 'pending';
    case PROCESSING = 'processing';
    case SHIPPED = 'shipped';
    case DELIVERED = 'delivered';

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
```

## 属性钩子（Property Hooks）（PHP 8.4+）

### 基础语法

- **语法**：在属性定义中使用 `get` 和 `set` 钩子
- **说明**：允许在属性的读取和写入操作中定义自定义逻辑，减少样板代码。

```php
<?php
declare(strict_types=1);

class Temperature
{
    private float $celsius;
    
    // 使用属性钩子自动转换
    public float $fahrenheit
    {
        get => ($this->celsius * 9/5) + 32;
        set (float $fahrenheit) {
            $this->celsius = ($fahrenheit - 32) * 5/9;
        }
    }
    
    public function __construct(float $celsius)
    {
        $this->celsius = $celsius;
    }
    
    public function getCelsius(): float
    {
        return $this->celsius;
    }
}

$temp = new Temperature(25);
echo $temp->fahrenheit; // 77
$temp->fahrenheit = 86;
echo $temp->getCelsius(); // 30
```

### 不对称可见性（PHP 8.4+）

- 允许属性的读取和写入具有不同的可见性修饰符。

```php
<?php
declare(strict_types=1);

class BankAccount
{
    private float $balance = 0.0;
    
    // 公有读取，私有写入
    public float $balance
    {
        get => $this->balance;
        private set;
    }
    
    public function deposit(float $amount): void
    {
        $this->balance = $this->balance + $amount; // 可以设置（在类内部）
    }
    
    public function withdraw(float $amount): bool
    {
        if ($this->balance >= $amount) {
            $this->balance = $this->balance - $amount;
            return true;
        }
        return false;
    }
}

$account = new BankAccount();
$account->deposit(100);
echo $account->balance; // 可以读取：100
// $account->balance = 200; // 错误：无法从外部设置
```

### 构造函数属性提升支持 final（PHP 8.4+）

```php
<?php
declare(strict_types=1);

class Base
{
    public function __construct(
        public final string $id  // PHP 8.4+ 支持 final
    ) {
    }
}

class Child extends Base
{
    // $id 属性不能被重写
}
```

## 练习

1. 使用构造器属性提升重构一个现有的类，比较代码行数的减少。

2. 创建一个 `readonly` 类 `Address`，包含 `street`、`city`、`postalCode` 属性，并提供 `__toString()` 方法。

3. **PHP 8.4+**：使用属性钩子创建一个 `Currency` 类，支持多种货币之间的自动转换。

4. **PHP 8.4+**：使用不对称可见性创建一个 `User` 类，`email` 属性可以读取但只能通过验证方法设置。

5. **PHP 8.4+**：使用属性钩子实现一个自动计算总价的 `OrderItem` 类。

3. 定义一个 `PaymentMethod` 枚举，包含 `CREDIT_CARD`、`PAYPAL`、`BANK_TRANSFER`，每个枚举值关联一个字符串描述，并实现 `getDescription()` 方法。

4. 创建一个 `readonly` 类 `Product`，使用枚举 `ProductCategory` 表示分类，使用 `Money` 类表示价格，确保所有属性不可变。

5. 实现一个 `Status` 枚举，包含 `DRAFT`、`PUBLISHED`、`ARCHIVED`，提供 `canEdit()` 和 `canDelete()` 方法，判断当前状态是否允许编辑或删除。

6. 编写一个 `DateRange` 类，使用 `readonly` 属性存储开始和结束日期，提供 `contains()` 方法判断某个日期是否在范围内，使用静态工厂方法 `fromString()` 从字符串创建对象。
