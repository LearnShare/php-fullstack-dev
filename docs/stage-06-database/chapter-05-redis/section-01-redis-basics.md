# 6.5.1 Redis 基础

## 概述

Redis 是高性能的内存数据结构存储。本节介绍 Redis 的基础知识、安装配置、PHP 扩展使用等，帮助零基础学员掌握 Redis 的基本操作。

**章节类型**：工具性章节

**主要内容**：
- Redis 概述
- Redis 安装配置
- PHP Redis 扩展
- 基本操作（SET、GET、DEL、EXISTS）
- 数据类型（String、Hash、List、Set、Sorted Set）
- 过期时间设置
- 完整示例

## 核心内容

### Redis 概述

- 什么是 Redis
- Redis 的特点
- Redis 的应用场景

### Redis 安装

- 安装方法
- 配置文件
- 启动服务
- 客户端连接

### PHP Redis 扩展

- 扩展安装
- 连接 Redis
- 基本操作
- 错误处理

### 数据类型

- String：字符串
- Hash：哈希
- List：列表
- Set：集合
- Sorted Set：有序集合

## 基本用法

### Redis 操作示例

```php
<?php
declare(strict_types=1);

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);

// 设置值
$redis->set('key', 'value');
$redis->setex('key', 3600, 'value'); // 带过期时间

// 获取值
$value = $redis->get('key');

// 删除键
$redis->del('key');
```

## 使用场景

- 数据缓存
- 会话存储
- 消息队列
- 计数器

## 注意事项

- 内存管理
- 数据持久化
- 连接管理
- 错误处理

## 常见问题

- 如何安装 Redis？
- 如何连接 Redis？
- Redis 的数据类型有哪些？
- 如何设置过期时间？

## 最佳实践

- 使用连接池
- 设置合理的过期时间
- 监控内存使用
- 实现错误处理
