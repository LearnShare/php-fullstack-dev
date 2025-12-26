# 5.10.1 认证基础（AuthN）

## 概述

认证是验证用户身份的过程。本节介绍认证的基本概念和方法，包括密码验证、Session 认证、Token 认证等，帮助零基础学员掌握用户认证技术。

**章节类型**：概念性章节

**主要内容**：
- 认证概念
- 密码验证
- Session 认证
- Token 认证
- 认证流程
- 安全考虑
- 完整示例

## 核心内容

### 认证概念

- 什么是认证
- 认证的目的
- 认证方式

### 密码验证

- 密码哈希验证
- password_verify() 函数
- 密码强度要求
- 密码重置

### Session 认证

- Session 认证流程
- 登录状态保持
- 会话管理
- 登出处理

### Token 认证

- Token 生成
- Token 验证
- Token 存储
- Token 刷新

## 基本用法

### 认证示例

```php
<?php
declare(strict_types=1);

// 密码验证
if (password_verify($password, $hash)) {
    // 创建会话
    session_start();
    $_SESSION['user_id'] = $userId;
    $_SESSION['authenticated'] = true;
}
```

## 使用场景

- 用户登录
- API 认证
- 第三方认证
- 单点登录

## 注意事项

- 密码安全存储
- 会话安全
- Token 安全
- 认证状态检查

## 常见问题

- 如何验证密码？
- Session 认证和 Token 认证的区别？
- 如何实现登录功能？
- 如何检查认证状态？

## 最佳实践

- 使用 password_hash() 存储密码
- 使用 password_verify() 验证密码
- 实现会话超时
- 使用 HTTPS 传输认证信息
