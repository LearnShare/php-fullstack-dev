# 4.1.2 时区处理

## 概述

时区处理是日期时间编程中的关键环节。在不同的应用场景中，我们经常需要处理不同时区的时间，比如国际化的应用需要显示用户所在时区的时间，或者系统集成时需要处理跨时区的时间数据。理解时区概念和掌握 PHP 的时区处理机制对于开发可靠的日期时间应用至关重要。

时区是地球上的区域使用同一个时间定义。全球分为 24 个时区，每个时区相差 1 小时。由于地球的自转和公转，不同地区的人们使用不同的本地时间。PHP 的 DateTimeZone 类和相关的时区处理方法提供了完整的时区支持，包括时区转换、夏令时（DST）处理等。

正确处理时区可以避免许多常见的时间显示错误。在数据库存储时使用 UTC（协调世界时），在显示时再转换为用户所在时区，这是处理时区的最佳实践。

**主要内容**：
- 时区的基础概念（UTC、GMT、本地时区、DST）
- DateTimeZone 类的使用
- 时区设置方法（全局设置、对象级别设置）
- 时区转换（保持时间戳不变、保持显示时间不变）
- 默认时区配置
- 夏令时（DST）处理
- 时区偏移量获取
- 时区列表获取
- 常见时区问题处理
- 完整示例和最佳实践

## 特性

- **完整的时区支持**：支持所有 IANA 时区数据库中的时区
- **自动 DST 处理**：自动处理夏令时转换，无需手动计算
- **时区转换**：灵活的时间转换机制
- **时区验证**：可以验证时区标识符的有效性
- **时区列表**：可以获取所有可用时区的列表
- **偏移量计算**：可以获取任意时区的偏移量

## 语法/定义

### DateTimeZone 构造函数

**语法**：`new DateTimeZone(string $timezone)`

**参数**：
- `$timezone`：时区标识符或偏移量字符串。支持的格式：
  - **时区标识符**：`"Asia/Shanghai"`、`"UTC"`、`"America/New_York"`（推荐）
  - **偏移量**：`"+08:00"`、`"-05:00"`（不推荐，无法处理 DST）

**返回值**：返回 `DateTimeZone` 对象实例。如果时区无效，抛出 `Exception`。

### setTimezone() 方法

**语法**：`setTimezone(DateTimeZone $timezone): DateTime`

**参数**：
- `$timezone`：`DateTimeZone` 对象，指定要设置的时区。

**返回值**：
- `DateTime`：返回 `DateTime` 对象自身（支持链式调用），会修改原对象
- `DateTimeImmutable`：返回新的 `DateTimeImmutable` 对象，原对象不变

### getTimezone() 方法

**语法**：`getTimezone(): DateTimeZone`

**返回值**：返回当前 `DateTime` 对象使用的 `DateTimeZone` 对象。

### date_default_timezone_set() 函数

**语法**：`date_default_timezone_set(string $timezone): bool`

**参数**：
- `$timezone`：时区标识符，如 `"Asia/Shanghai"`、`"UTC"`、`"America/New_York"`。

**返回值**：成功返回 `true`，失败返回 `false`。

## 基本用法

### 示例 1：创建时区对象

```php
<?php
declare(strict_types=1);

// 使用时区标识符（推荐）
$tz1 = new DateTimeZone('Asia/Shanghai');
$tz2 = new DateTimeZone('UTC');
$tz3 = new DateTimeZone('America/New_York');

// 使用偏移量（不推荐，无法处理 DST）
$tz4 = new DateTimeZone('+08:00');  // 不推荐

// 获取时区名称
echo $tz1->getName() . "\n";  // Asia/Shanghai
```

**说明**：
- 推荐使用时区标识符（如 `Asia/Shanghai`），而不是偏移量
- 时区标识符可以正确处理夏令时（DST）
- 偏移量无法处理 DST，不推荐使用

### 示例 2：创建带时区的 DateTime 对象

```php
<?php
declare(strict_types=1);

// 创建指定时区的 DateTime 对象
$date1 = new DateTime('2025-01-15 10:30:00', new DateTimeZone('UTC'));
echo $date1->format('Y-m-d H:i:s T') . "\n";  // 2025-01-15 10:30:00 UTC

// 创建北京时区的 DateTime 对象
$date2 = new DateTime('2025-01-15 10:30:00', new DateTimeZone('Asia/Shanghai'));
echo $date2->format('Y-m-d H:i:s T') . "\n";  // 2025-01-15 10:30:00 CST

// 使用 DateTimeImmutable
$date3 = new DateTimeImmutable('2025-01-15 10:30:00', new DateTimeZone('America/New_York'));
echo $date3->format('Y-m-d H:i:s T') . "\n";  // 2025-01-15 10:30:00 EST
```

**输出**：

```
2025-01-15 10:30:00 UTC
2025-01-15 10:30:00 CST
2025-01-15 10:30:00 EST
```

**说明**：
- 创建 DateTime 对象时可以指定时区
- 时区会影响时间的显示
- 使用 `T` 格式化字符可以显示时区缩写

### 示例 3：时区转换（DateTime）

```php
<?php
declare(strict_types=1);

// 创建 UTC 时间的 DateTime
$utc = new DateTime('2025-01-15 10:30:00', new DateTimeZone('UTC'));
echo "UTC: " . $utc->format('Y-m-d H:i:s T') . "\n";

// 转换到北京时间（修改原对象）
$utc->setTimezone(new DateTimeZone('Asia/Shanghai'));
echo "Beijing: " . $utc->format('Y-m-d H:i:s T') . "\n";

// 转换到纽约时间
$utc->setTimezone(new DateTimeZone('America/New_York'));
echo "New York: " . $utc->format('Y-m-d H:i:s T') . "\n";
```

**输出**：

```
UTC: 2025-01-15 10:30:00 UTC
Beijing: 2025-01-15 18:30:00 CST
New York: 2025-01-15 05:30:00 EST
```

**说明**：
- `DateTime` 的 `setTimezone()` 会修改原对象
- 时区转换保持时间戳不变，只改变显示时间
- UTC 时间 10:30 转换为北京时间是 18:30（UTC+8）

### 示例 4：时区转换（DateTimeImmutable）

```php
<?php
declare(strict_types=1);

// 创建 UTC 时间的 DateTimeImmutable
$utc = new DateTimeImmutable('2025-01-15 10:30:00', new DateTimeZone('UTC'));
echo "UTC: " . $utc->format('Y-m-d H:i:s T') . "\n";

// 转换到北京时间（返回新对象）
$beijing = $utc->setTimezone(new DateTimeZone('Asia/Shanghai'));
echo "Beijing: " . $beijing->format('Y-m-d H:i:s T') . "\n";

// 原对象不变
echo "UTC (原对象): " . $utc->format('Y-m-d H:i:s T') . "\n";

// 可以继续转换
$newYork = $utc->setTimezone(new DateTimeZone('America/New_York'));
echo "New York: " . $newYork->format('Y-m-d H:i:s T') . "\n";
```

**输出**：

```
UTC: 2025-01-15 10:30:00 UTC
Beijing: 2025-01-15 18:30:00 CST
UTC (原对象): 2025-01-15 10:30:00 UTC
New York: 2025-01-15 05:30:00 EST
```

**说明**：
- `DateTimeImmutable` 的 `setTimezone()` 返回新对象
- 原对象保持不变
- 可以基于原对象进行多次转换

### 示例 5：设置默认时区

```php
<?php
declare(strict_types=1);

// 获取当前默认时区
echo "默认时区: " . date_default_timezone_get() . "\n";

// 设置默认时区
date_default_timezone_set('Asia/Shanghai');
echo "新默认时区: " . date_default_timezone_get() . "\n";

// 创建 DateTime 对象时，如果不指定时区，使用默认时区
$date = new DateTime('2025-01-15 10:30:00');
echo $date->getTimezone()->getName() . "\n";  // Asia/Shanghai

// 推荐：明确指定时区，不要依赖默认时区
$date2 = new DateTime('2025-01-15 10:30:00', new DateTimeZone('Asia/Shanghai'));
```

**输出**：

```
默认时区: UTC
新默认时区: Asia/Shanghai
Asia/Shanghai
```

**说明**：
- `date_default_timezone_set()` 设置全局默认时区
- 不指定时区时，DateTime 使用默认时区
- 推荐始终明确指定时区，不要依赖默认值

### 示例 6：获取时区偏移量

```php
<?php
declare(strict_types=1);

// 获取时区偏移量（秒）
$tz = new DateTimeZone('Asia/Shanghai');
$date = new DateTime('2025-01-15 10:00:00', $tz);

$offset = $tz->getOffset($date);
echo "偏移量: {$offset} 秒\n";
echo "偏移量: " . ($offset / 3600) . " 小时\n";  // 8 小时

// 夏令时时区的偏移量会变化
$tzNY = new DateTimeZone('America/New_York');

// 冬季（标准时间 EST）
$winter = new DateTime('2025-01-15 10:00:00', $tzNY);
$offsetWinter = $tzNY->getOffset($winter);
echo "纽约冬季偏移: " . ($offsetWinter / 3600) . " 小时\n";  // -5 小时

// 夏季（夏令时 EDT）
$summer = new DateTime('2025-07-15 10:00:00', $tzNY);
$offsetSummer = $tzNY->getOffset($summer);
echo "纽约夏季偏移: " . ($offsetSummer / 3600) . " 小时\n";  // -4 小时
```

**输出**：

```
偏移量: 28800 秒
偏移量: 8 小时
纽约冬季偏移: -5 小时
纽约夏季偏移: -4 小时
```

**说明**：
- `getOffset()` 返回时区偏移量（秒数）
- 正数表示 UTC 以东，负数表示 UTC 以西
- 有夏令时的时区，偏移量会随日期变化

### 示例 7：夏令时（DST）处理

```php
<?php
declare(strict_types=1);

// 纽约时区（有夏令时）
$tz = new DateTimeZone('America/New_York');

// 冬季（标准时间 EST，UTC-5）
$winter = new DateTime('2025-01-15 10:00:00', $tz);
echo "冬季: " . $winter->format('Y-m-d H:i:s T') . "\n";  // EST

// 夏季（夏令时 EDT，UTC-4）
$summer = new DateTime('2025-07-15 10:00:00', $tz);
echo "夏季: " . $summer->format('Y-m-d H:i:s T') . "\n";  // EDT

// 中国时区（无夏令时）
$tzCN = new DateTimeZone('Asia/Shanghai');
$dateCN = new DateTime('2025-01-15 10:00:00', $tzCN);
echo "中国: " . $dateCN->format('Y-m-d H:i:s T') . "\n";  // CST（全年不变）

$dateCN2 = new DateTime('2025-07-15 10:00:00', $tzCN);
echo "中国: " . $dateCN2->format('Y-m-d H:i:s T') . "\n";  // CST（全年不变）
```

**输出**：

```
冬季: 2025-01-15 10:00:00 EST
夏季: 2025-07-15 10:00:00 EDT
中国: 2025-01-15 10:00:00 CST
中国: 2025-07-15 10:00:00 CST
```

**说明**：
- DateTimeZone 会自动处理夏令时转换
- 无需手动计算 DST，PHP 会自动处理
- 有夏令时的时区，时区缩写会变化（EST/EDT）

### 示例 8：获取时区列表

```php
<?php
declare(strict_types=1);

// 获取所有时区
$allTimezones = DateTimeZone::listIdentifiers();
echo "总时区数: " . count($allTimezones) . "\n";

// 获取亚洲时区
$asiaTimezones = DateTimeZone::listIdentifiers(DateTimeZone::ASIA);
echo "\n亚洲时区:\n";
foreach ($asiaTimezones as $tz) {
    if (str_starts_with($tz, 'Asia/')) {
        echo "  {$tz}\n";
    }
}

// 获取美国时区
$usTimezones = DateTimeZone::listIdentifiers(DateTimeZone::AMERICA);
echo "\n美国时区（部分）:\n";
foreach (array_slice($usTimezones, 0, 10) as $tz) {
    echo "  {$tz}\n";
}

// 按国家代码过滤
$cnTimezones = DateTimeZone::listIdentifiers(DateTimeZone::ALL, 'CN');
echo "\n中国时区:\n";
foreach ($cnTimezones as $tz) {
    echo "  {$tz}\n";
}
```

**输出**：

```
总时区数: 400+

亚洲时区:
  Asia/Shanghai
  Asia/Tokyo
  Asia/Seoul
  ...

中国时区:
  Asia/Shanghai
  Asia/Urumqi
```

**说明**：
- `listIdentifiers()` 可以获取所有可用时区
- 可以按地区或国家代码过滤
- 常用的地区常量包括：`ASIA`、`AMERICA`、`EUROPE` 等

### 示例 9：时区转换工具类

```php
<?php
declare(strict_types=1);

class TimezoneHelper
{
    /**
     * 将 UTC 时间转换为用户时区
     */
    public static function convertToUserTimezone(
        DateTimeImmutable $utcTime,
        string $userTimezone
    ): DateTimeImmutable {
        return $utcTime->setTimezone(new DateTimeZone($userTimezone));
    }
    
    /**
     * 将本地时间转换为 UTC
     */
    public static function convertToUtc(
        DateTimeImmutable $localTime,
        string $localTimezone
    ): DateTimeImmutable {
        // 先将本地时间设置为指定时区
        $local = $localTime->setTimezone(new DateTimeZone($localTimezone));
        // 然后转换为 UTC
        return $local->setTimezone(new DateTimeZone('UTC'));
    }
    
    /**
     * 验证时区标识符
     */
    public static function isValidTimezone(string $timezone): bool
    {
        return in_array($timezone, DateTimeZone::listIdentifiers(), true);
    }
    
    /**
     * 获取时区偏移量（小时）
     */
    public static function getOffsetHours(
        DateTimeImmutable $date,
        DateTimeZone $timezone
    ): float {
        $offset = $timezone->getOffset($date);
        return $offset / 3600;
    }
}

// 使用示例
$utc = new DateTimeImmutable('2025-01-15 10:30:00', new DateTimeZone('UTC'));

// 转换为北京时间
$beijing = TimezoneHelper::convertToUserTimezone($utc, 'Asia/Shanghai');
echo "UTC: " . $utc->format('Y-m-d H:i:s T') . "\n";
echo "Beijing: " . $beijing->format('Y-m-d H:i:s T') . "\n";

// 验证时区
if (TimezoneHelper::isValidTimezone('Asia/Shanghai')) {
    echo "时区有效\n";
}

// 获取偏移量
$tz = new DateTimeZone('Asia/Shanghai');
$offset = TimezoneHelper::getOffsetHours($utc, $tz);
echo "偏移量: {$offset} 小时\n";
```

**输出**：

```
UTC: 2025-01-15 10:30:00 UTC
Beijing: 2025-01-15 18:30:00 CST
时区有效
偏移量: 8 小时
```

**说明**：
- 封装时区转换逻辑，提高代码复用性
- 提供时区验证功能
- 简化时区相关的操作

## 使用场景

### 场景 1：多时区应用

在国际化的应用中，需要根据用户所在时区显示时间。

**示例**：

```php
<?php
declare(strict_types=1);

class UserTimezoneHandler
{
    public function displayTime(DateTimeImmutable $utcTime, string $userTimezone): string
    {
        $userTime = $utcTime->setTimezone(new DateTimeZone($userTimezone));
        return $userTime->format('Y-m-d H:i:s T');
    }
}

// 使用
$utcTime = new DateTimeImmutable('now', new DateTimeZone('UTC'));
$handler = new UserTimezoneHandler();

echo $handler->displayTime($utcTime, 'Asia/Shanghai') . "\n";
echo $handler->displayTime($utcTime, 'America/New_York') . "\n";
echo $handler->displayTime($utcTime, 'Europe/London') . "\n";
```

### 场景 2：数据库存储 UTC 时间

在数据库中存储 UTC 时间，显示时转换为用户时区。

**示例**：

```php
<?php
declare(strict_types=1);

// 存储时使用 UTC
$utcTime = new DateTimeImmutable('now', new DateTimeZone('UTC'));
$dbValue = $utcTime->format('Y-m-d H:i:s');
// 保存到数据库

// 从数据库读取并转换为用户时区
$dbTime = DateTimeImmutable::createFromFormat('Y-m-d H:i:s', $dbValue, new DateTimeZone('UTC'));
$userTime = $dbTime->setTimezone(new DateTimeZone('Asia/Shanghai'));
echo $userTime->format('Y-m-d H:i:s T');
```

## 注意事项

### 始终使用 UTC 存储

在数据库和 API 中应使用 UTC 时间，显示时再转换为用户时区。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：存储 UTC
$utc = new DateTimeImmutable('now', new DateTimeZone('UTC'));
$dbValue = $utc->format('Y-m-d H:i:s');

// 不推荐：存储本地时间
// $local = new DateTimeImmutable('now', new DateTimeZone('Asia/Shanghai'));
// $dbValue = $local->format('Y-m-d H:i:s');  // 不推荐
```

### 明确指定时区

创建 DateTime 时明确指定时区，不要依赖系统默认值。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：明确指定时区
$date = new DateTime('now', new DateTimeZone('Asia/Shanghai'));

// 不推荐：依赖默认时区
// $date = new DateTime('now');
```

### DST 自动处理

DateTimeZone 会自动处理夏令时，无需手动计算。

**示例**：

```php
<?php
declare(strict_types=1);

// PHP 自动处理 DST
$tz = new DateTimeZone('America/New_York');
$winter = new DateTime('2025-01-15', $tz);  // EST
$summer = new DateTime('2025-07-15', $tz);  // EDT
// 无需手动判断是否是夏令时
```

### 时区标识符 vs 偏移量

使用时区标识符而不是偏移量，以正确处理 DST。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：使用时区标识符
$tz = new DateTimeZone('America/New_York');  // 正确处理 DST

// 不推荐：使用偏移量
// $tz = new DateTimeZone('-05:00');  // 无法处理 DST
```

## 常见问题

### 问题 1：如何设置应用的默认时区？

**回答**：使用 `date_default_timezone_set()` 函数，或在 php.ini 中配置 `date.timezone`。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法 1：在代码中设置
date_default_timezone_set('Asia/Shanghai');

// 方法 2：在 php.ini 中设置
// date.timezone = "Asia/Shanghai"
```

### 问题 2：时区转换后时间戳会改变吗？

**回答**：不会。时区转换只改变显示时间，时间戳（Unix timestamp）保持不变。

**示例**：

```php
<?php
declare(strict_types=1);

$utc = new DateTimeImmutable('2025-01-15 10:30:00', new DateTimeZone('UTC'));
$timestamp1 = $utc->getTimestamp();

$beijing = $utc->setTimezone(new DateTimeZone('Asia/Shanghai'));
$timestamp2 = $beijing->getTimestamp();

var_dump($timestamp1 === $timestamp2);  // true（时间戳相同）
```

### 问题 3：如何处理夏令时？

**回答**：DateTimeZone 会自动处理夏令时，无需手动处理。

**示例**：见"示例 7：夏令时（DST）处理"

### 问题 4：如何验证时区标识符的有效性？

**回答**：使用 `DateTimeZone::listIdentifiers()` 检查时区是否在列表中。

**示例**：

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

## 最佳实践

### 1. 数据库存储 UTC 时间

数据库存储时间使用 UTC，显示时转换为用户时区。

**示例**：

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

### 2. 明确指定时区

创建 DateTime 时明确指定时区，不要依赖默认值。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：明确指定时区
$date = new DateTime('now', new DateTimeZone('Asia/Shanghai'));

// 不推荐：依赖默认时区
// $date = new DateTime('now');
```

### 3. 使用标准的时区标识符

使用完整的时区标识符（如 `Asia/Shanghai`），避免使用偏移量。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：使用时区标识符
$tz = new DateTimeZone('Asia/Shanghai');

// 不推荐：使用偏移量
// $tz = new DateTimeZone('+08:00');
```

### 4. 在配置文件中统一管理

在配置文件中统一管理时区设置，便于维护。

**示例**：

```php
<?php
// config/timezone.php
return [
    'default' => 'UTC',
    'display' => 'Asia/Shanghai',
];

// 使用
$config = require 'config/timezone.php';
date_default_timezone_set($config['default']);
```

## 对比分析

### 时区标识符 vs 偏移量

| 特性         | 时区标识符（Asia/Shanghai）          | 偏移量（+08:00）                   |
|:-------------|:-----------------------------------|:--------------------------------|
| **DST 支持**  | ✅ 自动处理                         | ❌ 无法处理                       |
| **准确性**    | ✅ 准确                             | ⚠️ 固定值                         |
| **可读性**    | ✅ 语义清晰                         | ⚠️ 不够直观                       |
| **推荐使用**  | ✅ 推荐                             | ❌ 不推荐                         |

### DateTime vs DateTimeImmutable（时区转换）

| 特性         | DateTime                           | DateTimeImmutable                |
|:-------------|:--------------------------------|:--------------------------------|
| **setTimezone** | 修改原对象                      | 返回新对象                        |
| **安全性**    | ⚠️ 可能意外修改                   | ✅ 不会意外修改                   |
| **推荐使用**  | 简单脚本                         | 业务逻辑、函数参数                |

## 练习任务

1. **时区转换工具类**：创建一个 `TimezoneConverter` 类，提供 UTC 与任意时区之间的转换方法。

2. **时区偏移量获取**：编写一个函数，获取指定时区在特定日期的偏移量（考虑 DST）。

3. **多时区时钟**：实现一个多时区时钟显示功能，同时显示多个时区的当前时间。

4. **时区验证工具**：创建一个函数，验证时区标识符的有效性，并提供友好的错误提示。

5. **时区列表工具**：编写一个函数，列出指定地区的所有时区，并提供搜索功能。

## 相关章节

- **[4.1.1 DateTime 基础](section-01-datetime-basics.md)**：了解 DateTime 类的基础用法
- **[4.1.3 格式化与解析](section-03-formatting-parsing.md)**：学习日期时间的格式化和解析
- **[4.1.4 时间计算与比较](section-04-calculations-comparisons.md)**：学习时间的计算和比较操作
