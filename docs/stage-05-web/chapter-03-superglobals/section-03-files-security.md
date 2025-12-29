# 5.3.3 $_FILES 与安全处理

## 概述

`$_FILES` 超全局变量用于处理通过 HTTP POST 上传的文件。文件上传是 Web 应用中的常见功能，但也是安全风险较高的功能。理解 `$_FILES` 的结构、文件上传的处理流程、文件验证方法、安全注意事项，对于构建安全的文件上传功能至关重要。本节详细介绍 `$_FILES` 的使用方法、文件上传处理、文件验证、安全注意事项等内容，帮助零基础学员掌握安全的文件上传处理。

文件上传涉及多个安全风险：恶意文件上传、文件类型伪造、路径遍历攻击、文件覆盖等。正确处理文件上传需要严格验证文件类型、文件大小、文件内容，使用安全的文件名和存储路径，设置正确的文件权限。

**主要内容**：
- `$_FILES` 超全局变量的结构
- 文件上传的处理流程
- 文件信息获取（name、type、size、tmp_name、error）
- 文件验证方法（类型验证、大小验证、内容验证）
- 文件移动和存储
- 安全注意事项（文件名安全、路径安全、权限设置）
- 多文件上传处理
- 实际应用示例和最佳实践

## 特性

- **文件信息**：`$_FILES` 提供完整的文件上传信息
- **临时存储**：上传的文件先存储在临时目录
- **自动处理**：PHP 自动处理文件上传
- **安全风险**：需要严格验证和处理
- **多文件支持**：支持单文件和多文件上传

## $_FILES 超全局变量结构

### 基本结构

`$_FILES` 是一个关联数组，每个上传的文件对应一个数组元素。

**单文件上传结构**：
```php
$_FILES = [
    'file' => [
        'name' => 'example.jpg',        // 原始文件名
        'type' => 'image/jpeg',         // MIME 类型
        'size' => 12345,                // 文件大小（字节）
        'tmp_name' => '/tmp/phpXXXXXX', // 临时文件路径
        'error' => UPLOAD_ERR_OK,       // 错误代码
    ],
];
```

**多文件上传结构**：
```php
$_FILES = [
    'files' => [
        'name' => [
            0 => 'file1.jpg',
            1 => 'file2.png',
        ],
        'type' => [
            0 => 'image/jpeg',
            1 => 'image/png',
        ],
        'size' => [
            0 => 12345,
            1 => 67890,
        ],
        'tmp_name' => [
            0 => '/tmp/phpXXXXXX',
            1 => '/tmp/phpYYYYYY',
        ],
        'error' => [
            0 => UPLOAD_ERR_OK,
            1 => UPLOAD_ERR_OK,
        ],
    ],
];
```

### 字段说明

| 字段 | 说明 | 示例 |
|:-----|:-----|:-----|
| `name` | 原始文件名 | `example.jpg` |
| `type` | MIME 类型（由客户端提供，不可信） | `image/jpeg` |
| `size` | 文件大小（字节） | `12345` |
| `tmp_name` | 临时文件路径 | `/tmp/phpXXXXXX` |
| `error` | 错误代码 | `UPLOAD_ERR_OK` |

### 错误代码

| 常量 | 值 | 说明 |
|:-----|:---|:-----|
| `UPLOAD_ERR_OK` | 0 | 上传成功 |
| `UPLOAD_ERR_INI_SIZE` | 1 | 文件大小超过 `upload_max_filesize` |
| `UPLOAD_ERR_FORM_SIZE` | 2 | 文件大小超过表单 `MAX_FILE_SIZE` |
| `UPLOAD_ERR_PARTIAL` | 3 | 文件只有部分被上传 |
| `UPLOAD_ERR_NO_FILE` | 4 | 没有文件被上传 |
| `UPLOAD_ERR_NO_TMP_DIR` | 6 | 找不到临时文件夹 |
| `UPLOAD_ERR_CANT_WRITE` | 7 | 文件写入失败 |
| `UPLOAD_ERR_EXTENSION` | 8 | PHP 扩展阻止了文件上传 |

## 文件上传处理

### 基本处理流程

1. **检查文件是否上传**：使用 `isset()` 检查 `$_FILES` 键
2. **检查上传错误**：检查 `error` 字段
3. **验证文件**：验证文件类型、大小、内容
4. **生成安全文件名**：生成安全的文件名
5. **移动文件**：使用 `move_uploaded_file()` 移动文件
6. **设置权限**：设置正确的文件权限

### 基本示例

**示例 1：单文件上传**
```php
<?php
declare(strict_types=1);

if (isset($_FILES['file'])) {
    $file = $_FILES['file'];
    
    // 检查上传错误
    if ($file['error'] !== UPLOAD_ERR_OK) {
        die('文件上传失败');
    }
    
    // 验证文件
    // ... 验证逻辑
    
    // 生成安全文件名
    $extension = pathinfo($file['name'], PATHINFO_EXTENSION);
    $safeFileName = uniqid() . '.' . $extension;
    $destination = __DIR__ . '/uploads/' . $safeFileName;
    
    // 移动文件
    if (move_uploaded_file($file['tmp_name'], $destination)) {
        echo "文件上传成功：{$safeFileName}\n";
    } else {
        die('文件移动失败');
    }
}
```

**HTML 表单**：
```html
<form method="POST" enctype="multipart/form-data">
    <input type="file" name="file">
    <button type="submit">上传</button>
</form>
```

### move_uploaded_file() 函数

**语法**：`move_uploaded_file(string $from, string $to): bool`

**参数**：
- `$from`：临时文件路径（`$_FILES['file']['tmp_name']`）
- `$to`：目标文件路径

**返回值**：成功返回 `true`，失败返回 `false`。

**作用**：将上传的文件移动到目标位置。只有通过 HTTP POST 上传的文件才能使用此函数移动。

**示例**：
```php
<?php
declare(strict_types=1);

$tmpName = $_FILES['file']['tmp_name'];
$destination = __DIR__ . '/uploads/' . basename($_FILES['file']['name']);

if (move_uploaded_file($tmpName, $destination)) {
    echo "文件移动成功\n";
} else {
    echo "文件移动失败\n";
}
```

## 文件验证

### 1. 错误检查

首先检查上传错误。

**示例**：
```php
<?php
declare(strict_types=1);

function checkUploadError(int $error): void
{
    switch ($error) {
        case UPLOAD_ERR_OK:
            return;  // 无错误
        case UPLOAD_ERR_INI_SIZE:
        case UPLOAD_ERR_FORM_SIZE:
            throw new RuntimeException('文件大小超过限制');
        case UPLOAD_ERR_PARTIAL:
            throw new RuntimeException('文件只有部分被上传');
        case UPLOAD_ERR_NO_FILE:
            throw new RuntimeException('没有文件被上传');
        case UPLOAD_ERR_NO_TMP_DIR:
            throw new RuntimeException('找不到临时文件夹');
        case UPLOAD_ERR_CANT_WRITE:
            throw new RuntimeException('文件写入失败');
        case UPLOAD_ERR_EXTENSION:
            throw new RuntimeException('PHP 扩展阻止了文件上传');
        default:
            throw new RuntimeException('未知的上传错误');
    }
}

$file = $_FILES['file'];
checkUploadError($file['error']);
```

### 2. 文件大小验证

验证文件大小是否在允许范围内。

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

$file = $_FILES['file'];
$maxSize = 5 * 1024 * 1024;  // 5MB

if (!validateFileSize($file['size'], $maxSize)) {
    die('文件大小超过限制（最大 5MB）');
}
```

### 3. 文件类型验证

验证文件类型。注意：`$_FILES['file']['type']` 由客户端提供，不可信，需要服务器端验证。

#### 方法一：扩展名验证

**示例**：
```php
<?php
declare(strict_types=1);

function validateFileExtension(string $filename, array $allowedExtensions): bool
{
    $extension = strtolower(pathinfo($filename, PATHINFO_EXTENSION));
    return in_array($extension, $allowedExtensions, true);
}

$file = $_FILES['file'];
$allowedExtensions = ['jpg', 'jpeg', 'png', 'gif'];

if (!validateFileExtension($file['name'], $allowedExtensions)) {
    die('不允许的文件类型');
}
```

#### 方法二：MIME 类型验证

**示例**：
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

$file = $_FILES['file'];
$allowedMimeTypes = ['image/jpeg', 'image/png', 'image/gif'];

if (!validateMimeType($file['tmp_name'], $allowedMimeTypes)) {
    die('不允许的文件类型');
}
```

#### 方法三：内容验证（推荐）

**示例**（图片验证）：
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

$file = $_FILES['file'];

if (!validateImageFile($file['tmp_name'])) {
    die('不是有效的图片文件');
}
```

### 4. 文件名安全

生成安全的文件名，防止路径遍历攻击。

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

$file = $_FILES['file'];
$safeFileName = generateSafeFileName($file['name']);
$destination = __DIR__ . '/uploads/' . $safeFileName;
```

### 5. 完整验证示例

```php
<?php
declare(strict_types=1);

class FileUploadValidator
{
    private const MAX_FILE_SIZE = 5 * 1024 * 1024;  // 5MB
    private const ALLOWED_EXTENSIONS = ['jpg', 'jpeg', 'png', 'gif'];
    private const ALLOWED_MIME_TYPES = ['image/jpeg', 'image/png', 'image/gif'];

    public static function validate(array $file): array
    {
        // 检查上传错误
        if ($file['error'] !== UPLOAD_ERR_OK) {
            throw new RuntimeException('文件上传失败：错误代码 ' . $file['error']);
        }

        // 验证文件大小
        if ($file['size'] === 0) {
            throw new RuntimeException('文件为空');
        }

        if ($file['size'] > self::MAX_FILE_SIZE) {
            throw new RuntimeException('文件大小超过限制（最大 5MB）');
        }

        // 验证扩展名
        $extension = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
        if (!in_array($extension, self::ALLOWED_EXTENSIONS, true)) {
            throw new RuntimeException('不允许的文件类型');
        }

        // 验证 MIME 类型（服务器端）
        $mimeType = self::getMimeType($file['tmp_name']);
        if ($mimeType === false || !in_array($mimeType, self::ALLOWED_MIME_TYPES, true)) {
            throw new RuntimeException('不允许的文件类型');
        }

        // 验证图片内容（如果是图片）
        if (!self::validateImageFile($file['tmp_name'])) {
            throw new RuntimeException('不是有效的图片文件');
        }

        return [
            'valid' => true,
            'extension' => $extension,
            'mime_type' => $mimeType,
        ];
    }

    private static function getMimeType(string $filepath): string|false
    {
        $finfo = finfo_open(FILEINFO_MIME_TYPE);
        $mimeType = finfo_file($finfo, $filepath);
        finfo_close($finfo);
        return $mimeType;
    }

    private static function validateImageFile(string $filepath): bool
    {
        $imageInfo = @getimagesize($filepath);
        return $imageInfo !== false;
    }
}

// 使用
try {
    $file = $_FILES['file'];
    $validation = FileUploadValidator::validate($file);
    
    // 生成安全文件名
    $safeFileName = bin2hex(random_bytes(16)) . '.' . $validation['extension'];
    $destination = __DIR__ . '/uploads/' . $safeFileName;
    
    // 移动文件
    if (move_uploaded_file($file['tmp_name'], $destination)) {
        echo "文件上传成功：{$safeFileName}\n";
    }
} catch (RuntimeException $e) {
    echo "错误：{$e->getMessage()}\n";
}
```

## 文件存储

### 存储路径安全

**安全路径**：
```php
<?php
declare(strict_types=1);

// 安全：使用绝对路径，限制在指定目录
$uploadDir = __DIR__ . '/uploads/';
$safeFileName = bin2hex(random_bytes(16)) . '.' . $extension;
$destination = $uploadDir . $safeFileName;

// 确保目录存在
if (!is_dir($uploadDir)) {
    mkdir($uploadDir, 0755, true);
}
```

**不安全路径**（避免）：
```php
<?php
// 危险：使用用户提供的文件名，可能包含路径遍历
$destination = __DIR__ . '/uploads/' . $_FILES['file']['name'];
// 攻击者可能使用 ../../../etc/passwd 进行路径遍历攻击
```

### 文件权限设置

**设置文件权限**：
```php
<?php
declare(strict_types=1);

if (move_uploaded_file($tmpName, $destination)) {
    // 设置文件权限（所有者可读写，其他用户只读）
    chmod($destination, 0644);
    
    echo "文件上传成功\n";
}
```

### 存储策略

**按日期组织**：
```php
<?php
declare(strict_types=1);

$uploadDir = __DIR__ . '/uploads/' . date('Y/m/d') . '/';
if (!is_dir($uploadDir)) {
    mkdir($uploadDir, 0755, true);
}

$safeFileName = bin2hex(random_bytes(16)) . '.' . $extension;
$destination = $uploadDir . $safeFileName;
```

**按类型组织**：
```php
<?php
declare(strict_types=1);

$extension = pathinfo($file['name'], PATHINFO_EXTENSION);
$typeDir = match ($extension) {
    'jpg', 'jpeg', 'png', 'gif' => 'images/',
    'pdf', 'doc', 'docx' => 'documents/',
    default => 'others/',
};

$uploadDir = __DIR__ . '/uploads/' . $typeDir;
if (!is_dir($uploadDir)) {
    mkdir($uploadDir, 0755, true);
}
```

## 多文件上传处理

### HTML 表单

```html
<form method="POST" enctype="multipart/form-data">
    <input type="file" name="files[]" multiple>
    <button type="submit">上传</button>
</form>
```

### 处理多文件上传

**示例**：
```php
<?php
declare(strict_types=1);

if (isset($_FILES['files'])) {
    $files = $_FILES['files'];
    
    // 规范化数组结构
    $fileCount = count($files['name']);
    $uploadedFiles = [];
    
    for ($i = 0; $i < $fileCount; $i++) {
        $file = [
            'name' => $files['name'][$i],
            'type' => $files['type'][$i],
            'size' => $files['size'][$i],
            'tmp_name' => $files['tmp_name'][$i],
            'error' => $files['error'][$i],
        ];
        
        // 检查错误
        if ($file['error'] !== UPLOAD_ERR_OK) {
            continue;  // 跳过错误文件
        }
        
        // 验证文件
        try {
            $validation = FileUploadValidator::validate($file);
            
            // 生成安全文件名
            $safeFileName = bin2hex(random_bytes(16)) . '.' . $validation['extension'];
            $destination = __DIR__ . '/uploads/' . $safeFileName;
            
            // 移动文件
            if (move_uploaded_file($file['tmp_name'], $destination)) {
                $uploadedFiles[] = $safeFileName;
            }
        } catch (RuntimeException $e) {
            echo "文件 {$file['name']} 验证失败：{$e->getMessage()}\n";
        }
    }
    
    echo "成功上传 " . count($uploadedFiles) . " 个文件\n";
}
```

## 安全注意事项

### 文件名安全

- **不要使用用户提供的文件名**：可能包含路径遍历字符（`../`）
- **生成唯一文件名**：使用 `uniqid()` 或 `random_bytes()`
- **清理扩展名**：只允许安全的字符
- **验证路径**：确保目标路径在允许的目录内

### 存储路径安全

- **使用绝对路径**：避免相对路径
- **限制存储目录**：文件只能存储在指定目录
- **防止路径遍历**：验证文件名不包含 `../`
- **目录权限**：设置正确的目录权限

### 文件类型验证

- **不要信任客户端 MIME 类型**：`$_FILES['file']['type']` 由客户端提供，不可信
- **服务器端验证**：使用 `finfo_file()` 或 `getimagesize()` 验证
- **扩展名验证**：验证文件扩展名
- **内容验证**：验证文件实际内容

### 文件大小限制

- **PHP 配置**：`upload_max_filesize`、`post_max_size`
- **表单限制**：`MAX_FILE_SIZE` 隐藏字段（客户端限制，不可信）
- **服务器端验证**：始终在服务器端验证文件大小

### 权限设置

- **上传目录权限**：设置正确的目录权限（通常 755）
- **文件权限**：设置正确的文件权限（通常 644）
- **Web 服务器权限**：确保 Web 服务器有写入权限

### 恶意文件防护

- **文件内容扫描**：扫描文件内容（如病毒扫描）
- **文件类型限制**：只允许安全的文件类型
- **执行权限**：上传目录不应有执行权限
- **隔离存储**：将上传文件存储在 Web 根目录外

## 使用场景

### 文件上传功能

- 用户头像上传
- 文档上传
- 图片上传
- 媒体文件上传

### 图片处理

- 图片上传和存储
- 图片验证和处理
- 缩略图生成

### 文档管理

- 文档上传
- 文档验证
- 文档存储和管理

### 媒体文件处理

- 视频上传
- 音频上传
- 媒体文件验证

## 注意事项

### 文件大小限制

- **PHP 配置**：`upload_max_filesize`、`post_max_size`
- **Web 服务器配置**：Nginx、Apache 的客户端最大请求体大小
- **表单限制**：`MAX_FILE_SIZE` 隐藏字段（仅客户端限制）

### 文件类型验证

- **不要信任客户端 MIME 类型**：必须服务器端验证
- **使用多种验证方法**：扩展名、MIME 类型、内容验证
- **严格限制**：只允许必要的文件类型

### 存储路径安全

- **使用绝对路径**：避免相对路径
- **限制存储目录**：文件只能存储在指定目录
- **防止路径遍历**：验证文件名

### 恶意文件防护

- **文件内容扫描**：扫描恶意内容
- **文件类型限制**：只允许安全类型
- **执行权限**：上传目录不应有执行权限

### 临时文件清理

- **自动清理**：PHP 会自动清理临时文件
- **手动清理**：处理失败时手动清理
- **定期清理**：定期清理上传目录中的旧文件

## 常见问题

### $_FILES 的结构是什么？

`$_FILES` 是关联数组，每个上传文件包含以下字段：
- `name`：原始文件名
- `type`：MIME 类型（客户端提供，不可信）
- `size`：文件大小（字节）
- `tmp_name`：临时文件路径
- `error`：错误代码

### 如何验证上传文件？

1. **检查错误**：检查 `error` 字段
2. **验证大小**：验证文件大小
3. **验证类型**：验证扩展名和 MIME 类型（服务器端）
4. **验证内容**：验证文件实际内容
5. **生成安全文件名**：生成安全的文件名

### 如何防止文件上传漏洞？

1. **严格验证文件类型**：只允许必要的文件类型
2. **服务器端验证**：不要信任客户端提供的信息
3. **生成安全文件名**：使用唯一文件名，避免路径遍历
4. **限制存储路径**：文件只能存储在指定目录
5. **设置正确权限**：上传目录不应有执行权限
6. **文件内容扫描**：扫描文件内容（如病毒扫描）

### 如何处理上传错误？

使用 `error` 字段检查上传错误：

```php
<?php
declare(strict_types=1);

$file = $_FILES['file'];

switch ($file['error']) {
    case UPLOAD_ERR_OK:
        // 处理文件
        break;
    case UPLOAD_ERR_INI_SIZE:
    case UPLOAD_ERR_FORM_SIZE:
        echo "文件大小超过限制\n";
        break;
    case UPLOAD_ERR_PARTIAL:
        echo "文件只有部分被上传\n";
        break;
    case UPLOAD_ERR_NO_FILE:
        echo "没有文件被上传\n";
        break;
    default:
        echo "上传错误\n";
}
```

## 最佳实践

### 严格验证文件类型

- 使用服务器端验证（`finfo_file()`、`getimagesize()`）
- 不要信任客户端 MIME 类型
- 使用多种验证方法（扩展名、MIME 类型、内容）

### 限制文件大小

- 配置 PHP 上传限制
- 配置 Web 服务器限制
- 服务器端验证文件大小

### 使用安全的文件名

- 生成唯一文件名（`uniqid()`、`random_bytes()`）
- 不要使用用户提供的文件名
- 清理扩展名，只允许安全字符

### 存储到安全目录

- 使用绝对路径
- 限制存储目录
- 防止路径遍历
- 设置正确权限

### 安全配置

- 上传目录不应有执行权限
- 设置正确的文件权限
- 使用 HTTPS 传输
- 定期清理临时文件

## 相关章节

- **[5.3.1 $_GET、$_POST 与 $_REQUEST](section-01-get-post-request.md)**：了解其他超全局变量的使用方法
- **[5.6 文件上传处理](../chapter-06-file-upload/readme.md)**：了解文件上传的详细处理方法
- **[4.2 文件系统操作](../../stage-04-system/chapter-02-filesystem/readme.md)**：了解文件系统操作的详细内容

## 练习任务

1. **实现单文件上传**
   - 创建文件上传表单
   - 处理文件上传
   - 验证文件类型和大小
   - 安全存储文件

2. **实现多文件上传**
   - 创建多文件上传表单
   - 处理多个文件上传
   - 验证每个文件
   - 批量存储文件

3. **实现文件验证函数**
   - 创建 `validateFileType()` 函数
   - 创建 `validateFileSize()` 函数
   - 创建 `generateSafeFileName()` 函数

4. **实现安全的文件上传**
   - 严格验证文件类型
   - 生成安全文件名
   - 使用安全存储路径
   - 设置正确权限

5. **实现文件上传安全防护**
   - 防止路径遍历攻击
   - 防止恶意文件上传
   - 实现文件内容验证
   - 测试安全防护效果
