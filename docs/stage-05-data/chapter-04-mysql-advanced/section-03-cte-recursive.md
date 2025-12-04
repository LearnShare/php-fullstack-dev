# 5.4.3 CTE 与递归查询

## 概述

CTE（Common Table Expression，公用表表达式）和递归查询是 MySQL 8.0+ 的强大功能，可以简化复杂查询和处理层次结构数据。本节详细介绍 CTE 基础、递归 CTE、递归查询应用场景，以及性能优化。

## CTE 基础

### 什么是 CTE

- **CTE**：Common Table Expression，公用表表达式
- 使用 `WITH` 子句定义临时结果集
- 可以在查询中多次引用

### 基础 CTE

```sql
-- WITH 子句
WITH top_customers AS (
    SELECT user_id, SUM(total) AS total_spent
    FROM orders
    GROUP BY user_id
    ORDER BY total_spent DESC
    LIMIT 10
)
SELECT u.*, tc.total_spent
FROM users u
INNER JOIN top_customers tc ON u.id = tc.user_id;
```

### 多个 CTE

```sql
WITH 
    top_customers AS (
        SELECT user_id, SUM(total) AS total_spent
        FROM orders
        GROUP BY user_id
        ORDER BY total_spent DESC
        LIMIT 10
    ),
    customer_stats AS (
        SELECT 
            user_id,
            COUNT(*) AS order_count,
            AVG(total) AS avg_order
        FROM orders
        GROUP BY user_id
    )
SELECT 
    u.*,
    tc.total_spent,
    cs.order_count,
    cs.avg_order
FROM users u
INNER JOIN top_customers tc ON u.id = tc.user_id
INNER JOIN customer_stats cs ON u.id = cs.user_id;
```

## 递归 CTE

### 递归查询语法

```sql
WITH RECURSIVE cte_name AS (
    -- 基础查询（锚点）
    SELECT ... FROM ...
    
    UNION ALL
    
    -- 递归查询
    SELECT ... FROM cte_name WHERE ...
)
SELECT * FROM cte_name;
```

### 层次结构查询

```sql
-- 查询组织架构（树形结构）
WITH RECURSIVE org_tree AS (
    -- 基础查询：根节点
    SELECT id, name, parent_id, 1 AS level
    FROM organizations
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- 递归查询：子节点
    SELECT o.id, o.name, o.parent_id, ot.level + 1
    FROM organizations o
    INNER JOIN org_tree ot ON o.parent_id = ot.id
)
SELECT * FROM org_tree;
```

### 路径查询

```sql
-- 查询从根到叶子的完整路径
WITH RECURSIVE category_path AS (
    -- 基础查询：根分类
    SELECT id, name, parent_id, name AS path
    FROM categories
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- 递归查询：构建路径
    SELECT 
        c.id,
        c.name,
        c.parent_id,
        CONCAT(cp.path, ' > ', c.name) AS path
    FROM categories c
    INNER JOIN category_path cp ON c.parent_id = cp.id
)
SELECT * FROM category_path;
```

## 递归查询应用场景

### 1. 组织架构

```php
<?php
declare(strict_types=1);

function getOrganizationTree(PDO $pdo, ?int $rootId = null): array
{
    $sql = "
        WITH RECURSIVE org_tree AS (
            SELECT id, name, parent_id, 1 AS level
            FROM organizations
            WHERE parent_id " . ($rootId === null ? "IS NULL" : "= ?") . "
            
            UNION ALL
            
            SELECT o.id, o.name, o.parent_id, ot.level + 1
            FROM organizations o
            INNER JOIN org_tree ot ON o.parent_id = ot.id
        )
        SELECT * FROM org_tree
    ";
    
    $stmt = $rootId === null 
        ? $pdo->query($sql)
        : $pdo->prepare($sql);
    
    if ($rootId !== null) {
        $stmt->execute([$rootId]);
    }
    
    return $stmt->fetchAll();
}
```

### 2. 分类树

```php
<?php
declare(strict_types=1);

function getCategoryPath(PDO $pdo, int $categoryId): ?string
{
    $stmt = $pdo->prepare("
        WITH RECURSIVE category_path AS (
            SELECT id, name, parent_id, name AS path
            FROM categories
            WHERE id = ?
            
            UNION ALL
            
            SELECT c.id, c.name, c.parent_id, CONCAT(c.name, ' > ', cp.path)
            FROM categories c
            INNER JOIN category_path cp ON c.id = cp.parent_id
        )
        SELECT path FROM category_path WHERE parent_id IS NULL
    ");
    
    $stmt->execute([$categoryId]);
    $result = $stmt->fetchColumn();
    
    return $result !== false ? $result : null;
}
```

## 性能优化

### 递归深度限制

```sql
-- 设置递归深度限制
SET SESSION cte_max_recursion_depth = 1000;

-- 或在 CTE 中限制
WITH RECURSIVE cte AS (
    SELECT ... FROM ...
    UNION ALL
    SELECT ... FROM cte WHERE level < 10  -- 限制深度
)
SELECT * FROM cte;
```

### 索引优化

```sql
-- 为递归查询创建索引
CREATE INDEX idx_parent_id ON organizations(parent_id);
CREATE INDEX idx_category_parent ON categories(parent_id);
```

## 完整示例

```php
<?php
declare(strict_types=1);

class CategoryService
{
    private PDO $pdo;

    public function getCategoryTree(?int $rootId = null): array
    {
        $sql = "
            WITH RECURSIVE category_tree AS (
                SELECT id, name, parent_id, 1 AS level, CAST(name AS CHAR(1000)) AS path
                FROM categories
                WHERE parent_id " . ($rootId === null ? "IS NULL" : "= ?") . "
                
                UNION ALL
                
                SELECT 
                    c.id,
                    c.name,
                    c.parent_id,
                    ct.level + 1,
                    CONCAT(ct.path, ' > ', c.name)
                FROM categories c
                INNER JOIN category_tree ct ON c.parent_id = ct.id
                WHERE ct.level < 10
            )
            SELECT * FROM category_tree ORDER BY path
        ";
        
        $stmt = $rootId === null 
            ? $pdo->query($sql)
            : $pdo->prepare($sql);
        
        if ($rootId !== null) {
            $stmt->execute([$rootId]);
        }
        
        return $stmt->fetchAll();
    }
}
```

## 注意事项

1. **递归深度**：设置合理的递归深度限制
2. **性能考虑**：递归查询可能较慢，需要索引优化
3. **终止条件**：确保递归查询有明确的终止条件
4. **MySQL 版本**：CTE 需要 MySQL 8.0+

## 练习

1. 实现一个组织架构查询功能，使用递归 CTE。

2. 创建一个分类树查询，获取完整的分类路径。

3. 编写一个递归查询，计算层级结构的深度。

4. 实现一个递归查询，查找所有子节点。
