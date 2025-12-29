# 5.7.2 API 路由与版本控制

## 概述

API 路由和版本控制是 API 管理的重要方面。良好的路由设计使 API 结构清晰、易于理解，版本控制则确保 API 的演进不会破坏现有客户端。理解 API 路由的设计方法、掌握版本控制策略、实现路由和版本管理，对于构建可维护、可扩展的 API 至关重要。本节详细介绍 API 路由的设计方法、路由组织方法、版本控制策略、URL 版本控制、Header 版本控制、版本兼容性等内容，帮助零基础学员掌握 API 管理技术。

API 路由决定了如何访问 API 资源，版本控制则管理 API 的演进。随着业务需求的变化，API 需要不断更新和优化，版本控制确保新旧版本可以共存，给客户端足够的迁移时间。

**主要内容**：
- API 路由设计（路由结构、资源路由、嵌套路由、路由组织）
- 路由组织方法（扁平路由、层级路由、模块化路由）
- 版本控制策略（为什么需要版本控制、版本控制方法、版本号格式、版本选择）
- URL 版本控制（路径版本 `/api/v1/users`、实现方法、路由配置、优势劣势）
- Header 版本控制（Accept 头版本、自定义 Header、实现方法、优势劣势）
- 版本兼容性（向后兼容、版本迁移、废弃策略）
- 实际应用示例和最佳实践

## 特性

- **清晰结构**：路由结构清晰，易于理解
- **版本管理**：支持多版本共存
- **向后兼容**：保持向后兼容性
- **灵活选择**：支持多种版本控制方法
- **易于维护**：路由和版本管理简单

## API 路由设计

### 路由结构

**基本结构**：
```
/api/{version}/{resource}
/api/{version}/{resource}/{id}
/api/{version}/{resource}/{id}/{sub-resource}
```

**示例**：
```
/api/v1/users
/api/v1/users/123
/api/v1/users/123/posts
/api/v1/users/123/posts/456
```

### 资源路由

**RESTful 资源路由**：
```php
<?php
declare(strict_types=1);

$routes = [
    'GET /api/v1/users' => 'UserController@index',
    'POST /api/v1/users' => 'UserController@store',
    'GET /api/v1/users/{id}' => 'UserController@show',
    'PUT /api/v1/users/{id}' => 'UserController@update',
    'PATCH /api/v1/users/{id}' => 'UserController@update',
    'DELETE /api/v1/users/{id}' => 'UserController@destroy',
];
```

### 嵌套路由

**嵌套资源路由**：
```php
<?php
declare(strict_types=1);

$routes = [
    'GET /api/v1/users/{userId}/posts' => 'PostController@index',
    'POST /api/v1/users/{userId}/posts' => 'PostController@store',
    'GET /api/v1/users/{userId}/posts/{postId}' => 'PostController@show',
    'PUT /api/v1/users/{userId}/posts/{postId}' => 'PostController@update',
    'DELETE /api/v1/users/{userId}/posts/{postId}' => 'PostController@destroy',
];
```

### 路由组织

**按模块组织**：
```php
<?php
declare(strict_types=1);

$routes = [
    // 用户模块
    'users' => [
        'GET /api/v1/users' => 'UserController@index',
        'POST /api/v1/users' => 'UserController@store',
        'GET /api/v1/users/{id}' => 'UserController@show',
    ],
    
    // 文章模块
    'posts' => [
        'GET /api/v1/posts' => 'PostController@index',
        'POST /api/v1/posts' => 'PostController@store',
        'GET /api/v1/posts/{id}' => 'PostController@show',
    ],
];
```

## 路由组织方法

### 扁平路由

**特点**：所有路由在同一层级

**示例**：
```php
<?php
declare(strict_types=1);

$routes = [
    'GET /api/v1/users' => 'UserController@index',
    'POST /api/v1/users' => 'UserController@store',
    'GET /api/v1/posts' => 'PostController@index',
    'POST /api/v1/posts' => 'PostController@store',
];
```

### 层级路由

**特点**：路由按资源层级组织

**示例**：
```php
<?php
declare(strict_types=1);

$routes = [
    'users' => [
        'index' => 'GET /api/v1/users',
        'store' => 'POST /api/v1/users',
        'show' => 'GET /api/v1/users/{id}',
    ],
    'posts' => [
        'index' => 'GET /api/v1/posts',
        'store' => 'POST /api/v1/posts',
        'show' => 'GET /api/v1/posts/{id}',
    ],
];
```

### 模块化路由

**特点**：按功能模块组织路由

**示例**：
```php
<?php
declare(strict_types=1);

class RouteManager
{
    private array $routes = [];

    public function registerModule(string $module, array $routes): void
    {
        $this->routes[$module] = $routes;
    }

    public function getRoutes(): array
    {
        return $this->routes;
    }
}

$router = new RouteManager();
$router->registerModule('users', [
    'GET /api/v1/users' => 'UserController@index',
    'POST /api/v1/users' => 'UserController@store',
]);
$router->registerModule('posts', [
    'GET /api/v1/posts' => 'PostController@index',
    'POST /api/v1/posts' => 'PostController@store',
]);
```

## 版本控制策略

### 为什么需要版本控制

1. **API 演进**：API 需要不断更新和优化
2. **向后兼容**：不能破坏现有客户端
3. **平滑迁移**：给客户端足够的迁移时间
4. **多版本共存**：新旧版本可以同时存在

### 版本控制方法

**主要方法**：
1. **URL 版本控制**：版本在 URL 路径中（`/api/v1/users`）
2. **Header 版本控制**：版本在 HTTP Header 中（`Accept: application/vnd.api+json;version=1`）
3. **查询参数版本**：版本在查询参数中（`/api/users?version=1`）

### 版本号格式

**常见格式**：
- **语义化版本**：`v1.0.0`、`v1.1.0`、`v2.0.0`
- **简单版本**：`v1`、`v2`、`v3`
- **日期版本**：`v2024-01`、`v2024-02`

**推荐**：使用简单版本号（`v1`、`v2`），易于理解和维护。

### 版本选择

**默认版本**：
- 如果没有指定版本，使用最新稳定版本
- 或者返回错误，要求指定版本

**版本选择逻辑**：
```php
<?php
declare(strict_types=1);

function getApiVersion(): string
{
    // 1. 从 URL 路径获取
    if (preg_match('#^/api/v(\d+)#', $_SERVER['REQUEST_URI'], $matches)) {
        return 'v' . $matches[1];
    }
    
    // 2. 从 Header 获取
    $accept = $_SERVER['HTTP_ACCEPT'] ?? '';
    if (preg_match('/version=(\d+)/', $accept, $matches)) {
        return 'v' . $matches[1];
    }
    
    // 3. 从查询参数获取
    $version = $_GET['version'] ?? null;
    if ($version !== null) {
        return 'v' . $version;
    }
    
    // 4. 返回默认版本
    return 'v1';
}
```

## URL 版本控制

### 路径版本

**格式**：`/api/{version}/{resource}`

**示例**：
```
/api/v1/users
/api/v2/users
/api/v1/users/123
/api/v2/users/123
```

### 实现方法

**示例**：
```php
<?php
declare(strict_types=1);

class ApiRouter
{
    private array $versions = [];

    public function registerVersion(string $version, array $routes): void
    {
        $this->versions[$version] = $routes;
    }

    public function handle(): void
    {
        $uri = $_SERVER['REQUEST_URI'];
        $method = $_SERVER['REQUEST_METHOD'];
        
        // 提取版本
        if (preg_match('#^/api/v(\d+)/(.+)$#', $uri, $matches)) {
            $version = 'v' . $matches[1];
            $path = '/' . $matches[2];
            
            // 查找路由
            $route = $method . ' ' . $path;
            if (isset($this->versions[$version][$route])) {
                $handler = $this->versions[$version][$route];
                $handler();
                return;
            }
        }
        
        http_response_code(404);
        echo json_encode(['error' => 'Route not found']);
    }
}

// 使用
$router = new ApiRouter();
$router->registerVersion('v1', [
    'GET /users' => function () {
        echo json_encode(['users' => []]);
    },
]);
$router->registerVersion('v2', [
    'GET /users' => function () {
        echo json_encode(['data' => ['users' => []]]);
    },
]);

$router->handle();
```

### 路由配置

**示例**：
```php
<?php
declare(strict_types=1);

$apiRoutes = [
    'v1' => [
        'GET /users' => 'UserController@index',
        'POST /users' => 'UserController@store',
        'GET /users/{id}' => 'UserController@show',
    ],
    'v2' => [
        'GET /users' => 'UserController@indexV2',
        'POST /users' => 'UserController@storeV2',
        'GET /users/{id}' => 'UserController@showV2',
    ],
];
```

### 优势劣势

**优势**：
- 简单直观
- 易于理解
- 易于实现
- 版本清晰可见

**劣势**：
- URL 变长
- 需要更新所有 URL
- 不够优雅

## Header 版本控制

### Accept 头版本

**格式**：`Accept: application/vnd.api+json;version=1`

**示例**：
```http
GET /api/users HTTP/1.1
Host: api.example.com
Accept: application/vnd.api+json;version=1
```

### 自定义 Header

**格式**：`X-API-Version: 1`

**示例**：
```http
GET /api/users HTTP/1.1
Host: api.example.com
X-API-Version: 1
```

### 实现方法

**示例**：
```php
<?php
declare(strict_types=1);

function getVersionFromHeader(): ?string
{
    // 方法 1：从 Accept 头获取
    $accept = $_SERVER['HTTP_ACCEPT'] ?? '';
    if (preg_match('/version=(\d+)/', $accept, $matches)) {
        return 'v' . $matches[1];
    }
    
    // 方法 2：从自定义 Header 获取
    $apiVersion = $_SERVER['HTTP_X_API_VERSION'] ?? null;
    if ($apiVersion !== null) {
        return 'v' . $apiVersion;
    }
    
    return null;
}

// 使用
$version = getVersionFromHeader();
if ($version === null) {
    http_response_code(400);
    echo json_encode(['error' => 'API version required']);
    exit;
}
```

### 优势劣势

**优势**：
- URL 保持不变
- 更加优雅
- 不影响 URL 结构

**劣势**：
- 不够直观
- 需要客户端支持
- 实现相对复杂

## 版本兼容性

### 向后兼容

**原则**：
- 添加新字段：兼容
- 删除字段：不兼容
- 修改字段类型：不兼容
- 修改字段含义：不兼容

**示例**：
```php
<?php
declare(strict_types=1);

// v1 响应格式
function getV1Response(array $user): array
{
    return [
        'id' => $user['id'],
        'name' => $user['name'],
        'email' => $user['email'],
    ];
}

// v2 响应格式（向后兼容）
function getV2Response(array $user): array
{
    return [
        'id' => $user['id'],
        'name' => $user['name'],
        'email' => $user['email'],
        'avatar' => $user['avatar'] ?? null,  // 新增字段，兼容
    ];
}
```

### 版本迁移

**策略**：
1. **通知客户端**：提前通知版本变更
2. **提供迁移指南**：提供详细的迁移文档
3. **保持旧版本**：保持旧版本运行一段时间
4. **逐步废弃**：逐步废弃旧版本

**示例**：
```php
<?php
declare(strict_types=1);

// 检查版本是否已废弃
function checkVersionDeprecated(string $version): void
{
    $deprecatedVersions = ['v1'];
    
    if (in_array($version, $deprecatedVersions, true)) {
        header('X-API-Deprecated: true');
        header('X-API-Sunset: 2024-12-31');
        header('Link: </api/v2>; rel="successor-version"');
    }
}
```

### 废弃策略

**步骤**：
1. **标记废弃**：在响应头中标记版本已废弃
2. **设置废弃日期**：设置版本废弃的具体日期
3. **提供替代版本**：提供新版本的链接
4. **逐步移除**：在废弃日期后移除旧版本

## 使用场景

### API 演进管理

- API 功能更新
- API 结构优化
- API 性能改进

### 向后兼容

- 保持现有客户端可用
- 平滑迁移到新版本
- 多版本共存

### 多版本共存

- 新旧版本同时运行
- 客户端逐步迁移
- 版本平滑过渡

## 注意事项

### 版本选择策略

- **明确版本**：要求客户端明确指定版本
- **默认版本**：提供合理的默认版本
- **版本验证**：验证版本是否有效

### 向后兼容性

- **添加字段**：可以添加新字段
- **删除字段**：需要创建新版本
- **修改字段**：需要创建新版本

### 版本文档

- **版本变更说明**：详细说明版本变更
- **迁移指南**：提供迁移指南
- **废弃通知**：提前通知版本废弃

### 废弃策略

- **提前通知**：提前通知版本废弃
- **设置日期**：设置明确的废弃日期
- **提供替代**：提供新版本的链接

## 常见问题

### 如何实现 API 版本控制？

主要方法：
1. **URL 版本控制**：版本在 URL 路径中
2. **Header 版本控制**：版本在 HTTP Header 中
3. **查询参数版本**：版本在查询参数中

### URL 版本和 Header 版本的区别？

| 特性 | URL 版本 | Header 版本 |
|:-----|:---------|:------------|
| 可见性 | 版本在 URL 中可见 | 版本在 Header 中 |
| URL 变化 | URL 包含版本号 | URL 保持不变 |
| 实现难度 | 简单 | 相对复杂 |
| 推荐度 | 推荐 | 可选 |

### 如何管理多个版本？

1. **版本隔离**：每个版本独立的代码和路由
2. **版本共享**：共享通用逻辑
3. **版本文档**：为每个版本维护文档

### 何时废弃旧版本？

1. **提前通知**：提前 6-12 个月通知
2. **设置日期**：设置明确的废弃日期
3. **提供替代**：提供新版本的完整文档
4. **逐步移除**：在废弃日期后移除

## 最佳实践

### 从第一个版本开始版本控制

- 即使只有一个版本，也要使用版本控制
- 为未来版本演进做好准备
- 建立版本管理规范

### 使用 URL 版本控制（简单直观）

- 推荐使用 URL 版本控制
- 版本清晰可见
- 易于实现和维护

### 提供版本文档

- 为每个版本维护文档
- 说明版本变更
- 提供迁移指南

### 制定版本废弃策略

- 提前通知版本废弃
- 设置明确的废弃日期
- 提供新版本的完整文档

## 相关章节

- **[5.7.1 RESTful 设计原则](section-01-restful-principles.md)**：了解 RESTful API 的设计原则
- **[5.14 路由设计](../chapter-14-routing/readme.md)**：了解路由设计的详细内容
- **[5.4.4 URL 路由与重写](../chapter-04-url-handling/section-04-url-routing.md)**：了解 URL 路由的基础内容

## 练习任务

1. **实现 API 路由系统**
   - 设计路由结构
   - 实现路由匹配
   - 支持参数提取

2. **实现版本控制**
   - URL 版本控制
   - Header 版本控制
   - 版本选择逻辑

3. **实现多版本管理**
   - 版本注册
   - 版本路由
   - 版本切换

4. **实现版本兼容性**
   - 向后兼容检查
   - 版本迁移支持
   - 废弃版本处理

5. **实现完整的 API 版本管理系统**
   - 版本注册和管理
   - 路由和版本绑定
   - 版本文档和迁移指南
