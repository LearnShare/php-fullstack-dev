# 6.6 数据库深度优化

## 目标

- 识别和解决 N+1 查询问题。
- 掌握慢查询分析方法（EXPLAIN、profiling）。
- 理解索引设计原则。
- 了解分库分表和读写分离的基础。

## N+1 问题

### 问题示例

```php
<?php
// N+1 问题
$users = User::all();  // 1 次查询

foreach ($users as $user) {
    echo $user->orders->count();  // N 次查询（每个用户一次）
}
// 总共：1 + N 次查询
```

### 解决方案：预加载

```php
<?php
// Eloquent 预加载
$users = User::with('orders')->get();  // 2 次查询（用户 + 订单）

foreach ($users as $user) {
    echo $user->orders->count();  // 0 次查询（已加载）
}

// 嵌套预加载
$users = User::with('orders.items')->get();
```

### Doctrine 预加载

```php
<?php
// Doctrine 预加载
$users = $entityManager->createQueryBuilder()
    ->select('u', 'o')
    ->from(User::class, 'u')
    ->leftJoin('u.orders', 'o')
    ->getQuery()
    ->getResult();
```

## 慢查询分析

### EXPLAIN

```sql
-- 分析查询计划
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';

-- 输出说明：
-- type: ALL（全表扫描）、index（索引扫描）、ref（索引查找）、const（常量）
-- key: 使用的索引
-- rows: 扫描的行数
-- Extra: 额外信息（Using index、Using where 等）
```

### 查询分析

```php
<?php
declare(strict_types=1);

class QueryAnalyzer
{
    private PDO $db;
    
    public function __construct(PDO $db)
    {
        $this->db = $db;
    }
    
    public function explain(string $sql, array $params = []): array
    {
        $stmt = $this->db->prepare("EXPLAIN {$sql}");
        $stmt->execute($params);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
    
    public function profile(string $sql, array $params = []): array
    {
        $this->db->exec('SET profiling = 1');
        
        $stmt = $this->db->prepare($sql);
        $stmt->execute($params);
        $stmt->fetchAll();
        
        $profile = $this->db->query('SHOW PROFILES')->fetchAll(PDO::FETCH_ASSOC);
        $this->db->exec('SET profiling = 0');
        
        return $profile;
    }
}
```

## 索引设计

### 索引类型

```sql
-- 普通索引
CREATE INDEX idx_email ON users(email);

-- 唯一索引
CREATE UNIQUE INDEX uk_email ON users(email);

-- 复合索引
CREATE INDEX idx_user_status ON orders(user_id, status);

-- 全文索引
CREATE FULLTEXT INDEX ft_content ON articles(title, content);
```

### 索引设计原则

1. **为 WHERE 条件创建索引**
2. **为 JOIN 条件创建索引**
3. **为 ORDER BY 创建索引**
4. **复合索引遵循最左前缀原则**

```sql
-- 复合索引：idx_user_status_created
-- 可以使用：
-- WHERE user_id = 1
-- WHERE user_id = 1 AND status = 'active'
-- WHERE user_id = 1 AND status = 'active' ORDER BY created_at

-- 不能使用：
-- WHERE status = 'active'  -- 不遵循最左前缀
```

## 分库分表

### 垂直分库

```php
<?php
// 按业务模块分库
$userDb = new PDO('mysql:host=user-db;dbname=users', ...);
$orderDb = new PDO('mysql:host=order-db;dbname=orders', ...);
```

### 水平分表

```php
<?php
declare(strict_types=1);

class ShardingManager
{
    private array $connections;
    private int $shardCount;
    
    public function __construct(array $connections)
    {
        $this->connections = $connections;
        $this->shardCount = count($connections);
    }
    
    public function getShard(int $userId): PDO
    {
        $shardIndex = $userId % $this->shardCount;
        return $this->connections[$shardIndex];
    }
    
    public function getTableName(int $userId): string
    {
        $shardIndex = $userId % $this->shardCount;
        return "users_{$shardIndex}";
    }
}
```

## 读写分离

```php
<?php
declare(strict_types=1);

class DatabaseManager
{
    private PDO $writeConnection;
    private array $readConnections;
    
    public function __construct(PDO $writeConnection, array $readConnections)
    {
        $this->writeConnection = $writeConnection;
        $this->readConnections = $readConnections;
    }
    
    public function getReadConnection(): PDO
    {
        // 随机选择读库（负载均衡）
        return $this->readConnections[array_rand($this->readConnections)];
    }
    
    public function getWriteConnection(): PDO
    {
        return $this->writeConnection;
    }
    
    public function query(string $sql, array $params = [], bool $isWrite = false): array
    {
        $connection = $isWrite ? $this->getWriteConnection() : $this->getReadConnection();
        $stmt = $connection->prepare($sql);
        $stmt->execute($params);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
}
```

## 练习

1. 识别并修复一个存在 N+1 查询问题的代码。

2. 使用 EXPLAIN 分析慢查询，优化索引设计。

3. 实现一个查询分析工具，自动识别性能问题。

4. 设计一个简单的分库分表方案，支持按用户 ID 分片。

5. 创建一个读写分离的数据库管理器。

6. 编写性能测试，比较优化前后的查询性能。
