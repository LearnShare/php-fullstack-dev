# 5.1.1 HTTP 请求响应流程

## 概述

HTTP 请求响应流程是 Web 应用的基础。本节介绍 HTTP 协议的基本概念、请求和响应的结构、完整的请求生命周期，帮助零基础学员理解 Web 交互的基本原理。

**章节类型**：概念性章节

**主要内容**：
- HTTP 协议概述
- HTTP 请求结构（请求行、请求头、请求体）
- HTTP 响应结构（状态行、响应头、响应体）
- HTTP 方法（GET、POST、PUT、DELETE 等）
- HTTP 状态码
- 请求生命周期
- 完整示例

## 核心内容

### HTTP 协议概述

- HTTP 协议的作用
- HTTP 版本（HTTP/1.1、HTTP/2、HTTP/3）
- 客户端-服务器模型

### HTTP 请求结构

- 请求行（方法、URI、协议版本）
- 请求头（Host、User-Agent、Content-Type 等）
- 请求体（POST、PUT 请求的数据）

### HTTP 响应结构

- 状态行（协议版本、状态码、状态文本）
- 响应头（Content-Type、Content-Length 等）
- 响应体（HTML、JSON 等数据）

### 请求生命周期

- 客户端发送请求
- 服务器处理请求
- 服务器返回响应
- 客户端处理响应

## 基本用法

### HTTP 请求示例

```http
GET /api/users HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: application/json
```

### HTTP 响应示例

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 123

{"status": "success", "data": [...]}
```

## 使用场景

- Web 应用开发
- API 开发
- 理解 Web 通信
- 调试网络问题

## 注意事项

- HTTP 是无状态协议
- 请求和响应的格式要求
- 状态码的含义
- 协议版本的选择

## 常见问题

- HTTP 请求的组成部分？
- HTTP 状态码的含义？
- GET 和 POST 的区别？
- 如何查看 HTTP 请求响应？

## 最佳实践

- 理解 HTTP 协议基础
- 使用合适的 HTTP 方法
- 正确设置响应头
- 遵循 HTTP 规范
