# 5.10.3 JWT 与 Token

## 概述

JWT（JSON Web Token）是现代的 Token 认证方案。本节介绍 JWT 的概念、结构、生成和验证方法，帮助零基础学员掌握 JWT 认证技术。

**章节类型**：语法性章节

**主要内容**：
- JWT 概念
- JWT 结构（Header、Payload、Signature）
- JWT 生成
- JWT 验证
- Token 刷新
- 安全考虑
- 完整示例

## 核心内容

### JWT 概念

- 什么是 JWT
- JWT 的优势
- JWT 的应用场景

### JWT 结构

- Header（头部）
- Payload（载荷）
- Signature（签名）
- Base64 编码

### JWT 生成

- 库的选择（firebase/php-jwt）
- Payload 设计
- 签名算法
- 过期时间设置

### JWT 验证

- 签名验证
- 过期时间检查
- 载荷解析
- 错误处理

## 基本用法

### JWT 使用示例

```php
<?php
declare(strict_types=1);

use Firebase\JWT\JWT;

// 生成 JWT
$payload = ['user_id' => 123, 'exp' => time() + 3600];
$token = JWT::encode($payload, $secretKey, 'HS256');

// 验证 JWT
$decoded = JWT::decode($token, $secretKey, ['HS256']);
```

## 使用场景

- API 认证
- 无状态认证
- 微服务认证
- 单点登录

## 注意事项

- 密钥安全
- Token 过期
- Token 刷新
- 签名验证

## 常见问题

- JWT 的结构是什么？
- 如何生成 JWT？
- 如何验证 JWT？
- JWT 的安全问题？

## 最佳实践

- 使用强密钥
- 设置合理的过期时间
- 实现 Token 刷新机制
- 验证签名和过期时间
