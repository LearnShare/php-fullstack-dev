# 2.15.4 时间计算与比较

## 概述

时间计算与比较是日期时间处理的核心功能。本节详细介绍 `diff()` 计算时间差、`modify()` 时间运算、日期比较、工作日计算，以及时间范围判断。

## diff() 计算时间差

### 基本用法

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2024-01-15');
$date2 = new DateTime('2024-01-20');
$diff = $date1->diff($date2);

echo $diff->days . " days\n";  // 5 days
echo $diff->format('%a days') . "\n";  // 5 days
```

### DateInterval 对象属性

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2024-01-15 10:30:00');
$date2 = new DateTime('2024-01-20 14:45:30');
$diff = $date1->diff($date2);

// 属性访问
echo "Years: " . $diff->y . "\n";
echo "Months: " . $diff->m . "\n";
echo "Days: " . $diff->d . "\n";
echo "Hours: " . $diff->h . "\n";
echo "Minutes: " . $diff->i . "\n";
echo "Seconds: " . $diff->s . "\n";
echo "Total days: " . $diff->days . "\n";
```

### format() 格式化时间差

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2024-01-15 10:30:00');
$date2 = new DateTime('2024-01-20 14:45:30');
$diff = $date1->diff($date2);

// 格式化字符串
echo $diff->format('%y years, %m months, %d days') . "\n";
echo $diff->format('%a total days') . "\n";
echo $diff->format('%h hours, %i minutes, %s seconds') . "\n";
```

### 格式化字符列表

| 字符 | 说明 | 示例 |
| :--- | :--- | :--- |
| `%Y` | 年份 | 1 |
| `%M` | 月份 | 2 |
| `%D` | 天数 | 15 |
| `%a` | 总天数 | 45 |
| `%H` | 小时 | 3 |
| `%I` | 分钟 | 30 |
| `%S` | 秒数 | 45 |
| `%R` | 符号（+/-） | + |
| `%r` | 符号（带空格） | - |

## modify() 时间运算

### 基本运算

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15');

// 加法
$date->modify('+1 day');
echo $date->format('Y-m-d') . "\n";  // 2024-01-16

$date->modify('+1 week');
echo $date->format('Y-m-d') . "\n";  // 2024-01-23

$date->modify('+1 month');
echo $date->format('Y-m-d') . "\n";  // 2024-02-23

// 减法
$date->modify('-1 day');
echo $date->format('Y-m-d') . "\n";  // 2024-02-22
```

### 相对时间字符串

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15 14:30:00');

// 相对时间
$date->modify('next Monday');
$date->modify('last Friday');
$date->modify('first day of next month');
$date->modify('last day of this month');

// 时间点
$date->modify('noon');
$date->modify('midnight');
$date->modify('start of day');
$date->modify('end of day');
```

### 使用 DateTimeImmutable

```php
<?php
declare(strict_types=1);

$date = new DateTimeImmutable('2024-01-15');
$newDate = $date->modify('+1 week');

echo "Original: " . $date->format('Y-m-d') . "\n";  // 2024-01-15
echo "New: " . $newDate->format('Y-m-d') . "\n";  // 2024-01-22
```

## 日期比较

### 基本比较

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2024-01-15');
$date2 = new DateTime('2024-01-20');

// 比较操作符
var_dump($date1 < $date2);  // true
var_dump($date1 > $date2);  // false
var_dump($date1 == $date2);  // false

// 使用 getTimestamp()
if ($date1->getTimestamp() < $date2->getTimestamp()) {
    echo "date1 is earlier\n";
}
```

### 比较方法

```php
<?php
declare(strict_types=1);

function compareDates(DateTime $date1, DateTime $date2): int
{
    if ($date1 < $date2) {
        return -1;
    } elseif ($date1 > $date2) {
        return 1;
    } else {
        return 0;
    }
}

$date1 = new DateTime('2024-01-15');
$date2 = new DateTime('2024-01-20');
echo compareDates($date1, $date2);  // -1
```

## 工作日计算

### 计算工作日

```php
<?php
declare(strict_types=1);

function countWeekdays(DateTimeImmutable $start, DateTimeImmutable $end): int
{
    $count = 0;
    $current = $start;
    
    while ($current <= $end) {
        $dayOfWeek = (int) $current->format('N');  // 1-7 (Monday-Sunday)
        if ($dayOfWeek <= 5) {  // Monday to Friday
            $count++;
        }
        $current = $current->modify('+1 day');
    }
    
    return $count;
}

$start = new DateTimeImmutable('2024-01-15');  // Monday
$end = new DateTimeImmutable('2024-01-21');  // Sunday
echo countWeekdays($start, $end) . " weekdays\n";  // 5 weekdays
```

### 排除节假日

```php
<?php
declare(strict_types=1);

function countBusinessDays(
    DateTimeImmutable $start,
    DateTimeImmutable $end,
    array $holidays = []
): int {
    $count = 0;
    $current = $start;
    $holidayStrings = array_map(fn($d) => $d->format('Y-m-d'), $holidays);
    
    while ($current <= $end) {
        $dayOfWeek = (int) $current->format('N');
        $dateString = $current->format('Y-m-d');
        
        if ($dayOfWeek <= 5 && !in_array($dateString, $holidayStrings, true)) {
            $count++;
        }
        $current = $current->modify('+1 day');
    }
    
    return $count;
}

$start = new DateTimeImmutable('2024-01-15');
$end = new DateTimeImmutable('2024-01-21');
$holidays = [
    new DateTimeImmutable('2024-01-17'),  // 假设是节假日
];
echo countBusinessDays($start, $end, $holidays) . " business days\n";
```

## 时间范围判断

### 判断是否在范围内

```php
<?php
declare(strict_types=1);

function isInRange(
    DateTimeImmutable $date,
    DateTimeImmutable $start,
    DateTimeImmutable $end
): bool {
    return $date >= $start && $date <= $end;
}

$date = new DateTimeImmutable('2024-01-18');
$start = new DateTimeImmutable('2024-01-15');
$end = new DateTimeImmutable('2024-01-20');
var_dump(isInRange($date, $start, $end));  // true
```

### 计算年龄

```php
<?php
declare(strict_types=1);

function calculateAge(DateTimeImmutable $birthday): int
{
    $now = new DateTimeImmutable();
    $diff = $now->diff($birthday);
    return $diff->y;
}

$birthday = new DateTimeImmutable('1990-05-15');
echo "Age: " . calculateAge($birthday) . " years\n";
```

## 完整示例

```php
<?php
declare(strict_types=1);

class CalculationsDemo
{
    public static function demonstrate(): void
    {
        echo "=== 时间差计算 ===\n";
        $date1 = new DateTime('2024-01-15 10:00:00');
        $date2 = new DateTime('2024-01-20 14:30:00');
        $diff = $date1->diff($date2);
        echo $diff->format('%a days, %h hours, %i minutes') . "\n";
        
        echo "\n=== 时间运算 ===\n";
        $date = new DateTimeImmutable('2024-01-15');
        $future = $date->modify('+1 month +1 week');
        echo "Future: " . $future->format('Y-m-d') . "\n";
        
        echo "\n=== 工作日计算 ===\n";
        $start = new DateTimeImmutable('2024-01-15');
        $end = new DateTimeImmutable('2024-01-21');
        echo countWeekdays($start, $end) . " weekdays\n";
    }
}

CalculationsDemo::demonstrate();
```

## 注意事项

1. **时区一致性**：比较和计算时确保时区一致。
2. **不可变对象**：优先使用 `DateTimeImmutable`，避免意外修改。
3. **边界处理**：注意包含/排除边界日期的逻辑。
4. **性能考虑**：大量日期计算时考虑使用缓存。

## 练习

1. 创建一个日期计算工具类，封装常用计算功能。
2. 编写一个函数，计算两个日期之间的工作日数量（排除节假日）。
3. 实现一个函数，判断日期是否在工作日范围内。
4. 创建一个函数，计算指定日期后的第 N 个工作日。
5. 编写一个函数，计算年龄（考虑闰年）。
