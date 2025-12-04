# 5.8.2 大文件处理

## 概述

大文件处理需要特殊的技术来避免内存溢出。本节详细介绍大文件读取、大文件写入、大文件上传、大文件下载，以及内存优化技巧。

## 大文件读取

### 逐行读取

```php
<?php
declare(strict_types=1);

function processLargeFile(string $filename, callable $processor): void
{
    $handle = fopen($filename, 'r');
    if ($handle === false) {
        throw new RuntimeException("Failed to open file: {$filename}");
    }
    
    try {
        while (($line = fgets($handle)) !== false) {
            $processor(trim($line));
        }
    } finally {
        fclose($handle);
    }
}
```

### 分块读取

```php
<?php
declare(strict_types=1);

function readFileInChunks(string $filename, int $chunkSize = 8192): Generator
{
    $handle = fopen($filename, 'r');
    if ($handle === false) {
        throw new RuntimeException("Failed to open file: {$filename}");
    }
    
    try {
        while (!feof($handle)) {
            yield fread($handle, $chunkSize);
        }
    } finally {
        fclose($handle);
    }
}
```

## 大文件写入

### 流式写入

```php
<?php
declare(strict_types=1);

function writeLargeData(string $filename, Generator $dataGenerator): void
{
    $handle = fopen($filename, 'w');
    if ($handle === false) {
        throw new RuntimeException("Failed to open file: {$filename}");
    }
    
    try {
        foreach ($dataGenerator as $chunk) {
            fwrite($handle, $chunk);
        }
    } finally {
        fclose($handle);
    }
}
```

## 大文件上传

```php
<?php
declare(strict_types=1);

function uploadLargeFile(string $sourcePath, string $destinationPath): void
{
    $source = fopen($sourcePath, 'rb');
    $target = fopen($destinationPath, 'wb');
    
    stream_copy_to_stream($source, $target);
    
    fclose($source);
    fclose($target);
}
```

## 大文件下载

```php
<?php
declare(strict_types=1);

function downloadFile(string $filePath): void
{
    $handle = fopen($filePath, 'rb');
    if ($handle === false) {
        http_response_code(404);
        exit;
    }
    
    header('Content-Type: application/octet-stream');
    header('Content-Disposition: attachment; filename="' . basename($filePath) . '"');
    header('Content-Length: ' . filesize($filePath));
    
    while (!feof($handle)) {
        echo fread($handle, 8192);
        flush();
    }
    
    fclose($handle);
}
```

## 内存优化

### 使用 Generator

```php
<?php
declare(strict_types=1);

function generateLargeData(): Generator
{
    for ($i = 0; $i < 1000000; $i++) {
        yield "Line {$i}\n";
    }
}

// 使用 Generator 不会一次性占用内存
foreach (generateLargeData() as $line) {
    // 处理每行
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class LargeFileHandler
{
    public function processLargeFile(string $filename, callable $processor): void
    {
        $handle = fopen($filename, 'r');
        if ($handle === false) {
            throw new RuntimeException("Failed to open file: {$filename}");
        }
        
        try {
            while (($line = fgets($handle)) !== false) {
                $processor(trim($line));
            }
        } finally {
            fclose($handle);
        }
    }

    public function downloadLargeFile(string $filePath): void
    {
        $handle = fopen($filePath, 'rb');
        header('Content-Type: application/octet-stream');
        header('Content-Length: ' . filesize($filePath));
        
        while (!feof($handle)) {
            echo fread($handle, 8192);
            flush();
        }
        
        fclose($handle);
    }
}
```

## 注意事项

1. **内存管理**：使用流式处理避免内存溢出
2. **资源释放**：及时关闭文件句柄
3. **错误处理**：处理文件操作失败
4. **性能优化**：合理设置缓冲区大小

## 练习

1. 实现大文件逐行处理功能。

2. 创建一个大文件下载工具。

3. 实现大文件分块上传。

4. 编写内存优化的大文件处理工具。
