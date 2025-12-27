# 4.1.4 时间计算与比较

## 概述

时间计算与比较是日期时间处理的核心功能。在实际应用中，我们经常需要计算两个日期之间的差值、进行日期的加减运算、判断日期的先后关系、计算工作日等。掌握时间计算与比较的方法对于实现复杂的日期时间逻辑至关重要。

时间计算包括添加或减去时间间隔、计算未来或过去的日期等操作。时间比较包括判断两个日期的先后关系、计算日期之间的差值等操作。PHP 的 DateTime 类提供了丰富的时间计算和比较方法，包括 `modify()`、`add()`、`sub()`、`diff()` 等方法，以及 DateInterval 类用于表示时间间隔。

理解时间计算与比较的机制可以帮助我们实现各种日期时间相关的业务逻辑，比如计算任务的截止日期、判断活动是否在有效期内、计算用户年龄、计算工作日等。

**主要内容**：
- `diff()` 方法计算时间差
- `modify()` 方法进行时间运算
- `add()` 和 `sub()` 方法进行时间加减
- DateInterval 类的使用
- 日期比较操作（比较运算符）
- 工作日计算
- 时间范围判断
- 完整示例和最佳实践

## 特性

- **精确的时间差计算**：支持年、月、日、时、分、秒的精确计算
- **灵活的时间运算**：支持相对时间字符串和 DateInterval 对象
- **工作日计算**：支持排除周末和节假日的计算
- **时间范围判断**：支持判断日期时间是否在指定范围内
- **DateInterval 对象**：提供丰富的时间差信息

## 语法/定义

### diff() 方法

**语法**：`diff(DateTimeInterface $targetObject, bool $absolute = false): DateInterval`

**参数**：
- `$targetObject`：要比较的目标 `DateTime` 或 `DateTimeImmutable` 对象
- `$absolute`：可选，是否返回绝对值（忽略正负），默认为 `false`。如果为 `true`，返回的时间差始终为正数

**返回值**：返回 `DateInterval` 对象，包含时间差的详细信息

### modify() 方法

**语法**：`modify(string $modifier): DateTime|DateTimeImmutable|false`

**参数**：
- `$modifier`：相对时间修改字符串，如 `"+1 day"`、`"-1 week"`、`"next Monday"` 等

**返回值**：
- `DateTime`：返回 `DateTime` 对象自身（会修改原对象）
- `DateTimeImmutable`：返回新的 `DateTimeImmutable` 对象（原对象不变）

### add() 方法

**语法**：`add(DateInterval $interval): DateTime|DateTimeImmutable`

**参数**：
- `$interval`：`DateInterval` 对象，表示要添加的时间间隔

**返回值**：
- `DateTime`：返回 `DateTime` 对象自身（会修改原对象）
- `DateTimeImmutable`：返回新的 `DateTimeImmutable` 对象（原对象不变）

### sub() 方法

**语法**：`sub(DateInterval $interval): DateTime|DateTimeImmutable`

**参数**：
- `$interval`：`DateInterval` 对象，表示要减去的时间间隔

**返回值**：
- `DateTime`：返回 `DateTime` 对象自身（会修改原对象）
- `DateTimeImmutable`：返回新的 `DateTimeImmutable` 对象（原对象不变）

## 基本用法

### 示例 1：使用 diff() 计算时间差

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2025-01-15 10:30:00');
$date2 = new DateTime('2025-01-20 14:45:30');

// 计算时间差
$diff = $date1->diff($date2);

// 访问 DateInterval 属性
echo "总天数: " . $diff->days . "\n";
echo "年份差: " . $diff->y . "\n";
echo "月份差: " . $diff->m . "\n";
echo "天数差: " . $diff->d . "\n";
echo "小时差: " . $diff->h . "\n";
echo "分钟差: " . $diff->i . "\n";
echo "秒数差: " . $diff->s . "\n";
echo "方向: " . ($diff->invert === 0 ? '正数' : '负数') . "\n";
```

**输出**：

```
总天数: 5
年份差: 0
月份差: 0
天数差: 5
小时差: 4
分钟差: 15
秒数差: 30
方向: 正数
```

**说明**：
- `diff()` 返回 `DateInterval` 对象
- `days` 属性表示总天数差（绝对值）
- `y`、`m`、`d`、`h`、`i`、`s` 分别表示年、月、日、时、分、秒的差值
- `invert` 表示方向（0=正数，1=负数）

### 示例 2：使用绝对值计算时间差

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2025-01-20');
$date2 = new DateTime('2025-01-15');

// 不使用绝对值（date2 在前，结果为负数）
$diff1 = $date1->diff($date2);
echo "天数: " . $diff1->days . "\n";  // 5
echo "方向: " . $diff1->invert . "\n";  // 1（负数）

// 使用绝对值（始终为正数）
$diff2 = $date1->diff($date2, true);
echo "天数: " . $diff2->days . "\n";  // 5
echo "方向: " . $diff2->invert . "\n";  // 0（正数）
```

**输出**：

```
天数: 5
方向: 1
天数: 5
方向: 0
```

**说明**：
- `absolute = false` 时，结果可能为负数（`invert = 1`）
- `absolute = true` 时，结果始终为正数（`invert = 0`）

### 示例 3：DateInterval 格式化

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2025-01-15 10:30:00');
$date2 = new DateTime('2025-01-20 14:45:30');
$diff = $date1->diff($date2);

// 格式化时间差
echo $diff->format('%y years, %m months, %d days') . "\n";  // 0 years, 0 months, 5 days
echo $diff->format('%a total days') . "\n";  // 5 total days
echo $diff->format('%h hours, %i minutes, %s seconds') . "\n";  // 4 hours, 15 minutes, 30 seconds
echo $diff->format('%R%a days') . "\n";  // +5 days（带符号）
```

**输出**：

```
0 years, 0 months, 5 days
5 total days
4 hours, 15 minutes, 30 seconds
+5 days
```

**说明**：
- `DateInterval::format()` 使用格式化字符串输出时间差
- 格式化字符以 `%` 开头
- `%a` 表示总天数差
- `%R` 表示符号（+/-）

### 示例 4：使用 modify() 进行时间运算

```php
<?php
declare(strict_types=1);

// DateTime：修改原对象
$date = new DateTime('2025-01-15');
$date->modify('+1 day');
echo "DateTime: " . $date->format('Y-m-d') . "\n";  // 2025-01-16

$date->modify('+1 week');
echo "DateTime: " . $date->format('Y-m-d') . "\n";  // 2025-01-23

$date->modify('+1 month');
echo "DateTime: " . $date->format('Y-m-d') . "\n";  // 2025-02-23

// DateTimeImmutable：返回新对象
$immutable = new DateTimeImmutable('2025-01-15');
$newDate = $immutable->modify('+1 day');
echo "原对象: " . $immutable->format('Y-m-d') . "\n";     // 2025-01-15
echo "新对象: " . $newDate->format('Y-m-d') . "\n";  // 2025-01-16
```

**输出**：

```
DateTime: 2025-01-16
DateTime: 2025-01-23
DateTime: 2025-02-23
原对象: 2025-01-15
新对象: 2025-01-16
```

**说明**：
- `modify()` 支持相对时间字符串
- `DateTime` 的 `modify()` 会修改原对象
- `DateTimeImmutable` 的 `modify()` 返回新对象

### 示例 5：使用 add() 和 sub() 方法

```php
<?php
declare(strict_types=1);

// 创建 DateInterval 对象
$interval = new DateInterval('P1D');  // 1 天（ISO 8601 格式：P1D = Period 1 Day）

$date = new DateTime('2025-01-15');

// 添加时间间隔
$date->add($interval);
echo "加1天: " . $date->format('Y-m-d') . "\n";  // 2025-01-16

// 减去时间间隔
$date->sub($interval);
echo "减1天: " . $date->format('Y-m-d') . "\n";  // 2025-01-15

// 使用 DateTimeImmutable
$immutable = new DateTimeImmutable('2025-01-15');
$newDate = $immutable->add($interval);
echo "原对象: " . $immutable->format('Y-m-d') . "\n";     // 2025-01-15
echo "新对象: " . $newDate->format('Y-m-d') . "\n";  // 2025-01-16
```

**输出**：

```
加1天: 2025-01-16
减1天: 2025-01-15
原对象: 2025-01-15
新对象: 2025-01-16
```

**说明**：
- `add()` 和 `sub()` 使用 `DateInterval` 对象
- ISO 8601 格式：`P1D`（1天）、`P1W`（1周）、`P1M`（1月）、`P1Y`（1年）

### 示例 6：DateInterval 的创建

```php
<?php
declare(strict_types=1);

// 方法 1：使用 ISO 8601 格式
$interval1 = new DateInterval('P1D');      // 1 天
$interval2 = new DateInterval('P1W');      // 1 周
$interval3 = new DateInterval('P1M');      // 1 月
$interval4 = new DateInterval('P1Y');      // 1 年
$interval5 = new DateInterval('P1Y2M3D');  // 1年2月3天
$interval6 = new DateInterval('PT1H');     // 1 小时（T 分隔日期和时间）
$interval7 = new DateInterval('PT1H30M');  // 1小时30分钟

// 方法 2：使用 createFromDateString()
$interval8 = DateInterval::createFromDateString('1 day');
$interval9 = DateInterval::createFromDateString('2 weeks');
$interval10 = DateInterval::createFromDateString('1 month 2 days');

// 测试
$date = new DateTime('2025-01-15');
$date->add($interval1);
echo "加1天: " . $date->format('Y-m-d') . "\n";  // 2025-01-16

$date = new DateTime('2025-01-15');
$date->add($interval7);
echo "加1小时30分钟: " . $date->format('Y-m-d H:i:s') . "\n";  // 2025-01-15 01:30:00
```

**输出**：

```
加1天: 2025-01-16
加1小时30分钟: 2025-01-15 01:30:00
```

**说明**：
- `DateInterval` 可以使用 ISO 8601 格式创建
- 也可以使用 `createFromDateString()` 从自然语言字符串创建

### 示例 7：日期比较

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2025-01-15');
$date2 = new DateTime('2025-01-20');
$date3 = new DateTime('2025-01-15');

// 使用比较运算符
var_dump($date1 < $date2);   // true
var_dump($date1 > $date2);   // false
var_dump($date1 <= $date2);  // true
var_dump($date1 >= $date2);  // false
var_dump($date1 == $date3);  // true（时间相同）
var_dump($date1 === $date3); // false（对象不同）

// 使用 getTimestamp() 比较
if ($date1->getTimestamp() < $date2->getTimestamp()) {
    echo "date1 在 date2 之前\n";
}
```

**输出**：

```
bool(true)
bool(false)
bool(true)
bool(false)
bool(true)
bool(false)
date1 在 date2 之前
```

**说明**：
- 可以直接使用比较运算符比较 DateTime 对象
- `==` 比较时间值，`===` 比较对象引用

### 示例 8：计算工作日

```php
<?php
declare(strict_types=1);

function countWeekdays(DateTimeImmutable $start, DateTimeImmutable $end): int
{
    $count = 0;
    $current = $start;
    
    while ($current <= $end) {
        $dayOfWeek = (int)$current->format('N');  // 1-7 (Monday-Sunday)
        if ($dayOfWeek <= 5) {  // Monday to Friday
            $count++;
        }
        $current = $current->modify('+1 day');
    }
    
    return $count;
}

$start = new DateTimeImmutable('2025-01-15');  // Wednesday
$end = new DateTimeImmutable('2025-01-21');    // Tuesday
echo countWeekdays($start, $end) . " 个工作日\n";  // 5 个工作日
```

**输出**：

```
5 个工作日
```

**说明**：
- `format('N')` 返回 ISO-8601 星期几（1=周一，7=周日）
- 工作日通常是周一到周五（1-5）

### 示例 9：计算年龄

```php
<?php
declare(strict_types=1);

function calculateAge(DateTimeImmutable $birthday): int
{
    $now = new DateTimeImmutable();
    $diff = $now->diff($birthday);
    return $diff->y;  // 返回年份差
}

$birthday = new DateTimeImmutable('1990-05-15');
$age = calculateAge($birthday);
echo "年龄: {$age} 岁\n";
```

**输出**：

```
年龄: 34 岁
```

**说明**：
- 使用 `diff()` 计算当前时间和生日的时间差
- `y` 属性表示年份差，即年龄

### 示例 10：判断日期是否在范围内

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

$date = new DateTimeImmutable('2025-01-18');
$start = new DateTimeImmutable('2025-01-15');
$end = new DateTimeImmutable('2025-01-20');

if (isInRange($date, $start, $end)) {
    echo "日期在范围内\n";
} else {
    echo "日期不在范围内\n";
}
```

**输出**：

```
日期在范围内
```

**说明**：
- 使用比较运算符判断日期是否在范围内
- 支持闭区间（包含边界）

## 使用场景

### 场景 1：计算任务截止日期

根据开始日期和持续时间计算截止日期。

**示例**：

```php
<?php
declare(strict_types=1);

function calculateDeadline(
    DateTimeImmutable $startDate,
    int $days
): DateTimeImmutable {
    return $startDate->modify("+{$days} days");
}

$start = new DateTimeImmutable('2025-01-15');
$deadline = calculateDeadline($start, 7);
echo "截止日期: " . $deadline->format('Y-m-d') . "\n";  // 2025-01-22
```

### 场景 2：判断活动是否在有效期内

判断当前时间是否在活动的有效期内。

**示例**：

```php
<?php
declare(strict_types=1);

function isActive(
    DateTimeImmutable $startDate,
    DateTimeImmutable $endDate
): bool {
    $now = new DateTimeImmutable();
    return $now >= $startDate && $now <= $endDate;
}

$start = new DateTimeImmutable('2025-01-01');
$end = new DateTimeImmutable('2025-01-31');
echo isActive($start, $end) ? "活动进行中\n" : "活动未开始或已结束\n";
```

## 注意事项

### modify() vs add()/sub()

- **modify()**：使用相对时间字符串，简单直观
- **add()/sub()**：使用 DateInterval 对象，更灵活和类型安全
- **推荐使用 add()/sub()**：对于复杂的时间计算，使用 DateInterval 更清晰

**示例**：

```php
<?php
declare(strict_types=1);

// 使用 modify()
$date1 = new DateTime('2025-01-15');
$date1->modify('+1 day +2 hours');

// 使用 add()（推荐）
$date2 = new DateTime('2025-01-15');
$interval = DateInterval::createFromDateString('1 day 2 hours');
$date2->add($interval);
```

### 月份和年份加减的特殊情况

月份和年份的加减可能产生预期外的结果，特别是涉及月末日期时。

**示例**：

```php
<?php
declare(strict_types=1);

// 1月31日加1个月
$date = new DateTime('2025-01-31');
$date->modify('+1 month');
echo $date->format('Y-m-d') . "\n";  // 2025-03-03（不是 2月31日）

// 2月29日加1年（非闰年）
$date2 = new DateTime('2024-02-29');  // 2024年是闰年
$date2->modify('+1 year');
echo $date2->format('Y-m-d') . "\n";  // 2025-03-01（不是 2月29日）
```

**说明**：
- 当日期不存在时（如2月31日），会自动调整到下一个有效日期
- 需要注意这种特殊情况

### 时区影响

时间计算时需要考虑时区因素。

**示例**：

```php
<?php
declare(strict_types=1);

$utc = new DateTime('2025-01-15 10:00:00', new DateTimeZone('UTC'));
$beijing = new DateTime('2025-01-15 10:00:00', new DateTimeZone('Asia/Shanghai'));

// 虽然显示时间相同，但时间戳不同
echo "UTC 时间戳: " . $utc->getTimestamp() . "\n";
echo "北京时间戳: " . $beijing->getTimestamp() . "\n";
```

## 常见问题

### 问题 1：diff() 返回的 days 和 d 有什么区别？

**回答**：
- `days`：总天数差（绝对值），表示两个日期之间的总天数
- `d`：天数差（相对值），表示在考虑年、月之后剩余的天数

**示例**：

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2025-01-15');
$date2 = new DateTime('2025-02-20');
$diff = $date1->diff($date2);

echo "总天数: " . $diff->days . "\n";  // 36（总天数）
echo "天数: " . $diff->d . "\n";       // 5（在 1 个月之后剩余 5 天）
echo "月份: " . $diff->m . "\n";       // 1（1 个月）
```

### 问题 2：如何判断 diff() 返回的时间差是正数还是负数？

**回答**：使用 `invert` 属性，`1` 表示负数（目标日期在前），`0` 表示正数（目标日期在后）。

**示例**：

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2025-01-20');
$date2 = new DateTime('2025-01-15');
$diff = $date1->diff($date2);

echo "天数: " . $diff->days . "\n";     // 5
echo "方向: " . $diff->invert . "\n";   // 1（负数，date2 在前）

if ($diff->invert === 1) {
    echo "date1 在 date2 之后\n";
} else {
    echo "date1 在 date2 之前\n";
}
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

**示例**：

```php
<?php
declare(strict_types=1);

$date = new DateTime('2025-01-15');

$date->modify('+1 second');
$date->modify('+1 minute');
$date->modify('+1 hour');
$date->modify('+1 day');
$date->modify('+1 week');
$date->modify('+1 month');
$date->modify('+1 year');
```

## 最佳实践

### 1. 优先使用 add()/sub() 方法

对于复杂的时间计算，使用 `add()`/`sub()` 方法配合 `DateInterval` 对象，比 `modify()` 更清晰和类型安全。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：使用 add()/sub()
$date = new DateTimeImmutable('2025-01-15');
$interval = new DateInterval('P1Y2M3DT4H5M6S');  // 1年2月3天4小时5分6秒
$newDate = $date->add($interval);

// 不推荐：使用 modify()（复杂情况）
// $date->modify('+1 year +2 months +3 days +4 hours +5 minutes +6 seconds');
```

### 2. 使用 DateInterval 对象进行复杂计算

对于复杂的时间间隔，使用 `DateInterval` 对象更清晰。

**示例**：

```php
<?php
declare(strict_types=1);

// 创建复杂的时间间隔
$interval = DateInterval::createFromDateString('1 year 2 months 3 days 4 hours');

$date = new DateTimeImmutable('2025-01-15');
$newDate = $date->add($interval);
echo $newDate->format('Y-m-d H:i:s') . "\n";
```

### 3. 注意月份和年份加减的特殊情况

月份和年份的加减可能产生预期外的结果，需要特别注意。

**示例**：见"注意事项"部分的示例

### 4. 时间计算时考虑时区

进行时间计算时，确保时区设置正确。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：明确指定时区
$date = new DateTimeImmutable('2025-01-15', new DateTimeZone('UTC'));
$newDate = $date->modify('+1 day');
```

### 5. 使用 DateTimeImmutable 避免副作用

在业务逻辑中使用 `DateTimeImmutable`，避免意外的修改。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：使用 DateTimeImmutable
function calculateFutureDate(DateTimeImmutable $start, int $days): DateTimeImmutable
{
    return $start->modify("+{$days} days");
}

$start = new DateTimeImmutable('2025-01-15');
$future = calculateFutureDate($start, 7);
// $start 保持不变
```

## 对比分析

### modify() vs add()/sub()

| 特性         | modify()                      | add()/sub()                   |
|:-------------|:------------------------------|:------------------------------|
| **参数类型**  | 字符串                        | DateInterval 对象             |
| **类型安全**  | ⚠️ 运行时检查                 | ✅ 编译时类型检查             |
| **灵活性**    | ✅ 支持自然语言               | ⚠️ 需要创建 DateInterval      |
| **复杂度**    | ✅ 简单（字符串）              | ⚠️ 较复杂（需要对象）         |
| **推荐使用**  | 简单的时间运算                | 复杂的时间计算                |

### diff() 的 days vs d

| 特性         | days                          | d                             |
|:-------------|:------------------------------|:------------------------------|
| **含义**      | 总天数差（绝对值）            | 天数差（相对值，考虑年月）     |
| **范围**      | 0 到很大的数                  | 0-30（或 0-31）               |
| **用途**      | 计算总天数                    | 格式化显示时使用              |

## 练习任务

1. **时间差计算工具**：创建一个函数，计算两个日期之间的天数、小时数、分钟数，并格式化输出。

2. **工作日计算工具**：实现一个函数，计算两个日期之间的工作日数量，排除周末和节假日。

3. **年龄计算工具**：编写一个函数，根据生日计算年龄，考虑闰年情况。

4. **时间范围判断工具**：创建一个函数，判断一个日期是否在指定的时间范围内，支持开区间和闭区间。

5. **相对时间格式化**：实现一个函数，将时间差格式化为相对时间（如"2小时前"、"3天后"）。

## 相关章节

- **[4.1.1 DateTime 基础](section-01-datetime-basics.md)**：了解 DateTime 类的基础用法
- **[4.1.2 时区处理](section-02-timezone.md)**：了解时区对时间计算的影响
- **[4.1.3 格式化与解析](section-03-formatting-parsing.md)**：学习日期时间的格式化和解析
