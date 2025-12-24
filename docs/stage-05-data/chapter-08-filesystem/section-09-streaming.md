# 5.8.9 流式操作与流包装器

## 概述

PHP 的流（Stream）机制提供了统一的数据访问接口，允许通过相同的函数处理不同类型的资源。流包装器（Stream Wrappers）是流机制的核心，它提供了一种统一的方式来访问本地文件、网络资源、压缩文件、内存等。理解流的概念、流包装器、流上下文、流过滤器，以及流式操作的使用，对于构建高效、灵活的 PHP 应用至关重要。

## 流的概念

**流（Stream）**是数据流的抽象，提供了一种统一的方式来访问不同类型的数据源：

- **统一接口**：使用相同的函数（`fopen()`、`fread()`、`file_get_contents()` 等）处理不同类型的资源
- **流式处理**：支持流式读取和写入，节省内存
- **灵活性**：可以轻松切换不同的数据源，无需修改代码逻辑
- **扩展性**：可以注册自定义流包装器，处理特殊的数据源

**流的优势**：
- 统一访问不同数据源（文件、网络、内存等）
- 支持流式处理，节省内存
- 可以轻松切换不同的数据源

## 流包装器

**流包装器**是一种协议，用于告诉 PHP 如何访问不同类型的资源。它提供了一种统一的方式来处理各种数据源，无论是本地文件、网络资源还是其他特殊资源。

**流包装器的优势**：
- **统一接口**：使用相同的函数（`fopen()`、`fread()` 等）处理不同类型的资源
- **灵活性**：可以轻松切换不同的数据源，无需修改代码逻辑
- **扩展性**：可以注册自定义流包装器，处理特殊的数据源

### file:// - 本地文件系统

**协议**：`file://`

**说明**：访问本地文件系统，这是默认的流包装器。如果不指定协议，PHP 会默认使用 `file://`。

**示例**：

```php
<?php
declare(strict_types=1);

// 以下两种方式等价
$handle1 = fopen(__DIR__ . '/data.txt', 'r');
$handle2 = fopen('file://' . __DIR__ . '/data.txt', 'r');

// 读取文件
$content = file_get_contents('file://' . __DIR__ . '/data.txt');
```

### php:// - PHP 输入/输出流

**协议**：`php://`

**说明**：访问 PHP 的各种输入/输出流和特殊资源。

**常用的 php:// 流**：

| 流 | 说明 | 模式 |
| :--- | :--- | :--- |
| `php://input` | 读取原始 POST 数据 | 只读 |
| `php://output` | 写入到输出缓冲区 | 只写 |
| `php://stdin` | 标准输入 | 只读 |
| `php://stdout` | 标准输出 | 只写 |
| `php://stderr` | 标准错误输出 | 只写 |
| `php://memory` | 内存流（可读写） | 读写 |
| `php://temp` | 临时文件流（超过 2MB 时写入磁盘） | 读写 |
| `php://filter` | 流过滤器 | 读写 |

**示例**：

```php
<?php
declare(strict_types=1);

// 读取原始 POST 数据
$postData = file_get_contents('php://input');
echo $postData;

// 写入到输出缓冲区
$handle = fopen('php://output', 'w');
fwrite($handle, 'Hello, World!');
fclose($handle);

// 使用内存流
$memory = fopen('php://memory', 'rw');
fwrite($memory, 'Data in memory');
rewind($memory);
$content = fread($memory, 1024);
fclose($memory);

// 使用临时文件流（自动管理）
$temp = fopen('php://temp', 'rw');
fwrite($temp, 'Temporary data');
rewind($temp);
$content = fread($temp, 1024);
fclose($temp);
```

**php://filter 流过滤器**：

```php
<?php
declare(strict_types=1);

// 读取文件并转换为大写
$content = file_get_contents('php://filter/read=string.toupper/resource=' . __DIR__ . '/data.txt');

// 读取文件并应用多个过滤器
$content = file_get_contents('php://filter/read=string.toupper|string.rot13/resource=' . __DIR__ . '/data.txt');

// 写入时应用过滤器
$handle = fopen('php://filter/write=string.tolower/resource=' . __DIR__ . '/output.txt', 'w');
fwrite($handle, 'HELLO WORLD');
fclose($handle);
```

### http:// 和 https:// - HTTP/HTTPS 资源

**协议**：`http://`、`https://`

**说明**：访问远程 HTTP 或 HTTPS 资源。需要启用 `allow_url_fopen` 配置选项。

**配置要求**：
- `allow_url_fopen=On`（在 `php.ini` 中）

**示例**：

```php
<?php
declare(strict_types=1);

// 读取远程文件
$content = file_get_contents('https://example.com/data.txt');
if ($content === false) {
    throw new RuntimeException('Cannot read remote file');
}

// 使用流式读取
$handle = fopen('https://example.com/large-file.zip', 'rb');
if ($handle === false) {
    throw new RuntimeException('Cannot open remote file');
}

while (($chunk = fread($handle, 8192)) !== false) {
    if ($chunk === '') {
        break;
    }
    echo $chunk;
}
fclose($handle);
```

**使用流上下文设置 HTTP 选项**：

```php
<?php
declare(strict_types=1);

// 创建流上下文
$context = stream_context_create([
    'http' => [
        'method' => 'POST',
        'header' => [
            'Content-Type: application/json',
            'Authorization: Bearer token123'
        ],
        'content' => json_encode(['key' => 'value']),
        'timeout' => 30,
        'follow_location' => true,
        'max_redirects' => 5
    ]
]);

// 使用上下文读取
$content = file_get_contents('https://api.example.com/data', false, $context);
```

### ftp:// - FTP 服务器

**协议**：`ftp://`

**说明**：访问 FTP 服务器上的文件。需要启用 `allow_url_fopen` 配置选项。

**配置要求**：
- `allow_url_fopen=On`

**示例**：

```php
<?php
declare(strict_types=1);

// 匿名 FTP 访问
$content = file_get_contents('ftp://ftp.example.com/pub/file.txt');

// 带认证的 FTP 访问
$content = file_get_contents('ftp://username:password@ftp.example.com/path/file.txt');

// 使用流上下文设置 FTP 选项
$context = stream_context_create([
    'ftp' => [
        'overwrite' => true,
        'resume_pos' => 0
    ]
]);

$handle = fopen('ftp://user:pass@ftp.example.com/file.txt', 'r', false, $context);
```

### data:// - 数据流

**协议**：`data://`

**说明**：访问数据 URI（Data URI），常用于内联数据。

**格式**：`data://[mediatype][;base64],<data>`

**示例**：

```php
<?php
declare(strict_types=1);

// 读取数据 URI
$data = 'data://text/plain;base64,SGVsbG8gV29ybGQ=';
$content = file_get_contents($data);
echo $content;  // 输出: Hello World

// 使用文本数据
$text = 'data://text/plain,Hello World';
$content = file_get_contents($text);
echo $content;  // 输出: Hello World
```

### zip:// - ZIP 压缩包

**协议**：`zip://`

**说明**：访问 ZIP 压缩包内的文件。

**格式**：`zip://<zip-file-path>#<file-in-zip>`

**示例**：

```php
<?php
declare(strict_types=1);

// 读取 ZIP 包内的文件
$content = file_get_contents('zip://' . __DIR__ . '/archive.zip#file.txt');

// 使用绝对路径
$content = file_get_contents('zip://' . realpath(__DIR__ . '/archive.zip') . '#file.txt');

// 流式读取
$handle = fopen('zip://' . __DIR__ . '/archive.zip#large-file.txt', 'r');
if ($handle !== false) {
    while (($line = fgets($handle)) !== false) {
        echo $line;
    }
    fclose($handle);
}
```

### zlib:// 和 bzip2:// - 压缩流

**协议**：`zlib://`、`bzip2://`

**说明**：访问 gzip 和 bzip2 压缩的文件。

**示例**：

```php
<?php
declare(strict_types=1);

// 读取 gzip 压缩文件
$content = file_get_contents('zlib://' . __DIR__ . '/data.txt.gz');

// 读取 bzip2 压缩文件
$content = file_get_contents('bzip2://' . __DIR__ . '/data.txt.bz2');
```

## 流上下文（Stream Context）

**流上下文**用于设置流包装器的选项，如 HTTP 头、超时、FTP 选项等。

### 创建流上下文

**语法**：`stream_context_create(?array $options = null, ?array $params = null): resource`

**示例**：

```php
<?php
declare(strict_types=1);

// 创建 HTTP 流上下文
$httpContext = stream_context_create([
    'http' => [
        'method' => 'GET',
        'header' => [
            'User-Agent: MyApp/1.0',
            'Accept: application/json'
        ],
        'timeout' => 10
    ]
]);

// 使用上下文
$content = file_get_contents('https://api.example.com/data', false, $httpContext);
```

### 常用流上下文选项

**HTTP/HTTPS 选项**：

```php
<?php
declare(strict_types=1);

$context = stream_context_create([
    'http' => [
        'method' => 'POST',
        'header' => [
            'Content-Type: application/json',
            'Authorization: Bearer token'
        ],
        'content' => json_encode(['data' => 'value']),
        'timeout' => 30,
        'follow_location' => true,      // 跟随重定向
        'max_redirects' => 5,           // 最大重定向次数
        'ignore_errors' => true,       // 忽略 HTTP 错误
        'user_agent' => 'MyApp/1.0'     // User-Agent
    ]
]);
```

**FTP 选项**：

```php
<?php
declare(strict_types=1);

$context = stream_context_create([
    'ftp' => [
        'overwrite' => true,            // 覆盖已存在的文件
        'resume_pos' => 0,              // 断点续传位置
        'proxy' => 'tcp://proxy:8080'   // 代理服务器
    ]
]);
```

## 流过滤器

流过滤器用于在读取或写入数据时对数据进行处理。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen('data.txt', 'r');

// 添加过滤器
stream_filter_append($handle, 'string.toupper');

$data = fread($handle, 1024);  // 自动转换为大写
fclose($handle);
```

## 流包装器的实际应用

### 应用 1：下载远程文件

```php
<?php
declare(strict_types=1);

function downloadFile(string $url, string $destination): bool
{
    $context = stream_context_create([
        'http' => [
            'timeout' => 30,
            'follow_location' => true
        ]
    ]);
    
    $content = file_get_contents($url, false, $context);
    if ($content === false) {
        return false;
    }
    
    return file_put_contents($destination, $content) !== false;
}

// 使用
downloadFile('https://example.com/file.zip', __DIR__ . '/downloaded.zip');
```

### 应用 2：流式处理大文件

```php
<?php
declare(strict_types=1);

function processLargeFile(string $url): void
{
    $context = stream_context_create([
        'http' => [
            'timeout' => 60
        ]
    ]);
    
    $handle = fopen($url, 'rb', false, $context);
    if ($handle === false) {
        throw new RuntimeException('Cannot open remote file');
    }
    
    $output = fopen('php://output', 'wb');
    
    try {
        while (($chunk = fread($handle, 8192)) !== false) {
            if ($chunk === '') {
                break;
            }
            fwrite($output, $chunk);
        }
    } finally {
        fclose($handle);
        fclose($output);
    }
}
```

### 应用 3：使用内存流处理数据

```php
<?php
declare(strict_types=1);

function processInMemory(string $data): string
{
    // 创建内存流
    $stream = fopen('php://memory', 'rw');
    if ($stream === false) {
        throw new RuntimeException('Cannot create memory stream');
    }
    
    try {
        // 写入数据
        fwrite($stream, $data);
        
        // 重置指针
        rewind($stream);
        
        // 应用过滤器（转换为大写）
        $filtered = stream_filter_append($stream, 'string.toupper', STREAM_FILTER_READ);
        
        // 读取处理后的数据
        $result = stream_get_contents($stream);
        
        return $result !== false ? $result : '';
    } finally {
        fclose($stream);
    }
}
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

1. **安全性**：使用 `http://`、`ftp://` 等远程流包装器时，注意验证 URL 来源，防止 SSRF（服务器端请求伪造）攻击
2. **性能**：远程流操作可能较慢，注意设置超时时间
3. **配置要求**：某些流包装器需要特定的 PHP 配置（如 `allow_url_fopen`）
4. **错误处理**：始终检查流操作的返回值
5. **资源释放**：使用 `fopen()` 打开流后，必须使用 `fclose()` 释放资源
6. **资源管理**：及时关闭文件句柄
7. **内存优化**：避免一次性加载大文件

## 练习

1. 实现一个流式文件处理工具。
2. 创建一个流包装器，支持自定义数据源。
3. 实现流过滤器，处理流数据。
4. 编写一个流式下载工具。
5. 编写一个函数，使用流包装器下载远程文件并保存到本地
6. 实现一个使用内存流处理数据的函数
7. 创建一个函数，使用流上下文设置 HTTP 头并读取远程 API 数据
8. 编写一个函数，读取 ZIP 压缩包内的特定文件
