# 5.4.2 URL 编码与解码

## 概述

URL 编码是将特殊字符转换为 URL 安全格式的过程。本节介绍 PHP 中 URL 编码和解码的方法，包括 urlencode()、urldecode()、rawurlencode()、rawurldecode() 等函数，帮助零基础学员掌握 URL 编码技术。

**章节类型**：语法性章节

**主要内容**：
- URL 编码概念
- urlencode() 函数
- urldecode() 函数
- rawurlencode() 函数
- rawurldecode() 函数
- 编码规则
- 完整示例

## 核心内容

### URL 编码概念

- 为什么需要编码
- 需要编码的字符
- 编码规则
- 编码格式

### urlencode() 函数

- 语法和参数
- 编码规则
- 空格处理（+）
- 使用场景

### rawurlencode() 函数

- 与 urlencode() 的区别
- 编码规则
- 空格处理（%20）
- 使用场景

### 编码规则

- 保留字符
- 非保留字符
- 特殊字符处理
- 编码一致性

## 基本用法

### URL 编码示例

```php
<?php
declare(strict_types=1);

// URL 编码
$encoded = urlencode('hello world'); // hello+world
$rawEncoded = rawurlencode('hello world'); // hello%20world

// URL 解码
$decoded = urldecode('hello+world');
```

## 使用场景

- 查询字符串编码
- URL 路径编码
- 参数传递
- API 调用

## 注意事项

- 编码一致性
- 双重编码问题
- 字符集问题
- 编码位置

## 常见问题

- urlencode() 和 rawurlencode() 的区别？
- 什么时候需要编码？
- 如何避免双重编码？
- 编码后的 URL 如何解码？

## 最佳实践

- 查询参数使用 urlencode()
- URL 路径使用 rawurlencode()
- 避免双重编码
- 统一编码标准
