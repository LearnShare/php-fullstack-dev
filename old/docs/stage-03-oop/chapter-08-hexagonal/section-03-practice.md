# 3.8.3 六边形架构实践

## 概述

本章介绍六边形架构的实际应用，包括完整的项目结构、依赖注入配置、测试策略和最佳实践。

## 完整项目结构

### 目录组织

```
src/
├── Domain/                    # 领域层（核心）
│   ├── User/
│   │   ├── User.php
│   │   ├── UserId.php
│   │   └── Email.php
│   ├── Ports/
│   │   ├── Inbound/
│   │   │   └── UserServicePort.php
│   │   └── Outbound/
│   │       ├── UserRepositoryPort.php
│   │       └── EmailServicePort.php
│   └── UserService.php
├── Application/               # 应用层（可选）
│   └── DTOs/
├── Adapters/
│   ├── Inbound/
│   │   ├── Http/
│   │   │   └── UserController.php
│   │   └── CLI/
│   │       └── UserCommand.php
│   └── Outbound/
│       ├── Database/
│       │   └── DatabaseUserRepository.php
│       ├── Email/
│       │   └── SmtpEmailService.php
│       └── Test/
│           ├── InMemoryUserRepository.php
│           └── MockEmailService.php
└── Infrastructure/            # 基础设施（可选）
    └── Container.php
```

## 依赖注入配置

### 容器实现

```php
namespace App\Infrastructure;

class Container
{
    private array $bindings = [];
    private array $instances = [];

    public function bind(string $interface, string $implementation): void
    {
        $this->bindings[$interface] = $implementation;
    }

    public function singleton(string $interface, string $implementation): void
    {
        $this->bind($interface, $implementation);
        $this->instances[$interface] = null;
    }

    public function make(string $interface): object
    {
        if (isset($this->instances[$interface])) {
            return $this->instances[$interface];
        }

        $implementation = $this->bindings[$interface] ?? $interface;
        $instance = $this->resolve($implementation);

        if (isset($this->instances[$interface])) {
            $this->instances[$interface] = $instance;
        }

        return $instance;
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
            if ($type instanceof ReflectionNamedType) {
                $dependencies[] = $this->make($type->getName());
            }
        }

        return $reflection->newInstanceArgs($dependencies);
    }
}

// 配置
$container = new Container();

// 绑定端口和适配器
$container->bind(UserRepositoryPort::class, DatabaseUserRepository::class);
$container->bind(EmailServicePort::class, SmtpEmailService::class);
$container->singleton(UserServicePort::class, UserService::class);
```

## 测试策略

### 单元测试

```php
use App\Domain\UserService;
use App\Adapters\Outbound\Test\{InMemoryUserRepository, MockEmailService};

class UserServiceTest extends TestCase
{
    public function testCreateUser(): void
    {
        $repository = new InMemoryUserRepository();
        $emailService = new MockEmailService();
        $service = new UserService($repository, $emailService);

        $user = $service->createUser('Alice', 'alice@example.com');

        $this->assertNotNull($user);
        $this->assertEquals('Alice', $user->getName());
        $this->assertCount(1, $emailService->getSentEmails());
    }
}
```

### 集成测试

```php
class UserControllerTest extends TestCase
{
    public function testShowUser(): void
    {
        // 使用测试适配器
        $repository = new InMemoryUserRepository();
        $emailService = new MockEmailService();
        $service = new UserService($repository, $emailService);
        $controller = new UserController($service, new JsonResponder());

        $user = $service->createUser('Alice', 'alice@example.com');
        $response = $controller->show($user->getId()->getValue());

        $this->assertEquals(200, $response->getStatusCode());
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 完整的六边形架构示例

// 1. 端口定义
namespace App\Domain\Ports\Inbound;

interface UserServicePort
{
    public function createUser(string $name, string $email): User;
    public function getUserById(int $id): User;
}

namespace App\Domain\Ports\Outbound;

interface UserRepositoryPort
{
    public function save(User $user): void;
    public function findById(int $id): ?User;
}

interface EmailServicePort
{
    public function send(string $to, string $subject, string $body): void;
}

// 2. 核心实现
namespace App\Domain;

use App\Domain\Ports\Inbound\UserServicePort;
use App\Domain\Ports\Outbound\{UserRepositoryPort, EmailServicePort};

class UserService implements UserServicePort
{
    public function __construct(
        private UserRepositoryPort $userRepository,
        private EmailServicePort $emailService
    ) {
    }

    public function createUser(string $name, string $email): User
    {
        $user = new User(null, $name, new Email($email));
        $this->userRepository->save($user);
        $this->emailService->send($email, 'Welcome', "Welcome, {$name}!");
        return $user;
    }

    public function getUserById(int $id): User
    {
        $user = $this->userRepository->findById($id);
        if ($user === null) {
            throw new UserNotFoundException($id);
        }
        return $user;
    }
}

// 3. 适配器实现
namespace App\Adapters\Outbound\Database;

use App\Domain\Ports\Outbound\UserRepositoryPort;

class DatabaseUserRepository implements UserRepositoryPort
{
    public function __construct(private PDO $db) {}

    public function save(User $user): void
    {
        // 保存到数据库
    }

    public function findById(int $id): ?User
    {
        // 从数据库查找
        return null;
    }
}

namespace App\Adapters\Inbound\Http;

use App\Domain\Ports\Inbound\UserServicePort;

class UserController
{
    public function __construct(
        private UserServicePort $userService,
        private JsonResponder $responder
    ) {
    }

    public function show(int $id): Response
    {
        try {
            $user = $this->userService->getUserById($id);
            return $this->responder->respond($user->toArray());
        } catch (UserNotFoundException $e) {
            return $this->responder->error($e->getMessage(), 404);
        }
    }
}

// 4. 依赖注入配置
$container = new Container();
$container->bind(UserRepositoryPort::class, DatabaseUserRepository::class);
$container->bind(EmailServicePort::class, SmtpEmailService::class);
$container->singleton(UserServicePort::class, UserService::class);

// 5. 使用
$controller = $container->make(UserController::class);
$response = $controller->show(1);
```

## 最佳实践

### 1. 端口设计

- 端口应该反映业务需求，而不是技术实现。
- 使用领域语言定义端口接口。

```php
// 好的端口设计
interface UserRepositoryPort
{
    public function save(User $user): void;
    public function findById(UserId $id): ?User;
}

// 不好的端口设计（暴露技术细节）
interface UserRepositoryPort
{
    public function insert(array $data): void;
    public function selectById(int $id): ?array;
}
```

### 2. 适配器隔离

- 所有技术相关代码都在适配器中。
- 核心代码不应该知道适配器的存在。

### 3. 测试策略

- 使用测试适配器进行单元测试。
- 使用真实适配器进行集成测试。

### 4. 依赖方向

- 核心依赖端口（接口）。
- 适配器实现端口。
- 适配器依赖核心。

## 注意事项

1. **端口设计**：端口应该使用领域语言，反映业务需求。

2. **适配器隔离**：确保所有技术代码都在适配器中。

3. **依赖注入**：使用依赖注入管理适配器的创建和绑定。

4. **测试友好**：创建测试适配器，便于单元测试。

5. **演进路径**：六边形架构为未来拆分为微服务提供了清晰的路径。

## 练习

1. 实现一个完整的六边形架构应用，包含用户管理的完整流程。

2. 创建多个适配器（HTTP、CLI、测试），演示适配器的可替换性。

3. 实现一个依赖注入容器，管理端口和适配器的绑定。

4. 编写单元测试和集成测试，使用测试适配器。

5. 设计一个订单处理系统，使用六边形架构组织代码。
