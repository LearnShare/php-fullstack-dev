# 2.9.2 循环结构

## 概述

循环结构用于重复执行代码块。PHP 提供了四种循环结构：`while`、`do-while`、`for` 和 `foreach`。每种循环都有其适用场景。

## while 循环

### 基本语法

```php
while (condition) {
    // 代码块
}
```

### 基本用法

```php
<?php
declare(strict_types=1);

$count = 0;
while ($count < 5) {
    echo $count . "\n";
    $count++;
}
// 输出：0, 1, 2, 3, 4
```

### 条件未知的循环

```php
<?php
declare(strict_types=1);

$data = [1, 2, 3, 4, 5];
reset($data);

while (($value = current($data)) !== false) {
    echo $value . "\n";
    next($data);
}
```

## do-while 循环

### 基本语法

```php
do {
    // 代码块
} while (condition);
```

### 基本用法

`do-while` 循环至少执行一次，因为条件检查在循环体之后：

```php
<?php
declare(strict_types=1);

$count = 0;
do {
    echo $count . "\n";
    $count++;
} while ($count < 5);
// 输出：0, 1, 2, 3, 4
```

### 至少执行一次的场景

```php
<?php
declare(strict_types=1);

function getUserInput(): string
{
    // 模拟用户输入
    return "yes";
}

$attempts = 0;
do {
    $input = getUserInput();
    $attempts++;
} while ($input !== "yes" && $attempts < 3);
```

## for 循环

### 基本语法

```php
for (init; condition; increment) {
    // 代码块
}
```

### 基本用法

```php
<?php
declare(strict_types=1);

for ($i = 0; $i < 5; $i++) {
    echo $i . "\n";
}
// 输出：0, 1, 2, 3, 4
```

### 倒序循环

```php
<?php
declare(strict_types=1);

for ($i = 5; $i > 0; $i--) {
    echo $i . "\n";
}
// 输出：5, 4, 3, 2, 1
```

### 多变量循环

```php
<?php
declare(strict_types=1);

for ($i = 0, $j = 10; $i < 5; $i++, $j--) {
    echo "i = {$i}, j = {$j}\n";
}
```

### 省略部分表达式

```php
<?php
declare(strict_types=1);

$i = 0;
for (; $i < 5;) {
    echo $i . "\n";
    $i++;
}
```

## foreach 循环

### 基本语法

```php
foreach ($array as $value) {
    // 代码块
}

foreach ($array as $key => $value) {
    // 代码块
}
```

### 遍历索引数组

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

foreach ($numbers as $number) {
    echo $number . "\n";
}

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

### 引用遍历

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

foreach ($numbers as &$number) {
    $number *= 2;
}
unset($number);  // 重要：取消引用

print_r($numbers);  // [2, 4, 6, 8, 10]
```

## 嵌套循环

### 二维数组遍历

```php
<?php
declare(strict_types=1);

$matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

for ($i = 0; $i < count($matrix); $i++) {
    for ($j = 0; $j < count($matrix[$i]); $j++) {
        echo "Matrix[{$i}][{$j}] = {$matrix[$i][$j]}\n";
    }
}
```

### foreach 嵌套

```php
<?php
declare(strict_types=1);

$users = [
    ['name' => 'Alice', 'roles' => ['admin', 'user']],
    ['name' => 'Bob', 'roles' => ['user']]
];

foreach ($users as $user) {
    echo "User: {$user['name']}\n";
    foreach ($user['roles'] as $role) {
        echo "  - {$role}\n";
    }
}
```

## 循环结构对比

| 循环类型   | 适用场景                     | 特点                 |
| :--------- | :--------------------------- | :------------------- |
| `while`    | 条件未知的循环               | 先检查条件           |
| `do-while` | 至少执行一次的循环           | 先执行后检查         |
| `for`      | 已知次数的循环               | 适合计数循环         |
| `foreach`  | 遍历数组或可迭代对象         | 最高效的数组遍历方式 |

## 完整示例

```php
<?php
declare(strict_types=1);

class Loops
{
    public static function demonstrateWhile(): void
    {
        echo "=== while 循环 ===\n";
        $count = 0;
        while ($count < 3) {
            echo "Count: {$count}\n";
            $count++;
        }
    }
    
    public static function demonstrateDoWhile(): void
    {
        echo "\n=== do-while 循环 ===\n";
        $count = 0;
        do {
            echo "Count: {$count}\n";
            $count++;
        } while ($count < 3);
    }
    
    public static function demonstrateFor(): void
    {
        echo "\n=== for 循环 ===\n";
        for ($i = 0; $i < 3; $i++) {
            echo "i = {$i}\n";
        }
    }
    
    public static function demonstrateForeach(): void
    {
        echo "\n=== foreach 循环 ===\n";
        $numbers = [1, 2, 3];
        foreach ($numbers as $index => $number) {
            echo "Index {$index}: {$number}\n";
        }
    }
}

Loops::demonstrateWhile();
Loops::demonstrateDoWhile();
Loops::demonstrateFor();
Loops::demonstrateForeach();
```

## 注意事项

1. **无限循环**：确保循环条件最终会变为 `false`，避免无限循环。

2. **性能考虑**：`foreach` 是遍历数组最高效的方式，优先使用。

3. **引用遍历**：在 `foreach` 中使用引用后，必须 `unset()` 取消引用。

4. **循环变量作用域**：`for` 循环的初始化变量在循环外部不可访问（PHP 7.1+）。

5. **嵌套循环**：嵌套循环的性能是 O(n²)，注意性能影响。

## 练习

1. 创建一个函数，使用 `while` 循环计算数字的阶乘。

2. 编写一个函数，使用 `do-while` 循环实现重试机制。

3. 实现一个函数，使用 `for` 循环生成斐波那契数列。

4. 创建一个函数，使用 `foreach` 循环处理嵌套数组。

5. 编写一个函数，比较不同循环结构的性能。
