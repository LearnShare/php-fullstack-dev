# 4.6.2 数据加密

## 概述

数据加密用于保护敏感数据的安全。本节介绍 PHP 中数据加密的方法，包括 openssl_encrypt()、openssl_decrypt() 函数，以及加密算法的选择，帮助零基础学员掌握数据加密技术。

**章节类型**：语法性章节

**主要内容**：
- 数据加密概述
- openssl_encrypt() 函数
- openssl_decrypt() 函数
- 加密算法选择
- 密钥管理
- IV（初始化向量）
- 完整示例

## 核心内容

### 数据加密概述

- 加密的目的
- 对称加密和非对称加密
- 加密强度

### openssl_encrypt()

- 语法和参数
- 加密算法
- 密钥和 IV
- 返回值

### openssl_decrypt()

- 解密操作
- 参数匹配
- 错误处理

### 加密算法

- AES-256-CBC
- AES-256-GCM
- 算法选择建议

### 密钥管理

- 密钥生成
- 密钥存储
- 密钥轮换

## 基本用法

### 数据加密示例

```php
<?php
declare(strict_types=1);

$data = '敏感数据';
$key = 'your-secret-key-32-chars-long!!';
$iv = openssl_random_pseudo_bytes(16);

// 加密
$encrypted = openssl_encrypt($data, 'AES-256-CBC', $key, 0, $iv);

// 解密
$decrypted = openssl_decrypt($encrypted, 'AES-256-CBC', $key, 0, $iv);
```

## 使用场景

- 敏感数据存储
- 数据传输保护
- 配置文件加密
- 数据库字段加密

## 注意事项

- 密钥安全管理
- IV 的随机性
- 加密算法的选择
- 性能考虑

## 常见问题

- 如何生成安全的密钥？
- IV 的作用是什么？
- 如何选择加密算法？
- 加密数据的存储格式？

## 最佳实践

- 使用强加密算法
- 安全管理密钥
- 使用随机 IV
- 验证加密结果
