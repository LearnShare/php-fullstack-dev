# 2.14.1 DateTime 基础

## 概述

PHP 提供了强大的日期和时间处理功能。`DateTime` 和 `DateTimeImmutable` 类是处理日期时间的现代方式，比传统的 `date()` 函数更强大和灵活。这两个类提供了面向对象的日期时间处理接口，支持时区、格式化、计算等完整功能。

## 特性

- **面向对象设计**：提供完整的面向对象接口，易于使用和维护
- **时区支持**：完整支持时区转换和夏令时（DST）处理
- **不可变对象**：`DateTimeImmutable` 提供不可变对象，避免意外修改
- **灵活创建**：支持多种时间字符串格式和时间戳创建
- **丰富方法**：提供格式化、计算、比较等完整方法集
- **自动验证**：自动验证日期时间的有效性
- **高性能**：相比传统函数，性能更优且功能更强大

## DateTime 类

### DateTime 构造函数

**语法**：`new DateTime(string $datetime = "now", ?DateTimeZone $timezone = null)`

**参数**：
- `$datetime`：可选，日期时间字符串，默认为 `"now"`。支持的格式包括：
  - `"now"`：当前时间
  - `"today"`：今天的 00:00:00
  - `"tomorrow"`：明天的 00:00:00
  - `"yesterday"`：昨天的 00:00:00
  - ISO 8601 格式：`"2024-01-15T10:30:00"`、`"2024-01-15T10:30:00+08:00"`
  - 标准格式：`"2024-01-15 10:30:00"`、`"2024-01-15"`
  - 相对时间：`"+1 day"`、`"-1 week"`、`"next Monday"`
  - Unix 时间戳：`"@1705285800"`（以 `@` 开头）
- `$timezone`：可选，`DateTimeZone` 对象，指定时区。如果为 `null`，使用默认时区。

**返回值**：返回 `DateTime` 对象实例。如果日期时间字符串无效，抛出 `Exception`。

### 创建 DateTime 对象

```php
<?php
declare(strict_types=1);

// 当前时间（使用默认时区）
$now = new DateTime();

// 指定时间字符串
$date = new DateTime('2024-01-15 10:30:00');

// 指定时区
$date = new DateTime('now', new DateTimeZone('Asia/Shanghai'));

// 使用相对时间
$date = new DateTime('+1 day');
$date = new DateTime('next Monday');

// 使用时间戳（以 @ 开头）
$date = new DateTime('@1705285800');

// 使用 ISO 8601 格式
$date = new DateTime('2024-01-15T10:30:00+08:00');
```

### format() - 格式化输出

**语法**：`format(string $format): string`

**参数**：
- `$format`：格式化字符串，使用特定字符表示日期时间各部分。详见 [格式化与解析](section-03-formatting-parsing.md) 章节。

**返回值**：返回格式化后的日期时间字符串。

**常用格式化字符**：
- `Y`：4 位数字年份（如：2024）
- `m`：带前导零的月份（01-12）
- `d`：带前导零的日期（01-31）
- `H`：24 小时制小时（00-23）
- `i`：分钟（00-59）
- `s`：秒（00-59）

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15 10:30:00');
echo $date->format('Y-m-d H:i:s') . "\n";  // 2024-01-15 10:30:00
echo $date->format('Y年m月d日') . "\n";    // 2024年01月15日
echo $date->format('l, F j, Y') . "\n";   // Monday, January 15, 2024
```

### modify() - 修改时间

**语法**：`modify(string $modifier): DateTime|false`

**参数**：
- `$modifier`：相对时间修改字符串，支持以下格式：
  - 相对时间：`"+1 day"`、`"-1 week"`、`"+2 months"`、`"+3 years"`
  - 相对时间点：`"next Monday"`、`"last Friday"`、`"first day of next month"`
  - 绝对时间：`"noon"`、`"midnight"`、`"start of day"`、`"end of day"`
  - 组合：`"+1 day +2 hours"`、`"next Monday +1 week"`

**返回值**：成功返回 `DateTime` 对象自身（支持链式调用），失败返回 `false`。

**注意**：`DateTime` 的 `modify()` 方法会修改原对象，而 `DateTimeImmutable` 的 `modify()` 方法返回新对象。

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15');
$date->modify('+1 day');
echo $date->format('Y-m-d') . "\n";  // 2024-01-16

$date->modify('+1 month');
echo $date->format('Y-m-d') . "\n";  // 2024-02-16

// 使用相对时间点
$date->modify('next Monday');
echo $date->format('Y-m-d') . "\n";  // 下一个周一的日期

// 组合使用
$date->modify('+1 day +2 hours');
echo $date->format('Y-m-d H:i:s') . "\n";
```

## DateTimeImmutable 类

### DateTimeImmutable 构造函数

**语法**：`new DateTimeImmutable(string $datetime = "now", ?DateTimeZone $timezone = null)`

**参数**：
- `$datetime`：可选，日期时间字符串，默认为 `"now"`。格式与 `DateTime` 构造函数相同。
- `$timezone`：可选，`DateTimeZone` 对象，指定时区。如果为 `null`，使用默认时区。

**返回值**：返回 `DateTimeImmutable` 对象实例。如果日期时间字符串无效，抛出 `Exception`。

### 不可变对象特性

`DateTimeImmutable` 每次操作返回新实例，不修改原对象。这是与 `DateTime` 的主要区别：

```php
<?php
declare(strict_types=1);

$date = new DateTimeImmutable('2024-01-15');
$newDate = $date->modify('+1 day');

echo $date->format('Y-m-d') . "\n";     // 2024-01-15（未改变）
echo $newDate->format('Y-m-d') . "\n";  // 2024-01-16（新对象）

// 原对象保持不变
var_dump($date === $newDate);  // false（不同的对象）
```

### modify() - 修改时间（返回新对象）

**语法**：`modify(string $modifier): DateTimeImmutable|false`

**参数**：
- `$modifier`：相对时间修改字符串，格式与 `DateTime::modify()` 相同。

**返回值**：成功返回新的 `DateTimeImmutable` 对象，失败返回 `false`。原对象不会被修改。

```php
<?php
declare(strict_types=1);

$date = new DateTimeImmutable('2024-01-15');
$newDate = $date->modify('+1 week');

// 原对象不变
echo $date->format('Y-m-d') . "\n";     // 2024-01-15
echo $newDate->format('Y-m-d') . "\n";  // 2024-01-22
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

### date_default_timezone_set() - 设置默认时区

**语法**：`date_default_timezone_set(string $timezone): bool`

**参数**：
- `$timezone`：时区标识符，如 `"Asia/Shanghai"`、`"UTC"`、`"America/New_York"`。

**返回值**：成功返回 `true`，失败返回 `false`。

```php
<?php
declare(strict_types=1);

// 设置默认时区
date_default_timezone_set('Asia/Shanghai');
echo date_default_timezone_get() . "\n";  // Asia/Shanghai
```

### setTimezone() - 设置时区

**语法**：`setTimezone(DateTimeZone $timezone): DateTime`

**参数**：
- `$timezone`：`DateTimeZone` 对象，指定要设置的时区。

**返回值**：返回 `DateTime` 对象自身（支持链式调用）。

**注意**：`DateTime` 的 `setTimezone()` 会修改原对象，`DateTimeImmutable` 的 `setTimezone()` 返回新对象。

```php
<?php
declare(strict_types=1);

// 创建带时区的 DateTime
$date = new DateTime('now', new DateTimeZone('UTC'));
echo $date->format('Y-m-d H:i:s T') . "\n";  // UTC 时间

// 转换时区（修改原对象）
$date->setTimezone(new DateTimeZone('Asia/Shanghai'));
echo $date->format('Y-m-d H:i:s T') . "\n";  // 北京时间

// DateTimeImmutable 返回新对象
$immutable = new DateTimeImmutable('now', new DateTimeZone('UTC'));
$newImmutable = $immutable->setTimezone(new DateTimeZone('Asia/Shanghai'));
echo $immutable->format('Y-m-d H:i:s T') . "\n";  // UTC 时间（未改变）
echo $newImmutable->format('Y-m-d H:i:s T') . "\n";  // 北京时间（新对象）
```

### getTimezone() - 获取时区

**语法**：`getTimezone(): DateTimeZone`

**参数**：无

**返回值**：返回当前 `DateTime` 对象使用的 `DateTimeZone` 对象。

```php
<?php
declare(strict_types=1);

$date = new DateTime('now', new DateTimeZone('Asia/Shanghai'));
$timezone = $date->getTimezone();
echo $timezone->getName() . "\n";  // Asia/Shanghai
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

### getTimestamp() - 获取时间戳

**语法**：`getTimestamp(): int`

**参数**：无

**返回值**：返回 Unix 时间戳（秒数，从 1970-01-01 00:00:00 UTC 开始）。

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15 10:30:00');
$timestamp = $date->getTimestamp();
echo $timestamp . "\n";  // 1705285800（示例值，实际值取决于时区）
```

### setTimestamp() - 设置时间戳

**语法**：`setTimestamp(int $timestamp): DateTime`

**参数**：
- `$timestamp`：Unix 时间戳（秒数）。

**返回值**：返回 `DateTime` 对象自身（支持链式调用）。

**注意**：`DateTime` 的 `setTimestamp()` 会修改原对象，`DateTimeImmutable` 的 `setTimestamp()` 返回新对象。

```php
<?php
declare(strict_types=1);

$timestamp = 1705285800;
$date = (new DateTime())->setTimestamp($timestamp);
echo $date->format('Y-m-d H:i:s') . "\n";  // 2024-01-15 10:30:00（取决于时区）

// 使用 DateTimeImmutable
$immutable = new DateTimeImmutable();
$newImmutable = $immutable->setTimestamp($timestamp);
echo $immutable->getTimestamp() . "\n";  // 当前时间戳（未改变）
echo $newImmutable->getTimestamp() . "\n";  // 1705285800（新对象）
```

### 从时间戳创建（使用 @ 前缀）

```php
<?php
declare(strict_types=1);

// 使用 @ 前缀直接创建
$timestamp = 1705285800;
$date = new DateTime('@' . $timestamp);
echo $date->format('Y-m-d H:i:s') . "\n";

// 注意：使用 @ 前缀时，时区会被设置为 UTC
echo $date->getTimezone()->getName() . "\n";  // UTC
```

## 时间差计算

### diff() - 计算时间差

**语法**：`diff(DateTimeInterface $targetObject, bool $absolute = false): DateInterval`

**参数**：
- `$targetObject`：要比较的目标 `DateTime` 或 `DateTimeImmutable` 对象。
- `$absolute`：可选，是否返回绝对值（忽略正负），默认为 `false`。如果为 `true`，返回的时间差始终为正数。

**返回值**：返回 `DateInterval` 对象，包含时间差的详细信息。详见 [时间计算与比较](section-04-calculations-comparisons.md) 章节。

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2024-01-15');
$date2 = new DateTime('2024-01-20');
$diff = $date1->diff($date2);

echo $diff->days . " days\n";        // 5 days
echo $diff->format('%a days') . "\n";  // 5 days

// 使用绝对值
$date3 = new DateTime('2024-01-20');
$date4 = new DateTime('2024-01-15');
$diff2 = $date3->diff($date4, true);  // 使用绝对值
echo $diff2->days . " days\n";        // 5 days（始终为正数）
```

## 其他常用方法

### add() - 添加时间间隔

**语法**：`add(DateInterval $interval): DateTime`

**参数**：
- `$interval`：`DateInterval` 对象，表示要添加的时间间隔。

**返回值**：返回 `DateTime` 对象自身（支持链式调用）。

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15');
$interval = new DateInterval('P1D');  // 1 天
$date->add($interval);
echo $date->format('Y-m-d') . "\n";  // 2024-01-16
```

### sub() - 减去时间间隔

**语法**：`sub(DateInterval $interval): DateTime`

**参数**：
- `$interval`：`DateInterval` 对象，表示要减去的时间间隔。

**返回值**：返回 `DateTime` 对象自身（支持链式调用）。

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15');
$interval = new DateInterval('P1D');  // 1 天
$date->sub($interval);
echo $date->format('Y-m-d') . "\n";  // 2024-01-14
```

### createFromFormat() - 从格式字符串创建

**语法**：`static createFromFormat(string $format, string $datetime, ?DateTimeZone $timezone = null): DateTime|false`

**参数**：
- `$format`：日期时间格式字符串，使用格式化字符定义输入格式。
- `$datetime`：要解析的日期时间字符串。
- `$timezone`：可选，`DateTimeZone` 对象，指定时区。

**返回值**：成功返回 `DateTime` 对象，失败返回 `false`。

```php
<?php
declare(strict_types=1);

// 从自定义格式解析
$date = DateTime::createFromFormat('d/m/Y', '15/01/2024');
if ($date !== false) {
    echo $date->format('Y-m-d') . "\n";  // 2024-01-15
}
```

### getOffset() - 获取时区偏移

**语法**：`getOffset(): int`

**参数**：无

**返回值**：返回时区偏移量（秒数）。正数表示 UTC 以东，负数表示 UTC 以西。

```php
<?php
declare(strict_types=1);

$date = new DateTime('now', new DateTimeZone('Asia/Shanghai'));
$offset = $date->getOffset();
echo ($offset / 3600) . " hours\n";  // 8 hours（UTC+8）
```

### getTimezone() - 获取时区

**语法**：`getTimezone(): DateTimeZone`

**参数**：无

**返回值**：返回当前 `DateTime` 对象使用的 `DateTimeZone` 对象。

```php
<?php
declare(strict_types=1);

$date = new DateTime('now', new DateTimeZone('Asia/Shanghai'));
$timezone = $date->getTimezone();
echo $timezone->getName() . "\n";  // Asia/Shanghai
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
        
        echo "\n=== 从格式字符串创建 ===\n";
        $date = DateTime::createFromFormat('d/m/Y', '15/01/2024');
        if ($date !== false) {
            echo $date->format('Y-m-d') . "\n";  // 2024-01-15
        }
        
        echo "\n=== 时区偏移 ===\n";
        $date = new DateTime('now', new DateTimeZone('Asia/Shanghai'));
        $offset = $date->getOffset();
        echo "Offset: " . ($offset / 3600) . " hours\n";  // 8 hours
    }
}

DateTimeBasics::demonstrate();
```

## 输出结果说明

运行上述示例代码，输出如下：

```
=== 创建 DateTime ===
2024-01-15 10:30:00

=== 格式化 ===
2024-01-15 10:30:00
Monday, January 15, 2024

=== 修改时间 ===
2024-01-22

=== DateTimeImmutable ===
Original: 2024-01-15
New: 2024-01-22

=== 时区 ===
2024-01-15 18:30:00 CST

=== 时间差 ===
5 days

=== 从格式字符串创建 ===
2024-01-15

=== 时区偏移 ===
Offset: 8 hours
```

**说明**：
- 创建 `DateTime` 时，如果不指定时区，使用系统默认时区
- 格式化输出时，时区信息会影响显示的时间
- `DateTimeImmutable` 的操作不会修改原对象
- 时间差计算返回 `DateInterval` 对象，包含详细的时间差信息

## 对比分析

### DateTime vs DateTimeImmutable

| 特性                 | DateTime            | DateTimeImmutable        |
|:---------------------|:--------------------|:-------------------------|
| **可变性**           | 可变对象            | 不可变对象               |
| **修改方法**         | 修改原对象          | 返回新对象               |
| **性能**             | 稍快（无需创建新对象）| 稍慢（每次操作创建新对象）|
| **内存使用**         | 较少                | 较多（可能产生多个对象）  |
| **安全性**           | 可能意外修改        | 更安全，避免意外修改     |
| **推荐使用场景**     | 临时计算、性能敏感  | 业务逻辑、函数参数       |

### DateTime vs date() 函数

| 特性                 | DateTime            | date() 函数              |
|:---------------------|:--------------------|:--------------------------|
| **面向对象**         | 是                  | 否（过程式）              |
| **时区支持**         | 完整支持            | 基础支持                 |
| **日期验证**         | 自动验证            | 需要手动验证             |
| **链式调用**         | 支持                | 不支持                   |
| **性能**             | 稍慢                | 稍快                     |
| **功能丰富度**       | 更丰富              | 基础功能                 |

## 常见问题

### 问题 1：为什么推荐使用 DateTimeImmutable？

**回答**：`DateTimeImmutable` 是不可变对象，每次操作返回新实例，不会修改原对象。这样可以避免在函数调用、对象传递等场景中意外修改日期时间，提高代码的安全性和可预测性。

```php
<?php
declare(strict_types=1);

// 使用 DateTime（可能意外修改）
function processDate(DateTime $date): void
{
    $date->modify('+1 day');  // 修改了原对象
}

$date = new DateTime('2024-01-15');
processDate($date);
echo $date->format('Y-m-d') . "\n";  // 2024-01-16（被修改了）

// 使用 DateTimeImmutable（安全）
function processDateImmutable(DateTimeImmutable $date): DateTimeImmutable
{
    return $date->modify('+1 day');  // 返回新对象
}

$date = new DateTimeImmutable('2024-01-15');
$newDate = processDateImmutable($date);
echo $date->format('Y-m-d') . "\n";     // 2024-01-15（未改变）
echo $newDate->format('Y-m-d') . "\n";  // 2024-01-16（新对象）
```

### 问题 2：如何验证日期字符串的有效性？

**回答**：使用 `DateTime::createFromFormat()` 或直接创建 `DateTime` 对象，如果日期无效会抛出异常或返回 `false`。

```php
<?php
declare(strict_types=1);

function isValidDate(string $dateString, string $format = 'Y-m-d'): bool
{
    $date = DateTime::createFromFormat($format, $dateString);
    return $date !== false && $date->format($format) === $dateString;
}

var_dump(isValidDate('2024-01-15'));  // true
var_dump(isValidDate('2024-13-45'));  // false（无效日期）
```

### 问题 3：如何处理时区转换？

**回答**：使用 `setTimezone()` 方法进行时区转换。注意 `DateTime` 会修改原对象，`DateTimeImmutable` 返回新对象。

```php
<?php
declare(strict_types=1);

// 存储 UTC 时间
$utc = new DateTimeImmutable('now', new DateTimeZone('UTC'));

// 转换为用户时区
$userTz = $utc->setTimezone(new DateTimeZone('Asia/Shanghai'));
echo $userTz->format('Y-m-d H:i:s T') . "\n";
```

### 问题 4：如何比较两个日期？

**回答**：可以直接使用比较运算符（`<`、`>`、`==`）或使用 `getTimestamp()` 方法比较时间戳。

```php
<?php
declare(strict_types=1);

$date1 = new DateTime('2024-01-15');
$date2 = new DateTime('2024-01-20');

// 使用比较运算符
var_dump($date1 < $date2);  // true

// 使用时间戳
var_dump($date1->getTimestamp() < $date2->getTimestamp());  // true
```

## 最佳实践

### 1. 优先使用 DateTimeImmutable

在业务逻辑中优先使用 `DateTimeImmutable`，避免意外的修改：

```php
<?php
declare(strict_types=1);

// 推荐：使用 DateTimeImmutable
function calculateDeadline(DateTimeImmutable $start, int $days): DateTimeImmutable
{
    return $start->modify("+{$days} days");
}
```

### 2. 明确设置时区

始终明确设置时区，避免依赖系统默认值：

```php
<?php
declare(strict_types=1);

// 推荐：明确指定时区
$date = new DateTime('now', new DateTimeZone('Asia/Shanghai'));

// 不推荐：依赖系统默认时区
$date = new DateTime('now');
```

### 3. 存储 UTC 时间

在数据库中存储 UTC 时间，显示时再转换为用户时区：

```php
<?php
declare(strict_types=1);

// 存储时使用 UTC
$utc = new DateTimeImmutable('now', new DateTimeZone('UTC'));
$dbValue = $utc->format('Y-m-d H:i:s');

// 显示时转换为用户时区
$userTz = $utc->setTimezone(new DateTimeZone('Asia/Shanghai'));
echo $userTz->format('Y-m-d H:i:s');
```

### 4. 使用类型声明

在函数中使用类型声明，提高代码可读性和安全性：

```php
<?php
declare(strict_types=1);

function formatDate(DateTimeImmutable $date): string
{
    return $date->format('Y-m-d');
}
```

### 5. 错误处理

创建 `DateTime` 对象时进行错误处理：

```php
<?php
declare(strict_types=1);

try {
    $date = new DateTime('invalid-date');
} catch (Exception $e) {
    echo "Invalid date: " . $e->getMessage() . "\n";
}

// 或使用 createFromFormat
$date = DateTime::createFromFormat('Y-m-d', 'invalid-date');
if ($date === false) {
    echo "Invalid date format\n";
}
```

## 注意事项

1. **时区设置**：始终明确设置时区，避免依赖系统默认值。不同服务器可能有不同的默认时区。

2. **不可变对象**：优先使用 `DateTimeImmutable`，避免意外的修改。特别是在函数参数和返回值中。

3. **格式化字符串**：注意格式化字符串的大小写，如 `Y`（4位年份）和 `y`（2位年份）是不同的。

4. **性能考虑**：`DateTime` 比 `date()` 函数稍慢，但功能更强大。在性能敏感的场景中可以考虑使用 `date()`。

5. **日期验证**：使用 `DateTime` 可以自动验证日期的有效性，无效日期会抛出异常。

6. **时区转换**：进行时区转换时，注意 `DateTime` 会修改原对象，`DateTimeImmutable` 返回新对象。

7. **时间戳精度**：`getTimestamp()` 返回秒级时间戳，如果需要微秒精度，使用 `getTimestamp()` 结合 `format('u')`。

## 练习

1. 创建一个日期工具类，封装常用的日期操作（创建、格式化、计算等）。

2. 编写一个函数，计算两个日期之间的工作日数量（排除周末）。

3. 实现一个函数，格式化相对时间（如"2小时前"、"3天前"、"刚刚"）。

4. 创建一个函数，处理不同时区的时间转换，支持 UTC 和任意时区之间的转换。

5. 编写一个函数，验证日期字符串的有效性，支持多种日期格式（`Y-m-d`、`m/d/Y`、`d-m-Y` 等）。
