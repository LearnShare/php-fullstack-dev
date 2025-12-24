# 5.8.2 文件写入

## 概述

文件写入是 PHP 开发中的常见任务。PHP 提供了多种文件写入方法，每种方法适用于不同的场景。理解不同写入方法的工作原理、适用场景，以及如何安全高效地写入文件，对于构建健壮的 PHP 应用至关重要。

## 写入方法的选择

PHP 提供了多种文件写入方法：

1. **`file_put_contents()`**：一次性写入整个文件，适合小文件
2. **`fopen()` + `fwrite()`**：流式写入，适合大文件或需要多次写入
3. **`fopen()` + `fputcsv()`**：写入 CSV 文件

## file_put_contents() - 写入整个文件

**适用场景**：
- 写入配置文件
- 写入小文件（< 10MB）
- 需要一次性写入全部内容

**语法**：`file_put_contents(string $filename, mixed $data, int $flags = 0, ?resource $context = null): int|false`

**参数详解**：
- `$filename`：要写入的文件路径
- `$data`：要写入的数据
  - 字符串：直接写入
  - 数组：自动转换为字符串（使用 `implode()`）
- `$flags`：写入标志（可以使用 `|` 组合）
  - `FILE_APPEND`：追加到文件末尾，而不是覆盖
  - `FILE_USE_INCLUDE_PATH`：在 `include_path` 中搜索文件
  - `LOCK_EX`：获取排他锁（防止并发写入）
- `$context`：流上下文资源

**返回值**：成功返回写入的字节数，失败返回 `false`。

**工作原理**：
1. 打开文件（如果使用 `LOCK_EX`，会获取排他锁）
2. 写入数据
3. 关闭文件
4. 返回写入的字节数

**示例**：

```php
<?php
declare(strict_types=1);

// 覆盖写入
$bytes = file_put_contents(__DIR__ . '/output.txt', 'Hello, World!');
if ($bytes === false) {
    throw new RuntimeException('Cannot write file');
}

// 追加写入
file_put_contents(__DIR__ . '/output.txt', "\nNew line", FILE_APPEND);

// 追加并加锁（防止并发写入）
file_put_contents(__DIR__ . '/log.txt', "Log entry\n", FILE_APPEND | LOCK_EX);

// 写入数组（会自动转换为字符串）
$data = ['Line 1', 'Line 2', 'Line 3'];
file_put_contents(__DIR__ . '/output.txt', $data);
// 等同于：file_put_contents(__DIR__ . '/output.txt', implode('', $data));
```

**注意事项**：
- 默认会覆盖文件内容，使用 `FILE_APPEND` 可以追加
- 使用 `LOCK_EX` 可以防止并发写入导致的数据损坏
- 写入失败时返回 `false`，需要检查返回值

## fwrite() - 写入数据

**语法**：`fwrite(resource $stream, string $data, ?int $length = null): int|false`

**参数**：
- `$stream`：文件句柄
- `$data`：要写入的数据
- `$length`：可选，写入的最大字节数，默认写入全部数据

**返回值**：成功返回写入的字节数，失败返回 `false`。

**工作原理**：
- 从当前文件指针位置写入数据
- 写入后文件指针会移动到写入数据的末尾
- 如果指定了 `$length`，只写入指定长度的数据

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/output.txt', 'w');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// 写入数据
$bytes = fwrite($handle, 'Line 1\n');
$bytes = fwrite($handle, 'Line 2\n');

// 限制写入长度
$data = 'Hello, World!';
fwrite($handle, $data, 5);  // 只写入 "Hello"

fclose($handle);
```

## fputcsv() - 写入 CSV 文件

**语法**：`fputcsv(resource $stream, array $fields, string $separator = ",", string $enclosure = "\"", string $escape = "\\"): int|false`

**参数**：
- `$stream`：文件句柄
- `$fields`：要写入的字段数组
- `$separator`：字段分隔符，默认为 `,`
- `$enclosure`：字段包围符，默认为 `"`
- `$escape`：转义字符，默认为 `\`

**返回值**：成功返回写入的字节数，失败返回 `false`。

**示例**：

```php
<?php
declare(strict_types=1);

$handle = fopen(__DIR__ . '/data.csv', 'w');
if ($handle === false) {
    throw new RuntimeException('Cannot open file');
}

// 写入标题行
fputcsv($handle, ['Name', 'Age', 'Email']);

// 写入数据行
fputcsv($handle, ['Alice', 25, 'alice@example.com']);
fputcsv($handle, ['Bob', 30, 'bob@example.com']);

fclose($handle);
```

## 注意事项

1. **错误处理**：始终检查文件操作的返回值。
2. **文件锁**：在多进程环境中，使用 `LOCK_EX` 防止并发写入。
3. **资源释放**：使用 `fopen()` 后必须使用 `fclose()` 释放资源。
4. **原子写入**：对于关键数据，先写入临时文件，再重命名，确保原子性。

## 练习

1. 编写一个函数，安全地写入文件（包含错误处理和文件锁）。
2. 实现一个 CSV 文件写入器，将数据写入 CSV 文件。
3. 创建一个日志记录功能，支持追加写入和文件锁。
4. 编写一个函数，实现文件的原子写入（先写入临时文件，再重命名）。
