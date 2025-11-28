# 2.14 文件系统操作

## 目标

- 掌握常见文件读写、目录遍历、路径处理方法。
- 理解流式读取/写入、文件锁、临时文件等后台任务必备技术。
- 知道如何在不同平台处理路径与权限问题。

## 路径与常量

- `__DIR__`：当前文件所在目录。
- `dirname(__DIR__)`：向上一级。
- `DIRECTORY_SEPARATOR`：平台相关分隔符（Windows 为 `\`，类 Unix 为 `/`）。
- `realpath($path)`：获取绝对路径。

## 文件读取

```php
$contents = file_get_contents(__DIR__ . '/data.json');
```

- 大文件建议使用流式方式：

```php
$handle = fopen($path, 'rb');
while (!feof($handle)) {
    $chunk = fread($handle, 8192);
    // 处理数据
}
fclose($handle);
```

## 文件写入

- 覆盖写入：`file_put_contents($path, $data);`
- 追加写入：`file_put_contents($path, $data, FILE_APPEND);`
- 流式写入：

```php
$handle = fopen($path, 'ab');
fwrite($handle, $data);
fclose($handle);
```

## 文件锁

- 使用 `flock($handle, LOCK_EX)` 防止并发写入冲突。

```php
$handle = fopen($path, 'cb');
if (flock($handle, LOCK_EX)) {
    fwrite($handle, $data);
    flock($handle, LOCK_UN);
}
fclose($handle);
```

## 目录操作

- 判断：`is_dir($path)`、`is_file($path)`、`file_exists($path)`。
- 创建：`mkdir($path, 0775, true);`
- 遍历：`scandir()` 或 SPL 迭代器 `RecursiveDirectoryIterator`.

## SPL 迭代器

```php
$iterator = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator($basePath)
);
foreach ($iterator as $file) {
    if ($file->isFile()) {
        // ...
    }
}
```

## 临时文件与目录

- `sys_get_temp_dir()` 获取临时目录。
- `tempnam($dir, $prefix)` 创建临时文件。
- 记得处理完毕后删除，或使用 `tmpfile()` 返回自动删除的资源。

## 上传文件存储

- 使用 `move_uploaded_file($_FILES['file']['tmp_name'], $destination)`.
- 生成随机文件名：`bin2hex(random_bytes(16))`.
- 将上传目录与公众目录分离，防止执行风险。

## 权限与安全

- 文件权限：类 Unix 使用 `chmod`，Windows 需注意 ACL。
- 禁止直接写入项目根目录，可配置 `storage/`、`runtime/` 等。
- 日志文件建议定期轮转并限制大小。

## 文件哈希与校验

- `hash_file('sha256', $path)`：生成校验值。
- `hash_equals($expected, $actual)`：安全比较，防止计时攻击。

## 练习

1. 编写脚本将 `storage/logs` 中超过 30 天的日志打包并删除。
2. 实现 `streamCsv(string $path, callable $callback)`，逐行读取 CSV 并传入回调。
3. 为上传功能添加哈希校验、文件锁与随机文件名策略，写成独立的 `FileStorage` 服务。
