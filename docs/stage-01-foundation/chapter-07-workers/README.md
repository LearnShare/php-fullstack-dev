# 1.7 Worker、协程、Event Loop

## 目标

- 理解 Worker 模式的工作原理。
- 掌握协程（Coroutine）的概念和使用。
- 了解 Event Loop 的工作机制。
- 熟悉 PHP 8 的 Fibers 基础能力。

## Worker 模式

### 什么是 Worker

- **Worker**：工作进程，重复处理请求的进程。
- **特点**：进程常驻内存，可以处理多个请求。

### 传统模式 vs Worker 模式

```
传统模式（PHP-FPM）：
请求 1 → 创建进程 → 处理 → 销毁进程
请求 2 → 创建进程 → 处理 → 销毁进程
请求 3 → 创建进程 → 处理 → 销毁进程

Worker 模式：
启动 → 创建 Worker 进程池
请求 1 → Worker 1 处理
请求 2 → Worker 2 处理
请求 3 → Worker 1 处理（复用）
```

### Worker 实现示例

```php
<?php
declare(strict_types=1);

// 简单的 Worker 实现
class Worker
{
    private $socket;
    private bool $running = true;
    
    public function __construct(string $address, int $port)
    {
        $this->socket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
        socket_bind($this->socket, $address, $port);
        socket_listen($this->socket);
    }
    
    public function start(): void
    {
        echo "Worker started on 0.0.0.0:8080\n";
        
        while ($this->running) {
            $client = socket_accept($this->socket);
            
            if ($client !== false) {
                $this->handleRequest($client);
            }
        }
    }
    
    private function handleRequest($client): void
    {
        $request = socket_read($client, 1024);
        
        // 处理请求
        $response = "HTTP/1.1 200 OK\r\n";
        $response .= "Content-Type: text/plain\r\n";
        $response .= "\r\n";
        $response .= "Hello from Worker";
        
        socket_write($client, $response);
        socket_close($client);
    }
    
    public function stop(): void
    {
        $this->running = false;
        socket_close($this->socket);
    }
}

// 使用
$worker = new Worker('0.0.0.0', 8080);
$worker->start();
```

## 协程（Coroutine）

### 与 Node.js 事件循环对比

**Node.js 事件循环**：
- 单线程事件循环
- 异步 I/O 操作不阻塞主线程
- 通过回调/Promise/async-await 处理异步操作

**PHP 协程**：
- 多协程并发执行
- 协程可以在 I/O 操作时暂停，让其他协程执行
- 通过协程实现类似 async-await 的效果

**示例对比**：

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

### 什么是协程

- **协程**：轻量级线程，可以在执行过程中暂停和恢复。
- **特点**：比线程更轻量，可以创建大量协程。
- **类比**：类似 Node.js 的 Promise，但更底层

### 协程 vs 线程 vs 进程

| 特性     | 进程       | 线程       | 协程       |
| :------- | :--------- | :--------- | :--------- |
| 创建成本 | 高         | 中         | 低         |
| 内存占用 | 高         | 中         | 低         |
| 切换成本 | 高         | 中         | 低         |
| 并发数量 | 少（数百） | 中（数千） | 多（数万） |

### OpenSwoole 协程示例

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

## Event Loop（事件循环）

### 什么是 Event Loop

- **Event Loop**：事件循环，持续监听和分发事件。
- **工作原理**：单线程处理多个 I/O 操作。

### Event Loop 工作流程

```
1. 初始化 Event Loop
   ↓
2. 注册事件监听器
   ↓
3. 进入循环
   ↓
4. 等待事件发生
   ↓
5. 处理事件
   ↓
6. 返回步骤 3
```

### OpenSwoole Event Loop 示例

```php
<?php
declare(strict_types=1);

use OpenSwoole\Event;

// 创建 TCP 服务器
$server = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
socket_bind($server, '0.0.0.0', 9501);
socket_listen($server);
socket_set_nonblock($server);

// 注册事件
Event::add($server, function ($server) {
    $client = socket_accept($server);
    if ($client !== false) {
        Event::add($client, function ($client) {
            $data = socket_read($client, 1024);
            if ($data !== false && $data !== '') {
                socket_write($client, "Echo: {$data}");
            } else {
                Event::del($client);
                socket_close($client);
            }
        });
    }
});

// 启动事件循环
Event::wait();
```

## PHP 8 Fibers

### 与 Node.js Promise/async-await 对比

如果你熟悉 Node.js 的 Promise 和 async-await，理解 PHP 异步处理的关键差异：

**Node.js 异步模型**：
```javascript
// Promise
fetch('https://api.example.com/users/1')
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error(error));

// async/await
async function fetchUser(id) {
    try {
        const response = await fetch(`https://api.example.com/users/${id}`);
        const data = await response.json();
        return data;
    } catch (error) {
        console.error(error);
    }
}
```

**PHP 异步模型**：
```php
// 传统同步方式（阻塞）
$response = file_get_contents('https://api.example.com/users/1');
$data = json_decode($response, true);

// 使用协程（非阻塞，类似 async/await）
use OpenSwoole\Coroutine;

Coroutine::create(function () {
    $response = Coroutine\Http\get('https://api.example.com/users/1');
    $data = json_decode($response->getBody(), true);
    return $data;
});
```

**关键差异**：

| 特性 | Node.js | PHP (传统) | PHP (协程) |
| :--- | :------ | :--------- | :---------- |
| 异步模型 | Promise/async-await | 同步阻塞 | 协程（OpenSwoole/Fibers） |
| 非阻塞 I/O | 原生支持 | 不支持 | 需要协程运行时 |
| 并发处理 | 事件循环 | 多进程 | 协程池 |
| 语法 | `async/await` | 同步语法 | 协程语法 |

### 什么是 Fibers

- **Fibers**：PHP 8.1+ 引入的轻量级协程实现。
- **特点**：原生支持，无需扩展。
- **类比**：类似 JavaScript 的 Generator，但更强大

### Fibers 基础

```php
<?php
declare(strict_types=1);

$fiber = new Fiber(function (): void {
    echo "Fiber started\n";
    
    // 暂停执行
    Fiber::suspend('First suspend');
    
    echo "Fiber resumed\n";
    
    // 再次暂停
    Fiber::suspend('Second suspend');
    
    echo "Fiber finished\n";
});

// 启动 Fiber
$value = $fiber->start();
echo "Suspended with: {$value}\n";

// 恢复执行
$value = $fiber->resume();
echo "Resumed with: {$value}\n";

// 再次恢复
$fiber->resume();
```

### Fibers 实际应用

```php
<?php
declare(strict_types=1);

class AsyncHttpClient
{
    private array $fibers = [];
    
    public function get(string $url): string
    {
        $fiber = new Fiber(function () use ($url): string {
            // 模拟异步 HTTP 请求
            $ch = curl_init($url);
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
            
            // 暂停，等待响应
            Fiber::suspend();
            
            $response = curl_exec($ch);
            curl_close($ch);
            
            return $response;
        });
        
        $this->fibers[] = $fiber;
        $fiber->start();
        
        // 返回一个占位符，实际数据稍后获取
        return "pending";
    }
    
    public function waitAll(): array
    {
        $results = [];
        
        foreach ($this->fibers as $fiber) {
            if ($fiber->isSuspended()) {
                // 模拟响应到达，恢复 Fiber
                $results[] = $fiber->resume();
            }
        }
        
        return $results;
    }
}
```

### Fibers 与生成器

```php
<?php
declare(strict_types=1);

// 使用生成器实现类似效果
function asyncTask(): Generator
{
    yield 'Task 1';
    yield 'Task 2';
    yield 'Task 3';
}

$gen = asyncTask();
foreach ($gen as $value) {
    echo $value . "\n";
}

// 使用 Fibers 实现
$fiber = new Fiber(function (): void {
    echo "Task 1\n";
    Fiber::suspend();
    echo "Task 2\n";
    Fiber::suspend();
    echo "Task 3\n";
});

$fiber->start();
$fiber->resume();
$fiber->resume();
```

## 实际应用场景

### 1. 并发 HTTP 请求

```php
<?php
declare(strict_types=1);

use OpenSwoole\Coroutine;

function fetchMultipleUrls(array $urls): array
{
    $results = [];
    
    foreach ($urls as $url) {
        Coroutine::create(function () use ($url, &$results) {
            $client = new \OpenSwoole\Coroutine\Http\Client(
                parse_url($url, PHP_URL_HOST),
                443,
                true
            );
            $client->get(parse_url($url, PHP_URL_PATH));
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

### 2. 数据库连接池

```php
<?php
declare(strict_types=1);

use OpenSwoole\Coroutine;

class ConnectionPool
{
    private array $connections = [];
    private int $maxConnections = 10;
    
    public function getConnection(): PDO
    {
        if (empty($this->connections)) {
            return $this->createConnection();
        }
        
        return array_pop($this->connections);
    }
    
    public function releaseConnection(PDO $connection): void
    {
        if (count($this->connections) < $this->maxConnections) {
            $this->connections[] = $connection;
        }
    }
    
    private function createConnection(): PDO
    {
        return new PDO($dsn, $user, $pass);
    }
}

// 在协程中使用
Coroutine::create(function () use ($pool) {
    $db = $pool->getConnection();
    try {
        // 使用数据库连接
        $stmt = $db->query('SELECT * FROM users');
        // ...
    } finally {
        $pool->releaseConnection($db);
    }
});
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
// 不要创建过多协程
// 错误示例
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

## 练习

1. 实现一个简单的 Worker 服务器，处理 HTTP 请求。

2. 使用 OpenSwoole 协程实现并发 HTTP 请求。

3. 创建一个基于 Fibers 的异步任务系统。

4. 实现一个协程化的数据库连接池。

5. 使用 Event Loop 创建一个简单的 TCP 服务器。

6. 对比传统同步代码和协程代码的性能差异。
