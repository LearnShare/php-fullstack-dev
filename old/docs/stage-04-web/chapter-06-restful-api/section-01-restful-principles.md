# 4.6.1 RESTful 设计原则

## 概述

REST（Representational State Transfer）是一种架构风格，基于 HTTP 协议设计 Web API。本节详细介绍 REST 核心原则、HTTP 方法语义、资源路径设计、状态码使用，以及幂等性与安全性。

## REST 核心原则

### 什么是 REST

- **REST**：Representational State Transfer（表述性状态转移）
- 基于 HTTP 协议，使用标准 HTTP 方法操作资源
- 资源通过 URI 标识，状态通过 HTTP 方法改变

### REST 原则

1. **资源（Resource）**：一切皆资源，通过 URI 标识
2. **表述（Representation）**：资源有多种表述形式（JSON、XML 等）
3. **状态转移（State Transfer）**：通过 HTTP 方法改变资源状态
4. **无状态（Stateless）**：每个请求包含所有必要信息
5. **统一接口（Uniform Interface）**：使用标准 HTTP 方法

## HTTP 方法语义

### 标准 HTTP 方法

| 方法 | 语义 | 幂等性 | 安全性 | 示例 |
| :--- | :--- | :--- | :--- | :--- |
| `GET` | 获取资源 | 是 | 是 | `GET /users/1` |
| `POST` | 创建资源 | 否 | 否 | `POST /users` |
| `PUT` | 完整更新资源 | 是 | 否 | `PUT /users/1` |
| `PATCH` | 部分更新资源 | 否 | 否 | `PATCH /users/1` |
| `DELETE` | 删除资源 | 是 | 否 | `DELETE /users/1` |
| `HEAD` | 获取响应头 | 是 | 是 | `HEAD /users/1` |
| `OPTIONS` | 获取支持的方法 | 是 | 是 | `OPTIONS /users` |

### 方法使用示例

```php
<?php
declare(strict_types=1);

$method = $_SERVER['REQUEST_METHOD'];
$path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

match ($method) {
    'GET' => handleGet($path),
    'POST' => handlePost($path),
    'PUT' => handlePut($path),
    'PATCH' => handlePatch($path),
    'DELETE' => handleDelete($path),
    default => http_response_code(405),
};

function handleGet(string $path): void
{
    // GET /users - 获取用户列表
    if (preg_match('#^/users$#', $path)) {
        $users = getAllUsers();
        sendJsonResponse($users);
        return;
    }
    
    // GET /users/1 - 获取单个用户
    if (preg_match('#^/users/(\d+)$#', $path, $matches)) {
        $userId = (int) $matches[1];
        $user = getUserById($userId);
        
        if ($user === null) {
            sendErrorResponse('User not found', 404);
            return;
        }
        
        sendJsonResponse($user);
        return;
    }
    
    sendErrorResponse('Not found', 404);
}

function handlePost(string $path): void
{
    // POST /users - 创建用户
    if (preg_match('#^/users$#', $path)) {
        $data = getJsonInput();
        
        if (empty($data['name']) || empty($data['email'])) {
            sendErrorResponse('Name and email are required', 422);
            return;
        }
        
        $user = createUser($data['name'], $data['email']);
        http_response_code(201);
        sendJsonResponse($user);
        return;
    }
    
    sendErrorResponse('Not found', 404);
}
```

## 资源路径设计

### 设计原则

- **使用名词**：`/users` 而不是 `/getUsers`
- **使用复数**：`/users` 而不是 `/user`
- **层级关系**：`/users/1/posts` 表示用户 1 的文章
- **避免动词**：使用 HTTP 方法表达动作

### 路径示例

```
GET    /users              # 获取用户列表
GET    /users/1            # 获取用户详情
POST   /users              # 创建用户
PUT    /users/1            # 完整更新用户
PATCH  /users/1            # 部分更新用户
DELETE /users/1            # 删除用户

GET    /users/1/posts      # 获取用户的文章列表
POST   /users/1/posts      # 为用户创建文章
GET    /users/1/posts/5    # 获取用户的某篇文章
```

## 状态码使用

### 常用状态码

| 状态码 | 说明 | 使用场景 |
| :--- | :--- | :--- |
| 200 | OK | 成功获取资源 |
| 201 | Created | 资源创建成功 |
| 204 | No Content | 删除成功，无返回内容 |
| 400 | Bad Request | 请求格式错误 |
| 401 | Unauthorized | 未认证 |
| 403 | Forbidden | 无权限 |
| 404 | Not Found | 资源不存在 |
| 422 | Unprocessable Entity | 验证失败 |
| 500 | Internal Server Error | 服务器错误 |

### 状态码使用示例

```php
<?php
declare(strict_types=1);

// 成功获取
http_response_code(200);
echo json_encode($data);

// 创建成功
http_response_code(201);
header('Location: /users/' . $user->id);
echo json_encode($user);

// 删除成功
http_response_code(204);
exit;

// 验证失败
http_response_code(422);
echo json_encode(['errors' => $validationErrors]);

// 资源不存在
http_response_code(404);
echo json_encode(['error' => 'Resource not found']);
```

## 幂等性与安全性

### 幂等性

- **幂等操作**：多次执行结果相同
- **GET、PUT、DELETE**：幂等
- **POST、PATCH**：非幂等

### 安全性

- **安全操作**：不改变资源状态
- **GET、HEAD、OPTIONS**：安全
- **POST、PUT、PATCH、DELETE**：不安全

## 完整示例

```php
<?php
declare(strict_types=1);

class RestfulApi
{
    public function handleRequest(): void
    {
        $method = $_SERVER['REQUEST_METHOD'];
        $path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
        
        match ($method) {
            'GET' => $this->handleGet($path),
            'POST' => $this->handlePost($path),
            'PUT' => $this->handlePut($path),
            'PATCH' => $this->handlePatch($path),
            'DELETE' => $this->handleDelete($path),
            default => $this->sendError('Method not allowed', 405),
        };
    }

    private function handleGet(string $path): void
    {
        if (preg_match('#^/users/(\d+)$#', $path, $matches)) {
            $userId = (int) $matches[1];
            $user = $this->getUser($userId);
            
            if ($user === null) {
                $this->sendError('User not found', 404);
                return;
            }
            
            $this->sendResponse($user, 200);
        } else {
            $this->sendError('Not found', 404);
        }
    }

    private function sendResponse(mixed $data, int $statusCode = 200): void
    {
        http_response_code($statusCode);
        header('Content-Type: application/json');
        echo json_encode($data, JSON_UNESCAPED_UNICODE);
    }

    private function sendError(string $message, int $statusCode): void
    {
        http_response_code($statusCode);
        header('Content-Type: application/json');
        echo json_encode(['error' => $message], JSON_UNESCAPED_UNICODE);
    }
}
```

## 注意事项

1. **使用正确的 HTTP 方法**：根据操作选择合适的方法
2. **返回正确的状态码**：准确反映操作结果
3. **资源路径设计**：使用名词，表达资源关系
4. **幂等性考虑**：确保幂等操作可以安全重试

## 练习

1. 设计一个用户管理 API，包含完整的 CRUD 操作。

2. 实现一个 RESTful 路由系统，支持资源嵌套。

3. 创建一个 API 响应处理类，统一处理成功和错误响应。

4. 设计一个文章系统 API，包含文章和评论的资源关系。
