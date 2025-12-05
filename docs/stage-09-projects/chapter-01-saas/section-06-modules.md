# 9.1.6 功能模块划分

## 一、模块划分原则

### 1.1 划分依据

- **业务领域**：按业务功能领域划分
- **职责单一**：每个模块职责明确，高内聚低耦合
- **可扩展性**：模块之间通过接口交互，便于扩展
- **可测试性**：模块独立，便于单元测试

### 1.2 模块结构

采用**模块化单体架构**，每个模块包含：
- **Domain 层**：领域模型（实体、值对象）
- **Application 层**：应用服务（业务逻辑）
- **Infrastructure 层**：基础设施（仓储、外部服务适配器）
- **Presentation 层**：表现层（控制器、API 端点）

## 二、核心模块列表

### 2.1 租户模块（Tenant Module）

**职责：**
- 租户的创建、查询、更新
- 租户识别和上下文管理
- 租户配置管理

**主要功能：**
- 租户注册
- 租户识别（子域名、自定义域名）
- 租户状态管理
- 租户设置管理

**依赖关系：**
- 被所有其他模块依赖（所有模块都需要租户上下文）

### 2.2 认证模块（Auth Module）

**职责：**
- 用户注册、登录、登出
- 密码管理（重置、修改）
- Token 管理（生成、验证、刷新）
- 邮箱验证

**主要功能：**
- 用户注册
- 用户登录
- 密码重置
- 邮箱验证
- JWT Token 管理

**依赖关系：**
- 依赖：租户模块
- 被依赖：所有需要认证的模块

### 2.3 权限模块（Permission Module）

**职责：**
- 角色定义和管理
- 权限定义和管理
- 用户角色分配
- 权限验证

**主要功能：**
- 角色 CRUD
- 权限定义
- 用户角色分配
- 权限验证中间件

**依赖关系：**
- 依赖：租户模块、认证模块
- 被依赖：所有需要权限控制的模块

### 2.4 订阅模块（Subscription Module）

**职责：**
- 套餐管理
- 订阅创建、更新、取消
- 支付处理（Stripe 集成）
- 账单和发票管理

**主要功能：**
- 套餐查询
- 订阅创建
- 订阅更新
- 订阅取消
- Stripe Webhook 处理
- 发票生成

**依赖关系：**
- 依赖：租户模块
- 被依赖：租户模块（用于限制检查）

### 2.5 工作空间模块（Workspace Module）

**职责：**
- 工作空间的创建、查询、更新、删除
- 工作空间成员管理
- 工作空间权限控制

**主要功能：**
- 工作空间 CRUD
- 成员邀请
- 成员管理
- 工作空间设置

**依赖关系：**
- 依赖：租户模块、认证模块、权限模块
- 被依赖：项目模块（如果后续扩展）

### 2.6 共享模块（Shared Module）

**职责：**
- 公共基础设施
- 通用工具类
- 中间件
- 事件系统

**主要功能：**
- 租户识别中间件
- 认证中间件
- 权限验证中间件
- 异常处理
- 日志记录
- 事件发布和订阅

**依赖关系：**
- 被所有模块依赖

## 三、模块详细设计

### 3.1 租户模块结构

```
Tenant/
├── Domain/
│   ├── Tenant.php              # 租户实体
│   └── TenantStatus.php         # 租户状态枚举
├── Application/
│   ├── TenantService.php        # 租户服务
│   └── TenantRegistrationService.php  # 租户注册服务
├── Infrastructure/
│   ├── TenantRepository.php     # 租户仓储
│   └── TenantContext.php       # 租户上下文管理
└── Presentation/
    └── TenantController.php    # 租户控制器（可选，通常只有内部 API）
```

**核心类说明：**

- **Tenant（实体）**
  - 属性：id, name, subdomain, custom_domain, status, subscription_id
  - 方法：isActive(), isSuspended(), isCancelled()

- **TenantService（应用服务）**
  - 方法：register(), findById(), findBySubdomain(), updateStatus()

- **TenantRepository（仓储）**
  - 方法：save(), findById(), findBySubdomain(), findByDomain()

### 3.2 认证模块结构

```
Auth/
├── Domain/
│   ├── User.php                 # 用户实体
│   ├── UserStatus.php           # 用户状态枚举
│   └── Token.php                # Token 值对象
├── Application/
│   ├── AuthService.php          # 认证服务
│   ├── RegistrationService.php  # 注册服务
│   └── PasswordService.php     # 密码服务
├── Infrastructure/
│   ├── UserRepository.php       # 用户仓储
│   ├── TokenRepository.php      # Token 仓储（可选）
│   └── EmailService.php         # 邮件服务适配器
└── Presentation/
    └── AuthController.php       # 认证控制器
```

**核心类说明：**

- **User（实体）**
  - 属性：id, tenant_id, email, name, password_hash, status
  - 方法：verifyPassword(), updatePassword(), markEmailAsVerified()

- **AuthService（应用服务）**
  - 方法：login(), logout(), refreshToken()

- **RegistrationService（应用服务）**
  - 方法：register(), verifyEmail()

### 3.3 权限模块结构

```
Permission/
├── Domain/
│   ├── Role.php                 # 角色实体
│   ├── Permission.php           # 权限值对象
│   └── UserRole.php             # 用户角色关联实体
├── Application/
│   ├── RoleService.php          # 角色服务
│   ├── PermissionService.php    # 权限服务
│   └── AuthorizationService.php # 授权服务
├── Infrastructure/
│   ├── RoleRepository.php       # 角色仓储
│   └── PermissionChecker.php    # 权限检查器
└── Presentation/
    └── RoleController.php       # 角色控制器
```

**核心类说明：**

- **Role（实体）**
  - 属性：id, tenant_id, name, display_name, permissions
  - 方法：hasPermission(), addPermission(), removePermission()

- **AuthorizationService（应用服务）**
  - 方法：checkPermission(), getUserPermissions()

### 3.4 订阅模块结构

```
Subscription/
├── Domain/
│   ├── Subscription.php         # 订阅实体
│   ├── Plan.php                 # 套餐实体
│   ├── Invoice.php              # 发票实体
│   └── SubscriptionStatus.php   # 订阅状态枚举
├── Application/
│   ├── SubscriptionService.php  # 订阅服务
│   ├── PlanService.php          # 套餐服务
│   └── BillingService.php       # 计费服务
├── Infrastructure/
│   ├── SubscriptionRepository.php # 订阅仓储
│   ├── PlanRepository.php       # 套餐仓储
│   └── StripeAdapter.php        # Stripe 适配器
└── Presentation/
    └── SubscriptionController.php # 订阅控制器
```

**核心类说明：**

- **Subscription（实体）**
  - 属性：id, tenant_id, plan_id, status, current_period_start, current_period_end
  - 方法：isActive(), isCancelled(), cancel(), renew()

- **SubscriptionService（应用服务）**
  - 方法：create(), update(), cancel(), handleWebhook()

- **StripeAdapter（基础设施）**
  - 方法：createSubscription(), cancelSubscription(), handleWebhook()

### 3.5 工作空间模块结构

```
Workspace/
├── Domain/
│   ├── Workspace.php            # 工作空间实体
│   ├── WorkspaceMember.php      # 工作空间成员实体
│   └── WorkspaceRole.php        # 工作空间角色枚举
├── Application/
│   ├── WorkspaceService.php     # 工作空间服务
│   └── WorkspaceMemberService.php # 成员服务
├── Infrastructure/
│   ├── WorkspaceRepository.php  # 工作空间仓储
│   └── WorkspaceMemberRepository.php # 成员仓储
└── Presentation/
    └── WorkspaceController.php  # 工作空间控制器
```

**核心类说明：**

- **Workspace（实体）**
  - 属性：id, tenant_id, name, description, owner_id
  - 方法：isOwner(), canAccess()

- **WorkspaceService（应用服务）**
  - 方法：create(), update(), delete(), list()

- **WorkspaceMemberService（应用服务）**
  - 方法：invite(), remove(), updateRole()

## 四、模块间交互

### 4.1 依赖关系图

```
Shared (共享模块)
  ↑
  ├── Tenant (租户模块)
  │     ↑
  │     ├── Auth (认证模块)
  │     │     ↑
  │     │     └── Permission (权限模块)
  │     │           ↑
  │     │           └── Workspace (工作空间模块)
  │     │
  │     └── Subscription (订阅模块)
  │
  └── 所有其他模块
```

### 4.2 模块通信方式

#### 4.2.1 直接调用

**场景：** 模块之间有明确的依赖关系

**示例：**
- 认证模块调用租户模块查询租户信息
- 工作空间模块调用权限模块验证权限

**实现：**
- 通过依赖注入（DI）注入服务
- 调用应用服务的方法

#### 4.2.2 事件驱动

**场景：** 模块之间需要解耦，异步处理

**示例：**
- 用户注册成功后，发送欢迎邮件（异步）
- 订阅创建成功后，发送确认邮件（异步）

**实现：**
- 使用事件系统
- 发布事件：`UserRegistered`, `SubscriptionCreated`
- 订阅事件：邮件服务订阅事件并处理

#### 4.2.3 共享上下文

**场景：** 所有模块都需要访问的信息

**示例：**
- 租户上下文（当前请求的租户）
- 用户上下文（当前登录的用户）

**实现：**
- 使用中间件设置上下文
- 通过依赖注入容器访问

## 五、模块边界

### 5.1 模块边界原则

- **数据隔离**：每个模块管理自己的数据
- **接口明确**：模块之间通过明确的接口交互
- **避免循环依赖**：模块之间不能循环依赖

### 5.2 跨模块访问规则

#### 5.2.1 允许的跨模块访问

- **应用服务调用**：模块 A 的应用服务可以调用模块 B 的应用服务
- **仓储查询**：模块可以查询其他模块的只读数据（通过仓储）

#### 5.2.2 禁止的跨模块访问

- **直接访问数据库**：不能直接访问其他模块的数据库表
- **直接访问领域实体**：不能直接修改其他模块的领域实体
- **循环依赖**：不能形成循环依赖

### 5.3 共享数据访问

**共享表：**
- `tenants`：所有模块都需要租户信息
- `users`：多个模块需要用户信息

**访问方式：**
- 通过仓储接口访问
- 不直接操作数据库

## 六、模块扩展点

### 6.1 插件化设计

**预留扩展点：**

1. **认证方式扩展**
   - 当前：邮箱密码登录
   - 可扩展：OAuth 登录（Google, GitHub 等）
   - 可扩展：SSO 单点登录

2. **支付方式扩展**
   - 当前：Stripe 信用卡支付
   - 可扩展：PayPal、支付宝、微信支付

3. **通知方式扩展**
   - 当前：邮件通知
   - 可扩展：短信通知、推送通知

### 6.2 功能模块扩展

**可添加的模块：**

1. **项目模块**（Project Module）
   - 项目管理功能
   - 依赖：工作空间模块

2. **任务模块**（Task Module）
   - 任务管理功能
   - 依赖：项目模块

3. **文件模块**（File Module）
   - 文件上传和管理
   - 依赖：工作空间模块

4. **通知模块**（Notification Module）
   - 系统通知管理
   - 依赖：所有业务模块

5. **报表模块**（Report Module）
   - 数据统计和报表
   - 依赖：所有业务模块

## 七、模块测试策略

### 7.1 单元测试

**测试范围：**
- 领域实体：测试业务逻辑
- 应用服务：测试业务流程
- 仓储：测试数据访问（使用 Mock）

### 7.2 集成测试

**测试范围：**
- 模块间交互：测试模块之间的协作
- API 端点：测试完整的请求响应流程

### 7.3 测试隔离

**原则：**
- 每个模块的测试独立
- 使用 Mock 隔离外部依赖
- 使用测试数据库隔离数据
