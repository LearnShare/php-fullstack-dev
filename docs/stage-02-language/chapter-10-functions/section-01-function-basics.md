# 2.10.1 函数基础

## 概述

函数是代码复用的基本单元。本节详细介绍函数定义、参数类型、默认值、可选参数、可空类型、联合类型、返回值类型、早返回、命名参数（PHP 8.0+）等函数基础知识。

理解函数的基础特性对于编写模块化、可维护的代码非常重要。掌握类型声明、默认值、命名参数等特性可以提高代码质量和开发效率。

## 特性

- **类型声明**：支持参数和返回值类型声明
- **默认值**：参数可以有默认值
- **命名参数**：可以按名称传递参数（PHP 8.0+）
- **早返回**：支持早期返回，提高代码可读性

## 语法/定义

### 函数定义

**基本语法**：
```php
function functionName(Type $param = default): ReturnType
{
    // 函数体
    return $value;
}
```

**组成部分**：
- 函数名：遵循 PHP 命名规则
- 参数列表：可以有多个参数，支持类型声明和默认值
- 返回值类型：可选，指定返回值的类型
- 函数体：包含函数执行的代码

### 参数类型声明

**标量类型**：`int`、`float`、`string`、`bool`

**复合类型**：`array`、`object`、`callable`

**特殊类型**：
- `?Type`：可空类型（`?string` 等价于 `string|null`）
- `Type1|Type2`：联合类型（PHP 8.0+）

### 返回值类型

**类型声明**：`: ReturnType`

**特殊类型**：
- `void`：无返回值
- `?ReturnType`：可空返回值
- `Type1|Type2`：联合返回值类型（PHP 8.0+）

### 命名参数（PHP 8.0+）

**语法**：`functionName(param1: $value1, param2: $value2)`

**特点**：
- 可以按名称传递参数
- 可以跳过有默认值的参数
- 提高代码可读性

## 基本用法

### 示例 1：基础函数

```php
<?php
declare(strict_types=1);

// 基本函数定义
function greet(string $name): string
{
    return "Hello, {$name}!\n";
}

echo greet("World");  // Hello, World!

// 无返回值函数
function logMessage(string $message): void
{
    echo "[LOG] {$message}\n";
}

logMessage("Application started");
```

### 示例 2：默认值参数

```php
<?php
declare(strict_types=1);

// 带默认值的参数
function greet(string $name, string $title = "Mr."): string
{
    return "Hello, {$title} {$name}!\n";
}

echo greet("John");           // Hello, Mr. John!
echo greet("Jane", "Ms.");    // Hello, Ms. Jane!

// 多个默认值参数
function createUser(
    string $name,
    int $age,
    string $email = "",
    bool $active = true
): array {
    return [
        'name' => $name,
        'age' => $age,
        'email' => $email,
        'active' => $active
    ];
}
```

### 示例 3：可空类型和联合类型

```php
<?php
declare(strict_types=1);

// 可空类型
function findUser(?int $id): ?array
{
    if ($id === null) {
        return null;
    }
    // 查找用户逻辑
    return ['id' => $id, 'name' => 'John'];
}

$user = findUser(1);
$user = findUser(null);  // 返回 null

// 联合类型（PHP 8.0+）
function processValue(string|int $value): string|int
{
    if (is_string($value)) {
        return strtoupper($value);
    }
    return $value * 2;
}

echo processValue("hello") . "\n";  // HELLO
echo processValue(5) . "\n";        // 10
```

### 示例 4：命名参数（PHP 8.0+）

```php
<?php
declare(strict_types=1);

function createUser(
    string $name,
    int $age,
    string $email = "",
    bool $active = true
): array {
    return [
        'name' => $name,
        'age' => $age,
        'email' => $email,
        'active' => $active
    ];
}

// 使用命名参数
$user = createUser(
    name: "John",
    age: 25,
    active: false
);

// 可以跳过有默认值的参数
$user = createUser(
    name: "Jane",
    age: 30,
    email: "jane@example.com"
);
```

### 示例 5：早返回（Early Return）

```php
<?php
declare(strict_types=1);

function processUser(?array $user): string
{
    // 早期返回：处理 null 情况
    if ($user === null) {
        return "User not found";
    }
    
    // 早期返回：处理无效数据
    if (!isset($user['name']) || !isset($user['age'])) {
        return "Invalid user data";
    }
    
    // 正常处理
    return "Processing user: {$user['name']}, age: {$user['age']}";
}

// 使用
echo processUser(['name' => 'John', 'age' => 25]) . "\n";
echo processUser(null) . "\n";
echo processUser(['name' => 'Jane']) . "\n";
```

## 完整代码示例

### 示例 1：用户管理函数

```php
<?php
declare(strict_types=1);

class UserManager
{
    public static function createUser(
        string $name,
        int $age,
        string $email = "",
        bool $verified = false
    ): array {
        return [
            'name' => $name,
            'age' => $age,
            'email' => $email,
            'verified' => $verified,
            'created_at' => time()
        ];
    }
    
    public static function validateUser(array $user): bool
    {
        // 早期返回：检查必需字段
        if (!isset($user['name']) || empty($user['name'])) {
            return false;
        }
        
        if (!isset($user['age']) || $user['age'] < 0) {
            return false;
        }
        
        // 验证通过
        return true;
    }
    
    public static function getUserInfo(array $user): string
    {
        $name = $user['name'] ?? 'Unknown';
        $age = $user['age'] ?? 0;
        $email = $user['email'] ?? 'N/A';
        
        return "Name: {$name}, Age: {$age}, Email: {$email}";
    }
}

// 使用
$user = UserManager::createUser(
    name: "John",
    age: 25,
    email: "john@example.com"
);

var_dump(UserManager::validateUser($user));  // bool(true)
echo UserManager::getUserInfo($user) . "\n";
```

### 示例 2：配置处理函数

```php
<?php
declare(strict_types=1);

function getConfig(
    array $config,
    string $key,
    mixed $default = null
): mixed {
    return $config[$key] ?? $default;
}

function setConfig(
    array &$config,
    string $key,
    mixed $value
): void {
    $config[$key] = $value;
}

function mergeConfigs(
    array $defaults,
    array $overrides
): array {
    return array_merge($defaults, $overrides);
}

// 使用
$defaults = ['host' => 'localhost', 'port' => 3306];
$overrides = ['port' => 5432];

$config = mergeConfigs($defaults, $overrides);
echo getConfig($config, 'host') . "\n";  // localhost
echo getConfig($config, 'port') . "\n";  // 5432
```

### 示例 3：类型安全的计算函数

```php
<?php
declare(strict_types=1);

function calculate(
    float $a,
    float $b,
    string $operation = 'add'
): float {
    return match($operation) {
        'add' => $a + $b,
        'subtract' => $a - $b,
        'multiply' => $a * $b,
        'divide' => $b != 0 ? $a / $b : throw new DivisionByZeroError(),
        default => throw new InvalidArgumentException("Unknown operation: {$operation}")
    };
}

function safeDivide(float $a, float $b): ?float
{
    if ($b == 0) {
        return null;  // 早期返回
    }
    return $a / $b;
}

// 使用
echo calculate(10, 5, 'add') . "\n";      // 15
echo calculate(10, 5, 'multiply') . "\n"; // 50

$result = safeDivide(10, 0);
var_dump($result);  // NULL
```

## 使用场景

### 代码复用

- **提取重复代码**：将重复代码提取为函数
- **功能模块化**：将功能模块化为函数
- **工具函数**：创建可重用的工具函数

### 类型安全

- **类型声明**：使用类型声明提高代码质量
- **参数验证**：通过类型声明自动验证参数
- **返回值保证**：通过返回值类型保证返回值的类型

### 代码组织

- **逻辑分离**：将复杂逻辑分离到函数中
- **可读性**：使用有意义的函数名提高可读性
- **可维护性**：函数化代码更容易维护

## 注意事项

### 类型声明

- **严格模式**：启用严格模式后类型必须匹配
- **类型转换**：非严格模式下会进行类型转换
- **类型检查**：类型声明在运行时检查

### 默认值

- **参数顺序**：有默认值的参数必须在无默认值参数之后
- **命名参数**：使用命名参数可以跳过有默认值的参数
- **null 默认值**：可以使用 `null` 作为默认值

### 命名参数

- **PHP 8.0+**：需要 PHP 8.0 或更高版本
- **位置参数**：命名参数必须在位置参数之后
- **可读性**：提高代码可读性，特别是参数较多的函数

## 常见问题

### 问题 1：类型不匹配错误

**症状**：传递的类型与声明不匹配

**原因**：类型声明与实际值不匹配

**错误示例**：

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

echo add("1", "2");  // TypeError: Argument #1 must be of type int, string given
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：传递正确类型
echo add(1, 2) . "\n";  // 3

// 方法2：使用类型转换
echo add((int) "1", (int) "2") . "\n";  // 3

// 方法3：使用联合类型
function add(string|int $a, string|int $b): int
{
    return (int) $a + (int) $b;
}
```

### 问题 2：默认值参数位置

**症状**：函数调用错误

**原因**：有默认值的参数在无默认值参数之前

**错误示例**：

```php
<?php
declare(strict_types=1);

function greet(string $title = "Mr.", string $name): string
{
    return "Hello, {$title} {$name}!\n";
}

echo greet("John");  // 错误：$name 没有值
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：调整参数顺序
function greet(string $name, string $title = "Mr."): string
{
    return "Hello, {$title} {$name}!\n";
}

echo greet("John");  // Hello, Mr. John!

// 方法2：使用命名参数（PHP 8.0+）
function greet(string $title = "Mr.", string $name = ""): string
{
    return "Hello, {$title} {$name}!\n";
}

echo greet(name: "John");  // Hello, Mr. John!
```

### 问题 3：返回值类型不匹配

**症状**：返回值类型错误

**原因**：返回值与声明类型不匹配

**错误示例**：

```php
<?php
declare(strict_types=1);

function getValue(): int
{
    return "123";  // TypeError: Return value must be of type int, string returned
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：返回正确类型
function getValue(): int
{
    return 123;
}

// 方法2：使用类型转换
function getValue(): int
{
    return (int) "123";
}

// 方法3：使用联合类型
function getValue(): int|string
{
    return "123";
}
```

## 最佳实践

### 函数设计

- **单一职责**：函数应该只做一件事
- **有意义的名称**：使用有意义的函数名
- **参数数量**：避免参数过多（建议不超过 5 个）

### 类型声明

- **始终使用**：始终使用类型声明
- **严格模式**：启用严格模式
- **联合类型**：合理使用联合类型（PHP 8.0+）

### 代码组织

- **早期返回**：使用早期返回提高可读性
- **避免深层嵌套**：避免过度嵌套
- **提取函数**：复杂逻辑提取到函数中

## 对比分析

### 位置参数 vs 命名参数

| 特性 | 位置参数 | 命名参数 |
|:-----|:---------|:---------|
| 可读性 | 中 | 高 |
| 灵活性 | 低 | 高 |
| 版本要求 | 所有版本 | PHP 8.0+ |
| 推荐度 | 推荐（简单） | 推荐（复杂） |

**选择建议**：
- **简单函数**：使用位置参数
- **参数较多**：使用命名参数提高可读性

### 可空类型 vs 联合类型

| 特性 | 可空类型（?Type） | 联合类型（Type1\|Type2） |
|:-----|:------------------|:------------------------|
| 语法 | `?string` | `string\|int` |
| 等价 | `string\|null` | 多种类型 |
| 推荐度 | 推荐（null） | 推荐（多种类型） |

**选择建议**：
- **null 值**：使用可空类型
- **多种类型**：使用联合类型

## 相关章节

- **2.4 数据类型**：了解数据类型
- **2.5 类型转换与比较**：了解类型转换
- **2.10.2 可变参数**：了解可变参数

## 练习任务

1. **函数定义练习**：
   - 练习定义各种类型的函数
   - 理解类型声明的作用
   - 测试各种参数组合
   - 观察类型检查行为

2. **默认值练习**：
   - 练习使用默认值参数
   - 理解参数顺序的重要性
   - 测试命名参数的使用
   - 测试各种参数场景

3. **类型声明练习**：
   - 练习使用可空类型和联合类型
   - 理解类型检查的行为
   - 测试类型不匹配的情况
   - 观察错误处理

4. **实际应用练习**：
   - 实现用户管理函数
   - 实现配置处理函数
   - 实现类型安全的计算函数
   - 测试各种应用场景

5. **综合练习**：
   - 创建一个函数库
   - 实现各种功能函数
   - 使用类型声明提高代码质量
   - 进行代码审查，确保正确性
