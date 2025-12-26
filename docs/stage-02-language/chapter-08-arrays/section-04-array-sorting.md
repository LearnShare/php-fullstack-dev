# 2.8.4 数组排序

## 概述

数组排序是常见的数组操作。本节详细介绍基于值的排序、保留键的排序、基于键的排序、自定义排序、多字段排序、排序标志及完整示例。

理解不同排序函数的特点和适用场景对于正确处理数组数据非常重要。选择合适的排序函数可以提高代码的可读性和性能。

## 特性

- **多种排序方式**：支持值排序、键排序、自定义排序
- **保留键**：某些函数保留键值关联
- **自定义比较**：支持自定义比较函数
- **排序标志**：支持不同的排序标志

## 语法/定义

### 基于值的排序

#### sort() - 升序排序（不保留键）

**语法**：`sort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：
- `$array`：要排序的数组（引用传递）
- `$flags`：可选，排序标志

**返回值**：成功返回 `true`，失败返回 `false`

**特点**：不保留键值关联，重新索引

#### rsort() - 降序排序（不保留键）

**语法**：`rsort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：与 `sort()` 相同

**返回值**：成功返回 `true`，失败返回 `false`

**特点**：不保留键值关联，重新索引

### 保留键的排序

#### asort() - 升序排序（保留键）

**语法**：`asort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：
- `$array`：要排序的数组（引用传递）
- `$flags`：可选，排序标志

**返回值**：成功返回 `true`，失败返回 `false`

**特点**：保留键值关联

#### arsort() - 降序排序（保留键）

**语法**：`arsort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：与 `asort()` 相同

**返回值**：成功返回 `true`，失败返回 `false`

**特点**：保留键值关联

### 基于键的排序

#### ksort() - 按键升序排序

**语法**：`ksort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：
- `$array`：要排序的数组（引用传递）
- `$flags`：可选，排序标志

**返回值**：成功返回 `true`，失败返回 `false`

**特点**：根据键排序，保留键值关联

#### krsort() - 按键降序排序

**语法**：`krsort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：与 `ksort()` 相同

**返回值**：成功返回 `true`，失败返回 `false`

**特点**：根据键排序，保留键值关联

### 自定义排序

#### usort() - 使用自定义函数排序（不保留键）

**语法**：`usort(array &$array, callable $callback): bool`

**参数**：
- `$array`：要排序的数组（引用传递）
- `$callback`：比较函数 `($a, $b) => int`

**返回值**：成功返回 `true`，失败返回 `false`

**特点**：不保留键值关联，使用自定义比较函数

#### uasort() - 使用自定义函数排序（保留键）

**语法**：`uasort(array &$array, callable $callback): bool`

**参数**：与 `usort()` 相同

**返回值**：成功返回 `true`，失败返回 `false`

**特点**：保留键值关联，使用自定义比较函数

#### uksort() - 使用自定义函数按键排序

**语法**：`uksort(array &$array, callable $callback): bool`

**参数**：
- `$array`：要排序的数组（引用传递）
- `$callback`：比较函数 `($a, $b) => int`（比较键）

**返回值**：成功返回 `true`，失败返回 `false`

**特点**：根据键排序，使用自定义比较函数

### 排序标志

- `SORT_REGULAR`：常规比较（默认）
- `SORT_NUMERIC`：数值比较
- `SORT_STRING`：字符串比较
- `SORT_LOCALE_STRING`：根据当前区域设置进行字符串比较
- `SORT_NATURAL`：自然排序（PHP 5.4+）
- `SORT_FLAG_CASE`：可以与 `SORT_STRING` 或 `SORT_NATURAL` 组合，不区分大小写

## 基本用法

### 示例 1：基于值的排序

```php
<?php
declare(strict_types=1);

// sort() - 升序排序
$numbers = [3, 1, 4, 1, 5, 9, 2, 6];
sort($numbers);
print_r($numbers);  // Array ( [0] => 1 [1] => 1 [2] => 2 [3] => 3 [4] => 4 [5] => 5 [6] => 6 [7] => 9 )

// rsort() - 降序排序
$numbers = [3, 1, 4, 1, 5, 9, 2, 6];
rsort($numbers);
print_r($numbers);  // Array ( [0] => 9 [1] => 6 [2] => 5 [3] => 4 [4] => 3 [5] => 2 [6] => 1 [7] => 1 )
```

### 示例 2：保留键的排序

```php
<?php
declare(strict_types=1);

// asort() - 升序排序（保留键）
$fruits = ['d' => 'lemon', 'a' => 'orange', 'b' => 'banana', 'c' => 'apple'];
asort($fruits);
print_r($fruits);
// Array
// (
//     [c] => apple
//     [b] => banana
//     [d] => lemon
//     [a] => orange
// )

// arsort() - 降序排序（保留键）
$fruits = ['d' => 'lemon', 'a' => 'orange', 'b' => 'banana', 'c' => 'apple'];
arsort($fruits);
print_r($fruits);
// Array
// (
//     [a] => orange
//     [d] => lemon
//     [b] => banana
//     [c] => apple
// )
```

### 示例 3：基于键的排序

```php
<?php
declare(strict_types=1);

// ksort() - 按键升序排序
$data = ['z' => 1, 'a' => 2, 'm' => 3, 'b' => 4];
ksort($data);
print_r($data);
// Array
// (
//     [a] => 2
//     [b] => 4
//     [m] => 3
//     [z] => 1
// )

// krsort() - 按键降序排序
$data = ['z' => 1, 'a' => 2, 'm' => 3, 'b' => 4];
krsort($data);
print_r($data);
// Array
// (
//     [z] => 1
//     [m] => 3
//     [b] => 4
//     [a] => 2
// )
```

### 示例 4：自定义排序

```php
<?php
declare(strict_types=1);

// usort() - 自定义排序
$users = [
    ['name' => 'John', 'age' => 25],
    ['name' => 'Jane', 'age' => 30],
    ['name' => 'Bob', 'age' => 20]
];

// 按年龄排序
usort($users, fn($a, $b) => $a['age'] <=> $b['age']);
print_r($users);
// Array
// (
//     [0] => Array ( [name] => Bob [age] => 20 )
//     [1] => Array ( [name] => John [age] => 25 )
//     [2] => Array ( [name] => Jane [age] => 30 )
// )

// uasort() - 自定义排序（保留键）
$products = [
    'p1' => ['price' => 100, 'name' => 'Product 1'],
    'p2' => ['price' => 50, 'name' => 'Product 2'],
    'p3' => ['price' => 200, 'name' => 'Product 3']
];

uasort($products, fn($a, $b) => $a['price'] <=> $b['price']);
print_r($products);
```

### 示例 5：排序标志

```php
<?php
declare(strict_types=1);

// SORT_NUMERIC - 数值比较
$numbers = ['10', '2', '1', '20'];
sort($numbers, SORT_NUMERIC);
print_r($numbers);  // Array ( [0] => 1 [1] => 2 [2] => 10 [3] => 20 )

// SORT_STRING - 字符串比较
$numbers = ['10', '2', '1', '20'];
sort($numbers, SORT_STRING);
print_r($numbers);  // Array ( [0] => 1 [1] => 10 [2] => 2 [3] => 20 )

// SORT_NATURAL - 自然排序
$files = ['file1.txt', 'file10.txt', 'file2.txt', 'file20.txt'];
sort($files, SORT_NATURAL);
print_r($files);  // Array ( [0] => file1.txt [1] => file2.txt [2] => file10.txt [3] => file20.txt )
```

## 完整代码示例

### 示例 1：多字段排序

```php
<?php
declare(strict_types=1);

function multiSort(array &$array, array $sortFields): void
{
    usort($array, function($a, $b) use ($sortFields) {
        foreach ($sortFields as $field => $direction) {
            $aValue = $a[$field] ?? null;
            $bValue = $b[$field] ?? null;
            
            if ($aValue === $bValue) {
                continue;
            }
            
            $result = $aValue <=> $bValue;
            return $direction === 'desc' ? -$result : $result;
        }
        return 0;
    });
}

// 使用
$users = [
    ['name' => 'John', 'age' => 25, 'score' => 80],
    ['name' => 'Jane', 'age' => 30, 'score' => 90],
    ['name' => 'Bob', 'age' => 25, 'score' => 85]
];

// 先按年龄升序，再按分数降序
multiSort($users, ['age' => 'asc', 'score' => 'desc']);
print_r($users);
```

### 示例 2：排序工具类

```php
<?php
declare(strict_types=1);

class ArraySorter
{
    public static function sortBy(array &$array, string|callable $key, string $direction = 'asc'): void
    {
        if (is_string($key)) {
            $callback = fn($a, $b) => ($a[$key] ?? null) <=> ($b[$key] ?? null);
        } else {
            $callback = $key;
        }
        
        if ($direction === 'desc') {
            $callback = fn($a, $b) => -$callback($a, $b);
        }
        
        usort($array, $callback);
    }
    
    public static function sortByKey(array &$array, string $direction = 'asc'): void
    {
        if ($direction === 'asc') {
            ksort($array);
        } else {
            krsort($array);
        }
    }
    
    public static function naturalSort(array &$array): void
    {
        sort($array, SORT_NATURAL);
    }
}

// 使用
$products = [
    ['name' => 'Product C', 'price' => 100],
    ['name' => 'Product A', 'price' => 200],
    ['name' => 'Product B', 'price' => 150]
];

ArraySorter::sortBy($products, 'price', 'desc');
print_r($products);
```

## 使用场景

### 数据展示

- **列表排序**：按不同字段排序列表
- **表格排序**：实现表格排序功能
- **搜索结果**：对搜索结果排序

### 数据分析

- **统计分析**：排序后进行分析
- **数据筛选**：排序后筛选数据
- **数据分组**：排序后分组数据

### 算法实现

- **排序算法**：实现各种排序算法
- **查找算法**：排序后使用二分查找
- **优化算法**：使用排序优化算法

## 注意事项

### 修改原数组

- **引用传递**：排序函数直接修改原数组
- **保留副本**：需要保留原数组时先复制
- **返回值**：排序函数返回布尔值，不是排序后的数组

### 保留键

- **理解区别**：理解保留键和不保留键的区别
- **选择函数**：根据需求选择合适的函数
- **键的重要性**：键重要时使用保留键的函数

### 自定义比较函数

- **返回值**：返回负数、0、正数表示小于、等于、大于
- **使用太空船运算符**：使用 `<=>` 简化比较
- **性能考虑**：自定义比较函数有性能开销

## 常见问题

### 问题 1：排序后键丢失

**症状**：排序后数组键被重新编号

**原因**：使用了不保留键的排序函数

**错误示例**：

```php
<?php
declare(strict_types=1);

$fruits = ['d' => 'lemon', 'a' => 'orange', 'b' => 'banana'];
sort($fruits);  // 键丢失
print_r($fruits);
// Array
// (
//     [0] => banana
//     [1] => lemon
//     [2] => orange
// )
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$fruits = ['d' => 'lemon', 'a' => 'orange', 'b' => 'banana'];
asort($fruits);  // 保留键
print_r($fruits);
// Array
// (
//     [b] => banana
//     [d] => lemon
//     [a] => orange
// )
```

### 问题 2：自定义排序函数返回值错误

**症状**：排序结果不符合预期

**原因**：自定义比较函数返回值不正确

**错误示例**：

```php
<?php
declare(strict_types=1);

$numbers = [3, 1, 4, 1, 5];
usort($numbers, fn($a, $b) => $a > $b);  // 错误：返回布尔值
print_r($numbers);  // 结果可能不符合预期
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$numbers = [3, 1, 4, 1, 5];

// 方法1：使用太空船运算符
usort($numbers, fn($a, $b) => $a <=> $b);
print_r($numbers);

// 方法2：返回整数
usort($numbers, function($a, $b) {
    if ($a < $b) return -1;
    if ($a > $b) return 1;
    return 0;
});
```

## 最佳实践

### 函数选择

- **需要保留键**：使用 `asort()`、`arsort()`、`uasort()`
- **不需要保留键**：使用 `sort()`、`rsort()`、`usort()`
- **按键排序**：使用 `ksort()`、`krsort()`、`uksort()`

### 自定义排序

- **使用太空船运算符**：使用 `<=>` 简化比较
- **多字段排序**：实现多字段排序函数
- **性能考虑**：注意自定义比较函数的性能

### 排序标志

- **数值排序**：使用 `SORT_NUMERIC`
- **字符串排序**：使用 `SORT_STRING`
- **自然排序**：使用 `SORT_NATURAL`

## 对比分析

### sort() vs asort()

| 特性 | sort() | asort() |
|:-----|:-------|:--------|
| 保留键 | 否 | 是 |
| 重新索引 | 是 | 否 |
| 适用场景 | 索引数组 | 关联数组 |
| 推荐度 | 索引数组 | 关联数组 |

**选择建议**：
- **索引数组**：使用 `sort()`
- **关联数组**：使用 `asort()`

### 排序函数性能

| 函数 | 性能 | 适用场景 |
|:-----|:-----|:---------|
| `sort()` | 最好 | 简单排序 |
| `usort()` | 稍差 | 自定义排序 |
| `uasort()` | 稍差 | 自定义排序+保留键 |

**选择建议**：
- **简单排序**：使用内置排序函数
- **复杂排序**：使用自定义排序函数

## 相关章节

- **2.8.1 数组基础**：了解数组基础
- **2.8.2 数组遍历**：了解数组遍历
- **2.6.5 match 表达式**：了解表达式

## 练习任务

1. **基本排序练习**：
   - 练习使用各种排序函数
   - 理解保留键和不保留键的区别
   - 测试各种排序场景
   - 观察排序结果

2. **自定义排序练习**：
   - 实现自定义比较函数
   - 实现多字段排序
   - 使用太空船运算符
   - 测试各种排序场景

3. **排序标志练习**：
   - 练习使用不同的排序标志
   - 理解数值排序和字符串排序的区别
   - 测试自然排序
   - 测试各种标志组合

4. **实际应用练习**：
   - 实现表格排序功能
   - 实现列表排序功能
   - 实现多字段排序工具
   - 测试各种应用场景

5. **综合练习**：
   - 创建一个排序工具类
   - 实现各种排序功能
   - 优化性能和可读性
   - 进行代码审查，确保正确性
