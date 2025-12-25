# 3.3.4 多态（Polymorphism）

## 概述

多态（Polymorphism）允许不同类的对象对同一消息做出不同响应。通过接口或抽象类实现，提高代码的灵活性和可扩展性。

## 多态概念

### 基本定义

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

## 多态在依赖注入中的应用

### 接口类型提示

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

## 完整示例：支付系统

```php
<?php
declare(strict_types=1);

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

## 多态优势

### 代码灵活性

```php
interface Logger
{
    public function log(string $message, string $level): void;
}

class FileLogger implements Logger
{
    public function log(string $message, string $level): void
    {
        file_put_contents('app.log', "[{$level}] {$message}\n", FILE_APPEND);
    }
}

class DatabaseLogger implements Logger
{
    public function log(string $message, string $level): void
    {
        // 写入数据库
    }
}

class LoggerService
{
    public function __construct(
        private Logger $logger
    ) {
    }

    public function info(string $message): void
    {
        $this->logger->log($message, 'INFO');
    }
}

// 可以轻松切换不同的日志实现
$fileLogger = new LoggerService(new FileLogger());
$dbLogger = new LoggerService(new DatabaseLogger());
```

## 注意事项

1. **接口设计**：接口应该定义清晰、简洁的方法签名。

2. **实现一致性**：所有实现类应该遵循接口的语义。

3. **依赖注入**：使用接口类型提示实现依赖注入和多态。

4. **扩展性**：通过添加新的实现类扩展功能，而不修改现有代码。

5. **测试友好**：接口可以轻松创建模拟对象进行测试。

## 练习

1. 创建一个 `Logger` 接口，实现 `FileLogger`、`DatabaseLogger`、`ConsoleLogger`，然后创建一个 `LoggerFactory` 类根据配置返回不同的日志器实例。

2. 设计一个 `Notification` 抽象类，包含 `send()` 抽象方法，创建 `EmailNotification`、`SmsNotification`、`PushNotification` 子类，使用多态实现一个通知管理器。

3. 实现一个 `Cache` 接口和多个实现类，创建一个 `CacheManager` 类使用接口类型提示。

4. 创建一个 `Shape` 接口，实现 `Rectangle`、`Circle`、`Triangle` 类，使用多态计算所有形状的总面积。

5. 设计一个 `Storage` 接口，实现 `FileStorage` 和 `S3Storage`，创建一个服务类可以切换不同的存储实现。
