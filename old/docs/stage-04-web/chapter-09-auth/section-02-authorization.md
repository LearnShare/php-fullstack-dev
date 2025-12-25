# 4.9.2 授权模型（AuthZ）

## 概述

授权（Authorization）是验证用户权限的过程。本节详细介绍 RBAC（基于角色的访问控制）、ABAC（基于属性的访问控制）、权限检查，以及完整的授权实现。

## RBAC（基于角色的访问控制）

### 核心概念

- **角色（Role）**：用户所属的角色
- **权限（Permission）**：具体的操作权限
- **角色-权限关联**：角色拥有多个权限

### 数据库设计

```sql
-- 用户表
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(255),
    role_id INT
);

-- 角色表
CREATE TABLE roles (
    id INT PRIMARY KEY,
    name VARCHAR(255)
);

-- 权限表
CREATE TABLE permissions (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    resource VARCHAR(255),
    action VARCHAR(255)
);

-- 角色权限关联表
CREATE TABLE role_permissions (
    role_id INT,
    permission_id INT,
    PRIMARY KEY (role_id, permission_id)
);
```

### RBAC 实现

```php
<?php
declare(strict_types=1);

class RBAC
{
    private \PDO $db;

    public function __construct(\PDO $db)
    {
        $this->db = $db;
    }

    public function hasPermission(int $userId, string $resource, string $action): bool
    {
        $stmt = $this->db->prepare('
            SELECT COUNT(*) 
            FROM users u
            JOIN roles r ON u.role_id = r.id
            JOIN role_permissions rp ON r.id = rp.role_id
            JOIN permissions p ON rp.permission_id = p.id
            WHERE u.id = ? AND p.resource = ? AND p.action = ?
        ');
        $stmt->execute([$userId, $resource, $action]);
        
        return $stmt->fetchColumn() > 0;
    }

    public function getUserRoles(int $userId): array
    {
        $stmt = $this->db->prepare('
            SELECT r.name 
            FROM users u
            JOIN roles r ON u.role_id = r.id
            WHERE u.id = ?
        ');
        $stmt->execute([$userId]);
        return $stmt->fetchAll(\PDO::FETCH_COLUMN);
    }
}
```

## ABAC（基于属性的访问控制）

### 核心概念

- **属性（Attribute）**：用户、资源、环境的属性
- **策略（Policy）**：基于属性的访问规则
- **动态决策**：根据属性动态判断权限

### ABAC 实现

```php
<?php
declare(strict_types=1);

class ABAC
{
    public function checkAccess(
        array $userAttributes,
        array $resourceAttributes,
        array $environmentAttributes
    ): bool {
        // 策略：用户是资源所有者
        if ($userAttributes['id'] === $resourceAttributes['owner_id']) {
            return true;
        }
        
        // 策略：用户是管理员
        if (in_array('admin', $userAttributes['roles'], true)) {
            return true;
        }
        
        // 策略：工作时间访问
        $hour = (int) date('H');
        if ($hour >= 9 && $hour <= 18) {
            return true;
        }
        
        return false;
    }
}
```

## 权限检查

### 中间件实现

```php
<?php
declare(strict_types=1);

class AuthorizationMiddleware
{
    private RBAC $rbac;

    public function __construct(RBAC $rbac)
    {
        $this->rbac = $rbac;
    }

    public function handle(string $resource, string $action): void
    {
        $userId = $this->getCurrentUserId();
        
        if (!$this->rbac->hasPermission($userId, $resource, $action)) {
            http_response_code(403);
            echo json_encode(['error' => 'Forbidden']);
            exit;
        }
    }

    private function getCurrentUserId(): int
    {
        // 从 Token 或 Session 获取用户 ID
        return 1;
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class AuthorizationService
{
    private RBAC $rbac;

    public function canAccess(int $userId, string $resource, string $action): bool
    {
        return $this->rbac->hasPermission($userId, $resource, $action);
    }

    public function requirePermission(string $resource, string $action): void
    {
        $userId = $this->getCurrentUserId();
        
        if (!$this->canAccess($userId, $resource, $action)) {
            throw new ForbiddenException("No permission to {$action} {$resource}");
        }
    }
}
```

## 注意事项

1. **最小权限原则**：只授予必要的权限
2. **权限缓存**：缓存权限检查结果，提升性能
3. **权限继承**：支持角色继承和权限继承
4. **审计日志**：记录权限检查结果

## 练习

1. 实现一个完整的 RBAC 系统，包含角色和权限管理。

2. 创建一个权限检查中间件，自动验证用户权限。

3. 实现 ABAC 系统，支持基于属性的动态权限判断。

4. 设计一个权限管理系统，支持权限的增删改查。
