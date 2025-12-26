# 5.7.2 API 路由与版本控制

## 概述

API 路由和版本控制是 API 管理的重要方面。本节介绍 API 路由的设计方法和版本控制策略，包括 URL 版本控制、Header 版本控制等，帮助零基础学员掌握 API 管理技术。

**章节类型**：配置性章节

**主要内容**：
- API 路由设计
- 路由组织方法
- 版本控制策略
- URL 版本控制（/v1/, /v2/）
- Header 版本控制（Accept: application/vnd.api+json;version=1）
- 版本兼容性
- 完整示例

## 核心内容

### API 路由设计

- 路由结构
- 资源路由
- 嵌套路由
- 路由组织

### 版本控制策略

- 为什么需要版本控制
- 版本控制方法
- 版本号格式
- 版本选择

### URL 版本控制

- 路径版本（/api/v1/users）
- 实现方法
- 路由配置
- 优势劣势

### Header 版本控制

- Accept 头版本
- 自定义 Header
- 实现方法
- 优势劣势

## 基本用法

### URL 版本控制示例

```php
<?php
declare(strict_types=1);

// /api/v1/users
// /api/v2/users
$version = explode('/', $_SERVER['REQUEST_URI'])[2] ?? 'v1';
```

### Header 版本控制示例

```php
<?php
declare(strict_types=1);

$accept = $_SERVER['HTTP_ACCEPT'] ?? '';
if (preg_match('/version=(\d+)/', $accept, $matches)) {
    $version = 'v' . $matches[1];
}
```

## 使用场景

- API 演进管理
- 向后兼容
- 多版本共存
- API 迁移

## 注意事项

- 版本选择策略
- 向后兼容性
- 版本文档
- 废弃策略

## 常见问题

- 如何实现 API 版本控制？
- URL 版本和 Header 版本的区别？
- 如何管理多个版本？
- 何时废弃旧版本？

## 最佳实践

- 从第一个版本开始版本控制
- 使用 URL 版本控制（简单直观）
- 提供版本文档
- 制定版本废弃策略
