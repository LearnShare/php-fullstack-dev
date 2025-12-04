# 6.6.3 数据库配置优化

## 概述

数据库配置优化是提升数据库性能的重要环节。本节介绍 MySQL 配置优化、连接池、读写分离、分库分表等内容。

## MySQL 配置优化

### 基础配置

```ini
# my.cnf
[mysqld]
# 连接数
max_connections = 500

# 缓冲区大小
innodb_buffer_pool_size = 2G
innodb_log_file_size = 256M

# 查询缓存（MySQL 8.0 已移除）
# query_cache_size = 64M

# 临时表
tmp_table_size = 64M
max_heap_table_size = 64M
```

### InnoDB 优化

```ini
# InnoDB 配置
[mysqld]
innodb_buffer_pool_size = 2G
innodb_log_file_size = 256M
innodb_log_buffer_size = 16M
innodb_flush_log_at_trx_commit = 2
innodb_file_per_table = 1
innodb_flush_method = O_DIRECT
```

## 连接池

### PDO 连接池

```php
<?php
declare(strict_types=1);

class ConnectionPool
{
    private array $connections = [];
    private int $maxConnections;
    private int $currentConnections = 0;
    
    public function __construct(int $maxConnections = 10)
    {
        $this->maxConnections = $maxConnections;
    }
    
    public function getConnection(string $dsn, string $user, string $password): PDO
    {
        if ($this->currentConnections < $this->maxConnections) {
            $pdo = new PDO($dsn, $user, $password, [
                PDO::ATTR_PERSISTENT => true,
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            ]);
            $this->connections[] = $pdo;
            $this->currentConnections++;
            return $pdo;
        }
        
        // 返回现有连接
        return $this->connections[array_rand($this->connections)];
    }
}
```

### 使用连接池

```php
<?php
declare(strict_types=1);

$pool = new ConnectionPool(10);
$pdo = $pool->getConnection($dsn, $user, $password);
```

## 读写分离

### 主从配置

```php
<?php
declare(strict_types=1);

class DatabaseManager
{
    private PDO $master;
    private array $slaves = [];
    private int $slaveIndex = 0;
    
    public function __construct(PDO $master, array $slaves)
    {
        $this->master = $master;
        $this->slaves = $slaves;
    }
    
    public function getReadConnection(): PDO
    {
        // 轮询选择从库
        $slave = $this->slaves[$this->slaveIndex];
        $this->slaveIndex = ($this->slaveIndex + 1) % count($this->slaves);
        return $slave;
    }
    
    public function getWriteConnection(): PDO
    {
        return $this->master;
    }
    
    public function read(callable $callback): mixed
    {
        $pdo = $this->getReadConnection();
        return $callback($pdo);
    }
    
    public function write(callable $callback): mixed
    {
        return $callback($this->master);
    }
}

// 使用
$db = new DatabaseManager($master, $slaves);

// 读操作使用从库
$users = $db->read(function(PDO $pdo) {
    return $pdo->query("SELECT * FROM users")->fetchAll();
});

// 写操作使用主库
$db->write(function(PDO $pdo) {
    $stmt = $pdo->prepare("INSERT INTO users (name, email) VALUES (?, ?)");
    $stmt->execute(['John', 'john@example.com']);
});
```

## 分库分表

### 分表策略

```php
<?php
declare(strict_types=1);

class ShardingManager
{
    private int $shardCount;
    
    public function __construct(int $shardCount = 10)
    {
        $this->shardCount = $shardCount;
    }
    
    public function getTableName(string $baseTable, int $userId): string
    {
        $shard = $userId % $this->shardCount;
        return "{$baseTable}_{$shard}";
    }
    
    public function getConnection(int $userId): PDO
    {
        $shard = $userId % $this->shardCount;
        // 返回对应分片的连接
        return $this->connections[$shard];
    }
}

// 使用
$sharding = new ShardingManager(10);
$tableName = $sharding->getTableName('users', $userId);
$pdo = $sharding->getConnection($userId);
```

## 完整示例

```php
<?php
declare(strict_types=1);

class OptimizedDatabase
{
    private PDO $master;
    private array $slaves;
    private ConnectionPool $pool;
    
    public function __construct(PDO $master, array $slaves)
    {
        $this->master = $master;
        $this->slaves = $slaves;
        $this->pool = new ConnectionPool(20);
    }
    
    public function query(string $sql, array $params = [], bool $isWrite = false): array
    {
        $pdo = $isWrite ? $this->master : $this->getSlave();
        $stmt = $pdo->prepare($sql);
        $stmt->execute($params);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
    
    private function getSlave(): PDO
    {
        return $this->slaves[array_rand($this->slaves)];
    }
}
```

## 最佳实践

1. **连接池**：使用连接池管理数据库连接
2. **读写分离**：读操作使用从库，写操作使用主库
3. **分库分表**：根据数据量选择分片策略
4. **配置优化**：根据硬件资源优化 MySQL 配置
5. **监控**：监控数据库性能，及时调整配置

## 注意事项

1. 读写分离需要考虑主从延迟
2. 分库分表会增加系统复杂度
3. 配置优化需要根据实际负载调整
4. 定期监控数据库性能指标

## 练习

1. 实现一个数据库连接池，管理数据库连接。

2. 配置读写分离，读操作使用从库，写操作使用主库。

3. 实现一个分表策略，根据用户 ID 分表。

4. 优化 MySQL 配置，提升数据库性能。
