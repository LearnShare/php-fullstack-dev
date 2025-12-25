# 3.7.3 模块化单体架构

## 概述

模块化单体架构（Modular Monolith）将应用组织为多个模块，每个模块有清晰的边界，但部署为单一应用。这种架构在保持单体简单性的同时，提供了模块化的优势。

## 分层架构

### 基本分层

```
┌─────────────────────────┐
│  User Interface Layer   │ 表现层（HTTP、CLI、WebSocket）
│  - 接收用户输入          │
│  - 返回响应              │
├─────────────────────────┤
│  Application Layer       │ 应用层（用例编排）
│  - 协调领域对象          │
│  - 处理事务              │
│  - 调用基础设施服务      │
├─────────────────────────┤
│  Domain Layer           │ 领域层（核心业务逻辑）
│  - 实体（Entity）        │
│  - 值对象（Value Object）│
│  - 领域服务（Domain Service）│
│  - 聚合根（Aggregate Root）│
├─────────────────────────┤
│  Infrastructure Layer   │ 基础设施层
│  - 数据库访问            │
│  - 外部 API 调用        │
│  - 文件系统操作          │
└─────────────────────────┘
```

### 依赖方向

```
依赖方向：从上到下
- 表现层 → 应用层 → 领域层
- 应用层 → 基础设施层
- 领域层 ← 基础设施层（通过接口）

关键原则：
- 领域层不依赖任何其他层
- 基础设施层实现领域层定义的接口
```

## 目录结构组织

### 按层次组织

```
src/
├── Domain/              # 领域层
│   ├── User/
│   │   ├── User.php
│   │   └── UserRepository.php
│   └── Product/
├── Application/         # 应用层
│   ├── Services/
│   └── DTOs/
├── Infrastructure/      # 基础设施层
│   ├── Database/
│   └── Cache/
└── Http/                # 表现层
    ├── Controllers/
    └── Middleware/
```

### 按模块组织

```
src/
├── Blog/
│   ├── Domain/
│   ├── Application/
│   ├── Infrastructure/
│   └── Http/
├── Shop/
│   ├── Domain/
│   ├── Application/
│   ├── Infrastructure/
│   └── Http/
└── User/
    ├── Domain/
    ├── Application/
    ├── Infrastructure/
    └── Http/
```

## 实际应用示例

### 领域层

```php
namespace App\Domain\User;

class User
{
    public function __construct(
        private UserId $id,
        private Email $email,
        private string $name
    ) {
    }

    public function changeEmail(Email $newEmail): void
    {
        if (!$newEmail->isValid()) {
            throw new InvalidEmailException($newEmail->getValue());
        }
        $this->email = $newEmail;
    }
}

interface UserRepository
{
    public function find(UserId $id): ?User;
    public function save(User $user): void;
}
```

### 应用层

```php
namespace App\Application;

use App\Domain\User\User;
use App\Domain\User\UserRepository;

class UserService
{
    public function __construct(
        private UserRepository $userRepository
    ) {
    }

    public function updateUserEmail(int $userId, string $email): void
    {
        $user = $this->userRepository->find(new UserId($userId));
        if ($user === null) {
            throw new UserNotFoundException($userId);
        }

        $user->changeEmail(new Email($email));
        $this->userRepository->save($user);
    }
}
```

### 基础设施层

```php
namespace App\Infrastructure\Database;

use App\Domain\User\User;
use App\Domain\User\UserRepository;
use App\Domain\User\UserId;

class DatabaseUserRepository implements UserRepository
{
    public function __construct(
        private PDO $db
    ) {
    }

    public function find(UserId $id): ?User
    {
        $stmt = $this->db->prepare('SELECT * FROM users WHERE id = ?');
        $stmt->execute([$id->getValue()]);
        $data = $stmt->fetch(PDO::FETCH_ASSOC);
        
        return $data ? $this->mapToUser($data) : null;
    }

    public function save(User $user): void
    {
        // 保存到数据库
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

### 表现层

```php
namespace App\Http\Controllers;

use App\Application\UserService;

class UserController
{
    public function __construct(
        private UserService $userService
    ) {
    }

    public function updateEmail(Request $request): Response
    {
        try {
            $this->userService->updateUserEmail(
                $request->get('user_id'),
                $request->get('email')
            );
            return new JsonResponse(['success' => true]);
        } catch (UserNotFoundException $e) {
            return new JsonResponse(['error' => $e->getMessage()], 404);
        } catch (InvalidEmailException $e) {
            return new JsonResponse(['error' => $e->getMessage()], 422);
        }
    }
}
```

## 模块化单体优势

### 优势

- **模块边界清晰**：每个模块有明确的职责和边界。
- **易于测试**：各层可以独立测试。
- **可扩展性**：可以逐步将模块拆分为微服务。
- **部署简单**：仍然是单体应用，部署简单。

### 适用场景

- 中等规模的应用
- 需要模块化但不需要微服务复杂度
- 团队规模适中
- 未来可能拆分为微服务

## 完整示例

```php
<?php
declare(strict_types=1);

// 项目结构
// src/
// ├── Domain/
// │   └── User/
// │       ├── User.php
// │       └── UserRepository.php
// ├── Application/
// │   └── UserService.php
// ├── Infrastructure/
// │   └── Database/
// │       └── DatabaseUserRepository.php
// └── Http/
//     └── Controllers/
//         └── UserController.php

// Domain Layer
namespace App\Domain\User;

class User
{
    public function __construct(
        private UserId $id,
        private Email $email
    ) {
    }
}

interface UserRepository
{
    public function find(UserId $id): ?User;
}

// Application Layer
namespace App\Application;

use App\Domain\User\{User, UserRepository};

class UserService
{
    public function __construct(
        private UserRepository $repository
    ) {
    }

    public function getUser(int $id): User
    {
        return $this->repository->find(new UserId($id))
            ?? throw new UserNotFoundException($id);
    }
}

// Infrastructure Layer
namespace App\Infrastructure\Database;

use App\Domain\User\{User, UserRepository, UserId};

class DatabaseUserRepository implements UserRepository
{
    public function find(UserId $id): ?User
    {
        // 数据库查询
        return null;
    }
}

// Presentation Layer
namespace App\Http\Controllers;

use App\Application\UserService;

class UserController
{
    public function __construct(
        private UserService $userService
    ) {
    }

    public function show(int $id): Response
    {
        $user = $this->userService->getUser($id);
        return new JsonResponse($user->toArray());
    }
}
```

## 注意事项

1. **依赖方向**：确保依赖方向正确，领域层不依赖其他层。

2. **接口定义**：在领域层定义接口，基础设施层实现接口。

3. **模块边界**：保持模块边界清晰，避免跨模块直接依赖。

4. **测试策略**：各层可以独立测试，使用接口进行模拟。

5. **演进路径**：设计时考虑未来可能拆分为微服务的路径。

## 练习

1. 创建一个模块化单体应用，包含 User 模块的完整分层结构。

2. 实现一个 Blog 模块，包含 Domain、Application、Infrastructure 和 Http 层。

3. 设计一个模块间的通信机制，避免直接依赖。

4. 创建一个可测试的模块化单体结构，各层可以独立测试。

5. 实现一个模块边界清晰的架构，为未来拆分为微服务做准备。
