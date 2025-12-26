# 5.4.4 URL 路由与重写

## 概述

URL 路由和重写用于创建友好的 URL 结构。本节介绍 URL 路由的概念、实现方法、URL 重写配置等，帮助零基础学员理解路由机制。

**章节类型**：概念性章节

**主要内容**：
- URL 路由概念
- 路由规则设计
- URL 重写实现
- .htaccess 配置（Apache）
- Nginx 配置
- 路由解析
- 完整示例

## 核心内容

### URL 路由概念

- 什么是路由
- 友好 URL 的优势
- 路由的作用

### 路由规则

- 路由模式
- 参数提取
- 路由匹配
- 默认路由

### URL 重写

- 重写规则
- Apache mod_rewrite
- Nginx rewrite
- 重定向规则

### 路由实现

- 路由表
- 路由匹配算法
- 参数提取
- 控制器映射

## 基本用法

### .htaccess 配置示例

```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php?route=$1 [L,QSA]
```

## 使用场景

- 友好 URL 实现
- RESTful API
- MVC 框架
- SEO 优化

## 注意事项

- 路由性能
- 路由冲突
- 参数验证
- 安全性考虑

## 常见问题

- 如何实现 URL 路由？
- .htaccess 如何配置？
- 如何提取路由参数？
- 路由和重写的区别？

## 最佳实践

- 设计清晰的路由规则
- 使用框架的路由系统
- 优化路由匹配性能
- 提供路由文档
