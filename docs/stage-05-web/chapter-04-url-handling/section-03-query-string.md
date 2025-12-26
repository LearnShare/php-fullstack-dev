# 5.4.3 查询字符串处理

## 概述

查询字符串是 URL 中传递参数的方式。本节介绍 PHP 中查询字符串的处理方法，包括 http_build_query()、parse_str() 等函数，帮助零基础学员掌握查询字符串的构建和解析。

**章节类型**：语法性章节

**主要内容**：
- 查询字符串概念
- http_build_query() 函数
- parse_str() 函数
- 参数构建
- 参数解析
- 数组参数处理
- 完整示例

## 核心内容

### 查询字符串概念

- 查询字符串的格式
- 参数格式（key=value）
- 多个参数（& 分隔）
- 特殊字符处理

### http_build_query() 函数

- 语法和参数
- 数组转查询字符串
- 嵌套数组处理
- 编码选项

### parse_str() 函数

- 语法和参数
- 查询字符串解析
- 数组参数解析
- 变量创建

## 基本用法

### 查询字符串构建示例

```php
<?php
declare(strict_types=1);

// 构建查询字符串
$params = ['name' => 'John', 'age' => 30];
$query = http_build_query($params); // name=John&age=30

// 解析查询字符串
parse_str($query, $result);
print_r($result);
```

## 使用场景

- URL 参数构建
- 表单数据提交
- API 参数传递
- 分页参数处理

## 注意事项

- 参数编码
- 数组参数格式
- 特殊字符处理
- 安全性考虑

## 常见问题

- 如何构建查询字符串？
- 如何解析查询字符串？
- 如何处理数组参数？
- parse_str() 的安全问题？

## 最佳实践

- 使用 http_build_query() 构建
- 使用 parse_str() 解析到数组
- 注意参数编码
- 验证解析结果
