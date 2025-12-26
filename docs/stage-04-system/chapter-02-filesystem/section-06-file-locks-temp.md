# 4.2.6 文件锁和临时文件

## 概述

文件锁用于防止并发访问冲突，临时文件用于存储临时数据。本节介绍 PHP 中文件锁机制和临时文件的创建方法，帮助零基础学员理解并发文件操作的处理。

**章节类型**：概念性章节

**主要内容**：
- 文件锁概念
- flock() 函数
- 锁的类型（共享锁、排他锁）
- 临时文件创建
- tmpfile() 和 tempnam() 函数
- 完整示例

## 核心内容

### 文件锁概念

- 为什么需要文件锁
- 并发访问的问题
- 锁的类型和作用

### flock() 函数

- 语法和参数
- LOCK_SH（共享锁）
- LOCK_EX（排他锁）
- LOCK_UN（释放锁）
- LOCK_NB（非阻塞）

### 临时文件

- tmpfile() 函数：创建临时文件
- tempnam() 函数：生成临时文件名
- sys_get_temp_dir() 函数
- 临时文件的清理

## 基本用法

### 文件锁示例

```php
<?php
declare(strict_types=1);

$handle = fopen('data.txt', 'r+');

// 获取排他锁
if (flock($handle, LOCK_EX)) {
    // 执行写入操作
    fwrite($handle, 'New data');
    // 释放锁
    flock($handle, LOCK_UN);
}
fclose($handle);
```

### 临时文件示例

```php
<?php
declare(strict_types=1);

// 创建临时文件
$temp = tmpfile();
fwrite($temp, 'Temporary data');
// 文件关闭时自动删除
fclose($temp);
```

## 使用场景

- 并发写入保护
- 配置文件更新
- 临时数据存储
- 缓存文件管理

## 注意事项

- 文件锁只在同一进程内有效
- 非阻塞锁的使用场景
- 临时文件的生命周期
- 锁的释放时机

## 常见问题

- 文件锁的作用范围？
- 如何实现非阻塞锁？
- 临时文件何时删除？
- 如何确保锁被正确释放？

## 最佳实践

- 写入操作使用排他锁
- 读取操作使用共享锁
- 使用 try-finally 确保锁释放
- 临时文件及时清理
