# 2.5.4 比较运算符与函数

## 概述

比较是编程中的基本操作，PHP 提供了多种比较运算符和函数。本节详细介绍宽松比较（`==`）与严格比较（`===`）的区别、常见陷阱、比较函数（`strcmp()`、`strcasecmp()`、`bccomp()`、`version_compare()`）、太空船运算符（`<=>`）及最佳实践。

理解比较运算符和函数的区别对于编写可靠的 PHP 代码至关重要。严格比较可以避免隐式转换的陷阱，比较函数提供了更专业的比较能力。

## 特性

- **宽松比较（==）**：会进行类型转换，可能导致意外结果
- **严格比较（===）**：不进行类型转换，更安全、更可预测
- **太空船运算符（<=>）**：返回 -1、0、1，用于排序和比较
- **比较函数**：提供专业的比较能力（字符串、数值、版本等）

## 语法/定义

### 宽松比较运算符（==）

**语法**：`$a == $b`

**特点**：
- 会进行类型转换
- 比较值是否相等，不考虑类型
- 可能导致意外的结果
- 不推荐使用

**返回值**：如果值相等返回 `true`，否则返回 `false`

### 严格比较运算符（===）

**语法**：`$a === $b`

**特点**：
- 不进行类型转换
- 比较类型和值是否都相等
- 更安全、更可预测
- 推荐使用

**返回值**：如果类型和值都相等返回 `true`，否则返回 `false`

### 不等比较运算符

**宽松不等（!= 或 <>）**：`$a != $b` 或 `$a <> $b`
- 会进行类型转换
- 不推荐使用

**严格不等（!==）**：`$a !== $b`
- 不进行类型转换
- 推荐使用

### 太空船运算符（<=>）（PHP 7.0+）

**语法**：`$a <=> $b`

**返回值**：
- `-1`：如果 `$a < $b`
- `0`：如果 `$a == $b`
- `1`：如果 `$a > $b`

**特点**：
- 用于排序和比较
- 简化比较代码
- 支持链式比较

### strcmp() - 字符串比较（区分大小写）

**语法**：`strcmp(string $string1, string $string2): int`

**参数**：
- `$string1`：第一个字符串
- `$string2`：第二个字符串

**返回值**：
- `< 0`：如果 `$string1` 小于 `$string2`
- `0`：如果 `$string1` 等于 `$string2`
- `> 0`：如果 `$string1` 大于 `$string2`

**特点**：
- 区分大小写
- 基于 ASCII 值比较
- 适合排序

### strcasecmp() - 字符串比较（不区分大小写）

**语法**：`strcasecmp(string $string1, string $string2): int`

**参数**：
- `$string1`：第一个字符串
- `$string2`：第二个字符串

**返回值**：
- `< 0`：如果 `$string1` 小于 `$string2`（不区分大小写）
- `0`：如果 `$string1` 等于 `$string2`（不区分大小写）
- `> 0`：如果 `$string1` 大于 `$string2`（不区分大小写）

**特点**：
- 不区分大小写
- 适合用户输入比较

### bccomp() - 任意精度数字比较

**语法**：`bccomp(string $num1, string $num2, ?int $scale = null): int`

**参数**：
- `$num1`：第一个数字（字符串形式）
- `$num2`：第二个数字（字符串形式）
- `$scale`：可选，小数位数，默认为 `bcscale()` 设置的值

**返回值**：
- `-1`：如果 `$num1` 小于 `$num2`
- `0`：如果 `$num1` 等于 `$num2`
- `1`：如果 `$num1` 大于 `$num2`

**特点**：
- 任意精度比较
- 适合大数和精确比较
- 需要 BCMath 扩展

### version_compare() - 版本号比较

**语法**：`version_compare(string $version1, string $version2, ?string $operator = null): int|bool`

**参数**：
- `$version1`：第一个版本号
- `$version2`：第二个版本号
- `$operator`：可选，比较运算符（`<`、`<=`、`>`、`>=`、`==`、`!=`、`<>`）

**返回值**：
- 如果提供 `$operator`：返回布尔值
- 如果未提供 `$operator`：返回 -1、0 或 1

**特点**：
- 专门用于版本号比较
- 支持语义化版本号
- 适合依赖管理

## 基本用法

### 示例 1：宽松比较 vs 严格比较

```php
<?php
declare(strict_types=1);

// 宽松比较（会进行类型转换）
$comparisons = [
    ['"123"', 123],
    ['"0"', false],
    ['""', false],
    ['"0"', 0],
    ['[]', false],
    ['null', false],
];

echo "=== 宽松比较（==）===\n";
foreach ($comparisons as [$desc1, $value2]) {
    $value1 = eval("return {$desc1};");
    $result = $value1 == $value2;
    echo "{$desc1} == ";
    var_dump($value2);
    echo "Result: " . ($result ? 'true' : 'false') . "\n";
    echo "---\n";
}

echo "\n=== 严格比较（===）===\n";
foreach ($comparisons as [$desc1, $value2]) {
    $value1 = eval("return {$desc1};");
    $result = $value1 === $value2;
    echo "{$desc1} === ";
    var_dump($value2);
    echo "Result: " . ($result ? 'true' : 'false') . "\n";
    echo "---\n";
}
```

**执行**：

```bash
php comparison-example.php
```

**输出**：

```
=== 宽松比较（==）===
"123" == int(123)
Result: true
---
"0" == bool(false)
Result: true
---
"" == bool(false)
Result: true
---
"0" == int(0)
Result: true
---
Array() == bool(false)
Result: true
---
NULL == bool(false)
Result: true
---

=== 严格比较（===）===
"123" === int(123)
Result: false
---
"0" === bool(false)
Result: false
---
"" === bool(false)
Result: false
---
"0" === int(0)
Result: false
---
Array() === bool(false)
Result: false
---
NULL === bool(false)
Result: false
---
```

### 示例 2：太空船运算符

```php
<?php
declare(strict_types=1);

// 基本使用
echo "1 <=> 2: " . (1 <=> 2) . "\n";  // -1
echo "2 <=> 2: " . (2 <=> 2) . "\n";  // 0
echo "3 <=> 2: " . (3 <=> 2) . "\n";  // 1

// 字符串比较
echo "\"a\" <=> \"b\": " . ("a" <=> "b") . "\n";  // -1
echo "\"b\" <=> \"a\": " . ("b" <=> "a") . "\n";  // 1

// 用于排序
$numbers = [3, 1, 4, 1, 5, 9, 2, 6];
usort($numbers, fn($a, $b) => $a <=> $b);
echo "Sorted: " . implode(', ', $numbers) . "\n";  // Sorted: 1, 1, 2, 3, 4, 5, 6, 9

// 多维数组排序
$users = [
    ['name' => 'Alice', 'age' => 25],
    ['name' => 'Bob', 'age' => 30],
    ['name' => 'Charlie', 'age' => 25],
];

usort($users, function($a, $b) {
    return $a['age'] <=> $b['age'] ?: $a['name'] <=> $b['name'];
});

foreach ($users as $user) {
    echo "{$user['name']}, {$user['age']}\n";
}
```

### 示例 3：字符串比较函数

```php
<?php
declare(strict_types=1);

// strcmp() - 区分大小写
$result1 = strcmp("Hello", "hello");
echo "strcmp('Hello', 'hello'): {$result1}\n";  // 非零（区分大小写）

$result2 = strcmp("Hello", "Hello");
echo "strcmp('Hello', 'Hello'): {$result2}\n";  // 0（相等）

// strcasecmp() - 不区分大小写
$result3 = strcasecmp("Hello", "hello");
echo "strcasecmp('Hello', 'hello'): {$result3}\n";  // 0（不区分大小写）

// 用于排序
$names = ["Charlie", "alice", "Bob"];
usort($names, 'strcmp');
echo "Sorted (case-sensitive): " . implode(', ', $names) . "\n";

usort($names, 'strcasecmp');
echo "Sorted (case-insensitive): " . implode(', ', $names) . "\n";
```

### 示例 4：数值比较函数

```php
<?php
declare(strict_types=1);

// bccomp() - 任意精度比较
if (function_exists('bccomp')) {
    $result1 = bccomp("1.23", "1.24", 2);
    echo "bccomp('1.23', '1.24', 2): {$result1}\n";  // -1
    
    $result2 = bccomp("1.234", "1.234", 3);
    echo "bccomp('1.234', '1.234', 3): {$result2}\n";  // 0
    
    // 大数比较
    $result3 = bccomp("999999999999999999", "1000000000000000000");
    echo "bccomp('999999999999999999', '1000000000000000000'): {$result3}\n";  // -1
}

// version_compare() - 版本号比较
$result1 = version_compare("1.2.3", "1.2.4");
echo "version_compare('1.2.3', '1.2.4'): {$result1}\n";  // -1

$result2 = version_compare("2.0.0", "1.9.9", ">");
echo "version_compare('2.0.0', '1.9.9', '>'): " . ($result2 ? 'true' : 'false') . "\n";  // true

$result3 = version_compare("1.2.3", "1.2.3");
echo "version_compare('1.2.3', '1.2.3'): {$result3}\n";  // 0
```

## 完整代码示例

### 示例 1：比较工具函数

```php
<?php
declare(strict_types=1);

class ComparisonHelper
{
    /**
     * 安全比较（使用严格比较）
     */
    public static function safeEquals(mixed $a, mixed $b): bool
    {
        return $a === $b;
    }
    
    /**
     * 安全不等（使用严格比较）
     */
    public static function safeNotEquals(mixed $a, mixed $b): bool
    {
        return $a !== $b;
    }
    
    /**
     * 字符串比较（不区分大小写）
     */
    public static function stringEquals(string $a, string $b): bool
    {
        return strcasecmp($a, $b) === 0;
    }
    
    /**
     * 数值比较（处理浮点数精度）
     */
    public static function numericEquals(float $a, float $b, float $epsilon = 0.00001): bool
    {
        return abs($a - $b) < $epsilon;
    }
    
    /**
     * 版本比较
     */
    public static function versionGreaterThan(string $version1, string $version2): bool
    {
        return version_compare($version1, $version2, '>');
    }
}

// 使用
var_dump(ComparisonHelper::safeEquals("123", 123));  // bool(false)
var_dump(ComparisonHelper::stringEquals("Hello", "hello"));  // bool(true)
var_dump(ComparisonHelper::numericEquals(0.1 + 0.2, 0.3));  // bool(true)
var_dump(ComparisonHelper::versionGreaterThan("2.0.0", "1.9.9"));  // bool(true)
```

### 示例 2：排序应用

```php
<?php
declare(strict_types=1);

// 使用太空船运算符排序
$products = [
    ['name' => 'Product A', 'price' => 99.99, 'stock' => 10],
    ['name' => 'Product B', 'price' => 49.99, 'stock' => 5],
    ['name' => 'Product C', 'price' => 149.99, 'stock' => 20],
];

// 按价格排序
usort($products, fn($a, $b) => $a['price'] <=> $b['price']);
echo "Sorted by price:\n";
foreach ($products as $product) {
    echo "{$product['name']}: \${$product['price']}\n";
}

// 多条件排序：先按库存，再按价格
usort($products, function($a, $b) {
    return $a['stock'] <=> $b['stock'] ?: $a['price'] <=> $b['price'];
});
echo "\nSorted by stock, then price:\n";
foreach ($products as $product) {
    echo "{$product['name']}: Stock={$product['stock']}, Price=\${$product['price']}\n";
}
```

### 示例 3：版本检查

```php
<?php
declare(strict_types=1);

class VersionChecker
{
    public static function checkPhpVersion(string $requiredVersion): bool
    {
        $currentVersion = PHP_VERSION;
        return version_compare($currentVersion, $requiredVersion, '>=');
    }
    
    public static function checkExtensionVersion(string $extension, string $requiredVersion): bool
    {
        if (!extension_loaded($extension)) {
            return false;
        }
        
        $currentVersion = phpversion($extension);
        if ($currentVersion === false) {
            return false;
        }
        
        return version_compare($currentVersion, $requiredVersion, '>=');
    }
}

// 使用
if (VersionChecker::checkPhpVersion('8.0.0')) {
    echo "PHP version is sufficient\n";
} else {
    echo "PHP version is too old\n";
}

if (VersionChecker::checkExtensionVersion('json', '1.0.0')) {
    echo "JSON extension version is sufficient\n";
} else {
    echo "JSON extension version is too old or not loaded\n";
}
```

## 使用场景

### 严格比较（===）

- **条件判断**：所有条件判断都应该使用严格比较
- **类型检查**：检查变量类型和值
- **安全比较**：避免隐式转换陷阱

### 太空船运算符（<=>）

- **排序**：简化排序代码
- **比较链**：支持链式比较
- **多条件排序**：实现多条件排序

### 字符串比较函数

- **用户输入**：比较用户输入的字符串
- **排序**：字符串数组排序
- **搜索**：字符串搜索和匹配

### 数值比较函数

- **精确计算**：使用 `bccomp()` 进行精确比较
- **大数比较**：比较超出整数范围的大数
- **浮点数比较**：处理浮点数精度问题

### 版本比较函数

- **依赖管理**：检查 PHP 版本和扩展版本
- **功能检测**：根据版本启用或禁用功能
- **兼容性检查**：检查系统兼容性

## 注意事项

### 严格比较

- **始终使用**：始终使用 `===` 和 `!==` 进行比较
- **避免宽松比较**：避免使用 `==` 和 `!=`
- **类型安全**：严格比较提供类型安全

### 浮点数比较

- **精度问题**：浮点数不能直接使用 `===` 比较
- **使用误差范围**：使用误差范围比较浮点数
- **使用 bccomp()**：需要精确比较时使用 `bccomp()`

### 字符串比较

- **区分大小写**：`strcmp()` 区分大小写
- **不区分大小写**：`strcasecmp()` 不区分大小写
- **编码问题**：注意字符串编码的一致性

### 性能考虑

- **严格比较性能更好**：严格比较比宽松比较性能更好
- **函数调用开销**：比较函数有函数调用开销
- **选择合适的方法**：根据场景选择最合适的方法

## 常见问题

### 问题 1：宽松比较陷阱

**症状**：比较结果不符合预期

**原因**：宽松比较会进行类型转换

**错误示例**：

```php
<?php
declare(strict_types=1);

$value = "0";
if ($value == false) {  // true - 意外结果
    echo "Value is false\n";
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$value = "0";
if ($value === false) {  // false - 正确结果
    echo "Value is false\n";
} else {
    echo "Value is not false\n";
}

// 或检查具体值
if ($value === "0") {
    echo "Value is '0'\n";
}
```

### 问题 2：浮点数比较

**症状**：浮点数比较结果不符合预期

**原因**：浮点数精度问题

**错误示例**：

```php
<?php
declare(strict_types=1);

$result = 0.1 + 0.2;
var_dump($result === 0.3);  // bool(false) - 精度问题
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$result = 0.1 + 0.2;

// 方法1：使用误差范围
$epsilon = 0.00001;
var_dump(abs($result - 0.3) < $epsilon);  // bool(true)

// 方法2：使用 bccomp()
if (function_exists('bccomp')) {
    var_dump(bccomp((string) $result, "0.3", 5) === 0);  // bool(true)
}
```

### 问题 3：字符串比较编码问题

**症状**：中文字符串比较结果不符合预期

**原因**：字符串编码不一致

**解决方法**：

```php
<?php
declare(strict_types=1);

// 确保编码一致
$str1 = mb_convert_encoding("中文", "UTF-8");
$str2 = mb_convert_encoding("中文", "UTF-8");

// 使用 mb_strcmp() 进行多字节字符串比较
if (function_exists('mb_strcmp')) {
    $result = mb_strcmp($str1, $str2);
    var_dump($result === 0);  // bool(true)
} else {
    var_dump($str1 === $str2);  // bool(true)
}
```

## 最佳实践

### 比较运算符

- **始终使用严格比较**：使用 `===` 和 `!==`
- **避免宽松比较**：避免使用 `==` 和 `!=`
- **使用太空船运算符**：简化排序和比较代码

### 比较函数

- **选择合适的函数**：根据场景选择合适的比较函数
- **字符串比较**：使用 `strcmp()` 或 `strcasecmp()`
- **数值比较**：使用 `bccomp()` 进行精确比较
- **版本比较**：使用 `version_compare()` 比较版本号

### 浮点数处理

- **使用误差范围**：比较浮点数时使用误差范围
- **使用 bccomp()**：需要精确比较时使用 `bccomp()`
- **避免直接比较**：不要直接使用 `===` 比较浮点数

### 代码质量

- **类型安全**：使用严格比较提高类型安全
- **代码可读性**：使用有意义的比较方式
- **文档说明**：为复杂的比较逻辑添加注释

## 对比分析

### 宽松比较 vs 严格比较

| 特性 | 宽松比较（==） | 严格比较（===） |
|:-----|:---------------|:----------------|
| 类型转换 | 是 | 否 |
| 安全性 | 低 | 高 |
| 可预测性 | 低 | 高 |
| 性能 | 略差 | 略好 |
| 推荐度 | 不推荐 | 推荐 |

**选择建议**：
- **始终使用严格比较**：使用 `===` 和 `!==`
- **避免宽松比较**：除非确实需要类型转换
- **理解区别**：理解两种比较方式的区别

### 比较函数对比

| 函数 | 用途 | 特点 | 推荐场景 |
|:-----|:-----|:-----|:---------|
| `strcmp()` | 字符串比较 | 区分大小写 | 排序、精确匹配 |
| `strcasecmp()` | 字符串比较 | 不区分大小写 | 用户输入比较 |
| `bccomp()` | 数值比较 | 任意精度 | 精确计算、大数 |
| `version_compare()` | 版本比较 | 语义化版本 | 依赖管理、兼容性 |

**选择建议**：
- **字符串比较**：使用 `strcmp()` 或 `strcasecmp()`
- **精确数值比较**：使用 `bccomp()`
- **版本比较**：使用 `version_compare()`

## 相关章节

- **2.5.1 隐式转换**：了解隐式转换规则
- **2.5.2 显式转换**：了解显式类型转换
- **2.6 表达式与运算符**：详细了解运算符
- **2.8 数组完整指南**：了解数组排序

## 练习任务

1. **比较运算符练习**：
   - 对比宽松比较和严格比较的结果
   - 理解比较运算中的类型转换
   - 使用严格比较改进代码
   - 测试各种类型的比较结果

2. **太空船运算符练习**：
   - 使用太空船运算符进行排序
   - 实现多条件排序
   - 理解太空船运算符的返回值
   - 测试不同场景下的排序

3. **字符串比较练习**：
   - 使用 `strcmp()` 和 `strcasecmp()` 比较字符串
   - 实现字符串数组排序
   - 理解区分大小写的区别
   - 测试各种字符串比较场景

4. **数值比较练习**：
   - 使用 `bccomp()` 进行精确数值比较
   - 处理浮点数精度问题
   - 比较大数
   - 测试各种数值比较场景

5. **综合练习**：
   - 创建一个比较工具类
   - 实现多种比较方法
   - 处理各种类型的比较
   - 进行代码审查，确保比较正确
