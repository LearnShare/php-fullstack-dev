# 11.2.5 API 接口设计

## 概述

API 接口设计是前后端分离架构的核心。本节介绍网站访问统计服务的 API 接口设计，包括 RESTful API 设计、统计 API 接口、数据分析 API 接口、管理 API 接口等，帮助零基础学员理解如何设计 API 接口。

**章节类型**：实践性章节

**主要内容**：
- API 设计概述
- API 设计原则
- API 响应格式
- 统计 API 接口
- 数据分析 API 接口
- 管理 API 接口
- API 认证
- API 限流
- 完整示例

## 核心内容

### API 设计概述

- API 设计的作用
- API-First 设计理念
- RESTful API 设计规范

### API 设计原则

- RESTful 规范
- 版本控制
- 统一响应格式
- 错误处理
- 认证授权
- 限流保护

### API 响应格式

#### 成功响应

- code: 200
- message: "success"
- data: 响应数据
- timestamp: 时间戳

#### 错误响应

- code: 错误码
- message: 错误消息
- error: 错误详情
- timestamp: 时间戳

### 统计 API 接口

#### 1. 上报统计数据

- 接口：`POST /api/v1/analytics/track`
- 功能：接收统计数据上报
- 请求头：Authorization、Content-Type
- 请求体：website_id、page_path、referrer、user_agent、ip、duration、events
- 响应：track_id

#### 2. 批量上报统计数据

- 接口：`POST /api/v1/analytics/batch`
- 功能：批量上报统计数据
- 请求体：tracks 数组

### 数据分析 API 接口

#### 1. 获取访问统计

- 接口：`GET /api/v1/analytics/stats`
- 查询参数：website_id、start_date、end_date、group_by、metrics
- 响应：summary（汇总数据）、series（时间序列数据）

#### 2. 获取来源统计

- 接口：`GET /api/v1/analytics/referrers`
- 查询参数：website_id、start_date、end_date、limit
- 响应：referrers 数组

#### 3. 获取设备统计

- 接口：`GET /api/v1/analytics/devices`
- 响应：devices、browsers、os 数组

#### 4. 获取地域统计

- 接口：`GET /api/v1/analytics/locations`
- 响应：countries、cities 数组

#### 5. 获取页面统计

- 接口：`GET /api/v1/analytics/pages`
- 响应：pages 数组（包含 path、pv、uv、avg_duration、bounce_rate）

#### 6. 获取事件统计

- 接口：`GET /api/v1/analytics/events`
- 响应：events 数组（包含 name、count、unique_users、conversion_rate）

#### 7. 获取实时数据

- 接口：`GET /api/v1/analytics/realtime`
- 响应：current_visitors、page_views_per_minute、top_pages

### 管理 API 接口

#### 1. 用户注册

- 接口：`POST /api/v1/auth/register`
- 请求体：username、email、password

#### 2. 用户登录

- 接口：`POST /api/v1/auth/login`
- 请求体：email、password
- 响应：token、user 信息

#### 3. 获取网站列表

- 接口：`GET /api/v1/websites`
- 响应：websites 数组

#### 4. 创建网站

- 接口：`POST /api/v1/websites`
- 请求体：name、domain

#### 5. 更新网站

- 接口：`PUT /api/v1/websites/{id}`
- 请求体：name、domain

#### 6. 删除网站

- 接口：`DELETE /api/v1/websites/{id}`

#### 7. 重新生成 API Key

- 接口：`POST /api/v1/websites/{id}/regenerate-key`

### API 认证

#### API Key 认证

- 统计 API 使用 API Key 认证
- 请求头：`Authorization: Bearer {api_key}`

#### JWT Token 认证

- 管理 API 使用 JWT Token 认证
- 请求头：`Authorization: Bearer {jwt_token}`

### API 限流

#### 限流规则

- 统计 API：1000 请求/分钟/API Key
- 数据分析 API：100 请求/分钟/用户
- 管理 API：60 请求/分钟/用户

#### 限流响应

- code: 429
- message: "Too Many Requests"
- error: 错误详情
- retry_after: 重试时间（秒）

## 基本用法

### API 设计流程

- 需求分析
- 接口设计
- 文档编写

## 完整示例

### API 文档示例

- 基础信息（Base URL、API Version、认证方式）
- 统计 API 接口列表
- 数据分析 API 接口列表
- 管理 API 接口列表

## 注意事项

- API 设计要规范
- 响应格式要统一
- 错误处理要完善
- 文档要完善

## 常见问题

**Q: 如何设计 API 版本控制？**

A: 使用 URL 路径进行版本控制（/api/v1/），便于后续版本升级。

**Q: 如何处理 API 限流？**

A: 使用 Redis 实现限流，返回 429 状态码和重试时间。

**Q: 如何保证 API 安全性？**

A: 使用 HTTPS、API Key 认证、JWT Token、输入验证等方式保证安全性。

## 最佳实践

- RESTful 设计
- 版本控制
- 统一响应
- 完善文档

## 对比分析

### API 设计模式对比

- RESTful vs GraphQL vs RPC

## 实践任务

1. API 设计练习：设计一个统计服务的 API 接口，设计请求和响应格式，编写 API 文档
2. API 实现：实现 API 接口，实现认证和授权，实现限流保护
3. API 测试：编写 API 测试用例，进行 API 测试，优化 API 性能
