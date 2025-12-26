# 2.2.3 调试函数和策略

## 概述

调试是开发过程中必不可少的技能。本节详细介绍 `var_dump`、`print_r`、`var_export`、`debug_backtrace`、`error_log` 等调试函数，以及开发/生产环境的调试策略（仅 CLI 环境）。

理解调试函数的使用场景和调试策略，能够快速定位问题，提高开发效率。建立"开发输出 vs 生产日志"的思维，避免在生产环境泄漏调试信息。

## 特性

- **详细信息**：`var_dump` 显示变量的类型和值
- **可读格式**：`print_r` 以可读格式显示变量
- **可执行输出**：`var_export` 输出可执行的 PHP 代码
- **堆栈跟踪**：`debug_backtrace` 显示函数调用链
- **日志记录**：`error_log` 记录错误和调试信息
- **环境区分**：开发和生产环境使用不同的调试策略

## 语法/定义

### var_dump() - 详细输出变量

**语法**：`var_dump(mixed ...$values): void`

**参数**：
- `...$values`：可变参数，要输出的变量（可传入多个变量）

**返回值**：无返回值（`void`），直接输出到标准输出

**特点**：
- 显示变量的类型、长度、层级结构
- 递归显示数组和对象的完整内容
- 输出格式详细但不易阅读
- 适合查看变量的完整信息

### print_r() - 可读格式输出

**语法**：`print_r(mixed $value, bool $return = false): string|bool`

**参数**：
- `$value`：要输出的变量（任意类型）
- `$return`：可选，如果为 `true`，返回字符串而不是直接输出，默认为 `false`

**返回值**：
- 如果 `$return` 为 `false`：返回 `true`（表示成功）
- 如果 `$return` 为 `true`：返回格式化的字符串（类型为 `string`）

**特点**：
- 输出格式更简洁易读
- 适合查看数组和对象的基本结构
- 不显示类型信息
- 可以返回字符串用于后续处理

### var_export() - 可执行输出

**语法**：`var_export(mixed $value, bool $return = false): ?string`

**参数**：
- `$value`：要输出的变量（任意类型）
- `$return`：可选，如果为 `true`，返回字符串而不是直接输出，默认为 `false`

**返回值**：
- 如果 `$return` 为 `false`：返回 `null`
- 如果 `$return` 为 `true`：返回格式化的字符串（有效的 PHP 代码），类型为 `string`

**特点**：
- 输出格式是有效的 PHP 代码
- 可以用于生成配置文件、缓存数据等
- 输出格式较详细

### debug_backtrace() - 回溯跟踪

**语法**：`debug_backtrace(int $options = DEBUG_BACKTRACE_PROVIDE_OBJECT, int $limit = 0): array`

**参数**：
- `$options`：可选，控制回溯信息的详细程度，默认为 `DEBUG_BACKTRACE_PROVIDE_OBJECT`
- `$limit`：可选，限制回溯的深度，0 表示不限制，默认为 0

**返回值**：返回回溯跟踪数组（类型为 `array`），包含函数调用链信息

**特点**：
- 显示函数调用链
- 包含文件名、行号、函数名等信息
- 可以用于调试和错误追踪

### error_log() - 记录错误日志

**语法**：`error_log(string $message, int $message_type = 0, ?string $destination = null, ?string $additional_headers = null): bool`

**参数**：
- `$message`：要记录的消息（类型为 `string`）
- `$message_type`：可选，消息类型，默认为 0（系统日志）
- `$destination`：可选，目标位置（文件路径或邮件地址）
- `$additional_headers`：可选，额外的邮件头（仅用于邮件类型）

**返回值**：成功返回 `true`，失败返回 `false`

**特点**：
- 记录错误和调试信息到日志文件
- 支持多种消息类型（系统日志、文件、邮件等）
- 适合生产环境的调试和错误追踪

## 基本用法

### 示例 1：var_dump 基本使用

```php
<?php
declare(strict_types=1);

// 标量类型
$name = "World";
$age = 25;
$price = 99.99;
$active = true;

var_dump($name);   // string(5) "World"
var_dump($age);    // int(25)
var_dump($price);  // float(99.99)
var_dump($active); // bool(true)

// 数组
$user = [
    'id' => 1,
    'name' => 'Alice',
    'tags' => ['php', 'mysql'],
];

var_dump($user);
// array(3) {
//   ["id"]=>
//   int(1)
//   ["name"]=>
//   string(5) "Alice"
//   ["tags"]=>
//   array(2) {
//     [0]=>
//     string(3) "php"
//     [1]=>
//     string(5) "mysql"
//   }
// }

// 多个值
var_dump($name, $age, $price);
```

**执行**：

```bash
php var-dump-example.php
```

**输出**：

```
string(5) "World"
int(25)
float(99.99)
bool(true)
array(3) {
  ["id"]=>
  int(1)
  ["name"]=>
  string(5) "Alice"
  ["tags"]=>
  array(2) {
    [0]=>
    string(3) "php"
    [1]=>
    string(5) "mysql"
  }
}
string(5) "World"
int(25)
float(99.99)
```

### 示例 2：print_r 基本使用

```php
<?php
declare(strict_types=1);

$data = [
    'id' => 1,
    'name' => 'Alice',
    'tags' => ['php', 'mysql'],
];

// 直接输出
print_r($data);
// Array
// (
//     [id] => 1
//     [name] => Alice
//     [tags] => Array
//         (
//             [0] => php
//             [1] => mysql
//         )
// )

// 返回字符串
$output = print_r($data, true);
echo "Output:\n{$output}\n";

// 保存到文件
file_put_contents('debug.log', $output);
```

**执行**：

```bash
php print-r-example.php
```

**输出**：

```
Array
(
    [id] => 1
    [name] => Alice
    [tags] => Array
        (
            [0] => php
            [1] => mysql
        )
)
Output:
Array
(
    [id] => 1
    [name] => Alice
    [tags] => Array
        (
            [0] => php
            [1] => mysql
        )
)
```

### 示例 3：var_export 基本使用

```php
<?php
declare(strict_types=1);

$data = [
    'id' => 1,
    'name' => 'Alice',
    'tags' => ['php', 'mysql'],
];

// 直接输出
var_export($data);
// array (
//   'id' => 1,
//   'name' => 'Alice',
//   'tags' => 
//   array (
//     0 => 'php',
//     1 => 'mysql',
//   ),
// )

// 返回字符串（可执行的 PHP 代码）
$code = var_export($data, true);
echo "Code:\n{$code}\n";

// 可以用于生成配置文件
$config = "<?php\nreturn {$code};\n";
file_put_contents('config.php', $config);
```

**执行**：

```bash
php var-export-example.php
```

**输出**：

```
array (
  'id' => 1,
  'name' => 'Alice',
  'tags' => 
  array (
    0 => 'php',
    1 => 'mysql',
  ),
)
Code:
array (
  'id' => 1,
  'name' => 'Alice',
  'tags' => 
  array (
    0 => 'php',
    1 => 'mysql',
  ),
)
```

### 示例 4：debug_backtrace 基本使用

```php
<?php
declare(strict_types=1);

function functionA(): void
{
    functionB();
}

function functionB(): void
{
    functionC();
}

function functionC(): void
{
    $trace = debug_backtrace();
    print_r($trace);
}

functionA();
```

**执行**：

```bash
php backtrace-example.php
```

**输出**：

```
Array
(
    [0] => Array
        (
            [file] => /path/to/backtrace-example.php
            [line] => 15
            [function] => debug_backtrace
        )
    [1] => Array
        (
            [file] => /path/to/backtrace-example.php
            [line] => 12
            [function] => functionC
        )
    [2] => Array
        (
            [file] => /path/to/backtrace-example.php
            [line] => 7
            [function] => functionB
        )
    [3] => Array
        (
            [file] => /path/to/backtrace-example.php
            [line] => 19
            [function] => functionA
        )
)
```

### 示例 5：error_log 基本使用

```php
<?php
declare(strict_types=1);

// 记录到系统日志
error_log("This is a test message");

// 记录到文件
error_log("This is a test message", 3, "debug.log");

// 记录调试信息
$data = ['id' => 1, 'name' => 'Alice'];
error_log(print_r($data, true), 3, "debug.log");
```

**执行**：

```bash
php error-log-example.php
```

**说明**：
- 第一条消息记录到系统日志
- 第二条和第三条消息记录到 `debug.log` 文件

## 完整代码示例

### 示例 1：调试函数对比

```php
<?php
declare(strict_types=1);

$data = [
    'id' => 1,
    'name' => 'Alice',
    'tags' => ['php', 'mysql'],
];

echo "=== var_dump ===\n";
var_dump($data);

echo "\n=== print_r ===\n";
print_r($data);

echo "\n=== var_export ===\n";
var_export($data);
echo "\n";
```

**执行**：

```bash
php compare-debug.php
```

**输出**：

```
=== var_dump ===
array(3) {
  ["id"]=>
  int(1)
  ["name"]=>
  string(5) "Alice"
  ["tags"]=>
  array(2) {
    [0]=>
    string(3) "php"
    [1]=>
    string(5) "mysql"
  }
}

=== print_r ===
Array
(
    [id] => 1
    [name] => Alice
    [tags] => Array
        (
            [0] => php
            [1] => mysql
        )
)

=== var_export ===
array (
  'id' => 1,
  'name' => 'Alice',
  'tags' => 
  array (
    0 => 'php',
    1 => 'mysql',
  ),
)
```

### 示例 2：调试工具函数

```php
<?php
declare(strict_types=1);

function debug(mixed $value, string $label = ''): void
{
    if ($label !== '') {
        echo "=== {$label} ===\n";
    }
    var_dump($value);
    echo "\n";
}

function debugTrace(string $label = ''): void
{
    if ($label !== '') {
        echo "=== {$label} ===\n";
    }
    $trace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 5);
    print_r($trace);
    echo "\n";
}

// 使用调试函数
$data = ['id' => 1, 'name' => 'Alice'];
debug($data, "User Data");

function test(): void
{
    debugTrace("Function Call Trace");
}

test();
```

**执行**：

```bash
php debug-tools.php
```

**输出**：

```
=== User Data ===
array(2) {
  ["id"]=>
  int(1)
  ["name"]=>
  string(5) "Alice"
}

=== Function Call Trace ===
Array
(
    [0] => Array
        (
            [file] => /path/to/debug-tools.php
            [line] => 25
            [function] => debugTrace
        )
    [1] => Array
        (
            [file] => /path/to/debug-tools.php
            [line] => 30
            [function] => test
        )
)
```

### 示例 3：开发/生产环境调试策略

```php
<?php
declare(strict_types=1);

function isDevelopment(): bool
{
    return ($_ENV['APP_ENV'] ?? 'production') === 'development';
}

function debugLog(mixed $value, string $label = ''): void
{
    if (!isDevelopment()) {
        return;  // 生产环境不输出调试信息
    }
    
    if ($label !== '') {
        echo "[DEBUG] {$label}:\n";
    }
    var_dump($value);
    echo "\n";
}

function productionLog(string $message, array $context = []): void
{
    $logMessage = sprintf(
        "[%s] %s %s\n",
        date('Y-m-d H:i:s'),
        $message,
        !empty($context) ? json_encode($context) : ''
    );
    error_log($logMessage, 3, "app.log");
}

// 开发环境：使用调试函数
debugLog(['id' => 1, 'name' => 'Alice'], "User Data");

// 生产环境：使用日志记录
productionLog("User logged in", ['user_id' => 123]);
```

**说明**：
- 开发环境：使用 `debugLog` 输出调试信息
- 生产环境：使用 `productionLog` 记录日志到文件
- 避免在生产环境泄漏调试信息

## 使用场景

### var_dump 适用场景

- **快速查看变量信息**：查看变量的类型和值
- **调试复杂数据结构**：查看数组和对象的完整结构
- **CLI 环境调试**：在命令行环境下调试

### print_r 适用场景

- **快速查看数组结构**：查看数组和对象的基本结构
- **保存调试信息**：将调试信息保存到文件
- **日志记录**：与 `error_log` 结合使用

### var_export 适用场景

- **生成配置文件**：生成 PHP 配置文件
- **缓存数据**：将数据导出为可执行的 PHP 代码
- **代码生成**：生成包含数据的 PHP 代码

### debug_backtrace 适用场景

- **错误追踪**：追踪函数调用链
- **调试复杂逻辑**：理解代码执行流程
- **性能分析**：分析函数调用关系

### error_log 适用场景

- **生产环境日志**：在生产环境记录错误和调试信息
- **错误追踪**：追踪错误发生的位置和上下文
- **系统监控**：监控系统运行状态

## 注意事项

### 仅 CLI 环境

- **本节仅介绍 CLI 环境的调试**：Web 环境的调试将在后续章节介绍
- **CLI 环境输出更直观**：命令行环境下输出格式更清晰
- **Web 环境需要特殊处理**：Web 环境需要使用 `<pre>` 标签等特殊处理

### 生产环境

- **不要在生产环境使用调试函数**：避免泄漏敏感信息
- **使用日志记录**：使用 `error_log` 或日志库记录信息
- **控制日志级别**：根据环境控制日志的详细程度

### 性能影响

- **调试代码可能影响性能**：调试函数有性能开销
- **及时移除调试代码**：调试完成后及时移除调试代码
- **使用条件编译**：使用条件判断控制调试代码的执行

### 信息安全

- **注意不要输出敏感信息**：避免输出密码、密钥等敏感信息
- **控制调试信息范围**：只输出必要的调试信息
- **使用日志级别**：根据重要性使用不同的日志级别

## 常见问题

### 问题 1：var_dump 输出格式混乱

**症状**：在 Web 环境下，`var_dump` 输出格式混乱

**原因**：Web 环境下 HTML 会破坏输出格式

**解决方法**：

1. **CLI 环境**：在 CLI 环境下使用，输出格式正常
2. **Web 环境**：使用 `<pre>` 标签包装（但本节仅介绍 CLI 环境）

**示例**（仅 CLI 环境）：

```php
<?php
declare(strict_types=1);

// CLI 环境：直接使用
$data = ['id' => 1, 'name' => 'Alice'];
var_dump($data);
```

### 问题 2：调试信息过多

**症状**：`debug_backtrace` 输出信息过多，难以阅读

**原因**：调用链很深，或没有限制回溯深度

**解决方法**：

1. **限制回溯深度**：使用 `$limit` 参数限制深度
2. **过滤信息**：只显示必要的信息

**示例**：

```php
<?php
declare(strict_types=1);

// 限制回溯深度
$trace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 3);
print_r($trace);

// 只显示函数名和行号
foreach ($trace as $frame) {
    echo sprintf(
        "%s:%d %s()\n",
        basename($frame['file'] ?? 'unknown'),
        $frame['line'] ?? 0,
        $frame['function'] ?? 'unknown'
    );
}
```

### 问题 3：生产环境泄漏调试信息

**症状**：生产环境输出了调试信息

**原因**：调试代码没有移除或没有条件判断

**解决方法**：

1. **使用环境判断**：根据环境决定是否输出调试信息
2. **使用日志记录**：使用 `error_log` 记录信息
3. **代码审查**：提交前检查是否有调试代码

**示例**：

```php
<?php
declare(strict_types=1);

function isDevelopment(): bool
{
    return ($_ENV['APP_ENV'] ?? 'production') === 'development';
}

// 只在开发环境输出
if (isDevelopment()) {
    var_dump($data);
} else {
    // 生产环境：记录日志
    error_log(print_r($data, true), 3, "debug.log");
}
```

## 最佳实践

### 调试函数选择

- **快速查看变量**：使用 `var_dump` 或 `print_r`
- **生成配置文件**：使用 `var_export`
- **追踪函数调用**：使用 `debug_backtrace`
- **生产环境日志**：使用 `error_log`

### 开发环境

- **使用调试函数**：在开发环境使用调试函数快速定位问题
- **格式化输出**：使用格式化输出提高可读性
- **及时清理**：调试完成后及时清理调试代码

### 生产环境

- **使用日志记录**：使用 `error_log` 或日志库记录信息
- **控制日志级别**：根据重要性使用不同的日志级别
- **避免输出调试信息**：不要在生产环境输出调试信息

### 代码组织

- **创建调试工具函数**：封装常用的调试函数
- **环境判断**：根据环境决定是否执行调试代码
- **代码审查**：提交前检查是否有调试代码

## 对比分析

### 调试函数对比

| 特性 | var_dump | print_r | var_export | debug_backtrace |
|:-----|:---------|:--------|:-----------|:-----------------|
| 类型信息 | 显示 | 不显示 | 不显示 | 不显示 |
| 可读性 | 低 | 高 | 中 | 中 |
| 可执行 | 否 | 否 | 是 | 否 |
| 返回值 | 无 | 可选 | 可选 | 是 |
| 适用场景 | 详细调试 | 快速查看 | 代码生成 | 调用追踪 |

**选择建议**：
- **详细调试**：使用 `var_dump`
- **快速查看**：使用 `print_r`
- **代码生成**：使用 `var_export`
- **调用追踪**：使用 `debug_backtrace`

### 开发 vs 生产环境

| 特性 | 开发环境 | 生产环境 |
|:-----|:---------|:---------|
| 调试函数 | 使用 | 不使用 |
| 日志记录 | 可选 | 必须 |
| 输出调试信息 | 可以 | 不可以 |
| 日志级别 | 详细 | 精简 |

**选择建议**：
- **开发环境**：使用调试函数快速定位问题
- **生产环境**：使用日志记录，避免输出调试信息

## 相关章节

- **2.2.1 输出 API**：了解基本输出函数
- **2.2.2 格式化占位符详解**：了解格式化输出
- **阶段四：系统编程**：详细了解错误处理和日志记录
- **阶段四：调试与问题排查**：深入学习调试技巧和工具

## 练习任务

1. **调试函数练习**：
   - 使用 `var_dump` 查看不同类型变量的信息
   - 使用 `print_r` 查看数组和对象结构
   - 使用 `var_export` 生成可执行的 PHP 代码
   - 对比不同调试函数的输出

2. **回溯跟踪练习**：
   - 使用 `debug_backtrace` 追踪函数调用链
   - 限制回溯深度，只显示必要信息
   - 格式化回溯信息，提高可读性

3. **日志记录练习**：
   - 使用 `error_log` 记录调试信息到文件
   - 创建日志工具函数，封装日志记录逻辑
   - 实现不同日志级别的记录

4. **环境区分练习**：
   - 创建开发/生产环境的调试函数
   - 根据环境决定是否输出调试信息
   - 实现环境判断和日志记录

5. **综合练习**：
   - 创建一个调试工具类
   - 实现多种调试方法
   - 实现环境判断和日志记录
   - 测试不同场景下的调试效果
