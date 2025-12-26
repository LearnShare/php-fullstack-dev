# 4.2.12 路径处理

## 概述

路径处理是文件系统操作中的基础功能。本节介绍 PHP 中路径的规范化、拼接、解析等操作方法，包括 realpath()、pathinfo()、dirname()、basename() 等函数，帮助零基础学员掌握路径处理技巧。

**章节类型**：语法性章节

**主要内容**：
- 路径处理函数概述
- realpath() 函数（规范化路径）
- pathinfo() 函数（路径信息）
- dirname() 和 basename() 函数
- 路径拼接
- 跨平台路径处理
- 完整示例

## 核心内容

### 路径规范化

- realpath() 函数
- 解析符号链接
- 相对路径转绝对路径

### 路径信息获取

- pathinfo() 函数
- 获取目录名、文件名、扩展名
- 路径组件提取

### 路径操作

- dirname()：获取目录部分
- basename()：获取文件名部分
- 路径拼接方法

### 跨平台处理

- 目录分隔符（DIRECTORY_SEPARATOR）
- Windows 和 Unix 路径差异
- 路径标准化

## 基本用法

### 路径处理示例

```php
<?php
declare(strict_types=1);

// 规范化路径
$realPath = realpath('../data/file.txt');

// 获取路径信息
$info = pathinfo('/path/to/file.txt');
echo $info['dirname'];   // /path/to
echo $info['basename'];  // file.txt
echo $info['extension']; // txt

// 路径拼接
$path = dirname(__FILE__) . DIRECTORY_SEPARATOR . 'data.txt';
```

## 使用场景

- 配置文件路径处理
- 文件包含路径解析
- 相对路径转绝对路径
- 跨平台应用开发

## 注意事项

- 路径分隔符的跨平台处理
- 符号链接的解析
- 路径安全性检查
- 相对路径的基准

## 常见问题

- 如何规范化路径？
- 如何获取文件扩展名？
- 如何安全地拼接路径？
- Windows 和 Linux 路径如何处理？

## 最佳实践

- 使用 realpath() 规范化路径
- 使用 DIRECTORY_SEPARATOR 跨平台
- 检查路径安全性
- 使用 pathinfo() 提取路径组件
