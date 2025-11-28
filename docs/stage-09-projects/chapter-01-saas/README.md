# 9.1 SaaS 平台（核心项目）

## 目标

- 设计模块化单体架构。
- 实现 DDD 分层架构。
- 构建多租户系统。
- 集成支付和计费功能。

## 项目概述

本项目是一个完整的 SaaS 平台，包含用户管理、多租户支持、订阅计费、权限管理等核心功能。采用模块化单体架构，使用 DDD 设计模式，确保代码的可维护性和可扩展性。

## 架构设计

### 模块化单体

```
src/
├── Modules/
│   ├── Auth/
│   │   ├── Domain/
│   │   │   ├── User.php
│   │   │   └── Role.php
│   │   ├── Application/
│   │   │   └── AuthService.php
│   │   └── Infrastructure/
│   │       └── UserRepository.php
│   ├── Billing/
│   │   ├── Domain/
│   │   │   ├── Subscription.php
│   │   │   └── Plan.php
│   │   └── Application/
│   │       └── BillingService.php
│   ├── Users/
│   │   └── ...
│   └── Workspaces/
│       └── ...
├── Shared/
│   ├── Kernel/
│   └── Infrastructure/
└── public/
    └── index.php
```

### 多租户实现

#### 租户识别

```php
<?php
declare(strict_types=1);

namespace App\Shared\Kernel;

class Tenant
{
    public function __construct(
        private string $id,
        private string $name,
        private string $database,
        private string $domain
    ) {}
    
    public function getId(): string
    {
        return $this->id;
    }
    
    public function getDatabase(): string
    {
        return $this->database;
    }
    
    public function getDomain(): string
    {
        return $this->domain;
    }
}
```

#### 租户中间件

```php
<?php
declare(strict_types=1);

namespace App\Shared\Kernel;

use PDO;

class TenantMiddleware
{
    public function __construct(
        private PDO $pdo
    ) {}
    
    public function handle(array $request, callable $next): array
    {
        // 从请求中识别租户（域名、子域名、Header 等）
        $tenant = $this->resolveTenant($request);
        
        if ($tenant === null) {
            http_response_code(400);
            echo json_encode(['error' => 'Invalid tenant']);
            exit;
        }
        
        // 设置租户上下文
        $this->setTenantContext($tenant);
        
        // 继续处理请求
        return $next($request);
    }
    
    private function resolveTenant(array $request): ?Tenant
    {
        // 方式 1：从子域名识别
        $host = $request['SERVER']['HTTP_HOST'] ?? '';
        $subdomain = explode('.', $host)[0];
        
        // 方式 2：从 Header 识别
        $tenantId = $request['SERVER']['HTTP_X_TENANT_ID'] ?? null;
        
        // 从数据库查询租户信息
        $stmt = $this->pdo->prepare(
            'SELECT id, name, database, domain FROM tenants WHERE id = ? OR domain = ?'
        );
        $stmt->execute([$tenantId ?? $subdomain, $host]);
        $data = $stmt->fetch(PDO::FETCH_ASSOC);
        
        if ($data === false) {
            return null;
        }
        
        return new Tenant(
            $data['id'],
            $data['name'],
            $data['database'],
            $data['domain']
        );
    }
    
    private function setTenantContext(Tenant $tenant): void
    {
        // 设置全局租户实例（使用依赖注入容器）
        // 或使用静态变量（不推荐）
        global $currentTenant;
        $currentTenant = $tenant;
        
        // 切换数据库连接
        // 这里简化处理，实际应该使用连接池
    }
}
```

### DDD 分层架构

#### 领域层：用户实体

```php
<?php
declare(strict_types=1);

namespace App\Modules\Auth\Domain;

class User
{
    public function __construct(
        private string $id,
        private string $email,
        private string $name,
        private string $passwordHash,
        private array $roles = []
    ) {}
    
    public function getId(): string
    {
        return $this->id;
    }
    
    public function getEmail(): string
    {
        return $this->email;
    }
    
    public function getName(): string
    {
        return $this->name;
    }
    
    public function hasRole(string $role): bool
    {
        return in_array($role, $this->roles, true);
    }
    
    public function addRole(string $role): void
    {
        if (!in_array($role, $this->roles, true)) {
            $this->roles[] = $role;
        }
    }
    
    public function verifyPassword(string $password): bool
    {
        return password_verify($password, $this->passwordHash);
    }
}
```

#### 应用层：认证服务

```php
<?php
declare(strict_types=1);

namespace App\Modules\Auth\Application;

use App\Modules\Auth\Domain\User;
use App\Modules\Auth\Infrastructure\UserRepository;

class AuthService
{
    public function __construct(
        private UserRepository $userRepository
    ) {}
    
    public function authenticate(string $email, string $password): ?User
    {
        $user = $this->userRepository->findByEmail($email);
        
        if ($user === null) {
            return null;
        }
        
        if (!$user->verifyPassword($password)) {
            return null;
        }
        
        return $user;
    }
    
    public function register(string $email, string $name, string $password): User
    {
        // 验证邮箱是否已存在
        if ($this->userRepository->findByEmail($email) !== null) {
            throw new \RuntimeException('Email already exists');
        }
        
        // 创建用户
        $passwordHash = password_hash($password, PASSWORD_ARGON2ID);
        $user = new User(
            uniqid('user_', true),
            $email,
            $name,
            $passwordHash
        );
        
        // 保存用户
        $this->userRepository->save($user);
        
        return $user;
    }
}
```

#### 基础设施层：用户仓储

```php
<?php
declare(strict_types=1);

namespace App\Modules\Auth\Infrastructure;

use App\Modules\Auth\Domain\User;
use PDO;

class UserRepository
{
    public function __construct(
        private PDO $pdo
    ) {}
    
    public function findByEmail(string $email): ?User
    {
        $stmt = $this->pdo->prepare(
            'SELECT id, email, name, password_hash, roles FROM users WHERE email = ?'
        );
        $stmt->execute([$email]);
        $data = $stmt->fetch(PDO::FETCH_ASSOC);
        
        if ($data === false) {
            return null;
        }
        
        return new User(
            $data['id'],
            $data['email'],
            $data['name'],
            $data['password_hash'],
            json_decode($data['roles'] ?? '[]', true)
        );
    }
    
    public function save(User $user): void
    {
        $stmt = $this->pdo->prepare(
            'INSERT INTO users (id, email, name, password_hash, roles) 
             VALUES (?, ?, ?, ?, ?)
             ON DUPLICATE KEY UPDATE 
                email = VALUES(email),
                name = VALUES(name),
                password_hash = VALUES(password_hash),
                roles = VALUES(roles)'
        );
        
        $stmt->execute([
            $user->getId(),
            $user->getEmail(),
            $user->getName(),
            $user->getPasswordHash(),
            json_encode($user->getRoles())
        ]);
    }
}
```

### 订阅和计费

#### 订阅实体

```php
<?php
declare(strict_types=1);

namespace App\Modules\Billing\Domain;

use Carbon\Carbon;

class Subscription
{
    public function __construct(
        private string $id,
        private string $tenantId,
        private string $planId,
        private Carbon $startDate,
        private ?Carbon $endDate = null,
        private string $status = 'active'
    ) {}
    
    public function isActive(): bool
    {
        return $this->status === 'active' 
            && ($this->endDate === null || $this->endDate->isFuture());
    }
    
    public function cancel(): void
    {
        $this->status = 'cancelled';
        $this->endDate = Carbon::now();
    }
}
```

#### 计费服务

```php
<?php
declare(strict_types=1);

namespace App\Modules\Billing\Application;

use App\Modules\Billing\Domain\Subscription;
use Stripe\StripeClient;

class BillingService
{
    public function __construct(
        private StripeClient $stripe
    ) {}
    
    public function createSubscription(
        string $tenantId,
        string $planId,
        string $paymentMethodId
    ): Subscription {
        // 创建 Stripe 订阅
        $stripeSubscription = $this->stripe->subscriptions->create([
            'customer' => $this->getOrCreateCustomer($tenantId),
            'items' => [['price' => $planId]],
            'payment_behavior' => 'default_incomplete',
            'payment_settings' => [
                'payment_method_types' => ['card'],
                'save_default_payment_method' => 'on_subscription',
            ],
            'expand' => ['latest_invoice.payment_intent'],
        ]);
        
        // 创建本地订阅记录
        $subscription = new Subscription(
            $stripeSubscription->id,
            $tenantId,
            $planId,
            Carbon::now()
        );
        
        return $subscription;
    }
    
    private function getOrCreateCustomer(string $tenantId): string
    {
        // 查询或创建 Stripe Customer
        // ...
    }
}
```

## 完整示例：用户注册流程

```php
<?php
declare(strict_types=1);

// public/api/register.php

require __DIR__ . '/../vendor/autoload.php';

use App\Modules\Auth\Application\AuthService;
use App\Modules\Auth\Infrastructure\UserRepository;

// 初始化依赖
$pdo = new PDO('mysql:host=localhost;dbname=saas', 'user', 'password');
$userRepository = new UserRepository($pdo);
$authService = new AuthService($userRepository);

// 处理请求
header('Content-Type: application/json');

try {
    // 读取请求体
    $input = json_decode(file_get_contents('php://input'), true);
    
    if (empty($input['email']) || empty($input['name']) || empty($input['password'])) {
        http_response_code(400);
        echo json_encode(['error' => 'Missing required fields']);
        exit;
    }
    
    // 注册用户
    $user = $authService->register(
        $input['email'],
        $input['name'],
        $input['password']
    );
    
    // 返回成功响应
    http_response_code(201);
    echo json_encode([
        'id' => $user->getId(),
        'email' => $user->getEmail(),
        'name' => $user->getName(),
    ]);
} catch (Exception $e) {
    http_response_code(500);
    echo json_encode(['error' => $e->getMessage()]);
}
```

## 数据库设计

### 租户表

```sql
CREATE TABLE tenants (
    id VARCHAR(255) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    domain VARCHAR(255) UNIQUE NOT NULL,
    database VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 用户表

```sql
CREATE TABLE users (
    id VARCHAR(255) PRIMARY KEY,
    tenant_id VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    roles JSON DEFAULT '[]',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_tenant_email (tenant_id, email),
    FOREIGN KEY (tenant_id) REFERENCES tenants(id)
);
```

### 订阅表

```sql
CREATE TABLE subscriptions (
    id VARCHAR(255) PRIMARY KEY,
    tenant_id VARCHAR(255) NOT NULL,
    plan_id VARCHAR(255) NOT NULL,
    status VARCHAR(50) NOT NULL,
    start_date TIMESTAMP NOT NULL,
    end_date TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (tenant_id) REFERENCES tenants(id)
);
```

## 练习

1. **设计并实现一个完整的 SaaS 平台**
   - 创建项目结构
   - 实现模块化单体架构
   - 建立 DDD 分层

2. **实现多租户数据隔离**
   - 从请求中识别租户
   - 实现租户中间件
   - 确保数据隔离

3. **集成 Stripe 支付系统**
   - 创建 Stripe 账户
   - 实现订阅创建
   - 处理支付回调

4. **实现订阅和计费功能**
   - 设计订阅模型
   - 实现计费逻辑
   - 处理订阅状态变更

5. **设计权限和角色系统**
   - 实现 RBAC 模型
   - 创建权限中间件
   - 保护 API 端点

6. **实现完整的用户管理功能**
   - 用户注册和登录
   - 密码重置
   - 用户资料管理

7. **部署到生产环境**
   - 配置 Docker 容器
   - 设置 CI/CD 流程
   - 监控和日志记录
