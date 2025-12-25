# 6.9.1 Mock 基础

## 概述

Mock 是测试中的重要工具。本节介绍 Mock vs Stub vs Fake、Mockery 使用、Mock 验证等内容。

## Mock vs Stub vs Fake

### Mock

Mock 验证对象之间的交互。

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;
use Mockery;

class OrderServiceTest extends TestCase
{
    public function testCreateOrder(): void
    {
        $emailService = Mockery::mock(EmailService::class);
        $emailService->shouldReceive('send')
            ->once()
            ->with('user@example.com', 'Order Confirmation')
            ->andReturn(true);
        
        $service = new OrderService($emailService);
        $service->createOrder(['item' => 'Product']);
        
        Mockery::close();
    }
}
```

### Stub

Stub 返回预设值，不验证交互。

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;
use Mockery;

class UserServiceTest extends TestCase
{
    public function testGetUser(): void
    {
        $repo = Mockery::mock(UserRepository::class);
        $repo->shouldReceive('find')
            ->with(1)
            ->andReturn(new User('John', 'john@example.com'));
        
        $service = new UserService($repo);
        $user = $service->getUser(1);
        
        $this->assertEquals('John', $user->getName());
    }
}
```

### Fake

Fake 是简化实现，用于测试。

```php
<?php
declare(strict_types=1);

class FakeEmailService implements EmailServiceInterface
{
    private array $sentEmails = [];
    
    public function send(string $to, string $subject): bool
    {
        $this->sentEmails[] = ['to' => $to, 'subject' => $subject];
        return true;
    }
    
    public function getSentEmails(): array
    {
        return $this->sentEmails;
    }
}
```

## Mockery 使用

### 基础 Mock

```php
<?php
declare(strict_types=1);

use Mockery;

$mock = Mockery::mock('MyClass');
$mock->shouldReceive('method')
    ->once()
    ->andReturn('value');
```

### 部分 Mock

```php
<?php
declare(strict_types=1);

use Mockery;

$mock = Mockery::mock(MyClass::class)->makePartial();
$mock->shouldReceive('method')
    ->once()
    ->andReturn('value');
```

## Mock 验证

### 验证调用次数

```php
<?php
declare(strict_types=1);

use Mockery;

$mock = Mockery::mock(Service::class);
$mock->shouldReceive('method')
    ->once(); // 调用一次
$mock->shouldReceive('method')
    ->twice(); // 调用两次
$mock->shouldReceive('method')
    ->times(3); // 调用三次
$mock->shouldReceive('method')
    ->never(); // 从不调用
```

### 验证参数

```php
<?php
declare(strict_types=1);

use Mockery;

$mock = Mockery::mock(Service::class);
$mock->shouldReceive('method')
    ->with('arg1', 'arg2')
    ->once();
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
        $emailService = Mockery::mock(EmailService::class);
        $emailService->shouldReceive('send')
            ->once()
            ->with('user@example.com', 'Order Confirmation')
            ->andReturn(true);
        
        $paymentService = Mockery::mock(PaymentService::class);
        $paymentService->shouldReceive('charge')
            ->once()
            ->with(100.0)
            ->andReturn(true);
        
        $service = new OrderService($emailService, $paymentService);
        $order = $service->createOrder(['total' => 100.0, 'email' => 'user@example.com']);
        
        $this->assertNotNull($order);
    }
}
```

## 最佳实践

1. **使用 Mock 验证交互**：验证对象之间的调用
2. **使用 Stub 返回数据**：预设返回值
3. **使用 Fake 简化实现**：提供简化但可用的实现
4. **清理 Mock**：在 tearDown 中清理

## 注意事项

1. 不要过度使用 Mock
2. Mock 应该验证行为，不只是返回值
3. 保持测试简洁
4. 使用合适的测试替身类型

## 练习

1. 使用 Mockery 创建一个 Mock 对象，验证方法调用。

2. 实现一个 Stub，返回预设值。

3. 创建一个 Fake 实现，用于测试。

4. 编写一个使用 Mock 的完整测试。
