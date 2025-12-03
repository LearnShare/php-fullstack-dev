# 2.12.3 $_FILES 与安全处理

## 概述

`$_FILES` 用于处理通过 HTTP POST 上传的文件。文件上传是 Web 应用中最容易受到攻击的功能之一，必须进行严格的安全验证。

## $_FILES 结构

### 基本结构

当上传文件时，`$_FILES` 数组包含以下信息：

```php
$_FILES['field_name'] = [
    'name' => 'original_filename.jpg',     // 原始文件名
    'type' => 'image/jpeg',               // MIME 类型
    'tmp_name' => '/tmp/phpXXXXXX',       // 临时文件路径
    'error' => UPLOAD_ERR_OK,             // 错误代码
    'size' => 102400                      // 文件大小（字节）
];
```

### 多文件上传

```php
<?php
declare(strict_types=1);

// HTML: <input type="file" name="files[]" multiple>
foreach ($_FILES['files']['name'] as $key => $name) {
    $file = [
        'name' => $_FILES['files']['name'][$key],
        'type' => $_FILES['files']['type'][$key],
        'tmp_name' => $_FILES['files']['tmp_name'][$key],
        'error' => $_FILES['files']['error'][$key],
        'size' => $_FILES['files']['size'][$key]
    ];
    // 处理每个文件
}
```

## 上传错误代码

| 常量                  | 值   | 说明                     |
| :-------------------- | :--- | :----------------------- |
| `UPLOAD_ERR_OK`       | 0    | 上传成功                 |
| `UPLOAD_ERR_INI_SIZE` | 1    | 超过 `upload_max_filesize` |
| `UPLOAD_ERR_FORM_SIZE` | 2  | 超过表单 `MAX_FILE_SIZE` |
| `UPLOAD_ERR_PARTIAL`  | 3    | 文件只有部分上传         |
| `UPLOAD_ERR_NO_FILE`  | 4    | 没有文件被上传           |
| `UPLOAD_ERR_NO_TMP_DIR` | 6 | 找不到临时文件夹       |
| `UPLOAD_ERR_CANT_WRITE` | 7 | 文件写入失败           |
| `UPLOAD_ERR_EXTENSION` | 8  | PHP 扩展阻止上传       |

### 检查错误

```php
<?php
declare(strict_types=1);

if ($_FILES['file']['error'] !== UPLOAD_ERR_OK) {
    switch ($_FILES['file']['error']) {
        case UPLOAD_ERR_INI_SIZE:
        case UPLOAD_ERR_FORM_SIZE:
            throw new RuntimeException('File too large');
        case UPLOAD_ERR_PARTIAL:
            throw new RuntimeException('File partially uploaded');
        case UPLOAD_ERR_NO_FILE:
            throw new RuntimeException('No file uploaded');
        default:
            throw new RuntimeException('Upload error');
    }
}
```

## 文件验证

### 验证文件类型

```php
<?php
declare(strict_types=1);

function validateFileType(array $file, array $allowedTypes): bool
{
    $finfo = finfo_open(FILEINFO_MIME_TYPE);
    $mimeType = finfo_file($finfo, $file['tmp_name']);
    finfo_close($finfo);
    
    return in_array($mimeType, $allowedTypes);
}

$allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
if (!validateFileType($_FILES['file'], $allowedTypes)) {
    throw new RuntimeException('Invalid file type');
}
```

### 验证文件扩展名

```php
<?php
declare(strict_types=1);

function validateExtension(string $filename, array $allowedExtensions): bool
{
    $extension = strtolower(pathinfo($filename, PATHINFO_EXTENSION));
    return in_array($extension, $allowedExtensions);
}

$allowedExtensions = ['jpg', 'jpeg', 'png', 'gif'];
if (!validateExtension($_FILES['file']['name'], $allowedExtensions)) {
    throw new RuntimeException('Invalid file extension');
}
```

### 验证文件大小

```php
<?php
declare(strict_types=1);

const MAX_FILE_SIZE = 5 * 1024 * 1024;  // 5MB

if ($_FILES['file']['size'] > MAX_FILE_SIZE) {
    throw new RuntimeException('File too large');
}
```

### 验证文件内容

```php
<?php
declare(strict_types=1);

function validateImageContent(string $tmpName): bool
{
    $imageInfo = @getimagesize($tmpName);
    return $imageInfo !== false;
}

if (!validateImageContent($_FILES['file']['tmp_name'])) {
    throw new RuntimeException('Invalid image file');
}
```

## move_uploaded_file() - 移动上传文件

### 语法

**语法**：`move_uploaded_file(string $from, string $to): bool`

**参数**：
- `$from`：临时文件路径
- `$to`：目标路径

**返回值**：成功返回 `true`，失败返回 `false`。

### 基本用法

```php
<?php
declare(strict_types=1);

$uploadDir = __DIR__ . '/uploads/';
$filename = uniqid() . '_' . basename($_FILES['file']['name']);
$targetPath = $uploadDir . $filename;

if (move_uploaded_file($_FILES['file']['tmp_name'], $targetPath)) {
    echo "File uploaded successfully: {$filename}\n";
} else {
    throw new RuntimeException('Failed to move uploaded file');
}
```

### 安全路径处理

```php
<?php
declare(strict_types=1);

function safeUpload(array $file, string $uploadDir): string
{
    // 验证上传目录
    if (!is_dir($uploadDir) || !is_writable($uploadDir)) {
        throw new RuntimeException('Upload directory not writable');
    }
    
    // 生成安全的文件名
    $extension = pathinfo($file['name'], PATHINFO_EXTENSION);
    $filename = bin2hex(random_bytes(16)) . '.' . $extension;
    $targetPath = $uploadDir . $filename;
    
    // 确保路径在允许的目录内
    $realUploadDir = realpath($uploadDir);
    $realTargetPath = realpath(dirname($targetPath));
    
    if ($realTargetPath !== $realUploadDir) {
        throw new RuntimeException('Invalid upload path');
    }
    
    if (!move_uploaded_file($file['tmp_name'], $targetPath)) {
        throw new RuntimeException('Failed to move uploaded file');
    }
    
    return $filename;
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class FileUploadHandler
{
    private const MAX_FILE_SIZE = 5 * 1024 * 1024;  // 5MB
    private const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/gif'];
    private const ALLOWED_EXTENSIONS = ['jpg', 'jpeg', 'png', 'gif'];
    
    public static function validateUpload(array $file): void
    {
        // 检查错误
        if ($file['error'] !== UPLOAD_ERR_OK) {
            throw new RuntimeException('Upload error: ' . $file['error']);
        }
        
        // 检查文件大小
        if ($file['size'] > self::MAX_FILE_SIZE) {
            throw new RuntimeException('File too large');
        }
        
        // 验证 MIME 类型
        $finfo = finfo_open(FILEINFO_MIME_TYPE);
        $mimeType = finfo_file($finfo, $file['tmp_name']);
        finfo_close($finfo);
        
        if (!in_array($mimeType, self::ALLOWED_TYPES)) {
            throw new RuntimeException('Invalid file type');
        }
        
        // 验证扩展名
        $extension = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
        if (!in_array($extension, self::ALLOWED_EXTENSIONS)) {
            throw new RuntimeException('Invalid file extension');
        }
        
        // 验证图像内容
        if (!@getimagesize($file['tmp_name'])) {
            throw new RuntimeException('Invalid image file');
        }
    }
    
    public static function upload(array $file, string $uploadDir): string
    {
        self::validateUpload($file);
        
        // 生成安全文件名
        $extension = pathinfo($file['name'], PATHINFO_EXTENSION);
        $filename = bin2hex(random_bytes(16)) . '.' . $extension;
        $targetPath = $uploadDir . $filename;
        
        // 移动文件
        if (!move_uploaded_file($file['tmp_name'], $targetPath)) {
            throw new RuntimeException('Failed to move uploaded file');
        }
        
        return $filename;
    }
}

// 使用示例
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['file'])) {
    try {
        $filename = FileUploadHandler::upload($_FILES['file'], __DIR__ . '/uploads/');
        echo "File uploaded: {$filename}\n";
    } catch (RuntimeException $e) {
        echo "Error: " . $e->getMessage() . "\n";
    }
}
```

## 安全建议

1. **验证文件类型**：使用 `finfo_file()` 验证 MIME 类型，不要仅依赖文件扩展名。

2. **限制文件大小**：设置合理的文件大小限制。

3. **安全文件名**：生成随机文件名，避免使用用户提供的文件名。

4. **存储位置**：将上传文件存储在 Web 根目录之外，或使用 `.htaccess` 限制访问。

5. **权限设置**：上传目录不应有执行权限。

6. **病毒扫描**：在生产环境中考虑集成病毒扫描。

7. **文件重命名**：始终重命名上传的文件，避免文件名冲突和路径遍历攻击。

## 注意事项

1. **临时文件**：上传的文件首先存储在临时目录，脚本结束后会自动删除。

2. **文件移动**：必须使用 `move_uploaded_file()` 移动文件，不能使用 `copy()` 或 `rename()`。

3. **路径遍历**：检查文件名，防止路径遍历攻击（如 `../../../etc/passwd`）。

4. **MIME 类型**：不要仅依赖 `$_FILES['type']`，客户端可能伪造。

5. **文件覆盖**：确保目标文件不会覆盖现有文件。

## 练习

1. 创建一个文件上传处理类，实现完整的验证和安全处理。

2. 编写一个函数，安全地处理多文件上传。

3. 实现一个函数，生成安全的文件名并防止冲突。

4. 创建一个函数，验证图像文件的尺寸和格式。

5. 编写一个函数，实现文件上传的进度跟踪。
