# 2.10.3 引用、箭头函数与闭包

## 概述

引用传参、箭头函数和闭包是 PHP 函数的高级特性。理解这些特性对于编写高效、灵活的代码至关重要。

## 引用传参

### 基本语法

```php
function functionName(type &$parameter): returnType
{
    // 修改 $parameter 会影响外部变量
}
```

### 基本用法

```php
<?php
declare(strict_types=1);

function increment(int &$value): void
{
    $value++;
}

$num = 5;
increment($num);
echo $num . "\n";  // 6（原值被修改）
```

### 交换变量

```php
<?php
declare(strict_types=1);

function swap(&$a, &$b): void
{
    $temp = $a;
    $a = $b;
    $b = $temp;
}

$x = 10;
$y = 20;
swap($x, $y);
echo "x = {$x}, y = {$y}\n";  // x = 20, y = 10
```

### 引用返回

函数可以返回引用：

```php
<?php
declare(strict_types=1);

$values = [10, 20, 30];

function &getValue(array &$arr, int $index): int
{
    return $arr[$index];
}

$ref =& getValue($values, 1);
$ref = 200;  // 直接修改 $values[1]

print_r($values);  // [10, 200, 30]
```

## 箭头函数（PHP 7.4+）

### 基本语法

```php
fn(parameters) => expression
```

### 基本用法

```php
<?php
declare(strict_types=1);

$double = fn(int $x): int => $x * 2;
echo $double(5) . "\n";  // 10

$add = fn(int $a, int $b): int => $a + $b;
echo $add(3, 4) . "\n";  // 7
```

### 与匿名函数对比

```php
<?php
declare(strict_types=1);

// 箭头函数
$arrow = fn(int $x): int => $x * 2;

// 匿名函数（等价）
$anonymous = function(int $x): int {
    return $x * 2;
};

echo $arrow(5) . "\n";      // 10
echo $anonymous(5) . "\n";  // 10
```

### 自动捕获变量

箭头函数自动按值捕获外部变量：

```php
<?php
declare(strict_types=1);

$multiplier = 3;
$arrow = fn(int $x): int => $x * $multiplier;  // 自动捕获 $multiplier

echo $arrow(5) . "\n";  // 15

$multiplier = 5;  // 修改外部变量不影响箭头函数
echo $arrow(5) . "\n";  // 仍然是 15
```

## 闭包（Closure）

### 基本语法

```php
$closure = function(parameters) use ($variables) {
    // 函数体
};
```

### 基本用法

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$greet = function() use ($name): string {
    return "Hello, {$name}!";
};

echo $greet() . "\n";  // Hello, Alice!
```

### 按值捕获

```php
<?php
declare(strict_types=1);

$count = 0;
$increment = function() use ($count): void {
    $count++;  // 修改的是副本，不影响外部变量
};

$increment();
echo $count . "\n";  // 仍然是 0
```

### 按引用捕获

使用 `&` 按引用捕获变量：

```php
<?php
declare(strict_types=1);

$count = 0;
$increment = function() use (&$count): void {
    $count++;  // 修改外部变量
};

$increment();
echo $count . "\n";  // 1
```

### 捕获多个变量

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$age = 25;
$city = 'Beijing';

$info = function() use ($name, $age, $city): string {
    return "{$name}, {$age}, {$city}";
};

echo $info() . "\n";  // Alice, 25, Beijing
```

## 闭包作为返回值

```php
<?php
declare(strict_types=1);

function createMultiplier(int $factor): Closure
{
    return function(int $x) use ($factor): int {
        return $x * $factor;
    };
}

$double = createMultiplier(2);
$triple = createMultiplier(3);

echo $double(5) . "\n";  // 10
echo $triple(5) . "\n";  // 15
```

## 闭包作为参数

```php
<?php
declare(strict_types=1);

function processNumbers(array $numbers, callable $callback): array
{
    return array_map($callback, $numbers);
}

$numbers = [1, 2, 3, 4, 5];
$multiplier = 2;

$result = processNumbers($numbers, function($n) use ($multiplier) {
    return $n * $multiplier;
});

print_r($result);  // [2, 4, 6, 8, 10]
```

## 箭头函数 vs 闭包

| 特性       | 箭头函数           | 闭包               |
| :--------- | :----------------- | :----------------- |
| 语法       | `fn() => expr`     | `function() {}`    |
| 变量捕获   | 自动按值捕获       | 需要 `use` 子句    |
| 按引用捕获 | 不支持             | 支持（`use (&$var)`） |
| 返回值     | 单表达式           | 可以有多条语句     |
| 性能       | 稍快               | 稍慢               |

### 示例对比

```php
<?php
declare(strict_types=1);

$multiplier = 2;
$numbers = [1, 2, 3, 4, 5];

// 箭头函数（自动捕获）
$arrow = array_map(fn($n) => $n * $multiplier, $numbers);

// 闭包（需要 use）
$closure = array_map(function($n) use ($multiplier) {
    return $n * $multiplier;
}, $numbers);
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ReferencesClosures
{
    public static function demonstrateReferences(): void
    {
        echo "=== 引用传参 ===\n";
        $num = 5;
        increment($num);
        echo "After increment: {$num}\n";  // 6
        
        $x = 10;
        $y = 20;
        swap($x, $y);
        echo "After swap: x = {$x}, y = {$y}\n";  // x = 20, y = 10
    }
    
    public static function demonstrateArrowFunctions(): void
    {
        echo "\n=== 箭头函数 ===\n";
        $multiplier = 3;
        $arrow = fn(int $x): int => $x * $multiplier;
        echo $arrow(5) . "\n";  // 15
    }
    
    public static function demonstrateClosures(): void
    {
        echo "\n=== 闭包 ===\n";
        $count = 0;
        $increment = function() use (&$count): void {
            $count++;
        };
        $increment();
        echo "Count: {$count}\n";  // 1
        
        $multiplier = createMultiplier(5);
        echo $multiplier(3) . "\n";  // 15
    }
}

function increment(int &$value): void
{
    $value++;
}

function swap(&$a, &$b): void
{
    $temp = $a;
    $a = $b;
    $b = $temp;
}

function createMultiplier(int $factor): Closure
{
    return function(int $x) use ($factor): int {
        return $x * $factor;
    };
}

ReferencesClosures::demonstrateReferences();
ReferencesClosures::demonstrateArrowFunctions();
ReferencesClosures::demonstrateClosures();
```

## 注意事项

1. **引用传参**：只在确实需要修改外部变量时使用引用，避免副作用。

2. **箭头函数限制**：箭头函数只能包含单个表达式，不能包含多条语句。

3. **变量捕获**：箭头函数自动按值捕获，闭包需要显式使用 `use` 子句。

4. **按引用捕获**：只有闭包支持按引用捕获，箭头函数不支持。

5. **性能考虑**：箭头函数性能略好于闭包，但差异很小。

## 练习

1. 创建一个函数，使用引用传参实现数组元素的批量修改。

2. 编写一个函数，使用箭头函数实现数组的高阶操作。

3. 实现一个函数工厂，使用闭包创建具有不同行为的函数。

4. 创建一个函数，演示箭头函数和闭包在变量捕获上的差异。

5. 编写一个函数，使用闭包实现计数器功能。
