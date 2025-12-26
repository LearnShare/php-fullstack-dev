# 2.6.3 逻辑运算符

## 概述

逻辑运算符用于进行逻辑运算。本节详细介绍逻辑与（`&&`）、逻辑或（`||`）、逻辑非（`!`）、`and`、`or`、`xor` 的区别、优先级及短路特性。

理解逻辑运算符的优先级差异和短路特性对于编写正确的条件判断很重要。`&&`/`||` 和 `and`/`or` 的优先级不同，可能导致意外的结果。

## 特性

- **短路求值**：`&&` 和 `||` 支持短路求值，可以提高性能
- **优先级差异**：`&&`/`||` 优先级高于 `and`/`or`
- **布尔转换**：操作数会自动转换为布尔值
- **多种形式**：提供多种逻辑运算符形式

## 语法/定义

### 逻辑运算符

| 运算符 | 名称 | 语法 | 说明 | 优先级 |
|:-------|:-----|:-----|:-----|:-------|
| `&&` | 逻辑与 | `$a && $b` | 两者都为 true 时返回 true | 高 |
| `\|\|` | 逻辑或 | `$a \|\| $b` | 任一为 true 时返回 true | 高 |
| `!` | 逻辑非 | `!$a` | 取反 | 高 |
| `and` | 逻辑与 | `$a and $b` | 两者都为 true 时返回 true | 低 |
| `or` | 逻辑或 | `$a or $b` | 任一为 true 时返回 true | 低 |
| `xor` | 逻辑异或 | `$a xor $b` | 两者不同时返回 true | 低 |

**特点**：
- `&&` 和 `and` 功能相同，但优先级不同
- `||` 和 `or` 功能相同，但优先级不同
- `!` 是一元运算符
- `xor` 是二元运算符

### 短路求值

**逻辑与（&&）**：
- 如果第一个操作数为 `false`，不计算第二个操作数
- 直接返回 `false`

**逻辑或（||）**：
- 如果第一个操作数为 `true`，不计算第二个操作数
- 直接返回 `true`

**特点**：
- 可以提高性能（避免不必要的计算）
- 可以用于条件执行
- 需要注意副作用

## 基本用法

### 示例 1：逻辑与（&&）

```php
<?php
declare(strict_types=1);

// 基本使用
$a = true;
$b = true;
var_dump($a && $b);  // bool(true)

$a = true;
$b = false;
var_dump($a && $b);  // bool(false)

$a = false;
$b = true;
var_dump($a && $b);  // bool(false)

// 多个条件
$age = 25;
$hasLicense = true;
$hasInsurance = true;

if ($age >= 18 && $hasLicense && $hasInsurance) {
    echo "Can drive\n";
}
```

### 示例 2：逻辑或（||）

```php
<?php
declare(strict_types=1);

// 基本使用
$a = true;
$b = false;
var_dump($a || $b);  // bool(true)

$a = false;
$b = false;
var_dump($a || $b);  // bool(false)

// 多个条件
$userRole = "admin";
$isOwner = false;

if ($userRole === "admin" || $isOwner) {
    echo "Has permission\n";
}
```

### 示例 3：逻辑非（!）

```php
<?php
declare(strict_types=1);

// 基本使用
$a = true;
var_dump(!$a);  // bool(false)

$a = false;
var_dump(!$a);  // bool(true)

// 条件判断
$isLoggedIn = false;
if (!$isLoggedIn) {
    echo "Please log in\n";
}
```

### 示例 4：短路求值

```php
<?php
declare(strict_types=1);

function expensive(): bool
{
    echo "Expensive operation executed\n";
    return true;
}

// 逻辑与短路
echo "=== Logical AND ===\n";
$result = false && expensive();  // 不执行 expensive()
echo "Result: " . ($result ? 'true' : 'false') . "\n";

$result = true && expensive();  // 执行 expensive()
echo "Result: " . ($result ? 'true' : 'false') . "\n";

// 逻辑或短路
echo "\n=== Logical OR ===\n";
$result = true || expensive();  // 不执行 expensive()
echo "Result: " . ($result ? 'true' : 'false') . "\n";

$result = false || expensive();  // 执行 expensive()
echo "Result: " . ($result ? 'true' : 'false') . "\n";
```

**执行**：

```bash
php short-circuit.php
```

**输出**：

```
=== Logical AND ===
Result: false
Expensive operation executed
Result: true

=== Logical OR ===
Result: true
Expensive operation executed
Result: true
```

### 示例 5：优先级差异

```php
<?php
declare(strict_types=1);

// && 和 and 的优先级差异
$a = true;
$b = false;
$c = true;

// && 优先级高
$result1 = $a && $b || $c;
echo "&& result: " . ($result1 ? 'true' : 'false') . "\n";  // true

// and 优先级低
$result2 = $a and $b || $c;
echo "and result: " . ($result2 ? 'true' : 'false') . "\n";  // false（优先级不同）

// 使用括号明确优先级
$result3 = ($a and $b) || $c;
echo "With parentheses: " . ($result3 ? 'true' : 'false') . "\n";  // true
```

## 完整代码示例

### 示例 1：权限检查

```php
<?php
declare(strict_types=1);

class PermissionChecker
{
    public function canEdit(array $user, int $resourceId): bool
    {
        $isOwner = $user['id'] === $resourceId;
        $isAdmin = $user['role'] === 'admin';
        $hasEditPermission = in_array('edit', $user['permissions'] ?? []);
        
        // 使用逻辑或：是所有者、管理员或有编辑权限
        return $isOwner || $isAdmin || $hasEditPermission;
    }
    
    public function canDelete(array $user, int $resourceId): bool
    {
        $isOwner = $user['id'] === $resourceId;
        $isAdmin = $user['role'] === 'admin';
        $hasDeletePermission = in_array('delete', $user['permissions'] ?? []);
        
        // 使用逻辑与：是所有者且不是系统资源，或是管理员
        return ($isOwner && $resourceId > 0) || ($isAdmin && $hasDeletePermission);
    }
}

$user = ['id' => 1, 'role' => 'user', 'permissions' => ['edit']];
$checker = new PermissionChecker();

var_dump($checker->canEdit($user, 1));    // bool(true) - 是所有者
var_dump($checker->canEdit($user, 2));    // bool(true) - 有编辑权限
var_dump($checker->canDelete($user, 1));  // bool(true) - 是所有者
```

### 示例 2：短路求值优化

```php
<?php
declare(strict_types=1);

// 使用短路求值避免不必要的计算
function processUser(?array $user): void
{
    // 如果 $user 为 null，不执行后续检查
    if ($user !== null && $user['active'] && $user['verified']) {
        echo "Processing user: {$user['name']}\n";
    } else {
        echo "User cannot be processed\n";
    }
}

// 使用短路求值提供默认值
function getConfigValue(array $config, string $key, mixed $default = null): mixed
{
    // 如果配置存在且不为 null，返回配置值；否则返回默认值
    return $config[$key] ?? $default;
}

// 使用短路求值进行安全检查
function safeDivide(float $a, ?float $b): ?float
{
    // 如果 $b 为 null 或 0，不执行除法
    if ($b !== null && $b != 0) {
        return $a / $b;
    }
    return null;
}
```

### 示例 3：逻辑异或（xor）

```php
<?php
declare(strict_types=1);

// 逻辑异或：两者不同时返回 true
$a = true;
$b = false;
var_dump($a xor $b);  // bool(true)

$a = true;
$b = true;
var_dump($a xor $b);  // bool(false)

$a = false;
$b = false;
var_dump($a xor $b);  // bool(false)

// 实际应用：互斥选项
function validateMutuallyExclusive(bool $option1, bool $option2): bool
{
    // 只能选择其中一个
    return $option1 xor $option2;
}

var_dump(validateMutuallyExclusive(true, false));   // bool(true)
var_dump(validateMutuallyExclusive(false, true));  // bool(true)
var_dump(validateMutuallyExclusive(true, true));   // bool(false)
var_dump(validateMutuallyExclusive(false, false)); // bool(false)
```

## 使用场景

### 条件判断

- **多条件判断**：组合多个条件进行判断
- **权限检查**：检查用户权限
- **数据验证**：验证数据是否符合多个条件

### 默认值提供

- **使用逻辑或**：使用 `||` 提供默认值（注意：`??` 更推荐）
- **配置读取**：从多个来源读取配置
- **参数处理**：处理可选参数

### 安全检查

- **使用逻辑与**：使用 `&&` 进行安全检查
- **空值检查**：检查变量是否存在
- **类型检查**：检查变量类型

### 性能优化

- **短路求值**：利用短路求值避免不必要的计算
- **条件执行**：使用短路求值实现条件执行
- **优化检查顺序**：将最可能失败的条件放在前面

## 注意事项

### 优先级差异

- **&&/|| 优先级高**：`&&` 和 `||` 优先级高于 `and` 和 `or`
- **统一使用**：建议统一使用 `&&` 和 `||`
- **使用括号**：不确定时使用括号明确优先级

### 短路求值

- **理解机制**：理解短路求值的执行顺序
- **利用优化**：利用短路求值优化性能
- **注意副作用**：注意短路求值可能跳过的副作用

### 布尔转换

- **自动转换**：操作数会自动转换为布尔值
- **理解规则**：理解各种值到布尔值的转换规则
- **使用严格比较**：需要时使用严格比较

## 常见问题

### 问题 1：优先级混淆

**症状**：逻辑表达式结果不符合预期

**原因**：不理解 `&&`/`||` 和 `and`/`or` 的优先级差异

**错误示例**：

```php
<?php
declare(strict_types=1);

$a = true;
$b = false;
$c = true;

// 期望：($a and $b) || $c
$result = $a and $b || $c;  // 实际：$a and ($b || $c)
var_dump($result);  // bool(true) - 可能不符合预期
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$a = true;
$b = false;
$c = true;

// 方法1：使用 && 和 ||
$result = $a && $b || $c;  // 明确优先级

// 方法2：使用括号
$result = ($a and $b) || $c;  // 明确优先级

// 方法3：统一使用 && 和 ||
$result = $a && $b || $c;  // 推荐
```

### 问题 2：短路求值误用

**症状**：代码行为不符合预期

**原因**：不理解短路求值的机制

**错误示例**：

```php
<?php
declare(strict_types=1);

function init(): bool
{
    echo "Initializing\n";
    return true;
}

// 期望：总是执行 init()
if (true || init()) {  // init() 不会执行
    echo "Condition is true\n";
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

function init(): bool
{
    echo "Initializing\n";
    return true;
}

// 方法1：分开执行
init();
if (true) {
    echo "Condition is true\n";
}

// 方法2：使用逻辑与
if (init() && true) {
    echo "Condition is true\n";
}
```

### 问题 3：逻辑异或理解错误

**症状**：逻辑异或结果不符合预期

**原因**：不理解逻辑异或的规则

**说明**：

```php
<?php
declare(strict_types=1);

// 逻辑异或：两者不同时返回 true
var_dump(true xor false);   // bool(true)
var_dump(false xor true);   // bool(true)
var_dump(true xor true);    // bool(false)
var_dump(false xor false);  // bool(false)
```

## 最佳实践

### 运算符选择

- **统一使用 &&/||**：统一使用 `&&` 和 `||`，避免 `and` 和 `or`
- **使用括号**：不确定优先级时使用括号
- **避免混用**：避免在同一表达式中混用不同优先级的运算符

### 短路求值

- **利用优化**：利用短路求值优化性能
- **优化顺序**：将最可能失败的条件放在前面
- **注意副作用**：注意短路求值可能跳过的副作用

### 代码可读性

- **明确意图**：使用有意义的条件表达式
- **避免复杂表达式**：避免过于复杂的逻辑表达式
- **提取函数**：将复杂逻辑提取到函数中

## 对比分析

### && vs and

| 特性 | && | and |
|:-----|:---|:-----|
| 优先级 | 高 | 低 |
| 功能 | 相同 | 相同 |
| 推荐度 | 推荐 | 不推荐 |

**选择建议**：
- **统一使用 &&**：避免优先级混淆
- **避免使用 and**：除非有特殊需求

### || vs or

| 特性 | \|\| | or |
|:-----|:-----|:-----|
| 优先级 | 高 | 低 |
| 功能 | 相同 | 相同 |
| 推荐度 | 推荐 | 不推荐 |

**选择建议**：
- **统一使用 ||**：避免优先级混淆
- **避免使用 or**：除非有特殊需求

### 短路求值 vs 完整求值

| 特性 | 短路求值 | 完整求值 |
|:-----|:---------|:---------|
| 性能 | 好 | 差 |
| 副作用 | 可能跳过 | 总是执行 |
| 适用场景 | 条件判断 | 需要副作用 |

**选择建议**：
- **条件判断**：使用短路求值
- **需要副作用**：确保所有操作都执行

## 相关章节

- **2.5.1 隐式转换**：了解布尔转换规则
- **2.6.4 三元运算符与空合并运算符**：了解条件运算符
- **2.6.6 运算符优先级与结合性**：了解运算符优先级
- **2.9 控制结构**：了解条件语句

## 练习任务

1. **基本逻辑运算练习**：
   - 练习使用 `&&`、`||`、`!` 运算符
   - 理解逻辑运算的真值表
   - 测试各种组合的结果
   - 理解布尔转换规则

2. **优先级练习**：
   - 对比 `&&`/`||` 和 `and`/`or` 的优先级
   - 使用括号明确优先级
   - 测试不同优先级组合
   - 理解优先级的影响

3. **短路求值练习**：
   - 观察短路求值的行为
   - 利用短路求值优化代码
   - 测试不同场景下的短路行为
   - 理解短路求值的优势

4. **实际应用练习**：
   - 实现权限检查函数
   - 实现数据验证函数
   - 使用逻辑运算符组合条件
   - 测试各种场景

5. **综合练习**：
   - 创建一个使用逻辑运算符的程序
   - 实现复杂的条件判断
   - 利用短路求值优化性能
   - 进行代码审查，确保逻辑正确
