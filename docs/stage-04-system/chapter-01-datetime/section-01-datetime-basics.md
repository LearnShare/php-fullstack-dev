# 4.1.1 DateTime 基础

## 概述

DateTime 类是 PHP 中处理日期和时间的核心类。本节介绍 DateTime 类的基本用法，包括如何创建 DateTime 对象、获取当前时间、以及基础的时间格式化操作。

**章节类型**：语法性章节

**主要内容**：
- DateTime 类的概述
- 创建 DateTime 对象的多种方式
- 获取当前时间
- DateTime 对象的基本属性访问
- 基础的时间格式化方法
- DateTime 对象的常用方法
- 完整示例

## 核心内容

### DateTime 类概述

DateTime 类是 PHP 5.2.0 引入的面向对象的日期时间处理类，提供了丰富的日期时间操作方法。

### 创建 DateTime 对象

- 使用 `new DateTime()` 创建当前时间的对象
- 使用 `new DateTime('时间字符串')` 创建指定时间的对象
- 使用 `DateTime::createFromFormat()` 从格式字符串创建对象

### 获取当前时间

- `new DateTime()` 创建当前时间的对象
- `DateTime::createFromFormat('格式', '时间字符串')` 从格式化字符串创建

### DateTime 对象的基本属性

- 时间戳获取
- 年月日时分秒的访问
- 时区信息

### 基础格式化

- `format()` 方法的使用
- 常用格式字符串

## 基本用法

### 创建 DateTime 对象示例

```php
<?php
declare(strict_types=1);

// 创建当前时间的 DateTime 对象
$now = new DateTime();

// 创建指定时间的 DateTime 对象
$date = new DateTime('2025-12-25 10:30:00');
```

## 使用场景

- 获取和显示当前时间
- 处理用户输入的日期时间
- 记录事件发生的时间
- 时间数据的存储和展示

## 注意事项

- DateTime 对象是不可变的，每次修改都会返回新的对象（PHP 5.5+）
- 时区设置会影响时间的显示
- 日期字符串的格式需要符合规范

## 常见问题

- 如何获取当前时间？
- 如何创建指定日期的 DateTime 对象？
- 如何格式化日期时间？
- DateTime 和 date() 函数的区别是什么？

## 最佳实践

- 优先使用 DateTime 类而非旧的日期函数
- 明确指定时区以避免时区问题
- 使用 format() 方法进行日期格式化
- 注意日期字符串的格式要求
