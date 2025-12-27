# 4.2.8 文件信息

## 概述

获取文件信息是文件系统操作中的常见需求。在实际应用中，我们经常需要获取文件的大小、修改时间、文件类型、权限等信息，用于文件管理、文件上传验证、缓存策略判断等场景。PHP 提供了丰富的文件信息获取函数，包括 `filesize()`、`filemtime()`、`filetype()`、`stat()` 等，以及面向对象的 `SplFileInfo` 类。

理解如何获取和使用文件信息对于构建健壮的文件管理应用至关重要。不同的函数适用于不同的场景，比如 `filesize()` 获取文件大小，`filemtime()` 获取修改时间，`stat()` 获取详细的文件统计信息等。

**主要内容**：
- 文件大小获取（`filesize()`）
- 文件时间信息（`filemtime()`、`filectime()`、`fileatime()`）
- 文件类型判断（`filetype()`、`mime_content_type()`、`finfo_file()`）
- 文件权限检查（`is_readable()`、`is_writable()`、`is_executable()`）
- 详细文件信息（`stat()`）
- `SplFileInfo` 类
- 实际应用场景和最佳实践

## 特性

- **丰富的信息**：支持获取文件大小、时间、类型、权限等多种信息
- **面向对象接口**：提供 `SplFileInfo` 类，更易于使用
- **详细统计**：`stat()` 函数提供详细的文件统计信息
- **跨平台兼容**：支持 Windows 和类 Unix 系统
- **性能优化**：部分函数支持缓存，提高性能

## 语法/定义

### filesize() 函数

**语法**：`filesize(string $filename): int|false`

**参数**：
- `$filename`：文件路径

**返回值**：成功返回文件大小（字节数），失败返回 `false`。

### filemtime() 函数

**语法**：`filemtime(string $filename): int|false`

**参数**：
- `$filename`：文件路径

**返回值**：成功返回最后修改时间的 Unix 时间戳，失败返回 `false`。

### filetype() 函数

**语法**：`filetype(string $filename): string|false`

**参数**：
- `$filename`：文件路径

**返回值**：成功返回文件类型（`file`、`dir`、`link` 等），失败返回 `false`。

### stat() 函数

**语法**：`stat(string $filename): array|false`

**参数**：
- `$filename`：文件路径

**返回值**：成功返回包含文件信息的数组，失败返回 `false`。

## 基本用法

### 示例 1：获取文件大小

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/data.txt';
$size = filesize($file);
if ($size !== false) {
    echo "File size: {$size} bytes\n";
    
    // 格式化文件大小
    function formatBytes(int $bytes): string
    {
        $units = ['B', 'KB', 'MB', 'GB', 'TB'];
        $i = 0;
        $size = $bytes;
        while ($size >= 1024 && $i < count($units) - 1) {
            $size /= 1024;
            $i++;
        }
        return round($size, 2) . ' ' . $units[$i];
    }
    
    echo "Formatted: " . formatBytes($size) . "\n";
}
```

**输出**：

```
File size: 1024 bytes
Formatted: 1 KB
```

**说明**：
- `filesize()` 返回文件大小（字节数）
- 对于大于 2GB 的文件，在 32 位系统上可能返回错误的值
- 可以格式化显示为更易读的单位（KB、MB、GB）

### 示例 2：获取文件时间信息

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/data.txt';

// 修改时间（文件内容被修改的时间）
$mtime = filemtime($file);
if ($mtime !== false) {
    echo "Last modified: " . date('Y-m-d H:i:s', $mtime) . "\n";
}

// 创建/变更时间
$ctime = filectime($file);
if ($ctime !== false) {
    echo "Created/Changed: " . date('Y-m-d H:i:s', $ctime) . "\n";
}

// 访问时间（文件被访问的时间）
$atime = fileatime($file);
if ($atime !== false) {
    echo "Last accessed: " . date('Y-m-d H:i:s', $atime) . "\n";
}
```

**说明**：
- `filemtime()`：文件内容被修改的时间
- `filectime()`：文件 inode 变更的时间（Linux）或创建时间（Windows）
- `fileatime()`：文件被访问的时间（某些文件系统可能禁用）

### 示例 3：获取文件类型

```php
<?php
declare(strict_types=1);

$path = __DIR__ . '/data.txt';

// 获取文件类型
$type = filetype($path);
if ($type !== false) {
    echo "File type: {$type}\n";  // file, dir, link, etc.
}

// 判断是否是文件
if (is_file($path)) {
    echo "It is a file\n";
}

// 判断是否是目录
if (is_dir($path)) {
    echo "It is a directory\n";
}

// 判断是否是链接
if (is_link($path)) {
    echo "It is a symbolic link\n";
}
```

**说明**：
- `filetype()` 返回文件类型字符串
- `is_file()`、`is_dir()`、`is_link()` 分别判断是否是文件、目录、链接

### 示例 4：检查文件权限

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/data.txt';

// 检查是否可读
if (is_readable($file)) {
    echo "File is readable\n";
}

// 检查是否可写
if (is_writable($file)) {
    echo "File is writable\n";
}

// 检查是否可执行
if (is_executable($file)) {
    echo "File is executable\n";
}

// 检查目录权限
$dir = __DIR__ . '/uploads';
if (is_dir($dir) && is_writable($dir)) {
    echo "Directory is writable\n";
}
```

**说明**：
- `is_readable()` 检查文件是否可读
- `is_writable()` 检查文件是否可写
- `is_executable()` 检查文件是否可执行

### 示例 5：使用 stat() 获取详细信息

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/data.txt';
$stat = stat($file);
if ($stat !== false) {
    echo "File size: {$stat['size']} bytes\n";
    echo "Last modified: " . date('Y-m-d H:i:s', $stat['mtime']) . "\n";
    echo "Last accessed: " . date('Y-m-d H:i:s', $stat['atime']) . "\n";
    echo "Last changed: " . date('Y-m-d H:i:s', $stat['ctime']) . "\n";
    echo "Permissions: " . substr(sprintf('%o', $stat['mode']), -4) . "\n";
    echo "Owner UID: {$stat['uid']}\n";
    echo "Group GID: {$stat['gid']}\n";
    echo "Device: {$stat['dev']}\n";
    echo "Inode: {$stat['ino']}\n";
}
```

**说明**：
- `stat()` 返回包含文件详细信息的数组
- 包含大小、时间、权限、所有者、设备号、inode 等信息
- 可以使用数字索引或字符串键访问

### 示例 6：使用 SplFileInfo 类

```php
<?php
declare(strict_types=1);

$fileInfo = new SplFileInfo(__DIR__ . '/data.txt');

// 文件基本信息
echo "Path: " . $fileInfo->getPathname() . "\n";
echo "Filename: " . $fileInfo->getFilename() . "\n";
echo "Basename: " . $fileInfo->getBasename() . "\n";
echo "Extension: " . $fileInfo->getExtension() . "\n";
echo "Path: " . $fileInfo->getPath() . "\n";

// 文件大小和时间
echo "Size: " . $fileInfo->getSize() . " bytes\n";
echo "Modified: " . date('Y-m-d H:i:s', $fileInfo->getMTime()) . "\n";
echo "Accessed: " . date('Y-m-d H:i:s', $fileInfo->getATime()) . "\n";
echo "Changed: " . date('Y-m-d H:i:s', $fileInfo->getCTime()) . "\n";

// 类型和权限
echo "Is file: " . ($fileInfo->isFile() ? 'yes' : 'no') . "\n";
echo "Is dir: " . ($fileInfo->isDir() ? 'yes' : 'no') . "\n";
echo "Is readable: " . ($fileInfo->isReadable() ? 'yes' : 'no') . "\n";
echo "Is writable: " . ($fileInfo->isWritable() ? 'yes' : 'no') . "\n";
echo "Is executable: " . ($fileInfo->isExecutable() ? 'yes' : 'no') . "\n";

// 文件句柄
$handle = $fileInfo->openFile('r');
$content = $handle->fread(1024);
$handle = null;
```

**说明**：
- `SplFileInfo` 提供面向对象的文件信息接口
- 提供丰富的方法获取文件的各种信息
- 可以打开文件句柄进行读写操作

### 示例 7：格式化文件大小

```php
<?php
declare(strict_types=1);

function formatBytes(int $bytes, int $precision = 2): string
{
    $units = ['B', 'KB', 'MB', 'GB', 'TB', 'PB'];
    
    $bytes = max($bytes, 0);
    $pow = floor(($bytes ? log($bytes) : 0) / log(1024));
    $pow = min($pow, count($units) - 1);
    
    $bytes /= (1 << (10 * $pow));
    
    return round($bytes, $precision) . ' ' . $units[$pow];
}

$sizes = [1024, 1048576, 1073741824, 1099511627776];
foreach ($sizes as $size) {
    echo formatBytes($size) . "\n";
}
```

**输出**：

```
1 KB
1 MB
1 GB
1 TB
```

**说明**：
- 格式化文件大小，使其更易读
- 自动选择合适的单位

### 示例 8：获取 MIME 类型

```php
<?php
declare(strict_types=1);

function getMimeType(string $file): ?string
{
    if (!file_exists($file)) {
        return null;
    }
    
    // 方法 1：使用 finfo_file()（推荐，需要 fileinfo 扩展）
    if (function_exists('finfo_file')) {
        $finfo = finfo_open(FILEINFO_MIME_TYPE);
        if ($finfo !== false) {
            $mime = finfo_file($finfo, $file);
            finfo_close($finfo);
            if ($mime !== false) {
                return $mime;
            }
        }
    }
    
    // 方法 2：使用 mime_content_type()（已废弃，但某些系统仍支持）
    if (function_exists('mime_content_type')) {
        $mime = mime_content_type($file);
        if ($mime !== false) {
            return $mime;
        }
    }
    
    // 方法 3：根据扩展名判断（备用方案）
    $extension = strtolower(pathinfo($file, PATHINFO_EXTENSION));
    $mimeTypes = [
        'jpg' => 'image/jpeg',
        'png' => 'image/png',
        'gif' => 'image/gif',
        'pdf' => 'application/pdf',
        'txt' => 'text/plain',
    ];
    
    return $mimeTypes[$extension] ?? 'application/octet-stream';
}

$mime = getMimeType(__DIR__ . '/image.jpg');
if ($mime !== null) {
    echo "MIME type: {$mime}\n";
}
```

**说明**：
- `finfo_file()` 是推荐的获取 MIME 类型的方法（需要 fileinfo 扩展）
- `mime_content_type()` 已废弃，但某些系统仍支持
- 可以根据扩展名作为备用方案

## 使用场景

### 场景 1：文件管理界面

在文件管理界面中显示文件的详细信息。

**示例**：

```php
<?php
declare(strict_types=1);

function getFileList(string $dir): array
{
    $files = [];
    $items = scandir($dir);
    
    if ($items === false) {
        return $files;
    }
    
    foreach ($items as $item) {
        if ($item === '.' || $item === '..') {
            continue;
        }
        
        $path = $dir . DIRECTORY_SEPARATOR . $item;
        $stat = stat($path);
        if ($stat === false) {
            continue;
        }
        
        $files[] = [
            'name' => $item,
            'path' => $path,
            'size' => $stat['size'],
            'size_formatted' => formatBytes($stat['size']),
            'modified' => date('Y-m-d H:i:s', $stat['mtime']),
            'type' => is_dir($path) ? 'directory' : 'file',
            'readable' => is_readable($path),
            'writable' => is_writable($path),
        ];
    }
    
    return $files;
}

$files = getFileList(__DIR__);
print_r($files);
```

### 场景 2：文件上传验证

验证上传文件的大小和类型。

**示例**：

```php
<?php
declare(strict_types=1);

function validateUploadedFile(string $filePath, int $maxSize = 5242880): bool
{
    if (!file_exists($filePath)) {
        return false;
    }
    
    // 检查文件大小
    $size = filesize($filePath);
    if ($size === false || $size > $maxSize) {
        return false;
    }
    
    // 检查文件类型（示例：只允许图片）
    $mime = getMimeType($filePath);
    if ($mime === null || !str_starts_with($mime, 'image/')) {
        return false;
    }
    
    return true;
}
```

## 注意事项

### 文件不存在时的处理

获取文件信息前，检查文件是否存在。

**示例**：

```php
<?php
declare(strict_types=1);

function safeGetFileSize(string $file): ?int
{
    if (!file_exists($file)) {
        return null;
    }
    
    $size = filesize($file);
    return $size !== false ? $size : null;
}
```

### 大文件大小的处理

对于大于 2GB 的文件，在 32 位系统上 `filesize()` 可能返回错误的值。

**示例**：

```php
<?php
declare(strict_types=1);

// 对于大文件，可以使用 SplFileInfo
$fileInfo = new SplFileInfo(__DIR__ . '/large-file.zip');
$size = $fileInfo->getSize();
echo "File size: {$size} bytes\n";
```

### 时间戳的时区

文件时间戳是 Unix 时间戳（UTC），格式化时注意时区。

**示例**：

```php
<?php
declare(strict_types=1);

$mtime = filemtime(__DIR__ . '/data.txt');
if ($mtime !== false) {
    // 使用 DateTime 处理时区
    $date = new DateTime('@' . $mtime);
    $date->setTimezone(new DateTimeZone('Asia/Shanghai'));
    echo "Modified: " . $date->format('Y-m-d H:i:s T') . "\n";
}
```

## 常见问题

### 问题 1：如何获取目录大小？

**回答**：需要递归遍历目录中的所有文件，累加文件大小。

**示例**：

```php
<?php
declare(strict_types=1);

function getDirectorySize(string $dir): int
{
    $size = 0;
    $iterator = new RecursiveIteratorIterator(
        new RecursiveDirectoryIterator($dir, RecursiveDirectoryIterator::SKIP_DOTS)
    );
    
    foreach ($iterator as $file) {
        if ($file->isFile()) {
            $size += $file->getSize();
        }
    }
    
    return $size;
}

$size = getDirectorySize(__DIR__);
echo "Directory size: " . formatBytes($size) . "\n";
```

### 问题 2：文件时间戳如何格式化？

**回答**：使用 `date()` 函数或 `DateTime` 类格式化时间戳。

**示例**：

```php
<?php
declare(strict_types=1);

$mtime = filemtime(__DIR__ . '/data.txt');
if ($mtime !== false) {
    // 使用 date() 函数
    echo date('Y-m-d H:i:s', $mtime) . "\n";
    
    // 使用 DateTime 类
    $date = new DateTime('@' . $mtime);
    echo $date->format('Y-m-d H:i:s') . "\n";
}
```

### 问题 3：如何判断文件类型？

**回答**：可以使用 `filetype()`、`mime_content_type()`、`finfo_file()` 或根据扩展名判断。

**示例**：见"示例 8：获取 MIME 类型"

### 问题 4：SplFileInfo 的优势是什么？

**回答**：`SplFileInfo` 提供面向对象的接口，方法更直观，可以打开文件句柄，支持链式调用。

**示例**：

```php
<?php
declare(strict_types=1);

// 使用 SplFileInfo 更直观
$fileInfo = new SplFileInfo(__DIR__ . '/data.txt');
if ($fileInfo->isFile() && $fileInfo->isReadable()) {
    echo "Size: " . $fileInfo->getSize() . "\n";
    echo "Modified: " . date('Y-m-d H:i:s', $fileInfo->getMTime()) . "\n";
}

// 函数式方法需要多次调用
$path = __DIR__ . '/data.txt';
if (is_file($path) && is_readable($path)) {
    $size = filesize($path);
    $mtime = filemtime($path);
    echo "Size: {$size}\n";
    echo "Modified: " . date('Y-m-d H:i:s', $mtime) . "\n";
}
```

## 最佳实践

### 1. 使用 SplFileInfo 获取文件信息

对于需要获取多个文件信息的场景，使用 `SplFileInfo` 更清晰。

**示例**：

```php
<?php
declare(strict_types=1);

$fileInfo = new SplFileInfo(__DIR__ . '/data.txt');
$info = [
    'size' => $fileInfo->getSize(),
    'modified' => $fileInfo->getMTime(),
    'readable' => $fileInfo->isReadable(),
    'writable' => $fileInfo->isWritable(),
];
```

### 2. 检查文件存在性再获取信息

获取文件信息前，先检查文件是否存在。

**示例**：

```php
<?php
declare(strict_types=1);

function getFileInfo(string $path): ?array
{
    if (!file_exists($path)) {
        return null;
    }
    
    $stat = stat($path);
    if ($stat === false) {
        return null;
    }
    
    return [
        'size' => $stat['size'],
        'modified' => $stat['mtime'],
        'type' => filetype($path),
    ];
}
```

### 3. 注意文件时间的时区

格式化文件时间时，注意时区设置。

**示例**：

```php
<?php
declare(strict_types=1);

$mtime = filemtime(__DIR__ . '/data.txt');
if ($mtime !== false) {
    $date = new DateTime('@' . $mtime);
    $date->setTimezone(new DateTimeZone('Asia/Shanghai'));
    echo $date->format('Y-m-d H:i:s T');
}
```

### 4. 大文件大小使用合适的单位显示

格式化文件大小，使用合适的单位（KB、MB、GB）。

**示例**：见"示例 7：格式化文件大小"

### 5. 使用 stat() 一次性获取多个信息

如果需要获取多个文件信息，使用 `stat()` 一次性获取更高效。

**示例**：

```php
<?php
declare(strict_types=1);

$stat = stat(__DIR__ . '/data.txt');
if ($stat !== false) {
    // 一次性获取所有信息
    $info = [
        'size' => $stat['size'],
        'modified' => $stat['mtime'],
        'accessed' => $stat['atime'],
        'changed' => $stat['ctime'],
        'permissions' => substr(sprintf('%o', $stat['mode']), -4),
    ];
}
```

## 对比分析

### 函数式 vs SplFileInfo

| 特性         | 函数式方法                      | SplFileInfo                    |
|:-------------|:-------------------------------|:-------------------------------|
| **接口**      | 函数式                         | 面向对象                       |
| **代码风格**  | 过程式                         | 面向对象                       |
| **易用性**    | ⚠️ 需要多次函数调用            | ✅ 方法调用更直观              |
| **性能**      | ✅ 稍快                        | ⚠️ 稍慢（对象开销）            |
| **功能**      | ⚠️ 基础功能                    | ✅ 更丰富（可打开文件句柄）    |

### filemtime() vs filectime() vs fileatime()

| 函数        | 说明                    | Linux                      | Windows                |
|:------------|:------------------------|:---------------------------|:-----------------------|
| **filemtime()** | 最后修改时间          | 文件内容被修改的时间       | 文件内容被修改的时间   |
| **filectime()** | 创建/变更时间          | inode 变更时间             | 文件创建时间           |
| **fileatime()** | 最后访问时间           | 文件被访问的时间（可能禁用）| 文件被访问的时间       |

## 练习任务

1. **文件信息工具类**：创建一个文件信息工具类，封装常用的文件信息获取方法，包含格式化输出。

2. **目录大小计算**：编写一个函数，计算目录及其所有文件的总大小，支持递归计算。

3. **文件类型检测工具**：实现一个函数，检测文件的 MIME 类型，支持多种检测方法。

4. **文件信息格式化工具**：创建一个工具，将文件信息格式化为易读的格式（大小、时间等）。

5. **文件监控工具**：编写一个工具，监控文件的变化（大小、修改时间等），并记录变化。

## 相关章节

- **[4.2.3 文件操作](section-03-file-operations.md)**：了解文件操作的相关内容
- **[4.2.7 目录操作](section-07-directory-operations.md)**：了解目录操作的相关内容
- **[4.2.11 文件权限](section-11-file-permissions.md)**：学习文件权限的详细内容
