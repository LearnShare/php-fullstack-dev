# 5.16.3 错误中间件

## 概述

错误中间件用于统一处理应用错误。本节介绍错误中间件的实现方法，包括异常捕获、错误格式化等，帮助零基础学员实现统一的错误处理机制。

**章节类型**：实践性章节

**主要内容**：
- 错误中间件概念
- 异常捕获
- 错误格式化
- 错误响应
- 错误日志
- 完整示例

## 核心内容

### 错误中间件概念

- 什么是错误中间件
- 错误中间件的作用
- 错误中间件的优势

### 异常捕获

- 全局异常处理
- 异常类型判断
- 异常信息提取
- 异常堆栈跟踪

### 错误格式化

- 错误格式设计
- 开发环境格式
- 生产环境格式
- 格式转换

### 错误响应

- HTTP 状态码设置
- 响应体格式化
- 错误消息处理
- 错误详情控制

## 基本用法

### 错误中间件示例

```php
<?php
declare(strict_types=1);

class ErrorMiddleware {
    public function handle($request, $next) {
        try {
            return $next($request);
        } catch (Exception $e) {
            return $this->formatError($e);
        }
    }
    
    private function formatError(Exception $e): Response {
        $statusCode = $e->getCode() ?: 500;
        $message = getenv('APP_ENV') === 'production' 
            ? 'Internal Server Error' 
            : $e->getMessage();
        
        return new Response(json_encode(['error' => $message]), $statusCode);
    }
}
```

## 使用场景

- 统一错误处理
- 错误格式化
- 错误日志记录
- 错误响应控制

## 注意事项

- 异常捕获范围
- 错误信息安全性
- 性能影响
- 错误日志

## 常见问题

- 如何实现错误中间件？
- 如何格式化错误？
- 如何控制错误详情？
- 错误中间件的性能影响？

## 最佳实践

- 实现全局错误处理
- 区分开发和生产环境
- 记录详细错误日志
- 提供用户友好错误
