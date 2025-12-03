# 3.9.2 实体与值对象

## 概述

实体（Entity）和值对象（Value Object）是 DDD 中的两个核心概念。理解它们的区别和使用场景对于设计良好的领域模型至关重要。

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

    public function equals(Money $other): bool
    {
        return $this->amount === $other->amount 
            && $this->currency === $other->currency;
    }

    public function add(Money $other): Money
    {
        if ($this->currency !== $other->currency) {
            throw new CurrencyMismatchException();
        }
        return new Money($this->amount + $other->amount, $this->currency);
    }
}
```

### 值对象示例

```php
readonly class Address
{
    public function __construct(
        public string $street,
        public string $city,
        public string $postalCode,
        public string $country
    ) {
    }

    public function equals(Address $other): bool
    {
        return $this->street === $other->street
            && $this->city === $other->city
            && $this->postalCode === $other->postalCode
            && $this->country === $other->country;
    }

    public function __toString(): string
    {
        return "{$this->street}, {$this->city}, {$this->postalCode}, {$this->country}";
    }
}
```

## 实体 vs 值对象

### 对比

| 特性 | 实体（Entity） | 值对象（Value Object） |
| :--- | :------------- | :--------------------- |
| 标识 | 有唯一标识（ID） | 无标识 |
| 相等性 | 通过 ID 判断 | 通过属性值判断 |
| 可变性 | 可变 | 不可变 |
| 生命周期 | 有生命周期 | 可替换 |
| 使用场景 | 需要跟踪的对象 | 描述性的值 |

### 选择原则

- **使用实体**：当对象需要唯一标识，需要跟踪其变化时。
- **使用值对象**：当对象只通过属性值区分，不需要跟踪变化时。

```php
// 实体：用户需要唯一标识，需要跟踪变化
class User
{
    private UserId $id;  // 唯一标识
    private Email $email; // 值对象
    private Address $address; // 值对象
}

// 值对象：邮箱、地址通过值区分，不可变
readonly class Email
{
    private string $value; // 通过值判断相等性
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 值对象
readonly class UserId
{
    public function __construct(private string $value)
    {
        if (empty($value)) {
            throw new InvalidArgumentException('User ID cannot be empty');
        }
    }

    public function getValue(): string
    {
        return $this->value;
    }

    public function equals(UserId $other): bool
    {
        return $this->value === $other->value;
    }
}

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

// 实体
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

    public function getEmail(): Email
    {
        return $this->email;
    }

    public function changeEmail(Email $newEmail): void
    {
        // 值对象可以整体替换
        $this->email = $newEmail;
    }

    public function equals(User $other): bool
    {
        // 实体通过 ID 判断相等性
        return $this->id->equals($other->id);
    }
}
```

## 注意事项

1. **实体标识**：实体必须有唯一标识，通常使用值对象表示 ID。

2. **值对象不可变**：值对象创建后不能修改，需要修改时创建新实例。

3. **相等性判断**：实体通过 ID 判断，值对象通过属性值判断。

4. **使用场景**：根据业务需求选择合适的类型。

5. **值对象验证**：值对象应该在构造时进行验证，确保有效性。

## 练习

1. 创建一个 `User` 实体，包含 `UserId`、`Email`、`Name` 等值对象。

2. 实现一个 `Money` 值对象，包含金额和货币，提供 `add()` 和 `subtract()` 方法。

3. 创建一个 `Address` 值对象，包含街道、城市、邮编等信息。

4. 实现一个 `Product` 实体，使用值对象表示价格、SKU 等。

5. 设计一个订单系统，区分实体和值对象，实现相等性判断。
