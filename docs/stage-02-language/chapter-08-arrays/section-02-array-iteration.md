# 2.8.2 数组遍历

## 概述

数组遍历是处理数组数据的基本操作。本节详细介绍 `foreach` 循环、引用遍历、嵌套数组遍历、数组指针操作（`reset()`、`next()`、`prev()`、`end()`、`current()`、`key()`）及注意事项。

`foreach` 是 PHP 中最常用、最高效的数组遍历方式。理解 `foreach` 的行为、引用遍历的陷阱、以及数组指针操作可以帮助编写更可靠的代码。

## 特性

- **foreach 高效**：`foreach` 是最高效的数组遍历方式
- **引用遍历**：可以修改数组元素
- **指针操作**：可以手动控制遍历位置
- **嵌套支持**：支持嵌套数组遍历

## 语法/定义

### foreach 循环

**值遍历语法**：`foreach ($array as $value) { ... }`

**键值遍历语法**：`foreach ($array as $key => $value) { ... }`

**引用遍历语法**：`foreach ($array as &$value) { ... }`

**特点**：
- 自动处理数组指针
- 遍历结束后指针指向数组末尾
- 引用遍历后必须 `unset()` 引用变量

### 数组指针操作函数

#### reset() - 重置指针到第一个元素

**语法**：`reset(array|object $array): mixed`

**参数**：
- `$array`：要操作的数组

**返回值**：返回数组的第一个元素值，如果数组为空返回 `false`

#### next() - 移动指针到下一个元素

**语法**：`next(array|object $array): mixed`

**参数**：
- `$array`：要操作的数组

**返回值**：返回下一个元素值，如果没有下一个元素返回 `false`

#### prev() - 移动指针到上一个元素

**语法**：`prev(array|object $array): mixed`

**参数**：
- `$array`：要操作的数组

**返回值**：返回上一个元素值，如果没有上一个元素返回 `false`

#### end() - 移动指针到最后一个元素

**语法**：`end(array|object $array): mixed`

**参数**：
- `$array`：要操作的数组

**返回值**：返回最后一个元素值，如果数组为空返回 `false`

#### current() - 获取当前元素值

**语法**：`current(array|object $array): mixed`

**参数**：
- `$array`：要操作的数组

**返回值**：返回当前元素值，如果指针超出范围返回 `false`

#### key() - 获取当前元素键

**语法**：`key(array|object $array): int|string|null`

**参数**：
- `$array`：要操作的数组

**返回值**：返回当前元素键，如果指针超出范围返回 `null`

## 基本用法

### 示例 1：foreach 值遍历

```php
<?php
declare(strict_types=1);

$fruits = ['apple', 'banana', 'orange'];

// 值遍历
foreach ($fruits as $fruit) {
    echo $fruit . "\n";
}
// apple
// banana
// orange
```

### 示例 2：foreach 键值遍历

```php
<?php
declare(strict_types=1);

$user = ['name' => 'John', 'age' => 25, 'email' => 'john@example.com'];

// 键值遍历
foreach ($user as $key => $value) {
    echo "{$key}: {$value}\n";
}
// name: John
// age: 25
// email: john@example.com
```

### 示例 3：引用遍历

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

// 引用遍历：修改数组元素
foreach ($numbers as &$number) {
    $number *= 2;  // 每个元素乘以 2
}
unset($number);  // 重要：解除引用

print_r($numbers);
// Array
// (
//     [0] => 2
//     [1] => 4
//     [2] => 6
//     [3] => 8
//     [4] => 10
// )
```

### 示例 4：嵌套数组遍历

```php
<?php
declare(strict_types=1);

$users = [
    ['name' => 'John', 'age' => 25],
    ['name' => 'Jane', 'age' => 30],
    ['name' => 'Bob', 'age' => 35]
];

// 嵌套遍历
foreach ($users as $index => $user) {
    echo "User {$index}:\n";
    foreach ($user as $key => $value) {
        echo "  {$key}: {$value}\n";
    }
}
// User 0:
//   name: John
//   age: 25
// User 1:
//   name: Jane
//   age: 30
// User 2:
//   name: Bob
//   age: 35
```

### 示例 5：数组指针操作

```php
<?php
declare(strict_types=1);

$arr = ['first', 'second', 'third', 'fourth'];

// 重置指针到第一个元素
reset($arr);
echo "First: " . current($arr) . "\n";  // First: first
echo "Key: " . key($arr) . "\n";        // Key: 0

// 移动到下一个元素
next($arr);
echo "Next: " . current($arr) . "\n";   // Next: second
echo "Key: " . key($arr) . "\n";        // Key: 1

// 移动到上一个元素
prev($arr);
echo "Prev: " . current($arr) . "\n";   // Prev: first

// 移动到最后一个元素
end($arr);
echo "Last: " . current($arr) . "\n";   // Last: fourth
echo "Key: " . key($arr) . "\n";        // Key: 3
```

## 完整代码示例

### 示例 1：数组处理工具

```php
<?php
declare(strict_types=1);

class ArrayIterator
{
    public static function map(array $array, callable $callback): array
    {
        $result = [];
        foreach ($array as $key => $value) {
            $result[$key] = $callback($value, $key);
        }
        return $result;
    }
    
    public static function filter(array $array, callable $callback): array
    {
        $result = [];
        foreach ($array as $key => $value) {
            if ($callback($value, $key)) {
                $result[$key] = $value;
            }
        }
        return $result;
    }
    
    public static function reduce(array $array, callable $callback, mixed $initial = null): mixed
    {
        $accumulator = $initial;
        foreach ($array as $key => $value) {
            $accumulator = $callback($accumulator, $value, $key);
        }
        return $accumulator;
    }
}

// 使用
$numbers = [1, 2, 3, 4, 5];

// map：每个元素乘以 2
$doubled = ArrayIterator::map($numbers, fn($n) => $n * 2);
print_r($doubled);  // [2, 4, 6, 8, 10]

// filter：过滤偶数
$evens = ArrayIterator::filter($numbers, fn($n) => $n % 2 === 0);
print_r($evens);  // [2, 4]

// reduce：求和
$sum = ArrayIterator::reduce($numbers, fn($acc, $n) => $acc + $n, 0);
echo "Sum: {$sum}\n";  // Sum: 15
```

### 示例 2：引用遍历陷阱

```php
<?php
declare(strict_types=1);

// 陷阱：引用遍历后未 unset
$arr = [1, 2, 3];
foreach ($arr as &$value) {
    $value *= 2;
}
// 未 unset($value)

// 后续使用 $value 可能导致意外结果
$value = 100;
print_r($arr);
// Array
// (
//     [0] => 2
//     [1] => 4
//     [2] => 100  // 意外：最后一个元素被修改
// )

// 正确方式
$arr = [1, 2, 3];
foreach ($arr as &$value) {
    $value *= 2;
}
unset($value);  // 解除引用

$value = 100;  // 不会影响数组
print_r($arr);
// Array
// (
//     [0] => 2
//     [1] => 4
//     [2] => 6
// )
```

### 示例 3：指针操作应用

```php
<?php
declare(strict_types=1);

function getFirstElement(array $array): mixed
{
    reset($array);
    return current($array);
}

function getLastElement(array $array): mixed
{
    end($array);
    return current($array);
}

function getNextElement(array $array, string|int $key): mixed
{
    reset($array);
    while (key($array) !== $key && key($array) !== null) {
        next($array);
    }
    next($array);
    return current($array);
}

// 使用
$arr = ['a' => 1, 'b' => 2, 'c' => 3];
echo "First: " . getFirstElement($arr) . "\n";  // First: 1
echo "Last: " . getLastElement($arr) . "\n";    // Last: 3
echo "Next after 'a': " . getNextElement($arr, 'a') . "\n";  // Next after 'a': 2
```

## 使用场景

### 数据处理

- **数据转换**：遍历数组转换数据格式
- **数据筛选**：筛选符合条件的元素
- **数据聚合**：聚合数组数据

### 数据展示

- **列表展示**：遍历数组展示列表
- **表格生成**：生成 HTML 表格
- **模板渲染**：渲染模板数据

### 算法实现

- **搜索算法**：在数组中搜索元素
- **排序算法**：实现排序算法
- **遍历算法**：实现各种遍历算法

## 注意事项

### 引用遍历

- **必须 unset**：引用遍历后必须 `unset()` 引用变量
- **避免陷阱**：避免引用遍历的常见陷阱
- **理解机制**：理解引用遍历的工作原理

### 性能考虑

- **foreach 最快**：`foreach` 是最高效的遍历方式
- **避免修改结构**：遍历时避免修改数组结构
- **使用引用**：需要修改元素时使用引用遍历

### 指针操作

- **影响 foreach**：指针操作可能影响 `foreach` 的行为
- **重置指针**：使用 `reset()` 重置指针
- **检查有效性**：检查指针操作的有效性

## 常见问题

### 问题 1：引用遍历后未 unset

**症状**：后续代码意外修改数组元素

**原因**：引用变量在循环后仍然存在

**错误示例**：

```php
<?php
declare(strict_types=1);

$arr = [1, 2, 3];
foreach ($arr as &$value) {
    $value *= 2;
}
// 未 unset($value)

$value = 100;  // 意外修改最后一个元素
print_r($arr);  // [2, 4, 100]
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$arr = [1, 2, 3];
foreach ($arr as &$value) {
    $value *= 2;
}
unset($value);  // 解除引用

$value = 100;  // 不会影响数组
print_r($arr);  // [2, 4, 6]
```

### 问题 2：遍历时修改数组结构

**症状**：遍历结果不符合预期

**原因**：遍历时修改数组结构可能导致问题

**错误示例**：

```php
<?php
declare(strict_types=1);

$arr = [1, 2, 3, 4, 5];
foreach ($arr as $key => $value) {
    if ($value % 2 === 0) {
        unset($arr[$key]);  // 遍历时删除元素
    }
}
print_r($arr);  // 可能不符合预期
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$arr = [1, 2, 3, 4, 5];

// 方法1：遍历数组副本
foreach ($arr as $key => $value) {
    if ($value % 2 === 0) {
        unset($arr[$key]);
    }
}

// 方法2：收集要删除的键，然后删除
$keysToRemove = [];
foreach ($arr as $key => $value) {
    if ($value % 2 === 0) {
        $keysToRemove[] = $key;
    }
}
foreach ($keysToRemove as $key) {
    unset($arr[$key]);
}

// 方法3：使用 array_filter()
$arr = array_filter($arr, fn($value) => $value % 2 !== 0);
```

### 问题 3：指针操作影响 foreach

**症状**：`foreach` 行为不符合预期

**原因**：指针操作影响了 `foreach` 的起始位置

**解决方法**：

```php
<?php
declare(strict_types=1);

$arr = [1, 2, 3, 4, 5];

// 指针操作
next($arr);
next($arr);

// foreach 不受影响（会重置指针）
foreach ($arr as $value) {
    echo $value . "\n";  // 从第一个元素开始
}
// 1
// 2
// 3
// 4
// 5
```

## 最佳实践

### foreach 使用

- **优先使用 foreach**：优先使用 `foreach` 遍历数组
- **引用遍历后 unset**：引用遍历后必须 `unset()` 引用变量
- **避免修改结构**：遍历时避免修改数组结构

### 性能优化

- **使用 foreach**：`foreach` 性能最好
- **避免不必要的操作**：避免在循环中进行不必要的操作
- **使用引用**：需要修改元素时使用引用遍历

### 代码可读性

- **使用有意义的变量名**：使用有意义的循环变量名
- **提取复杂逻辑**：将复杂逻辑提取到函数中
- **添加注释**：为复杂的遍历逻辑添加注释

## 对比分析

### foreach vs for 循环

| 特性 | foreach | for |
|:-----|:--------|:----|
| 性能 | 最好 | 稍差 |
| 可读性 | 高 | 中 |
| 适用场景 | 数组遍历 | 索引数组 |
| 推荐度 | 推荐 | 按需使用 |

**选择建议**：
- **数组遍历**：使用 `foreach`
- **索引数组**：可以使用 `for`，但 `foreach` 更推荐

### 值遍历 vs 引用遍历

| 特性 | 值遍历 | 引用遍历 |
|:-----|:-------|:---------|
| 修改元素 | 否 | 是 |
| 性能 | 稍好 | 稍差 |
| 陷阱 | 无 | 需要 unset |
| 推荐度 | 推荐（只读） | 按需使用（修改） |

**选择建议**：
- **只读遍历**：使用值遍历
- **需要修改**：使用引用遍历，记得 `unset()`

## 相关章节

- **2.8.1 数组基础**：了解数组基础
- **2.8.3 数组操作函数**：了解数组函数
- **2.9 控制结构**：了解循环结构

## 练习任务

1. **foreach 练习**：
   - 练习值遍历、键值遍历、引用遍历
   - 理解不同遍历方式的区别
   - 测试各种数组结构
   - 观察遍历行为

2. **引用遍历练习**：
   - 练习使用引用遍历修改数组
   - 理解 unset() 的重要性
   - 重现引用遍历陷阱
   - 测试各种场景

3. **嵌套遍历练习**：
   - 练习嵌套数组遍历
   - 实现多维数组处理
   - 测试各种嵌套结构
   - 优化遍历性能

4. **指针操作练习**：
   - 练习使用数组指针操作函数
   - 实现基于指针的遍历
   - 理解指针操作的影响
   - 测试各种场景

5. **综合练习**：
   - 创建一个数组遍历工具类
   - 实现各种遍历功能
   - 处理各种数组结构
   - 进行代码审查，确保正确性
