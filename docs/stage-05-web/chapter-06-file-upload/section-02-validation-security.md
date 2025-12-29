# 5.6.2 文件验证与安全

## 概述

文件验证和安全处理是文件上传功能的关键环节。文件上传是 Web 应用中最容易受到攻击的功能之一，恶意文件上传可能导致服务器被入侵、数据泄露等严重后果。理解文件验证的方法、掌握安全处理策略、防范常见攻击，对于构建安全的文件上传功能至关重要。本节详细介绍文件类型验证、文件大小验证、文件内容验证、安全文件名生成、恶意文件防护等内容，帮助零基础学员掌握安全的文件上传处理。

文件上传的安全风险包括：恶意脚本上传、路径遍历攻击、文件覆盖、DoS 攻击等。通过严格的验证和安全处理，可以有效防范这些风险。

**主要内容**：
- 文件验证概述（为什么需要验证、验证的层次）
- 文件类型验证（MIME 类型检查、扩展名验证、文件头验证）
- 文件大小验证（大小限制、空文件检查）
- 文件内容验证（图片验证、文档验证、二进制文件检查）
- 安全文件名生成（随机文件名、路径安全、扩展名处理）
- 恶意文件防护（可执行文件检测、脚本文件检测、文件内容扫描）
- 实际应用示例和最佳实践

## 特性

- **多层验证**：扩展名、MIME 类型、文件内容多重验证
- **服务器端验证**：不信任客户端提供的信息
- **白名单机制**：只允许安全的文件类型
- **内容验证**：验证文件实际内容而不仅是扩展名
- **安全存储**：生成安全文件名和存储路径

## 文件验证概述

### 为什么需要验证

文件上传存在多种安全风险：

1. **恶意脚本上传**：上传 PHP、JSP 等可执行脚本
2. **路径遍历攻击**：使用 `../` 访问系统文件
3. **文件覆盖**：覆盖重要文件
4. **DoS 攻击**：上传超大文件消耗服务器资源
5. **病毒文件**：上传包含病毒的文件

### 验证的层次

1. **客户端验证**：提供用户体验，但不可信
2. **服务器端验证**：必须进行，确保安全
   - 扩展名验证
   - MIME 类型验证
   - 文件内容验证
   - 文件大小验证

### 验证原则

- **白名单机制**：只允许明确安全的文件类型
- **不信任客户端**：客户端提供的信息都不可信
- **多重验证**：使用多种方法验证
- **内容验证**：验证文件实际内容

## 文件类型验证

### 方法一：扩展名验证

**基本用法**：
```php
<?php
declare(strict_types=1);

function validateFileExtension(string $filename, array $allowedExtensions): bool
{
    $extension = strtolower(pathinfo($filename, PATHINFO_EXTENSION));
    return in_array($extension, $allowedExtensions, true);
}

// 使用
$file = $_FILES['file'];
$allowedExtensions = ['jpg', 'jpeg', 'png', 'gif'];

if (!validateFileExtension($file['name'], $allowedExtensions)) {
    die('不允许的文件类型');
}
```

**注意**：扩展名验证不够安全，容易被绕过（如 `file.php.jpg`），必须结合其他验证方法。

### 方法二：MIME 类型验证

**使用 finfo_file()**：
```php
<?php
declare(strict_types=1);

function getMimeType(string $filepath): string|false
{
    $finfo = finfo_open(FILEINFO_MIME_TYPE);
    $mimeType = finfo_file($finfo, $filepath);
    finfo_close($finfo);
    return $mimeType;
}

function validateMimeType(string $filepath, array $allowedMimeTypes): bool
{
    $mimeType = getMimeType($filepath);
    if ($mimeType === false) {
        return false;
    }
    return in_array($mimeType, $allowedMimeTypes, true);
}

// 使用
$file = $_FILES['file'];
$allowedMimeTypes = ['image/jpeg', 'image/png', 'image/gif'];

if (!validateMimeType($file['tmp_name'], $allowedMimeTypes)) {
    die('不允许的文件类型');
}
```

**注意**：MIME 类型也可能被伪造，必须结合文件内容验证。

### 方法三：文件头验证（推荐）

**文件头（Magic Bytes）**是文件开头的特定字节序列，用于标识文件类型。

**常见文件头**：
| 文件类型 | 文件头（十六进制） |
|:---------|:------------------|
| JPEG | `FF D8 FF` |
| PNG | `89 50 4E 47` |
| GIF | `47 49 46 38` |
| PDF | `25 50 44 46` |

**示例**：
```php
<?php
declare(strict_types=1);

function validateFileHeader(string $filepath, array $allowedHeaders): bool
{
    $handle = fopen($filepath, 'rb');
    if ($handle === false) {
        return false;
    }
    
    $header = fread($handle, 8);
    fclose($handle);
    
    if ($header === false) {
        return false;
    }
    
    foreach ($allowedHeaders as $allowedHeader) {
        $headerHex = bin2hex($header);
        if (str_starts_with($headerHex, strtolower($allowedHeader))) {
            return true;
        }
    }
    
    return false;
}

// 使用
$file = $_FILES['file'];
$allowedHeaders = [
    'ffd8ff',  // JPEG
    '89504e47', // PNG
    '47494638', // GIF
];

if (!validateFileHeader($file['tmp_name'], $allowedHeaders)) {
    die('不允许的文件类型');
}
```

### 综合验证示例

**示例**：
```php
<?php
declare(strict_types=1);

class FileTypeValidator
{
    private const ALLOWED_EXTENSIONS = ['jpg', 'jpeg', 'png', 'gif'];
    private const ALLOWED_MIME_TYPES = ['image/jpeg', 'image/png', 'image/gif'];
    private const ALLOWED_HEADERS = [
        'ffd8ff',    // JPEG
        '89504e47',  // PNG
        '47494638',  // GIF
    ];

    public static function validate(array $file): bool
    {
        // 1. 扩展名验证
        $extension = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
        if (!in_array($extension, self::ALLOWED_EXTENSIONS, true)) {
            return false;
        }

        // 2. MIME 类型验证
        $mimeType = self::getMimeType($file['tmp_name']);
        if ($mimeType === false || !in_array($mimeType, self::ALLOWED_MIME_TYPES, true)) {
            return false;
        }

        // 3. 文件头验证
        if (!self::validateFileHeader($file['tmp_name'], self::ALLOWED_HEADERS)) {
            return false;
        }

        return true;
    }

    private static function getMimeType(string $filepath): string|false
    {
        $finfo = finfo_open(FILEINFO_MIME_TYPE);
        $mimeType = finfo_file($finfo, $filepath);
        finfo_close($finfo);
        return $mimeType;
    }

    private static function validateFileHeader(string $filepath, array $allowedHeaders): bool
    {
        $handle = fopen($filepath, 'rb');
        if ($handle === false) {
            return false;
        }
        
        $header = fread($handle, 8);
        fclose($handle);
        
        if ($header === false) {
            return false;
        }
        
        $headerHex = bin2hex($header);
        foreach ($allowedHeaders as $allowedHeader) {
            if (str_starts_with($headerHex, strtolower($allowedHeader))) {
                return true;
            }
        }
        
        return false;
    }
}
```

## 文件大小验证

### 基本验证

**示例**：
```php
<?php
declare(strict_types=1);

function validateFileSize(int $size, int $maxSize = 5242880): bool
{
    if ($size === 0) {
        return false;  // 空文件
    }
    
    if ($size > $maxSize) {
        return false;  // 超过最大大小
    }
    
    return true;
}

// 使用
$file = $_FILES['file'];
$maxSize = 5 * 1024 * 1024;  // 5MB

if (!validateFileSize($file['size'], $maxSize)) {
    die('文件大小超过限制（最大 5MB）');
}
```

### 大小单位转换

**示例**：
```php
<?php
declare(strict_types=1);

function parseSize(string $size): int
{
    $unit = strtolower(substr($size, -1));
    $value = (int) $size;
    
    return match ($unit) {
        'g' => $value * 1024 * 1024 * 1024,
        'm' => $value * 1024 * 1024,
        'k' => $value * 1024,
        default => $value,
    };
}

// 使用
$maxSize = parseSize('10M');  // 10MB
```

## 文件内容验证

### 图片文件验证

**使用 getimagesize()**：
```php
<?php
declare(strict_types=1);

function validateImageFile(string $filepath): bool
{
    // 检查文件是否存在
    if (!file_exists($filepath)) {
        return false;
    }
    
    // 使用 getimagesize() 验证图片
    $imageInfo = @getimagesize($filepath);
    if ($imageInfo === false) {
        return false;  // 不是有效的图片文件
    }
    
    // 验证 MIME 类型
    $allowedMimeTypes = ['image/jpeg', 'image/png', 'image/gif'];
    if (!in_array($imageInfo['mime'], $allowedMimeTypes, true)) {
        return false;
    }
    
    return true;
}

// 使用
$file = $_FILES['file'];
if (!validateImageFile($file['tmp_name'])) {
    die('不是有效的图片文件');
}
```

### 文档文件验证

**PDF 验证示例**：
```php
<?php
declare(strict_types=1);

function validatePdfFile(string $filepath): bool
{
    $handle = fopen($filepath, 'rb');
    if ($handle === false) {
        return false;
    }
    
    $header = fread($handle, 4);
    fclose($handle);
    
    // PDF 文件头：%PDF
    return $header === '%PDF';
}

// 使用
$file = $_FILES['file'];
if (!validatePdfFile($file['tmp_name'])) {
    die('不是有效的 PDF 文件');
}
```

## 安全文件名生成

### 随机文件名

**示例**：
```php
<?php
declare(strict_types=1);

function generateSafeFileName(string $originalName): string
{
    // 获取扩展名
    $extension = pathinfo($originalName, PATHINFO_EXTENSION);
    
    // 清理扩展名（只允许字母和数字）
    $extension = preg_replace('/[^a-zA-Z0-9]/', '', $extension);
    
    // 生成唯一文件名
    $safeName = bin2hex(random_bytes(16)) . '.' . $extension;
    
    return $safeName;
}

// 使用
$file = $_FILES['file'];
$safeFileName = generateSafeFileName($file['name']);
$destination = __DIR__ . '/uploads/' . $safeFileName;
```

### 基于哈希的文件名

**示例**：
```php
<?php
declare(strict_types=1);

function generateHashFileName(string $originalName, string $filepath): string
{
    // 计算文件哈希
    $hash = hash_file('sha256', $filepath);
    
    // 获取扩展名
    $extension = pathinfo($originalName, PATHINFO_EXTENSION);
    $extension = preg_replace('/[^a-zA-Z0-9]/', '', $extension);
    
    // 使用哈希的前 32 个字符作为文件名
    return substr($hash, 0, 32) . '.' . $extension;
}
```

### 路径安全

**示例**：
```php
<?php
declare(strict_types=1);

function getSafePath(string $baseDir, string $filename): string
{
    // 确保基础目录是绝对路径
    $baseDir = realpath($baseDir);
    if ($baseDir === false) {
        throw new RuntimeException('Invalid base directory');
    }
    
    // 清理文件名（移除路径分隔符）
    $filename = basename($filename);
    $filename = preg_replace('/[^a-zA-Z0-9._-]/', '', $filename);
    
    // 构建完整路径
    $fullPath = $baseDir . DIRECTORY_SEPARATOR . $filename;
    
    // 确保路径在基础目录内（防止路径遍历）
    $realPath = realpath(dirname($fullPath));
    if ($realPath !== $baseDir) {
        throw new RuntimeException('Path traversal detected');
    }
    
    return $fullPath;
}
```

## 恶意文件防护

### 可执行文件检测

**示例**：
```php
<?php
declare(strict_types=1);

function isExecutableFile(string $filepath): bool
{
    $dangerousExtensions = ['php', 'php3', 'php4', 'php5', 'phtml', 'jsp', 'asp', 'aspx', 'sh', 'bat', 'exe'];
    $extension = strtolower(pathinfo($filepath, PATHINFO_EXTENSION));
    
    return in_array($extension, $dangerousExtensions, true);
}

// 使用
$file = $_FILES['file'];
if (isExecutableFile($file['name'])) {
    die('不允许上传可执行文件');
}
```

### 脚本文件检测

**示例**：
```php
<?php
declare(strict_types=1);

function containsScriptContent(string $filepath): bool
{
    $handle = fopen($filepath, 'r');
    if ($handle === false) {
        return false;
    }
    
    // 读取前 1KB 内容
    $content = fread($handle, 1024);
    fclose($handle);
    
    if ($content === false) {
        return false;
    }
    
    // 检查是否包含脚本标签
    $dangerousPatterns = [
        '/<\?php/i',
        '/<script/i',
        '/<iframe/i',
        '/eval\s*\(/i',
        '/exec\s*\(/i',
    ];
    
    foreach ($dangerousPatterns as $pattern) {
        if (preg_match($pattern, $content)) {
            return true;
        }
    }
    
    return false;
}

// 使用
$file = $_FILES['file'];
if (containsScriptContent($file['tmp_name'])) {
    die('文件包含恶意内容');
}
```

### 文件内容扫描

**示例**：
```php
<?php
declare(strict_types=1);

class FileContentScanner
{
    private const DANGEROUS_PATTERNS = [
        '/<\?php/i',
        '/<script/i',
        '/eval\s*\(/i',
        '/exec\s*\(/i',
        '/system\s*\(/i',
        '/shell_exec\s*\(/i',
    ];

    public static function scan(string $filepath): bool
    {
        $handle = fopen($filepath, 'r');
        if ($handle === false) {
            return false;
        }
        
        // 读取文件内容（限制大小）
        $maxSize = 1024 * 1024;  // 1MB
        $content = fread($handle, $maxSize);
        fclose($handle);
        
        if ($content === false) {
            return false;
        }
        
        // 检查危险模式
        foreach (self::DANGEROUS_PATTERNS as $pattern) {
            if (preg_match($pattern, $content)) {
                return true;  // 发现危险内容
            }
        }
        
        return false;  // 安全
    }
}
```

## 使用场景

### 所有文件上传功能

- 用户头像上传
- 文档上传
- 图片上传
- 媒体文件上传

### 安全敏感应用

- 企业文件管理系统
- 内容管理系统
- 文件共享平台

### 用户生成内容

- 用户上传的图片
- 用户上传的文档
- 用户上传的媒体文件

## 注意事项

### 多重验证

- **扩展名验证**：快速过滤
- **MIME 类型验证**：服务器端验证
- **文件内容验证**：最可靠的验证
- **文件头验证**：验证文件实际类型

### 不要信任客户端信息

- **文件名**：可能包含路径遍历字符
- **MIME 类型**：可能被伪造
- **文件大小**：可能不准确
- **所有信息**：都需要服务器端验证

### 文件内容验证

- **验证实际内容**：而不仅是扩展名
- **使用文件头**：验证文件类型
- **扫描恶意内容**：检测脚本和恶意代码

### 存储路径安全

- **使用绝对路径**：避免相对路径
- **限制存储目录**：文件只能存储在指定目录
- **防止路径遍历**：验证文件名不包含 `../`

## 常见问题

### 如何验证文件类型？

使用多重验证：
1. 扩展名验证（快速过滤）
2. MIME 类型验证（服务器端）
3. 文件头验证（最可靠）

### 如何防止恶意文件上传？

1. **白名单机制**：只允许安全的文件类型
2. **内容验证**：验证文件实际内容
3. **脚本检测**：检测脚本和恶意代码
4. **安全存储**：生成安全文件名和路径

### 如何生成安全文件名？

1. **随机文件名**：使用 `random_bytes()` 生成
2. **哈希文件名**：使用文件哈希
3. **清理扩展名**：只允许安全字符
4. **路径验证**：确保路径安全

### 如何验证文件内容？

1. **文件头验证**：检查文件开头的 Magic Bytes
2. **图片验证**：使用 `getimagesize()`
3. **内容扫描**：扫描文件内容中的危险模式

## 最佳实践

### 使用白名单验证

- 只允许明确安全的文件类型
- 不要使用黑名单（容易被绕过）
- 严格限制允许的类型

### 验证文件内容而不仅是扩展名

- 使用文件头验证
- 使用 `getimagesize()` 验证图片
- 扫描文件内容

### 生成随机文件名

- 使用 `random_bytes()` 或 `uniqid()`
- 不要使用用户提供的文件名
- 清理扩展名

### 隔离上传文件存储

- 将上传文件存储在 Web 根目录外
- 设置正确的文件权限
- 限制直接访问

## 相关章节

- **[5.6.1 文件上传基础](section-01-upload-basics.md)**：了解文件上传的基础内容
- **[5.6.3 文件存储策略](section-03-storage-strategy.md)**：了解文件存储的详细内容
- **[5.3.3 $_FILES 与安全处理](../chapter-03-superglobals/section-03-files-security.md)**：了解 $_FILES 的基础使用

## 练习任务

1. **实现文件类型验证器**
   - 扩展名验证
   - MIME 类型验证
   - 文件头验证

2. **实现文件内容验证器**
   - 图片文件验证
   - 文档文件验证
   - 恶意内容检测

3. **实现安全文件名生成器**
   - 随机文件名生成
   - 路径安全验证
   - 扩展名清理

4. **实现完整的文件验证类**
   - 综合所有验证方法
   - 提供清晰的错误信息
   - 记录验证日志

5. **实现恶意文件防护**
   - 可执行文件检测
   - 脚本内容检测
   - 文件内容扫描
