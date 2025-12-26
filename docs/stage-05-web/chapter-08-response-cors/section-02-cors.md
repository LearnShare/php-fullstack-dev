# 5.8.2 CORS 跨域处理

## 概述

CORS（跨域资源共享）是处理跨域请求的机制。本节介绍 CORS 的概念、同源策略、CORS 请求流程、响应头设置等，帮助零基础学员理解并实现 CORS 跨域处理。

**章节类型**：概念性章节

**主要内容**：
- CORS 概念
- 同源策略
- 跨域请求类型
- CORS 响应头（Access-Control-Allow-Origin 等）
- 预检请求（OPTIONS）
- CORS 配置
- 完整示例

## 核心内容

### CORS 概念

- 什么是 CORS
- 为什么需要 CORS
- 同源策略限制

### 同源策略

- 同源的定义
- 跨域的场景
- 安全考虑

### CORS 响应头

- Access-Control-Allow-Origin
- Access-Control-Allow-Methods
- Access-Control-Allow-Headers
- Access-Control-Allow-Credentials

### 预检请求

- 什么是预检请求
- 何时发送预检请求
- 预检请求处理
- OPTIONS 方法处理

## 基本用法

### CORS 配置示例

```php
<?php
declare(strict_types=1);

// 处理预检请求
if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    header('Access-Control-Allow-Origin: *');
    header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE');
    header('Access-Control-Allow-Headers: Content-Type');
    exit;
}

// 设置 CORS 响应头
header('Access-Control-Allow-Origin: *');
header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE');
```

## 使用场景

- 前后端分离
- API 跨域访问
- 移动应用后端
- 第三方集成

## 注意事项

- 安全性考虑
- 通配符的使用
- 凭据传递
- 预检请求处理

## 常见问题

- 什么是 CORS？
- 如何配置 CORS？
- 预检请求是什么？
- 如何安全地配置 CORS？

## 最佳实践

- 明确指定允许的源
- 限制允许的方法和头部
- 正确处理预检请求
- 考虑安全性
