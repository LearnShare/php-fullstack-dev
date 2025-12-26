# 2.2.1 输出 API

## 概述

PHP 提供了多种输出函数，每种函数有不同的特点和适用场景。本节详细介绍 `echo`、`print`、`printf`、`sprintf` 及输出缓冲机制，帮助选择合适的输出方式。

理解不同输出函数的特点和适用场景，能够根据实际需求选择合适的输出方式，提高代码的可读性和性能。输出缓冲机制是模板渲染、邮件生成等场景的基础。

## 特性

- **echo**：语言结构，可以输出多个值，无返回值，性能最好
- **print**：语言结构，只能输出一个值，返回 1，性能次之
- **printf**：函数，格式化输出到标准输出，返回输出字符数
- **sprintf**：函数，格式化字符串并返回，不直接输出
- **输出缓冲**：可以捕获输出内容，用于模板渲染等场景

## 语法/定义

### echo - 输出语句

**语法**：`echo expression1, expression2, ...`

**特点**：
- 是语言结构（language construct），不是函数
- 可以输出多个值，用逗号分隔
- 无返回值
- 性能最好

**基本用法**：

```php
<?php
declare(strict_types=1);

// 输出单个值
echo "Hello, World!\n";

// 输出多个值
echo "Hello", " ", "World", "\n";

// 输出变量
$name = "World";
echo "Hello, {$name}!\n";
```

### print - 输出语句

**语法**：`print expression`

**特点**：
- 是语言结构（language construct），不是函数
- 只能输出一个值
- 返回 1（表示成功）
- 性能次之

**基本用法**：

```php
<?php
declare(strict_types=1);

// 输出单个值
print "Hello, World!\n";

// 不能输出多个值（语法错误）
// print "Hello", " ", "World";  // 错误
```

### printf() - 格式化输出

**语法**：`printf(string $format, mixed ...$values): int`

**参数**：
- `$format`：格式化字符串，包含占位符（如 `%s`、`%d`、`%f`）
- `...$values`：可变参数，要格式化的值

**返回值**：返回输出的字符数（类型为 `int`）

**特点**：
- 是函数，不是语言结构
- 格式化输出到标准输出
- 返回输出的字符数
- 详细说明见 [2.2.2 格式化占位符详解](section-02-format-placeholders.md)

**基本用法**：

```php
<?php
declare(strict_types=1);

// 格式化输出
$count = printf("Hello, %s!\n", "World");
echo "Output length: {$count}\n";  // Output length: 14
```

### sprintf() - 格式化字符串

**语法**：`sprintf(string $format, mixed ...$values): string`

**参数**：
- `$format`：格式化字符串，包含占位符（如 `%s`、`%d`、`%f`）
- `...$values`：可变参数，要格式化的值

**返回值**：返回格式化后的字符串（类型为 `string`），如果格式化失败返回 `false`

**特点**：
- 是函数，不是语言结构
- 格式化字符串并返回，不直接输出
- 适合需要格式化字符串用于后续处理的场景
- 详细说明见 [2.2.2 格式化占位符详解](section-02-format-placeholders.md)

**基本用法**：

```php
<?php
declare(strict_types=1);

// 格式化字符串
$message = sprintf("Hello, %s!", "World");
echo $message;  // Hello, World!
```

## 基本用法

### 示例 1：echo 基本使用

```php
<?php
declare(strict_types=1);

// 输出字符串
echo "Hello, World!\n";

// 输出多个值
echo "Hello", " ", "World", "\n";

// 输出变量
$name = "World";
echo "Hello, {$name}!\n";

// 输出表达式结果
echo "Sum: ", (10 + 20), "\n";
```

**执行**：

```bash
php example.php
```

**输出**：

```
Hello, World!
Hello World
Hello, World!
Sum: 30
```

### 示例 2：print 基本使用

```php
<?php
declare(strict_types=1);

// 输出字符串
$result = print "Hello, World!\n";
echo "Print result: {$result}\n";  // Print result: 1
```

**执行**：

```bash
php example.php
```

**输出**：

```
Hello, World!
Print result: 1
```

### 示例 3：printf 基本使用

```php
<?php
declare(strict_types=1);

// 格式化输出
$count = printf("Name: %s, Age: %d\n", "Alice", 25);
echo "Output length: {$count}\n";
```

**执行**：

```bash
php example.php
```

**输出**：

```
Name: Alice, Age: 25
Output length: 20
```

### 示例 4：sprintf 基本使用

```php
<?php
declare(strict_types=1);

// 格式化字符串
$message = sprintf("Name: %s, Age: %d", "Alice", 25);
echo $message . "\n";

// 用于后续处理
$logEntry = sprintf("[%s] %s: %s", date('Y-m-d H:i:s'), "INFO", "User logged in");
echo $logEntry . "\n";
```

**执行**：

```bash
php example.php
```

**输出**：

```
Name: Alice, Age: 25
[2024-01-15 10:30:00] INFO: User logged in
```

## 完整代码示例

### 示例 1：输出函数对比

```php
<?php
declare(strict_types=1);

$name = "World";
$age = 25;

// echo：可以输出多个值
echo "Name: ", $name, ", Age: ", $age, "\n";

// print：只能输出一个值
print "Name: {$name}, Age: {$age}\n";

// printf：格式化输出
printf("Name: %s, Age: %d\n", $name, $age);

// sprintf：格式化字符串
$message = sprintf("Name: %s, Age: %d", $name, $age);
echo $message . "\n";
```

**执行**：

```bash
php compare.php
```

**输出**：

```
Name: World, Age: 25
Name: World, Age: 25
Name: World, Age: 25
Name: World, Age: 25
```

### 示例 2：输出缓冲机制

```php
<?php
declare(strict_types=1);

// 开启输出缓冲
ob_start();

echo "This will be captured\n";
echo "Multiple lines\n";
echo "In the buffer\n";

// 获取缓冲区内容
$content = ob_get_clean();

// 输出缓冲区内容
echo "Captured content:\n";
echo $content;

// 使用输出缓冲生成内容
ob_start();
echo "<html><body>";
echo "<h1>Hello, World!</h1>";
echo "</body></html>";
$html = ob_get_clean();

// 可以保存到文件或发送邮件
file_put_contents('output.html', $html);
```

**执行**：

```bash
php buffer.php
```

**输出**：

```
Captured content:
This will be captured
Multiple lines
In the buffer
```

**说明**：
- `ob_start()`：开启输出缓冲
- `ob_get_clean()`：获取缓冲区内容并清空缓冲区
- 输出缓冲可以用于模板渲染、邮件生成等场景

### 示例 3：CLI 和 Web 环境差异

```php
<?php
declare(strict_types=1);

function isCli(): bool
{
    return php_sapi_name() === 'cli';
}

if (isCli()) {
    // CLI 模式：使用换行符
    echo "Line 1\n";
    echo "Line 2\n";
    echo "Line 3\n";
} else {
    // Web 模式：设置 HTTP 头，使用 HTML 标签
    header('Content-Type: text/html; charset=UTF-8');
    echo "Line 1<br>\n";
    echo "Line 2<br>\n";
    echo "Line 3<br>\n";
}
```

**说明**：
- CLI 模式下，`\n` 换行符有效
- Web 模式下，`\n` 换行符无效，需要使用 `<br>` 标签
- 详细说明见 [2.1.4 文件执行方式](../chapter-01-syntax/section-04-execution.md)

## 输出缓冲机制

### ob_start() - 开启输出缓冲

**语法**：`ob_start(callable|null $callback = null, int $chunk_size = 0, int $flags = PHP_OUTPUT_HANDLER_STDFLAGS): bool`

**参数**：
- `$callback`：可选，输出回调函数
- `$chunk_size`：可选，块大小
- `$flags`：可选，标志位

**返回值**：成功返回 `true`，失败返回 `false`

**作用**：开启输出缓冲，后续的输出会被捕获到缓冲区

### ob_get_clean() - 获取并清空缓冲区

**语法**：`ob_get_clean(): string|false`

**参数**：无参数

**返回值**：返回缓冲区内容（类型为 `string`），如果缓冲区为空或未开启返回 `false`

**作用**：获取缓冲区内容并清空缓冲区，关闭输出缓冲

### ob_get_contents() - 获取缓冲区内容

**语法**：`ob_get_contents(): string|false`

**参数**：无参数

**返回值**：返回缓冲区内容（类型为 `string`），如果缓冲区为空或未开启返回 `false`

**作用**：获取缓冲区内容，不清空缓冲区

### 输出缓冲示例

```php
<?php
declare(strict_types=1);

// 开启输出缓冲
ob_start();

echo "Line 1\n";
echo "Line 2\n";
echo "Line 3\n";

// 获取缓冲区内容（不清空）
$content1 = ob_get_contents();
echo "Content 1 length: " . strlen($content1) . "\n";

// 继续输出
echo "Line 4\n";

// 获取并清空缓冲区
$content2 = ob_get_clean();
echo "Content 2:\n";
echo $content2;
```

**执行**：

```bash
php buffer-example.php
```

**输出**：

```
Content 1 length: 18
Content 2:
Line 1
Line 2
Line 3
Line 4
```

## 使用场景

### echo 适用场景

- **简单输出**：大多数输出场景
- **HTML 输出**：输出 HTML 片段
- **调试输出**：快速调试输出
- **CLI 脚本**：命令行脚本的标准输出

### print 适用场景

- **需要返回值**：需要返回值的场景（较少使用）
- **单一输出**：只输出一个值的情况

### printf 适用场景

- **格式化输出**：需要格式化输出到标准输出
- **日志输出**：格式化日志消息
- **表格输出**：对齐的表格输出

### sprintf 适用场景

- **格式化字符串**：需要格式化字符串用于后续处理
- **日志格式**：生成格式化的日志消息
- **订单号**：生成格式化的订单号
- **模板渲染**：生成模板内容

### 输出缓冲适用场景

- **模板渲染**：捕获模板输出内容
- **邮件生成**：生成邮件内容
- **内容缓存**：缓存输出内容
- **内容处理**：对输出内容进行处理后再输出

## 注意事项

### 性能考虑

- **echo 性能最好**：`echo` 是语言结构，性能最好
- **print 性能次之**：`print` 也是语言结构，但性能略差
- **printf/sprintf 较慢**：函数调用有开销，格式化处理也需要时间

**性能对比示例**：

```php
<?php
declare(strict_types=1);

// 推荐：使用 echo（性能最好）
echo "Hello, ", "World", "\n";

// 不推荐：字符串连接（性能较差）
echo "Hello, " . "World" . "\n";

// 需要格式化时使用 sprintf
$message = sprintf("Hello, %s", "World");
echo $message . "\n";
```

### 返回值差异

- **echo**：无返回值
- **print**：返回 1（表示成功）
- **printf**：返回输出字符数
- **sprintf**：返回格式化后的字符串

### 格式化占位符

- **printf 和 sprintf**：支持格式化占位符（详细说明见 [2.2.2 格式化占位符详解](section-02-format-placeholders.md)）
- **占位符类型**：`%s`（字符串）、`%d`（整数）、`%f`（浮点数）等
- **参数数量**：参数数量必须足够

### 输出缓冲

- **嵌套缓冲**：可以嵌套多个输出缓冲
- **缓冲区管理**：注意及时清理缓冲区，避免内存泄漏
- **性能影响**：输出缓冲会占用内存，注意性能影响

## 常见问题

### 问题 1：echo 和 print 的区别

**问题**：什么时候使用 `echo`，什么时候使用 `print`？

**解答**：

| 特性 | echo | print |
|:-----|:-----|:------|
| 输出多个值 | 支持 | 不支持 |
| 返回值 | 无 | 返回 1 |
| 性能 | 最好 | 次之 |
| 推荐度 | 推荐 | 不推荐 |

**建议**：
- **大多数情况使用 echo**：`echo` 性能最好，功能更强大
- **需要返回值时使用 print**：但这种情况很少见
- **避免使用 print**：除非有特殊需求

### 问题 2：何时使用 sprintf

**问题**：什么时候使用 `sprintf` 而不是 `echo`？

**解答**：

**使用 sprintf 的场景**：
- 需要格式化字符串用于后续处理
- 生成日志格式、订单号等
- 需要重复使用格式化后的字符串

**使用 echo 的场景**：
- 简单输出，不需要格式化
- 直接输出到标准输出

**示例**：

```php
<?php
declare(strict_types=1);

// 使用 sprintf：需要格式化字符串用于后续处理
$logMessage = sprintf("[%s] %s: %s", date('Y-m-d H:i:s'), "INFO", "User logged in");
file_put_contents('app.log', $logMessage . "\n", FILE_APPEND);

// 使用 echo：直接输出
echo "User logged in\n";
```

### 问题 3：输出缓冲不工作

**症状**：`ob_start()` 后输出没有被捕获

**原因**：输出缓冲配置问题，或缓冲区已被清空

**解决方法**：

1. **检查配置**：检查 `php.ini` 中的 `output_buffering` 配置
2. **检查缓冲区状态**：使用 `ob_get_level()` 检查缓冲区层级
3. **正确使用**：确保在 `ob_start()` 之后才输出内容

**示例**：

```php
<?php
declare(strict_types=1);

// 检查缓冲区层级
echo "Buffer level before: " . ob_get_level() . "\n";

// 开启输出缓冲
ob_start();
echo "Buffer level after: " . ob_get_level() . "\n";

// 输出内容
echo "This will be captured\n";

// 获取缓冲区内容
$content = ob_get_clean();
echo "Captured: {$content}";
```

## 最佳实践

### 输出函数选择

- **大多数情况使用 echo**：性能最好，功能最强大
- **需要格式化时使用 sprintf**：格式化字符串用于后续处理
- **避免使用 print**：除非有特殊需求
- **printf 用于直接输出**：需要格式化输出到标准输出时使用

### 输出缓冲使用

- **模板渲染**：使用输出缓冲捕获模板输出
- **内容生成**：使用输出缓冲生成内容后处理
- **及时清理**：使用 `ob_get_clean()` 及时清理缓冲区
- **嵌套管理**：注意嵌套缓冲的管理

### 性能优化

- **使用 echo 多参数**：使用 `echo "a", "b"` 而不是 `echo "a" . "b"`
- **避免不必要的格式化**：简单输出不需要使用 `sprintf`
- **合理使用输出缓冲**：输出缓冲有内存开销，合理使用

### 代码规范

- **遵循 PSR 标准**：遵循 PSR-1 和 PSR-12 编码规范
- **使用类型声明**：为函数添加类型声明
- **添加注释**：为复杂逻辑添加注释

## 对比分析

### 输出函数对比

| 特性 | echo | print | printf | sprintf |
|:-----|:-----|:------|:-------|:---------|
| 类型 | 语言结构 | 语言结构 | 函数 | 函数 |
| 输出多个值 | 支持 | 不支持 | 支持 | 支持 |
| 返回值 | 无 | 1 | 字符数 | 字符串 |
| 格式化 | 不支持 | 不支持 | 支持 | 支持 |
| 性能 | 最好 | 次之 | 较慢 | 较慢 |
| 推荐度 | 推荐 | 不推荐 | 按需 | 按需 |

**选择建议**：
- **简单输出**：使用 `echo`
- **格式化输出**：使用 `printf` 或 `sprintf`
- **避免使用 print**：除非有特殊需求

### 输出方式对比

| 特性 | 直接输出 | 输出缓冲 |
|:-----|:---------|:---------|
| 性能 | 最好 | 有开销 |
| 灵活性 | 低 | 高 |
| 适用场景 | 简单输出 | 模板渲染、内容生成 |
| 内存占用 | 低 | 中等 |

**选择建议**：
- **简单输出**：直接使用 `echo`
- **模板渲染**：使用输出缓冲
- **内容生成**：使用输出缓冲

## 相关章节

- **2.2.2 格式化占位符详解**：详细了解 `printf` 和 `sprintf` 的格式化占位符
- **2.2.3 调试函数和策略**：了解调试函数的使用
- **2.1.4 文件执行方式**：了解 CLI 和 Web 环境的输出差异
- **阶段五：Web/API 开发**：了解 Web 环境下的输出处理

## 练习任务

1. **输出函数练习**：
   - 使用 `echo` 输出多个值
   - 使用 `printf` 格式化输出
   - 使用 `sprintf` 生成格式化字符串
   - 对比不同输出函数的性能

2. **输出缓冲练习**：
   - 使用 `ob_start()` 开启输出缓冲
   - 使用 `ob_get_clean()` 获取缓冲区内容
   - 使用输出缓冲生成 HTML 内容
   - 将生成的内容保存到文件

3. **格式化输出练习**：
   - 使用 `sprintf` 生成日志格式
   - 使用 `sprintf` 生成订单号
   - 使用 `sprintf` 格式化表格输出
   - 练习使用不同的格式化占位符

4. **环境差异练习**：
   - 在 CLI 和 Web 两种模式下执行相同的代码
   - 观察输出差异
   - 编写兼容两种模式的代码

5. **综合练习**：
   - 创建一个输出工具类，封装不同的输出方法
   - 实现模板渲染功能，使用输出缓冲
   - 实现日志记录功能，使用格式化输出
   - 测试不同场景下的输出效果
