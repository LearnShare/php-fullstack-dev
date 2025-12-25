# 5.4.2 窗口函数

## 概述

窗口函数（Window Functions）是 MySQL 8.0+ 引入的强大功能，可以在结果集上执行计算而不改变行数。本节详细介绍窗口函数基础、排名函数、聚合窗口函数，以及窗口函数的应用场景。

## 什么是窗口函数

- 对查询结果集的每一行执行计算
- 不改变结果集的行数，只添加计算列
- 支持分组和排序

## 排名函数

### ROW_NUMBER

```sql
-- 为每行分配序号
SELECT 
    id,
    name,
    price,
    ROW_NUMBER() OVER (ORDER BY price DESC) AS rank
FROM products;
```

### RANK 和 DENSE_RANK

```sql
-- RANK：相同值排名相同，下一个排名跳过
SELECT 
    id,
    name,
    price,
    RANK() OVER (ORDER BY price DESC) AS rank
FROM products;

-- DENSE_RANK：相同值排名相同，下一个排名不跳过
SELECT 
    id,
    name,
    price,
    DENSE_RANK() OVER (ORDER BY price DESC) AS rank
FROM products;
```

## 分组窗口函数

```sql
-- 按类别分组排名
SELECT 
    id,
    name,
    category_id,
    price,
    ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY price DESC) AS rank_in_category
FROM products;
```

## 累计统计

```sql
-- 累计求和
SELECT 
    date,
    amount,
    SUM(amount) OVER (ORDER BY date) AS cumulative_sum
FROM sales;

-- 移动平均
SELECT 
    date,
    amount,
    AVG(amount) OVER (ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg
FROM sales;
```

## 窗口函数在 PHP 中使用

```php
<?php
declare(strict_types=1);

// 查询带排名的产品
$stmt = $pdo->query("
    SELECT 
        id,
        name,
        price,
        ROW_NUMBER() OVER (ORDER BY price DESC) AS rank
    FROM products
");
$products = $stmt->fetchAll();

// 查询每个类别的 Top 3 产品
$stmt = $pdo->query("
    SELECT * FROM (
        SELECT 
            id,
            name,
            category_id,
            price,
            ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY price DESC) AS rank
        FROM products
    ) AS ranked
    WHERE rank <= 3
");
$topProducts = $stmt->fetchAll();
```

## 完整示例

```php
<?php
declare(strict_types=1);

class SalesReport
{
    private PDO $pdo;

    public function getMonthlySalesWithCumulative(): array
    {
        $stmt = $this->pdo->query("
            SELECT 
                DATE_FORMAT(date, '%Y-%m') AS month,
                SUM(amount) AS monthly_sales,
                SUM(SUM(amount)) OVER (ORDER BY DATE_FORMAT(date, '%Y-%m')) AS cumulative_sales
            FROM sales
            GROUP BY DATE_FORMAT(date, '%Y-%m')
            ORDER BY month
        ");
        return $stmt->fetchAll();
    }

    public function getTopProductsByCategory(int $limit = 3): array
    {
        $stmt = $this->pdo->prepare("
            SELECT * FROM (
                SELECT 
                    id,
                    name,
                    category_id,
                    price,
                    ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY price DESC) AS rank
                FROM products
            ) AS ranked
            WHERE rank <= ?
        ");
        $stmt->execute([$limit]);
        return $stmt->fetchAll();
    }
}
```

## 注意事项

1. **MySQL 版本**：窗口函数需要 MySQL 8.0+
2. **性能考虑**：窗口函数可能影响查询性能
3. **排序优化**：合理使用 ORDER BY 提升性能
4. **分区使用**：使用 PARTITION BY 进行分组计算

## 练习

1. 使用窗口函数实现产品销售排名功能。

2. 实现累计统计功能，计算累计销售额。

3. 编写一个查询，使用窗口函数计算移动平均。

4. 实现分组排名功能，按类别排名产品。
