# 2.11.2 闭包与作用域

## 概述

闭包（Closure）是能够访问外部作用域变量的匿名函数。PHP 使用 `use` 关键字显式捕获外部变量，支持按值捕获和按引用捕获。

## use 关键字

### 基本语法

```php
$closure = function() use ($variable) {
    // 可以访问 $variable
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

## 按值捕获

### 默认行为

默认情况下，`use` 按值捕获变量，闭包内修改不会影响外部变量：

```php
<?php
declare(strict_types=1);

$count = 0;
$increment = function() use ($count): void {
    $count++;  // 修改的是副本
    echo "Inside: {$count}\n";
};

$increment();  // Inside: 1
echo "Outside: {$count}\n";  // Outside: 0（未改变）
```

### 变量快照

闭包捕获的是变量在定义时的值：

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$greet = function() use ($name): string {
    return "Hello, {$name}!";
};

$name = 'Bob';  // 修改外部变量
echo $greet() . "\n";  // Hello, Alice!（仍然是捕获时的值）
```

## 按引用捕获

### 使用 & 符号

使用 `&` 按引用捕获变量，闭包内修改会影响外部变量：

```php
<?php
declare(strict_types=1);

$count = 0;
$increment = function() use (&$count): void {
    $count++;  // 修改外部变量
    echo "Inside: {$count}\n";
};

$increment();  // Inside: 1
echo "Outside: {$count}\n";  // Outside: 1（已改变）
```

### 实际应用

```php
<?php
declare(strict_types=1);

$total = 0;
$numbers = [1, 2, 3, 4, 5];

array_walk($numbers, function($n) use (&$total): void {
    $total += $n;
});

echo $total . "\n";  // 15
```

## 捕获多个变量

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

## 闭包工厂

### 创建具有不同行为的函数

```php
<?php
declare(strict_types=1);

function createCounter(int $start = 0): Closure
{
    $count = $start;
    return function() use (&$count): int {
        return $count++;
    };
}

$counter1 = createCounter(0);
$counter2 = createCounter(10);

echo $counter1() . "\n";  // 0
echo $counter1() . "\n";  // 1
echo $counter2() . "\n";  // 10
echo $counter2() . "\n";  // 11
```

### 配置函数

```php
<?php
declare(strict_types=1);

function createFormatter(string $prefix, string $suffix): Closure
{
    return function(string $text) use ($prefix, $suffix): string {
        return "{$prefix}{$text}{$suffix}";
    };
}

$bold = createFormatter('<b>', '</b>');
$italic = createFormatter('<i>', '</i>');

echo $bold('Hello') . "\n";   // <b>Hello</b>
echo $italic('World') . "\n"; // <i>World</i>
```

## 闭包与类

### 访问类属性

```php
<?php
declare(strict_types=1);

class Calculator
{
    private int $factor = 2;
    
    public function getMultiplier(): Closure
    {
        return function(int $x): int {
            return $x * $this->factor;  // 访问 $this
        };
    }
}

$calc = new Calculator();
$multiplier = $calc->getMultiplier();
echo $multiplier(5) . "\n";  // 10
```

### 静态方法中的闭包

```php
<?php
declare(strict_types=1);

class Math
{
    private static int $multiplier = 3;
    
    public static function getMultiplier(): Closure
    {
        return function(int $x): int {
            return $x * self::$multiplier;
        };
    }
}

$multiplier = Math::getMultiplier();
echo $multiplier(5) . "\n";  // 15
```

## Closure::bindTo() - 绑定作用域

### 语法

**语法**：`Closure::bindTo(?object $newThis, ?string $newScope = "static"): ?Closure`

### 基本用法

```php
<?php
declare(strict_types=1);

class User
{
    private string $name = 'Alice';
}

$closure = function(): string {
    return $this->name;
};

$user = new User();
$bound = $closure->bindTo($user, User::class);
echo $bound() . "\n";  // Alice
```

## 变量生命周期

### 闭包延长变量生命周期

```php
<?php
declare(strict_types=1);

function createClosure(): Closure
{
    $local = 'This variable will persist';
    
    return function() use ($local): string {
        return $local;
    };
}

$closure = createClosure();
// $local 在 createClosure() 执行完后仍然存在
echo $closure() . "\n";  // This variable will persist
```

## 与 JavaScript 对比

### JavaScript 闭包

```javascript
// JavaScript
let count = 0;
const increment = () => {
    count++;  // 自动捕获外部变量
    return count;
};
```

### PHP 闭包

```php
<?php
// PHP
$count = 0;
$increment = function() use (&$count): int {
    $count++;  // 需要显式使用 use
    return $count;
};
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ClosuresScope
{
    public static function demonstrate(): void
    {
        echo "=== 按值捕获 ===\n";
        $count = 0;
        $increment = function() use ($count): void {
            $count++;
            echo "Inside: {$count}\n";
        };
        $increment();
        echo "Outside: {$count}\n";  // 仍然是 0
        
        echo "\n=== 按引用捕获 ===\n";
        $count = 0;
        $increment = function() use (&$count): void {
            $count++;
            echo "Inside: {$count}\n";
        };
        $increment();
        echo "Outside: {$count}\n";  // 1
        
        echo "\n=== 闭包工厂 ===\n";
        $counter = createCounter(5);
        echo $counter() . "\n";  // 5
        echo $counter() . "\n";  // 6
        
        echo "\n=== 闭包与类 ===\n";
        $calc = new Calculator();
        $multiplier = $calc->getMultiplier();
        echo $multiplier(5) . "\n";  // 10
    }
}

function createCounter(int $start = 0): Closure
{
    $count = $start;
    return function() use (&$count): int {
        return $count++;
    };
}

class Calculator
{
    private int $factor = 2;
    
    public function getMultiplier(): Closure
    {
        return function(int $x): int {
            return $x * $this->factor;
        };
    }
}

ClosuresScope::demonstrate();
```

## 注意事项

1. **按值捕获**：默认按值捕获，修改不会影响外部变量。

2. **按引用捕获**：使用 `&` 按引用捕获，修改会影响外部变量。

3. **变量快照**：闭包捕获的是定义时的变量值。

4. **性能考虑**：闭包会延长变量的生命周期，注意内存使用。

5. **作用域绑定**：使用 `bindTo()` 可以改变闭包的作用域。

## 练习

1. 创建一个闭包工厂，生成具有不同配置的函数。

2. 编写一个函数，使用闭包实现缓存机制。

3. 实现一个事件系统，使用闭包注册和触发事件。

4. 创建一个函数，演示按值和按引用捕获的区别。

5. 编写一个函数，使用 `bindTo()` 改变闭包的作用域。
