# 5.14.1 路由设计概述

## 概述

路由是 Web 应用的核心组件，负责将 URL 映射到处理程序。理解路由的概念、掌握路由表设计、了解路由匹配算法，对于构建清晰、可维护的 Web 应用至关重要。本节详细介绍路由概念、路由的作用、路由表设计、路由匹配算法、路由优先级等内容，帮助零基础学员理解路由机制。

路由系统是现代 Web 框架的基础，通过路由可以将 URL 映射到相应的处理程序。理解路由的工作原理对于设计和实现路由系统至关重要。

**主要内容**：
- 路由概念（什么是路由、路由的作用、路由的优势）
- 路由的作用（URL 映射、请求分发、参数提取、友好 URL）
- 路由表设计（路由规则、路由存储、路由查找、性能优化）
- 路由匹配算法（精确匹配、模式匹配、正则匹配、参数提取）
- 路由优先级（路由顺序、路由冲突、优先级规则）
- 实际应用示例和最佳实践

## 特性

- **灵活映射**：灵活的 URL 到处理程序的映射
- **参数提取**：自动提取路由参数
- **性能优化**：高效的路由匹配算法
- **易于使用**：简洁的 API
- **可扩展**：易于扩展新功能

## 路由概念

### 什么是路由

路由（Route）是 URL 到处理程序的映射规则，定义了如何将请求 URL 映射到相应的处理程序。

### 路由的作用

1. **URL 映射**：将 URL 映射到处理程序
2. **请求分发**：根据 URL 分发请求
3. **参数提取**：从 URL 中提取参数
4. **友好 URL**：提供友好的 URL 结构

### 路由的优势

1. **清晰结构**：清晰的 URL 结构
2. **易于维护**：易于维护和扩展
3. **SEO 友好**：SEO 友好的 URL
4. **RESTful**：支持 RESTful 设计
5. **灵活配置**：灵活的配置方式

## 路由的作用

### URL 映射

**示例**：
```php
<?php
declare(strict_types=1);

$routes = [
    'GET /users' => 'UserController@index',
    'GET /users/{id}' => 'UserController@show',
    'POST /users' => 'UserController@store',
    'PUT /users/{id}' => 'UserController@update',
    'DELETE /users/{id}' => 'UserController@destroy',
];
```

### 请求分发

**示例**：
```php
<?php
declare(strict_types=1);

function dispatch($request)
{
    $method = $request->getMethod();
    $path = $request->getUri()->getPath();
    
    $route = matchRoute($method, $path);
    
    if ($route === null) {
        return new Response('Not Found', 404);
    }
    
    return call_user_func($route['handler'], $request);
}
```

### 参数提取

**示例**：
```php
<?php
declare(strict_types=1);

// 路由定义
$route = 'GET /users/{id}';

// URL: /users/123
// 提取参数: ['id' => '123']
```

### 友好 URL

**示例**：
```php
<?php
declare(strict_types=1);

// 友好的 URL
GET /users/123
GET /posts/2024/01/hello-world

// 而不是
GET /index.php?action=show&type=user&id=123
GET /index.php?action=show&type=post&year=2024&month=01&slug=hello-world
```

## 路由表设计

### 路由规则

**路由规则格式**：
```php
<?php
declare(strict_types=1);

$routes = [
    // 方法 路径 处理器
    ['GET', '/users', 'UserController@index'],
    ['GET', '/users/{id}', 'UserController@show'],
    ['POST', '/users', 'UserController@store'],
];
```

### 路由存储

**示例**：
```php
<?php
declare(strict_types=1);

class RouteTable
{
    private array $routes = [];

    public function add(string $method, string $path, callable $handler): void
    {
        $this->routes[] = [
            'method' => $method,
            'path' => $path,
            'handler' => $handler,
            'pattern' => $this->compilePattern($path),
        ];
    }

    public function find(string $method, string $path): ?array
    {
        foreach ($this->routes as $route) {
            if ($route['method'] === $method && $this->match($route['pattern'], $path)) {
                return $route;
            }
        }
        return null;
    }

    private function compilePattern(string $path): string
    {
        return preg_replace('/\{(\w+)\}/', '([^/]+)', $path);
    }

    private function match(string $pattern, string $path): bool
    {
        return (bool) preg_match("#^{$pattern}$#", $path);
    }
}
```

### 路由查找

**示例**：
```php
<?php
declare(strict_types=1);

function findRoute(string $method, string $path): ?array
{
    global $routes;
    
    foreach ($routes as $route) {
        if ($route['method'] === $method && matchPath($route['path'], $path)) {
            return $route;
        }
    }
    
    return null;
}

function matchPath(string $pattern, string $path): bool
{
    $pattern = preg_replace('/\{(\w+)\}/', '([^/]+)', $pattern);
    return (bool) preg_match("#^{$pattern}$#", $path);
}
```

### 性能优化

**示例**：
```php
<?php
declare(strict_types=1);

class OptimizedRouteTable
{
    private array $routes = [];
    private array $cache = [];

    public function add(string $method, string $path, callable $handler): void
    {
        $key = "{$method}:{$path}";
        $this->routes[$key] = [
            'method' => $method,
            'path' => $path,
            'handler' => $handler,
        ];
    }

    public function find(string $method, string $path): ?array
    {
        $cacheKey = "{$method}:{$path}";
        
        // 使用缓存
        if (isset($this->cache[$cacheKey])) {
            return $this->cache[$cacheKey];
        }
        
        // 查找路由
        $route = $this->doFind($method, $path);
        
        // 缓存结果
        $this->cache[$cacheKey] = $route;
        
        return $route;
    }

    private function doFind(string $method, string $path): ?array
    {
        foreach ($this->routes as $route) {
            if ($route['method'] === $method && $this->match($route['path'], $path)) {
                return $route;
            }
        }
        return null;
    }
}
```

## 路由匹配算法

### 精确匹配

**示例**：
```php
<?php
declare(strict_types=1);

function exactMatch(string $pattern, string $path): bool
{
    return $pattern === $path;
}

// 使用
$routes = [
    'GET /users' => 'UserController@index',
    'GET /posts' => 'PostController@index',
];
```

### 模式匹配

**示例**：
```php
<?php
declare(strict_types=1);

function patternMatch(string $pattern, string $path): array
{
    // 将 {id} 转换为正则表达式
    $regex = preg_replace('/\{(\w+)\}/', '(?P<$1>[^/]+)', $pattern);
    $regex = "#^{$regex}$#";
    
    if (preg_match($regex, $path, $matches)) {
        // 提取命名参数
        $params = [];
        foreach ($matches as $key => $value) {
            if (!is_numeric($key)) {
                $params[$key] = $value;
            }
        }
        return ['matched' => true, 'params' => $params];
    }
    
    return ['matched' => false];
}

// 使用
$result = patternMatch('/users/{id}', '/users/123');
// ['matched' => true, 'params' => ['id' => '123']]
```

### 正则匹配

**示例**：
```php
<?php
declare(strict_types=1);

function regexMatch(string $pattern, string $path): array
{
    if (preg_match($pattern, $path, $matches)) {
        return ['matched' => true, 'matches' => $matches];
    }
    return ['matched' => false];
}

// 使用
$result = regexMatch('#^/users/(\d+)$#', '/users/123');
// ['matched' => true, 'matches' => ['123']]
```

### 参数提取

**示例**：
```php
<?php
declare(strict_types=1);

function extractParams(string $pattern, string $path): ?array
{
    // 提取参数名
    preg_match_all('/\{(\w+)\}/', $pattern, $paramNames);
    $paramNames = $paramNames[1];
    
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
$params = extractParams('/users/{id}/posts/{postId}', '/users/123/posts/456');
// ['id' => '123', 'postId' => '456']
```

## 路由优先级

### 路由顺序

**原则**：更具体的路由应该放在前面。

**示例**：
```php
<?php
declare(strict_types=1);

// ✅ 好的顺序：具体在前
$routes = [
    'GET /users/{id}/posts/{postId}',  // 更具体
    'GET /users/{id}',                  // 较具体
    'GET /users',                       // 一般
];

// ❌ 不好的顺序：一般在前
$routes = [
    'GET /users',                       // 一般
    'GET /users/{id}',                  // 较具体
    'GET /users/{id}/posts/{postId}',  // 更具体
];
```

### 路由冲突

**示例**：
```php
<?php
declare(strict_types=1);

// 路由冲突检测
function detectConflict(array $routes): array
{
    $conflicts = [];
    
    for ($i = 0; $i < count($routes); $i++) {
        for ($j = $i + 1; $j < count($routes); $j++) {
            if ($routes[$i]['method'] === $routes[$j]['method'] &&
                $this->conflict($routes[$i]['path'], $routes[$j]['path'])) {
                $conflicts[] = [$routes[$i], $routes[$j]];
            }
        }
    }
    
    return $conflicts;
}
```

### 优先级规则

**优先级规则**：
1. **精确匹配** > **模式匹配**
2. **更具体** > **更一般**
3. **静态路径** > **动态路径**
4. **先注册** > **后注册**（相同优先级时）

## 使用场景

### Web 应用

- 页面路由
- 功能路由
- 管理路由

### RESTful API

- 资源路由
- RESTful 设计
- API 版本控制

### MVC 框架

- 控制器路由
- 动作路由
- 参数路由

### 路由系统

- 自定义路由系统
- 路由框架
- 路由库

## 注意事项

### 路由顺序

- **重要**：路由顺序很重要
- **具体优先**：更具体的路由应该在前
- **冲突检测**：检测路由冲突

### 路由冲突

- **避免冲突**：避免路由冲突
- **冲突检测**：实现冲突检测
- **优先级规则**：定义优先级规则

### 性能考虑

- **匹配算法**：优化匹配算法
- **路由缓存**：使用路由缓存
- **查找优化**：优化路由查找

### 路由缓存

- **缓存路由表**：缓存编译后的路由表
- **缓存匹配结果**：缓存匹配结果
- **缓存失效**：处理缓存失效

## 常见问题

### 什么是路由？

路由是 URL 到处理程序的映射规则，定义了如何将请求 URL 映射到相应的处理程序。

### 如何设计路由表？

使用数组或类存储路由规则，支持添加、查找、匹配等功能。

### 如何匹配路由？

使用正则表达式或模式匹配算法，支持精确匹配和参数提取。

### 路由的性能如何优化？

使用路由缓存、优化匹配算法、优化查找顺序等方法优化性能。

## 最佳实践

### 设计清晰的路由规则

- 使用 RESTful 设计
- 清晰的 URL 结构
- 有意义的路径

### 使用路由缓存

- 缓存编译后的路由表
- 缓存匹配结果
- 处理缓存失效

### 优化匹配算法

- 使用高效的正则表达式
- 优化匹配顺序
- 减少不必要的匹配

### 提供路由文档

- 文档化路由规则
- 提供路由示例
- 说明路由参数

## 相关章节

- **[5.14.2 路由组织与模块化](section-02-organization.md)**：了解路由组织的详细内容
- **[5.14.3 动态路由与参数](section-03-dynamic.md)**：了解动态路由的详细内容
- **[5.14.4 路由守卫与权限控制](section-04-guards.md)**：了解路由守卫的详细内容

## 练习任务

1. **实现基本路由系统**
   - 路由表设计
   - 路由匹配算法
   - 参数提取

2. **实现路由缓存**
   - 路由表缓存
   - 匹配结果缓存
   - 缓存失效处理

3. **优化路由性能**
   - 优化匹配算法
   - 优化查找顺序
   - 性能测试

4. **实现路由冲突检测**
   - 冲突检测算法
   - 冲突报告
   - 冲突解决

5. **实现完整的路由系统**
   - 路由注册和管理
   - 路由匹配和分发
   - 路由文档和测试
