# 2.4.1 标量类型

## 概述

标量类型是 PHP 的基本数据类型，包括布尔类型（`bool`）、整数类型（`int`）、浮点数类型（`float`）、字符串类型（`string`）。本节详细介绍这四种标量类型的语法、特性、常用函数、类型检测和使用场景。

理解标量类型是学习 PHP 的基础。每种标量类型都有其特定的用途和限制，掌握它们的特点可以帮助编写更可靠的代码。

## 特性

- **布尔类型（bool）**：表示真值（`true`）和假值（`false`），不区分大小写
- **整数类型（int）**：表示整数，支持多种进制表示（十进制、八进制、十六进制、二进制）
- **浮点数类型（float）**：表示浮点数，支持科学计数法，精度有限
- **字符串类型（string）**：表示文本数据，支持多种创建方式（单引号、双引号、Heredoc、Nowdoc）

## 语法/定义

### 布尔类型（bool）

**值**：
- `true`：真值（不区分大小写，`True`、`TRUE` 都可以）
- `false`：假值（不区分大小写，`False`、`FALSE` 都可以）

**类型检测**：`is_bool(mixed $value): bool`

**参数**：
- `$value`：要检测的值（任意类型）

**返回值**：如果 `$value` 是布尔类型返回 `true`，否则返回 `false`

**转换规则**：
- 以下值转换为 `false`：`false`、`0`、`0.0`、`""`（空字符串）、`"0"`、`[]`（空数组）、`null`
- 其他值转换为 `true`

### 整数类型（int）

**表示方式**：
- **十进制**：`42`（最常用）
- **八进制**：`052`（以 `0` 开头，PHP 8.1+ 已废弃，建议使用 `0o52`）
- **十六进制**：`0x2A` 或 `0X2A`（以 `0x` 或 `0X` 开头）
- **二进制**：`0b101010` 或 `0B101010`（以 `0b` 或 `0B` 开头）

**类型检测**：`is_int(mixed $value): bool`

**参数**：
- `$value`：要检测的值（任意类型）

**返回值**：如果 `$value` 是整数类型返回 `true`，否则返回 `false`

**范围**：
- 32 位系统：-2,147,483,648 到 2,147,483,647（-2^31 到 2^31-1）
- 64 位系统：-9,223,372,036,854,775,808 到 9,223,372,036,854,775,807（-2^63 到 2^63-1）
- 超出范围会转换为浮点数

### 浮点数类型（float）

**表示方式**：
- **普通形式**：`3.14`、`0.5`、`.5`（等同于 `0.5`）
- **科学计数法**：`1.23e4`（12300）、`1.23E4`、`1.23e-4`（0.000123）

**类型检测**：`is_float(mixed $value): bool` 或 `is_double(mixed $value): bool`（`float` 和 `double` 在 PHP 中相同）

**参数**：
- `$value`：要检测的值（任意类型）

**返回值**：如果 `$value` 是浮点数类型返回 `true`，否则返回 `false`

**精度**：
- 使用 IEEE 754 双精度格式
- 精度有限，不能精确表示所有小数
- 通常精度约为 14-15 位有效数字

### 字符串类型（string）

**创建方式**：
- **单引号**：`'Hello'`（不支持变量解析和转义序列，除了 `\'` 和 `\\`）
- **双引号**：`"Hello"`（支持变量解析和转义序列）
- **Heredoc**：`<<<TEXT ... TEXT;`（支持变量解析，类似双引号）
- **Nowdoc**：`<<<'TEXT' ... TEXT;`（不支持变量解析，类似单引号）

**类型检测**：`is_string(mixed $value): bool`

**参数**：
- `$value`：要检测的值（任意类型）

**返回值**：如果 `$value` 是字符串类型返回 `true`，否则返回 `false`

**长度**：理论上无限制，受 `memory_limit` 配置限制

**编码**：应使用 UTF-8 编码（无 BOM）

## 基本用法

### 示例 1：布尔类型

```php
<?php
declare(strict_types=1);

// 布尔值
$isActive = true;
$isDeleted = false;

// 类型检测
var_dump(is_bool($isActive));   // bool(true)
var_dump(is_bool($isDeleted));  // bool(true)
var_dump(is_bool(1));           // bool(false)

// 布尔转换
var_dump((bool) 0);        // bool(false)
var_dump((bool) 1);        // bool(true)
var_dump((bool) "");       // bool(false)
var_dump((bool) "0");      // bool(false)
var_dump((bool) "Hello");  // bool(true)
var_dump((bool) []);       // bool(false)
var_dump((bool) [1, 2]);   // bool(true)
var_dump((bool) null);     // bool(false)

// 条件判断
if ($isActive) {
    echo "User is active\n";
} else {
    echo "User is inactive\n";
}
```

**执行**：

```bash
php bool-example.php
```

**输出**：

```
bool(true)
bool(true)
bool(false)
bool(false)
bool(true)
bool(false)
bool(false)
bool(true)
bool(false)
bool(true)
bool(false)
User is active
```

### 示例 2：整数类型

```php
<?php
declare(strict_types=1);

// 十进制
$decimal = 42;
echo "Decimal: {$decimal}\n";

// 八进制（PHP 8.1+ 已废弃，建议使用 0o 前缀）
$octal = 052;  // 42（不推荐）
echo "Octal: {$octal}\n";

// 十六进制
$hex = 0x2A;   // 42
echo "Hex: {$hex}\n";

// 二进制
$binary = 0b101010;  // 42
echo "Binary: {$binary}\n";

// 类型检测
var_dump(is_int($decimal));  // bool(true)
var_dump(is_int(42.0));      // bool(false) - 浮点数
var_dump(is_int("42"));      // bool(false) - 字符串

// 整数范围
echo "PHP_INT_MAX: " . PHP_INT_MAX . "\n";
echo "PHP_INT_MIN: " . PHP_INT_MIN . "\n";
echo "PHP_INT_SIZE: " . PHP_INT_SIZE . " bytes\n";
```

**执行**：

```bash
php int-example.php
```

**输出**：

```
Decimal: 42
Octal: 42
Hex: 42
Binary: 42
bool(true)
bool(false)
bool(false)
PHP_INT_MAX: 9223372036854775807
PHP_INT_MIN: -9223372036854775808
PHP_INT_SIZE: 8 bytes
```

### 示例 3：浮点数类型

```php
<?php
declare(strict_types=1);

// 普通形式
$price = 99.99;
$rate = 0.5;
$small = .5;  // 等同于 0.5

// 科学计数法
$large = 1.23e4;    // 12300
$small = 1.23e-4;   // 0.000123
$veryLarge = 1.23E10;  // 12300000000

echo "Price: {$price}\n";
echo "Large: {$large}\n";
echo "Small: {$small}\n";

// 类型检测
var_dump(is_float($price));  // bool(true)
var_dump(is_float(99));      // bool(false) - 整数
var_dump(is_float("99.99")); // bool(false) - 字符串

// 浮点数精度问题
$result = 0.1 + 0.2;
echo "0.1 + 0.2 = {$result}\n";  // 0.30000000000000004
var_dump($result === 0.3);       // bool(false)

// 正确的比较方式
$epsilon = 0.00001;
var_dump(abs($result - 0.3) < $epsilon);  // bool(true)
```

**执行**：

```bash
php float-example.php
```

**输出**：

```
Price: 99.99
Large: 12300
Small: 0.000123
bool(true)
bool(false)
bool(false)
0.1 + 0.2 = 0.30000000000000004
bool(false)
bool(true)
```

### 示例 4：字符串类型

```php
<?php
declare(strict_types=1);

// 单引号（不支持变量解析）
$single = 'Hello, World!';
echo $single . "\n";

// 双引号（支持变量解析）
$name = "World";
$double = "Hello, {$name}!";
echo $double . "\n";

// Heredoc（支持变量解析）
$heredoc = <<<TEXT
Multi-line
string with {$name}
TEXT;
echo $heredoc . "\n";

// Nowdoc（不支持变量解析）
$nowdoc = <<<'TEXT'
Multi-line
string without variable parsing
TEXT;
echo $nowdoc . "\n";

// 类型检测
var_dump(is_string($single));  // bool(true)
var_dump(is_string(42));        // bool(false) - 整数

// 字符串长度
echo "Length: " . strlen($single) . "\n";

// 字符串连接
$concatenated = $single . " " . $double;
echo $concatenated . "\n";
```

**执行**：

```bash
php string-example.php
```

**输出**：

```
Hello, World!
Hello, World!
Multi-line
string with World
Multi-line
string without variable parsing
bool(true)
bool(false)
Length: 13
Hello, World! Hello, World!
```

## 完整代码示例

### 示例 1：类型检测函数

```php
<?php
declare(strict_types=1);

function checkType(mixed $value): void
{
    echo "Value: ";
    var_dump($value);
    echo "Type: " . gettype($value) . "\n";
    echo "is_bool: " . (is_bool($value) ? 'true' : 'false') . "\n";
    echo "is_int: " . (is_int($value) ? 'true' : 'false') . "\n";
    echo "is_float: " . (is_float($value) ? 'true' : 'false') . "\n";
    echo "is_string: " . (is_string($value) ? 'true' : 'false') . "\n";
    echo "---\n";
}

checkType(true);
checkType(42);
checkType(99.99);
checkType("Hello");
```

### 示例 2：类型转换

```php
<?php
declare(strict_types=1);

// 转换为布尔值
$bool1 = (bool) 1;        // true
$bool2 = (bool) 0;        // false
$bool3 = (bool) "Hello"; // true
$bool4 = (bool) "";      // false

// 转换为整数
$int1 = (int) 99.99;     // 99
$int2 = (int) "42";      // 42
$int3 = (int) "42abc";   // 42
$int4 = (int) true;      // 1
$int5 = (int) false;     // 0

// 转换为浮点数
$float1 = (float) 42;     // 42.0
$float2 = (float) "99.99"; // 99.99
$float3 = (float) "99.99abc"; // 99.99

// 转换为字符串
$str1 = (string) 42;      // "42"
$str2 = (string) 99.99;  // "99.99"
$str3 = (string) true;   // "1"
$str4 = (string) false;  // ""

echo "Bool: " . ($bool1 ? 'true' : 'false') . "\n";
echo "Int: {$int1}\n";
echo "Float: {$float1}\n";
echo "String: {$str1}\n";
```

### 示例 3：实际应用场景

```php
<?php
declare(strict_types=1);

// 用户状态管理
class UserStatus
{
    public function __construct(
        private bool $isActive,
        private int $userId,
        private float $balance,
        private string $username
    ) {}
    
    public function isActive(): bool
    {
        return $this->isActive;
    }
    
    public function getUserId(): int
    {
        return $this->userId;
    }
    
    public function getBalance(): float
    {
        return $this->balance;
    }
    
    public function getUsername(): string
    {
        return $this->username;
    }
    
    public function formatBalance(): string
    {
        return sprintf("$%.2f", $this->balance);
    }
}

$user = new UserStatus(true, 12345, 99.99, "alice");
echo "User: {$user->getUsername()}\n";
echo "Active: " . ($user->isActive() ? 'Yes' : 'No') . "\n";
echo "Balance: {$user->formatBalance()}\n";
```

## 使用场景

### 布尔类型

- **条件判断**：`if`、`while`、`for` 等控制结构
- **标志位**：表示开关状态、是否启用等
- **函数返回值**：表示操作成功或失败
- **配置选项**：表示配置的开启或关闭

### 整数类型

- **计数**：循环计数器、数组索引
- **ID**：用户ID、订单ID等唯一标识
- **状态码**：HTTP状态码、错误代码
- **数量**：商品数量、用户数量等

### 浮点数类型

- **价格**：商品价格、订单金额
- **坐标**：地理位置坐标、图形坐标
- **百分比**：折扣率、完成度等
- **科学计算**：物理计算、统计分析

### 字符串类型

- **文本处理**：用户输入、文本内容
- **数据展示**：格式化输出、日志记录
- **配置信息**：配置文件内容、环境变量
- **标识符**：用户名、邮箱地址、URL等

## 注意事项

### 布尔类型

- **不区分大小写**：`true`、`True`、`TRUE` 都可以
- **转换规则**：理解各种值到布尔值的转换规则
- **类型检测**：使用 `is_bool()` 而不是 `=== true` 或 `=== false`（除非确实需要严格比较）

### 整数类型

- **范围限制**：注意整数范围，超出范围会转换为浮点数
- **进制表示**：八进制表示在 PHP 8.1+ 已废弃，建议使用 `0o` 前缀
- **类型检测**：使用 `is_int()` 检测整数类型

### 浮点数类型

- **精度问题**：浮点数不能精确表示所有小数，比较时使用误差范围
- **科学计数法**：理解科学计数法的表示方法
- **类型检测**：`is_float()` 和 `is_double()` 相同

### 字符串类型

- **编码统一**：统一使用 UTF-8 编码（无 BOM）
- **单引号 vs 双引号**：理解单引号和双引号的区别
- **Heredoc/Nowdoc**：理解 Heredoc 和 Nowdoc 的使用场景
- **长度限制**：理论上无限制，但受内存限制

## 常见问题

### 问题 1：浮点数精度问题

**症状**：浮点数计算结果不符合预期

**原因**：浮点数精度有限，不能精确表示所有小数

**错误示例**：

```php
<?php
declare(strict_types=1);

$result = 0.1 + 0.2;
var_dump($result === 0.3);  // bool(false)
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：使用误差范围比较
$result = 0.1 + 0.2;
$epsilon = 0.00001;
var_dump(abs($result - 0.3) < $epsilon);  // bool(true)

// 方法2：使用 bcmath 扩展进行精确计算
// bcadd("0.1", "0.2", 2) 返回 "0.30"
```

**详细说明**：见 [2.5 类型转换与比较](../chapter-05-type-casting/readme.md)

### 问题 2：字符串编码问题

**症状**：中文字符显示乱码

**原因**：字符串编码不一致

**解决方法**：

1. **统一使用 UTF-8 编码**：确保文件使用 UTF-8 编码（无 BOM）
2. **设置 HTTP 头**（Web 模式）：`header('Content-Type: text/html; charset=UTF-8');`
3. **使用 mbstring 扩展**：处理多字节字符串

**详细说明**：见 [2.1.2 文件结构与编码](../chapter-01-syntax/section-02-file-structure.md) 和 [2.7 字符串操作](../chapter-07-strings/readme.md)

### 问题 3：整数溢出

**症状**：大整数计算结果不正确

**原因**：整数超出范围，转换为浮点数

**解决方法**：

```php
<?php
declare(strict_types=1);

// 检查整数范围
$large = PHP_INT_MAX;
echo "Max int: {$large}\n";

// 超出范围会转换为浮点数
$tooLarge = PHP_INT_MAX + 1;
var_dump(is_int($tooLarge));  // bool(false)
var_dump(is_float($tooLarge)); // bool(true)

// 使用 BCMath 扩展处理大数
// bcadd("9223372036854775807", "1") 返回 "9223372036854775808"
```

### 问题 4：布尔值比较

**症状**：布尔值比较结果不符合预期

**原因**：使用了错误的比较方式

**错误示例**：

```php
<?php
declare(strict_types=1);

$value = "0";
if ($value == false) {  // 使用 == 会进行类型转换
    echo "This will execute\n";
}

if ($value === false) {  // 使用 === 严格比较
    echo "This will not execute\n";
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$value = "0";

// 检查是否为布尔类型
if (is_bool($value)) {
    // 处理布尔值
}

// 检查是否为假值
if (!$value) {
    // 处理假值
}

// 严格比较
if ($value === false) {
    // 处理 false
}
```

## 最佳实践

### 类型声明

- **使用类型声明**：为函数参数和返回值添加类型声明
- **启用严格模式**：使用 `declare(strict_types=1);` 提高类型安全
- **类型检测**：使用 `is_*()` 系列函数进行类型检测

### 浮点数处理

- **注意精度问题**：比较浮点数时使用误差范围
- **使用 BCMath**：需要精确计算时使用 BCMath 扩展
- **避免直接比较**：不要直接使用 `===` 比较浮点数

### 字符串处理

- **统一编码**：统一使用 UTF-8 编码
- **选择合适的创建方式**：根据场景选择单引号、双引号、Heredoc 或 Nowdoc
- **注意转义**：理解转义序列的使用

### 类型转换

- **显式转换**：需要时进行显式类型转换
- **理解转换规则**：理解各种类型之间的转换规则
- **避免隐式转换**：在严格模式下，避免依赖隐式转换

## 对比分析

### 单引号 vs 双引号

| 特性 | 单引号 | 双引号 |
|:-----|:-------|:-------|
| 变量解析 | 不支持 | 支持 |
| 转义序列 | 仅 `\'` 和 `\\` | 支持所有转义序列 |
| 性能 | 略好 | 略差 |
| 推荐场景 | 简单字符串 | 需要变量解析 |

**选择建议**：
- **简单字符串**：使用单引号
- **需要变量解析**：使用双引号
- **多行字符串**：使用 Heredoc 或 Nowdoc

### 整数 vs 浮点数

| 特性 | 整数 | 浮点数 |
|:-----|:-----|:-------|
| 精度 | 精确 | 有限 |
| 范围 | 有限 | 更大 |
| 性能 | 更好 | 略差 |
| 适用场景 | 计数、ID | 价格、坐标 |

**选择建议**：
- **计数、ID**：使用整数
- **价格、坐标**：使用浮点数
- **需要精确计算**：使用 BCMath 扩展

## 相关章节

- **2.4.4 类型检测**：详细了解类型检测函数
- **2.5 类型转换与比较**：详细了解类型转换规则
- **2.7 字符串操作**：详细了解字符串处理
- **2.1.2 文件结构与编码**：了解编码规范

## 练习任务

1. **类型检测练习**：
   - 使用 `is_bool()`、`is_int()`、`is_float()`、`is_string()` 检测不同类型
   - 理解各种值的类型检测结果
   - 练习类型转换

2. **整数表示练习**：
   - 练习使用不同进制表示整数
   - 理解整数范围限制
   - 处理整数溢出情况

3. **浮点数精度练习**：
   - 观察浮点数精度问题
   - 练习使用误差范围比较浮点数
   - 理解科学计数法

4. **字符串创建练习**：
   - 练习使用单引号、双引号创建字符串
   - 练习使用 Heredoc 和 Nowdoc
   - 理解变量解析的区别

5. **综合练习**：
   - 创建一个类型检测工具函数
   - 实现一个数值计算函数，处理精度问题
   - 实现一个字符串格式化函数
   - 测试不同场景下的类型行为
