# 6.6.1 数据库连接池

## 概述

数据库连接池是管理数据库连接的重要机制，通过复用数据库连接，减少连接创建和销毁的开销，提高系统性能和资源利用率。理解连接池的工作原理和实现方式对于构建高性能的数据库应用至关重要。

本节详细介绍数据库连接池的概念、工作原理、实现方式、配置参数、连接管理策略等，帮助零基础学员掌握连接池的使用和优化。

**主要内容**：
- 连接池的概念和优势
- 连接池的工作原理
- 连接池的实现方式
- 连接池的配置参数
- 连接管理策略
- 完整示例和最佳实践

---

## 特性

- **连接复用**：复用数据库连接，减少开销
- **资源管理**：统一管理数据库连接
- **性能优化**：提高系统性能
- **连接控制**：控制连接数量，防止资源耗尽
- **自动管理**：自动创建、回收、销毁连接

---

## 连接池概念

### 什么是连接池

连接池（Connection Pool）是预先创建并维护一定数量的数据库连接，供应用复用的机制。

**传统方式的问题**：

- 每次请求都创建新连接，开销大
- 连接创建和销毁耗时
- 连接数可能过多，导致资源耗尽

**连接池的优势**：

- 复用连接，减少开销
- 控制连接数量
- 提高性能

### 连接池的工作原理

连接池的工作流程：

1. **初始化**：创建一定数量的连接
2. **获取连接**：从池中获取可用连接
3. **使用连接**：使用连接执行操作
4. **归还连接**：操作完成后归还连接到池中
5. **连接管理**：自动管理连接的生命周期

**示例**：

```php
<?php
declare(strict_types=1);

class ConnectionPool
{
    private array $connections = [];
    private int $maxConnections = 10;
    private int $currentConnections = 0;
    
    public function getConnection(): PDO
    {
        // 1. 检查是否有可用连接
        if (!empty($this->connections)) {
            return array_pop($this->connections);
        }
        
        // 2. 检查是否超过最大连接数
        if ($this->currentConnections >= $this->maxConnections) {
            throw new RuntimeException('连接池已满');
        }
        
        // 3. 创建新连接
        $connection = $this->createConnection();
        $this->currentConnections++;
        
        return $connection;
    }
    
    public function releaseConnection(PDO $connection): void
    {
        // 归还连接到池中
        $this->connections[] = $connection;
    }
    
    private function createConnection(): PDO
    {
        $dsn = 'mysql:host=localhost;dbname=test;charset=utf8mb4';
        return new PDO($dsn, 'user', 'pass', [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION
        ]);
    }
}
```

---

## 连接池实现

### 简单连接池实现

实现一个简单的连接池。

**示例**：

```php
<?php
declare(strict_types=1);

class SimpleConnectionPool
{
    private array $pool = [];
    private int $maxSize = 10;
    private int $minSize = 2;
    private string $dsn;
    private string $username;
    private string $password;
    private array $options;
    
    public function __construct(
        string $dsn,
        string $username,
        string $password,
        array $options = []
    ) {
        $this->dsn = $dsn;
        $this->username = $username;
        $this->password = $password;
        $this->options = $options;
        
        // 初始化最小连接数
        for ($i = 0; $i < $this->minSize; $i++) {
            $this->pool[] = $this->createConnection();
        }
    }
    
    public function getConnection(): PDO
    {
        if (!empty($this->pool)) {
            return array_pop($this->pool);
        }
        
        if (count($this->pool) < $this->maxSize) {
            return $this->createConnection();
        }
        
        throw new RuntimeException('连接池已满');
    }
    
    public function releaseConnection(PDO $connection): void
    {
        // 检查连接是否有效
        if ($this->isConnectionValid($connection)) {
            $this->pool[] = $connection;
        } else {
            // 连接无效，创建新连接
            $this->pool[] = $this->createConnection();
        }
    }
    
    private function createConnection(): PDO
    {
        return new PDO(
            $this->dsn,
            $this->username,
            $this->password,
            $this->options
        );
    }
    
    private function isConnectionValid(PDO $connection): bool
    {
        try {
            $connection->query('SELECT 1');
            return true;
        } catch (PDOException $e) {
            return false;
        }
    }
}
```

### 使用单例模式

使用单例模式管理连接池。

**示例**：

```php
<?php
declare(strict_types=1);

class ConnectionPoolManager
{
    private static ?ConnectionPoolManager $instance = null;
    private SimpleConnectionPool $pool;
    
    private function __construct()
    {
        $this->pool = new SimpleConnectionPool(
            'mysql:host=localhost;dbname=test',
            'user',
            'pass'
        );
    }
    
    public static function getInstance(): ConnectionPoolManager
    {
        if (self::$instance === null) {
            self::$instance = new self();
        }
        
        return self::$instance;
    }
    
    public function getConnection(): PDO
    {
        return $this->pool->getConnection();
    }
    
    public function releaseConnection(PDO $connection): void
    {
        $this->pool->releaseConnection($connection);
    }
}

// 使用
$pool = ConnectionPoolManager::getInstance();
$pdo = $pool->getConnection();
// 使用连接
$pool->releaseConnection($pdo);
```

---

## 连接池配置

### 最大连接数

设置连接池的最大连接数。

**示例**：

```php
<?php
declare(strict_types=1);

class ConnectionPool
{
    private int $maxConnections = 20;  // 最大连接数
    
    public function setMaxConnections(int $max): void
    {
        $this->maxConnections = $max;
    }
}
```

### 最小连接数

设置连接池的最小连接数（初始连接数）。

**示例**：

```php
<?php
declare(strict_types=1);

class ConnectionPool
{
    private int $minConnections = 5;  // 最小连接数
    
    public function setMinConnections(int $min): void
    {
        $this->minConnections = $min;
    }
}
```

### 连接超时

设置获取连接的超时时间。

**示例**：

```php
<?php
declare(strict_types=1);

class ConnectionPool
{
    private int $timeout = 5;  // 超时时间（秒）
    
    public function getConnection(): PDO
    {
        $startTime = time();
        
        while (empty($this->pool) && (time() - $startTime) < $this->timeout) {
            usleep(100000);  // 等待 100ms
        }
        
        if (empty($this->pool)) {
            throw new RuntimeException('获取连接超时');
        }
        
        return array_pop($this->pool);
    }
}
```

### 连接空闲时间

设置连接的最大空闲时间，超过时间自动回收。

**示例**：

```php
<?php
declare(strict_types=1);

class ConnectionPool
{
    private int $maxIdleTime = 3600;  // 最大空闲时间（秒）
    private array $connectionTimes = [];  // 连接时间记录
    
    public function releaseConnection(PDO $connection): void
    {
        $this->connectionTimes[spl_object_hash($connection)] = time();
        $this->pool[] = $connection;
    }
    
    public function cleanupIdleConnections(): void
    {
        $now = time();
        $validConnections = [];
        
        foreach ($this->pool as $connection) {
            $hash = spl_object_hash($connection);
            $idleTime = $now - ($this->connectionTimes[$hash] ?? $now);
            
            if ($idleTime < $this->maxIdleTime) {
                $validConnections[] = $connection;
            } else {
                // 关闭空闲连接
                unset($this->connectionTimes[$hash]);
            }
        }
        
        $this->pool = $validConnections;
    }
}
```

---

## 连接管理策略

### 连接验证

归还连接前验证连接是否有效。

**示例**：

```php
<?php
declare(strict_types=1);

public function releaseConnection(PDO $connection): void
{
    // 验证连接
    if ($this->isConnectionValid($connection)) {
        $this->pool[] = $connection;
    } else {
        // 连接无效，创建新连接
        $this->pool[] = $this->createConnection();
    }
}

private function isConnectionValid(PDO $connection): bool
{
    try {
        $connection->query('SELECT 1');
        return true;
    } catch (PDOException $e) {
        return false;
    }
}
```

### 连接回收

定期回收无效连接。

**示例**：

```php
<?php
declare(strict_types=1);

public function recycleConnections(): void
{
    $validConnections = [];
    
    foreach ($this->pool as $connection) {
        if ($this->isConnectionValid($connection)) {
            $validConnections[] = $connection;
        }
    }
    
    $this->pool = $validConnections;
}
```

---

## 完整示例

### 完整连接池实现

```php
<?php
declare(strict_types=1);

class DatabaseConnectionPool
{
    private array $pool = [];
    private array $inUse = [];
    private int $maxSize = 20;
    private int $minSize = 5;
    private int $timeout = 5;
    private int $maxIdleTime = 3600;
    private array $connectionTimes = [];
    private string $dsn;
    private string $username;
    private string $password;
    private array $options;
    
    public function __construct(
        string $dsn,
        string $username,
        string $password,
        array $options = []
    ) {
        $this->dsn = $dsn;
        $this->username = $username;
        $this->password = $password;
        $this->options = $options;
        
        // 初始化最小连接数
        for ($i = 0; $i < $this->minSize; $i++) {
            $this->pool[] = $this->createConnection();
        }
        
        // 启动清理任务
        $this->startCleanupTask();
    }
    
    public function getConnection(): PDO
    {
        // 1. 检查是否有可用连接
        if (!empty($this->pool)) {
            $connection = array_pop($this->pool);
            $this->inUse[spl_object_hash($connection)] = $connection;
            return $connection;
        }
        
        // 2. 检查是否超过最大连接数
        if (count($this->inUse) >= $this->maxSize) {
            // 等待可用连接
            $startTime = time();
            while (empty($this->pool) && (time() - $startTime) < $this->timeout) {
                usleep(100000);
            }
            
            if (empty($this->pool)) {
                throw new RuntimeException('获取连接超时');
            }
            
            $connection = array_pop($this->pool);
            $this->inUse[spl_object_hash($connection)] = $connection;
            return $connection;
        }
        
        // 3. 创建新连接
        $connection = $this->createConnection();
        $this->inUse[spl_object_hash($connection)] = $connection;
        
        return $connection;
    }
    
    public function releaseConnection(PDO $connection): void
    {
        $hash = spl_object_hash($connection);
        
        if (!isset($this->inUse[$hash])) {
            return;  // 连接不在使用中
        }
        
        unset($this->inUse[$hash]);
        
        // 验证连接
        if ($this->isConnectionValid($connection)) {
            $this->connectionTimes[$hash] = time();
            $this->pool[] = $connection;
        }
    }
    
    private function createConnection(): PDO
    {
        return new PDO(
            $this->dsn,
            $this->username,
            $this->password,
            $this->options
        );
    }
    
    private function isConnectionValid(PDO $connection): bool
    {
        try {
            $connection->query('SELECT 1');
            return true;
        } catch (PDOException $e) {
            return false;
        }
    }
    
    private function startCleanupTask(): void
    {
        // 定期清理空闲连接
        if (function_exists('pcntl_alarm')) {
            pcntl_alarm(60);  // 每分钟执行一次
            pcntl_signal(SIGALRM, function () {
                $this->cleanupIdleConnections();
                pcntl_alarm(60);
            });
        }
    }
    
    private function cleanupIdleConnections(): void
    {
        $now = time();
        $validConnections = [];
        
        foreach ($this->pool as $connection) {
            $hash = spl_object_hash($connection);
            $idleTime = $now - ($this->connectionTimes[$hash] ?? $now);
            
            if ($idleTime < $this->maxIdleTime && $this->isConnectionValid($connection)) {
                $validConnections[] = $connection;
            } else {
                unset($this->connectionTimes[$hash]);
            }
        }
        
        $this->pool = $validConnections;
    }
}

// 使用
$pool = new DatabaseConnectionPool(
    'mysql:host=localhost;dbname=test',
    'user',
    'pass'
);

$pdo = $pool->getConnection();
// 使用连接
$pool->releaseConnection($pdo);
```

---

## 使用场景

### 高并发应用

高并发应用中，连接池可以显著提高性能。

### 长时间运行的应用

长时间运行的应用，连接池可以复用连接。

### 资源受限的环境

资源受限的环境中，连接池可以控制连接数量。

---

## 注意事项

### 连接泄漏

注意防止连接泄漏，确保连接被正确归还。

### 连接数配置

合理配置连接数，避免过多或过少。

### 连接验证

定期验证连接有效性，及时回收无效连接。

---

## 常见问题

### 什么是连接池？

连接池是预先创建并维护一定数量的数据库连接，供应用复用的机制。

### 连接池的优势？

- 复用连接，减少开销
- 控制连接数量
- 提高性能

### 如何配置连接池？

根据应用需求配置最大连接数、最小连接数、超时时间等参数。

---

## 最佳实践

### 合理配置连接数

根据应用负载配置合适的连接数。

### 防止连接泄漏

确保连接被正确归还。

### 定期清理连接

定期清理无效和空闲连接。

---

## 练习任务

1. **实现连接池**
   - 实现基本连接池
   - 实现连接获取和归还
   - 测试连接池功能
   - 验证连接复用

2. **连接管理**
   - 实现连接验证
   - 实现连接回收
   - 实现连接清理
   - 测试连接管理

3. **性能测试**
   - 测试连接池性能
   - 对比传统方式
   - 分析性能差异
   - 优化连接池

4. **配置优化**
   - 优化连接数配置
   - 优化超时配置
   - 测试不同配置
   - 编写配置文档

5. **综合应用**
   - 创建一个完整的连接池系统
   - 实现所有功能
   - 优化性能
   - 编写最佳实践文档

---

**相关章节**：

- [6.2.1 PDO 基础与连接](../chapter-02-pdo/section-01-pdo-basics.md)
- [6.6.2 数据库备份与恢复](section-02-backup-restore.md)
- [6.6.3 消息队列基础](section-03-message-queue.md)
