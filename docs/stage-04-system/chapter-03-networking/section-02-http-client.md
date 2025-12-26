# 4.3.2 HTTP 客户端编程

## 概述

HTTP 客户端编程是 Web 开发中的常见需求。本节介绍 PHP 中进行 HTTP 请求的方法，包括 file_get_contents()、cURL 扩展、Guzzle 库等，帮助零基础学员掌握 HTTP 客户端编程。

**章节类型**：工具性章节

**主要内容**：
- HTTP 请求方法概述
- file_get_contents() 进行 HTTP 请求
- cURL 扩展使用
- Guzzle HTTP 客户端库
- 请求头设置
- POST 请求处理
- 响应处理
- 完整示例

## 核心内容

### HTTP 请求方法

- GET 请求
- POST 请求
- PUT、DELETE 等请求
- 请求头设置

### file_get_contents()

- 使用 HTTP 上下文
- stream_context_create() 函数
- 简单 GET 请求

### cURL 扩展

- curl_init() 函数
- curl_setopt() 函数
- curl_exec() 函数
- 高级选项配置

### Guzzle 库

- 库的安装
- 基本使用
- 请求和响应对象
- 异步请求

## 基本用法

### file_get_contents() 示例

```php
<?php
declare(strict_types=1);

// 简单 GET 请求
$content = file_get_contents('http://example.com/api');

// 带上下文的请求
$context = stream_context_create([
    'http' => [
        'method' => 'POST',
        'header' => 'Content-Type: application/json',
        'content' => json_encode(['key' => 'value'])
    ]
]);
$result = file_get_contents('http://example.com/api', false, $context);
```

### cURL 示例

```php
<?php
declare(strict_types=1);

$ch = curl_init('http://example.com/api');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$response = curl_exec($ch);
curl_close($ch);
```

## 使用场景

- API 调用
- 数据抓取
- 服务间通信
- 第三方服务集成

## 注意事项

- 错误处理
- 超时设置
- SSL 证书验证
- 请求头安全

## 常见问题

- file_get_contents() 和 cURL 的区别？
- 如何发送 POST 请求？
- 如何处理 HTTPS 请求？
- 如何设置请求超时？

## 最佳实践

- 优先使用 cURL 进行复杂请求
- 设置合理的超时时间
- 验证 SSL 证书
- 使用 Guzzle 简化操作
