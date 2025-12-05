# 9.1.4 账号及权限逻辑设计

## 一、账号体系

### 1.1 账号层级结构

```
平台级别（系统管理员）
  ↓
租户级别（企业）
  ↓
用户级别（企业员工）
  ↓
工作空间级别（部门/项目组）
```

**说明：**
- **平台级别**：我们自己的系统管理员，可以管理所有租户
- **租户级别**：每个企业是一个租户，数据完全隔离
- **用户级别**：租户内的员工账号
- **工作空间级别**：租户内的工作空间，用户可以加入多个工作空间

### 1.2 账号识别流程

#### 1.2.1 租户识别

**识别方式优先级：**

1. **子域名识别**（主要方式）
   - 请求：`https://abc.oursaas.com/login`
   - 提取子域名：`abc`
   - 查询租户表：`SELECT * FROM tenants WHERE subdomain = 'abc'`

2. **自定义域名识别**（高级功能）
   - 请求：`https://pm.abc.com/login`
   - 提取域名：`pm.abc.com`
   - 查询租户表：`SELECT * FROM tenants WHERE custom_domain = 'pm.abc.com'`

3. **Header 识别**（API 调用）
   - 请求头：`X-Tenant-ID: tenant_abc_123`
   - 直接使用 Header 中的租户 ID

**识别失败处理：**
- 如果无法识别租户，返回 400 错误："无法识别租户"
- 如果租户状态为 `suspended`，返回 403 错误："租户已被暂停"
- 如果租户状态为 `cancelled`，返回 403 错误："租户已取消"

#### 1.2.2 用户识别

**识别方式：**

1. **JWT Token**（主要方式）
   - Token 中包含：`user_id`, `tenant_id`, `roles`
   - 验证 Token 有效性
   - 从 Token 中提取用户信息

2. **Session**（可选，Web 应用）
   - Session 中存储：`user_id`, `tenant_id`
   - 验证 Session 有效性

**识别失败处理：**
- Token 无效或过期，返回 401 错误："未授权，请重新登录"
- 用户状态为 `suspended`，返回 403 错误："账号已被暂停"

### 1.3 账号状态管理

#### 1.3.1 租户状态

**状态类型：**

- **active**（激活）
  - 正常状态，所有功能可用
  - 用户可以正常登录和使用

- **suspended**（暂停）
  - 临时暂停，通常因为欠费或违规
  - 所有用户无法登录
  - 数据保留，恢复后可以继续使用

- **cancelled**（已取消）
  - 租户主动取消或到期未续费
  - 所有用户无法登录
  - 数据保留一段时间后删除（根据政策）

**状态转换规则：**

```
active → suspended: 管理员操作或自动（欠费）
suspended → active: 管理员操作或自动（续费）
active → cancelled: 租户主动取消或到期
cancelled → active: 不允许（需要重新注册）
```

#### 1.3.2 用户状态

**状态类型：**

- **active**（激活）
  - 正常状态，可以登录和使用

- **inactive**（未激活）
  - 新注册用户，邮箱未验证
  - 可以登录但功能受限

- **suspended**（暂停）
  - 被管理员暂停
  - 无法登录

**状态转换规则：**

```
inactive → active: 邮箱验证后自动激活
active → suspended: 管理员操作
suspended → active: 管理员操作
```

## 二、认证机制

### 2.1 登录认证流程

**流程图：**

```
用户提交邮箱和密码
  ↓
系统查找用户（邮箱 + 租户ID）
  ↓
验证密码哈希
  ↓
检查用户状态
  ↓
生成 JWT Token
  ↓
返回 Token 给客户端
  ↓
客户端存储 Token（Cookie 或 LocalStorage）
```

**详细步骤：**

1. **接收登录请求**
   - 邮箱：`user@example.com`
   - 密码：`Password123`
   - 租户：已通过中间件识别

2. **查找用户**
   ```sql
   SELECT * FROM users 
   WHERE tenant_id = ? AND email = ?
   ```

3. **验证密码**
   - 使用 `password_verify()` 验证密码哈希
   - 如果失败，记录失败次数
   - 失败 5 次后锁定账号 30 分钟

4. **检查状态**
   - 如果状态为 `suspended`，返回错误
   - 如果状态为 `inactive`，可以登录但功能受限

5. **生成 Token**
   - 包含信息：`user_id`, `tenant_id`, `roles`, `exp`
   - 有效期：24 小时
   - 签名：使用密钥签名

6. **更新登录信息**
   - 更新 `last_login_at`
   - 记录登录日志

### 2.2 Token 管理

#### 2.2.1 Token 结构

**JWT Payload 示例：**

```json
{
  "user_id": "user_abc_123",
  "tenant_id": "tenant_abc_123",
  "roles": ["super_admin", "admin"],
  "email": "user@example.com",
  "iat": 1704067200,
  "exp": 1704153600
}
```

#### 2.2.2 Token 刷新机制

**Refresh Token 流程：**

1. **登录时生成两个 Token**
   - Access Token：有效期 24 小时，用于 API 调用
   - Refresh Token：有效期 30 天，用于刷新 Access Token

2. **Token 过期处理**
   - Access Token 过期时，使用 Refresh Token 刷新
   - 刷新成功后，返回新的 Access Token
   - Refresh Token 过期时，需要重新登录

3. **Token 撤销**
   - 用户退出登录时，撤销 Refresh Token
   - 管理员可以撤销用户的 Token

### 2.3 密码管理

#### 2.3.1 密码规则

- 最少 8 个字符
- 必须包含大写字母
- 必须包含小写字母
- 必须包含数字
- 建议包含特殊字符

#### 2.3.2 密码加密

- 使用 `password_hash()` 函数
- 算法：`PASSWORD_ARGON2ID`
- 成本因子：根据服务器性能调整

#### 2.3.3 密码重置流程

```
用户请求重置密码
  ↓
系统生成重置令牌
  ↓
保存令牌到缓存（有效期1小时）
  ↓
发送重置邮件
  ↓
用户点击邮件链接
  ↓
验证令牌
  ↓
更新密码
  ↓
删除令牌
  ↓
自动登录
```

## 三、权限体系

### 3.1 权限层级

**三层权限体系：**

1. **租户级权限**（用户在整个租户内的权限）
   - 由角色决定
   - 例如：管理用户、管理订阅、查看报表

2. **工作空间级权限**（用户在工作空间内的权限）
   - 由工作空间成员角色决定
   - 例如：查看项目、创建任务、删除工作空间

3. **资源级权限**（用户对具体资源的权限）
   - 由资源所有者或共享设置决定
   - 例如：编辑任务、删除文件

### 3.2 角色定义

#### 3.2.1 系统预定义角色

**超级管理员（super_admin）**
- 权限：`["*"]`（所有权限）
- 可以执行的操作：
  - 管理所有用户
  - 管理订阅和计费
  - 管理所有工作空间
  - 删除租户
  - 查看所有数据

**管理员（admin）**
- 权限：`["users.manage", "workspaces.manage", "settings.view"]`
- 可以执行的操作：
  - 管理用户（邀请、移除、修改角色）
  - 管理工作空间（创建、删除、设置）
  - 查看设置（不能修改订阅）

**成员（member）**
- 权限：`["workspaces.view", "projects.view", "tasks.edit"]`
- 可以执行的操作：
  - 查看工作空间
  - 查看和编辑分配给自己的任务
  - 不能管理用户和工作空间

**查看者（viewer）**
- 权限：`["workspaces.view", "projects.view"]`
- 可以执行的操作：
  - 只能查看，不能编辑

#### 3.2.2 工作空间角色

**所有者（owner）**
- 拥有工作空间的所有权限
- 可以删除工作空间
- 可以管理所有成员

**管理员（admin）**
- 可以管理工作空间设置
- 可以管理成员
- 不能删除工作空间

**成员（member）**
- 可以创建和编辑项目
- 可以创建和编辑任务
- 不能管理设置和成员

**查看者（viewer）**
- 只能查看，不能编辑

### 3.3 权限验证流程

#### 3.3.1 权限检查步骤

```
接收请求
  ↓
识别租户和用户
  ↓
获取用户角色
  ↓
检查租户级权限
  ↓
检查工作空间级权限（如果涉及工作空间）
  ↓
检查资源级权限（如果涉及具体资源）
  ↓
所有检查通过，允许访问
```

#### 3.3.2 权限验证示例

**场景：用户要删除工作空间**

1. **检查租户级权限**
   - 用户角色：`["admin"]`
   - 需要权限：`workspaces.delete`
   - 检查：`admin` 角色是否有 `workspaces.delete` 权限
   - 结果：有权限，继续

2. **检查工作空间级权限**
   - 工作空间成员角色：`owner`
   - 需要权限：删除工作空间
   - 检查：`owner` 角色可以删除工作空间
   - 结果：有权限，继续

3. **检查资源级权限**
   - 工作空间所有者：`user_xyz_789`
   - 当前用户：`user_abc_123`
   - 检查：当前用户是否是所有者或管理员
   - 结果：是管理员，允许删除

4. **执行操作**
   - 删除工作空间

### 3.4 权限中间件

#### 3.4.1 中间件职责

1. **提取用户信息**
   - 从 Token 或 Session 中提取用户信息
   - 验证用户状态

2. **加载用户权限**
   - 查询用户的所有角色
   - 合并所有角色的权限（取并集）

3. **验证权限**
   - 检查用户是否有执行操作的权限
   - 如果没有权限，返回 403 错误

#### 3.4.2 中间件使用示例

**路由定义：**
```php
// 需要登录
Route::middleware(['auth'])->group(function () {
    // 需要管理员权限
    Route::middleware(['permission:users.manage'])->group(function () {
        Route::post('/users', 'UserController@create');
        Route::delete('/users/{id}', 'UserController@delete');
    });
    
    // 需要工作空间权限
    Route::middleware(['workspace.member'])->group(function () {
        Route::get('/workspaces/{id}', 'WorkspaceController@show');
    });
});
```

## 四、安全机制

### 4.1 数据隔离安全

#### 4.1.1 查询自动过滤

**原则：**
所有业务数据查询都必须自动添加租户过滤条件。

**实现方式：**
- 使用查询作用域（Query Scope）
- 中间件自动注入租户上下文
- ORM 自动添加 `WHERE tenant_id = ?` 条件

**示例：**
```php
// 错误：没有租户过滤
$users = User::where('email', $email)->get();

// 正确：自动添加租户过滤
$users = User::where('email', $email)->get();
// 实际执行的 SQL：
// SELECT * FROM users 
// WHERE tenant_id = 'tenant_abc_123' AND email = ?
```

#### 4.1.2 外键约束

**原则：**
所有关联数据必须属于同一租户。

**实现方式：**
- 数据库外键约束
- 应用层验证

**示例：**
```sql
-- 工作空间必须属于租户
FOREIGN KEY (tenant_id) REFERENCES tenants(id) ON DELETE CASCADE

-- 用户必须属于租户
FOREIGN KEY (tenant_id) REFERENCES tenants(id) ON DELETE CASCADE

-- 工作空间成员必须是同一租户的用户
-- 通过应用层验证：workspace.tenant_id = user.tenant_id
```

### 4.2 访问控制安全

#### 4.2.1 防止越权访问

**场景：用户 A 尝试访问用户 B 的数据**

**防护措施：**
1. 所有查询自动添加租户过滤
2. 资源访问时验证资源所有者
3. 权限中间件验证用户权限

**示例：**
```php
// 用户尝试访问其他用户的数据
GET /api/users/user_b_456

// 系统验证
1. 从 Token 获取当前用户：user_a_123
2. 查询用户 user_b_456
3. 检查：user_b_456.tenant_id === user_a_123.tenant_id
4. 检查：user_a_123 是否有 users.view 权限
5. 如果都通过，返回数据；否则返回 403
```

#### 4.2.2 防止 SQL 注入

**措施：**
- 使用预处理语句（PDO）
- 使用 ORM（Eloquent）
- 输入验证和转义

### 4.3 会话安全

#### 4.3.1 Token 安全

- Token 使用强密钥签名
- Token 设置合理的过期时间
- Token 存储在安全的 Cookie 中（HttpOnly, Secure, SameSite）

#### 4.3.2 密码安全

- 密码使用强哈希算法
- 密码不存储在日志中
- 密码重置链接有时效性

## 五、权限配置示例

### 5.1 权限列表定义

```php
// 权限定义
$permissions = [
    // 用户管理
    'users.view' => '查看用户',
    'users.create' => '创建用户',
    'users.edit' => '编辑用户',
    'users.delete' => '删除用户',
    
    // 工作空间管理
    'workspaces.view' => '查看工作空间',
    'workspaces.create' => '创建工作空间',
    'workspaces.edit' => '编辑工作空间',
    'workspaces.delete' => '删除工作空间',
    
    // 订阅管理
    'subscriptions.view' => '查看订阅',
    'subscriptions.manage' => '管理订阅',
    
    // 设置
    'settings.view' => '查看设置',
    'settings.edit' => '编辑设置',
];
```

### 5.2 角色权限配置

```php
// 角色权限配置
$rolePermissions = [
    'super_admin' => ['*'],  // 所有权限
    'admin' => [
        'users.view',
        'users.create',
        'users.edit',
        'users.delete',
        'workspaces.view',
        'workspaces.create',
        'workspaces.edit',
        'workspaces.delete',
        'settings.view',
    ],
    'member' => [
        'workspaces.view',
        'projects.view',
        'tasks.view',
        'tasks.edit',
    ],
    'viewer' => [
        'workspaces.view',
        'projects.view',
    ],
];
```
