# 2.10.2 可变参数

## 概述

可变参数允许函数接受不定数量的参数。本节详细介绍可变参数语法（`...` 运算符）、`func_get_args()`、`func_num_args()`、`func_get_arg()`、展开运算符的使用及完整示例。

理解可变参数的使用对于编写灵活的函数非常重要。掌握可变参数语法和展开运算符可以帮助编写更简洁、更强大的代码。

## 特性

- **灵活参数**：可以接受不定数量的参数
- **类型声明**：支持类型声明（PHP 8.0+）
- **展开运算符**：可以将数组展开为参数
- **向后兼容**：兼容传统的 `func_*` 函数

## 语法/定义

### 可变参数语法（... 运算符）

**基本语法**：`function func(...$args) { ... }`

**特点**：
- `$args` 是一个数组，包含所有传递的参数
- 必须放在参数列表的最后
- 可以与其他参数一起使用

**类型声明语法**（PHP 8.0+）：`function func(string ...$names) { ... }`

**特点**：
- 所有可变参数必须是同一类型
- 提供类型安全

### 展开运算符（... 运算符）

**语法**：`func(...$array)`

**功能**：将数组展开为函数参数

**特点**：
- 可以展开数组为参数
- 可以与其他参数一起使用
- 支持关联数组（PHP 8.0+ 命名参数）

### 传统函数

#### func_get_args() - 获取所有参数

**语法**：`func_get_args(): array`

**返回值**：返回包含所有参数的数组

#### func_num_args() - 获取参数数量

**语法**：`func_num_args(): int`

**返回值**：返回传递的参数数量

#### func_get_arg() - 获取指定参数

**语法**：`func_get_arg(int $position): mixed`

**参数**：
- `$position`：参数位置（从 0 开始）

**返回值**：返回指定位置的参数值

## 基本用法

### 示例 1：可变参数语法

```php
<?php
declare(strict_types=1);

// 基本可变参数
function sum(...$numbers): int
{
    return array_sum($numbers);
}

echo sum(1, 2, 3) . "\n";        // 6
echo sum(1, 2, 3, 4, 5) . "\n"; // 15

// 与其他参数一起使用
function greet(string $greeting, ...$names): string
{
    $nameList = implode(', ', $names);
    return "{$greeting}, {$nameList}!\n";
}

echo greet("Hello", "John", "Jane", "Bob");
// Hello, John, Jane, Bob!
```

### 示例 2：类型声明（PHP 8.0+）

```php
<?php
declare(strict_types=1);

// 类型声明的可变参数
function greet(string ...$names): void
{
    foreach ($names as $name) {
        echo "Hello, {$name}!\n";
    }
}

greet("John", "Jane", "Bob");
// Hello, John!
// Hello, Jane!
// Hello, Bob!

// 数值类型
function add(int ...$numbers): int
{
    return array_sum($numbers);
}

echo add(1, 2, 3, 4, 5) . "\n";  // 15
```

### 示例 3：展开运算符

```php
<?php
declare(strict_types=1);

// 基本展开
function add(int $a, int $b, int $c): int
{
    return $a + $b + $c;
}

$numbers = [1, 2, 3];
echo add(...$numbers) . "\n";  // 6

// 与其他参数一起使用
function greet(string $greeting, string $name, int $age): string
{
    return "{$greeting}, {$name}! You are {$age} years old.\n";
}

$params = ["Hello", "John", 25];
echo greet(...$params);
// Hello, John! You are 25 years old.

// 部分展开
function process(string $prefix, int ...$numbers): array
{
    return array_map(fn($n) => "{$prefix}{$n}", $numbers);
}

$numbers = [1, 2, 3];
$result = process("Item ", ...$numbers);
print_r($result);
// Array ( [0] => Item 1 [1] => Item 2 [2] => Item 3 )
```

### 示例 4：传统函数

```php
<?php
declare(strict_types=1);

// 使用 func_get_args()
function sum(): int
{
    $args = func_get_args();
    return array_sum($args);
}

echo sum(1, 2, 3) . "\n";  // 6

// 使用 func_num_args() 和 func_get_arg()
function process(): array
{
    $count = func_num_args();
    $result = [];
    for ($i = 0; $i < $count; $i++) {
        $result[] = func_get_arg($i) * 2;
    }
    return $result;
}

$result = process(1, 2, 3);
print_r($result);  // Array ( [0] => 2 [1] => 4 [2] => 6 )
```

## 完整代码示例

### 示例 1：日志函数

```php
<?php
declare(strict_types=1);

class Logger
{
    public static function log(string $level, string ...$messages): void
    {
        $timestamp = date('Y-m-d H:i:s');
        $message = implode(' ', $messages);
        echo "[{$timestamp}] [{$level}] {$message}\n";
    }
    
    public static function info(string ...$messages): void
    {
        self::log('INFO', ...$messages);
    }
    
    public static function error(string ...$messages): void
    {
        self::log('ERROR', ...$messages);
    }
}

// 使用
Logger::info("Application", "started", "successfully");
Logger::error("Failed", "to", "connect", "to", "database");
```

### 示例 2：数据聚合函数

```php
<?php
declare(strict_types=1);

function aggregate(string $operation, float ...$numbers): float
{
    return match($operation) {
        'sum' => array_sum($numbers),
        'avg' => count($numbers) > 0 ? array_sum($numbers) / count($numbers) : 0,
        'max' => count($numbers) > 0 ? max($numbers) : 0,
        'min' => count($numbers) > 0 ? min($numbers) : 0,
        'product' => array_reduce($numbers, fn($carry, $item) => $carry * $item, 1),
        default => throw new InvalidArgumentException("Unknown operation: {$operation}")
    };
}

// 使用
echo aggregate('sum', 1, 2, 3, 4, 5) . "\n";      // 15
echo aggregate('avg', 1, 2, 3, 4, 5) . "\n";     // 3
echo aggregate('max', 1, 2, 3, 4, 5) . "\n";     // 5
echo aggregate('product', 1, 2, 3, 4, 5) . "\n"; // 120
```

### 示例 3：函数包装器

```php
<?php
declare(strict_types=1);

function timedCall(callable $callback, ...$args): mixed
{
    $start = microtime(true);
    $result = $callback(...$args);
    $end = microtime(true);
    $duration = ($end - $start) * 1000;  // 转换为毫秒
    
    echo "Function executed in {$duration}ms\n";
    return $result;
}

// 使用
$result = timedCall('strtoupper', 'hello world');
echo $result . "\n";  // HELLO WORLD

$result = timedCall(fn($a, $b) => $a + $b, 10, 20);
echo $result . "\n";  // 30
```

### 示例 4：配置合并

```php
<?php
declare(strict_types=1);

function mergeConfigs(array ...$configs): array
{
    $result = [];
    foreach ($configs as $config) {
        $result = array_merge($result, $config);
    }
    return $result;
}

// 使用
$default = ['host' => 'localhost', 'port' => 3306];
$override1 = ['port' => 5432];
$override2 = ['database' => 'mydb'];

$config = mergeConfigs($default, $override1, $override2);
print_r($config);
// Array
// (
//     [host] => localhost
//     [port] => 5432
//     [database] => mydb
// )
```

## 使用场景

### 不定参数函数

- **聚合函数**：求和、求平均等聚合操作
- **日志函数**：接受多个日志消息
- **格式化函数**：格式化多个值

### 函数包装

- **性能监控**：包装函数进行性能监控
- **错误处理**：包装函数进行错误处理
- **缓存包装**：包装函数进行缓存

### 参数传递

- **动态调用**：动态调用函数并传递参数
- **参数转发**：将参数转发给其他函数
- **参数组合**：组合多个参数数组

## 注意事项

### 参数位置

- **必须在最后**：可变参数必须放在参数列表的最后
- **不能多个**：不能有多个可变参数
- **与其他参数**：可以与其他参数一起使用

### 类型声明

- **PHP 8.0+**：类型声明需要 PHP 8.0+
- **同一类型**：所有可变参数必须是同一类型
- **类型安全**：提供类型安全

### 性能考虑

- **数组创建**：可变参数会创建数组，有性能开销
- **展开运算符**：展开运算符有性能开销
- **合理使用**：合理使用，避免过度使用

## 常见问题

### 问题 1：可变参数位置错误

**症状**：语法错误

**原因**：可变参数不在参数列表最后

**错误示例**：

```php
<?php
declare(strict_types=1);

function func(...$args, string $name): void  // 错误
{
    // ...
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 正确：可变参数在最后
function func(string $name, ...$args): void
{
    // ...
}
```

### 问题 2：类型声明要求

**症状**：类型声明错误

**原因**：PHP 版本不支持或类型不匹配

**错误示例**：

```php
<?php
// PHP 7.x 不支持类型声明的可变参数
function greet(string ...$names): void  // PHP 8.0+ 才支持
{
    // ...
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：升级到 PHP 8.0+
function greet(string ...$names): void
{
    // ...
}

// 方法2：使用传统方法（兼容旧版本）
function greet(): void
{
    $names = func_get_args();
    foreach ($names as $name) {
        if (!is_string($name)) {
            throw new TypeError("All arguments must be strings");
        }
        echo "Hello, {$name}!\n";
    }
}
```

### 问题 3：展开运算符误用

**症状**：参数传递错误

**原因**：不理解展开运算符的作用

**解决方法**：

```php
<?php
declare(strict_types=1);

function add(int $a, int $b, int $c): int
{
    return $a + $b + $c;
}

$numbers = [1, 2, 3];

// 正确：展开数组为参数
echo add(...$numbers) . "\n";  // 6

// 错误：直接传递数组
// echo add($numbers);  // 错误：参数类型不匹配

// 正确：手动传递
echo add($numbers[0], $numbers[1], $numbers[2]) . "\n";  // 6
```

## 最佳实践

### 可变参数使用

- **使用新语法**：优先使用 `...` 语法
- **类型声明**：PHP 8.0+ 使用类型声明
- **合理使用**：避免过度使用

### 展开运算符

- **参数传递**：使用展开运算符传递数组参数
- **函数组合**：使用展开运算符组合函数
- **动态调用**：使用展开运算符进行动态调用

### 代码可读性

- **明确意图**：使用有意义的函数名
- **文档说明**：为可变参数函数添加文档
- **示例代码**：提供使用示例

## 对比分析

### 新语法 vs 传统函数

| 特性 | `...$args` | `func_get_args()` |
|:-----|:-----------|:------------------|
| 语法简洁 | 是 | 否 |
| 类型声明 | 支持（PHP 8.0+） | 不支持 |
| 性能 | 稍好 | 稍差 |
| 推荐度 | 推荐 | 兼容旧版本 |

**选择建议**：
- **PHP 5.6+**：使用 `...$args` 语法
- **兼容旧版本**：使用 `func_get_args()`

### 可变参数 vs 数组参数

| 特性 | 可变参数 | 数组参数 |
|:-----|:---------|:---------|
| 调用方式 | `func(1, 2, 3)` | `func([1, 2, 3])` |
| 可读性 | 高 | 中 |
| 灵活性 | 高 | 中 |
| 推荐度 | 推荐 | 按需使用 |

**选择建议**：
- **不定数量**：使用可变参数
- **固定结构**：使用数组参数

## 相关章节

- **2.10.1 函数基础**：了解函数基础
- **2.10.5 可调用类型与内置函数**：了解动态调用
- **2.8.3 数组操作函数**：了解数组函数

## 练习任务

1. **可变参数练习**：
   - 练习使用 `...` 语法定义可变参数函数
   - 理解类型声明的作用
   - 测试各种参数组合
   - 观察参数处理行为

2. **展开运算符练习**：
   - 练习使用展开运算符传递参数
   - 理解展开运算符的作用
   - 测试各种展开场景
   - 观察参数传递行为

3. **传统函数练习**：
   - 练习使用 `func_get_args()` 等函数
   - 理解传统方法的使用
   - 测试兼容性场景
   - 对比新旧方法

4. **实际应用练习**：
   - 实现日志函数
   - 实现数据聚合函数
   - 实现函数包装器
   - 测试各种应用场景

5. **综合练习**：
   - 创建一个使用可变参数的函数库
   - 实现各种功能函数
   - 使用类型声明提高代码质量
   - 进行代码审查，确保正确性
