# 5.3.1 $_GET、$_POST 与 $_REQUEST

## 概述

$_GET、$_POST 和 $_REQUEST 是获取 HTTP 请求数据的核心超全局变量。本节介绍这些超全局变量的使用方法、区别、数据获取和验证，帮助零基础学员掌握 Web 输入处理。

**章节类型**：语法性章节

**主要内容**：
- 超全局变量概述
- $_GET 超全局变量
- $_POST 超全局变量
- $_REQUEST 超全局变量
- 数据获取方法
- 数据验证和过滤
- 安全注意事项
- 完整示例

## 核心内容

### $_GET 超全局变量

- 获取 URL 参数
- 查询字符串解析
- 数据格式
- 使用场景

### $_POST 超全局变量

- 获取 POST 请求体数据
- 表单数据处理
- Content-Type 处理
- 使用场景

### $_REQUEST 超全局变量

- 包含 $_GET、$_POST、$_COOKIE
- 优先级问题
- 安全风险
- 使用建议

### 数据获取

- 直接访问
- 使用 filter_input()
- 数据验证
- 默认值处理

## 基本用法

### 数据获取示例

```php
<?php
declare(strict_types=1);

// 获取 GET 参数
$id = $_GET['id'] ?? null;

// 获取 POST 数据
$name = $_POST['name'] ?? '';

// 使用 filter_input 验证
$email = filter_input(INPUT_POST, 'email', FILTER_VALIDATE_EMAIL);
```

## 使用场景

- 表单数据处理
- URL 参数获取
- API 请求处理
- 数据验证

## 注意事项

- 数据验证和过滤
- XSS 防护
- SQL 注入防护
- 避免使用 $_REQUEST

## 常见问题

- $_GET 和 $_POST 的区别？
- 为什么避免使用 $_REQUEST？
- 如何安全地获取数据？
- 如何处理数组参数？

## 最佳实践

- 明确使用 $_GET 或 $_POST
- 始终验证和过滤输入
- 使用 filter_input() 函数
- 提供默认值
