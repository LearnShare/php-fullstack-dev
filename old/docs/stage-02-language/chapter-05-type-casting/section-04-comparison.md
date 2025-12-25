# 2.5.4 比较运算符与函数

## 概述

PHP 提供了多种比较运算符和函数，理解它们的差异对于编写正确的代码至关重要。宽松比较（`==`）和严格比较（`===`）是最常用的比较方式。

## 比较运算符

### 宽松比较（==）

宽松比较会先进行类型转换，再比较值。

| 操作符 | 行为                     | 示例                             |
| :----- | :----------------------- | :------------------------------- |
| `==`   | 比较前先进行类型转换     | `"0" == 0` 为 `true`            |
| `!=`   | 与 `==` 相反             | `"0" != 0` 为 `false`            |
| `<>`   | 与 `!=` 相同             | `"0" <> 0` 为 `false`            |

### 严格比较（===）

严格比较不进行类型转换，比较值和类型。

| 操作符 | 行为                     | 示例                             |
| :----- | :----------------------- | :------------------------------- |
| `===`  | 比较值与类型（不转换）  | `"0" === 0` 为 `false`          |
| `!==`  | 与 `===` 相反            | `"0" !== 0` 为 `true`           |

### 示例对比

```php
<?php
declare(strict_types=1);

// 宽松比较
var_dump("5" == 5);        // bool(true)
var_dump("0" == false);    // bool(true)
var_dump([] == false);     // bool(true)

// 严格比较
var_dump("5" === 5);       // bool(false)
var_dump("0" === false);   // bool(false)
var_dump([] === false);    // bool(false)
```

## 常见陷阱

### 陷阱 1：科学计数法

```php
<?php
declare(strict_types=1);

var_dump("0e123" == "0");      // bool(true) - "0e123" 被解析为 0
var_dump("0e123" == 0);        // bool(true)
var_dump("0e123" === "0");     // bool(false) - 严格比较
```

### 陷阱 2：非数字字符串

```php
<?php
declare(strict_types=1);

var_dump("foo" == 0);          // bool(true) - "foo" 转为 0
var_dump("10a" == 10);         // bool(true) - "10a" 转为 10
var_dump("foo" === 0);         // bool(false) - 严格比较
```

### 陷阱 3：数组比较

```php
<?php
declare(strict_types=1);

var_dump([] == false);         // bool(true)
var_dump([0] == false);        // bool(false) - 非空数组
var_dump([] === false);        // bool(false) - 严格比较
```

## 比较函数

### strcmp() - 字符串比较

**语法**：`strcmp(string $string1, string $string2): int`

**参数**：
- `$string1`：要比较的第一个字符串
- `$string2`：要比较的第二个字符串

**返回值**：
- `< 0`：`$string1` 小于 `$string2`
- `= 0`：`$string1` 等于 `$string2`
- `> 0`：`$string1` 大于 `$string2`

**比较逻辑**：
`strcmp()` 使用二进制安全的字节比较（binary-safe string comparison），比较过程如下：

1. **逐字节比较**：从字符串的第一个字符开始，按字节逐个比较
2. **ASCII 值比较**：比较每个字符的 ASCII 值（或字节值）
3. **首次差异决定结果**：遇到第一个不同的字符时，返回两个字符 ASCII 值的差（`$string1[i] - $string2[i]`）
4. **长度差异处理**：如果所有字符都相同，但长度不同，返回长度的差（`strlen($string1) - strlen($string2)`）

**比较示例**：
- `strcmp("apple", "banana")`：比较 'a' 和 'b'，'a'(97) < 'b'(98)，返回负数
- `strcmp("apple", "apricot")`：前两个字符 'a' 和 'a' 相同，比较第三个字符 'p' 和 'r'，'p'(112) < 'r'(114)，返回负数
- `strcmp("apple", "app")`：前三个字符相同，但长度不同，返回正数（"apple" 更长）

**注意事项**：
- `strcmp()` 区分大小写（'A' 的 ASCII 值为 65，'a' 为 97）
- 对于多字节字符（如中文），`strcmp()` 按字节比较，可能不符合预期，应使用 `mb_strcmp()` 或 `strcoll()`

```php
<?php
declare(strict_types=1);

echo strcmp("apple", "banana") . "\n";  // 负数
echo strcmp("banana", "apple") . "\n";  // 正数
echo strcmp("apple", "apple") . "\n";   // 0
```

### strcasecmp() - 不区分大小写比较

**语法**：`strcasecmp(string $string1, string $string2): int`

**参数**：
- `$string1`：要比较的第一个字符串
- `$string2`：要比较的第二个字符串

**返回值**：
- `< 0`：`$string1` 小于 `$string2`（不区分大小写）
- `= 0`：`$string1` 等于 `$string2`（不区分大小写）
- `> 0`：`$string1` 大于 `$string2`（不区分大小写）

```php
<?php
declare(strict_types=1);

echo strcasecmp("Apple", "apple") . "\n";  // 0（相等）
```

### bccomp() - 高精度比较

**语法**：`bccomp(string $num1, string $num2, ?int $scale = null): int`

**参数**：
- `$num1`：要比较的第一个数字（字符串格式）
- `$num2`：要比较的第二个数字（字符串格式）
- `$scale`：可选，小数位数精度，默认为 `null`（使用全局精度设置）

**返回值**：
- `< 0`：`$num1` 小于 `$num2`
- `= 0`：`$num1` 等于 `$num2`
- `> 0`：`$num1` 大于 `$num2`

用于比较大数或需要高精度的数值。

```php
<?php
declare(strict_types=1);

echo bccomp("10.5", "10.4", 1) . "\n";  // 1（大于）
echo bccomp("10.5", "10.5", 1) . "\n";  // 0（等于）
echo bccomp("10.4", "10.5", 1) . "\n";  // -1（小于）
```

### version_compare() - 版本比较

**语法**：`version_compare(string $version1, string $version2, ?string $operator = null): int|bool`

**参数**：
- `$version1`：要比较的第一个版本号
- `$version2`：要比较的第二个版本号
- `$operator`：可选，比较运算符（`'<'`、`'<='`、`'>'`、`'>='`、`'=='`、`'!='`、`'<>'`），默认为 `null`

**返回值**：如果指定了 `$operator`，返回布尔值（是否符合运算符条件）；否则返回整数（`-1`、`0` 或 `1`）。

```php
<?php
declare(strict_types=1);

// 比较版本
if (version_compare(PHP_VERSION, '8.2.0', '>=')) {
    echo "PHP 8.2.0 or higher\n";
}

// 返回比较结果
var_dump(version_compare('1.0.0', '1.0.1'));  // -1（小于）
var_dump(version_compare('1.0.1', '1.0.0'));  // 1（大于）
var_dump(version_compare('1.0.0', '1.0.0'));   // 0（等于）
```

## 太空船运算符（<=>）

PHP 7.0+ 引入了太空船运算符，用于三路比较：

**语法**：`$a <=> $b`

**返回值**：
- `< 0`：`$a` 小于 `$b`
- `= 0`：`$a` 等于 `$b`
- `> 0`：`$a` 大于 `$b`

```php
<?php
declare(strict_types=1);

echo (5 <=> 3) . "\n";   // 1（5 大于 3）
echo (3 <=> 5) . "\n";   // -1（3 小于 5）
echo (5 <=> 5) . "\n";   // 0（5 等于 5）

// 用于排序
$numbers = [3, 1, 4, 1, 5, 9, 2, 6];
usort($numbers, fn($a, $b) => $a <=> $b);
print_r($numbers);  // [1, 1, 2, 3, 4, 5, 6, 9]
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ComparisonDemo
{
    public static function demonstrate(): void
    {
        echo "=== 宽松比较 vs 严格比较 ===\n";
        $comparisons = [
            ['"5"', '5', "5" == 5, "5" === 5],
            ['"0"', 'false', "0" == false, "0" === false],
            ['[]', 'false', [] == false, [] === false],
        ];
        
        foreach ($comparisons as [$a, $b, $loose, $strict]) {
            echo "{$a} == {$b}: " . ($loose ? 'true' : 'false') . "\n";
            echo "{$a} === {$b}: " . ($strict ? 'true' : 'false') . "\n";
        }
        
        echo "\n=== 字符串比较 ===\n";
        echo "strcmp('apple', 'banana'): " . strcmp('apple', 'banana') . "\n";
        echo "strcasecmp('Apple', 'apple'): " . strcasecmp('Apple', 'apple') . "\n";
        
        echo "\n=== 版本比较 ===\n";
        echo "PHP Version: " . PHP_VERSION . "\n";
        echo "version_compare(PHP_VERSION, '8.0.0', '>='): " . 
             (version_compare(PHP_VERSION, '8.0.0', '>=') ? 'true' : 'false') . "\n";
    }
}

ComparisonDemo::demonstrate();
```

## 最佳实践

1. **始终使用严格比较**：使用 `===` 和 `!==` 而不是 `==` 和 `!=`

2. **密码和 token 验证**：必须使用严格比较，避免安全漏洞

3. **数值比较**：对于需要高精度的数值，使用 `bccomp()`

4. **版本比较**：使用 `version_compare()` 而不是字符串比较

5. **排序**：使用太空船运算符 `<=>` 简化排序逻辑

## 练习

1. 编写一个函数，比较两个值并返回详细的比较结果。

2. 创建一个安全的比较工具类，提供各种比较方法。

3. 实现一个版本管理工具，使用 `version_compare()` 处理版本号。

4. 编写测试用例，验证宽松比较和严格比较的差异。

5. 创建一个排序函数，使用太空船运算符实现多字段排序。
