# 2.8.4 数组排序

## 概述

数组排序是数据处理中的常见操作。PHP 提供了多种排序函数，可以根据值、键进行排序，也可以使用自定义比较函数。

## 基于值的排序

### sort() - 按值升序排序（重新索引）

**语法**：`sort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：
- `$array`：要排序的数组（按引用传递，会被修改）
- `$flags`：可选，排序标志（`SORT_REGULAR`、`SORT_NUMERIC`、`SORT_STRING` 等），默认为 `SORT_REGULAR`

**返回值**：成功返回 `true`，失败返回 `false`。

```php
<?php
declare(strict_types=1);

$numbers = [3, 1, 4, 1, 5, 9, 2, 6];
sort($numbers);
print_r($numbers);  // [1, 1, 2, 3, 4, 5, 6, 9] - 键被重新索引
```

### rsort() - 按值降序排序（重新索引）

**语法**：`rsort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：
- `$array`：要排序的数组（按引用传递，会被修改）
- `$flags`：可选，排序标志，默认为 `SORT_REGULAR`

**返回值**：成功返回 `true`，失败返回 `false`。

```php
<?php
declare(strict_types=1);

$numbers = [3, 1, 4, 1, 5, 9, 2, 6];
rsort($numbers);
print_r($numbers);  // [9, 6, 5, 4, 3, 2, 1, 1]
```

## 保留键的排序

### asort() - 按值升序排序（保留键）

**语法**：`asort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：
- `$array`：要排序的数组（按引用传递，会被修改）
- `$flags`：可选，排序标志，默认为 `SORT_REGULAR`

**返回值**：成功返回 `true`，失败返回 `false`。

```php
<?php
declare(strict_types=1);

$fruits = ['d' => 'lemon', 'a' => 'orange', 'b' => 'banana', 'c' => 'apple'];
asort($fruits);
print_r($fruits);
// 输出：
// Array
// (
//     [c] => apple
//     [b] => banana
//     [d] => lemon
//     [a] => orange
// )
```

### arsort() - 按值降序排序（保留键）

**语法**：`arsort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：
- `$array`：要排序的数组（按引用传递，会被修改）
- `$flags`：可选，排序标志，默认为 `SORT_REGULAR`

**返回值**：成功返回 `true`，失败返回 `false`。

```php
<?php
declare(strict_types=1);

$fruits = ['d' => 'lemon', 'a' => 'orange', 'b' => 'banana', 'c' => 'apple'];
arsort($fruits);
print_r($fruits);
// 输出：
// Array
// (
//     [a] => orange
//     [d] => lemon
//     [b] => banana
//     [c] => apple
// )
```

## 基于键的排序

### ksort() - 按键升序排序

**语法**：`ksort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：
- `$array`：要排序的数组（按引用传递，会被修改）
- `$flags`：可选，排序标志，默认为 `SORT_REGULAR`

**返回值**：成功返回 `true`，失败返回 `false`。

```php
<?php
declare(strict_types=1);

$fruits = ['d' => 'lemon', 'a' => 'orange', 'b' => 'banana', 'c' => 'apple'];
ksort($fruits);
print_r($fruits);
// 输出：
// Array
// (
//     [a] => orange
//     [b] => banana
//     [c] => apple
//     [d] => lemon
// )
```

### krsort() - 按键降序排序

**语法**：`krsort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：
- `$array`：要排序的数组（按引用传递，会被修改）
- `$flags`：可选，排序标志，默认为 `SORT_REGULAR`

**返回值**：成功返回 `true`，失败返回 `false`。

```php
<?php
declare(strict_types=1);

$fruits = ['d' => 'lemon', 'a' => 'orange', 'b' => 'banana', 'c' => 'apple'];
krsort($fruits);
print_r($fruits);
// 输出：
// Array
// (
//     [d] => lemon
//     [c] => apple
//     [b] => banana
//     [a] => orange
// )
```

## 自定义排序

### usort() - 自定义值排序

**语法**：`usort(array &$array, callable $callback): bool`

**参数**：
- `$array`：要排序的数组（按引用传递，会被修改）
- `$callback`：比较函数，接收两个参数，返回 `< 0`、`0` 或 `> 0`

**返回值**：成功返回 `true`，失败返回 `false`。

```php
<?php
declare(strict_types=1);

$users = [
    ['name' => 'Alice', 'age' => 25],
    ['name' => 'Bob', 'age' => 30],
    ['name' => 'Charlie', 'age' => 20]
];

// 按年龄排序
usort($users, fn($a, $b) => $a['age'] <=> $b['age']);
print_r($users);
// 输出：
// Array
// (
//     [0] => Array
//         (
//             [name] => Charlie
//             [age] => 20
//         )
//     [1] => Array
//         (
//             [name] => Alice
//             [age] => 25
//         )
//     [2] => Array
//         (
//             [name] => Bob
//             [age] => 30
//         )
// )
```

### uasort() - 自定义值排序（保留键）

**语法**：`uasort(array &$array, callable $callback): bool`

**参数**：
- `$array`：要排序的数组（按引用传递，会被修改）
- `$callback`：比较函数，接收两个参数，返回 `< 0`、`0` 或 `> 0`

**返回值**：成功返回 `true`，失败返回 `false`。

### uksort() - 自定义键排序

**语法**：`uksort(array &$array, callable $callback): bool`

**参数**：
- `$array`：要排序的数组（按引用传递，会被修改）
- `$callback`：比较函数，接收两个键作为参数，返回 `< 0`、`0` 或 `> 0`

**返回值**：成功返回 `true`，失败返回 `false`。

## 多字段排序

```php
<?php
declare(strict_types=1);

$users = [
    ['name' => 'Alice', 'age' => 25, 'score' => 80],
    ['name' => 'Bob', 'age' => 30, 'score' => 90],
    ['name' => 'Charlie', 'age' => 25, 'score' => 85]
];

// 先按年龄排序，年龄相同按分数排序
usort($users, function($a, $b) {
    $ageCompare = $a['age'] <=> $b['age'];
    if ($ageCompare !== 0) {
        return $ageCompare;
    }
    return $b['score'] <=> $a['score'];  // 分数降序
});

print_r($users);
```

## 排序标志

| 标志              | 说明                     |
| :---------------- | :----------------------- |
| `SORT_REGULAR`    | 常规比较（默认）         |
| `SORT_NUMERIC`    | 数值比较                 |
| `SORT_STRING`     | 字符串比较               |
| `SORT_LOCALE_STRING` | 根据当前区域设置比较 |
| `SORT_NATURAL`    | 自然排序                 |
| `SORT_FLAG_CASE`  | 可与 `SORT_STRING` 或 `SORT_NATURAL` 组合，不区分大小写 |

### 自然排序示例

```php
<?php
declare(strict_types=1);

$files = ['file1.txt', 'file10.txt', 'file2.txt', 'file20.txt'];

// 常规排序
$regular = $files;
sort($regular, SORT_STRING);
print_r($regular);  // ['file1.txt', 'file10.txt', 'file2.txt', 'file20.txt']

// 自然排序
$natural = $files;
sort($natural, SORT_NATURAL);
print_r($natural);  // ['file1.txt', 'file2.txt', 'file10.txt', 'file20.txt']
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ArraySorting
{
    public static function demonstrate(): void
    {
        echo "=== 基本排序 ===\n";
        $numbers = [3, 1, 4, 1, 5, 9, 2, 6];
        sort($numbers);
        echo "Sorted: " . implode(', ', $numbers) . "\n";
        
        echo "\n=== 保留键的排序 ===\n";
        $fruits = ['d' => 'lemon', 'a' => 'orange', 'b' => 'banana'];
        asort($fruits);
        print_r($fruits);
        
        echo "\n=== 自定义排序 ===\n";
        $users = [
            ['name' => 'Alice', 'age' => 25],
            ['name' => 'Bob', 'age' => 30],
            ['name' => 'Charlie', 'age' => 20]
        ];
        usort($users, fn($a, $b) => $a['age'] <=> $b['age']);
        print_r($users);
        
        echo "\n=== 多字段排序 ===\n";
        $products = [
            ['name' => 'Product A', 'price' => 100, 'stock' => 10],
            ['name' => 'Product B', 'price' => 100, 'stock' => 5],
            ['name' => 'Product C', 'price' => 50, 'stock' => 20]
        ];
        usort($products, function($a, $b) {
            $priceCompare = $a['price'] <=> $b['price'];
            return $priceCompare !== 0 ? $priceCompare : $b['stock'] <=> $a['stock'];
        });
        print_r($products);
    }
}

ArraySorting::demonstrate();
```

## 注意事项

1. **修改原数组**：所有排序函数都直接修改原数组，不返回新数组。

2. **键的保留**：`sort()` 和 `rsort()` 会重新索引，`asort()` 和 `arsort()` 会保留键。

3. **自定义比较**：比较函数应返回 `< 0`、`0` 或 `> 0`，使用 `<=>` 运算符最简洁。

4. **性能考虑**：对于大数组，自定义排序可能较慢，考虑使用更高效的算法。

5. **稳定性**：PHP 的排序函数不保证稳定性（相同值的相对顺序可能改变）。

## 练习

1. 创建一个函数，对用户数组按多个字段排序（年龄、分数、姓名）。

2. 编写一个函数，对文件列表进行自然排序。

3. 实现一个函数，对关联数组按值排序，但保留原始键。

4. 创建一个函数，使用自定义比较函数对复杂对象数组排序。

5. 编写一个函数，实现数组的稳定排序（保持相同值的相对顺序）。
