# 4.3.1 Socket 编程基础

## 概述

Socket 是网络编程的基础，提供了进程间通信的机制。通过 Socket，不同主机上的进程可以进行数据交换，实现网络通信。理解 Socket 的概念、TCP 和 UDP 的区别，以及如何进行 Socket 编程，对于进行网络应用开发至关重要。

PHP 提供了 Socket 扩展，支持 TCP 和 UDP Socket 编程。虽然在实际应用中，我们更多使用高级的网络库（如 cURL、Guzzle），但理解 Socket 编程基础有助于深入理解网络通信的原理。

**主要内容**：
- Socket 的概念和作用
- TCP Socket 编程
- UDP Socket 编程
- 服务器端编程
- 客户端编程
- 错误处理和资源管理
- 实际应用场景和最佳实践

## 特性

- **底层网络通信**：提供底层的网络通信能力
- **TCP 和 UDP 支持**：支持可靠的 TCP 和快速的 UDP
- **服务器和客户端**：支持服务器端和客户端编程
- **跨平台**：在 Unix/Linux 和 Windows 上都有支持
- **灵活控制**：可以精确控制网络通信的各个环节

## 语法/定义

### socket_create() 函数

**语法**：`socket_create(int $domain, int $type, int $protocol): Socket|false`

**参数**：
- `$domain`：协议域（`AF_INET` 表示 IPv4，`AF_INET6` 表示 IPv6）
- `$type`：Socket 类型（`SOCK_STREAM` 表示 TCP，`SOCK_DGRAM` 表示 UDP）
- `$protocol`：协议（`SOL_TCP` 表示 TCP，`SOL_UDP` 表示 UDP）

**返回值**：成功返回 Socket 资源，失败返回 `false`。

### socket_bind() 函数

**语法**：`socket_bind(Socket $socket, string $address, int $port = 0): bool`

**参数**：
- `$socket`：Socket 资源
- `$address`：IP 地址
- `$port`：端口号

**返回值**：成功返回 `true`，失败返回 `false`。

### socket_listen() 函数

**语法**：`socket_listen(Socket $socket, int $backlog = 0): bool`

**参数**：
- `$socket`：Socket 资源
- `$backlog`：等待连接队列的最大长度

**返回值**：成功返回 `true`，失败返回 `false`。

### socket_accept() 函数

**语法**：`socket_accept(Socket $socket): Socket|false`

**参数**：
- `$socket`：监听中的 Socket 资源

**返回值**：成功返回新的 Socket 资源（用于与客户端通信），失败返回 `false`。

### socket_connect() 函数

**语法**：`socket_connect(Socket $socket, string $address, int $port = 0): bool`

**参数**：
- `$socket`：Socket 资源
- `$address`：服务器 IP 地址
- `$port`：服务器端口号

**返回值**：成功返回 `true`，失败返回 `false`。

### socket_read() 函数

**语法**：`socket_read(Socket $socket, int $length, int $mode = PHP_BINARY_READ): string|false`

**参数**：
- `$socket`：Socket 资源
- `$length`：读取的最大字节数
- `$mode`：读取模式（`PHP_BINARY_READ` 或 `PHP_NORMAL_READ`）

**返回值**：成功返回读取的数据，失败返回 `false`。

### socket_write() 函数

**语法**：`socket_write(Socket $socket, string $data, ?int $length = null): int|false`

**参数**：
- `$socket`：Socket 资源
- `$data`：要写入的数据
- `$length`：可选，写入的字节数

**返回值**：成功返回写入的字节数，失败返回 `false`。

## 基本用法

### 示例 1：TCP 服务器端编程

```php
<?php
declare(strict_types=1);

// 创建 TCP Socket
$socket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
if ($socket === false) {
    throw new RuntimeException('Cannot create socket: ' . socket_strerror(socket_last_error()));
}

// 设置 Socket 选项（允许地址重用）
socket_set_option($socket, SOL_SOCKET, SO_REUSEADDR, 1);

// 绑定地址和端口
if (!socket_bind($socket, '127.0.0.1', 8080)) {
    $error = socket_strerror(socket_last_error($socket));
    socket_close($socket);
    throw new RuntimeException("Cannot bind socket: {$error}");
}

// 开始监听
if (!socket_listen($socket, 5)) {
    $error = socket_strerror(socket_last_error($socket));
    socket_close($socket);
    throw new RuntimeException("Cannot listen on socket: {$error}");
}

echo "Server listening on 127.0.0.1:8080\n";

// 接受客户端连接
$client = socket_accept($socket);
if ($client === false) {
    $error = socket_strerror(socket_last_error($socket));
    socket_close($socket);
    throw new RuntimeException("Cannot accept connection: {$error}");
}

// 读取客户端数据
$data = socket_read($client, 1024);
if ($data !== false) {
    echo "Received: {$data}\n";
    
    // 发送响应
    $response = "Hello, Client! You said: {$data}";
    socket_write($client, $response);
}

// 关闭连接
socket_close($client);
socket_close($socket);
```

**说明**：
- 创建 TCP Socket 服务器
- 绑定到本地地址和端口
- 监听连接请求
- 接受客户端连接并处理数据

### 示例 2：TCP 客户端编程

```php
<?php
declare(strict_types=1);

// 创建 TCP Socket
$socket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
if ($socket === false) {
    throw new RuntimeException('Cannot create socket: ' . socket_strerror(socket_last_error()));
}

// 连接到服务器
if (!socket_connect($socket, '127.0.0.1', 8080)) {
    $error = socket_strerror(socket_last_error($socket));
    socket_close($socket);
    throw new RuntimeException("Cannot connect to server: {$error}");
}

// 发送数据
$message = "Hello, Server!";
socket_write($socket, $message);

// 读取响应
$response = socket_read($socket, 1024);
if ($response !== false) {
    echo "Server response: {$response}\n";
}

// 关闭连接
socket_close($socket);
```

**说明**：
- 创建 TCP Socket 客户端
- 连接到服务器
- 发送数据并接收响应

### 示例 3：UDP 服务器端编程

```php
<?php
declare(strict_types=1);

// 创建 UDP Socket
$socket = socket_create(AF_INET, SOCK_DGRAM, SOL_UDP);
if ($socket === false) {
    throw new RuntimeException('Cannot create socket: ' . socket_strerror(socket_last_error()));
}

// 绑定地址和端口
if (!socket_bind($socket, '127.0.0.1', 8080)) {
    $error = socket_strerror(socket_last_error($socket));
    socket_close($socket);
    throw new RuntimeException("Cannot bind socket: {$error}");
}

echo "UDP Server listening on 127.0.0.1:8080\n";

// 接收数据（UDP 不需要 listen 和 accept）
$from = '';
$port = 0;
$data = socket_recvfrom($socket, $buffer, 1024, 0, $from, $port);
if ($data !== false) {
    echo "Received from {$from}:{$port}: {$buffer}\n";
    
    // 发送响应
    $response = "Echo: {$buffer}";
    socket_sendto($socket, $response, strlen($response), 0, $from, $port);
}

// 关闭 Socket
socket_close($socket);
```

**说明**：
- UDP 是无连接的，不需要 `listen` 和 `accept`
- 使用 `socket_recvfrom()` 接收数据
- 使用 `socket_sendto()` 发送数据

### 示例 4：UDP 客户端编程

```php
<?php
declare(strict_types=1);

// 创建 UDP Socket
$socket = socket_create(AF_INET, SOCK_DGRAM, SOL_UDP);
if ($socket === false) {
    throw new RuntimeException('Cannot create socket: ' . socket_strerror(socket_last_error()));
}

// 发送数据（UDP 不需要 connect）
$message = "Hello, UDP Server!";
socket_sendto($socket, $message, strlen($message), 0, '127.0.0.1', 8080);

// 接收响应
$from = '';
$port = 0;
$response = '';
socket_recvfrom($socket, $response, 1024, 0, $from, $port);
echo "Response from {$from}:{$port}: {$response}\n";

// 关闭 Socket
socket_close($socket);
```

**说明**：
- UDP 客户端不需要 `connect`
- 直接使用 `socket_sendto()` 发送数据
- 使用 `socket_recvfrom()` 接收响应

### 示例 5：带错误处理的 Socket 服务器

```php
<?php
declare(strict_types=1);

function createTCPServer(string $host, int $port): Socket
{
    $socket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
    if ($socket === false) {
        throw new RuntimeException('Cannot create socket: ' . socket_strerror(socket_last_error()));
    }
    
    // 设置选项
    socket_set_option($socket, SOL_SOCKET, SO_REUSEADDR, 1);
    
    // 绑定
    if (!socket_bind($socket, $host, $port)) {
        $error = socket_strerror(socket_last_error($socket));
        socket_close($socket);
        throw new RuntimeException("Cannot bind to {$host}:{$port}: {$error}");
    }
    
    // 监听
    if (!socket_listen($socket, 5)) {
        $error = socket_strerror(socket_last_error($socket));
        socket_close($socket);
        throw new RuntimeException("Cannot listen: {$error}");
    }
    
    return $socket;
}

function handleClient(Socket $client): void
{
    try {
        // 设置超时
        socket_set_option($client, SOL_SOCKET, SO_RCVTIMEO, ['sec' => 5, 'usec' => 0]);
        
        // 读取数据
        $data = socket_read($client, 1024);
        if ($data === false) {
            echo "Failed to read from client\n";
            return;
        }
        
        echo "Received: {$data}\n";
        
        // 处理数据并发送响应
        $response = "Echo: " . trim($data) . "\n";
        socket_write($client, $response);
    } finally {
        socket_close($client);
    }
}

// 使用
try {
    $server = createTCPServer('127.0.0.1', 8080);
    echo "Server listening on 127.0.0.1:8080\n";
    
    while (true) {
        $client = socket_accept($server);
        if ($client === false) {
            continue;
        }
        
        handleClient($client);
    }
} catch (Exception $e) {
    echo "Error: " . $e->getMessage() . "\n";
} finally {
    if (isset($server)) {
        socket_close($server);
    }
}
```

**说明**：
- 封装了 Socket 创建和错误处理
- 设置超时选项
- 使用 try-finally 确保资源释放

## 使用场景

### 场景 1：简单的 TCP 服务器

创建一个简单的 TCP 服务器，处理客户端连接。

**示例**：见"示例 1：TCP 服务器端编程"

### 场景 2：网络协议实现

实现自定义的网络协议。

**示例**：

```php
<?php
declare(strict_types=1);

function handleProtocolMessage(string $data): string
{
    // 解析协议消息
    $parts = explode('|', $data);
    if (count($parts) < 2) {
        return "ERROR: Invalid message format\n";
    }
    
    $command = $parts[0];
    $payload = $parts[1];
    
    // 处理命令
    return match ($command) {
        'ECHO' => "ECHO: {$payload}\n",
        'TIME' => "TIME: " . date('Y-m-d H:i:s') . "\n",
        'UPPER' => "UPPER: " . strtoupper($payload) . "\n",
        default => "ERROR: Unknown command\n",
    };
}

// 在服务器中使用
$client = socket_accept($server);
$data = socket_read($client, 1024);
$response = handleProtocolMessage($data);
socket_write($client, $response);
```

## 注意事项

### Socket 资源管理

及时关闭 Socket 资源，避免资源泄漏。

**示例**：

```php
<?php
declare(strict_types=1);

$socket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
if ($socket === false) {
    throw new RuntimeException('Cannot create socket');
}

try {
    // 使用 Socket
    // ...
} finally {
    // 确保关闭
    socket_close($socket);
}
```

### 错误处理

检查每个 Socket 操作的返回值，处理错误。

**示例**：

```php
<?php
declare(strict_types=1);

$socket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
if ($socket === false) {
    $error = socket_strerror(socket_last_error());
    throw new RuntimeException("Cannot create socket: {$error}");
}

if (!socket_bind($socket, '127.0.0.1', 8080)) {
    $error = socket_strerror(socket_last_error($socket));
    socket_close($socket);
    throw new RuntimeException("Cannot bind socket: {$error}");
}
```

### 超时设置

设置合适的超时时间，避免程序阻塞。

**示例**：

```php
<?php
declare(strict_types=1);

// 设置接收超时（5秒）
socket_set_option($socket, SOL_SOCKET, SO_RCVTIMEO, ['sec' => 5, 'usec' => 0]);

// 设置发送超时（5秒）
socket_set_option($socket, SOL_SOCKET, SO_SNDTIMEO, ['sec' => 5, 'usec' => 0]);
```

## 常见问题

### 问题 1：TCP 和 UDP 的区别是什么？

**回答**：
- **TCP（传输控制协议）**：面向连接、可靠、有序、有流量控制，适合需要可靠传输的场景
- **UDP（用户数据报协议）**：无连接、不可靠、无序、无流量控制，适合需要快速传输的场景

**示例**：

```php
<?php
declare(strict_types=1);

// TCP：需要连接，可靠传输
$tcpSocket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
socket_connect($tcpSocket, '127.0.0.1', 8080);
socket_write($tcpSocket, 'Data');  // 保证送达

// UDP：无需连接，快速传输
$udpSocket = socket_create(AF_INET, SOCK_DGRAM, SOL_UDP);
socket_sendto($udpSocket, 'Data', 4, 0, '127.0.0.1', 8080);  // 可能丢失
```

### 问题 2：如何处理多个客户端连接？

**回答**：使用循环接受连接，为每个客户端创建独立的处理逻辑（可以使用多进程或多线程，但 PHP 主要使用单线程）。

**示例**：

```php
<?php
declare(strict_types=1);

$server = createTCPServer('127.0.0.1', 8080);

while (true) {
    $client = socket_accept($server);
    if ($client === false) {
        continue;
    }
    
    // 处理客户端（单线程，顺序处理）
    handleClient($client);
}
```

### 问题 3：Socket 编程中的常见错误？

**回答**：常见错误包括端口被占用、连接失败、读取超时等，需要检查每个操作的返回值。

**示例**：

```php
<?php
declare(strict_types=1);

// 检查端口是否被占用
if (!socket_bind($socket, '127.0.0.1', 8080)) {
    $errorCode = socket_last_error($socket);
    if ($errorCode === 98) {  // Address already in use
        echo "Port 8080 is already in use\n";
    }
}
```

## 最佳实践

### 1. 使用异常处理 Socket 错误

封装 Socket 操作，使用异常处理错误。

**示例**：

```php
<?php
declare(strict_types=1);

function safeSocketCreate(int $domain, int $type, int $protocol): Socket
{
    $socket = socket_create($domain, $type, $protocol);
    if ($socket === false) {
        throw new RuntimeException('Cannot create socket: ' . socket_strerror(socket_last_error()));
    }
    return $socket;
}
```

### 2. 设置合适的超时时间

设置接收和发送超时，避免程序无限阻塞。

**示例**：见"超时设置"部分

### 3. 及时关闭 Socket 资源

使用 try-finally 确保 Socket 资源被正确关闭。

**示例**：见"Socket 资源管理"部分

### 4. 考虑使用更高级的网络库

对于复杂的网络应用，考虑使用 cURL、Guzzle 等高级库。

**示例**：

```php
<?php
declare(strict_types=1);

// 对于 HTTP 请求，使用 cURL 或 Guzzle 更简单
// 而不是直接使用 Socket
use GuzzleHttp\Client;

$client = new Client();
$response = $client->get('https://api.example.com/data');
```

## 对比分析

### TCP vs UDP

| 特性         | TCP                              | UDP                              |
|:-------------|:---------------------------------|:---------------------------------|
| **连接**     | 面向连接（需要连接）             | 无连接（无需连接）               |
| **可靠性**   | ✅ 可靠（保证送达）              | ⚠️ 不可靠（可能丢失）            |
| **有序性**   | ✅ 有序（保证顺序）              | ⚠️ 无序（不保证顺序）            |
| **速度**     | ⚠️ 较慢（有额外开销）            | ✅ 较快（无额外开销）            |
| **适用场景** | 需要可靠传输（HTTP、FTP 等）     | 需要快速传输（DNS、游戏等）      |

### Socket vs 高级网络库

| 特性         | Socket                           | cURL/Guzzle                      |
|:-------------|:---------------------------------|:---------------------------------|
| **控制力**   | ✅ 完全控制                      | ⚠️ 受限于库的功能                |
| **复杂度**   | ⚠️ 较复杂                        | ✅ 简单易用                      |
| **适用场景** | 自定义协议、底层网络编程          | HTTP/HTTPS 请求、API 调用        |
| **学习曲线** | ⚠️ 较陡                          | ✅ 平缓                          |

## 练习任务

1. **TCP Echo 服务器**：实现一个 TCP Echo 服务器，将客户端发送的数据原样返回。

2. **UDP 时间服务器**：实现一个 UDP 时间服务器，客户端请求时返回当前时间。

3. **简单的聊天服务器**：实现一个简单的聊天服务器，支持多个客户端连接。

4. **网络协议实现**：设计并实现一个简单的网络协议（如命令-响应协议）。

5. **Socket 工具类**：创建一个 Socket 工具类，封装常用的 Socket 操作。

## 相关章节

- **[4.3.2 HTTP 客户端编程](section-02-http-client.md)**：学习 HTTP 客户端编程
- **[4.3.3 网络协议理解](section-03-network-protocols.md)**：理解网络协议的基本原理
