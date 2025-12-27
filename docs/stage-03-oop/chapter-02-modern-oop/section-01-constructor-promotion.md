# 3.2.1 构造器属性提升

## 概述

构造器属性提升（Constructor Property Promotion）是 PHP 8.0 引入的特性，允许在构造函数的参数列表中直接声明类属性，从而简化类的定义。这一特性可以显著减少样板代码，提升代码可读性和维护性。

在传统写法中，我们需要先声明属性，然后在构造函数中接收参数并赋值给属性。构造器属性提升将这三个步骤合并为一步：直接在构造函数的参数列表中声明属性并指定可见性修饰符，PHP 会自动创建属性并完成赋值。

理解构造器属性提升对于编写现代 PHP 代码非常重要。它不仅减少了代码量，还让属性定义和初始化逻辑更加集中，便于理解和维护。这一特性特别适用于 DTO（数据传输对象）、值对象、配置类等场景。

**主要内容**：
- 构造器属性提升的基础语法
- 可见性修饰符的使用（`public`、`protected`、`private`）
- 类型声明的使用
- 默认值的设置
- 混合使用（提升属性和普通属性）
- 与传统写法的详细对比
- 使用场景和最佳实践

## 特性

- **简化代码**：减少重复的属性声明和赋值代码
- **类型安全**：支持完整的类型声明
- **灵活性**：支持所有可见性修饰符和默认值
- **可读性**：属性定义和初始化逻辑集中在一起
- **向后兼容**：可以混合使用提升属性和普通属性

## 语法/定义

### 基础语法

**语法**：`public function __construct(public Type $propertyName) { ... }`

**组成部分**：
- 可见性修饰符（`public`、`protected`、`private`）：定义属性的访问权限
- 类型声明（可选）：指定属性的数据类型
- 属性名：`$propertyName`，遵循 PHP 变量命名规范
- 默认值（可选）：可以设置参数默认值

**特点**：
- PHP 8.0+ 引入的特性
- 参数自动成为类的属性
- 支持所有可见性修饰符
- 支持类型声明和默认值

### 可见性修饰符

**支持的修饰符**：`public`、`protected`、`private`

**语法示例**：
- `public Type $property`：公开属性
- `protected Type $property`：受保护属性
- `private Type $property`：私有属性

### 类型声明

**支持的类型**：所有 PHP 类型（标量类型、复合类型、联合类型、交集类型等）

**语法示例**：
- `public string $name`
- `public int $age`
- `public array $tags`
- `public ?string $email`（可空类型）
- `public string|int $value`（联合类型，PHP 8.0+）

### 默认值

**语法**：`public Type $property = defaultValue`

**特点**：
- 可以为提升属性设置默认值
- 默认值必须是与类型兼容的值
- 有默认值的参数必须放在没有默认值的参数之后

## 基本用法

### 示例 1：基础构造器属性提升

```php
<?php
declare(strict_types=1);

// 传统写法
class UserTraditional
{
    private string $name;
    private int $age;
    private string $email;
    
    public function __construct(string $name, int $age, string $email)
    {
        $this->name = $name;
        $this->age = $age;
        $this->email = $email;
    }
    
    public function getName(): string { return $this->name; }
    public function getAge(): int { return $this->age; }
    public function getEmail(): string { return $this->email; }
}

// 构造器属性提升写法（PHP 8.0+）
class User
{
    public function __construct(
        private string $name,
        private int $age,
        private string $email
    ) {}
    
    public function getName(): string { return $this->name; }
    public function getAge(): int { return $this->age; }
    public function getEmail(): string { return $this->email; }
}

$user = new User("John Doe", 25, "john@example.com");
echo "Name: " . $user->getName() . "\n";
echo "Age: " . $user->getAge() . "\n";
echo "Email: " . $user->getEmail() . "\n";
```

**输出**：

```
Name: John Doe
Age: 25
Email: john@example.com
```

**说明**：
- 构造器属性提升将属性声明和赋值合并为一步
- 代码更加简洁，减少了重复代码
- 功能完全等价于传统写法

### 示例 2：使用不同的可见性修饰符

```php
<?php
declare(strict_types=1);

class BankAccount
{
    public function __construct(
        public int $accountNumber,      // 公开属性
        protected float $balance,       // 受保护属性
        private string $pin             // 私有属性
    ) {}
    
    public function getAccountNumber(): int
    {
        return $this->accountNumber;
    }
    
    public function getBalance(): float
    {
        return $this->balance;  // 可以访问 protected 属性
    }
    
    public function verifyPin(string $input): bool
    {
        return $this->pin === $input;  // 可以访问 private 属性
    }
}

$account = new BankAccount(12345, 1000.0, "1234");

// 可以访问 public 属性
echo "Account Number: " . $account->accountNumber . "\n";

// 不能直接访问 protected 和 private 属性
// echo $account->balance;  // 错误：不能访问 protected 属性
// echo $account->pin;      // 错误：不能访问 private 属性

echo "Balance: " . $account->getBalance() . "\n";
echo "PIN verified: " . ($account->verifyPin("1234") ? "Yes" : "No") . "\n";
```

**输出**：

```
Account Number: 12345
Balance: 1000
PIN verified: Yes
```

**说明**：
- 可以在构造器属性提升中使用所有可见性修饰符
- `public` 属性可以直接访问
- `protected` 和 `private` 属性需要通过方法访问（详细说明见 [3.1.2 可见性修饰符](../chapter-01-classes/section-02-visibility.md)）

### 示例 3：使用类型声明

```php
<?php
declare(strict_types=1);

class Product
{
    public function __construct(
        private string $name,
        private float $price,
        private int $stock = 0,              // 带默认值
        private ?string $description = null  // 可空类型，带默认值
    ) {}
    
    public function getName(): string { return $this->name; }
    public function getPrice(): float { return $this->price; }
    public function getStock(): int { return $this->stock; }
    public function getDescription(): ?string { return $this->description; }
}

// 使用默认值
$product1 = new Product("Laptop", 999.99);
echo "Product: " . $product1->getName() . "\n";
echo "Stock: " . $product1->getStock() . "\n";  // 0（默认值）

// 指定所有参数
$product2 = new Product("Mouse", 29.99, 10, "Wireless mouse");
echo "Product: " . $product2->getName() . "\n";
echo "Description: " . $product2->getDescription() . "\n";
```

**输出**：

```
Product: Laptop
Stock: 0
Product: Mouse
Description: Wireless mouse
```

**说明**：
- 支持完整的类型声明（包括可空类型）
- 可以为提升属性设置默认值
- 有默认值的参数可以省略

### 示例 4：混合使用提升属性和普通属性

```php
<?php
declare(strict_types=1);

class User
{
    private string $id;  // 普通属性
    private DateTime $createdAt;  // 普通属性
    
    public function __construct(
        private string $name,
        private string $email,
        private int $age
    ) {
        // 在构造函数体中初始化普通属性
        $this->id = uniqid('user_', true);
        $this->createdAt = new DateTime();
    }
    
    public function getId(): string { return $this->id; }
    public function getName(): string { return $this->name; }
    public function getEmail(): string { return $this->email; }
    public function getAge(): int { return $this->age; }
    public function getCreatedAt(): DateTime { return $this->createdAt; }
}

$user = new User("John", "john@example.com", 25);
echo "ID: " . $user->getId() . "\n";
echo "Name: " . $user->getName() . "\n";
echo "Created at: " . $user->getCreatedAt()->format('Y-m-d H:i:s') . "\n";
```

**输出**：

```
ID: user_676a1b2c3d4e5
Name: John
Created at: 2025-01-15 12:00:00
```

**说明**：
- 可以混合使用提升属性和普通属性
- 提升属性在参数列表中定义和初始化
- 普通属性在类体中声明，在构造函数体中初始化
- 适合需要复杂初始化逻辑的属性

### 示例 5：使用联合类型（PHP 8.0+）

```php
<?php
declare(strict_types=1);

class FlexibleValue
{
    public function __construct(
        private string|int $value,  // 联合类型
        private bool $isString = false
    ) {}
    
    public function getValue(): string|int
    {
        return $this->value;
    }
    
    public function getAsString(): string
    {
        return (string) $this->value;
    }
    
    public function getAsInt(): int
    {
        if (is_int($this->value)) {
            return $this->value;
        }
        return (int) $this->value;
    }
}

$value1 = new FlexibleValue("123");
$value2 = new FlexibleValue(456);

echo "Value 1: " . $value1->getValue() . "\n";
echo "Value 2: " . $value2->getValue() . "\n";
```

**输出**：

```
Value 1: 123
Value 2: 456
```

**说明**：
- 构造器属性提升支持联合类型（PHP 8.0+）
- 提供了更灵活的类型定义
- 需要在方法中进行类型检查

### 示例 6：参数验证和额外逻辑

```php
<?php
declare(strict_types=1);

class Email
{
    public function __construct(
        private string $address
    ) {
        // 可以在构造函数体中添加验证逻辑
        if (!filter_var($this->address, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Invalid email address: {$this->address}");
        }
    }
    
    public function getAddress(): string
    {
        return $this->address;
    }
}

// 有效邮箱
$email1 = new Email("user@example.com");
echo "Email: " . $email1->getAddress() . "\n";

// 无效邮箱（抛出异常）
try {
    $email2 = new Email("invalid-email");
} catch (InvalidArgumentException $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

**输出**：

```
Email: user@example.com
Error: Invalid email address: invalid-email
```

**说明**：
- 可以在构造函数体中添加验证逻辑
- 提升属性在构造函数体执行前已经赋值
- 可以在构造函数体中访问提升属性

## 使用场景

### 场景 1：DTO（数据传输对象）

构造器属性提升非常适合 DTO，简化数据传输对象的定义。

**示例**：API 响应 DTO

```php
<?php
declare(strict_types=1);

class UserResponse
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email,
        public DateTime $createdAt
    ) {}
}

// 从数据库或 API 创建 DTO
$response = new UserResponse(
    1,
    "John Doe",
    "john@example.com",
    new DateTime()
);

// 可以直接访问属性或转换为数组
$data = [
    'id' => $response->id,
    'name' => $response->name,
    'email' => $response->email,
    'created_at' => $response->createdAt->format('Y-m-d H:i:s')
];
```

### 场景 2：值对象

值对象可以使用构造器属性提升简化定义。

**示例**：货币值对象

```php
<?php
declare(strict_types=1);

class Money
{
    public function __construct(
        private float $amount,
        private string $currency = "USD"
    ) {
        if ($amount < 0) {
            throw new InvalidArgumentException("Amount cannot be negative");
        }
    }
    
    public function getAmount(): float { return $this->amount; }
    public function getCurrency(): string { return $this->currency; }
    
    public function add(Money $other): Money
    {
        if ($this->currency !== $other->currency) {
            throw new InvalidArgumentException("Currency mismatch");
        }
        return new Money($this->amount + $other->amount, $this->currency);
    }
}

$money1 = new Money(100.0, "USD");
$money2 = new Money(50.0, "USD");
$total = $money1->add($money2);
echo "Total: {$total->getCurrency()} {$total->getAmount()}\n";
```

### 场景 3：配置对象

配置对象可以使用构造器属性提升，支持默认值。

**示例**：数据库配置

```php
<?php
declare(strict_types=1);

class DatabaseConfig
{
    public function __construct(
        private string $host,
        private string $database,
        private string $username,
        private string $password,
        private int $port = 3306,
        private string $charset = "utf8mb4"
    ) {}
    
    public function getDsn(): string
    {
        return "mysql:host={$this->host};port={$this->port};dbname={$this->database};charset={$this->charset}";
    }
    
    public function getUsername(): string { return $this->username; }
    public function getPassword(): string { return $this->password; }
}

$config = new DatabaseConfig(
    "localhost",
    "mydb",
    "user",
    "password"
    // port 和 charset 使用默认值
);

echo "DSN: " . $config->getDsn() . "\n";
```

## 注意事项

### 版本要求

- **PHP 版本**：需要 PHP 8.0 或更高版本
- **兼容性**：旧版本 PHP 不支持此特性
- **检查方法**：使用 `phpversion()` 检查 PHP 版本

### 类型声明要求

- **推荐使用类型声明**：虽然类型声明是可选的，但强烈推荐使用
- **类型安全**：类型声明提供编译时类型检查
- **IDE 支持**：类型声明有助于 IDE 提供更好的代码提示

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：使用类型声明
class GoodExample
{
    public function __construct(
        private string $name,
        private int $age
    ) {}
}

// 不推荐：不使用类型声明
class BadExample
{
    public function __construct(
        private $name,  // 没有类型声明
        private $age    // 没有类型声明
    ) {}
}
```

### 默认值的顺序

- **规则**：有默认值的参数必须放在没有默认值的参数之后
- **原因**：PHP 参数传递的规则
- **示例**：正确和错误的顺序

**示例**：

```php
<?php
declare(strict_types=1);

// 正确：默认值参数在后面
class Correct
{
    public function __construct(
        private string $name,      // 无默认值
        private int $age = 0       // 有默认值
    ) {}
}

// 错误：默认值参数在前面
// class Wrong
// {
//     public function __construct(
//         private string $name = "Guest",  // 有默认值
//         private int $age                 // 无默认值（错误！）
//     ) {}
// }
```

### 不能用于抽象方法

- **限制**：构造器属性提升不能用于抽象方法
- **原因**：抽象方法不能有实现
- **替代方案**：在具体类的构造函数中使用

**示例**：

```php
<?php
declare(strict_types=1);

abstract class AbstractClass
{
    // 不能这样写
    // abstract public function __construct(public string $name);
    
    // 正确的做法：在具体类中使用
}

class ConcreteClass extends AbstractClass
{
    public function __construct(
        private string $name  // 可以在这里使用
    ) {}
}
```

### 与 readonly 的结合使用

- **PHP 8.1+**：可以在构造器属性提升中使用 `readonly` 修饰符
- **readonly 类**：PHP 8.2+ 支持 `readonly class`（详细说明见 [3.2.2 Readonly 属性](section-02-readonly.md)）

**示例**：

```php
<?php
declare(strict_types=1);

// PHP 8.1+：readonly 属性
class User81
{
    public function __construct(
        public readonly string $name,
        public readonly int $age
    ) {}
}

// PHP 8.2+：readonly 类
readonly class User82
{
    public function __construct(
        public string $name,
        public int $age
    ) {}
}
```

## 常见问题

### 问题 1：版本不兼容错误

**错误信息**：`Parse error: syntax error, unexpected 'public'`

**原因**：使用了低于 PHP 8.0 的版本

**解决方案**：
- 升级 PHP 版本到 8.0 或更高
- 检查 PHP 版本：`php -v`
- 在代码中检查版本兼容性

**示例**：

```php
<?php
// 检查 PHP 版本
if (version_compare(PHP_VERSION, '8.0.0', '<')) {
    die("This code requires PHP 8.0 or higher");
}

// 使用构造器属性提升
class User
{
    public function __construct(
        private string $name
    ) {}
}
```

### 问题 2：类型错误

**错误信息**：`TypeError` 或类型不匹配

**原因**：传递的参数类型与声明的类型不匹配

**解决方案**：
- 确保传递的参数类型正确
- 使用 `declare(strict_types=1);` 启用严格类型
- 进行类型检查或转换

**示例**：

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        private int $age
    ) {}
}

// 错误：传递字符串而不是整数
// $user = new User("25");  // TypeError

// 正确：传递整数
$user = new User(25);
```

### 问题 3：默认值顺序错误

**错误信息**：`Fatal error: Cannot use positional argument after named argument`

**原因**：有默认值的参数放在了没有默认值的参数之前

**解决方案**：调整参数顺序，将有默认值的参数放在后面

**示例**：

```php
<?php
declare(strict_types=1);

// 正确：默认值参数在后面
class Correct
{
    public function __construct(
        private string $name,
        private int $age = 0
    ) {}
}

// 错误：默认值参数在前面
// class Wrong
// {
//     public function __construct(
//         private string $name = "Guest",
//         private int $age  // 错误：无默认值的参数不能在有默认值的参数之后
//     ) {}
// }
```

## 最佳实践

### 1. 始终使用类型声明

类型声明提供类型安全，提高代码质量。

**示例**：

```php
<?php
declare(strict_types=1);

// 好的做法：使用类型声明
class GoodExample
{
    public function __construct(
        private string $name,
        private int $age,
        private ?string $email = null
    ) {}
}

// 不好的做法：不使用类型声明
class BadExample
{
    public function __construct(
        private $name,     // 没有类型
        private $age,      // 没有类型
        private $email = null
    ) {}
}
```

### 2. 合理使用可见性修饰符

根据属性的使用需求选择合适的可见性修饰符。

**示例**：

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name,        // public：需要外部访问
        protected int $age,          // protected：需要子类访问
        private string $password    // private：不需要外部访问
    ) {}
}
```

### 3. 使用默认值简化调用

为可选参数提供默认值，简化对象创建。

**示例**：

```php
<?php
declare(strict_types=1);

class Config
{
    public function __construct(
        private string $host,
        private string $database,
        private int $port = 3306,           // 默认端口
        private string $charset = "utf8mb4"  // 默认字符集
    ) {}
}

// 可以使用默认值
$config = new Config("localhost", "mydb");
```

### 4. 在构造函数体中添加验证逻辑

利用构造函数体添加参数验证和额外初始化。

**示例**：

```php
<?php
declare(strict_types=1);

class Product
{
    public function __construct(
        private string $name,
        private float $price
    ) {
        // 验证逻辑
        if (empty(trim($this->name))) {
            throw new InvalidArgumentException("Product name cannot be empty");
        }
        
        if ($this->price < 0) {
            throw new InvalidArgumentException("Price cannot be negative");
        }
    }
}
```

### 5. 理解与传统写法的等价性

构造器属性提升是语法糖，功能等价于传统写法。

**示例**：

```php
<?php
declare(strict_types=1);

// 这两种写法完全等价

// 传统写法
class Traditional
{
    private string $name;
    
    public function __construct(string $name)
    {
        $this->name = $name;
    }
}

// 构造器属性提升写法
class Modern
{
    public function __construct(
        private string $name
    ) {}
}
```

## 对比分析

### 构造器属性提升 vs 传统写法

| 特性         | 构造器属性提升（PHP 8.0+）      | 传统写法                        |
|:-------------|:--------------------------------|:--------------------------------|
| **代码量**    | 更少                            | 更多                            |
| **可读性**    | 更好（定义和初始化在一起）       | 较差（分散在不同位置）           |
| **类型安全**  | 支持                            | 支持                            |
| **灵活性**    | 支持所有可见性修饰符和默认值      | 支持所有特性                     |
| **版本要求**  | PHP 8.0+                        | 所有 PHP 版本                    |
| **性能**      | 相同（编译后等价）               | 相同                             |

**代码对比示例**：

```php
<?php
declare(strict_types=1);

// 传统写法：需要 3 步
class UserTraditional
{
    private string $name;    // 步骤 1：声明属性
    private int $age;        // 步骤 1：声明属性
    
    public function __construct(string $name, int $age)  // 步骤 2：定义参数
    {
        $this->name = $name;  // 步骤 3：赋值
        $this->age = $age;    // 步骤 3：赋值
    }
}

// 构造器属性提升：1 步完成
class UserModern
{
    public function __construct(
        private string $name,  // 一步完成：声明、定义、初始化
        private int $age
    ) {}
}
```

### 提升属性 vs 普通属性

| 特性         | 提升属性                       | 普通属性                        |
|:-------------|:-------------------------------|:--------------------------------|
| **定义位置**  | 构造函数参数列表                | 类体                            |
| **初始化**    | 自动（通过参数）                | 手动（在构造函数体中）           |
| **适用场景**  | 简单属性（直接赋值）            | 复杂属性（需要计算或验证）       |
| **灵活性**    | 较简单                         | 更灵活                          |

## 练习任务

1. **重构现有类**：选择一个使用传统构造函数的类，使用构造器属性提升重构它，比较代码差异。

2. **创建 DTO 类**：创建一个 `OrderDTO` 类，包含订单的相关信息（ID、用户ID、金额、创建时间等），使用构造器属性提升。

3. **混合使用练习**：创建一个 `User` 类，使用构造器属性提升定义基本属性（name、email），使用普通属性定义需要计算的属性（id、createdAt）。

4. **可见性修饰符练习**：创建一个 `BankAccount` 类，使用构造器属性提升定义不同可见性的属性（public accountNumber、protected balance、private pin）。

5. **默认值练习**：创建一个 `DatabaseConfig` 类，使用构造器属性提升定义配置参数，为可选参数（port、charset）设置合理的默认值。

## 相关章节

- **[3.1.3 构造函数与析构函数](../chapter-01-classes/section-03-constructors-destructors.md)**：了解构造函数的基础知识
- **[3.1.2 可见性修饰符](../chapter-01-classes/section-02-visibility.md)**：了解可见性修饰符的详细说明
- **[3.2.2 Readonly 属性](section-02-readonly.md)**：了解如何结合使用 readonly 修饰符
