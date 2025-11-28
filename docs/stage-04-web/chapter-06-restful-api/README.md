# 4.6 RESTful API 设计与接口规范

## 目标

- 理解 REST（Representational State Transfer）架构风格的核心原则。
- 掌握 HTTP 方法的语义与使用场景。
- 能够设计符合 RESTful 规范的资源路径。
- 熟悉标准化的错误响应格式与 JSON API 规范。

## REST 核心原则

### 什么是 REST

- **REST**：Representational State Transfer（表述性状态转移）。
- 基于 HTTP 协议，使用标准 HTTP 方法操作资源。
- 资源通过 URI 标识，状态通过 HTTP 方法改变。

### REST 原则

1. **资源（Resource）**：一切皆资源，通过 URI 标识。
2. **表述（Representation）**：资源有多种表述形式（JSON、XML 等）。
3. **状态转移（State Transfer）**：通过 HTTP 方法改变资源状态。
4. **无状态（Stateless）**：每个请求包含所有必要信息。
5. **统一接口（Uniform Interface）**：使用标准 HTTP 方法。

## HTTP 方法语义

### 标准 HTTP 方法

| 方法      | 语义           | 幂等性 | 安全性 | 示例                    |
| :-------- | :------------- | :----- | :----- | :---------------------- |
| `GET`     | 获取资源       | 是     | 是     | `GET /users/1`          |
| `POST`    | 创建资源       | 否     | 否     | `POST /users`           |
| `PUT`     | 完整更新资源   | 是     | 否     | `PUT /users/1`          |
| `PATCH`   | 部分更新资源   | 否     | 否     | `PATCH /users/1`        |
| `DELETE`  | 删除资源       | 是     | 否     | `DELETE /users/1`       |
| `HEAD`    | 获取响应头     | 是     | 是     | `HEAD /users/1`         |
| `OPTIONS` | 获取支持的方法 | 是     | 是     | `OPTIONS /users`        |

### 方法使用示例

```php
<?php
declare(strict_types=1);

$method = $_SERVER['REQUEST_METHOD'];
$path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

// 路由处理
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

### 路径规范

- 使用名词，不使用动词。
- 使用复数形式表示资源集合。
- 使用层级结构表示资源关系。
- 避免深层嵌套（建议不超过 2 层）。

```php
// 好的设计
GET    /users              // 获取用户列表
GET    /users/1            // 获取用户详情
POST   /users              // 创建用户
PUT    /users/1            // 更新用户
DELETE /users/1            // 删除用户

GET    /users/1/posts      // 获取用户的文章
POST   /users/1/posts      // 为用户创建文章

// 不好的设计
GET    /getUsers           // 使用动词
GET    /user               // 单数形式
POST   /createUser         // 使用动词
GET    /users/1/posts/2/comments/3  // 嵌套过深
```

### 资源关系

```php
// 一级资源
GET    /users
GET    /posts
GET    /comments

// 二级资源（关联资源）
GET    /users/1/posts          // 用户 1 的文章
GET    /posts/1/comments       // 文章 1 的评论

// 三级资源（通常避免）
GET    /users/1/posts/2/comments  // 用户 1 的文章 2 的评论
```

### 查询参数

```php
// 分页
GET /users?page=1&per_page=20

// 过滤
GET /users?status=active
GET /users?role=admin&status=active

// 排序
GET /users?sort=name&order=asc

// 搜索
GET /users?search=alice

// 字段选择
GET /users?fields=id,name,email
```

## HTTP 状态码

### 常用状态码

| 状态码 | 说明           | 使用场景                           |
| :----- | :------------- | :--------------------------------- |
| `200`  | OK             | 请求成功                           |
| `201`  | Created        | 资源创建成功                       |
| `204`  | No Content     | 删除成功，无返回内容               |
| `400`  | Bad Request    | 请求参数错误                       |
| `401`  | Unauthorized   | 未认证                             |
| `403`  | Forbidden      | 已认证但无权限                     |
| `404`  | Not Found      | 资源不存在                         |
| `422`  | Unprocessable  | 验证失败                           |
| `429`  | Too Many       | 请求过于频繁                       |
| `500`  | Server Error   | 服务器内部错误                     |

### 状态码使用示例

```php
<?php
declare(strict_types=1);

function sendResponse(mixed $data, int $statusCode = 200): void
{
    http_response_code($statusCode);
    header('Content-Type: application/json; charset=UTF-8');
    
    if ($statusCode === 204) {
        // 204 No Content 不返回 body
        return;
    }
    
    echo json_encode($data, JSON_UNESCAPED_UNICODE);
}

// 成功响应
sendResponse(['id' => 1, 'name' => 'Alice'], 200);

// 创建成功
sendResponse(['id' => 2, 'name' => 'Bob'], 201);

// 删除成功
sendResponse(null, 204);

// 错误响应
sendResponse(['error' => 'User not found'], 404);
```

## 错误响应格式

### 标准错误格式

```php
<?php
declare(strict_types=1);

class ApiResponse
{
    public static function success(mixed $data = null, int $code = 200): void
    {
        http_response_code($code);
        header('Content-Type: application/json; charset=UTF-8');
        
        $response = ['success' => true];
        if ($data !== null) {
            $response['data'] = $data;
        }
        
        echo json_encode($response, JSON_UNESCAPED_UNICODE);
    }

    public static function error(
        string $message,
        int $code = 400,
        ?array $errors = null,
        ?string $errorCode = null
    ): void {
        http_response_code($code);
        header('Content-Type: application/json; charset=UTF-8');
        
        $response = [
            'success' => false,
            'error' => [
                'message' => $message,
            ],
        ];
        
        if ($errorCode !== null) {
            $response['error']['code'] = $errorCode;
        }
        
        if ($errors !== null) {
            $response['error']['errors'] = $errors;
        }
        
        echo json_encode($response, JSON_UNESCAPED_UNICODE);
    }
}

// 使用示例
ApiResponse::success(['id' => 1, 'name' => 'Alice']);

ApiResponse::error('Validation failed', 422, [
    'email' => ['Email is required'],
    'name' => ['Name must be at least 3 characters'],
], 'VALIDATION_ERROR');
```

### 错误响应示例

```json
// 400 Bad Request
{
  "success": false,
  "error": {
    "message": "Invalid request parameters",
    "code": "INVALID_PARAMS"
  }
}

// 422 Unprocessable Entity
{
  "success": false,
  "error": {
    "message": "Validation failed",
    "code": "VALIDATION_ERROR",
    "errors": {
      "email": ["Email is required", "Email format is invalid"],
      "name": ["Name must be at least 3 characters"]
    }
  }
}

// 404 Not Found
{
  "success": false,
  "error": {
    "message": "User not found",
    "code": "RESOURCE_NOT_FOUND"
  }
}
```

## JSON API 标准

### JSON API 格式

```json
// 单个资源
{
  "data": {
    "type": "users",
    "id": "1",
    "attributes": {
      "name": "Alice",
      "email": "alice@example.com"
    },
    "relationships": {
      "posts": {
        "data": [
          {"type": "posts", "id": "1"},
          {"type": "posts", "id": "2"}
        ]
      }
    }
  }
}

// 资源集合
{
  "data": [
    {
      "type": "users",
      "id": "1",
      "attributes": {
        "name": "Alice",
        "email": "alice@example.com"
      }
    },
    {
      "type": "users",
      "id": "2",
      "attributes": {
        "name": "Bob",
        "email": "bob@example.com"
      }
    }
  ],
  "meta": {
    "total": 100,
    "page": 1,
    "per_page": 20
  },
  "links": {
    "self": "/users?page=1",
    "first": "/users?page=1",
    "last": "/users?page=5",
    "prev": null,
    "next": "/users?page=2"
  }
}
```

### 简化版本（常用）

```json
// 单个资源
{
  "data": {
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com"
  }
}

// 资源集合
{
  "data": [
    {"id": 1, "name": "Alice"},
    {"id": 2, "name": "Bob"}
  ],
  "meta": {
    "total": 100,
    "page": 1,
    "per_page": 20
  }
}
```

## 分页设计

### 分页参数

```php
<?php
declare(strict_types=1);

function getPaginationParams(): array
{
    $page = max(1, (int) ($_GET['page'] ?? 1));
    $perPage = min(100, max(1, (int) ($_GET['per_page'] ?? 20)));
    $offset = ($page - 1) * $perPage;
    
    return [
        'page' => $page,
        'per_page' => $perPage,
        'offset' => $offset,
        'limit' => $perPage,
    ];
}

function buildPaginationResponse(
    array $data,
    int $total,
    int $page,
    int $perPage,
    string $baseUrl
): array {
    $totalPages = (int) ceil($total / $perPage);
    
    return [
        'data' => $data,
        'meta' => [
            'total' => $total,
            'page' => $page,
            'per_page' => $perPage,
            'total_pages' => $totalPages,
        ],
        'links' => [
            'self' => "{$baseUrl}?page={$page}",
            'first' => "{$baseUrl}?page=1",
            'last' => "{$baseUrl}?page={$totalPages}",
            'prev' => $page > 1 ? "{$baseUrl}?page=" . ($page - 1) : null,
            'next' => $page < $totalPages ? "{$baseUrl}?page=" . ($page + 1) : null,
        ],
    ];
}
```

## 版本控制

### URL 版本控制

```php
// v1 API
GET /api/v1/users

// v2 API
GET /api/v2/users
```

### Header 版本控制

```php
// 请求头
Accept: application/vnd.api+json;version=2

// 处理
$acceptHeader = $_SERVER['HTTP_ACCEPT'] ?? '';
if (strpos($acceptHeader, 'version=2') !== false) {
    // 使用 v2 逻辑
}
```

## 完整 API 示例

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

header('Content-Type: application/json; charset=UTF-8');

$method = $_SERVER['REQUEST_METHOD'];
$path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

// 路由
if (preg_match('#^/api/v1/users(?:/(\d+))?$#', $path, $matches)) {
    $userId = isset($matches[1]) ? (int) $matches[1] : null;
    
    match ($method) {
        'GET' => $userId ? getUser($userId) : getUsers(),
        'POST' => createUser(),
        'PUT' => $userId ? updateUser($userId) : sendError('Method not allowed', 405),
        'DELETE' => $userId ? deleteUser($userId) : sendError('Method not allowed', 405),
        default => sendError('Method not allowed', 405),
    };
} else {
    sendError('Not found', 404);
}

function getUsers(): void
{
    $params = getPaginationParams();
    $users = fetchUsers($params['offset'], $params['limit']);
    $total = countUsers();
    
    $response = buildPaginationResponse(
        $users,
        $total,
        $params['page'],
        $params['per_page'],
        '/api/v1/users'
    );
    
    echo json_encode($response, JSON_UNESCAPED_UNICODE);
}

function getUser(int $id): void
{
    $user = fetchUserById($id);
    
    if ($user === null) {
        sendError('User not found', 404);
        return;
    }
    
    echo json_encode(['data' => $user], JSON_UNESCAPED_UNICODE);
}

function createUser(): void
{
    $data = getJsonInput();
    
    $errors = validateUser($data);
    if (!empty($errors)) {
        sendError('Validation failed', 422, $errors);
        return;
    }
    
    $user = saveUser($data);
    
    http_response_code(201);
    echo json_encode(['data' => $user], JSON_UNESCAPED_UNICODE);
}

function sendError(string $message, int $code, ?array $errors = null): void
{
    http_response_code($code);
    
    $response = [
        'success' => false,
        'error' => ['message' => $message],
    ];
    
    if ($errors !== null) {
        $response['error']['errors'] = $errors;
    }
    
    echo json_encode($response, JSON_UNESCAPED_UNICODE);
}
```

## 最佳实践

### 1. 使用名词，避免动词

```php
// 好的
GET /users
POST /users

// 不好的
GET /getUsers
POST /createUser
```

### 2. 使用 HTTP 状态码

```php
// 好的
http_response_code(404);

// 不好的
echo json_encode(['status' => 'error', 'code' => 404]);
```

### 3. 提供清晰的错误信息

```php
// 好的
{
  "error": {
    "message": "Validation failed",
    "errors": {
      "email": ["Email is required"]
    }
  }
}
```

### 4. 支持内容协商

```php
$accept = $_SERVER['HTTP_ACCEPT'] ?? 'application/json';
if (strpos($accept, 'application/json') !== false) {
    header('Content-Type: application/json');
} elseif (strpos($accept, 'application/xml') !== false) {
    header('Content-Type: application/xml');
}
```

## 练习

1. 设计一个 RESTful API，实现用户的 CRUD 操作，使用标准的 HTTP 方法和状态码。

2. 实现一个分页系统，支持 page、per_page 参数，返回包含 meta 和 links 的响应。

3. 创建一个错误处理中间件，统一处理异常并返回标准化的错误响应。

4. 设计一个资源关系 API，实现 `/users/1/posts` 这样的嵌套资源路径。

5. 实现 API 版本控制，支持 `/api/v1` 和 `/api/v2` 两个版本。

6. 创建一个 API 文档生成器，根据路由定义自动生成 OpenAPI/Swagger 文档。
