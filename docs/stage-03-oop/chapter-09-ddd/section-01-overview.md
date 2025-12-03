# 3.9.1 DDD 概述与分层

## 概述

领域驱动设计（Domain-Driven Design）是一种软件开发方法，强调以领域为中心，使用通用语言，让代码直接反映业务模型。

## 什么是 DDD

### 核心原则

领域驱动设计（Domain-Driven Design）是一种软件开发方法，强调：

- **以领域为中心**：业务逻辑是核心，技术实现是细节。
- **通用语言（Ubiquitous Language）**：开发团队与业务专家使用相同的术语。
- **模型驱动**：代码直接反映业务模型。

### 为什么需要 DDD

**问题**：传统开发方式的问题
- 业务逻辑分散在 Controller、Service、Repository 中
- 难以理解业务规则和约束
- 代码难以维护和扩展

**解决方案**：DDD 将业务逻辑集中在领域层
- 业务规则清晰可见
- 代码直接反映业务模型
- 易于理解和维护

## DDD 的分层

### 分层架构

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

## 实际应用示例

### 简单示例：用户注册

```php
<?php
declare(strict_types=1);

// 第一步：领域层定义实体和业务规则
namespace App\Domain\User;

/**
 * 用户实体（领域层）
 * 
 * 包含业务逻辑和业务规则
 */
class User
{
    public function __construct(
        private UserId $id,
        private Email $email,
        private string $name,
        private PasswordHash $passwordHash
    ) {
        // 业务规则：邮箱必须有效
        if (!$email->isValid()) {
            throw new InvalidEmailException("Invalid email: {$email->getValue()}");
        }
        
        // 业务规则：名称不能为空
        if (empty(trim($name))) {
            throw new InvalidNameException('Name cannot be empty');
        }
    }
    
    /**
     * 更改邮箱（业务操作）
     */
    public function changeEmail(Email $newEmail): void
    {
        // 业务规则：新邮箱必须与当前邮箱不同
        if ($newEmail->equals($this->email)) {
            throw new \DomainException('New email must be different from current email');
        }
        
        // 业务规则：新邮箱必须有效
        if (!$newEmail->isValid()) {
            throw new InvalidEmailException("Invalid email: {$newEmail->getValue()}");
        }
        
        $this->email = $newEmail;
    }
}

// 第二步：应用层编排用例
namespace App\Application;

use App\Domain\User\User;
use App\Domain\User\UserRepository;
use App\Domain\User\Email;

/**
 * 用户应用服务（应用层）
 * 
 * 编排领域对象，处理用例
 */
class UserApplicationService
{
    public function __construct(
        private UserRepository $userRepository
    ) {}
    
    /**
     * 注册用户（用例）
     */
    public function registerUser(string $email, string $name, string $password): User
    {
        // 1. 验证邮箱是否已存在（业务规则）
        $existingUser = $this->userRepository->findByEmail(new Email($email));
        if ($existingUser !== null) {
            throw new EmailAlreadyExistsException("Email already exists: {$email}");
        }
        
        // 2. 创建用户实体（领域对象）
        $user = new User(
            UserId::generate(),
            new Email($email),
            $name,
            PasswordHash::fromPlainText($password)
        );
        
        // 3. 保存用户（通过仓储）
        $this->userRepository->save($user);
        
        return $user;
    }
}

// 第三步：基础设施层实现仓储
namespace App\Infrastructure\Persistence;

use App\Domain\User\User;
use App\Domain\User\UserRepository;
use App\Domain\User\Email;
use App\Domain\User\UserId;
use PDO;

/**
 * 用户仓储实现（基础设施层）
 * 
 * 实现领域层定义的仓储接口
 */
class DatabaseUserRepository implements UserRepository
{
    public function __construct(
        private PDO $pdo
    ) {}
    
    public function findByEmail(Email $email): ?User
    {
        $stmt = $this->pdo->prepare(
            'SELECT id, email, name, password_hash FROM users WHERE email = ?'
        );
        $stmt->execute([$email->getValue()]);
        $data = $stmt->fetch(PDO::FETCH_ASSOC);
        
        if ($data === false) {
            return null;
        }
        
        return $this->mapToUser($data);
    }
    
    public function save(User $user): void
    {
        $stmt = $this->pdo->prepare(
            'INSERT INTO users (id, email, name, password_hash) 
             VALUES (?, ?, ?, ?)
             ON DUPLICATE KEY UPDATE 
                email = VALUES(email),
                name = VALUES(name),
                password_hash = VALUES(password_hash)'
        );
        
        $stmt->execute([
            $user->getId()->getValue(),
            $user->getEmail()->getValue(),
            $user->getName(),
            $user->getPasswordHash()->getValue(),
        ]);
    }
    
    private function mapToUser(array $data): User
    {
        return new User(
            new UserId($data['id']),
            new Email($data['email']),
            $data['name'],
            new PasswordHash($data['password_hash'])
        );
    }
}

// 第四步：表现层处理 HTTP 请求
namespace App\Http\Controllers;

use App\Application\UserApplicationService;

/**
 * 用户控制器（表现层）
 * 
 * 处理 HTTP 请求，调用应用服务
 */
class UserController
{
    public function __construct(
        private UserApplicationService $userService
    ) {}
    
    public function register(array $request): void
    {
        try {
            $input = json_decode($request['body'] ?? '{}', true);
            
            // 调用应用服务
            $user = $this->userService->registerUser(
                $input['email'] ?? '',
                $input['name'] ?? '',
                $input['password'] ?? ''
            );
            
            // 返回响应
            http_response_code(201);
            header('Content-Type: application/json');
            echo json_encode([
                'id' => $user->getId()->getValue(),
                'email' => $user->getEmail()->getValue(),
                'name' => $user->getName(),
            ]);
            
        } catch (EmailAlreadyExistsException $e) {
            http_response_code(409);
            echo json_encode(['error' => $e->getMessage()]);
        } catch (\Exception $e) {
            http_response_code(400);
            echo json_encode(['error' => $e->getMessage()]);
        }
    }
}
```

## 完整示例说明

1. **领域层**：定义 `User` 实体和业务规则（邮箱验证、名称验证等）
2. **应用层**：`UserApplicationService` 编排用例（注册用户）
3. **基础设施层**：`DatabaseUserRepository` 实现数据访问
4. **表现层**：`UserController` 处理 HTTP 请求

**优势**：
- 业务逻辑集中在领域层，易于理解和维护
- 各层职责清晰，易于测试
- 可以轻松替换基础设施层（如从 MySQL 切换到 PostgreSQL）

## 注意事项

1. **领域层独立性**：领域层不应该依赖任何其他层。

2. **接口定义**：在领域层定义接口，基础设施层实现接口。

3. **业务逻辑位置**：业务逻辑应该在领域层，而不是应用层或基础设施层。

4. **通用语言**：使用业务领域的术语，而不是技术术语。

5. **模型驱动**：代码应该直接反映业务模型。

## 练习

1. 创建一个简单的 DDD 应用，包含完整的四层结构。

2. 实现用户注册功能，演示各层的职责。

3. 设计一个订单处理系统，使用 DDD 分层架构。

4. 创建一个领域层，包含实体和业务规则。

5. 实现应用层服务，编排领域对象完成用例。
