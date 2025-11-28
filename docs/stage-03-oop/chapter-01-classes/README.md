# 3.1 类、对象与基础 OOP

## 目标

- 理解类（Class）与对象（Object）的基本概念，掌握如何定义类、创建对象、访问属性与方法。
- 熟悉可见性修饰符（`public`、`protected`、`private`）的作用与使用场景。
- 掌握构造函数（`__construct`）、析构函数（`__destruct`）与魔术方法的基础用法。
- 理解 `$this` 关键字、对象克隆（`clone`）与对象引用的区别。

## 类与对象基础

### 类定义

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

### 对象创建

- **语法**：`$object = new ClassName();`
- 使用 `new` 关键字实例化类，创建对象。
- 对象是类的实例，每个对象拥有独立的属性值。

```php
$user = new User();
$anotherUser = new User();
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
}
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

### `$this` 关键字

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

    public function getResult(): float
    {
        return $this->result;
    }
}

$calc = new Calculator();
$calc->add(10)->add(20);
echo $calc->getResult(); // 30
```

## 可见性修饰符

| 修饰符     | 说明                                           | 访问范围                           |
| :--------- | :--------------------------------------------- | :--------------------------------- |
| `public`   | 公开访问                                       | 类内、子类、外部均可访问           |
| `protected` | 受保护访问                                     | 类内、子类可访问，外部不可访问     |
| `private`  | 私有访问                                       | 仅类内可访问，子类与外部均不可访问 |

### 可见性最佳实践

- 属性通常设为 `private` 或 `protected`，通过公开方法（getter/setter）控制访问。
- 方法根据业务需求选择可见性，避免过度暴露内部实现。

```php
class BankAccount
{
    private float $balance = 0.0;

    public function deposit(float $amount): void
    {
        if ($amount <= 0) {
            throw new InvalidArgumentException('Amount must be positive');
        }
        $this->balance += $amount;
    }

    public function withdraw(float $amount): void
    {
        if ($amount > $this->balance) {
            throw new RuntimeException('Insufficient funds');
        }
        $this->balance -= $amount;
    }

    public function getBalance(): float
    {
        return $this->balance;
    }
}
```

## 构造函数（Constructor）

### `__construct` 方法

- **语法**：`public function __construct(参数列表) { ... }`
- 对象创建时自动调用，用于初始化对象属性。
- 支持参数类型声明与默认值。

```php
class User
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email
    ) {
        // 构造函数体（可选）
    }
}

// PHP 8.0+ 构造器属性提升，上述代码等价于：
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
```

### 构造函数示例

```php
class Product
{
    private string $name;
    private float $price;
    private int $stock;

    public function __construct(string $name, float $price, int $stock = 0)
    {
        $this->name = $name;
        $this->price = max(0, $price);
        $this->stock = max(0, $stock);
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function getPrice(): float
    {
        return $this->price;
    }

    public function getStock(): int
    {
        return $this->stock;
    }
}

$product = new Product('Laptop', 999.99, 10);
```

## 析构函数（Destructor）

### `__destruct` 方法

- **语法**：`public function __destruct() { ... }`
- 对象被销毁时自动调用（引用计数为 0 或脚本结束时）。
- 常用于释放资源（关闭文件、断开连接等）。

```php
class FileHandler
{
    private $handle;

    public function __construct(string $filename)
    {
        $this->handle = fopen($filename, 'r');
        if ($this->handle === false) {
            throw new RuntimeException("Cannot open file: {$filename}");
        }
    }

    public function __destruct()
    {
        if (is_resource($this->handle)) {
            fclose($this->handle);
        }
    }

    public function read(): string
    {
        return fread($this->handle, 8192);
    }
}
```

## 对象克隆（Clone）

### `clone` 关键字

- **语法**：`$clone = clone $object;`
- 创建对象的浅拷贝（shallow copy），属性值被复制，但对象引用仍指向同一对象。
- 如需深拷贝，需实现 `__clone()` 方法。

```php
class Point
{
    public function __construct(
        public int $x,
        public int $y
    ) {
    }
}

$point1 = new Point(10, 20);
$point2 = clone $point1;
$point2->x = 30;

echo $point1->x; // 10
echo $point2->x; // 30
```

### `__clone` 方法

- 在 `clone` 操作时自动调用，可用于自定义克隆逻辑。

```php
class User
{
    public function __construct(
        public int $id,
        public string $name
    ) {
    }

    public function __clone()
    {
        $this->id = 0; // 克隆时重置 ID
    }
}

$user1 = new User(1, 'Alice');
$user2 = clone $user1;
echo $user2->id; // 0
```

## 对象引用 vs 值传递

- 对象赋值是引用传递，多个变量指向同一对象。
- 使用 `clone` 创建独立副本。

```php
$user1 = new User(1, 'Alice');
$user2 = $user1; // $user2 与 $user1 指向同一对象
$user2->name = 'Bob';
echo $user1->name; // Bob

$user3 = clone $user1; // $user3 是独立副本
$user3->name = 'Charlie';
echo $user1->name; // Bob（未改变）
```

### 克隆时更新属性（Clone with）（PHP 8.5+）

- **语法**：`clone $object with ['property' => 'value']`
- 在克隆对象的同时更新其指定属性，简化不可变对象的更新。

```php
<?php
declare(strict_types=1);

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

// 等价于传统写法（需要重新构造）
$updatedUser = new User($user->name, 31, 'alice.new@example.com');
```

### 实际应用示例

```php
<?php
declare(strict_types=1);

readonly class Point
{
    public function __construct(
        public int $x,
        public int $y
    ) {
    }
    
    // 使用 clone with 创建新点
    public function move(int $dx, int $dy): self
    {
        return clone $this with [
            'x' => $this->x + $dx,
            'y' => $this->y + $dy,
        ];
    }
}

$point = new Point(10, 20);
$newPoint = $point->move(5, -5); // Point(15, 15)
```

## 常用魔术方法速览

| 方法名          | 触发时机                     | 用途示例                           |
| :-------------- | :--------------------------- | :--------------------------------- |
| `__construct`   | 对象创建时                   | 初始化属性                         |
| `__destruct`    | 对象销毁时                   | 释放资源                           |
| `__clone`       | `clone` 操作时                | 自定义克隆逻辑                     |
| `__toString`    | 对象被转换为字符串时         | 返回对象的字符串表示               |
| `__get`         | 访问不存在的属性时           | 动态属性访问                       |
| `__set`         | 设置不存在的属性时           | 动态属性设置                       |
| `__call`        | 调用不存在的方法时           | 动态方法调用                       |
| `__invoke`      | 对象被当作函数调用时         | 使对象可调用                       |

### `__toString` 示例

```php
class User
{
    public function __construct(
        public int $id,
        public string $name
    ) {
    }

    public function __toString(): string
    {
        return "User #{$this->id}: {$this->name}";
    }
}

$user = new User(1, 'Alice');
echo $user; // User #1: Alice
```

## 静态属性与方法

### 静态属性

- **语法**：`public static $property;`
- 属于类而非对象，所有实例共享同一静态属性。
- 通过 `ClassName::$property` 访问。

```php
class Counter
{
    public static int $count = 0;

    public function __construct()
    {
        self::$count++;
    }

    public static function getCount(): int
    {
        return self::$count;
    }
}

new Counter();
new Counter();
echo Counter::$count; // 2
echo Counter::getCount(); // 2
```

### 静态方法

- **语法**：`public static function methodName(): 返回类型 { ... }`
- 通过 `ClassName::methodName()` 调用，无需实例化。
- 静态方法内部不能使用 `$this`，只能访问静态属性或调用静态方法。

```php
class Math
{
    public static function add(float $a, float $b): float
    {
        return $a + $b;
    }

    public static function multiply(float $a, float $b): float
    {
        return $a * $b;
    }
}

echo Math::add(5, 3); // 8
echo Math::multiply(4, 7); // 28
```

## 对象比较

- `==`：比较对象属性值是否相等。
- `===`：比较是否为同一对象实例。

```php
$user1 = new User(1, 'Alice');
$user2 = new User(1, 'Alice');
$user3 = $user1;

var_dump($user1 == $user2);  // true（属性值相同）
var_dump($user1 === $user2); // false（不同实例）
var_dump($user1 === $user3); // true（同一实例）
```

## 练习

1. 创建一个 `Rectangle` 类，包含 `width` 和 `height` 属性，提供 `getArea()` 和 `getPerimeter()` 方法，使用构造函数初始化属性。

2. 实现一个 `BankAccount` 类，包含私有属性 `balance`，提供 `deposit()`、`withdraw()`、`getBalance()` 方法，并添加余额验证逻辑。

3. 创建一个 `Logger` 类，使用静态属性记录日志数量，提供静态方法 `log()` 和 `getLogCount()`，并实现 `__toString()` 方法返回日志摘要。

4. 编写 `ShoppingCart` 类，包含商品列表（数组），提供 `addItem()`、`removeItem()`、`getTotal()` 方法，使用 `__clone()` 确保克隆时创建商品列表的深拷贝。

5. 实现一个 `Config` 类，使用 `__get()` 和 `__set()` 魔术方法实现动态配置访问，支持 `$config->database->host` 这样的嵌套访问。
