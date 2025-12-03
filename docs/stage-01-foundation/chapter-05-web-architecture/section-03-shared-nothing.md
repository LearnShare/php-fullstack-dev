# 1.5.3 Shared Nothing 架构

## 概述

Shared Nothing 架构是 PHP-FPM 的核心设计理念。理解这个架构对于编写可扩展的 PHP 应用至关重要。

## 核心概念

### 什么是 Shared Nothing

- **Shared Nothing**：每个请求都是独立的，不共享状态
- **无状态**：服务器不保存客户端状态信息
- **水平扩展**：可以轻松添加更多服务器

### 与 Node.js 对比

| 特性 | Node.js | PHP (FPM) |
| :--- | :------ | :-------- |
| 全局变量 | 所有请求共享 | 每个请求独立 |
| 内存状态 | 请求间共享 | 请求间隔离 |
| 并发模型 | 事件循环（单线程） | 多进程（多线程可选） |
| 状态管理 | 需要小心处理全局状态 | 天然隔离，更安全 |

### 示例对比

```javascript
// Node.js：全局变量在所有请求间共享
let requestCount = 0;  // 危险：所有请求共享

app.get('/api', (req, res) => {
    requestCount++;  // 所有请求都会修改同一个变量
    res.json({ count: requestCount });
});
```

```php
<?php
// PHP-FPM：每个请求有独立的内存空间
// 全局变量只在当前请求中有效
$requestCount = 0;  // 安全：每个请求独立

function handleRequest(): array {
    global $requestCount;  // 只在当前请求中有效
    $requestCount++;
    return ['count' => $requestCount];
}
```

## 请求生命周期

### 完整流程

```
1. 请求到达
   ↓
2. 创建新的 PHP 进程/线程
   ↓
3. 加载 PHP 代码和配置
   ↓
4. 处理请求
   ↓
5. 生成响应
   ↓
6. 进程/线程结束（或返回进程池）
   ↓
7. 内存释放
```

### 内存隔离

每个请求都有独立的内存空间：

```php
<?php
declare(strict_types=1);

// 请求 1
$data = ['user' => 'Alice'];  // 独立内存

// 请求 2
$data = ['user' => 'Bob'];    // 不同的内存空间，互不影响
```

## 状态管理

### 不共享的状态

以下状态在每个请求中都是独立的：

- **全局变量**：每个请求独立
- **静态变量**：每个进程独立
- **对象实例**：每个请求独立

### 需要共享的状态

以下状态需要外部存储：

- **数据库**：持久化存储
- **缓存（Redis/Memcached）**：共享缓存
- **Session 存储**：文件、数据库或 Redis
- **文件系统**：共享存储

### 状态管理示例

```php
<?php
declare(strict_types=1);

class RequestHandler
{
    public function handle(): void
    {
        // 从外部存储获取状态
        $sessionId = $_COOKIE['session_id'] ?? null;
        $session = $this->getSession($sessionId); // 从 Redis/数据库获取
        
        // 处理请求
        $response = $this->process($session);
        
        // 保存状态到外部存储
        $this->saveSession($session);
        
        // 返回响应
        echo $response;
    }
    
    private function getSession(?string $sessionId): array
    {
        if ($sessionId === null) {
            return [];
        }
        
        // 从 Redis 获取
        $redis = new Redis();
        $redis->connect('127.0.0.1', 6379);
        $data = $redis->get("session:{$sessionId}");
        
        return $data !== false ? json_decode($data, true) : [];
    }
    
    private function saveSession(array $session): void
    {
        $sessionId = $_COOKIE['session_id'] ?? bin2hex(random_bytes(16));
        
        $redis = new Redis();
        $redis->connect('127.0.0.1', 6379);
        $redis->setex("session:{$sessionId}", 3600, json_encode($session));
        
        setcookie('session_id', $sessionId, time() + 3600);
    }
}
```

## 高并发处理

### 水平扩展

由于每个请求都是独立的，可以轻松实现水平扩展：

```
负载均衡器
├── 服务器 1 (PHP-FPM)
├── 服务器 2 (PHP-FPM)
├── 服务器 3 (PHP-FPM)
└── ...
```

### 无状态设计

```php
<?php
declare(strict_types=1);

// 无状态设计：不依赖服务器内存状态
class StatelessService
{
    public function processRequest(array $data): array
    {
        // 所有需要的数据都从参数或外部存储获取
        $userId = $data['user_id'];
        $user = $this->getUserFromDatabase($userId);  // 从数据库获取
        
        // 处理逻辑
        $result = $this->process($user, $data);
        
        // 保存结果到数据库
        $this->saveResult($result);
        
        return $result;
    }
    
    private function getUserFromDatabase(int $userId): array
    {
        // 从数据库获取，不依赖内存
        $pdo = new PDO($dsn, $user, $pass);
        $stmt = $pdo->prepare('SELECT * FROM users WHERE id = ?');
        $stmt->execute([$userId]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }
}
```

## 优势

### 1. 水平扩展

可以轻松添加更多服务器：

```nginx
# 负载均衡配置
upstream php_backend {
    server 192.168.1.10:9000;
    server 192.168.1.11:9000;
    server 192.168.1.12:9000;
}

server {
    location ~ \.php$ {
        fastcgi_pass php_backend;
    }
}
```

### 2. 故障隔离

一个请求的故障不影响其他请求：

```php
<?php
declare(strict_types=1);

// 请求 1 出错，不影响请求 2
try {
    processRequest1();  // 可能出错
} catch (Exception $e) {
    // 只影响当前请求
}

// 请求 2 正常处理
processRequest2();
```

### 3. 无状态

简化部署和扩展：

- 不需要考虑服务器间的状态同步
- 可以随时添加或移除服务器
- 故障恢复更容易

### 4. 负载均衡

可以轻松实现负载均衡：

```php
<?php
declare(strict_types=1);

// 任何服务器都可以处理任何请求
// 不需要考虑请求的"归属"
```

## 注意事项

### 1. 不要使用全局变量存储状态

```php
<?php
// 错误示例
$globalCounter = 0; // 每个请求都是独立的

// 正确做法
// 使用数据库或缓存存储计数器
$redis = new Redis();
$redis->incr('counter');
```

### 2. Session 存储

```php
<?php
// 不要依赖文件 Session（多服务器时会有问题）
// 使用 Redis 或数据库存储 Session

ini_set('session.save_handler', 'redis');
ini_set('session.save_path', 'tcp://127.0.0.1:6379');
```

### 3. 文件上传

```php
<?php
// 使用共享存储（如 S3、NFS）存储上传的文件
// 不要存储在本地文件系统

function uploadFile(array $file): string
{
    // 上传到 S3 或 NFS
    $s3 = new S3Client();
    $key = 'uploads/' . bin2hex(random_bytes(16)) . '.jpg';
    $s3->putObject([
        'Bucket' => 'my-bucket',
        'Key' => $key,
        'Body' => fopen($file['tmp_name'], 'r')
    ]);
    
    return $key;
}
```

### 4. 缓存策略

```php
<?php
declare(strict_types=1);

// 使用共享缓存（Redis/Memcached）
// 不要使用内存缓存

class CacheService
{
    private Redis $redis;
    
    public function __construct()
    {
        $this->redis = new Redis();
        $this->redis->connect('127.0.0.1', 6379);
    }
    
    public function get(string $key): mixed
    {
        $data = $this->redis->get($key);
        return $data !== false ? json_decode($data, true) : null;
    }
    
    public function set(string $key, mixed $value, int $ttl = 3600): void
    {
        $this->redis->setex($key, $ttl, json_encode($value));
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class SharedNothingExample
{
    private Redis $redis;
    private PDO $db;
    
    public function __construct()
    {
        // 外部存储连接
        $this->redis = new Redis();
        $this->redis->connect('127.0.0.1', 6379);
        
        $this->db = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');
    }
    
    public function handleRequest(): void
    {
        // 从外部存储获取状态
        $sessionId = $_COOKIE['session_id'] ?? null;
        $session = $this->getSession($sessionId);
        
        // 处理请求（无状态）
        $result = $this->process($session);
        
        // 保存状态到外部存储
        $this->saveSession($session);
        
        // 返回响应
        echo json_encode($result);
    }
    
    private function getSession(?string $sessionId): array
    {
        if ($sessionId === null) {
            return [];
        }
        
        $data = $this->redis->get("session:{$sessionId}");
        return $data !== false ? json_decode($data, true) : [];
    }
    
    private function saveSession(array $session): void
    {
        $sessionId = $_COOKIE['session_id'] ?? bin2hex(random_bytes(16));
        $this->redis->setex("session:{$sessionId}", 3600, json_encode($session));
        setcookie('session_id', $sessionId, time() + 3600);
    }
    
    private function process(array $session): array
    {
        // 无状态处理逻辑
        // 所有数据都从参数或外部存储获取
        return ['status' => 'success', 'data' => $session];
    }
}
```

## 注意事项

1. **状态存储**：所有需要共享的状态都存储在外部（数据库、缓存、文件系统）。

2. **Session 管理**：使用 Redis 或数据库存储 Session，不要使用文件 Session。

3. **文件存储**：使用共享存储（S3、NFS）存储文件，不要存储在本地。

4. **缓存策略**：使用共享缓存（Redis/Memcached），不要使用内存缓存。

5. **水平扩展**：设计时考虑水平扩展，避免依赖服务器本地状态。

## 练习

1. 实现一个无状态的 Session 管理系统，使用 Redis 存储。

2. 创建一个无状态的 API 服务，所有数据都从数据库获取。

3. 实现文件上传功能，使用共享存储（S3 或 NFS）。

4. 配置负载均衡，将请求分发到多个 PHP-FPM 服务器。

5. 对比有状态和无状态设计的差异，理解各自的适用场景。
