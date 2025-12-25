# 4.5.2 文件验证与安全

## 概述

文件上传存在严重的安全风险。本节详细介绍 MIME 类型验证、文件扩展名验证、文件大小限制、图片验证（防止图片木马），以及文件名安全处理的最佳实践。

## 安全风险

### 常见攻击方式

1. **MIME 欺骗**：伪造 `Content-Type`，上传可执行文件
2. **文件名伪造**：使用 `../../../` 进行目录遍历
3. **图片木马**：在图片中嵌入恶意代码
4. **文件覆盖**：使用相同文件名覆盖重要文件

## MIME 类型验证

### 服务器端检测

```php
<?php
declare(strict_types=1);

// [不安全] 不安全的做法
if ($_FILES['avatar']['type'] === 'image/jpeg') {
    // 危险！客户端可以伪造 type
}

// [安全] 安全的做法：使用服务器端检测
function getRealMimeType(string $filePath): string
{
    $finfo = finfo_open(FILEINFO_MIME_TYPE);
    $mimeType = finfo_file($finfo, $filePath);
    finfo_close($finfo);
    
    return $mimeType;
}

// 或使用 mime_content_type（PHP 8.1+）
$mimeType = mime_content_type($file['tmp_name']);

// 验证允许的 MIME 类型
$allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
if (!in_array($mimeType, $allowedTypes, true)) {
    throw new InvalidArgumentException('不允许的文件类型');
}
```

## 文件扩展名验证

```php
<?php
declare(strict_types=1);

function getFileExtension(string $filename): string
{
    return strtolower(pathinfo($filename, PATHINFO_EXTENSION));
}

function isAllowedExtension(string $extension, array $allowed): bool
{
    return in_array(strtolower($extension), $allowed, true);
}

// 验证
$extension = getFileExtension($_FILES['avatar']['name']);
$allowedExtensions = ['jpg', 'jpeg', 'png', 'gif'];

if (!isAllowedExtension($extension, $allowedExtensions)) {
    throw new InvalidArgumentException('不允许的文件扩展名');
}

// 验证扩展名与 MIME 类型匹配
$expectedMime = match ($extension) {
    'jpg', 'jpeg' => 'image/jpeg',
    'png' => 'image/png',
    'gif' => 'image/gif',
    default => null,
};

if ($mimeType !== $expectedMime) {
    throw new InvalidArgumentException('文件类型与扩展名不匹配');
}
```

## 文件大小限制

```php
<?php
declare(strict_types=1);

function validateFileSize(array $file, int $maxSize = 5242880): void // 5MB
{
    if ($file['size'] > $maxSize) {
        throw new InvalidArgumentException('文件大小超过限制');
    }
    
    if ($file['size'] === 0) {
        throw new InvalidArgumentException('文件为空');
    }
}
```

## 图片验证（防止图片木马）

```php
<?php
declare(strict_types=1);

function isImageFile(string $filePath): bool
{
    $imageInfo = @getimagesize($filePath);
    return $imageInfo !== false;
}

function validateImage(array $file): void
{
    if (!isImageFile($file['tmp_name'])) {
        throw new InvalidArgumentException('不是有效的图片文件');
    }
    
    // 获取图片信息
    $imageInfo = getimagesize($file['tmp_name']);
    $mimeType = $imageInfo['mime'];
    
    // 验证 MIME 类型
    $allowedImageTypes = ['image/jpeg', 'image/png', 'image/gif'];
    if (!in_array($mimeType, $allowedImageTypes, true)) {
        throw new InvalidArgumentException('不允许的图片类型');
    }
}
```

## 文件名安全处理

```php
<?php
declare(strict_types=1);

function generateSafeFilename(string $originalName, string $extension): string
{
    // 生成随机文件名
    $randomBytes = random_bytes(16);
    $randomName = bin2hex($randomBytes);
    
    return "{$randomName}.{$extension}";
}

function sanitizeFilename(string $filename): string
{
    // 移除路径分隔符
    $filename = basename($filename);
    
    // 移除特殊字符
    $filename = preg_replace('/[^a-zA-Z0-9._-]/', '', $filename);
    
    // 限制长度
    $filename = substr($filename, 0, 255);
    
    return $filename;
}
```

## 完整验证类

```php
<?php
declare(strict_types=1);

class FileValidator
{
    private array $allowedMimeTypes = [];
    private array $allowedExtensions = [];
    private int $maxFileSize = 5242880; // 5MB

    public function setAllowedTypes(array $mimeTypes, array $extensions): self
    {
        $this->allowedMimeTypes = $mimeTypes;
        $this->allowedExtensions = $extensions;
        return $this;
    }

    public function setMaxFileSize(int $bytes): self
    {
        $this->maxFileSize = $bytes;
        return $this;
    }

    public function validate(array $file): array
    {
        $errors = [];

        // 检查错误
        if ($file['error'] !== UPLOAD_ERR_OK) {
            $errors[] = $this->getErrorMessage($file['error']);
            return ['valid' => false, 'errors' => $errors];
        }

        // 验证文件大小
        if ($file['size'] > $this->maxFileSize) {
            $errors[] = '文件大小不能超过 ' . ($this->maxFileSize / 1024 / 1024) . 'MB';
        }

        // 获取真实 MIME 类型
        $mimeType = mime_content_type($file['tmp_name']);
        if (!in_array($mimeType, $this->allowedMimeTypes, true)) {
            $errors[] = "不允许的文件类型: {$mimeType}";
        }

        // 验证扩展名
        $extension = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
        if (!in_array($extension, $this->allowedExtensions, true)) {
            $errors[] = "不允许的文件扩展名: {$extension}";
        }

        return [
            'valid' => empty($errors),
            'errors' => $errors,
            'mime_type' => $mimeType,
            'extension' => $extension,
        ];
    }

    private function getErrorMessage(int $error): string
    {
        return match ($error) {
            UPLOAD_ERR_INI_SIZE => '文件大小超过服务器限制',
            UPLOAD_ERR_FORM_SIZE => '文件大小超过表单限制',
            UPLOAD_ERR_PARTIAL => '文件只有部分被上传',
            UPLOAD_ERR_NO_FILE => '没有文件被上传',
            UPLOAD_ERR_NO_TMP_DIR => '找不到临时文件夹',
            UPLOAD_ERR_CANT_WRITE => '文件写入失败',
            UPLOAD_ERR_EXTENSION => 'PHP 扩展阻止了文件上传',
            default => '未知错误',
        };
    }
}
```

## 注意事项

1. **永远不信任客户端**：所有验证都在服务器端进行
2. **多重验证**：MIME 类型、扩展名、文件内容都要验证
3. **文件名安全**：使用随机文件名，防止目录遍历
4. **图片验证**：使用 `getimagesize()` 验证图片有效性

## 练习

1. 创建一个文件验证类，包含类型、大小、扩展名的完整验证。

2. 实现图片验证功能，防止图片木马攻击。

3. 创建一个文件类型检测工具，使用多种方法验证文件类型。

4. 实现文件名安全处理，生成随机文件名并防止目录遍历。
