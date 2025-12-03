# 2.10.4 可调用类型与内置函数

## 概述

可调用类型（Callable）表示可以作为函数调用的值。PHP 支持多种可调用形式，并提供了丰富的内置函数用于处理可调用类型。

## 可调用类型的形式

### 1. 函数名字符串

```php
<?php
declare(strict_types=1);

function greet(string $name): string
{
    return "Hello, {$name}!";
}

$callable = 'greet';
echo $callable('Alice') . "\n";  // Hello, Alice!
```

### 2. 对象方法数组

```php
<?php
declare(strict_types=1);

class Greeter
{
    public function greet(string $name): string
    {
        return "Hello, {$name}!";
    }
}

$greeter = new Greeter();
$callable = [$greeter, 'greet'];
echo $callable('Bob') . "\n";  // Hello, Bob!
```

### 3. 静态方法数组

```php
<?php
declare(strict_types=1);

class Greeter
{
    public static function greet(string $name): string
    {
        return "Hello, {$name}!";
    }
}

$callable = [Greeter::class, 'greet'];
echo $callable('Charlie') . "\n";  // Hello, Charlie!
```

### 4. 匿名函数

```php
<?php
declare(strict_types=1);

$callable = function(string $name): string {
    return "Hello, {$name}!";
};

echo $callable('David') . "\n";  // Hello, David!
```

### 5. 箭头函数

```php
<?php
declare(strict_types=1);

$callable = fn(string $name): string => "Hello, {$name}!";
echo $callable('Eve') . "\n";  // Hello, Eve!
```

### 6. 实现了 __invoke() 的对象

```php
<?php
declare(strict_types=1);

class CallableClass
{
    public function __invoke(string $name): string
    {
        return "Hello, {$name}!";
    }
}

$callable = new CallableClass();
echo $callable('Frank') . "\n";  // Hello, Frank!
```

## is_callable() - 检查是否可调用

### 语法

**语法**：`is_callable(mixed $value, bool $syntax_only = false, string &$callable_name = null): bool`

**参数**：
- `$value`：要检查的值
- `$syntax_only`：如果为 `true`，只检查语法，不检查是否可调用
- `$callable_name`：可选，接收可调用名称

**返回值**：如果值可调用，返回 `true`；否则返回 `false`。

### 基本用法

```php
<?php
declare(strict_types=1);

function test(): void {}

var_dump(is_callable('test'));           // bool(true)
var_dump(is_callable('nonexistent'));    // bool(false)
var_dump(is_callable(fn() => null));     // bool(true)
var_dump(is_callable([new Greeter(), 'greet'])); // bool(true)
```

## call_user_func() - 调用可调用类型

### 语法

**语法**：`call_user_func(callable $callback, mixed ...$args): mixed`

**参数**：
- `$callback`：要调用的可调用类型
- `...$args`：传递给回调函数的参数

**返回值**：返回回调函数的返回值。

### 基本用法

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

$result = call_user_func('add', 5, 3);
echo $result . "\n";  // 8

// 使用匿名函数
$result = call_user_func(fn($a, $b) => $a + $b, 5, 3);
echo $result . "\n";  // 8
```

## call_user_func_array() - 使用数组调用

### 语法

**语法**：`call_user_func_array(callable $callback, array $args): mixed`

**参数**：
- `$callback`：要调用的可调用类型
- `$args`：包含参数的数组

**返回值**：返回回调函数的返回值。

### 基本用法

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

$args = [5, 3];
$result = call_user_func_array('add', $args);
echo $result . "\n";  // 8
```

### 动态参数调用

```php
<?php
declare(strict_types=1);

function process(...$args): array
{
    return $args;
}

$args = [1, 2, 3, 4, 5];
$result = call_user_func_array('process', $args);
print_r($result);  // [1, 2, 3, 4, 5]
```

## 常用内置函数

### array_map() - 映射

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];
$doubled = array_map(fn($n) => $n * 2, $numbers);
print_r($doubled);  // [2, 4, 6, 8, 10]
```

### array_filter() - 过滤

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
$evens = array_filter($numbers, fn($n) => $n % 2 === 0);
print_r($evens);  // [2, 4, 6, 8, 10]
```

### array_reduce() - 归约

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];
$sum = array_reduce($numbers, fn($carry, $n) => $carry + $n, 0);
echo $sum . "\n";  // 15
```

### usort() - 自定义排序

```php
<?php
declare(strict_types=1);

$users = [
    ['name' => 'Alice', 'age' => 25],
    ['name' => 'Bob', 'age' => 30],
    ['name' => 'Charlie', 'age' => 20]
];

usort($users, fn($a, $b) => $a['age'] <=> $b['age']);
print_r($users);
```

## 完整示例

```php
<?php
declare(strict_types=1);

class CallableTypes
{
    public static function demonstrate(): void
    {
        echo "=== 函数名字符串 ===\n";
        $callable = 'strtoupper';
        echo $callable('hello') . "\n";  // HELLO
        
        echo "\n=== 对象方法 ===\n";
        $greeter = new Greeter();
        $callable = [$greeter, 'greet'];
        echo $callable('Alice') . "\n";  // Hello, Alice!
        
        echo "\n=== 匿名函数 ===\n";
        $callable = fn(int $x): int => $x * 2;
        echo $callable(5) . "\n";  // 10
        
        echo "\n=== call_user_func ===\n";
        $result = call_user_func('add', 5, 3);
        echo $result . "\n";  // 8
        
        echo "\n=== call_user_func_array ===\n";
        $result = call_user_func_array('add', [5, 3]);
        echo $result . "\n";  // 8
        
        echo "\n=== 高阶函数 ===\n";
        $numbers = [1, 2, 3, 4, 5];
        $doubled = array_map(fn($n) => $n * 2, $numbers);
        print_r($doubled);
    }
}

class Greeter
{
    public function greet(string $name): string
    {
        return "Hello, {$name}!";
    }
}

function add(int $a, int $b): int
{
    return $a + $b;
}

CallableTypes::demonstrate();
```

## 注意事项

1. **类型检查**：在调用前使用 `is_callable()` 检查，避免运行时错误。

2. **性能考虑**：直接调用函数比通过可调用类型调用稍快。

3. **安全性**：从用户输入构建可调用类型时，要验证其安全性。

4. **类型提示**：函数参数可以使用 `callable` 作为类型提示。

5. **文档说明**：使用可调用类型时，应在文档中说明期望的回调函数签名。

## 练习

1. 创建一个函数，接受可调用类型作为参数，并安全地调用它。

2. 编写一个函数，使用 `call_user_func_array()` 实现动态函数调用。

3. 实现一个事件系统，使用可调用类型注册和触发事件。

4. 创建一个函数，验证可调用类型是否符合预期的签名。

5. 编写一个函数，使用可调用类型实现策略模式。
