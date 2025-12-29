# 5.17.1 WebSocket 概述

## 概述

WebSocket 提供了全双工实时通信能力，是构建实时 Web 应用的重要技术。理解 WebSocket 的概念、掌握 WebSocket 与 HTTP 的区别、了解 WebSocket 的应用场景和协议特点，对于构建实时通信应用至关重要。本节详细介绍 WebSocket 概念、WebSocket 与 HTTP 的区别、WebSocket 协议特点、应用场景、握手过程等内容，帮助零基础学员理解 WebSocket 协议。

WebSocket 是现代 Web 应用实现实时通信的标准协议，相比传统的 HTTP 轮询，WebSocket 提供了更高效、更低延迟的通信方式。理解 WebSocket 的工作原理对于实现实时应用至关重要。

**主要内容**：
- WebSocket 概念（什么是 WebSocket、WebSocket 的优势、WebSocket 的特点）
- WebSocket 与 HTTP 的区别（连接方式对比、通信方式对比、性能对比、使用场景对比）
- WebSocket 协议特点（全双工通信、低延迟、持久连接、协议升级）
- 应用场景（实时聊天、实时通知、在线游戏、实时数据推送）
- 握手过程（HTTP 升级请求、服务器响应、连接建立、消息传输）
- 实际应用示例和最佳实践

## 特性

- **全双工**：支持双向通信
- **低延迟**：低延迟通信
- **持久连接**：持久连接，无需重复握手
- **高效传输**：高效的二进制和文本传输
- **标准协议**：遵循 WebSocket 标准（RFC 6455）

## WebSocket 概念

### 什么是 WebSocket

WebSocket 是一种在单个 TCP 连接上进行全双工通信的协议，允许服务器主动向客户端推送数据。

### WebSocket 的优势

1. **实时通信**：支持实时双向通信
2. **低延迟**：相比 HTTP 轮询，延迟更低
3. **高效传输**：减少 HTTP 头部开销
4. **持久连接**：避免重复建立连接
5. **服务器推送**：服务器可以主动推送数据

### WebSocket 的特点

1. **全双工通信**：客户端和服务器可以同时发送数据
2. **持久连接**：连接建立后保持打开状态
3. **低开销**：相比 HTTP，头部开销更小
4. **跨域支持**：支持跨域通信

## WebSocket 与 HTTP 的区别

### 连接方式对比

| 特性 | HTTP | WebSocket |
|:-----|:-----|:----------|
| 连接方式 | 请求-响应 | 持久连接 |
| 连接建立 | 每次请求建立 | 一次建立，持续使用 |
| 连接关闭 | 响应后关闭 | 显式关闭 |

### 通信方式对比

| 特性 | HTTP | WebSocket |
|:-----|:-----|:----------|
| 通信方向 | 客户端→服务器 | 双向通信 |
| 服务器推送 | 不支持（需要轮询） | 支持 |
| 实时性 | 低（需要轮询） | 高（实时推送） |

### 性能对比

| 特性 | HTTP | WebSocket |
|:-----|:-----|:----------|
| 延迟 | 高（每次请求建立连接） | 低（持久连接） |
| 开销 | 高（每次请求包含头部） | 低（连接建立后头部很小） |
| 效率 | 低（需要轮询） | 高（实时推送） |

### 使用场景对比

| 场景 | HTTP | WebSocket |
|:-----|:-----|:----------|
| 传统 Web 页面 | ✅ 适合 | ❌ 不适合 |
| 实时聊天 | ⚠️ 可以（轮询） | ✅ 适合 |
| 实时通知 | ⚠️ 可以（轮询） | ✅ 适合 |
| 在线游戏 | ❌ 不适合 | ✅ 适合 |

## WebSocket 协议特点

### 全双工通信

**特点**：客户端和服务器可以同时发送和接收数据。

**示例**：
```javascript
// 客户端
const ws = new WebSocket('ws://example.com');
ws.send('Hello Server');  // 客户端发送

ws.onmessage = function(event) {
    console.log('Received:', event.data);  // 客户端接收
};
```

### 低延迟

**特点**：持久连接避免了每次请求建立连接的开销。

**优势**：
- 无需重复握手
- 减少网络延迟
- 提高响应速度

### 持久连接

**特点**：连接建立后保持打开状态，直到显式关闭。

**示例**：
```php
<?php
declare(strict_types=1);

// WebSocket 连接建立后保持打开
// 可以持续发送和接收消息
// 直到客户端或服务器关闭连接
```

### 协议升级

**握手过程**：
1. 客户端发送 HTTP 升级请求
2. 服务器响应升级确认
3. 连接升级为 WebSocket
4. 开始 WebSocket 通信

**示例**：
```http
# 客户端请求
GET /ws HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

# 服务器响应
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

## 应用场景

### 实时聊天

**示例**：
```php
<?php
declare(strict_types=1);

// 实时聊天应用
// 用户发送消息 -> WebSocket -> 服务器 -> WebSocket -> 其他用户
```

### 实时通知

**示例**：
```php
<?php
declare(strict_types=1);

// 实时通知系统
// 服务器 -> WebSocket -> 客户端
// 无需客户端轮询
```

### 在线游戏

**示例**：
```php
<?php
declare(strict_types=1);

// 在线游戏
// 玩家操作 -> WebSocket -> 服务器 -> WebSocket -> 其他玩家
// 实时同步游戏状态
```

### 实时数据推送

**示例**：
```php
<?php
declare(strict_types=1);

// 实时数据推送
// 股票价格、体育比分、监控数据等
// 服务器 -> WebSocket -> 客户端
```

## 握手过程

### HTTP 升级请求

**客户端请求**：
```http
GET /ws HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Origin: http://example.com
```

### 服务器响应

**服务器响应**：
```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

### 连接建立

**示例**：
```php
<?php
declare(strict_types=1);

function handleWebSocketHandshake($request): bool
{
    // 检查升级请求
    if (!isset($request['headers']['upgrade']) || 
        strtolower($request['headers']['upgrade']) !== 'websocket') {
        return false;
    }
    
    // 检查 WebSocket 版本
    if (!isset($request['headers']['sec-websocket-version']) ||
        $request['headers']['sec-websocket-version'] !== '13') {
        return false;
    }
    
    // 生成响应密钥
    $key = $request['headers']['sec-websocket-key'] ?? '';
    $accept = base64_encode(sha1($key . '258EAFA5-E914-47DA-95CA-C5AB0DC85B11', true));
    
    // 发送升级响应
    $response = "HTTP/1.1 101 Switching Protocols\r\n";
    $response .= "Upgrade: websocket\r\n";
    $response .= "Connection: Upgrade\r\n";
    $response .= "Sec-WebSocket-Accept: {$accept}\r\n\r\n";
    
    return true;
}
```

### 消息传输

**示例**：
```php
<?php
declare(strict_types=1);

// WebSocket 消息格式
// 帧格式：FIN(1) + RSV(3) + Opcode(4) + Mask(1) + Payload Length(7/7+16/7+64) + Masking Key(0/32) + Payload Data
```

## 使用场景

### 实时通信应用

- 实时聊天
- 实时协作
- 实时通知

### 实时数据推送

- 股票价格
- 体育比分
- 监控数据

### 在线协作

- 在线编辑
- 实时白板
- 协同工作

### 实时监控

- 系统监控
- 日志实时推送
- 性能监控

## 注意事项

### 连接管理

- **连接保持**：保持连接活跃
- **连接超时**：处理连接超时
- **连接重连**：实现自动重连

### 消息处理

- **消息格式**：定义消息格式
- **消息验证**：验证消息内容
- **消息队列**：处理消息队列

### 错误处理

- **连接错误**：处理连接错误
- **消息错误**：处理消息错误
- **重连机制**：实现重连机制

### 性能考虑

- **连接数限制**：限制连接数
- **消息大小**：限制消息大小
- **资源管理**：管理连接资源

## 常见问题

### WebSocket 和 HTTP 的区别？

- **连接方式**：HTTP 是请求-响应，WebSocket 是持久连接
- **通信方向**：HTTP 是单向，WebSocket 是双向
- **实时性**：HTTP 需要轮询，WebSocket 支持实时推送

### WebSocket 的应用场景？

- 实时聊天
- 实时通知
- 在线游戏
- 实时数据推送

### WebSocket 的性能优势？

- 低延迟（持久连接）
- 高效传输（减少头部开销）
- 服务器推送（无需轮询）

### 如何实现 WebSocket 服务器？

使用 Ratchet、ReactPHP 等库实现 WebSocket 服务器。

## 最佳实践

### 理解 WebSocket 协议

- 理解 WebSocket 协议特点
- 理解握手过程
- 理解消息格式

### 选择合适的实现方案

- 选择合适的 WebSocket 库
- 考虑性能和可扩展性
- 考虑开发复杂度

### 实现连接管理

- 管理连接生命周期
- 处理连接超时
- 实现连接重连

### 处理连接错误

- 捕获连接错误
- 实现错误恢复
- 记录错误日志

## 相关章节

- **[5.17.2 Ratchet](section-02-ratchet.md)**：了解 Ratchet 的详细内容
- **[5.17.3 ReactPHP WebSocket](section-03-reactphp.md)**：了解 ReactPHP WebSocket 的详细内容
- **[5.17.4 WebSocket 最佳实践](section-04-best-practices.md)**：了解 WebSocket 最佳实践的详细内容
- **[4.3.2 HTTP 客户端编程](../stage-04-system/chapter-03-network/section-02-http-client.md)**：了解网络编程的基础知识

## 练习任务

1. **理解 WebSocket 协议**
   - 理解 WebSocket 概念
   - 理解与 HTTP 的区别
   - 理解握手过程

2. **实现基本 WebSocket 客户端**
   - WebSocket 连接
   - 消息发送和接收
   - 连接管理

3. **实现 WebSocket 握手**
   - HTTP 升级请求
   - 服务器响应
   - 连接建立

4. **实现消息传输**
   - 消息格式
   - 消息编码和解码
   - 消息处理

5. **实现完整的 WebSocket 通信**
   - 连接管理
   - 消息处理
   - 错误处理
