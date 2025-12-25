# 6.9.2 Stub 与 Fakes

## 概述

Stub 和 Fakes 是测试替身的重要类型。本节介绍 Stub 的使用、Fake 的实现、测试替身的选择等内容。

## Stub 使用

### 基础 Stub

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

### 链式 Stub

```php
<?php
declare(strict_types=1);

use Mockery;

$mock = Mockery::mock(Service::class);
$mock->shouldReceive('getUser')
    ->with(1)
    ->andReturn(new User('John'))
    ->getMock()
    ->shouldReceive('getEmail')
    ->andReturn('john@example.com');
```

## Fake 实现

### 内存数据库 Fake

```php
<?php
declare(strict_types=1);

class FakeUserRepository implements UserRepositoryInterface
{
    private array $users = [];
    
    public function find(int $id): ?User
    {
        return $this->users[$id] ?? null;
    }
    
    public function save(User $user): void
    {
        if ($user->getId() === null) {
            $user->setId(count($this->users) + 1);
        }
        $this->users[$user->getId()] = $user;
    }
    
    public function delete(int $id): void
    {
        unset($this->users[$id]);
    }
}
```

### 邮件服务 Fake

```php
<?php
declare(strict_types=1);

class FakeEmailService implements EmailServiceInterface
{
    private array $sentEmails = [];
    
    public function send(string $to, string $subject, string $body): bool
    {
        $this->sentEmails[] = [
            'to' => $to,
            'subject' => $subject,
            'body' => $body,
            'sent_at' => time(),
        ];
        return true;
    }
    
    public function getSentEmails(): array
    {
        return $this->sentEmails;
    }
    
    public function clear(): void
    {
        $this->sentEmails = [];
    }
}
```

## 测试替身选择

### 选择指南

- **Mock**：需要验证交互时使用
- **Stub**：只需要返回值时使用
- **Fake**：需要简化但可用的实现时使用

## 完整示例

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class OrderServiceTest extends TestCase
{
    public function testCreateOrderWithFake(): void
    {
        $emailService = new FakeEmailService();
        $paymentService = Mockery::mock(PaymentService::class);
        $paymentService->shouldReceive('charge')
            ->once()
            ->andReturn(true);
        
        $service = new OrderService($emailService, $paymentService);
        $order = $service->createOrder(['total' => 100.0, 'email' => 'user@example.com']);
        
        $this->assertNotNull($order);
        $this->assertCount(1, $emailService->getSentEmails());
        $this->assertEquals('user@example.com', $emailService->getSentEmails()[0]['to']);
    }
}
```

## 最佳实践

1. **选择合适的替身**：根据测试需求选择
2. **使用 Fake 简化**：提供可用的简化实现
3. **验证行为**：使用 Mock 验证交互
4. **保持简单**：测试替身应该简单易用

## 注意事项

1. Fake 应该提供可用的实现
2. Stub 只返回预设值
3. Mock 验证交互
4. 保持测试替身简单

## 练习

1. 创建一个 Fake 实现，替代真实的服务。

2. 使用 Stub 返回预设值，测试业务逻辑。

3. 实现一个内存数据库 Fake，用于测试。

4. 编写一个使用 Fake 和 Stub 的完整测试。
