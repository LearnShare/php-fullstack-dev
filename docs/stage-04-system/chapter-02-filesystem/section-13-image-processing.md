# 4.2.13 图像处理

## 概述

图像处理是文件系统操作中的常见需求。在实际应用中，我们经常需要处理图片，比如生成缩略图、添加水印、调整图片大小、转换图片格式等。PHP 提供了 GD 库（Graphics Draw）来进行图像处理，这是一个强大的图像处理扩展。

理解 GD 库的基本使用方法、图像创建和操作函数，以及如何处理不同格式的图片，对于构建功能丰富的 PHP 应用至关重要。虽然本节只介绍基础内容，但为后续深入学习图像处理打下基础。

**主要内容**：
- GD 库概述和检查
- 图像创建（`imagecreate()`、`imagecreatetruecolor()`）
- 图像读取（`imagecreatefromjpeg()`、`imagecreatefrompng()` 等）
- 图像操作（缩放、裁剪、旋转）
- 图像输出（`imagejpeg()`、`imagepng()` 等）
- `getimagesize()` 函数
- 基本图像处理示例
- 实际应用场景和注意事项

## 特性

- **多种格式支持**：支持 JPEG、PNG、GIF、WebP 等常见图像格式
- **丰富的操作**：支持缩放、裁剪、旋转、水印等操作
- **内存管理**：需要及时释放图像资源
- **质量控制**：支持设置图像质量
- **灵活输出**：可以输出到文件或直接输出到浏览器

## 语法/定义

### getimagesize() 函数

**语法**：`getimagesize(string $filename, array &$image_info = null): array|false`

**参数**：
- `$filename`：图像文件路径
- `$image_info`：可选，用于接收额外的图像信息

**返回值**：成功返回包含图像信息的数组，失败返回 `false`。

### imagecreatefromjpeg() 函数

**语法**：`imagecreatefromjpeg(string $filename): GdImage|false`

**参数**：
- `$filename`：JPEG 图像文件路径

**返回值**：成功返回图像资源，失败返回 `false`。

### imagejpeg() 函数

**语法**：`imagejpeg(GdImage $image, ?string $file = null, int $quality = -1): bool`

**参数**：
- `$image`：图像资源
- `$file`：可选，输出文件路径，如果为 `null` 则直接输出到浏览器
- `$quality`：可选，JPEG 质量（0-100），默认为 -1（使用默认质量）

**返回值**：成功返回 `true`，失败返回 `false`。

### imagescale() 函数

**语法**：`imagescale(GdImage $image, int $width, int $height = -1, int $mode = IMG_BILINEAR_FIXED): GdImage|false`

**参数**：
- `$image`：图像资源
- `$width`：新宽度
- `$height`：可选，新高度，-1 表示按比例缩放
- `$mode`：可选，缩放算法，默认为 `IMG_BILINEAR_FIXED`

**返回值**：成功返回新的图像资源，失败返回 `false`。

## 基本用法

### 示例 1：检查 GD 库是否可用

```php
<?php
declare(strict_types=1);

// 检查 GD 库是否已安装
if (!extension_loaded('gd')) {
    throw new RuntimeException('GD extension is not loaded');
}

// 获取 GD 库信息
$gdInfo = gd_info();
print_r($gdInfo);

// 检查支持的图像格式
if (function_exists('imagecreatefromjpeg')) {
    echo "JPEG support: Yes\n";
}
if (function_exists('imagecreatefrompng')) {
    echo "PNG support: Yes\n";
}
if (function_exists('imagecreatefromgif')) {
    echo "GIF support: Yes\n";
}
if (function_exists('imagecreatefromwebp')) {
    echo "WebP support: Yes\n";
}
```

**说明**：
- 使用 `extension_loaded('gd')` 检查 GD 库是否已加载
- `gd_info()` 返回 GD 库的详细信息
- 检查特定的图像格式支持函数

### 示例 2：获取图像信息

```php
<?php
declare(strict_types=1);

function getImageInfo(string $imagePath): array
{
    $info = getimagesize($imagePath);
    if ($info === false) {
        throw new RuntimeException("Cannot get image info: {$imagePath}");
    }
    
    return [
        'width' => $info[0],
        'height' => $info[1],
        'type' => $info[2],  // IMAGETYPE_JPEG, IMAGETYPE_PNG, etc.
        'mime' => $info['mime'],
        'bits' => $info['bits'] ?? null,
        'channels' => $info['channels'] ?? null,
    ];
}

$info = getImageInfo(__DIR__ . '/photo.jpg');
echo "Width: {$info['width']}\n";
echo "Height: {$info['height']}\n";
echo "MIME: {$info['mime']}\n";
```

**说明**：
- `getimagesize()` 返回图像的基本信息
- 返回数组包含宽度、高度、类型、MIME 类型等信息

### 示例 3：创建图像

```php
<?php
declare(strict_types=1);

// 创建一个 800x600 的真彩色图像
$image = imagecreatetruecolor(800, 600);
if ($image === false) {
    throw new RuntimeException('Cannot create image');
}

// 分配颜色（RGB）
$white = imagecolorallocate($image, 255, 255, 255);
$black = imagecolorallocate($image, 0, 0, 0);
$red = imagecolorallocate($image, 255, 0, 0);

// 填充背景色
imagefill($image, 0, 0, $white);

// 绘制文本
imagestring($image, 5, 100, 100, 'Hello, World!', $black);

// 保存图像
imagejpeg($image, __DIR__ . '/output.jpg', 90);

// 释放图像资源
imagedestroy($image);
```

**说明**：
- `imagecreatetruecolor()` 创建真彩色图像
- `imagecolorallocate()` 分配颜色
- `imagefill()` 填充区域
- `imagedestroy()` 释放图像资源（重要）

### 示例 4：读取图像

```php
<?php
declare(strict_types=1);

// 从文件读取 JPEG 图像
$image = imagecreatefromjpeg(__DIR__ . '/photo.jpg');
if ($image === false) {
    throw new RuntimeException('Cannot load JPEG image');
}

// 从文件读取 PNG 图像
$pngImage = imagecreatefrompng(__DIR__ . '/photo.png');
if ($pngImage === false) {
    throw new RuntimeException('Cannot load PNG image');
}

// 从文件读取 GIF 图像
$gifImage = imagecreatefromgif(__DIR__ . '/photo.gif');
if ($gifImage === false) {
    throw new RuntimeException('Cannot load GIF image');
}

// 释放资源
imagedestroy($image);
imagedestroy($pngImage);
imagedestroy($gifImage);
```

**说明**：
- 使用 `imagecreatefrom*()` 函数读取不同格式的图像
- 读取失败时返回 `false`，需要检查返回值
- 使用完毕后释放图像资源

### 示例 5：缩放图像

```php
<?php
declare(strict_types=1);

function resizeImage(string $sourcePath, string $destinationPath, int $maxWidth, int $maxHeight): void
{
    // 获取原始图像信息
    $info = getimagesize($sourcePath);
    if ($info === false) {
        throw new RuntimeException("Cannot get image info: {$sourcePath}");
    }
    
    $originalWidth = $info[0];
    $originalHeight = $info[1];
    $mime = $info['mime'];
    
    // 计算新尺寸（保持宽高比）
    $ratio = min($maxWidth / $originalWidth, $maxHeight / $originalHeight);
    $newWidth = (int)($originalWidth * $ratio);
    $newHeight = (int)($originalHeight * $ratio);
    
    // 根据 MIME 类型读取图像
    $sourceImage = match ($mime) {
        'image/jpeg' => imagecreatefromjpeg($sourcePath),
        'image/png' => imagecreatefrompng($sourcePath),
        'image/gif' => imagecreatefromgif($sourcePath),
        default => throw new RuntimeException("Unsupported image type: {$mime}"),
    };
    
    if ($sourceImage === false) {
        throw new RuntimeException("Cannot load source image: {$sourcePath}");
    }
    
    // 创建新图像
    $newImage = imagecreatetruecolor($newWidth, $newHeight);
    if ($newImage === false) {
        imagedestroy($sourceImage);
        throw new RuntimeException('Cannot create new image');
    }
    
    // 缩放图像
    imagecopyresampled(
        $newImage,
        $sourceImage,
        0, 0, 0, 0,
        $newWidth, $newHeight,
        $originalWidth, $originalHeight
    );
    
    // 保存新图像
    $saved = match ($mime) {
        'image/jpeg' => imagejpeg($newImage, $destinationPath, 90),
        'image/png' => imagepng($newImage, $destinationPath),
        'image/gif' => imagegif($newImage, $destinationPath),
        default => false,
    };
    
    // 释放资源
    imagedestroy($sourceImage);
    imagedestroy($newImage);
    
    if (!$saved) {
        throw new RuntimeException("Cannot save resized image: {$destinationPath}");
    }
}

// 使用
resizeImage(__DIR__ . '/photo.jpg', __DIR__ . '/thumbnail.jpg', 200, 200);
```

**说明**：
- 使用 `imagecopyresampled()` 缩放图像，保持质量
- 根据原始图像的 MIME 类型选择相应的读取和保存函数
- 及时释放图像资源

### 示例 6：裁剪图像

```php
<?php
declare(strict_types=1);

function cropImage(string $sourcePath, string $destinationPath, int $x, int $y, int $width, int $height): void
{
    // 读取源图像
    $sourceImage = imagecreatefromjpeg($sourcePath);
    if ($sourceImage === false) {
        throw new RuntimeException("Cannot load source image: {$sourcePath}");
    }
    
    // 创建裁剪后的图像
    $croppedImage = imagecrop($sourceImage, [
        'x' => $x,
        'y' => $y,
        'width' => $width,
        'height' => $height,
    ]);
    
    if ($croppedImage === false) {
        imagedestroy($sourceImage);
        throw new RuntimeException('Cannot crop image');
    }
    
    // 保存裁剪后的图像
    imagejpeg($croppedImage, $destinationPath, 90);
    
    // 释放资源
    imagedestroy($sourceImage);
    imagedestroy($croppedImage);
}

// 使用：从 (100, 100) 位置裁剪 200x200 的区域
cropImage(__DIR__ . '/photo.jpg', __DIR__ . '/cropped.jpg', 100, 100, 200, 200);
```

**说明**：
- `imagecrop()` 裁剪图像
- 需要指定裁剪区域的坐标和尺寸
- 及时释放源图像和新图像资源

### 示例 7：输出图像到浏览器

```php
<?php
declare(strict_types=1);

// 读取图像
$image = imagecreatefromjpeg(__DIR__ . '/photo.jpg');
if ($image === false) {
    throw new RuntimeException('Cannot load image');
}

// 设置 HTTP 头
header('Content-Type: image/jpeg');

// 输出图像（不保存到文件）
imagejpeg($image, null, 90);

// 释放资源
imagedestroy($image);
exit;
```

**说明**：
- 输出到浏览器时，第二个参数传入 `null`
- 需要设置正确的 Content-Type 头
- 输出后通常需要 `exit` 退出脚本

### 示例 8：转换图像格式

```php
<?php
declare(strict_types=1);

function convertImageFormat(string $sourcePath, string $destinationPath, string $targetFormat): void
{
    // 读取源图像
    $info = getimagesize($sourcePath);
    if ($info === false) {
        throw new RuntimeException("Cannot get image info: {$sourcePath}");
    }
    
    $mime = $info['mime'];
    $sourceImage = match ($mime) {
        'image/jpeg' => imagecreatefromjpeg($sourcePath),
        'image/png' => imagecreatefrompng($sourcePath),
        'image/gif' => imagecreatefromgif($sourcePath),
        default => throw new RuntimeException("Unsupported source format: {$mime}"),
    };
    
    if ($sourceImage === false) {
        throw new RuntimeException("Cannot load source image: {$sourcePath}");
    }
    
    // 保存为目标格式
    $saved = match (strtolower($targetFormat)) {
        'jpeg', 'jpg' => imagejpeg($sourceImage, $destinationPath, 90),
        'png' => imagepng($sourceImage, $destinationPath),
        'gif' => imagegif($sourceImage, $destinationPath),
        default => throw new RuntimeException("Unsupported target format: {$targetFormat}"),
    };
    
    imagedestroy($sourceImage);
    
    if (!$saved) {
        throw new RuntimeException("Cannot save converted image: {$destinationPath}");
    }
}

// 使用：将 PNG 转换为 JPEG
convertImageFormat(__DIR__ . '/photo.png', __DIR__ . '/photo.jpg', 'jpeg');
```

**说明**：
- 读取一种格式的图像，保存为另一种格式
- 需要注意不同格式的特性（如 PNG 支持透明，JPEG 不支持）

## 使用场景

### 场景 1：生成缩略图

为上传的图片生成缩略图。

**示例**：见"示例 5：缩放图像"

### 场景 2：图片上传处理

处理用户上传的图片，调整大小、转换格式等。

**示例**：

```php
<?php
declare(strict_types=1);

function processUploadedImage(array $file, string $destinationDir): string
{
    $tmpFile = $file['tmp_name'];
    $originalName = $file['name'];
    
    // 获取图像信息
    $info = getimagesize($tmpFile);
    if ($info === false) {
        throw new RuntimeException('Invalid image file');
    }
    
    // 读取图像
    $image = match ($info['mime']) {
        'image/jpeg' => imagecreatefromjpeg($tmpFile),
        'image/png' => imagecreatefrompng($tmpFile),
        default => throw new RuntimeException('Unsupported image format'),
    };
    
    if ($image === false) {
        throw new RuntimeException('Cannot load image');
    }
    
    // 生成文件名
    $extension = pathinfo($originalName, PATHINFO_EXTENSION);
    $filename = uniqid() . '.' . $extension;
    $destination = $destinationDir . DIRECTORY_SEPARATOR . $filename;
    
    // 保存图像
    match ($info['mime']) {
        'image/jpeg' => imagejpeg($image, $destination, 90),
        'image/png' => imagepng($image, $destination),
        default => throw new RuntimeException('Unsupported image format'),
    };
    
    imagedestroy($image);
    
    return $filename;
}
```

## 注意事项

### 内存使用

处理大图像时，注意内存使用，及时释放资源。

**示例**：

```php
<?php
declare(strict_types=1);

function processLargeImage(string $sourcePath): void
{
    $image = imagecreatefromjpeg($sourcePath);
    if ($image === false) {
        throw new RuntimeException('Cannot load image');
    }
    
    try {
        // 处理图像
        // ...
    } finally {
        // 确保释放资源
        imagedestroy($image);
    }
}
```

### GD 库支持检查

在使用 GD 库功能前，检查 GD 库是否已安装。

**示例**：

```php
<?php
declare(strict_types=1);

if (!extension_loaded('gd')) {
    throw new RuntimeException('GD extension is required for image processing');
}

// 检查特定的图像格式支持
if (!function_exists('imagecreatefromjpeg')) {
    throw new RuntimeException('JPEG support is not available');
}
```

### 图像资源释放

使用 `imagedestroy()` 及时释放图像资源，避免内存泄漏。

**示例**：

```php
<?php
declare(strict_types=1);

$image = imagecreatefromjpeg(__DIR__ . '/photo.jpg');
if ($image !== false) {
    try {
        // 使用图像
        // ...
    } finally {
        // 释放资源
        imagedestroy($image);
    }
}
```

## 常见问题

### 问题 1：如何检查 GD 库是否可用？

**回答**：使用 `extension_loaded('gd')` 检查 GD 库是否已加载，使用 `gd_info()` 获取详细信息。

**示例**：

```php
<?php
declare(strict_types=1);

if (!extension_loaded('gd')) {
    echo "GD extension is not loaded\n";
} else {
    $info = gd_info();
    print_r($info);
}
```

### 问题 2：如何生成缩略图？

**回答**：读取原始图像，使用 `imagecopyresampled()` 或 `imagescale()` 缩放，保存为新的图像文件。

**示例**：见"示例 5：缩放图像"

### 问题 3：如何处理大图片的内存问题？

**回答**：对于大图片，考虑使用 `imagescale()` 先缩小尺寸，或使用第三方库（如 Intervention Image）处理，及时释放资源。

**示例**：

```php
<?php
declare(strict_types=1);

function processLargeImageSafely(string $sourcePath): void
{
    // 先检查图像尺寸
    $info = getimagesize($sourcePath);
    if ($info === false) {
        throw new RuntimeException('Cannot get image info');
    }
    
    // 如果图像太大，先缩小
    $maxDimension = 2000;
    if ($info[0] > $maxDimension || $info[1] > $maxDimension) {
        $ratio = min($maxDimension / $info[0], $maxDimension / $info[1]);
        $newWidth = (int)($info[0] * $ratio);
        $newHeight = (int)($info[1] * $ratio);
        
        $image = imagecreatefromjpeg($sourcePath);
        if ($image !== false) {
            $scaled = imagescale($image, $newWidth, $newHeight);
            imagedestroy($image);
            $image = $scaled;
        }
    } else {
        $image = imagecreatefromjpeg($sourcePath);
    }
    
    if ($image === false) {
        throw new RuntimeException('Cannot load image');
    }
    
    try {
        // 处理图像
        // ...
    } finally {
        imagedestroy($image);
    }
}
```

## 最佳实践

### 1. 检查 GD 库支持

在使用 GD 库功能前，检查 GD 库是否已安装和支持所需的图像格式。

**示例**：

```php
<?php
declare(strict_types=1);

function checkGDSupport(): void
{
    if (!extension_loaded('gd')) {
        throw new RuntimeException('GD extension is not loaded');
    }
    
    $required = ['imagecreatefromjpeg', 'imagecreatefrompng', 'imagejpeg', 'imagepng'];
    foreach ($required as $function) {
        if (!function_exists($function)) {
            throw new RuntimeException("Required function not available: {$function}");
        }
    }
}
```

### 2. 及时释放图像资源

使用 `imagedestroy()` 及时释放图像资源，避免内存泄漏。

**示例**：

```php
<?php
declare(strict_types=1);

$image = imagecreatefromjpeg(__DIR__ . '/photo.jpg');
if ($image !== false) {
    try {
        // 处理图像
        processImage($image);
    } finally {
        imagedestroy($image);
    }
}
```

### 3. 处理大图像时注意内存

对于大图像，考虑先缩小尺寸或使用流式处理。

**示例**：见"问题 3：如何处理大图片的内存问题？"

### 4. 设置合适的图像质量

保存 JPEG 图像时，设置合适的质量参数（通常 80-90 是较好的平衡点）。

**示例**：

```php
<?php
declare(strict_types=1);

// 高质量（文件较大）
imagejpeg($image, __DIR__ . '/high-quality.jpg', 95);

// 中等质量（推荐，文件大小和质量平衡）
imagejpeg($image, __DIR__ . '/medium-quality.jpg', 85);

// 低质量（文件较小，质量较差）
imagejpeg($image, __DIR__ . '/low-quality.jpg', 60);
```

### 5. 检查图像文件的有效性

在处理图像前，使用 `getimagesize()` 验证图像文件的有效性。

**示例**：

```php
<?php
declare(strict_types=1);

function validateImageFile(string $filePath): bool
{
    $info = getimagesize($filePath);
    if ($info === false) {
        return false;
    }
    
    // 检查是否是支持的图像类型
    $supportedTypes = [IMAGETYPE_JPEG, IMAGETYPE_PNG, IMAGETYPE_GIF];
    return in_array($info[2], $supportedTypes, true);
}

if (!validateImageFile(__DIR__ . '/uploaded.jpg')) {
    throw new RuntimeException('Invalid image file');
}
```

## 对比分析

### imagecreate() vs imagecreatetruecolor()

| 特性         | imagecreate()               | imagecreatetruecolor()      |
|:-------------|:----------------------------|:----------------------------|
| **颜色深度** | 调色板图像（256色）         | 真彩色图像（1677万色）      |
| **质量**     | ⚠️ 颜色有限                 | ✅ 高质量                   |
| **使用场景** | 不需要高质量时              | 推荐使用（现代标准）        |
| **性能**     | ✅ 稍快（内存占用小）       | ⚠️ 稍慢（内存占用大）       |

### imagecopyresized() vs imagecopyresampled()

| 特性         | imagecopyresized()          | imagecopyresampled()        |
|:-------------|:----------------------------|:----------------------------|
| **算法**     | 简单缩放（速度快，质量差）  | 高质量重采样（速度慢，质量好）|
| **使用场景** | 不需要高质量时              | 推荐使用（质量更好）        |
| **性能**     | ✅ 较快                     | ⚠️ 较慢                     |

## 练习任务

1. **图像处理工具类**：创建一个图像处理工具类，封装常用的图像操作（缩放、裁剪、格式转换等）。

2. **缩略图生成工具**：实现一个工具，为上传的图片生成缩略图，支持多种尺寸。

3. **图像格式转换工具**：编写一个函数，将图像从一种格式转换为另一种格式。

4. **图像验证工具**：创建一个工具，验证上传的图像文件的有效性和安全性。

5. **图像上传处理工具**：实现一个工具，处理用户上传的图像，包括验证、调整大小、保存等。

## 相关章节

- **[4.2.1 文件读取](section-01-file-reading.md)**：了解文件读取的基础知识
- **[4.2.2 文件写入](section-02-file-writing.md)**：了解文件写入的基础知识
- **[4.2.5 二进制文件](section-05-binary-files.md)**：了解二进制文件处理的相关内容
