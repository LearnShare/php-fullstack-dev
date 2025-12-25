# 2.5.3 转换函数

## 概述

PHP 提供了一系列函数用于类型转换，这些函数提供了比类型转换操作符更多的功能和选项。

## intval() - 转换为整数

### 语法

**语法**：`intval(mixed $value, int $base = 10): int`

**参数**：
- `$value`：要转换的值
- `$base`：可选，进制基数（2-36），默认为 10

**返回值**：返回转换后的整数值。

### 基本用法

```php
<?php
declare(strict_types=1);

echo intval(42) . "\n";           // 42
echo intval(42.7) . "\n";         // 42（截断）
echo intval("42") . "\n";         // 42
echo intval("42abc") . "\n";      // 42（忽略非数字字符）
echo intval("abc42") . "\n";      // 0（无法解析）
```

### 指定进制

```php
<?php
declare(strict_types=1);

echo intval('0xff', 16) . "\n";   // 255（十六进制）
echo intval('1010', 2) . "\n";    // 10（二进制）
echo intval('077', 8) . "\n";     // 63（八进制）
```

## floatval() - 转换为浮点数

### 语法

**语法**：`floatval(mixed $value): float`

**参数**：
- `$value`：要转换的值

**返回值**：返回转换后的浮点数值。

### 基本用法

```php
<?php
declare(strict_types=1);

echo floatval(42) . "\n";         // 42.0
echo floatval("3.14") . "\n";     // 3.14
echo floatval("19.95USD") . "\n"; // 19.95（忽略非数字字符）
echo floatval("abc3.14") . "\n";  // 0.0（无法解析）
```

## boolval() - 转换为布尔值

### 语法

**语法**：`boolval(mixed $value): bool`

**参数**：
- `$value`：要转换的值

**返回值**：返回转换后的布尔值。

### 基本用法

```php
<?php
declare(strict_types=1);

var_dump(boolval(0));        // bool(false)
var_dump(boolval(1));        // bool(true)
var_dump(boolval("0"));      // bool(false)
var_dump(boolval("false"));  // bool(true) - 注意：非空字符串
var_dump(boolval([]));       // bool(false)
var_dump(boolval([1, 2]));  // bool(true)
```

## strval() - 转换为字符串

### 语法

**语法**：`strval(mixed $value): string`

**参数**：
- `$value`：要转换的值

**返回值**：返回转换后的字符串。

### 基本用法

```php
<?php
declare(strict_types=1);

echo strval(123) . "\n";        // "123"
echo strval(3.14) . "\n";       // "3.14"
echo strval(true) . "\n";       // "1"
echo strval(false) . "\n";      // ""（空字符串）
echo strval([1, 2, 3]) . "\n";  // "Array"
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ConversionFunctions
{
    public static function demonstrate(): void
    {
        echo "=== intval() ===\n";
        echo "intval(42): " . intval(42) . "\n";
        echo "intval(42.7): " . intval(42.7) . "\n";
        echo "intval('42'): " . intval('42') . "\n";
        echo "intval('0xff', 16): " . intval('0xff', 16) . "\n";
        
        echo "\n=== floatval() ===\n";
        echo "floatval(42): " . floatval(42) . "\n";
        echo "floatval('3.14'): " . floatval('3.14') . "\n";
        echo "floatval('19.95USD'): " . floatval('19.95USD') . "\n";
        
        echo "\n=== boolval() ===\n";
        $values = [0, 1, "0", "false", [], [1, 2]];
        foreach ($values as $value) {
            echo "boolval(" . var_export($value, true) . "): " . 
                 (boolval($value) ? 'true' : 'false') . "\n";
        }
        
        echo "\n=== strval() ===\n";
        echo "strval(123): '" . strval(123) . "'\n";
        echo "strval(3.14): '" . strval(3.14) . "'\n";
        echo "strval(true): '" . strval(true) . "'\n";
        echo "strval(false): '" . strval(false) . "'\n";
    }
}

ConversionFunctions::demonstrate();
```

## 注意事项

1. **intval() 与 (int)**：`intval()` 支持指定进制，`(int)` 不支持。

2. **精度问题**：浮点数转整数会截断，注意精度损失。

3. **字符串解析**：转换函数会尽可能解析字符串，遇到无法解析的字符会停止。

4. **布尔转换**：注意 `"false"` 字符串会转为 `true`，因为它是非空字符串。

## 练习

1. 编写一个函数，使用 `intval()` 解析不同进制的数字字符串。

2. 创建一个类型转换工具，比较 `intval()` 和 `(int)` 的差异。

3. 实现一个安全的数值转换函数，处理各种边界情况。
