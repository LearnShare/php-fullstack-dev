# 1.7.5 实际应用与最佳实践

## 概述

本章总结 Worker 模式、协程、Event Loop 和 Fibers 的实际应用场景和最佳实践。

## 实际应用场景

### 1. 并发 HTTP 请求

```php
<?php
declare(strict_types=1);

use OpenSwoole\Coroutine;
use OpenSwoole\Coroutine\Http\Client;

function fetchMultipleUrls(array $urls): array
{
    $results = [];
    
    foreach ($urls as $url) {
        Coroutine::create(function () use ($url, &$results) {
            $host = parse_url($url, PHP_URL_HOST);
            $port = parse_url($url, PHP_URL_PORT) ?: 443;
            $path = parse_url($url, PHP_URL_PATH);
            
            $client = new Client($host, $port, true);
            $client->get($path);
            $results[$url] = $client->body;
            $client->close();
        });
    }
    
    Coroutine::wait();
    return $results;
}
```

### 2. 数据库连接池

```php
<?php
declare(strict_types=1);

use OpenSwoole\Coroutine;
use OpenSwoole\Coroutine\MySQL;

class ConnectionPool
{
    private array $connections = [];
    private int $maxConnections = 10;
    
    public function getConnection(): MySQL
    {
        if (empty($this->connections)) {
            return $this->createConnection();
        }
        
        return array_pop($this->connections);
    }
    
    public function releaseConnection(MySQL $connection): void
    {
        if (count($this->connections) < $this->maxConnections) {
            $this->connections[] = $connection;
        } else {
            $connection->close();
        }
    }
    
    private function createConnection(): MySQL
    {
        $mysql = new MySQL();
        $mysql->connect([
            'host' => '127.0.0.1',
            'port' => 3306,
            'user' => 'root',
            'password' => 'password',
            'database' => 'test'
        ]);
        
        return $mysql;
    }
}

// 在协程中使用
$pool = new ConnectionPool();

Coroutine::create(function () use ($pool) {
    $db = $pool->getConnection();
    try {
        $result = $db->query('SELECT * FROM users');
        // ...
    } finally {
        $pool->releaseConnection($db);
    }
});
```

### 3. 任务队列处理

```php
<?php
declare(strict_types=1);

use OpenSwoole\Coroutine;

class TaskQueue
{
    private array $queue = [];
    private int $maxWorkers = 10;
    
    public function addTask(callable $task): void
    {
        $this->queue[] = $task;
    }
    
    public function process(): void
    {
        while (!empty($this->queue)) {
            $activeWorkers = 0;
            
            foreach ($this->queue as $key => $task) {
                if ($activeWorkers < $this->maxWorkers) {
                    Coroutine::create(function () use ($task, $key) {
                        $task();
                        unset($this->queue[$key]);
                    });
                    $activeWorkers++;
                }
            }
            
            Coroutine::sleep(0.1);
        }
    }
}
```

## 最佳实践

### 1. 避免阻塞操作

```php
<?php
// 错误：阻塞操作
sleep(5); // 阻塞整个进程

// 正确：使用协程睡眠
Coroutine::sleep(5); // 只阻塞当前协程
```

### 2. 合理使用协程数量

```php
<?php
// 错误：创建过多协程
for ($i = 0; $i < 100000; $i++) {
    Coroutine::create(function () {
        // ...
    });
}

// 正确：使用协程池
class CoroutinePool
{
    private int $maxCoroutines = 100;
    private int $current = 0;
    
    public function run(callable $task): void
    {
        while ($this->current >= $this->maxCoroutines) {
            Coroutine::sleep(0.1);
        }
        
        $this->current++;
        Coroutine::create(function () use ($task) {
            try {
                $task();
            } finally {
                $this->current--;
            }
        });
    }
}
```

### 3. 错误处理

```php
<?php
Coroutine::create(function () {
    try {
        // 可能抛出异常的操作
        riskyOperation();
    } catch (Throwable $e) {
        // 处理异常
        error_log($e->getMessage());
    }
});
```

### 4. 资源管理

```php
<?php
Coroutine::create(function () use ($pool) {
    $resource = $pool->acquire();
    try {
        // 使用资源
        useResource($resource);
    } finally {
        // 确保释放资源
        $pool->release($resource);
    }
});
```

### 5. 状态管理

```php
<?php
// 错误：在 Worker 中存储状态
class Counter
{
    private static int $count = 0; // 会在 Worker 间共享
    
    public static function increment(): void
    {
        self::$count++;
    }
}

// 正确：使用外部存储
class Counter
{
    public static function increment(): void
    {
        $redis = new Redis();
        $redis->incr('counter');
    }
}
```

## 性能优化

### 1. 连接复用

```php
<?php
// 在 Worker 启动时创建连接
$db = new PDO($dsn, $user, $pass);

while ($request = $psr7->waitRequest()) {
    // 复用数据库连接
    $result = $db->query('SELECT * FROM users');
    // ...
}
```

### 2. 内存管理

```php
<?php
// 定期清理内存
use Laravel\Octane\Events\TaskReceived;

Event::listen(TaskReceived::class, function () {
    // 清理缓存
    Cache::flush();
    
    // 重置单例
    app()->forgetInstances();
});
```

### 3. 协程池

```php
<?php
class CoroutinePool
{
    private int $maxCoroutines;
    private int $current = 0;
    private array $queue = [];
    
    public function __construct(int $maxCoroutines = 100)
    {
        $this->maxCoroutines = $maxCoroutines;
    }
    
    public function execute(callable $task): void
    {
        if ($this->current >= $this->maxCoroutines) {
            $this->queue[] = $task;
            return;
        }
        
        $this->current++;
        Coroutine::create(function () use ($task) {
            try {
                $task();
            } finally {
                $this->current--;
                if (!empty($this->queue)) {
                    $next = array_shift($this->queue);
                    $this->execute($next);
                }
            }
        });
    }
}
```

## 完整示例

### 高性能 API 服务器

```php
<?php
declare(strict_types=1);

use OpenSwoole\Http\Server;
use OpenSwoole\Http\Request;
use OpenSwoole\Http\Response;
use OpenSwoole\Coroutine;

class HighPerformanceAPI
{
    private Server $server;
    private ConnectionPool $dbPool;
    
    public function __construct()
    {
        $this->server = new Server("0.0.0.0", 9501);
        $this->dbPool = new ConnectionPool();
        
        $this->setupHandlers();
    }
    
    private function setupHandlers(): void
    {
        $this->server->on("request", function (Request $request, Response $response) {
            $path = $request->server['request_uri'];
            
            match($path) {
                '/api/users' => $this->handleUsers($request, $response),
                '/api/posts' => $this->handlePosts($request, $response),
                default => $response->status(404)->end('Not Found')
            };
        });
    }
    
    private function handleUsers(Request $request, Response $response): void
    {
        Coroutine::create(function () use ($request, $response) {
            $db = $this->dbPool->getConnection();
            try {
                $result = $db->query('SELECT * FROM users');
                $response->header('Content-Type', 'application/json');
                $response->end(json_encode($result));
            } finally {
                $this->dbPool->releaseConnection($db);
            }
        });
    }
    
    public function start(): void
    {
        $this->server->start();
    }
}
```

## 注意事项

1. **状态管理**：避免在 Worker 中存储请求间共享的状态。

2. **内存泄漏**：定期清理不需要的数据，防止内存泄漏。

3. **错误处理**：妥善处理异常，避免 Worker 崩溃。

4. **资源管理**：正确管理资源，确保及时释放。

5. **性能监控**：监控 Worker 状态和内存使用。

## 练习

1. 实现一个高性能的 API 服务器，使用协程处理请求。

2. 创建一个数据库连接池，在协程中复用连接。

3. 实现一个任务队列系统，使用协程处理任务。

4. 实现性能监控，跟踪协程和 Worker 的状态。

5. 对比传统同步代码和协程代码的性能差异。
