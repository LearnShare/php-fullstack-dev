# 4.2.6 文件锁和临时文件

## 概述

文件锁用于防止多进程并发访问文件时的数据冲突，临时文件用于存储临时数据和处理上传文件。在多进程或多线程环境中，多个进程可能同时访问同一个文件，导致数据损坏、数据丢失或读取错误。文件锁可以确保同一时间只有一个进程可以访问文件，从而保证数据的一致性和完整性。

临时文件用于存储临时数据、处理上传文件、缓存数据等场景。理解文件锁的工作原理和临时文件的使用场景，对于构建健壮的多进程 PHP 应用至关重要。

**主要内容**：
- 文件锁的概念和为什么需要文件锁
- `flock()` 函数的使用
- 锁的类型（共享锁、排他锁、非阻塞锁）
- `tmpfile()` 函数创建临时文件
- `tempnam()` 函数生成临时文件名
- `sys_get_temp_dir()` 获取系统临时目录
- 实际应用场景和最佳实践

## 特性

- **并发控制**：防止多进程并发访问冲突
- **数据保护**：确保数据的一致性和完整性
- **自动清理**：临时文件可以自动删除
- **跨进程同步**：文件锁在同一服务器上跨进程有效
- **灵活配置**：支持阻塞和非阻塞模式

## 语法/定义

### flock() 函数

**语法**：`flock(resource $stream, int $operation, int &$would_block = null): bool`

**参数**：
- `$stream`：文件句柄（必须是可写模式）
- `$operation`：锁定操作，可选值：
  - `LOCK_SH`：共享锁（读锁），多个进程可以同时读取
  - `LOCK_EX`：排他锁（写锁），只有一个进程可以写入
  - `LOCK_UN`：释放锁
  - `LOCK_NB`：非阻塞模式（可以与上述标志组合，使用 `|`）
- `$would_block`：可选，如果锁被占用，此参数会被设置为 `1`

**返回值**：成功返回 `true`，失败返回 `false`。

### tmpfile() 函数

**语法**：`tmpfile(): resource|false`

**返回值**：成功返回文件句柄，失败返回 `false`。

### tempnam() 函数

**语法**：`tempnam(string $directory, string $prefix): string|false`

**参数**：
- `$directory`：临时文件目录（如果目录不存在，会在系统临时目录创建）
- `$prefix`：文件名前缀

**返回值**：成功返回临时文件路径，失败返回 `false`。

### sys_get_temp_dir() 函数

**语法**：`sys_get_temp_dir(): string`

**返回值**：返回系统临时目录路径。

## 基本用法

### 示例 1：使用排他锁写入文件

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'a');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    // 获取排他锁（阻塞模式）
    if (flock($handle, LOCK_EX)) {
        // 写入数据
        fwrite($handle, "New entry: " . date('Y-m-d H:i:s') . "\n");
        
        // 释放锁
        flock($handle, LOCK_UN);
    } else {
        throw new RuntimeException('Cannot acquire lock');
    }
} finally {
    fclose($handle);
}
```

**说明**：
- `LOCK_EX` 表示排他锁（写锁）
- 获取锁后，其他进程必须等待
- 操作完成后释放锁

### 示例 2：使用共享锁读取文件

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'r');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    // 获取共享锁（读锁）
    if (flock($handle, LOCK_SH)) {
        // 读取数据（多个进程可以同时读取）
        $content = fread($handle, 1024);
        echo $content;
        
        // 释放锁
        flock($handle, LOCK_UN);
    }
} finally {
    fclose($handle);
}
```

**说明**：
- `LOCK_SH` 表示共享锁（读锁）
- 多个进程可以同时获取共享锁
- 共享锁与排他锁互斥

### 示例 3：非阻塞锁

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'a');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

try {
    // 尝试获取锁（非阻塞模式）
    $wouldBlock = 0;
    if (flock($handle, LOCK_EX | LOCK_NB, $wouldBlock)) {
        // 获取锁成功
        fwrite($handle, "New entry\n");
        flock($handle, LOCK_UN);
    } else {
        if ($wouldBlock === 1) {
            echo "File is locked, please try again later\n";
        } else {
            echo "Cannot acquire lock\n";
        }
    }
} finally {
    fclose($handle);
}
```

**说明**：
- `LOCK_NB` 表示非阻塞模式
- 如果锁被占用，函数立即返回 `false`
- `$wouldBlock` 参数指示是否因为锁被占用而失败

### 示例 4：使用 tmpfile() 创建临时文件

```php
<?php
declare(strict_types=1);

// 创建临时文件
$handle = tmpfile();
if ($handle === false) {
    throw new RuntimeException('Cannot create temporary file');
}

try {
    // 写入数据
    fwrite($handle, 'Temporary data');
    
    // 读取数据
    rewind($handle);
    $content = fread($handle, 1024);
    echo $content . "\n";
} finally {
    // 关闭文件（会自动删除）
    fclose($handle);
}
```

**说明**：
- `tmpfile()` 创建临时文件并返回文件句柄
- 文件会在脚本结束时自动删除
- 文件句柄关闭时也会自动删除

### 示例 5：使用 tempnam() 生成临时文件名

```php
<?php
declare(strict_types=1);

// 在指定目录创建临时文件
$tempFile = tempnam(__DIR__ . '/tmp', 'upload_');
if ($tempFile === false) {
    throw new RuntimeException('Cannot create temporary file name');
}

try {
    // 使用临时文件
    file_put_contents($tempFile, 'Temporary data');
    
    // 读取文件
    $content = file_get_contents($tempFile);
    echo $content . "\n";
} finally {
    // 使用完毕后手动删除
    unlink($tempFile);
}
```

**说明**：
- `tempnam()` 只生成文件名，不创建文件
- 文件名是唯一的
- 需要手动删除文件

### 示例 6：获取系统临时目录

```php
<?php
declare(strict_types=1);

// 获取系统临时目录
$tempDir = sys_get_temp_dir();
echo "Temporary directory: {$tempDir}\n";

// 在系统临时目录创建临时文件
$tempFile = tempnam($tempDir, 'php_');
if ($tempFile !== false) {
    echo "Temporary file: {$tempFile}\n";
    
    // 使用完毕后删除
    unlink($tempFile);
}
```

**输出**：

```
Temporary directory: /tmp
Temporary file: /tmp/php_XXXXXX
```

**说明**：
- `sys_get_temp_dir()` 返回系统临时目录路径
- 在不同操作系统上路径不同（Windows、Linux、macOS）

### 示例 7：安全写入文件（带锁）

```php
<?php
declare(strict_types=1);

function safeWrite(string $file, string $content, bool $append = false): void
{
    $mode = $append ? 'ab' : 'wb';
    $handle = fopen($file, $mode);
    
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$file}");
    }
    
    try {
        // 获取排他锁
        if (!flock($handle, LOCK_EX)) {
            throw new RuntimeException("Cannot acquire lock: {$file}");
        }
        
        // 写入数据
        if (fwrite($handle, $content) === false) {
            throw new RuntimeException("Cannot write to file: {$file}");
        }
    } finally {
        // 释放锁并关闭文件
        flock($handle, LOCK_UN);
        fclose($handle);
    }
}

// 使用
safeWrite(__DIR__ . '/log.txt', "Log entry\n", true);
```

**说明**：
- 使用 `try-finally` 确保锁总是被释放
- 即使发生异常，锁也会被释放

### 示例 8：非阻塞写入

```php
<?php
declare(strict_types=1);

function tryWrite(string $file, string $content): bool
{
    $handle = fopen($file, 'ab');
    if ($handle === false) {
        return false;
    }
    
    try {
        $wouldBlock = 0;
        // 尝试获取锁（非阻塞）
        if (flock($handle, LOCK_EX | LOCK_NB, $wouldBlock)) {
            if ($wouldBlock === 1) {
                return false;  // 锁被占用
            }
            
            // 获取锁成功，写入数据
            $result = fwrite($handle, $content) !== false;
            flock($handle, LOCK_UN);
            return $result;
        }
        
        return false;
    } finally {
        fclose($handle);
    }
}

// 使用
if (tryWrite(__DIR__ . '/log.txt', "Entry\n")) {
    echo "Write successful\n";
} else {
    echo "File is locked, skipping write\n";
}
```

**说明**：
- 非阻塞模式不会等待锁释放
- 如果锁被占用，立即返回 `false`
- 适合不需要等待的场景

## 使用场景

### 场景 1：并发写入保护

多个进程同时写入日志文件时，使用文件锁防止数据冲突。

**示例**：

```php
<?php
declare(strict_types=1);

function writeLog(string $logFile, string $message): void
{
    $handle = fopen($logFile, 'ab');
    if ($handle === false) {
        throw new RuntimeException("Cannot open log file: {$logFile}");
    }
    
    try {
        if (flock($handle, LOCK_EX)) {
            $timestamp = date('Y-m-d H:i:s');
            fwrite($handle, "[{$timestamp}] {$message}\n");
            flock($handle, LOCK_UN);
        }
    } finally {
        fclose($handle);
    }
}

// 多个进程可以安全地调用
writeLog(__DIR__ . '/app.log', 'User logged in');
```

### 场景 2：配置文件更新

更新配置文件时，使用文件锁确保原子性。

**示例**：

```php
<?php
declare(strict_types=1);

function updateConfig(string $configFile, array $config): void
{
    $handle = fopen($configFile, 'wb');
    if ($handle === false) {
        throw new RuntimeException("Cannot open config file: {$configFile}");
    }
    
    try {
        if (flock($handle, LOCK_EX)) {
            $json = json_encode($config, JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE);
            fwrite($handle, $json);
            flock($handle, LOCK_UN);
        }
    } finally {
        fclose($handle);
    }
}
```

### 场景 3：临时文件处理上传数据

使用临时文件处理用户上传的数据。

**示例**：

```php
<?php
declare(strict_types=1);

function processUpload(string $uploadedFile): string
{
    // 创建临时文件
    $tempFile = tempnam(sys_get_temp_dir(), 'upload_');
    if ($tempFile === false) {
        throw new RuntimeException('Cannot create temporary file');
    }
    
    try {
        // 复制上传的文件到临时文件
        if (!copy($uploadedFile, $tempFile)) {
            throw new RuntimeException('Cannot copy uploaded file');
        }
        
        // 处理临时文件
        $content = file_get_contents($tempFile);
        // ... 处理逻辑 ...
        
        return $tempFile;
    } catch (Exception $e) {
        // 发生错误时删除临时文件
        if (file_exists($tempFile)) {
            unlink($tempFile);
        }
        throw $e;
    }
}
```

## 注意事项

### 文件锁的范围

文件锁只在同一台服务器上有效，不能跨服务器。

**示例**：

```php
<?php
declare(strict_types=1);

// 文件锁在同一服务器上的多个进程间有效
// 但不适用于分布式系统中的多个服务器
$handle = fopen(__DIR__ . '/shared.txt', 'a');
if (flock($handle, LOCK_EX)) {
    // 只保护同一服务器上的并发访问
    fwrite($handle, "Entry\n");
    flock($handle, LOCK_UN);
}
fclose($handle);
```

### 锁的释放

确保在异常情况下也能释放锁，使用 `try-finally`。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'a');
if ($handle !== false) {
    try {
        if (flock($handle, LOCK_EX)) {
            try {
                // 可能抛出异常的操作
                fwrite($handle, "Entry\n");
            } finally {
                // 确保锁被释放
                flock($handle, LOCK_UN);
            }
        }
    } finally {
        // 确保文件被关闭
        fclose($handle);
    }
}
```

### 临时文件的清理

使用 `tempnam()` 创建的临时文件需要手动删除，`tmpfile()` 创建的会自动删除。

**示例**：

```php
<?php
declare(strict_types=1);

// tmpfile()：自动删除
$handle = tmpfile();
fwrite($handle, 'data');
fclose($handle);  // 文件自动删除

// tempnam()：需要手动删除
$tempFile = tempnam(__DIR__ . '/tmp', 'upload_');
file_put_contents($tempFile, 'data');
// ... 使用文件 ...
unlink($tempFile);  // 手动删除
```

### 非阻塞锁的使用

非阻塞锁适用于不需要等待的场景，如果锁被占用，立即返回失败。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'a');
if ($handle !== false) {
    $wouldBlock = 0;
    if (flock($handle, LOCK_EX | LOCK_NB, $wouldBlock)) {
        // 获取锁成功
        fwrite($handle, "Entry\n");
        flock($handle, LOCK_UN);
    } else {
        // 锁被占用，不等待，直接返回
        if ($wouldBlock === 1) {
            echo "File is locked, skipping\n";
        }
    }
    fclose($handle);
}
```

## 常见问题

### 问题 1：文件锁的作用范围是什么？

**回答**：文件锁只在同一台服务器上的多个进程间有效，不能跨服务器。在分布式系统中，需要使用其他同步机制（如数据库锁、Redis 锁等）。

**示例**：

```php
<?php
declare(strict_types=1);

// 文件锁只在同一服务器上有效
// 服务器 A 的进程和服务器 B 的进程不能通过文件锁同步
$handle = fopen(__DIR__ . '/shared.txt', 'a');
if (flock($handle, LOCK_EX)) {
    // 只保护同一服务器上的并发访问
    fwrite($handle, "Entry\n");
    flock($handle, LOCK_UN);
}
fclose($handle);
```

### 问题 2：如何实现非阻塞锁？

**回答**：使用 `LOCK_NB` 标志与 `LOCK_SH` 或 `LOCK_EX` 组合。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'a');
if ($handle !== false) {
    $wouldBlock = 0;
    // 非阻塞排他锁
    if (flock($handle, LOCK_EX | LOCK_NB, $wouldBlock)) {
        // 获取锁成功
        fwrite($handle, "Entry\n");
        flock($handle, LOCK_UN);
    } else {
        if ($wouldBlock === 1) {
            echo "Lock is busy\n";
        }
    }
    fclose($handle);
}
```

### 问题 3：临时文件何时删除？

**回答**：
- `tmpfile()` 创建的临时文件：脚本结束时或文件句柄关闭时自动删除
- `tempnam()` 创建的临时文件：需要手动调用 `unlink()` 删除

**示例**：

```php
<?php
declare(strict_types=1);

// tmpfile()：自动删除
$handle1 = tmpfile();
fwrite($handle1, 'data');
fclose($handle1);  // 文件自动删除

// tempnam()：手动删除
$tempFile = tempnam(__DIR__ . '/tmp', 'upload_');
file_put_contents($tempFile, 'data');
unlink($tempFile);  // 手动删除
```

### 问题 4：如何确保锁被正确释放？

**回答**：使用 `try-finally` 确保锁总是被释放，即使发生异常。

**示例**：

```php
<?php
declare(strict_types=1);

function safeWriteWithLock(string $file, string $content): void
{
    $handle = fopen($file, 'a');
    if ($handle === false) {
        throw new RuntimeException("Cannot open file: {$file}");
    }
    
    try {
        if (flock($handle, LOCK_EX)) {
            try {
                // 可能抛出异常的操作
                fwrite($handle, $content);
            } finally {
                // 确保锁被释放
                flock($handle, LOCK_UN);
            }
        }
    } finally {
        // 确保文件被关闭
        fclose($handle);
    }
}
```

## 最佳实践

### 1. 写入操作使用排他锁

写入文件时使用排他锁（`LOCK_EX`），防止并发写入冲突。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'a');
if ($handle !== false) {
    try {
        if (flock($handle, LOCK_EX)) {
            fwrite($handle, "Entry\n");
            flock($handle, LOCK_UN);
        }
    } finally {
        fclose($handle);
    }
}
```

### 2. 读取操作使用共享锁

读取文件时使用共享锁（`LOCK_SH`），允许多个进程同时读取。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'r');
if ($handle !== false) {
    try {
        if (flock($handle, LOCK_SH)) {
            $content = fread($handle, 1024);
            echo $content;
            flock($handle, LOCK_UN);
        }
    } finally {
        fclose($handle);
    }
}
```

### 3. 使用 try-finally 确保锁释放

使用 `try-finally` 确保锁总是被释放，即使发生异常。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.txt', 'a');
if ($handle !== false) {
    try {
        if (flock($handle, LOCK_EX)) {
            try {
                fwrite($handle, "Entry\n");
            } finally {
                flock($handle, LOCK_UN);
            }
        }
    } finally {
        fclose($handle);
    }
}
```

### 4. 临时文件及时清理

使用 `tempnam()` 创建的临时文件要及时清理，避免占用磁盘空间。

**示例**：

```php
<?php
declare(strict_types=1);

function withTempFile(callable $callback): mixed
{
    $tempFile = tempnam(sys_get_temp_dir(), 'php_');
    if ($tempFile === false) {
        throw new RuntimeException('Cannot create temporary file');
    }
    
    try {
        return $callback($tempFile);
    } finally {
        if (file_exists($tempFile)) {
            unlink($tempFile);
        }
    }
}

// 使用
$result = withTempFile(function (string $tempFile) {
    file_put_contents($tempFile, 'data');
    return file_get_contents($tempFile);
});
```

### 5. 使用 file_put_contents() 的 LOCK_EX 标志

对于简单的写入操作，可以使用 `file_put_contents()` 的 `LOCK_EX` 标志。

**示例**：

```php
<?php
declare(strict_types=1);

// 使用 file_put_contents() 的 LOCK_EX 标志
file_put_contents(__DIR__ . '/log.txt', "Entry\n", FILE_APPEND | LOCK_EX);
```

## 对比分析

### LOCK_SH vs LOCK_EX

| 特性         | LOCK_SH（共享锁）              | LOCK_EX（排他锁）              |
|:-------------|:-------------------------------|:-------------------------------|
| **多个进程同时读取** | ✅ 允许                     | ❌ 不允许                     |
| **多个进程同时写入** | ❌ 不允许                     | ❌ 不允许                     |
| **适用场景** | 读取操作                       | 写入操作                       |

### tmpfile() vs tempnam()

| 特性         | tmpfile()                      | tempnam()                      |
|:-------------|:-------------------------------|:-------------------------------|
| **返回值**    | 文件句柄                       | 文件路径                       |
| **文件创建** | ✅ 自动创建                    | ❌ 只生成文件名                 |
| **自动删除** | ✅ 脚本结束或关闭时自动删除    | ❌ 需要手动删除                 |
| **使用场景** | 需要文件句柄的场景             | 需要文件路径的场景              |

## 练习任务

1. **文件锁管理器**：创建一个文件锁管理器类，封装文件锁的获取和释放，支持超时和重试机制。

2. **安全日志写入**：实现一个带文件锁的日志写入类，支持并发写入保护。

3. **非阻塞写入工具**：编写一个函数，实现非阻塞的文件写入，如果锁被占用则返回失败。

4. **临时文件处理工具**：创建一个工具类，使用临时文件处理上传数据，确保临时文件被正确清理。

5. **文件锁超时机制**：实现一个带超时的文件锁获取机制，如果超时则放弃获取锁。

## 相关章节

- **[4.2.2 文件写入](section-02-file-writing.md)**：了解文件写入的基础知识
- **[4.2.3 文件操作](section-03-file-operations.md)**：了解文件操作的相关内容
