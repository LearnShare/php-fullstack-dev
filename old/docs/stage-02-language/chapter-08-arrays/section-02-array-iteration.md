# 2.8.2 数组遍历

## 概述

遍历数组是 PHP 开发中最常见的操作之一。PHP 提供了多种遍历数组的方式，其中 `foreach` 是最常用和最高效的方法。

## foreach 循环

### 基本语法

```php
foreach ($array as $value) {
    // 处理 $value
}

foreach ($array as $key => $value) {
    // 处理 $key 和 $value
}
```

### 遍历索引数组

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

// 只遍历值
foreach ($numbers as $number) {
    echo $number . "\n";
}

// 遍历键和值
foreach ($numbers as $index => $number) {
    echo "Index {$index}: {$number}\n";
}
```

### 遍历关联数组

```php
<?php
declare(strict_types=1);

$user = [
    'name' => 'Alice',
    'age' => 25,
    'email' => 'alice@example.com'
];

foreach ($user as $key => $value) {
    echo "{$key}: {$value}\n";
}
```

## 引用遍历

### 使用引用修改数组元素

在 `foreach` 循环中使用引用（`&`）可以直接修改数组元素：

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

// 使用引用修改数组元素
foreach ($numbers as &$number) {
    $number *= 2;  // 将每个元素乘以 2
}
unset($number);  // 重要：取消引用

print_r($numbers);  // [2, 4, 6, 8, 10]
```

### 为什么需要 unset()

在 `foreach` 循环中使用引用后，必须使用 `unset()` 取消引用，否则后续对变量的赋值可能会意外修改数组的最后一个元素：

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3];

foreach ($numbers as &$number) {
    $number *= 2;
}
// 忘记 unset($number)

$number = 100;  // 意外修改了 $numbers[2]
print_r($numbers);  // [2, 4, 100] - 注意最后一个元素被修改了
```

## 嵌套数组遍历

### 二维数组

```php
<?php
declare(strict_types=1);

$matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

foreach ($matrix as $rowIndex => $row) {
    foreach ($row as $colIndex => $value) {
        echo "Matrix[{$rowIndex}][{$colIndex}] = {$value}\n";
    }
}
```

### 复杂嵌套结构

```php
<?php
declare(strict_types=1);

$users = [
    [
        'id' => 1,
        'name' => 'Alice',
        'roles' => ['admin', 'user']
    ],
    [
        'id' => 2,
        'name' => 'Bob',
        'roles' => ['user']
    ]
];

foreach ($users as $user) {
    echo "User: {$user['name']}\n";
    echo "Roles: " . implode(', ', $user['roles']) . "\n";
}
```

## 数组指针操作

### reset() - 重置指针

**语法**：`reset(array|object $array): mixed`

**参数**：
- `$array`：要重置指针的数组

**返回值**：返回数组的第一个元素的值，如果数组为空返回 `false`。

将数组内部指针指向第一个元素。

```php
<?php
declare(strict_types=1);

$arr = ['a', 'b', 'c'];
echo current($arr) . "\n";  // a
next($arr);
echo current($arr) . "\n";  // b
reset($arr);
echo current($arr) . "\n";  // a
```

### next() - 移动指针到下一个

**语法**：`next(array|object $array): mixed`

**参数**：
- `$array`：要移动指针的数组

**返回值**：返回下一个元素的值，如果没有下一个元素返回 `false`。

```php
<?php
declare(strict_types=1);

$arr = ['a', 'b', 'c'];
echo current($arr) . "\n";  // a
echo next($arr) . "\n";     // b
echo next($arr) . "\n";     // c
```

### prev() - 移动指针到上一个

**语法**：`prev(array|object $array): mixed`

**参数**：
- `$array`：要移动指针的数组

**返回值**：返回上一个元素的值，如果没有上一个元素返回 `false`。

```php
<?php
declare(strict_types=1);

$arr = ['a', 'b', 'c'];
end($arr);
echo current($arr) . "\n";  // c
echo prev($arr) . "\n";      // b
echo prev($arr) . "\n";      // a
```

### end() - 移动指针到最后一个

**语法**：`end(array|object $array): mixed`

**参数**：
- `$array`：要移动指针的数组

**返回值**：返回数组的最后一个元素的值，如果数组为空返回 `false`。

```php
<?php
declare(strict_types=1);

$arr = ['a', 'b', 'c'];
end($arr);
echo current($arr) . "\n";  // c
```

### current() - 获取当前元素

**语法**：`current(array|object $array): mixed`

**参数**：
- `$array`：要获取当前元素的数组

**返回值**：返回当前指针指向的元素的值，如果指针超出数组范围返回 `false`。

```php
<?php
declare(strict_types=1);

$arr = ['a', 'b', 'c'];
echo current($arr) . "\n";  // a
next($arr);
echo current($arr) . "\n";  // b
```

### key() - 获取当前键

**语法**：`key(array|object $array): int|string|null`

**参数**：
- `$array`：要获取当前键的数组

**返回值**：返回当前指针指向的元素的键（整数或字符串），如果指针超出数组范围返回 `null`。

```php
<?php
declare(strict_types=1);

$arr = ['first' => 'a', 'second' => 'b', 'third' => 'c'];
echo key($arr) . "\n";      // first
next($arr);
echo key($arr) . "\n";      // second
```

### 使用指针遍历数组

```php
<?php
declare(strict_types=1);

$arr = ['a', 'b', 'c'];

reset($arr);
while ($value = current($arr)) {
    echo key($arr) . ": {$value}\n";
    next($arr);
}
```

## 其他遍历方式

### for 循环（仅适用于索引数组）

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

for ($i = 0; $i < count($numbers); $i++) {
    echo $numbers[$i] . "\n";
}
```

### while + each()（已废弃，PHP 7.2+）

`each()` 函数在 PHP 7.2+ 中已废弃，不推荐使用。

## 完整示例

```php
<?php
declare(strict_types=1);

class ArrayIteration
{
    public static function demonstrate(): void
    {
        echo "=== foreach 遍历 ===\n";
        $numbers = [1, 2, 3, 4, 5];
        foreach ($numbers as $index => $number) {
            echo "Index {$index}: {$number}\n";
        }
        
        echo "\n=== 引用遍历修改 ===\n";
        $numbers = [1, 2, 3, 4, 5];
        foreach ($numbers as &$number) {
            $number *= 2;
        }
        unset($number);  // 重要
        print_r($numbers);
        
        echo "\n=== 嵌套数组遍历 ===\n";
        $users = [
            ['id' => 1, 'name' => 'Alice', 'roles' => ['admin']],
            ['id' => 2, 'name' => 'Bob', 'roles' => ['user']]
        ];
        foreach ($users as $user) {
            echo "User: {$user['name']}, Roles: " . implode(', ', $user['roles']) . "\n";
        }
        
        echo "\n=== 数组指针操作 ===\n";
        $arr = ['a', 'b', 'c'];
        reset($arr);
        while (($value = current($arr)) !== false) {
            echo key($arr) . ": {$value}\n";
            next($arr);
        }
    }
}

ArrayIteration::demonstrate();
```

## 注意事项

1. **引用遍历后必须 unset**：在 `foreach` 中使用引用后，必须 `unset()` 取消引用。

2. **foreach 不修改原数组**：默认情况下，`foreach` 遍历的是数组的副本，不会修改原数组（除非使用引用）。

3. **性能考虑**：`foreach` 是遍历数组最高效的方式，比 `for` 循环更快。

4. **指针操作**：数组指针操作主要用于特殊场景，日常开发中优先使用 `foreach`。

5. **空数组处理**：遍历空数组不会执行循环体，不会产生错误。

## 练习

1. 创建一个函数，使用 `foreach` 遍历数组并计算所有元素的总和。

2. 编写一个函数，使用引用遍历将数组中的所有字符串转换为大写。

3. 实现一个函数，遍历嵌套数组并输出所有键值对的路径。

4. 创建一个函数，使用数组指针操作实现数组的倒序遍历。

5. 编写一个函数，安全地遍历可能包含嵌套结构的数组。
