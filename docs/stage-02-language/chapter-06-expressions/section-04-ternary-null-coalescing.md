# 2.6.4 三元运算符与空合并运算符

## 概述

三元运算符和空合并运算符是简化条件赋值的常用方法。本节详细介绍三元运算符（`?:`）、省略中间值语法、空合并运算符（`??`）、空合并赋值运算符（`??=`）的语法、用法、优先级及使用场景。

理解这些运算符的区别和适用场景对于编写简洁、安全的代码很重要。空合并运算符专门用于处理 `null` 值，比三元运算符更安全。

## 特性

- **简洁性**：简化条件赋值代码
- **可读性**：提高代码可读性
- **安全性**：空合并运算符避免未定义变量错误
- **链式使用**：支持链式使用

## 语法/定义

### 三元运算符（?:）

**完整语法**：`condition ? value_if_true : value_if_false`

**功能**：根据条件返回不同的值

**省略中间值语法**：`value ?: default`

**功能**：当 `value` 为 truthy 时返回 `value`，否则返回 `default`

**特点**：
- 是表达式，可以赋值
- 支持嵌套（但不推荐）
- 优先级较低

### 空合并运算符（??）（PHP 7.0+）

**语法**：`$a ?? $b`

**功能**：如果 `$a` 存在且不为 `null`，返回 `$a`，否则返回 `$b`

**特点**：
- 只检查 `null`，不检查其他 falsy 值
- 不会产生未定义变量警告
- 支持链式使用

### 空合并赋值运算符（??=）（PHP 7.4+）

**语法**：`$a ??= $b`

**功能**：如果 `$a` 为 `null`，将 `$b` 赋值给 `$a`

**等价于**：`$a = $a ?? $b`

**特点**：
- 简化空合并赋值
- 只检查 `null`

## 基本用法

### 示例 1：三元运算符

```php
<?php
declare(strict_types=1);

// 基本使用
$age = 18;
$status = $age >= 18 ? "Adult" : "Minor";
echo "Status: {$status}\n";  // Status: Adult

// 嵌套使用（不推荐）
$score = 85;
$grade = $score >= 90 ? "A" : ($score >= 80 ? "B" : "C");
echo "Grade: {$grade}\n";  // Grade: B

// 省略中间值
$userName = "Alice";
$name = $userName ?: "Guest";
echo "Name: {$name}\n";  // Name: Alice

$userName = "";
$name = $userName ?: "Guest";
echo "Name: {$name}\n";  // Name: Guest
```

### 示例 2：空合并运算符

```php
<?php
declare(strict_types=1);

// 基本使用
$userName = null;
$name = $userName ?? "Guest";
echo "Name: {$name}\n";  // Name: Guest

$userName = "Alice";
$name = $userName ?? "Guest";
echo "Name: {$name}\n";  // Name: Alice

// 与 falsy 值的区别
$value = 0;
$result1 = $value ?: "default";  // "default"（0 是 falsy）
$result2 = $value ?? "default";  // 0（0 不是 null）

echo "Ternary: {$result1}\n";  // Ternary: default
echo "Null coalescing: {$result2}\n";  // Null coalescing: 0

// 链式使用
$value = $a ?? $b ?? $c ?? "default";
echo "Value: {$value}\n";  // Value: default（如果 $a、$b、$c 都是 null）
```

### 示例 3：空合并赋值运算符

```php
<?php
declare(strict_types=1);

// 基本使用
$name = null;
$name ??= "Guest";
echo "Name: {$name}\n";  // Name: Guest

$name = "Alice";
$name ??= "Guest";
echo "Name: {$name}\n";  // Name: Alice（不会改变）

// 配置处理
$config = [];
$config['timeout'] ??= 30;
$config['retries'] ??= 3;
print_r($config);
```

### 示例 4：实际应用

```php
<?php
declare(strict_types=1);

// 从数组读取配置
function getConfig(array $config, string $key, mixed $default = null): mixed
{
    return $config[$key] ?? $default;
}

// 从多个来源读取值
function getUserName(?array $session, ?array $cookie, ?string $param): string
{
    return $param ?? $session['username'] ?? $cookie['username'] ?? "Guest";
}

// 使用
$config = ['timeout' => 60];
echo "Timeout: " . getConfig($config, 'timeout', 30) . "\n";  // Timeout: 60
echo "Retries: " . getConfig($config, 'retries', 3) . "\n";   // Retries: 3

$session = ['username' => 'Alice'];
$cookie = null;
$param = null;
echo "Username: " . getUserName($session, $cookie, $param) . "\n";  // Username: Alice
```

## 完整代码示例

### 示例 1：配置管理

```php
<?php
declare(strict_types=1);

class Config
{
    private array $config = [];
    
    public function __construct(array $defaults = [])
    {
        $this->config = $defaults;
    }
    
    public function get(string $key, mixed $default = null): mixed
    {
        return $this->config[$key] ?? $default;
    }
    
    public function set(string $key, mixed $value): void
    {
        $this->config[$key] = $value;
    }
    
    public function ensure(string $key, mixed $default): void
    {
        $this->config[$key] ??= $default;
    }
    
    public function merge(array $config): void
    {
        foreach ($config as $key => $value) {
            $this->config[$key] ??= $value;
        }
    }
}

$config = new Config(['timeout' => 30, 'retries' => 3]);
$config->ensure('timeout', 60);  // 不会改变（已存在）
$config->ensure('debug', false);  // 会设置（不存在）

echo "Timeout: " . $config->get('timeout') . "\n";  // Timeout: 30
echo "Debug: " . ($config->get('debug') ? 'true' : 'false') . "\n";  // Debug: false
```

### 示例 2：用户信息处理

```php
<?php
declare(strict_types=1);

function formatUserInfo(?array $user): array
{
    return [
        'name' => $user['name'] ?? 'Anonymous',
        'email' => $user['email'] ?? 'N/A',
        'role' => $user['role'] ?? 'guest',
        'active' => $user['active'] ?? false,
    ];
}

function getUserDisplayName(?array $user): string
{
    // 优先使用昵称，然后是用户名，最后是默认值
    return $user['nickname'] ?? $user['username'] ?? 'Guest';
}

// 使用
$user1 = ['name' => 'Alice', 'email' => 'alice@example.com'];
$user2 = null;

$info1 = formatUserInfo($user1);
print_r($info1);

$info2 = formatUserInfo($user2);
print_r($info2);

echo "Display name: " . getUserDisplayName($user1) . "\n";
```

### 示例 3：API 响应处理

```php
<?php
declare(strict_types=1);

function processApiResponse(?array $response): array
{
    return [
        'status' => $response['status'] ?? 'unknown',
        'message' => $response['message'] ?? 'No message',
        'data' => $response['data'] ?? [],
        'timestamp' => $response['timestamp'] ?? time(),
    ];
}

function getApiError(?array $response): ?string
{
    // 使用三元运算符处理错误信息
    return isset($response['error']) 
        ? ($response['error']['message'] ?? 'Unknown error')
        : null;
}

// 使用
$response1 = ['status' => 'success', 'data' => ['id' => 1]];
$response2 = ['status' => 'error', 'error' => ['message' => 'Not found']];
$response3 = null;

$processed1 = processApiResponse($response1);
print_r($processed1);

$error = getApiError($response2);
echo "Error: {$error}\n";  // Error: Not found
```

## 使用场景

### 三元运算符

- **条件赋值**：根据条件赋值
- **简化 if-else**：简化简单的 if-else 语句
- **默认值**：为变量提供默认值（注意 falsy 值）

### 空合并运算符

- **null 处理**：专门处理 `null` 值
- **配置读取**：从配置数组读取值
- **链式默认值**：从多个来源读取值
- **避免警告**：避免未定义变量警告

### 空合并赋值运算符

- **配置初始化**：初始化配置值
- **默认值设置**：为变量设置默认值
- **合并配置**：合并配置数组

## 注意事项

### 优先级

- **三元运算符优先级较低**：需要注意与其他运算符的优先级
- **空合并运算符优先级较低**：但高于三元运算符
- **使用括号**：不确定时使用括号明确优先级

### null 检查

- **空合并运算符只检查 null**：不检查其他 falsy 值
- **三元运算符检查 truthy**：检查值是否为 truthy
- **选择合适运算符**：根据需求选择合适的运算符

### 可读性

- **避免过度嵌套**：避免过度嵌套三元运算符
- **提取为函数**：复杂逻辑提取为函数
- **使用 if-else**：复杂条件使用 if-else

## 常见问题

### 问题 1：三元运算符嵌套

**症状**：代码可读性差，难以理解

**原因**：过度嵌套三元运算符

**错误示例**：

```php
<?php
declare(strict_types=1);

$score = 85;
$grade = $score >= 90 ? "A" : ($score >= 80 ? "B" : ($score >= 70 ? "C" : ($score >= 60 ? "D" : "F")));
echo "Grade: {$grade}\n";  // 可读性差
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：使用 if-else
function getGrade(int $score): string
{
    if ($score >= 90) {
        return "A";
    } elseif ($score >= 80) {
        return "B";
    } elseif ($score >= 70) {
        return "C";
    } elseif ($score >= 60) {
        return "D";
    } else {
        return "F";
    }
}

// 方法2：使用 match 表达式（PHP 8.0+）
function getGradeMatch(int $score): string
{
    return match(true) {
        $score >= 90 => "A",
        $score >= 80 => "B",
        $score >= 70 => "C",
        $score >= 60 => "D",
        default => "F",
    };
}

$score = 85;
echo "Grade: " . getGrade($score) . "\n";  // Grade: B
```

### 问题 2：空合并运算符误用

**症状**：结果不符合预期

**原因**：不理解只检查 `null` 的特性

**错误示例**：

```php
<?php
declare(strict_types=1);

$value = 0;
$result = $value ?? "default";
echo "Result: {$result}\n";  // Result: 0（不是 "default"）

// 如果期望 0 时也使用默认值，应该使用三元运算符
$result = $value ?: "default";
echo "Result: {$result}\n";  // Result: default
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$value = 0;

// 如果只处理 null，使用空合并运算符
$result1 = $value ?? "default";  // 0

// 如果处理所有 falsy 值，使用三元运算符
$result2 = $value ?: "default";  // "default"

// 根据需求选择
if ($value === null) {
    // 使用 ??
} else {
    // 使用 ?:
}
```

### 问题 3：优先级混淆

**症状**：表达式结果不符合预期

**原因**：不理解运算符优先级

**错误示例**：

```php
<?php
declare(strict_types=1);

$a = null;
$b = "default";
$c = "fallback";

// 期望：($a ?? $b) ?: $c
$result = $a ?? $b ?: $c;  // 实际：$a ?? ($b ?: $c)
echo "Result: {$result}\n";  // Result: default（可能不符合预期）
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$a = null;
$b = "default";
$c = "fallback";

// 使用括号明确优先级
$result = ($a ?? $b) ?: $c;
echo "Result: {$result}\n";  // Result: default
```

## 最佳实践

### 运算符选择

- **处理 null**：使用空合并运算符（`??`）
- **处理 falsy**：使用三元运算符（`?:`）
- **条件赋值**：使用三元运算符
- **默认值**：优先使用空合并运算符

### 代码可读性

- **避免过度嵌套**：避免过度嵌套三元运算符
- **提取函数**：复杂逻辑提取为函数
- **使用 if-else**：复杂条件使用 if-else
- **使用 match**：多条件使用 match 表达式（PHP 8.0+）

### 性能考虑

- **短路求值**：利用短路求值优化性能
- **避免重复计算**：避免在条件中重复计算
- **缓存结果**：缓存复杂计算的结果

## 对比分析

### 三元运算符 vs 空合并运算符

| 特性 | 三元运算符（?:） | 空合并运算符（??） |
|:-----|:-----------------|:-------------------|
| 检查值 | truthy/falsy | null |
| 未定义变量 | 产生警告 | 不产生警告 |
| 适用场景 | 条件赋值 | null 处理 |
| 推荐度 | 按需使用 | 推荐（处理 null） |

**选择建议**：
- **处理 null**：使用空合并运算符
- **条件赋值**：使用三元运算符
- **默认值**：优先使用空合并运算符

### 省略中间值 vs 空合并运算符

| 特性 | 省略中间值（?:） | 空合并运算符（??） |
|:-----|:-----------------|:-------------------|
| 检查值 | truthy/falsy | null |
| 0 值 | 返回默认值 | 返回 0 |
| 空字符串 | 返回默认值 | 返回空字符串 |
| 推荐度 | 按需使用 | 推荐（处理 null） |

**选择建议**：
- **处理 null**：使用空合并运算符
- **处理 falsy**：使用省略中间值语法

## 相关章节

- **2.6.3 逻辑运算符**：了解逻辑运算符
- **2.6.5 match 表达式**：了解 match 表达式
- **2.6.6 运算符优先级与结合性**：了解运算符优先级
- **2.12 isset / empty / Null 体系**：详细了解 null 处理

## 练习任务

1. **三元运算符练习**：
   - 练习使用三元运算符进行条件赋值
   - 理解省略中间值语法
   - 测试各种条件组合
   - 避免过度嵌套

2. **空合并运算符练习**：
   - 练习使用空合并运算符处理 null
   - 理解与 falsy 值的区别
   - 练习链式使用
   - 测试各种场景

3. **空合并赋值运算符练习**：
   - 练习使用空合并赋值运算符
   - 实现配置初始化
   - 实现默认值设置
   - 测试各种场景

4. **实际应用练习**：
   - 实现配置管理类
   - 实现用户信息处理函数
   - 实现 API 响应处理函数
   - 测试各种场景

5. **综合练习**：
   - 创建一个使用各种条件运算符的程序
   - 实现复杂的条件逻辑
   - 选择合适的运算符
   - 进行代码审查，确保逻辑正确
