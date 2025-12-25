# 5.8.4 文件指针操作

## 概述

文件指针是文件操作中的核心概念，它指示当前在文件中的读取或写入位置。理解文件指针的工作原理和操作方法，对于精确控制文件读写位置、实现随机访问、断点续传等高级功能至关重要。

## 什么是文件指针

**文件指针**（File Pointer）是一个内部标记，指示当前在文件中的读取或写入位置。可以把它想象成文件中的一个"光标"，标记着下一次读写操作将从哪里开始。

**文件指针的特点**：
- 每个打开的文件句柄都有自己独立的文件指针
- 文件指针位置以字节为单位，从 0 开始（文件开头）
- 读取或写入操作会移动文件指针
- 文件指针可以移动到文件的任意位置（如果文件支持随机访问）

## 文件指针的工作原理

### 初始位置

文件指针的初始位置取决于文件的打开模式：

- `r`、`r+`、`w`、`w+`、`x`、`x+`、`c`、`c+` 模式：文件指针在文件开头（位置 0）
- `a`、`a+` 模式：文件指针在文件末尾

### 操作对文件指针的影响

不同的文件操作会对文件指针产生不同的影响：

| 操作 | 文件指针变化 | 说明 |
| :--- | :--- | :--- |
| `fread($handle, $length)` | 向前移动 `$length` 字节 | 读取后指针移动到读取数据的末尾 |
| `fwrite($handle, $data)` | 向前移动写入的字节数 | 写入后指针移动到写入数据的末尾 |
| `fgets($handle)` | 移动到下一行开头 | 读取一行后指针移动到换行符之后 |
| `fgetc($handle)` | 向前移动 1 字节 | 读取一个字符后指针移动到下一个字符 |
| `fseek($handle, $offset)` | 移动到指定位置 | 手动设置文件指针位置 |
| `rewind($handle)` | 移动到文件开头（位置 0） | 重置文件指针 |
| `ftell($handle)` | 不移动 | 返回当前文件指针位置 |

### 文件指针移动示例

```php
<?php
declare(strict_types=1);

// 创建一个测试文件
file_put_contents(__DIR__ . '/test.txt', 'Hello World');

$handle = fopen(__DIR__ . '/test.txt', 'r');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// 初始位置：0（文件开头）
echo "Position: " . ftell($handle) . "\n";  // 输出: Position: 0

// 读取 5 字节："Hello"
$chunk = fread($handle, 5);
echo "Read: {$chunk}\n";  // 输出: Read: Hello
echo "Position: " . ftell($handle) . "\n";  // 输出: Position: 5

// 读取 1 字节：" "（空格）
$chunk = fread($handle, 1);
echo "Read: '{$chunk}'\n";  // 输出: Read: ' '
echo "Position: " . ftell($handle) . "\n";  // 输出: Position: 6

// 读取剩余内容："World"
$chunk = fread($handle, 1024);
echo "Read: {$chunk}\n";  // 输出: Read: World
echo "Position: " . ftell($handle) . "\n";  // 输出: Position: 11（文件末尾）

fclose($handle);
```

## ftell() - 获取当前文件指针位置

**语法**：`ftell(resource $stream): int|false`

**参数**：
- `$stream`：文件句柄

**返回值**：成功返回当前文件指针位置（字节数，从 0 开始），失败返回 `false`。

**用途**：
- 调试文件读取位置
- 记录读取进度
- 实现断点续传
- 验证文件指针位置

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'r');
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

## fseek() - 移动文件指针

**语法**：`fseek(resource $stream, int $offset, int $whence = SEEK_SET): int`

**参数**：
- `$stream`：文件句柄
- `$offset`：偏移量（字节数）
- `$whence`：起始位置（见下表）

**`$whence` 参数**：

| 常量 | 值 | 说明 |
| :--- | :--- | :--- |
| `SEEK_SET` | 0 | 从文件开头计算偏移量（默认） |
| `SEEK_CUR` | 1 | 从当前位置计算偏移量 |
| `SEEK_END` | 2 | 从文件末尾计算偏移量 |

**返回值**：
- 成功返回 `0`
- 失败返回 `-1`（某些流可能不支持定位）

**工作原理**：
- `fseek($handle, 100, SEEK_SET)`：移动到文件开头后 100 字节的位置
- `fseek($handle, -50, SEEK_CUR)`：从当前位置向前移动 50 字节
- `fseek($handle, -100, SEEK_END)`：移动到文件末尾前 100 字节的位置

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'r');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// 移动到文件开头后 100 字节的位置
fseek($handle, 100, SEEK_SET);
echo "Position: " . ftell($handle) . "\n";  // 100

// 从当前位置向前移动 50 字节
fseek($handle, 50, SEEK_CUR);
echo "Position: " . ftell($handle) . "\n";  // 150

// 移动到文件末尾前 10 字节的位置
fseek($handle, -10, SEEK_END);
echo "Position: " . ftell($handle) . "\n";  // 文件大小 - 10

// 移动到文件开头
fseek($handle, 0, SEEK_SET);
echo "Position: " . ftell($handle) . "\n";  // 0

fclose($handle);
```

**注意事项**：
- 某些流（如网络流、`php://input`）不支持定位，`fseek()` 会失败
- 在写入模式下，移动到文件末尾之后的位置会扩展文件（用空字节填充）
- 在只读模式下，不能移动到文件末尾之后

## rewind() - 重置文件指针到开头

**语法**：`rewind(resource $stream): bool`

**参数**：
- `$stream`：文件句柄

**返回值**：成功返回 `true`，失败返回 `false`。

**等价操作**：`rewind($handle)` 等价于 `fseek($handle, 0, SEEK_SET)`

**用途**：
- 重新读取文件
- 重置文件指针到开头
- 在读写模式下，从开头重新写入

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'r');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// 读取文件
$content1 = fread($handle, 1024);
echo "First read: " . strlen($content1) . " bytes\n";

// 重置文件指针
rewind($handle);
echo "Position after rewind: " . ftell($handle) . "\n";  // 0

// 再次读取文件
$content2 = fread($handle, 1024);
echo "Second read: " . strlen($content2) . " bytes\n";

fclose($handle);
```

## feof() - 检查文件指针是否到达文件末尾

**语法**：`feof(resource $stream): bool`

**返回值**：如果文件指针到达文件末尾返回 `true`，否则返回 `false`。

**工作原理**：
- `feof()` 检查文件指针是否已经到达或超过文件末尾
- **重要**：只有在尝试读取超出文件末尾的数据后，`feof()` 才会返回 `true`
- 如果文件指针正好在文件末尾（最后一个字节之后），`feof()` 可能返回 `false`，直到尝试读取

**注意事项**：
- 只有在尝试读取超出文件末尾的数据后，`feof()` 才会返回 `true`
- 不要用 `feof()` 作为循环条件，应该检查 `fread()` 的返回值

**正确用法**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'r');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// ✅ 正确：检查 fread() 的返回值
while (($chunk = fread($handle, 8192)) !== false) {
    if ($chunk === '') {
        break;  // 到达文件末尾
    }
    echo $chunk;
}

// ❌ 错误：不要用 feof() 作为循环条件
while (!feof($handle)) {
    $chunk = fread($handle, 8192);
    // 如果 fread() 返回 false，$chunk 可能是 false，但仍会继续循环
}

fclose($handle);
```

## 文件指针的实际应用

### 应用 1：随机访问文件

```php
<?php
declare(strict_types=1);

function readFileAtPosition(string $file, int $position, int $length): string
{
    $handle = fopen($file, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$file}");
    }
    
    // 移动到指定位置
    if (fseek($handle, $position, SEEK_SET) !== 0) {
        fclose($handle);
        throw new RuntimeException("Cannot seek to position: {$position}");
    }
    
    // 读取指定长度的数据
    $data = fread($handle, $length);
    fclose($handle);
    
    return $data !== false ? $data : '';
}

// 使用：读取文件的中间部分（从第 100 字节开始，读取 50 字节）
$chunk = readFileAtPosition(__DIR__ . '/data.txt', 100, 50);
```

### 应用 2：实现断点续传

```php
<?php
declare(strict_types=1);

function resumeDownload(string $file, int $resumePosition): void
{
    $handle = fopen($file, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$file}");
    }
    
    // 移动到断点位置
    fseek($handle, $resumePosition, SEEK_SET);
    
    // 设置 HTTP 头
    header('Content-Range: bytes ' . $resumePosition . '-' . (filesize($file) - 1) . '/' . filesize($file));
    header('Content-Length: ' . (filesize($file) - $resumePosition));
    header('HTTP/1.1 206 Partial Content');
    
    // 从断点位置开始输出
    while (($chunk = fread($handle, 8192)) !== false) {
        if ($chunk === '') {
            break;
        }
        echo $chunk;
    }
    
    fclose($handle);
}
```

### 应用 3：读取文件的最后 N 行

```php
<?php
declare(strict_types=1);

function readLastLines(string $file, int $lineCount): array
{
    $handle = fopen($file, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$file}");
    }
    
    // 移动到文件末尾
    fseek($handle, -1, SEEK_END);
    
    $lines = [];
    $currentLine = '';
    $bytesRead = 0;
    $maxBytes = filesize($file);
    
    // 从后往前读取
    while ($bytesRead < $maxBytes && count($lines) < $lineCount) {
        $char = fgetc($handle);
        if ($char === false) {
            break;
        }
        
        if ($char === "\n") {
            if ($currentLine !== '') {
                array_unshift($lines, $currentLine);
                $currentLine = '';
            }
        } else {
            $currentLine = $char . $currentLine;
        }
        
        // 向前移动
        if (ftell($handle) > 1) {
            fseek($handle, -2, SEEK_CUR);
        } else {
            break;
        }
        
        $bytesRead++;
    }
    
    if ($currentLine !== '') {
        array_unshift($lines, $currentLine);
    }
    
    fclose($handle);
    return $lines;
}
```

### 应用 4：在文件中查找特定内容

```php
<?php
declare(strict_types=1);

function findInFile(string $file, string $search): ?int
{
    $handle = fopen($file, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$file}");
    }
    
    $searchLen = strlen($search);
    $buffer = '';
    $position = 0;
    
    while (($chunk = fread($handle, 8192)) !== false) {
        if ($chunk === '') {
            break;
        }
        
        $buffer .= $chunk;
        
        // 在缓冲区中查找
        $pos = strpos($buffer, $search);
        if ($pos !== false) {
            fclose($handle);
            return $position + $pos;
        }
        
        // 保留最后 $searchLen - 1 字节（可能跨块）
        $position += strlen($chunk);
        $buffer = substr($buffer, -($searchLen - 1));
    }
    
    fclose($handle);
    return null;  // 未找到
}
```

## 文件指针的注意事项

1. **文件指针是独立的**：每个文件句柄都有自己的文件指针，互不影响
2. **写入操作会移动指针**：写入数据后，文件指针会移动到写入数据的末尾
3. **追加模式特殊行为**：在 `a` 或 `a+` 模式下，写入操作总是追加到文件末尾，无论文件指针位置
4. **某些流不支持定位**：网络流、`php://input` 等不支持 `fseek()` 和 `rewind()`
5. **二进制模式推荐**：使用二进制模式（`b`）可以确保文件指针行为一致

## 注意事项

1. **错误处理**：始终检查文件指针操作的返回值
2. **流支持**：某些流（如网络流）不支持文件指针定位
3. **跨平台兼容性**：使用二进制模式确保文件指针行为一致
4. **性能考虑**：频繁的文件指针移动可能影响性能

## 练习

1. 编写一个函数，实现文件的随机访问（读取指定位置的数据）
2. 实现一个断点续传功能，支持从指定位置继续下载
3. 编写一个函数，读取文件的最后 N 行
4. 实现一个文件搜索功能，在文件中查找特定内容并返回位置
