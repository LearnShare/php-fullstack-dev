# 6.6.1 查询优化

## 概述

查询优化是数据库性能优化的核心。本节介绍 N+1 查询问题、慢查询分析方法、EXPLAIN 的使用，以及查询优化技巧。

## N+1 查询问题

### 问题示例

```php
<?php
declare(strict_types=1);

// N+1 问题：1 次查询用户 + N 次查询订单
function getUsersWithOrders(PDO $pdo): array
{
    // 1 次查询
    $users = $pdo->query("SELECT * FROM users")->fetchAll();
    
    // N 次查询（每个用户一次）
    foreach ($users as &$user) {
        $stmt = $pdo->prepare("SELECT * FROM orders WHERE user_id = ?");
        $stmt->execute([$user['id']]);
        $user['orders'] = $stmt->fetchAll();
    }
    
    return $users;
}
```

### 解决方案：预加载（Eager Loading）

```php
<?php
declare(strict_types=1);

// 优化：2 次查询
function getUsersWithOrders(PDO $pdo): array
{
    // 1. 查询所有用户
    $users = $pdo->query("SELECT * FROM users")->fetchAll(PDO::FETCH_ASSOC);
    $userIds = array_column($users, 'id');
    
    if (empty($userIds)) {
        return [];
    }
    
    // 2. 批量查询所有订单
    $placeholders = implode(',', array_fill(0, count($userIds), '?'));
    $stmt = $pdo->prepare("SELECT * FROM orders WHERE user_id IN ({$placeholders})");
    $stmt->execute($userIds);
    $orders = $stmt->fetchAll(PDO::FETCH_ASSOC);
    
    // 3. 在内存中关联数据
    $ordersByUserId = [];
    foreach ($orders as $order) {
        $ordersByUserId[$order['user_id']][] = $order;
    }
    
    // 4. 合并数据
    foreach ($users as &$user) {
        $user['orders'] = $ordersByUserId[$user['id']] ?? [];
    }
    
    return $users;
}
```

### 使用 JOIN 优化

```php
<?php
declare(strict_types=1);

// 使用 JOIN 一次查询
function getUsersWithOrders(PDO $pdo): array
{
    $stmt = $pdo->query("
        SELECT 
            u.id as user_id,
            u.name as user_name,
            u.email,
            o.id as order_id,
            o.total,
            o.created_at
        FROM users u
        LEFT JOIN orders o ON u.id = o.user_id
        ORDER BY u.id, o.id
    ");
    
    $result = [];
    $currentUser = null;
    
    while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
        if ($currentUser === null || $currentUser['id'] != $row['user_id']) {
            if ($currentUser !== null) {
                $result[] = $currentUser;
            }
            $currentUser = [
                'id' => $row['user_id'],
                'name' => $row['user_name'],
                'email' => $row['email'],
                'orders' => [],
            ];
        }
        
        if ($row['order_id'] !== null) {
            $currentUser['orders'][] = [
                'id' => $row['order_id'],
                'total' => $row['total'],
                'created_at' => $row['created_at'],
            ];
        }
    }
    
    if ($currentUser !== null) {
        $result[] = $currentUser;
    }
    
    return $result;
}
```

## 慢查询分析

### 启用慢查询日志

```sql
-- 启用慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1; -- 1 秒
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow.log';
```

### 分析慢查询

```php
<?php
declare(strict_types=1);

class SlowQueryAnalyzer
{
    private string $logFile;
    
    public function __construct(string $logFile = '/var/log/mysql/slow.log')
    {
        $this->logFile = $logFile;
    }
    
    public function analyze(): array
    {
        $queries = $this->parseLog();
        $stats = [];
        
        foreach ($queries as $query) {
            $normalized = $this->normalizeQuery($query['sql']);
            
            if (!isset($stats[$normalized])) {
                $stats[$normalized] = [
                    'count' => 0,
                    'total_time' => 0,
                    'max_time' => 0,
                    'min_time' => PHP_FLOAT_MAX,
                ];
            }
            
            $stats[$normalized]['count']++;
            $stats[$normalized]['total_time'] += $query['time'];
            $stats[$normalized]['max_time'] = max($stats[$normalized]['max_time'], $query['time']);
            $stats[$normalized]['min_time'] = min($stats[$normalized]['min_time'], $query['time']);
        }
        
        // 按总时间排序
        uasort($stats, fn($a, $b) => $b['total_time'] <=> $a['total_time']);
        
        return $stats;
    }
    
    private function normalizeQuery(string $sql): string
    {
        // 移除具体值，保留查询结构
        $sql = preg_replace('/\d+/', '?', $sql);
        $sql = preg_replace('/\'[^\']*\'/', '?', $sql);
        return trim($sql);
    }
    
    private function parseLog(): array
    {
        // 解析慢查询日志
        // 实际实现需要根据日志格式解析
        return [];
    }
}
```

## EXPLAIN 使用

### 基础用法

```php
<?php
declare(strict_types=1);

function explainQuery(PDO $pdo, string $sql): array
{
    $stmt = $pdo->prepare("EXPLAIN {$sql}");
    $stmt->execute();
    return $stmt->fetchAll(PDO::FETCH_ASSOC);
}

// 使用
$result = explainQuery($pdo, "SELECT * FROM users WHERE email = 'test@example.com'");
print_r($result);
```

### EXPLAIN 结果分析

```php
<?php
declare(strict_types=1);

class ExplainAnalyzer
{
    public function analyze(array $explain): array
    {
        $issues = [];
        
        foreach ($explain as $row) {
            // 检查是否使用索引
            if ($row['key'] === null && $row['type'] !== 'const') {
                $issues[] = "Query does not use index";
            }
            
            // 检查全表扫描
            if ($row['type'] === 'ALL') {
                $issues[] = "Full table scan detected";
            }
            
            // 检查临时表
            if ($row['Extra'] && strpos($row['Extra'], 'Using temporary') !== false) {
                $issues[] = "Temporary table used";
            }
            
            // 检查文件排序
            if ($row['Extra'] && strpos($row['Extra'], 'Using filesort') !== false) {
                $issues[] = "File sort used";
            }
        }
        
        return $issues;
    }
}
```

## 查询优化技巧

### 1. 使用 LIMIT

```php
<?php
declare(strict_types=1);

// 不推荐：查询所有数据
$users = $pdo->query("SELECT * FROM users")->fetchAll();

// 推荐：使用 LIMIT
$users = $pdo->query("SELECT * FROM users LIMIT 100")->fetchAll();
```

### 2. 只查询需要的字段

```php
<?php
declare(strict_types=1);

// 不推荐：SELECT *
$users = $pdo->query("SELECT * FROM users")->fetchAll();

// 推荐：只查询需要的字段
$users = $pdo->query("SELECT id, name, email FROM users")->fetchAll();
```

### 3. 使用索引字段

```php
<?php
declare(strict_types=1);

// 不推荐：使用非索引字段
$user = $pdo->prepare("SELECT * FROM users WHERE name = ?")->execute([$name]);

// 推荐：使用索引字段
$user = $pdo->prepare("SELECT * FROM users WHERE email = ?")->execute([$email]);
```

### 4. 避免在 WHERE 中使用函数

```php
<?php
declare(strict_types=1);

// 不推荐：使用函数
$users = $pdo->query("SELECT * FROM users WHERE DATE(created_at) = '2024-01-01'")->fetchAll();

// 推荐：直接比较
$users = $pdo->query("SELECT * FROM users WHERE created_at >= '2024-01-01' AND created_at < '2024-01-02'")->fetchAll();
```

## 完整示例

```php
<?php
declare(strict_types=1);

class QueryOptimizer
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function optimizeQuery(string $sql, array $params = []): array
    {
        // 1. 分析查询计划
        $explain = $this->explain($sql, $params);
        $issues = $this->analyzeExplain($explain);
        
        // 2. 执行查询
        $start = microtime(true);
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute($params);
        $results = $stmt->fetchAll(PDO::FETCH_ASSOC);
        $duration = microtime(true) - $start;
        
        // 3. 记录慢查询
        if ($duration > 1.0) {
            $this->logSlowQuery($sql, $params, $duration);
        }
        
        return [
            'results' => $results,
            'duration' => $duration,
            'issues' => $issues,
            'explain' => $explain,
        ];
    }
    
    private function explain(string $sql, array $params): array
    {
        $explainSql = "EXPLAIN {$sql}";
        $stmt = $this->pdo->prepare($explainSql);
        $stmt->execute($params);
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
    
    private function analyzeExplain(array $explain): array
    {
        $issues = [];
        
        foreach ($explain as $row) {
            if ($row['key'] === null && $row['type'] !== 'const') {
                $issues[] = "Missing index";
            }
            if ($row['type'] === 'ALL') {
                $issues[] = "Full table scan";
            }
        }
        
        return $issues;
    }
    
    private function logSlowQuery(string $sql, array $params, float $duration): void
    {
        error_log(sprintf(
            "Slow query: %.4fs\nSQL: %s\nParams: %s",
            $duration,
            $sql,
            json_encode($params)
        ));
    }
}
```

## 最佳实践

1. **避免 N+1 查询**：使用预加载或 JOIN
2. **使用索引**：为常用查询字段创建索引
3. **分析慢查询**：定期分析慢查询日志
4. **使用 EXPLAIN**：分析查询执行计划
5. **优化查询**：只查询需要的字段，使用 LIMIT

## 注意事项

1. 索引不是越多越好，会影响写入性能
2. 避免在 WHERE 子句中使用函数
3. 合理使用 JOIN，避免过度 JOIN
4. 定期分析慢查询，持续优化

## 练习

1. 识别并修复一个存在 N+1 查询问题的函数。

2. 使用 EXPLAIN 分析一个复杂查询，找出优化点。

3. 实现一个慢查询分析工具，分析慢查询日志。

4. 优化一个包含多个 JOIN 的查询，提升性能。
