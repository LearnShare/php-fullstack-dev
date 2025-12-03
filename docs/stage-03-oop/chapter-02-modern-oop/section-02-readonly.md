# 3.2.2 Readonly 属性

## 概述

`readonly` 属性（PHP 8.1+）和 `readonly` 类（PHP 8.2+）允许创建不可变对象，提高代码的安全性和可预测性。

## Readonly 属性（PHP 8.1+）

### 基础语法

- **语法**：`public readonly 类型 $property;`
- **说明**：属性只能在声明时或构造函数中初始化，之后不可修改。
- **适用场景**：创建不可变对象（Immutable Objects）、值对象（Value Objects）、DTO（Data Transfer Objects）。

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

### 限制

- 不能有默认值（除非是常量表达式）
- 不能是 `static`
- 不能使用 `unset()`

```php
class Config
{
    // 错误：readonly 属性不能有默认值（非常量表达式）
    // public readonly string $env = 'production';

    // 正确：在构造函数中初始化
    public function __construct(
        public readonly string $environment
    ) {
    }
}
```

### 与构造器属性提升结合

```php
class User
{
    public function __construct(
        public readonly int $id,
        public readonly string $name,
        public readonly string $email
    ) {
    }
}

$user = new User(1, 'Alice', 'alice@example.com');
// $user->name = 'Bob'; // 错误：Cannot modify readonly property
```

## Readonly 类（PHP 8.2+）

### 基础语法

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

### Readonly 类限制

- 所有属性必须是 `readonly`
- 不能包含非 `readonly` 属性
- 不能包含未类型化的属性

```php
readonly class Product
{
    public function __construct(
        public string $name,
        public float $price,
        public int $stock
    ) {
    }

    // 错误：readonly 类不能包含非 readonly 属性
    // private float $discount = 0.0;
}
```

## Readonly 与克隆

### 克隆限制

`readonly` 属性在克隆时不能被修改，需要重新构造对象：

```php
readonly class Config
{
    public function __construct(
        public string $environment,
        public array $settings
    ) {
    }

    public function withEnvironment(string $env): self
    {
        // 不能使用 clone，需要重新构造
        return new self($env, $this->settings);
    }
}
```

### Clone with（PHP 8.5+）

PHP 8.5+ 支持 `clone with` 语法，可以克隆并更新 readonly 属性：

```php
readonly class User
{
    public function __construct(
        public string $name,
        public int $age,
        public string $email
    ) {
    }
}

$user = new User('Alice', 30, 'alice@example.com');

// PHP 8.5+ clone with 语法
$updatedUser = clone $user with ['age' => 31, 'email' => 'alice.new@example.com'];
```

## Readonly 最佳实践

### 值对象（Value Objects）

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

    public function multiply(float $factor): Money
    {
        return new Money($this->amount * $factor, $this->currency);
    }
}

$money1 = new Money(100.0, 'USD');
$money2 = new Money(50.0, 'USD');
$total = $money1->add($money2); // Money(150.0, 'USD')
```

### DTO（Data Transfer Objects）

```php
readonly class UserDTO
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email,
        public DateTimeImmutable $createdAt
    ) {
    }

    public function toArray(): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->createdAt->format('Y-m-d H:i:s')
        ];
    }
}
```

### 配置对象

```php
readonly class DatabaseConfig
{
    public function __construct(
        public string $host,
        public int $port,
        public string $database,
        public string $username,
        public string $password
    ) {
    }

    public function getDsn(): string
    {
        return "mysql:host={$this->host};port={$this->port};dbname={$this->database}";
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

readonly class Address
{
    public function __construct(
        public string $street,
        public string $city,
        public string $postalCode,
        public string $country
    ) {
    }

    public function __toString(): string
    {
        return "{$this->street}, {$this->city}, {$this->postalCode}, {$this->country}";
    }
}

readonly class User
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email,
        public Address $address
    ) {
    }

    public function withAddress(Address $newAddress): self
    {
        return new self($this->id, $this->name, $this->email, $newAddress);
    }
}

$address = new Address('123 Main St', 'New York', '10001', 'USA');
$user = new User(1, 'Alice', 'alice@example.com', $address);

$newAddress = new Address('456 Oak Ave', 'Los Angeles', '90001', 'USA');
$updatedUser = $user->withAddress($newAddress);
```

## 注意事项

1. **不可变性**：`readonly` 属性创建后不能修改，适合值对象和配置对象。

2. **性能考虑**：创建新对象而不是修改现有对象，可能增加内存使用。

3. **克隆限制**：`readonly` 属性在克隆时不能修改，需要使用 `clone with`（PHP 8.5+）或重新构造。

4. **类型安全**：`readonly` 属性必须声明类型。

5. **适用场景**：适用于值对象、DTO、配置对象，不适用于需要频繁修改的对象。

## 练习

1. 创建一个 `readonly` 类 `Address`，包含 `street`、`city`、`postalCode` 属性，并提供 `__toString()` 方法。

2. 实现一个 `readonly` 类 `Money`，包含 `amount` 和 `currency` 属性，提供 `add()` 和 `multiply()` 方法。

3. 创建一个 `readonly` 类 `Product`，使用枚举 `ProductCategory` 表示分类，使用 `Money` 类表示价格。

4. 实现一个 `readonly` 类 `DateRange`，包含开始和结束日期，提供 `contains()` 方法判断某个日期是否在范围内。

5. 使用 `clone with` 语法（PHP 8.5+）创建一个 `readonly` 类，并演示如何更新属性。
