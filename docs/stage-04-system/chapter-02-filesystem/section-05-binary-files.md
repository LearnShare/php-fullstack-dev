# 4.2.5 二进制文件处理

## 概述

二进制文件是指包含非文本数据的文件，数据以字节形式存储，不能直接作为文本读取。常见的二进制文件包括图片、视频、音频、压缩文件、可执行文件等。处理二进制文件时，必须使用二进制模式（`b`），避免数据损坏。理解二进制文件的读写方法、文本模式与二进制模式的区别，以及如何安全高效地处理二进制文件，对于构建健壮的 PHP 应用至关重要。

在 Windows 系统上，文本模式会自动转换换行符，这对于二进制文件会破坏文件内容。因此，处理二进制文件时必须使用二进制模式。即使在类 Unix 系统上，使用二进制模式也可以确保代码在所有平台上行为一致。

掌握二进制文件处理的基础知识后，可以进一步学习文件格式解析、数据序列化、图片处理等高级主题。

**主要内容**：
- 二进制文件的概念和常见类型
- 文本模式 vs 二进制模式的区别
- 二进制文件的读取方法
- 二进制文件的写入方法
- `pack()` 和 `unpack()` 函数
- 文件格式识别
- 实际应用场景和最佳实践

## 特性

- **数据完整性**：使用二进制模式确保数据不被转换或损坏
- **跨平台兼容**：二进制模式在所有平台上行为一致
- **灵活的数据处理**：支持字节级的数据操作
- **格式支持**：支持处理各种二进制文件格式
- **内存高效**：支持流式处理大文件

## 语法/定义

### pack() 函数

**语法**：`pack(string $format, mixed ...$values): string`

**参数**：
- `$format`：格式字符串，指定如何打包数据
- `...$values`：要打包的值

**返回值**：返回打包后的二进制字符串。

### unpack() 函数

**语法**：`unpack(string $format, string $data, int $offset = 0): array|false`

**参数**：
- `$format`：格式字符串，指定如何解包数据
- `$data`：要解包的二进制字符串
- `$offset`：可选，开始解包的偏移量，默认为 0

**返回值**：成功返回包含解包数据的数组，失败返回 `false`。

## 基本用法

### 示例 1：二进制文件读取

```php
<?php
declare(strict_types=1);

// 使用 file_get_contents() 读取二进制文件（默认二进制模式）
$imageData = file_get_contents(__DIR__ . '/image.jpg');
if ($imageData === false) {
    throw new RuntimeException('Cannot read image file');
}

// 输出图片（Web 场景）
header('Content-Type: image/jpeg');
echo $imageData;

// 使用 fopen() + fread() 流式读取大文件
$handle = fopen(__DIR__ . '/large-video.mp4', 'rb');  // 必须使用 'rb' 模式
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    while (($chunk = fread($handle, 8192)) !== false) {
        if ($chunk === '') {
            break;
        }
        // 处理每一块二进制数据
        echo $chunk;
    }
} finally {
    fclose($handle);
}
```

**说明**：
- `file_get_contents()` 默认以二进制模式读取
- 对于大文件，使用 `fopen()` + `fread()` 流式读取
- **必须使用 `rb` 模式**打开二进制文件

### 示例 2：二进制文件写入

```php
<?php
declare(strict_types=1);

// 使用 file_put_contents() 写入二进制文件（默认二进制模式）
$imageData = file_get_contents(__DIR__ . '/source.jpg');
if ($imageData !== false) {
    $bytes = file_put_contents(__DIR__ . '/copy.jpg', $imageData);
    if ($bytes !== false) {
        echo "Written {$bytes} bytes\n";
    }
}

// 使用 fopen() + fwrite() 流式写入
$output = fopen(__DIR__ . '/output.bin', 'wb');  // 必须使用 'wb' 模式
if ($output !== false) {
    try {
        // 写入二进制数据
        $data = "\x00\x01\x02\x03\xFF";
        $bytes = fwrite($output, $data);
        if ($bytes !== false) {
            echo "Written {$bytes} bytes\n";
        }
    } finally {
        fclose($output);
    }
}
```

**说明**：
- `file_put_contents()` 默认以二进制模式写入
- 对于大文件，使用 `fopen()` + `fwrite()` 流式写入
- **必须使用 `wb` 模式**打开二进制文件进行写入

### 示例 3：文本模式 vs 二进制模式的区别

```php
<?php
declare(strict_types=1);

// 创建一个包含特殊字节的测试文件
$testData = "\x00\x01\x02\x0A\x0D\xFF";  // 包含 \n (0x0A) 和 \r (0x0D)
file_put_contents(__DIR__ . '/test.bin', $testData);

// 文本模式读取（Windows 上会转换换行符）
$handle = fopen(__DIR__ . '/test.bin', 'r');  // 文本模式
$textMode = fread($handle, 1024);
fclose($handle);

// 二进制模式读取（原样读取）
$handle = fopen(__DIR__ . '/test.bin', 'rb');  // 二进制模式
$binaryMode = fread($handle, 1024);
fclose($handle);

// 比较（在 Windows 上会不同）
echo "Original: " . bin2hex($testData) . "\n";
echo "Text mode: " . bin2hex($textMode) . "\n";    // Windows 上可能不同
echo "Binary mode: " . bin2hex($binaryMode) . "\n"; // 与原始数据相同

// 清理
unlink(__DIR__ . '/test.bin');
```

**输出**（在 Windows 上）：

```
Original: 0001020a0dff
Text mode: 0001020aff      （0x0D 被移除）
Binary mode: 0001020a0dff   （与原始数据相同）
```

**说明**：
- 文本模式在 Windows 上会转换换行符，破坏二进制数据
- 二进制模式原样读取，确保数据完整性
- **最佳实践**：始终使用二进制模式处理二进制文件

### 示例 4：使用 pack() 打包数据

```php
<?php
declare(strict_types=1);

// 打包一个 32 位整数（大端序）
$number = 12345;
$binary = pack('N', $number);  // N = 32 位无符号整数（大端序）
echo "Number: {$number}\n";
echo "Binary: " . bin2hex($binary) . "\n";  // 00003039

// 打包多个值
$data = pack('NnC', 12345, 6789, 255);
// N = 32 位无符号整数（大端序）
// n = 16 位无符号整数（大端序）
// C = 8 位无符号整数
echo "Packed: " . bin2hex($data) . "\n";

// 打包字符串
$text = pack('A10', 'Hello');  // A = 空格填充的字符串
echo "Text: " . bin2hex($text) . "\n";
```

**说明**：
- `pack()` 将数据打包为二进制字符串
- 格式字符串指定数据类型和字节序
- 可以打包多个值到一个二进制字符串

### 示例 5：使用 unpack() 解包数据

```php
<?php
declare(strict_types=1);

// 打包数据
$binary = pack('N', 12345);

// 解包数据
$result = unpack('N', $binary);
if ($result !== false) {
    echo "Unpacked: " . $result[1] . "\n";  // 12345
}

// 解包多个值
$data = pack('NnC', 12345, 6789, 255);
$result = unpack('Nnum1/nnum2/Cbyte', $data);
if ($result !== false) {
    echo "num1: {$result['num1']}\n";  // 12345
    echo "num2: {$result['num2']}\n";  // 6789
    echo "byte: {$result['byte']}\n";  // 255
}
```

**说明**：
- `unpack()` 从二进制字符串解包数据
- 可以使用命名键或数字索引访问解包的数据
- 格式字符串必须与打包时一致

### 示例 6：流式复制二进制文件

```php
<?php
declare(strict_types=1);

function copyBinaryFile(string $source, string $destination): void
{
    $sourceHandle = fopen($source, 'rb');
    if ($sourceHandle === false) {
        throw new RuntimeException("Cannot open source file: {$source}");
    }
    
    $destHandle = fopen($destination, 'wb');
    if ($destHandle === false) {
        fclose($sourceHandle);
        throw new RuntimeException("Cannot open destination file: {$destination}");
    }
    
    try {
        // 流式复制（每次 8KB）
        while (($chunk = fread($sourceHandle, 8192)) !== false) {
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

copyBinaryFile(__DIR__ . '/source.jpg', __DIR__ . '/copy.jpg');
echo "File copied successfully\n";
```

**说明**：
- 使用二进制模式打开源文件和目标文件
- 流式复制，避免一次性加载到内存
- 适合复制大文件

### 示例 7：计算文件哈希值

```php
<?php
declare(strict_types=1);

function calculateFileHash(string $path, string $algorithm = 'sha256'): string
{
    $handle = fopen($path, 'rb');  // 必须使用二进制模式
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$path}");
    }
    
    $hash = hash_init($algorithm);
    
    try {
        // 流式读取，避免内存溢出
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

$hash = calculateFileHash(__DIR__ . '/large-file.zip');
echo "SHA256: {$hash}\n";
```

**说明**：
- 使用二进制模式打开文件
- 流式读取并更新哈希
- 适合计算大文件的哈希值

### 示例 8：识别文件格式

```php
<?php
declare(strict_types=1);

function detectFileFormat(string $filePath): ?string
{
    $handle = fopen($filePath, 'rb');
    if ($handle === false) {
        return null;
    }
    
    try {
        // 读取文件头（前几个字节）
        $header = fread($handle, 16);
        if ($header === false || strlen($header) < 4) {
            return null;
        }
        
        // 根据文件头识别格式
        $hex = bin2hex(substr($header, 0, 4));
        
        // JPEG: FF D8 FF
        if (str_starts_with($hex, 'ffd8ff')) {
            return 'image/jpeg';
        }
        
        // PNG: 89 50 4E 47
        if ($hex === '89504e47') {
            return 'image/png';
        }
        
        // GIF: 47 49 46 38
        if (str_starts_with($hex, '47494638')) {
            return 'image/gif';
        }
        
        // ZIP: 50 4B 03 04
        if ($hex === '504b0304') {
            return 'application/zip';
        }
        
        return null;
    } finally {
        fclose($handle);
    }
}

$format = detectFileFormat(__DIR__ . '/file.bin');
if ($format !== null) {
    echo "File format: {$format}\n";
}
```

**说明**：
- 通过读取文件头（魔数）识别文件格式
- 不同文件格式有不同的文件头标识
- 使用二进制模式读取，确保准确性

## 使用场景

### 场景 1：图片文件处理

读取和复制图片文件。

**示例**：

```php
<?php
declare(strict_types=1);

function readImageFile(string $path): array
{
    $data = file_get_contents($path);
    if ($data === false) {
        throw new RuntimeException("Cannot read image: {$path}");
    }
    
    $info = getimagesizefromstring($data);
    if ($info === false) {
        throw new RuntimeException("Invalid image format: {$path}");
    }
    
    return [
        'data' => $data,
        'width' => $info[0],
        'height' => $info[1],
        'mime' => $info['mime'],
        'size' => strlen($data),
    ];
}

$image = readImageFile(__DIR__ . '/photo.jpg');
echo "Image size: {$image['width']}x{$image['height']}\n";
echo "MIME type: {$image['mime']}\n";
```

### 场景 2：文件格式验证

验证上传的文件格式是否正确。

**示例**：见"示例 8：识别文件格式"

## 注意事项

### 必须使用二进制模式

处理二进制文件时，**必须使用二进制模式**（`rb`、`wb`、`ab`），否则数据会被损坏。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 正确：使用二进制模式
$handle = fopen(__DIR__ . '/image.jpg', 'rb');
$handle = fopen(__DIR__ . '/video.mp4', 'rb');
$handle = fopen(__DIR__ . '/archive.zip', 'rb');

// ❌ 错误：使用文本模式（会损坏二进制文件）
// $handle = fopen(__DIR__ . '/image.jpg', 'r');  // 不推荐
```

### 跨平台兼容性

即使在类 Unix 系统上，也推荐使用二进制模式，确保代码在所有平台上行为一致。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：始终使用二进制模式
$handle = fopen(__DIR__ . '/data.bin', 'rb');   // 跨平台兼容
$handle = fopen(__DIR__ . '/data.txt', 'rb');   // 即使是文本文件也推荐使用

// 不推荐：使用文本模式
// $handle = fopen(__DIR__ . '/data.bin', 'r');  // 可能在 Windows 上出问题
```

### 大文件处理

对于大文件，使用流式处理，避免一次性加载到内存。

**示例**：

```php
<?php
declare(strict_types=1);

function processLargeBinaryFile(string $filePath, callable $processor): void
{
    $handle = fopen($filePath, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$filePath}");
    }
    
    try {
        while (($chunk = fread($handle, 8192)) !== false) {
            if ($chunk === '') {
                break;
            }
            $processor($chunk);
        }
    } finally {
        fclose($handle);
    }
}
```

## 常见问题

### 问题 1：二进制模式和文本模式的区别是什么？

**回答**：
- **文本模式**：在 Windows 上会自动转换换行符（`\r\n` ↔ `\n`），可能破坏二进制数据
- **二进制模式**：按字节原样读取和写入，不会进行任何转换

**示例**：见"示例 3：文本模式 vs 二进制模式的区别"

### 问题 2：如何使用 pack()/unpack()？

**回答**：`pack()` 将数据打包为二进制字符串，`unpack()` 从二进制字符串解包数据。需要指定格式字符串。

**示例**：见"示例 4：使用 pack() 打包数据"和"示例 5：使用 unpack() 解包数据"

### 问题 3：如何处理不同字节序的数据？

**回答**：使用 `pack()` 和 `unpack()` 的格式字符指定字节序（大端序或小端序）。

**示例**：

```php
<?php
declare(strict_types=1);

$number = 12345;

// 大端序（网络字节序）
$bigEndian = pack('N', $number);  // N = 32 位无符号整数（大端序）

// 小端序（主机字节序）
$littleEndian = pack('V', $number);  // V = 32 位无符号整数（小端序）

echo "Big endian: " . bin2hex($bigEndian) . "\n";
echo "Little endian: " . bin2hex($littleEndian) . "\n";
```

### 问题 4：如何识别文件格式？

**回答**：通过读取文件头（魔数）识别文件格式。不同文件格式有不同的文件头标识。

**示例**：见"示例 8：识别文件格式"

## 最佳实践

### 1. 始终使用二进制模式

处理二进制文件时，始终使用二进制模式（`rb`、`wb`、`ab`）。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 正确
$handle = fopen(__DIR__ . '/image.jpg', 'rb');
$handle = fopen(__DIR__ . '/output.bin', 'wb');
```

### 2. 使用 pack()/unpack() 处理结构化数据

对于需要打包/解包的结构化数据，使用 `pack()` 和 `unpack()`。

**示例**：

```php
<?php
declare(strict_types=1);

// 打包结构化的二进制数据
$data = pack('NnC', 12345, 6789, 255);

// 解包数据
$result = unpack('Nnum1/nnum2/Cbyte', $data);
```

### 3. 注意字节序和平台差异

处理跨平台数据时，注意字节序差异，使用明确的格式字符。

**示例**：

```php
<?php
declare(strict_types=1);

// 使用大端序（网络字节序）确保跨平台一致性
$data = pack('N', $number);  // N = 大端序
```

### 4. 验证文件格式和完整性

在处理文件前，验证文件格式和完整性。

**示例**：

```php
<?php
declare(strict_types=1);

function validateImageFile(string $filePath): bool
{
    $data = file_get_contents($filePath);
    if ($data === false) {
        return false;
    }
    
    $info = getimagesizefromstring($data);
    return $info !== false;
}
```

### 5. 大文件使用流式处理

对于大文件，使用流式处理，避免内存溢出。

**示例**：见"示例 6：流式复制二进制文件"

## 对比分析

### 文本模式 vs 二进制模式

| 特性         | 文本模式（`r`、`w`、`a`）      | 二进制模式（`rb`、`wb`、`ab`） |
|:-------------|:-------------------------------|:-------------------------------|
| **数据转换** | Windows 上转换换行符           | 不进行任何转换                 |
| **适用文件** | 纯文本文件                     | 所有文件（特别是二进制文件）   |
| **跨平台**   | Windows 上可能有问题           | 所有平台行为一致               |
| **推荐使用** | ❌ 不推荐（即使是文本文件）    | ✅ 推荐（始终使用）            |

### file_get_contents() vs fopen() + fread()

| 特性         | file_get_contents()            | fopen() + fread()              |
|:-------------|:-------------------------------|:-------------------------------|
| **适用大小** | 小到中等文件（< 10MB）         | 大文件（> 10MB）               |
| **内存使用** | 一次性加载到内存               | 流式处理，内存友好             |
| **代码复杂度** | ✅ 简单                       | ⚠️ 较复杂                      |

## 练习任务

1. **二进制文件复制工具**：创建一个函数，安全地复制二进制文件，包含完整的错误处理。

2. **文件格式识别工具**：实现一个函数，通过文件头识别常见的文件格式（JPEG、PNG、GIF、ZIP 等）。

3. **数据打包/解包工具**：创建一个工具类，使用 `pack()` 和 `unpack()` 处理结构化的二进制数据。

4. **文件哈希计算工具**：编写一个函数，计算大文件的哈希值（支持 MD5、SHA1、SHA256 等算法）。

5. **二进制文件验证工具**：实现一个函数，验证二进制文件的完整性和格式正确性。

## 相关章节

- **[4.2.1 文件读取](section-01-file-reading.md)**：了解文件读取的基础知识
- **[4.2.2 文件写入](section-02-file-writing.md)**：了解文件写入的基础知识
- **[4.2.4 文件指针](section-04-file-pointer.md)**：了解文件指针的使用
- **[4.2.13 图像处理](section-13-image-processing.md)**：学习图像处理的相关内容
