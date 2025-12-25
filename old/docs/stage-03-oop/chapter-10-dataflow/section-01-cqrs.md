# 3.10.1 CQRS 模式

## 概述

CQRS（Command Query Responsibility Segregation）将数据操作分为命令（Command）和查询（Query），实现读写分离，提高系统的可扩展性和性能。

## CQRS 核心概念

### 基本定义

CQRS（Command Query Responsibility Segregation）将数据操作分为：

- **Command（命令）**：修改状态的操作，不返回数据。
- **Query（查询）**：读取数据的操作，不修改状态。

```
┌─────────────┐
│   Command   │ → 写入模型 → 数据库（写）
└─────────────┘

┌─────────────┐
│    Query    │ → 读取模型 → 数据库（读）
└─────────────┘
```

### 优势

- **读写分离**：可以独立优化读写操作。
- **可扩展性**：可以独立扩展读写模型。
- **性能优化**：读取模型可以使用缓存、只读数据库等。

## Command 实现

### Command 接口

```php
namespace App\Application\Commands;

interface Command
{
}

interface CommandHandler
{
    public function handle(Command $command): void;
}

class CreateUserCommand implements Command
{
    public function __construct(
        public readonly string $name,
        public readonly string $email
    ) {
    }
}

class CreateUserCommandHandler implements CommandHandler
{
    public function __construct(
        private UserRepository $userRepository
    ) {
    }

    public function handle(Command $command): void
    {
        if (!$command instanceof CreateUserCommand) {
            throw new InvalidArgumentException('Invalid command type');
        }

        $email = new Email($command->email);
        
        if ($this->userRepository->existsByEmail($email)) {
            throw new EmailAlreadyExistsException($command->email);
        }

        $user = new User(null, $command->name, $email);
        $this->userRepository->save($user);
    }
}
```

## Query 实现

### Query 接口

```php
namespace App\Application\Queries;

interface Query
{
}

interface QueryHandler
{
    public function handle(Query $query): mixed;
}

class GetUserQuery implements Query
{
    public function __construct(
        public readonly int $userId
    ) {
    }
}

class GetUserQueryHandler implements QueryHandler
{
    public function __construct(
        private UserRepository $userRepository
    ) {
    }

    public function handle(Query $query): ?UserDTO
    {
        if (!$query instanceof GetUserQuery) {
            throw new InvalidArgumentException('Invalid query type');
        }

        $user = $this->userRepository->findById($query->userId);
        
        if ($user === null) {
            return null;
        }

        return new UserDTO(
            $user->getId()->getValue(),
            $user->getName(),
            $user->getEmail()->getValue()
        );
    }
}

class UserDTO
{
    public function __construct(
        public readonly int $id,
        public readonly string $name,
        public readonly string $email
    ) {
    }
}
```

## Command/Query 总线

### CommandBus

```php
namespace App\Application;

class CommandBus
{
    private array $handlers = [];

    public function register(string $commandClass, CommandHandler $handler): void
    {
        $this->handlers[$commandClass] = $handler;
    }

    public function dispatch(Command $command): void
    {
        $commandClass = get_class($command);
        
        if (!isset($this->handlers[$commandClass])) {
            throw new HandlerNotFoundException("No handler for {$commandClass}");
        }

        $this->handlers[$commandClass]->handle($command);
    }
}
```

### QueryBus

```php
class QueryBus
{
    private array $handlers = [];

    public function register(string $queryClass, QueryHandler $handler): void
    {
        $this->handlers[$queryClass] = $handler;
    }

    public function dispatch(Query $query): mixed
    {
        $queryClass = get_class($query);
        
        if (!isset($this->handlers[$queryClass])) {
            throw new HandlerNotFoundException("No handler for {$queryClass}");
        }

        return $this->handlers[$queryClass]->handle($query);
    }
}
```

## 使用示例

### 注册和使用

```php
// 注册处理器
$commandBus = new CommandBus();
$commandBus->register(CreateUserCommand::class, new CreateUserCommandHandler($userRepository));

$queryBus = new QueryBus();
$queryBus->register(GetUserQuery::class, new GetUserQueryHandler($userRepository));

// 执行命令
$commandBus->dispatch(new CreateUserCommand('Alice', 'alice@example.com'));

// 执行查询
$user = $queryBus->dispatch(new GetUserQuery(1));
```

## 完整示例

```php
<?php
declare(strict_types=1);

namespace App\Application\Commands;

class UpdateUserCommand implements Command
{
    public function __construct(
        public readonly int $userId,
        public readonly string $name,
        public readonly string $email
    ) {
    }
}

class UpdateUserCommandHandler implements CommandHandler
{
    public function __construct(
        private UserRepository $userRepository
    ) {
    }

    public function handle(Command $command): void
    {
        if (!$command instanceof UpdateUserCommand) {
            throw new InvalidArgumentException('Invalid command type');
        }

        $user = $this->userRepository->findById($command->userId);
        if ($user === null) {
            throw new UserNotFoundException($command->userId);
        }

        $user->changeName($command->name);
        $user->changeEmail(new Email($command->email));
        
        $this->userRepository->save($user);
    }
}

namespace App\Application\Queries;

class GetUserListQuery implements Query
{
    public function __construct(
        public readonly int $page = 1,
        public readonly int $perPage = 10
    ) {
    }
}

class GetUserListQueryHandler implements QueryHandler
{
    public function __construct(
        private UserReadRepository $readRepository
    ) {
    }

    public function handle(Query $query): array
    {
        if (!$query instanceof GetUserListQuery) {
            throw new InvalidArgumentException('Invalid query type');
        }

        return $this->readRepository->getList($query->page, $query->perPage);
    }
}
```

## 注意事项

1. **读写分离**：Command 和 Query 使用不同的模型和存储。

2. **最终一致性**：写入和读取之间可能存在延迟。

3. **复杂度**：CQRS 增加了系统复杂度，只在必要时使用。

4. **适用场景**：高并发读写、需要独立优化读写性能的场景。

5. **简单场景**：对于简单的 CRUD 应用，CQRS 可能是过度设计。

## 练习

1. 实现一个完整的 CQRS 系统，包含 Command 和 Query 总线。

2. 创建一个用户管理的 CQRS 应用，包含创建、更新、查询命令。

3. 实现读写分离，使用不同的仓储处理命令和查询。

4. 设计一个订单处理的 CQRS 系统。

5. 创建一个支持中间件的 CommandBus，支持日志、事务等功能。
