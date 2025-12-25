# 5.8.1 文件读取

## 概述

文件读取是 PHP 开发中的常见任务。PHP 提供了多种文件读取方法，每种方法适用于不同的场景。理解不同读取方法的工作原理、适用场景，以及如何安全高效地读取文件，对于构建健壮的 PHP 应用至关重要。

## 读取方法的选择

PHP 提供了多种文件读取方法，每种方法适用于不同的场景：

1. **`file_get_contents()`**：一次性读取整个文件，适合小文件
2. **`fopen()` + `fread()`**：流式读取，适合大文件
3. **`fopen()` + `fgets()`**：逐行读取，适合文本文件
4. **`fopen()` + `fgetc()`**：逐字符读取，适合特殊需求
5. **`fopen()` + `fgetcsv()`**：读取 CSV 文件
6. **`readfile()`**：直接输出文件内容，适合下载场景

> **相关章节**：
> - 关于文件指针的详细说明，请参考 [5.8.4 文件指针操作](section-04-file-pointer.md)
> - 关于二进制文件读取的详细说明，请参考 [5.8.5 二进制文件处理](section-05-binary-files.md)

## file_get_contents() - 读取整个文件

**适用场景**：
- 读取配置文件
- 读取小文件（< 10MB）
- 需要一次性获取全部内容

**语法**：`file_get_contents(string $filename, bool $use_include_path = false, ?resource $context = null, int $offset = 0, ?int $length = null): string|false`

**参数详解**：
- `$filename`：要读取的文件路径（相对或绝对路径）
- `$use_include_path`：是否在 `include_path` 中搜索文件
  - `true`：如果文件不在当前目录，会在 `include_path` 中搜索
  - `false`：只在指定路径中查找（推荐）
- `$context`：流上下文资源，用于设置 HTTP 头、超时等选项
- `$offset`：读取的起始位置（字节），从 0 开始
- `$length`：要读取的最大长度（字节），`null` 表示读取到文件末尾

**返回值**：成功返回文件内容（字符串），失败返回 `false`。

**工作原理**：
1. 打开文件
2. 读取指定范围的内容
3. 关闭文件
4. 返回内容

**示例**：

```php
<?php
declare(strict_types=1);

// 读取整个文件
$content = file_get_contents(__DIR__ . '/data.txt');
if ($content === false) {
    throw new RuntimeException('Cannot read file');
}
echo $content;

// 读取部分内容（从第 100 字节开始，读取 200 字节）
$content = file_get_contents(__DIR__ . '/data.txt', false, null, 100, 200);

// 从 URL 读取（需要启用 allow_url_fopen）
$content = file_get_contents('https://example.com/data.txt');

// 使用 include_path
$content = file_get_contents('config.php', true);  // 在 include_path 中搜索
```

**注意事项**：
- 对于大文件（> 10MB），建议使用流式读取
- 读取失败时返回 `false`，需要检查返回值
- 从 URL 读取需要 `allow_url_fopen=On`
- 二进制文件需要使用二进制模式读取（详见 [5.8.5 二进制文件处理](section-05-binary-files.md)）

## fopen() - 打开文件

**`fopen()`** 是文件操作的核心函数，它返回一个文件句柄（资源），用于后续的读写操作。

**语法**：`fopen(string $filename, string $mode, bool $use_include_path = false, ?resource $context = null): resource|false`

**参数详解**：
- `$filename`：文件路径
- `$mode`：打开模式（见下表）
- `$use_include_path`：是否在 `include_path` 中搜索
- `$context`：流上下文资源

**打开模式详解**：

| 模式 | 说明 | 文件不存在 | 文件存在 | 文件指针位置 |
| :--- | :--- | :--- | :--- | :--- |
| `r` | 只读 | 失败 | 打开 | 文件开头 |
| `r+` | 读写 | 失败 | 打开 | 文件开头 |
| `w` | 只写 | 创建 | 清空后打开 | 文件开头 |
| `w+` | 读写 | 创建 | 清空后打开 | 文件开头 |
| `a` | 追加（只写） | 创建 | 打开 | 文件末尾 |
| `a+` | 追加（读写） | 创建 | 打开 | 文件末尾 |
| `x` | 只写（排他） | 创建 | 失败 | 文件开头 |
| `x+` | 读写（排他） | 创建 | 失败 | 文件开头 |
| `c` | 只写 | 创建 | 打开（不清空） | 文件开头 |
| `c+` | 读写 | 创建 | 打开（不清空） | 文件开头 |

**模式后缀**：
- `b`：二进制模式（Binary mode）
- `t`：文本模式（Text mode，默认）

### 文本模式 vs 二进制模式

**文本模式（`t`）**：
- 默认模式，适用于文本文件
- 在 Windows 上会自动转换换行符（`\r\n` ↔ `\n`）
- 在类 Unix 系统上无影响（因为换行符是 `\n`）
- 适用于：`.txt`、`.php`、`.html`、`.json` 等文本文件

**二进制模式（`b`）**：
- 不进行任何转换，按字节原样读取
- 适用于所有文件类型，特别是二进制文件
- **在 Windows 上强烈推荐使用**，避免换行符转换问题
- 适用于：图片（`.jpg`、`.png`、`.gif`）、视频、音频、压缩文件、可执行文件等

**为什么需要二进制模式**：

在 Windows 系统上，文本模式会自动转换换行符：
- 读取时：`\r\n`（Windows）→ `\n`（PHP 内部）
- 写入时：`\n`（PHP 内部）→ `\r\n`（Windows）

对于二进制文件，这种转换会**破坏文件内容**。例如：
- 图片文件中如果包含 `0x0A`（`\n`），会被错误处理
- 压缩文件、可执行文件等会被损坏

**跨平台兼容性**：
- 在类 Unix 系统（Linux、macOS）上，文本模式和二进制模式行为相同
- 在 Windows 上，二进制模式是必需的
- **最佳实践**：始终使用二进制模式（`b`），确保跨平台兼容性

**示例**：

```php
<?php
declare(strict_types=1);

// 只读模式
$handle = fopen(__DIR__ . '/data.txt', 'r');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// 读写模式
$handle = fopen(__DIR__ . '/data.txt', 'r+');

// 写入模式（会清空文件）
$handle = fopen(__DIR__ . '/output.txt', 'w');

// 追加模式
$handle = fopen(__DIR__ . '/log.txt', 'a');

// 排他模式（防止并发创建）
$handle = fopen(__DIR__ . '/lock.txt', 'x');
if ($handle === false) {
    // 文件已存在
    throw new RuntimeException('File already exists');
}

// 二进制模式（推荐，跨平台兼容）
$handle = fopen(__DIR__ . '/image.jpg', 'rb');
$handle = fopen(__DIR__ . '/data.bin', 'rb');
$handle = fopen(__DIR__ . '/archive.zip', 'rb');
```

## fread() - 读取数据

**语法**：`fread(resource $stream, int $length): string|false`

**参数**：
- `$stream`：文件句柄
- `$length`：要读取的最大字节数

**返回值**：成功返回读取的字符串，失败或到达文件末尾返回 `false`。

**工作原理**：
- 从当前文件指针位置读取指定长度的数据
- 如果剩余数据少于 `$length`，返回剩余数据
- 到达文件末尾时返回 `false`

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'r');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// 读取 1024 字节
$chunk = fread($handle, 1024);

// 流式读取大文件（每次 8KB）
while (($chunk = fread($handle, 8192)) !== false) {
    echo $chunk;
}

fclose($handle);
```


## fclose() - 关闭文件

**语法**：`fclose(resource $stream): bool`

**返回值**：成功返回 `true`，失败返回 `false`。

**重要性**：
- **必须关闭文件句柄**，释放系统资源
- 未关闭的文件句柄会导致文件锁定，其他进程无法访问
- PHP 脚本结束时会自动关闭，但显式关闭是良好实践

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'r');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    // 文件操作
    $content = fread($handle, 1024);
} finally {
    // 确保文件被关闭
    fclose($handle);
}
```

## fgets() - 逐行读取

**适用场景**：
- 读取文本文件
- 处理日志文件
- 逐行处理数据

**语法**：`fgets(resource $stream, ?int $length = null): string|false`

**参数**：
- `$stream`：文件句柄
- `$length`：可选，读取的最大长度（字节），默认读取到行尾或 EOF

**返回值**：
- 成功返回一行内容（包含换行符），失败或到达文件末尾返回 `false`

**工作原理**：
- 从当前文件指针位置读取，直到遇到换行符（`\n`）或文件末尾
- 返回的字符串包含换行符
- 如果指定了 `$length`，最多读取 `$length - 1` 字节

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'r');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// 逐行读取
while (($line = fgets($handle)) !== false) {
    // $line 包含换行符，使用 trim() 去除
    echo trim($line) . "\n";
}

fclose($handle);

// 限制每行最大长度
$handle = fopen(__DIR__ . '/data.txt', 'r');
while (($line = fgets($handle, 100)) !== false) {
    // 最多读取 99 字节
    echo $line;
}
fclose($handle);
```

## fgetc() - 读取单个字符

**适用场景**：
- 逐字符处理文件
- 解析特定格式的文件
- 需要精确控制读取位置

**语法**：`fgetc(resource $stream): string|false`

**参数**：
- `$stream`：文件句柄

**返回值**：
- 成功返回一个字符（字符串，长度为 1），失败或到达文件末尾返回 `false`

**工作原理**：
- 从当前文件指针位置读取一个字符
- 读取后文件指针向前移动 1 字节
- 返回的字符包含换行符、空格等所有字符

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'r');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// 逐字符读取
while (($char = fgetc($handle)) !== false) {
    echo $char;
    // 可以在这里处理每个字符
}

fclose($handle);

// 读取前 10 个字符
$handle = fopen(__DIR__ . '/data.txt', 'r');
for ($i = 0; $i < 10; $i++) {
    $char = fgetc($handle);
    if ($char === false) {
        break;  // 到达文件末尾
    }
    echo $char;
}
fclose($handle);
```

**注意事项**：
- `fgetc()` 返回的字符串长度为 1，即使是多字节字符
- 对于 UTF-8 等多字节编码，应该使用其他方法（如 `fread()` + `mb_substr()`）
- 逐字符读取效率较低，适合小文件或特殊需求

## fgetcsv() - 读取 CSV 文件

**语法**：`fgetcsv(resource $stream, ?int $length = null, string $separator = ",", string $enclosure = "\"", string $escape = "\\"): array|false`

**参数**：
- `$stream`：文件句柄
- `$length`：可选，行的最大长度
- `$separator`：字段分隔符，默认为 `,`
- `$enclosure`：字段包围符，默认为 `"`
- `$escape`：转义字符，默认为 `\`

**返回值**：成功返回包含字段的数组，失败或到达文件末尾返回 `false`。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.csv', 'r');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// 跳过标题行（可选）
$header = fgetcsv($handle);

// 读取数据行
while (($row = fgetcsv($handle)) !== false) {
    // $row 是数组，包含 CSV 的各个字段
    print_r($row);
}

fclose($handle);
```

## readfile() - 直接输出文件

**适用场景**：
- 文件下载
- 输出静态文件
- 不需要处理文件内容

**语法**：`readfile(string $filename, bool $use_include_path = false, ?resource $context = null): int|false`

**返回值**：成功返回读取的字节数，失败返回 `false`。

**工作原理**：
- 直接读取文件并输出到输出缓冲区
- 不将文件内容加载到内存
- 适合大文件下载

**示例**：

```php
<?php
declare(strict_types=1);

// 下载文件
header('Content-Type: application/octet-stream');
header('Content-Disposition: attachment; filename="file.txt"');
readfile(__DIR__ . '/file.txt');
exit;
```


## 注意事项

1. **错误处理**：始终检查文件操作的返回值
2. **大文件处理**：对于大文件（> 10MB），使用流式操作而不是一次性读取
3. **资源释放**：使用 `fopen()` 后必须使用 `fclose()` 释放资源
4. **二进制文件**：处理二进制文件时，**必须**使用 `b` 模式（如 `rb`、`wb`），避免数据损坏。即使是文本文件，也推荐使用 `b` 模式以确保跨平台兼容性（详见 [5.8.5 二进制文件处理](section-05-binary-files.md)）

## 练习

1. 编写一个函数，安全地读取大文件（使用流式读取）。
2. 实现一个 CSV 文件读取器，解析并处理 CSV 数据。
3. 创建一个文件下载功能，支持断点续传。
4. 编写一个函数，逐行读取日志文件并进行分析。
