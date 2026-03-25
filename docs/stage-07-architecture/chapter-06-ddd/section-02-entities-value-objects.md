# 7.6.2 实体与值对象

## 概述

实体（Entity）和值对象（Value Object）是领域驱动设计中两个最基本的建模概念。它们分别用于建模不同性质的领域概念：正确别性的实体和无标识的值对象。正确区分和使用实体与值对象是 DDD 建模的关键技能。

实体是指具有唯一标识的对象，其身份在生命周期内保持不变。无论实体的其他属性如何变化，只要标识相同，它始终代表同一个对象。例如，用户账户是一个典型的实体：即使用户的姓名、邮箱、头像等属性发生变化，只要用户 ID 不变，它仍然是同一个用户。

值对象是指没有唯一标识、仅通过属性值来识别的对象。值对象通常用于描述实体的某些特征或属性，且通常是不可变的。例如，地址是一个典型的值对象：如果两个地址的街道、城市、邮编都相同，它们就是相等的，可以相互替换。

理解实体与值对象的区别对于建立准确的领域模型至关重要。使用不当会导致领域模型难以理解、维护困难，甚至导致业务逻辑错误。

**主要内容**：
- 实体的概念和特征
- 唯一标识的设计
- 实体的生命周期管理
- 值对象的概念和特征
- 不可变性设计
- 实体与值对象的区分原则
- 完整的代码示例

## 特性

### 实体的特性

- **唯一标识**：每个实体实例都有唯一的身份标识
- **可变性**：实体的属性可以随时间变化
- **生命周期**：实体有创建、更新、删除的生命周期
- **连续性**：实体的标识在整个生命周期内保持不变
- **行为封装**：实体通常包含业务行为方法

### 值对象的特性

- **无标识**：值对象没有唯一标识，不区分个体
- **不可变性**：值对象创建后不可修改
- **值相等**：值对象通过属性值判定相等性
- **可替换性**：相等的值对象可以相互替换
- **描述性**：值对象用于描述实体的特征

## 核心概念

### 实体的设计原则

实体的核心特征是唯一标识。标识的设计是实体建模的首要任务。在 PHP 中，标识可以是系统生成的自增 ID、UUID、字符串标识等。选择合适的标识类型需要考虑业务场景和性能要求。

```php
<?php
declare(strict_types=1);

namespace App\Domain\ValueObject;

class OrderId
{
    private int $value;

    public function __construct(int $value)
    {
        if ($value <= 0) {
            throw new \InvalidArgumentException('Order ID must be positive');
        }
        $this->value = $value;
    }

    public function getValue(): int
    {
        return $this->value;
    }

    public function equals(OrderId $other): bool
    {
        return $this->value === $other->value;
    }
}
```

通过将标识封装为值对象，可以在标识的构造过程中添加业务验证逻辑，确保标识的有效性。

### 值对象的设计原则

值对象的核心特征是不可变性和值相等。不可变性意味着一旦创建，值对象的属性就不能被修改。如果需要修改值对象，应该创建一个新的实例而不是修改原有实例。

```php
<?php
declare(strict_types=1);

namespace App\Domain\ValueObject;

class Address
{
    public function __construct(
        private readonly string $province,
        private readonly string $city,
        private readonly string $district,
        private readonly string $street,
        private readonly string $zipCode
    ) {
        $this->validate();
    }

    private function validate(): void
    {
        if (empty($this->province)) {
            throw new \InvalidArgumentException('Province is required');
        }
        if (empty($this->city)) {
            throw new \InvalidArgumentException('City is required');
        }
        if (!preg_match('/^\d{6}$/', $this->zipCode)) {
            throw new \InvalidArgumentException('Invalid zip code format');
        }
    }

    public function getProvince(): string
    {
        return $this->province;
    }

    public function getCity(): string
    {
        return $this->city;
    }

    public function getDistrict(): string
    {
        return $this->district;
    }

    public function getStreet(): string
    {
        return $this->street;
    }

    public function getZipCode(): string
    {
        return $this->zipCode;
    }

    public function equals(Address $other): bool
    {
        return $this->province === $other->province
            && $this->city === $other->city
            && $this->district === $other->district
            && $this->street === $other->street
            && $this->zipCode === $other->zipCode;
    }

    public function __toString(): string
    {
        return sprintf(
            '%s %s %s %s %s',
            $this->province,
            $this->city,
            $this->district,
            $this->street,
            $this->zipCode
        );
    }
}
```

## 语法/定义

### 实体类定义

```php
<?php
declare(strict_types=1);

// 语法：class EntityName { ... }

// 实体特征：
// - 有唯一标识
// - 属性可以变化
// - 包含业务行为
```

### 值对象类定义

```php
<?php
declare(strict_types=1);

// 语法：class ValueObjectName { ... }

// 值对象特征：
// - 无唯一标识
// - 属性不可变
// - 通过值判定相等
```

## 基本用法

### 1. 实体实现示例

以下是一个完整的用户实体实现，展示了实体的各种特性：

```php
<?php
declare(strict_types=1);

namespace App\Domain\Entity;

use App\Domain\ValueObject\Email;
use App\Domain\ValueObject\UserId;
use App\Domain\ValueObject\UserStatus;

class User
{
    private ?int $id = null;
    private UserStatus $status;
    private \DateTimeImmutable $createdAt;
    private \DateTimeImmutable $updatedAt;

    public function __construct(
        private readonly UserId $userId,
        private Email $email,
        private string $name
    ) {
        $this->status = UserStatus::ACTIVE;
        $this->createdAt = new \DateTimeImmutable();
        $this->updatedAt = new \DateTimeImmutable();
        $this->validate();
    }

    private function validate(): void
    {
        if (empty($this->name)) {
            throw new \InvalidArgumentException('User name cannot be empty');
        }
    }

    public function changeEmail(Email $newEmail): void
    {
        if ($this->email->equals($newEmail)) {
            return;
        }

        $this->email = $newEmail;
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function changeName(string $newName): void
    {
        if (empty($newName)) {
            throw new \InvalidArgumentException('User name cannot be empty');
        }

        $this->name = $newName;
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function deactivate(): void
    {
        if ($this->status === UserStatus::DEACTIVATED) {
            throw new \DomainException('User is already deactivated');
        }

        $this->status = UserStatus::DEACTIVATED;
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function activate(): void
    {
        if ($this->status === UserStatus::ACTIVE) {
            throw new \DomainException('User is already active');
        }

        $this->status = UserStatus::ACTIVE;
        $this->updatedAt = new \DateTimeImmutable();
    }

    public function getId(): ?int
    {
        return $this->id;
    }

    public function setId(int $id): void
    {
        if ($this->id !== null) {
            throw new \DomainException('User ID cannot be changed');
        }
        $this->id = $id;
    }

    public function getUserId(): UserId
    {
        return $this->userId;
    }

    public function getEmail(): Email
    {
        return $this->email;
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function getStatus(): UserStatus
    {
        return $this->status;
    }

    public function getCreatedAt(): \DateTimeImmutable
    {
        return $this->createdAt;
    }

    public function getUpdatedAt(): \DateTimeImmutable
    {
        return $this->updatedAt;
    }
}
```

用户实体展示了实体的几个重要特性：唯一标识（UserId）、可变性（email、name、status 可以修改）、生命周期（createdAt、updatedAt）、行为封装（changeEmail、deactivate 等方法包含业务规则验证）。

### 2. 值对象实现示例

以下是几个常用的值对象实现：

```php
<?php
declare(strict_types=1);

namespace App\Domain\ValueObject;

class Email
{
    private string $value;

    public function __construct(string $value)
    {
        $value = trim($value);
        
        if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
            throw new \InvalidArgumentException('Invalid email format');
        }

        $this->value = strtolower($value);
    }

    public function getValue(): string
    {
        return $this->value;
    }

    public function equals(Email $other): bool
    {
        return $this->value === $other->value;
    }

    public function __toString(): string
    {
        return $this->value;
    }
}
```

```php
<?php
declare(strict_types=1);

namespace App\Domain\ValueObject;

enum UserStatus: string
{
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
    case DEACTIVATED = 'deactivated';
    case PENDING = 'pending';

    public function canLogin(): bool
    {
        return $this === self::ACTIVE;
    }

    public function isVerified(): bool
    {
        return $this !== self::PENDING;
    }
}
```

```php
<?php
declare(strict_types=1);

namespace App\Domain\ValueObject;

class PhoneNumber
{
    private string $value;
    private string $countryCode;

    public function __construct(string $value, string $countryCode = '86')
    {
        $value = preg_replace('/[^\d]/', '', $value);
        
        if (strlen($value) < 7 || strlen($value) > 15) {
            throw new \InvalidArgumentException('Invalid phone number');
        }

        $this->value = $value;
        $this->countryCode = $countryCode;
    }

    public function getValue(): string
    {
        return $this->value;
    }

    public function getCountryCode(): string
    {
        return $this->countryCode;
    }

    public function getFullNumber(): string
    {
        return '+' . $this->countryCode . $this->value;
    }

    public function equals(PhoneNumber $other): bool
    {
        return $this->value === $other->value 
            && $this->countryCode === $other->countryCode;
    }
}
```

### 3. 实体中使用值对象

值对象经常被用作实体的属性类型，这种做法可以增强模型的表达力：

```php
<?php
declare(strict_types=1);

namespace App\Domain\Entity;

use App\Domain\ValueObject\Email;
use App\Domain\ValueObject\PhoneNumber;
use App\Domain\ValueObject\Address;

class Customer
{
    private ?int $id = null;

    public function __construct(
        private readonly string $customerCode,
        private string $name,
        private Email $email,
        private PhoneNumber $phone,
        private Address $billingAddress,
        private Address $shippingAddress
    ) {
        $this->validate();
    }

    private function validate(): void
    {
        if (empty($this->customerCode)) {
            throw new \InvalidArgumentException('Customer code is required');
        }
        if (empty($this->name)) {
            throw new \InvalidArgumentException('Customer name is required');
        }
    }

    public function changeEmail(Email $newEmail): void
    {
        $this->email = $newEmail;
    }

    public function changeShippingAddress(Address $newAddress): void
    {
        $this->shippingAddress = $newAddress;
    }

    public function hasSameBillingAndShippingAddress(): bool
    {
        return $this->billingAddress->equals($this->shippingAddress);
    }

    public function getId(): ?int
    {
        return $this->id;
    }

    public function setId(int $id): void
    {
        $this->id = $id;
    }

    public function getCustomerCode(): string
    {
        return $this->customerCode;
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function getEmail(): Email
    {
        return $this->email;
    }

    public function getPhone(): PhoneNumber
    {
        return $this->phone;
    }

    public function getBillingAddress(): Address
    {
        return $this->billingAddress;
    }

    public function getShippingAddress(): Address
    {
        return $this->shippingAddress;
    }
}
```

在这个示例中，`Customer` 实体使用了多个值对象作为属性：`Email`、`PhoneNumber`、`Address`。这种建模方式使得代码更加清晰，业务规则封装在值对象中。

### 4. 值对象组合

值对象可以组合使用，形成更复杂的值对象：

```php
<?php
declare(strict_types=1);

namespace App\Domain\ValueObject;

class FullName
{
    public function __construct(
        private readonly string $firstName,
        private readonly string $lastName,
        private readonly ?string $middleName = null
    ) {
        $this->validate();
    }

    private function validate(): void
    {
        if (empty($this->firstName)) {
            throw new \InvalidArgumentException('First name is required');
        }
        if (empty($this->lastName)) {
            throw new \InvalidArgumentException('Last name is required');
        }
    }

    public function getFirstName(): string
    {
        return $this->firstName;
    }

    public function getLastName(): string
    {
        return $this->lastName;
    }

    public function getMiddleName(): ?string
    {
        return $this->middleName;
    }

    public function getFullName(): string
    {
        $parts = array_filter([
            $this->lastName,
            $this->middleName,
            $this->firstName
        ]);
        return implode(' ', $parts);
    }

    public function equals(FullName $other): bool
    {
        return $this->firstName === $other->firstName
            && $this->lastName === $other->lastName
            && $this->middleName === $other->middleName;
    }
}
```

## 使用场景

### 实体的使用场景

- 用户、订单、产品等具有唯一身份的业务对象
- 需要追踪历史变化的对象
- 需要版本控制或审计的对象
- 有复杂生命周期管理的对象

### 值对象的使用场景

- 金额、数量、百分比等计量值
- 地址、电话、邮箱等联系信息
- 颜色、尺寸、规格等描述性属性
- 日期范围、价格范围等范围值

## 注意事项

### 1. 避免贫血模型

实体不应该只是数据的容器，还应该包含业务行为。充血模型将业务逻辑封装在实体中，使得业务规则更加清晰、更易于测试。

### 2. 值对象的不可变性

值对象的不可变性是其最重要的特性之一。确保值对象的所有属性都是只读的，不要提供任何修改值对象状态的方法。

### 3. 标识的选择

实体的标识应该根据业务场景选择。业务生成的标识（如订单号、会员号）可能比技术生成的 ID 更有业务意义。

### 4. 值对象的创建

值对象应该在构造时完成所有的验证和初始化，不应该在对象创建后再修改其状态。

## 常见问题

### Q1: 什么时候使用实体而不是值对象？

如果业务概念需要唯一标识来区分不同实例，应该使用实体。如果业务概念仅用来描述其他实体的特征，且相等性由属性值决定，应该使用值对象。

### Q2: 值对象可以包含行为吗？

是的，值对象可以包含辅助性的方法，如计算、格式化等。但这些行为不应该修改值对象自身的状态。

### Q3: 实体的标识可以修改吗？

通常不推荐修改实体的标识。标识是实体身份的核心，修改标识会导致数据一致性问题。如果确实需要修改标识，应该创建新的实体实例。

### Q4: 值对象可以引用实体吗？

可以，但需要谨慎。值对象引用实体可能会破坏值对象的不可变性。如果值对象需要引用实体，应该只保存实体的标识而不是实体对象本身。

## 最佳实践

### 1. 为标识创建值对象

为实体的标识创建专门的值对象类，可以在标识层面添加验证逻辑，提供更好的类型安全性。

### 2. 使用不可变数据结构

将实体的所有值对象属性设置为只读，确保值对象的不可变性。使用 PHP 8.1 的 readonly 属性可以简化实现。

### 3. 实现 equals 方法

为所有值对象实现 equals 方法，确保正确的值相等性比较。避免使用 == 比较对象，而是使用显式的 equals 方法。

### 4. 封装验证逻辑

在值对象的构造函数中封装所有验证逻辑，确保值对象在整个生命周期内都是有效的。

### 5. 保持值对象简单

值对象应该保持简单，只包含相关的属性和必要的辅助方法。避免在值对象中引入复杂的业务逻辑。

## 练习任务

### 练习 1：设计产品实体

为一个电商系统设计产品实体（Product），包含产品 ID、名称、价格、库存、分类等属性。识别哪些应该是实体属性，哪些应该是值对象属性。

### 练习 2：实现地址值对象

实现一个地址值对象（Address），包含省、市、区、街道、邮编等属性。实现验证逻辑和相等性比较方法。

### 练习 3：重构现有代码

将一个现有的数据模型（包含用户、订单等）重构为 DDD 风格的实体和值对象。识别哪些是实体，哪些是值对象。

### 练习 4：实现订单项实体

实现一个订单项（OrderItem）实体。思考：订单项应该是实体还是值对象？为什么？

### 练习 5：设计货币值对象

设计一个货币值对象（Money），支持不同的货币类型，实现货币的加法、减法、乘法运算。
