# 4.4.1 CLI 基础与参数处理

## 概述

CLI（命令行接口）模式是 PHP 的重要执行模式之一。与 Web 模式不同，CLI 模式在命令行终端中执行，适合开发命令行工具、批处理脚本、系统管理工具等。理解 CLI 模式的特点、如何检测 CLI 模式，以及如何处理命令行参数，是进行 CLI 编程的基础。

在实际应用中，我们经常需要开发命令行工具来处理各种任务，如数据迁移、定时任务、批处理等。掌握 CLI 编程基础，能够帮助我们开发更强大的命令行工具。

**主要内容**：
- CLI 模式概述和特点
- CLI 模式检测（`php_sapi_name()`）
- 命令行参数获取（`$argv`、`$argc`）
- 参数解析方法
- `getopt()` 函数（选项处理）
- 短选项和长选项
- 参数验证和错误处理
- 实际应用场景和最佳实践

## 特性

- **命令行执行**：在终端中直接执行，不依赖 Web 服务器
- **参数传递**：支持通过命令行传递参数和选项
- **标准输入输出**：使用标准输入（STDIN）、标准输出（STDOUT）、标准错误（STDERR）
- **退出码**：支持返回退出码，表示执行结果
- **交互式**：支持交互式输入和输出

## 语法/定义

### php_sapi_name() 函数

**语法**：`php_sapi_name(): string`

**参数**：无参数

**返回值**：返回当前 PHP 运行环境的 SAPI 名称。CLI 模式返回 `'cli'`。

### getopt() 函数

**语法**：`getopt(string $short_options, array $long_options = [], array &$rest_index = null): array|false`

**参数**：
- `$short_options`：短选项字符串，如 `'f:h'` 表示 `-f` 需要参数，`-h` 不需要参数
- `$long_options`：可选，长选项数组，如 `['file:', 'help']` 表示 `--file` 需要参数，`--help` 不需要参数
- `&$rest_index`：可选，用于接收剩余参数的起始索引

**返回值**：成功返回包含选项的关联数组，失败返回 `false`。

## 基本用法

### 示例 1：检测 CLI 模式

```php
<?php
declare(strict_types=1);

// 检测是否在 CLI 模式运行
if (php_sapi_name() !== 'cli') {
    die("This script must be run from the command line.\n");
}

echo "Running in CLI mode\n";
```

**说明**：
- `php_sapi_name()` 返回当前运行环境的 SAPI 名称
- CLI 模式返回 `'cli'`
- 可以在脚本开头检测，确保只在 CLI 模式运行

### 示例 2：获取命令行参数（$argv 和 $argc）

```php
<?php
declare(strict_types=1);

// $argv 是包含所有命令行参数的数组
// $argc 是参数数量（包括脚本名称）

echo "参数数量: {$argc}\n";
echo "参数列表:\n";
foreach ($argv as $index => $arg) {
    echo "  [{$index}] {$arg}\n";
}

// 执行: php script.php arg1 arg2 arg3
// 输出:
// 参数数量: 4
// 参数列表:
//   [0] script.php
//   [1] arg1
//   [2] arg2
//   [3] arg3
```

**说明**：
- `$argv[0]` 是脚本名称
- `$argv[1]` 是第一个参数
- `$argc` 是参数总数（包括脚本名称）

### 示例 3：简单参数解析

```php
<?php
declare(strict_types=1);

// 简单参数解析
if ($argc < 2) {
    echo "用法: php script.php <命令> [参数...]\n";
    exit(1);
}

$command = $argv[1];
$args = array_slice($argv, 2);

echo "命令: {$command}\n";
echo "参数: " . implode(', ', $args) . "\n";

// 执行: php script.php greet John
// 输出:
// 命令: greet
// 参数: John
```

**说明**：
- 检查参数数量，确保有足够的参数
- 使用 `array_slice()` 获取剩余参数
- 使用 `exit(1)` 表示错误退出

### 示例 4：使用 getopt() 处理短选项

```php
<?php
declare(strict_types=1);

// 短选项：-f 需要参数，-h 不需要参数
$options = getopt('f:h');

if (isset($options['h'])) {
    echo "帮助信息\n";
    echo "用法: php script.php -f <文件> [-h]\n";
    exit(0);
}

if (isset($options['f'])) {
    $file = $options['f'];
    echo "文件: {$file}\n";
} else {
    echo "错误: 必须指定文件 (-f)\n";
    exit(1);
}

// 执行: php script.php -f data.txt
// 输出: 文件: data.txt

// 执行: php script.php -h
// 输出: 帮助信息...
```

**说明**：
- `getopt('f:h')` 中，`f:` 表示 `-f` 需要参数，`h` 表示 `-h` 不需要参数
- 选项在数组中，键是选项名（不含 `-`），值是指定的参数或 `false`（无参数选项）

### 示例 5：使用 getopt() 处理长选项

```php
<?php
declare(strict_types=1);

// 长选项：--file 需要参数，--help 不需要参数
$options = getopt('', ['file:', 'help']);

if (isset($options['help'])) {
    echo "帮助信息\n";
    echo "用法: php script.php --file <文件> [--help]\n";
    exit(0);
}

if (isset($options['file'])) {
    $file = $options['file'];
    echo "文件: {$file}\n";
} else {
    echo "错误: 必须指定文件 (--file)\n";
    exit(1);
}

// 执行: php script.php --file data.txt
// 输出: 文件: data.txt
```

**说明**：
- 长选项数组中使用 `:` 表示需要参数
- `'file:'` 表示 `--file` 需要参数
- `'help'` 表示 `--help` 不需要参数

### 示例 6：同时处理短选项和长选项

```php
<?php
declare(strict_types=1);

// 短选项和长选项
$options = getopt('f:hv', ['file:', 'help', 'verbose']);

// 处理帮助选项
if (isset($options['h']) || isset($options['help'])) {
    echo "帮助信息\n";
    echo "用法: php script.php -f <文件> [选项]\n";
    echo "选项:\n";
    echo "  -f, --file <文件>    指定文件\n";
    echo "  -h, --help          显示帮助\n";
    echo "  -v, --verbose        详细输出\n";
    exit(0);
}

// 获取文件（支持短选项和长选项）
$file = $options['f'] ?? $options['file'] ?? null;
if ($file === null) {
    echo "错误: 必须指定文件\n";
    exit(1);
}

// 检查详细输出选项
$verbose = isset($options['v']) || isset($options['verbose']);

echo "文件: {$file}\n";
if ($verbose) {
    echo "详细模式已启用\n";
}

// 执行: php script.php -f data.txt -v
// 输出:
// 文件: data.txt
// 详细模式已启用
```

**说明**：
- 可以同时支持短选项和长选项
- 使用 `??` 运算符处理两种选项形式
- 无参数选项的值为 `false`

### 示例 7：获取剩余参数

```php
<?php
declare(strict_types=1);

$restIndex = null;
$options = getopt('f:h', ['file:', 'help'], $restIndex);

// $restIndex 是剩余参数的起始索引
if ($restIndex !== null && $restIndex < $argc) {
    $remainingArgs = array_slice($argv, $restIndex);
    echo "剩余参数: " . implode(', ', $remainingArgs) . "\n";
}

// 执行: php script.php -f data.txt arg1 arg2
// 输出: 剩余参数: arg1, arg2
```

**说明**：
- `$restIndex` 参数用于接收剩余参数的起始索引
- 剩余参数是选项之后的所有参数

### 示例 8：完整的 CLI 工具示例

```php
<?php
declare(strict_types=1);

// 检测 CLI 模式
if (php_sapi_name() !== 'cli') {
    die("This script must be run from the command line.\n");
}

// 解析选项
$options = getopt('f:o:hv', ['file:', 'output:', 'help', 'verbose']);

// 显示帮助
if (isset($options['h']) || isset($options['help'])) {
    echo "用法: php script.php -f <输入文件> [-o <输出文件>] [选项]\n";
    echo "\n选项:\n";
    echo "  -f, --file <文件>      输入文件（必需）\n";
    echo "  -o, --output <文件>    输出文件（可选）\n";
    echo "  -h, --help             显示帮助信息\n";
    echo "  -v, --verbose           详细输出\n";
    exit(0);
}

// 验证必需参数
$inputFile = $options['f'] ?? $options['file'] ?? null;
if ($inputFile === null) {
    echo "错误: 必须指定输入文件 (-f 或 --file)\n";
    exit(1);
}

// 获取可选参数
$outputFile = $options['o'] ?? $options['output'] ?? 'output.txt';
$verbose = isset($options['v']) || isset($options['verbose']);

// 处理文件
if ($verbose) {
    echo "处理文件: {$inputFile}\n";
}

if (!file_exists($inputFile)) {
    echo "错误: 文件不存在: {$inputFile}\n";
    exit(1);
}

// 处理逻辑...
echo "处理完成，输出到: {$outputFile}\n";
```

**说明**：
- 完整的 CLI 工具包含帮助信息、参数验证、错误处理
- 使用退出码表示执行结果（0 表示成功，非 0 表示错误）

## 使用场景

### 场景 1：命令行工具开发

开发命令行工具处理各种任务。

**示例**：见"示例 8：完整的 CLI 工具示例"

### 场景 2：批处理脚本

编写批处理脚本处理大量数据。

**示例**：

```php
<?php
declare(strict_types=1);

$options = getopt('d:h', ['directory:', 'help']);

$directory = $options['d'] ?? $options['directory'] ?? null;
if ($directory === null) {
    echo "用法: php batch.php -d <目录>\n";
    exit(1);
}

// 批处理逻辑
$files = glob($directory . '/*.txt');
foreach ($files as $file) {
    echo "处理: {$file}\n";
    // 处理文件...
}
```

## 注意事项

### CLI 模式检测

确保脚本只在 CLI 模式运行。

**示例**：

```php
<?php
declare(strict_types=1);

if (php_sapi_name() !== 'cli') {
    die("This script must be run from the command line.\n");
}
```

### 参数验证

验证参数的有效性，提供清晰的错误信息。

**示例**：

```php
<?php
declare(strict_types=1);

$file = $options['f'] ?? null;
if ($file === null) {
    echo "错误: 必须指定文件\n";
    exit(1);
}

if (!file_exists($file)) {
    echo "错误: 文件不存在: {$file}\n";
    exit(1);
}
```

### 退出码

使用退出码表示执行结果。

**示例**：

```php
<?php
declare(strict_types=1);

// 成功
exit(0);

// 错误
exit(1);
```

## 常见问题

### 问题 1：如何检测是否在 CLI 模式运行？

**回答**：使用 `php_sapi_name()` 函数，返回 `'cli'` 表示 CLI 模式。

**示例**：

```php
<?php
declare(strict_types=1);

if (php_sapi_name() === 'cli') {
    echo "Running in CLI mode\n";
}
```

### 问题 2：如何解析命令行参数？

**回答**：使用 `$argv` 和 `$argc` 获取参数，或使用 `getopt()` 解析选项。

**示例**：见"示例 2：获取命令行参数"和"示例 4：使用 getopt() 处理短选项"

### 问题 3：getopt() 的选项格式是什么？

**回答**：
- 短选项：`'f:h'` 表示 `-f` 需要参数，`-h` 不需要参数
- 长选项：`['file:', 'help']` 表示 `--file` 需要参数，`--help` 不需要参数
- `:` 表示需要参数

**示例**：

```php
<?php
declare(strict_types=1);

// f: 表示 -f 需要参数
// h 表示 -h 不需要参数
$options = getopt('f:h', ['file:', 'help']);
```

### 问题 4：如何处理必需参数？

**回答**：检查选项是否存在，如果不存在则显示错误信息并退出。

**示例**：

```php
<?php
declare(strict_types=1);

$file = $options['f'] ?? null;
if ($file === null) {
    echo "错误: 必须指定文件 (-f)\n";
    exit(1);
}
```

## 最佳实践

### 1. 使用 getopt() 处理复杂参数

对于需要多个选项的工具，使用 `getopt()` 更方便。

**示例**：见"示例 6：同时处理短选项和长选项"

### 2. 提供帮助信息（-h, --help）

为工具提供帮助信息，方便用户使用。

**示例**：

```php
<?php
declare(strict_types=1);

if (isset($options['h']) || isset($options['help'])) {
    echo "用法: php script.php [选项]\n";
    echo "选项:\n";
    echo "  -h, --help    显示帮助信息\n";
    exit(0);
}
```

### 3. 验证参数有效性

检查参数是否存在、格式是否正确、文件是否存在等。

**示例**：

```php
<?php
declare(strict_types=1);

$file = $options['f'] ?? null;
if ($file === null) {
    echo "错误: 必须指定文件\n";
    exit(1);
}

if (!file_exists($file)) {
    echo "错误: 文件不存在: {$file}\n";
    exit(1);
}
```

### 4. 提供清晰的错误提示

错误信息应该清晰、有用，帮助用户理解问题。

**示例**：

```php
<?php
declare(strict_types=1);

if ($file === null) {
    echo "错误: 必须指定文件\n";
    echo "用法: php script.php -f <文件>\n";
    exit(1);
}
```

### 5. 使用退出码

使用退出码表示执行结果（0 表示成功，非 0 表示错误）。

**示例**：

```php
<?php
declare(strict_types=1);

// 成功
exit(0);

// 一般错误
exit(1);

// 特定错误可以使用不同的退出码
// exit(2);  // 文件不存在
// exit(3);  // 权限错误
```

## 对比分析

### $argv vs getopt()

| 特性         | $argv                          | getopt()                       |
|:-------------|:-------------------------------|:-------------------------------|
| **简单性**   | ✅ 非常简单                    | ⚠️ 需要理解选项格式            |
| **功能**     | ⚠️ 功能有限，需要手动解析      | ✅ 功能强大，自动解析选项      |
| **适用场景** | 简单的参数列表                 | 复杂的选项和参数               |
| **灵活性**   | ⚠️ 需要手动处理所有情况        | ✅ 支持短选项、长选项、参数    |

### 短选项 vs 长选项

| 特性         | 短选项（-f）                   | 长选项（--file）               |
|:-------------|:-------------------------------|:-------------------------------|
| **简洁性**   | ✅ 更简洁                      | ⚠️ 较长                        |
| **可读性**   | ⚠️ 需要记忆选项含义            | ✅ 更易理解                    |
| **适用场景** | 常用选项                       | 不常用或需要明确含义的选项     |

## 练习任务

1. **CLI 参数解析工具类**：创建一个工具类，封装命令行参数的解析和验证功能。

2. **命令行工具框架**：实现一个简单的命令行工具框架，支持命令、选项、参数的定义和解析。

3. **帮助信息生成工具**：编写一个工具，根据选项定义自动生成帮助信息。

4. **参数验证工具**：创建一个工具，验证命令行参数的有效性（类型、范围、格式等）。

5. **CLI 工具示例**：开发一个完整的 CLI 工具，包含多个命令、选项和参数。

## 相关章节

- **[4.4.2 执行外部命令](section-02-exec-commands.md)**：学习如何执行外部命令
- **[4.4.3 交互式 CLI 与输出格式化](section-03-interactive-cli.md)**：学习交互式 CLI 开发
- **[4.4.6 环境变量与系统信息](section-06-environment-system.md)**：了解环境变量的使用
