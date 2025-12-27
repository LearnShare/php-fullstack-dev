# 4.2.4 文件指针操作

## 概述

文件指针是文件操作中的核心概念，它指示当前在文件中的读取或写入位置。可以把它想象成文件中的一个"光标"，标记着下一次读写操作将从哪里开始。理解文件指针的工作原理和操作方法，对于精确控制文件读写位置、实现随机访问、断点续传等高级功能至关重要。

文件指针的初始位置取决于文件的打开模式，不同的文件操作会对文件指针产生不同的影响。掌握文件指针的操作方法，可以帮助我们实现更灵活和高效的文件处理逻辑，比如随机访问文件内容、实现断点续传、读取文件的特定部分等。

**主要内容**：
- 文件指针的概念和工作原理
- `ftell()` 函数（获取当前位置）
- `fseek()` 函数（移动文件指针）
- `rewind()` 函数（重置指针）
- `feof()` 函数（检查是否到达文件末尾）
- 文件指针与读写操作的关系
- 实际应用场景和最佳实践

## 特性

- **精确定位**：可以移动到文件的任意位置（如果支持）
- **灵活控制**：支持相对位置和绝对位置的移动
- **状态查询**：可以查询当前文件指针位置
- **随机访问**：支持随机访问文件内容
- **断点续传**：支持实现断点续传功能

## 语法/定义

### ftell() 函数

**语法**：`ftell(resource $stream): int|false`

**参数**：
- `$stream`：文件句柄

**返回值**：成功返回当前文件指针位置（字节数，从 0 开始），失败返回 `false`。

### fseek() 函数

**语法**：`fseek(resource $stream, int $offset, int $whence = SEEK_SET): int`

**参数**：
- `$stream`：文件句柄
- `$offset`：偏移量（字节数），可以为负数
- `$whence`：起始位置，可选值：
  - `SEEK_SET`（0）：从文件开头计算偏移量（默认）
  - `SEEK_CUR`（1）：从当前位置计算偏移量
  - `SEEK_END`（2）：从文件末尾计算偏移量

**返回值**：成功返回 `0`，失败返回 `-1`。

### rewind() 函数

**语法**：`rewind(resource $stream): bool`

**参数**：
- `$stream`：文件句柄

**返回值**：成功返回 `true`，失败返回 `false`。

**等价操作**：`rewind($handle)` 等价于 `fseek($handle, 0, SEEK_SET)`

### feof() 函数

**语法**：`feof(resource $stream): bool`

**参数**：
- `$stream`：文件句柄

**返回值**：如果文件指针到达文件末尾返回 `true`，否则返回 `false`。

## 基本用法

### 示例 1：文件指针的初始位置

```php
<?php
declare(strict_types=1);

// 创建测试文件
file_put_contents(__DIR__ . '/test.txt', 'Hello World');

// 只读模式：文件指针在开头（位置 0）
$handle = fopen(__DIR__ . '/test.txt', 'rb');
if ($handle !== false) {
    echo "Initial position: " . ftell($handle) . "\n";  // 0
    fclose($handle);
}

// 追加模式：文件指针在末尾
$handle = fopen(__DIR__ . '/test.txt', 'ab');
if ($handle !== false) {
    $size = filesize(__DIR__ . '/test.txt');
    echo "Initial position (append mode): " . ftell($handle) . "\n";  // 等于文件大小
    fclose($handle);
}
```

**输出**：

```
Initial position: 0
Initial position (append mode): 11
```

**说明**：
- `r`、`r+`、`w`、`w+` 等模式：文件指针在文件开头（位置 0）
- `a`、`a+` 模式：文件指针在文件末尾

### 示例 2：读取操作对文件指针的影响

```php
<?php
declare(strict_types=1);

file_put_contents(__DIR__ . '/test.txt', 'Hello World');

$handle = fopen(__DIR__ . '/test.txt', 'rb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// 初始位置：0
echo "Position: " . ftell($handle) . "\n";  // 0

// 读取 5 字节："Hello"
$chunk = fread($handle, 5);
echo "Read: {$chunk}\n";  // Hello
echo "Position: " . ftell($handle) . "\n";  // 5

// 读取 1 字节：" "（空格）
$chunk = fread($handle, 1);
echo "Read: '{$chunk}'\n";  // ' '
echo "Position: " . ftell($handle) . "\n";  // 6

// 读取剩余内容："World"
$chunk = fread($handle, 1024);
echo "Read: {$chunk}\n";  // World
echo "Position: " . ftell($handle) . "\n";  // 11（文件末尾）

fclose($handle);
```

**说明**：
- `fread()` 读取数据后，文件指针向前移动读取的字节数
- 每次读取后，指针移动到读取数据的末尾

### 示例 3：使用 ftell() 获取当前位置

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// 获取初始位置
$startPos = ftell($handle);
echo "Start position: {$startPos}\n";  // 0

// 读取数据
$chunk = fread($handle, 100);

// 获取当前位置
$currentPos = ftell($handle);
echo "Current position: {$currentPos}\n";  // 100

// 计算已读取的字节数
$bytesRead = $currentPos - $startPos;
echo "Bytes read: {$bytesRead}\n";  // 100

fclose($handle);
```

**说明**：
- `ftell()` 返回当前文件指针位置（字节数，从 0 开始）
- 可以用于计算已读取或已写入的字节数
- 可以用于实现进度跟踪

### 示例 4：使用 fseek() 移动文件指针

```php
<?php
declare(strict_types=1);

file_put_contents(__DIR__ . '/data.txt', str_repeat('0123456789', 10));  // 100 字节

$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// SEEK_SET：从文件开头计算偏移量
fseek($handle, 50, SEEK_SET);
echo "Position after SEEK_SET(50): " . ftell($handle) . "\n";  // 50

// SEEK_CUR：从当前位置计算偏移量
fseek($handle, 25, SEEK_CUR);
echo "Position after SEEK_CUR(25): " . ftell($handle) . "\n";  // 75

// SEEK_END：从文件末尾计算偏移量（负数表示向前）
fseek($handle, -10, SEEK_END);
echo "Position after SEEK_END(-10): " . ftell($handle) . "\n";  // 90

// 移动到文件开头
fseek($handle, 0, SEEK_SET);
echo "Position after SEEK_SET(0): " . ftell($handle) . "\n";  // 0

fclose($handle);
```

**输出**：

```
Position after SEEK_SET(50): 50
Position after SEEK_CUR(25): 75
Position after SEEK_END(-10): 90
Position after SEEK_SET(0): 0
```

**说明**：
- `SEEK_SET`：从文件开头计算偏移量
- `SEEK_CUR`：从当前位置计算偏移量（可以向前或向后）
- `SEEK_END`：从文件末尾计算偏移量（通常使用负数）

### 示例 5：使用 rewind() 重置文件指针

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// 读取文件
$content1 = fread($handle, 1024);
echo "First read: " . strlen($content1) . " bytes\n";
echo "Position after first read: " . ftell($handle) . "\n";

// 重置文件指针到开头
rewind($handle);
echo "Position after rewind: " . ftell($handle) . "\n";  // 0

// 再次读取文件
$content2 = fread($handle, 1024);
echo "Second read: " . strlen($content2) . " bytes\n";
echo "Position after second read: " . ftell($handle) . "\n";

fclose($handle);
```

**说明**：
- `rewind()` 将文件指针重置到文件开头（位置 0）
- 等价于 `fseek($handle, 0, SEEK_SET)`
- 用于重新读取文件或从开头重新写入

### 示例 6：使用 feof() 检查文件末尾

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// ✅ 正确：检查 fread() 的返回值
while (($chunk = fread($handle, 8192)) !== false) {
    if ($chunk === '') {
        break;  // 到达文件末尾
    }
    echo $chunk;
    
    // 检查是否到达文件末尾（可选）
    if (feof($handle)) {
        echo "\nReached end of file\n";
        break;
    }
}

// ❌ 错误：不要用 feof() 作为循环条件
// while (!feof($handle)) {
//     $chunk = fread($handle, 8192);
//     // 如果 fread() 返回 false，$chunk 可能是 false，但仍会继续循环
// }

fclose($handle);
```

**说明**：
- `feof()` 只有在尝试读取超出文件末尾的数据后才会返回 `true`
- 不要用 `feof()` 作为循环条件，应该检查 `fread()` 的返回值
- 应该在读取后检查 `feof()` 来判断是否到达文件末尾

### 示例 7：随机访问文件内容

```php
<?php
declare(strict_types=1);

function readFileAtPosition(string $file, int $position, int $length): string
{
    $handle = fopen($file, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$file}");
    }
    
    try {
        // 移动到指定位置
        if (fseek($handle, $position, SEEK_SET) !== 0) {
            throw new RuntimeException("Cannot seek to position: {$position}");
        }
        
        // 读取指定长度的数据
        $data = fread($handle, $length);
        return $data !== false ? $data : '';
    } finally {
        fclose($handle);
    }
}

// 使用：读取文件的中间部分（从第 100 字节开始，读取 50 字节）
$chunk = readFileAtPosition(__DIR__ . '/data.txt', 100, 50);
echo "Read from position 100: {$chunk}\n";
```

**说明**：
- 使用 `fseek()` 移动到指定位置
- 然后从该位置读取数据
- 实现随机访问文件内容

### 示例 8：实现断点续传

```php
<?php
declare(strict_types=1);

function resumeDownload(string $file, int $resumePosition): void
{
    $handle = fopen($file, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$file}");
    }
    
    try {
        $fileSize = filesize($file);
        if ($fileSize === false) {
            throw new RuntimeException("Cannot get file size");
        }
        
        // 移动到断点位置
        if (fseek($handle, $resumePosition, SEEK_SET) !== 0) {
            throw new RuntimeException("Cannot seek to position: {$resumePosition}");
        }
        
        // 设置 HTTP 头
        header('Content-Range: bytes ' . $resumePosition . '-' . ($fileSize - 1) . '/' . $fileSize);
        header('Content-Length: ' . ($fileSize - $resumePosition));
        header('HTTP/1.1 206 Partial Content');
        
        // 从断点位置开始输出
        while (($chunk = fread($handle, 8192)) !== false) {
            if ($chunk === '') {
                break;
            }
            echo $chunk;
        }
    } finally {
        fclose($handle);
    }
}

// 使用示例（假设从第 1000 字节开始续传）
// resumeDownload(__DIR__ . '/large-file.zip', 1000);
```

**说明**：
- 使用 `fseek()` 移动到断点位置
- 设置 HTTP 206 响应头（部分内容）
- 从断点位置开始输出文件内容

## 使用场景

### 场景 1：随机访问文件内容

读取文件的特定部分，而不是从开头顺序读取。

**示例**：见"示例 7：随机访问文件内容"

### 场景 2：实现断点续传

支持文件下载的断点续传功能。

**示例**：见"示例 8：实现断点续传"

### 场景 3：读取文件的最后 N 行

从文件末尾向前读取，获取文件的最后几行。

**示例**：

```php
<?php
declare(strict_types=1);

function readLastLines(string $file, int $lineCount): array
{
    $handle = fopen($file, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$file}");
    }
    
    try {
        $fileSize = filesize($file);
        if ($fileSize === false || $fileSize === 0) {
            return [];
        }
        
        // 从文件末尾开始读取
        $chunkSize = min(8192, $fileSize);
        $position = max(0, $fileSize - $chunkSize);
        fseek($handle, $position, SEEK_SET);
        
        $lines = [];
        $buffer = '';
        
        while (($chunk = fread($handle, $chunkSize)) !== false && $chunk !== '') {
            $buffer = $chunk . $buffer;
            $parts = explode("\n", $buffer);
            $buffer = array_shift($parts) ?? '';
            
            // 如果还有更多内容需要读取
            if ($position > 0 && count($parts) < $lineCount) {
                $position = max(0, $position - $chunkSize);
                fseek($handle, $position, SEEK_SET);
                continue;
            }
            
            $lines = array_merge($parts, $lines);
            if (count($lines) >= $lineCount) {
                break;
            }
        }
        
        if ($buffer !== '') {
            array_unshift($lines, $buffer);
        }
        
        return array_slice($lines, -$lineCount);
    } finally {
        fclose($handle);
    }
}

$lastLines = readLastLines(__DIR__ . '/log.txt', 10);
print_r($lastLines);
```

## 注意事项

### 文件指针位置影响读写操作

文件指针的位置决定了下一次读写操作将从哪里开始。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle !== false) {
    // 读取前 10 字节
    $chunk1 = fread($handle, 10);
    
    // 移动到位置 50
    fseek($handle, 50, SEEK_SET);
    
    // 从位置 50 开始读取
    $chunk2 = fread($handle, 10);
    
    // 现在文件指针在位置 60
    echo "Current position: " . ftell($handle) . "\n";  // 60
    
    fclose($handle);
}
```

### 某些流不支持定位

某些流（如网络流、`php://input`）不支持定位，`fseek()` 会失败。

**示例**：

```php
<?php
declare(strict_types=1);

// 标准输入不支持定位
$handle = fopen('php://stdin', 'r');
if ($handle !== false) {
    $result = fseek($handle, 0, SEEK_SET);
    if ($result === -1) {
        echo "This stream does not support seeking\n";
    }
    fclose($handle);
}
```

### 写入模式下移动指针

在写入模式下，移动到文件末尾之后的位置会扩展文件（用空字节填充）。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/output.txt', 'wb');
if ($handle !== false) {
    // 写入一些数据
    fwrite($handle, 'Hello');
    
    // 移动到位置 100（文件只有 5 字节）
    fseek($handle, 100, SEEK_SET);
    
    // 写入数据
    fwrite($handle, 'World');
    
    // 文件现在有 105 字节（前 5 字节是 "Hello"，中间是空字节，最后是 "World"）
    fclose($handle);
}
```

## 常见问题

### 问题 1：文件指针的位置如何计算？

**回答**：文件指针位置以字节为单位，从 0 开始（文件开头）。可以使用 `ftell()` 获取当前位置。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle !== false) {
    echo "Initial position: " . ftell($handle) . "\n";  // 0
    
    fread($handle, 50);
    echo "After reading 50 bytes: " . ftell($handle) . "\n";  // 50
    
    fseek($handle, 100, SEEK_SET);
    echo "After seeking to 100: " . ftell($handle) . "\n";  // 100
    
    fclose($handle);
}
```

### 问题 2：fseek() 的三种模式有什么区别？

**回答**：
- `SEEK_SET`：从文件开头计算偏移量
- `SEEK_CUR`：从当前位置计算偏移量（可以向前或向后）
- `SEEK_END`：从文件末尾计算偏移量（通常使用负数）

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle !== false) {
    // SEEK_SET：移动到文件开头后 100 字节
    fseek($handle, 100, SEEK_SET);
    echo "Position: " . ftell($handle) . "\n";  // 100
    
    // SEEK_CUR：从当前位置向前移动 50 字节
    fseek($handle, 50, SEEK_CUR);
    echo "Position: " . ftell($handle) . "\n";  // 150
    
    // SEEK_END：移动到文件末尾前 10 字节
    fseek($handle, -10, SEEK_END);
    echo "Position: " . ftell($handle) . "\n";  // 文件大小 - 10
    
    fclose($handle);
}
```

### 问题 3：如何跳转到文件的特定位置？

**回答**：使用 `fseek()` 函数，指定目标位置和起始位置。

**示例**：

```php
<?php
declare(strict_types=1);

function jumpToPosition($handle, int $position): bool
{
    return fseek($handle, $position, SEEK_SET) === 0;
}

$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle !== false) {
    if (jumpToPosition($handle, 100)) {
        echo "Jumped to position 100\n";
        $data = fread($handle, 50);
        echo "Read: {$data}\n";
    }
    fclose($handle);
}
```

### 问题 4：文件指针会影响哪些操作？

**回答**：文件指针会影响所有读写操作，包括 `fread()`、`fwrite()`、`fgets()`、`fgetc()` 等。

**示例**：见"示例 2：读取操作对文件指针的影响"

## 最佳实践

### 1. 明确文件指针的位置

在进行读写操作前，明确文件指针的位置，避免意外覆盖数据。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle !== false) {
    // 记录当前位置
    $currentPos = ftell($handle);
    echo "Current position: {$currentPos}\n";
    
    // 执行操作后再次检查
    $data = fread($handle, 100);
    $newPos = ftell($handle);
    echo "New position: {$newPos}\n";
    
    fclose($handle);
}
```

### 2. 使用 ftell() 调试指针位置

在调试文件操作时，使用 `ftell()` 检查文件指针位置。

**示例**：

```php
<?php
declare(strict_types=1);

function debugFilePosition($handle, string $label): void
{
    $pos = ftell($handle);
    echo "[{$label}] File pointer position: {$pos}\n";
}

$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle !== false) {
    debugFilePosition($handle, 'Initial');
    
    fread($handle, 50);
    debugFilePosition($handle, 'After read');
    
    fseek($handle, 100, SEEK_SET);
    debugFilePosition($handle, 'After seek');
    
    fclose($handle);
}
```

### 3. 注意不同模式下的指针行为

不同打开模式下的文件指针初始位置不同，需要特别注意。

**示例**：

```php
<?php
declare(strict_types=1);

// 只读模式：指针在开头
$handle1 = fopen(__DIR__ . '/data.txt', 'rb');
echo "Read mode position: " . ftell($handle1) . "\n";  // 0
fclose($handle1);

// 追加模式：指针在末尾
$handle2 = fopen(__DIR__ . '/data.txt', 'ab');
$size = filesize(__DIR__ . '/data.txt');
echo "Append mode position: " . ftell($handle2) . "\n";  // 等于文件大小
fclose($handle2);
```

### 4. 操作后检查指针位置

在执行文件操作后，检查文件指针位置，确保操作符合预期。

**示例**：

```php
<?php
declare(strict_types=1);

function safeRead($handle, int $length): string
{
    $beforePos = ftell($handle);
    $data = fread($handle, $length);
    $afterPos = ftell($handle);
    
    $bytesRead = $afterPos - $beforePos;
    if ($bytesRead !== $length && $data !== '') {
        // 读取的字节数不符合预期，可能是到达文件末尾
        echo "Warning: Expected {$length} bytes, read {$bytesRead} bytes\n";
    }
    
    return $data !== false ? $data : '';
}
```

### 5. 不要用 feof() 作为循环条件

应该检查 `fread()` 的返回值，而不是用 `feof()` 作为循环条件。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'rb');
if ($handle !== false) {
    // ✅ 正确
    while (($chunk = fread($handle, 8192)) !== false) {
        if ($chunk === '') {
            break;
        }
        echo $chunk;
    }
    
    // ❌ 错误
    // while (!feof($handle)) {
    //     $chunk = fread($handle, 8192);
    //     // ...
    // }
    
    fclose($handle);
}
```

## 对比分析

### fseek() 的三种模式

| 模式       | 常量值 | 说明                        | 偏移量示例          |
|:-----------|:-------|:---------------------------|:-------------------|
| **SEEK_SET** | 0      | 从文件开头计算偏移量        | 正数（0, 100, 500）|
| **SEEK_CUR** | 1      | 从当前位置计算偏移量        | 正数或负数（+50, -20）|
| **SEEK_END** | 2      | 从文件末尾计算偏移量        | 负数（-10, -100）  |

### rewind() vs fseek($handle, 0, SEEK_SET)

| 特性         | rewind()                      | fseek($handle, 0, SEEK_SET)  |
|:-------------|:------------------------------|:-----------------------------|
| **功能**      | 重置到文件开头                | 移动到位置 0                  |
| **返回值**    | bool                          | int（0 表示成功）             |
| **使用场景**  | 简单重置                      | 需要检查返回值                |
| **等价性**    | 等价                          | 等价                          |

## 练习任务

1. **随机访问文件函数**：编写一个函数，读取文件的指定位置和长度的内容。

2. **文件指针跟踪工具**：创建一个工具类，跟踪文件指针的位置变化，用于调试。

3. **断点续传功能**：实现一个文件下载的断点续传功能，支持 HTTP 206 响应。

4. **读取最后 N 行**：编写一个函数，从文件末尾向前读取，获取文件的最后 N 行。

5. **文件查找功能**：实现一个函数，在文件中查找特定字符串，返回其位置。

## 相关章节

- **[4.2.1 文件读取](section-01-file-reading.md)**：了解文件读取的基础知识
- **[4.2.2 文件写入](section-02-file-writing.md)**：了解文件写入的基础知识
- **[4.2.10 大文件处理](section-10-large-files.md)**：学习如何处理大文件
