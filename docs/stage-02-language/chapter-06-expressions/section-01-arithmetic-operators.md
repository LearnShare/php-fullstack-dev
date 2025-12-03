# 2.6.1 算术运算符

## 概述

算术运算符用于执行基本的数学运算。PHP 支持标准的算术运算符，包括加法、减法、乘法、除法、取模和指数运算。

## 基本算术运算符

| 运算符 | 名称     | 示例      | 结果 |
| :----- | :------- | :-------- | :--- |
| `+`    | 加法     | `5 + 3`   | `8`  |
| `-`    | 减法     | `5 - 3`   | `2`  |
| `*`    | 乘法     | `5 * 3`   | `15` |
| `/`    | 除法     | `10 / 3`  | `3.333...` |
| `%`    | 取模     | `10 % 3`  | `1`  |
| `**`   | 指数     | `2 ** 3`  | `8`  |

## 加法运算符（+）

### 基本用法

```php
<?php
declare(strict_types=1);

echo 5 + 3 . "\n";        // 8
echo 5.5 + 2.3 . "\n";    // 7.8
echo -5 + 3 . "\n";       // -2
```

### 类型转换

```php
<?php
declare(strict_types=1);

echo "5" + 3 . "\n";      // 8（字符串转为数字）
echo 5 + "3" . "\n";      // 8（字符串转为数字）
echo "5.5" + 2 . "\n";    // 7.5（字符串转为浮点数）
```

## 减法运算符（-）

### 基本用法

```php
<?php
declare(strict_types=1);

echo 5 - 3 . "\n";        // 2
echo 5.5 - 2.3 . "\n";    // 3.2
echo -5 - 3 . "\n";       // -8
```

## 乘法运算符（*）

### 基本用法

```php
<?php
declare(strict_types=1);

echo 5 * 3 . "\n";        // 15
echo 5.5 * 2 . "\n";      // 11.0
echo -5 * 3 . "\n";       // -15
```

## 除法运算符（/）

### 基本用法

**注意**：除法运算符总是返回浮点数（即使两个操作数都是整数）。

```php
<?php
declare(strict_types=1);

echo 10 / 3 . "\n";       // 3.3333333333333
echo 10 / 2 . "\n";       // 5.0（注意：是浮点数）
echo 10.0 / 3 . "\n";     // 3.3333333333333
```

### 整数除法

如果需要整数除法，使用 `intdiv()` 函数（PHP 7.0+）：

```php
<?php
declare(strict_types=1);

echo intdiv(10, 3) . "\n";  // 3（整数除法）
echo intdiv(10, 2) . "\n";  // 5
echo intdiv(-10, 3) . "\n"; // -3
```

### 除零错误

```php
<?php
declare(strict_types=1);

// 除以零会产生警告或错误
// echo 10 / 0;  // Warning: Division by zero

// 使用 intdiv 除以零会抛出异常
try {
    echo intdiv(10, 0);
} catch (DivisionByZeroError $e) {
    echo "Division by zero error: " . $e->getMessage() . "\n";
}
```

## 取模运算符（%）

### 基本用法

取模运算符返回除法的余数。结果的符号与被除数（左操作数）一致。

```php
<?php
declare(strict_types=1);

echo 10 % 3 . "\n";       // 1
echo 10 % 2 . "\n";       // 0
echo -10 % 3 . "\n";      // -1（符号与被除数一致）
echo 10 % -3 . "\n";      // 1（符号与被除数一致）
```

### 浮点数取模

```php
<?php
declare(strict_types=1);

echo 10.5 % 3 . "\n";     // 1（浮点数转为整数后取模）
echo fmod(10.5, 3) . "\n"; // 1.5（使用 fmod 进行浮点数取模）
```

### fmod() 函数

**语法**：`fmod(float $num1, float $num2): float`

用于浮点数取模：

```php
<?php
declare(strict_types=1);

echo fmod(10.5, 3) . "\n";    // 1.5
echo fmod(10.5, 3.5) . "\n";  // 0.0
```

## 指数运算符（**）

### 基本用法

指数运算符（PHP 5.6+）用于计算幂运算。

```php
<?php
declare(strict_types=1);

echo 2 ** 3 . "\n";       // 8（2 的 3 次方）
echo 5 ** 2 . "\n";       // 25（5 的 2 次方）
echo 2 ** 0.5 . "\n";     // 1.4142135623731（2 的平方根）
echo 2 ** -2 . "\n";      // 0.25（2 的 -2 次方）
```

### 与 pow() 函数对比

```php
<?php
declare(strict_types=1);

// 使用 ** 运算符
echo 2 ** 3 . "\n";       // 8

// 使用 pow() 函数
echo pow(2, 3) . "\n";     // 8

// 两者功能相同，但 ** 运算符更简洁
```

## 运算符优先级

算术运算符的优先级（从高到低）：

1. `**`（指数）
2. `*`、`/`、`%`（乘法、除法、取模）
3. `+`、`-`（加法、减法）

```php
<?php
declare(strict_types=1);

echo 2 + 3 * 4 . "\n";        // 14（先乘后加）
echo (2 + 3) * 4 . "\n";     // 20（括号优先）
echo 2 ** 3 * 4 . "\n";      // 32（先指数后乘）
echo 2 ** (3 * 2) . "\n";    // 64（括号优先）
```

## 完整示例

```php
<?php
declare(strict_types=1);

class Calculator
{
    public static function add(float $a, float $b): float
    {
        return $a + $b;
    }
    
    public static function subtract(float $a, float $b): float
    {
        return $a - $b;
    }
    
    public static function multiply(float $a, float $b): float
    {
        return $a * $b;
    }
    
    public static function divide(float $a, float $b): float
    {
        if ($b == 0) {
            throw new DivisionByZeroError("Cannot divide by zero");
        }
        return $a / $b;
    }
    
    public static function modulo(float $a, float $b): float
    {
        if ($b == 0) {
            throw new DivisionByZeroError("Cannot modulo by zero");
        }
        return fmod($a, $b);
    }
    
    public static function power(float $base, float $exponent): float
    {
        return $base ** $exponent;
    }
    
    public static function calculate(string $expression): float
    {
        // 简单的表达式计算（仅作示例，实际应使用更安全的方法）
        return eval("return {$expression};");
    }
}

// 使用示例
echo "Addition: " . Calculator::add(10, 5) . "\n";
echo "Subtraction: " . Calculator::subtract(10, 5) . "\n";
echo "Multiplication: " . Calculator::multiply(10, 5) . "\n";
echo "Division: " . Calculator::divide(10, 5) . "\n";
echo "Modulo: " . Calculator::modulo(10, 3) . "\n";
echo "Power: " . Calculator::power(2, 3) . "\n";
```

## 注意事项

1. **除法结果**：除法运算符总是返回浮点数，即使两个操作数都是整数。

2. **除零处理**：除以零会产生警告，使用 `intdiv()` 除以零会抛出异常。

3. **取模符号**：取模结果的符号与被除数一致。

4. **浮点数精度**：浮点数运算可能存在精度问题，需要高精度计算时使用 `bcmath` 扩展。

5. **类型转换**：字符串参与算术运算时会被转换为数字。

## 练习

1. 创建一个 `Calculator` 类，实现基本的算术运算方法，包含错误处理。

2. 编写一个函数，计算圆的面积和周长，使用 `M_PI` 常量。

3. 实现一个函数，计算复利：`A = P(1 + r/n)^(nt)`，其中 P 是本金，r 是利率，n 是复利次数，t 是时间。

4. 编写一个函数，使用取模运算符判断一个数是否为偶数或奇数。

5. 创建一个函数，计算两个数的最大公约数（GCD）和最小公倍数（LCM）。

