# 4.2.7 目录操作

## 概述

目录操作是文件系统管理的重要组成部分。本节介绍 PHP 中目录的创建、删除、遍历等操作方法，包括 mkdir()、rmdir()、glob()、DirectoryIterator 等，帮助零基础学员掌握目录管理操作。

**章节类型**：语法性章节

**主要内容**：
- 目录操作函数概述
- mkdir() 函数（创建目录）
- rmdir() 函数（删除目录）
- 目录遍历方法
- glob() 函数
- DirectoryIterator 类
- RecursiveDirectoryIterator 类
- 完整示例

## 核心内容

### 目录创建

- mkdir() 函数的语法和参数
- 递归创建目录
- 目录权限设置

### 目录删除

- rmdir() 函数的用法
- 删除非空目录的方法
- 递归删除目录

### 目录遍历

- opendir()/readdir()/closedir()
- scandir() 函数
- glob() 函数（模式匹配）
- DirectoryIterator 类
- RecursiveDirectoryIterator 类

### 目录信息

- is_dir() 函数
- 目录权限检查
- 目录大小计算

## 基本用法

### 目录创建示例

```php
<?php
declare(strict_types=1);

// 创建目录
mkdir('new_directory', 0755, true); // 递归创建
```

### 目录遍历示例

```php
<?php
declare(strict_types=1);

// 使用 glob() 遍历
$files = glob('directory/*.txt');

// 使用 DirectoryIterator
$iterator = new DirectoryIterator('directory');
foreach ($iterator as $file) {
    if (!$file->isDot()) {
        echo $file->getFilename() . "\n";
    }
}
```

## 使用场景

- 文件管理系统
- 目录结构创建
- 文件搜索和过滤
- 批量文件处理

## 注意事项

- 检查目录是否存在
- 注意目录权限
- 递归操作的性能问题
- 隐藏文件的处理

## 常见问题

- 如何递归创建目录？
- 如何删除非空目录？
- glob() 和 DirectoryIterator 的区别？
- 如何遍历子目录？

## 最佳实践

- 使用递归标志创建目录
- 使用迭代器遍历大目录
- 检查目录权限
- 处理特殊文件（. 和 ..）
