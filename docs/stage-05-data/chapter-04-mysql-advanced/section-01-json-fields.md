# 5.4.1 JSON 字段

## 概述

MySQL 5.7+ 支持原生 JSON 数据类型，可以高效存储和查询 JSON 数据。本节详细介绍 JSON 字段的创建、插入、查询、更新、索引，以及 JSON 函数的使用。

## 创建 JSON 字段

```sql
CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    attributes JSON,
    metadata JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 插入 JSON 数据

### 方式一：直接插入 JSON 字符串

```php
<?php
declare(strict_types=1);

$attributes = json_encode([
    'color' => 'red',
    'size' => 'large',
    'weight' => 1.5,
]);

$stmt = $pdo->prepare("INSERT INTO products (name, attributes) VALUES (?, ?)");
$stmt->execute(['Product 1', $attributes]);
```

### 方式二：使用 JSON 函数

```php
<?php
declare(strict_types=1);

$stmt = $pdo->prepare("INSERT INTO products (name, attributes) VALUES (?, JSON_OBJECT('color', ?, 'size', ?))");
$stmt->execute(['Product 2', 'blue', 'medium']);
```

## 查询 JSON 数据

### 基础查询

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

## JSON 函数

| 函数 | 说明 | 示例 |
| :--- | :--- | :--- |
| `JSON_EXTRACT` | 提取 JSON 值 | `JSON_EXTRACT(attributes, '$.color')` |
| `JSON_UNQUOTE` | 去除 JSON 字符串的引号 | `JSON_UNQUOTE(JSON_EXTRACT(attributes, '$.color'))` |
| `->` | JSON_EXTRACT 的简写 | `attributes->'$.color'` |
| `->>` | JSON_UNQUOTE(JSON_EXTRACT) 的简写 | `attributes->>'$.color'` |
| `JSON_CONTAINS` | 检查 JSON 是否包含值 | `JSON_CONTAINS(attributes, '"red"', '$.color')` |
| `JSON_CONTAINS_PATH` | 检查 JSON 路径是否存在 | `JSON_CONTAINS_PATH(attributes, 'one', '$.color')` |
| `JSON_KEYS` | 获取 JSON 对象的键 | `JSON_KEYS(attributes)` |
| `JSON_ARRAY_APPEND` | 向 JSON 数组追加元素 | `JSON_ARRAY_APPEND(attributes, '$.tags', 'new')` |
| `JSON_SET` | 设置 JSON 值 | `JSON_SET(attributes, '$.color', '"green"')` |

## JSON 索引

```sql
-- 创建虚拟列并建立索引
ALTER TABLE products ADD COLUMN color VARCHAR(50) 
    GENERATED ALWAYS AS (attributes->>'$.color') VIRTUAL;

CREATE INDEX idx_color ON products(color);

-- 查询时使用索引
SELECT * FROM products WHERE color = 'red';
```

## JSON 更新

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

## 完整示例

```php
<?php
declare(strict_types=1);

class ProductService
{
    private PDO $pdo;

    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }

    public function createProduct(string $name, array $attributes): int
    {
        $stmt = $this->pdo->prepare("INSERT INTO products (name, attributes) VALUES (?, ?)");
        $stmt->execute([$name, json_encode($attributes)]);
        return (int) $this->pdo->lastInsertId();
    }

    public function findProductsByColor(string $color): array
    {
        $stmt = $this->pdo->prepare("SELECT * FROM products WHERE attributes->>'$.color' = ?");
        $stmt->execute([$color]);
        return $stmt->fetchAll();
    }

    public function updateProductAttributes(int $id, array $attributes): void
    {
        $stmt = $this->pdo->prepare("UPDATE products SET attributes = ? WHERE id = ?");
        $stmt->execute([json_encode($attributes), $id]);
    }
}
```

## 注意事项

1. **JSON 验证**：MySQL 会自动验证 JSON 格式
2. **索引优化**：使用虚拟列创建索引提升查询性能
3. **性能考虑**：JSON 查询可能比普通字段慢
4. **数据一致性**：确保 JSON 结构的一致性

## 练习

1. 创建一个包含 JSON 字段的表，实现 JSON 数据的增删改查。

2. 实现 JSON 字段的索引优化，提升查询性能。

3. 编写一个 JSON 数据验证函数，确保数据格式正确。

4. 实现 JSON 字段的复杂查询，如数组查询、嵌套查询。
