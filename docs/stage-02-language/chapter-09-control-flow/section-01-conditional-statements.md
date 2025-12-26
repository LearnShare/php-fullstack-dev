# 2.9.1 条件语句

## 概述

条件语句用于根据条件执行不同的代码。本节详细介绍 `if/elseif/else`、`switch`、`match` 表达式的语法、用法、比较方式、使用场景及完整示例。

理解不同条件语句的特点和适用场景对于编写清晰、高效的代码非常重要。选择合适的条件语句可以提高代码的可读性和性能。

## 特性

- **if/elseif/else**：灵活的条件判断，支持复杂条件
- **switch**：多个固定值匹配，使用宽松比较
- **match**：严格比较、返回值、无贯穿（PHP 8.0+）

## 语法/定义

### if/elseif/else 语句

**基本语法**：
```php
if (condition) {
    // 代码块
} elseif (condition) {
    // 代码块
} else {
    // 代码块
}
```

**特点**：
- 按顺序检查条件
- 第一个为真的条件执行后停止
- 支持嵌套
- 使用严格比较（`===`）

### switch 语句

**基本语法**：
```php
switch ($value) {
    case value1:
        // 代码块
        break;
    case value2:
        // 代码块
        break;
    default:
        // 代码块
}
```

**特点**：
- 使用宽松比较（`==`）
- 需要 `break` 防止贯穿
- 支持多个值匹配（`case value1: case value2:`）
- 必须有 `default` 分支（推荐）

### match 表达式（PHP 8.0+）

**基本语法**：
```php
$result = match ($value) {
    pattern1 => result1,
    pattern2 => result2,
    default => default_result,
};
```

**特点**：
- 使用严格比较（`===`）
- 可以返回值
- 不需要 `break`
- 如果没有匹配且无 `default`，会抛出异常

## 基本用法

### 示例 1：if/elseif/else

```php
<?php
declare(strict_types=1);

$age = 25;

// 基本 if 语句
if ($age >= 18) {
    echo "Adult\n";
}

// if-else 语句
if ($age >= 18) {
    echo "Adult\n";
} else {
    echo "Minor\n";
}

// if-elseif-else 语句
if ($age >= 18) {
    echo "Adult\n";
} elseif ($age >= 13) {
    echo "Teenager\n";
} else {
    echo "Child\n";
}

// 嵌套 if 语句
if ($age >= 18) {
    if ($age >= 65) {
        echo "Senior\n";
    } else {
        echo "Adult\n";
    }
} else {
    echo "Minor\n";
}
```

### 示例 2：switch 语句

```php
<?php
declare(strict_types=1);

$status = 'active';

// 基本 switch
switch ($status) {
    case 'active':
        echo "Status is active\n";
        break;
    case 'inactive':
        echo "Status is inactive\n";
        break;
    case 'pending':
        echo "Status is pending\n";
        break;
    default:
        echo "Unknown status\n";
}

// 多个值匹配
$day = 1;
switch ($day) {
    case 1:
    case 2:
    case 3:
    case 4:
    case 5:
        echo "Weekday\n";
        break;
    case 6:
    case 7:
        echo "Weekend\n";
        break;
    default:
        echo "Invalid day\n";
}
```

### 示例 3：match 表达式

```php
<?php
declare(strict_types=1);

$status = 'active';

// 基本 match
$message = match($status) {
    'active' => "Status is active",
    'inactive' => "Status is inactive",
    'pending' => "Status is pending",
    default => "Unknown status"
};
echo $message . "\n";

// 多个值匹配
$day = 1;
$type = match($day) {
    1, 2, 3, 4, 5 => "Weekday",
    6, 7 => "Weekend",
    default => "Invalid day"
};
echo $type . "\n";

// 表达式模式
$value = 42;
$result = match(true) {
    $value < 0 => "Negative",
    $value == 0 => "Zero",
    $value > 0 => "Positive"
};
echo $result . "\n";  // Positive
```

## 完整代码示例

### 示例 1：条件判断工具类

```php
<?php
declare(strict_types=1);

class ConditionHelper
{
    public static function checkAge(int $age): string
    {
        if ($age < 0) {
            return "Invalid age";
        } elseif ($age < 13) {
            return "Child";
        } elseif ($age < 18) {
            return "Teenager";
        } elseif ($age < 65) {
            return "Adult";
        } else {
            return "Senior";
        }
    }
    
    public static function getStatusMessage(string $status): string
    {
        return match($status) {
            'active' => "User is active",
            'inactive' => "User is inactive",
            'suspended' => "User is suspended",
            'deleted' => "User is deleted",
            default => "Unknown status"
        };
    }
    
    public static function getDayType(int $day): string
    {
        return match($day) {
            1, 2, 3, 4, 5 => "Weekday",
            6, 7 => "Weekend",
            default => "Invalid day"
        };
    }
}

// 使用
echo ConditionHelper::checkAge(25) . "\n";  // Adult
echo ConditionHelper::getStatusMessage('active') . "\n";  // User is active
echo ConditionHelper::getDayType(1) . "\n";  // Weekday
```

### 示例 2：权限检查

```php
<?php
declare(strict_types=1);

class PermissionChecker
{
    public static function canAccess(string $role, string $resource): bool
    {
        // 使用 if-elseif 处理复杂条件
        if ($role === 'admin') {
            return true;
        } elseif ($role === 'editor' && in_array($resource, ['posts', 'pages'])) {
            return true;
        } elseif ($role === 'viewer' && $resource === 'posts') {
            return true;
        } else {
            return false;
        }
    }
    
    public static function getPermissionLevel(string $role): string
    {
        // 使用 match 处理固定值匹配
        return match($role) {
            'admin' => 'full',
            'editor' => 'write',
            'viewer' => 'read',
            default => 'none'
        };
    }
}

// 使用
var_dump(PermissionChecker::canAccess('editor', 'posts'));  // bool(true)
echo PermissionChecker::getPermissionLevel('admin') . "\n";  // full
```

### 示例 3：HTTP 状态码处理

```php
<?php
declare(strict_types=1);

class HttpResponse
{
    public static function getStatusText(int $code): string
    {
        // 使用 match 处理状态码
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
        // 使用 if 处理范围判断
        if ($code >= 200 && $code < 300) {
            return true;
        }
        return false;
    }
}

// 使用
echo HttpResponse::getStatusText(200) . "\n";  // OK
var_dump(HttpResponse::isSuccess(201));  // bool(true)
```

## 使用场景

### if/elseif/else

- **复杂条件**：需要复杂条件判断
- **范围判断**：判断值是否在范围内
- **多条件组合**：需要组合多个条件
- **简单判断**：简单的二元判断

### switch

- **多个固定值**：需要匹配多个固定值
- **值映射**：将值映射到其他值
- **状态处理**：处理状态码、类型等

### match

- **多个固定值**：需要匹配多个固定值（PHP 8.0+）
- **需要返回值**：需要直接返回值的场景
- **严格比较**：需要严格比较的场景
- **状态处理**：处理状态码、类型等（推荐替代 switch）

## 注意事项

### 比较方式

- **if/match**：使用严格比较（`===`）
- **switch**：使用宽松比较（`==`）
- **类型安全**：严格比较更安全

### break 语句

- **switch**：需要 `break` 防止贯穿
- **match**：不需要 `break`，自动中断
- **忘记 break**：是 switch 的常见错误

### 返回值

- **if/switch**：不能直接返回值
- **match**：可以返回值，是表达式
- **赋值**：match 可以直接赋值

## 常见问题

### 问题 1：switch 缺少 break

**症状**：多个 case 被执行

**原因**：忘记添加 `break` 语句

**错误示例**：

```php
<?php
declare(strict_types=1);

$status = 'active';
switch ($status) {
    case 'active':
        echo "Active\n";
        // 缺少 break
    case 'inactive':
        echo "Inactive\n";
        break;
}
// 输出：
// Active
// Inactive（意外输出）
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$status = 'active';
switch ($status) {
    case 'active':
        echo "Active\n";
        break;  // 添加 break
    case 'inactive':
        echo "Inactive\n";
        break;
}

// 或使用 match（推荐）
$message = match($status) {
    'active' => "Active",
    'inactive' => "Inactive",
    default => "Unknown"
};
echo $message . "\n";
```

### 问题 2：switch 宽松比较陷阱

**症状**：匹配结果不符合预期

**原因**：switch 使用宽松比较（`==`）

**错误示例**：

```php
<?php
declare(strict_types=1);

$value = "123";
switch ($value) {
    case 123:  // 匹配（宽松比较）
        echo "Matched\n";
        break;
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$value = "123";

// 方法1：使用 match（严格比较）
$result = match($value) {
    "123" => "Matched",
    default => "Not matched"
};
echo $result . "\n";

// 方法2：在 switch 中转换类型
switch ((int) $value) {
    case 123:
        echo "Matched\n";
        break;
}
```

### 问题 3：match 未匹配值异常

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

// 添加 default
$status = match($code) {
    200 => "OK",
    404 => "Not Found",
    default => "Unknown"  // 添加 default
};
echo $status . "\n";  // Unknown
```

## 最佳实践

### 条件语句选择

- **简单条件**：使用 `if`
- **多个固定值**：使用 `match`（PHP 8.0+）
- **复杂条件**：使用 `if-elseif-else`
- **避免 switch**：推荐使用 `match` 替代 `switch`

### 代码可读性

- **明确条件**：使用有意义的条件表达式
- **避免嵌套过深**：避免过度嵌套
- **提取函数**：复杂逻辑提取到函数中

### 性能考虑

- **条件顺序**：将最可能为真的条件放在前面
- **使用 match**：match 性能与 switch 相近
- **避免重复计算**：缓存复杂条件的结果

## 对比分析

### if vs switch vs match

| 特性 | if/elseif/else | switch | match |
|:-----|:---------------|:-------|:------|
| 比较方式 | 严格（===） | 宽松（==） | 严格（===） |
| 返回值 | 否 | 否 | 是 |
| break | 不需要 | 需要 | 不需要 |
| 适用场景 | 复杂条件 | 多个固定值 | 多个固定值 |
| 推荐度 | 推荐（复杂） | 不推荐 | 推荐（固定值） |

**选择建议**：
- **复杂条件**：使用 `if-elseif-else`
- **多个固定值**：使用 `match`（PHP 8.0+）
- **避免 switch**：推荐使用 `match` 替代

## 相关章节

- **2.6.3 逻辑运算符**：了解条件表达式的构建
- **2.6.4 三元运算符与空合并运算符**：了解条件运算符
- **2.6.5 match 表达式**：详细了解 match 表达式

## 练习任务

1. **条件语句练习**：
   - 练习使用 `if/elseif/else`、`switch`、`match`
   - 理解不同方式的区别
   - 测试各种条件场景
   - 观察比较方式的差异

2. **实际应用练习**：
   - 实现权限检查功能
   - 实现状态处理功能
   - 实现条件判断工具
   - 测试各种应用场景

3. **代码重构练习**：
   - 将 `switch` 重构为 `match`
   - 优化条件判断逻辑
   - 提高代码可读性
   - 进行代码审查

4. **综合练习**：
   - 创建一个条件判断工具类
   - 实现各种条件判断功能
   - 使用合适的条件语句
   - 进行代码审查，确保正确性
