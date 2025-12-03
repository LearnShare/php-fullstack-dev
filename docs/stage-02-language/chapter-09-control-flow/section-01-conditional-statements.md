# 2.9.1 条件语句

## 概述

条件语句用于根据不同的条件执行不同的代码块。PHP 提供了 `if/elseif/else`、`switch` 和 `match` 三种条件语句，每种都有其适用场景。

## if/elseif/else 语句

### 基本语法

```php
if (condition) {
    // 代码块
} elseif (condition) {
    // 代码块
} else {
    // 代码块
}
```

### 基本用法

```php
<?php
declare(strict_types=1);

$age = 20;

if ($age >= 18) {
    echo "Adult\n";
} else {
    echo "Minor\n";
}
```

### 多个条件

```php
<?php
declare(strict_types=1);

$score = 85;

if ($score >= 90) {
    $grade = 'A';
} elseif ($score >= 80) {
    $grade = 'B';
} elseif ($score >= 70) {
    $grade = 'C';
} else {
    $grade = 'F';
}

echo "Grade: {$grade}\n";
```

### 简写语法（不推荐）

PHP 支持简写的 if 语句，但不推荐使用：

```php
<?php
declare(strict_types=1);

$age = 20;
if ($age >= 18):
    echo "Adult\n";
else:
    echo "Minor\n";
endif;
```

## switch 语句

### 基本语法

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

### 基本用法

```php
<?php
declare(strict_types=1);

$status = 200;

switch ($status) {
    case 200:
        echo "OK\n";
        break;
    case 404:
        echo "Not Found\n";
        break;
    case 500:
        echo "Internal Server Error\n";
        break;
    default:
        echo "Unknown Status\n";
}
```

### 多个值匹配

```php
<?php
declare(strict_types=1);

$status = 201;

switch ($status) {
    case 200:
    case 201:
    case 202:
        echo "Success\n";
        break;
    case 400:
    case 401:
    case 403:
        echo "Client Error\n";
        break;
    default:
        echo "Unknown\n";
}
```

### 贯穿（Fall-through）

如果不使用 `break`，代码会继续执行下一个 `case`：

```php
<?php
declare(strict_types=1);

$status = 200;

switch ($status) {
    case 200:
        echo "OK\n";
        // 注意：没有 break，会继续执行
    case 201:
        echo "Created\n";
        break;
    default:
        echo "Unknown\n";
}
// 输出：
// OK
// Created
```

### 宽松比较

`switch` 使用宽松比较（`==`），注意类型转换：

```php
<?php
declare(strict_types=1);

$value = "200";

switch ($value) {
    case 200:  // "200" == 200 为 true
        echo "Matched\n";
        break;
}
```

## match 表达式（PHP 8.0+）

### 基本语法

```php
$result = match ($value) {
    pattern1 => expression1,
    pattern2 => expression2,
    default => defaultExpression,
};
```

### 基本用法

```php
<?php
declare(strict_types=1);

$status = 200;
$message = match ($status) {
    200 => 'OK',
    404 => 'Not Found',
    500 => 'Internal Server Error',
    default => 'Unknown Status',
};

echo $message . "\n";  // OK
```

### 多个值匹配

```php
<?php
declare(strict_types=1);

$status = 201;
$label = match ($status) {
    200, 201, 202 => 'success',
    400, 401, 403 => 'client_error',
    500, 502, 503 => 'server_error',
    default => 'unknown',
};

echo $label . "\n";  // success
```

### 严格比较

`match` 使用严格比较（`===`），不会进行类型转换：

```php
<?php
declare(strict_types=1);

$value = "200";

$result = match ($value) {
    200 => "Matched",  // "200" === 200 为 false，不会匹配
    default => "Not matched",
};

echo $result . "\n";  // Not matched
```

### 返回值

`match` 表达式可以返回值，这是与 `switch` 的主要区别：

```php
<?php
declare(strict_types=1);

function getStatusMessage(int $code): string
{
    return match ($code) {
        200 => 'OK',
        404 => 'Not Found',
        500 => 'Internal Server Error',
        default => 'Unknown Status',
    };
}

echo getStatusMessage(200) . "\n";  // OK
```

### 表达式作为模式

```php
<?php
declare(strict_types=1);

$age = 25;
$category = match (true) {
    $age < 18 => 'Minor',
    $age < 65 => 'Adult',
    default => 'Senior',
};

echo $category . "\n";  // Adult
```

## 条件语句对比

| 特性       | `if/else`              | `switch`               | `match`                |
| :--------- | :--------------------- | :--------------------- | :--------------------- |
| 比较方式   | 按顺序检查             | 宽松比较（`==`）       | 严格比较（`===`）      |
| 返回值     | 不能直接返回值         | 不能直接返回值         | 可以返回值             |
| break      | 不需要                 | 需要（防止贯穿）       | 不需要                 |
| 适用场景   | 复杂条件、简单判断     | 多个固定值匹配         | 多个固定值匹配、需返回值 |
| 性能       | 按顺序检查，可能较慢   | 可能使用跳转表优化     | 与 switch 相近         |

## 完整示例

```php
<?php
declare(strict_types=1);

class ConditionalStatements
{
    public static function demonstrateIfElse(int $age): string
    {
        if ($age < 18) {
            return 'Minor';
        } elseif ($age < 65) {
            return 'Adult';
        } else {
            return 'Senior';
        }
    }
    
    public static function demonstrateSwitch(int $status): string
    {
        switch ($status) {
            case 200:
            case 201:
            case 202:
                return 'Success';
            case 400:
            case 401:
            case 403:
                return 'Client Error';
            case 500:
            case 502:
            case 503:
                return 'Server Error';
            default:
                return 'Unknown';
        }
    }
    
    public static function demonstrateMatch(int $status): string
    {
        return match ($status) {
            200, 201, 202 => 'Success',
            400, 401, 403 => 'Client Error',
            500, 502, 503 => 'Server Error',
            default => 'Unknown',
        };
    }
}

echo ConditionalStatements::demonstrateIfElse(25) . "\n";      // Adult
echo ConditionalStatements::demonstrateSwitch(200) . "\n";   // Success
echo ConditionalStatements::demonstrateMatch(404) . "\n";    // Client Error
```

## 注意事项

1. **选择合适的方式**：简单条件用 `if`，多个固定值用 `switch` 或 `match`，需要返回值用 `match`。

2. **switch 的 break**：`switch` 中必须使用 `break`，否则会贯穿到下一个 `case`。

3. **match 的严格比较**：`match` 使用严格比较，注意类型匹配。

4. **match 必须覆盖**：如果没有 `default` 分支，必须覆盖所有可能的值，否则会抛出异常。

5. **性能考虑**：对于大量固定值匹配，`switch` 和 `match` 可能比多个 `if/elseif` 更快。

## 练习

1. 创建一个函数，使用 `if/else` 根据分数返回等级（A、B、C、D、F）。

2. 编写一个函数，使用 `switch` 根据 HTTP 方法返回相应的处理函数名。

3. 实现一个函数，使用 `match` 表达式根据用户类型返回权限列表。

4. 创建一个函数，比较 `switch` 和 `match` 在处理类型转换时的差异。

5. 编写一个函数，将复杂的 `if/else` 语句重构为 `match` 表达式。
