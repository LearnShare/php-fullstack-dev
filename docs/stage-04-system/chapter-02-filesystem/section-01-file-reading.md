# 4.2.1 文件读取

## 概述

文件读取是文件系统操作的基础功能。本节介绍 PHP 中读取文件的多种方法，包括 file_get_contents()、fopen()/fread()、逐行读取等，帮助零基础学员掌握文件读取操作。

**章节类型**：语法性章节

**主要内容**：
- 文件读取方法概述
- file_get_contents() 函数
- fopen() 和 fread() 函数
- 逐行读取（fgets()、file()）
- 读取整个文件到数组
- 错误处理
- 完整示例

## 核心内容

### 文件读取方法

- file_get_contents()：一次性读取整个文件
- fopen()/fread()：按块读取文件
- fgets()：逐行读取文件
- file()：读取文件到数组

### file_get_contents() 函数

- 语法和参数
- 返回值
- 使用场景和限制

### fopen() 和 fread() 函数

- 文件打开模式
- 读取操作
- 文件句柄管理

### 逐行读取

- fgets() 函数
- fgetcsv() 函数（CSV 文件）
- 文件指针管理

## 基本用法

### file_get_contents() 示例

```php
<?php
declare(strict_types=1);

// 读取整个文件
$content = file_get_contents('data.txt');
```

### fopen()/fread() 示例

```php
<?php
declare(strict_types=1);

// 按块读取文件
$handle = fopen('data.txt', 'r');
$content = fread($handle, filesize('data.txt'));
fclose($handle);
```

## 使用场景

- 读取配置文件
- 读取日志文件
- 读取用户上传的文件内容
- 处理文本数据

## 注意事项

- 大文件应使用流式读取
- 注意文件编码问题
- 检查文件是否存在
- 及时关闭文件句柄

## 常见问题

- file_get_contents() 和 fopen()/fread() 的区别？
- 如何逐行读取大文件？
- 读取文件时如何处理编码问题？
- 文件不存在时如何处理？

## 最佳实践

- 小文件使用 file_get_contents()
- 大文件使用 fopen()/fread() 分块读取
- 始终检查文件是否存在
- 使用 try-catch 处理异常
