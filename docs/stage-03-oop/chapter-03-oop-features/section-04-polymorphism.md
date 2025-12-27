# 3.3.4 多态（Polymorphism）

## 概述

多态（Polymorphism）是面向对象编程的核心特性之一，允许不同类的对象对同一消息做出不同的响应。多态使得代码更加灵活、可扩展，是实现"开闭原则"（对扩展开放，对修改关闭）的关键机制。

理解多态对于掌握面向对象编程至关重要。多态通过接口和抽象类实现，使得程序可以在运行时决定调用哪个具体实现，而不需要在编译时确定。这种特性使得代码更加灵活，可以轻松地扩展功能而不修改现有代码。

多态通常与依赖注入（Dependency Injection）结合使用，通过接口类型提示，使得代码依赖于抽象而不是具体实现，提高了代码的可测试性和可维护性。

**主要内容**：
- 多态的概念和定义
- 多态的实现方式（接口、抽象类、继承）
- 多态在依赖注入中的应用
- 多态的优势和使用场景
- 完整示例（支付系统、图形计算等）
- 多态的最佳实践

## 特性

- **同一接口，不同实现**：不同的类可以实现相同的接口，提供不同的实现
- **运行时多态**：在运行时决定调用哪个实现
- **代码灵活性**：提高代码的灵活性和可扩展性
- **依赖倒置**：依赖于抽象而不是具体实现
- **开闭原则**：对扩展开放，对修改关闭

## 语法/定义

### 多态的概念

**定义**：多态是指同一个接口，可以有多个不同的实现。不同的对象可以对同一个消息做出不同的响应。

**实现方式**：
- 接口多态：通过接口实现
- 继承多态：通过继承和重写实现
- 抽象类多态：通过抽象类实现

**特点**：
- 运行时决定调用哪个实现
- 代码依赖于抽象（接口或抽象类）
- 提高代码的灵活性和可扩展性

## 基本用法

### 示例 1：接口多态

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

// 多态函数：可以处理任何实现 Shape 接口的对象
function printShapeInfo(Shape $shape): void
{
    echo "Area: " . $shape->getArea() . "\n";
    echo "Perimeter: " . $shape->getPerimeter() . "\n";
}

// 使用多态
$rectangle = new Rectangle(5, 3);
$circle = new Circle(4);

printShapeInfo($rectangle);  // 处理矩形
printShapeInfo($circle);     // 处理圆形

// 多态数组
$shapes = [
    new Rectangle(10, 5),
    new Circle(3),
    new Rectangle(4, 4)
];

$calculator = new class {
    public function calculateTotalArea(array $shapes): float
    {
        $total = 0.0;
        foreach ($shapes as $shape) {
            $total += $shape->getArea();  // 多态调用
        }
        return $total;
    }
};

echo "Total area: " . $calculator->calculateTotalArea($shapes) . "\n";
```

**输出**：

```
Area: 15
Perimeter: 16
Area: 50.265482457437
Perimeter: 25.132741228718
Total area: 78.265482457437
```

**说明**：
- `printShapeInfo()` 函数接受 `Shape` 接口类型
- 可以处理任何实现 `Shape` 接口的对象
- 实现了多态设计，代码更加灵活

### 示例 2：多态在依赖注入中的应用

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
        echo "Refunding credit card transaction: {$transactionId}\n";
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

// 使用接口进行依赖注入
class PaymentService
{
    public function __construct(
        private PaymentMethod $paymentMethod  // 依赖接口，而不是具体类
    ) {}
    
    public function processPayment(float $amount): bool
    {
        return $this->paymentMethod->pay($amount);  // 多态调用
    }
    
    public function refundPayment(string $transactionId): bool
    {
        return $this->paymentMethod->refund($transactionId);
    }
}

// 可以注入不同的实现
$creditCardService = new PaymentService(new CreditCardPayment());
$creditCardService->processPayment(100.0);

$paypalService = new PaymentService(new PayPalPayment());
$paypalService->processPayment(50.0);
```

**输出**：

```
Processing credit card payment: $100
Processing PayPal payment: $50
```

**说明**：
- `PaymentService` 依赖于 `PaymentMethod` 接口，而不是具体实现
- 可以注入不同的实现（CreditCardPayment 或 PayPalPayment）
- 实现了多态设计，提高了代码的灵活性

### 示例 3：继承多态

```php
<?php
declare(strict_types=1);

class Animal
{
    public function __construct(
        protected string $name
    ) {}
    
    public function makeSound(): string
    {
        return "Some sound";
    }
    
    public function introduce(): string
    {
        return "I am {$this->name}, {$this->makeSound()}";
    }
}

class Dog extends Animal
{
    public function makeSound(): string
    {
        return "Woof!";
    }
}

class Cat extends Animal
{
    public function makeSound(): string
    {
        return "Meow!";
    }
}

// 多态函数
function makeAnimalSound(Animal $animal): void
{
    echo $animal->makeSound() . "\n";  // 多态调用
}

$dog = new Dog("Buddy");
$cat = new Cat("Fluffy");

makeAnimalSound($dog);  // Woof!
makeAnimalSound($cat);  // Meow!

// 多态数组
$animals = [
    new Dog("Rex"),
    new Cat("Whiskers"),
    new Dog("Max")
];

foreach ($animals as $animal) {
    echo $animal->introduce() . "\n";  // 多态调用
}
```

**输出**：

```
Woof!
Meow!
I am Rex, Woof!
I am Whiskers, Meow!
I am Max, Woof!
```

**说明**：
- 通过继承实现多态
- `makeAnimalSound()` 函数接受 `Animal` 类型
- 不同子类的对象可以调用同一个函数，产生不同的结果

### 示例 4：抽象类多态

```php
<?php
declare(strict_types=1);

abstract class Logger
{
    abstract public function log(string $message, string $level): void;
    
    public function info(string $message): void
    {
        $this->log($message, "INFO");
    }
    
    public function error(string $message): void
    {
        $this->log($message, "ERROR");
    }
}

class FileLogger extends Logger
{
    public function log(string $message, string $level): void
    {
        $logLine = date('Y-m-d H:i:s') . " [{$level}] {$message}\n";
        file_put_contents('app.log', $logLine, FILE_APPEND);
        echo "Logged to file: [{$level}] {$message}\n";
    }
}

class ConsoleLogger extends Logger
{
    public function log(string $message, string $level): void
    {
        echo "[{$level}] {$message}\n";
    }
}

// 使用抽象类进行依赖注入
class LoggerService
{
    public function __construct(
        private Logger $logger
    ) {}
    
    public function logInfo(string $message): void
    {
        $this->logger->info($message);  // 多态调用
    }
    
    public function logError(string $message): void
    {
        $this->logger->error($message);  // 多态调用
    }
}

$fileLogger = new LoggerService(new FileLogger());
$fileLogger->logInfo("Application started");

$consoleLogger = new LoggerService(new ConsoleLogger());
$consoleLogger->logError("An error occurred");
```

**输出**：

```
Logged to file: [INFO] Application started
[ERROR] An error occurred
```

**说明**：
- 通过抽象类实现多态
- `LoggerService` 依赖于 `Logger` 抽象类
- 可以注入不同的实现（FileLogger 或 ConsoleLogger）

### 示例 5：完整支付系统示例

```php
<?php
declare(strict_types=1);

interface PaymentMethod
{
    public function process(float $amount): bool;
    public function refund(string $transactionId): bool;
    public function getPaymentMethodName(): string;
}

abstract class BasePaymentMethod implements PaymentMethod
{
    protected function log(string $message): void
    {
        echo "[LOG] {$message}\n";
    }
    
    abstract public function process(float $amount): bool;
    abstract public function refund(string $transactionId): bool;
    abstract public function getPaymentMethodName(): string;
}

class CreditCardPayment extends BasePaymentMethod
{
    public function process(float $amount): bool
    {
        $this->log("Processing credit card payment: \${$amount}");
        // 实际的支付逻辑
        return true;
    }
    
    public function refund(string $transactionId): bool
    {
        $this->log("Refunding credit card transaction: {$transactionId}");
        return true;
    }
    
    public function getPaymentMethodName(): string
    {
        return "Credit Card";
    }
}

class PayPalPayment extends BasePaymentMethod
{
    public function process(float $amount): bool
    {
        $this->log("Processing PayPal payment: \${$amount}");
        return true;
    }
    
    public function refund(string $transactionId): bool
    {
        $this->log("Refunding PayPal transaction: {$transactionId}");
        return true;
    }
    
    public function getPaymentMethodName(): string
    {
        return "PayPal";
    }
}

class BankTransferPayment extends BasePaymentMethod
{
    public function process(float $amount): bool
    {
        $this->log("Processing bank transfer: \${$amount}");
        return true;
    }
    
    public function refund(string $transactionId): bool
    {
        $this->log("Refunding bank transfer: {$transactionId}");
        return true;
    }
    
    public function getPaymentMethodName(): string
    {
        return "Bank Transfer";
    }
}

class PaymentService
{
    public function __construct(
        private PaymentMethod $paymentMethod
    ) {}
    
    public function pay(float $amount): bool
    {
        echo "Using payment method: {$this->paymentMethod->getPaymentMethodName()}\n";
        return $this->paymentMethod->process($amount);
    }
    
    public function refund(string $transactionId): bool
    {
        return $this->paymentMethod->refund($transactionId);
    }
    
    // 可以切换支付方式
    public function setPaymentMethod(PaymentMethod $paymentMethod): void
    {
        $this->paymentMethod = $paymentMethod;
    }
}

// 使用多态
$paymentService = new PaymentService(new CreditCardPayment());
$paymentService->pay(100.0);

// 切换支付方式
$paymentService->setPaymentMethod(new PayPalPayment());
$paymentService->pay(50.0);

// 使用不同的支付方式
$bankTransfer = new PaymentService(new BankTransferPayment());
$bankTransfer->pay(200.0);
```

**输出**：

```
Using payment method: Credit Card
[LOG] Processing credit card payment: $100
Using payment method: PayPal
[LOG] Processing PayPal payment: $50
Using payment method: Bank Transfer
[LOG] Processing bank transfer: $200
```

**说明**：
- 完整的多态支付系统示例
- `PaymentService` 可以接受任何实现 `PaymentMethod` 接口的对象
- 可以轻松添加新的支付方式，而不修改现有代码

### 示例 6：多态在策略模式中的应用

```php
<?php
declare(strict_types=1);

interface SortStrategy
{
    public function sort(array $data): array;
}

class BubbleSort implements SortStrategy
{
    public function sort(array $data): array
    {
        echo "Using Bubble Sort\n";
        $n = count($data);
        for ($i = 0; $i < $n - 1; $i++) {
            for ($j = 0; $j < $n - $i - 1; $j++) {
                if ($data[$j] > $data[$j + 1]) {
                    [$data[$j], $data[$j + 1]] = [$data[$j + 1], $data[$j]];
                }
            }
        }
        return $data;
    }
}

class QuickSort implements SortStrategy
{
    public function sort(array $data): array
    {
        echo "Using Quick Sort\n";
        if (count($data) <= 1) {
            return $data;
        }
        $pivot = $data[0];
        $left = $right = [];
        for ($i = 1; $i < count($data); $i++) {
            if ($data[$i] < $pivot) {
                $left[] = $data[$i];
            } else {
                $right[] = $data[$i];
            }
        }
        return array_merge($this->sort($left), [$pivot], $this->sort($right));
    }
}

class Sorter
{
    public function __construct(
        private SortStrategy $strategy
    ) {}
    
    public function sort(array $data): array
    {
        return $this->strategy->sort($data);  // 多态调用
    }
    
    public function setStrategy(SortStrategy $strategy): void
    {
        $this->strategy = $strategy;
    }
}

$data = [64, 34, 25, 12, 22, 11, 90];

$sorter = new Sorter(new BubbleSort());
$result1 = $sorter->sort($data);
echo "Sorted: " . implode(", ", $result1) . "\n";

$sorter->setStrategy(new QuickSort());
$result2 = $sorter->sort($data);
echo "Sorted: " . implode(", ", $result2) . "\n";
```

**输出**：

```
Using Bubble Sort
Sorted: 11, 12, 22, 25, 34, 64, 90
Using Quick Sort
Sorted: 11, 12, 22, 25, 34, 64, 90
```

**说明**：
- 策略模式使用多态实现
- `Sorter` 类可以接受不同的排序策略
- 可以在运行时切换策略

## 使用场景

### 场景 1：支付系统

多态非常适合支付系统，不同的支付方式可以有不同的实现。

**示例**：见"示例 5：完整支付系统示例"

### 场景 2：数据存储系统

多态可以用于数据存储系统，支持不同的存储后端。

**示例**：存储系统

```php
<?php
declare(strict_types=1);

interface Storage
{
    public function save(string $key, mixed $value): void;
    public function get(string $key): mixed;
    public function delete(string $key): void;
}

class FileStorage implements Storage
{
    private string $storageDir;
    
    public function __construct(string $storageDir = '/tmp/storage')
    {
        $this->storageDir = $storageDir;
    }
    
    public function save(string $key, mixed $value): void
    {
        file_put_contents("{$this->storageDir}/{$key}", serialize($value));
    }
    
    public function get(string $key): mixed
    {
        $file = "{$this->storageDir}/{$key}";
        if (!file_exists($file)) {
            return null;
        }
        return unserialize(file_get_contents($file));
    }
    
    public function delete(string $key): void
    {
        $file = "{$this->storageDir}/{$key}";
        if (file_exists($file)) {
            unlink($file);
        }
    }
}

class MemoryStorage implements Storage
{
    private array $storage = [];
    
    public function save(string $key, mixed $value): void
    {
        $this->storage[$key] = $value;
    }
    
    public function get(string $key): mixed
    {
        return $this->storage[$key] ?? null;
    }
    
    public function delete(string $key): void
    {
        unset($this->storage[$key]);
    }
}

class CacheService
{
    public function __construct(
        private Storage $storage
    ) {}
    
    public function set(string $key, mixed $value): void
    {
        $this->storage->save($key, $value);
    }
    
    public function get(string $key): mixed
    {
        return $this->storage->get($key);
    }
}
```

### 场景 3：通知系统

多态可以用于通知系统，支持不同的通知渠道。

**示例**：通知系统

```php
<?php
declare(strict_types=1);

interface Notifier
{
    public function send(string $to, string $message): bool;
}

class EmailNotifier implements Notifier
{
    public function send(string $to, string $message): bool
    {
        echo "Sending email to {$to}: {$message}\n";
        return true;
    }
}

class SmsNotifier implements Notifier
{
    public function send(string $to, string $message): bool
    {
        echo "Sending SMS to {$to}: {$message}\n";
        return true;
    }
}

class NotificationService
{
    public function __construct(
        private Notifier $notifier
    ) {}
    
    public function notify(string $to, string $message): bool
    {
        return $this->notifier->send($to, $message);
    }
}
```

## 注意事项

### 接口设计

- **合理设计接口**：接口应该定义清晰、简洁的方法签名
- **接口粒度**：接口不要过于庞大，也不要过于琐碎
- **接口稳定性**：接口一旦定义，应该保持稳定，避免频繁修改

### 实现一致性

- **遵循契约**：所有实现类应该遵循接口的语义
- **行为一致性**：不同实现应该有相似的行为特征
- **异常处理**：应该在接口中定义可能的异常

### 性能考虑

- **多态开销**：多态有轻微的性能开销（方法查找）
- **对于大多数应用**：性能影响可以忽略
- **优化建议**：只在必要时使用多态

## 常见问题

### 问题 1：接口设计过于复杂

**症状**：接口包含太多方法，实现困难

**解决方案**：
- 将大接口拆分为多个小接口
- 使用接口组合

**示例**：

```php
<?php
declare(strict_types=1);

// 不好的设计：接口过大
// interface DataProcessor
// {
//     public function load(): void;
//     public function transform(): void;
//     public function save(): void;
//     public function validate(): void;
//     public function log(): void;  // 职责过多
// }

// 好的设计：接口分离
interface Loadable
{
    public function load(): void;
}

interface Transformable
{
    public function transform(): void;
}

interface Savable
{
    public function save(): void;
}

// 需要多个功能时，实现多个接口
class DataProcessor implements Loadable, Transformable, Savable
{
    public function load(): void {}
    public function transform(): void {}
    public function save(): void {}
}
```

### 问题 2：实现不符合接口语义

**症状**：实现的行为不符合接口的预期

**解决方案**：
- 仔细阅读接口文档
- 确保实现符合接口的语义
- 编写测试验证实现

### 问题 3：过度使用多态

**症状**：为简单场景创建过多接口

**解决方案**：
- 只在需要灵活性的场景使用多态
- 避免过早优化
- 保持代码简洁

## 最佳实践

### 1. 使用接口定义多态契约

接口是定义多态契约的最佳方式。

**示例**：

```php
<?php
declare(strict_types=1);

interface Logger
{
    public function log(string $message, string $level): void;
}

class FileLogger implements Logger
{
    public function log(string $message, string $level): void
    {
        // 实现
    }
}
```

### 2. 通过依赖注入实现多态

使用依赖注入实现多态，提高代码的灵活性和可测试性。

**示例**：

```php
<?php
declare(strict_types=1);

class Service
{
    public function __construct(
        private Logger $logger  // 依赖接口
    ) {}
}
```

### 3. 保持接口稳定

接口一旦定义，应该保持稳定，避免频繁修改。

**示例**：

```php
<?php
declare(strict_types=1);

// 稳定的接口设计
interface Repository
{
    public function find(int $id): ?object;
    public function save(object $entity): void;
}

// 如果需要扩展，使用接口继承
interface AdvancedRepository extends Repository
{
    public function findBy(array $criteria): array;
}
```

### 4. 使用抽象类提供部分实现

当需要共享代码时，使用抽象类提供部分实现。

**示例**：

```php
<?php
declare(strict_types=1);

abstract class BaseLogger
{
    protected function formatMessage(string $message, string $level): string
    {
        return date('Y-m-d H:i:s') . " [{$level}] {$message}";
    }
    
    abstract public function log(string $message, string $level): void;
}
```

### 5. 理解多态的优势

- **灵活性**：可以轻松切换实现
- **可扩展性**：可以添加新实现而不修改现有代码
- **可测试性**：可以使用 Mock 对象进行测试
- **解耦**：代码依赖于抽象而不是具体实现

## 对比分析

### 多态 vs 条件判断

| 特性         | 多态                               | 条件判断（if/switch）              |
|:-------------|:-----------------------------------|:-----------------------------------|
| **扩展性**    | ✅ 容易扩展（添加新类）              | ❌ 需要修改现有代码                 |
| **可维护性**  | ✅ 代码更清晰                       | ❌ 条件判断增加复杂度               |
| **性能**      | 轻微开销                           | 直接判断                           |
| **适用场景**  | 需要灵活扩展的场景                  | 固定、简单的场景                   |

### 接口多态 vs 继承多态

| 特性         | 接口多态                           | 继承多态                           |
|:-------------|:-----------------------------------|:-----------------------------------|
| **灵活性**    | ✅ 可以实现多个接口                 | ❌ 只能继承一个类                   |
| **耦合度**    | ✅ 低耦合                           | ❌ 高耦合                           |
| **代码复用**  | ❌ 不能共享代码                     | ✅ 可以共享代码                     |
| **适用场景**  | 定义契约，多态设计                  | 代码复用，层次化设计                |

## 练习任务

1. **接口多态练习**：创建一个 `Shape` 接口，定义 `getArea()` 方法，实现 `Rectangle` 和 `Circle` 类，创建一个函数计算多个图形的总面积。

2. **依赖注入练习**：创建一个 `Logger` 接口，实现 `FileLogger` 和 `DatabaseLogger`，创建一个服务类依赖 `Logger` 接口。

3. **支付系统练习**：创建一个 `PaymentMethod` 接口，实现 `CreditCardPayment` 和 `PayPalPayment`，创建一个 `PaymentService` 类使用接口。

4. **策略模式练习**：创建一个 `SortStrategy` 接口，实现 `BubbleSort` 和 `QuickSort`，创建一个 `Sorter` 类可以使用不同的排序策略。

5. **通知系统练习**：创建一个 `Notifier` 接口，实现 `EmailNotifier` 和 `SmsNotifier`，创建一个 `NotificationService` 类可以发送不同类型的通知。

## 相关章节

- **[3.3.1 继承（Inheritance）](section-01-inheritance.md)**：了解继承如何为多态提供基础
- **[3.3.2 接口（Interface）](section-02-interfaces.md)**：了解接口在多态中的作用
- **[3.3.3 抽象类（Abstract Class）](section-03-abstract-classes.md)**：了解抽象类在多态中的应用
