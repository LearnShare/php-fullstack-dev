# 5.8.7 目录操作

## 概述

目录操作包括创建、删除、遍历目录等。PHP 提供了丰富的目录操作函数。

## 目录创建

### mkdir() - 创建目录

**语法**：`mkdir(string $directory, int $permissions = 0777, bool $recursive = false, ?resource $context = null): bool`

**参数**：
- `$directory`：要创建的目录路径
- `$permissions`：可选，目录权限（八进制），默认为 0777
- `$recursive`：可选，是否递归创建父目录，默认为 `false`
- `$context`：可选，流上下文资源，默认为 `null`

**返回值**：成功返回 `true`，失败返回 `false`。

```php
<?php
declare(strict_types=1);

// 创建单个目录
mkdir(__DIR__ . '/newdir');

// 递归创建多级目录
mkdir(__DIR__ . '/path/to/dir', 0755, true);
```

## 目录删除

### rmdir() - 删除空目录

**语法**：`rmdir(string $directory, ?resource $context = null): bool`

**参数**：
- `$directory`：要删除的目录路径（必须是空目录）
- `$context`：可选，流上下文资源，默认为 `null`

**返回值**：成功返回 `true`，失败返回 `false`。

```php
<?php
declare(strict_types=1);

// 只能删除空目录
rmdir(__DIR__ . '/emptydir');
```

### 递归删除目录

```php
<?php
declare(strict_types=1);

function deleteDirectory(string $dir): bool
{
    if (!is_dir($dir)) {
        return false;
    }
    
    $files = array_diff(scandir($dir), ['.', '..']);
    foreach ($files as $file) {
        $path = $dir . DIRECTORY_SEPARATOR . $file;
        if (is_dir($path)) {
            deleteDirectory($path);
        } else {
            unlink($path);
        }
    }
    
    return rmdir($dir);
}
```

## 目录遍历

### scandir() - 列出目录内容

**语法**：`scandir(string $directory, int $sorting_order = SCANDIR_SORT_ASCENDING, ?resource $context = null): array|false`

**参数**：
- `$directory`：要扫描的目录路径
- `$sorting_order`：可选，排序顺序（`SCANDIR_SORT_ASCENDING`、`SCANDIR_SORT_DESCENDING`、`SCANDIR_SORT_NONE`），默认为 `SCANDIR_SORT_ASCENDING`
- `$context`：可选，流上下文资源，默认为 `null`

**返回值**：成功返回包含文件和目录名的数组，失败返回 `false`。数组包含 `.` 和 `..` 条目。

```php
<?php
declare(strict_types=1);

$files = scandir(__DIR__);
foreach ($files as $file) {
    if ($file !== '.' && $file !== '..') {
        echo $file . "\n";
    }
}
```

### opendir() / readdir() / closedir() - 目录句柄

```php
<?php
declare(strict_types=1);

$handle = opendir(__DIR__);
while (($file = readdir($handle)) !== false) {
    if ($file !== '.' && $file !== '..') {
        echo $file . "\n";
    }
}
closedir($handle);
```

### 递归遍历

```php
<?php
declare(strict_types=1);

function scanDirectory(string $dir): array
{
    $files = [];
    $items = scandir($dir);
    
    foreach ($items as $item) {
        if ($item === '.' || $item === '..') {
            continue;
        }
        
        $path = $dir . DIRECTORY_SEPARATOR . $item;
        if (is_dir($path)) {
            $files = array_merge($files, scanDirectory($path));
        } else {
            $files[] = $path;
        }
    }
    
    return $files;
}
```

## 路径处理

### realpath() - 获取绝对路径

**语法**：`realpath(string $path): string|false`

**参数**：
- `$path`：要解析的路径

**返回值**：成功返回规范化的绝对路径，失败返回 `false`。

```php
<?php
declare(strict_types=1);

$path = realpath(__DIR__ . '/../config');
echo $path . "\n";
```

### dirname() - 获取目录名

**语法**：`dirname(string $path, int $levels = 1): string`

**参数**：
- `$path`：文件路径
- `$levels`：可选，向上返回的目录层级数，默认为 1

**返回值**：返回父目录的路径。

```php
<?php
declare(strict_types=1);

$file = '/path/to/file.txt';
$dir = dirname($file);  // /path/to
```

### basename() - 获取文件名

**语法**：`basename(string $path, string $suffix = ""): string`

**参数**：
- `$path`：文件路径
- `$suffix`：可选，如果文件名以该后缀结尾，则去除该后缀

**返回值**：返回路径中的文件名部分。

```php
<?php
declare(strict_types=1);

$file = '/path/to/file.txt';
$name = basename($file);  // file.txt
$nameWithoutExt = basename($file, '.txt');  // file
```

### pathinfo() - 获取路径信息

**语法**：`pathinfo(string $path, int $flags = PATHINFO_ALL): mixed`

**参数**：
- `$path`：文件路径
- `$flags`：可选，指定返回的信息（`PATHINFO_DIRNAME`、`PATHINFO_BASENAME`、`PATHINFO_EXTENSION`、`PATHINFO_FILENAME` 或 `PATHINFO_ALL`），默认为 `PATHINFO_ALL`

**返回值**：如果指定了 `$flags`，返回对应的字符串；否则返回包含所有信息的关联数组。

```php
<?php
declare(strict_types=1);

$info = pathinfo('/path/to/file.txt');
print_r($info);
// 输出：
// Array
// (
//     [dirname] => /path/to
//     [basename] => file.txt
//     [extension] => txt
//     [filename] => file
// )
```

## 完整示例

```php
<?php
declare(strict_types=1);

class DirectoryOperations
{
    public static function createDirectory(string $path, int $mode = 0755): void
    {
        if (!mkdir($path, $mode, true) && !is_dir($path)) {
            throw new RuntimeException("Cannot create directory: {$path}");
        }
    }
    
    public static function deleteDirectory(string $dir): void
    {
        if (!is_dir($dir)) {
            return;
        }
        
        $files = array_diff(scandir($dir), ['.', '..']);
        foreach ($files as $file) {
            $path = $dir . DIRECTORY_SEPARATOR . $file;
            if (is_dir($path)) {
                self::deleteDirectory($path);
            } else {
                unlink($path);
            }
        }
        
        rmdir($dir);
    }
    
    public static function scanDirectory(string $dir): array
    {
        $files = [];
        $items = scandir($dir);
        
        foreach ($items as $item) {
            if ($item === '.' || $item === '..') {
                continue;
            }
            
            $path = $dir . DIRECTORY_SEPARATOR . $item;
            if (is_dir($path)) {
                $files = array_merge($files, self::scanDirectory($path));
            } else {
                $files[] = $path;
            }
        }
        
        return $files;
    }
}

// 使用示例
DirectoryOperations::createDirectory(__DIR__ . '/test/dir');
$files = DirectoryOperations::scanDirectory(__DIR__);
print_r($files);
```

## 注意事项

1. **权限**：创建目录时注意设置正确的权限。
2. **递归创建**：使用 `recursive` 参数创建多级目录。
3. **路径分隔符**：使用 `DIRECTORY_SEPARATOR` 确保跨平台兼容。
4. **错误处理**：始终检查目录操作的返回值。
5. **性能考虑**：递归遍历大目录可能较慢，考虑使用迭代器。

## 练习

1. 创建一个目录工具类，封装常用的目录操作。
2. 编写一个函数，递归删除目录及其所有内容。
3. 实现一个函数，查找目录中所有特定扩展名的文件。
4. 创建一个函数，计算目录的总大小。
5. 编写一个函数，实现目录的复制功能。
