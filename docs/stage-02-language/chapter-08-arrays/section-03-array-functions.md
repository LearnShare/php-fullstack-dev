# 2.8.3 数组操作函数

## 概述

PHP 提供了丰富的数组操作函数，包括查找、合并、高阶函数、切片等。掌握这些函数可以大大提高开发效率。

## 查找函数

### array_key_exists() - 检查键是否存在

**语法**：`array_key_exists(string|int $key, array $array): bool`

**参数**：
- `$key`：要检查的键名
- `$array`：要检查的数组

**返回值**：如果键存在返回 `true`，否则返回 `false`。注意：即使值为 `null` 也会返回 `true`。

```php
<?php
declare(strict_types=1);

$user = ['name' => 'Alice', 'age' => 25];

var_dump(array_key_exists('name', $user));  // bool(true)
var_dump(array_key_exists('email', $user)); // bool(false)

// 与 isset() 的区别
$user['email'] = null;
var_dump(array_key_exists('email', $user)); // bool(true)
var_dump(isset($user['email']));             // bool(false)
```

### in_array() - 检查值是否存在

**语法**：`in_array(mixed $needle, array $haystack, bool $strict = false): bool`

**参数**：
- `$needle`：要查找的值
- `$haystack`：要搜索的数组
- `$strict`：可选，是否使用严格比较（`===`），默认为 `false`

**返回值**：如果值存在返回 `true`，否则返回 `false`。

```php
<?php
declare(strict_types=1);

$fruits = ['apple', 'banana', 'orange'];

var_dump(in_array('apple', $fruits));      // bool(true)
var_dump(in_array('grape', $fruits));      // bool(false)

// 严格模式
var_dump(in_array('1', [1, 2, 3]));        // bool(true) - 宽松比较
var_dump(in_array('1', [1, 2, 3], true));  // bool(false) - 严格比较
```

### array_search() - 查找值并返回键

**语法**：`array_search(mixed $needle, array $haystack, bool $strict = false): int|string|false`

**参数**：
- `$needle`：要查找的值
- `$haystack`：要搜索的数组
- `$strict`：可选，是否使用严格比较（`===`），默认为 `false`

**返回值**：找到返回对应的键（可能是整数或字符串），未找到返回 `false`。注意：使用 `!== false` 检查，因为键可能是 `0`。

```php
<?php
declare(strict_types=1);

$fruits = ['a' => 'apple', 'b' => 'banana', 'c' => 'orange'];

$key = array_search('banana', $fruits);
echo $key . "\n";  // b

// 未找到返回 false
$key = array_search('grape', $fruits);
var_dump($key);  // bool(false)

// 注意：使用 !== 检查，因为键可能是 0
if (array_search('banana', $fruits) !== false) {
    echo "Found\n";
}
```

### array_keys() - 获取所有键

**语法**：`array_keys(array $array, mixed $search_value = null, bool $strict = false): array`

**参数**：
- `$array`：要处理的数组
- `$search_value`：可选，如果指定，只返回包含该值的键
- `$strict`：可选，是否使用严格比较，默认为 `false`

**返回值**：返回包含所有键的数组。

```php
<?php
declare(strict_types=1);

$user = ['name' => 'Alice', 'age' => 25, 'city' => 'Beijing'];

$keys = array_keys($user);
print_r($keys);  // ['name', 'age', 'city']

// 查找特定值的键
$arr = ['a' => 1, 'b' => 2, 'c' => 1];
$keys = array_keys($arr, 1);
print_r($keys);  // ['a', 'c']
```

### array_values() - 获取所有值

**语法**：`array_values(array $array): array`

**参数**：
- `$array`：要处理的数组

**返回值**：返回包含所有值的数组，键会被重新索引为从 0 开始的连续整数。

```php
<?php
declare(strict_types=1);

$user = ['name' => 'Alice', 'age' => 25, 'city' => 'Beijing'];

$values = array_values($user);
print_r($values);  // ['Alice', 25, 'Beijing']

// 重新索引数组
$arr = [10 => 'a', 20 => 'b', 30 => 'c'];
$reindexed = array_values($arr);
print_r($reindexed);  // ['a', 'b', 'c'] - 键被重新索引为 0, 1, 2
```

## 合并函数

### array_merge() - 合并数组

**语法**：`array_merge(array ...$arrays): array`

**参数**：
- `...$arrays`：要合并的数组（可变参数，可传入多个数组）

**返回值**：返回合并后的新数组。数字键会被重新索引，字符串键会覆盖（后面的值覆盖前面的值）。

```php
<?php
declare(strict_types=1);

$arr1 = ['a' => 1, 'b' => 2];
$arr2 = ['c' => 3, 'd' => 4];
$merged = array_merge($arr1, $arr2);
print_r($merged);  // ['a' => 1, 'b' => 2, 'c' => 3, 'd' => 4]

// 数字键会重新索引
$arr1 = [1, 2, 3];
$arr2 = [4, 5, 6];
$merged = array_merge($arr1, $arr2);
print_r($merged);  // [0 => 1, 1 => 2, 2 => 3, 3 => 4, 4 => 5, 5 => 6]

// 字符串键会覆盖
$arr1 = ['a' => 1, 'b' => 2];
$arr2 = ['b' => 3, 'c' => 4];
$merged = array_merge($arr1, $arr2);
print_r($merged);  // ['a' => 1, 'b' => 3, 'c' => 4]
```

### array_merge_recursive() - 递归合并

**语法**：`array_merge_recursive(array ...$arrays): array`

**参数**：
- `...$arrays`：要合并的数组（可变参数，可传入多个数组）

**返回值**：返回递归合并后的新数组。如果键是数组，会递归合并；如果键是标量值，会将值转换为数组。

```php
<?php
declare(strict_types=1);

$arr1 = ['user' => ['name' => 'Alice'], 'count' => 1];
$arr2 = ['user' => ['age' => 25], 'count' => 2];
$merged = array_merge_recursive($arr1, $arr2);
print_r($merged);
// 输出：
// Array
// (
//     [user] => Array
//         (
//             [name] => Alice
//             [age] => 25
//         )
//     [count] => Array
//         (
//             [0] => 1
//             [1] => 2
//         )
// )
```

## 高阶函数

### array_map() - 映射

**语法**：`array_map(?callable $callback, array $array, array ...$arrays): array`

**参数**：
- `$callback`：回调函数，对每个元素执行。如果为 `null`，则返回原数组
- `$array`：要处理的数组
- `...$arrays`：可选，多个数组时，回调函数接收多个参数（每个数组对应一个参数）

**返回值**：返回处理后的新数组，键会被重新索引。

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

// 将每个元素乘以 2
$doubled = array_map(fn($n) => $n * 2, $numbers);
print_r($doubled);  // [2, 4, 6, 8, 10]

// 使用多个数组
$arr1 = [1, 2, 3];
$arr2 = [10, 20, 30];
$sum = array_map(fn($a, $b) => $a + $b, $arr1, $arr2);
print_r($sum);  // [11, 22, 33]
```

### array_filter() - 过滤

**语法**：`array_filter(array $array, ?callable $callback = null, int $mode = 0): array`

**参数**：
- `$array`：要过滤的数组
- `$callback`：可选，过滤函数，返回 `true` 保留元素，返回 `false` 移除元素。如果为 `null`，则过滤掉所有"空"值
- `$mode`：可选，过滤模式：`ARRAY_FILTER_USE_KEY`（只传递键给回调）、`ARRAY_FILTER_USE_BOTH`（传递键和值给回调），默认为 0（只传递值）

**返回值**：返回过滤后的新数组，原数组的键会被保留。

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 过滤偶数
$evens = array_filter($numbers, fn($n) => $n % 2 === 0);
print_r($evens);  // [1 => 2, 3 => 4, 5 => 6, 7 => 8, 9 => 10]

// 注意：键被保留，使用 array_values() 重新索引
$evens = array_values(array_filter($numbers, fn($n) => $n % 2 === 0));
print_r($evens);  // [2, 4, 6, 8, 10]

// 过滤空值
$data = ['a' => 1, 'b' => 0, 'c' => '', 'd' => null, 'e' => 2];
$filtered = array_filter($data);
print_r($filtered);  // ['a' => 1, 'e' => 2]
```

### array_reduce() - 归约

**语法**：`array_reduce(array $array, callable $callback, mixed $initial = null): mixed`

**参数**：
- `$array`：要归约的数组
- `$callback`：归约函数，接收累积值和当前值，返回新的累积值
- `$initial`：可选，初始值，默认为 `null`

**返回值**：返回归约后的结果。

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

// 求和
$sum = array_reduce($numbers, fn($carry, $n) => $carry + $n, 0);
echo $sum . "\n";  // 15

// 求积
$product = array_reduce($numbers, fn($carry, $n) => $carry * $n, 1);
echo $product . "\n";  // 120

// 连接字符串
$words = ['Hello', 'World', 'PHP'];
$sentence = array_reduce($words, fn($carry, $word) => $carry . ' ' . $word, '');
echo trim($sentence) . "\n";  // Hello World PHP
```

## 切片函数

### array_slice() - 截取数组片段

**语法**：`array_slice(array $array, int $offset, ?int $length = null, bool $preserve_keys = false): array`

**参数**：
- `$array`：要截取的数组
- `$offset`：起始位置（负数表示从末尾开始）
- `$length`：可选，要截取的长度（负数表示从末尾开始），默认为 `null`（截取到末尾）
- `$preserve_keys`：可选，是否保留原数组的键，默认为 `false`（重新索引）

**返回值**：返回截取后的新数组。

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 从索引 2 开始，取 3 个元素
$slice = array_slice($numbers, 2, 3);
print_r($slice);  // [3, 4, 5]

// 保留键
$slice = array_slice($numbers, 2, 3, true);
print_r($slice);  // [2 => 3, 3 => 4, 4 => 5]

// 从末尾开始
$slice = array_slice($numbers, -3);
print_r($slice);  // [8, 9, 10]
```

## PHP 8.4+ 新函数

### array_first() - 获取第一个元素

**语法**：`array_first(array $array, ?callable $callback = null): mixed`

**参数**：
- `$array`：要处理的数组
- `$callback`：可选，回调函数，用于查找第一个符合条件的元素

**返回值**：如果指定了回调，返回第一个符合条件的元素；否则返回数组的第一个元素。如果数组为空或未找到，返回 `null`。

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

// 获取第一个元素
$first = array_first($numbers);
echo $first . "\n";  // 1

// 使用回调查找第一个符合条件的元素
$firstEven = array_first($numbers, fn($n) => $n % 2 === 0);
echo $firstEven . "\n";  // 2
```

### array_last() - 获取最后一个元素

**语法**：`array_last(array $array, ?callable $callback = null): mixed`

**参数**：
- `$array`：要处理的数组
- `$callback`：可选，回调函数，用于查找最后一个符合条件的元素

**返回值**：如果指定了回调，返回最后一个符合条件的元素；否则返回数组的最后一个元素。如果数组为空或未找到，返回 `null`。

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

// 获取最后一个元素
$last = array_last($numbers);
echo $last . "\n";  // 5

// 使用回调查找最后一个符合条件的元素
$lastEven = array_last($numbers, fn($n) => $n % 2 === 0);
echo $lastEven . "\n";  // 4
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ArrayFunctions
{
    public static function demonstrate(): void
    {
        echo "=== 查找函数 ===\n";
        $user = ['name' => 'Alice', 'age' => 25];
        echo "Has 'name' key: " . (array_key_exists('name', $user) ? 'Yes' : 'No') . "\n";
        echo "Has 'Alice' value: " . (in_array('Alice', $user) ? 'Yes' : 'No') . "\n";
        
        echo "\n=== 合并函数 ===\n";
        $arr1 = ['a' => 1, 'b' => 2];
        $arr2 = ['b' => 3, 'c' => 4];
        $merged = array_merge($arr1, $arr2);
        print_r($merged);
        
        echo "\n=== 高阶函数 ===\n";
        $numbers = [1, 2, 3, 4, 5];
        $doubled = array_map(fn($n) => $n * 2, $numbers);
        $evens = array_filter($numbers, fn($n) => $n % 2 === 0);
        $sum = array_reduce($numbers, fn($carry, $n) => $carry + $n, 0);
        echo "Doubled: " . implode(', ', $doubled) . "\n";
        echo "Evens: " . implode(', ', $evens) . "\n";
        echo "Sum: {$sum}\n";
        
        echo "\n=== 切片函数 ===\n";
        $slice = array_slice($numbers, 1, 3);
        echo "Slice: " . implode(', ', $slice) . "\n";
    }
}

ArrayFunctions::demonstrate();
```

## 注意事项

1. **键的保留**：`array_filter()` 会保留原数组的键，需要重新索引时使用 `array_values()`。

2. **数字键重新索引**：`array_merge()` 会重新索引数字键，字符串键会覆盖。

3. **性能考虑**：高阶函数比循环稍慢，但代码更简洁，可读性更好。

4. **回调函数**：使用箭头函数（PHP 7.4+）可以简化代码。

5. **空数组处理**：大多数函数对空数组都能正确处理，不会产生错误。

## 练习

1. 创建一个函数，使用 `array_map()` 将数组中的所有字符串转换为大写。

2. 编写一个函数，使用 `array_filter()` 和 `array_reduce()` 计算数组中所有正数的平均值。

3. 实现一个函数，合并多个数组，处理键冲突的情况。

4. 创建一个函数，使用 `array_slice()` 实现数组分页功能。

5. 编写一个函数，使用高阶函数实现数组的链式操作。
