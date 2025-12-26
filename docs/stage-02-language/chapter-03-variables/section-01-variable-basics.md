# 2.3.1 变量基础

## 概述

变量是存储数据的容器，是编程的基础概念。本节详细介绍 PHP 变量的声明、赋值、命名规则、变量作用域等基础知识，帮助理解 PHP 变量的动态特性。

理解变量的基本概念和使用方法是学习 PHP 的基础。PHP 变量具有动态类型特性，不需要预先声明类型，这使得 PHP 开发更加灵活，但也需要注意类型安全。

## 特性

- **动态类型**：PHP 变量是动态类型的，不需要声明类型
- **弱类型**：变量类型可以自动转换（在非严格模式下）
- **作用域**：变量有作用域限制（局部、全局）
- **灵活性**：变量可以存储任何类型的数据
- **命名规则**：变量名以 `$` 开头，遵循特定命名规则

## 语法/定义

### 变量声明和赋值

**基本语法**：`$variable_name = value;`

**组成部分**：
- `$`：变量标识符（必需）
- `variable_name`：变量名（遵循命名规则）
- `=`：赋值运算符
- `value`：要赋的值（可以是字面量、表达式、其他变量等）

**特点**：
- 变量在使用前不需要声明类型
- 变量类型由赋值的值决定
- 可以随时改变变量的类型和值

**示例**：

```php
<?php
declare(strict_types=1);

// 声明并赋值
$name = "World";
$age = 25;
$price = 99.99;
$active = true;
```

### 变量命名规则

**基本规则**：
1. **必须以 `$` 开头**：所有变量名必须以 `$` 符号开头
2. **首字符**：变量名首字符必须是字母（a-z, A-Z）或下划线（`_`）
3. **后续字符**：后续字符可以是字母、数字、下划线
4. **区分大小写**：变量名区分大小写（`$name` 和 `$Name` 是不同的变量）
5. **不能使用关键字**：不能使用 PHP 关键字作为变量名

**有效变量名示例**：

```php
<?php
declare(strict_types=1);

$name = "World";        // 有效
$name1 = "World";      // 有效
$_name = "World";      // 有效
$name_1 = "World";     // 有效
$userName = "World";   // 有效（驼峰命名）
$user_name = "World";  // 有效（蛇形命名）
```

**无效变量名示例**：

```php
<?php
declare(strict_types=1);

// $1name = "World";     // 错误：不能以数字开头
// $name-1 = "World";    // 错误：不能包含连字符
// $name.1 = "World";    // 错误：不能包含点号
// $if = "World";        // 错误：不能使用关键字
```

### 变量作用域

**作用域类型**：
- **局部作用域**：在函数内部声明的变量，只能在函数内部访问
- **全局作用域**：在函数外部声明的变量，可以在脚本的任何地方访问（函数内部需要使用 `global` 关键字）

**详细说明**：见 [2.3.4 全局与静态](section-04-scope-static.md)

## 基本用法

### 示例 1：基本变量操作

```php
<?php
declare(strict_types=1);

// 声明和赋值
$name = "World";
$age = 25;
$price = 99.99;
$active = true;

// 使用变量
echo "Name: {$name}\n";
echo "Age: {$age}\n";
echo "Price: {$price}\n";
echo "Active: " . ($active ? 'true' : 'false') . "\n";

// 修改变量值
$age = 26;
echo "New age: {$age}\n";

// 变量可以改变类型
$value = "Hello";
echo "Value: {$value}\n";
$value = 42;
echo "Value: {$value}\n";
```

**执行**：

```bash
php basic.php
```

**输出**：

```
Name: World
Age: 25
Price: 99.99
Active: true
New age: 26
Value: Hello
Value: 42
```

### 示例 2：变量作用域

```php
<?php
declare(strict_types=1);

// 全局变量
$globalVar = "I am global";

function testFunction(): void
{
    // 局部变量
    $localVar = "I am local";
    echo "Local: {$localVar}\n";
    
    // 无法直接访问全局变量
    // echo "Global: {$globalVar}\n";  // 错误：未定义变量
    
    // 需要使用 global 关键字
    global $globalVar;
    echo "Global: {$globalVar}\n";
}

testFunction();

// 全局作用域可以访问全局变量
echo "Global in main: {$globalVar}\n";

// 无法访问函数内的局部变量
// echo "Local in main: {$localVar}\n";  // 错误：未定义变量
```

**执行**：

```bash
php scope.php
```

**输出**：

```
Local: I am local
Global: I am global
Global in main: I am global
```

### 示例 3：变量类型动态变化

```php
<?php
declare(strict_types=1);

// 变量类型由值决定
$value = "Hello";        // 字符串
echo "Type: " . gettype($value) . ", Value: {$value}\n";

$value = 42;             // 整数
echo "Type: " . gettype($value) . ", Value: {$value}\n";

$value = 99.99;          // 浮点数
echo "Type: " . gettype($value) . ", Value: {$value}\n";

$value = true;           // 布尔值
echo "Type: " . gettype($value) . ", Value: " . ($value ? 'true' : 'false') . "\n";

$value = [1, 2, 3];      // 数组
echo "Type: " . gettype($value) . ", Value: " . json_encode($value) . "\n";
```

**执行**：

```bash
php dynamic-type.php
```

**输出**：

```
Type: string, Value: Hello
Type: integer, Value: 42
Type: double, Value: 99.99
Type: boolean, Value: true
Type: array, Value: [1,2,3]
```

**说明**：
- PHP 变量是动态类型的，可以随时改变类型
- 在严格类型模式下（`declare(strict_types=1)`），类型转换会受到限制
- 详细说明见 [2.5 类型转换与比较](../chapter-05-type-casting/readme.md)

## 完整代码示例

### 示例 1：变量命名规范

```php
<?php
declare(strict_types=1);

// 推荐：使用有意义的变量名
$userName = "Alice";
$userAge = 25;
$userEmail = "alice@example.com";

// 推荐：使用驼峰命名（camelCase）
$firstName = "Alice";
$lastName = "Smith";

// 推荐：使用蛇形命名（snake_case）
$first_name = "Alice";
$last_name = "Smith";

// 推荐：布尔值使用 is/has/can 前缀
$isActive = true;
$hasPermission = false;
$canEdit = true;

// 推荐：数组使用复数形式
$users = ["Alice", "Bob", "Charlie"];
$items = [1, 2, 3];

// 不推荐：使用无意义的变量名
// $a = "Alice";
// $b = 25;
// $c = true;
```

### 示例 2：变量赋值方式

```php
<?php
declare(strict_types=1);

// 直接赋值
$name = "World";

// 从其他变量赋值
$anotherName = $name;

// 从表达式赋值
$sum = 10 + 20;
$product = $sum * 2;

// 链式赋值
$a = $b = $c = 10;
echo "a: {$a}, b: {$b}, c: {$c}\n";  // a: 10, b: 10, c: 10

// 引用赋值（详细说明见 2.3.2）
$original = "Hello";
$reference = &$original;
$reference = "World";
echo "original: {$original}\n";  // original: World
```

### 示例 3：变量作用域实践

```php
<?php
declare(strict_types=1);

// 全局变量
$globalCounter = 0;

function incrementCounter(): void
{
    // 局部变量
    $localCounter = 0;
    $localCounter++;
    echo "Local counter: {$localCounter}\n";
    
    // 访问全局变量
    global $globalCounter;
    $globalCounter++;
    echo "Global counter: {$globalCounter}\n";
}

incrementCounter();  // Local: 1, Global: 1
incrementCounter();  // Local: 1, Global: 2

echo "Final global counter: {$globalCounter}\n";  // Final: 2
```

## 使用场景

### 数据存储

- **存储用户输入**：存储表单数据、用户信息等
- **存储计算结果**：存储计算结果、中间值等
- **存储配置信息**：存储应用配置、环境变量等

### 状态管理

- **程序状态**：存储程序运行状态、标志位等
- **计数器**：存储计数、索引等
- **临时数据**：存储临时计算结果、缓存等

### 数据传递

- **函数参数**：通过变量传递参数给函数
- **返回值**：存储函数返回值
- **数据交换**：在代码块之间交换数据

## 注意事项

### 变量命名

- **使用有意义的变量名**：变量名应该清晰表达变量的用途
- **遵循命名规范**：使用驼峰命名或蛇形命名，保持一致性
- **避免使用关键字**：不能使用 PHP 关键字作为变量名
- **区分大小写**：注意变量名的大小写

### 变量作用域

- **理解作用域规则**：局部变量和全局变量的作用域不同
- **避免全局变量滥用**：过度使用全局变量会导致代码难以维护
- **使用参数传递**：优先使用函数参数传递数据，而不是全局变量

### 类型安全

- **启用严格类型**：使用 `declare(strict_types=1);` 提高类型安全
- **注意类型转换**：理解自动类型转换的规则
- **使用类型声明**：为函数参数和返回值添加类型声明

### 未定义变量

- **在使用前声明**：确保变量在使用前已经声明和赋值
- **检查变量是否存在**：使用 `isset()` 检查变量是否存在
- **避免未定义变量警告**：PHP 8.0+ 对未定义变量更加严格

## 常见问题

### 问题 1：变量未定义

**症状**：

```
Warning: Undefined variable $name
```

**原因**：变量未声明就使用

**错误示例**：

```php
<?php
declare(strict_types=1);

// 错误：变量未定义
echo $name;  // Warning: Undefined variable $name
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 正确：先声明再使用
$name = "World";
echo $name;  // World

// 或使用 isset() 检查
if (isset($name)) {
    echo $name;
} else {
    echo "Variable not set";
}
```

**详细说明**：见 [2.12 isset / empty / Null 体系](../chapter-12-null-system/readme.md)

### 问题 2：作用域问题

**症状**：在函数内部无法访问全局变量

**原因**：函数内部无法直接访问全局变量

**错误示例**：

```php
<?php
declare(strict_types=1);

$globalVar = "Global";

function test(): void
{
    echo $globalVar;  // Warning: Undefined variable
}

test();
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$globalVar = "Global";

function test(): void
{
    // 方法1：使用 global 关键字
    global $globalVar;
    echo $globalVar . "\n";
    
    // 方法2：使用 $GLOBALS 超全局变量
    echo $GLOBALS['globalVar'] . "\n";
    
    // 方法3：通过参数传递（推荐）
    // 在调用时传递参数
}

test();
```

**详细说明**：见 [2.3.4 全局与静态](section-04-scope-static.md)

### 问题 3：变量名拼写错误

**症状**：变量值不符合预期，可能是创建了新变量

**原因**：变量名拼写错误，PHP 会创建新变量而不是报错

**错误示例**：

```php
<?php
declare(strict_types=1);

$userName = "Alice";
echo $userName . "\n";  // Alice

// 拼写错误
$userNmae = "Bob";      // 创建了新变量，而不是修改 $userName
echo $userName . "\n";  // 仍然是 Alice
```

**解决方法**：

1. **使用 IDE 检查**：使用 IDE 的代码检查功能
2. **使用静态分析工具**：使用 PHPStan 或 Psalm 检查
3. **启用严格模式**：PHP 8.0+ 对未定义变量更严格
4. **代码审查**：进行代码审查，检查变量名拼写

### 问题 4：变量类型意外变化

**症状**：变量类型不符合预期

**原因**：PHP 变量是动态类型的，类型可能意外变化

**示例**：

```php
<?php
declare(strict_types=1);

$value = "42";
$result = $value + 10;  // 字符串 "42" 被转换为整数 42
echo $result . "\n";    // 52

// 在严格类型模式下，类型转换受到限制
function add(int $a, int $b): int
{
    return $a + $b;
}

// add("42", 10);  // TypeError: Argument #1 must be of type int, string given
```

**解决方法**：

1. **启用严格类型**：使用 `declare(strict_types=1);`
2. **使用类型声明**：为函数参数和返回值添加类型声明
3. **显式类型转换**：需要时进行显式类型转换

**详细说明**：见 [2.5 类型转换与比较](../chapter-05-type-casting/readme.md)

## 最佳实践

### 变量命名

- **使用有意义的变量名**：变量名应该清晰表达变量的用途
- **遵循命名规范**：使用驼峰命名（`$userName`）或蛇形命名（`$user_name`），保持一致性
- **布尔值使用前缀**：使用 `is`、`has`、`can` 等前缀（`$isActive`、`$hasPermission`）
- **数组使用复数形式**：使用复数形式表示数组（`$users`、`$items`）

### 变量声明

- **在使用前声明**：确保变量在使用前已经声明和赋值
- **初始化变量**：为变量提供初始值，避免未定义变量警告
- **避免未定义变量**：使用 `isset()` 检查变量是否存在

### 作用域管理

- **优先使用局部变量**：优先使用局部变量，避免全局变量
- **通过参数传递**：通过函数参数传递数据，而不是使用全局变量
- **理解作用域规则**：理解局部变量和全局变量的作用域规则

### 类型安全

- **启用严格类型**：使用 `declare(strict_types=1);` 提高类型安全
- **使用类型声明**：为函数参数和返回值添加类型声明
- **注意类型转换**：理解自动类型转换的规则和限制

## 对比分析

### 动态类型 vs 静态类型

| 特性 | PHP（动态类型） | 静态类型语言（如 Java） |
|:-----|:----------------|:------------------------|
| 类型声明 | 不需要 | 必须 |
| 类型检查 | 运行时 | 编译时 |
| 类型转换 | 自动 | 显式 |
| 灵活性 | 高 | 低 |
| 类型安全 | 低（默认） | 高 |
| 性能 | 中等 | 高 |

**PHP 8.0+ 改进**：
- 支持类型声明（函数参数、返回值、属性）
- 支持联合类型、交集类型
- 严格类型模式提高类型安全

### 变量作用域对比

| 特性 | 局部变量 | 全局变量 |
|:-----|:---------|:---------|
| 声明位置 | 函数内部 | 函数外部 |
| 可见范围 | 函数内部 | 脚本任何地方 |
| 生命周期 | 函数执行期间 | 脚本执行期间 |
| 推荐度 | 推荐 | 不推荐（过度使用） |

**选择建议**：
- **优先使用局部变量**：提高代码可维护性
- **避免全局变量**：除非必要，避免使用全局变量
- **通过参数传递**：通过函数参数传递数据

## 相关章节

- **2.3.2 引用与可变变量**：了解引用和可变变量的使用
- **2.3.3 常量**：了解常量的定义和使用
- **2.3.4 全局与静态**：详细了解全局变量和静态变量
- **2.4 数据类型**：了解 PHP 的数据类型系统
- **2.5 类型转换与比较**：了解类型转换规则
- **2.10 函数与作用域**：深入学习函数和作用域
- **2.12 isset / empty / Null 体系**：了解变量检查函数

## 练习任务

1. **变量声明练习**：
   - 声明不同类型的变量（字符串、整数、浮点数、布尔值、数组）
   - 练习修改变量值
   - 观察变量类型的动态变化

2. **变量命名练习**：
   - 练习使用有意义的变量名
   - 练习使用驼峰命名和蛇形命名
   - 练习布尔值变量的命名（is/has/can 前缀）

3. **变量作用域练习**：
   - 创建全局变量和局部变量
   - 练习在函数内部访问全局变量
   - 理解局部变量和全局变量的区别

4. **变量赋值练习**：
   - 练习直接赋值、从其他变量赋值、从表达式赋值
   - 练习链式赋值
   - 练习引用赋值（见 2.3.2）

5. **综合练习**：
   - 创建一个程序，使用不同类型的变量
   - 实现变量作用域的演示
   - 练习变量命名规范和最佳实践
   - 测试变量类型动态变化的行为
