# 1.6.3 OpenSwoole

## 概述

OpenSwoole 是一个高性能的 PHP 异步网络框架。它提供了事件驱动、协程、异步 I/O 等功能，可以实现极高的性能。

## 特点

- **语言**：C 扩展
- **架构**：事件驱动 + 协程
- **优势**：极高性能、支持 WebSocket、异步 I/O
- **适用场景**：极高性能要求、需要 WebSocket、实时应用

## 安装

### 方式一：PECL

```bash
# 安装 OpenSwoole
pecl install openswoole

# 在 php.ini 中添加
# extension=openswoole.so
```

### 方式二：包管理器

```bash
# Ubuntu/Debian
sudo apt install php-openswoole

# macOS
brew install openswoole
```

### 方式三：Docker

```dockerfile
FROM php:8.2-fpm

# 安装依赖
RUN apt-get update && apt-get install -y \
    libcurl4-openssl-dev \
    libssl-dev

# 安装 OpenSwoole
RUN pecl install openswoole && \
    docker-php-ext-enable openswoole
```

## 基础使用

### HTTP 服务器

```php
<?php
declare(strict_types=1);

use OpenSwoole\Http\Server;
use OpenSwoole\Http\Request;
use OpenSwoole\Http\Response;

$server = new Server("0.0.0.0", 9501);

$server->on("request", function (Request $request, Response $response) {
    $response->header("Content-Type", "text/plain");
    $response->end("Hello World\n");
});

$server->start();
```

### WebSocket 服务器

```php
<?php
declare(strict_types=1);

use OpenSwoole\WebSocket\Server;
use OpenSwoole\WebSocket\Frame;

$server = new Server("0.0.0.0", 9501);

$server->on("open", function (Server $server, $request) {
    echo "Connection open: {$request->fd}\n";
});

$server->on("message", function (Server $server, Frame $frame) {
    echo "Received: {$frame->data}\n";
    $server->push($frame->fd, "Server: {$frame->data}");
});

$server->on("close", function (Server $server, $fd) {
    echo "Connection close: {$fd}\n";
});

$server->start();
```

## 协程

### 基本协程

```php
<?php
declare(strict_types=1);

use OpenSwoole\Coroutine;

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

// 两个协程并发执行，总时间约 1 秒
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

// 并发请求
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

## 事件循环

### 基本事件循环

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

## 完整示例

### HTTP 服务器

```php
<?php
declare(strict_types=1);

use OpenSwoole\Http\Server;
use OpenSwoole\Http\Request;
use OpenSwoole\Http\Response;

$server = new Server("0.0.0.0", 9501);

$server->on("start", function (Server $server) {
    echo "OpenSwoole HTTP server is started at http://0.0.0.0:9501\n";
});

$server->on("request", function (Request $request, Response $response) {
    $path = $request->server['request_uri'];
    $method = $request->server['request_method'];
    
    // 路由处理
    match($path) {
        '/' => $response->end('Hello from OpenSwoole'),
        '/api/users' => handleUsers($request, $response),
        default => $response->status(404)->end('Not Found')
    };
});

function handleUsers(Request $request, Response $response): void
{
    $users = [
        ['id' => 1, 'name' => 'Alice'],
        ['id' => 2, 'name' => 'Bob']
    ];
    
    $response->header('Content-Type', 'application/json');
    $response->end(json_encode($users));
}

$server->start();
```

### WebSocket 服务器

```php
<?php
declare(strict_types=1);

use OpenSwoole\WebSocket\Server;
use OpenSwoole\WebSocket\Frame;

$server = new Server("0.0.0.0", 9501);

$server->on("open", function (Server $server, $request) {
    echo "Connection open: {$request->fd}\n";
    $server->push($request->fd, "Welcome to WebSocket server");
});

$server->on("message", function (Server $server, Frame $frame) {
    echo "Received from {$frame->fd}: {$frame->data}\n";
    
    // 广播消息
    foreach ($server->connections as $fd) {
        if ($fd !== $frame->fd) {
            $server->push($fd, "User {$frame->fd}: {$frame->data}");
        }
    }
});

$server->on("close", function (Server $server, $fd) {
    echo "Connection close: {$fd}\n";
});

$server->start();
```

## 注意事项

1. **阻塞操作**：避免在协程中使用阻塞操作，使用协程版本的函数。

2. **内存管理**：注意内存泄漏，定期清理不需要的数据。

3. **错误处理**：妥善处理异常，避免服务器崩溃。

4. **性能监控**：监控协程数量和内存使用。

5. **部署复杂度**：OpenSwoole 的部署和维护比 PHP-FPM 更复杂。

## 练习

1. 安装 OpenSwoole，创建一个简单的 HTTP 服务器。

2. 使用 OpenSwoole 创建一个 WebSocket 服务器。

3. 使用协程实现并发 HTTP 请求。

4. 实现协程化的数据库连接池。

5. 对比 OpenSwoole 和 PHP-FPM 的性能差异。
