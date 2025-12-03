# 2.6.6 运算符优先级与结合性

## 概述

运算符优先级决定了表达式中运算符的执行顺序。理解运算符优先级对于编写正确的代码至关重要。当优先级相同时，结合性决定了运算的方向。

## 运算符优先级表

| 优先级（高→低）      | 运算符示例                 | 结合性     |
| :------------------- | :------------------------- | :--------- |
| 括号                 | `( )`                      | 从左到右   |
| 指数                 | `**`                       | 从右到左   |
| 取正/取负、逻辑非    | `+ - !`                    | 从右到左   |
| 乘除模               | `* / %`                    | 从左到右   |
| 加减、字符串连接     | `+ - .`                    | 从左到右   |
| 移位                 | `<< >>`                    | 从左到右   |
| 比较                 | `< <= > >= <>`             | 无结合性   |
| 等于、不等           | `== != === !== <=>`        | 无结合性   |
| 位与/或/异或         | `& ^ \|`                   | 从左到右   |
| 逻辑与/或            | `&& \|\|`                  | 从左到右   |
| 空合并               | `??`                       | 从右到左   |
| 三元                 | `?:`                       | 从右到左   |
| 赋值                 | `=`、`+=` 等               | 从右到左   |
| `and`、`or`、`xor`   | `and`、`or`、`xor`         | 从左到右   |

## 常见优先级问题

### 算术运算符

```php
<?php
declare(strict_types=1);

// 指数优先级最高
echo 2 ** 3 * 4 . "\n";      // 32（先算 2**3=8，再算 8*4=32）
echo 2 ** (3 * 2) . "\n";    // 64（括号优先）

// 乘除优先级高于加减
echo 2 + 3 * 4 . "\n";       // 14（先算 3*4=12，再算 2+12=14）
echo (2 + 3) * 4 . "\n";     // 20（括号优先）
```

### 逻辑运算符

```php
<?php
declare(strict_types=1);

// && 优先级高于 ||
$result1 = true || false && false;  // true || (false && false) = true || false = true
var_dump($result1);  // bool(true)

// 使用括号明确意图
$result2 = (true || false) && false;  // true && false = false
var_dump($result2);  // bool(false)
```

### 空合并与三元运算符

```php
<?php
declare(strict_types=1);

// ?? 优先级高于 ?:
$result1 = $a ?? $b ? $c : $d;  // ($a ?? $b) ? $c : $d

// 但建议加括号明确意图
$result2 = ($a ?? $b) ? $c : $d;
```

### 赋值运算符

```php
<?php
declare(strict_types=1);

// 赋值运算符优先级很低
$a = 5;
$b = 10;
$result = $a = $b;  // 先执行 $a = $b，然后 $result = $a
echo "a = {$a}, result = {$result}\n";  // a = 10, result = 10
```

## 结合性

### 从左到右结合

大多数运算符从左到右结合：

```php
<?php
declare(strict_types=1);

// 从左到右
echo 10 - 5 - 2 . "\n";  // 3（(10-5)-2）
echo 10 / 5 / 2 . "\n";  // 1（(10/5)/2）
```

### 从右到左结合

某些运算符从右到左结合：

```php
<?php
declare(strict_types=1);

// 指数从右到左
echo 2 ** 3 ** 2 . "\n";  // 512（2**(3**2) = 2**9 = 512）

// 赋值从右到左
$a = $b = $c = 10;  // $c = 10, $b = $c, $a = $b

// 三元运算符从右到左
$result = $a ? $b : $c ? $d : $e;  // $a ? $b : ($c ? $d : $e)
```

## 使用括号

### 明确优先级

即使运算符优先级正确，使用括号可以提高代码可读性：

```php
<?php
declare(strict_types=1);

// 虽然正确，但可读性差
$result1 = $a && $b || $c && $d;

// 使用括号更清晰
$result2 = ($a && $b) || ($c && $d);
```

### 避免错误

```php
<?php
declare(strict_types=1);

// 可能不是预期结果
$result1 = $condition ? $a : $b + $c;  // $condition ? $a : ($b + $c)

// 使用括号明确意图
$result2 = ($condition ? $a : $b) + $c;
```

## 完整示例

```php
<?php
declare(strict_types=1);

class ExpressionEvaluator
{
    public static function demonstratePrecedence(): void
    {
        echo "=== 算术运算符 ===\n";
        echo "2 + 3 * 4 = " . (2 + 3 * 4) . "\n";           // 14
        echo "(2 + 3) * 4 = " . ((2 + 3) * 4) . "\n";     // 20
        echo "2 ** 3 * 4 = " . (2 ** 3 * 4) . "\n";       // 32
        echo "2 ** (3 * 2) = " . (2 ** (3 * 2)) . "\n";   // 64
        
        echo "\n=== 逻辑运算符 ===\n";
        $a = true;
        $b = false;
        $c = true;
        
        $result1 = $a && $b || $c;  // (true && false) || true = false || true = true
        echo "true && false || true = " . ($result1 ? 'true' : 'false') . "\n";
        
        $result2 = $a && ($b || $c);  // true && (false || true) = true && true = true
        echo "true && (false || true) = " . ($result2 ? 'true' : 'false') . "\n";
        
        echo "\n=== 赋值运算符 ===\n";
        $x = 5;
        $y = 10;
        $z = $x = $y;  // $x = $y, $z = $x
        echo "x = {$x}, y = {$y}, z = {$z}\n";  // x = 10, y = 10, z = 10
        
        echo "\n=== 空合并运算符 ===\n";
        $value1 = null;
        $value2 = null;
        $default = 'default';
        
        $result = $value1 ?? $value2 ?? $default;  // 从右到左结合
        echo "null ?? null ?? 'default' = {$result}\n";  // default
    }
}

ExpressionEvaluator::demonstratePrecedence();
```

## 最佳实践

1. **使用括号**：当优先级不明确时，使用括号明确意图。

2. **避免复杂表达式**：将复杂表达式拆分为多个步骤，提高可读性。

3. **理解结合性**：注意从右到左结合的运算符（如指数、赋值）。

4. **代码审查**：在代码审查中特别注意运算符优先级问题。

5. **文档说明**：对于复杂的表达式，添加注释说明计算顺序。

## 注意事项

1. **常见错误**：赋值运算符优先级很低，容易在条件表达式中出错。

2. **逻辑运算符**：注意 `&&` 和 `||` 的优先级高于 `and` 和 `or`。

3. **空合并**：`??` 的优先级低于 `?:`，组合使用时需加括号。

4. **指数运算**：指数运算符从右到左结合，注意计算顺序。

5. **可读性优先**：即使优先级正确，使用括号可以提高代码可读性。

## 练习

1. 编写一个函数，解析并计算数学表达式，正确处理运算符优先级。

2. 创建一个表达式验证工具，检查运算符优先级是否正确。

3. 实现一个计算器类，使用括号处理运算符优先级。

4. 编写测试用例，验证各种运算符优先级的计算结果。

5. 创建一个工具，将复杂表达式转换为使用括号的等价形式，提高可读性。

