# 4.1 Web 交互基础：从请求到响应

## 目标

- 理解 HTTP 请求响应的完整流程，从浏览器到服务器再到浏览器。
- 掌握 Web Server（Nginx/Apache）与 PHP-FPM 的协作机制。
- 理解 PHP 在 Web 请求处理中的角色与生命周期。
- 熟悉 Shared Nothing 架构的特点与优势。

## HTTP 请求响应流程

### 完整流程概览

```
浏览器
  ↓ (1. 发送 HTTP 请求)
Web Server (Nginx/Apache)
  ↓ (2. 转发请求)
PHP-FPM / 运行时
  ↓ (3. 执行 PHP 代码)
PHP 脚本处理
  ↓ (4. 生成响应)
Web Server
  ↓ (5. 返回 HTTP 响应)
浏览器
```

### 详细流程说明

#### 1. 浏览器发送请求

- 用户在浏览器输入 URL 或点击链接。
- 浏览器构造 HTTP 请求（包含方法、URL、Headers、Body）。
- 请求通过 TCP/IP 协议发送到服务器。

**示例请求：**

```
GET /users/1 HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: application/json
```

#### 2. Web Server 接收请求

- **Nginx** 或 **Apache** 接收 HTTP 请求。
- 根据配置决定如何处理请求：
  - 静态文件：直接返回文件内容。
  - PHP 文件：转发给 PHP-FPM 处理。

**Nginx 配置示例：**

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;

    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

#### 3. PHP-FPM 处理请求

- **FastCGI Process Manager (FPM)** 接收请求。
- 创建或复用 PHP 进程执行脚本。
- 将请求信息传递给 PHP 脚本（通过超全局变量）。

**PHP-FPM 工作原理：**

- **Master 进程**：管理 Worker 进程。
- **Worker 进程**：执行 PHP 代码，处理请求。
- **进程池**：`pm.max_children` 控制最大进程数。

#### 4. PHP 脚本执行

- PHP 脚本接收请求数据（`$_GET`、`$_POST`、`$_SERVER` 等）。
- 执行业务逻辑（数据库查询、计算等）。
- 生成响应内容（HTML、JSON、文件等）。

**示例 PHP 脚本：**

```php
<?php
declare(strict_types=1);

// 接收请求
$userId = $_GET['id'] ?? null;

// 处理业务逻辑
if ($userId === null) {
    http_response_code(400);
    echo json_encode(['error' => 'User ID required']);
    exit;
}

// 模拟数据库查询
$user = [
    'id' => (int) $userId,
    'name' => 'Alice',
    'email' => 'alice@example.com',
];

// 生成响应
header('Content-Type: application/json');
echo json_encode($user, JSON_UNESCAPED_UNICODE);
```

#### 5. 响应返回浏览器

- PHP 脚本输出内容。
- PHP-FPM 将响应返回给 Web Server。
- Web Server 添加 HTTP Headers，返回给浏览器。
- 浏览器解析响应并渲染页面。

**示例响应：**

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 65

{"id":1,"name":"Alice","email":"alice@example.com"}
```

## PHP-FPM 工作原理

### FastCGI 协议

- **FastCGI** 是 CGI 的改进版本，支持持久连接。
- PHP-FPM 通过 FastCGI 协议与 Web Server 通信。
- 避免了每次请求都启动新进程的开销。

### FPM Pool 配置

**php-fpm.conf 配置示例：**

```ini
[www]
user = www-data
group = www-data
listen = 127.0.0.1:9000
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
pm.max_requests = 500
```

**配置说明：**

| 配置项              | 说明                           |
| :------------------ | :----------------------------- |
| `pm.max_children`   | 最大 Worker 进程数            |
| `pm.start_servers`  | 启动时的进程数                 |
| `pm.min_spare_servers` | 最小空闲进程数             |
| `pm.max_spare_servers` | 最大空闲进程数             |
| `pm.max_requests`   | 每个进程处理的最大请求数（防止内存泄漏） |

### Worker 进程生命周期

```
请求到达
  ↓
Worker 进程接收请求
  ↓
执行 PHP 脚本
  ↓
返回响应
  ↓
等待下一个请求（或达到 max_requests 后重启）
```

## Shared Nothing 架构

### 核心特点

- **无状态**：每个请求独立处理，不共享内存状态。
- **进程隔离**：每个 Worker 进程独立，互不影响。
- **可扩展**：可以水平扩展，增加服务器或进程数。

### 优势

1. **高并发**：多个进程可以同时处理请求。
2. **容错性**：一个进程崩溃不影响其他进程。
3. **简单性**：无需处理共享状态、锁等复杂问题。

### 限制

- **状态存储**：需要外部存储（数据库、Redis、Session）保存状态。
- **内存共享**：进程间无法直接共享内存，需要外部机制。

### 状态管理方案

```php
// 不推荐：使用全局变量（进程间不共享）
$globalCounter = 0; // 每个进程独立

// 推荐：使用外部存储
// 1. 数据库
$db->query('UPDATE counters SET value = value + 1 WHERE name = "visits"');

// 2. Redis
$redis->incr('visits');

// 3. Session（存储在文件或 Redis）
$_SESSION['visits'] = ($_SESSION['visits'] ?? 0) + 1;
```

## 请求生命周期示例

### 完整示例

```php
<?php
declare(strict_types=1);

// 1. 请求开始
$startTime = microtime(true);

// 2. 获取请求信息
$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
$uri = $_SERVER['REQUEST_URI'] ?? '/';
$headers = getallheaders();

// 3. 路由处理（简化示例）
$routes = [
    'GET /users' => 'listUsers',
    'GET /users/{id}' => 'getUser',
    'POST /users' => 'createUser',
];

// 4. 业务逻辑处理
function listUsers(): array
{
    // 模拟数据库查询
    return [
        ['id' => 1, 'name' => 'Alice'],
        ['id' => 2, 'name' => 'Bob'],
    ];
}

function getUser(int $id): ?array
{
    if ($id === 1) {
        return ['id' => 1, 'name' => 'Alice', 'email' => 'alice@example.com'];
    }
    return null;
}

function createUser(array $data): array
{
    // 验证数据
    if (empty($data['name']) || empty($data['email'])) {
        http_response_code(400);
        return ['error' => 'Name and email are required'];
    }

    // 保存用户（模拟）
    return [
        'id' => 3,
        'name' => $data['name'],
        'email' => $data['email'],
    ];
}

// 5. 路由匹配与处理
$path = parse_url($uri, PHP_URL_PATH);
$route = $routes["{$method} {$path}"] ?? null;

if ($route === null) {
    http_response_code(404);
    echo json_encode(['error' => 'Not found']);
    exit;
}

// 6. 执行处理函数
try {
    $result = match ($route) {
        'listUsers' => listUsers(),
        'getUser' => getUser((int) ($_GET['id'] ?? 0)),
        'createUser' => createUser(json_decode(file_get_contents('php://input'), true) ?? []),
    };

    // 7. 生成响应
    header('Content-Type: application/json');
    echo json_encode($result, JSON_UNESCAPED_UNICODE);
} catch (Exception $e) {
    http_response_code(500);
    echo json_encode(['error' => $e->getMessage()]);
}

// 8. 请求结束（可选：记录日志）
$duration = microtime(true) - $startTime;
error_log("Request {$method} {$uri} completed in {$duration}s");
```

## 性能优化建议

### 1. OPcache 配置

- 启用 OPcache 缓存编译后的 PHP 代码。
- 减少重复编译开销。

```ini
; php.ini
opcache.enable=1
opcache.memory_consumption=128
opcache.max_accelerated_files=10000
```

### 2. 进程池优化

- 根据服务器资源调整进程数。
- 监控进程使用情况，避免过度分配。

### 3. 连接池

- 数据库连接使用连接池。
- 避免每个请求都创建新连接。

```php
// 使用 PDO 连接池（通过持久连接）
$pdo = new PDO(
    'mysql:host=localhost;dbname=app',
    'user',
    'password',
    [PDO::ATTR_PERSISTENT => true]
);
```

## 调试技巧

### 查看请求信息

```php
// 打印所有请求信息
echo "<pre>";
echo "Method: " . $_SERVER['REQUEST_METHOD'] . "\n";
echo "URI: " . $_SERVER['REQUEST_URI'] . "\n";
echo "Headers:\n";
print_r(getallheaders());
echo "GET:\n";
print_r($_GET);
echo "POST:\n";
print_r($_POST);
echo "</pre>";
```

### 查看 PHP-FPM 状态

```php
// 访问 php-fpm status 页面（需配置）
// http://example.com/status?full
```

### 日志记录

```php
// 记录请求日志
error_log(sprintf(
    '[%s] %s %s - %s',
    date('Y-m-d H:i:s'),
    $_SERVER['REQUEST_METHOD'],
    $_SERVER['REQUEST_URI'],
    $_SERVER['REMOTE_ADDR']
));
```

## 练习

1. 创建一个简单的 PHP 脚本，输出所有 `$_SERVER` 变量的值，理解请求信息的结构。

2. 实现一个请求日志记录器，记录每个请求的方法、URI、IP 地址和处理时间。

3. 编写一个脚本，模拟不同的 HTTP 方法（GET、POST、PUT、DELETE），并返回相应的响应。

4. 创建一个简单的路由系统，根据请求方法和路径调用不同的处理函数。

5. 实现一个请求性能监控器，记录每个请求的执行时间，并在响应头中返回 `X-Response-Time`。

6. 设计一个简单的 Web 应用，演示 Shared Nothing 架构的特点，使用外部存储（文件或数据库）保存应用状态。
