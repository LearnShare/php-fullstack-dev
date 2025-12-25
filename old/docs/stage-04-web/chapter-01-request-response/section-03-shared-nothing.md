# 4.1.3 Shared Nothing 架构

## 概述

Shared Nothing 架构是 PHP Web 应用的核心设计原则。本节详细介绍 Shared Nothing 的核心特点、优势与限制、状态管理方案、水平扩展策略，以及在实际项目中的最佳实践。

## Shared Nothing 核心特点

### 1. 无状态设计

每个请求独立处理，不依赖共享内存状态：

```php
<?php
// [不推荐] 使用全局变量（进程间不共享）
$globalCounter = 0;

function incrementCounter(): void
{
    global $globalCounter;
    $globalCounter++;  // 每个进程独立，无法跨请求共享
}

// [推荐] 使用外部存储
function incrementCounter(): void
{
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379);
    $redis->incr('global_counter');  // 所有进程共享
}
```

### 2. 进程隔离

每个 Worker 进程独立运行，互不影响：

```php
<?php
// 进程 A 崩溃不影响进程 B
// 每个进程有独立的内存空间
// 进程间无法直接共享变量
```

### 3. 可水平扩展

可以轻松增加服务器或进程数：

```
服务器 1 (3 个进程)
服务器 2 (3 个进程)
服务器 3 (3 个进程)
───────────────────
总计：9 个进程，可处理 9 倍并发
```

## 优势

### 1. 高并发

- 多个进程可以同时处理请求
- 无锁竞争，性能更好
- 可以充分利用多核 CPU

### 2. 容错性

- 一个进程崩溃不影响其他进程
- 进程自动重启机制
- 故障隔离

### 3. 简单性

- 无需处理共享状态、锁等复杂问题
- 代码逻辑更清晰
- 易于调试和维护

### 4. 可扩展性

- 水平扩展：增加服务器或进程
- 垂直扩展：增加服务器资源
- 弹性伸缩：根据负载动态调整

## 限制

### 1. 状态存储

需要外部存储保存状态：

```php
<?php
// [不推荐] 无法使用进程内变量
$userSession = [];  // 进程重启后丢失

// [推荐] 使用外部存储
// 1. 数据库
$db->query('SELECT * FROM sessions WHERE id = ?', [$sessionId]);

// 2. Redis
$redis->get("session:{$sessionId}");

// 3. 文件系统（不推荐，性能差）
file_get_contents("/tmp/session_{$sessionId}.json");
```

### 2. 内存共享

进程间无法直接共享内存：

```php
<?php
// [不推荐] 无法直接共享
$sharedCache = [];  // 每个进程独立

// [推荐] 使用外部缓存
$redis = new Redis();
$redis->set('cache:key', $value);
```

### 3. 会话管理

需要外部会话存储：

```php
<?php
// 使用 Redis 存储 Session
ini_set('session.save_handler', 'redis');
ini_set('session.save_path', 'tcp://127.0.0.1:6379');
session_start();
```

## 状态管理方案

### 1. 数据库存储

```php
<?php
declare(strict_types=1);

class DatabaseStateManager
{
    public function __construct(private PDO $pdo) {}

    public function getCounter(string $name): int
    {
        $stmt = $this->pdo->prepare('SELECT value FROM counters WHERE name = ?');
        $stmt->execute([$name]);
        $result = $stmt->fetch(PDO::FETCH_ASSOC);
        return (int) ($result['value'] ?? 0);
    }

    public function incrementCounter(string $name): void
    {
        $this->pdo->prepare(
            'INSERT INTO counters (name, value) VALUES (?, 1)
             ON DUPLICATE KEY UPDATE value = value + 1'
        )->execute([$name]);
    }
}
```

### 2. Redis 缓存

```php
<?php
declare(strict_types=1);

class RedisStateManager
{
    public function __construct(private Redis $redis) {}

    public function getCounter(string $name): int
    {
        return (int) $this->redis->get("counter:{$name}") ?: 0;
    }

    public function incrementCounter(string $name): void
    {
        $this->redis->incr("counter:{$name}");
    }

    public function setCache(string $key, mixed $value, int $ttl = 3600): void
    {
        $this->redis->setex($key, $ttl, serialize($value));
    }

    public function getCache(string $key): mixed
    {
        $data = $this->redis->get($key);
        return $data ? unserialize($data) : null;
    }
}
```

### 3. Session 存储

```php
<?php
declare(strict_types=1);

// 配置 Session 存储到 Redis
ini_set('session.save_handler', 'redis');
ini_set('session.save_path', 'tcp://127.0.0.1:6379?prefix=session:');

session_start();

// 使用 Session
$_SESSION['user_id'] = 123;
$_SESSION['cart'] = ['item1', 'item2'];
```

### 4. 文件系统（不推荐）

```php
<?php
// 仅用于开发环境或小规模应用
$stateFile = "/tmp/app_state_{$key}.json";
file_put_contents($stateFile, json_encode($state));
```

## 水平扩展策略

### 1. 负载均衡

```nginx
# Nginx 负载均衡配置
upstream php_backend {
    least_conn;  # 最少连接算法
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

### 2. 会话粘性（可选）

```nginx
# 基于 IP 的会话保持
upstream php_backend {
    ip_hash;  # 同一 IP 总是路由到同一服务器
    server 192.168.1.10:9000;
    server 192.168.1.11:9000;
}
```

### 3. 共享存储

- **数据库**：所有服务器共享同一数据库
- **Redis**：所有服务器共享 Redis 集群
- **文件系统**：使用 NFS 或对象存储（S3）

## 最佳实践

### 1. 避免全局状态

```php
<?php
// [不推荐] 不推荐
$globalConfig = [];

// [推荐] 推荐：使用配置类
class Config
{
    private static array $config = [];

    public static function get(string $key, mixed $default = null): mixed
    {
        return self::$config[$key] ?? $default;
    }

    public static function load(string $file): void
    {
        self::$config = require $file;
    }
}
```

### 2. 使用依赖注入

```php
<?php
declare(strict_types=1);

class UserService
{
    public function __construct(
        private UserRepository $repository,
        private CacheInterface $cache
    ) {}

    public function getUser(int $id): ?User
    {
        $cacheKey = "user:{$id}";
        $user = $this->cache->get($cacheKey);
        
        if ($user === null) {
            $user = $this->repository->find($id);
            $this->cache->set($cacheKey, $user, 3600);
        }
        
        return $user;
    }
}
```

### 3. 外部化配置

```php
<?php
// 从环境变量读取配置
$dbHost = getenv('DB_HOST') ?: 'localhost';
$dbName = getenv('DB_NAME') ?: 'app';
$redisHost = getenv('REDIS_HOST') ?: '127.0.0.1';
```

## 完整示例

```php
<?php
declare(strict_types=1);

class SharedNothingDemo
{
    public function __construct(
        private Redis $redis,
        private PDO $pdo
    ) {}

    /**
     * 演示计数器（使用 Redis）
     */
    public function incrementCounter(string $name): int
    {
        return $this->redis->incr("counter:{$name}");
    }

    /**
     * 演示会话管理（使用 Redis Session）
     */
    public function setSession(string $key, mixed $value): void
    {
        $_SESSION[$key] = $value;
    }

    /**
     * 演示缓存（使用 Redis）
     */
    public function getCachedData(string $key): mixed
    {
        $data = $this->redis->get("cache:{$key}");
        if ($data === false) {
            // 从数据库加载
            $data = $this->loadFromDatabase($key);
            $this->redis->setex("cache:{$key}", 3600, serialize($data));
            return $data;
        }
        return unserialize($data);
    }

    /**
     * 演示数据库状态（使用 MySQL）
     */
    public function updateUserStatus(int $userId, string $status): void
    {
        $this->pdo->prepare(
            'UPDATE users SET status = ? WHERE id = ?'
        )->execute([$status, $userId]);
    }

    private function loadFromDatabase(string $key): mixed
    {
        // 模拟数据库查询
        return ['data' => 'value'];
    }
}

// 使用示例
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

$pdo = new PDO('mysql:host=localhost;dbname=app', 'user', 'password');

$demo = new SharedNothingDemo($redis, $pdo);

// 计数器
$count = $demo->incrementCounter('visits');
echo "Visits: {$count}\n";

// 会话
session_start();
$demo->setSession('user_id', 123);

// 缓存
$data = $demo->getCachedData('user:123');
```

## 注意事项

1. **状态外部化**：所有需要跨请求共享的状态必须存储在外部
2. **会话存储**：使用 Redis 或数据库存储 Session，不要使用文件
3. **缓存策略**：合理使用缓存，避免缓存穿透和雪崩
4. **数据一致性**：使用事务和锁保证数据一致性

## 练习

1. 设计一个简单的 Web 应用，演示 Shared Nothing 架构的特点，使用外部存储（Redis/数据库）保存应用状态。

2. 实现一个分布式计数器，使用 Redis 实现跨进程的计数功能。

3. 配置 Redis Session 存储，实现跨服务器的会话共享。

4. 实现一个简单的负载均衡配置，演示水平扩展的能力。

5. 对比 Shared Nothing 架构与传统共享内存架构的优缺点，总结适用场景。
