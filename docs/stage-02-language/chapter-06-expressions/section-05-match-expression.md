# 2.6.5 match 表达式

## 概述

`match` 表达式是 PHP 8.0 引入的新特性，是 `switch` 语句的改进版本。本节详细介绍 `match` 表达式的语法、与 `switch` 的对比、严格比较、返回值、表达式模式及完整示例。

`match` 表达式提供了比 `switch` 更安全、更简洁的语法，支持返回值，使用严格比较，不需要 `break` 语句。理解 `match` 表达式的特性可以帮助编写更现代的 PHP 代码。

## 特性

- **严格比较**：使用 `===` 进行比较，更安全
- **返回值**：可以直接返回值，是表达式
- **无贯穿**：不需要 `break`，自动中断
- **表达式模式**：支持更复杂的匹配模式
- **必须匹配**：如果没有匹配且没有 `default`，会抛出异常

## 语法/定义

### match 表达式语法

**基本语法**：
```php
match ($value) {
    pattern1 => result1,
    pattern2 => result2,
    default => default_result,
}
```

**特点**：
- 是表达式，可以赋值
- 使用 `=>` 连接模式和结果
- 使用 `,` 分隔多个分支
- 不需要 `break`
- 使用严格比较（`===`）

### 与 switch 对比

| 特性 | `switch` | `match` |
|:-----|:---------|:--------|
| 比较方式 | 宽松比较（`==`） | 严格比较（`===`） |
| 返回值 | 不能直接返回值 | 可以返回值 |
| break | 需要 `break` | 不需要 |
| 默认分支 | `default:` | `default =>` |
| 类型 | 语句 | 表达式 |
| 必须匹配 | 否 | 是（无 default 时） |

## 基本用法

### 示例 1：基本 match 表达式

```php
<?php
declare(strict_types=1);

// 基本使用
$code = 200;
$status = match($code) {
    200 => "OK",
    404 => "Not Found",
    500 => "Server Error",
    default => "Unknown"
};
echo "Status: {$status}\n";  // Status: OK

// 多个值匹配
$value = 2;
$result = match($value) {
    1, 2, 3 => "Small",
    4, 5, 6 => "Medium",
    7, 8, 9 => "Large",
    default => "Unknown"
};
echo "Result: {$result}\n";  // Result: Small
```

### 示例 2：严格比较

```php
<?php
declare(strict_types=1);

// 严格比较示例
$value = "123";

// switch 使用宽松比较
switch ($value) {
    case 123:  // 匹配（宽松比较）
        echo "Switch matched\n";
        break;
}

// match 使用严格比较
$result = match($value) {
    123 => "Matched",  // 不匹配（严格比较）
    default => "Not matched"
};
echo "Match: {$result}\n";  // Match: Not matched

// 需要匹配字符串
$result = match($value) {
    "123" => "Matched",  // 匹配
    default => "Not matched"
};
echo "Match: {$result}\n";  // Match: Matched
```

### 示例 3：返回值

```php
<?php
declare(strict_types=1);

// match 是表达式，可以返回值
function getStatusMessage(int $code): string
{
    return match($code) {
        200 => "OK",
        201 => "Created",
        404 => "Not Found",
        500 => "Server Error",
        default => "Unknown Status"
    };
}

// 直接赋值
$message = match(404) {
    200 => "OK",
    404 => "Not Found",
    default => "Unknown"
};
echo "Message: {$message}\n";  // Message: Not Found
```

### 示例 4：表达式模式

```php
<?php
declare(strict_types=1);

// 使用表达式作为模式
$value = 42;
$result = match(true) {
    $value < 0 => "Negative",
    $value == 0 => "Zero",
    $value > 0 => "Positive"
};
echo "Result: {$result}\n";  // Result: Positive

// 范围检查
$age = 25;
$category = match(true) {
    $age < 18 => "Minor",
    $age < 65 => "Adult",
    default => "Senior"
};
echo "Category: {$category}\n";  // Category: Adult
```

## 完整代码示例

### 示例 1：HTTP 状态码处理

```php
<?php
declare(strict_types=1);

class HttpResponse
{
    public static function getStatusText(int $code): string
    {
        return match($code) {
            200 => "OK",
            201 => "Created",
            202 => "Accepted",
            204 => "No Content",
            400 => "Bad Request",
            401 => "Unauthorized",
            403 => "Forbidden",
            404 => "Not Found",
            500 => "Internal Server Error",
            502 => "Bad Gateway",
            503 => "Service Unavailable",
            default => "Unknown Status"
        };
    }
    
    public static function isSuccess(int $code): bool
    {
        return match($code) {
            200, 201, 202, 204 => true,
            default => false
        };
    }
}

// 使用
echo HttpResponse::getStatusText(200) . "\n";  // OK
echo HttpResponse::getStatusText(404) . "\n";  // Not Found
var_dump(HttpResponse::isSuccess(200));  // bool(true)
var_dump(HttpResponse::isSuccess(404));  // bool(false)
```

### 示例 2：类型处理

```php
<?php
declare(strict_types=1);

function processValue(mixed $value): string
{
    return match(gettype($value)) {
        'integer' => "Integer: {$value}",
        'double' => "Float: {$value}",
        'string' => "String: {$value}",
        'boolean' => "Boolean: " . ($value ? 'true' : 'false'),
        'array' => "Array with " . count($value) . " elements",
        'NULL' => "Null",
        default => "Unknown type"
    };
}

// 使用
echo processValue(42) . "\n";           // Integer: 42
echo processValue(3.14) . "\n";        // Float: 3.14
echo processValue("hello") . "\n";     // String: hello
echo processValue(true) . "\n";        // Boolean: true
echo processValue([1, 2, 3]) . "\n";   // Array with 3 elements
echo processValue(null) . "\n";        // Null
```

### 示例 3：复杂匹配

```php
<?php
declare(strict_types=1);

class UserRole
{
    public const ADMIN = 'admin';
    public const EDITOR = 'editor';
    public const VIEWER = 'viewer';
}

function getPermissions(string $role): array
{
    return match($role) {
        UserRole::ADMIN => ['read', 'write', 'delete', 'admin'],
        UserRole::EDITOR => ['read', 'write'],
        UserRole::VIEWER => ['read'],
        default => []
    };
}

function canPerformAction(string $role, string $action): bool
{
    $permissions = getPermissions($role);
    return in_array($action, $permissions);
}

// 使用
$permissions = getPermissions(UserRole::ADMIN);
print_r($permissions);  // ['read', 'write', 'delete', 'admin']

var_dump(canPerformAction(UserRole::ADMIN, 'delete'));  // bool(true)
var_dump(canPerformAction(UserRole::VIEWER, 'write'));  // bool(false)
```

### 示例 4：必须匹配

```php
<?php
declare(strict_types=1);

// 没有 default 时，必须匹配
function getDayName(int $day): string
{
    return match($day) {
        1 => "Monday",
        2 => "Tuesday",
        3 => "Wednesday",
        4 => "Thursday",
        5 => "Friday",
        6 => "Saturday",
        7 => "Sunday",
        // 没有 default，如果 $day 不在 1-7 范围内会抛出异常
    };
}

// 使用
try {
    echo getDayName(1) . "\n";  // Monday
    echo getDayName(8) . "\n";  // 抛出 UnhandledMatchError
} catch (UnhandledMatchError $e) {
    echo "Error: " . $e->getMessage() . "\n";
}

// 添加 default 避免异常
function getDayNameSafe(int $day): string
{
    return match($day) {
        1 => "Monday",
        2 => "Tuesday",
        3 => "Wednesday",
        4 => "Thursday",
        5 => "Friday",
        6 => "Saturday",
        7 => "Sunday",
        default => "Invalid day"
    };
}
```

## 使用场景

### 状态码处理

- **HTTP 状态码**：处理 HTTP 响应状态码
- **错误码**：处理错误码
- **状态映射**：将状态码映射到消息

### 类型判断

- **类型处理**：根据类型执行不同逻辑
- **多态处理**：处理不同类型的对象
- **类型转换**：根据类型进行转换

### 值映射

- **配置映射**：将配置值映射到其他值
- **枚举处理**：处理枚举值
- **常量映射**：将常量映射到其他值

### 条件判断

- **范围检查**：检查值是否在范围内
- **表达式匹配**：使用表达式进行匹配
- **复杂条件**：处理复杂的条件判断

## 注意事项

### 严格比较

- **使用 ===**：`match` 使用严格比较
- **类型匹配**：确保类型匹配
- **字符串比较**：注意字符串和数字的区别

### 必须匹配

- **无 default**：如果没有 `default` 且没有匹配，会抛出 `UnhandledMatchError`
- **添加 default**：不确定所有可能值时添加 `default`
- **异常处理**：处理可能的异常

### 表达式模式

- **使用 match(true)**：使用表达式作为模式时，使用 `match(true)`
- **条件顺序**：注意条件的顺序
- **性能考虑**：表达式模式可能影响性能

## 常见问题

### 问题 1：未匹配值异常

**症状**：抛出 `UnhandledMatchError`

**原因**：没有 `default` 且没有匹配的值

**错误示例**：

```php
<?php
declare(strict_types=1);

$code = 999;
$status = match($code) {
    200 => "OK",
    404 => "Not Found",
    // 没有 default
};
// 抛出 UnhandledMatchError
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$code = 999;
$status = match($code) {
    200 => "OK",
    404 => "Not Found",
    default => "Unknown"  // 添加 default
};
echo "Status: {$status}\n";  // Status: Unknown
```

### 问题 2：与 switch 混淆

**症状**：代码行为不符合预期

**原因**：不理解 `match` 和 `switch` 的区别

**错误示例**：

```php
<?php
declare(strict_types=1);

$value = "123";

// switch 使用宽松比较
switch ($value) {
    case 123:  // 匹配
        echo "Matched\n";
        break;
}

// match 使用严格比较
$result = match($value) {
    123 => "Matched",  // 不匹配
    default => "Not matched"
};
echo "Match: {$result}\n";  // Match: Not matched
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$value = "123";

// 使用 match 时确保类型匹配
$result = match($value) {
    "123" => "Matched",  // 匹配
    default => "Not matched"
};
echo "Match: {$result}\n";  // Match: Matched

// 或转换为数字
$result = match((int) $value) {
    123 => "Matched",
    default => "Not matched"
};
echo "Match: {$result}\n";  // Match: Matched
```

### 问题 3：表达式模式顺序

**症状**：匹配结果不符合预期

**原因**：表达式模式的顺序不正确

**错误示例**：

```php
<?php
declare(strict_types=1);

$value = 0;

// 顺序错误：$value < 0 永远不会匹配，因为 $value == 0 先匹配
$result = match(true) {
    $value == 0 => "Zero",
    $value < 0 => "Negative",
    $value > 0 => "Positive"
};
echo "Result: {$result}\n";  // Result: Zero（正确，但顺序不合理）
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$value = 0;

// 正确的顺序：先检查特殊值，再检查范围
$result = match(true) {
    $value < 0 => "Negative",
    $value == 0 => "Zero",
    $value > 0 => "Positive"
};
echo "Result: {$result}\n";  // Result: Zero
```

## 最佳实践

### 使用 match 替代 switch

- **简单 switch**：使用 `match` 替代简单的 `switch` 语句
- **返回值**：需要返回值时使用 `match`
- **严格比较**：需要严格比较时使用 `match`

### 添加 default

- **总是添加 default**：除非确定所有可能值
- **处理未知值**：在 `default` 中处理未知值
- **记录日志**：在 `default` 中记录日志

### 表达式模式

- **合理使用**：合理使用表达式模式
- **注意顺序**：注意条件的顺序
- **性能考虑**：考虑性能影响

## 对比分析

### match vs switch

| 特性 | match | switch |
|:-----|:------|:-------|
| 比较方式 | 严格（===） | 宽松（==） |
| 返回值 | 可以 | 不可以 |
| break | 不需要 | 需要 |
| 类型 | 表达式 | 语句 |
| 必须匹配 | 是 | 否 |
| 推荐度 | 推荐（简单） | 按需使用（复杂） |

**选择建议**：
- **简单匹配**：使用 `match`
- **复杂逻辑**：使用 `switch` 或 `if-else`
- **需要返回值**：使用 `match`
- **需要严格比较**：使用 `match`

### match vs if-else

| 特性 | match | if-else |
|:-----|:------|:--------|
| 语法 | 简洁 | 灵活 |
| 返回值 | 可以 | 可以 |
| 条件 | 值匹配 | 任意条件 |
| 推荐度 | 推荐（值匹配） | 推荐（复杂条件） |

**选择建议**：
- **值匹配**：使用 `match`
- **复杂条件**：使用 `if-else`
- **范围检查**：使用 `match` 的表达式模式

## 相关章节

- **2.6.4 三元运算符与空合并运算符**：了解条件运算符
- **2.9 控制结构**：了解 switch 语句
- **2.5.4 比较运算符与函数**：了解比较运算符

## 练习任务

1. **基本 match 练习**：
   - 练习使用 `match` 表达式
   - 理解严格比较的特性
   - 测试各种匹配模式
   - 理解返回值特性

2. **与 switch 对比练习**：
   - 对比 `match` 和 `switch` 的行为
   - 理解严格比较和宽松比较的区别
   - 重构 `switch` 为 `match`
   - 测试各种场景

3. **表达式模式练习**：
   - 练习使用表达式模式
   - 实现范围检查
   - 实现复杂条件匹配
   - 测试不同场景

4. **实际应用练习**：
   - 实现 HTTP 状态码处理
   - 实现类型处理函数
   - 实现权限检查函数
   - 测试各种场景

5. **综合练习**：
   - 创建一个使用 `match` 表达式的程序
   - 实现复杂的匹配逻辑
   - 处理异常情况
   - 进行代码审查，确保匹配正确
