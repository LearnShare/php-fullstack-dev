# 7.5.3 Adapters（适配器）

## 概述

适配器（Adapter）是六边形架构中连接端口与外部世界的桥梁。适配器负责将外部世界的请求转换为应用内部的格式，同时也将应用内部的响应转换为外部世界能够理解的格式。适配器的核心作用是实现技术的可替换性，使得应用核心业务逻辑与具体的技术实现解耦。

在六边形架构中，适配器分为两种类型：输入适配器（Driving Adapter）和输出适配器（Driven Adapter）。输入适配器接收外部请求并驱动应用执行操作，如 HTTP 控制器、命令行处理器、消息队列消费者等。输出适配器被应用核心调用来与外部世界交互，如数据库仓储、文件系统服务、外部 API 客户端等。

理解适配器的设计对于构建灵活、可维护的应用程序至关重要。当外部技术发生变化时，只需要更换相应的适配器，而不需要修改应用核心的业务逻辑。这种设计使得系统能够轻松适应技术演进、第三方服务更换等变化。

**主要内容**：
- 适配器的定义和作用
- 输入适配器的实现方法
- 输出适配器的实现方法
- 适配器与端口的协作
- 适配器的测试策略

## 特性

- **技术隔离**：适配器将具体技术实现与应用核心隔离，使得业务逻辑不依赖于特定技术
- **可替换性**：通过实现相同的端口接口，可以轻松替换不同的适配器实现
- **单向依赖**：适配器依赖端口，但端口和业务核心不依赖适配器
- **数据转换**：适配器负责外部数据格式与应用内部数据格式之间的转换
- **错误转换**：适配器将外部错误转换为应用内部能够处理的异常

## 核心概念

### 适配器类型

```
┌─────────────────────────────────────────────────────────────┐
│                        外部世界                              │
└─────────────────────────┬───────────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
   │ HTTP 适配器 │  │ CLI 适配器  │  │  消息队列   │
   │             │  │             │  │   适配器    │
   └──────┬──────┘  └──────┬──────┘  └──────┬──────┘
          │                │                │
          └────────┬───────┴────────┬───────┘
                   ▼                ▼
          ┌─────────────────┐  ┌─────────────────┐
          │   输入端口      │  │   输入端口      │
          │ (Driving Port) │  │ (Driving Port) │
          └────────┬────────┘  └────────┬────────┘
                   │                      │
                   └──────────┬───────────┘
                              ▼
                   ┌─────────────────┐
                   │    应用核心     │
                   │   (Application) │
                   └────────┬────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
   │   输出端口  │     │   输出端口   │     │   输出端口  │
   │(Driven Port)│     │(Driven Port) │     │(Driven Port)│
   └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
          │                   │                   │
          └────────┬──────────┴────────┬──────────┘
                   ▼                    ▼
   ┌─────────────────────┐  ┌─────────────────────┐
   │   数据库适配器      │  │  外部 API 适配器    │
   │  (MySQL/PostgreSQL) │  │  (REST/GraphQL)     │
   └─────────────────────┘  └─────────────────────┘
```

### 输入适配器

输入适配器也称为主动适配器（Driving Adapter），负责接收来自外部世界的请求并调用应用的业务逻辑。输入适配器通常包括 HTTP 控制器、命令行处理器、消息队列消费者、事件监听器等。

输入适配器的工作流程：首先接收外部请求（如 HTTP 请求），然后将请求数据转换为应用内部的数据结构（如命令对象或 DTO），接着调用输入端口（Use Case 或 Application Service）执行业务逻辑，最后将业务逻辑的执行结果转换为外部世界能够理解的格式（如 JSON 响应）并返回。

### 输出适配器

输出适配器也称为被动适配器（Driven Adapter），实现应用核心定义的输出端口，被业务逻辑调用来完成特定任务。输出适配器包括数据库仓储实现、文件系统服务、外部 API 客户端、消息队列生产者、邮件服务等。

输出适配器的工作流程：当业务逻辑需要与外部世界交互时，通过输出端口接口调用适配器，适配器负责将应用内部的数据格式转换为外部系统能够理解的格式，执行相应的操作，然后将操作结果或状态返回给应用核心。

## 语法/定义

### 输入适配器接口（以 HTTP 为例）

```php
<?php
declare(strict_types=1);

// 输入适配器基类接口
interface InputAdapterInterface
{
    public function handle(mixed $request): mixed;
}
```

### 输出适配器接口（以仓储为例）

```php
<?php
declare(strict_types=1);

// 输出适配器基类接口
interface OutputAdapterInterface
{
    public function execute(mixed $data): mixed;
}
```

## 基本用法

### 1. HTTP 输入适配器

HTTP 输入适配器是 Web 应用中最常见的输入适配器，负责接收 HTTP 请求并调用相应的业务逻辑。以下是一个完整的 HTTP 输入适配器实现示例。

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Input\Http;

use App\Port\Input\CreateUserUseCase;
use App\Port\Input\CreateUserCommand;
use App\Port\Input\GetUserUseCase;
use App\Port\Input\Dto\UserDto;

class UserController
{
    public function __construct(
        private readonly CreateUserUseCase $createUserUseCase,
        private readonly GetUserUseCase $getUserUseCase
    ) {}

    public function create(array $requestData): array
    {
        $command = new CreateUserCommand(
            name: $requestData['name'],
            email: $requestData['email'],
            phone: $requestData['phone'] ?? null
        );

        $user = $this->createUserUseCase->execute($command);

        return [
            'status' => 'success',
            'data' => [
                'id' => $user->getId(),
                'name' => $user->getName(),
                'email' => $user->getEmail(),
                'created_at' => $user->getCreatedAt()->format('Y-m-d H:i:s')
            ]
        ];
    }

    public function show(int $id): array
    {
        $user = $this->getUserUseCase->execute($id);

        if ($user === null) {
            return [
                'status' => 'error',
                'message' => 'User not found'
            ];
        }

        return [
            'status' => 'success',
            'data' => [
                'id' => $user->getId(),
                'name' => $user->getName(),
                'email' => $user->getEmail()
            ]
        ];
    }
}
```

在上述示例中，`UserController` 接收 HTTP 请求数据，将其转换为命令对象，然后调用相应的用例执行。响应被转换为标准的数组格式，包含状态码和数据部分。这种设计使得控制器只需要处理 HTTP 相关的逻辑，而业务逻辑完全由用例处理。

### 2. 命令行输入适配器

命令行适配器用于处理命令行参数和选项，常用于构建 CLI 工具、批量处理任务、定时任务等场景。

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Input\Cli;

use App\Port\Input\CreateUserUseCase;
use App\Port\Input\CreateUserCommand;

class UserCommandHandler
{
    public function __construct(
        private readonly CreateUserUseCase $createUserUseCase
    ) {}

    public function handle(array $args): void
    {
        $commandName = $args[1] ?? null;

        match ($commandName) {
            'create' => $this->handleCreate(array_slice($args, 2)),
            'list' => $this->handleList(array_slice($args, 2)),
            default => $this->showHelp()
        };
    }

    private function handleCreate(array $args): void
    {
        $options = $this->parseOptions($args);

        if (!isset($options['name']) || !isset($options['email'])) {
            echo "Error: --name and --email are required\n";
            return;
        }

        $command = new CreateUserCommand(
            name: $options['name'],
            email: $options['email'],
            phone: $options['phone'] ?? null
        );

        $user = $this->createUserUseCase->execute($command);

        echo "User created successfully!\n";
        echo "ID: {$user->getId()}\n";
        echo "Name: {$user->getName()}\n";
        echo "Email: {$user->getEmail()}\n";
    }

    private function handleList(array $args): void
    {
        echo "Listing users...\n";
    }

    private function parseOptions(array $args): array
    {
        $options = [];
        foreach ($args as $arg) {
            if (str_starts_with($arg, '--')) {
                $parts = explode('=', substr($arg, 2), 2);
                $options[$parts[0]] = $parts[1] ?? true;
            }
        }
        return $options;
    }

    private function showHelp(): void
    {
        echo "Usage: php command.php <command> [options]\n";
        echo "\nCommands:\n";
        echo "  create        Create a new user\n";
        echo "  list          List all users\n";
        echo "\nOptions:\n";
        echo "  --name=NAME   User name\n";
        echo "  --email=EMAIL User email\n";
        echo "  --phone=PHONE User phone\n";
    }
}
```

命令行适配器将命令行参数解析为应用内部的命令对象，然后调用相应的用例执行。这种设计使得命令行工具可以复用与 HTTP 接口相同的业务逻辑。

### 3. 数据库输出适配器

数据库适配器是最常见的输出适配器，负责将领域对象持久化到数据库，以及从数据库检索领域对象。

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\Database;

use App\Domain\Entity\User;
use App\Port\Output\UserRepositoryPort;
use PDO;

class UserRepository implements UserRepositoryPort
{
    public function __construct(
        private readonly PDO $pdo
    ) {}

    public function findById(int $id): ?User
    {
        $stmt = $this->pdo->prepare(
            'SELECT * FROM users WHERE id = :id'
        );
        $stmt->execute(['id' => $id]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);

        if ($row === false) {
            return null;
        }

        return $this->mapToEntity($row);
    }

    public function findByEmail(string $email): ?User
    {
        $stmt = $this->pdo->prepare(
            'SELECT * FROM users WHERE email = :email'
        );
        $stmt->execute(['email' => $email]);
        $row = $stmt->fetch(PDO::FETCH_ASSOC);

        if ($row === false) {
            return null;
        }

        return $this->mapToEntity($row);
    }

    public function findAll(int $page, int $limit, ?string $search): array
    {
        $offset = ($page - 1) * $limit;
        
        $sql = 'SELECT * FROM users';
        $params = [];
        
        if ($search !== null) {
            $sql .= ' WHERE name LIKE :search OR email LIKE :search';
            $params['search'] = "%{$search}%";
        }
        
        $sql .= ' LIMIT :limit OFFSET :offset';
        $params['limit'] = $limit;
        $params['offset'] = $offset;

        $stmt = $this->pdo->prepare($sql);
        $stmt->execute($params);
        $rows = $stmt->fetchAll(PDO::FETCH_ASSOC);

        return array_map([$this, 'mapToEntity'], $rows);
    }

    public function save(User $user): int
    {
        $stmt = $this->pdo->prepare(
            'INSERT INTO users (name, email, phone, created_at, updated_at) 
             VALUES (:name, :email, :phone, :created_at, :updated_at)'
        );

        $stmt->execute([
            'name' => $user->getName(),
            'email' => $user->getEmail(),
            'phone' => $user->getPhone(),
            'created_at' => $user->getCreatedAt()->format('Y-m-d H:i:s'),
            'updated_at' => $user->getUpdatedAt()->format('Y-m-d H:i:s')
        ]);

        return (int) $this->pdo->lastInsertId();
    }

    public function update(User $user): void
    {
        $stmt = $this->pdo->prepare(
            'UPDATE users 
             SET name = :name, email = :email, phone = :phone, updated_at = :updated_at 
             WHERE id = :id'
        );

        $stmt->execute([
            'id' => $user->getId(),
            'name' => $user->getName(),
            'email' => $user->getEmail(),
            'phone' => $user->getPhone(),
            'updated_at' => $user->getUpdatedAt()->format('Y-m-d H:i:s')
        ]);
    }

    public function delete(int $id): void
    {
        $stmt = $this->pdo->prepare('DELETE FROM users WHERE id = :id');
        $stmt->execute(['id' => $id]);
    }

    private function mapToEntity(array $row): User
    {
        return new User(
            id: (int) $row['id'],
            name: $row['name'],
            email: $row['email'],
            phone: $row['phone'] ?? null,
            createdAt: new DateTimeImmutable($row['created_at']),
            updatedAt: new DateTimeImmutable($row['updated_at'])
        );
    }
}
```

数据库适配器实现了 `UserRepositoryPort` 接口，将领域对象映射到数据库表结构。适配器负责 SQL 查询的构建和执行，以及数据库行与领域对象之间的相互转换。

### 4. 外部 API 输出适配器

外部 API 适配器用于与第三方服务集成，如支付网关、短信服务、邮件服务等。

```php
<?php
declare(strict_types=1);

namespace App\Adapter\Output\Api;

use App\Port\Output\PaymentGatewayPort;
use App\Port\Output\Dto\PaymentRequest;
use App\Port\Output\Dto\PaymentResponse;
use App\Port\Output\Dto\RefundRequest;
use App\Port\Output\Dto\RefundResponse;

class PaymentGatewayAdapter implements PaymentGatewayPort
{
    public function __construct(
        private readonly string $apiKey,
        private readonly string $baseUrl,
        private readonly HttpClient $httpClient
    ) {}

    public function charge(PaymentRequest $request): PaymentResponse
    {
        $response = $this->httpClient->post("{$this->baseUrl}/charges", [
            'headers' => [
                'Authorization' => "Bearer {$this->apiKey}",
                'Content-Type' => 'application/json'
            ],
            'json' => [
                'amount' => $request->getAmount(),
                'currency' => $request->getCurrency(),
                'description' => $request->getDescription(),
                'metadata' => $request->getMetadata()
            ]
        ]);

        $data = json_decode($response->getBody()->getContents(), true);

        return new PaymentResponse(
            transactionId: $data['id'],
            status: $data['status'],
            amount: $data['amount'],
            currency: $data['currency'],
            paidAt: new DateTimeImmutable($data['created_at'])
        );
    }

    public function refund(RefundRequest $request): RefundResponse
    {
        $response = $this->httpClient->post(
            "{$this->baseUrl}/refunds",
            [
                'headers' => [
                    'Authorization' => "Bearer {$this->apiKey}",
                    'Content-Type' => 'application/json'
                ],
                'json' => [
                    'charge' => $request->getTransactionId(),
                    'amount' => $request->getAmount()
                ]
            ]
        );

        $data = json_decode($response->getBody()->getContents(), true);

        return new RefundResponse(
            refundId: $data['id'],
            status: $data['status'],
            amount: $data['amount'],
            refundedAt: new DateTimeImmutable($data['created_at'])
        );
    }

    public function getTransactionStatus(string $transactionId): string
    {
        $response = $this->httpClient->get(
            "{$this->baseUrl}/charges/{$transactionId}",
            [
                'headers' => [
                    'Authorization' => "Bearer {$this->apiKey}"
                ]
            ]
        );

        $data = json_decode($response->getBody()->getContents(), true);

        return $data['status'];
    }
}
```

外部 API 适配器封装了与第三方支付网关的交互细节，将外部 API 的响应转换为应用内部的数据对象。这种设计使得业务逻辑不需要了解外部 API 的具体实现细节，只需要通过端口接口与适配器交互。

### 5. 完整的适配器组装示例

以下示例展示如何在应用入口将适配器与端口组装在一起。

```php
<?php
declare(strict_types=1);

namespace App;

use App\Adapter\Input\Http\UserController;
use App\Adapter\Output\Database\UserRepository;
use App\Adapter\Output\Api\PaymentGatewayAdapter;
use App\Port\Input\CreateUserUseCase;
use App\Port\Input\GetUserUseCase;
use App\Port\Output\UserRepositoryPort;
use App\Port\Output\PaymentGatewayPort;
use App\Application\UserService;

class Container
{
    private array $services = [];

    public function __construct(
        private readonly array $config
    ) {}

    public function getUserController(): UserController
    {
        return new UserController(
            $this->getCreateUserUseCase(),
            $this->getGetUserUseCase()
        );
    }

    public function getUserRepository(): UserRepositoryPort
    {
        if (!isset($this->services['userRepository'])) {
            $this->services['userRepository'] = new UserRepository(
                new PDO($this->config['database']['dsn'])
            );
        }
        return $this->services['userRepository'];
    }

    public function getPaymentGateway(): PaymentGatewayPort
    {
        if (!isset($this->services['paymentGateway'])) {
            $this->services['paymentGateway'] = new PaymentGatewayAdapter(
                apiKey: $this->config['payment']['api_key'],
                baseUrl: $this->config['payment']['base_url'],
                httpClient: new GuzzleHttpClient()
            );
        }
        return $this->services['paymentGateway'];
    }

    private function getCreateUserUseCase(): CreateUserUseCase
    {
        return new CreateUserUseCase(
            $this->getUserRepository()
        );
    }

    private function getGetUserUseCase(): GetUserUseCase
    {
        return new GetUserUseCase(
            $this->getUserRepository()
        );
    }
}
```

容器负责创建和组装所有的适配器和用例。通过依赖注入，适配器被注入到用例中，用例被注入到控制器中。这种组装方式使得各层之间的依赖关系清晰明确，便于测试和维护。

## 使用场景

### 1. 技术切换场景

当需要更换数据库从 MySQL 切换到 PostgreSQL 时，只需要创建一个新的 PostgreSQL 适配器实现相同的端口接口，而不需要修改任何业务逻辑代码。这种场景在技术升级、供应商更换等情况下非常常见。

### 2. 多端适配场景

同一个应用可能需要支持 Web、移动端、CLI 等多种客户端。每种客户端使用不同的输入适配器，但共享相同的业务逻辑和输出适配器。这种设计避免了业务逻辑的重复实现。

### 3. 第三方服务集成场景

应用需要集成多个第三方服务，如支付网关、短信服务、地图服务等。通过为每个服务创建专门的适配器，可以统一管理外部依赖，降低耦合度。

### 4. 测试场景

在单元测试中，可以使用内存适配器替代真实的数据库适配器，使得测试不依赖外部环境，运行更快、更稳定。在集成测试中，可以使用测试适配器模拟外部系统的响应。

## 注意事项

### 1. 适配器职责边界

适配器应该只负责数据格式转换和技术交互，不应该包含业务逻辑。业务逻辑应该放在用例或领域服务中。如果适配器中出现了业务逻辑，说明设计可能存在问题。

### 2. 错误处理策略

适配器应该将外部系统的错误转换为应用内部的异常。例如，当数据库连接失败时，适配器应该抛出适当的异常（如 `DatabaseConnectionException`），而不是让原始的 PDO 异常传播到业务逻辑层。

### 3. 数据转换准确性

适配器负责数据格式的转换，必须确保转换的准确性。特别是在类型转换、日期格式、字符编码等方面要格外注意。建议在适配器中添加数据验证逻辑。

### 4. 性能考虑

适配器可能成为性能瓶颈，特别是在外部 API 调用场景中。建议实现适当的缓存策略、连接池管理、重试机制等。同时要注意异步处理的使用场景。

### 5. 日志记录

适配器是与外部世界交互的边界点，应该添加适当的日志记录。记录请求参数、响应状态、错误信息等，有助于问题排查和系统监控。

## 常见问题

### Q1: 适配器和端口的区别是什么？

端口是应用内部定义的接口规范，规定了业务核心与外部世界交互的方式。适配器是端口的具体实现，将端口接口转换为具体技术的调用。简单来说，端口是"契约"，适配器是"实现"。

### Q2: 一个端口可以有多个适配器吗？

是的，一个端口可以有多个适配器实现。例如，`UserRepositoryPort` 可以有 MySQL 适配器、PostgreSQL 适配器、内存适配器（用于测试）等。根据运行环境的不同，可以切换不同的适配器实现。

### Q3: 适配器可以依赖其他适配器吗？

原则上不推荐适配器之间相互依赖。如果需要适配器之间的协作，应该通过端口接口进行解耦。但在某些场景下，适配器可能需要依赖其他输出端口，这时可以通过依赖注入的方式解决。

### Q4: 如何选择适配器的粒度？

适配器的粒度应该根据具体场景决定。粒度过细会导致适配器数量过多，增加维护成本；粒度过粗会导致适配器职责过多，难以复用。一般建议一个适配器对应一个外部系统或技术。

## 最佳实践

### 1. 遵循依赖规则

适配器依赖端口，但端口和业务核心不应该依赖适配器。在代码中，可以通过依赖注入容器或手动注入的方式确保依赖方向的正确性。

### 2. 使用工厂模式创建适配器

对于复杂的适配器，可以使用工厂模式封装创建逻辑。工厂可以根据配置或环境变量决定创建哪种适配器实现。

### 3. 实现适配器基类

可以为同类型的适配器创建基类，减少重复代码。例如，所有数据库适配器可以继承一个抽象的 `AbstractRepository` 类，实现通用的查询构建逻辑。

### 4. 添加适配器日志

在适配器中添加请求日志和响应日志，记录关键的交互信息。这对于调试和监控非常有帮助。建议使用结构化日志格式，便于后续分析。

### 5. 实现超时和重试机制

对于外部 API 适配器，必须实现请求超时和重试机制。网络请求可能因为各种原因失败，适当的重试可以提高系统的可靠性。

### 6. 使用接口隔离

在为外部系统创建适配器时，应该定义清晰的接口，避免将外部系统的所有方法都暴露出来。只暴露应用需要的方法，保持接口的简洁和稳定。

## 练习任务

### 练习 1：实现文件存储适配器

设计并实现一个文件存储适配器，实现 `StoragePort` 接口。该适配器需要支持文件上传、文件下载、文件删除、文件列表等操作。使用本地文件系统作为存储后端。

### 练习 2：实现消息队列适配器

设计并实现一个消息队列生产者适配器，实现 `MessageQueuePort` 接口。该适配器需要支持发送消息到队列、订阅消息等操作。使用 RabbitMQ 或其他消息队列作为后端。

### 练习 3：实现缓存适配器

设计并实现一个缓存适配器，实现 `CachePort` 接口。该适配器需要支持缓存的设置、获取、删除等操作。使用 Redis 作为缓存后端。

### 练习 4：创建适配器工厂

创建一个适配器工厂类，根据配置或环境变量动态创建不同类型的适配器。例如，根据数据库配置创建 MySQL 或 PostgreSQL 适配器。

### 练习 5：编写适配器单元测试

为实现的文件存储适配器编写单元测试。使用 mock 或内存实现替代真实的文件系统，确保测试不依赖外部环境。
