# 6.2.1 PDO 基础与连接

## 概述

PDO 是 PHP 访问数据库的标准接口。本节介绍 PDO 的基础知识、连接方法、配置选项等，帮助零基础学员掌握 PDO 的使用。

**章节类型**：语法性章节

**主要内容**：
- PDO 概述
- PDO 连接（DSN、用户名、密码）
- 连接配置（错误模式、属性设置）
- 错误处理
- 连接管理
- 完整示例

## 核心内容

### PDO 概述

- 什么是 PDO
- PDO 的优势
- PDO 与 MySQLi 的对比

### PDO 连接

- DSN（数据源名称）
- 连接参数
- 连接选项
- 连接字符串

### 连接配置

- 错误模式设置（ERRMODE_EXCEPTION）
- 属性设置
- 字符集设置
- 时区设置

### 错误处理

- 异常捕获
- 错误信息获取
- 错误码获取
- 错误处理策略

## 基本用法

### PDO 连接示例

```php
<?php
declare(strict_types=1);

try {
    $dsn = 'mysql:host=localhost;dbname=test;charset=utf8mb4';
    $pdo = new PDO($dsn, 'username', 'password', [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
    ]);
} catch (PDOException $e) {
    die('连接失败: ' . $e->getMessage());
}
```

## 使用场景

- 数据库连接
- 数据库操作
- 数据访问层
- 应用开发

## 注意事项

- 连接安全性
- 错误处理
- 连接池管理
- 字符集设置

## 常见问题

- 如何连接数据库？
- PDO 和 MySQLi 的区别？
- 如何设置错误模式？
- 如何处理连接错误？

## 最佳实践

- 使用异常模式
- 设置合适的字符集
- 实现连接管理
- 处理连接错误
