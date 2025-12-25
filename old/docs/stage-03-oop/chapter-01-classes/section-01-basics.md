# 3.1.1 类与对象基础

## 概述

类（Class）和对象（Object）是面向对象编程的基础。理解类与对象的概念对于掌握 OOP 至关重要。

## 类定义

### 基本语法

- **语法**：`class ClassName { ... }`
- 类名遵循 `StudlyCase` 命名规范（首字母大写，驼峰式）。
- 一个文件通常只包含一个类，文件名与类名保持一致。

```php
<?php
declare(strict_types=1);

class User
{
    // 属性与方法
}
```

### 类命名规范

- 使用 `StudlyCase`（PascalCase）
- 类名应该是有意义的名词
- 避免使用缩写（除非是通用缩写）

```php
// 推荐
class UserAccount {}
class PaymentProcessor {}
class DatabaseConnection {}

// 不推荐
class user {}  // 首字母小写
class usr {}   // 缩写
class UserAcct {}  // 不完整单词
```

## 对象创建

### 基本语法

- **语法**：`$object = new ClassName();`
- 使用 `new` 关键字实例化类，创建对象。
- 对象是类的实例，每个对象拥有独立的属性值。

```php
$user = new User();
$anotherUser = new User();
```

### 对象特性

- 每个对象都是独立的实例
- 对象拥有自己的属性值
- 对象共享类的方法定义

```php
class Counter
{
    public int $value = 0;
}

$counter1 = new Counter();
$counter2 = new Counter();

$counter1->value = 10;
$counter2->value = 20;

echo $counter1->value; // 10
echo $counter2->value; // 20
```

## 属性（Properties）

### 属性定义

- **语法**：`public|protected|private $propertyName;`
- 属性用于存储对象的状态数据。
- PHP 8.0+ 支持类型声明：`public string $name;`

```php
class User
{
    public int $id;
    public string $name;
    public string $email;
    private ?string $passwordHash = null;
}
```

### 属性类型声明

PHP 8.0+ 支持属性类型声明：

```php
class Product
{
    public int $id;
    public string $name;
    public float $price;
    public bool $inStock = true;
    public array $tags = [];
    public ?string $description = null;
}
```

### 属性访问

- 使用 `->` 操作符访问对象属性：`$user->name`
- `public` 属性可直接访问；`protected` 和 `private` 只能在类内部或子类中访问。

```php
$user = new User();
$user->id = 1;
$user->name = 'Alice';
echo $user->name; // Alice

// $user->passwordHash; // 错误：无法访问 private 属性
```

### 属性默认值

- 属性可在声明时设置默认值。
- 未初始化的属性在使用时会触发警告（PHP 8.0+ 会抛出 `TypeError`）。

```php
class Config
{
    public string $environment = 'production';
    public bool $debug = false;
    public array $settings = [];
    public int $timeout = 30;
}
```

### 只读属性（PHP 8.1+）

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

## 方法（Methods）

### 方法定义

- **语法**：`public|protected|private function methodName(参数列表): 返回类型 { ... }`
- 方法用于定义对象的行为。
- 支持参数类型声明与返回类型声明。

```php
class User
{
    public function getName(): string
    {
        return $this->name;
    }

    public function setEmail(string $email): void
    {
        $this->email = $email;
    }
}
```

### 方法参数

```php
class Calculator
{
    public function add(float $a, float $b): float
    {
        return $a + $b;
    }

    public function multiply(float $a, float $b, int $precision = 2): float
    {
        return round($a * $b, $precision);
    }
}
```

### 方法返回类型

```php
class UserService
{
    public function findUser(int $id): ?User
    {
        // 可能返回 User 或 null
        return null;
    }

    public function getAllUsers(): array
    {
        return [];
    }

    public function deleteUser(int $id): void
    {
        // 无返回值
    }
}
```

## `$this` 关键字

### 基本概念

- `$this` 指向当前对象实例。
- 在方法内部使用 `$this->property` 或 `$this->method()` 访问对象的属性和方法。

```php
class Calculator
{
    private float $result = 0.0;

    public function add(float $value): self
    {
        $this->result += $value;
        return $this;
    }

    public function subtract(float $value): self
    {
        $this->result -= $value;
        return $this;
    }

    public function getResult(): float
    {
        return $this->result;
    }
}

$calc = new Calculator();
$calc->add(10)->subtract(5)->add(20);
echo $calc->getResult(); // 25
```

### 方法链式调用

```php
class QueryBuilder
{
    private array $conditions = [];

    public function where(string $field, string $operator, mixed $value): self
    {
        $this->conditions[] = [$field, $operator, $value];
        return $this;
    }

    public function orderBy(string $field, string $direction = 'ASC'): self
    {
        $this->orderBy = [$field, $direction];
        return $this;
    }

    public function get(): array
    {
        // 执行查询
        return [];
    }
}

$query = new QueryBuilder();
$results = $query
    ->where('status', '=', 'active')
    ->where('age', '>', 18)
    ->orderBy('name', 'ASC')
    ->get();
```

## 完整示例

```php
<?php
declare(strict_types=1);

class BankAccount
{
    private float $balance = 0.0;
    private string $accountNumber;

    public function __construct(string $accountNumber)
    {
        $this->accountNumber = $accountNumber;
    }

    public function deposit(float $amount): self
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException('Amount must be positive');
        }
        $this->balance += $amount;
        return $this;
    }

    public function withdraw(float $amount): self
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException('Amount must be positive');
        }
        if ($amount > $this->balance) {
            throw new RuntimeException('Insufficient funds');
        }
        $this->balance -= $amount;
        return $this;
    }

    public function getBalance(): float
    {
        return $this->balance;
    }

    public function getAccountNumber(): string
    {
        return $this->accountNumber;
    }
}

// 使用
$account = new BankAccount('ACC-12345');
$account->deposit(1000)->withdraw(200);
echo $account->getBalance(); // 800
```

## 注意事项

1. **类命名**：使用 `StudlyCase`，首字母大写。

2. **文件组织**：一个文件一个类，文件名与类名一致。

3. **属性初始化**：PHP 8.0+ 未初始化的类型属性会抛出 `TypeError`。

4. **`$this` 使用**：只能在非静态方法中使用 `$this`。

5. **方法链**：返回 `self` 或 `$this` 可以实现方法链式调用。

## 练习

1. 创建一个 `Rectangle` 类，包含 `width` 和 `height` 属性，提供 `getArea()` 和 `getPerimeter()` 方法。

2. 实现一个 `Student` 类，包含 `name`、`age`、`grade` 属性，提供 `getInfo()` 方法返回学生信息。

3. 创建一个 `Temperature` 类，包含 `celsius` 属性，提供 `toFahrenheit()` 和 `toKelvin()` 方法进行温度转换。

4. 实现一个 `StringHelper` 类，提供静态方法 `reverse()`、`upper()`、`lower()` 处理字符串。

5. 创建一个 `ShoppingCart` 类，包含商品列表（数组），提供 `addItem()`、`removeItem()`、`getTotal()` 方法。
