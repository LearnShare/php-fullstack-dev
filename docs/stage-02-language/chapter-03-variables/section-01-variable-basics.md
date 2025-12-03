# 2.3.1 变量基础

## 概述

变量是 PHP 中存储数据的基本单元。与静态类型语言（如 Java、C++）不同，PHP 采用动态类型系统，变量无需显式声明类型，其类型由赋值时的值自动推断。

## 变量命名规则

### 基本规则

1. **必须以 `$` 符号开头**：这是 PHP 变量的标识符。
2. **第一个字符**：必须是字母（a-z、A-Z）或下划线（`_`）。
3. **后续字符**：可以是字母、数字或下划线。
4. **区分大小写**：`$name` 和 `$Name` 是两个不同的变量。
5. **不能使用关键字**：不能使用 PHP 保留关键字作为变量名（如 `if`、`else`、`function` 等）。

### 命名规则示例

```php
<?php
declare(strict_types=1);

// 正确的变量名
$name      = 'Alice';
$user_name = 'Bob';
$userName  = 'Charlie';
$name123   = 'David';
$_private  = 'Eve';
$Name      = 'Frank';  // 与 $name 不同

// 错误的变量名
// $123name = 'Error';     // 不能以数字开头
// $user-name = 'Error';   // 不能使用连字符
// $user name = 'Error';   // 不能包含空格
// $if = 'Error';          // 不能使用关键字
```

### 命名规范建议

虽然 PHP 允许多种命名风格，但建议遵循以下规范：

- **驼峰命名（camelCase）**：用于普通变量，如 `$userName`、`$orderTotal`。
- **下划线命名（snake_case）**：用于常量或配置项，如 `$user_name`、`$max_count`。
- **全大写**：通常用于常量，如 `$MAX_SIZE`（但常量更推荐使用 `const` 或 `define`）。

## 变量作用域

变量的作用域决定了变量在代码中的可见性和生命周期。PHP 中有三种主要的作用域：

### 1. 脚本级作用域（全局作用域）

在函数外部定义的变量具有全局作用域，可以在脚本的任何地方访问（但在函数内部需要使用 `global` 关键字）。

```php
<?php
declare(strict_types=1);

// 全局变量
$globalVar = 'I am global';

function testFunction(): void
{
    // 不能直接访问全局变量
    // echo $globalVar;  // 会产生警告：未定义变量
    
    // 需要使用 global 关键字
    global $globalVar;
    echo $globalVar . "\n";  // 输出：I am global
}

testFunction();
echo $globalVar . "\n";  // 输出：I am global
```

### 2. 函数级作用域（局部作用域）

在函数内部定义的变量具有局部作用域，只能在函数内部访问。

```php
<?php
declare(strict_types=1);

function testFunction(): void
{
    $localVar = 'I am local';
    echo $localVar . "\n";  // 输出：I am local
}

testFunction();
// echo $localVar;  // 错误：未定义变量
```

### 3. 静态变量

静态变量在函数调用之间保持其值，但只能在定义它的函数内部访问。

```php
<?php
declare(strict_types=1);

function counter(): int
{
    static $count = 0;  // 只在第一次调用时初始化
    $count++;
    return $count;
}

echo counter() . "\n";  // 输出：1
echo counter() . "\n";  // 输出：2
echo counter() . "\n";  // 输出：3
```

## 动态类型系统

PHP 是动态类型语言，变量的类型由赋值时的值决定，并且可以在运行时改变类型。

### 类型自动推断

```php
<?php
declare(strict_types=1);

$var = 'Hello';      // $var 是 string 类型
$var = 42;           // $var 现在是 int 类型
$var = 3.14;         // $var 现在是 float 类型
$var = true;         // $var 现在是 bool 类型
$var = [1, 2, 3];    // $var 现在是 array 类型
```

### 类型检查

可以使用 `gettype()` 函数获取变量的类型：

```php
<?php
declare(strict_types=1);

$name = 'Alice';
echo gettype($name) . "\n";  // 输出：string

$age = 25;
echo gettype($age) . "\n";   // 输出：integer

$price = 19.99;
echo gettype($price) . "\n"; // 输出：double（float 的别名）

$isActive = true;
echo gettype($isActive) . "\n"; // 输出：boolean
```

### 类型提示（PHP 7.0+）

虽然变量本身是动态类型的，但函数参数和返回值可以使用类型提示：

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

$result = add(5, 3);  // 正确：8
// $result = add('5', 3);  // 在严格模式下会报错
```

## 变量常用操作

### 赋值操作

```php
<?php
declare(strict_types=1);

// 直接赋值
$name = 'Alice';
$age  = 25;
$price = 19.99;

// 链式赋值
$a = $b = $c = 10;  // 三个变量都赋值为 10

// 复合赋值运算符
$count = 5;
$count += 3;   // 等同于 $count = $count + 3，结果为 8
$count -= 2;   // 等同于 $count = $count - 2，结果为 6
$count *= 2;   // 等同于 $count = $count * 2，结果为 12
$count /= 3;   // 等同于 $count = $count / 3，结果为 4
```

### 变量检查函数

#### `isset()` - 检查变量是否已设置

`isset()` 检查变量是否存在且不为 `null`。

**语法**：`isset(mixed ...$vars): bool`

**参数**：
- `$vars`：要检查的变量（可以是多个）

**返回值**：如果变量存在且不为 `null`，返回 `true`；否则返回 `false`。

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$age  = null;
$city;  // 未定义

var_dump(isset($name));  // bool(true)
var_dump(isset($age));   // bool(false) - 因为值为 null
var_dump(isset($city));  // bool(false) - 因为未定义
var_dump(isset($name, $age));  // bool(false) - 所有变量都必须存在且不为 null
```

#### `empty()` - 检查变量是否为空

`empty()` 检查变量是否被认为是"空"的。

**语法**：`empty(mixed $var): bool`

**参数**：
- `$var`：要检查的变量

**返回值**：如果变量被认为是"空"的，返回 `true`；否则返回 `false`。

**被认为是"空"的值**：
- `""`（空字符串）
- `0`（整数 0）
- `0.0`（浮点数 0.0）
- `"0"`（字符串 "0"）
- `null`
- `false`
- `[]`（空数组）

```php
<?php
declare(strict_types=1);

var_dump(empty(""));      // bool(true)
var_dump(empty(0));       // bool(true)
var_dump(empty(0.0));     // bool(true)
var_dump(empty("0"));     // bool(true)
var_dump(empty(null));    // bool(true)
var_dump(empty(false));   // bool(true)
var_dump(empty([]));      // bool(true)
var_dump(empty("hello")); // bool(false)
var_dump(empty(42));      // bool(false)
```

#### `unset()` - 销毁变量

`unset()` 销毁指定的变量。

**语法**：`unset(mixed $var, mixed ...$vars): void`

**参数**：
- `$var, ...$vars`：要销毁的变量（可以是多个）

**返回值**：无返回值。

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$age  = 25;

echo isset($name) ? "Name exists\n" : "Name doesn't exist\n";  // Name exists

unset($name);

echo isset($name) ? "Name exists\n" : "Name doesn't exist\n";  // Name doesn't exist

// 可以同时销毁多个变量
unset($age);
```

### 变量存在性检查的最佳实践

```php
<?php
declare(strict_types=1);

// 推荐：先检查再使用
if (isset($user['name'])) {
    echo $user['name'];
} else {
    echo 'Unknown';
}

// 或者使用 null 合并运算符（PHP 7.0+）
$name = $user['name'] ?? 'Unknown';

// 不推荐：直接使用可能不存在的变量
// echo $user['name'];  // 如果不存在会产生警告
```

## 完整示例

### 示例 1：变量基础操作

```php
<?php
declare(strict_types=1);

// 定义变量
$firstName = 'Alice';
$lastName  = 'Smith';
$age       = 28;
$height    = 1.65;
$isActive  = true;

// 输出变量信息
echo "Name: {$firstName} {$lastName}\n";
echo "Age: {$age}\n";
echo "Height: {$height}m\n";
echo "Active: " . ($isActive ? 'Yes' : 'No') . "\n";

// 检查变量
echo "\nVariable checks:\n";
echo "isset(\$firstName): " . (isset($firstName) ? 'true' : 'false') . "\n";
echo "empty(\$age): " . (empty($age) ? 'true' : 'false') . "\n";

// 获取类型
echo "\nTypes:\n";
echo "gettype(\$firstName): " . gettype($firstName) . "\n";
echo "gettype(\$age): " . gettype($age) . "\n";
echo "gettype(\$height): " . gettype($height) . "\n";
```

**执行结果**：
```
Name: Alice Smith
Age: 28
Height: 1.65m
Active: Yes

Variable checks:
isset($firstName): true
empty($age): false

Types:
gettype($firstName): string
gettype($age): integer
gettype($height): double
```

### 示例 2：作用域演示

```php
<?php
declare(strict_types=1);

// 全局变量
$globalCounter = 0;

function incrementGlobal(): void
{
    global $globalCounter;
    $globalCounter++;
    echo "Global counter: {$globalCounter}\n";
}

function incrementLocal(): void
{
    $localCounter = 0;  // 每次调用都重新初始化为 0
    $localCounter++;
    echo "Local counter: {$localCounter}\n";
}

function incrementStatic(): void
{
    static $staticCounter = 0;  // 只在第一次调用时初始化
    $staticCounter++;
    echo "Static counter: {$staticCounter}\n";
}

echo "=== Global variable ===\n";
incrementGlobal();  // Global counter: 1
incrementGlobal();  // Global counter: 2
incrementGlobal();  // Global counter: 3

echo "\n=== Local variable ===\n";
incrementLocal();   // Local counter: 1
incrementLocal();   // Local counter: 1
incrementLocal();   // Local counter: 1

echo "\n=== Static variable ===\n";
incrementStatic();  // Static counter: 1
incrementStatic();  // Static counter: 2
incrementStatic();  // Static counter: 3
```

### 示例 3：动态类型演示

```php
<?php
declare(strict_types=1);

$value = 'Hello';
echo "Type: " . gettype($value) . ", Value: {$value}\n";

$value = 42;
echo "Type: " . gettype($value) . ", Value: {$value}\n";

$value = 3.14;
echo "Type: " . gettype($value) . ", Value: {$value}\n";

$value = true;
echo "Type: " . gettype($value) . ", Value: " . ($value ? 'true' : 'false') . "\n";

$value = [1, 2, 3];
echo "Type: " . gettype($value) . ", Value: " . json_encode($value) . "\n";
```

**执行结果**：
```
Type: string, Value: Hello
Type: integer, Value: 42
Type: double, Value: 3.14
Type: boolean, Value: true
Type: array, Value: [1,2,3]
```

## 注意事项

1. **未定义变量**：直接使用未定义的变量会产生警告（在 PHP 8.0+ 中可能产生错误）。始终使用 `isset()` 检查变量是否存在。

2. **变量覆盖**：PHP 允许变量在运行时改变类型，但这可能导致难以调试的问题。建议在严格模式下使用类型提示。

3. **全局变量**：过度使用全局变量会使代码难以维护和测试。优先使用函数参数和返回值传递数据。

4. **静态变量**：静态变量在函数调用之间保持状态，但要注意线程安全（在 CLI 模式下通常不是问题）。

5. **命名规范**：遵循一致的命名规范可以提高代码可读性。建议使用驼峰命名或下划线命名，并在整个项目中保持一致。

## 练习

1. 创建一个脚本，定义不同类型的变量（字符串、整数、浮点数、布尔值、数组），并输出它们的类型和值。

2. 编写一个函数 `checkVariable($var)`，检查变量是否存在、是否为空，并输出变量的类型和值。

3. 创建一个计数器函数，使用静态变量记录调用次数，每次调用时输出当前计数。

4. 编写一个函数，演示全局变量和局部变量的区别，并说明为什么在函数内部需要使用 `global` 关键字。

5. 创建一个脚本，演示变量的动态类型特性，将同一个变量依次赋值为不同类型的值，并输出每次赋值后的类型。
