# 7.5.2 Ports（端口）

## 概述

端口（Ports）是六边形架构的核心概念之一，它定义了应用与外部世界交互的接口。端口就像 USB 接口一样，定义了标准，任何符合这个标准的设备都可以连接。在软件架构中，端口定义了业务核心与外部世界交互的方式，使得外部技术的变化不会影响业务逻辑。

**主要内容**：
- 端口的定义和作用
- 输入端口（Driving Port）的设计
- 输出端口（Driven Port）的设计
- 端口设计原则

## 端口类型

### 输入端口 vs 输出端口

```
输入端口（Driving Port）          输出端口（Driven Port）
        │                                  │
        │   外部驱动者调用              业务核心调用
        ▼                                  ▼
┌─────────────────┐            ┌─────────────────┐
│  Controller/    │            │  Repository/    │
│  Command       │            │  Service        │
└────────┬────────┘            └────────┬────────┘
         │                               │
         │  应用服务暴露                 │  应用服务依赖
         ▼                               ▼
   ┌─────────────┐                ┌─────────────┐
   │ Use Cases / │                │   Ports    │
   │ Services    │                │ (Interfaces)│
   └──────┬──────┘                └──────┬──────┘
          │                               │
          └───────────┬───────────────────┘
                      ▼
               ┌─────────────┐
               │  业务核心  │
               │ (Domain)   │
               └─────────────┘
```

## 输入端口设计

### 用例端口

```php
<?php
declare(strict_types=1);

// 输入端口 - 用例接口

// 用户创建用例
interface CreateUserUseCase
{
    public function execute(CreateUserCommand $command): User;
}

// 用户查询用例
interface GetUserUseCase
{
    public function execute(int $userId): ?User;
}

// 用户更新用例
interface UpdateUserUseCase
{
    public function execute(UpdateUserCommand $command): User;
}

// 用户删除用例
interface DeleteUserUseCase
{
    public function execute(int $userId): void;
}

// 命令对象
class CreateUserCommand
{
    public function __construct(
        public readonly string $name,
        public readonly string $email,
        public readonly ?string $phone = null
    ) {}
}

class UpdateUserCommand
{
    public function __construct(
        public readonly int $userId,
        public readonly ?string $name = null,
        public readonly ?string $email = null
    ) {}
}
```

### 服务端口

```php
<?php
declare(strict_types=1);

// 输入端口 - 应用服务接口

interface UserApplicationService
{
    public function register(string $name, string $email): User;
    public function updateProfile(int $userId, array $data): User;
    public function deactivate(int $userId): void;
    public function getUser(int $userId): ?User;
    public function listUsers(int $page = 1, int $limit = 20): array;
}

interface OrderApplicationService
{
    public function createOrder(int $customerId, array $items): Order;
    public function cancelOrder(int $orderId): void;
    public function getOrder(int $orderId): ?Order;
    public function listOrders(int $customerId): array;
}
```

## 输出端口设计

### 仓储端口

```php
<?php
declare(strict_types=1);

// 输出端口 - 仓储接口

interface UserRepositoryInterface
{
    public function findById(int $id): ?User;
    public function findByEmail(string $email): ?User;
    public function findAll(int $page = 1, int $limit = 20): array;
    public function save(User $user): int;
    public function update(User $user): void;
    public function delete(int $id): void;
    public function count(): int;
}

interface OrderRepositoryInterface
{
    public function findById(int $id): ?Order;
    public function findByCustomerId(int $customerId): array;
    public function save(Order $order): int;
    public function update(Order $order): void;
    public function delete(int $id): void;
}
```

### 服务端口

```php
<?php
declare(strict_types=1);

// 输出端口 - 外部服务接口

interface PaymentGatewayInterface
{
    public function charge(ChargeRequest $request): ChargeResponse;
    public function refund(RefundRequest $request): RefundResponse;
}

interface NotificationServiceInterface
{
    public function sendEmail(Email $email): void;
    public function sendSMS(SMS $sms): void;
    public function sendPushNotification(PushNotification $notification): void;
}

interface StorageServiceInterface
{
    public function upload(string $path, Stream $content): string;
    public function download(string $key): Stream;
    public function delete(string $key): void;
}
```

## 端口设计原则

### 1. 业务导向

```php
<?php
declare(strict_types=1);

// 好的设计 - 业务导向
interface OrderRepositoryInterface
{
    // 业务方法命名
    public function findActiveByCustomer(int $customerId): array;
    public function findPending(): array;
    public function findCompletedBetween(DateTime $start, DateTime $end): array;
}

// 不好的设计 - 技术导向
interface OrderRepositoryInterface
{
    // 技术方法命名
    public function select(array $conditions): array;
    public function insert(array $data): int;
    public function update(array $data, array $conditions): void;
}
```

### 2. 简单直接

```php
<?php
declare(strict_types=1);

// 好的设计 - 简单直接
interface UserServiceInterface
{
    public function create(string $name, string $email): User;
    public function getById(int $id): ?User;
}

// 不好的设计 - 过度抽象
interface UserServiceInterface
{
    public function execute(UserCommand $command): UserResult;
    public function query(UserQuery $query): UserResult;
}
```

### 3. 稳定接口

```php
<?php
declare(strict_types=1);

// 稳定的接口设计
interface PaymentGatewayInterface
{
    // 核心支付方法
    public function processPayment(float $amount, Currency $currency): PaymentResult;
    public function refundPayment(string $transactionId, float $amount): RefundResult;
    
    // 辅助方法（可选）
    public function getTransactionStatus(string $transactionId): TransactionStatus;
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

// 完整的端口设计示例

// ============ 输入端口 ============

// 用户服务输入端口
interface UserServicePort
{
    public function createUser(CreateUserInput $input): UserOutput;
    public function getUser(int $id): ?UserOutput;
    public function updateUser(int $id, UpdateUserInput $input): UserOutput;
    public function deleteUser(int $id): void;
    public function listUsers(ListUsersInput $input): ListUsersOutput;
}

// 输入 DTO
class CreateUserInput
{
    public function __construct(
        public readonly string $name,
        public readonly string $email,
        public readonly ?string $phone = null
    ) {}
}

class UpdateUserInput
{
    public function __construct(
        public readonly ?string $name = null,
        public readonly ?string $email = null,
        public readonly ?string $phone = null
    ) {}
}

class ListUsersInput
{
    public function __construct(
        public readonly int $page = 1,
        public readonly int $limit = 20,
        public readonly ?string $search = null
    ) {}
}

// 输出 DTO
class UserOutput
{
    public function __construct(
        public readonly int $id,
        public readonly string $name,
        public readonly string $email,
        public readonly ?string $phone,
        public readonly string $createdAt
    ) {}
}

class ListUsersOutput
{
    public function __construct(
        public readonly array $users,
        public readonly int $total,
        public readonly int $page,
        public readonly int $limit
    ) {}
}

// ============ 输出端口 ============

// 用户仓储输出端口
interface UserRepositoryPort
{
    public function findById(int $id): ?User;
    public function findByEmail(string $email): ?User;
    public function findAll(int $page, int $limit, ?string $search): array;
    public function count(?string $search): int;
    public function save(User $user): int;
    public function update(User $user): void;
    public function delete(int $id): void;
}

// 事件发布输出端口
interface EventPublisherPort
{
    public function publish(DomainEvent $event): void;
    public function publishBatch(array $events): void;
}
```

## 练习任务

1. **设计输入端口**：为一个订单管理系统设计输入端口

2. **设计输出端口**：设计订单管理的输出端口（仓储、服务）

3. **实现端口接口**：实现定义好的端口接口

4. **测试端口**：使用模拟适配器测试端口实现
