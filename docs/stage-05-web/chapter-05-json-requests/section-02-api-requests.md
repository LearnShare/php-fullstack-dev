# 5.5.2 API 请求处理

## 概述

API 请求处理是构建 API 接口的核心。本节介绍 API 请求的处理方法，包括 Content-Type 检查、请求验证、响应格式等，帮助零基础学员掌握 API 开发技术。

**章节类型**：实践性章节

**主要内容**：
- API 请求概述
- Content-Type 处理
- 请求方法检查
- 请求验证
- 响应格式
- 错误处理
- 完整示例

## 核心内容

### API 请求概述

- 什么是 API
- API 请求的特点
- RESTful API 概念

### Content-Type 处理

- Content-Type 检查
- application/json
- application/x-www-form-urlencoded
- multipart/form-data

### 请求验证

- 请求方法验证
- 数据格式验证
- 必填字段检查
- 数据类型验证

### 响应格式

- JSON 响应
- 状态码设置
- 响应结构
- 错误响应

## 基本用法

### API 请求处理示例

```php
<?php
declare(strict_types=1);

// 检查请求方法
if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    http_response_code(405);
    exit;
}

// 检查 Content-Type
$contentType = $_SERVER['CONTENT_TYPE'] ?? '';
if (strpos($contentType, 'application/json') === false) {
    http_response_code(400);
    exit;
}

// 解析请求
$json = file_get_contents('php://input');
$data = json_decode($json, true);
```

## 使用场景

- RESTful API 开发
- 微服务通信
- 前后端分离
- 移动应用后端

## 注意事项

- 请求方法验证
- Content-Type 检查
- 数据验证
- 错误处理

## 常见问题

- 如何检查请求方法？
- 如何验证 Content-Type？
- 如何统一 API 响应格式？
- 如何处理 API 错误？

## 最佳实践

- 验证请求方法和 Content-Type
- 统一响应格式
- 提供清晰的错误信息
- 实现请求验证中间件
