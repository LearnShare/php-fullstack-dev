# 5.4 现代 MySQL 高级特性

## 目标

- 掌握 MySQL JSON 字段的使用，包括查询、索引和存储结构。
- 理解窗口函数（Window Functions）的应用场景。
- 熟悉全文索引（Fulltext Index）与 LIKE 查询的区别。
- 能够在实际项目中应用这些高级特性。

## JSON 字段

### 创建 JSON 字段

```sql
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    attributes JSON,
    metadata JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 插入 JSON 数据

```php
<?php
declare(strict_types=1);

// 方式一：直接插入 JSON 字符串
$attributes = json_encode([
    'color' => 'red',
    'size' => 'large',
    'weight' => 1.5,
]);

$stmt = $pdo->prepare("INSERT INTO products (name, attributes) VALUES (?, ?)");
$stmt->execute(['Product 1', $attributes]);

// 方式二：使用 JSON 函数
$stmt = $pdo->prepare("INSERT INTO products (name, attributes) VALUES (?, JSON_OBJECT('color', ?, 'size', ?))");
$stmt->execute(['Product 2', 'blue', 'medium']);
```

### 查询 JSON 数据

```php
<?php
declare(strict_types=1);

// 查询 JSON 字段
$stmt = $pdo->query("SELECT name, attributes->>'$.color' AS color FROM products");
$products = $stmt->fetchAll();

// 使用 JSON_EXTRACT
$stmt = $pdo->query("SELECT name, JSON_EXTRACT(attributes, '$.color') AS color FROM products");

// 条件查询
$stmt = $pdo->prepare("SELECT * FROM products WHERE attributes->>'$.color' = ?");
$stmt->execute(['red']);

// 检查 JSON 路径是否存在
$stmt = $pdo->query("SELECT * FROM products WHERE JSON_CONTAINS_PATH(attributes, 'one', '$.color')");
```

### JSON 函数

| 函数                    | 说明                           | 示例                                           |
| :---------------------- | :----------------------------- | :--------------------------------------------- |
| `JSON_EXTRACT`          | 提取 JSON 值                   | `JSON_EXTRACT(attributes, '$.color')`          |
| `JSON_UNQUOTE`          | 去除 JSON 字符串的引号         | `JSON_UNQUOTE(JSON_EXTRACT(attributes, '$.color'))` |
| `->`                    | JSON_EXTRACT 的简写            | `attributes->'$.color'`                       |
| `->>`                   | JSON_UNQUOTE(JSON_EXTRACT) 的简写 | `attributes->>'$.color'`                    |
| `JSON_CONTAINS`         | 检查 JSON 是否包含值           | `JSON_CONTAINS(attributes, '"red"', '$.color')` |
| `JSON_CONTAINS_PATH`    | 检查 JSON 路径是否存在         | `JSON_CONTAINS_PATH(attributes, 'one', '$.color')` |
| `JSON_KEYS`             | 获取 JSON 对象的键             | `JSON_KEYS(attributes)`                        |
| `JSON_ARRAY_APPEND`     | 向 JSON 数组追加元素           | `JSON_ARRAY_APPEND(attributes, '$.tags', 'new')` |
| `JSON_SET`              | 设置 JSON 值                   | `JSON_SET(attributes, '$.color', '"green"')`   |

### JSON 索引

```sql
-- 创建虚拟列并建立索引
ALTER TABLE products ADD COLUMN color VARCHAR(50) 
    GENERATED ALWAYS AS (attributes->>'$.color') VIRTUAL;

CREATE INDEX idx_color ON products(color);

-- 查询时使用索引
SELECT * FROM products WHERE color = 'red';
```

### JSON 更新

```php
<?php
declare(strict_types=1);

// 更新 JSON 字段
$stmt = $pdo->prepare("UPDATE products SET attributes = JSON_SET(attributes, '$.color', ?) WHERE id = ?");
$stmt->execute(['blue', 1]);

// 添加新的 JSON 键值对
$stmt = $pdo->prepare("UPDATE products SET attributes = JSON_SET(attributes, '$.new_key', ?) WHERE id = ?");
$stmt->execute(['new_value', 1]);

// 删除 JSON 键
$stmt = $pdo->prepare("UPDATE products SET attributes = JSON_REMOVE(attributes, '$.old_key') WHERE id = ?");
$stmt->execute([1]);
```

## 窗口函数（Window Functions）

### 什么是窗口函数

- 对查询结果集的每一行执行计算。
- 不改变结果集的行数，只添加计算列。

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

### 分组窗口函数

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

### 累计统计

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

### 窗口函数在 PHP 中使用

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

## 全文索引

### 创建全文索引

```sql
-- 创建全文索引
CREATE TABLE articles (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255),
    content TEXT,
    FULLTEXT INDEX ft_content (title, content)
) ENGINE=InnoDB;
```

### 全文搜索

```sql
-- 自然语言模式（默认）
SELECT * FROM articles 
WHERE MATCH(title, content) AGAINST('PHP MySQL' IN NATURAL LANGUAGE MODE);

-- 布尔模式
SELECT * FROM articles 
WHERE MATCH(title, content) AGAINST('+PHP -Java' IN BOOLEAN MODE);

-- 查询扩展模式
SELECT * FROM articles 
WHERE MATCH(title, content) AGAINST('PHP' WITH QUERY EXPANSION);
```

### 全文索引 vs LIKE

| 特性           | 全文索引                    | LIKE                      |
| :------------- | :-------------------------- | :------------------------ |
| 性能           | 快（使用索引）              | 慢（全表扫描）            |
| 匹配方式       | 词匹配                      | 字符串匹配                |
| 相关性排序     | 支持                        | 不支持                    |
| 最小词长度     | 有要求（默认 3 或 4）       | 无要求                    |
| 停用词         | 忽略常见词                  | 不过滤                   |
| 适用场景       | 大文本搜索                  | 简单字符串匹配           |

### PHP 中使用全文搜索

```php
<?php
declare(strict_types=1);

// 全文搜索
$stmt = $pdo->prepare("
    SELECT 
        id,
        title,
        MATCH(title, content) AGAINST(? IN NATURAL LANGUAGE MODE) AS relevance
    FROM articles
    WHERE MATCH(title, content) AGAINST(? IN NATURAL LANGUAGE MODE)
    ORDER BY relevance DESC
");
$stmt->execute(['PHP MySQL', 'PHP MySQL']);
$articles = $stmt->fetchAll();

// 布尔模式搜索
$stmt = $pdo->prepare("
    SELECT * FROM articles
    WHERE MATCH(title, content) AGAINST(? IN BOOLEAN MODE)
");
$stmt->execute(['+PHP -Java']);  // 必须包含 PHP，不能包含 Java
$articles = $stmt->fetchAll();
```

## 其他高级特性

### 生成列（Generated Columns）

```sql
-- 虚拟列（不存储）
CREATE TABLE products (
    id INT PRIMARY KEY,
    price DECIMAL(10,2),
    tax_rate DECIMAL(5,2),
    price_with_tax DECIMAL(10,2) GENERATED ALWAYS AS (price * (1 + tax_rate)) VIRTUAL
);

-- 存储列（存储值）
CREATE TABLE products (
    id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    full_name VARCHAR(100) GENERATED ALWAYS AS (CONCAT(first_name, ' ', last_name)) STORED
);
```

### 公用表表达式（CTE）

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

### JSON 表函数

```sql
-- 将 JSON 数组转换为表
SELECT 
    id,
    jt.name,
    jt.value
FROM products,
JSON_TABLE(
    attributes,
    '$[*]' COLUMNS (
        name VARCHAR(50) PATH '$.name',
        value VARCHAR(50) PATH '$.value'
    )
) AS jt;
```

## 最佳实践

### 1. JSON 字段使用建议

- 适合存储半结构化数据。
- 不适合频繁查询的字段（考虑提取为普通列）。
- 使用虚拟列和索引优化查询性能。

### 2. 窗口函数使用

- 适合排名、累计统计等场景。
- 性能优于子查询。
- 注意分区和排序的性能影响。

### 3. 全文索引使用

- 适合大文本搜索。
- 注意最小词长度限制。
- 使用布尔模式实现复杂搜索。

## 练习

1. 设计一个产品表，使用 JSON 字段存储可变属性，并创建虚拟列索引优化查询。

2. 实现一个文章搜索功能，使用全文索引支持关键词搜索和相关性排序。

3. 编写一个销售报表查询，使用窗口函数计算累计销售额和移动平均。

4. 创建一个排行榜功能，使用窗口函数实现分类排名和总排名。

5. 设计一个配置表，使用 JSON 字段存储配置项，并实现配置的增删改查。

6. 实现一个数据分析查询，使用 CTE 和窗口函数进行复杂的数据分析。
