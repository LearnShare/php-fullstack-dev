# 2.2.2 格式化占位符详解

## 概述

格式化占位符是 `printf` 和 `sprintf` 的核心功能，用于生成格式化的字符串。本节详细介绍格式化占位符的语法、类型、参数位置指定及实际应用场景。

掌握格式化占位符的使用，能够生成格式化的日志消息、订单号、表格输出等，是实际开发中最常用的功能之一。

## 特性

- **类型丰富**：支持字符串、整数、浮点数、十六进制、八进制、二进制等多种类型
- **参数位置**：可以指定参数位置，重复使用参数
- **宽度控制**：可以控制输出宽度，实现对齐
- **精度控制**：可以控制浮点数精度
- **标志控制**：可以控制填充字符、对齐方式等

## 语法/定义

### 基本语法

**格式**：`%[参数位置$][标志][宽度][.精度]类型`

**组成部分**：
- **参数位置**：`%1$s`、`%2$d`（可选）
- **标志**：`-`（左对齐）、`0`（零填充）、`+`（显示正负号）等（可选）
- **宽度**：`10`（最小宽度）（可选）
- **精度**：`.2`（浮点数精度）（可选）
- **类型**：`s`（字符串）、`d`（整数）、`f`（浮点数）等（必需）

### 常用占位符类型

| 占位符 | 类型 | 说明 | 示例 |
|:-------|:-----|:-----|:-----|
| `%s` | string | 字符串 | `sprintf("%s", "Hello")` → `"Hello"` |
| `%d` | int | 整数（十进制） | `sprintf("%d", 42)` → `"42"` |
| `%f` | float | 浮点数 | `sprintf("%f", 99.99)` → `"99.990000"` |
| `%x` | int | 十六进制（小写） | `sprintf("%x", 255)` → `"ff"` |
| `%X` | int | 十六进制（大写） | `sprintf("%X", 255)` → `"FF"` |
| `%o` | int | 八进制 | `sprintf("%o", 64)` → `"100"` |
| `%b` | int | 二进制 | `sprintf("%b", 10)` → `"1010"` |
| `%c` | int | 字符（ASCII 码） | `sprintf("%c", 65)` → `"A"` |
| `%e` | float | 科学计数法（小写） | `sprintf("%e", 1000)` → `"1.000000e+3"` |
| `%E` | float | 科学计数法（大写） | `sprintf("%E", 1000)` → `"1.000000E+3"` |

### 参数位置指定

**语法**：`%1$s`、`%2$d`、`%3$f`

**作用**：
- 指定使用第几个参数
- 可以重复使用同一个参数
- 可以改变参数顺序

**示例**：

```php
<?php
declare(strict_types=1);

// 重复使用参数
sprintf("%2$s %1$s", "World", "Hello");  // "Hello World"

// 改变参数顺序
sprintf("%1$s %1$s", "Hello");  // "Hello Hello"
```

### 宽度和精度控制

**宽度控制**：
- `%10s`：最小宽度 10，右对齐
- `%-10s`：最小宽度 10，左对齐
- `%010d`：最小宽度 10，零填充

**精度控制**：
- `%.2f`：保留两位小数
- `%.4f`：保留四位小数

## 基本用法

### 示例 1：基础占位符

```php
<?php
declare(strict_types=1);

// 字符串
$name = sprintf("Hello, %s!", "World");
echo $name . "\n";  // Hello, World!

// 整数
$number = sprintf("Number: %d", 42);
echo $number . "\n";  // Number: 42

// 浮点数
$price = sprintf("Price: %.2f", 99.99);
echo $price . "\n";  // Price: 99.99

// 十六进制
$hex = sprintf("Hex: %x", 255);
echo $hex . "\n";  // Hex: ff
```

**执行**：

```bash
php basic.php
```

**输出**：

```
Hello, World!
Number: 42
Price: 99.99
Hex: ff
```

### 示例 2：参数位置指定

```php
<?php
declare(strict_types=1);

// 重复使用参数
$result1 = sprintf("%2$s %1$s", "World", "Hello");
echo $result1 . "\n";  // Hello World

// 改变参数顺序
$result2 = sprintf("%1$s %1$s", "Hello");
echo $result2 . "\n";  // Hello Hello

// 复杂示例
$result3 = sprintf("%2$s %1$s %2$s", "World", "Hello");
echo $result3 . "\n";  // Hello World Hello
```

**执行**：

```bash
php position.php
```

**输出**：

```
Hello World
Hello Hello
Hello World Hello
```

### 示例 3：宽度和精度控制

```php
<?php
declare(strict_types=1);

// 宽度控制（右对齐）
$result1 = sprintf("%10s", "Hello");
echo "'{$result1}'\n";  // '     Hello'

// 宽度控制（左对齐）
$result2 = sprintf("%-10s", "Hello");
echo "'{$result2}'\n";  // 'Hello     '

// 零填充
$result3 = sprintf("%05d", 42);
echo $result3 . "\n";  // 00042

// 精度控制
$result4 = sprintf("%.2f", 99.999);
echo $result4 . "\n";  // 99.99

// 宽度和精度组合
$result5 = sprintf("%10.2f", 99.999);
echo "'{$result5}'\n";  // '     99.99'
```

**执行**：

```bash
php width-precision.php
```

**输出**：

```
'     Hello'
'Hello     '
00042
99.99
'     99.99'
```

## 完整代码示例

### 示例 1：日志格式生成

```php
<?php
declare(strict_types=1);

function generateLogMessage(string $level, string $message, array $context = []): string
{
    $timestamp = date('Y-m-d H:i:s');
    $contextStr = !empty($context) ? json_encode($context) : '';
    
    return sprintf(
        "[%s] %-5s: %s%s\n",
        $timestamp,
        strtoupper($level),
        $message,
        $contextStr ? " " . $contextStr : ''
    );
}

// 生成日志消息
$log1 = generateLogMessage("info", "User logged in", ["user_id" => 123]);
$log2 = generateLogMessage("error", "Database connection failed");
$log3 = generateLogMessage("debug", "Processing request", ["request_id" => "abc123"]);

echo $log1;
echo $log2;
echo $log3;
```

**执行**：

```bash
php log-format.php
```

**输出**：

```
[2024-01-15 10:30:00] INFO : User logged in {"user_id":123}
[2024-01-15 10:30:00] ERROR: Database connection failed
[2024-01-15 10:30:00] DEBUG: Processing request {"request_id":"abc123"}
```

### 示例 2：订单号生成

```php
<?php
declare(strict_types=1);

function generateOrderNumber(int $userId, int $orderId): string
{
    $date = date('Ymd');
    $time = date('His');
    
    return sprintf(
        "ORD-%s-%s-%06d-%06d",
        $date,
        $time,
        $userId,
        $orderId
    );
}

// 生成订单号
$order1 = generateOrderNumber(12345, 67890);
$order2 = generateOrderNumber(11111, 22222);

echo "Order 1: {$order1}\n";
echo "Order 2: {$order2}\n";
```

**执行**：

```bash
php order-number.php
```

**输出**：

```
Order 1: ORD-20240115-103000-012345-067890
Order 2: ORD-20240115-103000-011111-022222
```

### 示例 3：表格输出

```php
<?php
declare(strict_types=1);

$users = [
    ['name' => 'Alice', 'age' => 25, 'score' => 95.5],
    ['name' => 'Bob', 'age' => 30, 'score' => 88.3],
    ['name' => 'Charlie', 'age' => 28, 'score' => 92.7],
];

// 表头
printf("%-10s %5s %8s\n", "Name", "Age", "Score");
echo str_repeat("-", 25) . "\n";

// 表格内容
foreach ($users as $user) {
    printf(
        "%-10s %5d %8.2f\n",
        $user['name'],
        $user['age'],
        $user['score']
    );
}
```

**执行**：

```bash
php table.php
```

**输出**：

```
Name       Age   Score
-------------------------
Alice        25    95.50
Bob          30    88.30
Charlie      28    92.70
```

### 示例 4：数据格式化

```php
<?php
declare(strict_types=1);

// 格式化价格
$price = 99.99;
$formattedPrice = sprintf("$%.2f", $price);
echo "Price: {$formattedPrice}\n";  // Price: $99.99

// 格式化百分比
$percentage = 0.8567;
$formattedPercentage = sprintf("%.1f%%", $percentage * 100);
echo "Percentage: {$formattedPercentage}\n";  // Percentage: 85.7%

// 格式化文件大小
$size = 1048576;
$formattedSize = sprintf("%.2f MB", $size / 1024 / 1024);
echo "Size: {$formattedSize}\n";  // Size: 1.00 MB

// 格式化十六进制颜色
$color = 0xFF5733;
$formattedColor = sprintf("#%06X", $color);
echo "Color: {$formattedColor}\n";  // Color: #FF5733
```

**执行**：

```bash
php format-data.php
```

**输出**：

```
Price: $99.99
Percentage: 85.7%
Size: 1.00 MB
Color: #FF5733
```

## 使用场景

### 日志格式

**场景**：生成格式化的日志消息

**示例**：

```php
<?php
declare(strict_types=1);

$logMessage = sprintf(
    "[%s] %s: %s (User: %d, IP: %s)",
    date('Y-m-d H:i:s'),
    "INFO",
    "User logged in",
    12345,
    "192.168.1.1"
);
echo $logMessage . "\n";
```

### 订单号生成

**场景**：生成格式化的订单号

**示例**：

```php
<?php
declare(strict_types=1);

$orderNumber = sprintf(
    "ORD-%s-%06d",
    date('YmdHis'),
    12345
);
echo $orderNumber . "\n";  // ORD-20240115103000-012345
```

### 表格输出

**场景**：生成对齐的表格输出

**示例**：

```php
<?php
declare(strict_types=1);

printf("%-15s %10s %10s\n", "Product", "Price", "Stock");
echo str_repeat("-", 35) . "\n";
printf("%-15s %10.2f %10d\n", "Apple", 5.99, 100);
printf("%-15s %10.2f %10d\n", "Banana", 3.49, 50);
```

### 数据格式化

**场景**：格式化数值、日期等数据

**示例**：

```php
<?php
declare(strict_types=1);

// 格式化货币
$amount = 1234.56;
$formatted = sprintf("$%.2f", $amount);

// 格式化百分比
$rate = 0.1234;
$formatted = sprintf("%.2f%%", $rate * 100);
```

## 注意事项

### 类型匹配

- **占位符类型必须与参数类型匹配**：使用 `%d` 对应整数，`%s` 对应字符串
- **类型不匹配可能导致错误**：如使用 `%d` 格式化字符串可能产生意外结果

**错误示例**：

```php
<?php
declare(strict_types=1);

// 错误：类型不匹配
sprintf("%d", "abc");  // 可能产生意外结果
```

**正确示例**：

```php
<?php
declare(strict_types=1);

// 正确：类型匹配
sprintf("%s", "abc");  // "abc"
sprintf("%d", 123);    // "123"
```

### 参数数量

- **参数数量必须足够**：占位符数量不能超过参数数量
- **参数不足会导致警告**：PHP 会发出警告

**错误示例**：

```php
<?php
declare(strict_types=1);

// 错误：参数不足
sprintf("%s %s", "Hello");  // Warning: Too few arguments
```

**正确示例**：

```php
<?php
declare(strict_types=1);

// 正确：参数足够
sprintf("%s %s", "Hello", "World");  // "Hello World"
```

### 精度问题

- **浮点数精度可能不准确**：由于浮点数表示的限制，精度可能不准确
- **使用合适的精度**：根据实际需求选择合适的精度

**示例**：

```php
<?php
declare(strict_types=1);

// 浮点数精度问题
$value = 0.1 + 0.2;
echo sprintf("%.20f", $value) . "\n";  // 可能显示 0.30000000000000004441

// 使用合适的精度
echo sprintf("%.2f", $value) . "\n";  // 0.30
```

### 特殊字符

- **百分号转义**：使用 `%%` 表示字面量 `%`
- **其他特殊字符**：注意处理特殊字符

**示例**：

```php
<?php
declare(strict_types=1);

// 百分号转义
$result = sprintf("Discount: %d%%", 20);
echo $result . "\n";  // Discount: 20%
```

## 常见问题

### 问题 1：参数数量不足

**症状**：

```
Warning: sprintf(): Too few arguments
```

**原因**：占位符数量多于参数数量

**解决方法**：

1. **检查占位符数量**：确保占位符数量不超过参数数量
2. **使用参数位置指定**：使用 `%1$s` 重复使用参数

**示例**：

```php
<?php
declare(strict_types=1);

// 错误：参数不足
// sprintf("%s %s", "Hello");  // Warning

// 正确：参数足够
sprintf("%s %s", "Hello", "World");  // "Hello World"

// 或使用参数位置指定
sprintf("%1$s %1$s", "Hello");  // "Hello Hello"
```

### 问题 2：类型不匹配

**症状**：输出结果不符合预期

**原因**：占位符类型与参数类型不匹配

**解决方法**：

1. **检查参数类型**：确保参数类型与占位符类型匹配
2. **使用正确的占位符**：根据参数类型选择正确的占位符

**示例**：

```php
<?php
declare(strict_types=1);

// 错误：类型不匹配
sprintf("%d", "abc");  // 可能产生意外结果

// 正确：类型匹配
sprintf("%s", "abc");  // "abc"
sprintf("%d", 123);    // "123"
```

### 问题 3：宽度和精度理解错误

**症状**：输出格式不符合预期

**原因**：对宽度和精度的理解不正确

**解决方法**：

1. **理解宽度**：宽度是最小宽度，不是最大宽度
2. **理解精度**：精度是小数位数，不是总位数

**示例**：

```php
<?php
declare(strict_types=1);

// 宽度：最小宽度，不足时填充
sprintf("%5s", "Hi");   // "   Hi"（右对齐，宽度 5）
sprintf("%-5s", "Hi");  // "Hi   "（左对齐，宽度 5）

// 精度：小数位数
sprintf("%.2f", 99.999);  // "99.99"（保留两位小数）
```

## 最佳实践

### 占位符选择

- **使用合适的占位符类型**：根据参数类型选择正确的占位符
- **使用参数位置指定**：提高可读性，避免参数顺序错误
- **使用宽度和精度控制**：实现对齐和格式化

### 格式化字符串

- **使用 sprintf 生成格式化字符串**：适合需要后续处理的场景
- **使用 printf 直接输出**：适合直接输出到标准输出
- **保持格式一致**：在整个项目中保持格式化字符串的一致性

### 代码可读性

- **使用有意义的格式**：格式化字符串应清晰表达意图
- **添加注释**：为复杂的格式化字符串添加注释
- **提取格式化逻辑**：将格式化逻辑提取到函数中

## 对比分析

### 占位符类型对比

| 占位符 | 类型 | 输出示例 | 适用场景 |
|:-------|:-----|:---------|:---------|
| `%s` | string | `"Hello"` | 字符串输出 |
| `%d` | int | `"42"` | 整数输出 |
| `%f` | float | `"99.99"` | 浮点数输出 |
| `%x` | int | `"ff"` | 十六进制输出（小写） |
| `%X` | int | `"FF"` | 十六进制输出（大写） |
| `%o` | int | `"100"` | 八进制输出 |
| `%b` | int | `"1010"` | 二进制输出 |

**选择建议**：
- **字符串**：使用 `%s`
- **整数**：使用 `%d`
- **浮点数**：使用 `%f` 并指定精度
- **十六进制**：使用 `%x` 或 `%X`

### sprintf vs 字符串连接

| 特性 | sprintf | 字符串连接 |
|:-----|:---------|:-----------|
| 可读性 | 高 | 低 |
| 性能 | 较慢 | 较快 |
| 格式化 | 支持 | 不支持 |
| 推荐度 | 推荐（格式化） | 推荐（简单） |

**选择建议**：
- **需要格式化**：使用 `sprintf`
- **简单连接**：使用字符串连接或 `echo` 多参数

## 相关章节

- **2.2.1 输出 API**：了解 `printf` 和 `sprintf` 的基本用法
- **2.2.3 调试函数和策略**：了解调试函数的使用
- **2.7 字符串操作**：了解字符串处理的其他方法

## 练习任务

1. **基础占位符练习**：
   - 使用不同的占位符类型格式化数据
   - 练习字符串、整数、浮点数的格式化
   - 练习十六进制、八进制、二进制的格式化

2. **参数位置练习**：
   - 使用参数位置指定改变参数顺序
   - 练习重复使用同一个参数
   - 练习复杂的参数位置组合

3. **宽度和精度练习**：
   - 练习宽度控制，实现对齐
   - 练习精度控制，格式化浮点数
   - 练习宽度和精度的组合使用

4. **实际应用练习**：
   - 生成格式化的日志消息
   - 生成格式化的订单号
   - 生成对齐的表格输出
   - 格式化数值、百分比等数据

5. **综合练习**：
   - 创建一个格式化工具类
   - 实现日志格式化功能
   - 实现数据格式化功能
   - 测试不同场景下的格式化效果
