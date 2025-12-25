# 4.5.1 文件上传基础

## 概述

文件上传是 Web 应用中的常见功能。本节详细介绍 `multipart/form-data` 编码格式、`$_FILES` 数据结构、单文件/多文件上传处理、规范化处理，以及错误处理。

## multipart/form-data

### 表单编码类型

- **`application/x-www-form-urlencoded`**：普通表单数据
- **`multipart/form-data`**：包含文件上传的表单

```html
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
    'tmp_name' => '/tmp/phpXXXXXX',  // 临时文件路径
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
```

## 上传错误处理

### 错误代码

| 常量 | 值 | 说明 |
| :--- | :--- | :--- |
| `UPLOAD_ERR_OK` | 0 | 上传成功 |
| `UPLOAD_ERR_INI_SIZE` | 1 | 超过 `upload_max_filesize` |
| `UPLOAD_ERR_FORM_SIZE` | 2 | 超过表单 `MAX_FILE_SIZE` |
| `UPLOAD_ERR_PARTIAL` | 3 | 文件部分上传 |
| `UPLOAD_ERR_NO_FILE` | 4 | 没有文件上传 |
| `UPLOAD_ERR_NO_TMP_DIR` | 6 | 临时目录不存在 |
| `UPLOAD_ERR_CANT_WRITE` | 7 | 写入失败 |
| `UPLOAD_ERR_EXTENSION` | 8 | PHP 扩展阻止上传 |

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

## 基础上传处理

```php
<?php
declare(strict_types=1);

if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['file'])) {
    $file = $_FILES['file'];
    
    // 检查错误
    if ($file['error'] !== UPLOAD_ERR_OK) {
        echo "上传失败: " . getUploadErrorMessage($file['error']);
        exit;
    }
    
    // 验证是否为上传文件
    if (!is_uploaded_file($file['tmp_name'])) {
        echo "文件验证失败";
        exit;
    }
    
    // 移动文件
    $uploadDir = __DIR__ . '/uploads/';
    $filename = basename($file['name']);
    $destination = $uploadDir . $filename;
    
    if (move_uploaded_file($file['tmp_name'], $destination)) {
        echo "文件上传成功: {$filename}";
    } else {
        echo "文件移动失败";
    }
}
```

## 注意事项

1. **验证上传文件**：使用 `is_uploaded_file()` 验证文件来源
2. **错误处理**：检查 `error` 字段，处理各种错误情况
3. **临时文件**：上传的文件存储在临时目录，需要及时移动
4. **多文件处理**：规范化处理多文件上传的数组结构

## 练习

1. 创建一个文件上传处理函数，包含错误处理和文件移动。

2. 实现多文件上传处理，支持批量上传和验证。

3. 编写一个文件上传规范化函数，统一处理单文件和多文件上传。

4. 创建一个文件上传错误处理类，提供详细的错误信息。
