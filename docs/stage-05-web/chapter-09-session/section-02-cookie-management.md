# 5.9.2 Cookie 管理

## 概述

Cookie 是客户端状态存储的机制。本节介绍 Cookie 的概念、设置方法、读取方法、安全选项等，帮助零基础学员掌握 Cookie 管理技术。

**章节类型**：语法性章节

**主要内容**：
- Cookie 概念
- setcookie() 函数
- $_COOKIE 超全局变量
- Cookie 属性（expires、path、domain、secure、httponly）
- Cookie 读取
- Cookie 删除
- 安全选项
- 完整示例

## 核心内容

### Cookie 概念

- 什么是 Cookie
- Cookie 的作用
- Cookie 的存储位置
- Cookie 的限制

### setcookie() 函数

- 语法和参数
- Cookie 设置
- 过期时间
- 路径和域名

### Cookie 属性

- expires：过期时间
- path：路径限制
- domain：域名限制
- secure：HTTPS 传输
- httponly：JavaScript 不可访问

### Cookie 安全

- Secure 标志
- HttpOnly 标志
- SameSite 属性
- 敏感数据处理

## 基本用法

### Cookie 设置示例

```php
<?php
declare(strict_types=1);

// 设置 Cookie
setcookie('username', 'john', time() + 3600, '/', '', true, true);

// 读取 Cookie
$username = $_COOKIE['username'] ?? '';

// 删除 Cookie
setcookie('username', '', time() - 3600);
```

## 使用场景

- 用户偏好设置
- 记住登录状态
- 跟踪用户行为
- 临时数据存储

## 注意事项

- Cookie 大小限制
- 域名和路径限制
- 安全选项设置
- 过期时间管理

## 常见问题

- 如何设置 Cookie？
- Cookie 的属性有什么作用？
- 如何删除 Cookie？
- 如何安全地使用 Cookie？

## 最佳实践

- 设置合理的过期时间
- 使用 Secure 和 HttpOnly 标志
- 限制 Cookie 的作用域
- 避免存储敏感信息
