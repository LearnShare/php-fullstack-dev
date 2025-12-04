# 5.8.1 流式操作

## 概述

PHP 的流（Stream）机制提供了统一的数据访问接口。本节详细介绍流的概念、流包装器、流上下文、流过滤器，以及流式操作的使用。

## 流的概念

- **流（Stream）**：数据流的抽象
- 统一访问不同数据源（文件、网络、内存等）
- 支持流式处理，节省内存

## 流包装器

### 文件流

```php
<?php
declare(strict_types=1);

$handle = fopen('data.txt', 'r');
$data = fread($handle, 1024);
fclose($handle);
```

### 网络流

```php
<?php
declare(strict_types=1);

$handle = fopen('http://example.com/data.txt', 'r');
$data = stream_get_contents($handle);
fclose($handle);
```

### 内存流

```php
<?php
declare(strict_types=1);

$handle = fopen('php://memory', 'r+');
fwrite($handle, 'Hello World');
rewind($handle);
$data = fread($handle, 1024);
fclose($handle);
```

## 流上下文

```php
<?php
declare(strict_types=1);

$context = stream_context_create([
    'http' => [
        'method' => 'POST',
        'header' => 'Content-Type: application/json',
        'content' => json_encode(['key' => 'value']),
    ],
]);

$handle = fopen('http://example.com/api', 'r', false, $context);
```

## 流过滤器

```php
<?php
declare(strict_types=1);

$handle = fopen('data.txt', 'r');

// 添加过滤器
stream_filter_append($handle, 'string.toupper');

$data = fread($handle, 1024);  // 自动转换为大写
fclose($handle);
```

## 完整示例

```php
<?php
declare(strict_types=1);

class StreamProcessor
{
    public function processFile(string $filename, callable $processor): void
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

    public function downloadFile(string $url, string $destination): void
    {
        $source = fopen($url, 'r');
        $target = fopen($destination, 'w');
        
        stream_copy_to_stream($source, $target);
        
        fclose($source);
        fclose($target);
    }
}
```

## 注意事项

1. **资源管理**：及时关闭文件句柄
2. **错误处理**：处理文件打开失败等错误
3. **性能考虑**：使用流式处理大文件
4. **内存优化**：避免一次性加载大文件

## 练习

1. 实现一个流式文件处理工具。

2. 创建一个流包装器，支持自定义数据源。

3. 实现流过滤器，处理流数据。

4. 编写一个流式下载工具。
