# 5.12.3 错误处理与重试

## 概述

错误处理和重试机制是构建健壮 HTTP 客户端的关键。本节介绍 Guzzle 中的错误处理方法、异常类型、重试机制等，帮助零基础学员实现可靠的 HTTP 客户端。

**章节类型**：最佳实践章节

**主要内容**：
- 错误处理概述
- Guzzle 异常类型
- 异常处理
- 重试机制
- 重试策略
- 指数退避
- 完整示例

## 核心内容

### 错误处理

- 异常类型
- 异常捕获
- 错误分类
- 错误响应处理

### Guzzle 异常

- RequestException
- ClientException（4xx）
- ServerException（5xx）
- ConnectException

### 重试机制

- 重试条件
- 重试次数
- 重试间隔
- 重试策略

### 重试策略

- 固定间隔重试
- 指数退避
- 条件重试
- 最大重试次数

## 基本用法

### 错误处理和重试示例

```php
<?php
declare(strict_types=1);

use GuzzleHttp\Client;
use GuzzleHttp\HandlerStack;
use GuzzleHttp\Middleware;

$stack = HandlerStack::create();
$stack->push(Middleware::retry(function($retries, $request, $response, $exception) {
    return $retries < 3 && ($exception || $response->getStatusCode() >= 500);
}));

$client = new Client(['handler' => $stack]);
```

## 使用场景

- 不稳定的网络环境
- 第三方服务调用
- 关键操作重试
- 提高可靠性

## 注意事项

- 重试条件设计
- 重试次数限制
- 幂等性考虑
- 性能影响

## 常见问题

- 如何处理 HTTP 错误？
- 如何实现重试机制？
- 如何选择重试策略？
- 重试的性能影响？

## 最佳实践

- 区分可重试和不可重试错误
- 实现指数退避
- 限制重试次数
- 记录重试日志
