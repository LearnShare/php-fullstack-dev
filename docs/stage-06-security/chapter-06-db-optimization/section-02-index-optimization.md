# 6.6.2 索引优化

## 概述

索引是提升数据库查询性能的关键。本节介绍索引设计原则、复合索引、覆盖索引、索引维护等内容。

## 索引设计原则

### 1. 为常用查询字段创建索引

```sql
-- 为经常用于 WHERE 条件的字段创建索引
CREATE INDEX idx_email ON users(email);
CREATE INDEX idx_created_at ON users(created_at);
```

### 2. 为外键创建索引

```sql
-- 外键字段通常用于 JOIN，应该创建索引
CREATE INDEX idx_user_id ON orders(user_id);
```

### 3. 为排序字段创建索引

```sql
-- 为 ORDER BY 字段创建索引
CREATE INDEX idx_created_at ON users(created_at);
```

## 复合索引

### 最左前缀原则

```sql
-- 复合索引：user_id, status, created_at
CREATE INDEX idx_user_status_created ON orders(user_id, status, created_at);

-- 可以使用索引的查询
SELECT * FROM orders WHERE user_id = 1; -- ✓ 使用索引
SELECT * FROM orders WHERE user_id = 1 AND status = 'pending'; -- ✓ 使用索引
SELECT * FROM orders WHERE user_id = 1 AND status = 'pending' AND created_at > '2024-01-01'; -- ✓ 使用索引

-- 不能使用索引的查询
SELECT * FROM orders WHERE status = 'pending'; -- ✗ 不使用索引（不是最左前缀）
SELECT * FROM orders WHERE created_at > '2024-01-01'; -- ✗ 不使用索引
```

### 索引列顺序

```sql
-- 原则：选择性高的列放在前面
-- 选择性 = 不同值的数量 / 总行数

-- 示例：user_id 选择性高，status 选择性低
CREATE INDEX idx_user_status ON orders(user_id, status);
```

## 覆盖索引

### 什么是覆盖索引

覆盖索引是指索引包含了查询所需的所有字段，不需要回表查询。

```sql
-- 创建覆盖索引
CREATE INDEX idx_user_email_name ON users(user_id, email, name);

-- 查询可以使用覆盖索引
SELECT user_id, email, name FROM users WHERE user_id = 1;
-- 不需要访问表数据，直接从索引获取
```

### 覆盖索引示例

```php
<?php
declare(strict_types=1);

// 使用覆盖索引优化查询
class UserRepository
{
    private PDO $pdo;
    
    public function getUserBasicInfo(int $userId): ?array
    {
        // 只查询索引包含的字段，使用覆盖索引
        $stmt = $this->pdo->prepare("
            SELECT user_id, email, name 
            FROM users 
            WHERE user_id = ?
        ");
        $stmt->execute([$userId]);
        return $stmt->fetch(PDO::FETCH_ASSOC) ?: null;
    }
}
```

## 索引类型

### 1. 普通索引

```sql
CREATE INDEX idx_email ON users(email);
```

### 2. 唯一索引

```sql
CREATE UNIQUE INDEX idx_email ON users(email);
```

### 3. 主键索引

```sql
ALTER TABLE users ADD PRIMARY KEY (id);
```

### 4. 全文索引

```sql
CREATE FULLTEXT INDEX idx_content ON articles(content);
```

## 索引维护

### 查看索引

```sql
-- 查看表的索引
SHOW INDEX FROM users;

-- 查看索引使用情况
SELECT * FROM information_schema.STATISTICS 
WHERE table_name = 'users';
```

### 分析索引使用

```sql
-- 分析查询是否使用索引
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
```

### 重建索引

```sql
-- 重建索引
ALTER TABLE users DROP INDEX idx_email;
CREATE INDEX idx_email ON users(email);

-- 或使用 OPTIMIZE TABLE
OPTIMIZE TABLE users;
```

## 索引优化工具

```php
<?php
declare(strict_types=1);

class IndexAnalyzer
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function analyzeTable(string $table): array
    {
        // 获取表的所有索引
        $indexes = $this->getIndexes($table);
        
        // 分析每个索引的使用情况
        $analysis = [];
        foreach ($indexes as $index) {
            $analysis[$index['Key_name']] = [
                'columns' => $index['Column_name'],
                'cardinality' => $index['Cardinality'],
                'usage' => $this->checkIndexUsage($table, $index),
            ];
        }
        
        return $analysis;
    }
    
    private function getIndexes(string $table): array
    {
        $stmt = $this->pdo->prepare("SHOW INDEX FROM {$table}");
        $stmt->execute();
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
    
    private function checkIndexUsage(string $table, array $index): string
    {
        // 检查索引是否被使用
        // 实际实现需要分析慢查询日志或使用 PERFORMANCE_SCHEMA
        return 'unknown';
    }
    
    public function suggestIndexes(string $table): array
    {
        // 分析查询模式，建议创建索引
        // 实际实现需要分析查询日志
        return [];
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class IndexOptimizer
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function optimizeIndexes(string $table): void
    {
        // 1. 分析现有索引
        $indexes = $this->analyzeIndexes($table);
        
        // 2. 识别未使用的索引
        $unused = $this->findUnusedIndexes($indexes);
        
        // 3. 建议新索引
        $suggested = $this->suggestIndexes($table);
        
        // 4. 生成优化建议
        return [
            'unused_indexes' => $unused,
            'suggested_indexes' => $suggested,
        ];
    }
    
    private function analyzeIndexes(string $table): array
    {
        $stmt = $this->pdo->prepare("SHOW INDEX FROM {$table}");
        $stmt->execute();
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
    
    private function findUnusedIndexes(array $indexes): array
    {
        // 查找未使用的索引
        // 实际实现需要分析查询日志
        return [];
    }
    
    private function suggestIndexes(string $table): array
    {
        // 建议创建新索引
        // 实际实现需要分析查询模式
        return [];
    }
}
```

## 最佳实践

1. **选择性高的字段**：为选择性高的字段创建索引
2. **复合索引顺序**：按选择性从高到低排列
3. **覆盖索引**：创建包含查询字段的覆盖索引
4. **定期维护**：定期分析和优化索引
5. **避免过度索引**：索引会影响写入性能

## 注意事项

1. 索引会占用存储空间
2. 索引会影响 INSERT、UPDATE、DELETE 性能
3. 不是所有查询都需要索引
4. 定期分析索引使用情况，删除未使用的索引

## 练习

1. 为一个表设计合适的索引，优化常用查询。

2. 创建一个复合索引，遵循最左前缀原则。

3. 实现一个索引分析工具，分析索引使用情况。

4. 优化一个查询，使用覆盖索引提升性能。
