# 5.12.2 异步请求

## 概述

异步请求能够提高并发处理能力。本节介绍 Guzzle 中异步请求的使用方法，包括 Promise、并发请求等，帮助零基础学员掌握异步 HTTP 请求技术。

**章节类型**：语法性章节

**主要内容**：
- 异步请求概念
- Promise 概念
- 并发请求
- Promise 处理
- 错误处理
- 完整示例

## 核心内容

### 异步请求概念

- 什么是异步请求
- 异步请求的优势
- 使用场景

### Promise

- Promise 概念
- Promise 状态
- Promise 处理
- Promise 链

### 并发请求

- 多个请求并发
- Promise 数组
- 等待所有请求
- 部分成功处理

### 异步处理

- 非阻塞执行
- 回调处理
- 结果收集
- 性能优势

## 基本用法

### 异步请求示例

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client();
$promises = [
    'users' => $client->getAsync('/api/users'),
    'posts' => $client->getAsync('/api/posts')
];

$results = Promise\Utils::settle($promises)->wait();
```

## 使用场景

- 并发 API 调用
- 数据聚合
- 性能优化
- 批量处理

## 注意事项

- Promise 处理
- 错误处理
- 资源管理
- 超时控制

## 常见问题

- 如何发送异步请求？
- Promise 如何处理？
- 如何等待多个请求？
- 异步请求的性能优势？

## 最佳实践

- 使用异步请求提高并发
- 正确处理 Promise
- 实现错误处理
- 控制并发数量
