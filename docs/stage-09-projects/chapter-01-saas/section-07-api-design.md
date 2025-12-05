# 9.1.7 API 设计

## 一、API 设计原则

### 1.1 RESTful 规范

- 使用 HTTP 方法表示操作（GET, POST, PUT, DELETE, PATCH）
- 使用资源名词表示资源（复数形式）
- 使用 URL 路径表示资源层级
- 使用 HTTP 状态码表示结果

### 1.2 统一响应格式

**成功响应：**
```json
{
  "success": true,
  "data": {
    // 响应数据
  },
  "meta": {
    // 元数据（分页、时间等）
  }
}
```

**错误响应：**
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "错误描述",
    "details": {
      // 详细错误信息（可选）
    }
  }
}
```

### 1.3 认证方式

- 使用 JWT Token 认证
- Token 通过 Header 传递：`Authorization: Bearer {token}`
- Token 过期后使用 Refresh Token 刷新

## 二、API 端点列表

### 2.1 公开 API（无需认证）

#### 2.1.1 租户注册

**POST** `/api/register`

**请求体：**
```json
{
  "name": "ABC 公司",
  "email": "admin@abc.com",
  "password": "Password123",
  "subdomain": "abc"
}
```

**响应：**
```json
{
  "success": true,
  "data": {
    "tenant_id": "tenant_abc_123",
    "user_id": "user_admin_456",
    "message": "注册成功，请查收验证邮件"
  }
}
```

**错误响应：**
```json
{
  "success": false,
  "error": {
    "code": "SUBDOMAIN_EXISTS",
    "message": "子域名已被使用"
  }
}
```

#### 2.1.2 用户登录

**POST** `/api/login`

**请求体：**
```json
{
  "email": "admin@abc.com",
  "password": "Password123",
  "remember": true
}
```

**响应：**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "refresh_token_xxx",
    "expires_in": 86400,
    "user": {
      "id": "user_admin_456",
      "email": "admin@abc.com",
      "name": "管理员",
      "roles": ["super_admin"]
    }
  }
}
```

#### 2.1.3 密码重置请求

**POST** `/api/password/reset-request`

**请求体：**
```json
{
  "email": "admin@abc.com"
}
```

**响应：**
```json
{
  "success": true,
  "data": {
    "message": "如果邮箱存在，我们将发送重置链接"
  }
}
```

#### 2.1.4 密码重置

**POST** `/api/password/reset`

**请求体：**
```json
{
  "token": "reset_token_xxx",
  "password": "NewPassword123"
}
```

**响应：**
```json
{
  "success": true,
  "data": {
    "token": "new_jwt_token",
    "message": "密码重置成功"
  }
}
```

### 2.2 认证 API（需要 Token）

#### 2.2.1 获取当前用户信息

**GET** `/api/user`

**Headers：**
```
Authorization: Bearer {token}
```

**响应：**
```json
{
  "success": true,
  "data": {
    "id": "user_admin_456",
    "email": "admin@abc.com",
    "name": "管理员",
    "avatar_url": "https://...",
    "roles": ["super_admin"],
    "tenant": {
      "id": "tenant_abc_123",
      "name": "ABC 公司",
      "subdomain": "abc"
    }
  }
}
```

#### 2.2.2 更新用户信息

**PATCH** `/api/user`

**请求体：**
```json
{
  "name": "新名称",
  "avatar_url": "https://..."
}
```

**响应：**
```json
{
  "success": true,
  "data": {
    "id": "user_admin_456",
    "email": "admin@abc.com",
    "name": "新名称",
    "avatar_url": "https://..."
  }
}
```

#### 2.2.3 修改密码

**POST** `/api/user/password`

**请求体：**
```json
{
  "current_password": "OldPassword123",
  "new_password": "NewPassword123"
}
```

**响应：**
```json
{
  "success": true,
  "data": {
    "message": "密码修改成功"
  }
}
```

#### 2.2.4 刷新 Token

**POST** `/api/auth/refresh`

**请求体：**
```json
{
  "refresh_token": "refresh_token_xxx"
}
```

**响应：**
```json
{
  "success": true,
  "data": {
    "token": "new_jwt_token",
    "expires_in": 86400
  }
}
```

### 2.3 团队管理 API

#### 2.3.1 获取团队成员列表

**GET** `/api/team/members`

**查询参数：**
- `page`: 页码（默认 1）
- `limit`: 每页数量（默认 20）
- `search`: 搜索关键词（可选）

**响应：**
```json
{
  "success": true,
  "data": {
    "members": [
      {
        "id": "user_admin_456",
        "email": "admin@abc.com",
        "name": "管理员",
        "avatar_url": "https://...",
        "roles": ["super_admin"],
        "joined_at": "2024-01-01T00:00:00Z"
      }
    ]
  },
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 10,
    "total_pages": 1
  }
}
```

#### 2.3.2 邀请成员

**POST** `/api/team/members/invite`

**请求体：**
```json
{
  "email": "member@abc.com",
  "role_id": "role_member_789"
}
```

**响应：**
```json
{
  "success": true,
  "data": {
    "message": "邀请已发送",
    "member": {
      "id": "user_member_999",
      "email": "member@abc.com",
      "status": "pending"
    }
  }
}
```

#### 2.3.3 更新成员角色

**PATCH** `/api/team/members/{member_id}/role`

**请求体：**
```json
{
  "role_id": "role_admin_888"
}
```

**响应：**
```json
{
  "success": true,
  "data": {
    "member": {
      "id": "user_member_999",
      "roles": ["admin"]
    }
  }
}
```

#### 2.3.4 移除成员

**DELETE** `/api/team/members/{member_id}`

**响应：**
```json
{
  "success": true,
  "data": {
    "message": "成员已移除"
  }
}
```

### 2.4 订阅管理 API

#### 2.4.1 获取套餐列表

**GET** `/api/plans`

**响应：**
```json
{
  "success": true,
  "data": {
    "plans": [
      {
        "id": "plan_basic_123",
        "name": "basic",
        "display_name": "基础版",
        "price_monthly": 9900,
        "price_yearly": 99000,
        "features": ["all_features", "email_support"],
        "limits": {
          "max_users": 20,
          "max_workspaces": -1,
          "max_storage": 10
        }
      }
    ]
  }
}
```

#### 2.4.2 获取当前订阅

**GET** `/api/subscription`

**响应：**
```json
{
  "success": true,
  "data": {
    "subscription": {
      "id": "sub_abc_123",
      "plan": {
        "id": "plan_basic_123",
        "name": "basic",
        "display_name": "基础版"
      },
      "status": "active",
      "current_period_start": "2024-01-01T00:00:00Z",
      "current_period_end": "2024-02-01T00:00:00Z",
      "cancel_at_period_end": false
    },
    "usage": {
      "users": 10,
      "workspaces": 5,
      "storage_gb": 2.5
    }
  }
}
```

#### 2.4.3 创建订阅

**POST** `/api/subscription`

**请求体：**
```json
{
  "plan_id": "plan_basic_123",
  "payment_method_id": "pm_xxx"
}
```

**响应：**
```json
{
  "success": true,
  "data": {
    "subscription": {
      "id": "sub_abc_123",
      "status": "active"
    },
    "message": "订阅创建成功"
  }
}
```

#### 2.4.4 取消订阅

**POST** `/api/subscription/cancel`

**请求体：**
```json
{
  "immediately": false
}
```

**响应：**
```json
{
  "success": true,
  "data": {
    "message": "订阅将在周期结束时取消",
    "cancel_at": "2024-02-01T00:00:00Z"
  }
}
```

### 2.5 工作空间 API

#### 2.5.1 获取工作空间列表

**GET** `/api/workspaces`

**查询参数：**
- `page`: 页码
- `limit`: 每页数量
- `search`: 搜索关键词

**响应：**
```json
{
  "success": true,
  "data": {
    "workspaces": [
      {
        "id": "workspace_xyz_123",
        "name": "市场部",
        "description": "市场部的工作空间",
        "owner": {
          "id": "user_admin_456",
          "name": "管理员"
        },
        "member_count": 5,
        "created_at": "2024-01-01T00:00:00Z"
      }
    ]
  },
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 10
  }
}
```

#### 2.5.2 创建工作空间

**POST** `/api/workspaces`

**请求体：**
```json
{
  "name": "市场部",
  "description": "市场部的工作空间"
}
```

**响应：**
```json
{
  "success": true,
  "data": {
    "workspace": {
      "id": "workspace_xyz_123",
      "name": "市场部",
      "description": "市场部的工作空间",
      "owner": {
        "id": "user_admin_456",
        "name": "管理员"
      },
      "created_at": "2024-01-01T00:00:00Z"
    }
  }
}
```

#### 2.5.3 获取工作空间详情

**GET** `/api/workspaces/{workspace_id}`

**响应：**
```json
{
  "success": true,
  "data": {
    "workspace": {
      "id": "workspace_xyz_123",
      "name": "市场部",
      "description": "市场部的工作空间",
      "owner": {
        "id": "user_admin_456",
        "name": "管理员"
      },
      "members": [
        {
          "id": "user_admin_456",
          "name": "管理员",
          "role": "owner"
        }
      ],
      "created_at": "2024-01-01T00:00:00Z"
    }
  }
}
```

#### 2.5.4 更新工作空间

**PATCH** `/api/workspaces/{workspace_id}`

**请求体：**
```json
{
  "name": "新名称",
  "description": "新描述"
}
```

**响应：**
```json
{
  "success": true,
  "data": {
    "workspace": {
      "id": "workspace_xyz_123",
      "name": "新名称",
      "description": "新描述"
    }
  }
}
```

#### 2.5.5 删除工作空间

**DELETE** `/api/workspaces/{workspace_id}`

**响应：**
```json
{
  "success": true,
  "data": {
    "message": "工作空间已删除"
  }
}
```

#### 2.5.6 邀请成员加入工作空间

**POST** `/api/workspaces/{workspace_id}/members/invite`

**请求体：**
```json
{
  "email": "member@abc.com",
  "role": "member"
}
```

**响应：**
```json
{
  "success": true,
  "data": {
    "message": "邀请已发送"
  }
}
```

## 三、错误码定义

### 3.1 认证错误

- `AUTH_REQUIRED`: 需要认证
- `AUTH_INVALID`: Token 无效
- `AUTH_EXPIRED`: Token 已过期
- `AUTH_FAILED`: 认证失败（邮箱或密码错误）

### 3.2 权限错误

- `PERMISSION_DENIED`: 权限不足
- `ROLE_REQUIRED`: 需要特定角色

### 3.3 验证错误

- `VALIDATION_FAILED`: 验证失败
- `EMAIL_INVALID`: 邮箱格式无效
- `PASSWORD_WEAK`: 密码强度不足
- `SUBDOMAIN_EXISTS`: 子域名已存在
- `EMAIL_EXISTS`: 邮箱已存在

### 3.4 资源错误

- `RESOURCE_NOT_FOUND`: 资源不存在
- `RESOURCE_LIMIT_EXCEEDED`: 达到资源限制
- `SUBSCRIPTION_REQUIRED`: 需要订阅

### 3.5 系统错误

- `INTERNAL_ERROR`: 内部错误
- `SERVICE_UNAVAILABLE`: 服务不可用

## 四、API 版本管理

### 4.1 版本策略

- URL 路径版本：`/api/v1/...`
- 当前版本：v1
- 向后兼容：新版本保持向后兼容至少 6 个月

### 4.2 版本升级

- 新功能使用新版本：`/api/v2/...`
- 旧版本继续支持
- 逐步迁移用户到新版本

## 五、API 文档

### 5.1 OpenAPI 规范

- 使用 OpenAPI 3.1 规范
- 自动生成 API 文档
- 提供 Swagger UI 界面

### 5.2 文档内容

- 所有端点的详细说明
- 请求和响应示例
- 错误码说明
- 认证方式说明
