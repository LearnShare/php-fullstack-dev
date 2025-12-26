# 4.2.8 文件信息

## 概述

获取文件信息是文件系统操作中的常见需求。本节介绍 PHP 中获取文件信息的方法，包括文件大小、修改时间、文件类型等，以及 SplFileInfo 类的使用，帮助零基础学员掌握文件信息获取操作。

**章节类型**：参考性章节

**主要内容**：
- 文件信息函数概述
- filesize() 函数（文件大小）
- filemtime()、filectime()、fileatime()（时间信息）
- filetype() 函数（文件类型）
- is_readable()、is_writable()（权限检查）
- SplFileInfo 类
- 完整示例

## 核心内容

### 文件大小

- filesize() 函数
- 大文件的处理
- 目录大小的计算

### 文件时间

- filemtime()：修改时间
- filectime()：创建时间
- fileatime()：访问时间
- 时间格式化

### 文件类型

- filetype() 函数
- mime_content_type() 函数
- finfo_file() 函数

### SplFileInfo 类

- 面向对象的文件信息获取
- 常用方法
- 与函数式方法的对比

## 基本用法

### 文件信息获取示例

```php
<?php
declare(strict_types=1);

// 文件大小
$size = filesize('file.txt');

// 修改时间
$mtime = filemtime('file.txt');
echo date('Y-m-d H:i:s', $mtime);

// 使用 SplFileInfo
$fileInfo = new SplFileInfo('file.txt');
echo $fileInfo->getSize();
echo $fileInfo->getMTime();
```

## 使用场景

- 文件管理界面
- 文件上传验证
- 缓存策略判断
- 文件监控系统

## 注意事项

- 文件不存在时的处理
- 符号链接的处理
- 大文件大小的表示
- 时间戳的时区问题

## 常见问题

- 如何获取目录大小？
- 文件时间戳如何格式化？
- 如何判断文件类型？
- SplFileInfo 的优势是什么？

## 最佳实践

- 使用 SplFileInfo 获取文件信息
- 检查文件存在性再获取信息
- 注意文件时间的时区
- 大文件大小使用合适的单位显示
