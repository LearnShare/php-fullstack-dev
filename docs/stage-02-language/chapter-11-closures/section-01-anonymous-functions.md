# 2.11.1 匿名函数基础

## 概述

匿名函数（Anonymous Functions）是没有名称的函数，也称为 lambda 函数。PHP 支持匿名函数，可以将其赋值给变量、作为参数传递或作为返回值。

## 匿名函数定义

### 基本语法

```php
$function = function(parameters) {
    // 函数体
    return value;
};
```

### 基本示例

```php
<?php
declare(strict_types=1);

$greet = function(string $name): string {
    return "Hello, {$name}!";
};

echo $greet("Alice") . "\n";  // Hello, Alice!
```

### 类型声明

匿名函数支持完整的类型声明：

```php
<?php
declare(strict_types=1);

$add = function(int $a, int $b): int {
    return $a + $b;
};

echo $add(5, 3) . "\n";  // 8
```

## 调用匿名函数

### 直接调用

```php
<?php
declare(strict_types=1);

$result = (function(int $x): int {
    return $x * 2;
})(5);

echo $result . "\n";  // 10
```

### 赋值后调用

```php
<?php
declare(strict_types=1);

$double = function(int $x): int {
    return $x * 2;
};

echo $double(5) . "\n";  // 10
```

## 在数组函数中使用

### array_map()

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];
$doubled = array_map(function($n) {
    return $n * 2;
}, $numbers);

print_r($doubled);  // [2, 4, 6, 8, 10]
```

### array_filter()

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
$evens = array_filter($numbers, function($n) {
    return $n % 2 === 0;
});

print_r($evens);  // [2, 4, 6, 8, 10]
```

### array_reduce()

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];
$sum = array_reduce($numbers, function($carry, $n) {
    return $carry + $n;
}, 0);

echo $sum . "\n";  // 15
```

## 立即执行函数（IIFE）

立即执行函数表达式（Immediately Invoked Function Expression）在定义后立即执行：

```php
<?php
declare(strict_types=1);

$result = (function(int $x): int {
    return $x * 2;
})(5);

echo $result . "\n";  // 10

// 用于创建作用域
(function(): void {
    $local = "This is local";
    echo $local . "\n";
})();
// $local 在这里不可访问
```

## 作为参数传递

```php
<?php
declare(strict_types=1);

function process(callable $callback, mixed $data): mixed
{
    return $callback($data);
}

$result = process(function($x) {
    return $x * 2;
}, 5);

echo $result . "\n";  // 10
```

## 作为返回值

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

## 完整示例

```php
<?php
declare(strict_types=1);

class AnonymousFunctions
{
    public static function demonstrate(): void
    {
        echo "=== 基本匿名函数 ===\n";
        $greet = function(string $name): string {
            return "Hello, {$name}!";
        };
        echo $greet("Alice") . "\n";
        
        echo "\n=== 在数组函数中使用 ===\n";
        $numbers = [1, 2, 3, 4, 5];
        $doubled = array_map(function($n) {
            return $n * 2;
        }, $numbers);
        print_r($doubled);
        
        echo "\n=== 立即执行函数 ===\n";
        $result = (function(int $x): int {
            return $x * 2;
        })(5);
        echo $result . "\n";
        
        echo "\n=== 作为返回值 ===\n";
        $multiplier = createMultiplier(5);
        echo $multiplier(3) . "\n";  // 15
    }
}

function createMultiplier(int $factor): Closure
{
    return function(int $x) use ($factor): int {
        return $x * $factor;
    };
}

AnonymousFunctions::demonstrate();
```

## 注意事项

1. **分号**：匿名函数定义后必须有分号。

2. **类型声明**：匿名函数支持完整的类型声明。

3. **变量捕获**：需要使用 `use` 关键字捕获外部变量。

4. **性能**：匿名函数比命名函数稍慢，但差异很小。

5. **可读性**：对于复杂逻辑，考虑使用命名函数提高可读性。

## 练习

1. 创建一个匿名函数，计算两个数的最大公约数。

2. 编写一个函数，使用匿名函数实现数组的链式操作。

3. 实现一个立即执行函数，创建独立的作用域。

4. 创建一个函数工厂，使用匿名函数生成不同行为的函数。

5. 编写一个函数，演示匿名函数在事件系统中的应用。
