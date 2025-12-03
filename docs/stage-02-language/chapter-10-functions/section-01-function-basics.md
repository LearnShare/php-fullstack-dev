# 2.10.1 函数基础

## 概述

函数是 PHP 中组织和复用代码的基本单元。PHP 7.0+ 引入了类型声明、返回值类型等现代特性，使函数更加安全和可维护。

## 函数定义

### 基本语法

```php
function functionName(parameter1, parameter2, ...): returnType
{
    // 函数体
    return value;
}
```

### 基本示例

```php
<?php
declare(strict_types=1);

function greet(string $name): string
{
    return "Hello, {$name}!";
}

echo greet("Alice") . "\n";  // Hello, Alice!
```

## 参数类型

### 标量类型

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

function calculate(float $price, int $quantity): float
{
    return $price * $quantity;
}

echo add(5, 3) . "\n";              // 8
echo calculate(19.99, 2) . "\n";   // 39.98
```

### 数组类型

```php
<?php
declare(strict_types=1);

function sum(array $numbers): int
{
    return array_sum($numbers);
}

echo sum([1, 2, 3, 4, 5]) . "\n";  // 15
```

### 对象类型

```php
<?php
declare(strict_types=1);

class User
{
    public string $name;
}

function greetUser(User $user): string
{
    return "Hello, {$user->name}!";
}

$user = new User();
$user->name = 'Alice';
echo greetUser($user) . "\n";  // Hello, Alice!
```

### 可调用类型

```php
<?php
declare(strict_types=1);

function execute(callable $callback, mixed $data): mixed
{
    return $callback($data);
}

$result = execute(fn($x) => $x * 2, 5);
echo $result . "\n";  // 10
```

## 默认参数值

### 基本用法

```php
<?php
declare(strict_types=1);

function greet(string $name, string $title = 'Mr.'): string
{
    return "Hello, {$title} {$name}!";
}

echo greet("Alice") . "\n";           // Hello, Mr. Alice!
echo greet("Alice", "Ms.") . "\n";    // Hello, Ms. Alice!
```

### 默认值规则

- 默认参数必须放在非默认参数之后
- 默认值必须是常量表达式，不能是变量或函数调用

```php
<?php
declare(strict_types=1);

// 正确
function example1(string $name, int $age = 0): void {}

// 错误：默认参数不能在非默认参数之前
// function example2(string $name = 'Guest', int $age): void {}
```

## 可选参数与可空类型

### 可空类型（Nullable Types）

使用 `?Type` 表示参数可以为 `null`：

```php
<?php
declare(strict_types=1);

function findUser(int $id): ?User
{
    // 如果找到返回 User 对象，否则返回 null
    return $users[$id] ?? null;
}

$user = findUser(1);
if ($user !== null) {
    echo $user->name;
}
```

### 可选参数

```php
<?php
declare(strict_types=1);

function createUser(string $name, ?string $email = null): array
{
    return [
        'name' => $name,
        'email' => $email ?? 'N/A'
    ];
}

$user1 = createUser('Alice');
$user2 = createUser('Bob', 'bob@example.com');
```

## 联合类型（PHP 8.0+）

### 基本语法

```php
function process(int|string $value): int|string
{
    return $value;
}
```

### 示例

```php
<?php
declare(strict_types=1);

function formatId(int|string $id): string
{
    return (string) $id;
}

function parseValue(int|string|float $value): mixed
{
    if (is_int($value)) {
        return $value * 2;
    } elseif (is_string($value)) {
        return strtoupper($value);
    } else {
        return round($value, 2);
    }
}
```

## 返回值类型

### 基本类型

```php
<?php
declare(strict_types=1);

function getAge(): int
{
    return 25;
}

function getName(): string
{
    return "Alice";
}

function isActive(): bool
{
    return true;
}
```

### 可空返回类型

```php
<?php
declare(strict_types=1);

function findUser(int $id): ?User
{
    return $users[$id] ?? null;
}
```

### void 返回类型

```php
<?php
declare(strict_types=1);

function logMessage(string $message): void
{
    echo $message . "\n";
    // 不需要 return 语句
}
```

### 联合返回类型

```php
<?php
declare(strict_types=1);

function getValue(bool $flag): int|string
{
    return $flag ? 42 : "forty-two";
}
```

## 命名参数（PHP 8.0+）

### 基本语法

```php
functionName(param1: value1, param2: value2);
```

### 基本用法

```php
<?php
declare(strict_types=1);

function createUser(string $name, int $age, string $email = ''): array
{
    return [
        'name' => $name,
        'age' => $age,
        'email' => $email
    ];
}

// 使用命名参数
$user = createUser(
    name: 'Alice',
    age: 25,
    email: 'alice@example.com'
);

// 可以改变参数顺序
$user2 = createUser(
    email: 'bob@example.com',
    name: 'Bob',
    age: 30
);
```

### 混合使用

```php
<?php
declare(strict_types=1);

function example(string $a, int $b, string $c = 'default'): void {}

// 位置参数和命名参数混合使用
example('first', b: 2, c: 'third');
```

## 早返回（Early Return）

使用 `return` 语句提前退出函数：

```php
<?php
declare(strict_types=1);

function processUser(?User $user): string
{
    if ($user === null) {
        return 'User not found';
    }
    
    if (!$user->isActive()) {
        return 'User is inactive';
    }
    
    return "Processing user: {$user->name}";
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class FunctionBasics
{
    public static function demonstrate(): void
    {
        echo "=== 基本函数 ===\n";
        echo greet("Alice") . "\n";
        
        echo "\n=== 类型声明 ===\n";
        echo add(5, 3) . "\n";
        echo calculate(19.99, 2) . "\n";
        
        echo "\n=== 默认参数 ===\n";
        echo greet("Alice") . "\n";
        echo greet("Alice", "Ms.") . "\n";
        
        echo "\n=== 可空类型 ===\n";
        $user = findUser(1);
        echo $user !== null ? $user->name : "Not found" . "\n";
        
        echo "\n=== 命名参数 ===\n";
        $user = createUser(name: 'Alice', age: 25, email: 'alice@example.com');
        print_r($user);
    }
}

function greet(string $name, string $title = 'Mr.'): string
{
    return "Hello, {$title} {$name}!";
}

function add(int $a, int $b): int
{
    return $a + $b;
}

function calculate(float $price, int $quantity): float
{
    return $price * $quantity;
}

function findUser(int $id): ?User
{
    // 模拟查找
    return null;
}

function createUser(string $name, int $age, string $email = ''): array
{
    return [
        'name' => $name,
        'age' => $age,
        'email' => $email
    ];
}

FunctionBasics::demonstrate();
```

## 注意事项

1. **严格模式**：始终在文件开头使用 `declare(strict_types=1);`

2. **类型提示位置**：类型提示只能用于函数参数和返回值，不能用于变量

3. **默认值**：默认参数必须放在非默认参数之后

4. **命名参数**：命名参数可以提高代码可读性，特别是对于参数较多的函数

5. **早返回**：使用早返回可以减少嵌套，提高代码可读性

## 练习

1. 创建一个函数，使用类型声明和默认参数实现用户信息格式化。

2. 编写一个函数，使用联合类型处理多种类型的输入。

3. 实现一个函数，使用命名参数提高代码可读性。

4. 创建一个函数，使用早返回优化代码结构。

5. 编写一个函数，演示可空类型和可选参数的区别。
