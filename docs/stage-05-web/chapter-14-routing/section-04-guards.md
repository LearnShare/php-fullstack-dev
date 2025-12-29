# 5.14.4 路由守卫与权限控制

## 概述

路由守卫用于控制路由的访问权限，是构建安全 Web 应用的重要机制。理解路由守卫的概念、掌握认证和权限守卫的实现、集成中间件，对于实现安全的路由访问控制至关重要。本节详细介绍路由守卫概念、认证守卫、权限守卫、守卫实现、中间件集成等内容，帮助零基础学员实现安全的路由访问控制。

路由守卫是路由系统的重要组成部分，通过守卫可以实现认证检查、权限验证、访问控制等功能。理解路由守卫的实现原理对于构建安全的 Web 应用至关重要。

**主要内容**：
- 路由守卫概念（什么是路由守卫、守卫的作用、守卫的类型）
- 认证守卫（登录检查、会话验证、Token 验证、认证失败处理）
- 权限守卫（权限检查、角色检查、资源权限、权限失败处理）
- 守卫实现（守卫类设计、守卫注册、守卫执行、守卫组合）
- 中间件集成（中间件实现守卫、守卫中间件、守卫链）
- 实际应用示例和最佳实践

## 特性

- **安全控制**：提供安全的路由访问控制
- **灵活配置**：支持灵活的守卫配置
- **易于使用**：简洁的 API
- **可组合**：可以组合多个守卫
- **可扩展**：易于扩展新守卫

## 路由守卫概念

### 什么是路由守卫

路由守卫（Route Guard）是控制路由访问权限的机制，在路由处理之前检查访问权限。

### 守卫的作用

1. **访问控制**：控制路由的访问权限
2. **认证检查**：检查用户是否已认证
3. **权限验证**：验证用户权限
4. **资源保护**：保护敏感资源

### 守卫的类型

**主要类型**：
- **认证守卫**：检查用户是否已认证
- **权限守卫**：检查用户权限
- **角色守卫**：检查用户角色
- **资源守卫**：检查资源访问权限

## 认证守卫

### 登录检查

**示例**：
```php
<?php
declare(strict_types=1);

class AuthGuard
{
    public function check($request): bool
    {
        session_start();
        return isset($_SESSION['user_id']);
    }

    public function handleFailure(): ResponseInterface
    {
        return new Response('Unauthorized', 401);
    }
}

// 使用
Route::get('/dashboard', 'DashboardController@index')
    ->guard(new AuthGuard());
```

### 会话验证

**示例**：
```php
<?php
declare(strict_types=1);

class SessionAuthGuard
{
    public function check($request): bool
    {
        session_start();
        
        if (!isset($_SESSION['user_id'])) {
            return false;
        }
        
        // 验证会话有效性
        if (isset($_SESSION['expires']) && $_SESSION['expires'] < time()) {
            return false;
        }
        
        return true;
    }
}
```

### Token 验证

**示例**：
```php
<?php
declare(strict_types=1);

class TokenAuthGuard
{
    public function check($request): bool
    {
        $token = $this->extractToken($request);
        
        if ($token === null) {
            return false;
        }
        
        return $this->validateToken($token);
    }

    private function extractToken($request): ?string
    {
        $header = $request->getHeaderLine('Authorization');
        if (preg_match('/Bearer\s+(.*)$/i', $header, $matches)) {
            return $matches[1];
        }
        return null;
    }

    private function validateToken(string $token): bool
    {
        // Token 验证逻辑
        return true;
    }
}
```

### 认证失败处理

**示例**：
```php
<?php
declare(strict_types=1);

class AuthGuard
{
    public function check($request): bool
    {
        return isAuthenticated($request);
    }

    public function handleFailure($request): ResponseInterface
    {
        // 重定向到登录页
        if ($request->getHeaderLine('Accept') === 'application/json') {
            return new Response(
                json_encode(['error' => 'Unauthorized']),
                401,
                ['Content-Type' => 'application/json']
            );
        }
        
        return new Response('', 302, [
            'Location' => '/login',
        ]);
    }
}
```

## 权限守卫

### 权限检查

**示例**：
```php
<?php
declare(strict_types=1);

class PermissionGuard
{
    public function __construct(
        private string $permission
    ) {}

    public function check($request): bool
    {
        $user = getCurrentUser($request);
        
        if ($user === null) {
            return false;
        }
        
        return $user->hasPermission($this->permission);
    }

    public function handleFailure($request): ResponseInterface
    {
        return new Response('Forbidden', 403);
    }
}

// 使用
Route::get('/admin/users', 'AdminController@users')
    ->guard(new PermissionGuard('users.manage'));
```

### 角色检查

**示例**：
```php
<?php
declare(strict_types=1);

class RoleGuard
{
    public function __construct(
        private array $allowedRoles
    ) {}

    public function check($request): bool
    {
        $user = getCurrentUser($request);
        
        if ($user === null) {
            return false;
        }
        
        return in_array($user->getRole(), $this->allowedRoles, true);
    }

    public function handleFailure($request): ResponseInterface
    {
        return new Response('Forbidden', 403);
    }
}

// 使用
Route::get('/admin/dashboard', 'AdminController@dashboard')
    ->guard(new RoleGuard(['admin', 'super_admin']));
```

### 资源权限

**示例**：
```php
<?php
declare(strict_types=1);

class ResourcePermissionGuard
{
    public function check($request): bool
    {
        $user = getCurrentUser($request);
        $resourceId = $request->getAttribute('id');
        
        if ($user === null || $resourceId === null) {
            return false;
        }
        
        return $this->checkResourcePermission($user, $resourceId);
    }

    private function checkResourcePermission($user, $resourceId): bool
    {
        // 检查用户是否有权限访问该资源
        return $user->canAccessResource($resourceId);
    }
}
```

### 权限失败处理

**示例**：
```php
<?php
declare(strict_types=1);

class PermissionGuard
{
    public function handleFailure($request): ResponseInterface
    {
        if ($request->getHeaderLine('Accept') === 'application/json') {
            return new Response(
                json_encode(['error' => 'Forbidden', 'message' => 'You do not have permission to access this resource']),
                403,
                ['Content-Type' => 'application/json']
            );
        }
        
        return new Response('Forbidden', 403);
    }
}
```

## 守卫实现

### 守卫类设计

**示例**：
```php
<?php
declare(strict_types=1);

interface GuardInterface
{
    public function check($request): bool;
    public function handleFailure($request): ResponseInterface;
}

abstract class BaseGuard implements GuardInterface
{
    public function handleFailure($request): ResponseInterface
    {
        return new Response('Access Denied', 403);
    }
}

class AuthGuard extends BaseGuard
{
    public function check($request): bool
    {
        return isAuthenticated($request);
    }

    public function handleFailure($request): ResponseInterface
    {
        return new Response('Unauthorized', 401);
    }
}
```

### 守卫注册

**示例**：
```php
<?php
declare(strict_types=1);

class Route
{
    private array $guards = [];

    public function guard(GuardInterface $guard): self
    {
        $this->guards[] = $guard;
        return $this;
    }

    public function checkGuards($request): ?ResponseInterface
    {
        foreach ($this->guards as $guard) {
            if (!$guard->check($request)) {
                return $guard->handleFailure($request);
            }
        }
        return null;
    }
}

// 使用
Route::get('/admin/users', 'AdminController@users')
    ->guard(new AuthGuard())
    ->guard(new PermissionGuard('users.manage'));
```

### 守卫执行

**示例**：
```php
<?php
declare(strict_types=1);

function dispatchRoute(Route $route, $request)
{
    // 执行守卫检查
    $guardResponse = $route->checkGuards($request);
    if ($guardResponse !== null) {
        return $guardResponse;
    }
    
    // 守卫通过，执行路由处理器
    return $route->getHandler()($request);
}
```

### 守卫组合

**示例**：
```php
<?php
declare(strict_types=1);

class CompositeGuard implements GuardInterface
{
    private array $guards = [];

    public function add(GuardInterface $guard): self
    {
        $this->guards[] = $guard;
        return $this;
    }

    public function check($request): bool
    {
        foreach ($this->guards as $guard) {
            if (!$guard->check($request)) {
                return false;
            }
        }
        return true;
    }

    public function handleFailure($request): ResponseInterface
    {
        // 返回第一个失败的守卫的响应
        foreach ($this->guards as $guard) {
            if (!$guard->check($request)) {
                return $guard->handleFailure($request);
            }
        }
        return new Response('Access Denied', 403);
    }
}

// 使用
$guard = (new CompositeGuard())
    ->add(new AuthGuard())
    ->add(new PermissionGuard('users.manage'));

Route::get('/admin/users', 'AdminController@users')
    ->guard($guard);
```

## 中间件集成

### 中间件实现守卫

**示例**：
```php
<?php
declare(strict_types=1);

use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

class AuthMiddleware implements MiddlewareInterface
{
    public function process($request, RequestHandlerInterface $handler)
    {
        if (!isAuthenticated($request)) {
            return new Response('Unauthorized', 401);
        }
        
        return $handler->handle($request);
    }
}

// 使用
Route::get('/dashboard', 'DashboardController@index')
    ->middleware(new AuthMiddleware());
```

### 守卫中间件

**示例**：
```php
<?php
declare(strict_types=1);

class GuardMiddleware implements MiddlewareInterface
{
    public function __construct(
        private GuardInterface $guard
    ) {}

    public function process($request, RequestHandlerInterface $handler)
    {
        if (!$this->guard->check($request)) {
            return $this->guard->handleFailure($request);
        }
        
        return $handler->handle($request);
    }
}

// 使用
Route::get('/admin/users', 'AdminController@users')
    ->middleware(new GuardMiddleware(new PermissionGuard('users.manage')));
```

### 守卫链

**示例**：
```php
<?php
declare(strict_types=1);

class GuardChainMiddleware implements MiddlewareInterface
{
    private array $guards = [];

    public function add(GuardInterface $guard): self
    {
        $this->guards[] = $guard;
        return $this;
    }

    public function process($request, RequestHandlerInterface $handler)
    {
        foreach ($this->guards as $guard) {
            if (!$guard->check($request)) {
                return $guard->handleFailure($request);
            }
        }
        
        return $handler->handle($request);
    }
}

// 使用
$middleware = (new GuardChainMiddleware())
    ->add(new AuthGuard())
    ->add(new PermissionGuard('users.manage'));

Route::get('/admin/users', 'AdminController@users')
    ->middleware($middleware);
```

## 使用场景

### 权限控制

- 页面访问控制
- 功能访问控制
- 数据访问控制

### 认证检查

- 登录状态检查
- 会话验证
- Token 验证

### 资源保护

- 敏感资源保护
- 数据保护
- API 保护

### 访问控制

- 基于角色的访问控制
- 基于权限的访问控制
- 基于资源的访问控制

## 注意事项

### 守卫顺序

- **重要**：守卫的执行顺序很重要
- **认证在前**：认证守卫应该在权限守卫之前
- **快速失败**：快速失败的守卫应该在前

### 性能考虑

- **缓存权限**：缓存权限检查结果
- **减少查询**：减少数据库查询
- **优化检查**：优化守卫检查逻辑

### 错误处理

- **清晰错误**：提供清晰的错误信息
- **错误日志**：记录访问失败日志
- **错误响应**：返回适当的错误响应

### 权限缓存

- **缓存策略**：实现权限缓存策略
- **缓存失效**：处理缓存失效
- **缓存更新**：及时更新缓存

## 常见问题

### 如何实现路由守卫？

实现 GuardInterface 接口，定义 check 和 handleFailure 方法。

### 认证和权限守卫的区别？

- **认证守卫**：检查用户是否已认证
- **权限守卫**：检查用户是否有权限

### 如何组合多个守卫？

使用 CompositeGuard 或 GuardChainMiddleware 组合多个守卫。

### 守卫的性能影响？

通过缓存权限检查结果、优化检查逻辑等方法减少性能影响。

## 最佳实践

### 使用中间件实现守卫

- 使用中间件实现守卫功能
- 保持守卫简单
- 易于测试和维护

### 实现权限缓存

- 缓存权限检查结果
- 设置合理的缓存时间
- 处理缓存失效

### 提供清晰的错误信息

- 区分认证错误和权限错误
- 提供清晰的错误消息
- 记录访问失败日志

### 记录访问日志

- 记录访问尝试
- 记录访问失败
- 用于安全审计

## 相关章节

- **[5.14.1 路由设计概述](section-01-overview.md)**：了解路由设计概述的详细内容
- **[5.14.2 路由组织与模块化](section-02-organization.md)**：了解路由组织的详细内容
- **[5.14.3 动态路由与参数](section-03-dynamic.md)**：了解动态路由的详细内容
- **[5.10.1 认证基础（AuthN）](../chapter-10-auth/section-01-authentication.md)**：了解认证的详细内容
- **[5.10.2 授权模型（AuthZ）](../chapter-10-auth/section-02-authorization.md)**：了解授权的详细内容

## 练习任务

1. **实现认证守卫**
   - 登录检查
   - 会话验证
   - Token 验证

2. **实现权限守卫**
   - 权限检查
   - 角色检查
   - 资源权限

3. **实现守卫系统**
   - 守卫注册
   - 守卫执行
   - 守卫组合

4. **集成中间件**
   - 中间件实现守卫
   - 守卫中间件
   - 守卫链

5. **实现完整的路由守卫系统**
   - 认证和权限守卫
   - 守卫管理和执行
   - 权限缓存和日志
