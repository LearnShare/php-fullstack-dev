# 9.1.3 数据结构设计

## 一、核心数据实体

### 1.1 租户（Tenant）

**业务含义：**
代表一个企业客户，是数据隔离的基本单位。

**核心属性：**
- `id`: 租户唯一标识（UUID）
- `name`: 企业名称
- `subdomain`: 子域名（用于识别租户）
- `custom_domain`: 自定义域名（可选，高级功能）
- `status`: 状态（active, suspended, cancelled）
- `subscription_id`: 当前订阅 ID
- `settings`: 租户配置（JSON）
- `created_at`: 创建时间
- `updated_at`: 更新时间

**业务规则：**
- 子域名全局唯一
- 自定义域名全局唯一（如果设置）
- 状态为 `suspended` 时，所有用户无法登录
- 状态为 `cancelled` 时，数据保留但无法访问

### 1.2 用户（User）

**业务含义：**
代表租户内的一个用户账号。

**核心属性：**
- `id`: 用户唯一标识（UUID）
- `tenant_id`: 所属租户 ID（外键）
- `email`: 邮箱（租户内唯一）
- `name`: 姓名
- `password_hash`: 密码哈希
- `avatar_url`: 头像 URL
- `email_verified_at`: 邮箱验证时间
- `status`: 状态（active, inactive, suspended）
- `last_login_at`: 最后登录时间
- `created_at`: 创建时间
- `updated_at`: 更新时间

**业务规则：**
- 邮箱在租户内唯一（不同租户可以有相同邮箱）
- 密码使用 Argon2ID 算法加密
- 邮箱未验证时，某些功能受限
- 状态为 `suspended` 时，无法登录

### 1.3 角色（Role）

**业务含义：**
定义用户在租户内的角色和权限。

**核心属性：**
- `id`: 角色唯一标识（UUID）
- `tenant_id`: 所属租户 ID（外键）
- `name`: 角色名称（租户内唯一）
- `display_name`: 显示名称
- `permissions`: 权限列表（JSON 数组）
- `is_system`: 是否为系统预定义角色
- `created_at`: 创建时间

**业务规则：**
- 角色名称在租户内唯一
- 系统角色不能删除
- 权限列表格式：`["permission1", "permission2", ...]`
- 每个租户至少有一个"超级管理员"角色

### 1.4 用户角色关联（UserRole）

**业务含义：**
用户和角色的多对多关系。

**核心属性：**
- `user_id`: 用户 ID（外键）
- `role_id`: 角色 ID（外键）
- `assigned_at`: 分配时间

**业务规则：**
- 一个用户可以有多个角色（权限取并集）
- 每个租户至少有一个用户拥有"超级管理员"角色

### 1.5 订阅（Subscription）

**业务含义：**
租户的订阅信息，关联到套餐。

**核心属性：**
- `id`: 订阅唯一标识（UUID）
- `tenant_id`: 所属租户 ID（外键）
- `plan_id`: 套餐 ID
- `stripe_subscription_id`: Stripe 订阅 ID
- `status`: 状态（active, cancelled, past_due, unpaid）
- `current_period_start`: 当前周期开始时间
- `current_period_end`: 当前周期结束时间
- `cancel_at_period_end`: 是否在周期结束时取消
- `cancelled_at`: 取消时间
- `created_at`: 创建时间
- `updated_at`: 更新时间

**业务规则：**
- 每个租户只能有一个活跃订阅
- 状态为 `active` 时，租户可以正常使用
- 状态为 `past_due` 时，功能受限
- 取消订阅后，当前周期内仍可使用

### 1.6 套餐（Plan）

**业务含义：**
系统预定义的订阅套餐。

**核心属性：**
- `id`: 套餐唯一标识（UUID）
- `name`: 套餐名称
- `display_name`: 显示名称
- `price_monthly`: 月价格（分）
- `price_yearly`: 年价格（分，可选）
- `features`: 功能列表（JSON）
- `limits`: 限制配置（JSON）
  - `max_users`: 最大用户数
  - `max_workspaces`: 最大工作空间数
  - `max_storage`: 最大存储空间（GB）
- `is_active`: 是否激活
- `sort_order`: 排序顺序
- `created_at`: 创建时间

**业务规则：**
- 套餐是系统级别的，所有租户共享
- 可以禁用套餐（不再显示给新用户）
- 限制配置决定租户能使用的资源

### 1.7 工作空间（Workspace）

**业务含义：**
租户内的工作空间，用于组织项目。

**核心属性：**
- `id`: 工作空间唯一标识（UUID）
- `tenant_id`: 所属租户 ID（外键）
- `name`: 工作空间名称
- `description`: 描述
- `owner_id`: 所有者用户 ID（外键）
- `settings`: 工作空间配置（JSON）
- `created_at`: 创建时间
- `updated_at`: 更新时间

**业务规则：**
- 工作空间名称在租户内唯一
- 创建者自动成为所有者
- 所有者拥有所有权限

### 1.8 工作空间成员（WorkspaceMember）

**业务含义：**
工作空间和用户的多对多关系。

**核心属性：**
- `workspace_id`: 工作空间 ID（外键）
- `user_id`: 用户 ID（外键）
- `role`: 在工作空间内的角色（owner, admin, member, viewer）
- `joined_at`: 加入时间

**业务规则：**
- 一个用户可以加入多个工作空间
- 一个工作空间可以有多个成员
- 角色决定成员在工作空间内的权限

## 二、数据库表设计

### 2.1 租户表（tenants）

```sql
CREATE TABLE tenants (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(255) NOT NULL COMMENT '企业名称',
    subdomain VARCHAR(100) UNIQUE NOT NULL COMMENT '子域名',
    custom_domain VARCHAR(255) UNIQUE NULL COMMENT '自定义域名',
    status ENUM('active', 'suspended', 'cancelled') DEFAULT 'active' COMMENT '状态',
    subscription_id VARCHAR(36) NULL COMMENT '当前订阅ID',
    settings JSON DEFAULT '{}' COMMENT '租户配置',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_subdomain (subdomain),
    INDEX idx_custom_domain (custom_domain),
    INDEX idx_status (status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 2.2 用户表（users）

```sql
CREATE TABLE users (
    id VARCHAR(36) PRIMARY KEY,
    tenant_id VARCHAR(36) NOT NULL COMMENT '所属租户ID',
    email VARCHAR(255) NOT NULL COMMENT '邮箱',
    name VARCHAR(255) NOT NULL COMMENT '姓名',
    password_hash VARCHAR(255) NOT NULL COMMENT '密码哈希',
    avatar_url VARCHAR(500) NULL COMMENT '头像URL',
    email_verified_at TIMESTAMP NULL COMMENT '邮箱验证时间',
    status ENUM('active', 'inactive', 'suspended') DEFAULT 'active' COMMENT '状态',
    last_login_at TIMESTAMP NULL COMMENT '最后登录时间',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY uk_tenant_email (tenant_id, email),
    INDEX idx_tenant_id (tenant_id),
    INDEX idx_email (email),
    FOREIGN KEY (tenant_id) REFERENCES tenants(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 2.3 角色表（roles）

```sql
CREATE TABLE roles (
    id VARCHAR(36) PRIMARY KEY,
    tenant_id VARCHAR(36) NOT NULL COMMENT '所属租户ID',
    name VARCHAR(100) NOT NULL COMMENT '角色名称',
    display_name VARCHAR(255) NOT NULL COMMENT '显示名称',
    permissions JSON NOT NULL COMMENT '权限列表',
    is_system BOOLEAN DEFAULT FALSE COMMENT '是否为系统角色',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE KEY uk_tenant_role (tenant_id, name),
    INDEX idx_tenant_id (tenant_id),
    FOREIGN KEY (tenant_id) REFERENCES tenants(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 2.4 用户角色关联表（user_roles）

```sql
CREATE TABLE user_roles (
    user_id VARCHAR(36) NOT NULL,
    role_id VARCHAR(36) NOT NULL,
    assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, role_id),
    INDEX idx_role_id (role_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 2.5 套餐表（plans）

```sql
CREATE TABLE plans (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL COMMENT '套餐名称',
    display_name VARCHAR(255) NOT NULL COMMENT '显示名称',
    price_monthly INT NOT NULL COMMENT '月价格（分）',
    price_yearly INT NULL COMMENT '年价格（分）',
    features JSON NOT NULL COMMENT '功能列表',
    limits JSON NOT NULL COMMENT '限制配置',
    is_active BOOLEAN DEFAULT TRUE COMMENT '是否激活',
    sort_order INT DEFAULT 0 COMMENT '排序顺序',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_is_active (is_active),
    INDEX idx_sort_order (sort_order)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 2.6 订阅表（subscriptions）

```sql
CREATE TABLE subscriptions (
    id VARCHAR(36) PRIMARY KEY,
    tenant_id VARCHAR(36) NOT NULL COMMENT '所属租户ID',
    plan_id VARCHAR(36) NOT NULL COMMENT '套餐ID',
    stripe_subscription_id VARCHAR(255) UNIQUE NULL COMMENT 'Stripe订阅ID',
    status ENUM('active', 'cancelled', 'past_due', 'unpaid') DEFAULT 'active' COMMENT '状态',
    current_period_start TIMESTAMP NOT NULL COMMENT '当前周期开始时间',
    current_period_end TIMESTAMP NOT NULL COMMENT '当前周期结束时间',
    cancel_at_period_end BOOLEAN DEFAULT FALSE COMMENT '是否在周期结束时取消',
    cancelled_at TIMESTAMP NULL COMMENT '取消时间',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_tenant_id (tenant_id),
    INDEX idx_status (status),
    INDEX idx_stripe_subscription_id (stripe_subscription_id),
    FOREIGN KEY (tenant_id) REFERENCES tenants(id) ON DELETE CASCADE,
    FOREIGN KEY (plan_id) REFERENCES plans(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 2.7 工作空间表（workspaces）

```sql
CREATE TABLE workspaces (
    id VARCHAR(36) PRIMARY KEY,
    tenant_id VARCHAR(36) NOT NULL COMMENT '所属租户ID',
    name VARCHAR(255) NOT NULL COMMENT '工作空间名称',
    description TEXT NULL COMMENT '描述',
    owner_id VARCHAR(36) NOT NULL COMMENT '所有者用户ID',
    settings JSON DEFAULT '{}' COMMENT '工作空间配置',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY uk_tenant_name (tenant_id, name),
    INDEX idx_tenant_id (tenant_id),
    INDEX idx_owner_id (owner_id),
    FOREIGN KEY (tenant_id) REFERENCES tenants(id) ON DELETE CASCADE,
    FOREIGN KEY (owner_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 2.8 工作空间成员表（workspace_members）

```sql
CREATE TABLE workspace_members (
    workspace_id VARCHAR(36) NOT NULL,
    user_id VARCHAR(36) NOT NULL,
    role ENUM('owner', 'admin', 'member', 'viewer') DEFAULT 'member' COMMENT '角色',
    joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (workspace_id, user_id),
    INDEX idx_user_id (user_id),
    FOREIGN KEY (workspace_id) REFERENCES workspaces(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## 三、数据关系图

### 3.1 ER 关系

```
tenants (1) ──< (N) users
tenants (1) ──< (N) roles
tenants (1) ──< (1) subscriptions
tenants (1) ──< (N) workspaces

users (N) >──< (N) roles (通过 user_roles)
users (1) ──< (N) workspaces (作为 owner)
users (N) >──< (N) workspaces (通过 workspace_members)

subscriptions (N) ──> (1) plans
workspaces (1) ──< (N) workspace_members
```

### 3.2 数据隔离策略

**核心原则：**
所有业务数据表都包含 `tenant_id` 字段，所有查询都必须添加租户过滤条件。

**实现方式：**
1. **中间件自动注入**：在请求处理前，识别租户并设置到上下文
2. **查询自动过滤**：所有数据库查询自动添加 `WHERE tenant_id = ?` 条件
3. **外键约束**：确保关联数据属于同一租户

**示例：**
```sql
-- 查询用户时，自动添加租户过滤
SELECT * FROM users 
WHERE tenant_id = 'tenant_abc_123' 
AND email = 'user@example.com';

-- 查询工作空间时，自动添加租户过滤
SELECT * FROM workspaces 
WHERE tenant_id = 'tenant_abc_123' 
AND id = 'workspace_xyz_456';
```

## 四、索引设计

### 4.1 主键索引

所有表使用 UUID 作为主键，自动创建主键索引。

### 4.2 唯一索引

- `tenants.subdomain`: 子域名唯一
- `tenants.custom_domain`: 自定义域名唯一
- `users (tenant_id, email)`: 租户内邮箱唯一
- `roles (tenant_id, name)`: 租户内角色名称唯一
- `workspaces (tenant_id, name)`: 租户内工作空间名称唯一

### 4.3 普通索引

- `users.tenant_id`: 按租户查询用户
- `users.email`: 邮箱查询（跨租户查询，仅管理员）
- `subscriptions.tenant_id`: 按租户查询订阅
- `subscriptions.status`: 按状态查询订阅
- `workspaces.tenant_id`: 按租户查询工作空间
- `workspaces.owner_id`: 按所有者查询工作空间

## 五、数据初始化

### 5.1 系统角色初始化

系统预定义角色（每个租户创建时自动创建）：

```sql
-- 超级管理员角色
INSERT INTO roles (id, tenant_id, name, display_name, permissions, is_system) 
VALUES (
    UUID(),
    'tenant_id',
    'super_admin',
    '超级管理员',
    '["*"]',  -- 所有权限
    TRUE
);

-- 管理员角色
INSERT INTO roles (id, tenant_id, name, display_name, permissions, is_system) 
VALUES (
    UUID(),
    'tenant_id',
    'admin',
    '管理员',
    '["users.manage", "workspaces.manage", "settings.view"]',
    TRUE
);

-- 成员角色
INSERT INTO roles (id, tenant_id, name, display_name, permissions, is_system) 
VALUES (
    UUID(),
    'tenant_id',
    'member',
    '成员',
    '["workspaces.view", "projects.view", "tasks.edit"]',
    TRUE
);
```

### 5.2 套餐初始化

系统预定义套餐：

```sql
-- 免费版
INSERT INTO plans (id, name, display_name, price_monthly, features, limits, sort_order) 
VALUES (
    UUID(),
    'free',
    '免费版',
    0,
    '["basic_features"]',
    '{"max_users": 5, "max_workspaces": 3, "max_storage": 1}',
    1
);

-- 基础版
INSERT INTO plans (id, name, display_name, price_monthly, features, limits, sort_order) 
VALUES (
    UUID(),
    'basic',
    '基础版',
    9900,  -- 99元
    '["all_features", "email_support"]',
    '{"max_users": 20, "max_workspaces": -1, "max_storage": 10}',
    2
);

-- 高级版
INSERT INTO plans (id, name, display_name, price_monthly, features, limits, sort_order) 
VALUES (
    UUID(),
    'premium',
    '高级版',
    29900,  -- 299元
    '["all_features", "priority_support", "advanced_reports"]',
    '{"max_users": 100, "max_workspaces": -1, "max_storage": 50}',
    3
);

-- 企业版
INSERT INTO plans (id, name, display_name, price_monthly, features, limits, sort_order) 
VALUES (
    UUID(),
    'enterprise',
    '企业版',
    99900,  -- 999元
    '["all_features", "dedicated_support", "custom_domain", "api_access"]',
    '{"max_users": -1, "max_workspaces": -1, "max_storage": -1}',
    4
);
```
