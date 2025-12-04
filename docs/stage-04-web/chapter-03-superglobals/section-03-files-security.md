# 4.3.3 $_FILES 与安全处理

## 概述

文件上传是 Web 应用中的常见功能，但也存在安全风险。本节详细介绍文件上传的基础处理、文件验证、安全存储、文件类型检测、大小限制，以及安全最佳实践。

## $_FILES - 文件上传

### 基础结构

```php
<?php
// $_FILES 数组结构
$_FILES = [
    'file' => [
        'name' => 'photo.jpg',        // 原始文件名
        'type' => 'image/jpeg',       // MIME 类型
        'tmp_name' => '/tmp/phpXXXX', // 临时文件路径
        'error' => UPLOAD_ERR_OK,     // 错误码
        'size' => 102400,             // 文件大小（字节）
    ]
];
```

### 错误码

| 错误码 | 常量 | 说明 |
| :--- | :--- | :--- |
| 0 | `UPLOAD_ERR_OK` | 上传成功 |
| 1 | `UPLOAD_ERR_INI_SIZE` | 超过 `upload_max_filesize` |
| 2 | `UPLOAD_ERR_FORM_SIZE` | 超过表单 `MAX_FILE_SIZE` |
| 3 | `UPLOAD_ERR_PARTIAL` | 文件部分上传 |
| 4 | `UPLOAD_ERR_NO_FILE` | 没有文件上传 |
| 6 | `UPLOAD_ERR_NO_TMP_DIR` | 临时目录不存在 |
| 7 | `UPLOAD_ERR_CANT_WRITE` | 写入失败 |
| 8 | `UPLOAD_ERR_EXTENSION` | PHP 扩展阻止上传 |

### 基础处理

```php
<?php
declare(strict_types=1);

if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['file'])) {
    $file = $_FILES['file'];
    
    // 检查错误
    if ($file['error'] !== UPLOAD_ERR_OK) {
        echo "上传失败，错误码: {$file['error']}\n";
        exit;
    }
    
    // 移动文件
    $uploadDir = __DIR__ . '/uploads/';
    $filename = basename($file['name']);
    $destination = $uploadDir . $filename;
    
    if (move_uploaded_file($file['tmp_name'], $destination)) {
        echo "文件上传成功: {$filename}\n";
    } else {
        echo "文件移动失败\n";
    }
}
```

## 文件验证

### 类型验证

```php
<?php
declare(strict_types=1);

class FileValidator
{
    private const ALLOWED_TYPES = [
        'image/jpeg',
        'image/png',
        'image/gif',
        'application/pdf',
    ];

    private const ALLOWED_EXTENSIONS = ['jpg', 'jpeg', 'png', 'gif', 'pdf'];

    public function validate(array $file): array
    {
        $errors = [];

        // 检查错误
        if ($file['error'] !== UPLOAD_ERR_OK) {
            $errors[] = "上传失败，错误码: {$file['error']}";
            return ['valid' => false, 'errors' => $errors];
        }

        // 检查文件大小（2MB）
        if ($file['size'] > 2 * 1024 * 1024) {
            $errors[] = '文件大小不能超过 2MB';
        }

        // 检查 MIME 类型
        $finfo = finfo_open(FILEINFO_MIME_TYPE);
        $mimeType = finfo_file($finfo, $file['tmp_name']);
        finfo_close($finfo);

        if (!in_array($mimeType, self::ALLOWED_TYPES, true)) {
            $errors[] = "不允许的文件类型: {$mimeType}";
        }

        // 检查文件扩展名
        $extension = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
        if (!in_array($extension, self::ALLOWED_EXTENSIONS, true)) {
            $errors[] = "不允许的文件扩展名: {$extension}";
        }

        // 验证扩展名与 MIME 类型匹配
        $expectedMime = $this->getMimeFromExtension($extension);
        if ($mimeType !== $expectedMime) {
            $errors[] = '文件类型与扩展名不匹配';
        }

        return [
            'valid' => empty($errors),
            'errors' => $errors,
            'mime_type' => $mimeType,
            'extension' => $extension,
        ];
    }

    private function getMimeFromExtension(string $ext): ?string
    {
        $map = [
            'jpg' => 'image/jpeg',
            'jpeg' => 'image/jpeg',
            'png' => 'image/png',
            'gif' => 'image/gif',
            'pdf' => 'application/pdf',
        ];
        return $map[$ext] ?? null;
    }
}
```

### 文件名安全

```php
<?php
declare(strict_types=1);

function generateSafeFilename(string $originalName): string
{
    // 获取扩展名
    $extension = strtolower(pathinfo($originalName, PATHINFO_EXTENSION));
    
    // 生成唯一文件名
    $basename = bin2hex(random_bytes(16));
    
    return "{$basename}.{$extension}";
}

// 使用
$safeFilename = generateSafeFilename($_FILES['file']['name']);
```

## 安全存储

### 存储策略

```php
<?php
declare(strict_types=1);

class FileUploader
{
    private string $uploadDir;
    private int $maxSize;
    private array $allowedTypes;

    public function __construct(
        string $uploadDir,
        int $maxSize = 2 * 1024 * 1024,
        array $allowedTypes = []
    ) {
        $this->uploadDir = rtrim($uploadDir, '/') . '/';
        $this->maxSize = $maxSize;
        $this->allowedTypes = $allowedTypes;

        // 确保上传目录存在
        if (!is_dir($this->uploadDir)) {
            mkdir($this->uploadDir, 0755, true);
        }
    }

    public function upload(array $file): array
    {
        // 验证
        $validator = new FileValidator();
        $validation = $validator->validate($file);
        
        if (!$validation['valid']) {
            return ['success' => false, 'errors' => $validation['errors']];
        }

        // 生成安全文件名
        $filename = $this->generateSafeFilename($file['name'], $validation['extension']);
        $destination = $this->uploadDir . $filename;

        // 移动文件
        if (!move_uploaded_file($file['tmp_name'], $destination)) {
            return ['success' => false, 'errors' => ['文件移动失败']];
        }

        // 设置文件权限
        chmod($destination, 0644);

        return [
            'success' => true,
            'filename' => $filename,
            'path' => $destination,
            'size' => $file['size'],
            'mime_type' => $validation['mime_type'],
        ];
    }

    private function generateSafeFilename(string $originalName, string $extension): string
    {
        // 使用日期目录 + 随机文件名
        $dateDir = date('Y/m/d');
        $dir = $this->uploadDir . $dateDir . '/';
        
        if (!is_dir($dir)) {
            mkdir($dir, 0755, true);
        }

        $basename = bin2hex(random_bytes(16));
        return "{$dateDir}/{$basename}.{$extension}";
    }
}
```

### 防止目录遍历

```php
<?php
declare(strict_types=1);

function sanitizeFilename(string $filename): string
{
    // 移除路径分隔符
    $filename = basename($filename);
    
    // 移除特殊字符
    $filename = preg_replace('/[^a-zA-Z0-9._-]/', '', $filename);
    
    return $filename;
}
```

## 图片处理

### 图片验证

```php
<?php
declare(strict_types=1);

function validateImage(array $file): bool
{
    // 检查是否为图片
    $imageInfo = @getimagesize($file['tmp_name']);
    if ($imageInfo === false) {
        return false;
    }

    // 检查图片类型
    $allowedTypes = [IMAGETYPE_JPEG, IMAGETYPE_PNG, IMAGETYPE_GIF];
    if (!in_array($imageInfo[2], $allowedTypes, true)) {
        return false;
    }

    // 检查图片尺寸
    $maxWidth = 2000;
    $maxHeight = 2000;
    if ($imageInfo[0] > $maxWidth || $imageInfo[1] > $maxHeight) {
        return false;
    }

    return true;
}
```

### 图片处理

```php
<?php
declare(strict_types=1);

function processImage(string $sourcePath, string $destinationPath): bool
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

    // 调整大小（示例：缩略图）
    $thumbWidth = 200;
    $thumbHeight = 200;
    $thumb = imagecreatetruecolor($thumbWidth, $thumbHeight);
    imagecopyresampled($thumb, $source, 0, 0, 0, 0, $thumbWidth, $thumbHeight, $width, $height);

    // 保存
    $result = imagejpeg($thumb, $destinationPath, 85);
    imagedestroy($source);
    imagedestroy($thumb);

    return $result !== false;
}
```

## 安全最佳实践

### 1. 配置限制

```ini
; php.ini
upload_max_filesize = 2M
post_max_size = 8M
max_file_uploads = 20
```

### 2. 验证清单

- [ ] 检查文件大小
- [ ] 验证 MIME 类型（使用 `finfo`）
- [ ] 验证文件扩展名
- [ ] 验证扩展名与 MIME 类型匹配
- [ ] 检查文件内容（图片验证）
- [ ] 生成安全文件名
- [ ] 限制存储目录
- [ ] 设置文件权限

### 3. 存储目录安全

```php
<?php
// 上传目录应在 Web 根目录外，或使用 .htaccess 保护
// 推荐结构：
// /var/www/uploads/  (Web 根目录外)
// 或
// /var/www/html/uploads/  (使用 .htaccess 禁止执行 PHP)
```

```apache
# .htaccess
<FilesMatch "\.php$">
    Deny from all
</FilesMatch>
```

## 完整示例

```php
<?php
declare(strict_types=1);

class SecureFileUploader
{
    public function handleUpload(array $file): array
    {
        // 1. 基础验证
        if ($file['error'] !== UPLOAD_ERR_OK) {
            return ['success' => false, 'error' => '上传失败'];
        }

        // 2. 类型验证
        $validator = new FileValidator();
        $validation = $validator->validate($file);
        if (!$validation['valid']) {
            return ['success' => false, 'errors' => $validation['errors']];
        }

        // 3. 生成安全文件名
        $filename = $this->generateSafeFilename($validation['extension']);

        // 4. 移动文件
        $destination = $this->uploadDir . $filename;
        if (!move_uploaded_file($file['tmp_name'], $destination)) {
            return ['success' => false, 'error' => '文件移动失败'];
        }

        // 5. 设置权限
        chmod($destination, 0644);

        return [
            'success' => true,
            'filename' => $filename,
            'url' => "/uploads/{$filename}",
        ];
    }
}
```

## 注意事项

1. **永远不要信任文件名**：使用服务器端验证，不要依赖客户端提供的信息
2. **限制文件类型**：只允许必要的文件类型
3. **限制文件大小**：防止资源耗尽
4. **安全存储**：文件存储在 Web 根目录外，或禁止执行
5. **扫描恶意内容**：对上传的文件进行病毒扫描（可选）

## 练习

1. 实现一个完整的文件上传处理类，包含验证、存储、错误处理。

2. 创建一个图片上传功能，支持图片验证、缩略图生成。

3. 实现一个安全的文件存储系统，使用日期目录和随机文件名。

4. 创建一个文件类型检测工具，使用多种方法验证文件类型。

5. 实现一个文件上传进度监控功能（使用 AJAX）。
