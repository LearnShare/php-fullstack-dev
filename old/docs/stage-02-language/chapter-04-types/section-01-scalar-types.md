# 2.4.1 标量类型

## 概述

标量类型（Scalar Types）是 PHP 中最基本的数据类型，包括布尔值（`bool`）、整数（`int`）、浮点数（`float`）和字符串（`string`）。这些类型只能存储单个值，不能包含其他值。

## 布尔类型（bool）

### 基本概念

布尔类型表示逻辑值，只有两个可能的值：`true` 和 `false`。

### 语法

```php
$bool = true;
$bool = false;
```

### 转换为布尔值的规则

以下值在转换为布尔类型时会被视为 `false`（falsy 值）：

- `false` 本身
- `0`（整数零）
- `0.0`（浮点数零）
- `"0"`（字符串 "0"）
- `""`（空字符串）
- `[]`（空数组）
- `null`

所有其他值都被视为 `true`。

### 类型检测

#### `is_bool()`

**语法**：`is_bool(mixed $value): bool`

**参数**：
- `$value`：要检测的值

**返回值**：如果 `$value` 是布尔类型，返回 `true`；否则返回 `false`。

```php
<?php
declare(strict_types=1);

var_dump(is_bool(true));   // bool(true)
var_dump(is_bool(false));  // bool(true)
var_dump(is_bool(0));      // bool(false)
var_dump(is_bool("true")); // bool(false)
```

### 从字符串解析布尔值

#### `filter_var()` 与 `FILTER_VALIDATE_BOOLEAN`

**语法**：`filter_var(mixed $value, int $filter = FILTER_DEFAULT, array|int $options = null): mixed`

**参数**：
- `$value`：要过滤的值
- `$filter`：过滤器类型，使用 `FILTER_VALIDATE_BOOLEAN`
- `$options`：可选，过滤器选项

**返回值**：成功返回布尔值，失败返回 `false`。

```php
<?php
declare(strict_types=1);

// 使用 filter_var 解析布尔值
var_dump(filter_var('true', FILTER_VALIDATE_BOOLEAN));   // bool(true)
var_dump(filter_var('false', FILTER_VALIDATE_BOOLEAN));  // bool(false)
var_dump(filter_var('yes', FILTER_VALIDATE_BOOLEAN));     // bool(true)
var_dump(filter_var('no', FILTER_VALIDATE_BOOLEAN));      // bool(false)
var_dump(filter_var('1', FILTER_VALIDATE_BOOLEAN));      // bool(true)
var_dump(filter_var('0', FILTER_VALIDATE_BOOLEAN));       // bool(false)
var_dump(filter_var('on', FILTER_VALIDATE_BOOLEAN));      // bool(true)
var_dump(filter_var('off', FILTER_VALIDATE_BOOLEAN));     // bool(false)
```

### 完整示例

```php
<?php
declare(strict_types=1);

// 布尔值定义
$isActive = true;
$isDeleted = false;

// 条件判断
if ($isActive) {
    echo "用户处于活跃状态\n";
}

// 类型检测
function checkBoolean(mixed $value): void
{
    if (is_bool($value)) {
        echo "是布尔类型: " . ($value ? 'true' : 'false') . "\n";
    } else {
        echo "不是布尔类型\n";
    }
}

checkBoolean(true);   // 是布尔类型: true
checkBoolean(false);  // 是布尔类型: false
checkBoolean(1);      // 不是布尔类型

// 从用户输入解析布尔值
function parseBooleanInput(string $input): bool
{
    return filter_var($input, FILTER_VALIDATE_BOOLEAN) !== false;
}

echo parseBooleanInput('true') ? 'true' : 'false';   // true
echo parseBooleanInput('false') ? 'true' : 'false';   // false
```

## 整数类型（int）

### 基本概念

整数类型表示没有小数部分的数字。PHP 中的整数大小取决于平台：

- **32 位平台**：范围约为 `-2,147,483,648` 到 `2,147,483,647`（`-2^31` 到 `2^31-1`）
- **64 位平台**：范围约为 `-9,223,372,036,854,775,808` 到 `9,223,372,036,854,775,807`（`-2^63` 到 `2^63-1`）

### 语法

#### 十进制

```php
$decimal = 42;
$negative = -100;
```

#### 二进制（PHP 5.4+）

```php
$binary = 0b1010;  // 等于十进制的 10
```

#### 八进制

```php
$octal = 012;  // 等于十进制的 10（注意：PHP 8.1+ 中 0 前缀已废弃，建议使用 0o12）
$octal = 0o12; // PHP 8.1+ 推荐写法
```

#### 十六进制

```php
$hex = 0x1A;  // 等于十进制的 26
```

### 类型检测

#### `is_int()`

**语法**：`is_int(mixed $value): bool`

**参数**：
- `$value`：要检测的值

**返回值**：如果 `$value` 是整数类型，返回 `true`；否则返回 `false`。

```php
<?php
declare(strict_types=1);

var_dump(is_int(42));      // bool(true)
var_dump(is_int(42.0));    // bool(false)
var_dump(is_int("42"));    // bool(false)
var_dump(is_int(0));       // bool(true)
var_dump(is_int(-100));    // bool(true)
```

#### `is_integer()` 和 `is_long()`

`is_integer()` 和 `is_long()` 是 `is_int()` 的别名，功能完全相同。

### 整数运算函数

#### `intdiv()` - 整数除法

**语法**：`intdiv(int $dividend, int $divisor): int`

**参数**：
- `$dividend`：被除数
- `$divisor`：除数

**返回值**：返回整数除法结果。

**注意**：如果除数为 0，会抛出 `DivisionByZeroError` 异常。

```php
<?php
declare(strict_types=1);

echo intdiv(10, 3) . "\n";   // 3
echo intdiv(10, 2) . "\n";   // 5
echo intdiv(-10, 3) . "\n";  // -3

// 注意：不能除以 0
// echo intdiv(10, 0);  // DivisionByZeroError
```

#### `intval()` - 转换为整数

**语法**：`intval(mixed $value, int $base = 10): int`

**参数**：
- `$value`：要转换的值
- `$base`：进制基数（2-36），默认为 10

**返回值**：返回转换后的整数值。

```php
<?php
declare(strict_types=1);

echo intval(42) . "\n";           // 42
echo intval(42.7) . "\n";         // 42
echo intval("42") . "\n";         // 42
echo intval("42abc") . "\n";      // 42（忽略非数字字符）
echo intval("0xff", 16) . "\n";   // 255（十六进制）
echo intval("1010", 2) . "\n";    // 10（二进制）
```

### 整数溢出

当整数超出平台支持的最大值时，PHP 会自动将其转换为浮点数：

```php
<?php
declare(strict_types=1);

$large = PHP_INT_MAX;  // 最大整数值
echo gettype($large) . "\n";  // integer

$overflow = $large + 1;
echo gettype($overflow) . "\n";  // double（浮点数）
```

### 完整示例

```php
<?php
declare(strict_types=1);

// 不同进制的整数
$decimal = 42;
$binary  = 0b1010;   // 10
$octal   = 0o12;    // 10（PHP 8.1+）
$hex     = 0x1A;    // 26

echo "Decimal: {$decimal}\n";
echo "Binary: {$binary}\n";
echo "Octal: {$octal}\n";
echo "Hex: {$hex}\n";

// 整数运算
function calculate(int $a, int $b): array
{
    return [
        'sum' => $a + $b,
        'difference' => $a - $b,
        'product' => $a * $b,
        'quotient' => intdiv($a, $b),
        'remainder' => $a % $b,
        'power' => $a ** $b
    ];
}

$result = calculate(10, 3);
print_r($result);
// 输出：
// Array
// (
//     [sum] => 13
//     [difference] => 7
//     [product] => 30
//     [quotient] => 3
//     [remainder] => 1
//     [power] => 1000
// )

// 类型检测
function checkInteger(mixed $value): void
{
    if (is_int($value)) {
        echo "是整数: {$value}\n";
    } else {
        echo "不是整数: " . gettype($value) . "\n";
    }
}

checkInteger(42);      // 是整数: 42
checkInteger(42.0);    // 不是整数: double
checkInteger("42");    // 不是整数: string
```

## 浮点数类型（float）

### 基本概念

浮点数（也称为双精度数）用于表示带小数部分的数字。PHP 使用 IEEE 754 双精度浮点数标准。

### 语法

```php
$float = 3.14;
$float = 0.5;
$float = -10.25;
$float = 1.2e3;  // 科学计数法：1.2 × 10^3 = 1200
$float = 1.2E-3; // 科学计数法：1.2 × 10^-3 = 0.0012
```

### 类型检测

#### `is_float()`

**语法**：`is_float(mixed $value): bool`

**参数**：
- `$value`：要检测的值

**返回值**：如果 `$value` 是浮点数类型，返回 `true`；否则返回 `false`。

```php
<?php
declare(strict_types=1);

var_dump(is_float(3.14));   // bool(true)
var_dump(is_float(42));     // bool(false)
var_dump(is_float("3.14"));  // bool(false)
```

#### `is_double()`

`is_double()` 是 `is_float()` 的别名。

### 浮点数精度问题

浮点数使用二进制表示，某些十进制数无法精确表示，导致精度误差：

```php
<?php
declare(strict_types=1);

$a = 0.1;
$b = 0.2;
$c = $a + $b;

echo $c . "\n";  // 0.3
var_dump($c === 0.3);  // bool(false) - 注意：不相等！

// 正确比较浮点数
$epsilon = 0.00001;
var_dump(abs($c - 0.3) < $epsilon);  // bool(true)
```

### 浮点数运算函数

#### `round()` - 四舍五入

**语法**：`round(float $num, int $precision = 0, int $mode = PHP_ROUND_HALF_UP): float`

**参数**：
- `$num`：要舍入的数字
- `$precision`：小数位数，默认为 0
- `$mode`：舍入模式

**返回值**：返回舍入后的浮点数。

```php
<?php
declare(strict_types=1);

echo round(3.4) . "\n";   // 3
echo round(3.5) . "\n";   // 4
echo round(3.6) . "\n";   // 4
echo round(3.14159, 2) . "\n";  // 3.14
echo round(3.14159, 3) . "\n";  // 3.142
```

#### `floor()` - 向下取整

**语法**：`floor(float $num): float`

**参数**：
- `$num`：要取整的数字

**返回值**：返回向下取整后的浮点数。

```php
<?php
declare(strict_types=1);

echo floor(3.7) . "\n";   // 3
echo floor(-3.7) . "\n";  // -4
```

#### `ceil()` - 向上取整

**语法**：`ceil(float $num): float`

**参数**：
- `$num`：要取整的数字

**返回值**：返回向上取整后的浮点数。

```php
<?php
declare(strict_types=1);

echo ceil(3.2) . "\n";   // 4
echo ceil(-3.2) . "\n";  // -3
```

#### `floatval()` - 转换为浮点数

**语法**：`floatval(mixed $value): float`

**参数**：
- `$value`：要转换的值

**返回值**：返回转换后的浮点数。

```php
<?php
declare(strict_types=1);

echo floatval(42) . "\n";        // 42
echo floatval("42.5") . "\n";    // 42.5
echo floatval("42.5abc") . "\n"; // 42.5（忽略非数字字符）
```

### 高精度计算

对于需要高精度的场景（如货币计算），应使用 `bcmath` 扩展：

```php
<?php
declare(strict_types=1);

// 使用 bcmath 进行高精度计算
$price1 = "19.99";
$price2 = "0.01";
$total = bcadd($price1, $price2, 2);  // 精度为 2 位小数

echo $total . "\n";  // 20.00

// bcmath 函数
$result1 = bcmul("10.5", "2.3", 2);  // 24.15
$result2 = bcdiv("10", "3", 4);      // 3.3333
$result3 = bcsub("10.5", "2.3", 2);  // 8.20
```

### 完整示例

```php
<?php
declare(strict_types=1);

// 浮点数定义
$pi = 3.14159;
$e = 2.71828;
$scientific = 1.2e3;  // 1200

// 浮点数运算
function calculateArea(float $radius): float
{
    return M_PI * $radius ** 2;
}

$area = calculateArea(5.5);
echo "Area: " . round($area, 2) . "\n";  // Area: 95.03

// 处理精度问题
function safeFloatCompare(float $a, float $b, float $epsilon = 0.00001): bool
{
    return abs($a - $b) < $epsilon;
}

var_dump(safeFloatCompare(0.1 + 0.2, 0.3));  // bool(true)

// 货币计算（使用 bcmath）
function calculateTotal(array $prices): string
{
    $total = "0.00";
    foreach ($prices as $price) {
        $total = bcadd($total, (string)$price, 2);
    }
    return $total;
}

$prices = ["19.99", "29.99", "9.99"];
$total = calculateTotal($prices);
echo "Total: \${$total}\n";  // Total: $59.97
```

## 字符串类型（string）

### 基本概念

字符串是一系列字符的序列。PHP 中的字符串可以包含任意字节，包括二进制数据。

### 语法

#### 单引号

单引号字符串中的内容会被原样输出，不支持变量插值和转义序列（除了 `\'` 和 `\\`）：

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$str1 = 'Hello, $name';  // 输出：Hello, $name（变量不会被解析）
$str2 = 'It\'s a string';  // 输出：It's a string
```

#### 双引号

双引号字符串支持变量插值和转义序列：

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$str1 = "Hello, {$name}";  // 输出：Hello, Alice
$str2 = "Line 1\nLine 2";  // 支持换行符
$str3 = "Price: \$10";     // 输出：Price: $10
```

#### Heredoc

Heredoc 语法用于定义多行字符串，支持变量插值：

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$str = <<<EOT
Hello, {$name}!
This is a multi-line string.
You can use variables here.
EOT;

echo $str;
```

#### Nowdoc

Nowdoc 语法类似于 Heredoc，但不支持变量插值：

```php
<?php
declare(strict_types=1);

$str = <<<'EOT'
This is a nowdoc string.
Variables like {$name} won't be parsed.
EOT;

echo $str;
```

### 类型检测

#### `is_string()`

**语法**：`is_string(mixed $value): bool`

**参数**：
- `$value`：要检测的值

**返回值**：如果 `$value` 是字符串类型，返回 `true`；否则返回 `false`。

```php
<?php
declare(strict_types=1);

var_dump(is_string("hello"));  // bool(true)
var_dump(is_string(42));       // bool(false)
var_dump(is_string(null));     // bool(false)
```

### 字符串长度

#### `strlen()` - 字节长度

**语法**：`strlen(string $string): int`

**参数**：
- `$string`：要计算长度的字符串

**返回值**：返回字符串的字节数（不是字符数）。

```php
<?php
declare(strict_types=1);

echo strlen("Hello") . "\n";        // 5
echo strlen("你好") . "\n";          // 6（UTF-8 编码，每个中文字符 3 字节）
```

#### `mb_strlen()` - 字符长度

**语法**：`mb_strlen(string $string, ?string $encoding = null): int`

**参数**：
- `$string`：要计算长度的字符串
- `$encoding`：字符编码，默认为内部编码

**返回值**：返回字符串的字符数。

```php
<?php
declare(strict_types=1);

echo mb_strlen("Hello", 'UTF-8') . "\n";   // 5
echo mb_strlen("你好", 'UTF-8') . "\n";     // 2（正确计算中文字符数）
```

### 字符串截取

#### `substr()` - 截取子串（按字节）

**语法**：`substr(string $string, int $offset, ?int $length = null): string`

**参数**：
- `$string`：原字符串
- `$offset`：起始位置（负数表示从末尾开始）
- `$length`：可选，要截取的长度（负数表示从末尾开始）

**返回值**：返回截取的子串。

```php
<?php
declare(strict_types=1);

$str = "Hello, World!";
echo substr($str, 0, 5) . "\n";    // Hello
echo substr($str, 7) . "\n";       // World!
echo substr($str, -6) . "\n";      // World!
echo substr($str, 0, -1) . "\n";   // Hello, World
```

#### `mb_substr()` - 截取子串（按字符）

**语法**：`mb_substr(string $string, int $start, ?int $length = null, ?string $encoding = null): string`

**参数**：
- `$string`：原字符串
- `$start`：起始位置（负数表示从末尾开始）
- `$length`：可选，要截取的长度（负数表示从末尾开始），默认为 `null`（截取到末尾）
- `$encoding`：可选，字符编码，默认为内部编码

**返回值**：返回截取后的子串。

```php
<?php
declare(strict_types=1);

$str = "你好，世界！";
echo mb_substr($str, 0, 2, 'UTF-8') . "\n";  // 你好
echo mb_substr($str, 3, 2, 'UTF-8') . "\n";  // 世界
```

### 字符串查找

#### `strpos()` - 查找子串位置

**语法**：`strpos(string $haystack, string $needle, int $offset = 0): int|false`

**参数**：
- `$haystack`：要搜索的字符串
- `$needle`：要查找的子串
- `$offset`：可选，搜索起始位置

**返回值**：找到返回位置（从 0 开始），未找到返回 `false`。

```php
<?php
declare(strict_types=1);

$str = "Hello, World!";
$pos = strpos($str, "World");
if ($pos !== false) {
    echo "Found at position: {$pos}\n";  // Found at position: 7
}
```

#### `mb_strpos()` - 查找子串位置（多字节）

**语法**：`mb_strpos(string $haystack, string $needle, int $offset = 0, ?string $encoding = null): int|false`

### 字符串分割与拼接

#### `explode()` - 分割字符串

**语法**：`explode(string $separator, string $string, int $limit = PHP_INT_MAX): array`

**参数**：
- `$separator`：分隔符
- `$string`：要分割的字符串
- `$limit`：可选，限制返回数组的元素数量

**返回值**：返回分割后的数组。

```php
<?php
declare(strict_types=1);

$str = "apple,banana,orange";
$fruits = explode(",", $str);
print_r($fruits);
// 输出：
// Array
// (
//     [0] => apple
//     [1] => banana
//     [2] => orange
// )

// 限制分割数量
$parts = explode(",", $str, 2);
print_r($parts);
// 输出：
// Array
// (
//     [0] => apple
//     [1] => banana,orange
// )
```

#### `implode()` - 拼接数组

**语法**：`implode(string $separator, array $array): string`

**参数**：
- `$separator`：分隔符
- `$array`：要拼接的数组

**返回值**：返回拼接后的字符串。

```php
<?php
declare(strict_types=1);

$fruits = ["apple", "banana", "orange"];
$str = implode(", ", $fruits);
echo $str . "\n";  // apple, banana, orange
```

### 完整示例

```php
<?php
declare(strict_types=1);

// 字符串定义
$single = 'Single quoted string';
$double = "Double quoted string with {$variable}";
$heredoc = <<<EOT
Heredoc string
Supports variables: {$variable}
EOT;
$nowdoc = <<<'EOT'
Nowdoc string
No variable parsing
EOT;

// 字符串操作
function processString(string $str): array
{
    return [
        'original' => $str,
        'length_bytes' => strlen($str),
        'length_chars' => mb_strlen($str, 'UTF-8'),
        'uppercase' => mb_strtoupper($str, 'UTF-8'),
        'lowercase' => mb_strtolower($str, 'UTF-8'),
        'words' => explode(' ', $str)
    ];
}

$result = processString("Hello, 世界!");
print_r($result);

// UTF-8 字符串处理
function truncateUtf8(string $str, int $length, string $suffix = '...'): string
{
    if (mb_strlen($str, 'UTF-8') <= $length) {
        return $str;
    }
    
    return mb_substr($str, 0, $length, 'UTF-8') . $suffix;
}

$text = "这是一段很长的中文文本，需要截断显示";
echo truncateUtf8($text, 10) . "\n";  // 这是一段很长的中文文本，需要...
```

## 注意事项

1. **布尔值转换**：注意 `"0"` 和空字符串在布尔上下文中被视为 `false`。

2. **整数溢出**：大整数可能自动转换为浮点数，注意精度损失。

3. **浮点数精度**：不要直接比较浮点数是否相等，应使用误差范围比较。

4. **字符串编码**：处理多字节字符（如中文）时，始终使用 `mb_*` 系列函数。

5. **类型检测**：使用 `is_*()` 函数进行类型检测，而不是 `gettype()`。

## 练习

1. 编写一个函数 `isTruthy(mixed $value): bool`，判断值在布尔上下文中是否为真。

2. 实现一个函数 `parseNumber(string $str): int|float|null`，能够解析包含数字的字符串（如 "42", "3.14", "10.5abc"），返回相应的数字类型或 `null`。

3. 创建一个函数 `formatCurrency(float $amount, string $currency = 'USD'): string`，使用 `NumberFormatter` 格式化货币（需要 `intl` 扩展）。

4. 编写一个函数 `truncateString(string $str, int $maxLength, string $suffix = '...'): string`，正确处理 UTF-8 多字节字符的截断。

5. 实现一个函数 `parseCsvLine(string $line, string $delimiter = ','): array`，解析 CSV 格式的字符串行，正确处理引号和转义。
