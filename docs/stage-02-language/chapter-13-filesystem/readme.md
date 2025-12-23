# 2.13 文件系统操作

## 目标

- 掌握常见文件读写、目录遍历、路径处理方法。
- 理解流式读取/写入、文件锁、临时文件等后台任务必备技术。
- 知道如何在不同平台处理路径与权限问题。

## 章节内容

本章分为六个独立小节，每节提供详细的概念解释、语法说明、参数列表和完整示例：

1. **[文件读取](section-01-file-reading.md)**：文件读取方法选择、`file_get_contents()`、`fopen()`/`fread()`、`fgets()`、`fgetcsv()`、`readfile()` 及完整示例。

2. **[文件写入](section-02-file-writing.md)**：文件写入方法选择、`file_put_contents()`、`fwrite()`、`fputcsv()` 及完整示例。

3. **[文件操作](section-03-file-operations.md)**：文件复制（`copy()`）、重命名/移动（`rename()`）、删除（`unlink()`）及完整示例。

4. **[文件锁和临时文件](section-04-file-locks-temp.md)**：文件锁（`flock()`）、临时文件（`tmpfile()`、`tempnam()`）、系统临时目录（`sys_get_temp_dir()`）及完整示例。

5. **[文件信息](section-05-file-info.md)**：文件存在性检查（`file_exists()`、`is_file()`、`is_dir()`）、文件大小（`filesize()`）、时间戳（`filemtime()`、`filectime()`、`fileatime()`）、权限检查（`is_readable()`、`is_writable()`、`is_executable()`）、详细信息（`stat()`）及完整示例。

6. **[目录操作](section-06-directory-operations.md)**：目录创建（`mkdir()`）、目录删除（`rmdir()`）、目录遍历（`scandir()`、`opendir()`/`readdir()`）、路径处理函数（`realpath()`、`dirname()`、`basename()`、`pathinfo()`）及完整示例。

## 路径与常量

- `__DIR__`：当前文件所在目录。
- `dirname(__DIR__)`：向上一级。
- `DIRECTORY_SEPARATOR`：平台相关分隔符（Windows 为 `\`，类 Unix 为 `/`）。
- `realpath($path)`：获取绝对路径。

## 学习建议

1. **按顺序学习**：按照章节顺序学习，先掌握文件读取和写入，再学习文件操作、文件锁和临时文件，最后学习文件信息和目录操作。

2. **重点掌握**：
   - 文件读写的基本方法和适用场景
   - 流式操作处理大文件
   - 文件锁的使用和临时文件的处理
   - 文件信息的获取和权限检查
   - 目录遍历和递归操作
   - 路径处理的最佳实践

3. **实践练习**：
   - 完成每小节后的练习题目
   - 实现实际的文件处理功能
   - 处理目录结构和文件组织
   - 实现带文件锁的日志系统

## 完成本章后

- 能够进行基本的文件读写操作。
- 理解不同文件操作方法的适用场景。
- 掌握文件锁的使用，防止并发冲突。
- 能够使用临时文件处理数据。
- 掌握文件信息的获取和权限检查。
- 掌握目录的创建、删除和遍历。
- 理解路径处理的最佳实践。
- 能够处理大文件和目录结构。

## 相关章节

- **2.11 超级全局变量**：了解 `$_FILES` 的处理。
- **阶段五：数据持久化与日志管理**：深入学习文件系统的高级特性。
