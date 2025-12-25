# 2.5.1 隐式转换（Type Juggling）

## 概述

PHP 是动态类型语言，在算术运算、字符串连接、比较等操作中，会根据上下文自动进行类型转换，这称为**隐式转换**（Type Juggling）或**类型杂耍**。理解隐式转换规则对于编写正确的 PHP 代码至关重要。

## 字符串与数字的转换

### 字符串转数字

当字符串参与算术运算时，PHP 会尝试将字符串转换为数字：

```php
<?php
declare(strict_types=1);

echo "5" + 3 . "\n";      // 8（字符串 "5" 转为整数 5）
echo "5.5" + 2 . "\n";    // 7.5（字符串 "5.5" 转为浮点数 5.5）
echo "5a" + 3 . "\n";     // 8（"5a" 解析为 5，忽略非数字字符）
echo "a5" + 3 . "\n";     // 3（"a5" 无法解析，转为 0）
echo "abc" + 3 . "\n";    // 3（"abc" 无法解析，转为 0）
```

### 数字转字符串

当数字参与字符串连接时，数字会被转换为字符串：

```php
<?php
declare(strict_types=1);

echo "Price: " . 99.99 . "\n";  // Price: 99.99
echo "Count: " . 42 . "\n";     // Count: 42
```

## 布尔上下文转换

在布尔上下文（如 `if`、`while`、逻辑运算符）中，以下值会被视为 `false`（falsy 值）：

- `false` 本身
- `0`（整数零）
- `0.0`（浮点数零）
- `"0"`（字符串 "0"）
- `""`（空字符串）
- `[]`（空数组）
- `null`

所有其他值都被视为 `true`（truthy 值）。

### 示例

```php
<?php
declare(strict_types=1);

// Falsy 值
if (0) echo "This won't print\n";
if (0.0) echo "This won't print\n";
if ("0") echo "This won't print\n";
if ("") echo "This won't print\n";
if ([]) echo "This won't print\n";
if (null) echo "This won't print\n";

// Truthy 值
if (1) echo "This will print\n";
if (-1) echo "This will print\n";
if ("1") echo "This will print\n";
if ("false") echo "This will print\n";  // 注意：非空字符串为 true
if ([1, 2, 3]) echo "This will print\n";
```

## 比较运算中的隐式转换

### 宽松比较（==）

宽松比较会先进行类型转换，再比较值：

```php
<?php
declare(strict_types=1);

var_dump("5" == 5);        // bool(true) - 字符串转为数字
var_dump("5.0" == 5);      // bool(true) - 浮点数转为整数
var_dump("0" == false);    // bool(true) - 字符串 "0" 转为 false
var_dump("" == false);     // bool(true) - 空字符串转为 false
var_dump([] == false);     // bool(true) - 空数组转为 false
var_dump(null == false);   // bool(true) - null 转为 false
```

### 常见陷阱

```php
<?php
declare(strict_types=1);

// 陷阱 1：科学计数法
var_dump("0e123" == "0");      // bool(true) - "0e123" 被解析为 0
var_dump("0e123" == 0);        // bool(true)

// 陷阱 2：非数字字符串转数字
var_dump("foo" == 0);          // bool(true) - "foo" 转为 0
var_dump("10a" == 10);         // bool(true) - "10a" 转为 10

// 陷阱 3：数组比较
var_dump([] == false);         // bool(true)
var_dump([0] == false);        // bool(false) - 非空数组

// 陷阱 4：null 比较
var_dump(null == false);       // bool(true)
var_dump(null == 0);           // bool(false)
var_dump(null == "");          // bool(true)
```

## 算术运算中的隐式转换

### 混合类型运算

```php
<?php
declare(strict_types=1);

// 整数和浮点数
echo 5 + 3.14 . "\n";          // 8.14（整数转为浮点数）

// 字符串和数字
echo "10" + 5 . "\n";          // 15（字符串转为整数）
echo "10.5" + 5 . "\n";        // 15.5（字符串转为浮点数）

// 布尔值参与运算
echo true + 5 . "\n";           // 6（true 转为 1）
echo false + 5 . "\n";         // 5（false 转为 0）
```

### 取模运算

```php
<?php
declare(strict_types=1);

echo "10" % 3 . "\n";          // 1（字符串转为整数）
echo "10.5" % 3 . "\n";        // 1（浮点数转为整数，小数部分被忽略）
echo 10 % "3" . "\n";          // 1（字符串转为整数）
```

## 字符串连接中的隐式转换

使用 `.` 运算符连接字符串时，非字符串值会被转换为字符串：

```php
<?php
declare(strict_types=1);

echo "Number: " . 42 . "\n";           // Number: 42
echo "Price: $" . 99.99 . "\n";        // Price: $99.99
echo "Active: " . true . "\n";         // Active: 1（布尔值转为字符串）
echo "Count: " . [1, 2, 3] . "\n";     // Count: Array（数组转为 "Array"）
```

## 数组键的隐式转换

数组键如果是字符串形式的数字，会被转换为整数：

```php
<?php
declare(strict_types=1);

$arr = [];
$arr["0"] = "zero";
$arr["1"] = "one";
$arr["2"] = "two";

print_r($arr);
// 输出：
// Array
// (
//     [0] => zero
//     [1] => one
//     [2] => two
// )

// 浮点数键会被截断为整数
$arr2 = [];
$arr2["3.14"] = "pi";
$arr2["5.99"] = "almost six";

print_r($arr2);
// 输出：
// Array
// (
//     [3] => pi
//     [5] => almost six
// )
```

## 完整示例

```php
<?php
declare(strict_types=1);

function demonstrateTypeJuggling(): void
{
    echo "=== 字符串转数字 ===\n";
    echo "\"5\" + 3 = " . ("5" + 3) . "\n";
    echo "\"5.5\" + 2 = " . ("5.5" + 2) . "\n";
    echo "\"5a\" + 3 = " . ("5a" + 3) . "\n";
    echo "\"abc\" + 3 = " . ("abc" + 3) . "\n";
    
    echo "\n=== 布尔上下文 ===\n";
    $values = [0, 0.0, "0", "", [], null, 1, "1", "false", [1, 2]];
    foreach ($values as $value) {
        $result = $value ? 'true' : 'false';
        echo var_export($value, true) . " => {$result}\n";
    }
    
    echo "\n=== 宽松比较陷阱 ===\n";
    $comparisons = [
        ['"0e123"', '"0"', "0e123" == "0"],
        ['"0"', 'false', "0" == false],
        ['"foo"', '0', "foo" == 0],
        ['[]', 'false', [] == false],
    ];
    
    foreach ($comparisons as [$a, $b, $result]) {
        echo "{$a} == {$b}: " . ($result ? 'true' : 'false') . "\n";
    }
}

demonstrateTypeJuggling();
```

## 注意事项

1. **避免隐式转换**：除非十分确定规则，否则尽量使用显式转换和严格比较。

2. **使用严格比较**：始终使用 `===` 和 `!==` 进行比较，避免隐式转换带来的问题。

3. **类型提示**：在函数参数中使用类型提示，减少隐式转换的需要。

4. **严格模式**：使用 `declare(strict_types=1);` 启用严格模式，减少隐式转换。

5. **文档说明**：如果必须依赖隐式转换，添加注释说明原因。

## 练习

1. 编写一个函数，演示各种隐式转换场景，并输出转换结果。

2. 分析以下表达式的值并解释原因：
   - `"10" + "5"`
   - `"10" . "5"`
   - `"10" == 10`
   - `"10" === 10`
   - `0 == "0e123"`
   - `[] == false`

3. 创建一个函数 `safeAdd(mixed $a, mixed $b): int|float`，安全地进行加法运算，避免隐式转换陷阱。

4. 编写一个函数，检查值在布尔上下文中是否为真，并返回详细的转换信息。

5. 实现一个类型转换演示工具，展示各种隐式转换规则。
