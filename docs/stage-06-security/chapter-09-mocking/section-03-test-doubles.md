# 6.9.3 测试替身实践

## 概述

测试替身实践是编写可测试代码的关键。本节介绍可测试代码设计、依赖注入、接口抽象等内容。

## 可测试代码设计

### 依赖注入

```php
<?php
declare(strict_types=1);

// 可测试的设计
class OrderService
{
    public function __construct(
        private EmailServiceInterface $emailService,
        private PaymentServiceInterface $paymentService,
    ) {}
    
    public function createOrder(array $data): Order
    {
        // 业务逻辑
        $order = new Order($data);
        
        // 使用注入的服务
        $this->paymentService->charge($order->getTotal());
        $this->emailService->send($order->getEmail(), 'Order Confirmation');
        
        return $order;
    }
}
```

### 接口抽象

```php
<?php
declare(strict_types=1);

interface EmailServiceInterface
{
    public function send(string $to, string $subject, string $body): bool;
}

class SmtpEmailService implements EmailServiceInterface
{
    public function send(string $to, string $subject, string $body): bool
    {
        // 真实实现
        return mail($to, $subject, $body);
    }
}

class FakeEmailService implements EmailServiceInterface
{
    public function send(string $to, string $subject, string $body): bool
    {
        // 测试实现
        return true;
    }
}
```

## 依赖注入容器

### 简单容器

```php
<?php
declare(strict_types=1);

class Container
{
    private array $bindings = [];
    private array $instances = [];
    
    public function bind(string $abstract, callable $concrete): void
    {
        $this->bindings[$abstract] = $concrete;
    }
    
    public function singleton(string $abstract, callable $concrete): void
    {
        $this->bindings[$abstract] = function() use ($concrete) {
            if (!isset($this->instances[$abstract])) {
                $this->instances[$abstract] = $concrete($this);
            }
            return $this->instances[$abstract];
        };
    }
    
    public function make(string $abstract): mixed
    {
        if (isset($this->instances[$abstract])) {
            return $this->instances[$abstract];
        }
        
        if (isset($this->bindings[$abstract])) {
            return $this->bindings[$abstract]($this);
        }
        
        // 自动解析
        return $this->resolve($abstract);
    }
    
    private function resolve(string $class): object
    {
        $reflection = new ReflectionClass($class);
        $constructor = $reflection->getConstructor();
        
        if ($constructor === null) {
            return new $class();
        }
        
        $parameters = $constructor->getParameters();
        $dependencies = [];
        
        foreach ($parameters as $parameter) {
            $type = $parameter->getType();
            if ($type instanceof ReflectionNamedType && !$type->isBuiltin()) {
                $dependencies[] = $this->make($type->getName());
            }
        }
        
        return $reflection->newInstanceArgs($dependencies);
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;
use Mockery;

class OrderServiceTest extends TestCase
{
    protected function tearDown(): void
    {
        Mockery::close();
    }
    
    public function testCreateOrder(): void
    {
        // 创建测试替身
        $emailService = Mockery::mock(EmailServiceInterface::class);
        $emailService->shouldReceive('send')
            ->once()
            ->andReturn(true);
        
        $paymentService = Mockery::mock(PaymentServiceInterface::class);
        $paymentService->shouldReceive('charge')
            ->once()
            ->with(100.0)
            ->andReturn(true);
        
        // 注入测试替身
        $service = new OrderService($emailService, $paymentService);
        
        // 测试
        $order = $service->createOrder([
            'total' => 100.0,
            'email' => 'user@example.com',
        ]);
        
        $this->assertNotNull($order);
    }
}
```

## 最佳实践

1. **依赖注入**：通过构造函数注入依赖
2. **接口抽象**：使用接口定义契约
3. **可测试设计**：设计易于测试的代码
4. **测试替身**：合理使用测试替身

## 注意事项

1. 不要过度使用测试替身
2. 保持代码可测试性
3. 使用接口定义契约
4. 测试应该简单清晰

## 练习

1. 重构一个类，使其可测试，使用依赖注入。

2. 创建接口抽象，定义服务契约。

3. 实现一个简单的依赖注入容器。

4. 编写使用测试替身的完整测试套件。
