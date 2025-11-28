# 3.3 OOP 三大特性

## 目标

- 理解继承（Inheritance）的概念，掌握 `extends` 关键字与 `final` 关键字的使用。
- 掌握接口（Interface）的定义与实现，理解接口作为契约的作用。
- 熟悉抽象类（Abstract Class）的使用场景，理解抽象类与接口的区别。
- 理解多态（Polymorphism）的概念，能够在实际项目中应用多态设计。

## 继承（Inheritance）

### 基础语法

- **语法**：`class ChildClass extends ParentClass { ... }`
- 子类继承父类的所有 `public` 和 `protected` 属性和方法。
- 子类可以重写（override）父类方法，也可以添加新方法。

```php
class Animal
{
    public function __construct(
        public string $name
    ) {
    }

    public function makeSound(): string
    {
        return 'Some sound';
    }

    public function move(): string
    {
        return 'Moving';
    }
}

class Dog extends Animal
{
    public function makeSound(): string
    {
        return 'Woof!';
    }

    public function fetch(): string
    {
        return 'Fetching the ball';
    }
}

$dog = new Dog('Buddy');
echo $dog->makeSound(); // Woof!
echo $dog->move();      // Moving（继承自父类）
echo $dog->fetch();     // Fetching the ball
```

### 访问父类方法

- 使用 `parent::methodName()` 调用父类方法。

```php
class Vehicle
{
    public function start(): string
    {
        return 'Engine started';
    }
}

class Car extends Vehicle
{
    public function start(): string
    {
        $parentResult = parent::start();
        return $parentResult . ' - Car is ready';
    }
}

$car = new Car();
echo $car->start(); // Engine started - Car is ready
```

### 构造函数继承

- 子类构造函数可以调用父类构造函数。

```php
class Person
{
    public function __construct(
        public string $name,
        public int $age
    ) {
    }
}

class Student extends Person
{
    public function __construct(
        string $name,
        int $age,
        public string $studentId
    ) {
        parent::__construct($name, $age);
    }
}

$student = new Student('Alice', 20, 'S12345');
echo $student->name;       // Alice
echo $student->studentId; // S12345
```

### `final` 关键字

- `final class`：禁止被继承。
- `final function`：禁止被子类重写。

```php
final class Math
{
    public static function add(float $a, float $b): float
    {
        return $a + $b;
    }
}

// class AdvancedMath extends Math {} // 错误：Cannot extend final class

class Base
{
    public final function criticalMethod(): void
    {
        // 关键逻辑，不允许重写
    }
}

class Derived extends Base
{
    // public function criticalMethod(): void {} // 错误：Cannot override final method
}
```

### 继承层次示例

```php
class Shape
{
    public function __construct(
        public string $color
    ) {
    }

    public function getColor(): string
    {
        return $this->color;
    }

    public function getArea(): float
    {
        return 0.0; // 基类默认实现
    }
}

class Rectangle extends Shape
{
    public function __construct(
        string $color,
        public float $width,
        public float $height
    ) {
        parent::__construct($color);
    }

    public function getArea(): float
    {
        return $this->width * $this->height;
    }
}

class Circle extends Shape
{
    public function __construct(
        string $color,
        public float $radius
    ) {
        parent::__construct($color);
    }

    public function getArea(): float
    {
        return pi() * $this->radius ** 2;
    }
}

$rect = new Rectangle('red', 10, 5);
$circle = new Circle('blue', 3);

echo $rect->getArea();  // 50
echo $circle->getArea(); // 约 28.27
```

## 接口（Interface）

### 基础语法

- **语法**：`interface InterfaceName { ... }`
- 接口定义方法签名，不包含实现。
- 类使用 `implements` 关键字实现接口。

```php
interface Logger
{
    public function log(string $message, string $level = 'info'): void;
}

class FileLogger implements Logger
{
    public function log(string $message, string $level = 'info'): void
    {
        $line = sprintf("[%s] %s: %s\n", date('Y-m-d H:i:s'), strtoupper($level), $message);
        file_put_contents('app.log', $line, FILE_APPEND);
    }
}

class DatabaseLogger implements Logger
{
    public function log(string $message, string $level = 'info'): void
    {
        // 写入数据库的逻辑
        echo "Database log: [{$level}] {$message}\n";
    }
}
```

### 接口方法可见性

- 接口方法默认为 `public`，不能使用其他可见性修饰符。
- PHP 8.0+ 支持在接口中定义常量。

```php
interface PaymentGateway
{
    public const STATUS_PENDING = 'pending';
    public const STATUS_COMPLETED = 'completed';
    public const STATUS_FAILED = 'failed';

    public function processPayment(float $amount): bool;
    public function refund(string $transactionId): bool;
}
```

### 实现多个接口

- 一个类可以实现多个接口。

```php
interface Serializable
{
    public function serialize(): string;
}

interface Deserializable
{
    public static function deserialize(string $data): self;
}

class User implements Serializable, Deserializable
{
    public function __construct(
        public int $id,
        public string $name
    ) {
    }

    public function serialize(): string
    {
        return json_encode(['id' => $this->id, 'name' => $this->name]);
    }

    public static function deserialize(string $data): self
    {
        $decoded = json_decode($data, true);
        return new self($decoded['id'], $decoded['name']);
    }
}
```

### 接口继承

- 接口可以继承其他接口。

```php
interface Readable
{
    public function read(): string;
}

interface Writable
{
    public function write(string $data): void;
}

interface ReadWritable extends Readable, Writable
{
    // 继承了两个接口的所有方法
}

class FileHandler implements ReadWritable
{
    public function read(): string
    {
        return 'File content';
    }

    public function write(string $data): void
    {
        echo "Writing: {$data}\n";
    }
}
```

### 接口作为类型提示

- 接口可以用作类型提示，实现多态。

```php
interface Notifier
{
    public function send(string $message): void;
}

class EmailNotifier implements Notifier
{
    public function send(string $message): void
    {
        echo "Sending email: {$message}\n";
    }
}

class SmsNotifier implements Notifier
{
    public function send(string $message): void
    {
        echo "Sending SMS: {$message}\n";
    }
}

class NotificationService
{
    public function __construct(
        private Notifier $notifier
    ) {
    }

    public function notify(string $message): void
    {
        $this->notifier->send($message);
    }
}

$emailService = new NotificationService(new EmailNotifier());
$emailService->notify('Hello via email');

$smsService = new NotificationService(new SmsNotifier());
$smsService->notify('Hello via SMS');
```

## 抽象类（Abstract Class）

### 基础语法

- **语法**：`abstract class AbstractClassName { ... }`
- 抽象类不能被实例化，只能被继承。
- 可以包含抽象方法（无实现）和普通方法（有实现）。

```php
abstract class Animal
{
    public function __construct(
        public string $name
    ) {
    }

    // 抽象方法：子类必须实现
    abstract public function makeSound(): string;

    // 普通方法：子类可以继承或重写
    public function sleep(): string
    {
        return "{$this->name} is sleeping";
    }
}

class Dog extends Animal
{
    public function makeSound(): string
    {
        return 'Woof!';
    }
}

class Cat extends Animal
{
    public function makeSound(): string
    {
        return 'Meow!';
    }
}

$dog = new Dog('Buddy');
echo $dog->makeSound(); // Woof!
echo $dog->sleep();     // Buddy is sleeping
```

### 抽象方法

- 抽象方法只有签名，没有实现。
- 子类必须实现所有抽象方法。

```php
abstract class DatabaseConnection
{
    abstract public function connect(): void;
    abstract public function query(string $sql): array;
    abstract public function disconnect(): void;

    public function execute(string $sql): array
    {
        $this->connect();
        $result = $this->query($sql);
        $this->disconnect();
        return $result;
    }
}

class MySQLConnection extends DatabaseConnection
{
    public function connect(): void
    {
        echo "Connecting to MySQL...\n";
    }

    public function query(string $sql): array
    {
        echo "Executing: {$sql}\n";
        return [];
    }

    public function disconnect(): void
    {
        echo "Disconnecting from MySQL...\n";
    }
}
```

### 抽象类 vs 接口

| 特性           | 抽象类                     | 接口                       |
| :------------- | :------------------------- | :------------------------- |
| 实例化         | 不能实例化                 | 不能实例化                 |
| 方法实现       | 可以包含实现               | 不能包含实现（PHP 8.0+ 除外） |
| 属性           | 可以包含属性               | 只能包含常量               |
| 继承           | 单继承                     | 多实现                     |
| 使用场景       | 提供部分实现的模板         | 定义契约                   |
| 构造函数       | 可以有构造函数             | 不能有构造函数             |

### 抽象类示例

```php
abstract class PaymentProcessor
{
    public function __construct(
        protected float $amount
    ) {
    }

    // 模板方法模式
    final public function process(): bool
    {
        $this->validate();
        $this->authorize();
        $result = $this->charge();
        $this->log($result);
        return $result;
    }

    abstract protected function validate(): void;
    abstract protected function authorize(): void;
    abstract protected function charge(): bool;

    protected function log(bool $success): void
    {
        $status = $success ? 'SUCCESS' : 'FAILED';
        echo "Payment {$status}: {$this->amount}\n";
    }
}

class CreditCardProcessor extends PaymentProcessor
{
    protected function validate(): void
    {
        echo "Validating credit card...\n";
    }

    protected function authorize(): void
    {
        echo "Authorizing payment...\n";
    }

    protected function charge(): bool
    {
        echo "Charging credit card...\n";
        return true;
    }
}
```

## 多态（Polymorphism）

### 概念

- 多态允许不同类的对象对同一消息做出不同响应。
- 通过接口或抽象类实现，提高代码的灵活性和可扩展性。

```php
interface Shape
{
    public function getArea(): float;
    public function getPerimeter(): float;
}

class Rectangle implements Shape
{
    public function __construct(
        public float $width,
        public float $height
    ) {
    }

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
        public float $radius
    ) {
    }

    public function getArea(): float
    {
        return pi() * $this->radius ** 2;
    }

    public function getPerimeter(): float
    {
        return 2 * pi() * $this->radius;
    }
}

class ShapeCalculator
{
    public function calculateTotalArea(array $shapes): float
    {
        $total = 0.0;
        foreach ($shapes as $shape) {
            $total += $shape->getArea(); // 多态：不同对象调用相同方法
        }
        return $total;
    }
}

$shapes = [
    new Rectangle(10, 5),
    new Circle(3),
    new Rectangle(4, 4),
];

$calculator = new ShapeCalculator();
echo $calculator->calculateTotalArea($shapes); // 约 78.27
```

### 多态在依赖注入中的应用

```php
interface Repository
{
    public function find(int $id): ?object;
    public function save(object $entity): void;
}

class UserRepository implements Repository
{
    public function find(int $id): ?object
    {
        // 从数据库查找用户
        return null;
    }

    public function save(object $entity): void
    {
        // 保存用户到数据库
    }
}

class ProductRepository implements Repository
{
    public function find(int $id): ?object
    {
        // 从数据库查找产品
        return null;
    }

    public function save(object $entity): void
    {
        // 保存产品到数据库
    }
}

class Service
{
    public function __construct(
        private Repository $repository
    ) {
    }

    public function getEntity(int $id): ?object
    {
        return $this->repository->find($id);
    }
}

// 可以注入不同的 Repository 实现
$userService = new Service(new UserRepository());
$productService = new Service(new ProductRepository());
```

## 综合示例：支付系统

```php
interface PaymentMethod
{
    public function process(float $amount): bool;
    public function refund(string $transactionId): bool;
}

abstract class BasePaymentMethod implements PaymentMethod
{
    protected function log(string $message): void
    {
        echo "[LOG] {$message}\n";
    }

    abstract public function process(float $amount): bool;
    abstract public function refund(string $transactionId): bool;
}

class CreditCardPayment extends BasePaymentMethod
{
    public function process(float $amount): bool
    {
        $this->log("Processing credit card payment: {$amount}");
        // 实际支付逻辑
        return true;
    }

    public function refund(string $transactionId): bool
    {
        $this->log("Refunding credit card transaction: {$transactionId}");
        return true;
    }
}

class PayPalPayment extends BasePaymentMethod
{
    public function process(float $amount): bool
    {
        $this->log("Processing PayPal payment: {$amount}");
        return true;
    }

    public function refund(string $transactionId): bool
    {
        $this->log("Refunding PayPal transaction: {$transactionId}");
        return true;
    }
}

class PaymentService
{
    public function __construct(
        private PaymentMethod $paymentMethod
    ) {
    }

    public function pay(float $amount): bool
    {
        return $this->paymentMethod->process($amount);
    }

    public function refundPayment(string $transactionId): bool
    {
        return $this->paymentMethod->refund($transactionId);
    }
}

// 使用多态，可以轻松切换支付方式
$creditCardService = new PaymentService(new CreditCardPayment());
$creditCardService->pay(100.0);

$paypalService = new PaymentService(new PayPalPayment());
$paypalService->pay(50.0);
```

## 练习

1. 创建一个 `Vehicle` 抽象类，包含抽象方法 `start()` 和 `stop()`，然后创建 `Car` 和 `Motorcycle` 子类实现这些方法。

2. 定义一个 `Cache` 接口，包含 `get()`、`set()`、`delete()` 方法，然后实现 `FileCache` 和 `MemoryCache` 类。

3. 创建一个 `Animal` 抽象类，包含 `name` 属性和抽象方法 `makeSound()`，创建 `Dog`、`Cat`、`Bird` 子类，使用多态创建一个 `Zoo` 类管理所有动物。

4. 实现一个 `Repository` 接口和 `UserRepository`、`ProductRepository` 实现类，创建一个 `Service` 类使用接口类型提示，演示依赖注入和多态。

5. 创建一个 `Logger` 接口，实现 `FileLogger`、`DatabaseLogger`、`ConsoleLogger`，然后创建一个 `LoggerFactory` 类根据配置返回不同的日志器实例。

6. 设计一个 `Notification` 抽象类，包含 `send()` 抽象方法，创建 `EmailNotification`、`SmsNotification`、`PushNotification` 子类，使用多态实现一个通知管理器。
