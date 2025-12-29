# 5.7.1 RESTful 设计原则

## 概述

RESTful API 是一种遵循 REST（Representational State Transfer，表述性状态转移）架构风格的 Web API 设计方法。理解 REST 的核心概念、掌握 RESTful API 的设计原则、正确使用 HTTP 方法和状态码，对于构建现代化、可维护、易用的 API 接口至关重要。本节详细介绍 REST 的核心概念、资源设计原则、HTTP 方法使用、HTTP 状态码使用、无状态设计、统一接口等内容，帮助零基础学员理解 RESTful API 设计。

REST 是一种架构风格，不是标准或协议。它定义了一组约束条件，满足这些约束条件的 Web 服务称为 RESTful Web 服务。RESTful API 使用标准的 HTTP 方法、状态码和资源标识符，使 API 设计更加统一和可预测。

**主要内容**：
- REST 概念（什么是 REST、REST 的约束条件、RESTful 的优势）
- 资源设计原则（资源的概念、资源命名规范、资源层级设计、资源标识 URI）
- HTTP 方法使用（GET、POST、PUT、DELETE、PATCH 等）
- HTTP 状态码使用（2xx 成功、4xx 客户端错误、5xx 服务器错误）
- 无状态设计（无状态的含义、如何实现无状态）
- 统一接口（统一接口的原则、如何实现）
- 实际应用示例和最佳实践

## 特性

- **资源导向**：以资源为中心设计 API
- **标准 HTTP**：使用标准 HTTP 方法和状态码
- **无状态**：每个请求包含所有必要信息
- **统一接口**：使用统一的接口设计
- **可缓存**：支持 HTTP 缓存机制

## REST 概念

### 什么是 REST

REST（Representational State Transfer）是一种架构风格，由 Roy Fielding 在 2000 年提出。REST 定义了一组约束条件，满足这些约束条件的 Web 服务称为 RESTful Web 服务。

### REST 的约束条件

1. **客户端-服务器架构**：客户端和服务器分离
2. **无状态**：每个请求包含所有必要信息
3. **可缓存**：响应可以被缓存
4. **统一接口**：使用统一的接口设计
5. **分层系统**：支持分层架构
6. **按需代码**（可选）：服务器可以发送可执行代码

### RESTful 的优势

1. **简单性**：使用标准 HTTP 方法，易于理解
2. **可扩展性**：支持水平扩展
3. **可缓存**：利用 HTTP 缓存机制
4. **统一性**：统一的接口设计
5. **可维护性**：清晰的资源结构

## 资源设计原则

### 资源的概念

资源（Resource）是 RESTful API 中的核心概念，是任何可以被命名、可以被标识、可以被操作的事物。

**资源的特征**：
- 有唯一标识符（URI）
- 有状态（可以被表示）
- 可以被操作（通过 HTTP 方法）

### 资源命名规范

**原则**：
- 使用名词，不使用动词
- 使用复数形式
- 使用小写字母
- 使用连字符分隔单词

**示例**：

| ✅ 正确 | ❌ 错误 |
|:--------|:--------|
| `/users` | `/getUsers` |
| `/users/123` | `/user/123` |
| `/user-posts` | `/userPosts` |
| `/orders/456/items` | `/order/456/getItems` |

### 资源层级设计

**扁平结构**：
```
/users
/posts
/comments
```

**层级结构**：
```
/users/{id}/posts
/users/{id}/posts/{postId}/comments
/orders/{id}/items
```

**选择原则**：
- 如果资源是独立的，使用扁平结构
- 如果资源有明确的父子关系，使用层级结构
- 避免过深的嵌套（通常不超过 3 层）

### 资源标识（URI）

**URI 设计原则**：
- 使用有意义的路径
- 保持简洁
- 使用标准格式
- 避免暴露实现细节

**示例**：

| ✅ 正确 | ❌ 错误 |
|:--------|:--------|
| `/api/v1/users/123` | `/api/v1/getUser.php?id=123` |
| `/api/v1/users/123/posts` | `/api/v1/user_posts?userId=123` |
| `/api/v1/orders/456` | `/api/v1/order?id=456` |

## HTTP 方法使用

### GET - 获取资源

**用途**：获取资源或资源列表

**特点**：
- 幂等性：多次请求结果相同
- 安全性：不修改服务器状态
- 可缓存：响应可以被缓存

**示例**：
```php
<?php
declare(strict_types=1);

// GET /api/users - 获取用户列表
if ($_SERVER['REQUEST_METHOD'] === 'GET' && $_SERVER['REQUEST_URI'] === '/api/users') {
    $users = [
        ['id' => 1, 'name' => 'Alice'],
        ['id' => 2, 'name' => 'Bob'],
    ];
    
    http_response_code(200);
    header('Content-Type: application/json');
    echo json_encode(['data' => $users]);
    exit;
}

// GET /api/users/123 - 获取单个用户
if ($_SERVER['REQUEST_METHOD'] === 'GET' && preg_match('#^/api/users/(\d+)$#', $_SERVER['REQUEST_URI'], $matches)) {
    $userId = (int) $matches[1];
    $user = ['id' => $userId, 'name' => 'Alice'];
    
    http_response_code(200);
    header('Content-Type: application/json');
    echo json_encode(['data' => $user]);
    exit;
}
```

### POST - 创建资源

**用途**：创建新资源

**特点**：
- 非幂等性：每次请求可能产生不同结果
- 非安全性：会修改服务器状态
- 不可缓存：响应通常不被缓存

**示例**：
```php
<?php
declare(strict_types=1);

// POST /api/users - 创建用户
if ($_SERVER['REQUEST_METHOD'] === 'POST' && $_SERVER['REQUEST_URI'] === '/api/users') {
    $json = file_get_contents('php://input');
    $data = json_decode($json, true);
    
    // 验证数据
    // 创建用户
    $newUser = [
        'id' => 123,
        'name' => $data['name'],
        'email' => $data['email'],
    ];
    
    http_response_code(201);
    header('Content-Type: application/json');
    header('Location: /api/users/123');
    echo json_encode(['data' => $newUser]);
    exit;
}
```

### PUT - 更新资源

**用途**：完整更新资源（替换整个资源）

**特点**：
- 幂等性：多次请求结果相同
- 非安全性：会修改服务器状态
- 需要提供完整资源数据

**示例**：
```php
<?php
declare(strict_types=1);

// PUT /api/users/123 - 更新用户
if ($_SERVER['REQUEST_METHOD'] === 'PUT' && preg_match('#^/api/users/(\d+)$#', $_SERVER['REQUEST_URI'], $matches)) {
    $userId = (int) $matches[1];
    $json = file_get_contents('php://input');
    $data = json_decode($json, true);
    
    // 更新用户（完整替换）
    $updatedUser = [
        'id' => $userId,
        'name' => $data['name'],
        'email' => $data['email'],
    ];
    
    http_response_code(200);
    header('Content-Type: application/json');
    echo json_encode(['data' => $updatedUser]);
    exit;
}
```

### DELETE - 删除资源

**用途**：删除资源

**特点**：
- 幂等性：多次请求结果相同
- 非安全性：会修改服务器状态
- 删除后资源不存在

**示例**：
```php
<?php
declare(strict_types=1);

// DELETE /api/users/123 - 删除用户
if ($_SERVER['REQUEST_METHOD'] === 'DELETE' && preg_match('#^/api/users/(\d+)$#', $_SERVER['REQUEST_URI'], $matches)) {
    $userId = (int) $matches[1];
    
    // 删除用户
    // ...
    
    http_response_code(204);  // No Content
    exit;
}
```

### PATCH - 部分更新资源

**用途**：部分更新资源（只更新提供的字段）

**特点**：
- 非幂等性：取决于实现
- 非安全性：会修改服务器状态
- 只需要提供要更新的字段

**示例**：
```php
<?php
declare(strict_types=1);

// PATCH /api/users/123 - 部分更新用户
if ($_SERVER['REQUEST_METHOD'] === 'PATCH' && preg_match('#^/api/users/(\d+)$#', $_SERVER['REQUEST_URI'], $matches)) {
    $userId = (int) $matches[1];
    $json = file_get_contents('php://input');
    $data = json_decode($json, true);
    
    // 部分更新用户（只更新提供的字段）
    $updatedUser = [
        'id' => $userId,
        'name' => $data['name'] ?? 'Existing Name',
        'email' => $data['email'] ?? 'existing@example.com',
    ];
    
    http_response_code(200);
    header('Content-Type: application/json');
    echo json_encode(['data' => $updatedUser]);
    exit;
}
```

### HTTP 方法总结

| 方法 | 用途 | 幂等性 | 安全性 | 请求体 | 响应体 |
|:-----|:-----|:-------|:-------|:-------|:-------|
| GET | 获取资源 | ✅ | ✅ | ❌ | ✅ |
| POST | 创建资源 | ❌ | ❌ | ✅ | ✅ |
| PUT | 完整更新 | ✅ | ❌ | ✅ | ✅ |
| PATCH | 部分更新 | ⚠️ | ❌ | ✅ | ✅ |
| DELETE | 删除资源 | ✅ | ❌ | ❌ | ❌ |

## HTTP 状态码使用

### 2xx - 成功

| 状态码 | 说明 | 使用场景 |
|:-------|:-----|:---------|
| `200 OK` | 请求成功 | GET、PUT、PATCH 成功 |
| `201 Created` | 资源创建成功 | POST 创建资源成功 |
| `204 No Content` | 请求成功，无返回内容 | DELETE 成功 |

**示例**：
```php
<?php
declare(strict_types=1);

// 200 OK - 获取资源成功
http_response_code(200);
echo json_encode(['data' => $user]);

// 201 Created - 创建资源成功
http_response_code(201);
header('Location: /api/users/123');
echo json_encode(['data' => $newUser]);

// 204 No Content - 删除成功
http_response_code(204);
exit;
```

### 4xx - 客户端错误

| 状态码 | 说明 | 使用场景 |
|:-------|:-----|:---------|
| `400 Bad Request` | 请求参数错误 | 请求格式错误 |
| `401 Unauthorized` | 未认证 | 需要登录 |
| `403 Forbidden` | 无权限 | 已认证但无权限 |
| `404 Not Found` | 资源不存在 | 资源未找到 |
| `422 Unprocessable Entity` | 验证失败 | 数据验证失败 |

**示例**：
```php
<?php
declare(strict_types=1);

// 400 Bad Request
http_response_code(400);
echo json_encode(['error' => 'Invalid request']);

// 401 Unauthorized
http_response_code(401);
echo json_encode(['error' => 'Authentication required']);

// 403 Forbidden
http_response_code(403);
echo json_encode(['error' => 'Access denied']);

// 404 Not Found
http_response_code(404);
echo json_encode(['error' => 'Resource not found']);

// 422 Unprocessable Entity
http_response_code(422);
echo json_encode(['errors' => ['email' => 'Invalid email format']]);
```

### 5xx - 服务器错误

| 状态码 | 说明 | 使用场景 |
|:-------|:-----|:---------|
| `500 Internal Server Error` | 服务器内部错误 | 服务器错误 |
| `503 Service Unavailable` | 服务不可用 | 服务维护中 |

**示例**：
```php
<?php
declare(strict_types=1);

// 500 Internal Server Error
http_response_code(500);
echo json_encode(['error' => 'Internal server error']);

// 503 Service Unavailable
http_response_code(503);
echo json_encode(['error' => 'Service temporarily unavailable']);
```

## 无状态设计

### 无状态的含义

无状态（Stateless）是指每个请求都包含所有必要的信息，服务器不保存客户端的状态。

### 如何实现无状态

1. **不在服务器存储会话状态**：使用 Token 而不是 Session
2. **请求包含所有必要信息**：认证信息、参数等都在请求中
3. **使用 Token 认证**：JWT、OAuth Token 等

**示例**：
```php
<?php
declare(strict_types=1);

// 有状态（不推荐）
session_start();
if (!isset($_SESSION['user_id'])) {
    http_response_code(401);
    exit;
}
$userId = $_SESSION['user_id'];

// 无状态（推荐）
$token = $_SERVER['HTTP_AUTHORIZATION'] ?? '';
if (empty($token)) {
    http_response_code(401);
    exit;
}
$userId = validateToken($token);
```

## 统一接口

### 统一接口的原则

1. **统一的资源标识**：使用 URI 标识资源
2. **统一的操作方法**：使用标准 HTTP 方法
3. **统一的响应格式**：使用 JSON 格式
4. **统一的状态码**：使用标准 HTTP 状态码

### 如何实现统一接口

**示例**：
```php
<?php
declare(strict_types=1);

class ApiResponse
{
    public static function success(mixed $data = null, int $statusCode = 200): void
    {
        http_response_code($statusCode);
        header('Content-Type: application/json');
        
        $response = ['success' => true];
        if ($data !== null) {
            $response['data'] = $data;
        }
        
        echo json_encode($response);
        exit;
    }

    public static function error(string $message, int $statusCode = 400, array $errors = []): void
    {
        http_response_code($statusCode);
        header('Content-Type: application/json');
        
        $response = [
            'success' => false,
            'message' => $message,
        ];
        
        if (!empty($errors)) {
            $response['errors'] = $errors;
        }
        
        echo json_encode($response);
        exit;
    }
}

// 使用
ApiResponse::success(['users' => $users]);
ApiResponse::error('Validation failed', 422, ['email' => 'Invalid format']);
```

## 使用场景

### Web API 设计

- 前后端分离架构
- 移动应用后端
- 第三方 API 集成

### 微服务接口

- 服务间通信
- API 网关
- 服务发现

### 前后端分离

- SPA 应用后端
- 移动应用后端
- 桌面应用后端

## 注意事项

### 遵循 REST 原则

- 使用名词表示资源
- 正确使用 HTTP 方法
- 使用标准状态码
- 保持无状态

### 正确使用 HTTP 方法

- GET：获取资源
- POST：创建资源
- PUT：完整更新
- PATCH：部分更新
- DELETE：删除资源

### 合理使用状态码

- 2xx：成功
- 4xx：客户端错误
- 5xx：服务器错误
- 使用标准状态码

### 保持无状态

- 不使用 Session
- 使用 Token 认证
- 请求包含所有必要信息

## 常见问题

### 什么是 RESTful API？

RESTful API 是遵循 REST 架构风格的 Web API，使用标准 HTTP 方法、状态码和资源标识符。

### 如何设计资源 URI？

- 使用名词表示资源
- 使用复数形式
- 使用层级结构表示关系
- 保持简洁和有意义

### 如何选择 HTTP 方法？

- GET：获取资源
- POST：创建资源
- PUT：完整更新资源
- PATCH：部分更新资源
- DELETE：删除资源

### 如何选择状态码？

- 2xx：请求成功
- 4xx：客户端错误
- 5xx：服务器错误
- 使用标准状态码

## 最佳实践

### 使用名词表示资源

- `/users` 而不是 `/getUsers`
- `/orders` 而不是 `/createOrder`
- 保持资源名称简洁

### 正确使用 HTTP 方法

- 根据操作选择正确的方法
- 遵循方法的语义
- 保持方法的一致性

### 使用标准状态码

- 使用标准 HTTP 状态码
- 提供清晰的错误信息
- 保持状态码的一致性

### 保持 API 一致性

- 统一的响应格式
- 统一的错误处理
- 统一的认证方式

## 相关章节

- **[5.7.2 API 路由与版本控制](section-02-routing-versioning.md)**：了解 API 路由和版本控制
- **[5.5 请求体解析](../chapter-05-json-requests/readme.md)**：了解 API 请求的处理
- **[5.8 响应处理与跨域](../chapter-08-response-cors/readme.md)**：了解 API 响应的处理

## 练习任务

1. **设计 RESTful API**
   - 设计资源 URI
   - 选择 HTTP 方法
   - 定义响应格式

2. **实现 RESTful API 端点**
   - 实现 GET 端点
   - 实现 POST 端点
   - 实现 PUT、PATCH、DELETE 端点

3. **实现统一响应格式**
   - 创建响应类
   - 统一成功响应
   - 统一错误响应

4. **实现无状态认证**
   - 使用 Token 认证
   - 实现 Token 验证
   - 处理认证错误

5. **实现完整的 RESTful API**
   - 设计资源结构
   - 实现所有 CRUD 操作
   - 使用标准状态码
