# 4.6.2 API 路由与版本控制

## 概述

API 路由设计和版本控制是构建可维护 API 的关键。本节详细介绍路由设计、资源嵌套、版本控制策略、查询参数处理，以及分页与排序。

## 路由设计

### 基础路由

```php
<?php
declare(strict_types=1);

class Router
{
    private array $routes = [];

    public function get(string $path, callable $handler): void
    {
        $this->addRoute('GET', $path, $handler);
    }

    public function post(string $path, callable $handler): void
    {
        $this->addRoute('POST', $path, $handler);
    }

    public function addRoute(string $method, string $path, callable $handler): void
    {
        $this->routes[] = [
            'method' => $method,
            'path' => $path,
            'handler' => $handler,
        ];
    }

    public function dispatch(): void
    {
        $method = $_SERVER['REQUEST_METHOD'];
        $path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

        foreach ($this->routes as $route) {
            if ($route['method'] === $method && $this->matchPath($route['path'], $path, $params)) {
                call_user_func($route['handler'], $params);
                return;
            }
        }

        http_response_code(404);
        echo json_encode(['error' => 'Not found']);
    }

    private function matchPath(string $pattern, string $path, array &$params): bool
    {
        $pattern = preg_replace('#\{(\w+)\}#', '([^/]+)', $pattern);
        $pattern = "#^{$pattern}$#";
        
        if (preg_match($pattern, $path, $matches)) {
            array_shift($matches);
            $params = $matches;
            return true;
        }
        
        return false;
    }
}

// 使用
$router = new Router();
$router->get('/users/{id}', function ($params) {
    $userId = (int) $params[0];
    // 处理逻辑
});
$router->dispatch();
```

## 资源嵌套

### 嵌套资源设计

```php
<?php
// 用户文章 API
GET    /users/1/posts           # 获取用户 1 的文章列表
POST   /users/1/posts           # 为用户 1 创建文章
GET    /users/1/posts/5         # 获取用户 1 的文章 5
PUT    /users/1/posts/5         # 更新用户 1 的文章 5
DELETE /users/1/posts/5         # 删除用户 1 的文章 5

// 实现
$router->get('/users/{userId}/posts', function ($params) {
    $userId = (int) $params[0];
    $posts = getPostsByUserId($userId);
    sendJsonResponse($posts);
});

$router->post('/users/{userId}/posts', function ($params) {
    $userId = (int) $params[0];
    $data = getJsonInput();
    $post = createPost($userId, $data);
    http_response_code(201);
    sendJsonResponse($post);
});
```

## 版本控制

### URL 版本控制（推荐）

```php
<?php
// v1 API
GET /api/v1/users
GET /api/v2/users  // 新版本

// 路由处理
$version = $this->extractVersion($path);
match ($version) {
    'v1' => $this->handleV1($path),
    'v2' => $this->handleV2($path),
    default => $this->sendError('Unsupported version', 400),
};
```

### Header 版本控制

```php
<?php
// 通过 Accept Header
Accept: application/vnd.api+json;version=2

// 处理
$accept = $_SERVER['HTTP_ACCEPT'] ?? '';
if (preg_match('/version=(\d+)/', $accept, $matches)) {
    $version = 'v' . $matches[1];
}
```

## 查询参数

### 分页

```php
<?php
declare(strict_types=1);

function getUsers(int $page = 1, int $perPage = 20): array
{
    $offset = ($page - 1) * $perPage;
    
    // 查询数据
    $users = queryUsers($offset, $perPage);
    $total = countUsers();
    
    return [
        'data' => $users,
        'meta' => [
            'page' => $page,
            'per_page' => $perPage,
            'total' => $total,
            'total_pages' => (int) ceil($total / $perPage),
        ],
    ];
}

// 使用
$page = (int) ($_GET['page'] ?? 1);
$perPage = (int) ($_GET['per_page'] ?? 20);
$result = getUsers($page, $perPage);
```

### 排序

```php
<?php
declare(strict_types=1);

function getUsersWithSort(string $sort = 'id', string $order = 'asc'): array
{
    $allowedSorts = ['id', 'name', 'email', 'created_at'];
    $allowedOrders = ['asc', 'desc'];
    
    if (!in_array($sort, $allowedSorts, true)) {
        $sort = 'id';
    }
    
    if (!in_array($order, $allowedOrders, true)) {
        $order = 'asc';
    }
    
    return queryUsersWithSort($sort, $order);
}

// 使用
$sort = $_GET['sort'] ?? 'id';
$order = $_GET['order'] ?? 'asc';
$users = getUsersWithSort($sort, $order);
```

### 过滤

```php
<?php
declare(strict_types=1);

function getUsersWithFilter(array $filters): array
{
    $query = 'SELECT * FROM users WHERE 1=1';
    $params = [];
    
    if (isset($filters['status'])) {
        $query .= ' AND status = ?';
        $params[] = $filters['status'];
    }
    
    if (isset($filters['email'])) {
        $query .= ' AND email LIKE ?';
        $params[] = '%' . $filters['email'] . '%';
    }
    
    return queryWithParams($query, $params);
}

// 使用
$filters = [
    'status' => $_GET['status'] ?? null,
    'email' => $_GET['email'] ?? null,
];
$users = getUsersWithFilter(array_filter($filters));
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ApiRouter
{
    public function route(string $method, string $path): void
    {
        // 提取版本
        if (preg_match('#^/api/v(\d+)/(.+)$#', $path, $matches)) {
            $version = 'v' . $matches[1];
            $resourcePath = '/' . $matches[2];
            
            $this->handleVersioned($version, $method, $resourcePath);
            return;
        }
        
        $this->sendError('Invalid API path', 400);
    }

    private function handleVersioned(string $version, string $method, string $path): void
    {
        match ($version) {
            'v1' => $this->handleV1($method, $path),
            'v2' => $this->handleV2($method, $path),
            default => $this->sendError('Unsupported version', 400),
        };
    }

    private function handleV1(string $method, string $path): void
    {
        // v1 实现
    }
}
```

## 注意事项

1. **版本策略**：选择一种版本控制方式并保持一致
2. **向后兼容**：尽量保持旧版本可用
3. **查询参数**：统一分页、排序、过滤的格式
4. **资源嵌套**：合理设计嵌套层级，避免过深

## 练习

1. 实现一个 RESTful 路由系统，支持资源嵌套和参数提取。

2. 创建一个 API 版本控制系统，支持 URL 和 Header 两种方式。

3. 实现分页、排序、过滤功能，提供统一的查询参数处理。

4. 设计一个嵌套资源 API，如用户-文章-评论的三层嵌套。
