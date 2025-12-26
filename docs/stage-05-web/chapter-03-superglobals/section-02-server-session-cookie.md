# 5.3.2 $_SERVER、$_SESSION 与 $_COOKIE

## 概述

$_SERVER、$_SESSION 和 $_COOKIE 提供了服务器信息、会话数据和 Cookie 数据的访问。本节介绍这些超全局变量的使用方法和应用场景，帮助零基础学员掌握服务器信息和状态管理。

**章节类型**：参考性章节

**主要内容**：
- $_SERVER 超全局变量
- $_SESSION 超全局变量
- $_COOKIE 超全局变量
- 会话管理
- Cookie 管理
- 安全注意事项
- 完整示例

## 核心内容

### $_SERVER 超全局变量

- 服务器信息获取
- 请求信息获取
- 常用键值（REQUEST_URI、HTTP_HOST、REMOTE_ADDR 等）
- 使用场景

### $_SESSION 超全局变量

- 会话数据存储
- session_start() 函数
- 会话数据访问
- 会话销毁

### $_COOKIE 超全局变量

- Cookie 数据获取
- setcookie() 函数
- Cookie 属性设置
- 安全选项

### 会话管理

- 会话启动
- 会话数据操作
- 会话销毁
- 会话配置

## 基本用法

### $_SERVER 示例

```php
<?php
declare(strict_types=1);

// 获取请求 URI
$uri = $_SERVER['REQUEST_URI'];

// 获取客户端 IP
$ip = $_SERVER['REMOTE_ADDR'];
```

### $_SESSION 示例

```php
<?php
declare(strict_types=1);

session_start();
$_SESSION['user_id'] = 123;
$userId = $_SESSION['user_id'] ?? null;
```

## 使用场景

- 获取服务器和请求信息
- 用户会话管理
- 用户偏好设置
- 状态保持

## 注意事项

- 会话安全配置
- Cookie 安全选项
- 数据验证
- 敏感信息处理

## 常见问题

- $_SERVER 包含哪些信息？
- 如何启动和管理会话？
- Cookie 和 Session 的区别？
- 如何安全地使用 Cookie？

## 最佳实践

- 使用 $_SERVER 获取请求信息
- 合理使用会话存储
- 设置安全的 Cookie 选项
- 及时销毁会话数据
