# 2.15.2 时区处理

## 概述

时区处理是日期时间编程中的关键环节。本节详细介绍时区概念、`DateTimeZone` 的使用、时区转换、DST（夏令时）处理，以及常见时区列表。

## 时区基础概念

### 什么是时区

时区是地球上的区域使用同一个时间定义。全球分为 24 个时区，每个时区相差 1 小时。

### UTC 与本地时间

- **UTC（协调世界时）**：标准时间参考，不随季节变化
- **本地时间**：根据时区偏移计算的时间
- **DST（夏令时）**：某些地区在夏季将时钟拨快 1 小时

## DateTimeZone 类

### 创建时区对象

```php
<?php
declare(strict_types=1);

// 使用时区标识符
$tz = new DateTimeZone('Asia/Shanghai');
$tz = new DateTimeZone('UTC');
$tz = new DateTimeZone('America/New_York');

// 使用偏移量（不推荐，无法处理 DST）
$tz = new DateTimeZone('+08:00');
```

### 常用时区标识符

```php
<?php
declare(strict_types=1);

// 中国
$tz = new DateTimeZone('Asia/Shanghai');  // UTC+8

// 美国
$tz = new DateTimeZone('America/New_York');  // UTC-5 (EST) / UTC-4 (EDT)
$tz = new DateTimeZone('America/Los_Angeles');  // UTC-8 (PST) / UTC-7 (PDT)

// 欧洲
$tz = new DateTimeZone('Europe/London');  // UTC+0 (GMT) / UTC+1 (BST)
$tz = new DateTimeZone('Europe/Paris');  // UTC+1 (CET) / UTC+2 (CEST)

// 日本
$tz = new DateTimeZone('Asia/Tokyo');  // UTC+9
```

## 设置默认时区

### 全局设置

```php
<?php
// 在 php.ini 中设置
// date.timezone = "Asia/Shanghai"

// 在代码中设置
date_default_timezone_set('Asia/Shanghai');
echo date_default_timezone_get();  // Asia/Shanghai
```

### 在 DateTime 中使用时区

```php
<?php
declare(strict_types=1);

// 创建带时区的 DateTime
$date = new DateTime('now', new DateTimeZone('UTC'));
echo $date->format('Y-m-d H:i:s T') . "\n";  // 2024-01-15 10:30:00 UTC

// 转换时区
$date->setTimezone(new DateTimeZone('Asia/Shanghai'));
echo $date->format('Y-m-d H:i:s T') . "\n";  // 2024-01-15 18:30:00 CST
```

## 时区转换

### 基本转换

```php
<?php
declare(strict_types=1);

// 从 UTC 转换到北京时间
$utc = new DateTime('2024-01-15 10:30:00', new DateTimeZone('UTC'));
$beijing = clone $utc;
$beijing->setTimezone(new DateTimeZone('Asia/Shanghai'));
echo "UTC: " . $utc->format('Y-m-d H:i:s T') . "\n";
echo "Beijing: " . $beijing->format('Y-m-d H:i:s T') . "\n";
```

### 使用 DateTimeImmutable

```php
<?php
declare(strict_types=1);

$utc = new DateTimeImmutable('2024-01-15 10:30:00', new DateTimeZone('UTC'));
$beijing = $utc->setTimezone(new DateTimeZone('Asia/Shanghai'));
// 原对象不变
echo "UTC: " . $utc->format('Y-m-d H:i:s T') . "\n";
echo "Beijing: " . $beijing->format('Y-m-d H:i:s T') . "\n";
```

## DST（夏令时）处理

### 自动处理 DST

`DateTimeZone` 会自动处理夏令时转换：

```php
<?php
declare(strict_types=1);

// 纽约时间（有夏令时）
$tz = new DateTimeZone('America/New_York');

// 冬季（标准时间 EST）
$winter = new DateTime('2024-01-15 10:00:00', $tz);
echo $winter->format('Y-m-d H:i:s T') . "\n";  // 2024-01-15 10:00:00 EST

// 夏季（夏令时 EDT）
$summer = new DateTime('2024-07-15 10:00:00', $tz);
echo $summer->format('Y-m-d H:i:s T') . "\n";  // 2024-07-15 10:00:00 EDT
```

### 获取时区信息

```php
<?php
declare(strict_types=1);

$tz = new DateTimeZone('America/New_York');
$date = new DateTime('2024-07-15 10:00:00', $tz);

// 获取时区偏移（秒）
$offset = $tz->getOffset($date);
echo "Offset: " . ($offset / 3600) . " hours\n";  // -4 hours (EDT)

// 获取时区名称
echo "Timezone: " . $tz->getName() . "\n";  // America/New_York

// 获取时区缩写
echo "Abbreviation: " . $date->format('T') . "\n";  // EDT
```

## 获取时区列表

### 所有时区

```php
<?php
declare(strict_types=1);

$timezones = DateTimeZone::listIdentifiers();
foreach ($timezones as $timezone) {
    echo $timezone . "\n";
}
```

### 按地区过滤

```php
<?php
declare(strict_types=1);

// 亚洲时区
$asiaTimezones = DateTimeZone::listIdentifiers(DateTimeZone::ASIA);
foreach ($asiaTimezones as $tz) {
    echo $tz . "\n";
}

// 美国时区
$usTimezones = DateTimeZone::listIdentifiers(DateTimeZone::AMERICA);
foreach ($usTimezones as $tz) {
    echo $tz . "\n";
}
```

## 最佳实践

### 1. 存储 UTC 时间

```php
<?php
declare(strict_types=1);

// 存储时使用 UTC
$utc = new DateTimeImmutable('now', new DateTimeZone('UTC'));
$dbValue = $utc->format('Y-m-d H:i:s');

// 显示时转换为用户时区
$userTz = new DateTimeZone('Asia/Shanghai');
$display = $utc->setTimezone($userTz);
echo $display->format('Y-m-d H:i:s T');
```

### 2. 用户时区处理

```php
<?php
declare(strict_types=1);

class TimezoneHelper
{
    public static function convertToUserTimezone(
        DateTimeImmutable $utcTime,
        string $userTimezone
    ): DateTimeImmutable {
        return $utcTime->setTimezone(new DateTimeZone($userTimezone));
    }

    public static function convertToUtc(
        DateTimeImmutable $localTime,
        string $localTimezone
    ): DateTimeImmutable {
        $local = $localTime->setTimezone(new DateTimeZone($localTimezone));
        return $local->setTimezone(new DateTimeZone('UTC'));
    }
}
```

### 3. 时区验证

```php
<?php
declare(strict_types=1);

function isValidTimezone(string $timezone): bool
{
    return in_array($timezone, DateTimeZone::listIdentifiers(), true);
}

if (!isValidTimezone('Asia/Shanghai')) {
    throw new InvalidArgumentException('Invalid timezone');
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class TimezoneDemo
{
    public static function demonstrate(): void
    {
        echo "=== 时区转换 ===\n";
        $utc = new DateTimeImmutable('2024-01-15 10:00:00', new DateTimeZone('UTC'));
        $beijing = $utc->setTimezone(new DateTimeZone('Asia/Shanghai'));
        $newYork = $utc->setTimezone(new DateTimeZone('America/New_York'));
        
        echo "UTC: " . $utc->format('Y-m-d H:i:s T') . "\n";
        echo "Beijing: " . $beijing->format('Y-m-d H:i:s T') . "\n";
        echo "New York: " . $newYork->format('Y-m-d H:i:s T') . "\n";
        
        echo "\n=== DST 处理 ===\n";
        $tz = new DateTimeZone('America/New_York');
        $winter = new DateTime('2024-01-15 10:00:00', $tz);
        $summer = new DateTime('2024-07-15 10:00:00', $tz);
        echo "Winter: " . $winter->format('Y-m-d H:i:s T') . "\n";
        echo "Summer: " . $summer->format('Y-m-d H:i:s T') . "\n";
    }
}

TimezoneDemo::demonstrate();
```

## 注意事项

1. **始终使用 UTC 存储**：数据库和 API 应使用 UTC 时间，显示时再转换。
2. **明确时区**：创建 `DateTime` 时明确指定时区，不要依赖系统默认值。
3. **DST 自动处理**：`DateTimeZone` 会自动处理夏令时，无需手动计算。
4. **时区标识符**：使用完整的时区标识符（如 `Asia/Shanghai`），避免使用偏移量。

## 练习

1. 创建一个时区转换工具类，支持 UTC 与任意时区的转换。
2. 编写一个函数，获取指定时区在特定日期的偏移量。
3. 实现一个多时区时钟显示功能。
4. 创建一个函数，验证时区标识符的有效性。
5. 编写一个函数，列出所有支持 DST 的时区。
