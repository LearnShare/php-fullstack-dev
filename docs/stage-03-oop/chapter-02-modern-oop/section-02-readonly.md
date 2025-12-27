# 3.2.2 Readonly 属性

## 概述

Readonly 属性是 PHP 8.1 引入的特性，用于创建不可变对象（Immutable Objects）。`readonly` 关键字可以修饰类属性，使属性在初始化后不能被修改。PHP 8.2 进一步引入了 `readonly class`，可以将整个类的所有属性自动设为 readonly。

不可变对象是函数式编程和面向对象编程中的重要概念。不可变对象一旦创建就不能被修改，这带来了许多好处：线程安全、更好的可预测性、更容易的测试和调试。Readonly 属性是实现不可变对象的关键机制。

理解 readonly 属性对于编写现代 PHP 代码非常重要。它特别适用于值对象（Value Objects）、DTO（数据传输对象）、配置对象等场景，可以防止意外的属性修改，提高代码的安全性。

**主要内容**：
- Readonly 属性的定义和使用（PHP 8.1+）
- Readonly 类的定义和使用（PHP 8.2+）
- Readonly 属性与构造器属性提升的结合使用
- 与对象克隆的关系
- 初始化规则和限制
- 使用场景和最佳实践

## 特性

- **不可变性**：属性只能初始化一次，之后不能被修改
- **类型安全**：readonly 属性必须有类型声明
- **简化代码**：使用 readonly 类可以简化不可变对象的定义
- **线程安全**：不可变对象天然线程安全
- **可预测性**：对象状态不会改变，更容易推理

## 语法/定义

### Readonly 属性（PHP 8.1+）

**语法**：`public readonly Type $propertyName;` 或 `public function __construct(public readonly Type $propertyName)`

**组成部分**：
- `readonly` 关键字：修饰属性，使其不可变
- 可见性修饰符：`public`、`protected` 或 `private`
- 类型声明：必须指定类型（readonly 属性的强制要求）
- 属性名：遵循 PHP 变量命名规范

**特点**：
- PHP 8.1+ 引入的特性
- 属性只能在声明时或构造函数中初始化
- 初始化后不能被修改
- 必须指定类型声明

### Readonly 类（PHP 8.2+）

**语法**：`readonly class ClassName { ... }`

**组成部分**：
- `readonly` 关键字：修饰类
- 类体：包含属性和方法

**特点**：
- PHP 8.2+ 引入的特性
- 类的所有属性自动为 readonly
- 属性必须在构造器中初始化
- 不能包含非 readonly 属性
- 所有属性必须有类型声明

## 基本用法

### 示例 1：Readonly 属性

```php
<?php
declare(strict_types=1);

class Point
{
    public function __construct(
        public readonly int $x,
        public readonly int $y
    ) {}
    
    public function getX(): int
    {
        return $this->x;
    }
    
    public function getY(): int
    {
        return $this->y;
    }
}

$point = new Point(10, 20);
echo "X: {$point->x}, Y: {$point->y}\n";  // X: 10, Y: 20

// 错误：不能修改 readonly 属性
// $point->x = 30;  // Fatal error: Cannot modify readonly property Point::$x
// $point->y = 40;  // Fatal error: Cannot modify readonly property Point::$y
```

**输出**：

```
X: 10, Y: 20
```

**说明**：
- `readonly` 属性在构造器中初始化
- 初始化后不能被修改
- 试图修改会抛出致命错误

### 示例 2：Readonly 类（PHP 8.2+）

```php
<?php
declare(strict_types=1);

readonly class User
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email
    ) {}
    
    public function getId(): int
    {
        return $this->id;
    }
    
    public function getName(): string
    {
        return $this->name;
    }
}

$user = new User(1, "John Doe", "john@example.com");
echo "ID: {$user->id}, Name: {$user->name}\n";

// 错误：不能修改 readonly 类的属性
// $user->name = "Jane Doe";  // Fatal error: Cannot modify readonly property User::$name
```

**输出**：

```
ID: 1, Name: John Doe
```

**说明**：
- `readonly class` 的所有属性自动为 readonly
- 不需要在每个属性前添加 `readonly` 关键字
- 所有属性必须在构造器中初始化

### 示例 3：Readonly 属性与构造器属性提升结合

```php
<?php
declare(strict_types=1);

class Money
{
    public function __construct(
        public readonly float $amount,
        public readonly string $currency = "USD"
    ) {}
    
    public function add(Money $other): Money
    {
        if ($this->currency !== $other->currency) {
            throw new InvalidArgumentException("Currency mismatch");
        }
        // 创建新对象而不是修改现有对象
        return new Money($this->amount + $other->amount, $this->currency);
    }
    
    public function getAmount(): float
    {
        return $this->amount;
    }
    
    public function getCurrency(): string
    {
        return $this->currency;
    }
}

$money1 = new Money(100.0, "USD");
$money2 = new Money(50.0, "USD");
$total = $money1->add($money2);

echo "Total: {$total->getCurrency()} {$total->getAmount()}\n";  // Total: USD 150
```

**输出**：

```
Total: USD 150
```

**说明**：
- Readonly 属性可以与构造器属性提升结合使用
- 由于属性不可变，操作返回新对象而不是修改现有对象
- 这是不可变对象的典型使用模式

### 示例 4：与克隆的关系

```php
<?php
declare(strict_types=1);

readonly class User
{
    public function __construct(
        public string $name,
        public int $age
    ) {}
}

$user1 = new User("John", 25);
echo "User 1: {$user1->name}, {$user1->age}\n";

// 可以克隆 readonly 对象
$user2 = clone $user1;
echo "User 2: {$user2->name}, {$user2->age}\n";  // User 2: John, 25

// 但是克隆后的对象属性仍然是 readonly，不能修改
// $user2->name = "Jane";  // 错误：不能修改 readonly 属性

// 如果需要不同的对象，创建新对象
$user3 = new User("Jane", 30);
echo "User 3: {$user3->name}, {$user3->age}\n";
```

**输出**：

```
User 1: John, 25
User 2: John, 25
User 3: Jane, 30
```

**说明**：
- Readonly 对象可以克隆
- 但克隆后的对象属性仍然是 readonly
- 如果需要不同的对象，应该创建新对象

### 示例 5：值对象模式

```php
<?php
declare(strict_types=1);

readonly class Email
{
    public function __construct(
        public string $address
    ) {
        if (!filter_var($this->address, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email address: {$this->address}");
        }
    }
    
    public function getDomain(): string
    {
        $parts = explode('@', $this->address);
        return $parts[1] ?? '';
    }
    
    public function equals(Email $other): bool
    {
        return $this->address === $other->address;
    }
}

$email1 = new Email("user@example.com");
$email2 = new Email("user@example.com");
$email3 = new Email("admin@example.com");

echo "Email 1: {$email1->address}\n";
echo "Email 2: {$email2->address}\n";
echo "Email 1 equals Email 2: " . ($email1->equals($email2) ? "Yes" : "No") . "\n";
echo "Email 1 equals Email 3: " . ($email1->equals($email3) ? "Yes" : "No") . "\n";
```

**输出**：

```
Email 1: user@example.com
Email 2: user@example.com
Email 1 equals Email 2: Yes
Email 1 equals Email 3: No
```

**说明**：
- Readonly 类非常适合实现值对象
- 值对象是不可变的，相等性基于值而不是引用
- 提供了验证逻辑和辅助方法

### 示例 6：混合使用 readonly 和普通属性

```php
<?php
declare(strict_types=1);

class Product
{
    // 普通属性：可以修改
    private int $viewCount = 0;
    
    public function __construct(
        // Readonly 属性：不可修改
        public readonly string $name,
        public readonly float $price,
        public readonly string $sku
    ) {}
    
    public function incrementViewCount(): void
    {
        $this->viewCount++;  // 可以修改普通属性
    }
    
    public function getViewCount(): int
    {
        return $this->viewCount;
    }
    
    // 错误：不能修改 readonly 属性
    // public function setPrice(float $newPrice): void
    // {
    //     $this->price = $newPrice;  // 错误：Cannot modify readonly property
    // }
}

$product = new Product("Laptop", 999.99, "LAP-001");
$product->incrementViewCount();
echo "Product: {$product->name}, Views: {$product->getViewCount()}\n";
```

**输出**：

```
Product: Laptop, Views: 1
```

**说明**：
- 可以在类中混合使用 readonly 属性和普通属性
- Readonly 属性不可修改，普通属性可以修改
- 适合需要部分不可变的场景

## 使用场景

### 场景 1：值对象（Value Objects）

值对象应该是不可变的，readonly 类非常适合实现值对象。

**示例**：坐标点值对象

```php
<?php
declare(strict_types=1);

readonly class Point
{
    public function __construct(
        public int $x,
        public int $y
    ) {}
    
    public function add(Point $other): Point
    {
        return new Point($this->x + $other->x, $this->y + $other->y);
    }
    
    public function distance(Point $other): float
    {
        $dx = $this->x - $other->x;
        $dy = $this->y - $other->y;
        return sqrt($dx * $dx + $dy * $dy);
    }
}

$point1 = new Point(0, 0);
$point2 = new Point(3, 4);
$point3 = $point1->add($point2);

echo "Point 1: ({$point1->x}, {$point1->y})\n";
echo "Point 2: ({$point2->x}, {$point2->y})\n";
echo "Point 3: ({$point3->x}, {$point3->y})\n";
echo "Distance: " . $point1->distance($point2) . "\n";
```

### 场景 2：DTO（数据传输对象）

DTO 通常用于在不同层之间传输数据，应该是不可变的。

**示例**：API 响应 DTO

```php
<?php
declare(strict_types=1);

readonly class UserResponse
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email,
        public DateTime $createdAt
    ) {}
    
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

$user = new UserResponse(1, "John Doe", "john@example.com", new DateTime());
print_r($user->toArray());
```

### 场景 3：配置对象

配置对象应该是不可变的，防止配置被意外修改。

**示例**：应用配置

```php
<?php
declare(strict_types=1);

readonly class AppConfig
{
    public function __construct(
        public string $environment,
        public string $databaseHost,
        public int $databasePort,
        public bool $debug = false
    ) {}
    
    public function isProduction(): bool
    {
        return $this->environment === 'production';
    }
    
    public function getDatabaseDsn(): string
    {
        return "mysql:host={$this->databaseHost};port={$this->databasePort}";
    }
}

$config = new AppConfig('production', 'localhost', 3306);
echo "Environment: {$config->environment}\n";
echo "Is Production: " . ($config->isProduction() ? "Yes" : "No") . "\n";
```

## 注意事项

### Readonly 属性的限制

#### 1. 必须初始化

- Readonly 属性必须在声明时或构造器中初始化
- 不能在构造器外赋值
- 不能有默认值（除非是常量表达式）

**示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    // 错误：不能有非常量表达式的默认值
    // public readonly string $name = "Default";
    
    // 正确：在构造器中初始化
    public function __construct(
        public readonly string $name
    ) {}
    
    // 错误：不能在构造器外赋值
    // public function setName(string $name): void
    // {
    //     $this->name = $name;  // 错误：Cannot modify readonly property
    // }
}
```

#### 2. 类型声明要求

- Readonly 属性必须有类型声明
- 不能使用无类型属性

**示例**：

```php
<?php
declare(strict_types=1);

// 正确：有类型声明
class GoodExample
{
    public function __construct(
        public readonly string $name
    ) {}
}

// 错误：没有类型声明
// class BadExample
// {
//     public function __construct(
//         public readonly $name  // 错误：Readonly property must have a type
//     ) {}
// }
```

#### 3. 不能是 static

- Readonly 属性不能是静态属性
- 静态属性不属于对象实例，不能使用 readonly

**示例**：

```php
<?php
declare(strict_types=1);

// 错误：不能是 static
// class Example
// {
//     public static readonly int $count = 0;  // 错误：不能使用 static readonly
// }

// 正确：使用普通静态属性
class CorrectExample
{
    public static int $count = 0;
}
```

### Readonly 类的限制

#### 1. 所有属性必须是 readonly

- Readonly 类不能包含非 readonly 属性
- 所有属性自动为 readonly

**示例**：

```php
<?php
declare(strict_types=1);

// 错误：不能有非 readonly 属性
// readonly class BadExample
// {
//     public string $name;  // 错误：Readonly class cannot have non-readonly properties
//     private int $count = 0;  // 错误：不能有非 readonly 属性
// }

// 正确：所有属性都是 readonly
readonly class GoodExample
{
    public function __construct(
        public string $name,
        public int $count
    ) {}
}
```

#### 2. 属性类型声明要求

- Readonly 类的所有属性必须有类型声明
- 不能使用无类型属性

#### 3. 继承规则

- Readonly 类可以继承其他类
- Readonly 类只能被 readonly 类继承
- 普通类不能继承 readonly 类

**示例**：

```php
<?php
declare(strict_types=1);

readonly class Base
{
    public function __construct(public string $name) {}
}

// 正确：readonly 类继承 readonly 类
readonly class Child extends Base
{
    public function __construct(
        string $name,
        public int $age
    ) {
        parent::__construct($name);
    }
}

// 错误：普通类不能继承 readonly 类
// class Normal extends Base {}  // 错误：Non-readonly class cannot extend readonly class
```

## 常见问题

### 问题 1：修改 readonly 属性错误

**错误信息**：`Fatal error: Cannot modify readonly property`

**原因**：试图修改已经初始化的 readonly 属性

**解决方案**：
- 创建新对象而不是修改现有对象
- 使用不可变对象模式（返回新对象）

**示例**：

```php
<?php
declare(strict_types=1);

readonly class Point
{
    public function __construct(
        public int $x,
        public int $y
    ) {}
    
    // 错误的方式：试图修改属性
    // public function setX(int $x): void
    // {
    //     $this->x = $x;  // 错误：Cannot modify readonly property
    // }
    
    // 正确的方式：返回新对象
    public function withX(int $x): Point
    {
        return new Point($x, $this->y);
    }
    
    public function withY(int $y): Point
    {
        return new Point($this->x, $y);
    }
}

$point = new Point(10, 20);
$newPoint = $point->withX(30);  // 创建新对象
echo "Original: ({$point->x}, {$point->y})\n";
echo "New: ({$newPoint->x}, {$newPoint->y})\n";
```

### 问题 2：未初始化错误

**错误信息**：`Fatal error: Readonly property must be initialized`

**原因**：Readonly 属性未在构造器中初始化

**解决方案**：在构造器中初始化所有 readonly 属性

**示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    public readonly string $name;
    
    // 错误：未初始化
    // public function __construct() {}  // 错误：必须初始化 $name
    
    // 正确：在构造器中初始化
    public function __construct(string $name)
    {
        $this->name = $name;
    }
}
```

### 问题 3：类型声明缺失

**错误信息**：`Fatal error: Readonly property must have a type`

**原因**：Readonly 属性没有类型声明

**解决方案**：为所有 readonly 属性指定类型声明

**示例**：

```php
<?php
declare(strict_types=1);

// 错误：没有类型声明
// class BadExample
// {
//     public readonly $name;  // 错误：Readonly property must have a type
// }

// 正确：有类型声明
class GoodExample
{
    public function __construct(
        public readonly string $name
    ) {}
}
```

## 最佳实践

### 1. 值对象使用 readonly 类

值对象应该是不可变的，使用 `readonly class` 可以确保不可变性。

**示例**：

```php
<?php
declare(strict_types=1);

readonly class Money
{
    public function __construct(
        public float $amount,
        public string $currency
    ) {
        if ($amount < 0) {
            throw new InvalidArgumentException("Amount cannot be negative");
        }
    }
    
    public function add(Money $other): Money
    {
        if ($this->currency !== $other->currency) {
            throw new InvalidArgumentException("Currency mismatch");
        }
        return new Money($this->amount + $other->amount, $this->currency);
    }
}
```

### 2. DTO 使用 readonly 类

DTO 用于传输数据，应该是不可变的。

**示例**：

```php
<?php
declare(strict_types=1);

readonly class OrderDTO
{
    public function __construct(
        public int $id,
        public string $customerName,
        public float $total,
        public DateTime $createdAt
    ) {}
}
```

### 3. 实现不可变对象的修改方法

对于需要"修改"的不可变对象，提供返回新对象的方法。

**示例**：

```php
<?php
declare(strict_types=1);

readonly class User
{
    public function __construct(
        public string $name,
        public int $age,
        public string $email
    ) {}
    
    public function withName(string $name): self
    {
        return new self($name, $this->age, $this->email);
    }
    
    public function withAge(int $age): self
    {
        return new self($this->name, $age, $this->email);
    }
    
    public function withEmail(string $email): self
    {
        return new self($this->name, $this->age, $email);
    }
}

$user = new User("John", 25, "john@example.com");
$updatedUser = $user->withAge(26);  // 创建新对象，不修改原对象
```

### 4. 在构造器中添加验证

利用构造器添加验证逻辑，确保对象创建时就是有效的。

**示例**：

```php
<?php
declare(strict_types=1);

readonly class Email
{
    public function __construct(
        public string $address
    ) {
        if (!filter_var($this->address, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email: {$this->address}");
        }
    }
}
```

## 对比分析

### Readonly 属性 vs 普通属性

| 特性         | Readonly 属性                  | 普通属性                       |
|:-------------|:-------------------------------|:-------------------------------|
| **可修改性**  | 不可修改（初始化后）            | 可以修改                       |
| **类型声明**  | 必须                          | 可选                           |
| **初始化**    | 必须在声明时或构造器中          | 可以延迟初始化                 |
| **适用场景**  | 不可变对象、值对象              | 可变对象                       |
| **线程安全**  | 是（不可变）                   | 否（需要同步）                 |

### Readonly 类 vs 普通类

| 特性         | Readonly 类                    | 普通类                         |
|:-------------|:-------------------------------|:-------------------------------|
| **属性性质**  | 所有属性自动为 readonly        | 属性可以修改                   |
| **类型要求**  | 所有属性必须有类型声明          | 类型声明可选                   |
| **继承规则**  | 只能被 readonly 类继承          | 可以被任何类继承               |
| **适用场景**  | 值对象、DTO、配置对象           | 可变对象、实体类                |

### Readonly vs final

| 特性         | Readonly                       | final                          |
|:-------------|:-------------------------------|:-------------------------------|
| **作用对象**  | 属性（不能修改值）              | 类/方法（不能继承/覆盖）        |
| **目的**      | 实现不可变性                    | 防止继承/覆盖                  |
| **层次**      | 对象状态                       | 类结构                         |

## 练习任务

1. **创建值对象**：使用 `readonly class` 创建一个 `Point` 值对象，包含 `x` 和 `y` 坐标，实现 `add()` 和 `distance()` 方法。

2. **创建 DTO**：使用 `readonly class` 创建一个 `ProductDTO` 类，包含产品信息（ID、名称、价格、描述），实现 `toArray()` 方法。

3. **不可变配置类**：使用 `readonly class` 创建一个 `DatabaseConfig` 类，包含数据库连接信息，实现验证逻辑。

4. **实现修改方法**：创建一个 `readonly class User`，实现 `withName()`、`withAge()` 等方法，返回新对象而不是修改现有对象。

5. **混合使用练习**：创建一个类，部分属性使用 `readonly`（不可变），部分属性为普通属性（可变），展示混合使用的场景。

## 相关章节

- **[3.2.1 构造器属性提升](section-01-constructor-promotion.md)**：了解如何结合使用构造器属性提升
- **[3.1.4 对象克隆与引用](../chapter-01-classes/section-04-clone-references.md)**：了解 readonly 对象与克隆的关系
- **[3.3.1 继承（Inheritance）](../chapter-03-oop-features/section-01-inheritance.md)**：了解 readonly 类的继承规则
