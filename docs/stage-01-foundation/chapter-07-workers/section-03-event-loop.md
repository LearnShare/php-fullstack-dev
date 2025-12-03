# 1.7.3 Event Loop（事件循环）

## 概述

Event Loop（事件循环）是异步编程的核心机制。理解事件循环对于使用协程和异步 I/O 至关重要。

## 什么是 Event Loop

### 基本概念

- **Event Loop**：事件循环，持续监听和分发事件
- **工作原理**：单线程处理多个 I/O 操作
- **类比**：类似 Node.js 的事件循环

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

## 与 Node.js 对比

### Node.js 事件循环

```javascript
// Node.js 事件循环
const http = require('http');

const server = http.createServer((req, res) => {
    res.end('Hello from Node.js');
});

server.listen(3000, () => {
    console.log('Server running on port 3000');
});

// 事件循环自动处理所有 I/O 操作
```

### PHP 事件循环

```php
<?php
// PHP 事件循环（使用 OpenSwoole）
use OpenSwoole\Event;

$server = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
socket_bind($server, '0.0.0.0', 3000);
socket_listen($server);
socket_set_nonblock($server);

Event::add($server, function ($server) {
    $client = socket_accept($server);
    if ($client !== false) {
        Event::add($client, function ($client) {
            $data = socket_read($client, 1024);
            socket_write($client, "Echo: {$data}");
            Event::del($client);
            socket_close($client);
        });
    }
});

Event::wait(); // 启动事件循环
```

## OpenSwoole Event Loop

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
        socket_set_nonblock($client);
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

### HTTP 服务器事件循环

```php
<?php
declare(strict_types=1);

use OpenSwoole\Http\Server;
use OpenSwoole\Http\Request;
use OpenSwoole\Http\Response;

$server = new Server("0.0.0.0", 9501);

// 注册事件
$server->on("start", function (Server $server) {
    echo "Server started\n";
});

$server->on("request", function (Request $request, Response $response) {
    $response->header("Content-Type", "text/plain");
    $response->end("Hello from Event Loop\n");
});

// 启动事件循环
$server->start();
```

## 事件类型

### 网络事件

```php
<?php
declare(strict_types=1);

use OpenSwoole\Event;

// 可读事件
Event::add($socket, function ($socket) {
    $data = socket_read($socket, 1024);
    // 处理数据
}, Event::READ);

// 可写事件
Event::add($socket, function ($socket) {
    socket_write($socket, "Data");
}, Event::WRITE);
```

### 定时器事件

```php
<?php
declare(strict_types=1);

use OpenSwoole\Timer;

// 一次性定时器
Timer::after(1000, function () {
    echo "Timer fired after 1 second\n";
});

// 周期性定时器
Timer::tick(1000, function () {
    echo "Timer fired every 1 second\n";
});

// 清除定时器
$timerId = Timer::tick(1000, function () {
    echo "This will be cleared\n";
});
Timer::clear($timerId);
```

## 完整示例

```php
<?php
declare(strict_types=1);

use OpenSwoole\Event;
use OpenSwoole\Timer;

class EventLoopExample
{
    public static function demonstrate(): void
    {
        echo "=== TCP Server with Event Loop ===\n";
        
        $server = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
        socket_bind($server, '0.0.0.0', 9501);
        socket_listen($server);
        socket_set_nonblock($server);
        
        Event::add($server, function ($server) {
            $client = socket_accept($server);
            if ($client !== false) {
                socket_set_nonblock($client);
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
        
        // 定时器
        Timer::tick(5000, function () {
            echo "Timer: " . date('Y-m-d H:i:s') . "\n";
        });
        
        echo "Event loop started. Press Ctrl+C to stop.\n";
        Event::wait();
    }
}
```

## 注意事项

1. **非阻塞 I/O**：事件循环需要非阻塞 I/O，使用 `socket_set_nonblock()`。

2. **事件注册**：正确注册和注销事件，避免内存泄漏。

3. **错误处理**：妥善处理事件处理中的异常。

4. **性能考虑**：事件循环是单线程的，CPU 密集型任务会影响性能。

5. **资源管理**：正确管理事件循环中的资源。

## 练习

1. 使用 Event Loop 创建一个简单的 TCP 服务器。

2. 实现定时器功能，定期执行任务。

3. 创建一个基于事件循环的 HTTP 服务器。

4. 实现事件驱动的任务队列。

5. 对比事件循环和传统阻塞 I/O 的性能差异。
