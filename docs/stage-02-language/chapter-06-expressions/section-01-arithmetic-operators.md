# 2.6.1 算术运算符

## 概述

算术运算符用于进行数学计算。本节详细介绍加法、减法、乘法、除法、取模、指数运算符的语法、用法、类型转换及完整示例。

掌握算术运算符是进行数值计算的基础。理解运算符的行为、类型转换规则和常见陷阱可以帮助编写更可靠的代码。

## 特性

- **基本运算**：支持基本的数学运算（加、减、乘、除、取模、指数）
- **类型转换**：操作数会自动转换为数字类型
- **精度问题**：浮点数运算可能有精度问题
- **除零处理**：除以零会产生警告或错误

## 语法/定义

### 算术运算符

| 运算符 | 名称 | 语法 | 说明 | 示例 |
|:-------|:-----|:-----|:-----|:-----|
| `+` | 加法 | `$a + $b` | 两数相加 | `10 + 5` → `15` |
| `-` | 减法 | `$a - $b` | 两数相减 | `10 - 5` → `5` |
| `*` | 乘法 | `$a * $b` | 两数相乘 | `10 * 5` → `50` |
| `/` | 除法 | `$a / $b` | 两数相除 | `10 / 5` → `2.0` |
| `%` | 取模 | `$a % $b` | 取余数 | `10 % 3` → `1` |
| `**` | 指数 | `$a ** $b` | 幂运算 | `2 ** 3` → `8` |

**特点**：
- 所有运算符都是二元运算符（需要两个操作数）
- 操作数会自动转换为数字类型
- 除法运算结果始终是浮点数（即使能整除）
- 取模运算的操作数会转换为整数

## 基本用法

### 示例 1：基本算术运算

```php
<?php
declare(strict_types=1);

$a = 10;
$b = 3;

// 加法
$sum = $a + $b;
echo "Sum: {$sum}\n";  // Sum: 13

// 减法
$diff = $a - $b;
echo "Difference: {$diff}\n";  // Difference: 7

// 乘法
$product = $a * $b;
echo "Product: {$product}\n";  // Product: 30

// 除法
$quotient = $a / $b;
echo "Quotient: {$quotient}\n";  // Quotient: 3.3333333333333

// 取模
$remainder = $a % $b;
echo "Remainder: {$remainder}\n";  // Remainder: 1

// 指数
$power = $a ** $b;
echo "Power: {$power}\n";  // Power: 1000
```

**执行**：

```bash
php arithmetic-basic.php
```

**输出**：

```
Sum: 13
Difference: 7
Product: 30
Quotient: 3.3333333333333
Remainder: 1
Power: 1000
```

### 示例 2：类型转换

```php
<?php
declare(strict_types=1);

// 字符串转数字
$str = "10";
$num = 5;
$result = $str + $num;
echo "String + Number: {$result}\n";  // String + Number: 15

$str = "10.5";
$result = $str + $num;
echo "String + Number: {$result}\n";  // String + Number: 15.5

// 非数字字符串
$str = "abc";
$result = $str + $num;
echo "Non-numeric + Number: {$result}\n";  // Non-numeric + Number: 5（"abc" 转换为 0）

// 布尔值转换
$bool = true;
$result = $bool + 5;
echo "Bool + Number: {$result}\n";  // Bool + Number: 6（true 转换为 1）

$bool = false;
$result = $bool + 5;
echo "Bool + Number: {$result}\n";  // Bool + Number: 5（false 转换为 0）
```

### 示例 3：浮点数精度问题

```php
<?php
declare(strict_types=1);

// 浮点数精度问题
$result = 0.1 + 0.2;
echo "0.1 + 0.2 = {$result}\n";  // 0.1 + 0.2 = 0.30000000000000004

// 比较浮点数
var_dump($result === 0.3);  // bool(false)

// 使用误差范围比较
$epsilon = 0.00001;
var_dump(abs($result - 0.3) < $epsilon);  // bool(true)

// 使用 BCMath 扩展（如果可用）
if (function_exists('bcadd')) {
    $result = bcadd("0.1", "0.2", 2);
    echo "BCMath: 0.1 + 0.2 = {$result}\n";  // BCMath: 0.1 + 0.2 = 0.30
}
```

## 完整代码示例

### 示例 1：计算器函数

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
            throw new DivisionByZeroError("Division by zero");
        }
        return $a / $b;
    }
    
    public static function modulo(int $a, int $b): int
    {
        if ($b == 0) {
            throw new DivisionByZeroError("Modulo by zero");
        }
        return $a % $b;
    }
    
    public static function power(float $a, float $b): float
    {
        return $a ** $b;
    }
}

// 使用
echo Calculator::add(10, 5) . "\n";        // 15
echo Calculator::subtract(10, 5) . "\n";   // 5
echo Calculator::multiply(10, 5) . "\n";   // 50
echo Calculator::divide(10, 5) . "\n";     // 2
echo Calculator::modulo(10, 3) . "\n";     // 1
echo Calculator::power(2, 3) . "\n";       // 8
```

### 示例 2：统计计算

```php
<?php
declare(strict_types=1);

function calculateStatistics(array $numbers): array
{
    if (empty($numbers)) {
        return [
            'count' => 0,
            'sum' => 0,
            'average' => 0,
            'min' => null,
            'max' => null,
        ];
    }
    
    $count = count($numbers);
    $sum = array_sum($numbers);
    $average = $sum / $count;
    $min = min($numbers);
    $max = max($numbers);
    
    return [
        'count' => $count,
        'sum' => $sum,
        'average' => $average,
        'min' => $min,
        'max' => $max,
    ];
}

$numbers = [10, 20, 30, 40, 50];
$stats = calculateStatistics($numbers);
print_r($stats);
```

### 示例 3：除零处理

```php
<?php
declare(strict_types=1);

function safeDivide(float $a, float $b): ?float
{
    if ($b == 0) {
        return null;  // 或抛出异常
    }
    return $a / $b;
}

function safeModulo(int $a, int $b): ?int
{
    if ($b == 0) {
        return null;  // 或抛出异常
    }
    return $a % $b;
}

// 使用
$result = safeDivide(10, 5);
if ($result !== null) {
    echo "Result: {$result}\n";
} else {
    echo "Division by zero\n";
}

$result = safeDivide(10, 0);
if ($result !== null) {
    echo "Result: {$result}\n";
} else {
    echo "Division by zero\n";
}
```

## 使用场景

### 数值计算

- **数学运算**：进行各种数学计算
- **数据处理**：计算统计信息、平均值等
- **算法实现**：实现各种算法（排序、搜索等）

### 业务逻辑

- **价格计算**：计算商品价格、折扣等
- **数量统计**：统计数量、库存等
- **百分比计算**：计算百分比、增长率等

### 科学计算

- **物理计算**：进行物理量计算
- **统计分析**：进行统计分析
- **工程计算**：进行工程计算

## 注意事项

### 除零错误

- **除法运算**：除以零会产生 `DivisionByZeroError`（PHP 8.0+）或警告（PHP 7.x）
- **取模运算**：取模零会产生 `DivisionByZeroError`（PHP 8.0+）或警告（PHP 7.x）
- **检查除数**：在进行除法或取模运算前检查除数是否为零

### 精度问题

- **浮点数精度**：浮点数不能精确表示所有小数
- **比较浮点数**：比较浮点数时使用误差范围
- **精确计算**：需要精确计算时使用 BCMath 扩展

### 类型转换

- **自动转换**：操作数会自动转换为数字类型
- **字符串解析**：字符串从开头解析数字
- **布尔值转换**：`true` → 1，`false` → 0

### 整数溢出

- **范围限制**：整数有范围限制
- **超出范围**：超出范围会转换为浮点数
- **大数计算**：大数计算使用 BCMath 扩展

## 常见问题

### 问题 1：除零错误

**症状**：

```
Warning: Division by zero
或
Fatal error: Uncaught DivisionByZeroError
```

**原因**：除数为零

**错误示例**：

```php
<?php
declare(strict_types=1);

$result = 10 / 0;  // 错误
```

**解决方法**：

```php
<?php
declare(strict_types=1);

function divide(float $a, float $b): float
{
    if ($b == 0) {
        throw new DivisionByZeroError("Division by zero");
    }
    return $a / $b;
}

// 使用
try {
    $result = divide(10, 0);
} catch (DivisionByZeroError $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

### 问题 2：浮点数精度问题

**症状**：浮点数计算结果不符合预期

**原因**：浮点数精度有限

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

$result = 0.1 + 0.2;

// 方法1：使用误差范围
$epsilon = 0.00001;
var_dump(abs($result - 0.3) < $epsilon);  // bool(true)

// 方法2：使用 BCMath 扩展
if (function_exists('bcadd')) {
    $result = bcadd("0.1", "0.2", 2);
    var_dump($result === "0.30");  // bool(true)
}
```

### 问题 3：整数溢出

**症状**：大整数计算结果不正确

**原因**：整数超出范围

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
if (function_exists('bcadd')) {
    $result = bcadd((string) PHP_INT_MAX, "1");
    echo "BCMath result: {$result}\n";
}
```

## 最佳实践

### 除零处理

- **检查除数**：在进行除法或取模运算前检查除数
- **异常处理**：使用异常处理除零错误
- **提供默认值**：除零时提供合理的默认值

### 精度处理

- **使用误差范围**：比较浮点数时使用误差范围
- **使用 BCMath**：需要精确计算时使用 BCMath 扩展
- **避免直接比较**：不要直接使用 `===` 比较浮点数

### 类型安全

- **类型声明**：为函数参数添加类型声明
- **显式转换**：需要时进行显式类型转换
- **验证输入**：验证用户输入的类型

## 对比分析

### 整数运算 vs 浮点数运算

| 特性 | 整数运算 | 浮点数运算 |
|:-----|:---------|:-----------|
| 精度 | 精确 | 有限 |
| 范围 | 有限 | 更大 |
| 性能 | 更好 | 略差 |
| 适用场景 | 计数、ID | 价格、坐标 |

**选择建议**：
- **计数、ID**：使用整数
- **价格、坐标**：使用浮点数
- **精确计算**：使用 BCMath 扩展

### 算术运算符 vs 数学函数

| 特性 | 算术运算符 | 数学函数 |
|:-----|:-----------|:---------|
| 语法 | 简洁 | 函数调用 |
| 性能 | 更好 | 略差 |
| 功能 | 基本运算 | 高级运算 |
| 推荐度 | 推荐（基本） | 推荐（高级） |

**选择建议**：
- **基本运算**：使用算术运算符
- **高级运算**：使用数学函数（如 `sqrt()`、`sin()` 等）

## 相关章节

- **2.4.1 标量类型**：了解整数和浮点数类型
- **2.5.1 隐式转换**：了解类型转换规则
- **2.5.4 比较运算符与函数**：了解比较运算符
- **阶段四：系统编程**：了解 BCMath 扩展的使用

## 练习任务

1. **基本运算练习**：
   - 练习使用各种算术运算符
   - 理解运算符的行为
   - 测试不同类型操作数的运算
   - 观察类型转换结果

2. **除零处理练习**：
   - 实现安全的除法函数
   - 处理除零错误
   - 使用异常处理
   - 测试各种场景

3. **精度处理练习**：
   - 观察浮点数精度问题
   - 使用误差范围比较浮点数
   - 使用 BCMath 扩展进行精确计算
   - 测试不同场景下的精度

4. **实际应用练习**：
   - 实现一个计算器类
   - 实现统计计算函数
   - 处理用户输入的数值计算
   - 测试各种计算场景

5. **综合练习**：
   - 创建一个完整的数值计算工具
   - 实现各种数学运算
   - 处理除零和精度问题
   - 进行代码审查，确保计算正确
