# 4.1.1 DateTime 基础

## 概述

DateTime 类是 PHP 5.2.0 引入的面向对象日期时间处理类，提供了比传统 `date()` 函数更强大和灵活的功能。DateTime 类支持时区转换、格式化、时间计算、比较等完整功能，是现代 PHP 应用中处理日期时间的标准方式。

理解 DateTime 类对于掌握 PHP 系统编程至关重要。日期和时间处理是几乎所有应用的核心需求，无论是记录日志时间、处理用户输入的日期、计算时间差、还是进行时区转换，都需要使用 DateTime 类。

DateTime 类提供了完整的面向对象接口，使得日期时间操作更加直观和安全。与传统的 `date()`、`strtotime()` 等函数相比，DateTime 类提供了更好的类型安全、更丰富的功能，以及更好的错误处理机制。

**主要内容**：
- DateTime 类和 DateTimeImmutable 类的概述
- 创建 DateTime 对象的多种方式
- 获取当前时间和指定时间
- DateTime 对象的基本属性访问
- 时间格式化方法（format）
- DateTime 对象的常用方法
- DateTime 与 DateTimeImmutable 的区别
- 完整示例和最佳实践

## 特性

- **面向对象设计**：提供完整的面向对象接口，易于使用和维护
- **时区支持**：完整支持时区转换和夏令时（DST）处理
- **不可变对象**：`DateTimeImmutable` 提供不可变对象，避免意外修改
- **灵活创建**：支持多种时间字符串格式和时间戳创建
- **丰富方法**：提供格式化、计算、比较等完整方法集
- **自动验证**：自动验证日期时间的有效性
- **类型安全**：使用类型声明，提高代码安全性

## 语法/定义

### DateTime 构造函数

**语法**：`new DateTime(string $datetime = "now", ?DateTimeZone $timezone = null)`

**参数**：
- `$datetime`：可选，日期时间字符串，默认为 `"now"`。支持的格式包括：
  - `"now"`：当前时间
  - `"today"`：今天的 00:00:00
  - `"tomorrow"`：明天的 00:00:00
  - `"yesterday"`：昨天的 00:00:00
  - ISO 8601 格式：`"2025-01-15T10:30:00"`、`"2025-01-15T10:30:00+08:00"`
  - 标准格式：`"2025-01-15 10:30:00"`、`"2025-01-15"`
  - 相对时间：`"+1 day"`、`"-1 week"`、`"next Monday"`
  - Unix 时间戳：`"@1705285800"`（以 `@` 开头）
- `$timezone`：可选，`DateTimeZone` 对象，指定时区。如果为 `null`，使用默认时区。

**返回值**：返回 `DateTime` 对象实例。如果日期时间字符串无效，抛出 `Exception`。

### DateTimeImmutable 构造函数

**语法**：`new DateTimeImmutable(string $datetime = "now", ?DateTimeZone $timezone = null)`

**参数**：与 `DateTime` 构造函数相同。

**返回值**：返回 `DateTimeImmutable` 对象实例。

**区别**：`DateTimeImmutable` 是不可变对象，所有修改操作都返回新对象，不修改原对象。

## 基本用法

### 示例 1：创建当前时间的 DateTime 对象

```php
<?php
declare(strict_types=1);

// 创建当前时间的 DateTime 对象（使用默认时区）
$now = new DateTime();
echo $now->format('Y-m-d H:i:s') . "\n";

// 创建当前时间的 DateTimeImmutable 对象
$nowImmutable = new DateTimeImmutable();
echo $nowImmutable->format('Y-m-d H:i:s') . "\n";

// 使用 "now" 字符串显式指定
$now2 = new DateTime('now');
echo $now2->format('Y-m-d H:i:s') . "\n";
```

**输出**：

```
2025-01-15 14:30:00
2025-01-15 14:30:00
2025-01-15 14:30:00
```

**说明**：
- `new DateTime()` 和 `new DateTime('now')` 效果相同，都创建当前时间的对象
- DateTime 和 DateTimeImmutable 的创建方式相同
- 使用默认时区（可以通过 `date_default_timezone_set()` 或 php.ini 配置）

### 示例 2：创建指定时间的 DateTime 对象

```php
<?php
declare(strict_types=1);

// 使用标准日期格式
$date1 = new DateTime('2025-01-15');
echo $date1->format('Y-m-d H:i:s') . "\n";  // 2025-01-15 00:00:00

// 使用日期时间格式
$date2 = new DateTime('2025-01-15 10:30:00');
echo $date2->format('Y-m-d H:i:s') . "\n";  // 2025-01-15 10:30:00

// 使用 ISO 8601 格式
$date3 = new DateTime('2025-01-15T10:30:00');
echo $date3->format('Y-m-d H:i:s') . "\n";  // 2025-01-15 10:30:00

// 使用相对时间字符串
$date4 = new DateTime('tomorrow');
echo $date4->format('Y-m-d') . "\n";  // 明天的日期

$date5 = new DateTime('next Monday');
echo $date5->format('Y-m-d') . "\n";  // 下一个周一的日期

// 使用时间戳（以 @ 开头）
$date6 = new DateTime('@1705285800');
echo $date6->format('Y-m-d H:i:s') . "\n";  // 对应时间戳的日期时间
```

**输出**：

```
2025-01-15 00:00:00
2025-01-15 10:30:00
2025-01-15 10:30:00
2025-01-16
2025-01-20
2024-01-15 10:30:00
```

**说明**：
- 支持多种日期时间字符串格式
- 相对时间字符串（如 `"tomorrow"`、`"next Monday"`）会自动计算
- 使用 `@` 前缀可以从 Unix 时间戳创建

### 示例 3：使用 DateTimeZone 指定时区

```php
<?php
declare(strict_types=1);

// 创建指定时区的 DateTime 对象
$date = new DateTime('2025-01-15 10:30:00', new DateTimeZone('Asia/Shanghai'));
echo $date->format('Y-m-d H:i:s T') . "\n";  // 2025-01-15 10:30:00 CST

// 创建 UTC 时区的 DateTime 对象
$utc = new DateTime('2025-01-15 10:30:00', new DateTimeZone('UTC'));
echo $utc->format('Y-m-d H:i:s T') . "\n";  // 2025-01-15 10:30:00 UTC

// 使用 DateTimeImmutable
$immutable = new DateTimeImmutable('2025-01-15 10:30:00', new DateTimeZone('America/New_York'));
echo $immutable->format('Y-m-d H:i:s T') . "\n";  // 2025-01-15 10:30:00 EST
```

**输出**：

```
2025-01-15 10:30:00 CST
2025-01-15 10:30:00 UTC
2025-01-15 10:30:00 EST
```

**说明**：
- 可以通过 `DateTimeZone` 对象指定时区
- 时区会影响时间的显示和计算
- 常见的时区标识符包括：`"UTC"`、`"Asia/Shanghai"`、`"America/New_York"` 等

### 示例 4：format() 方法格式化日期时间

```php
<?php
declare(strict_types=1);

$date = new DateTime('2025-01-15 14:30:45');

// 标准格式
echo $date->format('Y-m-d H:i:s') . "\n";  // 2025-01-15 14:30:45

// 仅日期
echo $date->format('Y-m-d') . "\n";  // 2025-01-15

// 仅时间
echo $date->format('H:i:s') . "\n";  // 14:30:45

// 12 小时制
echo $date->format('Y-m-d g:i:s A') . "\n";  // 2025-01-15 2:30:45 PM

// 星期和月份名称
echo $date->format('l, F j, Y') . "\n";  // Wednesday, January 15, 2025

// 中文格式（需要系统支持）
echo $date->format('Y年m月d日 H:i:s') . "\n";  // 2025年01月15日 14:30:45

// 时间戳
echo $date->format('U') . "\n";  // Unix 时间戳（秒）

// ISO 8601 格式
echo $date->format('c') . "\n";  // 2025-01-15T14:30:45+08:00

// RFC 2822 格式
echo $date->format('r') . "\n";  // Wed, 15 Jan 2025 14:30:45 +0800
```

**输出**：

```
2025-01-15 14:30:45
2025-01-15
14:30:45
2025-01-15 2:30:45 PM
Wednesday, January 15, 2025
2025年01月15日 14:30:45
1736931045
2025-01-15T14:30:45+08:00
Wed, 15 Jan 2025 14:30:45 +0800
```

**说明**：
- `format()` 方法使用格式化字符串来输出日期时间的各个部分
- 常用格式化字符：
  - `Y`：4 位年份
  - `m`：2 位月份（01-12）
  - `d`：2 位日期（01-31）
  - `H`：24 小时制小时（00-23）
  - `i`：分钟（00-59）
  - `s`：秒（00-59）
  - `l`：完整的星期名称
  - `F`：完整的月份名称

### 示例 5：getTimestamp() 和 setTimestamp()

```php
<?php
declare(strict_types=1);

// 获取时间戳
$date = new DateTime('2025-01-15 10:30:00');
$timestamp = $date->getTimestamp();
echo "时间戳: {$timestamp}\n";  // Unix 时间戳（秒）

// 从时间戳创建 DateTime
$timestamp = 1705285800;
$date2 = new DateTime();
$date2->setTimestamp($timestamp);
echo $date2->format('Y-m-d H:i:s') . "\n";

// 使用 @ 前缀从时间戳创建（注意时区会设置为 UTC）
$date3 = new DateTime('@' . $timestamp);
echo $date3->format('Y-m-d H:i:s T') . "\n";  // UTC 时区
echo $date3->getTimezone()->getName() . "\n";  // UTC
```

**输出**：

```
时间戳: 1736899800
2025-01-15 10:30:00
2025-01-15 10:30:00 UTC
UTC
```

**说明**：
- `getTimestamp()` 返回 Unix 时间戳（从 1970-01-01 00:00:00 UTC 开始的秒数）
- `setTimestamp()` 从时间戳设置日期时间
- 使用 `@` 前缀创建时，时区会被设置为 UTC

### 示例 6：DateTime 与 DateTimeImmutable 的区别

```php
<?php
declare(strict_types=1);

// DateTime：修改原对象
$date = new DateTime('2025-01-15');
$date->modify('+1 day');
echo "DateTime: " . $date->format('Y-m-d') . "\n";  // 2025-01-16（原对象被修改）

// DateTimeImmutable：返回新对象
$immutable = new DateTimeImmutable('2025-01-15');
$newImmutable = $immutable->modify('+1 day');
echo "原对象: " . $immutable->format('Y-m-d') . "\n";     // 2025-01-15（未改变）
echo "新对象: " . $newImmutable->format('Y-m-d') . "\n";  // 2025-01-16（新对象）

// 验证对象是否相同
var_dump($immutable === $newImmutable);  // false（不同的对象）
```

**输出**：

```
DateTime: 2025-01-16
原对象: 2025-01-15
新对象: 2025-01-16
bool(false)
```

**说明**：
- `DateTime` 的修改方法（如 `modify()`、`setTimestamp()`）会修改原对象
- `DateTimeImmutable` 的所有修改方法都返回新对象，原对象保持不变
- 推荐在业务逻辑中使用 `DateTimeImmutable`，避免意外的修改

### 示例 7：获取日期时间的各个部分

```php
<?php
declare(strict_types=1);

$date = new DateTime('2025-01-15 14:30:45');

// 获取年份
echo "年份: " . $date->format('Y') . "\n";  // 2025

// 获取月份
echo "月份: " . $date->format('m') . "\n";  // 01（带前导零）
echo "月份: " . $date->format('n') . "\n";  // 1（不带前导零）

// 获取日期
echo "日期: " . $date->format('d') . "\n";  // 15

// 获取星期
echo "星期: " . $date->format('l') . "\n";  // Wednesday（完整名称）
echo "星期: " . $date->format('D') . "\n";  // Wed（缩写）

// 获取小时
echo "小时（24制）: " . $date->format('H') . "\n";  // 14
echo "小时（12制）: " . $date->format('g') . "\n";  // 2（不带前导零）

// 获取分钟
echo "分钟: " . $date->format('i') . "\n";  // 30

// 获取秒
echo "秒: " . $date->format('s') . "\n";  // 45

// 获取时区
echo "时区: " . $date->getTimezone()->getName() . "\n";

// 获取时区偏移（秒）
$offset = $date->getOffset();
echo "时区偏移: " . ($offset / 3600) . " 小时\n";
```

**输出**：

```
年份: 2025
月份: 01
月份: 1
日期: 15
星期: Wednesday
星期: Wed
小时（24制）: 14
小时（12制）: 2
分钟: 30
秒: 45
时区: Asia/Shanghai
时区偏移: 8 小时
```

**说明**：
- 使用 `format()` 方法可以获取日期时间的各个部分
- `getTimezone()` 返回时区对象
- `getOffset()` 返回时区偏移（秒数）

### 示例 8：使用相对时间字符串

```php
<?php
declare(strict_types=1);

// 相对日期
$tomorrow = new DateTime('tomorrow');
echo "明天: " . $tomorrow->format('Y-m-d') . "\n";

$yesterday = new DateTime('yesterday');
echo "昨天: " . $yesterday->format('Y-m-d') . "\n";

// 相对时间
$nextWeek = new DateTime('+1 week');
echo "一周后: " . $nextWeek->format('Y-m-d') . "\n";

$lastMonth = new DateTime('-1 month');
echo "一个月前: " . $lastMonth->format('Y-m-d') . "\n";

// 相对时间点
$nextMonday = new DateTime('next Monday');
echo "下一个周一: " . $nextMonday->format('Y-m-d l') . "\n";

$lastFriday = new DateTime('last Friday');
echo "上一个周五: " . $lastFriday->format('Y-m-d l') . "\n";

// 组合相对时间
$nextWeekMonday = new DateTime('next Monday +1 week');
echo "下下周周一: " . $nextWeekMonday->format('Y-m-d l') . "\n";
```

**输出**：

```
明天: 2025-01-16
昨天: 2025-01-14
一周后: 2025-01-22
一个月前: 2024-12-15
下一个周一: 2025-01-20 Monday
上一个周五: 2025-01-10 Friday
下下周周一: 2025-01-27 Monday
```

**说明**：
- 支持丰富的相对时间字符串
- 可以组合多个相对时间表达式
- 相对时间会自动计算正确的日期

## 使用场景

### 场景 1：记录事件时间

使用 DateTime 记录事件发生的时间。

**示例**：

```php
<?php
declare(strict_types=1);

class Event
{
    public function __construct(
        private string $name,
        private DateTimeImmutable $occurredAt
    ) {}
    
    public function getOccurredAt(): DateTimeImmutable
    {
        return $this->occurredAt;
    }
    
    public function getOccurredAtFormatted(): string
    {
        return $this->occurredAt->format('Y-m-d H:i:s');
    }
}

// 创建事件
$event = new Event('User Login', new DateTimeImmutable());
echo "事件: {$event->getOccurredAtFormatted()}\n";
```

### 场景 2：处理用户输入的日期

从用户输入的日期字符串创建 DateTime 对象。

**示例**：

```php
<?php
declare(strict_types=1);

function parseUserDate(string $dateString): ?DateTimeImmutable
{
    try {
        return new DateTimeImmutable($dateString);
    } catch (Exception $e) {
        return null;
    }
}

$userInput = '2025-01-15';
$date = parseUserDate($userInput);
if ($date !== null) {
    echo "解析成功: " . $date->format('Y-m-d') . "\n";
} else {
    echo "日期格式无效\n";
}
```

## 注意事项

### 时区设置

- **默认时区**：如果不指定时区，使用系统默认时区
- **明确指定**：建议始终明确指定时区，避免依赖系统配置
- **UTC 存储**：在数据库中存储 UTC 时间，显示时再转换为用户时区

**示例**：

```php
<?php
declare(strict_types=1);

// 不推荐：依赖默认时区
$date = new DateTime('2025-01-15 10:30:00');

// 推荐：明确指定时区
$date = new DateTime('2025-01-15 10:30:00', new DateTimeZone('Asia/Shanghai'));

// 存储 UTC 时间
$utc = new DateTimeImmutable('now', new DateTimeZone('UTC'));
```

### DateTime vs DateTimeImmutable

- **DateTime**：修改方法会修改原对象
- **DateTimeImmutable**：所有修改返回新对象，原对象不变
- **推荐使用 DateTimeImmutable**：在业务逻辑中避免意外的修改

**示例**：

```php
<?php
declare(strict_types=1);

// DateTime：可能意外修改
function processDate(DateTime $date): void
{
    $date->modify('+1 day');  // 修改了原对象
}

$date = new DateTime('2025-01-15');
processDate($date);
echo $date->format('Y-m-d') . "\n";  // 2025-01-16（被修改了）

// DateTimeImmutable：安全
function processDateImmutable(DateTimeImmutable $date): DateTimeImmutable
{
    return $date->modify('+1 day');  // 返回新对象
}

$date = new DateTimeImmutable('2025-01-15');
$newDate = processDateImmutable($date);
echo $date->format('Y-m-d') . "\n";     // 2025-01-15（未改变）
echo $newDate->format('Y-m-d') . "\n";  // 2025-01-16（新对象）
```

### 日期字符串格式

- **支持的格式**：支持多种日期时间字符串格式
- **格式验证**：无效格式会抛出异常
- **使用 createFromFormat**：对于特定格式，使用 `DateTime::createFromFormat()` 更安全

**示例**：

```php
<?php
declare(strict_types=1);

// 标准格式（自动识别）
$date1 = new DateTime('2025-01-15 10:30:00');

// 特定格式（使用 createFromFormat）
$date2 = DateTime::createFromFormat('d/m/Y', '15/01/2025');
if ($date2 !== false) {
    echo $date2->format('Y-m-d') . "\n";  // 2025-01-15
}
```

## 常见问题

### 问题 1：如何获取当前时间？

**回答**：使用 `new DateTime()` 或 `new DateTime('now')` 创建当前时间的对象。

**示例**：

```php
<?php
declare(strict_types=1);

$now = new DateTime();
echo $now->format('Y-m-d H:i:s') . "\n";
```

### 问题 2：DateTime 和 date() 函数的区别是什么？

**回答**：
- **DateTime**：面向对象，支持时区、更丰富的功能，类型安全
- **date()**：函数式，不支持时区对象，功能相对简单，性能稍好

**示例**：

```php
<?php
declare(strict_types=1);

// 使用 date() 函数
echo date('Y-m-d H:i:s') . "\n";

// 使用 DateTime 类
$date = new DateTime();
echo $date->format('Y-m-d H:i:s') . "\n";

// DateTime 支持时区
$date = new DateTime('now', new DateTimeZone('UTC'));
echo $date->format('Y-m-d H:i:s T') . "\n";
```

### 问题 3：为什么推荐使用 DateTimeImmutable？

**回答**：`DateTimeImmutable` 是不可变对象，每次操作返回新实例，不会修改原对象。这样可以避免在函数调用、对象传递等场景中意外修改日期时间，提高代码的安全性和可预测性。

**示例**：见"示例 6：DateTime 与 DateTimeImmutable 的区别"

### 问题 4：如何处理无效的日期字符串？

**回答**：使用 try-catch 捕获异常，或使用 `DateTime::createFromFormat()` 并检查返回值。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法 1：使用 try-catch
try {
    $date = new DateTime('invalid-date');
} catch (Exception $e) {
    echo "无效日期: " . $e->getMessage() . "\n";
}

// 方法 2：使用 createFromFormat
$date = DateTime::createFromFormat('Y-m-d', 'invalid-date');
if ($date === false) {
    echo "日期格式无效\n";
}
```

## 最佳实践

### 1. 优先使用 DateTimeImmutable

在业务逻辑中优先使用 `DateTimeImmutable`，避免意外的修改。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：使用 DateTimeImmutable
function calculateDeadline(DateTimeImmutable $start, int $days): DateTimeImmutable
{
    return $start->modify("+{$days} days");
}

$start = new DateTimeImmutable('2025-01-15');
$deadline = calculateDeadline($start, 7);
echo $deadline->format('Y-m-d') . "\n";  // 2025-01-22
```

### 2. 明确设置时区

始终明确设置时区，避免依赖系统默认值。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：明确指定时区
$date = new DateTime('now', new DateTimeZone('Asia/Shanghai'));

// 不推荐：依赖系统默认时区
$date = new DateTime('now');
```

### 3. 存储 UTC 时间

在数据库中存储 UTC 时间，显示时再转换为用户时区。

**示例**：

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

在函数中使用类型声明，提高代码可读性和安全性。

**示例**：

```php
<?php
declare(strict_types=1);

function formatDate(DateTimeImmutable $date): string
{
    return $date->format('Y-m-d');
}

function isValidDate(DateTimeImmutable $date): bool
{
    return $date instanceof DateTimeImmutable;
}
```

### 5. 错误处理

创建 DateTime 对象时进行错误处理。

**示例**：

```php
<?php
declare(strict_types=1);

function safeCreateDateTime(string $dateString): ?DateTimeImmutable
{
    try {
        return new DateTimeImmutable($dateString);
    } catch (Exception $e) {
        error_log("Invalid date string: {$dateString} - " . $e->getMessage());
        return null;
    }
}

$date = safeCreateDateTime('2025-01-15');
if ($date !== null) {
    echo $date->format('Y-m-d') . "\n";
}
```

## 对比分析

### DateTime vs DateTimeImmutable

| 特性         | DateTime                         | DateTimeImmutable                |
|:-------------|:--------------------------------|:--------------------------------|
| **可变性**    | ✅ 可变（修改原对象）              | ❌ 不可变（返回新对象）           |
| **性能**      | 稍快（不需要创建新对象）            | 稍慢（需要创建新对象）            |
| **安全性**    | ⚠️ 可能意外修改                   | ✅ 不会意外修改                   |
| **推荐使用**  | 简单脚本                         | 业务逻辑、函数参数                |

### DateTime vs date() 函数

| 特性         | DateTime                         | date() 函数                      |
|:-------------|:--------------------------------|:--------------------------------|
| **设计**      | 面向对象                         | 函数式                           |
| **时区支持**  | ✅ 完整的时区对象支持              | ⚠️ 仅支持时区字符串               |
| **功能**      | ✅ 丰富（格式化、计算、比较）       | ⚠️ 相对简单                       |
| **类型安全**  | ✅ 类型声明支持                   | ❌ 无类型支持                     |
| **性能**      | 稍慢                             | 稍快                             |
| **推荐使用**  | ✅ 现代 PHP 应用                 | 简单脚本、性能敏感场景            |

## 练习任务

1. **创建日期工具类**：创建一个 `DateHelper` 类，封装常用的日期操作（创建、格式化、获取当前时间等）。

2. **日期验证函数**：编写一个函数，验证日期字符串的有效性，支持多种日期格式。

3. **相对时间格式化**：实现一个函数，将 DateTime 对象格式化为相对时间（如"2小时前"、"3天前"、"刚刚"）。

4. **时区转换工具**：创建一个函数，处理不同时区的时间转换，支持 UTC 和任意时区之间的转换。

5. **日期范围生成**：编写一个函数，生成指定日期范围内的所有日期（如从 2025-01-01 到 2025-01-31）。

## 相关章节

- **[4.1.2 时区处理](section-02-timezone.md)**：深入学习时区处理的详细内容
- **[4.1.3 格式化与解析](section-03-formatting-parsing.md)**：学习日期时间的格式化和解析
- **[4.1.4 时间计算与比较](section-04-calculations-comparisons.md)**：学习时间的计算和比较操作
