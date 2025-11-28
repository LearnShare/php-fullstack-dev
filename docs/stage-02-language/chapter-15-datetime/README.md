# 2.15 时间与日期处理

## 目标

- 掌握基于 `DateTime`、`DateTimeImmutable` 的现代日期处理方式。
- 理解时区、DST、时间戳、格式化字符串。
- 能使用 `Carbon` 等扩展库处理复杂时间逻辑。

## 时间戳与基础函数

- `time()`：返回当前 Unix 时间戳（秒）。
- `microtime(true)`：返回浮点数，包含微秒。
- `date($format, $timestamp = time())`：格式化时间戳。

示例：

```php
echo date('Y-m-d H:i:s');           // 2025-11-28 14:30:00
echo gmdate('c');                  // ISO8601，UTC 时间
```

## DateTime / DateTimeImmutable

```php
$now = new DateTime('now', new DateTimeZone('Asia/Shanghai'));
$deadline = (new DateTimeImmutable('2025-12-01'))->setTime(18, 0);
```

- `DateTime` 可变；`DateTimeImmutable` 每次操作返回新实例。
- 建议在业务逻辑中优先使用不可变对象。

### 常用方法

- `->format('Y-m-d H:i:s')`：格式化。
- `->modify('+2 days')`：相对时间。
- `->setTimezone(new DateTimeZone('UTC'))`：切换时区。
- `->diff($other)`：返回 `DateInterval`，可获取天数、月数等。

## 时间间隔（DateInterval）

- 语法：`new DateInterval('P2DT3H')`（ISO8601）
  - `P` 开头，`T` 分隔日期/时间部分。
  - `P2D` 表示 2 天，`PT3H` 表示 3 小时。
- 与 `DatePeriod` 搭配创建时间序列。

## 时区

- `date_default_timezone_set('Asia/Shanghai');`
- `DateTimeZone::listIdentifiers()` 查看可用时区。
- 应该在入口文件统一设置时区，避免默认 `UTC` 与 `php.ini` 不一致。

## 国际化时间格式

- `IntlDateFormatter`：

```php
$formatter = new IntlDateFormatter(
    'zh_CN',
    IntlDateFormatter::LONG,
    IntlDateFormatter::SHORT,
    'Asia/Shanghai',
);
echo $formatter->format(new DateTimeImmutable('now'));
```

## 常见格式占位符

| 代码 | 含义             | 示例        |
| :--- | :--------------- | :---------- |
| `Y`  | 四位年份        | 2025        |
| `m`  | 月（01-12）      | 11          |
| `d`  | 日（01-31）      | 28          |
| `H`  | 24 小时制小时   | 17          |
| `i`  | 分钟（00-59）    | 05          |
| `s`  | 秒（00-59）      | 22          |
| `c`  | ISO8601          | 2025-11-28T17:05:22+08:00 |

## Carbon 日期库详细使用

### 什么是 Carbon

- **Carbon**：`nesbot/carbon` 是 PHP 最流行的日期时间库
- **特点**：扩展了 `DateTime`，提供链式 API、人性化时间、多语言支持
- **安装**：`composer require nesbot/carbon`

### 基础使用

```php
<?php
declare(strict_types=1);

use Carbon\Carbon;
use Carbon\CarbonImmutable;

// 创建当前时间
$now = Carbon::now();
$nowImmutable = CarbonImmutable::now();

// 创建指定时间
$date = Carbon::create(2025, 1, 15, 14, 30, 0);
$date = Carbon::parse('2025-01-15 14:30:00');
$date = Carbon::parse('next Monday');

// 时区设置
$shanghai = Carbon::now('Asia/Shanghai');
$utc = Carbon::now('UTC');
```

### 链式操作

```php
<?php
declare(strict_types=1);

use Carbon\Carbon;

// 链式操作
$nextWeek = Carbon::now()
    ->addWeek()
    ->startOfDay()
    ->setTimezone('UTC');

// 时间计算
$future = Carbon::now()
    ->addDays(7)
    ->addHours(3)
    ->subMinutes(30);

// 时间比较
$isPast = Carbon::parse('2024-01-01')->isPast();
$isFuture = Carbon::parse('2026-01-01')->isFuture();
$isToday = Carbon::now()->isToday();
$isWeekend = Carbon::now()->isWeekend();
```

### 格式化输出

```php
<?php
declare(strict_types=1);

use Carbon\Carbon;

$date = Carbon::now();

// 标准格式化
echo $date->format('Y-m-d H:i:s'); // 2025-01-15 14:30:00
echo $date->toDateString();        // 2025-01-15
echo $date->toTimeString();         // 14:30:00
echo $date->toDateTimeString();    // 2025-01-15 14:30:00
echo $date->toIso8601String();     // ISO8601 格式

// 人性化时间
echo $date->diffForHumans();       // "刚刚"、"1小时前"、"2天后"
echo $date->fromNow();              // "从现在起 2 小时"
echo $date->toNow();                // "2 小时前"
```

### 时间差计算

```php
<?php
declare(strict_types=1);

use Carbon\Carbon;

$start = Carbon::parse('2025-01-01');
$end = Carbon::parse('2025-01-15');

// 计算差值
$diff = $start->diffInDays($end);        // 14 天
$diff = $start->diffInHours($end);        // 336 小时
$diff = $start->diffInMinutes($end);      // 20160 分钟
$diff = $start->diffInSeconds($end);      // 1209600 秒

// 人性化差值
echo $start->diffForHumans($end);        // "14 天前"
echo $end->diffForHumans($start);         // "14 天后"
```

### 时间范围判断

```php
<?php
declare(strict_types=1);

use Carbon\Carbon;

$date = Carbon::now();

// 判断是否在范围内
$start = Carbon::parse('2025-01-01');
$end = Carbon::parse('2025-12-31');

$isBetween = $date->between($start, $end);
$isBetweenInclusive = $date->betweenIncluded($start, $end);

// 判断是否在时间范围内
$isBetweenTime = $date->between(
    Carbon::parse('09:00'),
    Carbon::parse('18:00')
);
```

### 工作日和节假日

```php
<?php
declare(strict_types=1);

use Carbon\Carbon;

$date = Carbon::now();

// 判断工作日
$isWeekday = $date->isWeekday();   // 周一到周五
$isWeekend = $date->isWeekend();   // 周六和周日

// 获取下一个工作日
$nextWeekday = $date->nextWeekday();

// 获取上一个工作日
$prevWeekday = $date->previousWeekday();

// 添加工作日（跳过周末）
$nextBusinessDay = $date->addWeekdays(5);
```

### 时间周期

```php
<?php
declare(strict_types=1);

use Carbon\Carbon;

$date = Carbon::now();

// 获取周期开始和结束
$startOfDay = $date->startOfDay();        // 00:00:00
$endOfDay = $date->endOfDay();            // 23:59:59
$startOfWeek = $date->startOfWeek();      // 本周一
$endOfWeek = $date->endOfWeek();          // 本周日
$startOfMonth = $date->startOfMonth();    // 本月第一天
$endOfMonth = $date->endOfMonth();        // 本月最后一天
$startOfYear = $date->startOfYear();      // 今年第一天
$endOfYear = $date->endOfYear();          // 今年最后一天
```

### 多语言支持

```php
<?php
declare(strict_types=1);

use Carbon\Carbon;

// 设置语言
Carbon::setLocale('zh_CN');

$date = Carbon::now();
echo $date->locale('zh_CN')->diffForHumans(); // "刚刚"
echo $date->locale('en')->diffForHumans();      // "just now"

// 格式化月份和星期
echo $date->locale('zh_CN')->format('F');     // "一月"
echo $date->locale('en')->format('F');          // "January"
```

### 不可变对象（推荐）

```php
<?php
declare(strict_types=1);

use Carbon\CarbonImmutable;

// CarbonImmutable 每次操作返回新实例
$date = CarbonImmutable::now();
$nextWeek = $date->addWeek();  // 返回新实例，不修改原对象

// 原对象不变
echo $date->toDateString();     // 仍然是当前日期
echo $nextWeek->toDateString(); // 一周后的日期

// 适合在业务逻辑中使用，避免意外修改
```

### 实际应用示例

```php
<?php
declare(strict_types=1);

use Carbon\Carbon;

// 示例 1：计算用户年龄
function calculateAge(string $birthday): int
{
    return Carbon::parse($birthday)->age;
}

// 示例 2：判断是否在工作时间
function isBusinessHours(Carbon $time): bool
{
    return $time->isWeekday() 
        && $time->between(
            Carbon::parse('09:00'),
            Carbon::parse('18:00')
        );
}

// 示例 3：获取下个工作日
function getNextBusinessDay(Carbon $date): Carbon
{
    $next = $date->copy()->addDay();
    
    while ($next->isWeekend()) {
        $next->addDay();
    }
    
    return $next;
}

// 示例 4：格式化相对时间
function formatRelativeTime(Carbon $time): string
{
    if ($time->isToday()) {
        return $time->format('H:i');
    }
    
    if ($time->isYesterday()) {
        return '昨天 ' . $time->format('H:i');
    }
    
    if ($time->isCurrentWeek()) {
        return $time->format('l H:i'); // 星期几
    }
    
    return $time->format('Y-m-d H:i');
}
```

### Carbon 与数据库

```php
<?php
declare(strict_types=1);

use Carbon\Carbon;

// Laravel 模型自动使用 Carbon
// $user->created_at 是 Carbon 实例
$user = User::find(1);
echo $user->created_at->diffForHumans();
echo $user->created_at->format('Y-m-d');

// 手动转换
$timestamp = Carbon::parse($user->created_at)
    ->setTimezone('Asia/Shanghai')
    ->toDateTimeString();
```

## 时间与数据库

- 统一使用 UTC 存储，应用层转换至用户所在时区。
- 使用 `TIMESTAMP` 或 `DATETIME(6)` 存精度可控的时间。
- Laravel 等框架默认基于 `Carbon`，可直接 `$model->created_at->timezone('Asia/Shanghai')`.

## 时间差与性能

- `hrtime(true)`：返回纳秒级高精度时间戳。
- `memory_get_usage()` 结合 `hrtime()` 可衡量脚本性能。

## 练习

1. 编写 `Countdown` 类，输入未来时间，支持输出剩余的日/时/分/秒。
2. 实现一个 `BusinessHours` 服务，判断传入时间是否在工作日 9:00-18:00 内。
3. 使用 `DatePeriod` 生成指定区间内的每个星期一 09:00 的时间列表。
