# 4.6.4 序列化与反序列化

## 概述

序列化是将对象转换为可存储或传输格式的过程，反序列化是相反的过程。本节介绍 PHP 中序列化和反序列化的方法，包括 serialize()、unserialize()、JSON 序列化等，以及安全注意事项，帮助零基础学员掌握序列化技术。

**章节类型**：语法性章节

**主要内容**：
- 序列化概述
- serialize() 函数
- unserialize() 函数
- JSON 序列化（json_encode()、json_decode()）
- 序列化格式对比
- 安全注意事项
- 完整示例

## 核心内容

### 序列化概述

- 什么是序列化
- 序列化的用途
- 序列化格式

### serialize() 函数

- 语法和参数
- 序列化规则
- 对象序列化
- 序列化格式

### unserialize() 函数

- 反序列化操作
- 对象恢复
- 类自动加载
- 安全风险

### JSON 序列化

- json_encode() 函数
- json_decode() 函数
- 与 serialize() 的对比
- 使用场景

## 基本用法

### 序列化示例

```php
<?php
declare(strict_types=1);

$data = ['name' => 'John', 'age' => 30];

// PHP 序列化
$serialized = serialize($data);
$unserialized = unserialize($serialized);

// JSON 序列化
$json = json_encode($data);
$decoded = json_decode($json, true);
```

## 使用场景

- 数据存储
- 数据传输
- 缓存存储
- 会话存储

## 注意事项

- unserialize() 的安全风险
- 类自动加载
- 版本兼容性
- 性能考虑

## 常见问题

- serialize() 和 json_encode() 的区别？
- 反序列化的安全风险？
- 如何安全地反序列化？
- 序列化对象的版本兼容？

## 最佳实践

- 优先使用 JSON 序列化
- 避免反序列化不可信数据
- 使用白名单验证
- 考虑版本兼容性
