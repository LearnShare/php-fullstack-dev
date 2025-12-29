# 5.14.3 动态路由与参数

## 概述

动态路由允许在 URL 中包含参数，是实现灵活路由系统的重要特性。理解动态路由的概念、掌握参数提取和验证、实现参数约束，对于构建灵活、强大的路由系统至关重要。本节详细介绍动态路由概念、路由参数定义、参数提取、参数验证、可选参数、参数约束等内容，帮助零基础学员掌握动态路由技术。

动态路由是现代 Web 框架的核心特性，通过动态路由可以实现 RESTful API、资源路由等功能。理解动态路由的实现原理对于设计和实现路由系统至关重要。

**主要内容**：
- 动态路由概念（什么是动态路由、参数占位符、参数格式）
- 路由参数（必需参数、可选参数、参数命名、参数类型）
- 参数提取（参数匹配、参数获取、参数传递、参数验证）
- 参数验证（类型验证、格式验证、正则约束、自定义约束）
- 可选参数（可选参数定义、可选参数匹配、可选参数处理）
- 参数约束（类型约束、格式约束、正则约束、自定义约束）
- 实际应用示例和最佳实践

## 特性

- **灵活参数**：支持多种参数类型
- **参数验证**：支持参数验证
- **参数约束**：支持参数约束
- **易于使用**：简洁的 API
- **高性能**：高效的参数提取

## 动态路由概念

### 什么是动态路由

动态路由是包含参数占位符的路由，可以从 URL 中提取参数值。

### 参数占位符

**常见占位符格式**：
- `{id}`：简单占位符
- `{id:\d+}`：带约束的占位符
- `{id?}`：可选参数

### 参数格式

**示例**：
```php
<?php
declare(strict_types=1);

// 动态路由示例
GET /users/{id}              // 必需参数
GET /users/{id}/posts/{postId}  // 多个参数
GET /posts/{year}/{month}/{slug}  // 多个参数
GET /users/{id?}             // 可选参数
```

## 路由参数

### 必需参数

**示例**：
```php
<?php
declare(strict_types=1);

// 必需参数
Route::get('/users/{id}', function($id) {
    return getUser($id);
});

// URL: /users/123
// 参数: ['id' => '123']
```

### 可选参数

**示例**：
```php
<?php
declare(strict_types=1);

// 可选参数
Route::get('/users/{id?}', function($id = null) {
    if ($id === null) {
        return getAllUsers();
    }
    return getUser($id);
});

// URL: /users      -> 参数: []
// URL: /users/123  -> 参数: ['id' => '123']
```

### 参数命名

**示例**：
```php
<?php
declare(strict_types=1);

// 有意义的参数名
Route::get('/users/{userId}', function($userId) {
    return getUser($userId);
});

Route::get('/posts/{postId}/comments/{commentId}', function($postId, $commentId) {
    return getComment($postId, $commentId);
});
```

### 参数类型

**示例**：
```php
<?php
declare(strict_types=1);

// 参数类型提示
Route::get('/users/{id}', function(int $id) {
    return getUser($id);
});

Route::get('/posts/{slug}', function(string $slug) {
    return getPost($slug);
});
```

## 参数提取

### 参数匹配

**示例**：
```php
<?php
declare(strict_types=1);

function extractParams(string $pattern, string $path): ?array
{
    // 提取参数名
    preg_match_all('/\{(\w+)\}/', $pattern, $matches);
    $paramNames = $matches[1];
    
    if (empty($paramNames)) {
        return $pattern === $path ? [] : null;
    }
    
    // 构建正则表达式
    $regex = preg_replace('/\{(\w+)\}/', '([^/]+)', $pattern);
    $regex = "#^{$regex}$#";
    
    if (preg_match($regex, $path, $matches)) {
        $params = [];
        for ($i = 0; $i < count($paramNames); $i++) {
            $params[$paramNames[$i]] = $matches[$i + 1];
        }
        return $params;
    }
    
    return null;
}

// 使用
$params = extractParams('/users/{id}', '/users/123');
// ['id' => '123']
```

### 参数获取

**示例**：
```php
<?php
declare(strict_types=1);

class Route
{
    private array $params = [];

    public function getParams(): array
    {
        return $this->params;
    }

    public function getParam(string $name, $default = null)
    {
        return $this->params[$name] ?? $default;
    }

    public function setParams(array $params): void
    {
        $this->params = $params;
    }
}

// 使用
$route = new Route();
$route->setParams(['id' => '123', 'postId' => '456']);

$id = $route->getParam('id');
$postId = $route->getParam('postId');
```

### 参数传递

**示例**：
```php
<?php
declare(strict_types=1);

function dispatchRoute(Route $route, $request)
{
    $handler = $route->getHandler();
    $params = $route->getParams();
    
    // 传递参数到处理器
    if (is_array($handler)) {
        [$controller, $method] = $handler;
        return call_user_func_array([$controller, $method], $params);
    }
    
    return call_user_func_array($handler, $params);
}
```

### 参数验证

**示例**：
```php
<?php
declare(strict_types=1);

function validateParams(array $params, array $rules): array
{
    $errors = [];
    
    foreach ($rules as $name => $rule) {
        $value = $params[$name] ?? null;
        
        if ($value === null && !isset($rule['optional'])) {
            $errors[$name] = "Parameter {$name} is required";
            continue;
        }
        
        if ($value !== null) {
            // 类型验证
            if (isset($rule['type'])) {
                if (!$this->validateType($value, $rule['type'])) {
                    $errors[$name] = "Parameter {$name} must be {$rule['type']}";
                }
            }
            
            // 格式验证
            if (isset($rule['pattern'])) {
                if (!preg_match($rule['pattern'], $value)) {
                    $errors[$name] = "Parameter {$name} format is invalid";
                }
            }
        }
    }
    
    return $errors;
}
```

## 参数验证

### 类型验证

**示例**：
```php
<?php
declare(strict_types=1);

function validateType($value, string $type): bool
{
    return match ($type) {
        'int' => filter_var($value, FILTER_VALIDATE_INT) !== false,
        'string' => is_string($value),
        'float' => filter_var($value, FILTER_VALIDATE_FLOAT) !== false,
        'email' => filter_var($value, FILTER_VALIDATE_EMAIL) !== false,
        'url' => filter_var($value, FILTER_VALIDATE_URL) !== false,
        default => false,
    };
}

// 使用
Route::get('/users/{id}', function($id) {
    if (!validateType($id, 'int')) {
        return new Response('Invalid ID', 400);
    }
    return getUser((int) $id);
});
```

### 格式验证

**示例**：
```php
<?php
declare(strict_types=1);

function validateFormat($value, string $pattern): bool
{
    return (bool) preg_match($pattern, $value);
}

// 使用
Route::get('/posts/{slug}', function($slug) {
    if (!validateFormat($slug, '/^[a-z0-9-]+$/')) {
        return new Response('Invalid slug', 400);
    }
    return getPost($slug);
});
```

### 正则约束

**示例**：
```php
<?php
declare(strict_types=1);

// 在路由定义中使用正则约束
Route::get('/users/{id}', function($id) {
    return getUser($id);
})->where('id', '[0-9]+');

Route::get('/posts/{slug}', function($slug) {
    return getPost($slug);
})->where('slug', '[a-z0-9-]+');
```

### 自定义约束

**示例**：
```php
<?php
declare(strict_types=1);

interface ConstraintInterface
{
    public function validate($value): bool;
}

class IntegerConstraint implements ConstraintInterface
{
    public function validate($value): bool
    {
        return filter_var($value, FILTER_VALIDATE_INT) !== false;
    }
}

class SlugConstraint implements ConstraintInterface
{
    public function validate($value): bool
    {
        return (bool) preg_match('/^[a-z0-9-]+$/', $value);
    }
}

// 使用
Route::get('/users/{id}', function($id) {
    return getUser($id);
})->constraint('id', new IntegerConstraint());
```

## 可选参数

### 可选参数定义

**示例**：
```php
<?php
declare(strict_types=1);

// 可选参数使用 ? 标记
Route::get('/users/{id?}', function($id = null) {
    if ($id === null) {
        return getAllUsers();
    }
    return getUser($id);
});
```

### 可选参数匹配

**示例**：
```php
<?php
declare(strict_types=1);

function matchOptionalParam(string $pattern, string $path): ?array
{
    // 处理可选参数
    $optionalPattern = preg_replace('/\{(\w+)\?\}/', '(?:/([^/]+))?', $pattern);
    $regex = "#^{$optionalPattern}$#";
    
    if (preg_match($regex, $path, $matches)) {
        $params = [];
        preg_match_all('/\{(\w+)\??\}/', $pattern, $paramNames);
        
        foreach ($paramNames[1] as $index => $name) {
            if (isset($matches[$index + 1]) && $matches[$index + 1] !== '') {
                $params[$name] = $matches[$index + 1];
            }
        }
        
        return $params;
    }
    
    return null;
}
```

### 可选参数处理

**示例**：
```php
<?php
declare(strict_types=1);

Route::get('/posts/{year}/{month}/{day?}', function($year, $month, $day = null) {
    if ($day === null) {
        return getPostsByMonth($year, $month);
    }
    return getPostsByDay($year, $month, $day);
});

// URL: /posts/2024/01        -> 参数: ['year' => '2024', 'month' => '01']
// URL: /posts/2024/01/15     -> 参数: ['year' => '2024', 'month' => '01', 'day' => '15']
```

## 参数约束

### 类型约束

**示例**：
```php
<?php
declare(strict_types=1);

Route::get('/users/{id}', function(int $id) {
    return getUser($id);
})->where('id', 'integer');

Route::get('/posts/{slug}', function(string $slug) {
    return getPost($slug);
})->where('slug', 'string');
```

### 格式约束

**示例**：
```php
<?php
declare(strict_types=1);

Route::get('/users/{email}', function(string $email) {
    return getUserByEmail($email);
})->where('email', '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}');
```

### 正则约束

**示例**：
```php
<?php
declare(strict_types=1);

Route::get('/posts/{year}/{month}', function($year, $month) {
    return getPostsByMonth($year, $month);
})->where([
    'year' => '[0-9]{4}',
    'month' => '(0[1-9]|1[0-2])',
]);
```

### 自定义约束

**示例**：
```php
<?php
declare(strict_types=1);

class RouteConstraint
{
    private array $constraints = [];

    public function where(string $param, string|callable $constraint): self
    {
        $this->constraints[$param] = $constraint;
        return $this;
    }

    public function validate(string $param, $value): bool
    {
        if (!isset($this->constraints[$param])) {
            return true;
        }
        
        $constraint = $this->constraints[$param];
        
        if (is_callable($constraint)) {
            return $constraint($value);
        }
        
        return (bool) preg_match("#^{$constraint}$#", $value);
    }
}

// 使用
$route = new Route();
$route->where('id', function($value) {
    return filter_var($value, FILTER_VALIDATE_INT) !== false && (int) $value > 0;
});
```

## 使用场景

### RESTful API

- 资源路由
- 参数化 URL
- RESTful 设计

### 资源路由

- 资源 CRUD 操作
- 嵌套资源
- 资源参数

### 参数化 URL

- 动态内容
- 用户友好 URL
- SEO 友好

### 友好 URL

- 清晰的 URL 结构
- 有意义的参数
- 易于理解

## 注意事项

### 参数验证

- **重要**：始终验证参数
- **类型验证**：验证参数类型
- **格式验证**：验证参数格式
- **安全验证**：防止注入攻击

### 参数类型

- **类型转换**：正确转换参数类型
- **类型提示**：使用类型提示
- **类型安全**：确保类型安全

### 路由冲突

- **避免冲突**：避免路由冲突
- **参数顺序**：注意参数顺序
- **可选参数**：谨慎使用可选参数

### 性能考虑

- **匹配效率**：优化匹配效率
- **参数提取**：优化参数提取
- **缓存使用**：使用路由缓存

## 常见问题

### 如何定义动态路由？

使用参数占位符：`/users/{id}`

### 如何提取路由参数？

使用正则表达式匹配和提取参数。

### 如何验证参数？

使用类型验证、格式验证、正则约束等方法。

### 如何处理可选参数？

使用 `?` 标记可选参数，在处理器中提供默认值。

## 最佳实践

### 使用有意义的参数名

- 使用描述性名称
- 避免缩写
- 保持一致性

### 实现参数验证

- 验证所有参数
- 使用类型验证
- 使用格式验证

### 使用参数约束

- 定义参数约束
- 使用正则表达式
- 自定义约束类

### 提供参数文档

- 文档化参数
- 说明参数类型
- 提供参数示例

## 相关章节

- **[5.14.1 路由设计概述](section-01-overview.md)**：了解路由设计概述的详细内容
- **[5.14.2 路由组织与模块化](section-02-organization.md)**：了解路由组织的详细内容
- **[5.14.4 路由守卫与权限控制](section-04-guards.md)**：了解路由守卫的详细内容

## 练习任务

1. **实现动态路由**
   - 参数占位符支持
   - 参数提取
   - 参数传递

2. **实现参数验证**
   - 类型验证
   - 格式验证
   - 自定义约束

3. **实现可选参数**
   - 可选参数定义
   - 可选参数匹配
   - 可选参数处理

4. **实现参数约束**
   - 类型约束
   - 格式约束
   - 正则约束

5. **实现完整的动态路由系统**
   - 动态路由匹配
   - 参数提取和验证
   - 参数约束和文档
