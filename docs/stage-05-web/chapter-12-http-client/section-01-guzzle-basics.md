# 5.12.1 Guzzle 基础

## 概述

Guzzle 是 PHP 最流行的 HTTP 客户端库。本节介绍 Guzzle 的基础使用方法，包括安装配置、基本请求、请求选项等，帮助零基础学员掌握 Guzzle 的使用。

**章节类型**：工具性章节

**主要内容**：
- Guzzle 概述
- 安装配置
- 基本请求（GET、POST、PUT、DELETE）
- 请求选项（headers、query、json、timeout）
- 响应处理
- 完整示例

## 核心内容

### Guzzle 概述

- Guzzle 的功能
- Guzzle 的优势
- 使用场景

### 基本请求

- GET 请求
- POST 请求
- PUT 请求
- DELETE 请求

### 请求选项

- headers：请求头
- query：查询参数
- json：JSON 数据
- timeout：超时设置

### 响应处理

- 响应对象
- 状态码获取
- 响应体获取
- 响应头获取

## 基本用法

### Guzzle 使用示例

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;

$client = new Client();
$response = $client->get('http://api.example.com/users', [
    'headers' => ['Authorization' => 'Bearer token'],
    'query' => ['page' => 1]
]);

$data = json_decode($response->getBody()->getContents(), true);
```

## 使用场景

- API 调用
- 服务间通信
- 数据抓取
- 第三方服务集成

## 注意事项

- 错误处理
- 超时设置
- 请求重试
- 资源释放

## 常见问题

- 如何安装 Guzzle？
- 如何发送 POST 请求？
- 如何设置请求头？
- 如何处理响应？

## 最佳实践

- 使用 Client 实例复用
- 设置合理的超时时间
- 实现错误处理
- 使用请求选项简化配置
