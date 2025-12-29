# 5.17.2 Ratchet

## 概述

Ratchet 是 PHP 的 WebSocket 服务器库，基于 ReactPHP 构建。理解 Ratchet 的使用方法、掌握服务器实现、实现消息处理和连接管理，对于构建实时 WebSocket 应用至关重要。本节详细介绍 Ratchet 概述、安装配置、服务器实现、消息处理、连接管理等内容，帮助零基础学员掌握 Ratchet 的使用。

Ratchet 是 PHP 生态中最流行的 WebSocket 服务器库之一，提供了简洁的 API 和强大的功能。掌握 Ratchet 的使用可以快速构建 WebSocket 应用。

**主要内容**：
- Ratchet 概述（Ratchet 的功能、Ratchet 的优势、使用场景）
- 安装配置（Composer 安装、基本配置、服务器创建）
- 服务器实现（服务器类、消息组件、连接管理、事件处理）
- 消息处理（消息接收、消息发送、消息广播、消息格式）
- 连接管理（连接建立、连接关闭、连接存储、连接查找）
- 实际应用示例和最佳实践

## 特性

- **简洁 API**：提供简洁的 API
- **功能完整**：功能完整且强大
- **易于使用**：易于使用和扩展
- **高性能**：基于 ReactPHP，性能优秀
- **文档完善**：文档完善

## Ratchet 概述

### Ratchet 的功能

1. **WebSocket 服务器**：实现 WebSocket 服务器
2. **消息处理**：处理 WebSocket 消息
3. **连接管理**：管理 WebSocket 连接
4. **事件处理**：处理连接事件

### Ratchet 的优势

1. **易于使用**：简洁的 API
2. **功能完整**：功能完整
3. **高性能**：基于 ReactPHP
4. **可扩展**：易于扩展
5. **文档完善**：文档完善

### 使用场景

1. **实时聊天**：实时聊天应用
2. **实时通知**：实时通知系统
3. **在线游戏**：在线游戏
4. **实时数据推送**：实时数据推送

## 安装配置

### Composer 安装

**安装命令**：
```bash
composer require cboden/ratchet
```

### 基本配置

**示例**：
```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Ratchet\MessageComponentInterface;
use Ratchet\ConnectionInterface;
use Ratchet\Server\IoServer;
use Ratchet\Http\HttpServer;
use Ratchet\WebSocket\WsServer;

$server = IoServer::factory(
    new HttpServer(
        new WsServer(
            new MyWebSocket()
        )
    ),
    8080
);

$server->run();
```

### 服务器创建

**示例**：
```php
<?php
declare(strict_types=1);

use Ratchet\Server\IoServer;
use Ratchet\Http\HttpServer;
use Ratchet\WebSocket\WsServer;

function createWebSocketServer(MessageComponentInterface $app, int $port = 8080): IoServer
{
    return IoServer::factory(
        new HttpServer(
            new WsServer($app)
        ),
        $port
    );
}
```

## 服务器实现

### 服务器类

**示例**：
```php
<?php
declare(strict_types=1);

use Ratchet\MessageComponentInterface;
use Ratchet\ConnectionInterface;

class MyWebSocket implements MessageComponentInterface
{
    protected \SplObjectStorage $clients;

    public function __construct()
    {
        $this->clients = new \SplObjectStorage();
    }

    public function onOpen(ConnectionInterface $conn): void
    {
        $this->clients->attach($conn);
        echo "New connection: {$conn->resourceId}\n";
    }

    public function onMessage(ConnectionInterface $from, $msg): void
    {
        foreach ($this->clients as $client) {
            if ($from !== $client) {
                $client->send($msg);
            }
        }
    }

    public function onClose(ConnectionInterface $conn): void
    {
        $this->clients->detach($conn);
        echo "Connection closed: {$conn->resourceId}\n";
    }

    public function onError(ConnectionInterface $conn, \Exception $e): void
    {
        echo "Error: {$e->getMessage()}\n";
        $conn->close();
    }
}
```

### 消息组件

**示例**：
```php
<?php
declare(strict_types=1);

use Ratchet\MessageComponentInterface;
use Ratchet\ConnectionInterface;

class ChatComponent implements MessageComponentInterface
{
    protected \SplObjectStorage $clients;

    public function __construct()
    {
        $this->clients = new \SplObjectStorage();
    }

    public function onOpen(ConnectionInterface $conn): void
    {
        $this->clients->attach($conn);
        $conn->send(json_encode(['type' => 'welcome', 'message' => '欢迎加入聊天室']));
    }

    public function onMessage(ConnectionInterface $from, $msg): void
    {
        $data = json_decode($msg, true);
        
        if ($data && isset($data['message'])) {
            $response = [
                'type' => 'message',
                'user' => $data['user'] ?? 'Anonymous',
                'message' => $data['message'],
                'timestamp' => time(),
            ];
            
            foreach ($this->clients as $client) {
                $client->send(json_encode($response));
            }
        }
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

class ConnectionManager
{
    protected \SplObjectStorage $clients;
    protected array $userConnections = [];

    public function addConnection(ConnectionInterface $conn, string $userId): void
    {
        $this->clients->attach($conn);
        $this->userConnections[$userId] = $conn;
        $conn->userId = $userId;
    }

    public function removeConnection(ConnectionInterface $conn): void
    {
        if (isset($conn->userId)) {
            unset($this->userConnections[$conn->userId]);
        }
        $this->clients->detach($conn);
    }

    public function getConnectionByUserId(string $userId): ?ConnectionInterface
    {
        return $this->userConnections[$userId] ?? null;
    }

    public function broadcast($message, ?ConnectionInterface $exclude = null): void
    {
        foreach ($this->clients as $client) {
            if ($exclude === null || $client !== $exclude) {
                $client->send($message);
            }
        }
    }
}
```

### 事件处理

**示例**：
```php
<?php
declare(strict_types=1);

class EventHandler implements MessageComponentInterface
{
    public function onOpen(ConnectionInterface $conn): void
    {
        // 连接建立事件
        $this->handleConnect($conn);
    }

    public function onMessage(ConnectionInterface $from, $msg): void
    {
        // 消息接收事件
        $this->handleMessage($from, $msg);
    }

    public function onClose(ConnectionInterface $conn): void
    {
        // 连接关闭事件
        $this->handleDisconnect($conn);
    }

    public function onError(ConnectionInterface $conn, \Exception $e): void
    {
        // 错误事件
        $this->handleError($conn, $e);
    }
}
```

## 消息处理

### 消息接收

**示例**：
```php
<?php
declare(strict_types=1);

public function onMessage(ConnectionInterface $from, $msg): void
{
    // 解析消息
    $data = json_decode($msg, true);
    
    if ($data === null) {
        $from->send(json_encode(['error' => 'Invalid message format']));
        return;
    }
    
    // 处理不同类型的消息
    match ($data['type'] ?? '') {
        'chat' => $this->handleChatMessage($from, $data),
        'ping' => $this->handlePing($from),
        'join' => $this->handleJoin($from, $data),
        default => $this->handleUnknownMessage($from, $data),
    };
}
```

### 消息发送

**示例**：
```php
<?php
declare(strict_types=1);

public function sendToClient(ConnectionInterface $conn, array $data): void
{
    $message = json_encode($data);
    $conn->send($message);
}

public function sendToUser(string $userId, array $data): void
{
    $conn = $this->connectionManager->getConnectionByUserId($userId);
    if ($conn !== null) {
        $this->sendToClient($conn, $data);
    }
}
```

### 消息广播

**示例**：
```php
<?php
declare(strict_types=1);

public function broadcast(array $data, ?ConnectionInterface $exclude = null): void
{
    $message = json_encode($data);
    
    foreach ($this->clients as $client) {
        if ($exclude === null || $client !== $exclude) {
            $client->send($message);
        }
    }
}

public function broadcastToRoom(string $roomId, array $data): void
{
    $message = json_encode($data);
    
    foreach ($this->clients as $client) {
        if (isset($client->roomId) && $client->roomId === $roomId) {
            $client->send($message);
        }
    }
}
```

### 消息格式

**示例**：
```php
<?php
declare(strict_types=1);

class MessageFormat
{
    public static function chat(string $user, string $message): array
    {
        return [
            'type' => 'chat',
            'user' => $user,
            'message' => $message,
            'timestamp' => time(),
        ];
    }

    public static function notification(string $title, string $content): array
    {
        return [
            'type' => 'notification',
            'title' => $title,
            'content' => $content,
            'timestamp' => time(),
        ];
    }

    public static function error(string $message): array
    {
        return [
            'type' => 'error',
            'message' => $message,
            'timestamp' => time(),
        ];
    }
}
```

## 连接管理

### 连接建立

**示例**：
```php
<?php
declare(strict_types=1);

public function onOpen(ConnectionInterface $conn): void
{
    // 存储连接信息
    $conn->id = uniqid();
    $conn->connectedAt = time();
    
    $this->clients->attach($conn);
    
    // 发送欢迎消息
    $conn->send(json_encode([
        'type' => 'welcome',
        'message' => '连接成功',
        'connection_id' => $conn->id,
    ]));
    
    echo "New connection: {$conn->id}\n";
}
```

### 连接关闭

**示例**：
```php
<?php
declare(strict_types=1);

public function onClose(ConnectionInterface $conn): void
{
    // 清理连接信息
    if (isset($conn->userId)) {
        $this->notifyUserDisconnected($conn->userId);
    }
    
    $this->clients->detach($conn);
    
    echo "Connection closed: {$conn->id}\n";
}
```

### 连接存储

**示例**：
```php
<?php
declare(strict_types=1);

class ConnectionStorage
{
    private \SplObjectStorage $clients;
    private array $userConnections = [];
    private array $roomConnections = [];

    public function addConnection(ConnectionInterface $conn, string $userId): void
    {
        $this->clients->attach($conn);
        $this->userConnections[$userId] = $conn;
        $conn->userId = $userId;
    }

    public function addToRoom(ConnectionInterface $conn, string $roomId): void
    {
        if (!isset($this->roomConnections[$roomId])) {
            $this->roomConnections[$roomId] = [];
        }
        $this->roomConnections[$roomId][] = $conn;
        $conn->roomId = $roomId;
    }

    public function getRoomConnections(string $roomId): array
    {
        return $this->roomConnections[$roomId] ?? [];
    }
}
```

### 连接查找

**示例**：
```php
<?php
declare(strict_types=1);

public function findConnectionByUserId(string $userId): ?ConnectionInterface
{
    foreach ($this->clients as $client) {
        if (isset($client->userId) && $client->userId === $userId) {
            return $client;
        }
    }
    return null;
}

public function findConnectionsByRoom(string $roomId): array
{
    $connections = [];
    foreach ($this->clients as $client) {
        if (isset($client->roomId) && $client->roomId === $roomId) {
            $connections[] = $client;
        }
    }
    return $connections;
}
```

## 使用场景

### 实时聊天

- 聊天室
- 私聊
- 群聊

### 实时通知

- 系统通知
- 用户通知
- 推送通知

### 在线游戏

- 多人游戏
- 实时对战
- 游戏状态同步

### 实时数据推送

- 股票价格
- 体育比分
- 监控数据

## 注意事项

### 连接管理

- **连接存储**：正确存储连接信息
- **连接清理**：及时清理断开连接
- **连接查找**：高效查找连接

### 消息处理

- **消息验证**：验证消息格式
- **消息队列**：处理消息队列
- **消息大小**：限制消息大小

### 错误处理

- **连接错误**：处理连接错误
- **消息错误**：处理消息错误
- **异常捕获**：捕获所有异常

### 性能考虑

- **连接数限制**：限制连接数
- **消息大小**：限制消息大小
- **资源管理**：管理连接资源

## 常见问题

### 如何安装 Ratchet？

使用 Composer 安装：
```bash
composer require cboden/ratchet
```

### 如何创建 WebSocket 服务器？

实现 MessageComponentInterface 接口，使用 IoServer::factory() 创建服务器。

### 如何处理消息？

在 onMessage 方法中处理接收到的消息。

### 如何管理连接？

使用 SplObjectStorage 存储连接，实现连接查找和管理。

## 最佳实践

### 实现连接管理

- 使用连接管理器
- 存储连接信息
- 实现连接查找

### 实现消息处理

- 定义消息格式
- 验证消息内容
- 处理不同类型消息

### 实现错误处理

- 捕获所有异常
- 处理连接错误
- 记录错误日志

### 优化性能

- 限制连接数
- 限制消息大小
- 优化消息处理

## 相关章节

- **[5.17.1 WebSocket 概述](section-01-overview.md)**：了解 WebSocket 概述的详细内容
- **[5.17.3 ReactPHP WebSocket](section-03-reactphp.md)**：了解 ReactPHP WebSocket 的详细内容
- **[5.17.4 WebSocket 最佳实践](section-04-best-practices.md)**：了解 WebSocket 最佳实践的详细内容

## 练习任务

1. **安装和配置 Ratchet**
   - Composer 安装
   - 基本配置
   - 服务器创建

2. **实现基本 WebSocket 服务器**
   - 实现 MessageComponentInterface
   - 处理连接事件
   - 处理消息

3. **实现消息处理**
   - 消息接收和发送
   - 消息广播
   - 消息格式

4. **实现连接管理**
   - 连接存储
   - 连接查找
   - 连接清理

5. **实现完整的 WebSocket 应用**
   - 实时聊天
   - 连接管理
   - 错误处理
