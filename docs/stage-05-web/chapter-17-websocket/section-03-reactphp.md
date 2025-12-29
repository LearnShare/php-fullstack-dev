# 5.17.3 ReactPHP WebSocket

## 概述

ReactPHP 是 PHP 的异步事件驱动库，可以用于实现 WebSocket 服务器。理解 ReactPHP 的使用方法、掌握 WebSocket 实现、实现事件处理和性能优化，对于构建高性能的 WebSocket 应用至关重要。本节详细介绍 ReactPHP 概述、WebSocket 实现、事件处理、性能优化等内容，帮助零基础学员掌握 ReactPHP WebSocket 的使用。

ReactPHP 提供了底层的事件循环和异步 I/O 能力，可以用于构建高性能的 WebSocket 服务器。理解 ReactPHP 的工作原理对于实现高性能 WebSocket 应用至关重要。

**主要内容**：
- ReactPHP 概述（ReactPHP 的功能、ReactPHP 的优势、使用场景）
- WebSocket 实现（WebSocket 服务器、WebSocket 客户端、消息处理、连接管理）
- 事件处理（事件循环、事件监听、事件处理、异步操作）
- 性能优化（连接池管理、消息队列、资源优化、并发处理）
- 实际应用示例和最佳实践

## 特性

- **异步非阻塞**：异步非阻塞 I/O
- **事件驱动**：基于事件驱动的编程模型
- **高性能**：高性能的并发处理
- **可扩展**：易于扩展
- **灵活**：灵活的架构

## ReactPHP 概述

### ReactPHP 的功能

1. **事件循环**：提供事件循环机制
2. **异步 I/O**：支持异步 I/O 操作
3. **网络编程**：支持网络编程
4. **并发处理**：支持并发处理

### ReactPHP 的优势

1. **高性能**：高性能的异步处理
2. **可扩展**：易于扩展
3. **灵活**：灵活的架构
4. **标准库**：提供标准库组件

### 使用场景

1. **WebSocket 服务器**：实现 WebSocket 服务器
2. **HTTP 服务器**：实现 HTTP 服务器
3. **TCP 服务器**：实现 TCP 服务器
4. **异步任务**：处理异步任务

## WebSocket 实现

### WebSocket 服务器

**示例**：
```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use React\Socket\Server;
use React\Http\Server as HttpServer;
use Ratchet\WebSocket\WsServer;
use Ratchet\Http\HttpServer as RatchetHttpServer;
use Ratchet\Server\IoServer;

$loop = React\EventLoop\Factory::create();
$socket = new Server('0.0.0.0:8080', $loop);

$wsServer = new WsServer(new MyWebSocket());
$httpServer = new RatchetHttpServer($wsServer);

$server = new IoServer($httpServer, $socket, $loop);
$server->run();
```

### WebSocket 客户端

**示例**：
```php
<?php
declare(strict_types=1);

use React\Socket\Connector;
use React\Stream\WritableResourceStream;

$loop = React\EventLoop\Factory::create();
$connector = new Connector($loop);

$connector->connect('ws://example.com:8080')->then(function ($connection) {
    $connection->write('Hello Server');
    
    $connection->on('data', function ($data) {
        echo "Received: {$data}\n";
    });
    
    $connection->on('close', function () {
        echo "Connection closed\n";
    });
});

$loop->run();
```

### 消息处理

**示例**：
```php
<?php
declare(strict_types=1);

use Ratchet\MessageComponentInterface;
use Ratchet\ConnectionInterface;

class ReactWebSocket implements MessageComponentInterface
{
    private \SplObjectStorage $clients;

    public function __construct()
    {
        $this->clients = new \SplObjectStorage();
    }

    public function onOpen(ConnectionInterface $conn): void
    {
        $this->clients->attach($conn);
    }

    public function onMessage(ConnectionInterface $from, $msg): void
    {
        // 异步处理消息
        $this->processMessageAsync($from, $msg);
    }

    private function processMessageAsync(ConnectionInterface $from, $msg): void
    {
        // 使用 ReactPHP 的异步处理
        $loop = \React\EventLoop\Factory::create();
        
        $loop->addTimer(0.1, function () use ($from, $msg) {
            $response = $this->processMessage($msg);
            $from->send($response);
        });
    }

    public function onClose(ConnectionInterface $conn): void
    {
        $this->clients->detach($conn);
    }

    public function onError(ConnectionInterface $conn, \Exception $e): void
    {
        echo "Error: {$e->getMessage()}\n";
        $conn->close();
    }
}
```

### 连接管理

**示例**：
```php
<?php
declare(strict_types=1);

class ReactConnectionManager
{
    private \SplObjectStorage $clients;
    private \React\EventLoop\LoopInterface $loop;

    public function __construct(\React\EventLoop\LoopInterface $loop)
    {
        $this->clients = new \SplObjectStorage();
        $this->loop = $loop;
        
        // 定期清理断开连接
        $this->loop->addPeriodicTimer(60, function () {
            $this->cleanupConnections();
        });
    }

    public function addConnection(ConnectionInterface $conn): void
    {
        $this->clients->attach($conn);
    }

    public function removeConnection(ConnectionInterface $conn): void
    {
        $this->clients->detach($conn);
    }

    private function cleanupConnections(): void
    {
        // 清理逻辑
    }
}
```

## 事件处理

### 事件循环

**示例**：
```php
<?php
declare(strict_types=1);

use React\EventLoop\Factory;

$loop = Factory::create();

// 定时器
$loop->addTimer(1.0, function () {
    echo "Timer executed\n";
});

// 周期性定时器
$loop->addPeriodicTimer(1.0, function () {
    echo "Periodic timer\n";
});

$loop->run();
```

### 事件监听

**示例**：
```php
<?php
declare(strict_types=1);

use React\Socket\Server;
use React\EventLoop\Factory;

$loop = Factory::create();
$socket = new Server('0.0.0.0:8080', $loop);

$socket->on('connection', function ($connection) {
    echo "New connection\n";
    
    $connection->on('data', function ($data) use ($connection) {
        echo "Received: {$data}\n";
        $connection->write("Echo: {$data}");
    });
    
    $connection->on('close', function () {
        echo "Connection closed\n";
    });
});

$loop->run();
```

### 事件处理

**示例**：
```php
<?php
declare(strict_types=1);

use React\EventLoop\Factory;

$loop = Factory::create();

// 处理多个事件
$loop->addReadStream(STDIN, function ($stream) {
    $data = fread($stream, 1024);
    echo "Input: {$data}\n";
});

$loop->addTimer(5.0, function () {
    echo "5 seconds passed\n";
});

$loop->run();
```

### 异步操作

**示例**：
```php
<?php
declare(strict_types=1);

use React\EventLoop\Factory;
use React\Promise\Promise;

$loop = Factory::create();

function asyncOperation(): Promise
{
    return new Promise(function ($resolve, $reject) use ($loop) {
        $loop->addTimer(1.0, function () use ($resolve) {
            $resolve('Operation completed');
        });
    });
}

asyncOperation()->then(function ($result) {
    echo "Result: {$result}\n";
});

$loop->run();
```

## 性能优化

### 连接池管理

**示例**：
```php
<?php
declare(strict_types=1);

class ConnectionPool
{
    private array $connections = [];
    private int $maxConnections = 1000;

    public function addConnection(ConnectionInterface $conn): bool
    {
        if (count($this->connections) >= $this->maxConnections) {
            return false;  // 连接池已满
        }
        
        $this->connections[$conn->resourceId] = $conn;
        return true;
    }

    public function removeConnection(ConnectionInterface $conn): void
    {
        unset($this->connections[$conn->resourceId]);
    }

    public function getConnectionCount(): int
    {
        return count($this->connections);
    }
}
```

### 消息队列

**示例**：
```php
<?php
declare(strict_types=1);

use React\EventLoop\Factory;

class MessageQueue
{
    private array $queue = [];
    private \React\EventLoop\LoopInterface $loop;

    public function __construct(\React\EventLoop\LoopInterface $loop)
    {
        $this->loop = $loop;
        
        // 定期处理消息队列
        $this->loop->addPeriodicTimer(0.1, function () {
            $this->processQueue();
        });
    }

    public function enqueue(array $message): void
    {
        $this->queue[] = $message;
    }

    private function processQueue(): void
    {
        if (empty($this->queue)) {
            return;
        }
        
        $message = array_shift($this->queue);
        $this->handleMessage($message);
    }
}
```

### 资源优化

**示例**：
```php
<?php
declare(strict_types=1);

class ResourceManager
{
    private array $resources = [];

    public function registerResource($resource, callable $cleanup): void
    {
        $id = spl_object_hash($resource);
        $this->resources[$id] = [
            'resource' => $resource,
            'cleanup' => $cleanup,
        ];
    }

    public function cleanup(): void
    {
        foreach ($this->resources as $id => $data) {
            $data['cleanup']($data['resource']);
        }
        $this->resources = [];
    }
}
```

### 并发处理

**示例**：
```php
<?php
declare(strict_types=1);

use React\EventLoop\Factory;

$loop = Factory::create();

// 并发处理多个任务
$promises = [];
for ($i = 0; $i < 10; $i++) {
    $promises[] = asyncTask($i);
}

\React\Promise\all($promises)->then(function ($results) {
    foreach ($results as $result) {
        echo "Task completed: {$result}\n";
    }
});

$loop->run();
```

## 使用场景

### 实时通信应用

- 实时聊天
- 实时协作
- 实时通知

### 高性能服务器

- 高并发 WebSocket 服务器
- 实时数据推送
- 在线游戏服务器

### 异步任务处理

- 异步任务队列
- 后台任务处理
- 定时任务

### 网络编程

- TCP 服务器
- UDP 服务器
- HTTP 服务器

## 注意事项

### 事件循环

- **单线程**：事件循环是单线程的
- **非阻塞**：避免阻塞操作
- **异步操作**：使用异步操作

### 连接管理

- **连接数限制**：限制连接数
- **连接清理**：及时清理断开连接
- **资源管理**：管理连接资源

### 性能优化

- **消息队列**：使用消息队列
- **连接池**：使用连接池
- **资源优化**：优化资源使用

### 错误处理

- **异常捕获**：捕获所有异常
- **错误恢复**：实现错误恢复
- **错误日志**：记录错误日志

## 常见问题

### ReactPHP 和 Ratchet 的关系？

Ratchet 基于 ReactPHP 构建，提供了更高层的 WebSocket API。

### 如何实现 WebSocket 服务器？

使用 ReactPHP 的事件循环和 Ratchet 的 WebSocket 组件。

### 如何处理异步操作？

使用 Promise 和事件循环处理异步操作。

### 如何优化性能？

使用连接池、消息队列、资源优化等方法。

## 最佳实践

### 使用事件循环

- 理解事件循环机制
- 避免阻塞操作
- 使用异步操作

### 实现连接管理

- 使用连接池
- 及时清理连接
- 限制连接数

### 优化性能

- 使用消息队列
- 优化资源使用
- 实现并发处理

### 处理错误

- 捕获所有异常
- 实现错误恢复
- 记录错误日志

## 相关章节

- **[5.17.1 WebSocket 概述](section-01-overview.md)**：了解 WebSocket 概述的详细内容
- **[5.17.2 Ratchet](section-02-ratchet.md)**：了解 Ratchet 的详细内容
- **[5.17.4 WebSocket 最佳实践](section-04-best-practices.md)**：了解 WebSocket 最佳实践的详细内容

## 练习任务

1. **安装和配置 ReactPHP**
   - Composer 安装
   - 基本配置
   - 事件循环使用

2. **实现基本 WebSocket 服务器**
   - WebSocket 服务器实现
   - 消息处理
   - 连接管理

3. **实现事件处理**
   - 事件循环
   - 事件监听
   - 异步操作

4. **实现性能优化**
   - 连接池管理
   - 消息队列
   - 资源优化

5. **实现完整的 ReactPHP WebSocket 应用**
   - 高性能服务器
   - 异步处理
   - 错误处理和优化
