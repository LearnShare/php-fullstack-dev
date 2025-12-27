# 4.2.10 大文件处理

## 概述

大文件处理是文件系统操作中的常见挑战。当处理大文件（> 10MB）时，如果使用传统的文件读取方法（如 `file_get_contents()`），会将整个文件加载到内存中，可能导致内存溢出、执行超时等问题。因此，需要使用特殊的内存优化策略，如分块读取、流式处理、生成器等。

理解大文件处理的挑战、分块读取策略、内存优化方法，以及如何使用生成器和流式处理来高效处理大文件，对于构建健壮的 PHP 应用至关重要。

**主要内容**：
- 大文件处理的挑战
- 分块读取策略
- 内存优化方法
- 生成器的使用
- 流式处理
- 大文件上传处理
- 性能优化技巧
- 实际应用场景和最佳实践

## 特性

- **内存高效**：避免一次性加载整个文件到内存
- **流式处理**：支持流式读取和写入，节省内存
- **进度跟踪**：支持跟踪处理进度
- **性能优化**：优化处理速度，避免超时
- **灵活配置**：支持自定义块大小和处理逻辑

## 语法/定义

### Generator（生成器）

生成器是 PHP 5.5+ 引入的功能，可以使用 `yield` 关键字创建可迭代的对象，用于高效处理大量数据，避免一次性加载到内存。

**基本语法**：

```php
function generator(): Generator
{
    yield $value;
}
```

## 基本用法

### 示例 1：分块读取大文件

```php
<?php
declare(strict_types=1);

function readLargeFileChunks(string $filename, int $chunkSize = 8192): Generator
{
    $handle = fopen($filename, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$filename}");
    }
    
    try {
        while (($chunk = fread($handle, $chunkSize)) !== false) {
            if ($chunk === '') {
                break;
            }
            yield $chunk;
        }
    } finally {
        fclose($handle);
    }
}

// 使用生成器处理大文件
foreach (readLargeFileChunks(__DIR__ . '/large-file.txt') as $chunk) {
    // 处理每个块，不会占用太多内存
    processChunk($chunk);
}
```

**说明**：
- 使用生成器分块读取文件，避免一次性加载到内存
- `$chunkSize` 控制每次读取的字节数（推荐 8KB）
- 适合处理大文件

### 示例 2：逐行读取大文件

```php
<?php
declare(strict_types=1);

function readLargeFileLines(string $filename): Generator
{
    $handle = fopen($filename, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$filename}");
    }
    
    try {
        while (($line = fgets($handle)) !== false) {
            yield trim($line);
        }
    } finally {
        fclose($handle);
    }
}

// 逐行处理大文件
foreach (readLargeFileLines(__DIR__ . '/large-log.txt') as $line) {
    // 处理每一行
    processLine($line);
}
```

**说明**：
- 使用生成器逐行读取文件
- 适合处理大文本文件（如日志文件）
- 不会一次性加载整个文件到内存

### 示例 3：流式复制大文件

```php
<?php
declare(strict_types=1);

function copyLargeFile(string $source, string $destination, int $chunkSize = 8192): void
{
    $sourceHandle = fopen($source, 'rb');
    if ($sourceHandle === false) {
        throw new RuntimeException("Cannot open source file: {$source}");
    }
    
    $destHandle = fopen($destination, 'wb');
    if ($destHandle === false) {
        fclose($sourceHandle);
        throw new RuntimeException("Cannot create destination file: {$destination}");
    }
    
    try {
        // 流式复制
        while (($chunk = fread($sourceHandle, $chunkSize)) !== false) {
            if ($chunk === '') {
                break;
            }
            if (fwrite($destHandle, $chunk) === false) {
                throw new RuntimeException("Cannot write to destination file");
            }
        }
    } finally {
        fclose($sourceHandle);
        fclose($destHandle);
    }
}

copyLargeFile(__DIR__ . '/large-file.zip', __DIR__ . '/copy.zip');
```

**说明**：
- 使用流式复制，避免一次性加载整个文件
- 适合复制大文件
- 可以控制块大小

### 示例 4：计算大文件的哈希值

```php
<?php
declare(strict_types=1);

function calculateLargeFileHash(string $filename, string $algorithm = 'sha256'): string
{
    $handle = fopen($filename, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$filename}");
    }
    
    $hash = hash_init($algorithm);
    
    try {
        // 流式读取并更新哈希
        while (($chunk = fread($handle, 8192)) !== false) {
            if ($chunk === '') {
                break;
            }
            hash_update($hash, $chunk);
        }
        
        return hash_final($hash);
    } finally {
        fclose($handle);
    }
}

$hash = calculateLargeFileHash(__DIR__ . '/large-file.zip');
echo "SHA256: {$hash}\n";
```

**说明**：
- 使用流式读取计算大文件的哈希值
- 避免一次性加载整个文件
- 适合处理大文件

### 示例 5：带进度跟踪的大文件处理

```php
<?php
declare(strict_types=1);

function processLargeFileWithProgress(
    string $filename,
    callable $processor,
    callable $progressCallback = null
): void {
    $fileSize = filesize($filename);
    if ($fileSize === false) {
        throw new RuntimeException("Cannot get file size: {$filename}");
    }
    
    $handle = fopen($filename, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$filename}");
    }
    
    $processed = 0;
    $chunkSize = 8192;
    
    try {
        while (($chunk = fread($handle, $chunkSize)) !== false) {
            if ($chunk === '') {
                break;
            }
            
            // 处理块
            $processor($chunk);
            
            // 更新进度
            $processed += strlen($chunk);
            if ($progressCallback !== null) {
                $progress = ($processed / $fileSize) * 100;
                $progressCallback($progress, $processed, $fileSize);
            }
        }
    } finally {
        fclose($handle);
    }
}

// 使用
processLargeFileWithProgress(
    __DIR__ . '/large-file.txt',
    function ($chunk) {
        // 处理块
        echo $chunk;
    },
    function ($progress, $processed, $total) {
        echo sprintf("Progress: %.2f%% (%d/%d bytes)\n", $progress, $processed, $total);
    }
);
```

**说明**：
- 支持进度跟踪，可以显示处理进度
- 适合需要用户反馈的场景

### 示例 6：大文件上传处理

```php
<?php
declare(strict_types=1);

function handleLargeFileUpload(string $tmpFile, string $destination): bool
{
    $sourceHandle = fopen($tmpFile, 'rb');
    if ($sourceHandle === false) {
        return false;
    }
    
    $destHandle = fopen($destination, 'wb');
    if ($destHandle === false) {
        fclose($sourceHandle);
        return false;
    }
    
    try {
        // 流式复制
        while (($chunk = fread($sourceHandle, 8192)) !== false) {
            if ($chunk === '') {
                break;
            }
            if (fwrite($destHandle, $chunk) === false) {
                return false;
            }
        }
        
        return true;
    } finally {
        fclose($sourceHandle);
        fclose($destHandle);
    }
}

// 处理上传的文件
if (isset($_FILES['file'])) {
    $tmpFile = $_FILES['file']['tmp_name'];
    $destination = __DIR__ . '/uploads/' . $_FILES['file']['name'];
    
    if (handleLargeFileUpload($tmpFile, $destination)) {
        echo "File uploaded successfully\n";
    }
}
```

**说明**：
- 使用流式处理上传的大文件
- 避免内存溢出
- 适合处理大文件上传

## 使用场景

### 场景 1：日志文件分析

分析大型日志文件，逐行处理。

**示例**：

```php
<?php
declare(strict_types=1);

function analyzeLargeLogFile(string $logFile): array
{
    $errors = [];
    $warnings = [];
    $info = [];
    
    $handle = fopen($logFile, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open log file: {$logFile}");
    }
    
    try {
        while (($line = fgets($handle)) !== false) {
            $line = trim($line);
            if (str_contains($line, 'ERROR')) {
                $errors[] = $line;
            } elseif (str_contains($line, 'WARNING')) {
                $warnings[] = $line;
            } else {
                $info[] = $line;
            }
        }
    } finally {
        fclose($handle);
    }
    
    return [
        'errors' => count($errors),
        'warnings' => count($warnings),
        'info' => count($info),
    ];
}
```

### 场景 2：数据导入导出

导入导出大量数据。

**示例**：

```php
<?php
declare(strict_types=1);

function exportLargeDatasetToCsv(string $outputFile, callable $dataProvider): void
{
    $handle = fopen($outputFile, 'wb');
    if ($handle === false) {
        throw new RuntimeException("Cannot create output file: {$outputFile}");
    }
    
    try {
        // 写入标题行
        fputcsv($handle, ['ID', 'Name', 'Email']);
        
        // 流式写入数据
        foreach ($dataProvider() as $row) {
            fputcsv($handle, $row);
        }
    } finally {
        fclose($handle);
    }
}
```

## 注意事项

### 内存使用监控

处理大文件时，监控内存使用情况。

**示例**：

```php
<?php
declare(strict_types=1);

function processLargeFileWithMemoryCheck(string $filename, callable $processor): void
{
    $initialMemory = memory_get_usage();
    
    $handle = fopen($filename, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$filename}");
    }
    
    try {
        while (($chunk = fread($handle, 8192)) !== false) {
            if ($chunk === '') {
                break;
            }
            
            $processor($chunk);
            
            // 检查内存使用
            $currentMemory = memory_get_usage();
            $usedMemory = $currentMemory - $initialMemory;
            if ($usedMemory > 100 * 1024 * 1024) {  // 100MB
                echo "Warning: High memory usage: " . ($usedMemory / 1024 / 1024) . " MB\n";
            }
        }
    } finally {
        fclose($handle);
    }
}
```

### 执行时间限制

处理大文件可能需要较长时间，注意执行时间限制。

**示例**：

```php
<?php
declare(strict_types=1);

// 设置执行时间限制（0 表示无限制，不推荐在生产环境使用）
set_time_limit(0);

// 或者在处理过程中检查时间
function processWithTimeCheck(string $filename, callable $processor, int $maxTime = 300): void
{
    $startTime = time();
    
    $handle = fopen($filename, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$filename}");
    }
    
    try {
        while (($chunk = fread($handle, 8192)) !== false) {
            if ($chunk === '') {
                break;
            }
            
            $processor($chunk);
            
            // 检查执行时间
            if ((time() - $startTime) > $maxTime) {
                echo "Processing timeout\n";
                break;
            }
        }
    } finally {
        fclose($handle);
    }
}
```

### 块大小的选择

选择合适的块大小，平衡性能和内存使用。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐的块大小
$chunkSizes = [
    'small' => 4096,   // 4KB，适合小文件
    'medium' => 8192,  // 8KB，推荐的默认值
    'large' => 65536,  // 64KB，适合大文件和高性能场景
];

// 根据文件大小选择块大小
$fileSize = filesize(__DIR__ . '/large-file.txt');
$chunkSize = $fileSize > 100 * 1024 * 1024 ? 65536 : 8192;  // 大于100MB使用64KB，否则使用8KB
```

## 常见问题

### 问题 1：如何避免内存溢出？

**回答**：使用流式处理，避免一次性加载整个文件到内存。使用生成器分块读取，及时释放资源。

**示例**：见"示例 1：分块读取大文件"

### 问题 2：大文件读取的最佳块大小是什么？

**回答**：推荐使用 8KB（8192 字节）作为默认块大小。对于非常大的文件，可以考虑使用 64KB。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐的块大小
$chunkSize = 8192;  // 8KB，推荐的默认值

// 对于非常大的文件
$largeChunkSize = 65536;  // 64KB
```

### 问题 3：如何处理超时问题？

**回答**：增加执行时间限制，或者在处理过程中检查时间，必要时暂停处理。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法 1：增加执行时间限制
set_time_limit(300);  // 5分钟

// 方法 2：在处理过程中检查时间
$startTime = time();
$maxTime = 300;  // 5分钟

while (($chunk = fread($handle, 8192)) !== false) {
    // 处理块
    
    // 检查时间
    if ((time() - $startTime) > $maxTime) {
        break;
    }
}
```

### 问题 4：如何显示处理进度？

**回答**：在处理过程中计算已处理的字节数和百分比，通过回调函数或输出显示进度。

**示例**：见"示例 5：带进度跟踪的大文件处理"

## 最佳实践

### 1. 使用分块读取处理大文件

对于大文件，始终使用分块读取，避免一次性加载到内存。

**示例**：见"示例 1：分块读取大文件"

### 2. 使用生成器减少内存占用

使用生成器（Generator）处理大量数据，避免创建大型数组。

**示例**：

```php
<?php
declare(strict_types=1);

function readLargeFileLines(string $filename): Generator
{
    $handle = fopen($filename, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$filename}");
    }
    
    try {
        while (($line = fgets($handle)) !== false) {
            yield trim($line);
        }
    } finally {
        fclose($handle);
    }
}

// 使用生成器，不会一次性加载所有行到内存
foreach (readLargeFileLines(__DIR__ . '/large-file.txt') as $line) {
    processLine($line);
}
```

### 3. 监控内存和执行时间

处理大文件时，监控内存使用和执行时间。

**示例**：

```php
<?php
declare(strict_types=1);

$initialMemory = memory_get_usage();
$startTime = microtime(true);

// 处理大文件
processLargeFile(__DIR__ . '/large-file.txt');

$usedMemory = memory_get_usage() - $initialMemory;
$executionTime = microtime(true) - $startTime;

echo "Memory used: " . ($usedMemory / 1024 / 1024) . " MB\n";
echo "Execution time: {$executionTime} seconds\n";
```

### 4. 提供进度反馈机制

对于长时间运行的处理，提供进度反馈。

**示例**：见"示例 5：带进度跟踪的大文件处理"

### 5. 使用流式处理

对于大文件的读写，使用流式处理。

**示例**：见"示例 3：流式复制大文件"

## 对比分析

### 一次性读取 vs 分块读取

| 特性         | 一次性读取（file_get_contents()） | 分块读取（fread()）          |
|:-------------|:----------------------------------|:-----------------------------|
| **内存使用** | ⚠️ 整个文件加载到内存             | ✅ 只加载一小块数据           |
| **适用大小** | 小文件（< 10MB）                  | 大文件（> 10MB）              |
| **性能**     | ✅ 小文件时更快                   | ⚠️ 有额外开销（函数调用）     |
| **风险**     | ⚠️ 可能导致内存溢出               | ✅ 内存安全                   |

### 生成器 vs 数组

| 特性         | 数组                               | 生成器（Generator）           |
|:-------------|:-----------------------------------|:------------------------------|
| **内存使用** | ⚠️ 所有数据加载到内存              | ✅ 逐个生成，内存友好          |
| **适用场景** | 数据量较小                         | 数据量较大                    |
| **性能**     | ✅ 随机访问快                      | ⚠️ 只能顺序访问               |
| **灵活性**   | ⚠️ 需要预先知道所有数据            | ✅ 可以按需生成               |

## 练习任务

1. **大文件处理工具类**：创建一个工具类，封装大文件处理的常用操作（分块读取、流式复制等）。

2. **日志文件分析工具**：实现一个工具，分析大型日志文件，统计不同级别的日志数量。

3. **大文件哈希计算工具**：编写一个函数，使用流式方式计算大文件的哈希值。

4. **带进度的大文件复制工具**：创建一个工具，复制大文件并显示进度。

5. **大文件搜索工具**：实现一个工具，在大文件中搜索特定内容，支持正则表达式。

## 相关章节

- **[4.2.1 文件读取](section-01-file-reading.md)**：了解文件读取的基础知识
- **[4.2.2 文件写入](section-02-file-writing.md)**：了解文件写入的基础知识
- **[4.2.9 流处理](section-09-streaming.md)**：学习流处理的相关内容
