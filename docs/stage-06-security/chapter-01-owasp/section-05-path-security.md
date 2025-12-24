# 6.1.5 路径安全

## 概述

路径安全是 Web 应用安全的重要组成部分，涉及防止路径遍历攻击、验证文件路径、限制文件访问范围等。理解路径安全的概念、常见攻击方式以及防护措施，对于构建安全的 PHP 应用至关重要。

## 路径遍历攻击

### 什么是路径遍历攻击

**路径遍历攻击**（Path Traversal Attack，也称为目录遍历攻击）是一种安全漏洞，攻击者通过在文件路径中插入特殊字符（如 `../`、`..\\`），试图访问应用程序预期范围之外的文件或目录。

### 攻击示例

**示例 1：基本路径遍历**

```php
<?php
// 不安全的代码
$filename = $_GET['file'];
$filepath = '/var/www/uploads/' . $filename;

// 如果 $filename = '../../etc/passwd'
// 则 $filepath = '/var/www/uploads/../../etc/passwd'
// 实际访问：/etc/passwd（系统密码文件）
readfile($filepath);
```

**示例 2：编码绕过**

```php
<?php
// 攻击者可能使用编码绕过简单的检查
$filename = $_GET['file'];

// URL 编码：%2e%2e%2f%2e%2e%2fetc%2fpasswd
// 解码后：../../etc/passwd

// Unicode 编码：%c0%ae%c0%ae%c0%af
// 也可能被某些系统解析为 ../
```

**示例 3：空字节注入**

```php
<?php
// PHP < 5.3.4 的漏洞
$filename = $_GET['file'];
$filepath = '/var/www/uploads/' . $filename . '.txt';

// 如果 $filename = '../../etc/passwd%00'
// 则 $filepath = '/var/www/uploads/../../etc/passwd%00.txt'
// 在某些旧版本 PHP 中，%00 会被截断，实际访问 /etc/passwd
```

## 防护措施

### 1. 使用 realpath() 规范化路径

**`realpath()`** 可以解析路径中的符号链接和相对路径，返回绝对路径。

**语法**：`realpath(string $path): string|false`

**示例**：

```php
<?php
declare(strict_types=1);

function safeReadFile(string $filename, string $baseDir): ?string
{
    // 规范化基础目录
    $baseDir = realpath($baseDir);
    if ($baseDir === false) {
        return null;
    }
    
    // 规范化文件路径
    $filepath = realpath($baseDir . '/' . $filename);
    if ($filepath === false) {
        return null;
    }
    
    // 检查文件是否在基础目录内
    if (strpos($filepath, $baseDir) !== 0) {
        return null;  // 路径遍历攻击
    }
    
    // 安全读取文件
    return file_get_contents($filepath);
}

// 使用
$content = safeReadFile('user-file.txt', '/var/www/uploads');
```

### 2. 使用 basename() 获取文件名

**`basename()`** 返回路径中的文件名部分，自动去除目录部分。

**语法**：`basename(string $path, string $suffix = ""): string`

**示例**：

```php
<?php
declare(strict_types=1);

function safeGetFilename(string $input): string
{
    // 获取文件名，自动去除路径
    $filename = basename($input);
    
    // 进一步验证文件名
    if (preg_match('/[^a-zA-Z0-9._-]/', $filename)) {
        throw new InvalidArgumentException('Invalid filename');
    }
    
    return $filename;
}

// 使用
$filename = safeGetFilename($_GET['file']);
$filepath = __DIR__ . '/uploads/' . $filename;
```

### 3. 白名单验证

**白名单验证**只允许访问预定义的文件列表。

**示例**：

```php
<?php
declare(strict_types=1);

class SecureFileAccess
{
    private array $allowedFiles;
    private string $baseDir;
    
    public function __construct(string $baseDir, array $allowedFiles)
    {
        $this->baseDir = realpath($baseDir);
        if ($this->baseDir === false) {
            throw new InvalidArgumentException('Invalid base directory');
        }
        $this->allowedFiles = $allowedFiles;
    }
    
    public function readFile(string $filename): ?string
    {
        // 白名单验证
        if (!in_array($filename, $this->allowedFiles, true)) {
            return null;
        }
        
        // 使用 basename() 进一步确保安全
        $safeFilename = basename($filename);
        
        $filepath = $this->baseDir . '/' . $safeFilename;
        
        // 再次验证路径
        $realpath = realpath($filepath);
        if ($realpath === false || strpos($realpath, $this->baseDir) !== 0) {
            return null;
        }
        
        return file_get_contents($realpath);
    }
}

// 使用
$access = new SecureFileAccess('/var/www/uploads', [
    'file1.txt',
    'file2.txt',
    'document.pdf'
]);

$content = $access->readFile($_GET['file']);
```

### 4. 文件名验证

**严格验证文件名**，只允许安全的字符。

**示例**：

```php
<?php
declare(strict_types=1);

function validateFilename(string $filename): bool
{
    // 检查是否包含路径分隔符
    if (strpos($filename, '/') !== false || strpos($filename, '\\') !== false) {
        return false;
    }
    
    // 检查是否包含空字节
    if (strpos($filename, "\0") !== false) {
        return false;
    }
    
    // 检查是否包含危险字符
    $dangerous = ['..', '<', '>', ':', '"', '|', '?', '*'];
    foreach ($dangerous as $char) {
        if (strpos($filename, $char) !== false) {
            return false;
        }
    }
    
    // 只允许字母、数字、点、下划线、连字符
    if (!preg_match('/^[a-zA-Z0-9._-]+$/', $filename)) {
        return false;
    }
    
    return true;
}

// 使用
$filename = $_GET['file'];
if (!validateFilename($filename)) {
    throw new InvalidArgumentException('Invalid filename');
}
```

### 5. 限制目录访问

**将文件操作限制在特定目录内**，使用绝对路径。

**示例**：

```php
<?php
declare(strict_types=1);

class RestrictedFileAccess
{
    private string $allowedDir;
    
    public function __construct(string $allowedDir)
    {
        $this->allowedDir = realpath($allowedDir);
        if ($this->allowedDir === false) {
            throw new InvalidArgumentException('Invalid directory');
        }
    }
    
    public function getFile(string $relativePath): ?string
    {
        // 规范化相对路径
        $relativePath = ltrim($relativePath, '/\\');
        
        // 构建完整路径
        $fullPath = $this->allowedDir . '/' . $relativePath;
        
        // 规范化路径
        $realPath = realpath($fullPath);
        if ($realPath === false) {
            return null;
        }
        
        // 确保文件在允许的目录内
        if (strpos($realPath, $this->allowedDir) !== 0) {
            return null;  // 路径遍历攻击
        }
        
        // 确保是文件而不是目录
        if (!is_file($realPath)) {
            return null;
        }
        
        return $realPath;
    }
    
    public function readFile(string $relativePath): ?string
    {
        $filepath = $this->getFile($relativePath);
        if ($filepath === null) {
            return null;
        }
        
        return file_get_contents($filepath);
    }
}

// 使用
$access = new RestrictedFileAccess('/var/www/uploads');
$content = $access->readFile($_GET['file']);
```

## 安全最佳实践

### 1. 输入验证

**始终验证和清理用户输入**：

```php
<?php
declare(strict_types=1);

function sanitizeFilename(string $input): string
{
    // 移除所有路径分隔符
    $filename = basename($input);
    
    // 移除危险字符
    $filename = preg_replace('/[^a-zA-Z0-9._-]/', '', $filename);
    
    // 限制长度
    if (strlen($filename) > 255) {
        $filename = substr($filename, 0, 255);
    }
    
    return $filename;
}
```

### 2. 使用绝对路径

**使用绝对路径而非相对路径**：

```php
<?php
declare(strict_types=1);

// ❌ 不安全：使用相对路径
$filepath = './uploads/' . $filename;

// ✅ 安全：使用绝对路径
$baseDir = realpath(__DIR__ . '/uploads');
$filepath = $baseDir . '/' . basename($filename);
```

### 3. 路径规范化

**始终规范化路径**：

```php
<?php
declare(strict_types=1);

function getSafePath(string $baseDir, string $filename): ?string
{
    // 规范化基础目录
    $baseDir = realpath($baseDir);
    if ($baseDir === false) {
        return null;
    }
    
    // 获取安全的文件名
    $safeFilename = basename($filename);
    
    // 构建完整路径
    $fullPath = $baseDir . '/' . $safeFilename;
    
    // 规范化并验证
    $realPath = realpath($fullPath);
    if ($realPath === false) {
        return null;
    }
    
    // 确保在基础目录内
    if (strpos($realPath, $baseDir) !== 0) {
        return null;
    }
    
    return $realPath;
}
```

### 4. 记录和监控

**记录可疑的路径访问尝试**：

```php
<?php
declare(strict_types=1);

function logSuspiciousAccess(string $filename, string $ip): void
{
    $suspicious = ['..', '../', '..\\', '/etc/', '/var/', 'C:\\'];
    
    foreach ($suspicious as $pattern) {
        if (strpos($filename, $pattern) !== false) {
            error_log(sprintf(
                'Suspicious file access attempt: %s from %s',
                $filename,
                $ip
            ));
            break;
        }
    }
}

// 使用
logSuspiciousAccess($_GET['file'], $_SERVER['REMOTE_ADDR']);
```

## 完整示例

```php
<?php
declare(strict_types=1);

class SecureFileHandler
{
    private string $baseDir;
    private array $allowedExtensions;
    
    public function __construct(string $baseDir, array $allowedExtensions = [])
    {
        $this->baseDir = realpath($baseDir);
        if ($this->baseDir === false) {
            throw new InvalidArgumentException('Invalid base directory');
        }
        $this->allowedExtensions = $allowedExtensions;
    }
    
    public function validateFilename(string $filename): bool
    {
        // 使用 basename() 去除路径
        $filename = basename($filename);
        
        // 检查路径遍历
        if (strpos($filename, '..') !== false) {
            return false;
        }
        
        // 检查空字节
        if (strpos($filename, "\0") !== false) {
            return false;
        }
        
        // 检查扩展名
        if (!empty($this->allowedExtensions)) {
            $ext = strtolower(pathinfo($filename, PATHINFO_EXTENSION));
            if (!in_array($ext, $this->allowedExtensions, true)) {
                return false;
            }
        }
        
        return true;
    }
    
    public function getSafePath(string $filename): ?string
    {
        if (!$this->validateFilename($filename)) {
            return null;
        }
        
        $safeFilename = basename($filename);
        $fullPath = $this->baseDir . '/' . $safeFilename;
        
        $realPath = realpath($fullPath);
        if ($realPath === false) {
            return null;
        }
        
        // 确保在基础目录内
        if (strpos($realPath, $this->baseDir) !== 0) {
            return null;
        }
        
        return $realPath;
    }
    
    public function readFile(string $filename): ?string
    {
        $filepath = $this->getSafePath($filename);
        if ($filepath === null) {
            return null;
        }
        
        if (!is_file($filepath)) {
            return null;
        }
        
        return file_get_contents($filepath);
    }
}

// 使用
$handler = new SecureFileHandler('/var/www/uploads', ['txt', 'pdf', 'jpg']);
$content = $handler->readFile($_GET['file']);
```

## 注意事项

1. **永远不要信任用户输入**：始终验证和清理所有用户提供的路径
2. **使用白名单**：尽可能使用白名单而非黑名单
3. **规范化路径**：始终使用 `realpath()` 规范化路径
4. **限制目录**：将文件操作限制在特定目录内
5. **记录日志**：记录可疑的访问尝试，便于安全审计
6. **定期审查**：定期审查文件访问代码，确保没有安全漏洞

## 练习

1. 编写一个函数，安全地验证和处理文件路径
2. 实现一个安全的文件访问类，防止路径遍历攻击
3. 创建一个函数，记录可疑的文件访问尝试
4. 编写一个函数，验证文件扩展名是否在允许列表中
