# 1.7.2 协程（Coroutine）

## 概述

协程是轻量级的并发执行单元，可以在执行过程中暂停和恢复。PHP 通过 OpenSwoole 等扩展支持协程，可以实现类似 Node.js 的异步编程模型。

## 什么是协程

### 基本概念

- **协程**：轻量级线程，可以在执行过程中暂停和恢复
- **特点**：比线程更轻量，可以创建大量协程
- **类比**：类似 Node.js 的 Promise，但更底层

### 与 Node.js 事件循环对比

**Node.js 事件循环**：
- 单线程事件循环
- 异步 I/O 操作不阻塞主线程
- 通过回调/Promise/async-await 处理异步操作

**PHP 协程**：
- 多协程并发执行
- 协程可以在 I/O 操作时暂停，让其他协程执行
- 通过协程实现类似 async-await 的效果

### 示例对比

```javascript
// Node.js：并发请求
async function fetchMultiple() {
    const [user, posts, comments] = await Promise.all([
        fetch('/api/user/1'),
        fetch('/api/posts?user_id=1'),
        fetch('/api/comments?user_id=1'),
    ]);
    
    return {
        user: await user.json(),
        posts: await posts.json(),
        comments: await comments.json(),
    };
}
```

```php
<?php
// PHP 协程：并发请求
use OpenSwoole\Coroutine;

function fetchMultiple(): array {
    $results = [];
    
    // 创建多个协程并发执行
    Coroutine::create(function () use (&$results) {
        $results['user'] = Coroutine\Http\get('http://api.example.com/user/1');
    });
    
    Coroutine::create(function () use (&$results) {
        $results['posts'] = Coroutine\Http\get('http://api.example.com/posts?user_id=1');
    });
    
    Coroutine::create(function () use (&$results) {
        $results['comments'] = Coroutine\Http\get('http://api.example.com/comments?user_id=1');
    });
    
    // 等待所有协程完成
    Coroutine::wait();
    
    return $results;
}
```

## 协程 vs 线程 vs 进程

| 特性 | 进程 | 线程 | 协程 |
| :--- | :--- | :--- | :--- |
| 创建成本 | 高 | 中 | 低 |
| 内存占用 | 高 | 中 | 低 |
| 切换成本 | 高 | 中 | 低 |
| 并发数量 | 少（数百） | 中（数千） | 多（数万） |

## OpenSwoole 协程

### 基本协程

```php
<?php
declare(strict_types=1);

use OpenSwoole\Coroutine;

// 创建多个协程
Coroutine::create(function () {
    echo "Coroutine 1 started\n";
    Coroutine::sleep(1);
    echo "Coroutine 1 finished\n";
});

Coroutine::create(function () {
    echo "Coroutine 2 started\n";
    Coroutine::sleep(1);
    echo "Coroutine 2 finished\n";
});

// 两个协程并发执行，总时间约 1 秒（而不是 2 秒）
```

### 协程 HTTP 客户端

```php
<?php
declare(strict_types=1);

use OpenSwoole\Coroutine;
use OpenSwoole\Coroutine\Http\Client;

Coroutine::create(function () {
    $client = new Client('example.com', 443, true);
    $client->get('/api/users');
    
    echo $client->body;
    $client->close();
});

// 可以创建多个协程并发请求
for ($i = 0; $i < 10; $i++) {
    Coroutine::create(function () use ($i) {
        $client = new Client('api.example.com', 443, true);
        $client->get("/api/data/{$i}");
        echo "Request {$i}: " . $client->body . "\n";
        $client->close();
    });
}
```

### 协程 MySQL

```php
<?php
declare(strict_types=1);

use OpenSwoole\Coroutine;
use OpenSwoole\Coroutine\MySQL;

Coroutine::create(function () {
    $mysql = new MySQL();
    $mysql->connect([
        'host' => '127.0.0.1',
        'port' => 3306,
        'user' => 'root',
        'password' => 'password',
        'database' => 'test'
    ]);
    
    $result = $mysql->query('SELECT * FROM users');
    print_r($result);
});
```

## 并发 HTTP 请求

### 实现并发请求

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
    
    // 等待所有协程完成
    Coroutine::wait();
    
    return $results;
}

// 使用
$urls = [
    'https://api1.example.com/data',
    'https://api2.example.com/data',
    'https://api3.example.com/data',
];

$results = fetchMultipleUrls($urls);
```

## 数据库连接池

### 协程连接池

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
        // 使用数据库连接
        $result = $db->query('SELECT * FROM users');
        // ...
    } finally {
        $pool->releaseConnection($db);
    }
});
```

## 完整示例

```php
<?php
declare(strict_types=1);

use OpenSwoole\Coroutine;
use OpenSwoole\Coroutine\Http\Client;

class CoroutineExample
{
    public static function demonstrate(): void
    {
        echo "=== Basic Coroutines ===\n";
        Coroutine::create(function () {
            echo "Coroutine 1\n";
            Coroutine::sleep(1);
        });
        
        Coroutine::create(function () {
            echo "Coroutine 2\n";
            Coroutine::sleep(1);
        });
        
        Coroutine::wait();
        
        echo "\n=== Concurrent HTTP Requests ===\n";
        $urls = [
            'https://api.example.com/data1',
            'https://api.example.com/data2',
            'https://api.example.com/data3'
        ];
        
        $results = self::fetchMultiple($urls);
        print_r($results);
    }
    
    private static function fetchMultiple(array $urls): array
    {
        $results = [];
        
        foreach ($urls as $url) {
            Coroutine::create(function () use ($url, &$results) {
                $host = parse_url($url, PHP_URL_HOST);
                $client = new Client($host, 443, true);
                $client->get(parse_url($url, PHP_URL_PATH));
                $results[$url] = $client->body;
                $client->close();
            });
        }
        
        Coroutine::wait();
        return $results;
    }
}
```

## 注意事项

1. **阻塞操作**：避免在协程中使用阻塞操作，使用协程版本的函数。

2. **协程数量**：不要创建过多协程，合理控制协程数量。

3. **错误处理**：妥善处理异常，避免协程崩溃。

4. **资源管理**：正确管理协程中的资源，避免泄漏。

5. **性能考虑**：协程虽然轻量，但大量协程仍会影响性能。

## 练习

1. 使用 OpenSwoole 协程实现并发 HTTP 请求。

2. 创建一个协程化的数据库连接池。

3. 实现协程化的任务队列处理。

4. 对比同步和协程代码的性能差异。

5. 实现一个基于协程的 WebSocket 服务器。
