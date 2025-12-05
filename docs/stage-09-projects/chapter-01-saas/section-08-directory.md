# 9.1.8 文件目录设计

## 一、项目根目录结构

```
saas-platform/
├── app/                          # 应用代码（Laravel）
│   ├── Modules/                  # 业务模块
│   ├── Shared/                   # 共享代码
│   └── Http/                     # HTTP 层（控制器、中间件等）
├── config/                       # 配置文件
├── database/                     # 数据库相关
│   ├── migrations/              # 数据库迁移
│   ├── seeders/                 # 数据填充
│   └── factories/               # 模型工厂
├── public/                       # 公共入口
│   └── index.php                # 入口文件
├── resources/                    # 资源文件
│   ├── views/                   # 视图文件（如果使用）
│   └── lang/                    # 语言文件
├── routes/                       # 路由定义
│   ├── api.php                  # API 路由
│   └── web.php                  # Web 路由
├── storage/                      # 存储目录
│   ├── app/                     # 应用存储
│   ├── framework/               # 框架文件
│   └── logs/                    # 日志文件
├── tests/                        # 测试文件
│   ├── Unit/                    # 单元测试
│   ├── Feature/                 # 功能测试
│   └── Integration/             # 集成测试
├── vendor/                       # Composer 依赖
├── .env.example                 # 环境变量示例
├── .gitignore                   # Git 忽略文件
├── composer.json                # Composer 配置
├── package.json                 # NPM 配置
├── phpunit.xml                  # PHPUnit 配置
└── README.md                    # 项目说明
```

## 二、模块目录结构

### 2.1 模块标准结构

```
app/Modules/{ModuleName}/
├── Domain/                       # 领域层
│   ├── Entities/               # 实体
│   │   └── {Entity}.php
│   ├── ValueObjects/            # 值对象
│   │   └── {ValueObject}.php
│   └── Enums/                   # 枚举
│       └── {Enum}.php
├── Application/                  # 应用层
│   ├── Services/                # 应用服务
│   │   └── {Service}.php
│   └── DTOs/                    # 数据传输对象
│       └── {DTO}.php
├── Infrastructure/               # 基础设施层
│   ├── Repositories/            # 仓储实现
│   │   └── {Repository}.php
│   └── Adapters/                # 外部服务适配器
│       └── {Adapter}.php
└── Presentation/                # 表现层
    ├── Controllers/             # 控制器
    │   └── {Controller}.php
    ├── Requests/                # 请求验证
    │   └── {Request}.php
    └── Resources/               # API 资源
        └── {Resource}.php
```

### 2.2 租户模块目录

```
app/Modules/Tenant/
├── Domain/
│   ├── Entities/
│   │   └── Tenant.php
│   └── Enums/
│       └── TenantStatus.php
├── Application/
│   └── Services/
│       ├── TenantService.php
│       └── TenantRegistrationService.php
├── Infrastructure/
│   ├── Repositories/
│   │   └── TenantRepository.php
│   └── Context/
│       └── TenantContext.php
└── Presentation/
    └── Controllers/
        └── TenantController.php
```

### 2.3 认证模块目录

```
app/Modules/Auth/
├── Domain/
│   ├── Entities/
│   │   └── User.php
│   └── Enums/
│       └── UserStatus.php
├── Application/
│   ├── Services/
│   │   ├── AuthService.php
│   │   ├── RegistrationService.php
│   │   └── PasswordService.php
│   └── DTOs/
│       └── LoginDTO.php
├── Infrastructure/
│   ├── Repositories/
│   │   └── UserRepository.php
│   └── Adapters/
│       └── EmailServiceAdapter.php
└── Presentation/
    ├── Controllers/
    │   └── AuthController.php
    └── Requests/
        ├── RegisterRequest.php
        └── LoginRequest.php
```

### 2.4 权限模块目录

```
app/Modules/Permission/
├── Domain/
│   ├── Entities/
│   │   ├── Role.php
│   │   └── UserRole.php
│   └── ValueObjects/
│       └── Permission.php
├── Application/
│   └── Services/
│       ├── RoleService.php
│       ├── PermissionService.php
│       └── AuthorizationService.php
├── Infrastructure/
│   ├── Repositories/
│   │   └── RoleRepository.php
│   └── Checkers/
│       └── PermissionChecker.php
└── Presentation/
    ├── Controllers/
    │   └── RoleController.php
    └── Resources/
        └── RoleResource.php
```

### 2.5 订阅模块目录

```
app/Modules/Subscription/
├── Domain/
│   ├── Entities/
│   │   ├── Subscription.php
│   │   ├── Plan.php
│   │   └── Invoice.php
│   └── Enums/
│       └── SubscriptionStatus.php
├── Application/
│   └── Services/
│       ├── SubscriptionService.php
│       ├── PlanService.php
│       └── BillingService.php
├── Infrastructure/
│   ├── Repositories/
│   │   ├── SubscriptionRepository.php
│   │   └── PlanRepository.php
│   └── Adapters/
│       └── StripeAdapter.php
└── Presentation/
    ├── Controllers/
    │   └── SubscriptionController.php
    └── Requests/
        └── CreateSubscriptionRequest.php
```

### 2.6 工作空间模块目录

```
app/Modules/Workspace/
├── Domain/
│   ├── Entities/
│   │   ├── Workspace.php
│   │   └── WorkspaceMember.php
│   └── Enums/
│       └── WorkspaceRole.php
├── Application/
│   └── Services/
│       ├── WorkspaceService.php
│       └── WorkspaceMemberService.php
├── Infrastructure/
│   └── Repositories/
│       ├── WorkspaceRepository.php
│       └── WorkspaceMemberRepository.php
└── Presentation/
    ├── Controllers/
    │   └── WorkspaceController.php
    └── Resources/
        └── WorkspaceResource.php
```

## 三、共享模块目录

```
app/Shared/
├── Kernel/                       # 核心功能
│   ├── Middleware/               # 中间件
│   │   ├── TenantMiddleware.php
│   │   ├── AuthMiddleware.php
│   │   └── PermissionMiddleware.php
│   ├── Events/                   # 事件
│   │   └── TenantIdentified.php
│   └── Exceptions/               # 异常
│       └── TenantNotFoundException.php
├── Infrastructure/               # 基础设施
│   ├── Database/                 # 数据库相关
│   │   └── TenantScope.php      # 租户查询作用域
│   ├── Cache/                    # 缓存
│   │   └── CacheService.php
│   └── Logging/                  # 日志
│       └── Logger.php
└── Domain/                       # 共享领域
    └── ValueObjects/
        └── Email.php
```

## 四、配置文件结构

```
config/
├── app.php                       # 应用配置
├── database.php                   # 数据库配置
├── cache.php                     # 缓存配置
├── auth.php                      # 认证配置
├── modules.php                   # 模块配置
├── tenant.php                    # 租户配置
├── stripe.php                    # Stripe 配置
└── mail.php                      # 邮件配置
```

## 五、路由文件结构

```
routes/
├── api.php                       # API 路由
│   ├── auth.php                 # 认证路由（包含）
│   ├── user.php                 # 用户路由（包含）
│   ├── team.php                 # 团队路由（包含）
│   ├── subscription.php         # 订阅路由（包含）
│   └── workspace.php            # 工作空间路由（包含）
└── web.php                       # Web 路由
    └── webhook.php              # Webhook 路由（包含）
```

**路由组织示例：**

```php
// routes/api.php
Route::prefix('api/v1')->group(function () {
    // 公开路由（无需认证）
    Route::post('/register', [AuthController::class, 'register']);
    Route::post('/login', [AuthController::class, 'login']);
    
    // 需要认证的路由
    Route::middleware(['auth'])->group(function () {
        // 用户相关
        Route::prefix('user')->group(function () {
            Route::get('/', [UserController::class, 'show']);
            Route::patch('/', [UserController::class, 'update']);
        });
        
        // 团队相关
        Route::prefix('team')->group(function () {
            Route::get('/members', [TeamController::class, 'listMembers']);
            Route::post('/members/invite', [TeamController::class, 'invite']);
        });
        
        // 订阅相关
        Route::prefix('subscription')->group(function () {
            Route::get('/', [SubscriptionController::class, 'show']);
            Route::post('/', [SubscriptionController::class, 'create']);
        });
        
        // 工作空间相关
        Route::prefix('workspaces')->group(function () {
            Route::get('/', [WorkspaceController::class, 'list']);
            Route::post('/', [WorkspaceController::class, 'create']);
            Route::get('/{id}', [WorkspaceController::class, 'show']);
            Route::patch('/{id}', [WorkspaceController::class, 'update']);
            Route::delete('/{id}', [WorkspaceController::class, 'delete']);
        });
    });
});
```

## 六、测试文件结构

```
tests/
├── Unit/                         # 单元测试
│   ├── Domain/                   # 领域层测试
│   │   └── TenantTest.php
│   ├── Application/              # 应用层测试
│   │   └── TenantServiceTest.php
│   └── Infrastructure/           # 基础设施层测试
│       └── TenantRepositoryTest.php
├── Feature/                      # 功能测试
│   ├── AuthTest.php
│   ├── SubscriptionTest.php
│   └── WorkspaceTest.php
└── Integration/                  # 集成测试
    └── ApiTest.php
```

## 七、数据库迁移文件结构

```
database/migrations/
├── 2024_01_01_000001_create_tenants_table.php
├── 2024_01_01_000002_create_users_table.php
├── 2024_01_01_000003_create_roles_table.php
├── 2024_01_01_000004_create_user_roles_table.php
├── 2024_01_01_000005_create_plans_table.php
├── 2024_01_01_000006_create_subscriptions_table.php
├── 2024_01_01_000007_create_workspaces_table.php
└── 2024_01_01_000008_create_workspace_members_table.php
```

## 八、前端目录结构（如果使用）

```
resources/
├── js/                           # JavaScript 文件
│   ├── app.js                   # 应用入口
│   ├── components/              # 组件
│   ├── pages/                   # 页面
│   ├── services/                # API 服务
│   └── store/                    # 状态管理
├── css/                          # CSS 文件
│   └── app.css
└── views/                        # 视图文件（如果使用 Blade）
    └── layouts/
        └── app.blade.php
```

## 九、命名规范

### 9.1 文件命名

- **类文件**：大驼峰命名，如 `TenantService.php`
- **测试文件**：类名 + `Test`，如 `TenantServiceTest.php`
- **迁移文件**：时间戳 + 描述，如 `2024_01_01_000001_create_tenants_table.php`

### 9.2 目录命名

- **模块目录**：大驼峰命名，如 `Tenant/`
- **子目录**：大驼峰命名，如 `Entities/`, `Services/`

### 9.3 命名空间

- **模块命名空间**：`App\Modules\{ModuleName}\{Layer}`
- **共享命名空间**：`App\Shared\{Category}`

**示例：**
```php
// 租户服务
namespace App\Modules\Tenant\Application\Services;

// 用户实体
namespace App\Modules\Auth\Domain\Entities;

// 共享中间件
namespace App\Shared\Kernel\Middleware;
```

## 十、自动加载配置

### 10.1 Composer 自动加载

```json
{
  "autoload": {
    "psr-4": {
      "App\\": "app/",
      "App\\Modules\\": "app/Modules/",
      "App\\Shared\\": "app/Shared/"
    }
  }
}
```

### 10.2 模块自动发现

使用 Laravel 的模块化包或自定义服务提供者自动发现和注册模块。
