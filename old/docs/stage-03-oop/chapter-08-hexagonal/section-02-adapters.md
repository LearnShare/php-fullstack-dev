# 3.8.2 Adapters（适配器）

## 概述

适配器（Adapters）实现端口，连接外部世界。输入适配器将外部请求转换为领域调用，输出适配器实现领域层定义的接口。

## 输入适配器（Inbound Adapters）

### HTTP 适配器

- 实现输入端口，将外部请求转换为领域调用。

```php
namespace App\Adapters\Inbound\Http;

use App\Domain\Ports\Inbound\UserServicePort;
use App\Responders\JsonResponder;

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

    public function create(Request $request): Response
    {
        try {
            $user = $this->userService->createUser(
                $request->get('name'),
                $request->get('email')
            );
            return $this->responder->respond($user->toArray(), 201);
        } catch (ValidationException $e) {
            return $this->responder->error($e->getErrors(), 422);
        }
    }
}
```

### CLI 适配器

```php
namespace App\Adapters\Inbound\CLI;

use App\Domain\Ports\Inbound\UserServicePort;

class UserCommand
{
    public function __construct(
        private UserServicePort $userService
    ) {
    }

    public function createUser(string $name, string $email): void
    {
        try {
            $user = $this->userService->createUser($name, $email);
            echo "User created: {$user->getId()}\n";
        } catch (Exception $e) {
            echo "Error: {$e->getMessage()}\n";
            exit(1);
        }
    }
}
```

## 输出适配器（Outbound Adapters）

### 数据库适配器

```php
namespace App\Adapters\Outbound\Database;

use App\Domain\Ports\Outbound\UserRepositoryPort;
use App\Domain\User;

class DatabaseUserRepository implements UserRepositoryPort
{
    public function __construct(
        private PDO $db
    ) {
    }

    public function save(User $user): void
    {
        $stmt = $this->db->prepare(
            'INSERT INTO users (name, email) VALUES (?, ?)'
        );
        $stmt->execute([$user->getName(), $user->getEmail()->getValue()]);
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
        return new User(
            new UserId($data['id']),
            new Email($data['email']),
            $data['name']
        );
    }
}
```

### 邮件服务适配器

```php
namespace App\Adapters\Outbound\Email;

use App\Domain\Ports\Outbound\EmailServicePort;

class SmtpEmailService implements EmailServicePort
{
    public function __construct(
        private string $smtpHost,
        private int $smtpPort
    ) {
    }

    public function send(string $to, string $subject, string $body): void
    {
        // 使用 SMTP 发送邮件
        mail($to, $subject, $body);
    }
}

class SendGridEmailService implements EmailServicePort
{
    public function __construct(
        private string $apiKey
    ) {
    }

    public function send(string $to, string $subject, string $body): void
    {
        // 使用 SendGrid API 发送邮件
        // ...
    }
}
```

### 测试适配器

```php
namespace App\Adapters\Outbound\Test;

use App\Domain\Ports\Outbound\{UserRepositoryPort, EmailServicePort};

class InMemoryUserRepository implements UserRepositoryPort
{
    private array $users = [];

    public function save(User $user): void
    {
        $this->users[$user->getId()->getValue()] = $user;
    }

    public function findById(int $id): ?User
    {
        return $this->users[$id] ?? null;
    }

    public function findByEmail(string $email): ?User
    {
        foreach ($this->users as $user) {
            if ($user->getEmail()->getValue() === $email) {
                return $user;
            }
        }
        return null;
    }
}

class MockEmailService implements EmailServicePort
{
    private array $sentEmails = [];

    public function send(string $to, string $subject, string $body): void
    {
        $this->sentEmails[] = [
            'to' => $to,
            'subject' => $subject,
            'body' => $body
        ];
    }

    public function getSentEmails(): array
    {
        return $this->sentEmails;
    }
}
```

## 依赖注入配置

### 容器配置

```php
class Container
{
    public function configure(): void
    {
        // 端口实现绑定
        $this->bind(UserRepositoryPort::class, DatabaseUserRepository::class);
        $this->bind(EmailServicePort::class, SmtpEmailService::class);
        
        // 核心服务
        $this->bind(UserServicePort::class, UserService::class);
        
        // 适配器
        $this->bind(UserController::class, UserController::class);
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

// Ports
namespace App\Domain\Ports\Outbound;

interface UserRepositoryPort
{
    public function save(User $user): void;
    public function findById(int $id): ?User;
}

// Adapters
namespace App\Adapters\Outbound\Database;

use App\Domain\Ports\Outbound\UserRepositoryPort;
use App\Domain\User;

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

namespace App\Adapters\Outbound\Test;

use App\Domain\Ports\Outbound\UserRepositoryPort;
use App\Domain\User;

class InMemoryUserRepository implements UserRepositoryPort
{
    private array $users = [];

    public function save(User $user): void
    {
        $this->users[$user->getId()->getValue()] = $user;
    }

    public function findById(int $id): ?User
    {
        return $this->users[$id] ?? null;
    }
}

// 使用
// 生产环境
$repository = new DatabaseUserRepository($pdo);

// 测试环境
$repository = new InMemoryUserRepository();
```

## 注意事项

1. **适配器实现**：适配器实现端口接口，提供具体的技术实现。

2. **可替换性**：可以轻松替换适配器，不影响核心逻辑。

3. **测试适配器**：创建测试用的适配器，便于单元测试。

4. **依赖注入**：使用依赖注入容器管理适配器的创建和绑定。

5. **技术隔离**：适配器包含所有技术相关代码，核心保持纯净。

## 练习

1. 实现一个数据库适配器，实现用户仓储端口。

2. 创建一个邮件服务适配器，支持多种邮件服务提供商。

3. 实现测试适配器，用于单元测试。

4. 创建一个 HTTP 适配器，将 HTTP 请求转换为领域调用。

5. 设计一个依赖注入配置，绑定端口和适配器。
