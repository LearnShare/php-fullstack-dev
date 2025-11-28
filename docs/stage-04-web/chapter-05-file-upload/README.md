# 4.5 文件上传处理

## 目标

- 理解 `multipart/form-data` 编码格式。
- 掌握 `$_FILES` 数据结构与文件上传处理流程。
- 识别常见安全风险：MIME 欺骗、文件名伪造、图片木马。
- 实现安全的文件上传策略：类型验证、大小限制、随机文件名。

## multipart/form-data

### 表单编码类型

- **`application/x-www-form-urlencoded`**：普通表单数据。
- **`multipart/form-data`**：包含文件上传的表单。

```html
<!-- 普通表单 -->
<form method="POST" action="/submit.php">
    <input type="text" name="name">
    <button type="submit">提交</button>
</form>

<!-- 文件上传表单 -->
<form method="POST" action="/upload.php" enctype="multipart/form-data">
    <input type="file" name="avatar">
    <input type="text" name="name">
    <button type="submit">上传</button>
</form>
```

### 请求格式

```
POST /upload.php HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="name"

Alice
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="avatar"; filename="photo.jpg"
Content-Type: image/jpeg

[文件二进制内容]
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

## $_FILES 数据结构

### 单文件上传

```php
<?php
// $_FILES 结构
$_FILES['avatar'] = [
    'name' => 'photo.jpg',           // 原始文件名
    'type' => 'image/jpeg',          // MIME 类型（客户端提供，不可信）
    'tmp_name' => '/tmp/phpXXXXXX',   // 临时文件路径
    'error' => UPLOAD_ERR_OK,        // 错误代码
    'size' => 123456,                // 文件大小（字节）
];
```

### 多文件上传

```php
<?php
// HTML: <input type="file" name="photos[]" multiple>

// 方式一：数组形式
$_FILES['photos'] = [
    'name' => ['photo1.jpg', 'photo2.jpg'],
    'type' => ['image/jpeg', 'image/png'],
    'tmp_name' => ['/tmp/phpXXX1', '/tmp/phpXXX2'],
    'error' => [UPLOAD_ERR_OK, UPLOAD_ERR_OK],
    'size' => [123456, 789012],
];

// 方式二：多个字段
$_FILES['avatar'] = [...];
$_FILES['document'] = [...];
```

### 规范化处理

```php
<?php
declare(strict_types=1);

function normalizeFiles(array $files): array
{
    $normalized = [];
    
    foreach ($files as $key => $file) {
        if (is_array($file['name'])) {
            // 多文件上传
            $count = count($file['name']);
            for ($i = 0; $i < $count; $i++) {
                $normalized["{$key}[{$i}]"] = [
                    'name' => $file['name'][$i],
                    'type' => $file['type'][$i],
                    'tmp_name' => $file['tmp_name'][$i],
                    'error' => $file['error'][$i],
                    'size' => $file['size'][$i],
                ];
            }
        } else {
            // 单文件上传
            $normalized[$key] = $file;
        }
    }
    
    return $normalized;
}

$files = normalizeFiles($_FILES);
```

## 上传错误处理

### 错误代码

| 常量                    | 值   | 说明                           |
| :---------------------- | :--- | :----------------------------- |
| `UPLOAD_ERR_OK`         | 0    | 上传成功                       |
| `UPLOAD_ERR_INI_SIZE`   | 1    | 超过 `upload_max_filesize`    |
| `UPLOAD_ERR_FORM_SIZE` | 2    | 超过表单 `MAX_FILE_SIZE`       |
| `UPLOAD_ERR_PARTIAL`    | 3    | 文件只有部分被上传             |
| `UPLOAD_ERR_NO_FILE`    | 4    | 没有文件被上传                 |
| `UPLOAD_ERR_NO_TMP_DIR` | 6   | 找不到临时文件夹               |
| `UPLOAD_ERR_CANT_WRITE` | 7   | 文件写入失败                   |
| `UPLOAD_ERR_EXTENSION`  | 8   | PHP 扩展阻止了文件上传         |

### 错误处理函数

```php
<?php
declare(strict_types=1);

function getUploadErrorMessage(int $error): string
{
    return match ($error) {
        UPLOAD_ERR_OK => '上传成功',
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

function validateUpload(array $file): array
{
    $errors = [];
    
    if ($file['error'] !== UPLOAD_ERR_OK) {
        $errors[] = getUploadErrorMessage($file['error']);
        return $errors;
    }
    
    if (!is_uploaded_file($file['tmp_name'])) {
        $errors[] = '文件不是通过 HTTP POST 上传的';
    }
    
    return $errors;
}
```

## 安全验证

### 1. MIME 类型验证

**问题**：客户端可以伪造 `Content-Type`。

```php
<?php
// 不安全的做法
if ($_FILES['avatar']['type'] === 'image/jpeg') {
    // 危险！客户端可以伪造 type
}

// 安全的做法：使用服务器端检测
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

### 2. 文件扩展名验证

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
```

### 3. 文件大小限制

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

### 4. 图片验证（防止图片木马）

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

### 5. 文件名安全处理

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

## 完整上传处理类

```php
<?php
declare(strict_types=1);

class FileUploader
{
    private array $allowedMimeTypes = [];
    private array $allowedExtensions = [];
    private int $maxFileSize = 5242880; // 5MB
    private string $uploadDir;

    public function __construct(string $uploadDir)
    {
        $this->uploadDir = rtrim($uploadDir, '/') . '/';
        
        if (!is_dir($this->uploadDir)) {
            mkdir($this->uploadDir, 0755, true);
        }
    }

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

    public function upload(array $file, ?string $customName = null): string
    {
        // 验证上传错误
        if ($file['error'] !== UPLOAD_ERR_OK) {
            throw new RuntimeException('上传失败: ' . $this->getErrorMessage($file['error']));
        }

        // 验证是否为上传文件
        if (!is_uploaded_file($file['tmp_name'])) {
            throw new RuntimeException('文件不是通过 HTTP POST 上传的');
        }

        // 验证文件大小
        if ($file['size'] > $this->maxFileSize) {
            throw new RuntimeException('文件大小超过限制');
        }

        // 获取真实 MIME 类型
        $realMimeType = mime_content_type($file['tmp_name']);
        
        // 验证 MIME 类型
        if (!empty($this->allowedMimeTypes) && !in_array($realMimeType, $this->allowedMimeTypes, true)) {
            throw new RuntimeException('不允许的文件类型');
        }

        // 获取扩展名
        $extension = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
        
        // 验证扩展名
        if (!empty($this->allowedExtensions) && !in_array($extension, $this->allowedExtensions, true)) {
            throw new RuntimeException('不允许的文件扩展名');
        }

        // 生成安全文件名
        $filename = $customName ?? $this->generateSafeFilename($extension);
        $filePath = $this->uploadDir . $filename;

        // 移动文件
        if (!move_uploaded_file($file['tmp_name'], $filePath)) {
            throw new RuntimeException('文件移动失败');
        }

        // 设置文件权限
        chmod($filePath, 0644);

        return $filename;
    }

    private function generateSafeFilename(string $extension): string
    {
        $randomBytes = random_bytes(16);
        $randomName = bin2hex($randomBytes);
        return "{$randomName}.{$extension}";
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

// 使用
$uploader = new FileUploader(__DIR__ . '/uploads');
$uploader->setAllowedTypes(
    ['image/jpeg', 'image/png', 'image/gif'],
    ['jpg', 'jpeg', 'png', 'gif']
);
$uploader->setMaxFileSize(5 * 1024 * 1024); // 5MB

try {
    $filename = $uploader->upload($_FILES['avatar']);
    echo "上传成功: {$filename}";
} catch (RuntimeException $e) {
    echo "上传失败: " . $e->getMessage();
}
```

## 安全最佳实践

### 1. 独立上传目录

```php
// 不推荐：上传到 Web 根目录
$uploadDir = __DIR__ . '/uploads/';

// 推荐：上传到 Web 根目录外
$uploadDir = __DIR__ . '/../storage/uploads/';
```

### 2. 禁用执行权限

```php
// 设置目录权限（禁止执行）
chmod($uploadDir, 0755);

// 设置文件权限（只读）
chmod($filePath, 0644);
```

### 3. 添加 .htaccess 保护

```apache
# .htaccess（Apache）
<FilesMatch "\.(php|php3|php4|php5|phtml|pl|py|jsp|asp|sh|cgi)$">
    Deny from all
</FilesMatch>
```

### 4. 文件名白名单

```php
// 只允许特定字符
$filename = preg_replace('/[^a-zA-Z0-9._-]/', '', $filename);
```

### 5. 定期清理临时文件

```php
function cleanupOldFiles(string $dir, int $maxAge = 3600): void
{
    $files = glob($dir . '*');
    $now = time();
    
    foreach ($files as $file) {
        if (is_file($file) && ($now - filemtime($file)) > $maxAge) {
            unlink($file);
        }
    }
}
```

## 练习

1. 创建一个文件上传处理类，包含完整的验证逻辑（类型、大小、MIME）。

2. 实现一个图片上传功能，验证图片有效性，生成缩略图，并保存原图和缩略图。

3. 编写一个多文件上传处理函数，支持批量上传和验证。

4. 创建一个安全的文件存储系统，使用随机文件名，并记录文件元数据（原始名称、上传时间等）。

5. 实现一个文件类型检测器，使用多种方法（MIME、扩展名、文件头）验证文件类型。

6. 设计一个文件上传 API，返回上传结果、文件 URL 和元数据，支持进度跟踪。
