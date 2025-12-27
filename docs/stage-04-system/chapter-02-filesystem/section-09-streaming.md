# 4.2.9 流式操作与流包装器

## 概述

流（Stream）是 PHP 中统一的数据访问抽象，提供了访问不同类型数据源的统一接口。无论是本地文件、网络资源、压缩文件还是内存数据，都可以通过流来访问。流包装器（Stream Wrappers）是流机制的核心，它提供了一种统一的方式来处理各种数据源。

理解流的概念、流包装器、流上下文、流过滤器，以及流式操作的使用，对于构建高效、灵活的 PHP 应用至关重要。流机制使得代码可以处理各种数据源，而无需关心底层的具体实现，提高了代码的复用性和灵活性。

**主要内容**：
- 流的概念和优势
- 流包装器概述
- 内置流包装器（`file://`、`http://`、`https://`、`php://`、`data://`）
- 流上下文（`stream_context_create()`）
- 流过滤器
- 自定义流包装器
- 实际应用场景和最佳实践

## 特性

- **统一接口**：使用相同的函数处理不同类型的资源
- **流式处理**：支持流式读取和写入，节省内存
- **灵活性**：可以轻松切换不同的数据源，无需修改代码逻辑
- **扩展性**：可以注册自定义流包装器，处理特殊的数据源
- **功能丰富**：支持上下文配置、过滤器等功能

## 语法/定义

### stream_context_create() 函数

**语法**：`stream_context_create(?array $options = null, ?array $params = null): resource|false`

**参数**：
- `$options`：可选，流选项数组，键为协议名称，值为选项数组
- `$params`：可选，流参数数组

**返回值**：成功返回流上下文资源，失败返回 `false`。

### stream_filter_append() 函数

**语法**：`stream_filter_append(resource $stream, string $filtername, int $mode = 0, mixed $params = null): resource|false`

**参数**：
- `$stream`：流资源
- `$filtername`：过滤器名称
- `$mode`：过滤器模式（`STREAM_FILTER_READ`、`STREAM_FILTER_WRITE` 或 `STREAM_FILTER_ALL`）
- `$params`：可选，过滤器参数

**返回值**：成功返回过滤器资源，失败返回 `false`。

## 基本用法

### 示例 1：使用 file:// 流包装器

```php
<?php
declare(strict_types=1);

// file:// 是默认的流包装器，以下两种方式等价
$handle1 = fopen(__DIR__ . '/data.txt', 'r');
$handle2 = fopen('file://' . __DIR__ . '/data.txt', 'r');

// 使用 file_get_contents() 读取文件（默认使用 file://）
$content = file_get_contents('file://' . __DIR__ . '/data.txt');
echo $content;

if ($handle1 !== false) {
    fclose($handle1);
}
if ($handle2 !== false) {
    fclose($handle2);
}
```

**说明**：
- `file://` 是访问本地文件系统的流包装器
- 如果不指定协议，PHP 默认使用 `file://`

### 示例 2：使用 php:// 流包装器

```php
<?php
declare(strict_types=1);

// php://input：读取原始 POST 数据
$postData = file_get_contents('php://input');
echo "POST data: {$postData}\n";

// php://output：写入到输出缓冲区
$handle = fopen('php://output', 'w');
if ($handle !== false) {
    fwrite($handle, 'Hello, World!');
    fclose($handle);
}

// php://memory：内存流（可读写）
$memory = fopen('php://memory', 'rw');
if ($memory !== false) {
    fwrite($memory, 'Data in memory');
    rewind($memory);
    $content = fread($memory, 1024);
    echo "Memory content: {$content}\n";
    fclose($memory);
}

// php://temp：临时文件流（超过 2MB 时写入磁盘）
$temp = fopen('php://temp', 'rw');
if ($temp !== false) {
    fwrite($temp, 'Temporary data');
    rewind($temp);
    $content = fread($temp, 1024);
    echo "Temp content: {$content}\n";
    fclose($temp);
}
```

**说明**：
- `php://input`：读取原始 POST 数据（只读）
- `php://output`：写入到输出缓冲区（只写）
- `php://memory`：内存流，数据存储在内存中（可读写）
- `php://temp`：临时文件流，超过 2MB 时自动写入磁盘（可读写）

### 示例 3：使用 http:// 和 https:// 流包装器

```php
<?php
declare(strict_types=1);

// 需要启用 allow_url_fopen 配置选项
// 使用 HTTP 流读取远程文件
$content = file_get_contents('http://example.com/data.txt');
if ($content !== false) {
    echo $content;
}

// 使用 HTTPS 流
$secureContent = file_get_contents('https://example.com/secure-data.txt');
if ($secureContent !== false) {
    echo $secureContent;
}
```

**说明**：
- `http://` 和 `https://` 用于访问远程 HTTP/HTTPS 资源
- 需要启用 `allow_url_fopen` 配置选项
- 支持流式读取，适合处理大文件

### 示例 4：使用流上下文

```php
<?php
declare(strict_types=1);

// 创建 HTTP 流上下文
$httpContext = stream_context_create([
    'http' => [
        'method' => 'GET',
        'header' => [
            'User-Agent: MyApp/1.0',
            'Accept: application/json',
        ],
        'timeout' => 10,
    ],
]);

// 使用上下文读取远程文件
$content = file_get_contents('https://api.example.com/data', false, $httpContext);
if ($content !== false) {
    echo $content;
}

// POST 请求
$postContext = stream_context_create([
    'http' => [
        'method' => 'POST',
        'header' => [
            'Content-Type: application/json',
            'Authorization: Bearer token123',
        ],
        'content' => json_encode(['key' => 'value']),
        'timeout' => 30,
    ],
]);

$response = file_get_contents('https://api.example.com/endpoint', false, $postContext);
```

**说明**：
- `stream_context_create()` 创建流上下文
- 可以为不同的协议设置不同的选项
- HTTP 上下文支持设置请求方法、头部、超时等

### 示例 5：使用 php://filter 流过滤器

```php
<?php
declare(strict_types=1);

// 读取文件并转换为大写
$content = file_get_contents('php://filter/read=string.toupper/resource=' . __DIR__ . '/data.txt');
echo $content;

// 读取文件并应用多个过滤器
$content = file_get_contents('php://filter/read=string.toupper|string.rot13/resource=' . __DIR__ . '/data.txt');
echo $content;

// 写入时应用过滤器
$handle = fopen('php://filter/write=string.tolower/resource=' . __DIR__ . '/output.txt', 'w');
if ($handle !== false) {
    fwrite($handle, 'HELLO WORLD');
    fclose($handle);
}
```

**说明**：
- `php://filter` 用于应用流过滤器
- 可以在读取或写入时应用过滤器
- 支持多个过滤器，使用 `|` 分隔

### 示例 6：使用 stream_filter_append() 添加过滤器

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'r');
if ($handle !== false) {
    // 添加转换为大写的过滤器
    stream_filter_append($handle, 'string.toupper', STREAM_FILTER_READ);
    
    // 读取数据（自动转换为大写）
    $data = fread($handle, 1024);
    echo $data;
    
    fclose($handle);
}
```

**说明**：
- `stream_filter_append()` 为流添加过滤器
- `STREAM_FILTER_READ` 表示在读取时应用过滤器
- `STREAM_FILTER_WRITE` 表示在写入时应用过滤器

### 示例 7：流式处理大文件

```php
<?php
declare(strict_types=1);

function streamLargeFile(string $url, string $destination): void
{
    // 创建流上下文
    $context = stream_context_create([
        'http' => [
            'timeout' => 60,
            'follow_location' => true,
        ],
    ]);
    
    // 打开远程文件和本地文件
    $source = fopen($url, 'rb', false, $context);
    if ($source === false) {
        throw new RuntimeException('Cannot open remote file');
    }
    
    $dest = fopen($destination, 'wb');
    if ($dest === false) {
        fclose($source);
        throw new RuntimeException('Cannot create destination file');
    }
    
    try {
        // 流式复制（每次 8KB）
        while (($chunk = fread($source, 8192)) !== false) {
            if ($chunk === '') {
                break;
            }
            fwrite($dest, $chunk);
        }
    } finally {
        fclose($source);
        fclose($dest);
    }
}

// 使用
streamLargeFile('https://example.com/large-file.zip', __DIR__ . '/downloaded.zip');
```

**说明**：
- 流式处理适合大文件，避免一次性加载到内存
- 使用流上下文配置 HTTP 选项（超时、跟随重定向等）

### 示例 8：使用内存流处理数据

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
        stream_filter_append($stream, 'string.toupper', STREAM_FILTER_READ);
        
        // 读取处理后的数据
        $result = stream_get_contents($stream);
        
        return $result !== false ? $result : '';
    } finally {
        fclose($stream);
    }
}

$result = processInMemory('hello world');
echo $result;  // HELLO WORLD
```

**说明**：
- `php://memory` 在内存中存储数据，适合临时数据处理
- 可以在内存流上应用过滤器进行数据处理

## 使用场景

### 场景 1：下载远程文件

使用 HTTP 流下载远程文件。

**示例**：

```php
<?php
declare(strict_types=1);

function downloadFile(string $url, string $destination): bool
{
    $context = stream_context_create([
        'http' => [
            'timeout' => 30,
            'follow_location' => true,
            'max_redirects' => 5,
        ],
    ]);
    
    $content = file_get_contents($url, false, $context);
    if ($content === false) {
        return false;
    }
    
    return file_put_contents($destination, $content) !== false;
}

downloadFile('https://example.com/file.zip', __DIR__ . '/downloaded.zip');
```

### 场景 2：流式处理远程数据

流式处理远程大文件，避免内存溢出。

**示例**：见"示例 7：流式处理大文件"

### 场景 3：内存数据处理

使用内存流处理临时数据。

**示例**：见"示例 8：使用内存流处理数据"

## 注意事项

### allow_url_fopen 配置

使用 `http://` 和 `https://` 流包装器需要启用 `allow_url_fopen` 配置选项。

**示例**：

```php
<?php
declare(strict_types=1);

// 检查是否启用了 allow_url_fopen
if (!ini_get('allow_url_fopen')) {
    throw new RuntimeException('allow_url_fopen is disabled');
}

// 使用 HTTP 流
$content = file_get_contents('http://example.com/data.txt');
```

### 流上下文的性能

创建流上下文有轻微的性能开销，但对于需要配置选项的场景是必需的。

**示例**：

```php
<?php
declare(strict_types=1);

// 如果需要配置选项，使用流上下文
$context = stream_context_create(['http' => ['timeout' => 10]]);
$content = file_get_contents('http://example.com/data.txt', false, $context);

// 如果不需要配置选项，直接使用
$content = file_get_contents('http://example.com/data.txt');
```

### 内存流的限制

`php://memory` 流的数据存储在内存中，注意内存使用。

**示例**：

```php
<?php
declare(strict_types=1);

// 对于大数据，考虑使用 php://temp
$largeData = str_repeat('x', 1024 * 1024 * 10);  // 10MB

// php://temp 会在超过 2MB 时写入磁盘
$temp = fopen('php://temp', 'rw');
fwrite($temp, $largeData);
```

## 常见问题

### 问题 1：流和文件句柄的区别是什么？

**回答**：流是更广泛的概念，文件句柄是流的一种。流可以指向文件、网络资源、内存等，文件句柄通常指向文件。

**示例**：

```php
<?php
declare(strict_types=1);

// 文件句柄（文件流）
$fileHandle = fopen(__DIR__ . '/data.txt', 'r');

// 内存流
$memoryHandle = fopen('php://memory', 'rw');

// 网络流
$networkHandle = fopen('http://example.com/data.txt', 'r');
```

### 问题 2：如何使用 HTTP 流？

**回答**：使用 `http://` 或 `https://` 协议，需要启用 `allow_url_fopen`。可以使用流上下文配置请求选项。

**示例**：见"示例 3：使用 http:// 和 https:// 流包装器"和"示例 4：使用流上下文"

### 问题 3：流过滤器的用途是什么？

**回答**：流过滤器用于在读取或写入数据时对数据进行处理，比如转换大小写、加密、压缩等。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'r');
if ($handle !== false) {
    // 添加转换为大写的过滤器
    stream_filter_append($handle, 'string.toupper', STREAM_FILTER_READ);
    
    // 读取的数据会自动转换为大写
    $data = fread($handle, 1024);
    echo $data;
    
    fclose($handle);
}
```

## 最佳实践

### 1. 使用流统一访问不同数据源

使用流机制统一访问文件、网络、内存等不同数据源。

**示例**：

```php
<?php
declare(strict_types=1);

function readData(string $source): string
{
    // 无论是文件、HTTP URL 还是其他源，都使用相同的函数
    $content = file_get_contents($source);
    return $content !== false ? $content : '';
}

// 可以处理不同类型的源
readData(__DIR__ . '/data.txt');           // 文件
readData('http://example.com/data.txt');   // HTTP
readData('php://memory');                   // 内存流
```

### 2. 利用流上下文配置选项

使用流上下文配置请求选项，如超时、头部等。

**示例**：见"示例 4：使用流上下文"

### 3. 大文件使用流式处理

对于大文件，使用流式处理，避免内存溢出。

**示例**：见"示例 7：流式处理大文件"

### 4. 理解不同流包装器的特性

了解不同流包装器的特性和限制，选择合适的包装器。

**示例**：

```php
<?php
declare(strict_types=1);

// php://memory：适合小数据，完全在内存中
$memory = fopen('php://memory', 'rw');
fwrite($memory, 'Small data');

// php://temp：适合大数据，超过 2MB 时写入磁盘
$temp = fopen('php://temp', 'rw');
fwrite($temp, str_repeat('x', 1024 * 1024 * 5));  // 5MB
```

## 对比分析

### php://memory vs php://temp

| 特性         | php://memory                | php://temp                  |
|:-------------|:----------------------------|:----------------------------|
| **存储位置** | 完全在内存中                | 超过 2MB 时写入磁盘         |
| **适用大小** | 小数据（< 2MB）             | 大数据（> 2MB）             |
| **性能**     | ✅ 更快                     | ⚠️ 稍慢（可能涉及磁盘 I/O） |
| **内存使用** | ⚠️ 占用内存                 | ✅ 内存友好                 |

### file_get_contents() vs fopen() + fread()

| 特性         | file_get_contents()         | fopen() + fread()           |
|:-------------|:----------------------------|:----------------------------|
| **使用场景** | 一次性读取整个文件          | 流式读取                    |
| **内存使用** | ⚠️ 一次性加载到内存         | ✅ 流式处理，内存友好       |
| **适用大小** | 小到中等文件（< 10MB）      | 大文件（> 10MB）            |
| **代码复杂度** | ✅ 简单                    | ⚠️ 较复杂                   |

## 练习任务

1. **流工具类**：创建一个流工具类，封装常用的流操作，支持不同的流包装器。

2. **HTTP 下载工具**：实现一个 HTTP 下载工具，支持断点续传、进度显示等功能。

3. **流过滤器工具**：创建一个工具，应用多个流过滤器处理数据（如压缩、加密等）。

4. **内存流处理工具**：编写一个函数，使用内存流处理数据，支持数据处理和转换。

5. **流上下文配置工具**：实现一个工具，方便地创建和配置不同协议的流上下文。

## 相关章节

- **[4.2.1 文件读取](section-01-file-reading.md)**：了解文件读取的基础知识
- **[4.2.2 文件写入](section-02-file-writing.md)**：了解文件写入的基础知识
- **[4.2.10 大文件处理](section-10-large-files.md)**：学习大文件处理的详细内容
