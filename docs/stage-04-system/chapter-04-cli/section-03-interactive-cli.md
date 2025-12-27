# 4.4.3 交互式 CLI 与输出格式化

## 概述

交互式 CLI 程序能够与用户进行实时交互，输出格式化能够提升用户体验。在实际应用中，我们经常需要开发交互式工具、配置向导、数据输入工具等，这些都需要处理用户输入和格式化输出。PHP 提供了多种方式来实现交互式 CLI 程序，包括 `readline()`、`fgets()`、ANSI 转义码等。

理解交互式输入处理、输出格式化、颜色输出、进度条等技术的使用方法，对于开发用户友好的 CLI 工具至关重要。

**主要内容**：
- 交互式输入处理（`readline()`、`fgets(STDIN)`）
- 输出格式化（文本对齐、表格输出）
- 颜色输出（ANSI 转义码）
- 进度条实现
- 表格输出
- 跨平台兼容性
- 实际应用场景和最佳实践

## 特性

- **交互式输入**：支持从标准输入读取用户输入
- **输出格式化**：支持文本对齐、表格、多行输出等
- **颜色输出**：支持 ANSI 转义码实现颜色和样式
- **进度显示**：支持进度条和状态显示
- **跨平台**：需要考虑不同平台的兼容性

## 语法/定义

### readline() 函数

**语法**：`readline(?string $prompt = null): string|false`

**参数**：
- `$prompt`：可选，提示信息

**返回值**：成功返回用户输入的一行文本，失败返回 `false`。

**注意**：需要启用 `readline` 扩展。

### fgets() 函数（从 STDIN 读取）

**语法**：`fgets(resource $stream, ?int $length = null): string|false`

**参数**：
- `$stream`：文件流（使用 `STDIN` 常量）
- `$length`：可选，读取的最大字节数

**返回值**：成功返回读取的一行文本，失败返回 `false`。

## 基本用法

### 示例 1：使用 readline() 读取输入

```php
<?php
declare(strict_types=1);

// 检查 readline 扩展是否可用
if (!function_exists('readline')) {
    die("readline extension is not available\n");
}

// 读取用户输入
$name = readline('请输入您的姓名: ');
if ($name !== false) {
    echo "您好, {$name}!\n";
}

// 读取多个输入
$age = readline('请输入您的年龄: ');
$city = readline('请输入您的城市: ');

echo "姓名: {$name}, 年龄: {$age}, 城市: {$city}\n";
```

**说明**：
- `readline()` 提供更好的输入体验（支持历史记录、编辑等）
- 需要启用 `readline` 扩展
- 返回用户输入的一行文本

### 示例 2：使用 fgets() 从 STDIN 读取

```php
<?php
declare(strict_types=1);

// 从标准输入读取（不依赖 readline 扩展）
echo "请输入您的姓名: ";
$name = fgets(STDIN);
if ($name !== false) {
    $name = trim($name);  // 去除换行符
    echo "您好, {$name}!\n";
}
```

**说明**：
- `fgets(STDIN)` 不依赖扩展，更通用
- 需要手动去除换行符（`trim()`）
- 读取到的一行包含换行符

### 示例 3：输入验证

```php
<?php
declare(strict_types=1);

function readInteger(string $prompt, ?int $min = null, ?int $max = null): int
{
    while (true) {
        echo $prompt;
        $input = fgets(STDIN);
        if ($input === false) {
            throw new RuntimeException('Cannot read input');
        }
        
        $value = trim($input);
        if (!is_numeric($value)) {
            echo "错误: 请输入数字\n";
            continue;
        }
        
        $intValue = (int)$value;
        
        if ($min !== null && $intValue < $min) {
            echo "错误: 值不能小于 {$min}\n";
            continue;
        }
        
        if ($max !== null && $intValue > $max) {
            echo "错误: 值不能大于 {$max}\n";
            continue;
        }
        
        return $intValue;
    }
}

// 使用
$age = readInteger('请输入年龄 (0-150): ', 0, 150);
echo "年龄: {$age}\n";
```

**说明**：
- 实现输入验证，确保输入符合要求
- 使用循环直到输入有效

### 示例 4：颜色输出（ANSI 转义码）

```php
<?php
declare(strict_types=1);

// ANSI 颜色代码
class Colors
{
    public const RESET = "\033[0m";
    public const BOLD = "\033[1m";
    
    // 前景色
    public const BLACK = "\033[30m";
    public const RED = "\033[31m";
    public const GREEN = "\033[32m";
    public const YELLOW = "\033[33m";
    public const BLUE = "\033[34m";
    public const MAGENTA = "\033[35m";
    public const CYAN = "\033[36m";
    public const WHITE = "\033[37m";
    
    // 背景色
    public const BG_BLACK = "\033[40m";
    public const BG_RED = "\033[41m";
    public const BG_GREEN = "\033[42m";
    public const BG_YELLOW = "\033[43m";
    public const BG_BLUE = "\033[44m";
    public const BG_MAGENTA = "\033[45m";
    public const BG_CYAN = "\033[46m";
    public const BG_WHITE = "\033[47m";
}

// 使用颜色
echo Colors::RED . "错误信息" . Colors::RESET . "\n";
echo Colors::GREEN . "成功信息" . Colors::RESET . "\n";
echo Colors::YELLOW . "警告信息" . Colors::RESET . "\n";
echo Colors::BLUE . "信息" . Colors::RESET . "\n";

// 组合样式
echo Colors::BOLD . Colors::RED . "粗体红色文本" . Colors::RESET . "\n";
```

**说明**：
- ANSI 转义码用于控制终端颜色和样式
- `\033[0m` 重置所有样式
- 颜色代码在 Unix/Linux/macOS 上工作，Windows 需要额外处理

### 示例 5：检测终端是否支持颜色

```php
<?php
declare(strict_types=1);

function supportsColors(): bool
{
    // Windows 10+ 支持 ANSI，但需要检测
    if (PHP_OS_FAMILY === 'Windows') {
        // Windows 10+ (1607) 支持 ANSI
        return getenv('ANSICON') !== false || 
               getenv('ConEmuANSI') === 'ON' ||
               getenv('TERM') === 'xterm';
    }
    
    // Unix/Linux/macOS
    return function_exists('posix_isatty') && posix_isatty(STDOUT);
}

// 使用
if (supportsColors()) {
    echo Colors::GREEN . "支持颜色输出\n" . Colors::RESET;
} else {
    echo "不支持颜色输出\n";
}
```

**说明**：
- 检测终端是否支持颜色输出
- 在支持时使用颜色，不支持时使用普通文本

### 示例 6：进度条实现

```php
<?php
declare(strict_types=1);

function showProgress(int $current, int $total, int $barWidth = 50): void
{
    $percent = ($current / $total) * 100;
    $filled = (int)(($current / $total) * $barWidth);
    $empty = $barWidth - $filled;
    
    $bar = str_repeat('=', $filled) . str_repeat(' ', $empty);
    
    // 使用 \r 回到行首，实现动态更新
    printf("\r[%s] %d/%d (%.1f%%)", $bar, $current, $total, $percent);
    
    if ($current >= $total) {
        echo "\n";  // 完成后换行
    }
}

// 模拟处理过程
$total = 100;
for ($i = 1; $i <= $total; $i++) {
    showProgress($i, $total);
    usleep(50000);  // 延迟 50ms，模拟处理
}
```

**说明**：
- 使用 `\r` 回到行首，实现进度条动态更新
- 显示百分比和进度条

### 示例 7：表格输出

```php
<?php
declare(strict_types=1);

function printTable(array $headers, array $rows): void
{
    // 计算列宽
    $widths = [];
    foreach ($headers as $index => $header) {
        $widths[$index] = strlen($header);
    }
    
    foreach ($rows as $row) {
        foreach ($row as $index => $cell) {
            $width = strlen((string)$cell);
            if ($width > ($widths[$index] ?? 0)) {
                $widths[$index] = $width;
            }
        }
    }
    
    // 打印表头
    $headerLine = '';
    foreach ($headers as $index => $header) {
        $headerLine .= str_pad($header, $widths[$index] + 2);
    }
    echo $headerLine . "\n";
    echo str_repeat('-', strlen($headerLine)) . "\n";
    
    // 打印数据行
    foreach ($rows as $row) {
        $rowLine = '';
        foreach ($row as $index => $cell) {
            $rowLine .= str_pad((string)$cell, $widths[$index] + 2);
        }
        echo $rowLine . "\n";
    }
}

// 使用
$headers = ['姓名', '年龄', '城市'];
$rows = [
    ['Alice', 25, '北京'],
    ['Bob', 30, '上海'],
    ['Charlie', 28, '广州'],
];

printTable($headers, $rows);
```

**说明**：
- 实现简单的表格输出
- 自动计算列宽，确保对齐

### 示例 8：使用 printf() 格式化输出

```php
<?php
declare(strict_types=1);

// 格式化输出
$name = 'Alice';
$age = 25;
$score = 95.5;

// 左对齐
printf("%-10s %5d %8.2f\n", $name, $age, $score);

// 表格格式化
printf("%-15s %8s %10s\n", "Name", "Age", "Score");
echo str_repeat("-", 35) . "\n";
printf("%-15s %8d %10.2f\n", "Alice", 25, 95.5);
printf("%-15s %8d %10.2f\n", "Bob", 30, 88.3);
```

**说明**：
- 使用 `printf()` 进行格式化输出
- `%-10s` 表示左对齐的字符串，宽度 10
- `%5d` 表示整数，宽度 5
- `%8.2f` 表示浮点数，总宽度 8，小数 2 位

## 使用场景

### 场景 1：配置向导

创建交互式配置向导。

**示例**：

```php
<?php
declare(strict_types=1);

echo "=== 应用配置向导 ===\n\n";

$config = [];

echo "请输入数据库主机: ";
$config['db_host'] = trim(fgets(STDIN));

echo "请输入数据库端口 (默认 3306): ";
$port = trim(fgets(STDIN));
$config['db_port'] = $port !== '' ? (int)$port : 3306;

echo "请输入数据库名称: ";
$config['db_name'] = trim(fgets(STDIN));

echo "\n配置完成:\n";
print_r($config);
```

### 场景 2：交互式菜单

创建交互式菜单。

**示例**：

```php
<?php
declare(strict_types=1);

function showMenu(): void
{
    echo "请选择操作:\n";
    echo "1. 查看用户\n";
    echo "2. 添加用户\n";
    echo "3. 删除用户\n";
    echo "0. 退出\n";
    echo "请输入选项: ";
}

while (true) {
    showMenu();
    $choice = trim(fgets(STDIN));
    
    match ($choice) {
        '1' => echo "查看用户\n",
        '2' => echo "添加用户\n",
        '3' => echo "删除用户\n",
        '0' => exit(0),
        default => echo "无效选项\n",
    };
    
    echo "\n";
}
```

## 注意事项

### readline 扩展的可用性

`readline()` 需要启用 `readline` 扩展，在生产环境可能不可用。

**示例**：

```php
<?php
declare(strict_types=1);

if (function_exists('readline')) {
    $input = readline('请输入: ');
} else {
    echo "请输入: ";
    $input = trim(fgets(STDIN));
}
```

### 跨平台兼容性

ANSI 颜色代码在 Windows 上可能需要额外处理。

**示例**：见"示例 5：检测终端是否支持颜色"

### 输入验证

始终验证用户输入，确保数据有效性。

**示例**：见"示例 3：输入验证"

## 常见问题

### 问题 1：如何实现交互式输入？

**回答**：使用 `readline()` 或 `fgets(STDIN)` 从标准输入读取。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法1：使用 readline（需要扩展）
$input = readline('请输入: ');

// 方法2：使用 fgets（不需要扩展）
echo "请输入: ";
$input = trim(fgets(STDIN));
```

### 问题 2：如何输出彩色文本？

**回答**：使用 ANSI 转义码，但需要检测终端是否支持。

**示例**：见"示例 4：颜色输出（ANSI 转义码）"

### 问题 3：如何实现进度条？

**回答**：使用 `\r` 回到行首，动态更新进度条。

**示例**：见"示例 6：进度条实现"

### 问题 4：Windows 下如何处理颜色？

**回答**：Windows 10+ 支持 ANSI，但需要检测。可以使用 `symfony/console` 等库处理跨平台兼容性。

**示例**：

```php
<?php
declare(strict_types=1);

// Windows 10+ 支持 ANSI，但建议使用库处理兼容性
// 推荐使用 symfony/console 组件
```

## 最佳实践

### 1. 检测终端能力

在使用颜色或特殊功能前，检测终端是否支持。

**示例**：见"示例 5：检测终端是否支持颜色"

### 2. 提供清晰的提示信息

提示信息应该清晰、有用。

**示例**：

```php
<?php
declare(strict_types=1);

echo "请输入文件名 (按 Enter 使用默认值 'data.txt'): ";
```

### 3. 验证用户输入

始终验证用户输入，确保数据有效性。

**示例**：见"示例 3：输入验证"

### 4. 考虑使用第三方库

对于复杂的 CLI 界面，考虑使用 `symfony/console` 等库。

**示例**：

```php
<?php
declare(strict_types=1);

// 使用 Symfony Console 组件可以提供更好的 CLI 体验
// composer require symfony/console
```

### 5. 提供默认值

对于可选输入，提供默认值。

**示例**：

```php
<?php
declare(strict_types=1);

echo "请输入端口 (默认 3306): ";
$port = trim(fgets(STDIN));
$port = $port !== '' ? (int)$port : 3306;
```

## 对比分析

### readline() vs fgets(STDIN)

| 特性         | readline()                   | fgets(STDIN)                 |
|:-------------|:-----------------------------|:-----------------------------|
| **扩展要求** | ⚠️ 需要 readline 扩展        | ✅ 不需要扩展                |
| **功能**     | ✅ 支持历史记录、编辑等      | ⚠️ 基础功能                  |
| **可用性**   | ⚠️ 可能在生产环境不可用      | ✅ 广泛可用                  |
| **推荐使用** | 开发环境                     | 生产环境                     |

## 练习任务

1. **交互式输入工具类**：创建一个工具类，封装交互式输入处理功能。

2. **颜色输出工具**：实现一个工具，提供跨平台的颜色输出功能。

3. **进度条工具**：创建一个可复用的进度条工具类。

4. **表格输出工具**：编写一个工具，格式化输出表格数据。

5. **交互式配置向导**：开发一个完整的交互式配置向导工具。

## 相关章节

- **[4.4.1 CLI 基础与参数处理](section-01-cli-basics.md)**：了解 CLI 编程基础
- **[4.4.5 CLI 工具开发框架](section-05-cli-frameworks.md)**：学习使用 CLI 框架
- **[4.4.7 CLI 最佳实践](section-07-best-practices.md)**：了解 CLI 最佳实践
