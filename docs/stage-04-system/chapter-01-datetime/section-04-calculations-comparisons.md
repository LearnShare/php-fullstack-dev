# 4.1.4 时间计算与比较

## 概述

时间计算和比较是日期时间处理中的常见需求。本节介绍如何进行时间的加减运算、使用 DateInterval 表示时间间隔、进行时间比较操作，以及计算日期差值的方法。

**章节类型**：语法性章节

**主要内容**：
- 时间加减运算（modify(), add(), sub()）
- DateInterval 类的使用
- 时间比较方法（diff(), 比较运算符）
- 日期差值计算
- 时间间隔的创建和应用
- 完整示例

## 核心内容

### 时间加减运算

- `modify()` 方法的使用
- `add()` 方法添加时间间隔
- `sub()` 方法减去时间间隔
- 相对时间字符串（如 '+1 day', '-2 weeks'）

### DateInterval 类

- DateInterval 对象的创建
- 时间间隔的表示（年、月、日、时、分、秒）
- 使用 DateInterval::createFromDateString() 创建间隔

### 时间比较

- `diff()` 方法计算时间差
- 比较运算符（<, >, ==, <=, >=）
- DateInterval 对象的使用

### 日期差值计算

- 计算两个日期之间的差值
- 获取差值的各个组成部分
- 格式化差值显示

## 基本用法

### 时间加减示例

```php
<?php
declare(strict_types=1);

$date = new DateTime('2025-12-25');

// 添加一天
$date->modify('+1 day');

// 使用 add() 方法
$interval = new DateInterval('P1D'); // 1天
$date->add($interval);
```

### 时间比较示例

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2025-12-25');
$date2 = new DateTime('2025-12-26');

// 计算差值
$diff = $date1->diff($date2);
echo $diff->days; // 1
```

## 使用场景

- 计算未来或过去的日期
- 计算两个日期之间的差值
- 判断日期的先后关系
- 处理周期性任务的时间计算

## 注意事项

- modify() 方法会修改原对象（PHP 5.5 之前）或返回新对象
- DateInterval 使用 ISO 8601 持续时间格式
- 月份和年份的加减可能产生预期外的结果（如月末日期）
- 时区会影响时间计算的结果

## 常见问题

- 如何计算两个日期之间的天数？
- 如何添加或减去指定的时间间隔？
- 月份加减时遇到月末日期怎么办？
- DateInterval 的格式字符串如何编写？

## 最佳实践

- 优先使用 add()/sub() 方法而非 modify()
- 使用 DateInterval 对象进行复杂的时间计算
- 注意月份和年份加减的特殊情况
- 时间计算时考虑时区因素
