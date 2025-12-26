# 4.6.1 密码哈希

## 概述

密码哈希是安全存储密码的标准方法。本节介绍 PHP 中密码哈希的使用，包括 password_hash()、password_verify() 函数，以及哈希算法的选择，帮助零基础学员掌握安全的密码存储方法。

**章节类型**：语法性章节

**主要内容**：
- 密码哈希概述
- password_hash() 函数
- password_verify() 函数
- 哈希算法（PASSWORD_DEFAULT、PASSWORD_BCRYPT、PASSWORD_ARGON2）
- 密码策略
- 完整示例

## 核心内容

### 密码哈希概述

- 为什么需要哈希
- 哈希与加密的区别
- 安全哈希的要求

### password_hash()

- 语法和参数
- 算法选择
- 成本参数
- 返回值格式

### password_verify()

- 密码验证
- 与哈希值比较
- 返回值

### 哈希算法

- PASSWORD_DEFAULT：默认算法
- PASSWORD_BCRYPT：bcrypt 算法
- PASSWORD_ARGON2：Argon2 算法
- 算法选择建议

## 基本用法

### 密码哈希示例

```php
<?php
declare(strict_types=1);

// 创建密码哈希
$hash = password_hash('my_password', PASSWORD_DEFAULT);

// 验证密码
if (password_verify('my_password', $hash)) {
    echo "密码正确\n";
}
```

## 使用场景

- 用户密码存储
- 身份验证系统
- 安全敏感应用
- 密码管理系统

## 注意事项

- 永远不要存储明文密码
- 使用 PASSWORD_DEFAULT
- 定期更新哈希算法
- 密码强度要求

## 常见问题

- 为什么不能存储明文密码？
- password_hash() 和 md5() 的区别？
- 如何选择哈希算法？
- 如何更新旧密码哈希？

## 最佳实践

- 始终使用 password_hash()
- 使用 PASSWORD_DEFAULT
- 设置合理的成本参数
- 实现密码强度策略
