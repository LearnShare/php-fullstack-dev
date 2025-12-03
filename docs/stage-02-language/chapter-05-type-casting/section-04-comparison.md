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

**返回值**：
- `< 0`：`$string1` 小于 `$string2`
- `= 0`：`$string1` 等于 `$string2`
- `> 0`：`$string1` 大于 `$string2`

```php
<?php
declare(strict_types=1);

echo strcmp("apple", "banana") . "\n";  // 负数
echo strcmp("banana", "apple") . "\n";  // 正数
echo strcmp("apple", "apple") . "\n";   // 0
```

### strcasecmp() - 不区分大小写比较

**语法**：`strcasecmp(string $string1, string $string2): int`

```php
<?php
declare(strict_types=1);

echo strcasecmp("Apple", "apple") . "\n";  // 0（相等）
```

### bccomp() - 高精度比较

**语法**：`bccomp(string $num1, string $num2, ?int $scale = null): int`

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
