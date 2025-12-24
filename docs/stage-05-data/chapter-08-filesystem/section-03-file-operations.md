# 5.8.3 文件操作

## 概述

文件操作包括复制、重命名、移动和删除文件等操作。这些操作是文件管理的基础功能，理解它们的工作原理和使用场景对于构建健壮的 PHP 应用至关重要。

## copy() - 复制文件

**语法**：`copy(string $from, string $to): bool`

**参数**：
- `$from`：源文件路径
- `$to`：目标文件路径

**返回值**：成功返回 `true`，失败返回 `false`。

**工作原理**：
- 将源文件的内容复制到目标文件
- 如果目标文件已存在，会被覆盖
- 复制操作不会删除源文件

**示例**：

```php
<?php
declare(strict_types=1);

// 基本复制
if (!copy(__DIR__ . '/source.txt', __DIR__ . '/destination.txt')) {
    throw new RuntimeException('Cannot copy file');
}

// 安全复制（检查源文件是否存在）
$source = __DIR__ . '/source.txt';
$destination = __DIR__ . '/destination.txt';

if (!file_exists($source)) {
    throw new RuntimeException("Source file does not exist: {$source}");
}

if (!is_readable($source)) {
    throw new RuntimeException("Source file is not readable: {$source}");
}

if (!copy($source, $destination)) {
    throw new RuntimeException("Cannot copy file: {$source} to {$destination}");
}
```

**注意事项**：
- 目标文件如果已存在，会被覆盖
- 需要确保目标目录存在且有写权限
- 复制大文件可能较慢，考虑使用流式复制

## rename() - 重命名/移动文件

**语法**：`rename(string $from, string $to, ?resource $context = null): bool`

**参数**：
- `$from`：原文件路径
- `$to`：新文件路径
- `$context`：可选，流上下文资源，默认为 `null`

**返回值**：成功返回 `true`，失败返回 `false`。

**工作原理**：
- 如果源文件和目标文件在同一目录，执行重命名操作
- 如果源文件和目标文件在不同目录，执行移动操作
- 操作是原子的（在支持的文件系统上）

**示例**：

```php
<?php
declare(strict_types=1);

// 重命名文件
if (!rename(__DIR__ . '/old.txt', __DIR__ . '/new.txt')) {
    throw new RuntimeException('Cannot rename file');
}

// 移动文件
if (!rename(__DIR__ . '/file.txt', __DIR__ . '/backup/file.txt')) {
    throw new RuntimeException('Cannot move file');
}

// 安全重命名（检查源文件是否存在）
$oldFile = __DIR__ . '/old.txt';
$newFile = __DIR__ . '/new.txt';

if (!file_exists($oldFile)) {
    throw new RuntimeException("Source file does not exist: {$oldFile}");
}

if (!rename($oldFile, $newFile)) {
    throw new RuntimeException("Cannot rename file: {$oldFile} to {$newFile}");
}
```

**注意事项**：
- 如果目标文件已存在，会被覆盖
- 跨文件系统移动可能较慢
- 在某些系统上，移动操作可能不是原子的

## unlink() - 删除文件

**语法**：`unlink(string $filename, ?resource $context = null): bool`

**参数**：
- `$filename`：要删除的文件路径
- `$context`：可选，流上下文资源，默认为 `null`

**返回值**：成功返回 `true`，失败返回 `false`。

**注意事项**：
- 只能删除文件，不能删除目录（使用 `rmdir()` 删除目录）
- 删除操作不可逆，需要谨慎使用
- 删除前应该检查文件是否存在和权限

**示例**：

```php
<?php
declare(strict_types=1);

// 安全删除文件
$file = __DIR__ . '/file.txt';
if (file_exists($file) && is_file($file)) {
    if (unlink($file)) {
        echo "File deleted successfully\n";
    } else {
        throw new RuntimeException("Cannot delete file: {$file}");
    }
}

// 删除多个文件
$files = [
    __DIR__ . '/file1.txt',
    __DIR__ . '/file2.txt',
    __DIR__ . '/file3.txt',
];

foreach ($files as $file) {
    if (file_exists($file) && is_file($file)) {
        unlink($file);
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class FileOperations
{
    /**
     * 复制文件
     */
    public static function copyFile(string $from, string $to): void
    {
        if (!file_exists($from)) {
            throw new RuntimeException("Source file does not exist: {$from}");
        }
        
        if (!is_readable($from)) {
            throw new RuntimeException("Source file is not readable: {$from}");
        }
        
        // 确保目标目录存在
        $targetDir = dirname($to);
        if (!is_dir($targetDir)) {
            mkdir($targetDir, 0755, true);
        }
        
        if (!copy($from, $to)) {
            throw new RuntimeException("Cannot copy file: {$from} to {$to}");
        }
    }
    
    /**
     * 移动文件
     */
    public static function moveFile(string $from, string $to): void
    {
        if (!file_exists($from)) {
            throw new RuntimeException("Source file does not exist: {$from}");
        }
        
        // 确保目标目录存在
        $targetDir = dirname($to);
        if (!is_dir($targetDir)) {
            mkdir($targetDir, 0755, true);
        }
        
        if (!rename($from, $to)) {
            throw new RuntimeException("Cannot move file: {$from} to {$to}");
        }
    }
    
    /**
     * 删除文件
     */
    public static function deleteFile(string $file): void
    {
        if (!file_exists($file)) {
            return;  // 文件不存在，无需删除
        }
        
        if (!is_file($file)) {
            throw new RuntimeException("Path is not a file: {$file}");
        }
        
        if (!unlink($file)) {
            throw new RuntimeException("Cannot delete file: {$file}");
        }
    }
    
    /**
     * 备份文件（复制并添加时间戳）
     */
    public static function backupFile(string $file): string
    {
        if (!file_exists($file)) {
            throw new RuntimeException("File does not exist: {$file}");
        }
        
        $backupFile = $file . '.' . date('Y-m-d_H-i-s') . '.bak';
        self::copyFile($file, $backupFile);
        
        return $backupFile;
    }
}

// 使用示例
try {
    // 复制文件
    FileOperations::copyFile(__DIR__ . '/source.txt', __DIR__ . '/destination.txt');
    
    // 移动文件
    FileOperations::moveFile(__DIR__ . '/old.txt', __DIR__ . '/new.txt');
    
    // 备份文件
    $backup = FileOperations::backupFile(__DIR__ . '/important.txt');
    echo "Backup created: {$backup}\n";
    
    // 删除文件
    FileOperations::deleteFile(__DIR__ . '/temp.txt');
} catch (RuntimeException $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

## 注意事项

1. **错误处理**：始终检查文件操作的返回值。
2. **权限检查**：操作前检查文件权限。
3. **路径安全**：验证文件路径，防止路径遍历攻击。
4. **原子操作**：`rename()` 在支持的文件系统上是原子的，适合用于原子写入。

## 练习

1. 创建一个文件工具类，封装常用的文件操作。
2. 实现一个文件备份功能，复制文件并添加时间戳。
3. 编写一个函数，安全地删除文件（包含权限检查）。
4. 实现一个文件移动功能，支持跨目录移动。
