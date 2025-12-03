# 2.10.2 可变参数

## 概述

可变参数允许函数接受不定数量的参数。PHP 提供了多种方式处理可变参数，包括 `...` 运算符和 `func_*` 系列函数。

## 可变参数语法（PHP 5.6+）

### 基本语法

```php
function functionName(...$args) {
    // 处理 $args 数组
}
```

### 基本用法

```php
<?php
declare(strict_types=1);

function sum(...$numbers): int
{
    return array_sum($numbers);
}

echo sum(1, 2, 3) . "\n";        // 6
echo sum(1, 2, 3, 4, 5) . "\n";  // 15
```

### 与固定参数结合

```php
<?php
declare(strict_types=1);

function greet(string $name, ...$titles): string
{
    $fullTitle = implode(' ', $titles);
    return "Hello, {$fullTitle} {$name}!";
}

echo greet("Alice", "Dr.", "Prof.") . "\n";  // Hello, Dr. Prof. Alice!
```

### 类型约束

```php
<?php
declare(strict_types=1);

function sumInts(int ...$numbers): int
{
    return array_sum($numbers);
}

echo sumInts(1, 2, 3) . "\n";  // 6
// echo sumInts(1, 2.5, 3);    // 类型错误
```

## func_get_args() - 获取所有参数

### 语法

**语法**：`func_get_args(): array`

**返回值**：返回包含所有参数的数组。

### 基本用法

```php
<?php
declare(strict_types=1);

function example(): void
{
    $args = func_get_args();
    print_r($args);
}

example(1, 2, 3, 'hello');
// 输出：
// Array
// (
//     [0] => 1
//     [1] => 2
//     [2] => 3
//     [3] => hello
// )
```

### 与可变参数对比

```php
<?php
declare(strict_types=1);

// 使用可变参数（推荐）
function sum1(...$numbers): int
{
    return array_sum($numbers);
}

// 使用 func_get_args()（旧方式）
function sum2(): int
{
    $numbers = func_get_args();
    return array_sum($numbers);
}

echo sum1(1, 2, 3) . "\n";  // 6
echo sum2(1, 2, 3) . "\n";  // 6
```

## func_num_args() - 获取参数数量

### 语法

**语法**：`func_num_args(): int`

**返回值**：返回传递给函数的参数数量。

### 基本用法

```php
<?php
declare(strict_types=1);

function example(): void
{
    $count = func_num_args();
    echo "Number of arguments: {$count}\n";
}

example(1, 2, 3);  // Number of arguments: 3
example();         // Number of arguments: 0
```

## func_get_arg() - 获取指定参数

### 语法

**语法**：`func_get_arg(int $position): mixed`

**参数**：
- `$position`：参数位置（从 0 开始）

**返回值**：返回指定位置的参数值。

### 基本用法

```php
<?php
declare(strict_types=1);

function example(): void
{
    $first = func_get_arg(0);
    $second = func_get_arg(1);
    echo "First: {$first}, Second: {$second}\n";
}

example('hello', 'world');  // First: hello, Second: world
```

## 展开运算符（Spread Operator）

### 基本语法

```php
functionName(...$array);
```

### 基本用法

展开运算符可以将数组展开为函数参数：

```php
<?php
declare(strict_types=1);

function add(int $a, int $b, int $c): int
{
    return $a + $b + $c;
}

$numbers = [1, 2, 3];
echo add(...$numbers) . "\n";  // 6

// 等价于
echo add(1, 2, 3) . "\n";  // 6
```

### 与可变参数结合

```php
<?php
declare(strict_types=1);

function mergeArrays(...$arrays): array
{
    return array_merge(...$arrays);
}

$arr1 = [1, 2, 3];
$arr2 = [4, 5, 6];
$arr3 = [7, 8, 9];

$merged = mergeArrays($arr1, $arr2, $arr3);
print_r($merged);  // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### 部分展开

```php
<?php
declare(strict_types=1);

function example(string $a, int $b, int $c): void
{
    echo "a = {$a}, b = {$b}, c = {$c}\n";
}

$args = [10, 20];
example('hello', ...$args);  // a = hello, b = 10, c = 20
```

## 完整示例

```php
<?php
declare(strict_types=1);

class VariableArguments
{
    public static function demonstrate(): void
    {
        echo "=== 可变参数 ===\n";
        echo sum(1, 2, 3, 4, 5) . "\n";
        
        echo "\n=== func_get_args() ===\n";
        example(1, 2, 3, 'hello');
        
        echo "\n=== func_num_args() ===\n";
        countArgs(1, 2, 3);
        
        echo "\n=== 展开运算符 ===\n";
        $numbers = [1, 2, 3];
        echo add(...$numbers) . "\n";
    }
}

function sum(...$numbers): int
{
    return array_sum($numbers);
}

function example(): void
{
    $args = func_get_args();
    echo "Arguments: " . implode(', ', $args) . "\n";
}

function countArgs(): void
{
    $count = func_num_args();
    echo "Number of arguments: {$count}\n";
}

function add(int $a, int $b, int $c): int
{
    return $a + $b + $c;
}

VariableArguments::demonstrate();
```

## 注意事项

1. **类型约束**：可变参数可以使用类型约束，如 `int ...$numbers`

2. **性能考虑**：可变参数会创建数组，对于大量参数可能有性能影响

3. **展开运算符**：展开运算符可以用于任何可迭代对象

4. **兼容性**：`func_*` 函数是旧方式，推荐使用 `...` 运算符

5. **文档说明**：使用可变参数时，应在文档中说明参数的用途和类型

## 练习

1. 创建一个函数，使用可变参数计算多个数字的最大值。

2. 编写一个函数，使用展开运算符将数组传递给另一个函数。

3. 实现一个函数，使用可变参数实现字符串格式化功能。

4. 创建一个函数，使用 `func_get_args()` 实现向后兼容的参数处理。

5. 编写一个函数，演示可变参数与固定参数的组合使用。
