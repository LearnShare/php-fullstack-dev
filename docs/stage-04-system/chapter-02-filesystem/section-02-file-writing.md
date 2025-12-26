# 4.2.2 文件写入

## 概述

文件写入是文件系统操作的重要功能。本节介绍 PHP 中写入文件的多种方法，包括 file_put_contents()、fopen()/fwrite()、追加写入等，帮助零基础学员掌握文件写入操作。

**章节类型**：语法性章节

**主要内容**：
- 文件写入方法概述
- file_put_contents() 函数
- fopen() 和 fwrite() 函数
- 追加写入模式
- 文件锁定写入
- 错误处理
- 完整示例

## 核心内容

### 文件写入方法

- file_put_contents()：一次性写入整个文件
- fopen()/fwrite()：按块写入文件
- 追加写入模式（'a'、'a+'）
- 覆盖写入模式（'w'、'w+'）

### file_put_contents() 函数

- 语法和参数
- 返回值
- 标志参数（FILE_APPEND、LOCK_EX）

### fopen() 和 fwrite() 函数

- 文件打开模式
- 写入操作
- 文件句柄管理

### 追加写入

- 追加模式的使用
- 文件指针位置
- 并发写入处理

## 基本用法

### file_put_contents() 示例

```php
<?php
declare(strict_types=1);

// 写入文件
file_put_contents('data.txt', 'Hello, World!');

// 追加写入
file_put_contents('data.txt', "\nNew line", FILE_APPEND);
```

### fopen()/fwrite() 示例

```php
<?php
declare(strict_types=1);

// 写入文件
$handle = fopen('data.txt', 'w');
fwrite($handle, 'Hello, World!');
fclose($handle);
```

## 使用场景

- 写入配置文件
- 写入日志文件
- 保存用户数据
- 生成报告文件

## 注意事项

- 检查目录写入权限
- 大文件应使用流式写入
- 注意文件编码问题
- 使用文件锁防止并发冲突

## 常见问题

- file_put_contents() 和 fopen()/fwrite() 的区别？
- 如何追加内容到文件？
- 写入文件时如何处理权限问题？
- 如何防止并发写入冲突？

## 最佳实践

- 小文件使用 file_put_contents()
- 大文件使用 fopen()/fwrite() 分块写入
- 使用文件锁保护并发写入
- 检查写入权限和磁盘空间
