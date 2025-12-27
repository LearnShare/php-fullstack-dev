# 4.2.1 文件读取

## 概述

文件读取是文件系统操作的基础功能。在 PHP 开发中，我们经常需要读取文件内容，比如读取配置文件、日志文件、用户上传的文件等。PHP 提供了多种文件读取方法，每种方法适用于不同的场景。理解不同读取方法的工作原理、适用场景，以及如何安全高效地读取文件，对于构建健壮的 PHP 应用至关重要。

文件读取操作需要考虑多个因素：文件大小、文件类型（文本文件还是二进制文件）、读取方式（一次性读取还是流式读取）、错误处理等。选择正确的读取方法可以提高代码的性能和可维护性。

掌握文件读取的基础知识后，可以进一步学习文件写入、文件操作、文件指针、二进制文件处理等高级主题。

**主要内容**：
- 文件读取方法概述
- `file_get_contents()` 函数
- `fopen()` 和 `fread()` 函数
- `fgets()` 逐行读取
- `fgetc()` 逐字符读取
- `fgetcsv()` 读取 CSV 文件
- `readfile()` 直接输出文件
- `file()` 读取文件到数组
- 错误处理和最佳实践

## 特性

- **多种读取方式**：支持一次性读取、流式读取、逐行读取等多种方式
- **灵活的文件处理**：支持文本文件和二进制文件
- **高效的内存使用**：流式读取适合大文件，避免内存溢出
- **完善的错误处理**：提供错误检查和处理机制
- **跨平台兼容**：支持 Windows 和类 Unix 系统

## 语法/定义

### file_get_contents() 函数

**语法**：`file_get_contents(string $filename, bool $use_include_path = false, ?resource $context = null, int $offset = 0, ?int $length = null): string|false`

**参数**：
- `$filename`：要读取的文件路径（相对或绝对路径），也可以是一个 URL
- `$use_include_path`：可选，是否在 `include_path` 中搜索文件，默认为 `false`
- `$context`：可选，流上下文资源，用于设置 HTTP 头、超时等选项
- `$offset`：可选，读取的起始位置（字节），从 0 开始，默认为 0
- `$length`：可选，要读取的最大长度（字节），`null` 表示读取到文件末尾

**返回值**：成功返回文件内容（字符串），失败返回 `false`。

### fopen() 函数

**语法**：`fopen(string $filename, string $mode, bool $use_include_path = false, ?resource $context = null): resource|false`

**参数**：
- `$filename`：文件路径
- `$mode`：打开模式（`r`、`r+`、`w`、`w+`、`a`、`a+`、`x`、`x+`、`c`、`c+`，可加 `b` 或 `t` 后缀）
- `$use_include_path`：可选，是否在 `include_path` 中搜索
- `$context`：可选，流上下文资源

**返回值**：成功返回文件句柄（资源），失败返回 `false`。

### fread() 函数

**语法**：`fread(resource $stream, int $length): string|false`

**参数**：
- `$stream`：文件句柄
- `$length`：要读取的最大字节数

**返回值**：成功返回读取的字符串，失败或到达文件末尾返回 `false`。

### fgets() 函数

**语法**：`fgets(resource $stream, ?int $length = null): string|false`

**参数**：
- `$stream`：文件句柄
- `$length`：可选，读取的最大长度（字节），默认读取到行尾或 EOF

**返回值**：成功返回一行内容（包含换行符），失败或到达文件末尾返回 `false`。

## 基本用法

### 示例 1：使用 file_get_contents() 读取整个文件

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
if ($content !== false) {
    echo $content;
}
```

**说明**：
- `file_get_contents()` 一次性读取整个文件到内存
- 适合小文件（< 10MB）
- 读取失败时返回 `false`，需要检查返回值

### 示例 2：使用 fopen() 和 fread() 读取文件

```php
<?php
declare(strict_types=1);

// 打开文件
$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    // 读取整个文件
    $fileSize = filesize(__DIR__ . '/data.txt');
    if ($fileSize !== false) {
        $content = fread($handle, $fileSize);
        echo $content;
    }
} finally {
    // 确保文件被关闭
    fclose($handle);
}
```

**输出**：

```
文件内容...
```

**说明**：
- `fopen()` 打开文件返回文件句柄
- `fread()` 从文件句柄读取指定长度的数据
- 使用完毕后必须调用 `fclose()` 关闭文件句柄

### 示例 3：流式读取大文件

```php
<?php
declare(strict_types=1);

// 流式读取大文件（每次 8KB）
$handle = fopen(__DIR__ . '/large-file.txt', 'rb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    while (($chunk = fread($handle, 8192)) !== false) {
        if ($chunk === '') {
            break;  // 到达文件末尾
        }
        echo $chunk;
        // 可以在这里处理每一块数据
    }
} finally {
    fclose($handle);
}
```

**说明**：
- 流式读取适合大文件，避免一次性加载到内存
- 每次读取固定大小的块（如 8KB），逐步处理
- 到达文件末尾时 `fread()` 返回空字符串

### 示例 4：使用 fgets() 逐行读取

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    // 逐行读取
    while (($line = fgets($handle)) !== false) {
        // $line 包含换行符，使用 trim() 去除
        echo trim($line) . "\n";
    }
} finally {
    fclose($handle);
}
```

**输出**：

```
第一行
第二行
第三行
```

**说明**：
- `fgets()` 每次读取一行，包含换行符
- 适合处理文本文件，特别是日志文件
- 到达文件末尾时返回 `false`

### 示例 5：使用 fgetc() 逐字符读取

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    // 逐字符读取
    while (($char = fgetc($handle)) !== false) {
        echo $char;
        // 可以在这里处理每个字符
    }
} finally {
    fclose($handle);
}

// 读取前 10 个字符
$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle !== false) {
    try {
        for ($i = 0; $i < 10; $i++) {
            $char = fgetc($handle);
            if ($char === false) {
                break;  // 到达文件末尾
            }
            echo $char;
        }
    } finally {
        fclose($handle);
    }
}
```

**说明**：
- `fgetc()` 每次读取一个字符
- 返回的字符串长度为 1
- 效率较低，适合小文件或特殊需求

### 示例 6：使用 fgetcsv() 读取 CSV 文件

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.csv', 'rb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    // 跳过标题行（可选）
    $header = fgetcsv($handle);
    if ($header !== false) {
        print_r($header);
    }
    
    // 读取数据行
    while (($row = fgetcsv($handle)) !== false) {
        // $row 是数组，包含 CSV 的各个字段
        print_r($row);
    }
} finally {
    fclose($handle);
}
```

**输出**：

```
Array
(
    [0] => 姓名
    [1] => 年龄
    [2] => 城市
)
Array
(
    [0] => 张三
    [1] => 25
    [2] => 北京
)
```

**说明**：
- `fgetcsv()` 自动解析 CSV 格式
- 返回包含字段的数组
- 自动处理引号和转义字符

### 示例 7：使用 file() 读取文件到数组

```php
<?php
declare(strict_types=1);

// 读取文件到数组（每行一个元素）
$lines = file(__DIR__ . '/data.txt', FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
if ($lines !== false) {
    foreach ($lines as $lineNum => $line) {
        echo "Line " . ($lineNum + 1) . ": {$line}\n";
    }
}
```

**输出**：

```
Line 1: 第一行
Line 2: 第二行
Line 3: 第三行
```

**说明**：
- `file()` 一次性读取整个文件到数组
- 每个数组元素对应文件的一行
- `FILE_IGNORE_NEW_LINES` 标志去除换行符
- `FILE_SKIP_EMPTY_LINES` 标志跳过空行

### 示例 8：使用 readfile() 直接输出文件

```php
<?php
declare(strict_types=1);

// 直接输出文件内容（适合下载场景）
header('Content-Type: application/octet-stream');
header('Content-Disposition: attachment; filename="file.txt"');
$bytesRead = readfile(__DIR__ . '/file.txt');
if ($bytesRead === false) {
    throw new RuntimeException('Cannot read file');
}
exit;
```

**说明**：
- `readfile()` 直接读取文件并输出到输出缓冲区
- 不将文件内容加载到内存
- 适合大文件下载场景
- 返回读取的字节数

### 示例 9：从 URL 读取文件

```php
<?php
declare(strict_types=1);

// 从 URL 读取（需要启用 allow_url_fopen）
$content = file_get_contents('https://example.com/data.txt');
if ($content !== false) {
    echo $content;
} else {
    echo "Failed to read from URL\n";
}

// 使用流上下文设置选项
$context = stream_context_create([
    'http' => [
        'method' => 'GET',
        'header' => 'User-Agent: PHP Script',
        'timeout' => 30,
    ],
]);

$content = file_get_contents('https://example.com/api/data', false, $context);
if ($content !== false) {
    echo $content;
}
```

**说明**：
- `file_get_contents()` 可以从 URL 读取内容
- 需要 `allow_url_fopen` 配置项为 `On`
- 可以使用流上下文设置 HTTP 头、超时等选项

## 使用场景

### 场景 1：读取配置文件

读取应用配置文件（如 JSON、INI 格式）。

**示例**：

```php
<?php
declare(strict_types=1);

function loadConfig(string $configPath): array
{
    $content = file_get_contents($configPath);
    if ($content === false) {
        throw new RuntimeException("Cannot read config file: {$configPath}");
    }
    
    $config = json_decode($content, true);
    if (json_last_error() !== JSON_ERROR_NONE) {
        throw new RuntimeException('Invalid JSON config: ' . json_last_error_msg());
    }
    
    return $config;
}

$config = loadConfig(__DIR__ . '/config.json');
print_r($config);
```

### 场景 2：读取日志文件

逐行读取日志文件进行分析。

**示例**：

```php
<?php
declare(strict_types=1);

function analyzeLog(string $logPath): array
{
    $handle = fopen($logPath, 'rb');
    if ($handle === false) {
        throw new RuntimeException('Cannot open log file');
    }
    
    $errors = [];
    $lineNum = 0;
    
    try {
        while (($line = fgets($handle)) !== false) {
            $lineNum++;
            if (str_contains($line, 'ERROR')) {
                $errors[] = [
                    'line' => $lineNum,
                    'content' => trim($line),
                ];
            }
        }
    } finally {
        fclose($handle);
    }
    
    return $errors;
}

$errors = analyzeLog(__DIR__ . '/app.log');
print_r($errors);
```

### 场景 3：读取用户上传的文件

读取用户上传的文件内容。

**示例**：

```php
<?php
declare(strict_types=1);

// 假设文件已上传到临时目录
$uploadedFile = $_FILES['file']['tmp_name'] ?? '';

if ($uploadedFile !== '' && is_uploaded_file($uploadedFile)) {
    $content = file_get_contents($uploadedFile);
    if ($content !== false) {
        // 处理文件内容
        echo "File size: " . strlen($content) . " bytes\n";
    }
}
```

## 注意事项

### 文件大小限制

对于大文件（> 10MB），应使用流式读取而不是一次性读取，避免内存溢出。

**示例**：

```php
<?php
declare(strict_types=1);

// 不推荐：大文件一次性读取
// $content = file_get_contents('large-file.txt');  // 可能导致内存溢出

// 推荐：使用流式读取
$handle = fopen('large-file.txt', 'rb');
if ($handle !== false) {
    try {
        while (($chunk = fread($handle, 8192)) !== false) {
            if ($chunk === '') {
                break;
            }
            // 处理每一块
            echo $chunk;
        }
    } finally {
        fclose($handle);
    }
}
```

### 二进制模式 vs 文本模式

对于二进制文件（图片、视频等），必须使用二进制模式（`b`）打开，避免数据损坏。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：始终使用二进制模式（跨平台兼容）
$handle = fopen(__DIR__ . '/image.jpg', 'rb');  // 二进制模式
$handle = fopen(__DIR__ . '/data.txt', 'rb');   // 即使是文本文件，也推荐使用 'rb'

// 不推荐：文本模式（Windows 上可能有问题）
// $handle = fopen(__DIR__ . '/image.jpg', 'r');  // 文本模式，可能损坏二进制文件
```

### 错误处理

始终检查文件操作的返回值，进行适当的错误处理。

**示例**：

```php
<?php
declare(strict_types=1);

function safeReadFile(string $filePath): ?string
{
    if (!file_exists($filePath)) {
        error_log("File not found: {$filePath}");
        return null;
    }
    
    if (!is_readable($filePath)) {
        error_log("File not readable: {$filePath}");
        return null;
    }
    
    $content = file_get_contents($filePath);
    if ($content === false) {
        error_log("Failed to read file: {$filePath}");
        return null;
    }
    
    return $content;
}

$content = safeReadFile(__DIR__ . '/data.txt');
if ($content !== null) {
    echo $content;
}
```

### 文件句柄管理

使用 `fopen()` 后必须使用 `fclose()` 关闭文件句柄，释放系统资源。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    // 文件操作
    $content = fread($handle, 1024);
} finally {
    // 确保文件被关闭（即使发生异常）
    fclose($handle);
}
```

## 常见问题

### 问题 1：file_get_contents() 和 fopen()/fread() 的区别是什么？

**回答**：
- **file_get_contents()**：一次性读取整个文件，适合小文件，代码简单
- **fopen()/fread()**：流式读取，适合大文件，可以控制读取过程

**示例**：

```php
<?php
declare(strict_types=1);

// file_get_contents()：简单，但会一次性加载到内存
$content = file_get_contents('small-file.txt');

// fopen()/fread()：复杂，但可以流式处理
$handle = fopen('large-file.txt', 'rb');
$content = '';
while (($chunk = fread($handle, 8192)) !== false) {
    if ($chunk === '') {
        break;
    }
    $content .= $chunk;
}
fclose($handle);
```

### 问题 2：如何逐行读取大文件？

**回答**：使用 `fgets()` 逐行读取，避免一次性加载整个文件。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen('large-log.txt', 'rb');
if ($handle !== false) {
    try {
        while (($line = fgets($handle)) !== false) {
            // 处理每一行，不占用太多内存
            echo trim($line) . "\n";
        }
    } finally {
        fclose($handle);
    }
}
```

### 问题 3：读取文件时如何处理编码问题？

**回答**：使用 `mb_convert_encoding()` 或 `iconv()` 函数进行编码转换。

**示例**：

```php
<?php
declare(strict_types=1);

// 读取 GBK 编码的文件
$content = file_get_contents('gbk-file.txt');
if ($content !== false) {
    // 转换为 UTF-8
    $utf8Content = mb_convert_encoding($content, 'UTF-8', 'GBK');
    echo $utf8Content;
}
```

### 问题 4：文件不存在时如何处理？

**回答**：使用 `file_exists()` 检查文件是否存在，或检查函数返回值。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法 1：先检查文件是否存在
if (file_exists(__DIR__ . '/data.txt')) {
    $content = file_get_contents(__DIR__ . '/data.txt');
} else {
    echo "File does not exist\n";
}

// 方法 2：检查函数返回值（推荐）
$content = file_get_contents(__DIR__ . '/data.txt');
if ($content === false) {
    echo "Failed to read file\n";
} else {
    echo $content;
}
```

## 最佳实践

### 1. 小文件使用 file_get_contents()

对于小文件（< 10MB），使用 `file_get_contents()` 简单高效。

**示例**：

```php
<?php
declare(strict_types=1);

$content = file_get_contents(__DIR__ . '/config.json');
if ($content !== false) {
    $config = json_decode($content, true);
}
```

### 2. 大文件使用流式读取

对于大文件，使用 `fopen()`/`fread()` 流式读取，避免内存溢出。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/large-file.txt', 'rb');
if ($handle !== false) {
    try {
        while (($chunk = fread($handle, 8192)) !== false) {
            if ($chunk === '') {
                break;
            }
            // 处理每一块
            processChunk($chunk);
        }
    } finally {
        fclose($handle);
    }
}
```

### 3. 始终检查文件是否存在

读取文件前检查文件是否存在和可读。

**示例**：

```php
<?php
declare(strict_types=1);

function safeRead(string $filePath): ?string
{
    if (!file_exists($filePath)) {
        return null;
    }
    
    if (!is_readable($filePath)) {
        return null;
    }
    
    $content = file_get_contents($filePath);
    return $content !== false ? $content : null;
}
```

### 4. 使用 try-finally 确保文件关闭

使用 `try-finally` 确保文件句柄总是被关闭。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    // 文件操作
    $content = fread($handle, 1024);
} finally {
    fclose($handle);  // 总是执行
}
```

### 5. 使用二进制模式确保跨平台兼容

即使是文本文件，也推荐使用二进制模式（`rb`），确保跨平台兼容。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：使用二进制模式
$handle = fopen(__DIR__ . '/data.txt', 'rb');

// 不推荐：文本模式（Windows 上可能有问题）
// $handle = fopen(__DIR__ . '/data.txt', 'r');
```

## 对比分析

### file_get_contents() vs fopen()/fread()

| 特性         | file_get_contents()              | fopen()/fread()                  |
|:-------------|:--------------------------------|:--------------------------------|
| **代码复杂度** | ✅ 简单                         | ⚠️ 较复杂                        |
| **内存使用**   | ⚠️ 一次性加载到内存              | ✅ 流式处理，内存友好             |
| **适用场景**   | 小文件（< 10MB）                 | 大文件（> 10MB）                 |
| **功能灵活性** | ⚠️ 功能有限                      | ✅ 功能丰富（可控制读取过程）     |
| **性能**       | ✅ 小文件时更快                   | ⚠️ 有额外开销（函数调用）         |

### fgets() vs file()

| 特性         | fgets()                          | file()                           |
|:-------------|:--------------------------------|:--------------------------------|
| **内存使用**   | ✅ 逐行读取，内存友好             | ⚠️ 一次性加载到内存              |
| **适用场景**   | 大文件、逐行处理                 | 小文件、需要所有行               |
| **返回值**     | 字符串（一行）                    | 数组（所有行）                    |
| **性能**       | ✅ 大文件时更好                   | ⚠️ 大文件时可能内存溢出           |

## 练习任务

1. **安全文件读取函数**：编写一个函数，安全地读取文件，包含完整的错误处理和文件存在检查。

2. **大文件流式读取**：实现一个函数，使用流式方式读取大文件，每次读取指定大小的块，并提供一个回调函数处理每一块。

3. **CSV 文件读取器**：创建一个 CSV 文件读取器类，支持读取标题行、跳过空行、处理不同的分隔符。

4. **日志文件分析工具**：编写一个工具，逐行读取日志文件，统计不同级别的日志数量（ERROR、WARNING、INFO）。

5. **文件编码转换工具**：创建一个函数，读取指定编码的文件，并转换为 UTF-8 编码。

## 相关章节

- **[4.2.2 文件写入](section-02-file-writing.md)**：学习如何写入文件
- **[4.2.3 文件操作](section-03-file-operations.md)**：学习文件的复制、移动、删除等操作
- **[4.2.4 文件指针](section-04-file-pointer.md)**：深入了解文件指针的使用
- **[4.2.5 二进制文件](section-05-binary-files.md)**：学习二进制文件的处理方法
