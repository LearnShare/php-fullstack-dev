# 5.7.1 RESTful 设计原则

## 概述

RESTful API 设计遵循 REST 架构风格的原则。本节介绍 REST 的核心概念和设计原则，包括资源设计、HTTP 方法使用、状态码使用等，帮助零基础学员理解 RESTful API 设计。

**章节类型**：概念性章节

**主要内容**：
- REST 概念
- 资源设计原则
- HTTP 方法使用（GET、POST、PUT、DELETE、PATCH）
- HTTP 状态码使用
- 无状态设计
- 统一接口
- 完整示例

## 核心内容

### REST 概念

- 什么是 REST
- REST 的约束条件
- RESTful 的优势

### 资源设计

- 资源的概念
- 资源命名规范
- 资源层级设计
- 资源标识（URI）

### HTTP 方法

- GET：获取资源
- POST：创建资源
- PUT：更新资源
- DELETE：删除资源
- PATCH：部分更新

### HTTP 状态码

- 2xx：成功
- 4xx：客户端错误
- 5xx：服务器错误
- 状态码选择

## 基本用法

### RESTful API 示例

```php
<?php
declare(strict_types=1);

// GET /api/users - 获取用户列表
// POST /api/users - 创建用户
// GET /api/users/1 - 获取用户
// PUT /api/users/1 - 更新用户
// DELETE /api/users/1 - 删除用户
```

## 使用场景

- Web API 设计
- 微服务接口
- 前后端分离
- 移动应用后端

## 注意事项

- 遵循 REST 原则
- 正确使用 HTTP 方法
- 合理使用状态码
- 保持无状态

## 常见问题

- 什么是 RESTful API？
- 如何设计资源 URI？
- 如何选择 HTTP 方法？
- 如何选择状态码？

## 最佳实践

- 使用名词表示资源
- 正确使用 HTTP 方法
- 使用标准状态码
- 保持 API 一致性
