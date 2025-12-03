# 2.14.1 文件操作

## 概述

文件操作是 PHP 开发中的常见任务。PHP 提供了丰富的文件操作函数，包括读取、写入、复制、删除等。

## 文件读取

### file_get_contents() - 读取整个文件

**语法**：`file_get_contents(string $filename, bool $use_include_path = false, ?resource $context = null, int $offset = 0, ?int $length = null): string|false`

```php
<?php
declare(strict_types=1);

// 读取整个文件
$content = file_get_contents(__DIR__ . '/data.txt');
echo $content;

// 读取部分内容
$content = file_get_contents(__DIR__ . '/data.txt', false, null, 0, 100);
```

### fopen() / fread() / fclose() - 流式读取

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'r');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

while (!feof($handle)) {
    $chunk = fread($handle, 8192);  // 每次读取 8KB
    echo $chunk;
}

fclose($handle);
```

### fgets() - 逐行读取

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'r');
while (($line = fgets($handle)) !== false) {
    echo $line;
}
fclose($handle);
```

## 文件写入

### file_put_contents() - 写入整个文件

**语法**：`file_put_contents(string $filename, mixed $data, int $flags = 0, ?resource $context = null): int|false`

```php
<?php
declare(strict_types=1);

// 覆盖写入
file_put_contents(__DIR__ . '/output.txt', 'Hello, World!');

// 追加写入
file_put_contents(__DIR__ . '/output.txt', "\nNew line", FILE_APPEND);
```

### fopen() / fwrite() / fclose() - 流式写入

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/output.txt', 'w');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

fwrite($handle, 'Line 1\n');
fwrite($handle, 'Line 2\n');

fclose($handle);
```

## 文件操作函数

### copy() - 复制文件

**语法**：`copy(string $from, string $to): bool`

```php
<?php
declare(strict_types=1);

copy(__DIR__ . '/source.txt', __DIR__ . '/destination.txt');
```

### rename() - 重命名/移动文件

**语法**：`rename(string $from, string $to, ?resource $context = null): bool`

```php
<?php
declare(strict_types=1);

rename(__DIR__ . '/old.txt', __DIR__ . '/new.txt');
```

### unlink() - 删除文件

**语法**：`unlink(string $filename, ?resource $context = null): bool`

```php
<?php
declare(strict_types=1);

unlink(__DIR__ . '/file.txt');
```

## 文件信息

### file_exists() - 检查文件是否存在

```php
<?php
declare(strict_types=1);

if (file_exists(__DIR__ . '/file.txt')) {
    echo "File exists\n";
}
```

### filesize() - 获取文件大小

```php
<?php
declare(strict_types=1);

$size = filesize(__DIR__ . '/file.txt');
echo "File size: {$size} bytes\n";
```

### filemtime() - 获取修改时间

```php
<?php
declare(strict_types=1);

$mtime = filemtime(__DIR__ . '/file.txt');
echo "Last modified: " . date('Y-m-d H:i:s', $mtime) . "\n";
```

## 完整示例

```php
<?php
declare(strict_types=1);

class FileOperations
{
    public static function readFile(string $path): string
    {
        $content = file_get_contents($path);
        if ($content === false) {
            throw new RuntimeException("Cannot read file: {$path}");
        }
        return $content;
    }
    
    public static function writeFile(string $path, string $content, bool $append = false): void
    {
        $flags = $append ? FILE_APPEND : 0;
        if (file_put_contents($path, $content, $flags) === false) {
            throw new RuntimeException("Cannot write file: {$path}");
        }
    }
    
    public static function copyFile(string $from, string $to): void
    {
        if (!copy($from, $to)) {
            throw new RuntimeException("Cannot copy file: {$from} to {$to}");
        }
    }
    
    public static function getFileInfo(string $path): array
    {
        if (!file_exists($path)) {
            throw new RuntimeException("File does not exist: {$path}");
        }
        
        return [
            'size' => filesize($path),
            'modified' => filemtime($path),
            'type' => filetype($path),
            'readable' => is_readable($path),
            'writable' => is_writable($path)
        ];
    }
}

// 使用示例
try {
    $content = FileOperations::readFile(__DIR__ . '/data.txt');
    FileOperations::writeFile(__DIR__ . '/output.txt', $content);
    $info = FileOperations::getFileInfo(__DIR__ . '/data.txt');
    print_r($info);
} catch (RuntimeException $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

## 注意事项

1. **错误处理**：始终检查文件操作的返回值。

2. **权限检查**：操作前检查文件权限（`is_readable()`、`is_writable()`）。

3. **路径安全**：验证文件路径，防止路径遍历攻击。

4. **大文件处理**：对于大文件，使用流式操作而不是一次性读取。

5. **资源释放**：使用 `fopen()` 后必须使用 `fclose()` 释放资源。

## 练习

1. 创建一个文件工具类，封装常用的文件操作。

2. 编写一个函数，安全地读取大文件（使用流式读取）。

3. 实现一个文件备份功能，复制文件并添加时间戳。

4. 创建一个函数，获取目录中所有文件的信息。

5. 编写一个函数，实现文件的原子写入（先写入临时文件，再重命名）。
