# 3.8 六边形架构

## 目标

- 理解六边形架构（Hexagonal Architecture / Ports & Adapters）的核心概念。
- 掌握 Port（端口）和 Adapter（适配器）的设计原则。
- 理解如何通过六边形架构实现业务逻辑与技术实现的解耦。
- 能够在实际项目中应用六边形架构，设计可测试、可扩展的应用。

## 六边形架构概述

### 核心思想

六边形架构（也称为 Ports & Adapters）将应用分为：

- **核心（Core）**：包含业务逻辑，不依赖外部技术。
- **端口（Ports）**：定义接口，描述应用需要什么功能。
- **适配器（Adapters）**：实现端口，连接外部世界。

```
         ┌─────────────┐
         │   Adapters  │
         │  (外部世界)  │
         └──────┬──────┘
                │
         ┌──────▼──────┐
         │   Ports     │
         │  (接口定义)  │
         └──────┬──────┘
                │
         ┌──────▼──────┐
         │    Core     │
         │  (业务逻辑)  │
         └─────────────┘
```

### 为什么需要六边形架构

- **技术无关性**：业务逻辑不依赖数据库、Web 框架等具体技术。
- **可测试性**：可以轻松替换适配器进行测试。
- **可扩展性**：添加新的适配器不影响核心逻辑。

## Ports（端口）

### 输入端口（Inbound Ports）

- 定义应用如何被外部调用。
- 通常表现为接口或抽象类。

```php
namespace App\Domain\Ports\Inbound;

use App\Domain\User;

interface UserServicePort
{
    public function createUser(string $name, string $email): User;
    public function getUserById(int $id): User;
    public function updateUser(int $id, string $name): void;
}
```

### 输出端口（Outbound Ports）

- 定义应用需要什么外部服务。
- 描述应用对外部世界的需求。

```php
namespace App\Domain\Ports\Outbound;

use App\Domain\User;

interface UserRepositoryPort
{
    public function save(User $user): void;
    public function findById(int $id): ?User;
    public function findByEmail(string $email): ?User;
}

interface EmailServicePort
{
    public function send(string $to, string $subject, string $body): void;
}

interface NotificationPort
{
    public function notify(string $message): void;
}
```

## Adapters（适配器）

### 输入适配器（Inbound Adapters）

- 实现输入端口，将外部请求转换为领域调用。

```php
namespace App\Adapters\Inbound\Http;

use App\Domain\Ports\Inbound\UserServicePort;

class UserController
{
    public function __construct(
        private UserServicePort $userService
    ) {
    }

    public function create(Request $request): Response
    {
        try {
            $user = $this->userService->createUser(
                $request->get('name'),
                $request->get('email')
            );

            return new JsonResponse([
                'id' => $user->getId(),
                'name' => $user->getName(),
            ], 201);
        } catch (Exception $e) {
            return new JsonResponse(['error' => $e->getMessage()], 400);
        }
    }
}
```

### 输出适配器（Outbound Adapters）

- 实现输出端口，连接具体的技术实现。

```php
namespace App\Adapters\Outbound\Persistence;

use App\Domain\Ports\Outbound\UserRepositoryPort;
use App\Domain\User;
use PDO;

class DatabaseUserRepository implements UserRepositoryPort
{
    public function __construct(private PDO $db)
    {
    }

    public function save(User $user): void
    {
        $stmt = $this->db->prepare(
            'INSERT INTO users (name, email) VALUES (?, ?)'
        );
        $stmt->execute([$user->getName(), $user->getEmail()]);
    }

    public function findById(int $id): ?User
    {
        $stmt = $this->db->prepare('SELECT * FROM users WHERE id = ?');
        $stmt->execute([$id]);
        $data = $stmt->fetch(PDO::FETCH_ASSOC);

        return $data ? $this->mapToUser($data) : null;
    }

    public function findByEmail(string $email): ?User
    {
        $stmt = $this->db->prepare('SELECT * FROM users WHERE email = ?');
        $stmt->execute([$email]);
        $data = $stmt->fetch(PDO::FETCH_ASSOC);

        return $data ? $this->mapToUser($data) : null;
    }

    private function mapToUser(array $data): User
    {
        return new User($data['id'], $data['name'], $data['email']);
    }
}
```

### 测试适配器（Test Adapters）

- 用于测试的适配器实现。

```php
namespace App\Adapters\Outbound\Persistence\Test;

use App\Domain\Ports\Outbound\UserRepositoryPort;
use App\Domain\User;

class InMemoryUserRepository implements UserRepositoryPort
{
    private array $users = [];

    public function save(User $user): void
    {
        $this->users[$user->getId()] = $user;
    }

    public function findById(int $id): ?User
    {
        return $this->users[$id] ?? null;
    }

    public function findByEmail(string $email): ?User
    {
        foreach ($this->users as $user) {
            if ($user->getEmail() === $email) {
                return $user;
            }
        }
        return null;
    }
}
```

## 核心领域（Core Domain）

### 领域实体

```php
namespace App\Domain;

class User
{
    public function __construct(
        private ?int $id,
        private string $name,
        private Email $email
    ) {
        if (empty($name)) {
            throw new InvalidArgumentException('Name cannot be empty');
        }
    }

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getName(): string
    {
        return $this->name;
    }

    public function getEmail(): Email
    {
        return $this->email;
    }

    public function changeEmail(Email $newEmail): void
    {
        $this->email = $newEmail;
    }
}
```

### 领域服务

```php
namespace App\Domain;

use App\Domain\Ports\Outbound\UserRepositoryPort;
use App\Domain\Ports\Outbound\EmailServicePort;

class UserService implements UserServicePort
{
    public function __construct(
        private UserRepositoryPort $userRepository,
        private EmailServicePort $emailService
    ) {
    }

    public function createUser(string $name, string $email): User
    {
        $emailObj = new Email($email);

        // 检查邮箱是否已存在
        if ($this->userRepository->findByEmail($emailObj->getValue()) !== null) {
            throw new EmailAlreadyExistsException($email);
        }

        $user = new User(null, $name, $emailObj);
        $this->userRepository->save($user);

        // 发送欢迎邮件
        $this->emailService->send(
            $emailObj->getValue(),
            'Welcome',
            "Hello {$name}, welcome to our platform!"
        );

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

    public function updateUser(int $id, string $name): void
    {
        $user = $this->getUserById($id);
        // 业务逻辑：更新用户名称
        $user = new User($user->getId(), $name, $user->getEmail());
        $this->userRepository->save($user);
    }
}
```

## 完整示例

### 目录结构

```
src/
├── Domain/                    # 核心领域
│   ├── User.php
│   ├── Email.php
│   └── Ports/
│       ├── Inbound/
│       │   └── UserServicePort.php
│       └── Outbound/
│           ├── UserRepositoryPort.php
│           └── EmailServicePort.php
├── Adapters/
│   ├── Inbound/              # 输入适配器
│   │   └── Http/
│   │       └── UserController.php
│   └── Outbound/              # 输出适配器
│       ├── Persistence/
│       │   └── DatabaseUserRepository.php
│       └── Email/
│           └── SmtpEmailService.php
└── Infrastructure/            # 基础设施配置
    └── Container.php
```

### 依赖注入配置

```php
namespace App\Infrastructure;

use App\Domain\Ports\Inbound\UserServicePort;
use App\Domain\Ports\Outbound\UserRepositoryPort;
use App\Domain\Ports\Outbound\EmailServicePort;
use App\Domain\UserService;
use App\Adapters\Outbound\Persistence\DatabaseUserRepository;
use App\Adapters\Outbound\Email\SmtpEmailService;

class Container
{
    private array $bindings = [];

    public function __construct()
    {
        // 绑定端口到适配器
        $this->bind(UserRepositoryPort::class, DatabaseUserRepository::class);
        $this->bind(EmailServicePort::class, SmtpEmailService::class);
        
        // 绑定服务
        $this->bind(UserServicePort::class, UserService::class);
    }

    public function bind(string $interface, string $implementation): void
    {
        $this->bindings[$interface] = $implementation;
    }

    public function make(string $interface): object
    {
        $class = $this->bindings[$interface] ?? $interface;
        return new $class(...$this->resolveDependencies($class));
    }

    private function resolveDependencies(string $class): array
    {
        $reflection = new \ReflectionClass($class);
        $constructor = $reflection->getConstructor();
        
        if ($constructor === null) {
            return [];
        }

        $dependencies = [];
        foreach ($constructor->getParameters() as $parameter) {
            $type = $parameter->getType();
            if ($type instanceof \ReflectionNamedType) {
                $dependencies[] = $this->make($type->getName());
            }
        }

        return $dependencies;
    }
}
```

## 测试策略

### 使用测试适配器

```php
use App\Domain\UserService;
use App\Adapters\Outbound\Persistence\Test\InMemoryUserRepository;
use App\Adapters\Outbound\Email\Test\FakeEmailService;

class UserServiceTest
{
    public function testCreateUser(): void
    {
        // 使用测试适配器
        $repository = new InMemoryUserRepository();
        $emailService = new FakeEmailService();
        
        $userService = new UserService($repository, $emailService);
        
        $user = $userService->createUser('Alice', 'alice@example.com');
        
        $this->assertEquals('Alice', $user->getName());
        $this->assertTrue($emailService->wasEmailSent());
    }
}
```

## 六边形架构的优势

### 1. 技术无关性

- 核心业务逻辑不依赖具体技术实现。
- 可以轻松切换数据库、框架、外部服务。

### 2. 可测试性

- 使用测试适配器，无需真实的外部依赖。
- 可以快速运行单元测试。

### 3. 可扩展性

- 添加新的适配器不影响核心逻辑。
- 支持多种输入/输出方式。

### 4. 清晰的边界

- 端口定义了清晰的接口边界。
- 适配器负责技术细节。

## 最佳实践

### 1. 端口定义在领域层

- 端口是领域的一部分，定义在 `Domain/Ports` 中。

### 2. 适配器实现技术细节

- 适配器在 `Adapters` 目录中，实现具体技术。

### 3. 依赖方向

- 适配器依赖端口，核心不依赖适配器。

### 4. 使用依赖注入

- 通过容器管理依赖，实现松耦合。

## 练习

1. 设计一个订单处理系统，使用六边形架构，定义输入/输出端口，实现 HTTP 和 CLI 两种输入适配器。

2. 创建一个用户认证系统，定义 `AuthenticationPort` 和 `TokenStoragePort`，实现数据库和 Redis 两种存储适配器。

3. 实现一个支付处理系统，核心逻辑不依赖具体的支付网关，通过端口定义支付接口，实现多种支付适配器（Stripe、PayPal）。

4. 设计一个文件上传系统，定义 `FileStoragePort`，实现本地存储和云存储（S3）两种适配器。

5. 创建一个事件发布系统，定义 `EventPublisherPort`，实现同步和异步（消息队列）两种适配器。

6. 实现一个配置管理系统，核心逻辑通过端口读取配置，实现文件、数据库、环境变量三种配置适配器。
