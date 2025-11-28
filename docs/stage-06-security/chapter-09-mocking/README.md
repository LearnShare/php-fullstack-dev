# 6.9 Mock、Stub、Fakes 深度讲解

## 目标

- 深入理解 Mock、Stub、Fake 的区别和使用场景。
- 掌握 Mockery 的高级用法。
- 熟悉 Pest Fakes 的使用。
- 能够编写可测试的代码。

## Mock vs Stub vs Fake

### 概念区别

| 类型 | 说明                           | 使用场景                 |
| :--- | :----------------------------- | :----------------------- |
| **Mock** | 验证交互，检查方法是否被调用   | 验证行为                 |
| **Stub** | 返回预设值，不验证交互         | 提供数据                 |
| **Fake** | 简化实现，提供真实功能的部分实现 | 替代真实实现（如内存数据库） |

### 示例对比

```php
<?php
declare(strict_types=1);

// Mock：验证方法是否被调用
$mock = Mockery::mock(UserRepository::class);
$mock->shouldReceive('save')
    ->once()
    ->with(Mockery::type(User::class))
    ->andReturn(true);

$service = new UserService($mock);
$service->createUser('Alice');

// Stub：只返回预设值
$stub = Mockery::mock(UserRepository::class);
$stub->shouldReceive('find')
    ->andReturn(new User(['id' => 1, 'name' => 'Alice']));

$service = new UserService($stub);
$user = $service->getUser(1);

// Fake：提供简化实现
class FakeUserRepository implements UserRepository
{
    private array $users = [];
    
    public function save(User $user): bool
    {
        $this->users[$user->id] = $user;
        return true;
    }
    
    public function find(int $id): ?User
    {
        return $this->users[$id] ?? null;
    }
}

$fake = new FakeUserRepository();
$service = new UserService($fake);
```

## Mockery 高级用法

### 基础 Mock

```php
<?php
declare(strict_types=1);

use Mockery;

// 创建 Mock
$mock = Mockery::mock(UserRepository::class);

// 设置期望
$mock->shouldReceive('find')
    ->once()
    ->with(1)
    ->andReturn(new User(['id' => 1, 'name' => 'Alice']));

// 使用
$user = $mock->find(1);

// 验证所有期望
Mockery::close();
```

### 部分 Mock

```php
<?php
declare(strict_types=1);

// 部分 Mock：只 Mock 部分方法
$user = Mockery::mock(User::class)->makePartial();
$user->shouldReceive('sendEmail')
    ->once()
    ->andReturn(true);

// 其他方法使用真实实现
$user->getName(); // 调用真实方法
$user->sendEmail(); // 调用 Mock 方法
```

### 命名 Mock

```php
<?php
declare(strict_types=1);

// 创建命名 Mock（不需要真实类）
$mock = Mockery::mock('UserRepository');
$mock->shouldReceive('find')
    ->andReturn(new User());

// 或使用别名
$mock = Mockery::mock('alias:App\Services\EmailService');
```

### 期望链

```php
<?php
declare(strict_types=1);

$mock = Mockery::mock(UserRepository::class);

// 链式调用期望
$mock->shouldReceive('query')
    ->once()
    ->andReturnSelf()
    ->getMock()
    ->shouldReceive('where')
    ->once()
    ->with('status', 'active')
    ->andReturnSelf()
    ->getMock()
    ->shouldReceive('get')
    ->once()
    ->andReturn([new User(), new User()]);
```

### 参数匹配

```php
<?php
declare(strict_types=1);

$mock = Mockery::mock(UserRepository::class);

// 类型匹配
$mock->shouldReceive('save')
    ->with(Mockery::type(User::class));

// 值匹配
$mock->shouldReceive('update')
    ->with(1, Mockery::on(function ($data) {
        return isset($data['name']) && $data['name'] === 'Alice';
    }));

// 任意参数
$mock->shouldReceive('delete')
    ->with(Mockery::any());

// 数组匹配
$mock->shouldReceive('bulkUpdate')
    ->with(Mockery::subset(['status' => 'active']));
```

### 返回值变化

```php
<?php
declare(strict_types=1);

$mock = Mockery::mock(UserRepository::class);

// 第一次返回一个值，第二次返回另一个值
$mock->shouldReceive('find')
    ->andReturn(null, new User(['id' => 1]));

// 或使用序列
$mock->shouldReceive('getStatus')
    ->andReturnValues(['pending', 'active', 'completed']);

// 抛出异常
$mock->shouldReceive('save')
    ->andThrow(new DatabaseException('Connection failed'));
```

### 调用次数

```php
<?php
declare(strict_types=1);

$mock = Mockery::mock(UserRepository::class);

// 精确次数
$mock->shouldReceive('save')->once();
$mock->shouldReceive('find')->twice();
$mock->shouldReceive('delete')->times(3);

// 至少/最多
$mock->shouldReceive('update')->atLeast()->once();
$mock->shouldReceive('update')->atMost()->times(5);

// 零次或多次
$mock->shouldReceive('log')->zeroOrMoreTimes();
```

## Pest Fakes

### 基础使用

```php
<?php
declare(strict_types=1);

use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\Queue;

test('sends welcome email', function () {
    Mail::fake();
    
    $user = createUser();
    
    // 断言邮件已发送
    Mail::assertSent(WelcomeEmail::class);
    
    // 断言邮件发送给特定用户
    Mail::assertSent(WelcomeEmail::class, function ($mail) use ($user) {
        return $mail->hasTo($user->email);
    });
});

test('dispatches job to queue', function () {
    Queue::fake();
    
    processOrder($order);
    
    Queue::assertPushed(ProcessOrderJob::class);
    
    Queue::assertPushed(ProcessOrderJob::class, function ($job) use ($order) {
        return $job->order->id === $order->id;
    });
});
```

### 自定义 Fake

```php
<?php
declare(strict_types=1);

use Illuminate\Support\Facades\Storage;

test('uploads file', function () {
    Storage::fake('local');
    
    $file = UploadedFile::fake()->image('avatar.jpg');
    
    $path = Storage::disk('local')->putFile('avatars', $file);
    
    Storage::disk('local')->assertExists($path);
});
```

### HTTP Fake

```php
<?php
declare(strict_types=1);

use Illuminate\Support\Facades\Http;

test('fetches data from API', function () {
    Http::fake([
        'api.example.com/*' => Http::response(['data' => 'test'], 200),
    ]);
    
    $response = Http::get('api.example.com/users');
    
    expect($response->json())->toBe(['data' => 'test']);
    
    Http::assertSent(function ($request) {
        return $request->url() === 'api.example.com/users';
    });
});
```

## 编写可测试的代码

### 依赖注入

```php
<?php
declare(strict_types=1);

// 可测试的设计
class UserService
{
    public function __construct(
        private UserRepository $repository,
        private EmailService $emailService
    ) {
    }
    
    public function createUser(string $name, string $email): User
    {
        $user = new User(['name' => $name, 'email' => $email]);
        $this->repository->save($user);
        $this->emailService->sendWelcomeEmail($user);
        return $user;
    }
}

// 测试
test('creates user and sends email', function () {
    $repository = Mockery::mock(UserRepository::class);
    $emailService = Mockery::mock(EmailService::class);
    
    $repository->shouldReceive('save')
        ->once()
        ->with(Mockery::type(User::class));
    
    $emailService->shouldReceive('sendWelcomeEmail')
        ->once()
        ->with(Mockery::type(User::class));
    
    $service = new UserService($repository, $emailService);
    $user = $service->createUser('Alice', 'alice@example.com');
    
    expect($user->name)->toBe('Alice');
});
```

### 接口抽象

```php
<?php
declare(strict_types=1);

// 定义接口
interface PaymentGateway
{
    public function charge(float $amount, string $token): bool;
}

// 实现
class StripeGateway implements PaymentGateway
{
    public function charge(float $amount, string $token): bool
    {
        // Stripe API 调用
    }
}

// 在测试中使用 Mock
test('processes payment', function () {
    $gateway = Mockery::mock(PaymentGateway::class);
    $gateway->shouldReceive('charge')
        ->once()
        ->with(100.0, 'token_123')
        ->andReturn(true);
    
    $service = new PaymentService($gateway);
    $result = $service->processPayment(100.0, 'token_123');
    
    expect($result)->toBeTrue();
});
```

### 时间依赖

```php
<?php
declare(strict_types=1);

// 使用 Carbon 的 now() 可以被 Mock
use Carbon\Carbon;

class OrderService
{
    public function createOrder(array $items): Order
    {
        $order = new Order([
            'items' => $items,
            'created_at' => Carbon::now(),
        ]);
        
        return $order;
    }
}

// 测试
test('creates order with current time', function () {
    $fixedTime = Carbon::parse('2024-01-01 12:00:00');
    Carbon::setTestNow($fixedTime);
    
    $service = new OrderService();
    $order = $service->createOrder([...]);
    
    expect($order->created_at)->toEqual($fixedTime);
    
    Carbon::setTestNow(); // 恢复
});
```

## 高级测试模式

### Spy 模式

```php
<?php
declare(strict_types=1);

// Spy：记录调用，事后验证
$spy = Mockery::spy(UserRepository::class);

$service = new UserService($spy);
$service->createUser('Alice');

// 事后验证
$spy->shouldHaveReceived('save')
    ->once()
    ->with(Mockery::type(User::class));
```

### 测试替身层次

```php
<?php
declare(strict_types=1);

// Dummy：不需要实现，只占位
$dummy = Mockery::mock(Logger::class);

// Stub：返回预设值
$stub = Mockery::mock(UserRepository::class);
$stub->shouldReceive('find')->andReturn(new User());

// Mock：验证交互
$mock = Mockery::mock(EmailService::class);
$mock->shouldReceive('send')->once();

// Fake：简化实现
$fake = new FakeDatabase();
```

## 最佳实践

### 1. 只 Mock 外部依赖

```php
<?php
// 错误：Mock 被测试的类
$mock = Mockery::mock(UserService::class);

// 正确：Mock 依赖
$mock = Mockery::mock(UserRepository::class);
$service = new UserService($mock);
```

### 2. 使用接口而非具体类

```php
<?php
// 推荐：依赖接口
class UserService
{
    public function __construct(
        private UserRepositoryInterface $repository
    ) {
    }
}
```

### 3. 避免过度 Mock

```php
<?php
// 错误：Mock 太多
$mock1 = Mockery::mock(Repository::class);
$mock2 = Mockery::mock(Service::class);
$mock3 = Mockery::mock(Logger::class);
// ...

// 正确：只 Mock 必要的依赖
$mock = Mockery::mock(ExternalApi::class);
```

## 练习

1. 使用 Mockery 创建一个完整的测试套件，测试用户服务。

2. 使用 Pest Fakes 测试邮件发送和队列任务。

3. 创建一个 Fake 实现，替代真实的数据库访问。

4. 使用 Spy 模式验证方法的调用顺序。

5. 编写可测试的代码，使用依赖注入和接口抽象。

6. 创建一个测试替身层次，包含 Dummy、Stub、Mock、Fake。

