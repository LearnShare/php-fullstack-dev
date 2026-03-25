# 7.3.3 模块化单体架构

## 概述

模块化单体（Modular Monolith）架构是一种介于传统单体应用和微服务架构之间的折中方案。它保留了单体应用的部署简单性，同时通过模块化设计实现了代码的清晰组织和一定程度的解耦。

模块化单体的核心思想是：在单个应用中创建清晰定义的模块边界，每个模块有自己的职责、数据和接口。模块之间通过定义良好的接口进行通信，而不是直接相互调用内部实现。这种架构为未来的微服务拆分奠定了基础。

在 PHP 开发中，模块化单体是一种非常实用的架构选择，特别适合中小型项目或团队。本节将详细介绍模块化单体的概念、模块划分方法、模块通信方式和实现细节。

**主要内容**：
- 模块化单体的定义和特点
- 模块划分原则和方法
- 模块通信方式
- 模块边界和依赖管理
- 模块化单体的演进路径

## 模块化单体详解

### 什么是模块化单体

模块化单体是一个单一的部署单元，但内部被划分为多个独立的模块。每个模块：
- 有明确的职责边界
- 有自己的数据和业务逻辑
- 通过定义良好的接口与其他模块通信
- 可以独立开发、测试

### 与传统单体的对比

| 特性 | 传统单体 | 模块化单体 |
|:-----|:---------|:-----------|
| 代码组织 | 随意 | 按模块划分 |
| 数据存储 | 共享数据库 | 可以独立 |
| 模块通信 | 直接调用 | 通过接口 |
| 可测试性 | 较低 | 较高 |
| 演进性 | 困难 | 较易 |

### 与微服务的对比

| 特性 | 模块化单体 | 微服务 |
|:-----|:-----------|:-------|
| 部署 | 单一部署 | 独立部署 |
| 进程 | 单进程 | 多进程 |
| 通信 | 进程内调用 | 网络调用 |
| 复杂度 | 较低 | 较高 |

## 模块划分原则

### 业务能力划分

按业务能力划分模块是最常见的方法：

```php
<?php
declare(strict_types=1);

// 应用根命名空间
namespace App;

// 用户模块
namespace App\Modules\User;

// 用户实体
class User
{
    public function __construct(
        public readonly int $id,
        public readonly string $name,
        public readonly string $email
    ) {}
}

// 用户仓储
interface UserRepository
{
    public function findById(int $id): ?User;
    public function save(User $user): void;
}

// 模块内接口（对外暴露）
interface UserServiceInterface
{
    public function getUser(int $id): ?User;
    public function createUser(string $name, string $email): User;
}

// 模块内服务
class UserService implements UserServiceInterface
{
    public function __construct(private UserRepository $repository) {}
    
    public function getUser(int $id): ?User
    {
        return $this->repository->findById($id);
    }
    
    public function createUser(string $name, string $email): User
    {
        $user = new User(0, $name, $email);
        $this->repository->save($user);
        return $user;
    }
}

// 订单模块
namespace App\Modules\Order;

// 订单实体
class Order
{
    public function __construct(
        public readonly int $id,
        public readonly int $userId,
        public readonly float $total
    ) {}
}

// 订单仓储
interface OrderRepository
{
    public function findByUserId(int $userId): array;
    public function save(Order $order): void;
}

// 订单服务
interface OrderServiceInterface
{
    public function getUserOrders(int $userId): array;
    public function createOrder(int $userId, float $total): Order;
}

class OrderService implements OrderServiceInterface
{
    public function __construct(
        private OrderRepository $repository,
        private UserServiceInterface $userService  // 依赖其他模块的接口
    ) {}
    
    public function getUserOrders(int $userId): array
    {
        // 可以验证用户存在
        $user = $this->userService->getUser($userId);
        if ($user === null) {
            throw new \InvalidArgumentException("User not found");
        }
        
        return $this->repository->findByUserId($userId);
    }
    
    public function createOrder(int $userId, float $total): Order
    {
        $user = $this->userService->getUser($userId);
        if ($user === null) {
            throw new \InvalidArgumentException("User not found");
        }
        
        $order = new Order(0, $userId, $total);
        $this->repository->save($order);
        
        return $order;
    }
}
```

### 按子域划分

在领域驱动设计中，可以按子域（Subdomain）划分模块：

```php
<?php
declare(strict_types=1);

// 核心域模块
namespace App\Modules\Order\Core;

// 支撑域模块
namespace App\Modules\Payment\Core;

// 通用域模块
namespace App\Modules\Common;

// 基础设施模块
namespace App\Modules\Infrastructure;
```

## 模块通信方式

### 方式 1：直接依赖接口

模块之间通过接口进行依赖，这是最直接的方式：

```php
<?php
declare(strict_types=1);

// 订单模块依赖用户模块
namespace App\Modules\Order;

class OrderService
{
    public function __construct(
        private OrderRepository $repository,
        private \App\Modules\User\Services\UserServiceInterface $userService  // 通过接口依赖
    ) {}
}
```

### 方式 2：事件驱动通信

模块之间通过事件进行松耦合通信：

```php
<?php
declare(strict_types=1);

// 事件系统
class EventDispatcher
{
    private array $listeners = [];
    
    public function dispatch(object $event): void
    {
        $eventName = get_class($event);
        
        if (isset($this->listeners[$eventName])) {
            foreach ($this->listeners[$eventName] as $listener) {
                $listener($event);
            }
        }
    }
    
    public function addListener(string $eventName, callable $listener): void
    {
        $this->listeners[$eventName][] = $listener;
    }
}

// 订单模块发布事件
namespace App\Modules\Order;

class OrderCreatedEvent
{
    public function __construct(
        public readonly int $orderId,
        public readonly int $userId,
        public readonly float $total
    ) {}
}

class OrderService
{
    public function __construct(
        private OrderRepository $repository,
        private EventDispatcher $events
    ) {}
    
    public function createOrder(int $userId, array $items): Order
    {
        // 创建订单逻辑...
        $order = new Order(0, $userId, 100.0);
        $this->repository->save($order);
        
        // 发布事件
        $this->events->dispatch(new OrderCreatedEvent(
            $order->id,
            $order->userId,
            $order->total
        ));
        
        return $order;
    }
}

// 通知模块监听事件
namespace App\Modules\Notification;

class OrderNotificationListener
{
    private EmailService $emailService;
    
    public function __construct(EmailService $emailService)
    {
        $this->emailService = $emailService;
    }
    
    public function handle(\App\Modules\Order\OrderCreatedEvent $event): void
    {
        // 发送订单创建通知
        $this->emailService->sendOrderConfirmation($event->userId, $event->orderId);
    }
}
```

### 方式 3：共享数据模型

模块之间共享数据模型，但不直接调用：

```php
<?php
declare(strict_types=1);

// 共享的 DTO/数据模型
namespace App\Shared\DTO;

class UserDTO
{
    public function __construct(
        public readonly int $id,
        public readonly string $name,
        public readonly string $email
    ) {}
    
    public static function fromEntity(\App\Modules\User\User $user): self
    {
        return new self($user->getId(), $user->getName(), $user->getEmail());
    }
}

class OrderDTO
{
    public function __construct(
        public readonly int $id,
        public readonly int $userId,
        public readonly float $total,
        public readonly string $status
    ) {}
}
```

## 模块边界和依赖管理

### 清晰的模块边界

每个模块应该有清晰的边界：

```php
<?php
declare(strict_types=1);

// 模块目录结构
// src/
//   Modules/
//     User/
//       Domain/           # 领域层（实体、服务、仓储接口）
//       Application/      # 应用层（用例、DTO）
//       Infrastructure/   # 基础设施层（仓储实现）
//       Http/             # HTTP 层（控制器、请求、响应）
//     Order/
//       Domain/
//       Application/
//       Infrastructure/
//       Http/
//     Payment/
//       Domain/
//       Application/
//       Infrastructure/
//       Http/
//     Common/             # 共享代码
//       Traits/
//       Interfaces/
//       Exceptions/
```

### 依赖规则

```php
<?php
declare(strict_types=1);

// 依赖规则示例

// 用户模块 - 不依赖其他业务模块
namespace App\Modules\User\Domain;

// 允许的依赖
use App\Modules\Common\Exceptions\DomainException;  // Common 模块
use App\Modules\Common\Interfaces\RepositoryInterface;  // Common 接口

// 订单模块 - 只依赖接口，不依赖实现
namespace App\Modules\Order\Domain;

// 允许的依赖
use App\Modules\User\Domain\Services\UserServiceInterface;  // 其他模块的接口
use App\Modules\Common\Interfaces\LoggerInterface;  // Common 接口

// 不允许的依赖
// use App\Modules\User\Infrastructure\MySQLUserRepository;  // ❌ 禁止依赖其他模块的实现
```

## 模块化单体的实现

```php
<?php
declare(strict_types=1);

// 简单的模块化单体框架

// 模块注册表
class ModuleRegistry
{
    private array $modules = [];
    private array $bindings = [];
    
    public function register(Module $module): void
    {
        $module->register($this);
        $this->modules[] = $module;
    }
    
    public function bind(string $abstract, callable $factory): void
    {
        $this->bindings[$abstract] = $factory;
    }
    
    public function resolve(string $abstract): mixed
    {
        if (isset($this->bindings[$abstract])) {
            return ($this->bindings[$abstract])($this);
        }
        
        throw new RuntimeException("Cannot resolve: {$abstract}");
    }
    
    public function boot(): void
    {
        foreach ($this->modules as $module) {
            $module->boot($this);
        }
    }
}

// 模块接口
interface Module
{
    public function register(ModuleRegistry $registry): void;
    public function boot(ModuleRegistry $registry): void;
}

// 用户模块
class UserModule implements Module
{
    public function register(ModuleRegistry $registry): void
    {
        // 注册依赖
        $registry->bind(
            \App\Modules\User\Domain\UserRepositoryInterface::class,
            fn() => new \App\Modules\User\Infrastructure\MySQLUserRepository(
                new PDO('mysql:host=localhost;dbname=app', 'root', '')
            )
        );
        
        $registry->bind(
            \App\Modules\User\Domain\UserServiceInterface::class,
            fn($c) => new \App\Modules\User\Domain\UserService(
                $c->resolve(\App\Modules\User\Domain\UserRepositoryInterface::class)
            )
        );
    }
    
    public function boot(ModuleRegistry $registry): void
    {
        // 启动逻辑
    }
}

// 订单模块
class OrderModule implements Module
{
    public function register(ModuleRegistry $registry): void
    {
        $registry->bind(
            \App\Modules\Order\Domain\OrderRepositoryInterface::class,
            fn() => new \App\Modules\Order\Infrastructure\MySQLOrderRepository(
                new PDO('mysql:host=localhost;dbname=app', 'root', '')
            )
        );
        
        $registry->bind(
            \App\Modules\Order\Domain\OrderServiceInterface::class,
            fn($c) => new \App\Modules\Order\Domain\OrderService(
                $c->resolve(\App\Modules\Order\Domain\OrderRepositoryInterface::class),
                $c->resolve(\App\Modules\User\Domain\UserServiceInterface::class)
            )
        );
    }
    
    public function boot(ModuleRegistry $registry): void
    {
        // 启动逻辑
    }
}

// 应用启动
$registry = new ModuleRegistry();
$registry->register(new UserModule());
$registry->register(new OrderModule());
$registry->boot();

// 使用
$userService = $registry->resolve(\App\Modules\User\Domain\UserServiceInterface::class);
$orderService = $registry->resolve(\App\Modules\Order\Domain\OrderServiceInterface::class);
```

## 模块化单体的演进路径

### 演进阶段

1. **阶段一：模块化单体**
   - 清晰的模块划分
   - 统一的代码组织
   - 通过接口通信

2. **阶段二：独立部署的模块**
   - 模块可以独立部署
   - 使用消息队列通信
   - 独立的数据库

3. **阶段三：微服务**
   - 完全独立的部署
   - API 通信
   - 每个服务有自己的数据

## 使用场景

模块化单体适用于：
- 中小型项目
- 团队较小的项目
- 需要清晰代码组织的项目
- 未来可能拆分微服务的项目

## 最佳实践

1. **清晰的模块边界**：每个模块有明确的职责
2. **通过接口通信**：模块之间通过接口依赖
3. **独立的数据库**：条件允许时，每个模块有独立的数据库
4. **事件驱动**：使用事件进行松耦合通信
5. **规划演进**：考虑未来的拆分需求

## 练习任务

1. **设计模块结构**：为一个电商系统设计模块化单体架构

2. **实现模块系统**：使用 PHP 实现一个简单的模块系统

3. **设计模块通信**：设计模块之间的事件通信系统

4. **演进规划**：为一个现有的单体应用规划模块化拆分路径

5. **对比分析**：分析模块化单体和微服务的适用场景
