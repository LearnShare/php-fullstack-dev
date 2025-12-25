# 5.8.8 文件信息

## 概述

获取文件信息是文件操作中的常见需求。PHP 提供了丰富的文件信息函数，包括检查文件是否存在、获取文件大小、时间戳、权限等。理解这些函数的使用方法和注意事项，对于构建健壮的 PHP 应用至关重要。

## 文件存在性检查

### file_exists() - 检查文件是否存在

**语法**：`file_exists(string $filename): bool`

**参数**：
- `$filename`：要检查的文件或目录路径

**返回值**：如果文件或目录存在返回 `true`，否则返回 `false`。

**注意事项**：
- 同时检查文件和目录
- 对于符号链接，检查链接指向的目标
- 对于断开的符号链接，返回 `false`

**示例**：

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';
if (file_exists($file)) {
    echo "File exists\n";
} else {
    echo "File does not exist\n";
}

// 区分文件和目录
if (file_exists($file)) {
    if (is_file($file)) {
        echo "It's a file\n";
    } elseif (is_dir($file)) {
        echo "It's a directory\n";
    }
}
```

### is_file() - 检查是否为文件

**语法**：`is_file(string $filename): bool`

**返回值**：如果是文件返回 `true`，否则返回 `false`。

**与 `file_exists()` 的区别**：
- `file_exists()`：检查文件或目录是否存在
- `is_file()`：只检查是否为文件（且存在）

**示例**：

```php
<?php
declare(strict_types=1);

$path = __DIR__ . '/file.txt';
if (is_file($path)) {
    echo "It's a file\n";
}
```

### is_dir() - 检查是否为目录

**语法**：`is_dir(string $filename): bool`

**返回值**：如果是目录返回 `true`，否则返回 `false`。

**示例**：

```php
<?php
declare(strict_types=1);

$path = __DIR__ . '/directory';
if (is_dir($path)) {
    echo "It's a directory\n";
}
```

### is_link() - 检查是否为符号链接

**语法**：`is_link(string $filename): bool`

**返回值**：如果是符号链接返回 `true`，否则返回 `false`。

**示例**：

```php
<?php
declare(strict_types=1);

$path = __DIR__ . '/symlink';
if (is_link($path)) {
    echo "It's a symbolic link\n";
    echo "Target: " . readlink($path) . "\n";
}
```

## 文件大小

### filesize() - 获取文件大小

**语法**：`filesize(string $filename): int|false`

**参数**：
- `$filename`：文件路径

**返回值**：成功返回文件大小（字节数），失败返回 `false`。

**注意事项**：
- 对于大于 2GB 的文件，在 32 位系统上可能返回错误的值
- 对于目录，返回的值取决于系统

**示例**：

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';
$size = filesize($file);
if ($size !== false) {
    echo "File size: {$size} bytes\n";
    
    // 格式化文件大小
    $units = ['B', 'KB', 'MB', 'GB'];
    $i = 0;
    while ($size >= 1024 && $i < count($units) - 1) {
        $size /= 1024;
        $i++;
    }
    echo "Formatted: " . round($size, 2) . " {$units[$i]}\n";
}
```

## 文件类型

### filetype() - 获取文件类型

**语法**：`filetype(string $filename): string|false`

**返回值**：
- `file`：文件
- `dir`：目录
- `link`：符号链接
- `fifo`：命名管道
- `char`：字符设备
- `block`：块设备
- `unknown`：未知类型
- `false`：失败

**示例**：

```php
<?php
declare(strict_types=1);

$path = __DIR__ . '/file.txt';
$type = filetype($path);
if ($type !== false) {
    echo "File type: {$type}\n";
}
```

## 文件时间戳

PHP 提供了三个时间戳函数：

| 函数 | 说明 | 返回时间 |
| :--- | :--- | :--- |
| `filemtime()` | 最后修改时间 | 文件内容被修改的时间 |
| `filectime()` | 创建/变更时间 | 文件 inode 变更的时间（Linux）或创建时间（Windows） |
| `fileatime()` | 最后访问时间 | 文件被访问的时间 |

### filemtime() - 获取修改时间

**语法**：`filemtime(string $filename): int|false`

**返回值**：成功返回 Unix 时间戳，失败返回 `false`。

**示例**：

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';
$mtime = filemtime($file);
if ($mtime !== false) {
    echo "Last modified: " . date('Y-m-d H:i:s', $mtime) . "\n";
}
```

### filectime() - 获取创建/变更时间

**语法**：`filectime(string $filename): int|false`

**返回值**：成功返回 Unix 时间戳，失败返回 `false`。

**注意事项**：
- Linux：返回 inode 变更时间（权限、所有者等变更）
- Windows：返回文件创建时间

**示例**：

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';
$ctime = filectime($file);
if ($ctime !== false) {
    echo "Created/Changed: " . date('Y-m-d H:i:s', $ctime) . "\n";
}
```

### fileatime() - 获取访问时间

**语法**：`fileatime(string $filename): int|false`

**返回值**：成功返回 Unix 时间戳，失败返回 `false`。

**注意事项**：
- 某些文件系统可能禁用了访问时间更新（性能优化）
- 访问时间可能不准确

**示例**：

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';
$atime = fileatime($file);
if ($atime !== false) {
    echo "Last accessed: " . date('Y-m-d H:i:s', $atime) . "\n";
}
```

## 文件权限检查

### is_readable() - 检查是否可读

**语法**：`is_readable(string $filename): bool`

**返回值**：如果文件存在且可读返回 `true`，否则返回 `false`。

### is_writable() - 检查是否可写

**语法**：`is_writable(string $filename): bool`

**返回值**：如果文件存在且可写返回 `true`，否则返回 `false`。

### is_executable() - 检查是否可执行

**语法**：`is_executable(string $filename): bool`

**返回值**：如果文件存在且可执行返回 `true`，否则返回 `false`。

**示例**：

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';

if (is_readable($file)) {
    echo "File is readable\n";
}

if (is_writable($file)) {
    echo "File is writable\n";
}

if (is_executable($file)) {
    echo "File is executable\n";
}

// 检查目录权限
$dir = __DIR__ . '/directory';
if (is_writable($dir)) {
    echo "Directory is writable\n";
}
```

## stat() - 获取文件详细信息

**语法**：`stat(string $filename): array|false`

**返回值**：成功返回包含文件信息的数组，失败返回 `false`。

**返回的数组包含**：
- `dev`：设备号
- `ino`：inode 号
- `mode`：文件模式（权限）
- `nlink`：硬链接数
- `uid`：所有者用户 ID
- `gid`：所有者组 ID
- `rdev`：设备类型
- `size`：文件大小（字节）
- `atime`：最后访问时间
- `mtime`：最后修改时间
- `ctime`：最后变更时间
- `blksize`：文件系统 I/O 块大小
- `blocks`：分配的 512 字节块数

**示例**：

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/file.txt';
$stat = stat($file);
if ($stat !== false) {
    echo "File size: {$stat['size']} bytes\n";
    echo "Last modified: " . date('Y-m-d H:i:s', $stat['mtime']) . "\n";
    echo "Permissions: " . substr(sprintf('%o', $stat['mode']), -4) . "\n";
    echo "Owner UID: {$stat['uid']}\n";
    echo "Group GID: {$stat['gid']}\n";
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class FileInfo
{
    /**
     * 获取文件详细信息
     */
    public static function getFileInfo(string $path): array
    {
        if (!file_exists($path)) {
            throw new RuntimeException("File does not exist: {$path}");
        }
        
        $stat = stat($path);
        if ($stat === false) {
            throw new RuntimeException("Cannot get file info: {$path}");
        }
        
        return [
            'path' => $path,
            'exists' => true,
            'is_file' => is_file($path),
            'is_dir' => is_dir($path),
            'is_link' => is_link($path),
            'size' => $stat['size'],
            'size_formatted' => self::formatBytes($stat['size']),
            'readable' => is_readable($path),
            'writable' => is_writable($path),
            'executable' => is_executable($path),
            'modified' => $stat['mtime'],
            'modified_formatted' => date('Y-m-d H:i:s', $stat['mtime']),
            'accessed' => $stat['atime'],
            'accessed_formatted' => date('Y-m-d H:i:s', $stat['atime']),
            'changed' => $stat['ctime'],
            'changed_formatted' => date('Y-m-d H:i:s', $stat['ctime']),
            'type' => filetype($path),
            'permissions' => substr(sprintf('%o', $stat['mode']), -4),
            'owner_uid' => $stat['uid'],
            'group_gid' => $stat['gid'],
        ];
    }
    
    /**
     * 格式化文件大小
     */
    private static function formatBytes(int $bytes): string
    {
        $units = ['B', 'KB', 'MB', 'GB', 'TB'];
        $i = 0;
        while ($bytes >= 1024 && $i < count($units) - 1) {
            $bytes /= 1024;
            $i++;
        }
        return round($bytes, 2) . ' ' . $units[$i];
    }
}

// 使用示例
try {
    $info = FileInfo::getFileInfo(__DIR__ . '/file.txt');
    print_r($info);
} catch (RuntimeException $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

## 注意事项

1. **文件存在性**：使用 `file_exists()` 检查文件是否存在，但无法区分文件和目录。
2. **类型检查**：使用 `is_file()` 和 `is_dir()` 区分文件和目录。
3. **权限检查**：操作前检查文件权限（`is_readable()`、`is_writable()`）。
4. **时间戳差异**：理解 `filemtime()`、`filectime()` 和 `fileatime()` 的区别。

## 练习

1. 创建一个文件信息工具类，封装常用的文件信息获取功能。
2. 编写一个函数，获取目录中所有文件的详细信息。
3. 实现一个文件监控功能，检测文件的变化。
4. 创建一个函数，格式化文件大小和时间戳。
