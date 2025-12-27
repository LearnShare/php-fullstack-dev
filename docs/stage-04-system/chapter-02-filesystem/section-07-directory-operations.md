# 4.2.7 目录操作

## 概述

目录操作是文件系统管理的重要组成部分。在实际应用中，我们经常需要创建目录、删除目录、遍历目录内容、搜索文件等操作。PHP 提供了丰富的目录操作函数和类，包括 `mkdir()`、`rmdir()`、`scandir()`、`glob()`、`DirectoryIterator` 等。理解这些操作方法的工作原理和使用场景，对于构建健壮的文件管理应用至关重要。

目录操作需要考虑多个因素：目录权限、路径分隔符的跨平台兼容性、递归操作的性能、特殊文件（`.` 和 `..`）的处理等。掌握目录操作的基础知识后，可以进一步学习文件信息获取、路径处理、文件搜索等高级主题。

**主要内容**：
- `mkdir()` 函数（创建目录）
- `rmdir()` 函数（删除目录）
- `scandir()` 函数（列出目录内容）
- `opendir()`/`readdir()`/`closedir()`（目录句柄操作）
- `glob()` 函数（模式匹配）
- `DirectoryIterator` 类
- `RecursiveDirectoryIterator` 类
- 路径处理函数（`realpath()`、`dirname()`、`basename()`、`pathinfo()`）

## 特性

- **灵活的目录管理**：支持创建、删除、遍历等操作
- **递归操作支持**：支持递归创建和删除目录
- **模式匹配**：支持使用通配符模式匹配文件
- **迭代器支持**：提供迭代器类方便遍历目录
- **跨平台兼容**：支持 Windows 和类 Unix 系统

## 语法/定义

### mkdir() 函数

**语法**：`mkdir(string $directory, int $permissions = 0777, bool $recursive = false, ?resource $context = null): bool`

**参数**：
- `$directory`：要创建的目录路径
- `$permissions`：可选，目录权限（八进制），默认为 0777
- `$recursive`：可选，是否递归创建父目录，默认为 `false`
- `$context`：可选，流上下文资源，默认为 `null`

**返回值**：成功返回 `true`，失败返回 `false`。

### rmdir() 函数

**语法**：`rmdir(string $directory, ?resource $context = null): bool`

**参数**：
- `$directory`：要删除的目录路径（必须是空目录）
- `$context`：可选，流上下文资源，默认为 `null`

**返回值**：成功返回 `true`，失败返回 `false`。

### scandir() 函数

**语法**：`scandir(string $directory, int $sorting_order = SCANDIR_SORT_ASCENDING, ?resource $context = null): array|false`

**参数**：
- `$directory`：要扫描的目录路径
- `$sorting_order`：可选，排序顺序，默认为 `SCANDIR_SORT_ASCENDING`
- `$context`：可选，流上下文资源，默认为 `null`

**返回值**：成功返回包含文件和目录名的数组，失败返回 `false`。

## 基本用法

### 示例 1：创建目录

```php
<?php
declare(strict_types=1);

// 创建单个目录
mkdir(__DIR__ . '/newdir', 0755);
if (is_dir(__DIR__ . '/newdir')) {
    echo "Directory created\n";
}

// 递归创建多级目录
mkdir(__DIR__ . '/path/to/dir', 0755, true);
if (is_dir(__DIR__ . '/path/to/dir')) {
    echo "Recursive directory created\n";
}
```

**说明**：
- `mkdir()` 创建单个目录
- 第三个参数 `true` 表示递归创建父目录
- 权限 0755 表示所有者可读写执行，组和其他用户可读执行

### 示例 2：删除空目录

```php
<?php
declare(strict_types=1);

// 只能删除空目录
if (is_dir(__DIR__ . '/emptydir')) {
    if (rmdir(__DIR__ . '/emptydir')) {
        echo "Directory deleted\n";
    } else {
        echo "Cannot delete directory (may not be empty)\n";
    }
}
```

**说明**：
- `rmdir()` 只能删除空目录
- 如果目录不为空，删除会失败
- 删除非空目录需要先删除目录中的内容

### 示例 3：递归删除目录

```php
<?php
declare(strict_types=1);

function deleteDirectory(string $dir): bool
{
    if (!is_dir($dir)) {
        return false;
    }
    
    // 获取目录中的所有文件和目录（排除 . 和 ..）
    $files = array_diff(scandir($dir), ['.', '..']);
    
    foreach ($files as $file) {
        $path = $dir . DIRECTORY_SEPARATOR . $file;
        
        if (is_dir($path)) {
            // 递归删除子目录
            deleteDirectory($path);
        } else {
            // 删除文件
            unlink($path);
        }
    }
    
    // 删除空目录
    return rmdir($dir);
}

// 使用
if (deleteDirectory(__DIR__ . '/testdir')) {
    echo "Directory deleted recursively\n";
}
```

**说明**：
- 递归删除目录需要先删除目录中的所有文件和子目录
- 使用 `array_diff()` 排除 `.` 和 `..`
- 使用 `DIRECTORY_SEPARATOR` 确保跨平台兼容

### 示例 4：使用 scandir() 遍历目录

```php
<?php
declare(strict_types=1);

// 列出目录内容
$files = scandir(__DIR__);
if ($files !== false) {
    foreach ($files as $file) {
        // 排除 . 和 ..
        if ($file !== '.' && $file !== '..') {
            $path = __DIR__ . DIRECTORY_SEPARATOR . $file;
            if (is_dir($path)) {
                echo "Directory: {$file}\n";
            } else {
                echo "File: {$file}\n";
            }
        }
    }
}

// 降序排序
$files = scandir(__DIR__, SCANDIR_SORT_DESCENDING);
if ($files !== false) {
    foreach ($files as $file) {
        if ($file !== '.' && $file !== '..') {
            echo "{$file}\n";
        }
    }
}
```

**说明**：
- `scandir()` 返回包含所有文件和目录名的数组
- 数组包含 `.` 和 `..` 条目，需要过滤
- 可以指定排序顺序

### 示例 5：使用 opendir()/readdir()/closedir()

```php
<?php
declare(strict_types=1);

// 打开目录句柄
$handle = opendir(__DIR__);
if ($handle === false) {
    throw new RuntimeException('Cannot open directory');
}

try {
    // 遍历目录
    while (($file = readdir($handle)) !== false) {
        // 排除 . 和 ..
        if ($file !== '.' && $file !== '..') {
            $path = __DIR__ . DIRECTORY_SEPARATOR . $file;
            if (is_dir($path)) {
                echo "Directory: {$file}\n";
            } else {
                echo "File: {$file}\n";
            }
        }
    }
} finally {
    // 关闭目录句柄
    closedir($handle);
}
```

**说明**：
- `opendir()` 打开目录返回目录句柄
- `readdir()` 每次读取一个条目，返回 `false` 表示结束
- `closedir()` 关闭目录句柄

### 示例 6：使用 glob() 模式匹配

```php
<?php
declare(strict_types=1);

// 匹配所有 .txt 文件
$txtFiles = glob(__DIR__ . '/*.txt');
foreach ($txtFiles as $file) {
    echo "Text file: {$file}\n";
}

// 匹配所有 .php 文件
$phpFiles = glob(__DIR__ . '/*.php');
foreach ($phpFiles as $file) {
    echo "PHP file: {$file}\n";
}

// 匹配所有子目录中的 .jpg 文件
$jpgFiles = glob(__DIR__ . '/**/*.jpg', GLOB_BRACE);
foreach ($jpgFiles as $file) {
    echo "Image file: {$file}\n";
}

// 匹配多种扩展名
$imageFiles = glob(__DIR__ . '/*.{jpg,png,gif}', GLOB_BRACE);
foreach ($imageFiles as $file) {
    echo "Image: {$file}\n";
}
```

**说明**：
- `glob()` 使用通配符模式匹配文件
- `*` 匹配任意字符
- `?` 匹配单个字符
- `GLOB_BRACE` 支持扩展的匹配模式

### 示例 7：使用 DirectoryIterator 遍历目录

```php
<?php
declare(strict_types=1);

$iterator = new DirectoryIterator(__DIR__);
foreach ($iterator as $file) {
    // 跳过 . 和 ..
    if ($file->isDot()) {
        continue;
    }
    
    if ($file->isDir()) {
        echo "Directory: " . $file->getFilename() . "\n";
    } else {
        echo "File: " . $file->getFilename() . "\n";
        echo "  Size: " . $file->getSize() . " bytes\n";
        echo "  Modified: " . date('Y-m-d H:i:s', $file->getMTime()) . "\n";
    }
}
```

**说明**：
- `DirectoryIterator` 提供面向对象的目录遍历接口
- `isDot()` 检查是否是 `.` 或 `..`
- `isDir()` 检查是否是目录
- 可以获取文件的详细信息

### 示例 8：使用 RecursiveDirectoryIterator 递归遍历

```php
<?php
declare(strict_types=1);

function scanDirectoryRecursive(string $dir): array
{
    $files = [];
    
    $iterator = new RecursiveIteratorIterator(
        new RecursiveDirectoryIterator($dir, RecursiveDirectoryIterator::SKIP_DOTS),
        RecursiveIteratorIterator::SELF_FIRST
    );
    
    foreach ($iterator as $file) {
        if ($file->isFile()) {
            $files[] = $file->getPathname();
        }
    }
    
    return $files;
}

// 使用
$allFiles = scanDirectoryRecursive(__DIR__);
foreach ($allFiles as $file) {
    echo "{$file}\n";
}
```

**说明**：
- `RecursiveDirectoryIterator` 支持递归遍历目录
- `RecursiveIteratorIterator` 将递归迭代器展开为平面迭代
- `SKIP_DOTS` 标志自动跳过 `.` 和 `..`

### 示例 9：获取路径信息

```php
<?php
declare(strict_types=1);

$file = '/path/to/file.txt';

// 获取目录名
$dir = dirname($file);
echo "Directory: {$dir}\n";  // /path/to

// 获取文件名
$name = basename($file);
echo "Filename: {$name}\n";  // file.txt

// 获取文件名（不含扩展名）
$nameWithoutExt = basename($file, '.txt');
echo "Name without ext: {$nameWithoutExt}\n";  // file

// 获取路径信息数组
$info = pathinfo($file);
print_r($info);
// Array
// (
//     [dirname] => /path/to
//     [basename] => file.txt
//     [extension] => txt
//     [filename] => file
// )

// 获取绝对路径
$realPath = realpath(__DIR__ . '/../config');
if ($realPath !== false) {
    echo "Real path: {$realPath}\n";
}
```

**说明**：
- `dirname()` 获取目录名
- `basename()` 获取文件名，可以去除扩展名
- `pathinfo()` 获取路径的详细信息
- `realpath()` 获取规范化的绝对路径

## 使用场景

### 场景 1：创建目录结构

根据配置创建目录结构。

**示例**：

```php
<?php
declare(strict_types=1);

function createDirectoryStructure(array $structure, string $basePath): void
{
    foreach ($structure as $path) {
        $fullPath = $basePath . DIRECTORY_SEPARATOR . $path;
        if (!is_dir($fullPath)) {
            if (!mkdir($fullPath, 0755, true)) {
                throw new RuntimeException("Cannot create directory: {$fullPath}");
            }
        }
    }
}

$structure = [
    'uploads/images',
    'uploads/documents',
    'cache',
    'logs',
];

createDirectoryStructure($structure, __DIR__);
```

### 场景 2：查找特定类型的文件

在目录中查找特定扩展名的文件。

**示例**：

```php
<?php
declare(strict_types=1);

function findFilesByExtension(string $dir, string $extension): array
{
    $files = [];
    $pattern = $dir . DIRECTORY_SEPARATOR . '*.' . $extension;
    
    $matched = glob($pattern);
    if ($matched !== false) {
        foreach ($matched as $file) {
            if (is_file($file)) {
                $files[] = $file;
            }
        }
    }
    
    return $files;
}

$phpFiles = findFilesByExtension(__DIR__, 'php');
foreach ($phpFiles as $file) {
    echo "{$file}\n";
}
```

### 场景 3：清理临时目录

清理指定目录中的旧文件。

**示例**：

```php
<?php
declare(strict_types=1);

function cleanupOldFiles(string $dir, int $maxAge = 3600): void
{
    if (!is_dir($dir)) {
        return;
    }
    
    $files = scandir($dir);
    if ($files === false) {
        return;
    }
    
    $now = time();
    foreach ($files as $file) {
        if ($file === '.' || $file === '..') {
            continue;
        }
        
        $path = $dir . DIRECTORY_SEPARATOR . $file;
        if (is_file($path)) {
            $fileTime = filemtime($path);
            if ($fileTime !== false && ($now - $fileTime) > $maxAge) {
                unlink($path);
                echo "Deleted old file: {$file}\n";
            }
        }
    }
}

cleanupOldFiles(__DIR__ . '/tmp', 3600);  // 删除超过1小时的文件
```

## 注意事项

### 目录权限

创建目录时注意设置正确的权限，确保应用可以访问。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐的权限设置
$publicDir = __DIR__ . '/public';
mkdir($publicDir, 0755, true);  // 755：所有者可读写执行，其他用户可读执行

$privateDir = __DIR__ . '/private';
mkdir($privateDir, 0700, true);  // 700：只有所有者可以访问
```

### 路径分隔符

使用 `DIRECTORY_SEPARATOR` 确保跨平台兼容性。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 正确：使用 DIRECTORY_SEPARATOR
$path = __DIR__ . DIRECTORY_SEPARATOR . 'subdir' . DIRECTORY_SEPARATOR . 'file.txt';

// ❌ 错误：硬编码路径分隔符（可能在不同平台上失败）
// $path = __DIR__ . '/subdir/file.txt';  // 在 Windows 上可能有问题
```

### 递归操作的性能

递归遍历大目录可能较慢，考虑使用迭代器而不是递归函数。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 推荐：使用迭代器（更高效）
function scanDirectoryWithIterator(string $dir): array
{
    $files = [];
    $iterator = new RecursiveIteratorIterator(
        new RecursiveDirectoryIterator($dir, RecursiveDirectoryIterator::SKIP_DOTS)
    );
    
    foreach ($iterator as $file) {
        if ($file->isFile()) {
            $files[] = $file->getPathname();
        }
    }
    
    return $files;
}

// ⚠️ 不推荐：递归函数（对于大目录可能较慢）
// function scanDirectoryRecursive(string $dir): array { ... }
```

### 特殊文件处理

处理 `.` 和 `..` 特殊目录条目。

**示例**：

```php
<?php
declare(strict_types=1);

$files = scandir(__DIR__);
if ($files !== false) {
    foreach ($files as $file) {
        // 必须排除 . 和 ..
        if ($file !== '.' && $file !== '..') {
            $path = __DIR__ . DIRECTORY_SEPARATOR . $file;
            // 处理文件或目录
        }
    }
}
```

## 常见问题

### 问题 1：如何递归创建目录？

**回答**：使用 `mkdir()` 的第三个参数 `true` 表示递归创建父目录。

**示例**：

```php
<?php
declare(strict_types=1);

// 递归创建多级目录
mkdir(__DIR__ . '/path/to/deep/directory', 0755, true);
```

### 问题 2：如何删除非空目录？

**回答**：需要先递归删除目录中的所有文件和子目录，然后删除空目录。

**示例**：见"示例 3：递归删除目录"

### 问题 3：glob() 和 DirectoryIterator 的区别是什么？

**回答**：
- **glob()**：使用模式匹配，返回匹配的文件路径数组，简单快速
- **DirectoryIterator**：提供面向对象的接口，可以获取文件详细信息，更灵活

**示例**：

```php
<?php
declare(strict_types=1);

// glob()：简单快速
$files = glob(__DIR__ . '/*.php');
foreach ($files as $file) {
    echo "{$file}\n";
}

// DirectoryIterator：更灵活
$iterator = new DirectoryIterator(__DIR__);
foreach ($iterator as $file) {
    if ($file->isFile() && $file->getExtension() === 'php') {
        echo "{$file->getPathname()}\n";
        echo "  Size: {$file->getSize()} bytes\n";
    }
}
```

### 问题 4：如何遍历子目录？

**回答**：使用 `RecursiveDirectoryIterator` 和 `RecursiveIteratorIterator` 进行递归遍历。

**示例**：见"示例 8：使用 RecursiveDirectoryIterator 递归遍历"

## 最佳实践

### 1. 使用递归标志创建目录

创建多级目录时，使用 `recursive` 参数。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 推荐：使用递归标志
mkdir(__DIR__ . '/path/to/dir', 0755, true);

// ❌ 不推荐：手动创建每一级
// mkdir(__DIR__ . '/path', 0755);
// mkdir(__DIR__ . '/path/to', 0755);
// mkdir(__DIR__ . '/path/to/dir', 0755);
```

### 2. 使用迭代器遍历大目录

对于大目录，使用迭代器而不是递归函数，性能更好。

**示例**：

```php
<?php
declare(strict_types=1);

function scanLargeDirectory(string $dir): array
{
    $files = [];
    $iterator = new RecursiveIteratorIterator(
        new RecursiveDirectoryIterator($dir, RecursiveDirectoryIterator::SKIP_DOTS)
    );
    
    foreach ($iterator as $file) {
        if ($file->isFile()) {
            $files[] = $file->getPathname();
        }
    }
    
    return $files;
}
```

### 3. 检查目录权限

操作目录前检查目录是否存在和是否有权限。

**示例**：

```php
<?php
declare(strict_types=1);

function canAccessDirectory(string $dir): bool
{
    if (!is_dir($dir)) {
        return false;
    }
    
    if (!is_readable($dir)) {
        return false;
    }
    
    return true;
}

if (canAccessDirectory(__DIR__)) {
    $files = scandir(__DIR__);
}
```

### 4. 处理特殊文件

遍历目录时，正确处理 `.` 和 `..` 特殊文件。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法 1：使用 array_diff
$files = array_diff(scandir(__DIR__), ['.', '..']);

// 方法 2：手动过滤
$allFiles = scandir(__DIR__);
$files = [];
foreach ($allFiles as $file) {
    if ($file !== '.' && $file !== '..') {
        $files[] = $file;
    }
}

// 方法 3：使用 DirectoryIterator
$iterator = new DirectoryIterator(__DIR__);
foreach ($iterator as $file) {
    if (!$file->isDot()) {
        // 处理文件
    }
}
```

### 5. 使用 DIRECTORY_SEPARATOR 确保跨平台兼容

构建路径时使用 `DIRECTORY_SEPARATOR`，确保跨平台兼容。

**示例**：

```php
<?php
declare(strict_types=1);

function buildPath(string ...$parts): string
{
    return implode(DIRECTORY_SEPARATOR, $parts);
}

$path = buildPath(__DIR__, 'subdir', 'file.txt');
```

## 对比分析

### scandir() vs opendir()/readdir()

| 特性         | scandir()                      | opendir()/readdir()            |
|:-------------|:-------------------------------|:-------------------------------|
| **返回值**    | 数组（所有条目）               | 一次返回一个条目               |
| **内存使用** | 一次性加载所有条目             | 逐个处理，内存友好              |
| **使用场景** | 小到中等目录                   | 大目录                         |
| **代码复杂度** | ✅ 简单                       | ⚠️ 较复杂                      |

### glob() vs DirectoryIterator

| 特性         | glob()                         | DirectoryIterator              |
|:-------------|:-------------------------------|:-------------------------------|
| **模式匹配** | ✅ 支持通配符模式               | ⚠️ 需要手动过滤                |
| **文件信息** | ⚠️ 只有路径                    | ✅ 提供详细信息                |
| **使用场景** | 简单模式匹配                   | 需要详细信息时                 |
| **代码复杂度** | ✅ 简单                       | ⚠️ 较复杂                      |

## 练习任务

1. **目录工具类**：创建一个目录工具类，封装常用的目录操作（创建、删除、遍历），包含完整的错误处理。

2. **递归删除函数**：实现一个函数，递归删除目录及其所有内容，包含权限检查和错误处理。

3. **文件搜索工具**：编写一个函数，在目录中查找特定扩展名的文件，支持递归搜索。

4. **目录大小计算**：创建一个函数，计算目录及其所有文件的总大小。

5. **目录清理工具**：实现一个工具，清理指定目录中超过指定时间的文件。

## 相关章节

- **[4.2.3 文件操作](section-03-file-operations.md)**：了解文件操作的相关内容
- **[4.2.8 文件信息](section-08-file-info.md)**：学习如何获取文件信息
- **[4.2.12 路径处理](section-12-path-handling.md)**：学习路径处理的详细内容
