# 4.2.3 文件操作

## 概述

文件操作包括复制、重命名、移动和删除文件等基本操作。这些操作是文件管理的基础功能，在 PHP 应用中经常需要用到，比如文件备份、文件上传后的处理、临时文件清理等。理解这些操作的工作原理、适用场景，以及如何安全高效地执行这些操作，对于构建健壮的 PHP 应用至关重要。

文件操作需要特别注意权限检查、错误处理、路径安全性等问题。不同的操作有不同的使用场景和注意事项，比如复制操作不会删除源文件，移动操作在支持的文件系统上是原子的，删除操作不可逆等。

掌握文件操作的基础知识后，可以进一步学习目录操作、文件权限、文件锁定等高级主题。

**主要内容**：
- `copy()` 函数（文件复制）
- `rename()` 函数（移动/重命名）
- `unlink()` 函数（文件删除）
- `file_exists()` 函数（文件存在性检查）
- `is_file()` 和 `is_dir()` 函数
- 文件权限检查（`is_readable()`、`is_writable()`）
- 错误处理和最佳实践

## 特性

- **基本的文件管理**：支持复制、移动、删除等基本操作
- **原子操作支持**：某些操作（如 `rename()`）在支持的文件系统上是原子的
- **权限检查**：支持检查文件权限
- **错误处理**：提供错误返回值，便于错误处理
- **跨平台兼容**：支持 Windows 和类 Unix 系统

## 语法/定义

### copy() 函数

**语法**：`copy(string $from, string $to): bool`

**参数**：
- `$from`：源文件路径
- `$to`：目标文件路径

**返回值**：成功返回 `true`，失败返回 `false`。

### rename() 函数

**语法**：`rename(string $from, string $to, ?resource $context = null): bool`

**参数**：
- `$from`：原文件路径
- `$to`：新文件路径
- `$context`：可选，流上下文资源，默认为 `null`

**返回值**：成功返回 `true`，失败返回 `false`。

### unlink() 函数

**语法**：`unlink(string $filename, ?resource $context = null): bool`

**参数**：
- `$filename`：要删除的文件路径
- `$context`：可选，流上下文资源，默认为 `null`

**返回值**：成功返回 `true`，失败返回 `false`。

### file_exists() 函数

**语法**：`file_exists(string $filename): bool`

**参数**：
- `$filename`：文件或目录路径

**返回值**：文件或目录存在返回 `true`，不存在返回 `false`。

## 基本用法

### 示例 1：复制文件

```php
<?php
declare(strict_types=1);

// 基本复制
$source = __DIR__ . '/source.txt';
$destination = __DIR__ . '/destination.txt';

if (copy($source, $destination)) {
    echo "File copied successfully\n";
} else {
    echo "Failed to copy file\n";
}

// 安全复制（检查源文件是否存在和可读）
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

**说明**：
- `copy()` 将源文件的内容复制到目标文件
- 如果目标文件已存在，会被覆盖
- 复制操作不会删除源文件
- 需要确保目标目录存在且有写权限

### 示例 2：重命名文件

```php
<?php
declare(strict_types=1);

// 重命名文件（同一目录）
$oldFile = __DIR__ . '/old.txt';
$newFile = __DIR__ . '/new.txt';

if (rename($oldFile, $newFile)) {
    echo "File renamed successfully\n";
} else {
    echo "Failed to rename file\n";
}

// 安全重命名（检查源文件是否存在）
if (!file_exists($oldFile)) {
    throw new RuntimeException("Source file does not exist: {$oldFile}");
}

if (!rename($oldFile, $newFile)) {
    throw new RuntimeException("Cannot rename file: {$oldFile} to {$newFile}");
}
```

**说明**：
- `rename()` 可以重命名文件（同一目录）或移动文件（不同目录）
- 如果目标文件已存在，会被覆盖
- 操作在支持的文件系统上是原子的

### 示例 3：移动文件

```php
<?php
declare(strict_types=1);

// 移动文件（跨目录）
$source = __DIR__ . '/file.txt';
$destination = __DIR__ . '/backup/file.txt';

// 确保目标目录存在
$targetDir = dirname($destination);
if (!is_dir($targetDir)) {
    mkdir($targetDir, 0755, true);
}

if (rename($source, $destination)) {
    echo "File moved successfully\n";
} else {
    echo "Failed to move file\n";
}
```

**说明**：
- `rename()` 在不同目录之间移动文件
- 需要确保目标目录存在
- 跨文件系统移动可能较慢

### 示例 4：删除文件

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
} else {
    echo "File does not exist\n";
}
```

**说明**：
- `unlink()` 只能删除文件，不能删除目录（使用 `rmdir()` 删除目录）
- 删除操作不可逆，需要谨慎使用
- 删除前应该检查文件是否存在和权限

### 示例 5：检查文件是否存在

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/data.txt';

// 检查文件是否存在
if (file_exists($file)) {
    echo "File exists\n";
    
    // 检查是否是文件（不是目录）
    if (is_file($file)) {
        echo "It is a file\n";
    }
    
    // 检查是否是目录
    if (is_dir($file)) {
        echo "It is a directory\n";
    }
} else {
    echo "File does not exist\n";
}
```

**说明**：
- `file_exists()` 检查文件或目录是否存在
- `is_file()` 检查是否是文件（不是目录）
- `is_dir()` 检查是否是目录

### 示例 6：检查文件权限

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/data.txt';

if (file_exists($file)) {
    // 检查是否可读
    if (is_readable($file)) {
        echo "File is readable\n";
    } else {
        echo "File is not readable\n";
    }
    
    // 检查是否可写
    if (is_writable($file)) {
        echo "File is writable\n";
    } else {
        echo "File is not writable\n";
    }
    
    // 检查是否可执行
    if (is_executable($file)) {
        echo "File is executable\n";
    } else {
        echo "File is not executable\n";
    }
}
```

**说明**：
- `is_readable()` 检查文件是否可读
- `is_writable()` 检查文件是否可写
- `is_executable()` 检查文件是否可执行

### 示例 7：批量删除文件

```php
<?php
declare(strict_types=1);

// 删除多个文件
$files = [
    __DIR__ . '/file1.txt',
    __DIR__ . '/file2.txt',
    __DIR__ . '/file3.txt',
];

foreach ($files as $file) {
    if (file_exists($file) && is_file($file)) {
        if (unlink($file)) {
            echo "Deleted: {$file}\n";
        } else {
            echo "Failed to delete: {$file}\n";
        }
    }
}
```

**说明**：
- 可以循环删除多个文件
- 每次删除前都应该检查文件是否存在

### 示例 8：文件备份（复制并添加时间戳）

```php
<?php
declare(strict_types=1);

function backupFile(string $file): string
{
    if (!file_exists($file)) {
        throw new RuntimeException("File does not exist: {$file}");
    }
    
    if (!is_file($file)) {
        throw new RuntimeException("Path is not a file: {$file}");
    }
    
    // 创建备份文件名（添加时间戳）
    $backupFile = $file . '.' . date('Y-m-d_H-i-s') . '.bak';
    
    if (!copy($file, $backupFile)) {
        throw new RuntimeException("Cannot create backup: {$backupFile}");
    }
    
    return $backupFile;
}

// 使用
try {
    $backup = backupFile(__DIR__ . '/important.txt');
    echo "Backup created: {$backup}\n";
} catch (RuntimeException $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

**说明**：
- 备份文件通常添加时间戳或日期后缀
- 使用 `copy()` 而不是 `rename()`，保留原文件

## 使用场景

### 场景 1：文件备份

创建文件备份。

**示例**：

```php
<?php
declare(strict_types=1);

function createBackup(string $file): string
{
    if (!file_exists($file)) {
        throw new RuntimeException("File does not exist: {$file}");
    }
    
    $backupFile = $file . '.backup.' . date('YmdHis');
    if (!copy($file, $backupFile)) {
        throw new RuntimeException("Cannot create backup");
    }
    
    return $backupFile;
}

$backup = createBackup(__DIR__ . '/config.json');
echo "Backup: {$backup}\n";
```

### 场景 2：文件上传后的处理

移动上传的文件到目标目录。

**示例**：

```php
<?php
declare(strict_types=1);

function moveUploadedFile(string $tmpFile, string $destination): bool
{
    // 确保目标目录存在
    $targetDir = dirname($destination);
    if (!is_dir($targetDir)) {
        mkdir($targetDir, 0755, true);
    }
    
    // 移动文件
    return rename($tmpFile, $destination);
}

$uploadedFile = $_FILES['file']['tmp_name'] ?? '';
$destination = __DIR__ . '/uploads/' . basename($_FILES['file']['name']);

if ($uploadedFile !== '' && is_uploaded_file($uploadedFile)) {
    if (moveUploadedFile($uploadedFile, $destination)) {
        echo "File uploaded successfully\n";
    }
}
```

### 场景 3：临时文件清理

删除临时文件。

**示例**：

```php
<?php
declare(strict_types=1);

function cleanupTempFiles(string $tempDir, int $maxAge = 3600): void
{
    if (!is_dir($tempDir)) {
        return;
    }
    
    $files = glob($tempDir . '/*');
    $now = time();
    
    foreach ($files as $file) {
        if (is_file($file)) {
            // 删除超过指定时间的文件
            if (($now - filemtime($file)) > $maxAge) {
                unlink($file);
                echo "Deleted old temp file: {$file}\n";
            }
        }
    }
}

cleanupTempFiles(__DIR__ . '/tmp', 3600);  // 删除超过1小时的文件
```

## 注意事项

### 权限检查

操作文件前应该检查文件权限，确保有足够的权限执行操作。

**示例**：

```php
<?php
declare(strict_types=1);

function safeCopy(string $source, string $destination): bool
{
    if (!file_exists($source)) {
        return false;
    }
    
    if (!is_readable($source)) {
        return false;
    }
    
    $targetDir = dirname($destination);
    if (!is_dir($targetDir) || !is_writable($targetDir)) {
        return false;
    }
    
    return copy($source, $destination);
}
```

### 目标目录不存在

复制或移动文件前，确保目标目录存在。

**示例**：

```php
<?php
declare(strict_types=1);

function safeMove(string $source, string $destination): bool
{
    $targetDir = dirname($destination);
    if (!is_dir($targetDir)) {
        if (!mkdir($targetDir, 0755, true)) {
            return false;
        }
    }
    
    return rename($source, $destination);
}
```

### 删除操作不可逆

删除操作不可逆，删除前应该确认。

**示例**：

```php
<?php
declare(strict_types=1);

function safeDelete(string $file, bool $confirm = false): bool
{
    if (!$confirm) {
        return false;  // 需要确认
    }
    
    if (!file_exists($file) || !is_file($file)) {
        return false;
    }
    
    return unlink($file);
}

// 需要明确确认才能删除
if (safeDelete(__DIR__ . '/important.txt', true)) {
    echo "File deleted\n";
}
```

### 跨文件系统移动

跨文件系统移动可能较慢，在某些情况下可能失败。

**示例**：

```php
<?php
declare(strict_types=1);

function moveFileCrossFilesystem(string $source, string $destination): bool
{
    // 先尝试直接移动
    if (rename($source, $destination)) {
        return true;
    }
    
    // 如果失败，使用复制+删除的方式
    if (copy($source, $destination)) {
        return unlink($source);
    }
    
    return false;
}
```

## 常见问题

### 问题 1：copy() 和 rename() 的区别是什么？

**回答**：
- **copy()**：复制文件，保留源文件，创建新文件
- **rename()**：重命名或移动文件，源文件不存在了（在同一文件系统上）

**示例**：

```php
<?php
declare(strict_types=1);

// copy()：源文件保留
copy('source.txt', 'copy.txt');  // source.txt 仍然存在

// rename()：源文件不存在了（被移动/重命名）
rename('old.txt', 'new.txt');  // old.txt 不存在了，变成了 new.txt
```

### 问题 2：如何安全地删除文件？

**回答**：删除前检查文件是否存在、是否是文件、是否有权限。

**示例**：

```php
<?php
declare(strict_types=1);

function safeDeleteFile(string $file): bool
{
    if (!file_exists($file)) {
        return false;  // 文件不存在
    }
    
    if (!is_file($file)) {
        return false;  // 不是文件（可能是目录）
    }
    
    if (!is_writable($file)) {
        return false;  // 没有写权限
    }
    
    return unlink($file);
}
```

### 问题 3：文件移动失败的原因？

**回答**：可能的原因包括：源文件不存在、目标目录不存在、权限不足、跨文件系统问题等。

**示例**：

```php
<?php
declare(strict_types=1);

function moveFileSafely(string $source, string $destination): bool
{
    // 检查源文件
    if (!file_exists($source)) {
        error_log("Source file does not exist: {$source}");
        return false;
    }
    
    // 确保目标目录存在
    $targetDir = dirname($destination);
    if (!is_dir($targetDir)) {
        if (!mkdir($targetDir, 0755, true)) {
            error_log("Cannot create target directory: {$targetDir}");
            return false;
        }
    }
    
    // 检查目标目录权限
    if (!is_writable($targetDir)) {
        error_log("Target directory is not writable: {$targetDir}");
        return false;
    }
    
    return rename($source, $destination);
}
```

### 问题 4：如何检查文件是否可写？

**回答**：使用 `is_writable()` 函数检查文件或目录是否可写。

**示例**：

```php
<?php
declare(strict_types=1);

$file = __DIR__ . '/data.txt';

// 检查文件是否可写
if (is_writable($file)) {
    echo "File is writable\n";
} else {
    echo "File is not writable\n";
}

// 检查目录是否可写
$dir = __DIR__ . '/uploads';
if (is_writable($dir)) {
    echo "Directory is writable\n";
} else {
    echo "Directory is not writable\n";
}
```

## 最佳实践

### 1. 操作前检查文件存在性

在执行文件操作前，先检查文件是否存在。

**示例**：

```php
<?php
declare(strict_types=1);

function safeCopyFile(string $source, string $destination): bool
{
    if (!file_exists($source)) {
        return false;
    }
    
    return copy($source, $destination);
}
```

### 2. 使用异常处理错误情况

对于关键操作，使用异常处理错误。

**示例**：

```php
<?php
declare(strict_types=1);

function copyFileWithException(string $source, string $destination): void
{
    if (!file_exists($source)) {
        throw new RuntimeException("Source file does not exist: {$source}");
    }
    
    if (!copy($source, $destination)) {
        throw new RuntimeException("Cannot copy file: {$source} to {$destination}");
    }
}
```

### 3. 重要文件删除前备份

删除重要文件前，先创建备份。

**示例**：

```php
<?php
declare(strict_types=1);

function deleteWithBackup(string $file): bool
{
    if (!file_exists($file)) {
        return false;
    }
    
    // 创建备份
    $backup = $file . '.backup.' . date('YmdHis');
    if (!copy($file, $backup)) {
        return false;
    }
    
    // 删除原文件
    return unlink($file);
}
```

### 4. 检查操作权限

操作前检查是否有足够的权限。

**示例**：

```php
<?php
declare(strict_types=1);

function canOperateFile(string $file, string $operation): bool
{
    if (!file_exists($file)) {
        return false;
    }
    
    switch ($operation) {
        case 'read':
            return is_readable($file);
        case 'write':
            return is_writable($file);
        case 'delete':
            return is_writable(dirname($file));
        default:
            return false;
    }
}
```

### 5. 确保目标目录存在

复制或移动文件前，确保目标目录存在。

**示例**：

```php
<?php
declare(strict_types=1);

function ensureDirectoryExists(string $path): bool
{
    $dir = dirname($path);
    if (!is_dir($dir)) {
        return mkdir($dir, 0755, true);
    }
    return true;
}

function copyFileSafe(string $source, string $destination): bool
{
    if (!ensureDirectoryExists($destination)) {
        return false;
    }
    
    return copy($source, $destination);
}
```

## 对比分析

### copy() vs rename()

| 特性         | copy()                        | rename()                      |
|:-------------|:------------------------------|:------------------------------|
| **源文件**    | 保留                          | 不存在了（移动/重命名）        |
| **目标文件**  | 新创建                        | 源文件改名或移动               |
| **适用场景**  | 备份、复制                    | 移动、重命名                   |
| **性能**      | 较慢（需要复制数据）           | 较快（只是修改目录项）         |

### file_exists() vs is_file()/is_dir()

| 特性         | file_exists()                 | is_file()/is_dir()            |
|:-------------|:------------------------------|:------------------------------|
| **检查对象**  | 文件或目录                     | 文件或目录（分别检查）          |
| **返回值**    | 存在返回 true                 | 是文件/目录返回 true            |
| **使用场景**  | 检查路径是否存在               | 区分文件和目录                 |

## 练习任务

1. **文件工具类**：创建一个文件工具类，封装常用的文件操作（复制、移动、删除），包含完整的错误处理。

2. **文件备份功能**：实现一个文件备份功能，复制文件并添加时间戳后缀。

3. **安全删除函数**：编写一个函数，安全地删除文件，包含权限检查、确认机制。

4. **文件移动功能**：实现一个文件移动功能，支持跨目录移动，自动创建目标目录。

5. **临时文件清理工具**：创建一个工具，清理指定目录中超过指定时间的临时文件。

## 相关章节

- **[4.2.1 文件读取](section-01-file-reading.md)**：学习如何读取文件
- **[4.2.2 文件写入](section-02-file-writing.md)**：学习如何写入文件
- **[4.2.7 目录操作](section-07-directory-operations.md)**：学习目录操作的相关内容
