# 5.9.1 Session 基础

## 概述

Session 是服务器端会话管理的基础。本节介绍 Session 的概念、使用方法、配置选项等，帮助零基础学员掌握 Session 会话管理技术。

**章节类型**：语法性章节

**主要内容**：
- Session 概念
- session_start() 函数
- $_SESSION 超全局变量
- 会话数据操作
- 会话配置（php.ini）
- 会话安全
- 会话销毁
- 完整示例

## 核心内容

### Session 概念

- 什么是 Session
- Session 的作用
- Session ID 的传递
- 服务器端存储

### session_start()

- 语法和参数
- 启动会话
- 会话 ID 生成
- 调用时机

### $_SESSION 使用

- 数据存储
- 数据读取
- 数据修改
- 数据删除

### 会话配置

- session.save_path
- session.cookie_lifetime
- session.cookie_secure
- session.cookie_httponly

## 基本用法

### Session 使用示例

```php
<?php
declare(strict_types=1);

// 启动会话
session_start();

// 存储数据
$_SESSION['user_id'] = 123;
$_SESSION['username'] = 'john';

// 读取数据
$userId = $_SESSION['user_id'] ?? null;

// 销毁会话
session_destroy();
```

## 使用场景

- 用户登录状态
- 购物车数据
- 临时数据存储
- 用户偏好设置

## 注意事项

- session_start() 必须在输出前调用
- 会话安全配置
- 会话数据大小
- 会话过期处理

## 常见问题

- 如何启动 Session？
- Session 数据存储在哪里？
- 如何销毁 Session？
- Session 安全如何配置？

## 最佳实践

- 在输出前启动 Session
- 配置安全的 Session Cookie
- 及时销毁不需要的 Session
- 限制 Session 数据大小
