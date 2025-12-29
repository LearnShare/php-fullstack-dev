# 5.1.3 Shared Nothing 架构

## 概述

Shared Nothing（无共享）架构是 PHP Web 应用的重要特性，也是 PHP 能够轻松实现水平扩展的基础。理解 Shared Nothing 架构对于设计可扩展的 PHP Web 应用至关重要。本节详细介绍 Shared Nothing 架构的概念、PHP 的实现方式、无状态设计的优势、扩展性优势等内容，帮助零基础学员理解 PHP Web 应用的特点和优势。

Shared Nothing 架构的核心思想是：每个请求处理单元（在 PHP 中是进程或线程）不共享任何状态信息，每个请求都是独立处理的。这种设计使得 PHP Web 应用可以轻松实现水平扩展：只需要增加服务器或进程数量，无需担心状态同步问题。

PHP 的 Shared Nothing 特性体现在：
- 每个请求在独立的进程中处理
- 进程之间不共享内存
- 请求之间不共享状态
- 状态信息存储在外部（数据库、缓存、Session 存储等）

理解 Shared Nothing 架构有助于：
- 设计可扩展的 Web 应用
- 实现水平扩展
- 理解 PHP Web 应用的特点
- 优化应用架构

**主要内容**：
- Shared Nothing 架构的概念和原理
- PHP 的 Shared Nothing 特性实现
- 无状态设计的含义和实现方式
- Shared Nothing 架构的优势（扩展性、故障隔离等）
- 状态管理策略（Session、数据库、缓存等）
- 水平扩展的实现方法
- 与其他架构的对比
- 实际应用示例和最佳实践

## 特性

- **请求隔离**：每个请求在独立的进程中处理，互不干扰
- **无状态设计**：请求之间不共享状态，每个请求都是独立的
- **易于扩展**：可以轻松增加服务器或进程数量，实现水平扩展
- **故障隔离**：单个请求的故障不会影响其他请求
- **资源隔离**：每个进程有独立的内存空间，资源使用互不影响
- **负载均衡友好**：可以轻松实现负载均衡，请求可以分发到任意服务器

## Shared Nothing 架构概念

### 什么是 Shared Nothing

Shared Nothing（无共享）架构是一种分布式系统架构模式，其核心特点是：

- **无共享内存**：不同进程或服务器之间不共享内存
- **无共享存储**：每个节点有独立的存储（或通过外部存储共享数据）
- **无共享状态**：每个节点不共享运行时状态

在 PHP Web 应用中，Shared Nothing 体现在：
- 每个 PHP 进程独立运行
- 进程之间不共享内存
- 请求之间不共享状态
- 状态信息存储在外部（数据库、Redis、Session 存储等）

### 与其他架构的对比

#### Shared Memory（共享内存）

**特点**：
- 多个进程共享同一块内存
- 进程间可以直接通信
- 需要同步机制防止竞争

**问题**：
- 难以扩展（受内存限制）
- 同步复杂
- 故障影响范围大

#### Shared Disk（共享磁盘）

**特点**：
- 多个节点共享存储
- 通过文件系统共享数据
- 需要文件锁机制

**问题**：
- 存储成为瓶颈
- 文件锁影响性能
- 扩展性受限

#### Shared Nothing（无共享）

**特点**：
- 每个节点完全独立
- 通过外部存储共享数据
- 无共享状态

**优势**：
- 易于扩展
- 故障隔离
- 性能可预测

### 架构优势

Shared Nothing 架构的主要优势：

1. **易于水平扩展**：
   - 可以轻松增加服务器或进程数量
   - 无需担心状态同步问题
   - 扩展成本低

2. **故障隔离**：
   - 单个请求的故障不影响其他请求
   - 单个服务器的故障不影响其他服务器
   - 提高系统可用性

3. **性能可预测**：
   - 每个请求独立处理
   - 不受其他请求影响
   - 性能稳定

4. **负载均衡友好**：
   - 请求可以分发到任意服务器
   - 无需考虑状态粘性
   - 负载均衡简单

## PHP 的 Shared Nothing 特性

### 请求隔离

在 PHP Web 应用中，每个 HTTP 请求在独立的 PHP 进程中处理。

**处理流程**：
1. Web Server 接收 HTTP 请求
2. 转发给 PHP-FPM
3. PHP-FPM 分配空闲的 Worker 进程
4. Worker 进程独立处理请求
5. 返回响应后，进程可以处理下一个请求

**隔离效果**：
- 请求之间互不干扰
- 一个请求的错误不影响其他请求
- 一个请求的慢处理不影响其他请求

### 进程独立

每个 PHP-FPM Worker 进程完全独立运行。

**独立性**：
- 独立的内存空间
- 独立的文件句柄
- 独立的数据库连接
- 独立的全局变量

**示例**：
```php
<?php
declare(strict_types=1);

// 每个进程有独立的全局变量
$globalCounter = 0;

function incrementCounter(): int {
    global $globalCounter;
    $globalCounter++;
    return $globalCounter;
}

// 不同请求的 $globalCounter 是独立的
// 请求 A：$globalCounter 从 0 开始
// 请求 B：$globalCounter 也从 0 开始（独立进程）
```

### 内存隔离

每个 PHP 进程有独立的内存空间，进程之间不共享内存。

**内存管理**：
- 每个进程有独立的内存堆
- 进程之间无法直接访问对方的内存
- 进程结束后，内存自动释放

**优势**：
- 内存泄漏只影响单个进程
- 进程崩溃不影响其他进程
- 内存使用可预测

### 状态管理

由于进程之间不共享状态，状态信息需要存储在外部。

**状态存储方式**：
- **数据库**：持久化状态（用户数据、业务数据等）
- **缓存（Redis/Memcached）**：临时状态（缓存数据、计数器等）
- **Session 存储**：用户会话状态（文件、数据库、Redis 等）
- **文件系统**：文件存储（上传的文件、日志等）

## 无状态设计

### 无状态的含义

无状态（Stateless）是指服务器不保存客户端的状态信息，每个请求都包含处理该请求所需的全部信息。

**有状态 vs 无状态**：

| 特性 | 有状态 | 无状态 |
|:-----|:-------|:-------|
| 状态存储 | 服务器内存 | 外部存储（数据库、缓存等） |
| 请求依赖 | 依赖之前的请求 | 每个请求独立 |
| 扩展性 | 难以扩展 | 易于扩展 |
| 故障影响 | 状态丢失影响大 | 故障影响小 |

### 无状态设计的实现

#### 1. 不在服务器内存中保存状态

**错误示例**（有状态）：
```php
<?php
declare(strict_types=1);

// 错误：在服务器内存中保存状态
$userSessions = []; // 全局变量

function login(string $username): void {
    global $userSessions;
    $userSessions[$username] = time();
}

function isLoggedIn(string $username): bool {
    global $userSessions;
    return isset($userSessions[$username]);
}
```

**问题**：
- 状态存储在服务器内存中
- 进程重启后状态丢失
- 无法在多服务器间共享
- 难以扩展

**正确示例**（无状态）：
```php
<?php
declare(strict_types=1);

// 正确：状态存储在外部（Redis）
function login(string $username, Redis $redis): void {
    $sessionId = bin2hex(random_bytes(16));
    $redis->setex("session:{$sessionId}", $username, 3600);
    setcookie('session_id', $sessionId, time() + 3600);
}

function isLoggedIn(Redis $redis): bool {
    $sessionId = $_COOKIE['session_id'] ?? '';
    if (empty($sessionId)) {
        return false;
    }
    return $redis->exists("session:{$sessionId}");
}
```

**优势**：
- 状态存储在外部（Redis）
- 进程重启后状态不丢失
- 可以在多服务器间共享
- 易于扩展

#### 2. 请求包含所有必要信息

**错误示例**（依赖服务器状态）：
```php
<?php
declare(strict_types=1);

// 错误：依赖服务器内存中的状态
$currentUser = null; // 全局变量

function setCurrentUser(string $username): void {
    global $currentUser;
    $currentUser = $username;
}

function getCurrentUser(): ?string {
    global $currentUser;
    return $currentUser;
}
```

**正确示例**（请求包含信息）：
```php
<?php
declare(strict_types=1);

// 正确：从请求中获取信息（Session、Token 等）
function getCurrentUser(Redis $redis): ?string {
    $sessionId = $_COOKIE['session_id'] ?? '';
    if (empty($sessionId)) {
        return null;
    }
    return $redis->get("session:{$sessionId}");
}
```

#### 3. 使用外部存储管理状态

**状态存储选择**：

| 存储类型 | 适用场景 | 示例 |
|:---------|:---------|:-----|
| **数据库** | 持久化数据 | 用户信息、业务数据 |
| **Redis** | 临时数据、缓存 | Session、计数器、缓存 |
| **Memcached** | 缓存数据 | 页面缓存、数据缓存 |
| **文件系统** | 文件存储 | 上传的文件、日志 |

**示例**（使用 Redis 存储 Session）：
```php
<?php
declare(strict_types=1);

class SessionManager
{
    public function __construct(
        private Redis $redis
    ) {
    }

    public function start(string $sessionId): void
    {
        if (empty($sessionId)) {
            $sessionId = bin2hex(random_bytes(16));
            setcookie('session_id', $sessionId, time() + 3600);
        }
    }

    public function set(string $key, mixed $value, int $ttl = 3600): void
    {
        $sessionId = $_COOKIE['session_id'] ?? '';
        if (empty($sessionId)) {
            return;
        }
        $this->redis->setex("session:{$sessionId}:{$key}", $ttl, serialize($value));
    }

    public function get(string $key): mixed
    {
        $sessionId = $_COOKIE['session_id'] ?? '';
        if (empty($sessionId)) {
            return null;
        }
        $data = $this->redis->get("session:{$sessionId}:{$key}");
        return $data ? unserialize($data) : null;
    }

    public function destroy(): void
    {
        $sessionId = $_COOKIE['session_id'] ?? '';
        if (empty($sessionId)) {
            return;
        }
        $keys = $this->redis->keys("session:{$sessionId}:*");
        if (!empty($keys)) {
            $this->redis->del($keys);
        }
        setcookie('session_id', '', time() - 3600);
    }
}
```

## 扩展性优势

### 水平扩展

Shared Nothing 架构使得 PHP Web 应用可以轻松实现水平扩展。

**扩展方式**：
1. **增加 PHP-FPM 进程**：在同一服务器上增加 Worker 进程数量
2. **增加服务器**：添加新的服务器，运行 PHP-FPM
3. **负载均衡**：使用负载均衡器分发请求到多个服务器

**扩展示例**：

**单服务器架构**：
```
客户端 → Nginx → PHP-FPM (10 进程)
```

**多服务器架构**：
```
客户端 → 负载均衡器 → 服务器1 (PHP-FPM 10 进程)
                    → 服务器2 (PHP-FPM 10 进程)
                    → 服务器3 (PHP-FPM 10 进程)
```

### 负载均衡

由于每个请求是独立的，可以轻松实现负载均衡。

**负载均衡策略**：
- **轮询（Round Robin）**：依次分发请求
- **加权轮询（Weighted Round Robin）**：根据服务器性能分配权重
- **最少连接（Least Connections）**：分发到连接数最少的服务器
- **IP 哈希（IP Hash）**：根据客户端 IP 分发（需要 Session 粘性时使用）

**Nginx 负载均衡配置**：
```nginx
upstream php_backend {
    server 192.168.1.10:9000;
    server 192.168.1.11:9000;
    server 192.168.1.12:9000;
}

server {
    location ~ \.php$ {
        fastcgi_pass php_backend;
        # ... 其他配置
    }
}
```

### 故障隔离

Shared Nothing 架构提供了良好的故障隔离能力。

**隔离效果**：
- **请求级别**：单个请求的故障不影响其他请求
- **进程级别**：单个进程的崩溃不影响其他进程
- **服务器级别**：单个服务器的故障不影响其他服务器

**故障处理**：
- 进程崩溃：PHP-FPM 自动重启进程
- 服务器故障：负载均衡器自动移除故障服务器
- 请求超时：终止超时请求，不影响其他请求

### 资源利用

每个进程独立管理资源，资源利用更加高效。

**资源管理**：
- **内存**：每个进程有独立的内存空间，内存使用可预测
- **CPU**：进程可以充分利用多核 CPU
- **文件句柄**：每个进程独立管理文件句柄
- **数据库连接**：每个进程可以维护独立的数据库连接池

## 状态管理策略

### Session 管理

Session 用于在多个请求之间保持用户状态。

**Session 存储方式**：

#### 1. 文件存储（默认）

**配置**（php.ini）：
```ini
session.save_handler = files
session.save_path = "/var/lib/php/sessions"
```

**特点**：
- 简单易用
- 不适合多服务器部署
- 性能较低

#### 2. 数据库存储

**配置**：
```ini
session.save_handler = user
```

**实现**：
```php
<?php
declare(strict_types=1);

class DatabaseSessionHandler implements SessionHandlerInterface
{
    public function __construct(
        private PDO $pdo
    ) {
    }

    public function open(string $path, string $name): bool
    {
        return true;
    }

    public function close(): bool
    {
        return true;
    }

    public function read(string $id): string
    {
        $stmt = $this->pdo->prepare('SELECT data FROM sessions WHERE id = ?');
        $stmt->execute([$id]);
        $result = $stmt->fetchColumn();
        return $result ?: '';
    }

    public function write(string $id, string $data): bool
    {
        $stmt = $this->pdo->prepare(
            'INSERT INTO sessions (id, data, last_activity) 
             VALUES (?, ?, ?) 
             ON DUPLICATE KEY UPDATE data = ?, last_activity = ?'
        );
        return $stmt->execute([$id, $data, time(), $data, time()]);
    }

    public function destroy(string $id): bool
    {
        $stmt = $this->pdo->prepare('DELETE FROM sessions WHERE id = ?');
        return $stmt->execute([$id]);
    }

    public function gc(int $max_lifetime): int|false
    {
        $stmt = $this->pdo->prepare('DELETE FROM sessions WHERE last_activity < ?');
        $stmt->execute([time() - $max_lifetime]);
        return $stmt->rowCount();
    }
}

// 使用
$handler = new DatabaseSessionHandler($pdo);
session_set_save_handler($handler, true);
session_start();
```

#### 3. Redis 存储

**配置**：
```ini
session.save_handler = redis
session.save_path = "tcp://127.0.0.1:6379"
```

**特点**：
- 性能高
- 支持多服务器共享
- 适合高并发场景

### 缓存管理

使用外部缓存（Redis、Memcached）存储临时数据。

**缓存使用场景**：
- 页面缓存
- 数据缓存
- 计数器
- 分布式锁

**示例**（使用 Redis 缓存）：
```php
<?php
declare(strict_types=1);

class CacheManager
{
    public function __construct(
        private Redis $redis
    ) {
    }

    public function get(string $key): mixed
    {
        $data = $this->redis->get($key);
        return $data ? unserialize($data) : null;
    }

    public function set(string $key, mixed $value, int $ttl = 3600): bool
    {
        return $this->redis->setex($key, $ttl, serialize($value));
    }

    public function increment(string $key, int $value = 1): int
    {
        return $this->redis->incrBy($key, $value);
    }
}
```

### 数据持久化

使用数据库持久化业务数据。

**数据存储**：
- **关系型数据库**（MySQL、PostgreSQL）：结构化数据
- **NoSQL 数据库**（MongoDB）：非结构化数据
- **文件系统**：文件存储

## 使用场景

### Web 应用架构设计

- 设计可扩展的 Web 应用架构
- 实现无状态设计
- 规划扩展方案

### 扩展性规划

- 规划水平扩展方案
- 设计负载均衡策略
- 规划状态存储方案

### 性能优化

- 优化状态管理
- 优化缓存策略
- 优化资源利用

### 分布式系统

- 实现分布式 Web 应用
- 实现微服务架构
- 实现高可用系统

## 注意事项

### 状态管理策略

- 选择合适的状态存储方式（数据库、Redis、文件等）
- 考虑状态的一致性和可用性
- 考虑状态的过期和清理

### 会话处理

- 使用外部存储（Redis、数据库）存储 Session
- 避免使用文件存储（不适合多服务器部署）
- 设置合理的 Session 过期时间

### 数据一致性

- 使用事务保证数据一致性
- 使用分布式锁防止并发问题
- 考虑最终一致性

### 缓存策略

- 合理使用缓存提高性能
- 注意缓存失效和更新
- 避免缓存穿透和雪崩

### 扩展性考虑

- 设计时考虑水平扩展
- 避免硬编码服务器信息
- 使用配置管理服务器地址

## 常见问题

### Shared Nothing 的优势？

1. **易于扩展**：可以轻松增加服务器或进程数量
2. **故障隔离**：单个请求或服务器的故障不影响其他部分
3. **性能可预测**：每个请求独立处理，性能稳定
4. **负载均衡友好**：请求可以分发到任意服务器

### 如何实现无状态设计？

1. **不在服务器内存中保存状态**：状态存储在外部（数据库、Redis 等）
2. **请求包含所有必要信息**：从请求中获取所需信息（Session、Token 等）
3. **使用外部存储管理状态**：选择合适的存储方式（数据库、Redis、文件等）

### 如何处理会话？

1. **使用外部存储**：将 Session 存储在 Redis 或数据库中
2. **Session ID 传递**：通过 Cookie 或 URL 参数传递 Session ID
3. **Session 过期管理**：设置合理的过期时间，定期清理过期 Session

### 如何实现水平扩展？

1. **增加进程数量**：在同一服务器上增加 PHP-FPM Worker 进程
2. **增加服务器**：添加新的服务器，运行 PHP-FPM
3. **负载均衡**：使用负载均衡器分发请求到多个服务器
4. **状态外部化**：确保状态存储在外部（数据库、Redis 等），可以在多服务器间共享

## 最佳实践

### 设计无状态应用

- 不在服务器内存中保存状态
- 请求包含所有必要信息
- 使用外部存储管理状态

### 外部化状态存储

- 使用数据库持久化业务数据
- 使用 Redis 存储临时数据和缓存
- 使用文件系统存储文件

### 使用会话管理

- 使用外部存储（Redis、数据库）存储 Session
- 设置合理的 Session 过期时间
- 定期清理过期 Session

### 考虑扩展性

- 设计时考虑水平扩展
- 避免硬编码服务器信息
- 使用配置管理服务器地址
- 实现负载均衡

### 监控和优化

- 监控状态存储的使用情况
- 优化缓存策略
- 优化数据库查询
- 根据实际负载调整配置

## 相关章节

- **[5.1.1 HTTP 请求响应流程](section-01-http-flow.md)**：了解 HTTP 协议基础
- **[5.1.2 Web Server 与 PHP-FPM 协作](section-02-web-server-fpm.md)**：了解 Web Server 与 PHP-FPM 的协作机制
- **[5.9 会话与状态管理](../chapter-09-session/readme.md)**：了解 Session 和 Cookie 的详细使用方法
- **[6.5 Redis](../stage-06-database/chapter-05-redis/readme.md)**：了解 Redis 的使用方法

## 练习任务

1. **实现无状态设计**
   - 创建一个简单的 Web 应用
   - 确保应用是无状态的（状态存储在外部）
   - 测试多个请求的独立性

2. **配置 Session 存储**
   - 配置 Session 使用 Redis 存储
   - 测试 Session 在多进程间的共享
   - 验证 Session 的过期和清理

3. **实现水平扩展**
   - 配置多个 PHP-FPM 服务器
   - 配置负载均衡器
   - 测试请求分发和故障转移

4. **状态管理实践**
   - 使用 Redis 存储临时数据
   - 使用数据库持久化业务数据
   - 实现缓存策略

5. **监控和优化**
   - 监控状态存储的使用情况
   - 分析性能瓶颈
   - 优化状态管理策略
