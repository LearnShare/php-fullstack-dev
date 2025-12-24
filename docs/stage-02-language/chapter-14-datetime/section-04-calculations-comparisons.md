# 2.14.4 时间计算与比较

## 概述

时间计算与比较是日期时间处理的核心功能。本节详细介绍 `diff()` 计算时间差、`modify()` 时间运算、日期比较、工作日计算，以及时间范围判断。掌握这些内容对于实现复杂的日期时间逻辑至关重要。

## 特性

- **精确的时间差计算**：支持年、月、日、时、分、秒的精确计算
- **灵活的时间运算**：支持相对时间字符串和绝对时间点
- **工作日计算**：支持排除周末和节假日的计算
- **时间范围判断**：支持判断日期时间是否在指定范围内
- **DateInterval 对象**：提供丰富的时间差信息

## diff() 计算时间差

### diff() - 计算时间差

**语法**：`diff(DateTimeInterface $targetObject, bool $absolute = false): DateInterval`

**参数**：
- `$targetObject`：要比较的目标 `DateTime` 或 `DateTimeImmutable` 对象。
- `$absolute`：可选，是否返回绝对值（忽略正负），默认为 `false`。如果为 `true`，返回的时间差始终为正数。

**返回值**：返回 `DateInterval` 对象，包含时间差的详细信息。

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

`DateInterval` 对象包含以下属性：

| 属性   | 类型 | 说明                    | 示例 |
|:-------|:-----|:------------------------|:-----|
| `y`    | int  | 年份差                  | 1    |
| `m`    | int  | 月份差                  | 2    |
| `d`    | int  | 天数差                  | 15   |
| `h`    | int  | 小时差                  | 3    |
| `i`    | int  | 分钟差                  | 30   |
| `s`    | int  | 秒数差                  | 45   |
| `f`    | float| 微秒差                  | 0.5  |
| `days` | int  | 总天数差（绝对值）      | 45   |
| `invert` | int | 方向标志（1=负，0=正） | 0    |

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2024-01-15 10:30:00');
$date2 = new DateTime('2024-01-20 14:45:30');
$diff = $date1->diff($date2);

// 属性访问
echo "Years: " . $diff->y . "\n";      // 0
echo "Months: " . $diff->m . "\n";      // 0
echo "Days: " . $diff->d . "\n";       // 5
echo "Hours: " . $diff->h . "\n";      // 4
echo "Minutes: " . $diff->i . "\n";    // 15
echo "Seconds: " . $diff->s . "\n";    // 30
echo "Total days: " . $diff->days . "\n";  // 5
echo "Invert: " . $diff->invert . "\n";    // 0（正数）
```

### DateInterval::format() - 格式化时间差

**语法**：`format(string $format): string`

**参数**：
- `$format`：格式化字符串，使用特定字符表示时间差各部分。格式化字符以 `%` 开头。

**返回值**：返回格式化后的时间差字符串。

### 格式化字符完整列表

| 字符 | 说明                    | 示例   | 范围           |
|:-----|:------------------------|:-------|:---------------|
| `%Y` | 年份差                  | 1      | 整数           |
| `%y` | 年份差（与 %Y 相同）    | 1      | 整数           |
| `%M` | 月份差                  | 2      | 0-11           |
| `%m` | 月份差（与 %M 相同）    | 2      | 0-11           |
| `%D` | 天数差                  | 15     | 0-30           |
| `%d` | 天数差（与 %D 相同）    | 15     | 0-30           |
| `%a` | 总天数差（绝对值）      | 45     | 非负整数       |
| `%H` | 小时差                  | 3      | 0-23           |
| `%h` | 小时差（与 %H 相同）    | 3      | 0-23           |
| `%I` | 分钟差                  | 30     | 0-59           |
| `%i` | 分钟差（与 %I 相同）    | 30     | 0-59           |
| `%S` | 秒数差                  | 45     | 0-59           |
| `%s` | 秒数差（与 %S 相同）    | 45     | 0-59           |
| `%F` | 微秒差                  | 500000 | 0-999999       |
| `%f` | 微秒差（与 %F 相同）    | 500000 | 0-999999       |
| `%R` | 符号（+/-，无空格）     | +      | + 或 -         |
| `%r` | 符号（+/-，带空格）     | -      | + 或 -         |
| `%%` | 百分号                  | %      | %              |

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2024-01-15 10:30:00');
$date2 = new DateTime('2024-01-20 14:45:30');
$diff = $date1->diff($date2);

// 格式化字符串
echo $diff->format('%y years, %m months, %d days') . "\n";  // 0 years, 0 months, 5 days
echo $diff->format('%a total days') . "\n";  // 5 total days
echo $diff->format('%h hours, %i minutes, %s seconds') . "\n";  // 4 hours, 15 minutes, 30 seconds
echo $diff->format('%R%a days') . "\n";  // +5 days（带符号）
```

## modify() 时间运算

### modify() - 修改时间

**语法**：`modify(string $modifier): DateTime|false`

**参数**：
- `$modifier`：相对时间修改字符串，支持以下格式：
  - **相对时间**：`"+1 day"`、`"-1 week"`、`"+2 months"`、`"+3 years"`、`"+1 hour"`、`"+30 minutes"`
  - **相对时间点**：`"next Monday"`、`"last Friday"`、`"first day of next month"`、`"last day of this month"`
  - **绝对时间点**：`"noon"`、`"midnight"`、`"start of day"`、`"end of day"`
  - **组合**：`"+1 day +2 hours"`、`"next Monday +1 week"`

**返回值**：成功返回 `DateTime` 对象自身（支持链式调用），失败返回 `false`。

**注意**：`DateTime` 的 `modify()` 方法会修改原对象，而 `DateTimeImmutable` 的 `modify()` 方法返回新对象。

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

支持的相对时间字符串格式：

| 格式类型           | 示例                          | 说明                    |
|:-------------------|:------------------------------|:------------------------|
| **相对时间**       | `"+1 day"`、`"-1 week"`       | 加减指定时间单位        |
| **相对时间点**     | `"next Monday"`               | 下一个指定星期几        |
| **相对时间点**     | `"last Friday"`              | 上一个指定星期几        |
| **月份操作**       | `"first day of next month"`   | 下个月的第一天          |
| **月份操作**       | `"last day of this month"`    | 本月的最后一天          |
| **绝对时间点**     | `"noon"`、`"midnight"`        | 中午、午夜              |
| **绝对时间点**     | `"start of day"`、`"end of day"` | 一天的开始、结束        |

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15 14:30:00');

// 相对时间
$date->modify('next Monday');
echo $date->format('Y-m-d') . "\n";  // 下一个周一的日期

$date->modify('last Friday');
echo $date->format('Y-m-d') . "\n";  // 上一个周五的日期

$date->modify('first day of next month');
echo $date->format('Y-m-d') . "\n";  // 下个月的第一天

$date->modify('last day of this month');
echo $date->format('Y-m-d') . "\n";  // 本月的最后一天

// 时间点
$date->modify('noon');
echo $date->format('Y-m-d H:i:s') . "\n";  // 12:00:00

$date->modify('midnight');
echo $date->format('Y-m-d H:i:s') . "\n";  // 00:00:00

$date->modify('start of day');
echo $date->format('Y-m-d H:i:s') . "\n";  // 00:00:00

$date->modify('end of day');
echo $date->format('Y-m-d H:i:s') . "\n";  // 23:59:59
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

## 常见问题

### 问题 1：diff() 返回的 DateInterval 中的 days 和 d 有什么区别？

**回答**：
- `days`：总天数差（绝对值），表示两个日期之间的总天数
- `d`：天数差（相对值），表示在考虑年、月之后剩余的天数

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2024-01-15');
$date2 = new DateTime('2024-02-20');
$diff = $date1->diff($date2);

echo "Total days: " . $diff->days . "\n";  // 36（总天数）
echo "Days: " . $diff->d . "\n";            // 5（在 1 个月之后剩余 5 天）
echo "Months: " . $diff->m . "\n";          // 1（1 个月）
```

### 问题 2：如何判断 diff() 返回的时间差是正数还是负数？

**回答**：使用 `invert` 属性，`1` 表示负数（目标日期在前），`0` 表示正数（目标日期在后）：

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2024-01-20');
$date2 = new DateTime('2024-01-15');
$diff = $date1->diff($date2);

echo "Days: " . $diff->days . "\n";      // 5
echo "Invert: " . $diff->invert . "\n";   // 1（负数，date2 在前）
```

### 问题 3：modify() 支持哪些时间单位？

**回答**：支持以下时间单位：
- `second`、`seconds`、`sec`、`secs`
- `minute`、`minutes`、`min`、`mins`
- `hour`、`hours`
- `day`、`days`
- `week`、`weeks`
- `month`、`months`
- `year`、`years`

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15 10:30:00');
$date->modify('+1 hour +30 minutes');
echo $date->format('Y-m-d H:i:s') . "\n";  // 2024-01-15 12:00:00
```

## 最佳实践

### 1. 使用 DateTimeImmutable 进行计算

在时间计算中优先使用 `DateTimeImmutable`，避免意外修改：

```php
<?php
declare(strict_types=1);

// 推荐：使用 DateTimeImmutable
function addDays(DateTimeImmutable $date, int $days): DateTimeImmutable
{
    return $date->modify("+{$days} days");
}

// 不推荐：使用 DateTime（可能意外修改）
function addDaysMutable(DateTime $date, int $days): DateTime
{
    $date->modify("+{$days} days");  // 修改了原对象
    return $date;
}
```

### 2. 确保时区一致

在进行时间计算和比较时，确保时区一致：

```php
<?php
declare(strict_types=1);

// 推荐：明确指定时区
$date1 = new DateTimeImmutable('2024-01-15', new DateTimeZone('UTC'));
$date2 = new DateTimeImmutable('2024-01-20', new DateTimeZone('UTC'));
$diff = $date1->diff($date2);
```

### 3. 使用绝对值参数

在不确定时间顺序时，使用 `diff()` 的 `$absolute` 参数：

```php
<?php
declare(strict_types=1);

function getDaysDifference(DateTimeImmutable $date1, DateTimeImmutable $date2): int
{
    $diff = $date1->diff($date2, true);  // 使用绝对值
    return $diff->days;
}
```

### 4. 工作日计算优化

对于大量日期的工作日计算，考虑优化算法：

```php
<?php
declare(strict_types=1);

function countWeekdaysOptimized(
    DateTimeImmutable $start,
    DateTimeImmutable $end
): int {
    $totalDays = $start->diff($end)->days;
    $weeks = intval($totalDays / 7);
    $remainingDays = $totalDays % 7;
    
    // 计算剩余天数中的工作日
    $weekdays = $weeks * 5;
    $current = $start->modify("+{$weeks} weeks");
    
    for ($i = 0; $i < $remainingDays; $i++) {
        $dayOfWeek = (int) $current->format('N');
        if ($dayOfWeek <= 5) {
            $weekdays++;
        }
        $current = $current->modify('+1 day');
    }
    
    return $weekdays;
}
```

## 对比分析

### diff() vs 手动计算

| 特性                 | diff()                    | 手动计算                  |
|:---------------------|:--------------------------|:--------------------------|
| **精确度**           | 高（考虑月份天数、闰年）   | 低（可能不准确）          |
| **易用性**           | 简单                      | 复杂                      |
| **功能丰富度**       | 丰富（年、月、日、时、分、秒）| 基础（仅天数）            |
| **性能**             | 稍慢                      | 稍快                      |
| **推荐使用场景**     | 大多数场景                | 简单天数计算、性能敏感    |

### modify() vs add()/sub()

| 特性                 | modify()                  | add()/sub()               |
|:---------------------|:--------------------------|:--------------------------|
| **灵活性**           | 高（支持相对时间字符串）   | 低（需要 DateInterval）   |
| **易用性**           | 简单                      | 稍复杂                    |
| **功能丰富度**       | 丰富（支持相对时间点）     | 基础（仅时间间隔）        |
| **性能**             | 稍慢                      | 稍快                      |
| **推荐使用场景**     | 大多数场景                | 精确时间间隔计算          |

## 注意事项

1. **时区一致性**：比较和计算时确保时区一致，避免因时区差异导致的计算错误。

2. **不可变对象**：优先使用 `DateTimeImmutable`，避免意外修改。特别是在函数参数和返回值中。

3. **边界处理**：注意包含/排除边界日期的逻辑，根据业务需求决定是否包含边界。

4. **性能考虑**：大量日期计算时考虑使用缓存或优化算法，提高性能。

5. **月份天数差异**：使用 `modify('+1 month')` 时，注意不同月份的天数差异，可能导致日期跳跃。

6. **闰年处理**：计算涉及年份的时间差时，注意闰年的影响。

7. **工作日计算**：工作日计算需要考虑周末和节假日，使用专门的函数处理。

8. **时间差方向**：使用 `diff()` 时，注意 `invert` 属性表示时间差的方向。

## 练习

1. 创建一个日期计算工具类，封装常用计算功能（时间差、加减天数、工作日计算等）。

2. 编写一个函数，计算两个日期之间的工作日数量（排除周末和节假日），支持自定义节假日列表。

3. 实现一个函数，判断日期是否在工作日范围内，支持排除节假日。

4. 创建一个函数，计算指定日期后的第 N 个工作日，支持排除节假日。

5. 编写一个函数，计算年龄（考虑闰年），返回精确的年龄（年、月、日）。
