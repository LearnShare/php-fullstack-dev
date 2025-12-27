# 4.2.2 文件写入

## 概述

文件写入是文件系统操作的重要功能。在 PHP 开发中，我们经常需要写入文件，比如保存配置、记录日志、生成报告、保存用户数据等。PHP 提供了多种文件写入方法，每种方法适用于不同的场景。理解不同写入方法的工作原理、适用场景，以及如何安全高效地写入文件，对于构建健壮的 PHP 应用至关重要。

文件写入操作需要考虑多个因素：写入模式（覆盖还是追加）、文件大小（小文件还大文件）、并发控制（是否需要文件锁）、错误处理、权限检查等。选择正确的写入方法可以提高代码的性能和可靠性。

掌握文件写入的基础知识后，可以进一步学习文件操作、文件锁定、二进制文件处理等高级主题。

**主要内容**：
- 文件写入方法概述
- `file_put_contents()` 函数
- `fopen()` 和 `fwrite()` 函数
- 追加写入模式
- 文件锁定写入
- `fputcsv()` 写入 CSV 文件
- 原子写入
- 错误处理和最佳实践

## 特性

- **多种写入方式**：支持一次性写入、流式写入、追加写入等多种方式
- **并发控制**：支持文件锁定，防止并发写入冲突
- **灵活的数据类型**：支持字符串、数组等数据类型的写入
- **完善的错误处理**：提供错误检查和处理机制
- **跨平台兼容**：支持 Windows 和类 Unix 系统

## 语法/定义

### file_put_contents() 函数

**语法**：`file_put_contents(string $filename, mixed $data, int $flags = 0, ?resource $context = null): int|false`

**参数**：
- `$filename`：要写入的文件路径（相对或绝对路径）
- `$data`：要写入的数据，可以是字符串或数组
- `$flags`：可选，写入标志（可以使用 `|` 组合），默认为 0。可选值：
  - `FILE_APPEND`：追加到文件末尾，而不是覆盖
  - `FILE_USE_INCLUDE_PATH`：在 `include_path` 中搜索文件
  - `LOCK_EX`：获取排他锁（防止并发写入）
- `$context`：可选，流上下文资源

**返回值**：成功返回写入的字节数，失败返回 `false`。

### fwrite() 函数

**语法**：`fwrite(resource $stream, string $data, ?int $length = null): int|false`

**参数**：
- `$stream`：文件句柄
- `$data`：要写入的数据
- `$length`：可选，写入的最大字节数，默认写入全部数据

**返回值**：成功返回写入的字节数，失败返回 `false`。

### fputcsv() 函数

**语法**：`fputcsv(resource $stream, array $fields, string $separator = ",", string $enclosure = "\"", string $escape = "\\"): int|false`

**参数**：
- `$stream`：文件句柄
- `$fields`：要写入的字段数组
- `$separator`：字段分隔符，默认为 `,`
- `$enclosure`：字段包围符，默认为 `"`
- `$escape`：转义字符，默认为 `\`

**返回值**：成功返回写入的字节数，失败返回 `false`。

## 基本用法

### 示例 1：使用 file_put_contents() 覆盖写入

```php
<?php
declare(strict_types=1);

// 覆盖写入
$bytes = file_put_contents(__DIR__ . '/output.txt', 'Hello, World!');
if ($bytes === false) {
    throw new RuntimeException('Cannot write file');
}
echo "Written {$bytes} bytes\n";

// 写入数组（会自动转换为字符串）
$data = ['Line 1', 'Line 2', 'Line 3'];
$bytes = file_put_contents(__DIR__ . '/output.txt', $data);
// 等同于：file_put_contents(__DIR__ . '/output.txt', implode('', $data));
```

**输出**：

```
Written 13 bytes
```

**说明**：
- `file_put_contents()` 默认会覆盖文件内容
- 返回写入的字节数
- 可以写入字符串或数组（数组会自动转换）

### 示例 2：使用 file_put_contents() 追加写入

```php
<?php
declare(strict_types=1);

// 追加写入
file_put_contents(__DIR__ . '/log.txt', "Log entry 1\n", FILE_APPEND);
file_put_contents(__DIR__ . '/log.txt', "Log entry 2\n", FILE_APPEND);

// 追加并加锁（防止并发写入）
file_put_contents(__DIR__ . '/log.txt', "Log entry 3\n", FILE_APPEND | LOCK_EX);
```

**说明**：
- `FILE_APPEND` 标志表示追加到文件末尾
- `LOCK_EX` 标志表示获取排他锁，防止并发写入冲突
- 可以组合使用多个标志（使用 `|` 运算符）

### 示例 3：使用 fopen() 和 fwrite() 写入文件

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/output.txt', 'wb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    // 写入数据
    $bytes1 = fwrite($handle, 'Line 1\n');
    $bytes2 = fwrite($handle, 'Line 2\n');
    
    echo "Written {$bytes1} bytes (Line 1)\n";
    echo "Written {$bytes2} bytes (Line 2)\n";
    
    // 限制写入长度
    $data = 'Hello, World!';
    $bytes3 = fwrite($handle, $data, 5);  // 只写入 "Hello"
    echo "Written {$bytes3} bytes (Hello)\n";
} finally {
    fclose($handle);
}
```

**输出**：

```
Written 7 bytes (Line 1)
Written 7 bytes (Line 2)
Written 5 bytes (Hello)
```

**说明**：
- `fopen()` 打开文件返回文件句柄
- `fwrite()` 从当前文件指针位置写入数据
- 可以指定写入的最大长度
- 使用完毕后必须调用 `fclose()` 关闭文件句柄

### 示例 4：追加模式写入

```php
<?php
declare(strict_types=1);

// 追加模式打开文件
$handle = fopen(__DIR__ . '/log.txt', 'ab');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    fwrite($handle, "New log entry\n");
    // 文件指针自动定位到文件末尾
} finally {
    fclose($handle);
}
```

**说明**：
- `a` 模式表示追加模式，文件指针自动定位到文件末尾
- 写入的数据会自动追加到文件末尾
- `ab` 表示追加模式的二进制模式（推荐）

### 示例 5：使用文件锁写入

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/log.txt', 'ab');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    // 获取排他锁
    if (flock($handle, LOCK_EX)) {
        fwrite($handle, "Locked log entry\n");
        // 解锁（可选，文件关闭时自动解锁）
        flock($handle, LOCK_UN);
    } else {
        throw new RuntimeException('Cannot acquire lock');
    }
} finally {
    fclose($handle);
}
```

**说明**：
- `flock()` 用于获取文件锁
- `LOCK_EX` 表示排他锁（写锁）
- `LOCK_UN` 表示解锁
- 文件关闭时自动释放锁

### 示例 6：使用 fputcsv() 写入 CSV 文件

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.csv', 'wb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    // 写入标题行
    fputcsv($handle, ['Name', 'Age', 'Email']);
    
    // 写入数据行
    fputcsv($handle, ['Alice', 25, 'alice@example.com']);
    fputcsv($handle, ['Bob', 30, 'bob@example.com']);
    fputcsv($handle, ['Charlie', 35, 'charlie@example.com']);
} finally {
    fclose($handle);
}
```

**输出**（CSV 文件内容）：

```
Name,Age,Email
Alice,25,alice@example.com
Bob,30,bob@example.com
Charlie,35,charlie@example.com
```

**说明**：
- `fputcsv()` 自动处理 CSV 格式（分隔符、引号、转义）
- 字段数组中的值会自动转换为字符串
- 自动处理包含逗号、引号等特殊字符的字段

### 示例 7：流式写入大文件

```php
<?php
declare(strict_types=1);

// 流式写入大文件
$handle = fopen(__DIR__ . '/large-output.txt', 'wb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    // 模拟大量数据写入
    for ($i = 0; $i < 1000; $i++) {
        $data = "Line {$i}\n";
        fwrite($handle, $data);
    }
} finally {
    fclose($handle);
}
```

**说明**：
- 流式写入适合大文件，避免一次性将所有数据加载到内存
- 可以多次调用 `fwrite()` 写入数据
- 每次写入后文件指针会自动移动

### 示例 8：原子写入

```php
<?php
declare(strict_types=1);

function atomicWrite(string $filePath, string $data): bool
{
    // 先写入临时文件
    $tempFile = $filePath . '.tmp';
    $bytes = file_put_contents($tempFile, $data, LOCK_EX);
    if ($bytes === false) {
        return false;
    }
    
    // 原子性地重命名（在大多数系统上是原子操作）
    return rename($tempFile, $filePath);
}

// 使用原子写入
if (atomicWrite(__DIR__ . '/config.json', '{"key": "value"}')) {
    echo "File written atomically\n";
} else {
    echo "Failed to write file\n";
}
```

**说明**：
- 原子写入先写入临时文件，然后重命名
- 可以确保文件要么完全写入，要么不存在（不会出现部分写入的情况）
- 适合关键数据的写入

## 使用场景

### 场景 1：写入配置文件

保存应用配置到文件。

**示例**：

```php
<?php
declare(strict_types=1);

function saveConfig(string $configPath, array $config): bool
{
    $json = json_encode($config, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
    if ($json === false) {
        return false;
    }
    
    $bytes = file_put_contents($configPath, $json, LOCK_EX);
    return $bytes !== false;
}

$config = [
    'database' => [
        'host' => 'localhost',
        'port' => 3306,
        'name' => 'myapp',
    ],
    'cache' => [
        'enabled' => true,
        'ttl' => 3600,
    ],
];

if (saveConfig(__DIR__ . '/config.json', $config)) {
    echo "Config saved\n";
}
```

### 场景 2：写入日志文件

记录日志到文件。

**示例**：

```php
<?php
declare(strict_types=1);

function writeLog(string $logPath, string $message, string $level = 'INFO'): void
{
    $timestamp = date('Y-m-d H:i:s');
    $logEntry = "[{$timestamp}] [{$level}] {$message}\n";
    
    // 追加写入并使用文件锁
    file_put_contents($logPath, $logEntry, FILE_APPEND | LOCK_EX);
}

writeLog(__DIR__ . '/app.log', 'Application started', 'INFO');
writeLog(__DIR__ . '/app.log', 'User logged in', 'INFO');
writeLog(__DIR__ . '/app.log', 'Database connection failed', 'ERROR');
```

### 场景 3：生成 CSV 报告

将数据导出为 CSV 文件。

**示例**：

```php
<?php
declare(strict_types=1);

function exportToCsv(string $filePath, array $data): bool
{
    $handle = fopen($filePath, 'wb');
    if ($handle === false) {
        return false;
    }
    
    try {
        // 写入标题行
        if (!empty($data)) {
            $headers = array_keys($data[0]);
            fputcsv($handle, $headers);
            
            // 写入数据行
            foreach ($data as $row) {
                fputcsv($handle, array_values($row));
            }
        }
        
        return true;
    } finally {
        fclose($handle);
    }
}

$data = [
    ['name' => 'Alice', 'age' => 25, 'email' => 'alice@example.com'],
    ['name' => 'Bob', 'age' => 30, 'email' => 'bob@example.com'],
];

if (exportToCsv(__DIR__ . '/report.csv', $data)) {
    echo "CSV exported\n";
}
```

## 注意事项

### 文件权限

写入文件前需要确保目录可写，文件可写（如果文件已存在）。

**示例**：

```php
<?php
declare(strict_types=1);

function canWrite(string $filePath): bool
{
    $dir = dirname($filePath);
    
    // 检查目录是否存在且可写
    if (!is_dir($dir) || !is_writable($dir)) {
        return false;
    }
    
    // 如果文件已存在，检查是否可写
    if (file_exists($filePath) && !is_writable($filePath)) {
        return false;
    }
    
    return true;
}

$filePath = __DIR__ . '/output.txt';
if (canWrite($filePath)) {
    file_put_contents($filePath, 'Hello, World!');
} else {
    echo "Cannot write to file\n";
}
```

### 并发写入处理

在多进程或多线程环境中，使用文件锁防止并发写入冲突。

**示例**：

```php
<?php
declare(strict_types=1);

function safeWrite(string $filePath, string $data): bool
{
    $handle = fopen($filePath, 'ab');
    if ($handle === false) {
        return false;
    }
    
    try {
        // 获取排他锁
        if (flock($handle, LOCK_EX)) {
            fwrite($handle, $data);
            flock($handle, LOCK_UN);
            return true;
        }
        
        return false;
    } finally {
        fclose($handle);
    }
}
```

### 大文件处理

对于大文件，使用流式写入而不是一次性写入，避免内存溢出。

**示例**：

```php
<?php
declare(strict_types=1);

function writeLargeFile(string $filePath, array $data): bool
{
    $handle = fopen($filePath, 'wb');
    if ($handle === false) {
        return false;
    }
    
    try {
        foreach ($data as $chunk) {
            $bytes = fwrite($handle, $chunk);
            if ($bytes === false) {
                return false;
            }
        }
        
        return true;
    } finally {
        fclose($handle);
    }
}
```

### 错误处理

始终检查文件操作的返回值，进行适当的错误处理。

**示例**：

```php
<?php
declare(strict_types=1);

function safeFilePutContents(string $filePath, string $data): bool
{
    $bytes = file_put_contents($filePath, $data, LOCK_EX);
    if ($bytes === false) {
        error_log("Failed to write file: {$filePath}");
        return false;
    }
    
    return true;
}
```

## 常见问题

### 问题 1：file_put_contents() 和 fopen()/fwrite() 的区别是什么？

**回答**：
- **file_put_contents()**：一次性写入整个文件，适合小文件，代码简单
- **fopen()/fwrite()**：流式写入，适合大文件，可以多次写入，功能更灵活

**示例**：

```php
<?php
declare(strict_types=1);

// file_put_contents()：简单，但需要一次性提供所有数据
file_put_contents('output.txt', 'All data at once');

// fopen()/fwrite()：复杂，但可以分多次写入
$handle = fopen('output.txt', 'wb');
fwrite($handle, 'First chunk');
fwrite($handle, 'Second chunk');
fwrite($handle, 'Third chunk');
fclose($handle);
```

### 问题 2：如何追加内容到文件？

**回答**：使用 `FILE_APPEND` 标志或 `a` 模式。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法 1：使用 file_put_contents() 的 FILE_APPEND 标志
file_put_contents('log.txt', "New entry\n", FILE_APPEND);

// 方法 2：使用 fopen() 的 'a' 模式
$handle = fopen('log.txt', 'ab');
fwrite($handle, "New entry\n");
fclose($handle);
```

### 问题 3：写入文件时如何处理权限问题？

**回答**：检查目录和文件的写入权限，使用 `is_writable()` 函数。

**示例**：见"注意事项"部分的示例

### 问题 4：如何防止并发写入冲突？

**回答**：使用文件锁（`LOCK_EX` 标志或 `flock()` 函数）。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法 1：使用 file_put_contents() 的 LOCK_EX 标志
file_put_contents('log.txt', "Entry\n", FILE_APPEND | LOCK_EX);

// 方法 2：使用 flock() 函数
$handle = fopen('log.txt', 'ab');
if (flock($handle, LOCK_EX)) {
    fwrite($handle, "Entry\n");
    flock($handle, LOCK_UN);
}
fclose($handle);
```

## 最佳实践

### 1. 小文件使用 file_put_contents()

对于小文件（< 10MB），使用 `file_put_contents()` 简单高效。

**示例**：

```php
<?php
declare(strict_types=1);

$config = json_encode(['key' => 'value'], JSON_PRETTY_PRINT);
file_put_contents(__DIR__ . '/config.json', $config, LOCK_EX);
```

### 2. 大文件使用流式写入

对于大文件，使用 `fopen()`/`fwrite()` 流式写入，避免内存溢出。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/large-output.txt', 'wb');
if ($handle !== false) {
    try {
        foreach ($dataChunks as $chunk) {
            fwrite($handle, $chunk);
        }
    } finally {
        fclose($handle);
    }
}
```

### 3. 使用文件锁保护并发写入

在多进程环境中，使用文件锁防止并发写入冲突。

**示例**：

```php
<?php
declare(strict_types=1);

file_put_contents(__DIR__ . '/log.txt', $message, FILE_APPEND | LOCK_EX);
```

### 4. 检查写入权限和磁盘空间

写入文件前检查目录和文件的写入权限，以及磁盘空间。

**示例**：

```php
<?php
declare(strict_types=1);

function canWriteFile(string $filePath): bool
{
    $dir = dirname($filePath);
    return is_dir($dir) && is_writable($dir);
}

if (canWriteFile(__DIR__ . '/output.txt')) {
    file_put_contents(__DIR__ . '/output.txt', $data);
}
```

### 5. 使用原子写入保护关键数据

对于关键数据，使用原子写入（先写临时文件，再重命名）。

**示例**：见"示例 8：原子写入"

### 6. 使用二进制模式确保跨平台兼容

即使是文本文件，也推荐使用二进制模式（`wb`、`ab`），确保跨平台兼容。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：使用二进制模式
$handle = fopen(__DIR__ . '/data.txt', 'wb');
$handle = fopen(__DIR__ . '/log.txt', 'ab');
```

## 对比分析

### file_put_contents() vs fopen()/fwrite()

| 特性         | file_put_contents()              | fopen()/fwrite()                 |
|:-------------|:--------------------------------|:--------------------------------|
| **代码复杂度** | ✅ 简单                         | ⚠️ 较复杂                        |
| **内存使用**   | ⚠️ 一次性加载到内存              | ✅ 流式处理，内存友好             |
| **适用场景**   | 小文件（< 10MB）                 | 大文件（> 10MB）                 |
| **功能灵活性** | ⚠️ 功能有限                      | ✅ 功能丰富（可多次写入、控制指针） |
| **性能**       | ✅ 小文件时更快                   | ⚠️ 有额外开销（函数调用）         |

### 覆盖写入 vs 追加写入

| 特性         | 覆盖写入（默认）                  | 追加写入（FILE_APPEND）          |
|:-------------|:--------------------------------|:--------------------------------|
| **文件指针位置** | 文件开头                         | 文件末尾                         |
| **原文件内容** | 被清空                           | 保留                             |
| **适用场景**   | 创建新文件、更新文件内容          | 日志文件、追加数据               |

## 练习任务

1. **安全文件写入函数**：编写一个函数，安全地写入文件，包含完整的错误处理、权限检查和文件锁。

2. **CSV 文件写入器**：创建一个 CSV 文件写入器类，支持写入标题行、数据行，处理不同的分隔符和编码。

3. **日志记录功能**：实现一个日志记录功能，支持追加写入、文件锁、日志级别、自动轮转。

4. **原子写入函数**：编写一个函数，实现文件的原子写入（先写入临时文件，再重命名）。

5. **流式写入工具**：创建一个函数，使用流式方式写入大文件，支持进度回调。

## 相关章节

- **[4.2.1 文件读取](section-01-file-reading.md)**：学习如何读取文件
- **[4.2.3 文件操作](section-03-file-operations.md)**：学习文件的复制、移动、删除等操作
- **[4.2.6 文件锁定](section-06-file-locks-temp.md)**：深入了解文件锁定的使用
