# 4.5.3 文件存储策略

## 概述

文件存储策略直接影响安全性和性能。本节详细介绍存储目录设计、文件命名策略、权限设置、图片处理（缩略图生成）、文件清理，以及 CDN 集成。

## 存储目录设计

### 目录结构

```php
<?php
// 推荐结构
storage/
├── uploads/
│   ├── 2024/
│   │   ├── 01/
│   │   │   ├── 15/
│   │   │   │   ├── abc123def456.jpg
│   │   │   │   └── xyz789ghi012.png
│   │   │   └── ...
│   │   └── ...
│   └── ...
└── thumbnails/
    └── [相同结构]
```

### 目录创建

```php
<?php
declare(strict_types=1);

function createUploadDirectory(string $baseDir): string
{
    // 按日期创建目录
    $dateDir = date('Y/m/d');
    $fullPath = rtrim($baseDir, '/') . '/' . $dateDir;
    
    if (!is_dir($fullPath)) {
        mkdir($fullPath, 0755, true);
    }
    
    return $fullPath;
}
```

## 文件命名策略

### 随机文件名

```php
<?php
declare(strict_types=1);

function generateSafeFilename(string $extension): string
{
    // 使用日期 + 随机字符串
    $date = date('Ymd');
    $random = bin2hex(random_bytes(16));
    return "{$date}_{$random}.{$extension}";
}

// 或使用 UUID
function generateUuidFilename(string $extension): string
{
    $uuid = sprintf(
        '%04x%04x-%04x-%04x-%04x-%04x%04x%04x',
        mt_rand(0, 0xffff), mt_rand(0, 0xffff),
        mt_rand(0, 0xffff),
        mt_rand(0, 0x0fff) | 0x4000,
        mt_rand(0, 0x3fff) | 0x8000,
        mt_rand(0, 0xffff), mt_rand(0, 0xffff), mt_rand(0, 0xffff)
    );
    return "{$uuid}.{$extension}";
}
```

## 权限设置

### 目录和文件权限

```php
<?php
declare(strict_types=1);

// 设置目录权限（禁止执行）
chmod($uploadDir, 0755);

// 设置文件权限（只读）
chmod($filePath, 0644);
```

### .htaccess 保护

```apache
# .htaccess（Apache）
<FilesMatch "\.(php|php3|php4|php5|phtml|pl|py|jsp|asp|sh|cgi)$">
    Deny from all
</FilesMatch>
```

## 图片处理

### 缩略图生成

```php
<?php
declare(strict_types=1);

function generateThumbnail(string $sourcePath, string $destinationPath, int $maxWidth = 200, int $maxHeight = 200): bool
{
    $imageInfo = getimagesize($sourcePath);
    if ($imageInfo === false) {
        return false;
    }

    $width = $imageInfo[0];
    $height = $imageInfo[1];
    $type = $imageInfo[2];

    // 创建图片资源
    $source = match ($type) {
        IMAGETYPE_JPEG => imagecreatefromjpeg($sourcePath),
        IMAGETYPE_PNG => imagecreatefrompng($sourcePath),
        IMAGETYPE_GIF => imagecreatefromgif($sourcePath),
        default => null,
    };

    if ($source === null) {
        return false;
    }

    // 计算缩略图尺寸
    $ratio = min($maxWidth / $width, $maxHeight / $height);
    $thumbWidth = (int) ($width * $ratio);
    $thumbHeight = (int) ($height * $ratio);

    // 创建缩略图
    $thumb = imagecreatetruecolor($thumbWidth, $thumbHeight);
    imagecopyresampled($thumb, $source, 0, 0, 0, 0, $thumbWidth, $thumbHeight, $width, $height);

    // 保存
    $result = match ($type) {
        IMAGETYPE_JPEG => imagejpeg($thumb, $destinationPath, 85),
        IMAGETYPE_PNG => imagepng($thumb, $destinationPath),
        IMAGETYPE_GIF => imagegif($thumb, $destinationPath),
        default => false,
    };

    imagedestroy($source);
    imagedestroy($thumb);

    return $result !== false;
}
```

## 文件清理

### 定期清理

```php
<?php
declare(strict_types=1);

function cleanupOldFiles(string $dir, int $maxAge = 86400): void
{
    $files = glob($dir . '**/*', GLOB_BRACE);
    $now = time();
    
    foreach ($files as $file) {
        if (is_file($file) && ($now - filemtime($file)) > $maxAge) {
            unlink($file);
        }
    }
}
```

## 完整存储类

```php
<?php
declare(strict_types=1);

class FileStorage
{
    private string $baseDir;
    private string $baseUrl;

    public function __construct(string $baseDir, string $baseUrl = '/uploads')
    {
        $this->baseDir = rtrim($baseDir, '/');
        $this->baseUrl = rtrim($baseUrl, '/');
    }

    public function store(array $file, ?string $customName = null): array
    {
        // 生成文件名
        $extension = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
        $filename = $customName ?? $this->generateFilename($extension);
        
        // 创建目录
        $dateDir = date('Y/m/d');
        $fullDir = $this->baseDir . '/' . $dateDir;
        if (!is_dir($fullDir)) {
            mkdir($fullDir, 0755, true);
        }
        
        // 存储文件
        $filePath = $fullDir . '/' . $filename;
        if (!move_uploaded_file($file['tmp_name'], $filePath)) {
            throw new RuntimeException('文件存储失败');
        }
        
        chmod($filePath, 0644);
        
        return [
            'filename' => $filename,
            'path' => $filePath,
            'url' => "{$this->baseUrl}/{$dateDir}/{$filename}",
            'size' => $file['size'],
        ];
    }

    private function generateFilename(string $extension): string
    {
        $random = bin2hex(random_bytes(16));
        return "{$random}.{$extension}";
    }
}
```

## 注意事项

1. **存储位置**：文件存储在 Web 根目录外，或禁止执行
2. **权限设置**：目录 755，文件 644
3. **文件名**：使用随机文件名，防止猜测和覆盖
4. **目录结构**：按日期组织，便于管理和清理

## 练习

1. 实现一个安全的文件存储系统，使用日期目录和随机文件名。

2. 创建一个图片上传功能，支持缩略图生成。

3. 实现文件清理功能，定期删除过期文件。

4. 设计一个文件存储系统，支持 CDN 集成。
