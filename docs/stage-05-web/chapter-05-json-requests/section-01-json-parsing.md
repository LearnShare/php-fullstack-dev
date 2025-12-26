# 5.5.1 JSON 请求解析

## 概述

JSON 请求解析是 API 开发的基础。本节介绍如何解析 JSON 格式的请求数据，包括 json_decode() 函数的使用、请求体读取、错误处理等，帮助零基础学员掌握 JSON 数据处理。

**章节类型**：语法性章节

**主要内容**：
- JSON 格式概述
- json_decode() 函数
- 请求体读取（php://input）
- JSON 解析选项
- 错误处理
- 数据验证
- 完整示例

## 核心内容

### JSON 格式概述

- JSON 语法规则
- JSON 数据类型
- JSON 与数组/对象的对应关系

### json_decode() 函数

- 语法和参数
- 返回值类型
- 关联数组模式
- 深度限制

### 请求体读取

- php://input 流
- 读取 POST 数据
- 一次性读取
- 大小限制

### 错误处理

- JSON 解析错误
- json_last_error() 函数
- 错误类型判断
- 错误信息获取

## 基本用法

### JSON 解析示例

```php
<?php
declare(strict_types=1);

// 读取请求体
$json = file_get_contents('php://input');

// 解析 JSON
$data = json_decode($json, true);

// 检查错误
if (json_last_error() !== JSON_ERROR_NONE) {
    throw new InvalidArgumentException('Invalid JSON');
}
```

## 使用场景

- API 请求处理
- 数据交换
- 配置读取
- 数据存储

## 注意事项

- JSON 格式验证
- 深度限制
- 内存使用
- 错误处理

## 常见问题

- 如何读取 JSON 请求体？
- json_decode() 的参数含义？
- 如何检查 JSON 解析错误？
- 如何处理大 JSON 数据？

## 最佳实践

- 始终验证 JSON 格式
- 使用关联数组模式
- 检查解析错误
- 限制 JSON 深度和大小
