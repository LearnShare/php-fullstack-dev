# 5.8.5 二进制文件处理

## 概述

二进制文件是指包含非文本数据的文件，数据以字节形式存储，不能直接作为文本读取。处理二进制文件时，必须使用二进制模式（`b`），避免数据损坏。理解二进制文件的读写方法、文本模式与二进制模式的区别，以及如何安全高效地处理二进制文件，对于构建健壮的 PHP 应用至关重要。

## 什么是二进制文件

**二进制文件**是指包含非文本数据的文件，数据以字节形式存储，不能直接作为文本读取。常见的二进制文件包括：

- **图片文件**：`.jpg`、`.png`、`.gif`、`.bmp`、`.webp`、`.svg`
- **视频文件**：`.mp4`、`.avi`、`.mov`、`.mkv`、`.webm`
- **音频文件**：`.mp3`、`.wav`、`.flac`、`.ogg`、`.aac`
- **压缩文件**：`.zip`、`.rar`、`.tar.gz`、`.7z`
- **可执行文件**：`.exe`、`.dll`、`.so`、`.dylib`
- **数据库文件**：`.db`、`.sqlite`、`.mdb`
- **Office 文档**：`.docx`、`.xlsx`、`.pptx`（实际上是 ZIP 格式）
- **PDF 文件**：`.pdf`
- **字体文件**：`.ttf`、`.otf`、`.woff`

## 文本模式 vs 二进制模式

### 文本模式（默认，或显式使用 `t`）

**文本模式**适用于纯文本文件：

- **在 Windows 上**：会自动转换换行符
  - 读取时：`\r\n`（Windows）→ `\n`（PHP 内部）
  - 写入时：`\n`（PHP 内部）→ `\r\n`（Windows）
- **在类 Unix 系统上**：无影响（因为换行符是 `\n`）
- **适用于**：`.txt`、`.php`、`.html`、`.json`、`.xml`、`.css`、`.js` 等文本文件

### 二进制模式（使用 `b`）

**二进制模式**不进行任何转换，按字节原样读取和写入：

- 适用于所有文件类型，特别是二进制文件
- **在 Windows 上强烈推荐使用**，避免换行符转换问题
- **在类 Unix 系统上**：虽然行为与文本模式相同，但使用二进制模式可以确保跨平台兼容性
- **适用于**：图片、视频、音频、压缩文件、可执行文件等

### 为什么需要二进制模式

在 Windows 系统上，文本模式会自动转换换行符。对于二进制文件，这种转换会**破坏文件内容**：

**问题示例**：
- 图片文件中如果包含 `0x0A`（`\n`）或 `0x0D`（`\r`），会被错误处理
- 压缩文件、可执行文件等会被损坏
- 文件哈希值会改变
- 文件大小可能改变
- 文件无法正常使用

**最佳实践**：
- **始终使用二进制模式（`b`）**：即使是文本文件，使用 `b` 模式也能正常工作，且确保跨平台兼容性
- 在 Windows 上，二进制模式是必需的
- 在类 Unix 系统上，使用二进制模式可以确保代码在所有平台上行为一致

## 二进制文件读取

### 使用 file_get_contents() 读取二进制文件

`file_get_contents()` 默认以二进制模式读取，适合读取小到中等大小的二进制文件：

```php
<?php
declare(strict_types=1);

// 读取图片文件
$imageData = file_get_contents(__DIR__ . '/image.jpg');
if ($imageData === false) {
    throw new RuntimeException('Cannot read image file');
}

// 输出图片（Web 场景）
header('Content-Type: image/jpeg');
echo $imageData;

// 读取 ZIP 文件
$zipData = file_get_contents(__DIR__ . '/archive.zip');
if ($zipData === false) {
    throw new RuntimeException('Cannot read ZIP file');
}

// 保存到新位置
file_put_contents(__DIR__ . '/backup.zip', $zipData);
```

### 使用 fopen() + fread() 读取二进制文件

对于大文件，使用流式读取，**必须使用 `rb` 模式**：

```php
<?php
declare(strict_types=1);

// 使用二进制模式打开文件（rb = read binary）
$handle = fopen(__DIR__ . '/large-video.mp4', 'rb');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// 流式读取二进制数据（每次 8KB）
$output = fopen('php://output', 'wb');  // 输出流也使用二进制模式
while (($chunk = fread($handle, 8192)) !== false) {
    if ($chunk === '') {
        break;  // 到达文件末尾
    }
    fwrite($output, $chunk);
}

fclose($handle);
fclose($output);
```

### 二进制文件读取示例

**示例 1：读取图片并获取信息**

```php
<?php
declare(strict_types=1);

function readImageFile(string $path): array
{
    // file_get_contents() 默认以二进制模式读取
    $data = file_get_contents($path);
    if ($data === false) {
        throw new RuntimeException("Cannot read image: {$path}");
    }
    
    // 获取图片信息
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

// 使用
$image = readImageFile(__DIR__ . '/photo.jpg');
echo "Image size: {$image['width']}x{$image['height']}\n";
echo "MIME type: {$image['mime']}\n";
echo "File size: {$image['size']} bytes\n";
```

**示例 2：流式读取大文件并计算哈希**

```php
<?php
declare(strict_types=1);

function calculateFileHash(string $path, string $algorithm = 'sha256'): string
{
    // 使用二进制模式打开
    $handle = fopen($path, 'rb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$path}");
    }
    
    $hash = hash_init($algorithm);
    
    try {
        // 流式读取，避免内存溢出
        while (($chunk = fread($handle, 8192)) !== false) {
            if ($chunk === '') {
                break;  // 到达文件末尾
            }
            hash_update($hash, $chunk);
        }
        
        return hash_final($hash);
    } finally {
        fclose($handle);
    }
}

// 使用
$hash = calculateFileHash(__DIR__ . '/large-file.zip');
echo "SHA256: {$hash}\n";
```

**示例 3：比较文本模式和二进制模式的区别（Windows）**

```php
<?php
declare(strict_types=1);

// 创建一个包含特殊字节的文件
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

## 二进制文件写入

### 使用 file_put_contents() 写入二进制文件

`file_put_contents()` 默认以二进制模式写入，适合写入小到中等大小的二进制文件：

```php
<?php
declare(strict_types=1);

// 读取二进制数据
$imageData = file_get_contents(__DIR__ . '/source.jpg');
if ($imageData === false) {
    throw new RuntimeException('Cannot read source image');
}

// 写入二进制文件
$bytes = file_put_contents(__DIR__ . '/copy.jpg', $imageData);
if ($bytes === false) {
    throw new RuntimeException('Cannot write image file');
}

echo "Written {$bytes} bytes\n";
```

### 使用 fopen() + fwrite() 写入二进制文件

对于大文件，使用流式写入，**必须使用 `wb` 模式**：

```php
<?php
declare(strict_types=1);

// 使用二进制模式打开文件（wb = write binary）
$output = fopen(__DIR__ . '/output.bin', 'wb');
if ($output === false) {
    throw new RuntimeException('Cannot open file for writing');
}

// 写入二进制数据
$data = "\x00\x01\x02\x03\xFF";
$bytes = fwrite($output, $data);
if ($bytes === false) {
    fclose($output);
    throw new RuntimeException('Cannot write data');
}

fclose($output);
echo "Written {$bytes} bytes\n";
```

### 二进制文件写入示例

**示例 1：复制二进制文件**

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
                break;  // 到达文件末尾
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

// 使用
copyBinaryFile(__DIR__ . '/large-video.mp4', __DIR__ . '/copy.mp4');
```

**示例 2：生成二进制文件**

```php
<?php
declare(strict_types=1);

function generateBinaryFile(string $path, int $size): void
{
    $handle = fopen($path, 'wb');
    if ($handle === false) {
        throw new RuntimeException("Cannot create file: {$path}");
    }
    
    // 生成随机二进制数据
    $chunkSize = 8192;
    $written = 0;
    
    while ($written < $size) {
        $remaining = $size - $written;
        $toWrite = min($chunkSize, $remaining);
        
        // 生成随机字节
        $data = random_bytes($toWrite);
        
        if (fwrite($handle, $data) === false) {
            fclose($handle);
            throw new RuntimeException("Cannot write data");
        }
        
        $written += $toWrite;
    }
    
    fclose($handle);
}

// 使用：生成 1MB 的随机二进制文件
generateBinaryFile(__DIR__ . '/random.bin', 1024 * 1024);
```

**示例 3：追加二进制数据**

```php
<?php
declare(strict_types=1);

// 追加二进制数据到文件
$handle = fopen(__DIR__ . '/data.bin', 'ab');  // 'a' = append, 'b' = binary
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

$newData = "\xAA\xBB\xCC\xDD";
if (fwrite($handle, $newData) === false) {
    fclose($handle);
    throw new RuntimeException('Cannot write data');
}

fclose($handle);
```

## 二进制文件处理的最佳实践

1. **始终使用 `b` 模式**：即使是文本文件，使用 `b` 模式也能正常工作，且确保跨平台兼容性
2. **大文件使用流式操作**：避免一次性加载到内存，使用 `fopen()` + `fread()`/`fwrite()` 循环处理
3. **检查返回值**：`fread()` 返回 `false` 表示错误，返回空字符串表示到达文件末尾
4. **使用 `file_get_contents()`/`file_put_contents()` 处理小文件**：对于小于 10MB 的二进制文件，这些函数更方便
5. **注意内存限制**：处理大文件时注意 PHP 的 `memory_limit` 设置
6. **输出二进制数据时也使用二进制模式**：使用 `fopen('php://output', 'wb')` 而不是 `'w'`
7. **验证文件类型**：在处理二进制文件前，验证文件类型和大小
8. **错误处理**：始终检查文件操作的返回值

## 注意事项

1. **错误处理**：始终检查文件操作的返回值
2. **大文件处理**：对于大文件（> 10MB），使用流式操作而不是一次性读取/写入
3. **资源释放**：使用 `fopen()` 后必须使用 `fclose()` 释放资源
4. **二进制文件必须使用 `b` 模式**：处理二进制文件时，**必须**使用 `b` 模式（如 `rb`、`wb`），避免数据损坏
5. **跨平台兼容性**：即使是文本文件，也推荐使用 `b` 模式以确保跨平台兼容性
6. **内存管理**：处理大文件时注意内存使用，使用流式操作避免内存溢出

## 练习

1. 编写一个函数，安全地读取二进制文件（使用流式读取）
2. 实现一个二进制文件复制功能，支持大文件
3. 创建一个函数，计算二进制文件的哈希值（支持大文件）
4. 编写一个函数，比较两个二进制文件是否相同
5. 实现一个二进制文件分割功能，将大文件分割成多个小文件
