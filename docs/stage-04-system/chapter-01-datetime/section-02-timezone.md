# 4.1.2 时区处理

## 概述

时区是处理日期时间时必须考虑的重要因素。本节介绍 PHP 中时区的概念、DateTimeZone 类的使用、时区的设置与转换，以及常见时区问题的处理方法。

**章节类型**：概念性章节

**主要内容**：
- 时区的概念和重要性
- DateTimeZone 类的使用
- 时区设置方法
- 时区转换
- 默认时区配置
- 常见时区问题处理
- 完整示例

## 核心内容

### 时区概念

- 什么是时区
- 为什么需要时区处理
- UTC、GMT 和本地时区的关系

### DateTimeZone 类

- DateTimeZone 类的创建
- 时区标识符（如 'Asia/Shanghai'、'UTC'）
- 可用时区列表获取

### 时区设置

- 在 DateTime 对象中设置时区
- 使用 `setTimezone()` 方法转换时区
- 默认时区配置（php.ini、date_default_timezone_set()）

### 时区转换

- 不同时区之间的时间转换
- 保持时间戳不变转换显示时区
- 保持显示时间不变转换时区

## 基本用法

### 时区设置示例

```php
<?php
declare(strict_types=1);

// 创建指定时区的 DateTime 对象
$timezone = new DateTimeZone('Asia/Shanghai');
$date = new DateTime('now', $timezone);

// 转换时区
$utcTimezone = new DateTimeZone('UTC');
$date->setTimezone($utcTimezone);
```

## 使用场景

- 多时区应用的开发
- 国际化应用的时间显示
- 时间数据的正确存储和显示
- 跨时区的系统集成

## 注意事项

- 始终明确指定时区，不要依赖系统默认时区
- 存储时间数据时建议使用 UTC
- 显示时间时转换为用户所在时区
- 注意夏令时（DST）的影响

## 常见问题

- 如何设置应用的默认时区？
- 如何在不同时区间转换时间？
- 时区转换后时间戳会改变吗？
- 如何处理夏令时？

## 最佳实践

- 数据库存储时间使用 UTC
- 应用层面转换为用户时区显示
- 使用标准的时区标识符
- 在配置文件中统一管理时区设置
