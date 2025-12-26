# 5.4.1 URL 解析与构建

## 概述

URL 解析是将 URL 分解为各个组成部分的过程。本节介绍 PHP 中 URL 解析和构建的方法，包括 parse_url() 函数的使用，帮助零基础学员掌握 URL 处理技术。

**章节类型**：语法性章节

**主要内容**：
- URL 结构概述
- parse_url() 函数
- URL 组件获取（scheme、host、path、query 等）
- URL 构建方法
- 相对 URL 和绝对 URL
- 完整示例

## 核心内容

### URL 结构

- URL 的组成部分
- scheme（协议）
- host（主机）
- path（路径）
- query（查询字符串）
- fragment（片段）

### parse_url() 函数

- 语法和参数
- 返回值结构
- 组件访问
- 错误处理

### URL 构建

- 手动构建 URL
- 使用组件构建
- 查询字符串添加
- 路径拼接

## 基本用法

### URL 解析示例

```php
<?php
declare(strict_types=1);

$url = 'https://example.com/path?key=value#fragment';
$parts = parse_url($url);

echo $parts['scheme'];  // https
echo $parts['host'];    // example.com
echo $parts['path'];    // /path
echo $parts['query'];   // key=value
```

## 使用场景

- URL 验证
- URL 重定向
- API 调用
- 链接生成

## 注意事项

- URL 格式验证
- 组件缺失处理
- 编码问题
- 安全性检查

## 常见问题

- parse_url() 返回什么？
- 如何构建 URL？
- 如何处理相对 URL？
- URL 解析失败怎么办？

## 最佳实践

- 验证 URL 格式
- 处理缺失组件
- 使用安全的 URL 构建方法
- 注意编码问题
