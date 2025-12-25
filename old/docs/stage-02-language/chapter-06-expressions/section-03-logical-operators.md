# 2.6.3 逻辑运算符

## 概述

逻辑运算符用于组合和操作布尔值，实现条件判断和逻辑控制。PHP 提供了多种逻辑运算符，包括 `&&`、`||`、`!`、`and`、`or`、`xor` 等。

## 逻辑与（&&）

### 语法

```php
$condition1 && $condition2
```

### 行为

- 如果两个条件都为 `true`，返回 `true`
- 如果任一条件为 `false`，返回 `false`
- 具有短路特性：如果第一个条件为 `false`，不会评估第二个条件

### 示例

```php
<?php
declare(strict_types=1);

$age = 25;
$hasLicense = true;

if ($age >= 18 && $hasLicense) {
    echo "Can drive\n";
}

// 短路示例
function expensiveCheck(): bool
{
    echo "Expensive check executed\n";
    return true;
}

$result = false && expensiveCheck();  // expensiveCheck() 不会执行
```

## 逻辑或（||）

### 语法

```php
$condition1 || $condition2
```

### 行为

- 如果任一条件为 `true`，返回 `true`
- 如果两个条件都为 `false`，返回 `false`
- 具有短路特性：如果第一个条件为 `true`，不会评估第二个条件

### 示例

```php
<?php
declare(strict_types=1);

$age = 16;
$hasParent = true;

if ($age >= 18 || $hasParent) {
    echo "Can access\n";
}

// 短路示例
function expensiveCheck(): bool
{
    echo "Expensive check executed\n";
    return false;
}

$result = true || expensiveCheck();  // expensiveCheck() 不会执行
```

## 逻辑非（!）

### 语法

```php
!$condition
```

### 行为

- 如果条件为 `true`，返回 `false`
- 如果条件为 `false`，返回 `true`

### 示例

```php
<?php
declare(strict_types=1);

$isActive = false;

if (!$isActive) {
    echo "User is inactive\n";
}

// 双重否定
$isActive = true;
if (!!$isActive) {
    echo "User is active\n";
}
```

## and 和 or 运算符

### 与 && 和 || 的区别

`and` 和 `or` 运算符的功能与 `&&` 和 `||` 相同，但**优先级更低**。

| 运算符 | 优先级 | 说明           |
| :----- | :----- | :------------- |
| `&&`   | 高     | 逻辑与（推荐） |
| `and`  | 低     | 逻辑与         |
| `||`   | 高     | 逻辑或（推荐） |
| `or`   | 低     | 逻辑或         |

### 示例

```php
<?php
declare(strict_types=1);

// && 优先级高于 =
$result1 = true && false;  // $result1 = false
var_dump($result1);  // bool(false)

// and 优先级低于 =
$result2 = true and false;  // ($result2 = true) and false
var_dump($result2);  // bool(true)

// 正确的使用方式
$result3 = (true and false);
var_dump($result3);  // bool(false)
```

### 使用场景

`and` 和 `or` 主要用于控制流语句，利用其低优先级特性：

```php
<?php
declare(strict_types=1);

// 使用 and 连接多个操作
$user = getUser() and processUser($user);

// 使用 or 提供默认值
$config = loadConfig() or $config = getDefaultConfig();
```

## 逻辑异或（xor）

### 语法

```php
$condition1 xor $condition2
```

### 行为

- 如果两个条件不同（一个为 `true`，一个为 `false`），返回 `true`
- 如果两个条件相同（都为 `true` 或都为 `false`），返回 `false`

### 示例

```php
<?php
declare(strict_types=1);

var_dump(true xor true);   // bool(false)
var_dump(true xor false);  // bool(true)
var_dump(false xor true);  // bool(true)
var_dump(false xor false); // bool(false)

// 实际应用：互斥条件
$hasCreditCard = true;
$hasPayPal = false;

if ($hasCreditCard xor $hasPayPal) {
    echo "Exactly one payment method\n";
}
```

## 逻辑运算符优先级

从高到低：

1. `!`（逻辑非）
2. `&&`（逻辑与）
3. `||`（逻辑或）
4. `and`（逻辑与，低优先级）
5. `or`（逻辑或，低优先级）
6. `xor`（逻辑异或）

### 示例

```php
<?php
declare(strict_types=1);

// 使用括号明确优先级
$result1 = true && false || true;  // (true && false) || true = false || true = true
$result2 = true && (false || true); // true && true = true

var_dump($result1);  // bool(true)
var_dump($result2);  // bool(true)
```

## 完整示例

```php
<?php
declare(strict_types=1);

class AccessControl
{
    private bool $isAuthenticated = false;
    private bool $isAdmin = false;
    private bool $hasPermission = false;
    
    public function __construct(bool $isAuthenticated, bool $isAdmin, bool $hasPermission)
    {
        $this->isAuthenticated = $isAuthenticated;
        $this->isAdmin = $isAdmin;
        $this->hasPermission = $hasPermission;
    }
    
    public function canAccess(): bool
    {
        // 必须认证且（是管理员或有权限）
        return $this->isAuthenticated && ($this->isAdmin || $this->hasPermission);
    }
    
    public function canEdit(): bool
    {
        // 必须认证且是管理员
        return $this->isAuthenticated && $this->isAdmin;
    }
    
    public function canView(): bool
    {
        // 认证或（未认证但有权限）
        return $this->isAuthenticated || (!$this->isAuthenticated && $this->hasPermission);
    }
}

// 使用示例
$user1 = new AccessControl(true, true, false);
echo "User1 can access: " . ($user1->canAccess() ? 'Yes' : 'No') . "\n";  // Yes

$user2 = new AccessControl(true, false, true);
echo "User2 can access: " . ($user2->canAccess() ? 'Yes' : 'No') . "\n";  // Yes

$user3 = new AccessControl(false, false, true);
echo "User3 can access: " . ($user3->canAccess() ? 'Yes' : 'No') . "\n";  // No
```

## 注意事项

1. **短路特性**：利用 `&&` 和 `||` 的短路特性，将快速失败的条件放在前面。

2. **优先级**：注意 `and` 和 `or` 的优先级低于 `&&` 和 `||`，使用时要加括号。

3. **推荐使用**：优先使用 `&&` 和 `||`，而不是 `and` 和 `or`。

4. **可读性**：复杂的逻辑表达式使用括号明确优先级。

5. **类型转换**：逻辑运算符会将值转换为布尔值，注意 falsy 值的处理。

## 练习

1. 创建一个权限检查类，使用逻辑运算符实现复杂的权限判断逻辑。

2. 编写一个函数，使用短路特性优化性能，避免不必要的计算。

3. 实现一个配置验证函数，使用逻辑运算符检查多个配置项。

4. 创建一个函数，演示 `and`、`or` 与 `&&`、`||` 的优先级差异。

5. 编写一个函数，使用逻辑异或实现互斥条件检查。

