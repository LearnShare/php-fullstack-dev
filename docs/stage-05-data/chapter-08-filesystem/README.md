# 5.8 文件系统流式操作

## 目标

- 掌握文件流式读写操作。
- 理解文件句柄与缓冲区的作用。
- 能够高效处理大文件。
- 了解如何避免文件操作的竞争条件。

## 文件流基础

### 打开文件

```php
<?php
declare(strict_types=1);

// 打开文件（只读）
$handle = fopen('data.txt', 'r');
if ($handle === false) {
    throw new RuntimeException('Failed to open file');
}

// 打开文件（写入，覆盖）
$handle = fopen('data.txt', 'w');

// 打开文件（追加）
$handle = fopen('data.txt', 'a');

// 打开文件（读写）
$handle = fopen('data.txt', 'r+');

// 二进制模式
$handle = fopen('image.jpg', 'rb');
```

### 文件模式

| 模式 | 说明                     | 文件不存在 | 文件存在       |
| :--- | :----------------------- | :--------- | :------------- |
| `r`  | 只读                     | 失败       | 打开           |
| `r+` | 读写                     | 失败       | 打开           |
| `w`  | 只写（覆盖）             | 创建       | 清空后打开     |
| `w+` | 读写（覆盖）             | 创建       | 清空后打开     |
| `a`  | 追加（只写）             | 创建       | 追加到末尾     |
| `a+` | 追加（读写）             | 创建       | 追加到末尾     |
| `x`  | 只写（必须不存在）       | 创建       | 失败           |
| `x+` | 读写（必须不存在）       | 创建       | 失败           |
| `c`  | 只写（不截断）           | 创建       | 打开           |
| `c+` | 读写（不截断）           | 创建       | 打开           |

### 读取文件

```php
<?php
declare(strict_types=1);

// 读取指定长度
$handle = fopen('data.txt', 'r');
$data = fread($handle, 1024);  // 读取 1024 字节
fclose($handle);

// 读取一行
$handle = fopen('data.txt', 'r');
while (($line = fgets($handle)) !== false) {
    echo $line;
}
fclose($handle);

// 读取 CSV
$handle = fopen('data.csv', 'r');
while (($row = fgetcsv($handle)) !== false) {
    print_r($row);
}
fclose($handle);

// 读取所有内容（小文件）
$content = file_get_contents('data.txt');
```

### 写入文件

```php
<?php
declare(strict_types=1);

// 写入字符串
$handle = fopen('data.txt', 'w');
fwrite($handle, 'Hello World');
fclose($handle);

// 写入多行
$handle = fopen('data.txt', 'w');
fwrite($handle, "Line 1\n");
fwrite($handle, "Line 2\n");
fclose($handle);

// 写入 CSV
$handle = fopen('data.csv', 'w');
fputcsv($handle, ['Name', 'Email']);
fputcsv($handle, ['Alice', 'alice@example.com']);
fclose($handle);

// 快速写入（小文件）
file_put_contents('data.txt', 'Hello World');
```

## 流式处理大文件

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

// 使用
processLargeFile('large.log', function (string $line) {
    if (strpos($line, 'ERROR') !== false) {
        echo $line . "\n";
    }
});
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

// 使用
foreach (readFileInChunks('large.bin', 8192) as $chunk) {
    // 处理每个块
    processChunk($chunk);
}
```

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

// 使用
function generateData(): Generator
{
    for ($i = 0; $i < 1000000; $i++) {
        yield "Line {$i}\n";
    }
}

writeLargeData('output.txt', generateData());
```

## 文件句柄与缓冲区

### 缓冲区控制

```php
<?php
declare(strict_types=1);

// 设置缓冲区大小
$handle = fopen('data.txt', 'r');
stream_set_read_buffer($handle, 8192);  // 8KB 缓冲区
stream_set_write_buffer($handle, 8192);

// 禁用缓冲区（立即写入）
stream_set_write_buffer($handle, 0);

// 刷新缓冲区
fflush($handle);
```

### 文件指针操作

```php
<?php
declare(strict_types=1);

$handle = fopen('data.txt', 'r+');

// 获取当前位置
$position = ftell($handle);

// 移动到指定位置
fseek($handle, 0, SEEK_SET);  // 文件开头
fseek($handle, 0, SEEK_END);  // 文件末尾
fseek($handle, 100, SEEK_CUR);  // 当前位置 +100

// 回到开头
rewind($handle);

// 获取文件大小
$size = filesize('data.txt');
```

## 避免竞争条件

### 文件锁

```php
<?php
declare(strict_types=1);

function safeWrite(string $filename, string $data): void
{
    $handle = fopen($filename, 'c+');  // 创建或打开，不截断
    
    if ($handle === false) {
        throw new RuntimeException("Failed to open file: {$filename}");
    }
    
    try {
        // 获取排他锁
        if (!flock($handle, LOCK_EX)) {
            throw new RuntimeException("Failed to acquire lock");
        }
        
        // 写入数据
        fwrite($handle, $data);
        fflush($handle);
        
        // 锁会在 fclose 时自动释放
    } finally {
        fclose($handle);
    }
}
```

### 原子写入

```php
<?php
declare(strict_types=1);

function atomicWrite(string $filename, string $data): void
{
    // 写入临时文件
    $tempFile = $filename . '.tmp';
    file_put_contents($tempFile, $data);
    
    // 原子性重命名
    if (!rename($tempFile, $filename)) {
        unlink($tempFile);
        throw new RuntimeException("Failed to rename file");
    }
}
```

## 实际应用示例

### CSV 流式处理

```php
<?php
declare(strict_types=1);

class CsvStreamProcessor
{
    public function process(string $filename, callable $processor): void
    {
        $handle = fopen($filename, 'r');
        if ($handle === false) {
            throw new RuntimeException("Failed to open file: {$filename}");
        }
        
        try {
            // 跳过标题行
            fgetcsv($handle);
            
            while (($row = fgetcsv($handle)) !== false) {
                $processor($row);
            }
        } finally {
            fclose($handle);
        }
    }
    
    public function write(string $filename, array $headers, Generator $rows): void
    {
        $handle = fopen($filename, 'w');
        if ($handle === false) {
            throw new RuntimeException("Failed to open file: {$filename}");
        }
        
        try {
            fputcsv($handle, $headers);
            
            foreach ($rows as $row) {
                fputcsv($handle, $row);
            }
        } finally {
            fclose($handle);
        }
    }
}
```

### 日志文件处理

```php
<?php
declare(strict_types=1);

class LogProcessor
{
    public function tail(string $filename, int $lines = 10): array
    {
        $handle = fopen($filename, 'r');
        if ($handle === false) {
            return [];
        }
        
        try {
            // 移动到文件末尾
            fseek($handle, 0, SEEK_END);
            $position = ftell($handle);
            
            $buffer = '';
            $lineCount = 0;
            
            // 从后往前读取
            while ($position > 0 && $lineCount < $lines) {
                $position--;
                fseek($handle, $position, SEEK_SET);
                $char = fgetc($handle);
                
                if ($char === "\n") {
                    $lineCount++;
                }
                
                $buffer = $char . $buffer;
            }
            
            return array_filter(explode("\n", $buffer));
        } finally {
            fclose($handle);
        }
    }
    
    public function rotate(string $filename, int $maxSize = 10485760): void
    {
        if (filesize($filename) > $maxSize) {
            $backup = $filename . '.' . date('Y-m-d');
            rename($filename, $backup);
            
            // 压缩旧日志
            if (function_exists('gzencode')) {
                $compressed = gzencode(file_get_contents($backup));
                file_put_contents($backup . '.gz', $compressed);
                unlink($backup);
            }
        }
    }
}
```

## 最佳实践

### 1. 始终检查文件操作结果

```php
$handle = fopen($filename, 'r');
if ($handle === false) {
    throw new RuntimeException("Failed to open file");
}
```

### 2. 使用 try-finally 确保关闭

```php
$handle = fopen($filename, 'r');
try {
    // 操作文件
} finally {
    fclose($handle);
}
```

### 3. 大文件使用流式处理

```php
// 不推荐：一次性读取大文件
$content = file_get_contents('large.txt');  // 可能内存溢出

// 推荐：流式处理
$handle = fopen('large.txt', 'r');
while (($line = fgets($handle)) !== false) {
    // 处理每一行
}
fclose($handle);
```

### 4. 使用文件锁防止竞争

```php
$handle = fopen($filename, 'c+');
flock($handle, LOCK_EX);
// 操作文件
flock($handle, LOCK_UN);
fclose($handle);
```

## 练习

1. 实现一个 CSV 流式处理器，支持逐行读取和写入大文件。

2. 创建一个日志文件处理器，支持日志轮转和压缩。

3. 编写一个文件合并工具，使用流式处理合并多个大文件。

4. 实现一个文件下载器，支持断点续传和流式下载。

5. 设计一个文件监控系统，实时监控文件变化并处理。

6. 创建一个文件备份工具，使用流式处理备份大文件。
