# 4.2.12 路径处理

## 概述

路径处理是文件系统操作中的基础功能。在实际应用中，我们经常需要对路径进行规范化、拼接、解析等操作，比如获取文件的目录名、文件名、扩展名，将相对路径转换为绝对路径等。PHP 提供了丰富的路径处理函数，包括 `realpath()`、`pathinfo()`、`dirname()`、`basename()` 等。

理解路径处理函数的使用方法、跨平台路径处理的注意事项，以及如何安全地拼接和处理路径，对于构建健壮的 PHP 应用至关重要。

**主要内容**：
- `realpath()` 函数（规范化路径）
- `pathinfo()` 函数（路径信息）
- `dirname()` 函数（获取目录名）
- `basename()` 函数（获取文件名）
- `DIRECTORY_SEPARATOR` 常量
- 路径拼接方法
- 跨平台路径处理
- 实际应用场景和最佳实践

## 特性

- **路径规范化**：支持将相对路径转换为绝对路径
- **路径解析**：支持解析路径的各个组件
- **跨平台兼容**：支持 Windows 和类 Unix 系统
- **安全性**：支持安全地拼接路径，防止路径遍历攻击
- **功能丰富**：提供多种路径操作方法

## 语法/定义

### realpath() 函数

**语法**：`realpath(string $path): string|false`

**参数**：
- `$path`：要解析的路径

**返回值**：成功返回规范化的绝对路径，失败返回 `false`。

### pathinfo() 函数

**语法**：`pathinfo(string $path, int $flags = PATHINFO_ALL): mixed`

**参数**：
- `$path`：文件路径
- `$flags`：可选，指定返回的信息（`PATHINFO_DIRNAME`、`PATHINFO_BASENAME`、`PATHINFO_EXTENSION`、`PATHINFO_FILENAME` 或 `PATHINFO_ALL`），默认为 `PATHINFO_ALL`

**返回值**：如果指定了 `$flags`，返回对应的字符串；否则返回包含所有信息的关联数组。

### dirname() 函数

**语法**：`dirname(string $path, int $levels = 1): string`

**参数**：
- `$path`：文件路径
- `$levels`：可选，向上返回的目录层级数，默认为 1

**返回值**：返回父目录的路径。

### basename() 函数

**语法**：`basename(string $path, string $suffix = ""): string`

**参数**：
- `$path`：文件路径
- `$suffix`：可选，如果文件名以该后缀结尾，则去除该后缀

**返回值**：返回路径中的文件名部分。

## 基本用法

### 示例 1：使用 realpath() 规范化路径

```php
<?php
declare(strict_types=1);

// 将相对路径转换为绝对路径
$path = realpath(__DIR__ . '/../config/data.txt');
if ($path !== false) {
    echo "Real path: {$path}\n";
}

// 解析符号链接
$linkPath = realpath(__DIR__ . '/symlink');
if ($linkPath !== false) {
    echo "Symlink target: {$linkPath}\n";
}

// 规范化包含 .. 和 . 的路径
$complexPath = realpath(__DIR__ . '/./../data/./file.txt');
if ($complexPath !== false) {
    echo "Normalized path: {$complexPath}\n";
}
```

**说明**：
- `realpath()` 将相对路径转换为绝对路径
- 解析符号链接，返回链接指向的实际路径
- 规范化路径中的 `.` 和 `..`
- 如果路径不存在，返回 `false`

### 示例 2：使用 pathinfo() 获取路径信息

```php
<?php
declare(strict_types=1);

$file = '/path/to/file.txt';

// 获取所有路径信息
$info = pathinfo($file);
print_r($info);
// Array
// (
//     [dirname] => /path/to
//     [basename] => file.txt
//     [extension] => txt
//     [filename] => file
// )

// 获取特定信息
$dir = pathinfo($file, PATHINFO_DIRNAME);     // /path/to
$basename = pathinfo($file, PATHINFO_BASENAME);  // file.txt
$extension = pathinfo($file, PATHINFO_EXTENSION); // txt
$filename = pathinfo($file, PATHINFO_FILENAME);  // file

echo "Directory: {$dir}\n";
echo "Basename: {$basename}\n";
echo "Extension: {$extension}\n";
echo "Filename: {$filename}\n";
```

**说明**：
- `pathinfo()` 解析路径的各个组件
- 不指定 `$flags` 时返回包含所有信息的数组
- 指定 `$flags` 时返回对应的字符串

### 示例 3：使用 dirname() 获取目录名

```php
<?php
declare(strict_types=1);

$file = '/path/to/file.txt';

// 获取父目录
$dir = dirname($file);
echo "Directory: {$dir}\n";  // /path/to

// 向上返回多个层级
$parentDir = dirname($file, 2);
echo "Parent directory: {$parentDir}\n";  // /path

// 对于相对路径
$relativeFile = 'path/to/file.txt';
$relativeDir = dirname($relativeFile);
echo "Relative directory: {$relativeDir}\n";  // path/to
```

**说明**：
- `dirname()` 返回文件的父目录
- 可以使用 `$levels` 参数向上返回多个层级
- 对于相对路径，返回相对目录

### 示例 4：使用 basename() 获取文件名

```php
<?php
declare(strict_types=1);

$file = '/path/to/file.txt';

// 获取文件名
$name = basename($file);
echo "Filename: {$name}\n";  // file.txt

// 去除扩展名
$nameWithoutExt = basename($file, '.txt');
echo "Name without ext: {$nameWithoutExt}\n";  // file

// 对于目录路径
$dir = '/path/to/directory';
$dirName = basename($dir);
echo "Directory name: {$dirName}\n";  // directory
```

**说明**：
- `basename()` 返回路径中的文件名部分
- 可以使用 `$suffix` 参数去除文件扩展名
- 对于目录路径，返回目录名

### 示例 5：路径拼接

```php
<?php
declare(strict_types=1);

// 方法 1：使用 DIRECTORY_SEPARATOR（推荐）
$path1 = __DIR__ . DIRECTORY_SEPARATOR . 'subdir' . DIRECTORY_SEPARATOR . 'file.txt';

// 方法 2：使用 rtrim() 和 DIRECTORY_SEPARATOR
function joinPath(string ...$parts): string
{
    $path = '';
    foreach ($parts as $part) {
        $path = rtrim($path, DIRECTORY_SEPARATOR) . DIRECTORY_SEPARATOR . trim($part, DIRECTORY_SEPARATOR);
    }
    return $path;
}

$path2 = joinPath(__DIR__, 'subdir', 'file.txt');
echo "Path 1: {$path1}\n";
echo "Path 2: {$path2}\n";

// 方法 3：使用 realpath() 规范化拼接后的路径
$baseDir = realpath(__DIR__);
if ($baseDir !== false) {
    $fullPath = $baseDir . DIRECTORY_SEPARATOR . 'subdir' . DIRECTORY_SEPARATOR . 'file.txt';
    $normalizedPath = realpath($fullPath);
    if ($normalizedPath !== false) {
        echo "Normalized path: {$normalizedPath}\n";
    }
}
```

**说明**：
- 使用 `DIRECTORY_SEPARATOR` 确保跨平台兼容
- 可以创建辅助函数方便路径拼接
- 使用 `realpath()` 规范化拼接后的路径

### 示例 6：安全路径拼接（防止路径遍历攻击）

```php
<?php
declare(strict_types=1);

function safeJoinPath(string $baseDir, string ...$parts): string
{
    // 规范化基准目录
    $base = realpath($baseDir);
    if ($base === false) {
        throw new RuntimeException("Base directory does not exist: {$baseDir}");
    }
    
    // 拼接路径
    $path = $base;
    foreach ($parts as $part) {
        // 去除前导和尾随的分隔符
        $part = trim($part, DIRECTORY_SEPARATOR);
        if ($part === '' || $part === '.') {
            continue;
        }
        
        // 防止路径遍历攻击（..）
        if ($part === '..') {
            throw new InvalidArgumentException('Path traversal (..) is not allowed');
        }
        
        // 拼接
        $path .= DIRECTORY_SEPARATOR . $part;
    }
    
    // 规范化路径
    $realPath = realpath($path);
    if ($realPath === false || !str_starts_with($realPath, $base)) {
        throw new RuntimeException("Invalid path: {$path}");
    }
    
    return $realPath;
}

// 使用
try {
    $safePath = safeJoinPath(__DIR__, 'uploads', 'file.txt');
    echo "Safe path: {$safePath}\n";
} catch (Exception $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

**说明**：
- 安全路径拼接需要验证路径，防止路径遍历攻击
- 确保最终路径在基准目录内
- 使用 `realpath()` 规范化路径

### 示例 7：获取文件扩展名

```php
<?php
declare(strict_types=1);

$file = '/path/to/file.txt';

// 方法 1：使用 pathinfo()
$extension = pathinfo($file, PATHINFO_EXTENSION);
echo "Extension: {$extension}\n";  // txt

// 方法 2：使用 basename() 和 strrpos()
$basename = basename($file);
$extension2 = str_contains($basename, '.') ? substr($basename, strrpos($basename, '.') + 1) : '';
echo "Extension 2: {$extension2}\n";

// 方法 3：使用 SplFileInfo（PHP 5.1.2+）
$fileInfo = new SplFileInfo($file);
$extension3 = $fileInfo->getExtension();
echo "Extension 3: {$extension3}\n";
```

**说明**：
- 多种方法可以获取文件扩展名
- `pathinfo()` 是最常用的方法
- `SplFileInfo::getExtension()` 提供面向对象的接口

### 示例 8：跨平台路径处理

```php
<?php
declare(strict_types=1);

// 使用 DIRECTORY_SEPARATOR 确保跨平台兼容
function buildPath(string ...$parts): string
{
    return implode(DIRECTORY_SEPARATOR, array_filter($parts));
}

$path = buildPath(__DIR__, 'subdir', 'file.txt');
echo "Path: {$path}\n";

// Windows 示例
// DIRECTORY_SEPARATOR = \
// 路径: C:\path\to\file.txt

// Unix/Linux 示例
// DIRECTORY_SEPARATOR = /
// 路径: /path/to/file.txt
```

**说明**：
- 使用 `DIRECTORY_SEPARATOR` 确保跨平台兼容
- Windows 使用反斜杠（`\`），Unix/Linux 使用正斜杠（`/`）
- `DIRECTORY_SEPARATOR` 自动适配当前操作系统

## 使用场景

### 场景 1：配置文件路径处理

处理配置文件的路径，支持相对路径和绝对路径。

**示例**：

```php
<?php
declare(strict_types=1);

function getConfigPath(string $configFile): string
{
    // 如果是绝对路径，直接使用
    if ($configFile[0] === '/' || (PHP_OS_FAMILY === 'Windows' && preg_match('/^[A-Z]:/i', $configFile))) {
        $path = realpath($configFile);
    } else {
        // 相对路径，相对于当前脚本目录
        $path = realpath(__DIR__ . DIRECTORY_SEPARATOR . $configFile);
    }
    
    if ($path === false) {
        throw new RuntimeException("Config file not found: {$configFile}");
    }
    
    return $path;
}

$configPath = getConfigPath('config.json');
echo "Config path: {$configPath}\n";
```

### 场景 2：安全路径验证

验证用户提供的路径是否安全。

**示例**：

```php
<?php
declare(strict_types=1);

function validatePath(string $userPath, string $baseDir): string
{
    // 规范化基准目录
    $base = realpath($baseDir);
    if ($base === false) {
        throw new RuntimeException("Base directory does not exist: {$baseDir}");
    }
    
    // 拼接路径
    $fullPath = $base . DIRECTORY_SEPARATOR . ltrim($userPath, DIRECTORY_SEPARATOR);
    
    // 规范化路径
    $realPath = realpath($fullPath);
    if ($realPath === false) {
        throw new RuntimeException("Path does not exist: {$userPath}");
    }
    
    // 确保路径在基准目录内
    if (!str_starts_with($realPath, $base)) {
        throw new RuntimeException("Path outside base directory: {$userPath}");
    }
    
    return $realPath;
}

// 使用
try {
    $safePath = validatePath('uploads/file.txt', __DIR__);
    echo "Safe path: {$safePath}\n";
} catch (Exception $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

## 注意事项

### 路径分隔符的跨平台处理

使用 `DIRECTORY_SEPARATOR` 确保跨平台兼容性。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 推荐：使用 DIRECTORY_SEPARATOR
$path = __DIR__ . DIRECTORY_SEPARATOR . 'subdir' . DIRECTORY_SEPARATOR . 'file.txt';

// ❌ 不推荐：硬编码路径分隔符（可能在不同平台上失败）
// $path = __DIR__ . '/subdir/file.txt';  // 在 Windows 上可能有问题
```

### 符号链接的解析

`realpath()` 会解析符号链接，返回链接指向的实际路径。

**示例**：

```php
<?php
declare(strict_types=1);

// 如果 path/to 是符号链接
$linkPath = __DIR__ . '/path/to/file.txt';
$realPath = realpath($linkPath);
// $realPath 是符号链接指向的实际路径
```

### 路径安全性检查

处理用户提供的路径时，需要检查路径安全性，防止路径遍历攻击。

**示例**：见"示例 6：安全路径拼接（防止路径遍历攻击）"

## 常见问题

### 问题 1：如何规范化路径？

**回答**：使用 `realpath()` 函数将相对路径转换为绝对路径，并规范化路径中的 `.` 和 `..`。

**示例**：

```php
<?php
declare(strict_types=1);

$path = realpath(__DIR__ . '/../data/./file.txt');
if ($path !== false) {
    echo "Normalized path: {$path}\n";
}
```

### 问题 2：如何获取文件扩展名？

**回答**：使用 `pathinfo()` 函数，指定 `PATHINFO_EXTENSION` 标志。

**示例**：

```php
<?php
declare(strict_types=1);

$extension = pathinfo('/path/to/file.txt', PATHINFO_EXTENSION);
echo "Extension: {$extension}\n";  // txt
```

### 问题 3：如何安全地拼接路径？

**回答**：使用 `DIRECTORY_SEPARATOR` 拼接路径，使用 `realpath()` 规范化路径，并验证路径安全性。

**示例**：见"示例 5：路径拼接"和"示例 6：安全路径拼接"

### 问题 4：Windows 和 Linux 路径如何处理？

**回答**：使用 `DIRECTORY_SEPARATOR` 常量，它会自动适配当前操作系统的路径分隔符。

**示例**：

```php
<?php
declare(strict_types=1);

// DIRECTORY_SEPARATOR 自动适配
// Windows: \
// Unix/Linux: /

$path = __DIR__ . DIRECTORY_SEPARATOR . 'file.txt';
```

## 最佳实践

### 1. 使用 realpath() 规范化路径

对于需要绝对路径的场景，使用 `realpath()` 规范化路径。

**示例**：

```php
<?php
declare(strict_types=1);

$path = realpath(__DIR__ . '/../data/file.txt');
if ($path !== false) {
    // 使用规范化的路径
    $content = file_get_contents($path);
}
```

### 2. 使用 DIRECTORY_SEPARATOR 跨平台

构建路径时，使用 `DIRECTORY_SEPARATOR` 确保跨平台兼容。

**示例**：

```php
<?php
declare(strict_types=1);

function buildPath(string ...$parts): string
{
    return implode(DIRECTORY_SEPARATOR, array_filter($parts));
}

$path = buildPath(__DIR__, 'subdir', 'file.txt');
```

### 3. 检查路径安全性

处理用户提供的路径时，验证路径安全性，防止路径遍历攻击。

**示例**：见"示例 6：安全路径拼接（防止路径遍历攻击）"

### 4. 使用 pathinfo() 提取路径组件

需要获取路径的各个组件时，使用 `pathinfo()` 函数。

**示例**：

```php
<?php
declare(strict_types=1);

$info = pathinfo('/path/to/file.txt');
echo "Directory: {$info['dirname']}\n";
echo "Basename: {$info['basename']}\n";
echo "Extension: {$info['extension']}\n";
echo "Filename: {$info['filename']}\n";
```

### 5. 使用 dirname() 和 basename() 提取部分路径

需要获取目录名或文件名时，使用 `dirname()` 和 `basename()`。

**示例**：

```php
<?php
declare(strict_types=1);

$file = '/path/to/file.txt';
$dir = dirname($file);      // /path/to
$name = basename($file);    // file.txt
```

## 对比分析

### pathinfo() vs dirname()/basename()

| 特性         | pathinfo()                  | dirname()/basename()        |
|:-------------|:----------------------------|:----------------------------|
| **功能**      | 获取所有路径信息            | 分别获取目录名和文件名      |
| **返回值**    | 数组或字符串                | 字符串                      |
| **使用场景**  | 需要多个路径组件            | 只需要目录名或文件名        |
| **灵活性**    | ✅ 更灵活（可指定标志）     | ⚠️ 功能单一                  |

### realpath() vs 相对路径

| 特性         | realpath()                  | 相对路径                    |
|:-------------|:----------------------------|:----------------------------|
| **路径类型**  | 绝对路径                    | 相对路径                    |
| **规范化**    | ✅ 自动规范化               | ⚠️ 可能包含 . 和 ..          |
| **符号链接**  | ✅ 自动解析                 | ⚠️ 不解析                   |
| **使用场景**  | 需要绝对路径时              | 相对路径更方便时            |

## 练习任务

1. **路径工具类**：创建一个路径工具类，封装常用的路径操作（拼接、规范化、验证等）。

2. **安全路径验证工具**：实现一个工具，验证用户提供的路径是否安全，防止路径遍历攻击。

3. **路径规范化工具**：编写一个函数，规范化包含 `.` 和 `..` 的复杂路径。

4. **跨平台路径处理工具**：创建一个工具，在不同平台上正确处理路径分隔符。

5. **路径组件提取工具**：实现一个工具，方便地提取路径的各个组件（目录、文件名、扩展名等）。

## 相关章节

- **[4.2.3 文件操作](section-03-file-operations.md)**：了解文件操作的相关内容
- **[4.2.7 目录操作](section-07-directory-operations.md)**：了解目录操作的相关内容
- **[4.2.8 文件信息](section-08-file-info.md)**：了解如何获取文件信息
