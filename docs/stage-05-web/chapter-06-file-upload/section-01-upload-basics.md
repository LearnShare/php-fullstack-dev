# 5.6.1 文件上传基础

## 概述

文件上传是 Web 应用中的常见功能，允许用户将文件从客户端传输到服务器。理解文件上传的流程、掌握 `$_FILES` 超全局变量的使用、了解 `move_uploaded_file()` 函数、配置上传参数，对于实现文件上传功能至关重要。本节详细介绍文件上传的基础处理方法，包括表单配置、`$_FILES` 处理、文件移动、错误处理、上传配置等内容，帮助零基础学员掌握文件上传技术。

文件上传涉及多个环节：HTML 表单配置、PHP 接收文件、临时文件处理、文件移动、错误处理等。理解这些环节的工作原理，对于实现安全、可靠的文件上传功能至关重要。

**主要内容**：
- 文件上传概述（流程、原理、限制）
- HTML 表单配置（`enctype="multipart/form-data"`、`MAX_FILE_SIZE`）
- `$_FILES` 超全局变量的结构和处理
- `move_uploaded_file()` 函数的语法、参数和使用方法
- 上传错误处理和错误码
- PHP 上传配置（`upload_max_filesize`、`post_max_size`、`max_file_uploads` 等）
- 临时文件处理
- 实际应用示例和最佳实践

## 特性

- **自动处理**：PHP 自动处理文件上传
- **临时存储**：上传的文件先存储在临时目录
- **安全移动**：使用 `move_uploaded_file()` 安全移动文件
- **错误处理**：提供详细的错误码
- **配置灵活**：支持多种上传配置

## 文件上传概述

### 文件上传流程

1. **客户端**：用户选择文件并提交表单
2. **传输**：文件通过 HTTP POST 请求传输到服务器
3. **接收**：PHP 接收文件并存储在临时目录
4. **验证**：验证文件信息（大小、类型等）
5. **移动**：使用 `move_uploaded_file()` 移动到目标位置
6. **处理**：进行后续处理（存储、处理等）

### 文件上传原理

- **HTTP 协议**：使用 `multipart/form-data` 编码格式
- **临时存储**：PHP 将上传的文件存储在临时目录（`sys_get_temp_dir()`）
- **自动清理**：脚本结束后，临时文件自动删除
- **安全检查**：`move_uploaded_file()` 只移动通过 HTTP POST 上传的文件

### 文件上传限制

- **PHP 配置**：`upload_max_filesize`、`post_max_size`
- **Web 服务器**：Nginx、Apache 的客户端最大请求体大小
- **表单限制**：`MAX_FILE_SIZE` 隐藏字段（仅客户端限制）

## HTML 表单配置

### 基本表单

**必需配置**：
- `method="POST"`：必须使用 POST 方法
- `enctype="multipart/form-data"`：必须设置此编码类型

**示例**：
```html
<form method="POST" action="upload.php" enctype="multipart/form-data">
    <input type="file" name="file">
    <button type="submit">上传</button>
</form>
```

### MAX_FILE_SIZE 字段

`MAX_FILE_SIZE` 是一个隐藏字段，用于在客户端限制文件大小（仅客户端限制，不可信）。

**示例**：
```html
<form method="POST" action="upload.php" enctype="multipart/form-data">
    <input type="hidden" name="MAX_FILE_SIZE" value="5242880"> <!-- 5MB -->
    <input type="file" name="file">
    <button type="submit">上传</button>
</form>
```

**注意**：`MAX_FILE_SIZE` 必须在文件输入字段之前，且仅提供客户端限制，服务器端仍需验证。

### 多文件上传

**示例**：
```html
<form method="POST" action="upload.php" enctype="multipart/form-data">
    <input type="file" name="files[]" multiple>
    <button type="submit">上传多个文件</button>
</form>
```

## $_FILES 超全局变量

### 基本结构

`$_FILES` 是一个关联数组，每个上传的文件对应一个数组元素。

**单文件上传结构**：
```php
$_FILES = [
    'file' => [
        'name' => 'example.jpg',        // 原始文件名
        'type' => 'image/jpeg',         // MIME 类型（客户端提供，不可信）
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
| `type` | MIME 类型（客户端提供，不可信） | `image/jpeg` |
| `size` | 文件大小（字节） | `12345` |
| `tmp_name` | 临时文件路径 | `/tmp/phpXXXXXX` |
| `error` | 错误代码 | `UPLOAD_ERR_OK` |

### 基本用法

**示例 1：检查文件是否上传**
```php
<?php
declare(strict_types=1);

if (isset($_FILES['file'])) {
    $file = $_FILES['file'];
    echo "文件名: {$file['name']}\n";
    echo "文件大小: {$file['size']} 字节\n";
} else {
    echo "没有文件上传\n";
}
```

**示例 2：获取文件信息**
```php
<?php
declare(strict_types=1);

if (isset($_FILES['file'])) {
    $file = $_FILES['file'];
    
    echo "原始文件名: {$file['name']}\n";
    echo "MIME 类型: {$file['type']}\n";
    echo "文件大小: " . number_format($file['size']) . " 字节\n";
    echo "临时文件路径: {$file['tmp_name']}\n";
    echo "错误代码: {$file['error']}\n";
}
```

## move_uploaded_file() 函数

### 语法

**语法**：`move_uploaded_file(string $from, string $to): bool`

### 参数

- `$from`：临时文件路径（`$_FILES['file']['tmp_name']`）
- `$to`：目标文件路径

### 返回值

成功返回 `true`，失败返回 `false`。

### 特性

- **安全检查**：只移动通过 HTTP POST 上传的文件
- **自动验证**：自动验证文件是否是通过上传获得的
- **原子操作**：移动操作是原子的

### 基本用法

**示例 1：基本文件移动**
```php
<?php
declare(strict_types=1);

if (isset($_FILES['file'])) {
    $file = $_FILES['file'];
    
    // 检查上传错误
    if ($file['error'] !== UPLOAD_ERR_OK) {
        die('文件上传失败');
    }
    
    // 目标路径
    $uploadDir = __DIR__ . '/uploads/';
    $destination = $uploadDir . basename($file['name']);
    
    // 移动文件
    if (move_uploaded_file($file['tmp_name'], $destination)) {
        echo "文件上传成功: {$file['name']}\n";
    } else {
        echo "文件移动失败\n";
    }
}
```

**示例 2：生成安全文件名**
```php
<?php
declare(strict_types=1);

if (isset($_FILES['file'])) {
    $file = $_FILES['file'];
    
    if ($file['error'] !== UPLOAD_ERR_OK) {
        die('文件上传失败');
    }
    
    // 生成安全文件名
    $extension = pathinfo($file['name'], PATHINFO_EXTENSION);
    $safeFileName = uniqid() . '.' . $extension;
    
    $uploadDir = __DIR__ . '/uploads/';
    $destination = $uploadDir . $safeFileName;
    
    if (move_uploaded_file($file['tmp_name'], $destination)) {
        echo "文件上传成功: {$safeFileName}\n";
    }
}
```

### 与 copy() 的区别

| 特性 | move_uploaded_file() | copy() |
|:-----|:---------------------|:-------|
| 安全检查 | ✅ 只移动上传的文件 | ❌ 无安全检查 |
| 验证来源 | ✅ 验证文件来源 | ❌ 不验证来源 |
| 安全性 | ✅ 更安全 | ⚠️ 可能不安全 |
| 使用场景 | 文件上传 | 一般文件复制 |

**注意**：文件上传必须使用 `move_uploaded_file()`，不要使用 `copy()` 或 `rename()`。

## 上传错误处理

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

### 错误处理示例

**示例 1：基本错误检查**
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

// 使用
if (isset($_FILES['file'])) {
    $file = $_FILES['file'];
    try {
        checkUploadError($file['error']);
        // 处理文件上传
    } catch (RuntimeException $e) {
        echo "错误: {$e->getMessage()}\n";
    }
}
```

**示例 2：详细错误处理**
```php
<?php
declare(strict_types=1);

function getUploadErrorMessage(int $error): string
{
    return match ($error) {
        UPLOAD_ERR_OK => '上传成功',
        UPLOAD_ERR_INI_SIZE => '文件大小超过 PHP 配置限制',
        UPLOAD_ERR_FORM_SIZE => '文件大小超过表单限制',
        UPLOAD_ERR_PARTIAL => '文件只有部分被上传',
        UPLOAD_ERR_NO_FILE => '没有文件被上传',
        UPLOAD_ERR_NO_TMP_DIR => '找不到临时文件夹',
        UPLOAD_ERR_CANT_WRITE => '文件写入失败',
        UPLOAD_ERR_EXTENSION => 'PHP 扩展阻止了文件上传',
        default => '未知错误',
    };
}

// 使用
if (isset($_FILES['file'])) {
    $file = $_FILES['file'];
    $errorMsg = getUploadErrorMessage($file['error']);
    echo "上传结果: {$errorMsg}\n";
}
```

## PHP 上传配置

### 主要配置项

| 配置项 | 说明 | 默认值 |
|:-------|:-----|:-------|
| `file_uploads` | 是否允许文件上传 | `On` |
| `upload_max_filesize` | 单个文件最大大小 | `2M` |
| `post_max_size` | POST 请求最大大小 | `8M` |
| `max_file_uploads` | 单次请求最大文件数 | `20` |
| `upload_tmp_dir` | 临时文件目录 | 系统临时目录 |

### 查看配置

**示例**：
```php
<?php
declare(strict_types=1);

echo "文件上传: " . (ini_get('file_uploads') ? '启用' : '禁用') . "\n";
echo "最大文件大小: " . ini_get('upload_max_filesize') . "\n";
echo "POST 最大大小: " . ini_get('post_max_size') . "\n";
echo "最大文件数: " . ini_get('max_file_uploads') . "\n";
echo "临时目录: " . ini_get('upload_tmp_dir') ?: sys_get_temp_dir() . "\n";
```

### 配置说明

**upload_max_filesize**：
- 限制单个文件的最大大小
- 必须小于 `post_max_size`
- 示例：`upload_max_filesize = 10M`

**post_max_size**：
- 限制整个 POST 请求的最大大小
- 必须大于 `upload_max_filesize`
- 示例：`post_max_size = 20M`

**max_file_uploads**：
- 限制单次请求可以上传的最大文件数
- 示例：`max_file_uploads = 50`

**upload_tmp_dir**：
- 指定临时文件存储目录
- 如果未设置，使用系统临时目录
- 示例：`upload_tmp_dir = /tmp/uploads`

### 运行时配置

**注意**：某些配置项（如 `upload_max_filesize`）不能在运行时修改。

**示例**：
```php
<?php
declare(strict_types=1);

// 可以修改的配置
ini_set('max_file_uploads', 50);

// 不能修改的配置（会失败）
ini_set('upload_max_filesize', '10M');  // 无效
```

## 临时文件处理

### 临时文件位置

**获取临时目录**：
```php
<?php
declare(strict_types=1);

// 方法 1：使用 sys_get_temp_dir()
$tempDir = sys_get_temp_dir();
echo "系统临时目录: {$tempDir}\n";

// 方法 2：使用 ini_get()
$uploadTmpDir = ini_get('upload_tmp_dir');
if (empty($uploadTmpDir)) {
    $uploadTmpDir = sys_get_temp_dir();
}
echo "上传临时目录: {$uploadTmpDir}\n";
```

### 临时文件生命周期

1. **创建**：文件上传时，PHP 在临时目录创建临时文件
2. **使用**：脚本可以使用临时文件
3. **移动**：使用 `move_uploaded_file()` 移动到目标位置
4. **清理**：脚本结束后，未移动的临时文件自动删除

**注意**：临时文件在脚本结束后会自动删除，因此必须及时处理。

### 临时文件检查

**示例**：
```php
<?php
declare(strict_types=1);

if (isset($_FILES['file'])) {
    $file = $_FILES['file'];
    $tmpName = $file['tmp_name'];
    
    // 检查临时文件是否存在
    if (!file_exists($tmpName)) {
        die('临时文件不存在');
    }
    
    // 检查是否为上传的文件
    if (!is_uploaded_file($tmpName)) {
        die('不是上传的文件');
    }
    
    // 处理文件
    // ...
}
```

### is_uploaded_file() 函数

**语法**：`is_uploaded_file(string $filename): bool`

**作用**：检查文件是否是通过 HTTP POST 上传的。

**示例**：
```php
<?php
declare(strict_types=1);

if (isset($_FILES['file'])) {
    $tmpName = $_FILES['file']['tmp_name'];
    
    if (is_uploaded_file($tmpName)) {
        echo "是上传的文件\n";
    } else {
        echo "不是上传的文件\n";
    }
}
```

## 使用场景

### 用户头像上传

- 上传用户头像
- 图片处理和裁剪
- 存储用户头像

### 文档上传

- 上传文档文件
- 文档验证和处理
- 文档存储和管理

### 图片上传

- 上传图片文件
- 图片验证和处理
- 图片存储和展示

### 媒体文件上传

- 上传视频、音频文件
- 媒体文件验证
- 媒体文件存储

## 注意事项

### 表单 enctype 配置

- **必须设置**：`enctype="multipart/form-data"`
- **位置**：在 `<form>` 标签中
- **作用**：告诉浏览器使用 multipart 编码

### 上传大小限制

- **PHP 配置**：`upload_max_filesize`、`post_max_size`
- **Web 服务器**：Nginx、Apache 的客户端最大请求体大小
- **表单限制**：`MAX_FILE_SIZE`（仅客户端限制）

### 临时文件处理

- **及时处理**：临时文件在脚本结束后自动删除
- **使用 move_uploaded_file()**：不要使用 `copy()` 或 `rename()`
- **检查文件**：使用 `is_uploaded_file()` 检查文件来源

### 错误处理

- **检查错误码**：始终检查 `error` 字段
- **提供错误信息**：给用户提供清晰的错误信息
- **记录错误**：记录错误日志便于调试

## 常见问题

### 如何配置文件上传表单？

必须设置：
- `method="POST"`
- `enctype="multipart/form-data"`

```html
<form method="POST" action="upload.php" enctype="multipart/form-data">
    <input type="file" name="file">
    <button type="submit">上传</button>
</form>
```

### move_uploaded_file() 的作用？

- 将上传的临时文件移动到目标位置
- 只移动通过 HTTP POST 上传的文件
- 提供安全检查

### 上传文件大小如何限制？

1. **PHP 配置**：`upload_max_filesize`、`post_max_size`
2. **Web 服务器配置**：Nginx、Apache 的客户端最大请求体大小
3. **代码验证**：在代码中验证文件大小

### 如何处理上传错误？

检查 `$_FILES['file']['error']` 字段：

```php
<?php
declare(strict_types=1);

if (isset($_FILES['file'])) {
    $error = $_FILES['file']['error'];
    if ($error !== UPLOAD_ERR_OK) {
        // 处理错误
    }
}
```

## 最佳实践

### 检查上传错误

- 始终检查 `error` 字段
- 提供清晰的错误信息
- 记录错误日志

### 验证文件信息

- 验证文件大小
- 验证文件类型（服务器端）
- 验证文件内容

### 使用 move_uploaded_file()

- 必须使用 `move_uploaded_file()` 移动文件
- 不要使用 `copy()` 或 `rename()`
- 使用 `is_uploaded_file()` 检查文件来源

### 配置合理的上传限制

- 根据需求设置合理的文件大小限制
- 确保 `post_max_size` 大于 `upload_max_filesize`
- 考虑服务器资源限制

## 相关章节

- **[5.6.2 文件验证与安全](section-02-validation-security.md)**：了解文件验证的详细内容
- **[5.3.3 $_FILES 与安全处理](../chapter-03-superglobals/section-03-files-security.md)**：了解 $_FILES 的基础使用
- **[4.2 文件系统操作](../../stage-04-system/chapter-02-filesystem/readme.md)**：了解文件系统操作的详细内容

## 练习任务

1. **实现基本文件上传**
   - 创建文件上传表单
   - 处理文件上传
   - 移动文件到目标位置

2. **实现错误处理**
   - 检查所有上传错误
   - 提供清晰的错误信息
   - 处理各种错误情况

3. **实现多文件上传**
   - 处理多个文件上传
   - 逐个验证和移动文件
   - 处理部分成功的情况

4. **实现上传配置检查**
   - 检查 PHP 上传配置
   - 验证配置是否合理
   - 提供配置建议

5. **实现完整的文件上传处理类**
   - 创建文件上传处理类
   - 封装上传逻辑
   - 提供错误处理和日志记录
