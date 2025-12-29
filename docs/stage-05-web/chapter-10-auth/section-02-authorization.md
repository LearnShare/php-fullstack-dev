# 5.10.2 授权模型（AuthZ）

## 概述

授权（Authorization，AuthZ）是控制用户访问权限的过程，是认证之后的下一步。理解授权的概念、掌握 RBAC（基于角色的访问控制）和 ACL（访问控制列表）等授权模型、实现权限检查机制，对于构建安全、灵活的权限管理系统至关重要。本节详细介绍授权的概念、RBAC 模型、ACL 模型、权限检查、权限继承等内容，帮助零基础学员掌握权限控制技术。

授权是确认"你能做什么"的过程，与认证（确认"你是谁"）不同。授权系统需要灵活、可扩展、易于管理，同时要保证安全性。

**主要内容**：
- 授权概念（什么是授权、授权与认证的区别、授权的粒度）
- RBAC 模型（角色概念、权限概念、角色-权限关联、用户-角色关联）
- ACL 模型（ACL 概念、资源-权限关联、用户-资源-权限、ACL 实现）
- 权限检查（权限验证方法、中间件检查、装饰器模式、权限缓存）
- 权限继承（角色继承、权限继承、继承规则）
- 实际应用示例和最佳实践

## 特性

- **灵活模型**：支持 RBAC、ACL 等多种模型
- **细粒度控制**：支持资源级别的权限控制
- **易于扩展**：支持权限继承和组合
- **高性能**：支持权限缓存
- **易于管理**：提供清晰的权限管理接口

## 授权概念

### 什么是授权

授权（Authorization）是控制用户访问权限的过程，决定用户能够访问哪些资源和执行哪些操作。

### 授权与认证的区别

| 特性 | 认证（Authentication） | 授权（Authorization） |
|:-----|:----------------------|:----------------------|
| 目的 | 验证用户身份 | 控制访问权限 |
| 问题 | "你是谁？" | "你能做什么？" |
| 时机 | 登录时 | 访问资源时 |
| 结果 | 确认身份 | 允许或拒绝访问 |

### 授权的粒度

**不同粒度**：
1. **页面级**：控制页面访问
2. **功能级**：控制功能访问
3. **数据级**：控制数据访问
4. **字段级**：控制字段访问

## RBAC 模型

### 角色概念

**角色（Role）**：一组权限的集合，用于简化权限管理。

**示例角色**：
- **管理员**：拥有所有权限
- **编辑**：可以创建和编辑内容
- **查看者**：只能查看内容

### 权限概念

**权限（Permission）**：对资源的操作权限。

**示例权限**：
- `users.create`：创建用户
- `users.read`：查看用户
- `users.update`：更新用户
- `users.delete`：删除用户

### 角色-权限关联

**数据库设计**：
```sql
-- 角色表
CREATE TABLE roles (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    description TEXT
);

-- 权限表
CREATE TABLE permissions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    resource VARCHAR(50) NOT NULL,
    action VARCHAR(50) NOT NULL
);

-- 角色-权限关联表
CREATE TABLE role_permissions (
    role_id INT,
    permission_id INT,
    PRIMARY KEY (role_id, permission_id),
    FOREIGN KEY (role_id) REFERENCES roles(id),
    FOREIGN KEY (permission_id) REFERENCES permissions(id)
);
```

### 用户-角色关联

**数据库设计**：
```sql
-- 用户-角色关联表
CREATE TABLE user_roles (
    user_id INT,
    role_id INT,
    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (role_id) REFERENCES roles(id)
);
```

### RBAC 实现示例

**示例**：
```php
<?php
declare(strict_types=1);

class RBAC
{
    public function hasPermission(int $userId, string $permission): bool
    {
        // 1. 获取用户角色
        $roles = $this->getUserRoles($userId);
        
        // 2. 获取角色权限
        $permissions = $this->getRolePermissions($roles);
        
        // 3. 检查权限
        return in_array($permission, $permissions, true);
    }

    private function getUserRoles(int $userId): array
    {
        // 从数据库获取用户角色
        // SELECT role_id FROM user_roles WHERE user_id = ?
        return [];
    }

    private function getRolePermissions(array $roleIds): array
    {
        if (empty($roleIds)) {
            return [];
        }
        
        // 从数据库获取角色权限
        // SELECT permission_id FROM role_permissions WHERE role_id IN (?)
        return [];
    }
}
```

## ACL 模型

### ACL 概念

**ACL（Access Control List）**：访问控制列表，直接关联用户和资源权限。

### 资源-权限关联

**数据库设计**：
```sql
-- ACL 表
CREATE TABLE acl (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    resource_type VARCHAR(50) NOT NULL,
    resource_id INT,
    permission VARCHAR(50) NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### 用户-资源-权限

**示例**：
```php
<?php
declare(strict_types=1);

class ACL
{
    public function hasPermission(int $userId, string $resourceType, int $resourceId, string $permission): bool
    {
        // 检查 ACL
        $stmt = $this->db->prepare(
            'SELECT COUNT(*) FROM acl 
             WHERE user_id = ? AND resource_type = ? AND resource_id = ? AND permission = ?'
        );
        $stmt->execute([$userId, $resourceType, $resourceId, $permission]);
        return $stmt->fetchColumn() > 0;
    }

    public function grantPermission(int $userId, string $resourceType, int $resourceId, string $permission): void
    {
        $stmt = $this->db->prepare(
            'INSERT INTO acl (user_id, resource_type, resource_id, permission) 
             VALUES (?, ?, ?, ?)'
        );
        $stmt->execute([$userId, $resourceType, $resourceId, $permission]);
    }

    public function revokePermission(int $userId, string $resourceType, int $resourceId, string $permission): void
    {
        $stmt = $this->db->prepare(
            'DELETE FROM acl 
             WHERE user_id = ? AND resource_type = ? AND resource_id = ? AND permission = ?'
        );
        $stmt->execute([$userId, $resourceType, $resourceId, $permission]);
    }
}
```

### ACL 实现

**完整示例**：
```php
<?php
declare(strict_types=1);

class ACLManager
{
    public function checkPermission(int $userId, string $resource, string $action): bool
    {
        // 1. 检查直接权限
        if ($this->hasDirectPermission($userId, $resource, $action)) {
            return true;
        }

        // 2. 检查角色权限
        $roles = $this->getUserRoles($userId);
        foreach ($roles as $role) {
            if ($this->hasRolePermission($role, $resource, $action)) {
                return true;
            }
        }

        return false;
    }

    private function hasDirectPermission(int $userId, string $resource, string $action): bool
    {
        $stmt = $this->db->prepare(
            'SELECT COUNT(*) FROM acl 
             WHERE user_id = ? AND resource = ? AND action = ?'
        );
        $stmt->execute([$userId, $resource, $action]);
        return $stmt->fetchColumn() > 0;
    }
}
```

## 权限检查

### 权限验证方法

**示例**：
```php
<?php
declare(strict_types=1);

function checkPermission(string $permission): bool
{
    $user = getCurrentUser();
    if ($user === null) {
        return false;
    }
    
    return $user->hasPermission($permission);
}

// 使用
if (checkPermission('users.create')) {
    // 允许创建用户
} else {
    http_response_code(403);
    echo json_encode(['error' => 'Permission denied']);
    exit;
}
```

### 中间件检查

**示例**：
```php
<?php
declare(strict_types=1);

class PermissionMiddleware
{
    public function __construct(private array $requiredPermissions) {}

    public function handle(): void
    {
        $user = $this->getCurrentUser();
        if ($user === null) {
            http_response_code(401);
            echo json_encode(['error' => 'Unauthorized']);
            exit;
        }

        foreach ($this->requiredPermissions as $permission) {
            if (!$user->hasPermission($permission)) {
                http_response_code(403);
                echo json_encode(['error' => 'Permission denied']);
                exit;
            }
        }
    }

    private function getCurrentUser(): ?User
    {
        // 获取当前用户
        // ...
        return null;
    }
}

// 使用
$middleware = new PermissionMiddleware(['users.create']);
$middleware->handle();
```

### 装饰器模式

**示例**：
```php
<?php
declare(strict_types=1);

function requirePermission(string $permission): callable
{
    return function (callable $handler) use ($permission) {
        return function () use ($permission, $handler) {
            if (!checkPermission($permission)) {
                http_response_code(403);
                echo json_encode(['error' => 'Permission denied']);
                exit;
            }
            return $handler();
        };
    };
}

// 使用
$handler = requirePermission('users.create')(function () {
    // 创建用户的逻辑
    return ['success' => true];
});
```

### 权限缓存

**示例**：
```php
<?php
declare(strict_types=1);

class CachedPermissionChecker
{
    private array $cache = [];

    public function hasPermission(int $userId, string $permission): bool
    {
        $cacheKey = "user_{$userId}_permission_{$permission}";
        
        if (isset($this->cache[$cacheKey])) {
            return $this->cache[$cacheKey];
        }

        $hasPermission = $this->checkPermission($userId, $permission);
        $this->cache[$cacheKey] = $hasPermission;
        
        return $hasPermission;
    }

    public function clearCache(int $userId): void
    {
        foreach (array_keys($this->cache) as $key) {
            if (str_starts_with($key, "user_{$userId}_")) {
                unset($this->cache[$key]);
            }
        }
    }

    private function checkPermission(int $userId, string $permission): bool
    {
        // 实际的权限检查逻辑
        // ...
        return false;
    }
}
```

## 权限继承

### 角色继承

**示例**：
```php
<?php
declare(strict_types=1);

class RoleHierarchy
{
    private array $hierarchy = [
        'admin' => ['editor', 'viewer'],
        'editor' => ['viewer'],
    ];

    public function getInheritedRoles(string $role): array
    {
        $inherited = [];
        $this->collectInherited($role, $inherited);
        return $inherited;
    }

    private function collectInherited(string $role, array &$inherited): void
    {
        if (isset($this->hierarchy[$role])) {
            foreach ($this->hierarchy[$role] as $childRole) {
                if (!in_array($childRole, $inherited, true)) {
                    $inherited[] = $childRole;
                    $this->collectInherited($childRole, $inherited);
                }
            }
        }
    }
}
```

### 权限继承

**示例**：
```php
<?php
declare(strict_types=1);

class PermissionInheritance
{
    public function getEffectivePermissions(int $userId): array
    {
        // 1. 获取直接权限
        $directPermissions = $this->getDirectPermissions($userId);
        
        // 2. 获取角色权限
        $rolePermissions = $this->getRolePermissions($userId);
        
        // 3. 获取继承权限
        $inheritedPermissions = $this->getInheritedPermissions($userId);
        
        // 4. 合并权限（去重）
        return array_unique(array_merge($directPermissions, $rolePermissions, $inheritedPermissions));
    }
}
```

## 使用场景

### 多用户系统

- 不同用户不同权限
- 角色管理
- 权限分配

### 内容管理系统

- 内容创建权限
- 内容编辑权限
- 内容删除权限

### 企业应用

- 部门权限
- 职位权限
- 数据权限

## 注意事项

### 权限粒度设计

- **合理粒度**：不要过细或过粗
- **易于管理**：权限易于理解和管理
- **性能考虑**：权限检查的性能影响

### 性能考虑

- **权限缓存**：缓存权限检查结果
- **批量检查**：批量检查权限
- **数据库优化**：优化权限查询

### 权限缓存

- **缓存策略**：合理的缓存策略
- **缓存失效**：权限变更时清除缓存
- **缓存更新**：及时更新缓存

### 权限继承

- **继承规则**：清晰的继承规则
- **循环继承**：避免循环继承
- **性能影响**：继承计算的性能

## 常见问题

### 认证和授权的区别？

- **认证**：验证用户身份（"你是谁"）
- **授权**：控制访问权限（"你能做什么"）

### RBAC 和 ACL 的区别？

| 特性 | RBAC | ACL |
|:-----|:-----|:-----|
| 模型 | 基于角色 | 基于资源 |
| 粒度 | 角色级别 | 资源级别 |
| 管理 | 易于管理 | 灵活但复杂 |
| 适用场景 | 通用权限 | 细粒度权限 |

### 如何设计权限系统？

1. **确定权限粒度**：页面、功能、数据
2. **选择模型**：RBAC 或 ACL
3. **设计数据结构**：角色、权限、关联表
4. **实现检查机制**：权限检查、中间件

### 如何检查权限？

1. **获取用户权限**：从数据库或缓存获取
2. **检查权限**：验证是否有权限
3. **返回结果**：允许或拒绝访问

## 最佳实践

### 使用 RBAC 模型

- 使用角色简化权限管理
- 角色-权限分离
- 用户-角色关联

### 实现权限缓存

- 缓存权限检查结果
- 权限变更时清除缓存
- 使用 Redis 等缓存系统

### 提供权限检查中间件

- 创建权限检查中间件
- 统一权限检查逻辑
- 提供清晰的错误信息

### 记录权限操作日志

- 记录权限变更
- 记录权限检查
- 审计追踪

## 相关章节

- **[5.10.1 认证基础（AuthN）](section-01-authentication.md)**：了解认证的详细内容
- **[5.13 中间件深入](../chapter-13-middleware/readme.md)**：了解中间件的详细内容

## 练习任务

1. **实现 RBAC 系统**
   - 设计角色和权限表
   - 实现角色-权限关联
   - 实现权限检查

2. **实现 ACL 系统**
   - 设计 ACL 表
   - 实现资源权限管理
   - 实现权限检查

3. **实现权限中间件**
   - 创建权限检查中间件
   - 集成到路由系统
   - 处理权限错误

4. **实现权限缓存**
   - 缓存权限检查结果
   - 实现缓存失效机制
   - 优化权限查询性能

5. **实现完整的权限管理系统**
   - 权限模型设计
   - 权限检查机制
   - 权限管理界面
