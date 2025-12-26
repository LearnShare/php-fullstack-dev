# 5.8.1 响应处理

## 概述

响应处理是 Web 应用返回数据给客户端的过程。本节介绍 HTTP 响应的处理方法，包括响应头设置、状态码设置、响应体格式等，帮助零基础学员掌握响应处理技术。

**章节类型**：语法性章节

**主要内容**：
- HTTP 响应概述
- header() 函数
- http_response_code() 函数
- 响应头设置（Content-Type、Content-Length 等）
- JSON 响应
- 响应格式统一
- 完整示例

## 核心内容

### HTTP 响应概述

- 响应结构
- 响应头作用
- 状态码作用
- 响应体内容

### header() 函数

- 语法和参数
- 响应头设置
- 设置时机
- 多次设置

### http_response_code()

- 状态码设置
- 状态码获取
- 标准状态码

### 响应格式

- JSON 响应
- XML 响应
- HTML 响应
- 格式选择

## 基本用法

### 响应处理示例

```php
<?php
declare(strict_types=1);

// 设置响应头
header('Content-Type: application/json');

// 设置状态码
http_response_code(200);

// 输出 JSON
echo json_encode(['status' => 'success', 'data' => $data]);
```

## 使用场景

- API 响应
- 错误响应
- 重定向响应
- 文件下载

## 注意事项

- header() 必须在输出前调用
- 状态码的正确使用
- 响应格式一致性
- 字符编码设置

## 常见问题

- 如何设置响应头？
- 如何设置状态码？
- header() 调用失败的原因？
- 如何统一响应格式？

## 最佳实践

- 在输出前设置响应头
- 使用标准状态码
- 统一响应格式
- 设置正确的 Content-Type
