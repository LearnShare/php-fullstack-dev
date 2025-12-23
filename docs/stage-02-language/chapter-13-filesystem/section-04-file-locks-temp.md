# 2.13.4 文件锁和临时文件

## 概述

文件锁用于防止多进程并发访问文件时的数据冲突，临时文件用于存储临时数据和处理上传文件。理解文件锁的工作原理和临时文件的使用场景，对于构建健壮的多进程 PHP 应用至关重要。

## 文件锁

### 为什么需要文件锁

在多进程或多线程环境中，多个进程可能同时访问同一个文件，导致：
- **数据损坏**：并发写入导致数据混乱
- **数据丢失**：一个进程的写入覆盖另一个进程的写入
- **读取错误**：读取时文件正在被写入

**文件锁**可以确保同一时间只有一个进程可以访问文件。

### flock() - 文件锁定

**语法**：`flock(resource $stream, int $operation, int &$would_block = null): bool`

**参数**：
- `$stream`：文件句柄（必须是可写模式）
- `$operation`：锁定操作
  - `LOCK_SH`：共享锁（读锁），多个进程可以同时读取
  - `LOCK_EX`：排他锁（写锁），只有一个进程可以写入
  - `LOCK_UN`：释放锁
  - `LOCK_NB`：非阻塞模式（可以与上述标志组合）
- `$would_block`：可选，如果锁被占用，此参数会被设置为 `1`

**返回值**：成功返回 `true`，失败返回 `false`。

**锁定类型**：

| 类型 | 说明 | 多个进程同时读取 | 多个进程同时写入 |
| :--- | :--- | :--- | :--- |
| `LOCK_SH` | 共享锁（读锁） | ✅ 允许 | ❌ 不允许 |
| `LOCK_EX` | 排他锁（写锁） | ❌ 不允许 | ❌ 不允许 |

**工作原理**：
1. 进程 A 获取锁后，其他进程必须等待
2. 进程 A 释放锁后，其他进程才能获取锁
3. 脚本结束时锁会自动释放

**示例**：

```php
<?php
declare(strict_types=1);

// 排他锁（写入时）
$handle = fopen(__DIR__ . '/data.txt', 'a');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// 获取排他锁（阻塞模式）
if (flock($handle, LOCK_EX)) {
    // 写入数据
    fwrite($handle, "New entry: " . date('Y-m-d H:i:s') . "\n");
    
    // 释放锁
    flock($handle, LOCK_UN);
} else {
    throw new RuntimeException('Cannot acquire lock');
}

fclose($handle);

// 非阻塞模式
$handle = fopen(__DIR__ . '/data.txt', 'a');
if (flock($handle, LOCK_EX | LOCK_NB, $wouldBlock)) {
    if ($wouldBlock) {
        // 锁被占用，稍后重试
        echo "File is locked, please try again later\n";
    } else {
        // 获取锁成功
        fwrite($handle, "New entry\n");
        flock($handle, LOCK_UN);
    }
}
fclose($handle);
```

**注意事项**：
- 文件锁只在同一台服务器上有效，不能跨服务器
- 使用 `LOCK_NB` 时，如果锁被占用，函数会立即返回 `false`
- 脚本异常退出时，锁会自动释放
- 在 Windows 上，必须先 `fopen()` 才能使用 `flock()`

## 临时文件

### 为什么需要临时文件

临时文件用于：
- 存储临时数据
- 处理上传文件
- 缓存数据
- 避免文件名冲突

### tmpfile() - 创建临时文件

**语法**：`tmpfile(): resource|false`

**返回值**：成功返回文件句柄，失败返回 `false`。

**特点**：
- 文件会在脚本结束时自动删除
- 文件句柄关闭时也会自动删除
- 文件存储在系统临时目录

**示例**：

```php
<?php
declare(strict_types=1);

// 创建临时文件
$handle = tmpfile();
if ($handle === false) {
    throw new RuntimeException('Cannot create temporary file');
}

// 写入数据
fwrite($handle, 'Temporary data');

// 读取数据
rewind($handle);
$content = fread($handle, 1024);

// 关闭文件（会自动删除）
fclose($handle);
```

### tempnam() - 生成临时文件名

**语法**：`tempnam(string $directory, string $prefix): string|false`

**参数**：
- `$directory`：临时文件目录（如果为 `null`，使用系统临时目录）
- `$prefix`：文件名前缀

**返回值**：成功返回临时文件路径，失败返回 `false`。

**特点**：
- 只生成文件名，不创建文件
- 文件名是唯一的
- 需要手动删除文件

**示例**：

```php
<?php
declare(strict_types=1);

// 在指定目录创建临时文件
$tempFile = tempnam(__DIR__ . '/tmp', 'upload_');
if ($tempFile === false) {
    throw new RuntimeException('Cannot create temporary file name');
}

// 使用临时文件
file_put_contents($tempFile, 'Temporary data');

// 使用完毕后删除
unlink($tempFile);
```

### sys_get_temp_dir() - 获取系统临时目录

**语法**：`sys_get_temp_dir(): string`

**返回值**：返回系统临时目录路径。

**示例**：

```php
<?php
declare(strict_types=1);

$tempDir = sys_get_temp_dir();
echo "Temporary directory: {$tempDir}\n";

// 在系统临时目录创建临时文件
$tempFile = tempnam($tempDir, 'php_');
```

## 完整示例

```php
<?php
declare(strict_types=1);

class FileLockManager
{
    /**
     * 安全写入文件（带锁）
     */
    public static function safeWrite(string $file, string $content, bool $append = false): void
    {
        $mode = $append ? 'a' : 'w';
        $handle = fopen($file, $mode);
        
        if ($handle === false) {
            throw new RuntimeException("Cannot open file: {$file}");
        }
        
        // 获取排他锁
        if (!flock($handle, LOCK_EX)) {
            fclose($handle);
            throw new RuntimeException("Cannot acquire lock: {$file}");
        }
        
        try {
            // 写入数据
            fwrite($handle, $content);
        } finally {
            // 释放锁
            flock($handle, LOCK_UN);
            fclose($handle);
        }
    }
    
    /**
     * 非阻塞写入
     */
    public static function tryWrite(string $file, string $content): bool
    {
        $handle = fopen($file, 'a');
        if ($handle === false) {
            return false;
        }
        
        // 尝试获取锁（非阻塞）
        if (flock($handle, LOCK_EX | LOCK_NB, $wouldBlock)) {
            if ($wouldBlock) {
                fclose($handle);
                return false;  // 锁被占用
            }
            
            fwrite($handle, $content);
            flock($handle, LOCK_UN);
            fclose($handle);
            return true;
        }
        
        fclose($handle);
        return false;
    }
}

// 使用示例
try {
    // 安全写入（带锁）
    FileLockManager::safeWrite(__DIR__ . '/log.txt', "Log entry\n", true);
    
    // 尝试写入（非阻塞）
    if (!FileLockManager::tryWrite(__DIR__ . '/log.txt', "Entry\n")) {
        echo "File is locked, skipping write\n";
    }
} catch (RuntimeException $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

## 注意事项

1. **文件锁范围**：文件锁只在同一台服务器上有效。
2. **锁的释放**：确保在异常情况下也能释放锁（使用 try-finally）。
3. **临时文件清理**：使用 `tempnam()` 创建的临时文件需要手动删除。
4. **性能考虑**：文件锁会影响并发性能，只在必要时使用。

## 练习

1. 创建一个带文件锁的日志写入类。
2. 实现一个非阻塞的文件写入功能。
3. 编写一个函数，使用临时文件处理上传数据。
4. 创建一个文件锁管理器，支持超时和重试机制。
