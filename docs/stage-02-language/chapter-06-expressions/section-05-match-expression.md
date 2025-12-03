# 2.6.5 match 表达式

## 概述

`match` 表达式（PHP 8.0+）是 `switch` 语句的现代化替代品，提供了更安全、更简洁的条件分支语法。`match` 使用严格比较（`===`），必须覆盖所有可能的分支，并且可以返回值。

## 基本语法

```php
$result = match ($value) {
    pattern1 => expression1,
    pattern2 => expression2,
    default => defaultExpression,
};
```

## 基本用法

### 简单匹配

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

$httpStatus = 201;
$label = match ($httpStatus) {
    200, 201 => 'success',
    400, 401, 403 => 'client_error',
    500, 502, 503 => 'server_error',
    default => 'unknown',
};

echo $label . "\n";  // success
```

## 与 switch 的对比

### switch 语句

```php
<?php
declare(strict_types=1);

$status = 200;
switch ($status) {
    case 200:
        $message = 'OK';
        break;
    case 404:
        $message = 'Not Found';
        break;
    case 500:
        $message = 'Internal Server Error';
        break;
    default:
        $message = 'Unknown Status';
}
echo $message . "\n";
```

### match 表达式

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
echo $message . "\n";
```

### 主要区别

| 特性       | switch                    | match                     |
| :--------- | :------------------------ | :------------------------ |
| 比较方式   | 宽松比较（`==`）          | 严格比较（`===`）         |
| 返回值     | 不能直接返回值            | 可以返回值                |
| break      | 需要 `break`               | 不需要 `break`            |
| 默认分支   | 可选                      | 如果未覆盖所有情况则必需  |
| 表达式     | 只能使用常量               | 可以使用表达式            |

## 严格比较

`match` 使用严格比较（`===`），不会进行类型转换：

```php
<?php
declare(strict_types=1);

$value = "200";

// switch 使用宽松比较
switch ($value) {
    case 200:  // "200" == 200 为 true
        echo "Matched in switch\n";
        break;
}

// match 使用严格比较
$result = match ($value) {
    200 => "Matched",  // "200" === 200 为 false，不会匹配
    default => "Not matched",
};
echo $result . "\n";  // Not matched
```

## 返回值

`match` 表达式可以返回值，这是与 `switch` 的主要区别之一：

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

## 表达式作为模式

`match` 允许在模式中使用表达式：

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

## 复杂示例

### HTTP 状态码处理

```php
<?php
declare(strict_types=1);

class HttpResponse
{
    public static function getStatusInfo(int $status): array
    {
        return match ($status) {
            200, 201, 202 => [
                'success' => true,
                'message' => 'Success',
                'category' => '2xx'
            ],
            400, 401, 403, 404 => [
                'success' => false,
                'message' => 'Client Error',
                'category' => '4xx'
            ],
            500, 502, 503 => [
                'success' => false,
                'message' => 'Server Error',
                'category' => '5xx'
            ],
            default => [
                'success' => false,
                'message' => 'Unknown Status',
                'category' => 'unknown'
            ],
        };
    }
}

$info = HttpResponse::getStatusInfo(404);
print_r($info);
// 输出：
// Array
// (
//     [success] => 
//     [message] => Client Error
//     [category] => 4xx
// )
```

### 用户角色处理

```php
<?php
declare(strict_types=1);

class UserRole
{
    public static function getPermissions(string $role): array
    {
        return match ($role) {
            'admin' => ['read', 'write', 'delete', 'manage'],
            'editor' => ['read', 'write'],
            'viewer' => ['read'],
            default => throw new InvalidArgumentException("Unknown role: {$role}"),
        };
    }
    
    public static function canAccess(string $role, string $permission): bool
    {
        $permissions = self::getPermissions($role);
        return in_array($permission, $permissions);
    }
}

$adminPerms = UserRole::getPermissions('admin');
print_r($adminPerms);  // ['read', 'write', 'delete', 'manage']

echo UserRole::canAccess('editor', 'write') ? 'Yes' : 'No';  // Yes
echo UserRole::canAccess('viewer', 'write') ? 'Yes' : 'No';  // No
```

## 抛出异常

`match` 表达式的分支可以抛出异常：

```php
<?php
declare(strict_types=1);

function processValue(mixed $value): string
{
    return match (true) {
        is_string($value) => "String: {$value}",
        is_int($value) => "Integer: {$value}",
        is_float($value) => "Float: {$value}",
        default => throw new InvalidArgumentException("Unsupported type: " . gettype($value)),
    };
}

echo processValue("hello") . "\n";  // String: hello
echo processValue(42) . "\n";       // Integer: 42
// echo processValue([]) . "\n";    // 抛出异常
```

## 必须覆盖所有情况

如果 `match` 表达式的值可能不匹配任何模式，且没有 `default` 分支，会抛出 `UnhandledMatchError`：

```php
<?php
declare(strict_types=1);

$status = 301;

// 没有 default 分支，且 301 不匹配任何模式
try {
    $message = match ($status) {
        200 => 'OK',
        404 => 'Not Found',
    };
} catch (UnhandledMatchError $e) {
    echo "Unhandled match value: " . $e->getMessage() . "\n";
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class OrderProcessor
{
    public function processOrder(string $status, float $amount): array
    {
        $action = match ($status) {
            'pending' => $this->handlePending($amount),
            'processing' => $this->handleProcessing($amount),
            'shipped' => $this->handleShipped($amount),
            'delivered' => $this->handleDelivered($amount),
            'cancelled' => $this->handleCancelled($amount),
            default => throw new InvalidArgumentException("Unknown status: {$status}"),
        };
        
        return [
            'status' => $status,
            'amount' => $amount,
            'action' => $action,
            'timestamp' => time(),
        ];
    }
    
    private function handlePending(float $amount): string
    {
        return "Order pending, amount: \${$amount}";
    }
    
    private function handleProcessing(float $amount): string
    {
        return "Processing order, amount: \${$amount}";
    }
    
    private function handleShipped(float $amount): string
    {
        return "Order shipped, amount: \${$amount}";
    }
    
    private function handleDelivered(float $amount): string
    {
        return "Order delivered, amount: \${$amount}";
    }
    
    private function handleCancelled(float $amount): string
    {
        return "Order cancelled, amount: \${$amount}";
    }
}

$processor = new OrderProcessor();
$result = $processor->processOrder('processing', 99.99);
print_r($result);
```

## 注意事项

1. **严格比较**：`match` 使用严格比较，注意类型匹配。

2. **必须覆盖**：如果没有 `default` 分支，必须覆盖所有可能的值，否则会抛出异常。

3. **返回值**：`match` 表达式必须返回一个值，每个分支都必须有返回值。

4. **表达式**：模式可以是表达式，使用 `match (true)` 可以实现复杂条件。

5. **可读性**：对于简单条件，`match` 比 `switch` 更简洁；对于复杂逻辑，考虑使用 if-else。

## 练习

1. 创建一个函数，使用 `match` 表达式根据 HTTP 方法返回相应的处理函数名。

2. 编写一个函数，使用 `match` 表达式根据用户类型返回权限列表。

3. 实现一个状态机，使用 `match` 表达式处理状态转换。

4. 创建一个函数，使用 `match` 表达式根据文件扩展名返回 MIME 类型。

5. 编写一个函数，演示 `match` 表达式与 `switch` 语句的差异。

