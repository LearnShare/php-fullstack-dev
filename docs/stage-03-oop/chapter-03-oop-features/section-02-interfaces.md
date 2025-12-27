# 3.3.2 接口（Interface）

## 概述

接口（Interface）定义了类的契约，规定实现该接口的类必须实现哪些方法。接口是面向对象编程中实现多态和代码解耦的重要机制。接口只定义方法签名，不提供实现，实现接口的类必须提供所有方法的具体实现。

理解接口对于掌握面向对象编程至关重要。接口提供了一种定义契约的方式，实现了"约定优于配置"的设计思想。通过接口，可以定义类必须实现的功能，而不关心具体的实现细节。这使得代码更加灵活，更容易测试和扩展。

与继承不同，PHP 支持多接口实现，一个类可以实现多个接口，这在一定程度上弥补了 PHP 单继承的限制。接口还可以作为类型提示，实现依赖注入和多态设计。

**主要内容**：
- 接口的定义语法
- 接口的实现（`implements` 关键字）
- 接口方法的可见性规则
- 实现多个接口
- 接口继承（接口扩展接口）
- 接口作为类型提示
- 接口在多态中的应用
- 接口与抽象类的区别
- 接口的最佳实践

## 特性

- **契约定义**：定义类必须实现的方法签名
- **多实现**：一个类可以实现多个接口
- **类型安全**：接口可以作为类型提示，提供编译时类型检查
- **多态基础**：接口是实现多态的重要机制
- **代码解耦**：通过接口解耦依赖关系，提高代码灵活性

## 语法/定义

### 接口定义

**语法**：`interface InterfaceName { public function method(): ReturnType; }`

**组成部分**：
- `interface` 关键字：声明接口
- `InterfaceName`：接口名称，遵循类命名规范（`StudlyCase`）
- 方法签名：只定义方法签名，不包含实现体

**特点**：
- 接口只能包含方法签名，不能包含方法实现
- 接口方法必须是 `public`（不能使用其他可见性修饰符）
- 接口不能包含属性（可以有常量）
- 接口可以包含常量（PHP 8.0+）

### 接口实现

**语法**：`class ClassName implements InterfaceName { ... }`

**组成部分**：
- `class` 关键字：声明类
- `ClassName`：类名称
- `implements` 关键字：表示实现接口
- `InterfaceName`：接口名称

**特点**：
- 使用 `implements` 关键字实现接口
- 实现接口的类必须实现接口中定义的所有方法
- 实现的方法签名必须与接口定义完全匹配

### 多接口实现

**语法**：`class ClassName implements Interface1, Interface2, ... { ... }`

**特点**：
- 一个类可以实现多个接口
- 用逗号分隔多个接口
- 必须实现所有接口中定义的方法

## 基本用法

### 示例 1：基础接口定义和实现

```php
<?php
declare(strict_types=1);

interface Logger
{
    public function log(string $message, string $level = 'info'): void;
}

class FileLogger implements Logger
{
    public function log(string $message, string $level = 'info'): void
    {
        $timestamp = date('Y-m-d H:i:s');
        $logLine = "[{$timestamp}] [{$level}] {$message}\n";
        file_put_contents('app.log', $logLine, FILE_APPEND);
    }
}

class DatabaseLogger implements Logger
{
    public function log(string $message, string $level = 'info'): void
    {
        // 模拟数据库写入
        echo "Database log: [{$level}] {$message}\n";
    }
}

// 使用接口
$fileLogger = new FileLogger();
$fileLogger->log("Application started");

$dbLogger = new DatabaseLogger();
$dbLogger->log("User logged in", "info");
```

**输出**：

```
Database log: [info] User logged in
```

**说明**：
- `Logger` 接口定义了 `log()` 方法的签名
- `FileLogger` 和 `DatabaseLogger` 都实现了 `Logger` 接口
- 每个实现类提供了自己的具体实现

### 示例 2：接口作为类型提示

```php
<?php
declare(strict_types=1);

interface PaymentMethod
{
    public function pay(float $amount): bool;
    public function refund(string $transactionId): bool;
}

class CreditCardPayment implements PaymentMethod
{
    public function pay(float $amount): bool
    {
        echo "Processing credit card payment: \${$amount}\n";
        // 实际的支付逻辑
        return true;
    }
    
    public function refund(string $transactionId): bool
    {
        echo "Refunding transaction: {$transactionId}\n";
        return true;
    }
}

class PayPalPayment implements PaymentMethod
{
    public function pay(float $amount): bool
    {
        echo "Processing PayPal payment: \${$amount}\n";
        return true;
    }
    
    public function refund(string $transactionId): bool
    {
        echo "Refunding PayPal transaction: {$transactionId}\n";
        return true;
    }
}

// 使用接口作为类型提示
function processPayment(PaymentMethod $method, float $amount): void
{
    $method->pay($amount);
}

$creditCard = new CreditCardPayment();
$paypal = new PayPalPayment();

processPayment($creditCard, 100.0);  // Processing credit card payment: $100
processPayment($paypal, 50.0);       // Processing PayPal payment: $50
```

**输出**：

```
Processing credit card payment: $100
Processing PayPal payment: $50
```

**说明**：
- `processPayment()` 函数接受 `PaymentMethod` 接口类型
- 任何实现 `PaymentMethod` 接口的类都可以传递给该函数
- 实现了多态设计

### 示例 3：实现多个接口

```php
<?php
declare(strict_types=1);

interface Readable
{
    public function read(): string;
}

interface Writable
{
    public function write(string $data): void;
}

interface Serializable
{
    public function serialize(): string;
}

// 一个类可以实现多个接口
class FileHandler implements Readable, Writable, Serializable
{
    private string $content = "";
    
    public function read(): string
    {
        return $this->content;
    }
    
    public function write(string $data): void
    {
        $this->content = $data;
    }
    
    public function serialize(): string
    {
        return json_encode(['content' => $this->content]);
    }
}

$handler = new FileHandler();
$handler->write("Hello, World!");
echo "Content: " . $handler->read() . "\n";
echo "Serialized: " . $handler->serialize() . "\n";
```

**输出**：

```
Content: Hello, World!
Serialized: {"content":"Hello, World!"}
```

**说明**：
- `FileHandler` 类实现了三个接口：`Readable`、`Writable`、`Serializable`
- 必须实现所有接口中定义的方法
- 通过多接口实现，类可以满足多个契约

### 示例 4：接口继承（接口扩展接口）

```php
<?php
declare(strict_types=1);

interface Readable
{
    public function read(): string;
}

interface Writable
{
    public function write(string $data): void;
}

// 接口可以继承其他接口
interface ReadWritable extends Readable, Writable
{
    // 继承了 Readable 和 Writable 的所有方法
    // 可以添加新方法
    public function clear(): void;
}

class File implements ReadWritable
{
    private string $content = "";
    
    public function read(): string
    {
        return $this->content;
    }
    
    public function write(string $data): void
    {
        $this->content = $data;
    }
    
    public function clear(): void
    {
        $this->content = "";
    }
}

$file = new File();
$file->write("Test content");
echo "Content: " . $file->read() . "\n";
$file->clear();
echo "After clear: " . $file->read() . "\n";
```

**输出**：

```
Content: Test content
After clear: 
```

**说明**：
- `ReadWritable` 接口继承了 `Readable` 和 `Writable` 接口
- 接口可以继承多个接口
- 实现 `ReadWritable` 接口的类必须实现所有继承的方法

### 示例 5：接口常量（PHP 8.0+）

```php
<?php
declare(strict_types=1);

interface PaymentStatus
{
    public const STATUS_PENDING = 'pending';
    public const STATUS_COMPLETED = 'completed';
    public const STATUS_FAILED = 'failed';
    public const STATUS_REFUNDED = 'refunded';
    
    public function getStatus(): string;
}

class Payment implements PaymentStatus
{
    private string $status;
    
    public function __construct()
    {
        $this->status = self::STATUS_PENDING;
    }
    
    public function getStatus(): string
    {
        return $this->status;
    }
    
    public function markCompleted(): void
    {
        $this->status = self::STATUS_COMPLETED;
    }
}

$payment = new Payment();
echo "Initial status: " . $payment->getStatus() . "\n";  // pending
echo "Constant value: " . PaymentStatus::STATUS_COMPLETED . "\n";  // completed

$payment->markCompleted();
echo "After completion: " . $payment->getStatus() . "\n";  // completed
```

**输出**：

```
Initial status: pending
Constant value: completed
After completion: completed
```

**说明**：
- 接口可以定义常量（PHP 8.0+）
- 常量通过 `InterfaceName::CONSTANT` 或 `self::CONSTANT` 访问
- 实现接口的类可以使用这些常量

### 示例 6：接口在依赖注入中的应用

```php
<?php
declare(strict_types=1);

interface NotificationService
{
    public function send(string $to, string $message): bool;
}

class EmailService implements NotificationService
{
    public function send(string $to, string $message): bool
    {
        echo "Sending email to {$to}: {$message}\n";
        return true;
    }
}

class SmsService implements NotificationService
{
    public function send(string $to, string $message): bool
    {
        echo "Sending SMS to {$to}: {$message}\n";
        return true;
    }
}

// 使用接口进行依赖注入
class OrderService
{
    public function __construct(
        private NotificationService $notifier
    ) {}
    
    public function createOrder(string $userId, float $amount): void
    {
        // 创建订单逻辑
        echo "Order created for user {$userId}, amount: \${$amount}\n";
        
        // 使用注入的通知服务
        $this->notifier->send($userId, "Your order has been created");
    }
}

// 可以注入不同的实现
$emailService = new EmailService();
$orderService1 = new OrderService($emailService);
$orderService1->createOrder("user123", 100.0);

$smsService = new SmsService();
$orderService2 = new OrderService($smsService);
$orderService2->createOrder("user456", 200.0);
```

**输出**：

```
Order created for user user123, amount: $100
Sending email to user123: Your order has been created
Order created for user user456, amount: $200
Sending SMS to user456: Your order has been created
```

**说明**：
- `OrderService` 依赖于 `NotificationService` 接口，而不是具体实现
- 可以注入不同的实现（EmailService 或 SmsService）
- 提高了代码的灵活性和可测试性

### 示例 7：接口多态示例

```php
<?php
declare(strict_types=1);

interface Shape
{
    public function getArea(): float;
    public function getPerimeter(): float;
}

class Rectangle implements Shape
{
    public function __construct(
        private float $width,
        private float $height
    ) {}
    
    public function getArea(): float
    {
        return $this->width * $this->height;
    }
    
    public function getPerimeter(): float
    {
        return 2 * ($this->width + $this->height);
    }
}

class Circle implements Shape
{
    public function __construct(
        private float $radius
    ) {}
    
    public function getArea(): float
    {
        return pi() * $this->radius * $this->radius;
    }
    
    public function getPerimeter(): float
    {
        return 2 * pi() * $this->radius;
    }
}

// 多态：可以处理任何实现 Shape 接口的对象
function printShapeInfo(Shape $shape): void
{
    echo "Area: " . $shape->getArea() . "\n";
    echo "Perimeter: " . $shape->getPerimeter() . "\n";
}

$rectangle = new Rectangle(5, 3);
$circle = new Circle(4);

printShapeInfo($rectangle);  // 处理矩形
printShapeInfo($circle);     // 处理圆形
```

**输出**：

```
Area: 15
Perimeter: 16
Area: 50.265482457437
Perimeter: 25.132741228718
```

**说明**：
- `printShapeInfo()` 函数接受 `Shape` 接口类型
- 可以处理任何实现 `Shape` 接口的对象
- 实现了多态设计，代码更加灵活

## 使用场景

### 场景 1：定义服务契约

接口用于定义服务的契约，不同的实现可以替换使用。

**示例**：缓存服务接口

```php
<?php
declare(strict_types=1);

interface Cache
{
    public function get(string $key): mixed;
    public function set(string $key, mixed $value, int $ttl = 3600): void;
    public function delete(string $key): void;
    public function clear(): void;
}

class FileCache implements Cache
{
    private string $cacheDir;
    
    public function __construct(string $cacheDir = '/tmp/cache')
    {
        $this->cacheDir = $cacheDir;
    }
    
    public function get(string $key): mixed
    {
        $file = $this->getFilePath($key);
        if (!file_exists($file)) {
            return null;
        }
        $data = unserialize(file_get_contents($file));
        if ($data['expires'] < time()) {
            unlink($file);
            return null;
        }
        return $data['value'];
    }
    
    public function set(string $key, mixed $value, int $ttl = 3600): void
    {
        $file = $this->getFilePath($key);
        $data = [
            'value' => $value,
            'expires' => time() + $ttl
        ];
        file_put_contents($file, serialize($data));
    }
    
    public function delete(string $key): void
    {
        $file = $this->getFilePath($key);
        if (file_exists($file)) {
            unlink($file);
        }
    }
    
    public function clear(): void
    {
        $files = glob($this->cacheDir . '/*');
        foreach ($files as $file) {
            unlink($file);
        }
    }
    
    private function getFilePath(string $key): string
    {
        return $this->cacheDir . '/' . md5($key);
    }
}
```

### 场景 2：依赖注入

接口用于依赖注入，提高代码的可测试性和灵活性。

**示例**：仓库模式

```php
<?php
declare(strict_types=1);

interface UserRepository
{
    public function findById(int $id): ?User;
    public function save(User $user): void;
    public function delete(int $id): void;
}

class User
{
    public function __construct(
        public int $id,
        public string $name
    ) {}
}

class DatabaseUserRepository implements UserRepository
{
    public function findById(int $id): ?User
    {
        // 从数据库查询
        return new User($id, "User {$id}");
    }
    
    public function save(User $user): void
    {
        // 保存到数据库
    }
    
    public function delete(int $id): void
    {
        // 从数据库删除
    }
}

class UserService
{
    public function __construct(
        private UserRepository $repository
    ) {}
    
    public function getUser(int $id): ?User
    {
        return $this->repository->findById($id);
    }
}
```

## 注意事项

### 接口方法必须是 public

- 接口中定义的方法必须是 `public`
- 不能使用 `protected` 或 `private`
- 实现类中的方法也必须是 `public`

**示例**：

```php
<?php
declare(strict_types=1);

interface Example
{
    // 正确：public 方法
    public function publicMethod(): void;
    
    // 错误：不能使用 protected 或 private
    // protected function protectedMethod(): void;
    // private function privateMethod(): void;
}
```

### 必须实现所有方法

- 实现接口的类必须实现接口中定义的所有方法
- 缺少任何方法都会导致致命错误

**示例**：

```php
<?php
declare(strict_types=1);

interface Example
{
    public function method1(): void;
    public function method2(): void;
}

// 错误：未实现所有方法
// class Incomplete implements Example
// {
//     public function method1(): void {}  // 缺少 method2
// }

// 正确：实现所有方法
class Complete implements Example
{
    public function method1(): void {}
    public function method2(): void {}
}
```

### 方法签名必须匹配

- 实现的方法签名必须与接口定义完全匹配
- 包括方法名、参数类型、返回类型

**示例**：

```php
<?php
declare(strict_types=1);

interface Example
{
    public function process(string $data): bool;
}

// 错误：方法签名不匹配
// class Wrong implements Example
// {
//     public function process($data): void {}  // 参数类型和返回类型不匹配
// }

// 正确：方法签名匹配
class Correct implements Example
{
    public function process(string $data): bool
    {
        return true;
    }
}
```

## 常见问题

### 问题 1：未实现所有接口方法

**错误信息**：`Fatal error: Class X contains Y abstract method(s) and must therefore be declared abstract`

**原因**：实现接口的类未实现所有接口方法

**解决方案**：实现接口中定义的所有方法

**示例**：

```php
<?php
declare(strict_types=1);

interface Example
{
    public function method1(): void;
    public function method2(): void;
}

// 错误：未实现所有方法
// class Incomplete implements Example
// {
//     public function method1(): void {}
// }

// 正确：实现所有方法
class Complete implements Example
{
    public function method1(): void {}
    public function method2(): void {}
}
```

### 问题 2：方法可见性错误

**错误信息**：`Fatal error: Access type for interface method X::method() must be omitted`

**原因**：接口方法不能指定可见性（默认为 public）

**解决方案**：接口方法不指定可见性，实现类中使用 `public`

**示例**：

```php
<?php
declare(strict_types=1);

// 正确：接口方法不指定可见性
interface Example
{
    public function method(): void;
}

class Implementation implements Example
{
    public function method(): void {}  // 必须是 public
}
```

### 问题 3：方法签名不匹配

**错误信息**：`Declaration must be compatible with interface`

**原因**：实现的方法签名与接口定义不匹配

**解决方案**：确保方法签名完全匹配（包括参数类型和返回类型）

**示例**：

```php
<?php
declare(strict_types=1);

interface Example
{
    public function process(string $data): bool;
}

// 错误：签名不匹配
// class Wrong implements Example
// {
//     public function process($data): void {}  // 参数类型和返回类型不匹配
// }

// 正确：签名匹配
class Correct implements Example
{
    public function process(string $data): bool
    {
        return true;
    }
}
```

## 最佳实践

### 1. 使用接口定义契约

接口应该定义清晰的契约，描述类必须实现的功能。

**示例**：

```php
<?php
declare(strict_types=1);

// 好的接口设计：职责单一，命名清晰
interface CacheInterface
{
    public function get(string $key): mixed;
    public function set(string $key, mixed $value, int $ttl = 3600): void;
    public function delete(string $key): void;
}

// 不好的接口设计：职责过多
// interface BadInterface
// {
//     public function cache(): void;
//     public function log(): void;
//     public function validate(): void;  // 职责混杂
// }
```

### 2. 接口粒度设计

接口应该保持适当的粒度，不要过于庞大，也不要过于琐碎。

**示例**：

```php
<?php
declare(strict_types=1);

// 好的设计：职责单一
interface Readable
{
    public function read(): string;
}

interface Writable
{
    public function write(string $data): void;
}

// 需要两者时，使用接口继承
interface ReadWritable extends Readable, Writable {}
```

### 3. 接口命名规范

- 接口名称应该清晰描述其功能
- 可以使用 `Interface` 后缀（可选）
- 使用 `StudlyCase` 命名

**示例**：

```php
<?php
declare(strict_types=1);

// 好的命名
interface Logger {}
interface PaymentProcessor {}
interface UserRepository {}

// 也可以使用 Interface 后缀
interface LoggerInterface {}
```

### 4. 使用接口进行依赖注入

接口是依赖注入的理想选择，提高代码的可测试性和灵活性。

**示例**：

```php
<?php
declare(strict_types=1);

interface EmailService
{
    public function send(string $to, string $subject, string $body): bool;
}

class OrderService
{
    public function __construct(
        private EmailService $emailService  // 依赖接口而不是具体类
    ) {}
}
```

### 5. 接口 vs 抽象类

根据需求选择接口或抽象类：
- **接口**：定义契约，不需要共享代码
- **抽象类**：需要提供部分实现，共享代码

**示例**：

```php
<?php
declare(strict_types=1);

// 接口：只定义契约
interface Loggable
{
    public function log(string $message): void;
}

// 抽象类：提供部分实现
abstract class AbstractLogger
{
    protected function formatMessage(string $message): string
    {
        return date('Y-m-d H:i:s') . " - {$message}";
    }
    
    abstract public function write(string $message): void;
}
```

## 对比分析

### 接口 vs 抽象类

| 特性         | 接口（Interface）                 | 抽象类（Abstract Class）          |
|:-------------|:--------------------------------|:--------------------------------|
| **实现**      | 只定义方法签名                    | 可以提供部分实现                  |
| **数量**      | 可以实现多个接口                  | 只能继承一个抽象类                |
| **属性**      | 不能有属性（可以有常量）           | 可以有属性                        |
| **可见性**    | 方法必须是 public                | 方法可以有不同可见性              |
| **构造函数**  | 不能有构造函数                    | 可以有构造函数                    |
| **适用场景**  | 定义契约，多态设计                | 提供部分实现，代码复用            |

### 接口 vs 继承

| 特性         | 接口                               | 继承                               |
|:-------------|:-----------------------------------|:-----------------------------------|
| **关系**      | can-do（能做什么）                  | is-a（是什么）                      |
| **数量**      | 可以实现多个接口                    | 只能继承一个父类                    |
| **实现**      | 只定义契约                          | 提供实现                            |
| **耦合度**    | 低耦合                              | 高耦合                              |
| **适用场景**  | 定义能力、行为                      | 代码复用、层次化设计                |

## 练习任务

1. **定义缓存接口**：创建一个 `Cache` 接口，包含 `get()`、`set()`、`delete()` 方法，然后实现 `FileCache` 和 `MemoryCache` 类。

2. **多接口实现**：创建 `Readable` 和 `Writable` 接口，创建一个类同时实现这两个接口。

3. **接口继承**：创建 `Readable` 接口，然后创建 `ReadWritable` 接口继承 `Readable` 并添加 `write()` 方法。

4. **依赖注入练习**：创建一个 `Logger` 接口，实现 `FileLogger` 和 `DatabaseLogger`，创建一个服务类依赖 `Logger` 接口。

5. **多态练习**：创建一个 `Shape` 接口，定义 `getArea()` 方法，实现 `Rectangle` 和 `Circle` 类，创建函数处理 `Shape` 类型。

## 相关章节

- **[3.3.1 继承（Inheritance）](section-01-inheritance.md)**：了解继承的概念
- **[3.3.3 抽象类（Abstract Class）](section-03-abstract-classes.md)**：了解抽象类与接口的区别
- **[3.3.4 多态（Polymorphism）](section-04-polymorphism.md)**：了解接口在多态中的应用
- **[3.1.2 可见性修饰符](../chapter-01-classes/section-02-visibility.md)**：了解可见性修饰符
