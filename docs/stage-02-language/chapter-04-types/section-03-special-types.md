# 2.4.3 特殊类型

## 概述

特殊类型包括 `null` 类型和 `resource` 类型，它们在 PHP 中有特殊的用途和处理方式。本节详细介绍这两种特殊类型的语法、检测方法、使用场景和注意事项。

理解特殊类型对于编写健壮的 PHP 代码很重要。`null` 表示"无值"，是可选值的重要表示方式；`resource` 是外部资源的句柄，在 PHP 8.0+ 中已对象化。

## 特性

- **null 类型**：表示"无值"，只有一个值 `null`，不区分大小写
- **resource 类型**：外部资源的句柄，PHP 8.0+ 已对象化，不再是独立的 resource 类型
- **空合并运算符**：`??` 运算符用于处理 null 值，提供默认值

## 语法/定义

### null 类型

**值**：`null`（不区分大小写，`NULL`、`Null` 都可以）

**类型检测**：
- `is_null(mixed $value): bool`：检测是否为 null
- `$value === null`：严格比较是否为 null

**特点**：
- 表示变量没有值
- 是可选值的表示方式
- 在类型系统中是独立的类型（PHP 8.0+）

### resource 类型

**创建**：通过函数返回（如 `fopen()`、`curl_init()`、`mysql_connect()` 等）

**类型检测**：
- `is_resource(mixed $value): bool`：检测是否为 resource 类型（PHP 7.x）
- `$value instanceof ResourceType`：检测是否为资源对象（PHP 8.0+）

**特点**：
- PHP 7.x：独立的 resource 类型
- PHP 8.0+：资源对象化，不再是 resource 类型，而是特定类的对象
- 需要及时释放，避免资源泄漏

### 空合并运算符（??）

**语法**：`$value ?? $default`

**作用**：
- 如果 `$value` 存在且不为 `null`，返回 `$value`
- 否则返回 `$default`

**链式使用**：`$value ?? $default1 ?? $default2`

## 基本用法

### 示例 1：null 类型

```php
<?php
declare(strict_types=1);

// null 值
$value = null;
var_dump($value);  // NULL

// 类型检测
var_dump(is_null($value));     // bool(true)
var_dump($value === null);      // bool(true)
var_dump($value == null);       // bool(true) - 但不推荐

// null 不区分大小写
$value1 = null;
$value2 = NULL;
$value3 = Null;
var_dump($value1 === $value2);  // bool(true)
var_dump($value1 === $value3);  // bool(true)

// 未定义的变量
$undefined = null;  // 显式设置为 null
var_dump(is_null($undefined));  // bool(true)

// 函数返回 null
function getValue(): ?string
{
    return null;  // 返回 null
}

$result = getValue();
var_dump(is_null($result));  // bool(true)
```

**执行**：

```bash
php null-example.php
```

**输出**：

```
NULL
bool(true)
bool(true)
bool(true)
bool(true)
bool(true)
bool(true)
```

### 示例 2：空合并运算符

```php
<?php
declare(strict_types=1);

// 基本用法
$name = null;
$default = "Guest";
$result = $name ?? $default;
echo "Name: {$result}\n";  // Name: Guest

// 变量存在时
$name = "Alice";
$result = $name ?? $default;
echo "Name: {$result}\n";  // Name: Alice

// 链式使用
$config1 = null;
$config2 = null;
$config3 = "Default Config";
$result = $config1 ?? $config2 ?? $config3;
echo "Config: {$result}\n";  // Config: Default Config

// 与 isset() 的区别
$arr = ['key' => 'value'];
$value1 = $arr['key'] ?? 'default';      // 'value'
$value2 = $arr['nonexistent'] ?? 'default';  // 'default'
echo "Value 1: {$value1}\n";
echo "Value 2: {$value2}\n";
```

**执行**：

```bash
php null-coalescing-example.php
```

**输出**：

```
Name: Guest
Name: Alice
Config: Default Config
Value 1: value
Value 2: default
```

### 示例 3：resource 类型（PHP 7.x）

```php
<?php
declare(strict_types=1);

// PHP 7.x：resource 类型
$file = fopen('test.txt', 'r');
if ($file !== false) {
    var_dump(is_resource($file));  // bool(true)
    var_dump(get_resource_type($file));  // "stream"
    
    // 使用资源
    $content = fread($file, 1024);
    
    // 释放资源
    fclose($file);
    var_dump(is_resource($file));  // bool(false) - 已关闭
}
```

### 示例 4：resource 对象化（PHP 8.0+）

```php
<?php
declare(strict_types=1);

// PHP 8.0+：资源对象化
$file = fopen('test.txt', 'r');
if ($file !== false) {
    // 不再是 resource 类型，而是对象
    var_dump(is_resource($file));  // bool(false)
    var_dump(is_object($file));    // bool(true)
    
    // 使用 instanceof 检测
    var_dump($file instanceof \SplFileObject);  // 可能是 true 或 false，取决于具体类型
    
    // 使用资源
    $content = fread($file, 1024);
    
    // 释放资源（对象会被自动销毁）
    fclose($file);
}
```

## 完整代码示例

### 示例 1：null 值处理

```php
<?php
declare(strict_types=1);

function getUserName(?int $userId): ?string
{
    // 模拟数据库查询
    $users = [
        1 => "Alice",
        2 => "Bob",
    ];
    
    if ($userId === null) {
        return null;
    }
    
    return $users[$userId] ?? null;
}

// 使用空合并运算符提供默认值
$userId = 1;
$userName = getUserName($userId) ?? "Guest";
echo "User: {$userName}\n";  // User: Alice

$userId = 999;
$userName = getUserName($userId) ?? "Guest";
echo "User: {$userName}\n";  // User: Guest

$userId = null;
$userName = getUserName($userId) ?? "Guest";
echo "User: {$userName}\n";  // User: Guest
```

### 示例 2：资源管理

```php
<?php
declare(strict_types=1);

// 正确的资源管理
function readFileContent(string $filename): ?string
{
    $file = fopen($filename, 'r');
    if ($file === false) {
        return null;
    }
    
    try {
        $content = '';
        while (!feof($file)) {
            $content .= fread($file, 8192);
        }
        return $content;
    } finally {
        // 确保资源被释放
        fclose($file);
    }
}

$content = readFileContent('test.txt');
if ($content !== null) {
    echo "Content: {$content}\n";
} else {
    echo "Failed to read file\n";
}
```

### 示例 3：空合并运算符链式使用

```php
<?php
declare(strict_types=1);

// 配置优先级：环境变量 > 配置文件 > 默认值
function getConfig(string $key): string
{
    // 尝试从环境变量获取
    $value = $_ENV[$key] ?? null;
    
    // 如果环境变量不存在，尝试从配置文件获取
    $value = $value ?? getConfigFromFile($key);
    
    // 如果配置文件不存在，使用默认值
    $value = $value ?? getDefaultValue($key);
    
    return $value;
}

// 或使用链式语法
function getConfigChain(string $key): string
{
    return $_ENV[$key] 
        ?? getConfigFromFile($key) 
        ?? getDefaultValue($key) 
        ?? '';
}

function getConfigFromFile(string $key): ?string
{
    // 模拟从配置文件读取
    return null;
}

function getDefaultValue(string $key): ?string
{
    $defaults = [
        'db_host' => 'localhost',
        'db_port' => '3306',
    ];
    return $defaults[$key] ?? null;
}
```

## 使用场景

### null 类型

- **可选值**：表示可选参数、可选返回值
- **占位符**：表示尚未初始化的值
- **空值表示**：表示"无值"的状态
- **数据库 NULL**：表示数据库中的 NULL 值

### resource 类型

- **文件操作**：文件句柄（`fopen()`、`opendir()`）
- **数据库连接**：数据库连接句柄（已废弃的函数）
- **网络连接**：网络资源句柄（`curl_init()`）
- **图像处理**：图像资源（`imagecreate()`）

**注意**：PHP 8.0+ 中，大多数资源已对象化，不再是 resource 类型。

### 空合并运算符

- **默认值**：为可能为 null 的值提供默认值
- **配置获取**：从多个来源获取配置，使用优先级
- **简化代码**：简化 null 值检查代码

## 注意事项

### null 类型

- **严格比较**：使用 `=== null` 而不是 `== null` 进行严格比较
- **类型声明**：使用 `?Type` 表示可空类型（PHP 7.1+）
- **null 安全**：注意 null 值可能导致错误，使用空合并运算符或条件检查

### resource 类型

- **及时释放**：使用完资源后及时释放，避免资源泄漏
- **错误处理**：检查资源创建是否成功（返回 `false` 表示失败）
- **PHP 8.0+ 变化**：资源已对象化，使用 `is_object()` 和 `instanceof` 检测

### 空合并运算符

- **与 isset() 的区别**：`??` 检查变量是否存在且不为 null，`isset()` 检查变量是否已设置且不为 null
- **链式使用**：可以链式使用多个 `??` 运算符
- **性能**：`??` 运算符性能较好

## 常见问题

### 问题 1：null 判断错误

**症状**：null 判断结果不符合预期

**原因**：使用 `==` 判断 null，可能产生意外的类型转换

**错误示例**：

```php
<?php
declare(strict_types=1);

$value = 0;
if ($value == null) {  // true - 意外的结果
    echo "Value is null\n";
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$value = 0;
if ($value === null) {  // false - 正确的结果
    echo "Value is null\n";
} else {
    echo "Value is not null\n";
}

// 或使用 is_null()
if (is_null($value)) {
    echo "Value is null\n";
}
```

### 问题 2：资源泄漏

**症状**：程序运行时间越长，资源占用越多

**原因**：未关闭资源

**错误示例**：

```php
<?php
declare(strict_types=1);

function readFile(string $filename): string
{
    $file = fopen($filename, 'r');
    $content = fread($file, 1024);
    // 忘记关闭文件
    return $content;
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

function readFile(string $filename): string
{
    $file = fopen($filename, 'r');
    if ($file === false) {
        throw new RuntimeException("Failed to open file");
    }
    
    try {
        $content = fread($file, 1024);
        return $content;
    } finally {
        // 确保资源被释放
        fclose($file);
    }
}
```

### 问题 3：空合并运算符理解错误

**症状**：空合并运算符结果不符合预期

**原因**：不理解 `??` 运算符的检查规则

**说明**：

```php
<?php
declare(strict_types=1);

// ?? 检查变量是否存在且不为 null
$value1 = null;
$result1 = $value1 ?? 'default';  // 'default'

$value2 = 0;
$result2 = $value2 ?? 'default';  // 0（不是 'default'）

$value3 = '';
$result3 = $value3 ?? 'default';  // ''（不是 'default'）

$value4 = false;
$result4 = $value4 ?? 'default';  // false（不是 'default'）

// 只有 null 或未定义才会使用默认值
```

## 最佳实践

### null 处理

- **使用严格比较**：使用 `=== null` 或 `is_null()` 检查 null
- **使用可空类型**：使用 `?Type` 表示可空类型
- **使用空合并运算符**：使用 `??` 提供默认值
- **文档说明**：为可空参数和返回值添加文档说明

### 资源管理

- **及时释放**：使用完资源后立即释放
- **使用 try-finally**：使用 `try-finally` 确保资源被释放
- **检查返回值**：检查资源创建是否成功
- **理解 PHP 8.0+ 变化**：理解资源对象化的变化

### 空合并运算符

- **简化代码**：使用 `??` 简化 null 值检查
- **链式使用**：使用链式 `??` 实现优先级
- **理解规则**：理解 `??` 的检查规则（只检查 null，不检查其他假值）

## 对比分析

### null vs false vs 0 vs ""

| 值 | 类型 | 布尔值 | === null | == null |
|:---|:-----|:-------|:---------|:--------|
| `null` | null | false | true | true |
| `false` | bool | false | false | true |
| `0` | int | false | false | true |
| `""` | string | false | false | true |

**选择建议**：
- **表示"无值"**：使用 `null`
- **表示"假"**：使用 `false`
- **表示"零"**：使用 `0`
- **表示"空字符串"**：使用 `""`

### ?? vs isset() vs empty()

| 特性 | ?? | isset() | empty() |
|:-----|:---|:--------|:---------|
| 检查 null | 是 | 是 | 是 |
| 检查未定义 | 是 | 是 | 是 |
| 检查其他假值 | 否 | 否 | 是 |
| 返回值 | 值或默认值 | bool | bool |

**选择建议**：
- **提供默认值**：使用 `??`
- **检查变量是否存在**：使用 `isset()`
- **检查是否为假值**：使用 `empty()`

## 相关章节

- **2.4.1 标量类型**：了解基本数据类型
- **2.4.4 类型检测**：了解类型检测方法
- **2.12 isset / empty / Null 体系**：详细了解 null 处理
- **阶段四：系统编程**：详细了解资源管理

## 练习任务

1. **null 处理练习**：
   - 练习使用 `is_null()` 和 `=== null` 检查 null
   - 练习使用可空类型声明
   - 练习使用空合并运算符提供默认值
   - 理解 null 与其他假值的区别

2. **空合并运算符练习**：
   - 练习基本用法
   - 练习链式使用
   - 实现配置优先级获取
   - 理解与 `isset()` 的区别

3. **资源管理练习**：
   - 练习文件资源的管理
   - 使用 `try-finally` 确保资源释放
   - 检查资源创建是否成功
   - 理解 PHP 8.0+ 的资源对象化

4. **综合练习**：
   - 创建一个配置管理类，使用空合并运算符
   - 实现一个文件读取函数，正确处理资源
   - 实现一个用户查询函数，正确处理 null 值
   - 测试不同场景下的 null 和资源处理

5. **最佳实践练习**：
   - 为函数添加可空类型声明
   - 使用空合并运算符简化代码
   - 确保资源正确释放
   - 进行代码审查，确保 null 和资源处理正确
