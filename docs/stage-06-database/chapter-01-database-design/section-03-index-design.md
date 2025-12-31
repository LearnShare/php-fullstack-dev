# 6.1.3 索引设计

## 概述

索引是提高数据库查询性能的关键。本节介绍索引的概念、类型、设计原则等，帮助零基础学员掌握索引设计技术。

**章节类型**：配置性章节

**主要内容**：
- 索引概念
- 索引类型（B-Tree、Hash、全文索引）
- 索引设计原则
- 单列索引和复合索引
- 索引优化
- 索引维护
- 完整示例

---

## 索引概念

### 什么是索引

索引（Index）是数据库中用于快速查找数据的数据结构，类似于书籍的目录。索引包含表中一列或多列的值，以及指向表中对应行的指针。

**索引的类比**：

- 书籍目录：帮助快速找到章节位置
- 字典索引：帮助快速找到单词位置
- 数据库索引：帮助快速找到数据行

### 索引的作用

索引的主要作用包括：

1. **提高查询速度**：通过索引快速定位数据，避免全表扫描
2. **加速排序**：索引已排序，可以加速 ORDER BY 操作
3. **加速连接**：外键索引可以加速表连接操作
4. **保证唯一性**：唯一索引保证字段值的唯一性

**示例**：

```sql
-- 没有索引：需要全表扫描
SELECT * FROM users WHERE email = 'user@example.com';
-- 执行计划：全表扫描，扫描 10000 行

-- 有索引：使用索引查找
CREATE INDEX idx_email ON users(email);
SELECT * FROM users WHERE email = 'user@example.com';
-- 执行计划：使用索引，只扫描 1 行
```

### 索引的代价

索引虽然能提高查询性能，但也有代价：

1. **存储空间**：索引需要额外的存储空间
2. **维护成本**：插入、更新、删除操作需要维护索引
3. **创建时间**：创建索引需要时间
4. **内存占用**：索引会占用内存空间

**权衡**：

- **读多写少**：索引收益大于代价，适合创建索引
- **写多读少**：索引代价大于收益，谨慎创建索引
- **平衡场景**：根据实际业务场景权衡

---

## 索引类型

### B-Tree 索引

B-Tree（平衡树）索引是 MySQL 默认的索引类型，适用于大多数场景。

**B-Tree 索引的特点**：

- 支持范围查询：`WHERE age > 18 AND age < 65`
- 支持排序：`ORDER BY age`
- 支持前缀匹配：`WHERE name LIKE 'John%'`
- 支持等值查询：`WHERE id = 1`

**示例**：

```sql
-- 创建 B-Tree 索引
CREATE INDEX idx_age ON users(age);

-- 使用索引的查询
SELECT * FROM users WHERE age > 18 AND age < 65;  -- 范围查询
SELECT * FROM users ORDER BY age;  -- 排序
SELECT * FROM users WHERE name LIKE 'John%';  -- 前缀匹配
```

### Hash 索引

Hash 索引使用哈希表存储索引值，适用于等值查询。

**Hash 索引的特点**：

- 只支持等值查询：`WHERE id = 1`
- 不支持范围查询：`WHERE age > 18`
- 不支持排序：`ORDER BY age`
- 查询速度极快：O(1) 时间复杂度

**示例**：

```sql
-- 创建 Hash 索引（Memory 存储引擎）
CREATE TABLE users_memory (
    id INT PRIMARY KEY,
    username VARCHAR(50),
    INDEX USING HASH (username)
) ENGINE=MEMORY;

-- 使用 Hash 索引的查询
SELECT * FROM users_memory WHERE username = 'john';  -- 等值查询
```

### 全文索引

全文索引（FULLTEXT）用于全文搜索，支持文本内容的搜索。

**全文索引的特点**：

- 支持全文搜索：`MATCH ... AGAINST`
- 支持相关性排序
- 支持中文分词（需要配置）
- 只适用于 MyISAM 和 InnoDB 存储引擎

**示例**：

```sql
-- 创建全文索引
CREATE TABLE articles (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(200),
    content TEXT,
    FULLTEXT INDEX idx_content (content)
) ENGINE=InnoDB;

-- 使用全文索引搜索
SELECT * FROM articles
WHERE MATCH(content) AGAINST('PHP MySQL' IN NATURAL LANGUAGE MODE);
```

### 空间索引

空间索引（SPATIAL）用于地理空间数据，支持地理位置查询。

**空间索引的特点**：

- 支持地理位置查询
- 支持距离计算
- 支持范围查询
- 只适用于 MyISAM 存储引擎

**示例**：

```sql
-- 创建空间索引
CREATE TABLE locations (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    location POINT NOT NULL,
    SPATIAL INDEX idx_location (location)
) ENGINE=MyISAM;

-- 使用空间索引查询
SELECT * FROM locations
WHERE ST_Distance_Sphere(location, POINT(116.3974, 39.9093)) < 1000;
```

---

## 索引设计原则

### 选择性高的列

选择性（Selectivity）是指索引列中不同值的比例。选择性越高，索引效果越好。

**选择性计算**：

```
选择性 = 不同值的数量 / 总行数
```

**示例**：

```sql
-- 选择性高的列：email（每个值都不同）
CREATE INDEX idx_email ON users(email);
-- 选择性：10000 / 10000 = 1.0（完美）

-- 选择性低的列：gender（只有两个值）
CREATE INDEX idx_gender ON users(gender);
-- 选择性：2 / 10000 = 0.0002（很差）
```

**建议**：

- 选择性 > 0.1：适合创建索引
- 选择性 < 0.1：不适合创建索引

### 经常查询的列

为经常在 WHERE 子句中使用的列创建索引。

**示例**：

```sql
-- 经常查询的列
CREATE INDEX idx_user_id ON orders(user_id);
CREATE INDEX idx_status ON orders(status);
CREATE INDEX idx_created_at ON orders(created_at);

-- 使用索引的查询
SELECT * FROM orders WHERE user_id = 1;
SELECT * FROM orders WHERE status = 'pending';
SELECT * FROM orders WHERE created_at > '2024-01-01';
```

### 外键列

为外键列创建索引，可以加速表连接操作。

**示例**：

```sql
-- 外键列索引
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    product_id INT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (product_id) REFERENCES products(id),
    INDEX idx_user_id (user_id),  -- 外键索引
    INDEX idx_product_id (product_id)  -- 外键索引
);
```

### 排序和分组列

为经常用于 ORDER BY 和 GROUP BY 的列创建索引。

**示例**：

```sql
-- 排序和分组列索引
CREATE INDEX idx_created_at ON orders(created_at);
CREATE INDEX idx_status ON orders(status);

-- 使用索引的查询
SELECT * FROM orders ORDER BY created_at DESC;
SELECT status, COUNT(*) FROM orders GROUP BY status;
```

---

## 单列索引和复合索引

### 单列索引

单列索引只包含一个列，适用于单列查询。

**示例**：

```sql
-- 单列索引
CREATE INDEX idx_email ON users(email);
CREATE INDEX idx_username ON users(username);

-- 使用单列索引的查询
SELECT * FROM users WHERE email = 'user@example.com';
SELECT * FROM users WHERE username = 'john';
```

### 复合索引

复合索引包含多个列，适用于多列查询。

**示例**：

```sql
-- 复合索引
CREATE INDEX idx_user_status ON orders(user_id, status);

-- 使用复合索引的查询
SELECT * FROM orders WHERE user_id = 1 AND status = 'pending';
SELECT * FROM orders WHERE user_id = 1;  -- 可以使用索引的前缀
```

### 列顺序选择

复合索引的列顺序很重要，应遵循以下原则：

1. **最左前缀原则**：索引只能从左到右使用
2. **选择性高的列在前**：选择性高的列放在前面
3. **等值查询在前**：等值查询的列放在范围查询的列前面

**示例**：

```sql
-- 好的列顺序：选择性高的列在前
CREATE INDEX idx_user_status ON orders(user_id, status);
-- user_id 选择性高，status 选择性低

-- 不好的列顺序
CREATE INDEX idx_status_user ON orders(status, user_id);
-- status 选择性低，user_id 选择性高
```

### 最左前缀原则

最左前缀原则是指复合索引只能从左到右使用，不能跳过左边的列。

**示例**：

```sql
-- 复合索引
CREATE INDEX idx_user_status_created ON orders(user_id, status, created_at);

-- 可以使用索引的查询
SELECT * FROM orders WHERE user_id = 1;  -- 使用索引的第一列
SELECT * FROM orders WHERE user_id = 1 AND status = 'pending';  -- 使用索引的前两列
SELECT * FROM orders WHERE user_id = 1 AND status = 'pending' AND created_at > '2024-01-01';  -- 使用所有列

-- 不能使用索引的查询
SELECT * FROM orders WHERE status = 'pending';  -- 跳过了第一列
SELECT * FROM orders WHERE created_at > '2024-01-01';  -- 跳过了前两列
```

### 覆盖索引

覆盖索引（Covering Index）是指索引包含了查询所需的所有列，不需要回表查询。

**示例**：

```sql
-- 覆盖索引
CREATE INDEX idx_user_status ON orders(user_id, status, total_amount);

-- 使用覆盖索引的查询
SELECT user_id, status, total_amount FROM orders WHERE user_id = 1;
-- 不需要回表查询，直接从索引获取数据
```

**优势**：

- 减少 I/O 操作：不需要访问数据表
- 提高查询速度：减少磁盘读取
- 减少内存占用：索引通常比数据表小

---

## 索引优化

### 索引选择性分析

分析索引的选择性，确定是否需要创建索引。

**示例**：

```sql
-- 分析索引选择性
SELECT
    COUNT(DISTINCT email) / COUNT(*) AS email_selectivity,
    COUNT(DISTINCT gender) / COUNT(*) AS gender_selectivity
FROM users;

-- email_selectivity: 1.0（完美，适合创建索引）
-- gender_selectivity: 0.0002（很差，不适合创建索引）
```

### 索引使用分析

使用 EXPLAIN 分析索引的使用情况。

**示例**：

```sql
-- 分析索引使用
EXPLAIN SELECT * FROM users WHERE email = 'user@example.com';

-- 结果：
-- type: ref（使用索引）
-- key: idx_email（使用的索引）
-- rows: 1（扫描的行数）
```

### 索引优化建议

根据查询模式优化索引：

1. **分析慢查询日志**：找出慢查询，优化索引
2. **使用 EXPLAIN**：分析查询执行计划
3. **监控索引使用**：定期检查索引使用情况
4. **删除无用索引**：删除未使用的索引

**示例**：

```sql
-- 查看索引使用情况
SELECT
    object_schema,
    object_name,
    index_name,
    count_read,
    count_write
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE object_schema = 'mydb'
ORDER BY count_read DESC;

-- 删除未使用的索引
DROP INDEX idx_unused ON users;
```

---

## 索引维护

### 索引重建

定期重建索引，优化索引性能。

**示例**：

```sql
-- 重建索引
ALTER TABLE users DROP INDEX idx_email;
CREATE INDEX idx_email ON users(email);

-- 或者使用 OPTIMIZE TABLE
OPTIMIZE TABLE users;
```

### 索引统计信息更新

更新索引统计信息，帮助优化器选择最佳执行计划。

**示例**：

```sql
-- 更新索引统计信息
ANALYZE TABLE users;

-- 查看索引统计信息
SHOW INDEX FROM users;
```

### 索引监控

监控索引的使用情况和性能。

**示例**：

```sql
-- 查看索引大小
SELECT
    table_name,
    index_name,
    ROUND(stat_value * @@innodb_page_size / 1024 / 1024, 2) AS index_size_mb
FROM mysql.innodb_index_stats
WHERE database_name = 'mydb'
ORDER BY index_size_mb DESC;
```

---

## 完整示例

### 电商系统索引设计

**需求**：

- 用户表：按 email 查询，按 created_at 排序
- 订单表：按 user_id 查询，按 status 和 created_at 查询
- 商品表：按 name 搜索，按 price 排序

**索引设计**：

```sql
-- 用户表索引
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_email (email),  -- 按 email 查询
    INDEX idx_created_at (created_at)  -- 按 created_at 排序
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 订单表索引
CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    status VARCHAR(20) NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    INDEX idx_user_id (user_id),  -- 按 user_id 查询
    INDEX idx_status_created (status, created_at),  -- 按 status 和 created_at 查询
    INDEX idx_created_at (created_at)  -- 按 created_at 排序
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 商品表索引
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(200) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    stock INT NOT NULL DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FULLTEXT INDEX idx_name (name),  -- 全文搜索
    INDEX idx_price (price)  -- 按 price 排序
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**查询优化**：

```sql
-- 使用索引的查询
SELECT * FROM users WHERE email = 'user@example.com';  -- 使用 idx_email
SELECT * FROM users ORDER BY created_at DESC;  -- 使用 idx_created_at

SELECT * FROM orders WHERE user_id = 1;  -- 使用 idx_user_id
SELECT * FROM orders WHERE status = 'pending' ORDER BY created_at DESC;  -- 使用 idx_status_created

SELECT * FROM products WHERE MATCH(name) AGAINST('laptop');  -- 使用全文索引
SELECT * FROM products ORDER BY price;  -- 使用 idx_price
```

---

## 使用场景

### 查询优化

为经常查询的列创建索引，提高查询速度。

**示例**：

```sql
-- 为经常查询的列创建索引
CREATE INDEX idx_user_id ON orders(user_id);
CREATE INDEX idx_status ON orders(status);
```

### 排序优化

为经常用于排序的列创建索引，加速排序操作。

**示例**：

```sql
-- 为排序列创建索引
CREATE INDEX idx_created_at ON orders(created_at);
SELECT * FROM orders ORDER BY created_at DESC;
```

### 连接优化

为外键列创建索引，加速表连接操作。

**示例**：

```sql
-- 为外键列创建索引
CREATE INDEX idx_user_id ON orders(user_id);
SELECT o.*, u.username
FROM orders o
JOIN users u ON o.user_id = u.id;
```

---

## 注意事项

### 索引的维护成本

索引需要维护，插入、更新、删除操作会更新索引。

**影响**：

- 插入操作：需要更新索引
- 更新操作：如果更新索引列，需要更新索引
- 删除操作：需要更新索引

**建议**：

- 读多写少的表：适合创建多个索引
- 写多读少的表：谨慎创建索引

### 索引的选择性

选择性低的列不适合创建索引。

**示例**：

```sql
-- 选择性低的列：gender（只有两个值）
CREATE INDEX idx_gender ON users(gender);
-- 不建议：选择性太低，索引效果差
```

### 索引的数量

索引数量过多会影响写入性能。

**建议**：

- 每个表索引数量不超过 5-10 个
- 根据实际查询需求创建索引
- 定期审查和删除无用索引

### 索引的使用

不是所有查询都能使用索引。

**限制**：

- 函数操作：`WHERE UPPER(name) = 'JOHN'` 不能使用索引
- 类型转换：`WHERE id = '1'` 可能不能使用索引
- 前导模糊查询：`WHERE name LIKE '%john'` 不能使用索引

---

## 常见问题

### 如何选择索引列？

选择索引列的原则：

1. 选择性高的列
2. 经常查询的列
3. 外键列
4. 排序和分组列

### 复合索引如何设计？

复合索引设计原则：

1. 最左前缀原则：索引只能从左到右使用
2. 选择性高的列在前
3. 等值查询的列在范围查询的列前面

### 索引对性能的影响？

索引的影响：

- **查询性能**：提高查询速度
- **写入性能**：降低写入速度（需要维护索引）
- **存储空间**：占用额外存储空间

### 如何优化索引？

索引优化方法：

1. 分析慢查询日志
2. 使用 EXPLAIN 分析查询执行计划
3. 监控索引使用情况
4. 删除无用索引
5. 重建和优化索引

---

## 最佳实践

### 为经常查询的列创建索引

- 分析查询模式，识别经常查询的列
- 为这些列创建索引
- 定期审查索引使用情况

### 使用复合索引优化多列查询

- 为多列查询创建复合索引
- 遵循最左前缀原则
- 考虑覆盖索引

### 避免过度索引

- 不要为每个列都创建索引
- 根据实际查询需求创建索引
- 定期审查和删除无用索引

### 定期分析索引使用情况

- 使用 EXPLAIN 分析查询执行计划
- 监控索引使用情况
- 根据使用情况优化索引

---

## 练习任务

1. **索引设计**
   - 设计一个用户表
   - 分析查询需求
   - 为合适的列创建索引
   - 使用 EXPLAIN 验证索引使用

2. **复合索引设计**
   - 设计一个订单表
   - 分析多列查询需求
   - 设计复合索引
   - 验证最左前缀原则

3. **索引优化**
   - 分析现有表的索引
   - 识别慢查询
   - 优化索引设计
   - 验证性能提升

4. **索引维护**
   - 监控索引使用情况
   - 识别无用索引
   - 删除无用索引
   - 重建和优化索引

5. **全文索引应用**
   - 设计一个文章表
   - 创建全文索引
   - 实现全文搜索功能
   - 优化搜索性能

---

**相关章节**：

- [6.1.1 数据库设计原则](section-01-design-principles.md)
- [6.1.2 范式与反范式](section-02-normalization.md)
- [6.2 PDO 入门与高安全模式](../chapter-02-pdo/readme.md)
