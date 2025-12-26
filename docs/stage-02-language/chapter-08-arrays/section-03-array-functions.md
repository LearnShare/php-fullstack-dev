# 2.8.3 数组操作函数

## 概述

PHP 提供了丰富的数组操作函数。本节详细介绍查找函数、合并函数、高阶函数（`array_map()`、`array_filter()`、`array_reduce()`）、切片函数、PHP 8.4+ 新函数（`array_first()`、`array_last()`）等常用数组操作函数。

掌握这些数组操作函数对于高效处理数组数据非常重要。理解函数的参数、返回值和使用场景可以帮助编写更简洁、更可读的代码。

## 特性

- **函数丰富**：提供大量数组操作函数
- **函数式编程**：支持函数式编程风格（高阶函数）
- **性能优化**：选择合适的函数提高性能
- **版本支持**：部分新函数需要特定 PHP 版本

## 语法/定义

### 查找函数

#### array_key_exists() - 检查键是否存在

**语法**：`array_key_exists(string|int $key, array $array): bool`

**参数**：
- `$key`：要检查的键
- `$array`：要搜索的数组

**返回值**：如果键存在返回 `true`，否则返回 `false`

**特点**：
- 即使值为 `null` 也返回 `true`
- 与 `isset()` 的区别：`isset()` 对 `null` 值返回 `false`

#### in_array() - 检查值是否存在

**语法**：`in_array(mixed $needle, array $haystack, bool $strict = false): bool`

**参数**：
- `$needle`：要查找的值
- `$haystack`：要搜索的数组
- `$strict`：可选，是否使用严格比较（`===`），默认为 `false`

**返回值**：如果值存在返回 `true`，否则返回 `false`

#### array_search() - 查找值的键

**语法**：`array_search(mixed $needle, array $haystack, bool $strict = false): int|string|false`

**参数**：
- `$needle`：要查找的值
- `$haystack`：要搜索的数组
- `$strict`：可选，是否使用严格比较，默认为 `false`

**返回值**：返回第一个匹配的键，未找到返回 `false`

**注意**：返回值可能是 `0`（有效键），需要使用 `===` 判断

#### array_keys() - 获取所有键

**语法**：`array_keys(array $array, mixed $search_value = null, bool $strict = false): array`

**参数**：
- `$array`：要处理的数组
- `$search_value`：可选，如果指定，只返回该值的键
- `$strict`：可选，是否使用严格比较

**返回值**：返回包含所有键的数组

#### array_values() - 获取所有值

**语法**：`array_values(array $array): array`

**参数**：
- `$array`：要处理的数组

**返回值**：返回包含所有值的数组（重新索引）

### 合并函数

#### array_merge() - 合并数组

**语法**：`array_merge(array ...$arrays): array`

**参数**：
- `...$arrays`：要合并的数组（可变参数）

**返回值**：返回合并后的数组

**特点**：
- 数字键会重新编号
- 字符串键会覆盖前面的值

#### array_merge_recursive() - 递归合并数组

**语法**：`array_merge_recursive(array ...$arrays): array`

**参数**：
- `...$arrays`：要合并的数组

**返回值**：返回递归合并后的数组

**特点**：
- 相同键的值会合并为数组
- 递归处理嵌套数组

### 高阶函数

#### array_map() - 对每个元素应用函数

**语法**：`array_map(?callable $callback, array $array, array ...$arrays): array`

**参数**：
- `$callback`：要应用的函数（`null` 时返回数组的数组）
- `$array`：要处理的数组
- `...$arrays`：可选，额外的数组

**返回值**：返回处理后的数组

#### array_filter() - 过滤数组元素

**语法**：`array_filter(array $array, ?callable $callback = null, int $mode = 0): array`

**参数**：
- `$array`：要过滤的数组
- `$callback`：可选，过滤函数（返回 `true` 保留，`false` 移除）
- `$mode`：可选，`ARRAY_FILTER_USE_KEY` 或 `ARRAY_FILTER_USE_BOTH`

**返回值**：返回过滤后的数组（保留键）

#### array_reduce() - 归约数组为单个值

**语法**：`array_reduce(array $array, callable $callback, mixed $initial = null): mixed`

**参数**：
- `$array`：要归约的数组
- `$callback`：归约函数 `($carry, $item) => $result`
- `$initial`：可选，初始值

**返回值**：返回归约后的值

### 切片函数

#### array_slice() - 提取数组片段

**语法**：`array_slice(array $array, int $offset, ?int $length = null, bool $preserve_keys = false): array`

**参数**：
- `$array`：要处理的数组
- `$offset`：起始位置（负数表示从末尾开始）
- `$length`：可选，长度（负数表示到倒数第 N 个）
- `$preserve_keys`：可选，是否保留键，默认为 `false`

**返回值**：返回提取的数组片段

### PHP 8.4+ 新函数

#### array_first() - 获取第一个元素

**语法**：`array_first(array $array, ?callable $callback = null): mixed`

**参数**：
- `$array`：要处理的数组
- `$callback`：可选，过滤函数

**返回值**：返回第一个元素或第一个通过回调的元素

**要求**：PHP 8.4+

#### array_last() - 获取最后一个元素

**语法**：`array_last(array $array, ?callable $callback = null): mixed`

**参数**：
- `$array`：要处理的数组
- `$callback`：可选，过滤函数

**返回值**：返回最后一个元素或最后一个通过回调的元素

**要求**：PHP 8.4+

## 基本用法

### 示例 1：查找函数

```php
<?php
declare(strict_types=1);

$user = ['name' => 'John', 'age' => 25, 'email' => null];

// array_key_exists() - 检查键是否存在
var_dump(array_key_exists('name', $user));  // bool(true)
var_dump(array_key_exists('email', $user));  // bool(true)（即使值为 null）

// in_array() - 检查值是否存在
$fruits = ['apple', 'banana', 'orange'];
var_dump(in_array('banana', $fruits));  // bool(true)
var_dump(in_array('grape', $fruits));   // bool(false)

// array_search() - 查找值的键
$key = array_search('banana', $fruits);
echo "Key: {$key}\n";  // Key: 1

// array_keys() - 获取所有键
$keys = array_keys($user);
print_r($keys);  // Array ( [0] => name [1] => age [2] => email )

// array_values() - 获取所有值
$values = array_values($user);
print_r($values);  // Array ( [0] => John [1] => 25 [2] => )
```

### 示例 2：合并函数

```php
<?php
declare(strict_types=1);

// array_merge() - 合并数组
$arr1 = ['a' => 1, 'b' => 2];
$arr2 = ['c' => 3, 'd' => 4];
$merged = array_merge($arr1, $arr2);
print_r($merged);
// Array
// (
//     [a] => 1
//     [b] => 2
//     [c] => 3
//     [d] => 4
// )

// 数字键重新编号
$arr1 = [1, 2, 3];
$arr2 = [4, 5, 6];
$merged = array_merge($arr1, $arr2);
print_r($merged);  // Array ( [0] => 1 [1] => 2 [2] => 3 [3] => 4 [4] => 5 [5] => 6 )

// array_merge_recursive() - 递归合并
$arr1 = ['user' => ['name' => 'John'], 'age' => 25];
$arr2 = ['user' => ['email' => 'john@example.com'], 'age' => 30];
$merged = array_merge_recursive($arr1, $arr2);
print_r($merged);
// Array
// (
//     [user] => Array
//         (
//             [name] => John
//             [email] => john@example.com
//         )
//     [age] => Array
//         (
//             [0] => 25
//             [1] => 30
//         )
// )
```

### 示例 3：高阶函数

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

// array_map() - 对每个元素应用函数
$doubled = array_map(fn($n) => $n * 2, $numbers);
print_r($doubled);  // Array ( [0] => 2 [1] => 4 [2] => 6 [3] => 8 [4] => 10 )

// array_filter() - 过滤数组元素
$evens = array_filter($numbers, fn($n) => $n % 2 === 0);
print_r($evens);  // Array ( [1] => 2 [3] => 4 )

// array_reduce() - 归约数组
$sum = array_reduce($numbers, fn($carry, $item) => $carry + $item, 0);
echo "Sum: {$sum}\n";  // Sum: 15

$product = array_reduce($numbers, fn($carry, $item) => $carry * $item, 1);
echo "Product: {$product}\n";  // Product: 120
```

### 示例 4：切片函数

```php
<?php
declare(strict_types=1);

$arr = ['a', 'b', 'c', 'd', 'e'];

// 基本切片
$slice = array_slice($arr, 1, 3);
print_r($slice);  // Array ( [0] => b [1] => c [2] => d )

// 保留键
$slice = array_slice($arr, 1, 3, true);
print_r($slice);  // Array ( [1] => b [2] => c [3] => d )

// 负数偏移
$slice = array_slice($arr, -3);
print_r($slice);  // Array ( [0] => c [1] => d [2] => e )

// 负数长度
$slice = array_slice($arr, 1, -1);
print_r($slice);  // Array ( [0] => b [1] => c [2] => d )
```

## 完整代码示例

### 示例 1：数据处理工具

```php
<?php
declare(strict_types=1);

class ArrayProcessor
{
    public static function find(array $array, callable $callback): mixed
    {
        foreach ($array as $key => $value) {
            if ($callback($value, $key)) {
                return $value;
            }
        }
        return null;
    }
    
    public static function findAll(array $array, callable $callback): array
    {
        return array_filter($array, $callback);
    }
    
    public static function groupBy(array $array, callable $keyFunc): array
    {
        $result = [];
        foreach ($array as $item) {
            $key = $keyFunc($item);
            if (!isset($result[$key])) {
                $result[$key] = [];
            }
            $result[$key][] = $item;
        }
        return $result;
    }
    
    public static function pluck(array $array, string|int $key): array
    {
        return array_map(fn($item) => $item[$key] ?? null, $array);
    }
}

// 使用
$users = [
    ['name' => 'John', 'age' => 25, 'role' => 'admin'],
    ['name' => 'Jane', 'age' => 30, 'role' => 'user'],
    ['name' => 'Bob', 'age' => 25, 'role' => 'user']
];

// 查找
$admin = ArrayProcessor::find($users, fn($user) => $user['role'] === 'admin');
print_r($admin);

// 分组
$grouped = ArrayProcessor::groupBy($users, fn($user) => $user['age']);
print_r($grouped);

// 提取
$names = ArrayProcessor::pluck($users, 'name');
print_r($names);  // Array ( [0] => John [1] => Jane [2] => Bob )
```

### 示例 2：函数式编程风格

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 函数式编程：链式操作
$result = array_reduce(
    array_filter(
        array_map(fn($n) => $n * $n, $numbers),
        fn($n) => $n > 20
    ),
    fn($carry, $item) => $carry + $item,
    0
);

echo "Result: {$result}\n";  // Result: 355 (25+36+49+64+81+100)

// 更清晰的方式：分步处理
$squared = array_map(fn($n) => $n * $n, $numbers);
$filtered = array_filter($squared, fn($n) => $n > 20);
$sum = array_reduce($filtered, fn($carry, $item) => $carry + $item, 0);
echo "Sum: {$sum}\n";
```

### 示例 3：配置合并

```php
<?php
declare(strict_types=1);

class ConfigMerger
{
    public static function merge(array $defaults, array $overrides): array
    {
        return array_merge($defaults, $overrides);
    }
    
    public static function mergeRecursive(array $defaults, array $overrides): array
    {
        return array_merge_recursive($defaults, $overrides);
    }
    
    public static function deepMerge(array $defaults, array $overrides): array
    {
        foreach ($overrides as $key => $value) {
            if (isset($defaults[$key]) && is_array($defaults[$key]) && is_array($value)) {
                $defaults[$key] = self::deepMerge($defaults[$key], $value);
            } else {
                $defaults[$key] = $value;
            }
        }
        return $defaults;
    }
}

// 使用
$defaults = [
    'database' => ['host' => 'localhost', 'port' => 3306],
    'cache' => ['enabled' => true]
];

$overrides = [
    'database' => ['host' => 'remote.host'],
    'cache' => ['ttl' => 3600]
];

$merged = ConfigMerger::deepMerge($defaults, $overrides);
print_r($merged);
```

## 使用场景

### 查找函数

- **数据验证**：检查键或值是否存在
- **数据搜索**：在数组中搜索特定值
- **键值提取**：提取数组的键或值

### 合并函数

- **配置合并**：合并默认配置和用户配置
- **数据聚合**：聚合多个数据源
- **数组组合**：组合多个数组

### 高阶函数

- **数据转换**：使用 `array_map()` 转换数据
- **数据过滤**：使用 `array_filter()` 过滤数据
- **数据聚合**：使用 `array_reduce()` 聚合数据

### 切片函数

- **分页处理**：提取数组片段实现分页
- **数据截取**：截取数组的一部分
- **数组分割**：将数组分割为多个片段

## 注意事项

### 性能考虑

- **选择合适的函数**：根据场景选择合适的函数
- **避免过度使用**：避免过度使用高阶函数
- **批量处理**：批量处理数据时考虑性能

### 函数式编程

- **理解回调**：理解回调函数的使用
- **链式操作**：合理使用链式操作
- **可读性**：平衡函数式编程和可读性

### 版本要求

- **新函数**：注意新函数的版本要求
- **兼容性**：考虑代码兼容性
- **降级方案**：为新函数提供降级方案

## 常见问题

### 问题 1：array_key_exists vs isset

**症状**：检查键存在时结果不一致

**原因**：`isset()` 对 `null` 值返回 `false`

**错误示例**：

```php
<?php
declare(strict_types=1);

$arr = ['key' => null];

var_dump(isset($arr['key']));        // bool(false)
var_dump(array_key_exists('key', $arr));  // bool(true)
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$arr = ['key' => null];

// 如果需要检查键存在（包括 null 值），使用 array_key_exists()
if (array_key_exists('key', $arr)) {
    echo "Key exists\n";
}

// 如果需要检查键存在且不为 null，使用 isset()
if (isset($arr['key'])) {
    echo "Key exists and is not null\n";
}
```

### 问题 2：array_search() 返回 false

**症状**：`array_search()` 返回 `0` 时被误判为 `false`

**原因**：使用 `==` 而不是 `===` 比较

**错误示例**：

```php
<?php
declare(strict_types=1);

$arr = ['first', 'second'];
$key = array_search('first', $arr);
if ($key == false) {  // 错误：0 == false 为 true
    echo "Not found\n";
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$arr = ['first', 'second'];
$key = array_search('first', $arr);
if ($key === false) {  // 正确：使用严格比较
    echo "Not found\n";
} else {
    echo "Found at key: {$key}\n";
}
```

### 问题 3：高阶函数性能

**症状**：使用高阶函数后性能下降

**原因**：高阶函数有函数调用开销

**解决方法**：

```php
<?php
declare(strict_types=1);

$numbers = range(1, 1000);

// 方法1：使用高阶函数（可读性好，性能稍差）
$doubled = array_map(fn($n) => $n * 2, $numbers);

// 方法2：使用 foreach（性能更好）
$doubled = [];
foreach ($numbers as $n) {
    $doubled[] = $n * 2;
}

// 根据场景选择：小数组用高阶函数，大数组用循环
```

## 最佳实践

### 函数选择

- **理解区别**：理解不同函数的区别和适用场景
- **性能考虑**：在性能和可读性间平衡
- **版本兼容**：注意函数的版本要求

### 函数式编程

- **合理使用**：合理使用高阶函数
- **提高可读性**：使用高阶函数提高代码可读性
- **避免过度**：避免过度使用函数式编程

### 错误处理

- **检查返回值**：检查函数返回值
- **使用严格比较**：使用 `===` 比较返回值
- **处理边界情况**：处理空数组等边界情况

## 对比分析

### array_key_exists() vs isset()

| 特性 | array_key_exists() | isset() |
|:-----|:-------------------|:--------|
| null 值 | 返回 true | 返回 false |
| 性能 | 稍差 | 稍好 |
| 推荐度 | 检查键存在 | 检查键存在且不为 null |

**选择建议**：
- **检查键存在**：使用 `array_key_exists()`
- **检查键存在且不为 null**：使用 `isset()`

### 高阶函数 vs 循环

| 特性 | 高阶函数 | 循环 |
|:-----|:---------|:-----|
| 可读性 | 高 | 中 |
| 性能 | 稍差 | 稍好 |
| 函数式 | 是 | 否 |
| 推荐度 | 小数组 | 大数组 |

**选择建议**：
- **小数组、可读性优先**：使用高阶函数
- **大数组、性能优先**：使用循环

## 相关章节

- **2.8.1 数组基础**：了解数组基础
- **2.8.2 数组遍历**：了解数组遍历
- **2.10 函数与作用域**：了解函数和回调

## 练习任务

1. **查找函数练习**：
   - 练习使用各种查找函数
   - 理解函数的区别和适用场景
   - 测试各种查找场景
   - 注意返回值判断

2. **合并函数练习**：
   - 练习使用 `array_merge()` 和 `array_merge_recursive()`
   - 实现配置合并功能
   - 理解合并规则
   - 测试各种合并场景

3. **高阶函数练习**：
   - 练习使用 `array_map()`、`array_filter()`、`array_reduce()`
   - 实现函数式编程风格的数据处理
   - 理解回调函数的使用
   - 测试各种数据处理场景

4. **实际应用练习**：
   - 实现数据处理工具类
   - 实现配置管理功能
   - 实现数据转换功能
   - 测试各种应用场景

5. **综合练习**：
   - 创建一个数组操作工具库
   - 实现各种数组处理功能
   - 优化性能和可读性
   - 进行代码审查，确保正确性
