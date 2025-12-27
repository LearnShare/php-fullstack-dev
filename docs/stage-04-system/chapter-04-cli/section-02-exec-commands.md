# 4.4.2 执行外部命令

## 概述

执行外部命令是 CLI 编程中的常见需求。在实际应用中，我们经常需要调用系统命令、执行脚本、与其他程序交互等。PHP 提供了多种函数来执行外部命令，包括 `exec()`、`shell_exec()`、`system()`、`passthru()`、`proc_open()` 等，每种方法有不同的特点和适用场景。

理解不同命令执行函数的特点、使用方法，以及如何进行安全地执行命令，对于构建健壮的 CLI 应用至关重要。同时，需要注意命令注入等安全风险。

**主要内容**：
- 命令执行函数概述
- `exec()` 函数（执行命令并返回最后一行）
- `shell_exec()` 函数（执行命令并返回完整输出）
- `system()` 函数（执行命令并直接输出）
- `passthru()` 函数（执行命令并直接输出二进制数据）
- `proc_open()` 函数（高级进程控制）
- 命令执行安全（命令注入防护、参数转义）
- 实际应用场景和最佳实践

## 特性

- **多种方法**：提供多种命令执行方法，适应不同场景
- **输出控制**：可以捕获输出、直接输出或处理二进制数据
- **进程控制**：支持高级进程控制和交互
- **安全考虑**：需要注意命令注入等安全风险
- **灵活配置**：支持设置工作目录、环境变量等

## 语法/定义

### exec() 函数

**语法**：`exec(string $command, array &$output = null, int &$result_code = null): string|false`

**参数**：
- `$command`：要执行的命令
- `$output`：可选，用于接收命令输出的数组（按行）
- `$result_code`：可选，用于接收命令的退出码

**返回值**：返回命令输出的最后一行，失败返回 `false`。

### shell_exec() 函数

**语法**：`shell_exec(string $command): string|null|false`

**参数**：
- `$command`：要执行的命令

**返回值**：返回命令的完整输出，失败返回 `null` 或 `false`。

### system() 函数

**语法**：`system(string $command, int &$result_code = null): string|false`

**参数**：
- `$command`：要执行的命令
- `$result_code`：可选，用于接收命令的退出码

**返回值**：返回命令输出的最后一行，失败返回 `false`。

### passthru() 函数

**语法**：`passthru(string $command, int &$result_code = null): ?false`

**参数**：
- `$command`：要执行的命令
- `$result_code`：可选，用于接收命令的退出码

**返回值**：成功返回 `null`，失败返回 `false`。

### proc_open() 函数

**语法**：`proc_open(array|string $command, array $descriptor_spec, array &$pipes, ?string $cwd = null, ?array $env_vars = null, ?array $options = null): resource|false`

**参数**：
- `$command`：要执行的命令（字符串或数组）
- `$descriptor_spec`：描述符规范数组
- `$pipes`：用于接收管道资源的数组
- `$cwd`：可选，工作目录
- `$env_vars`：可选，环境变量数组
- `$options`：可选，额外选项

**返回值**：成功返回进程资源，失败返回 `false`。

## 基本用法

### 示例 1：使用 exec() 执行命令

```php
<?php
declare(strict_types=1);

// 执行命令并获取输出
exec('ls -la', $output, $returnCode);

echo "退出码: {$returnCode}\n";
echo "输出:\n";
foreach ($output as $line) {
    echo "  {$line}\n";
}

// 获取最后一行输出
$lastLine = exec('echo "Hello, World!"');
echo "最后一行: {$lastLine}\n";
```

**说明**：
- `exec()` 执行命令并将输出按行存储到数组中
- 返回最后一行输出
- 可以通过第三个参数获取退出码

### 示例 2：使用 shell_exec() 执行命令

```php
<?php
declare(strict_types=1);

// 执行命令并获取完整输出
$output = shell_exec('ls -la');
if ($output !== null) {
    echo $output;
} else {
    echo "命令执行失败\n";
}

// 执行命令并处理输出
$result = shell_exec('php -v');
echo "PHP 版本信息:\n{$result}\n";
```

**说明**：
- `shell_exec()` 返回命令的完整输出（字符串）
- 适合需要完整输出的场景
- 失败时返回 `null`

### 示例 3：使用 system() 执行命令

```php
<?php
declare(strict_types=1);

// system() 直接输出命令结果
$lastLine = system('ls -la', $returnCode);
echo "\n退出码: {$returnCode}\n";
echo "最后一行: {$lastLine}\n";
```

**说明**：
- `system()` 直接输出命令结果到标准输出
- 返回最后一行输出
- 可以通过第二个参数获取退出码

### 示例 4：使用 passthru() 执行命令

```php
<?php
declare(strict_types=1);

// passthru() 直接输出原始数据（适合二进制数据）
passthru('cat image.jpg', $returnCode);
echo "\n退出码: {$returnCode}\n";
```

**说明**：
- `passthru()` 直接输出原始数据，不进行任何处理
- 适合输出二进制数据（如图片）
- 不返回输出内容

### 示例 5：使用 proc_open() 进行高级进程控制

```php
<?php
declare(strict_types=1);

// 定义描述符
$descriptors = [
    0 => ['pipe', 'r'],  // stdin
    1 => ['pipe', 'w'],  // stdout
    2 => ['pipe', 'w'],  // stderr
];

// 创建进程
$process = proc_open('php -r "echo fgets(STDIN);"', $descriptors, $pipes);
if (!is_resource($process)) {
    throw new RuntimeException('Cannot create process');
}

// 写入输入
fwrite($pipes[0], "Hello, Process!\n");
fclose($pipes[0]);

// 读取输出
$output = stream_get_contents($pipes[1]);
fclose($pipes[1]);

// 读取错误
$error = stream_get_contents($pipes[2]);
fclose($pipes[2]);

// 关闭进程
$returnCode = proc_close($process);

echo "输出: {$output}\n";
if ($error !== '') {
    echo "错误: {$error}\n";
}
echo "退出码: {$returnCode}\n";
```

**说明**：
- `proc_open()` 提供最灵活的控制
- 可以控制标准输入、输出、错误流
- 适合需要交互的场景

### 示例 6：安全地执行命令（参数转义）

```php
<?php
declare(strict_types=1);

// 危险：直接拼接用户输入（存在命令注入风险）
// $filename = $_GET['file'];
// exec("cat {$filename}");  // 危险！

// 安全：使用 escapeshellarg() 转义参数
function safeExec(string $command, string $arg): string
{
    $escapedArg = escapeshellarg($arg);
    $fullCommand = "{$command} {$escapedArg}";
    
    $output = shell_exec($fullCommand);
    return $output !== null ? $output : '';
}

// 使用
$filename = 'file.txt';
$content = safeExec('cat', $filename);
echo $content;
```

**说明**：
- 使用 `escapeshellarg()` 转义参数，防止命令注入
- 永远不要直接拼接用户输入到命令中

### 示例 7：使用 proc_open() 设置工作目录和环境变量

```php
<?php
declare(strict_types=1);

$descriptors = [
    0 => ['pipe', 'r'],
    1 => ['pipe', 'w'],
    2 => ['pipe', 'w'],
];

$cwd = '/tmp';  // 工作目录
$env = ['PATH' => '/usr/bin:/bin'];  // 环境变量

$process = proc_open('pwd', $descriptors, $pipes, $cwd, $env);
if (is_resource($process)) {
    $output = stream_get_contents($pipes[1]);
    fclose($pipes[1]);
    fclose($pipes[2]);
    proc_close($process);
    
    echo "工作目录: {$output}\n";
}
```

**说明**：
- `proc_open()` 可以设置工作目录和环境变量
- 提供更精细的控制

### 示例 8：获取进程状态

```php
<?php
declare(strict_types=1);

$descriptors = [
    1 => ['pipe', 'w'],
    2 => ['pipe', 'w'],
];

$process = proc_open('sleep 5', $descriptors, $pipes);
if (is_resource($process)) {
    // 获取进程状态
    $status = proc_get_status($process);
    print_r($status);
    
    // 等待进程结束
    proc_close($process);
}
```

**说明**：
- `proc_get_status()` 获取进程的当前状态
- 返回包含进程信息的数组

## 使用场景

### 场景 1：系统命令调用

调用系统命令执行各种任务。

**示例**：

```php
<?php
declare(strict_types=1);

// 获取系统信息
$uptime = shell_exec('uptime');
echo "系统运行时间: {$uptime}\n";

// 列出文件
exec('ls -la /tmp', $files);
foreach ($files as $file) {
    echo "{$file}\n";
}
```

### 场景 2：脚本执行

执行其他脚本或程序。

**示例**：

```php
<?php
declare(strict_types=1);

// 执行 Python 脚本
$output = shell_exec('python3 script.py');
echo $output;

// 执行 Shell 脚本
exec('bash process.sh', $output, $returnCode);
if ($returnCode === 0) {
    echo "脚本执行成功\n";
} else {
    echo "脚本执行失败，退出码: {$returnCode}\n";
}
```

## 注意事项

### 命令注入安全风险

永远不要直接拼接用户输入到命令中，使用 `escapeshellarg()` 转义。

**示例**：

```php
<?php
declare(strict_types=1);

// ❌ 危险：直接拼接
$userInput = $_GET['file'];
exec("cat {$userInput}");  // 如果 $userInput = "file.txt; rm -rf /"，会执行删除命令！

// ✅ 安全：转义参数
$userInput = $_GET['file'];
$escaped = escapeshellarg($userInput);
exec("cat {$escaped}");  // 安全
```

### 超时控制

对于可能长时间运行的命令，设置超时。

**示例**：

```php
<?php
declare(strict_types=1);

// 使用 proc_open() 可以更好地控制超时
$descriptors = [
    1 => ['pipe', 'w'],
    2 => ['pipe', 'w'],
];

$process = proc_open('long-running-command', $descriptors, $pipes);
if (is_resource($process)) {
    // 设置超时（需要配合其他机制，如信号处理）
    // ...
}
```

### 错误处理

检查命令执行结果和退出码。

**示例**：

```php
<?php
declare(strict_types=1);

exec('command', $output, $returnCode);
if ($returnCode !== 0) {
    echo "命令执行失败，退出码: {$returnCode}\n";
    exit(1);
}
```

## 常见问题

### 问题 1：exec() 和 system() 的区别是什么？

**回答**：
- `exec()`：返回最后一行输出，需要第二个参数获取所有输出
- `system()`：直接输出所有内容，返回最后一行

**示例**：

```php
<?php
declare(strict_types=1);

// exec()：需要数组参数获取所有输出
exec('ls', $output);
print_r($output);

// system()：直接输出
system('ls');  // 直接显示在终端
```

### 问题 2：如何安全地执行外部命令？

**回答**：使用 `escapeshellarg()` 或 `escapeshellcmd()` 转义参数，验证命令白名单。

**示例**：见"示例 6：安全地执行命令（参数转义）"

### 问题 3：如何处理命令输出？

**回答**：根据需求选择合适的方法：
- 需要完整输出：使用 `shell_exec()`
- 需要逐行处理：使用 `exec()` 的数组参数
- 需要直接输出：使用 `system()` 或 `passthru()`
- 需要交互：使用 `proc_open()`

### 问题 4：proc_open() 的优势是什么？

**回答**：`proc_open()` 提供最灵活的控制，可以控制标准输入、输出、错误流，设置工作目录和环境变量，适合复杂的进程控制场景。

**示例**：见"示例 5：使用 proc_open() 进行高级进程控制"

## 最佳实践

### 1. 使用 escapeshellarg() 转义参数

永远转义用户输入，防止命令注入。

**示例**：

```php
<?php
declare(strict_types=1);

$userInput = $_GET['file'];
$escaped = escapeshellarg($userInput);
exec("cat {$escaped}");
```

### 2. 验证命令白名单

只允许执行预定义的命令。

**示例**：

```php
<?php
declare(strict_types=1);

$allowedCommands = ['ls', 'cat', 'pwd'];
$command = $_GET['cmd'];

if (!in_array($command, $allowedCommands, true)) {
    throw new InvalidArgumentException("Command not allowed: {$command}");
}

exec($command);
```

### 3. 使用 proc_open() 进行高级控制

对于需要精细控制的场景，使用 `proc_open()`。

**示例**：见"示例 5：使用 proc_open() 进行高级进程控制"

### 4. 设置执行超时

对于可能长时间运行的命令，设置超时机制。

**示例**：

```php
<?php
declare(strict_types=1);

// 使用 proc_open() 可以更好地控制超时
// 需要配合信号处理或其他机制
```

### 5. 检查退出码

检查命令的退出码，判断执行是否成功。

**示例**：

```php
<?php
declare(strict_types=1);

exec('command', $output, $returnCode);
if ($returnCode !== 0) {
    throw new RuntimeException("Command failed with exit code: {$returnCode}");
}
```

## 对比分析

### exec() vs shell_exec() vs system() vs passthru()

| 特性         | exec()                        | shell_exec()                  | system()                      | passthru()                    |
|:-------------|:------------------------------|:------------------------------|:------------------------------|:------------------------------|
| **返回值**   | 最后一行                      | 完整输出（字符串）            | 最后一行                      | 无返回值                      |
| **输出方式** | 需要数组参数获取              | 返回字符串                    | 直接输出到标准输出             | 直接输出原始数据               |
| **适用场景** | 需要逐行处理                  | 需要完整输出字符串            | 简单命令执行                  | 二进制数据输出                |
| **控制力**   | ⚠️ 有限                       | ⚠️ 有限                       | ⚠️ 有限                       | ⚠️ 有限                       |

### 简单函数 vs proc_open()

| 特性         | exec()/shell_exec()/system()  | proc_open()                   |
|:-------------|:------------------------------|:------------------------------|
| **简单性**   | ✅ 简单易用                    | ⚠️ 较复杂                     |
| **控制力**   | ⚠️ 控制有限                   | ✅ 完全控制                    |
| **适用场景** | 简单命令执行                  | 复杂进程控制、交互式命令      |
| **灵活性**   | ⚠️ 有限                       | ✅ 非常灵活                    |

## 练习任务

1. **命令执行工具类**：创建一个工具类，封装安全的命令执行功能。

2. **命令注入防护工具**：实现一个工具，验证和转义命令参数，防止命令注入。

3. **进程管理工具**：编写一个工具，使用 `proc_open()` 管理进程的生命周期。

4. **命令执行监控工具**：创建一个工具，监控命令执行时间、资源使用等。

5. **安全命令执行框架**：实现一个框架，提供安全的命令执行接口，包括白名单验证、参数转义等。

## 相关章节

- **[4.4.1 CLI 基础与参数处理](section-01-cli-basics.md)**：了解 CLI 编程基础
- **[4.4.4 进程管理与守护进程](section-04-process-management.md)**：学习进程管理的相关内容
- **[4.7.1 错误处理机制](../chapter-07-errors/section-01-error-handling.md)**：了解错误处理的相关内容
