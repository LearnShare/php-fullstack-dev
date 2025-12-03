# 2.15.1 DateTime 基础

## 概述

PHP 提供了强大的日期和时间处理功能。`DateTime` 和 `DateTimeImmutable` 类是处理日期时间的现代方式，比传统的 `date()` 函数更强大和灵活。

## DateTime 类

### 创建 DateTime 对象

```php
<?php
declare(strict_types=1);

// 当前时间
$now = new DateTime();

// 指定时间
$date = new DateTime('2024-01-15 10:30:00');

// 指定时区
$date = new DateTime('now', new DateTimeZone('Asia/Shanghai'));
```

### 格式化输出

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15 10:30:00');
echo $date->format('Y-m-d H:i:s') . "\n";  // 2024-01-15 10:30:00
echo $date->format('Y年m月d日') . "\n";    // 2024年01月15日
```

### 修改时间

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15');
$date->modify('+1 day');
echo $date->format('Y-m-d') . "\n";  // 2024-01-16

$date->modify('+1 month');
echo $date->format('Y-m-d') . "\n";  // 2024-02-16
```

## DateTimeImmutable 类

### 不可变对象

`DateTimeImmutable` 每次操作返回新实例，不修改原对象：

```php
<?php
declare(strict_types=1);

$date = new DateTimeImmutable('2024-01-15');
$newDate = $date->modify('+1 day');

echo $date->format('Y-m-d') . "\n";     // 2024-01-15（未改变）
echo $newDate->format('Y-m-d') . "\n";  // 2024-01-16（新对象）
```

### 推荐使用

在业务逻辑中优先使用 `DateTimeImmutable`，避免意外的修改：

```php
<?php
declare(strict_types=1);

function calculateDeadline(DateTimeImmutable $start, int $days): DateTimeImmutable
{
    return $start->modify("+{$days} days");
}

$start = new DateTimeImmutable('2024-01-15');
$deadline = calculateDeadline($start, 7);
echo $deadline->format('Y-m-d') . "\n";  // 2024-01-22
```

## 时区处理

### 设置时区

```php
<?php
declare(strict_types=1);

// 设置默认时区
date_default_timezone_set('Asia/Shanghai');

// 创建带时区的 DateTime
$date = new DateTime('now', new DateTimeZone('UTC'));
echo $date->format('Y-m-d H:i:s T') . "\n";  // UTC 时间

// 转换时区
$date->setTimezone(new DateTimeZone('Asia/Shanghai'));
echo $date->format('Y-m-d H:i:s T') . "\n";  // 北京时间
```

### 时区列表

```php
<?php
declare(strict_types=1);

$timezones = DateTimeZone::listIdentifiers();
foreach ($timezones as $timezone) {
    if (strpos($timezone, 'Asia') === 0) {
        echo $timezone . "\n";
    }
}
```

## 时间戳

### 转换为时间戳

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15 10:30:00');
$timestamp = $date->getTimestamp();
echo $timestamp . "\n";  // Unix 时间戳
```

### 从时间戳创建

```php
<?php
declare(strict_types=1);

$timestamp = 1705285800;
$date = (new DateTime())->setTimestamp($timestamp);
echo $date->format('Y-m-d H:i:s') . "\n";
```

## 时间差计算

### diff() - 计算时间差

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2024-01-15');
$date2 = new DateTime('2024-01-20');
$diff = $date1->diff($date2);

echo $diff->days . " days\n";        // 5 days
echo $diff->format('%a days') . "\n";  // 5 days
```

## 完整示例

```php
<?php
declare(strict_types=1);

class DateTimeBasics
{
    public static function demonstrate(): void
    {
        echo "=== 创建 DateTime ===\n";
        $now = new DateTime();
        echo $now->format('Y-m-d H:i:s') . "\n";
        
        echo "\n=== 格式化 ===\n";
        $date = new DateTime('2024-01-15 10:30:00');
        echo $date->format('Y-m-d H:i:s') . "\n";
        echo $date->format('l, F j, Y') . "\n";  // Monday, January 15, 2024
        
        echo "\n=== 修改时间 ===\n";
        $date = new DateTime('2024-01-15');
        $date->modify('+1 week');
        echo $date->format('Y-m-d') . "\n";  // 2024-01-22
        
        echo "\n=== DateTimeImmutable ===\n";
        $date = new DateTimeImmutable('2024-01-15');
        $newDate = $date->modify('+1 week');
        echo "Original: " . $date->format('Y-m-d') . "\n";
        echo "New: " . $newDate->format('Y-m-d') . "\n";
        
        echo "\n=== 时区 ===\n";
        $utc = new DateTime('now', new DateTimeZone('UTC'));
        $utc->setTimezone(new DateTimeZone('Asia/Shanghai'));
        echo $utc->format('Y-m-d H:i:s T') . "\n";
        
        echo "\n=== 时间差 ===\n";
        $date1 = new DateTime('2024-01-15');
        $date2 = new DateTime('2024-01-20');
        $diff = $date1->diff($date2);
        echo $diff->days . " days\n";
    }
}

DateTimeBasics::demonstrate();
```

## 注意事项

1. **时区设置**：始终明确设置时区，避免依赖系统默认值。

2. **不可变对象**：优先使用 `DateTimeImmutable`，避免意外的修改。

3. **格式化字符串**：注意格式化字符串的大小写，如 `Y`（4位年份）和 `y`（2位年份）。

4. **性能考虑**：`DateTime` 比 `date()` 函数稍慢，但功能更强大。

5. **日期验证**：使用 `DateTime` 可以自动验证日期的有效性。

## 练习

1. 创建一个日期工具类，封装常用的日期操作。

2. 编写一个函数，计算两个日期之间的工作日数量。

3. 实现一个函数，格式化相对时间（如"2小时前"、"3天前"）。

4. 创建一个函数，处理不同时区的时间转换。

5. 编写一个函数，验证日期字符串的有效性。
