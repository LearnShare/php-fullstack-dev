# 3.1.1 类与对象基础

## 概述

类与对象是面向对象编程的基础。本节详细介绍类定义、对象创建（实例化）、属性定义与访问、方法定义与调用、`$this` 关键字的使用，以及类与对象的关系。

理解类与对象的概念是从过程式编程转向面向对象编程的关键一步。类（Class）是对象的模板，定义了对象应具有的属性和方法；对象（Object）是类的实例，是类的具体实现，拥有独立的属性值。

在 PHP 中，类使用 `class` 关键字定义，对象通过 `new` 关键字创建。类的成员包括属性（Property）和方法（Method）。属性用于存储数据，方法用于定义行为。通过 `$this` 关键字，可以在方法内部访问当前对象的属性和方法。

**主要内容**：
- 类的定义语法和命名规范
- 对象的创建（实例化）过程
- 属性的定义、初始化和访问
- 方法的定义和调用
- `$this` 关键字的作用和使用
- 类与对象的区别和关系
- 完整的代码示例和练习

## 特性

- **类（Class）**：对象的模板，定义属性和方法的蓝图
- **对象（Object）**：类的实例，拥有独立的属性值
- **封装（Encapsulation）**：通过类将数据和行为封装在一起
- **独立性**：每个对象都有独立的属性值，互不干扰
- **复用性**：通过类可以创建多个对象，实现代码复用

## 语法/定义

### 类定义

**语法**：`class ClassName { ... }`

**组成部分**：
- `class` 关键字：用于声明一个类
- `ClassName`：类名，使用 `StudlyCase` 命名规范（首字母大写的驼峰命名）
- 类体 `{ ... }`：包含类的属性和方法

**命名规范**：
- 使用 `StudlyCase` 命名（首字母大写的驼峰命名）
- 类名应该是有意义的名词或名词短语
- 例如：`User`、`Product`、`OrderService`

### 属性定义

**语法**：`public Type $propertyName;`

**组成部分**：
- 可见性修饰符（`public`、`protected`、`private`）：控制属性的访问权限
- 类型声明（可选）：指定属性的数据类型
- `$propertyName`：属性名，使用 `camelCase` 命名规范

**属性初始化**：
- 可以在声明时初始化：`public string $name = "default";`
- 也可以在构造器中初始化（推荐方式）

### 方法定义

**语法**：`public function methodName(Type $param): ReturnType { ... }`

**组成部分**：
- 可见性修饰符：控制方法的访问权限
- `function` 关键字：用于声明方法
- `methodName`：方法名，使用 `camelCase` 命名规范
- 参数列表：方法的参数，支持类型声明和默认值
- 返回值类型（可选）：指定方法的返回类型
- 方法体：包含方法执行的代码

### 对象创建

**语法**：`$object = new ClassName();`

**组成部分**：
- `new` 关键字：用于创建对象实例
- `ClassName()`：调用类的构造函数
- `$object`：变量名，存储创建的对象实例

### 属性访问

**语法**：`$object->propertyName`

**组成部分**：
- `$object`：对象实例
- `->` 操作符：用于访问对象的成员（属性或方法）
- `propertyName`：属性名

### 方法调用

**语法**：`$object->methodName($arg1, $arg2)`

**组成部分**：
- `$object`：对象实例
- `->` 操作符：用于访问对象的成员
- `methodName()`：方法名和参数列表

### $this 关键字

**语法**：`$this->propertyName` 或 `$this->methodName()`

**特点**：
- `$this` 只能在类的实例方法中使用
- `$this` 指向当前对象实例
- 用于访问当前对象的属性和方法
- 不能在静态方法中使用

## 基本用法

### 示例 1：基础类定义和对象创建

```php
<?php
declare(strict_types=1);

// 定义一个简单的 User 类
class User
{
    public string $name;
    public int $age;
    
    public function greet(): string
    {
        return "Hello, {$this->name}!";
    }
    
    public function getAge(): int
    {
        return $this->age;
    }
}

// 创建对象
$user1 = new User();
$user1->name = "John";
$user1->age = 25;

// 访问属性和调用方法
echo $user1->greet() . "\n";      // Hello, John!
echo $user1->getAge() . "\n";     // 25

// 创建另一个对象
$user2 = new User();
$user2->name = "Jane";
$user2->age = 30;

echo $user2->greet() . "\n";      // Hello, Jane!
echo $user2->getAge() . "\n";     // 30
```

**输出**：

```
Hello, John!
25
Hello, Jane!
30
```

**说明**：
- `User` 类定义了 `name` 和 `age` 两个属性
- `greet()` 方法使用 `$this->name` 访问当前对象的 `name` 属性
- `getAge()` 方法返回当前对象的 `age` 属性
- 每个对象都有独立的属性值，互不干扰

### 示例 2：带类型声明的属性

```php
<?php
declare(strict_types=1);

class Product
{
    public string $name;
    public float $price;
    public int $stock;
    
    public function getTotalValue(): float
    {
        return $this->price * $this->stock;
    }
    
    public function displayInfo(): string
    {
        return "Product: {$this->name}, Price: {$this->price}, Stock: {$this->stock}";
    }
}

$product = new Product();
$product->name = "Laptop";
$product->price = 999.99;
$product->stock = 10;

echo $product->displayInfo() . "\n";
echo "Total Value: " . $product->getTotalValue() . "\n";
```

**输出**：

```
Product: Laptop, Price: 999.99, Stock: 10
Total Value: 9999.9
```

**说明**：
- 属性使用类型声明（`string`、`float`、`int`），提供类型安全
- `getTotalValue()` 方法计算总价值
- `displayInfo()` 方法返回产品信息的字符串

### 示例 3：带默认值的属性

```php
<?php
declare(strict_types=1);

class Counter
{
    public int $count = 0;  // 属性初始化
    public string $name = "Counter";  // 带默认值
    
    public function increment(): void
    {
        $this->count++;
    }
    
    public function reset(): void
    {
        $this->count = 0;
    }
    
    public function getStatus(): string
    {
        return "{$this->name}: {$this->count}";
    }
}

$counter = new Counter();
echo $counter->getStatus() . "\n";  // Counter: 0

$counter->increment();
$counter->increment();
echo $counter->getStatus() . "\n";  // Counter: 2

$counter->reset();
echo $counter->getStatus() . "\n";  // Counter: 0
```

**输出**：

```
Counter: 0
Counter: 2
Counter: 0
```

**说明**：
- 属性可以在声明时初始化，提供默认值
- 未初始化的属性在使用前必须赋值
- 对象创建后，属性有默认值可以直接使用

### 示例 4：方法的参数和返回值

```php
<?php
declare(strict_types=1);

class Calculator
{
    public float $result = 0.0;
    
    public function add(float $value): void
    {
        $this->result += $value;
    }
    
    public function subtract(float $value): void
    {
        $this->result -= $value;
    }
    
    public function multiply(float $value): void
    {
        $this->result *= $value;
    }
    
    public function getResult(): float
    {
        return $this->result;
    }
    
    public function reset(): void
    {
        $this->result = 0.0;
    }
}

$calc = new Calculator();
$calc->add(10);
$calc->add(5);
$calc->subtract(3);
$calc->multiply(2);

echo "Result: " . $calc->getResult() . "\n";  // Result: 24

$calc->reset();
echo "After reset: " . $calc->getResult() . "\n";  // After reset: 0
```

**输出**：

```
Result: 24
After reset: 0
```

**说明**：
- 方法可以有参数，参数支持类型声明
- 方法可以有返回值，返回值类型可以声明
- `void` 表示方法没有返回值
- 方法通过 `$this` 访问和修改对象的属性

### 示例 5：$this 关键字的使用

```php
<?php
declare(strict_types=1);

class BankAccount
{
    public string $accountNumber;
    public float $balance = 0.0;
    
    public function deposit(float $amount): void
    {
        $this->balance += $amount;
        $this->logTransaction("Deposit", $amount);
    }
    
    public function withdraw(float $amount): bool
    {
        if ($amount > $this->balance) {
            return false;
        }
        
        $this->balance -= $amount;
        $this->logTransaction("Withdraw", $amount);
        return true;
    }
    
    private function logTransaction(string $type, float $amount): void
    {
        echo "Transaction: {$type} {$amount}, Balance: {$this->balance}\n";
    }
    
    public function getBalance(): float
    {
        return $this->balance;
    }
}

$account = new BankAccount();
$account->accountNumber = "ACC001";

$account->deposit(1000);
$account->deposit(500);

$success = $account->withdraw(300);
if ($success) {
    echo "Withdrawal successful\n";
}

echo "Final Balance: " . $account->getBalance() . "\n";
```

**输出**：

```
Transaction: Deposit 1000, Balance: 1000
Transaction: Deposit 500, Balance: 1500
Transaction: Withdraw 300, Balance: 1200
Withdrawal successful
Final Balance: 1200
```

**说明**：
- `$this` 用于在方法内部访问当前对象的属性和方法
- `deposit()` 方法使用 `$this->balance` 访问和修改余额
- `logTransaction()` 是私有方法，只能通过 `$this` 在类内部调用
- `$this` 指向调用该方法的对象实例

## 使用场景

### 场景 1：数据封装

将相关的数据和行为封装在一起，形成一个独立的单元。

**示例**：用户数据封装

```php
<?php
declare(strict_types=1);

class User
{
    public string $name;
    public string $email;
    public int $age;
    
    public function isAdult(): bool
    {
        return $this->age >= 18;
    }
    
    public function getDisplayName(): string
    {
        return "{$this->name} ({$this->email})";
    }
}

$user = new User();
$user->name = "John Doe";
$user->email = "john@example.com";
$user->age = 25;

echo $user->getDisplayName() . "\n";
echo "Is Adult: " . ($user->isAdult() ? "Yes" : "No") . "\n";
```

### 场景 2：代码复用

通过类可以创建多个对象，避免重复代码。

**示例**：多个产品对象

```php
<?php
declare(strict_types=1);

class Product
{
    public string $name;
    public float $price;
    
    public function getPriceWithTax(float $taxRate = 0.1): float
    {
        return $this->price * (1 + $taxRate);
    }
}

// 创建多个产品
$product1 = new Product();
$product1->name = "Laptop";
$product1->price = 1000;

$product2 = new Product();
$product2->name = "Mouse";
$product2->price = 20;

echo "{$product1->name}: " . $product1->getPriceWithTax() . "\n";
echo "{$product2->name}: " . $product2->getPriceWithTax() . "\n";
```

### 场景 3：模块化设计

将功能模块化，每个类负责一个特定的功能。

**示例**：日志记录模块

```php
<?php
declare(strict_types=1);

class Logger
{
    public string $logFile = "app.log";
    
    public function log(string $message): void
    {
        $timestamp = date('Y-m-d H:i:s');
        $logEntry = "[{$timestamp}] {$message}\n";
        file_put_contents($this->logFile, $logEntry, FILE_APPEND);
    }
    
    public function error(string $message): void
    {
        $this->log("ERROR: {$message}");
    }
    
    public function info(string $message): void
    {
        $this->log("INFO: {$message}");
    }
}

$logger = new Logger();
$logger->info("Application started");
$logger->error("Something went wrong");
```

## 注意事项

### 类名规范

- **使用 StudlyCase 命名**：类名使用首字母大写的驼峰命名（`StudlyCase`）
- **有意义的名字**：类名应该是有意义的名词或名词短语
- **避免缩写**：除非是广泛使用的缩写，否则避免使用缩写

**正确示例**：
- `User`
- `ProductService`
- `OrderRepository`

**错误示例**：
- `usr`（缩写不清晰）
- `product`（未使用大写字母开头）
- `Order_Service`（使用下划线不符合规范）

### 属性初始化

- **必须初始化**：属性在使用前必须初始化，否则会报错或产生警告
- **默认值**：可以在声明时提供默认值
- **构造器初始化**：推荐在构造器中初始化属性（详细说明见 [3.1.3 构造函数与析构函数](section-03-constructors-destructors.md)）

**示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    public string $name = "Default";  // 声明时初始化
    public int $count = 0;             // 带默认值
    
    // 未初始化的属性在使用前必须赋值
    public string $description;  // 需要在构造器或使用前初始化
}
```

### $this 关键字限制

- **只能在实例方法中使用**：`$this` 只能在非静态方法中使用
- **不能在静态方法中使用**：静态方法中没有 `$this`
- **不能在类外部使用**：`$this` 只能在类的方法内部使用

**错误示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    public string $name = "Test";
    
    // 错误：在静态方法中不能使用 $this
    public static function staticMethod(): void
    {
        // echo $this->name;  // 错误：不能使用 $this
    }
}

// 错误：在类外部不能使用 $this
// echo $this->name;  // 错误：未定义变量 $this
```

### 对象与类的区别

- **类（Class）**：是模板或蓝图，定义对象的属性和方法
- **对象（Object）**：是类的实例，是类的具体实现
- **一个类可以有多个对象**：通过 `new` 关键字可以创建多个对象实例
- **每个对象独立**：每个对象都有独立的属性值，互不干扰

**示例**：

```php
<?php
declare(strict_types=1);

class Person
{
    public string $name;
}

// 一个类，多个对象
$person1 = new Person();
$person1->name = "Alice";

$person2 = new Person();
$person2->name = "Bob";

// 每个对象独立
echo $person1->name . "\n";  // Alice
echo $person2->name . "\n";  // Bob
```

### 属性访问权限

- **public 属性**：可以直接访问和修改（本节示例使用 `public`，详细说明见 [3.1.2 可见性修饰符](section-02-visibility.md)）
- **类型安全**：使用类型声明可以提供类型安全
- **访问控制**：使用可见性修饰符控制属性的访问权限

## 常见问题

### 问题 1：$this 未定义错误

**错误信息**：`Fatal error: Using $this when not in object context`

**原因**：
- 在静态方法中使用了 `$this`
- 在类外部使用了 `$this`
- 在函数中使用了 `$this`（不是类的方法）

**解决方案**：
- 只在实例方法中使用 `$this`
- 静态方法应使用 `self::` 访问静态成员
- 确保 `$this` 只在类的方法内部使用

**示例**：

```php
<?php
declare(strict_types=1);

class Example
{
    public string $name = "Test";
    
    // 正确：实例方法中使用 $this
    public function instanceMethod(): void
    {
        echo $this->name . "\n";
    }
    
    // 错误：静态方法中不能使用 $this
    public static function staticMethod(): void
    {
        // echo $this->name;  // 错误：不能使用 $this
    }
}
```

### 问题 2：属性未初始化错误

**错误信息**：`Warning: Undefined property` 或类型错误

**原因**：
- 访问了未初始化的属性
- 属性没有默认值，也没有在使用前赋值

**解决方案**：
- 在声明时提供默认值
- 在构造器中初始化属性
- 在使用前检查属性是否已初始化

**示例**：

```php
<?php
declare(strict_types=1);

class User
{
    // 方案 1：声明时初始化
    public string $name = "";
    
    // 方案 2：使用可空类型
    public ?string $email = null;
    
    // 方案 3：在构造器中初始化（推荐）
    public int $age;
    
    public function __construct(int $age)
    {
        $this->age = $age;
    }
}
```

### 问题 3：类名大小写错误

**错误信息**：`Fatal error: Class 'ClassName' not found`

**原因**：
- 类名大小写不匹配
- PHP 类名是大小写敏感的（虽然 PHP 在某些情况下不区分大小写，但应保持一致）

**解决方案**：
- 使用一致的命名规范（`StudlyCase`）
- 确保类名定义和使用时大小写一致

**示例**：

```php
<?php
declare(strict_types=1);

// 定义类
class User
{
    // ...
}

// 正确：大小写一致
$user = new User();

// 错误：大小写不一致（虽然可能不会报错，但不推荐）
// $user = new user();  // 不推荐
```

### 问题 4：对象属性访问权限错误

**错误信息**：`Fatal error: Cannot access private/protected property`

**原因**：
- 试图访问 `private` 或 `protected` 属性（详细说明见 [3.1.2 可见性修饰符](section-02-visibility.md)）

**解决方案**：
- 使用 `public` 属性（如果允许外部访问）
- 使用 getter/setter 方法访问私有属性
- 理解可见性修饰符的作用

**示例**：

```php
<?php
declare(strict_types=1);

class User
{
    private string $email;  // 私有属性
    
    public function setEmail(string $email): void
    {
        $this->email = $email;
    }
    
    public function getEmail(): string
    {
        return $this->email;
    }
}

$user = new User();
// $user->email = "test@example.com";  // 错误：不能访问私有属性
$user->setEmail("test@example.com");   // 正确：使用 setter 方法
echo $user->getEmail() . "\n";         // 正确：使用 getter 方法
```

## 最佳实践

### 1. 使用有意义的类名

类名应该清晰表达类的用途，使用名词或名词短语。

**好的示例**：
- `User`：用户
- `OrderProcessor`：订单处理器
- `DatabaseConnection`：数据库连接

**不好的示例**：
- `Data`：太泛泛
- `Handler`：不够具体
- `Utils`：太宽泛

### 2. 合理设计类的属性和方法

- **单一职责**：每个类应该有一个明确的职责
- **高内聚**：相关的方法和属性应该组织在一起
- **低耦合**：类之间的依赖应该最小化

**示例**：

```php
<?php
declare(strict_types=1);

// 好的设计：职责清晰
class EmailValidator
{
    public function validate(string $email): bool
    {
        return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
    }
}

// 不好的设计：职责混乱
class UserManager
{
    // 包含太多不相关的功能
    public function validateEmail(string $email): bool { }
    public function sendEmail(string $email, string $message): void { }
    public function saveToDatabase(array $data): void { }
    public function generateReport(): void { }
}
```

### 3. 使用类型声明

为属性和方法参数添加类型声明，提高代码质量和可维护性。

**示例**：

```php
<?php
declare(strict_types=1);

class Calculator
{
    public float $result = 0.0;  // 类型声明
    
    public function add(float $value): void  // 参数和返回值类型
    {
        $this->result += $value;
    }
    
    public function getResult(): float  // 返回值类型
    {
        return $this->result;
    }
}
```

### 4. 遵循 PSR 命名规范

- **类名**：使用 `StudlyCase`（如：`UserService`）
- **方法名和属性名**：使用 `camelCase`（如：`getUserName`、`$userName`）
- **常量名**：使用 `UPPER_SNAKE_CASE`（如：`MAX_SIZE`）

**示例**：

```php
<?php
declare(strict_types=1);

class UserService
{
    private const MAX_ATTEMPTS = 3;
    
    private string $userName;
    
    public function getUserName(): string
    {
        return $this->userName;
    }
}
```

### 5. 理解类与对象的关系

- 类是模板，对象是实例
- 一个类可以创建多个对象
- 每个对象都有独立的属性值
- 类的方法被所有对象共享

**示例**：

```php
<?php
declare(strict_types=1);

class Counter
{
    public int $count = 0;
    
    public function increment(): void
    {
        $this->count++;
    }
}

// 创建多个对象，每个对象独立计数
$counter1 = new Counter();
$counter2 = new Counter();

$counter1->increment();
$counter1->increment();

$counter2->increment();

echo $counter1->count . "\n";  // 2
echo $counter2->count . "\n";  // 1
```

## 对比分析

### 类 vs 对象

| 特性         | 类（Class）                      | 对象（Object）                   |
|:-------------|:--------------------------------|:--------------------------------|
| **定义**      | 对象的模板或蓝图                  | 类的实例                        |
| **数量**      | 一个类定义                       | 可以创建多个对象实例              |
| **存储**      | 定义在代码中                     | 在内存中创建                     |
| **关系**      | 类是对象的定义                   | 对象是类的实现                   |
| **使用**      | 用于创建对象                     | 用于存储数据和执行方法            |

### 属性 vs 方法

| 特性         | 属性（Property）                 | 方法（Method）                   |
|:-------------|:--------------------------------|:--------------------------------|
| **作用**      | 存储数据                         | 定义行为                        |
| **访问**      | 直接访问或通过方法访问            | 通过调用执行                     |
| **语法**      | `$object->property`              | `$object->method()`              |
| **类型**      | 可以是各种数据类型                | 函数，可以有参数和返回值          |
| **状态**      | 存储对象的状态                   | 操作对象的状态                   |

### 过程式 vs 面向对象

| 特性         | 过程式编程                       | 面向对象编程                     |
|:-------------|:--------------------------------|:--------------------------------|
| **组织方式**  | 函数和数据分离                   | 数据和方法封装在类中              |
| **数据访问**  | 通过参数传递                     | 通过 `$this` 访问对象属性        |
| **代码复用**  | 函数复用                         | 类和对象复用                     |
| **状态管理**  | 全局变量或参数传递                | 对象属性存储状态                 |

**过程式示例**：

```php
<?php
declare(strict_types=1);

// 过程式：数据和函数分离
$userName = "John";
$userAge = 25;

function greet(string $name): string
{
    return "Hello, {$name}!";
}

function getAge(int $age): int
{
    return $age;
}

echo greet($userName) . "\n";
echo getAge($userAge) . "\n";
```

**面向对象示例**：

```php
<?php
declare(strict_types=1);

// 面向对象：数据和方法封装
class User
{
    public string $name;
    public int $age;
    
    public function greet(): string
    {
        return "Hello, {$this->name}!";
    }
    
    public function getAge(): int
    {
        return $this->age;
    }
}

$user = new User();
$user->name = "John";
$user->age = 25;

echo $user->greet() . "\n";
echo $user->getAge() . "\n";
```

## 练习任务

1. **创建图书类**：定义一个 `Book` 类，包含 `title`（书名）、`author`（作者）、`price`（价格）属性，以及 `getInfo()` 方法返回图书信息。

2. **创建学生类**：定义一个 `Student` 类，包含 `name`（姓名）、`age`（年龄）、`grade`（成绩）属性，以及 `isPass()` 方法判断是否及格（成绩 >= 60）。

3. **创建计算器类**：定义一个 `Calculator` 类，包含 `result` 属性（初始值为 0），以及 `add()`、`subtract()`、`multiply()`、`divide()` 方法进行四则运算，最后添加 `getResult()` 方法返回结果。

4. **创建银行账户类**：定义一个 `BankAccount` 类，包含 `accountNumber`（账号）和 `balance`（余额）属性，以及 `deposit()`（存款）、`withdraw()`（取款）、`getBalance()`（查询余额）方法。

5. **创建多个对象**：使用上述任意一个类，创建至少 3 个不同的对象实例，并调用它们的方法，观察每个对象的独立性。

## 相关章节

- **[3.1.2 可见性修饰符](section-02-visibility.md)**：了解如何控制类成员的访问权限
- **[3.1.3 构造函数与析构函数](section-03-constructors-destructors.md)**：了解如何初始化对象和清理资源
- **[2.10 函数与作用域](../../stage-02-language/chapter-10-functions/readme.md)**：回顾函数的基础知识
