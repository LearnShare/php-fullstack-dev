# 4.1.3 格式化与解析

## 概述

日期时间的格式化与解析是日常开发中的高频操作。格式化是将 DateTime 对象转换为字符串的过程，用于显示或存储；解析是将字符串转换为 DateTime 对象的过程，用于处理用户输入或从外部数据源读取日期时间。掌握格式化与解析对于正确处理各种日期时间格式至关重要。

理解格式化与解析的机制对于开发可靠的应用非常重要。不同的应用场景可能需要不同的日期时间格式，比如数据库存储使用标准格式，用户界面显示使用可读格式，API 接口可能需要 ISO 8601 格式。同时，处理用户输入的日期时间时，需要能够解析各种可能的格式。

PHP 的 DateTime 类提供了强大的格式化功能，支持 40+ 种格式化字符，可以满足各种格式化需求。`format()` 方法用于格式化，`createFromFormat()` 方法用于解析，两者配合使用可以处理各种日期时间格式需求。

**主要内容**：
- `format()` 方法的使用和格式化字符串详解
- `createFromFormat()` 方法的使用和解析技巧
- 格式化字符完整列表和说明
- 常用格式化模式
- 转义字符的使用
- 解析错误处理
- 多语言格式化支持
- 相对时间格式化
- 完整示例和最佳实践

## 特性

- **丰富的格式化字符**：支持 40+ 种格式化字符，满足各种格式化需求
- **灵活的解析功能**：支持从任意格式字符串解析日期时间
- **错误处理**：提供完善的错误处理机制
- **多语言支持**：支持使用 `IntlDateFormatter` 进行多语言格式化
- **相对时间格式化**：支持将日期时间格式化为相对时间（如"2小时前"）

## 语法/定义

### format() 方法

**语法**：`format(string $format): string`

**参数**：
- `$format`：格式化字符串，使用特定字符表示日期时间各部分。格式化字符区分大小写，特殊字符需要使用反斜杠转义。

**返回值**：返回格式化后的日期时间字符串。

### createFromFormat() 方法

**语法**：`static createFromFormat(string $format, string $datetime, ?DateTimeZone $timezone = null): DateTime|false`

**参数**：
- `$format`：日期时间格式字符串，使用格式化字符定义输入格式。格式字符串必须与 `$datetime` 参数完全匹配。
- `$datetime`：要解析的日期时间字符串，必须与 `$format` 参数定义的格式完全匹配。
- `$timezone`：可选，`DateTimeZone` 对象，指定时区。如果为 `null`，使用默认时区。

**返回值**：成功返回 `DateTime` 对象，失败返回 `false`。可以使用 `DateTime::getLastErrors()` 获取错误信息。

## 基本用法

### 示例 1：基本格式化

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
2025-01-15T14:30:45+08:00
Wed, 15 Jan 2025 14:30:45 +0800
```

**说明**：
- `format()` 方法使用格式化字符串输出日期时间
- 常用的格式字符包括：`Y`（4位年份）、`m`（月份）、`d`（日期）、`H`（小时）、`i`（分钟）、`s`（秒）

### 示例 2：年份格式化

```php
<?php
declare(strict_types=1);

$date = new DateTime('2025-01-15');

// 4 位数字年份
echo $date->format('Y') . "\n";  // 2025

// 2 位数字年份
echo $date->format('y') . "\n";  // 25

// ISO-8601 周数年份
echo $date->format('o') . "\n";  // 2025

// 是否为闰年
echo $date->format('L') . "\n";  // 0（2025 年不是闰年）
```

**输出**：

```
2025
25
2025
0
```

**说明**：
- `Y` 和 `y` 的区别在于位数不同
- `L` 返回 1 表示闰年，0 表示平年

### 示例 3：月份格式化

```php
<?php
declare(strict_types=1);

$date = new DateTime('2025-01-15');

// 带前导零的月份
echo $date->format('m') . "\n";  // 01

// 不带前导零的月份
echo $date->format('n') . "\n";  // 1

// 3 字母月份缩写
echo $date->format('M') . "\n";  // Jan

// 完整月份名称
echo $date->format('F') . "\n";  // January

// 月份天数
echo $date->format('t') . "\n";  // 31
```

**输出**：

```
01
1
Jan
January
31
```

**说明**：
- `m` 和 `n` 的区别在于是否带前导零
- `M` 和 `F` 返回英文月份名称（缩写和完整）

### 示例 4：日期格式化

```php
<?php
declare(strict_types=1);

$date = new DateTime('2025-01-15');

// 带前导零的日期
echo $date->format('d') . "\n";  // 15

// 不带前导零的日期
echo $date->format('j') . "\n";  // 15

// 日期后缀（英文）
echo $date->format('jS') . "\n";  // 15th

// 一年中的第几天
echo $date->format('z') . "\n";  // 14（从 0 开始）

// 星期几（数字，0=周日）
echo $date->format('w') . "\n";  // 3（周三）

// ISO-8601 星期几（1=周一）
echo $date->format('N') . "\n";  // 3

// 3 字母星期缩写
echo $date->format('D') . "\n";  // Wed

// 完整星期名称
echo $date->format('l') . "\n";  // Wednesday
```

**输出**：

```
15
15
15th
14
3
3
Wed
Wednesday
```

**说明**：
- `d` 和 `j` 的区别在于是否带前导零
- `w` 和 `N` 的区别在于周日的值（0 或 7）

### 示例 5：时间格式化

```php
<?php
declare(strict_types=1);

$date = new DateTime('2025-01-15 14:30:45');

// 24 小时制（带前导零）
echo $date->format('H') . "\n";  // 14

// 24 小时制（不带前导零）
echo $date->format('G') . "\n";  // 14

// 12 小时制（带前导零）
echo $date->format('h') . "\n";  // 02

// 12 小时制（不带前导零）
echo $date->format('g') . "\n";  // 2

// 分钟
echo $date->format('i') . "\n";  // 30

// 秒
echo $date->format('s') . "\n";  // 45

// 微秒
echo $date->format('u') . "\n";  // 000000

// am/pm（小写）
echo $date->format('a') . "\n";  // pm

// AM/PM（大写）
echo $date->format('A') . "\n";  // PM

// 12 小时制完整格式
echo $date->format('g:i A') . "\n";  // 2:30 PM
```

**输出**：

```
14
14
02
2
30
45
000000
pm
PM
2:30 PM
```

**说明**：
- `H` 和 `G` 是 24 小时制，区别在于是否带前导零
- `h` 和 `g` 是 12 小时制，区别在于是否带前导零

### 示例 6：时区格式化

```php
<?php
declare(strict_types=1);

$date = new DateTime('2025-01-15 14:30:45', new DateTimeZone('Asia/Shanghai'));

// 时区标识符
echo $date->format('e') . "\n";  // Asia/Shanghai

// 时区缩写
echo $date->format('T') . "\n";  // CST

// 时区偏移（小时，无冒号）
echo $date->format('O') . "\n";  // +0800

// 时区偏移（小时，带冒号）
echo $date->format('P') . "\n";  // +08:00

// 时区偏移（秒）
echo $date->format('Z') . "\n";  // 28800

// 是否为夏令时
echo $date->format('I') . "\n";  // 0（中国不使用夏令时）
```

**输出**：

```
Asia/Shanghai
CST
+0800
+08:00
28800
0
```

**说明**：
- 时区格式化字符用于显示时区相关信息
- `I` 返回 1 表示夏令时，0 表示标准时间

### 示例 7：常用格式化模式

```php
<?php
declare(strict_types=1);

$date = new DateTime('2025-01-15 14:30:45');

// 数据库格式
echo $date->format('Y-m-d H:i:s') . "\n";  // 2025-01-15 14:30:45

// ISO 8601 格式
echo $date->format('c') . "\n";  // 2025-01-15T14:30:45+08:00

// RFC 2822 格式
echo $date->format('r') . "\n";  // Wed, 15 Jan 2025 14:30:45 +0800

// 可读格式
echo $date->format('l, F j, Y') . "\n";  // Wednesday, January 15, 2025

// 简短格式
echo $date->format('M j, Y') . "\n";  // Jan 15, 2025

// 时间格式
echo $date->format('g:i A') . "\n";  // 2:30 PM

// 日期时间格式
echo $date->format('Y年m月d日 H:i:s') . "\n";  // 2025年01月15日 14:30:45
```

**输出**：

```
2025-01-15 14:30:45
2025-01-15T14:30:45+08:00
Wed, 15 Jan 2025 14:30:45 +0800
Wednesday, January 15, 2025
Jan 15, 2025
2:30 PM
2025年01月15日 14:30:45
```

**说明**：
- 不同的格式化模式适用于不同的场景
- 可以根据需求选择合适的格式化模式

### 示例 8：转义字符

```php
<?php
declare(strict_types=1);

$date = new DateTime('2025-01-15 14:30:45');

// 转义格式化字符
echo $date->format('\Y\e\a\r: Y') . "\n";  // Year: 2025

// 转义普通字符
echo $date->format('Y-m-d \a\t H:i:s') . "\n";  // 2025-01-15 at 14:30:45

// 混合使用
echo $date->format('Date: Y-m-d, Time: H:i:s') . "\n";  // Date: 2025-01-15, Time: 14:30:45
```

**输出**：

```
Year: 2025
2025-01-15 at 14:30:45
Date: 2025-01-15, Time: 14:30:45
```

**说明**：
- 使用反斜杠 `\` 转义格式化字符
- 转义后的字符会原样输出

### 示例 9：createFromFormat() 基本用法

```php
<?php
declare(strict_types=1);

// 从标准格式解析
$date1 = DateTime::createFromFormat('Y-m-d H:i:s', '2025-01-15 14:30:45');
if ($date1 !== false) {
    echo $date1->format('Y-m-d H:i:s') . "\n";  // 2025-01-15 14:30:45
}

// 从自定义格式解析
$date2 = DateTime::createFromFormat('d/m/Y', '15/01/2025');
if ($date2 !== false) {
    echo $date2->format('Y-m-d') . "\n";  // 2025-01-15
}

// 从美国日期格式解析
$date3 = DateTime::createFromFormat('m/d/Y', '01/15/2025');
if ($date3 !== false) {
    echo $date3->format('Y-m-d') . "\n";  // 2025-01-15
}
```

**输出**：

```
2025-01-15 14:30:45
2025-01-15
2025-01-15
```

**说明**：
- `createFromFormat()` 可以从特定格式的字符串解析日期时间
- 格式字符串必须与输入字符串完全匹配
- 解析失败时返回 `false`

### 示例 10：解析错误处理

```php
<?php
declare(strict_types=1);

function parseDateSafely(string $dateString, string $format): ?DateTime
{
    $date = DateTime::createFromFormat($format, $dateString);
    if ($date === false) {
        $errors = DateTime::getLastErrors();
        if ($errors !== false) {
            echo "解析错误:\n";
            foreach ($errors['errors'] as $error) {
                echo "  - {$error}\n";
            }
            if (!empty($errors['warnings'])) {
                echo "警告:\n";
                foreach ($errors['warnings'] as $warning) {
                    echo "  - {$warning}\n";
                }
            }
        }
        return null;
    }
    return $date;
}

// 成功解析
$date1 = parseDateSafely('2025-01-15', 'Y-m-d');
if ($date1 !== null) {
    echo "解析成功: " . $date1->format('Y-m-d') . "\n";
}

// 解析失败
$date2 = parseDateSafely('invalid-date', 'Y-m-d');
if ($date2 === null) {
    echo "解析失败\n";
}
```

**输出**：

```
解析成功: 2025-01-15
解析错误:
  - The parsed date was invalid
解析失败
```

**说明**：
- 使用 `DateTime::getLastErrors()` 获取详细的错误信息
- 错误信息包括 `errors` 和 `warnings` 两个数组

### 示例 11：使用 ! 前缀重置未指定部分

```php
<?php
declare(strict_types=1);

// 不使用 ! 前缀：未指定的部分使用当前时间
$date1 = DateTime::createFromFormat('Y-m-d', '2025-01-15');
echo $date1->format('Y-m-d H:i:s') . "\n";  // 2025-01-15 当前时间

// 使用 ! 前缀：未指定的部分重置为 00:00:00
$date2 = DateTime::createFromFormat('!Y-m-d', '2025-01-15');
echo $date2->format('Y-m-d H:i:s') . "\n";  // 2025-01-15 00:00:00
```

**输出**：

```
2025-01-15 14:30:45
2025-01-15 00:00:00
```

**说明**：
- `!` 前缀可以重置未指定的部分为 Unix 纪元（1970-01-01 00:00:00）
- 这对于只解析日期部分很有用

### 示例 12：解析多种格式

```php
<?php
declare(strict_types=1);

function parseDateMultiple(string $dateString): ?DateTime
{
    $formats = [
        'Y-m-d H:i:s',    // 2025-01-15 14:30:45
        'Y-m-d',          // 2025-01-15
        'm/d/Y',          // 01/15/2025
        'd-m-Y',          // 15-01-2025
        'Y/m/d',          // 2025/01/15
        'd.m.Y',          // 15.01.2025
    ];
    
    foreach ($formats as $format) {
        $date = DateTime::createFromFormat($format, $dateString);
        if ($date !== false) {
            // 验证格式是否完全匹配
            if ($date->format($format) === $dateString) {
                return $date;
            }
        }
    }
    
    return null;
}

// 测试不同的格式
$dates = [
    '2025-01-15 14:30:45',
    '2025-01-15',
    '01/15/2025',
    '15-01-2025',
];

foreach ($dates as $dateStr) {
    $date = parseDateMultiple($dateStr);
    if ($date !== null) {
        echo "解析 {$dateStr}: " . $date->format('Y-m-d H:i:s') . "\n";
    } else {
        echo "无法解析: {$dateStr}\n";
    }
}
```

**输出**：

```
解析 2025-01-15 14:30:45: 2025-01-15 14:30:45
解析 2025-01-15: 2025-01-15 00:00:00
解析 01/15/2025: 2025-01-15 00:00:00
解析 15-01-2025: 2025-01-15 00:00:00
```

**说明**：
- 可以尝试多种格式，直到解析成功
- 验证格式是否完全匹配很重要，避免误解析

### 示例 13：相对时间格式化

```php
<?php
declare(strict_types=1);

function formatRelativeTime(DateTimeImmutable $date): string
{
    $now = new DateTimeImmutable();
    $diff = $now->diff($date);
    
    if ($diff->invert === 1) {
        // 过去时间
        if ($diff->y > 0) {
            return "{$diff->y}年前";
        } elseif ($diff->m > 0) {
            return "{$diff->m}个月前";
        } elseif ($diff->d > 0) {
            return "{$diff->d}天前";
        } elseif ($diff->h > 0) {
            return "{$diff->h}小时前";
        } elseif ($diff->i > 0) {
            return "{$diff->i}分钟前";
        } else {
            return "刚刚";
        }
    } else {
        // 未来时间
        if ($diff->y > 0) {
            return "{$diff->y}年后";
        } elseif ($diff->m > 0) {
            return "{$diff->m}个月后";
        } elseif ($diff->d > 0) {
            return "{$diff->d}天后";
        } elseif ($diff->h > 0) {
            return "{$diff->h}小时后";
        } elseif ($diff->i > 0) {
            return "{$diff->i}分钟后";
        } else {
            return "即将";
        }
    }
}

// 测试相对时间格式化
$past = new DateTimeImmutable('-2 hours');
echo formatRelativeTime($past) . "\n";  // 2小时前

$future = new DateTimeImmutable('+3 days');
echo formatRelativeTime($future) . "\n";  // 3天后

$recent = new DateTimeImmutable('-30 seconds');
echo formatRelativeTime($recent) . "\n";  // 刚刚
```

**输出**：

```
2小时前
3天后
刚刚
```

**说明**：
- 相对时间格式化将日期时间转换为人类可读的相对时间
- 适用于社交网络、评论系统等场景

## 使用场景

### 场景 1：数据库存储格式

使用标准格式存储日期时间到数据库。

**示例**：

```php
<?php
declare(strict_types=1);

// 存储时使用标准格式
$date = new DateTimeImmutable('now', new DateTimeZone('UTC'));
$dbValue = $date->format('Y-m-d H:i:s');
// 保存到数据库

// 从数据库读取并格式化
$dbTime = DateTimeImmutable::createFromFormat('Y-m-d H:i:s', $dbValue, new DateTimeZone('UTC'));
echo $dbTime->format('Y-m-d H:i:s') . "\n";
```

### 场景 2：用户输入解析

解析用户输入的各种日期时间格式。

**示例**：见"示例 12：解析多种格式"

### 场景 3：API 接口格式

在 API 接口中使用标准格式（ISO 8601）。

**示例**：

```php
<?php
declare(strict_types=1);

class ApiResponse
{
    public function formatDateTime(DateTimeImmutable $date): string
    {
        return $date->format('c');  // ISO 8601 格式
    }
}

$date = new DateTimeImmutable('now', new DateTimeZone('UTC'));
$api = new ApiResponse();
echo $api->formatDateTime($date);  // 2025-01-15T14:30:45+00:00
```

## 注意事项

### 格式化字符串区分大小写

格式化字符串严格区分大小写，使用时需注意。

**示例**：

```php
<?php
declare(strict_types=1);

$date = new DateTime('2025-01-15 14:30:45');
echo $date->format('Y') . "\n";  // 2025（4位年份）
echo $date->format('y') . "\n";  // 25（2位年份）
echo $date->format('H') . "\n";  // 14（24小时制）
echo $date->format('h') . "\n";  // 02（12小时制）
```

### 格式字符串必须匹配

`createFromFormat()` 要求格式字符串与输入字符串完全匹配。

**示例**：

```php
<?php
declare(strict_types=1);

// 正确：格式匹配
$date1 = DateTime::createFromFormat('Y-m-d', '2025-01-15');  // 成功

// 错误：格式不匹配
$date2 = DateTime::createFromFormat('Y-m-d', '2025/01/15');  // 失败，返回 false
```

### 解析失败的处理

`createFromFormat()` 可能返回 `false`，必须进行错误处理。

**示例**：见"示例 10：解析错误处理"

### 时区影响

格式化时会考虑时区，确保时区设置正确。

**示例**：

```php
<?php
declare(strict_types=1);

$utc = new DateTime('2025-01-15 10:00:00', new DateTimeZone('UTC'));
echo $utc->format('Y-m-d H:i:s T') . "\n";  // 2025-01-15 10:00:00 UTC

$beijing = $utc->setTimezone(new DateTimeZone('Asia/Shanghai'));
echo $beijing->format('Y-m-d H:i:s T') . "\n";  // 2025-01-15 18:00:00 CST
```

## 常见问题

### 问题 1：格式化字符串区分大小写吗？

**回答**：是的，格式化字符串严格区分大小写。例如：
- `Y` 表示 4 位数字年份（2025）
- `y` 表示 2 位数字年份（25）
- `H` 表示 24 小时制小时（00-23）
- `h` 表示 12 小时制小时（01-12）

**示例**：见"示例 2：年份格式化"

### 问题 2：如何输出格式化字符本身？

**回答**：使用反斜杠 `\` 转义格式化字符。

**示例**：见"示例 8：转义字符"

### 问题 3：createFromFormat() 解析失败怎么办？

**回答**：`createFromFormat()` 失败时返回 `false`，可以使用 `DateTime::getLastErrors()` 获取详细错误信息。

**示例**：见"示例 10：解析错误处理"

### 问题 4：如何解析多种日期格式？

**回答**：可以尝试多种格式，直到解析成功。

**示例**：见"示例 12：解析多种格式"

### 问题 5：如何格式化中文日期？

**回答**：`format()` 方法不支持中文，需要使用 `IntlDateFormatter` 或自定义格式化函数。

**示例**：

```php
<?php
declare(strict_types=1);

// 方法 1：使用 IntlDateFormatter（需要 intl 扩展）
if (extension_loaded('intl')) {
    $date = new DateTime('2025-01-15 14:30:45');
    $formatter = new IntlDateFormatter(
        'zh_CN',
        IntlDateFormatter::FULL,
        IntlDateFormatter::FULL,
        'Asia/Shanghai'
    );
    echo $formatter->format($date) . "\n";  // 中文格式
}

// 方法 2：自定义格式化（简单情况）
function formatChineseDate(DateTime $date): string
{
    $months = ['一月', '二月', '三月', '四月', '五月', '六月', 
               '七月', '八月', '九月', '十月', '十一月', '十二月'];
    $month = (int)$date->format('n') - 1;
    return $date->format('Y') . '年' . $months[$month] . $date->format('d') . '日';
}

$date = new DateTime('2025-01-15');
echo formatChineseDate($date) . "\n";  // 2025年一月15日
```

## 最佳实践

### 1. 使用标准格式存储

在数据库中存储日期时间时，使用标准格式（ISO 8601 或 `Y-m-d H:i:s`）。

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：使用标准格式
$date = new DateTimeImmutable('now', new DateTimeZone('UTC'));
$dbValue = $date->format('Y-m-d H:i:s');
```

### 2. 明确格式化字符含义

使用格式化字符时，明确其含义，避免混淆。

**示例**：

```php
<?php
declare(strict_types=1);

$date = new DateTime('2025-01-15 14:30:45');

// 推荐：使用明确的格式化字符
echo $date->format('Y-m-d H:i:s') . "\n";  // 明确：4位年份，24小时制

// 不推荐：使用容易混淆的字符
echo $date->format('y-m-d h:i:s') . "\n";  // 容易混淆：2位年份，12小时制
```

### 3. 错误处理

解析日期时间时，始终进行错误处理。

**示例**：

```php
<?php
declare(strict_types=1);

function safeParseDate(string $dateString, string $format): ?DateTime
{
    $date = DateTime::createFromFormat($format, $dateString);
    if ($date === false) {
        return null;
    }
    return $date;
}

$date = safeParseDate('2025-01-15', 'Y-m-d');
if ($date !== null) {
    echo $date->format('Y-m-d') . "\n";
}
```

### 4. 使用配置管理格式

在配置中统一管理日期格式，便于维护。

**示例**：

```php
<?php
// config/date_formats.php
return [
    'database' => 'Y-m-d H:i:s',
    'display' => 'Y-m-d H:i',
    'api' => 'c',  // ISO 8601
];

// 使用
$config = require 'config/date_formats.php';
$date = new DateTimeImmutable();
echo $date->format($config['database']) . "\n";
```

### 5. 使用 IntlDateFormatter 进行多语言格式化

对于多语言应用，使用 `IntlDateFormatter` 而不是 `format()`。

**示例**：

```php
<?php
declare(strict_types=1);

if (extension_loaded('intl')) {
    $date = new DateTime('2025-01-15 14:30:45');
    $formatter = new IntlDateFormatter(
        'zh_CN',
        IntlDateFormatter::FULL,
        IntlDateFormatter::FULL,
        'Asia/Shanghai'
    );
    echo $formatter->format($date) . "\n";  // 中文格式
}
```

## 对比分析

### format() vs createFromFormat()

| 特性         | format()                        | createFromFormat()              |
|:-------------|:--------------------------------|:--------------------------------|
| **用途**      | DateTime → 字符串                | 字符串 → DateTime                |
| **方向**      | 输出                            | 输入                            |
| **参数**      | 格式化字符串                    | 格式字符串 + 输入字符串          |
| **返回值**    | 字符串                          | DateTime\|false                 |
| **错误处理**  | 不返回错误                      | 返回 false 表示失败              |

### 格式化字符 vs IntlDateFormatter

| 特性         | 格式化字符（format()）           | IntlDateFormatter               |
|:-------------|:--------------------------------|:--------------------------------|
| **多语言支持** | ❌ 不支持（仅英文）              | ✅ 完整支持                     |
| **性能**      | ✅ 更快                         | ⚠️ 较慢                         |
| **功能丰富度** | ⚠️ 基础功能                     | ✅ 更丰富（本地化、时区等）      |
| **依赖**      | ✅ 无需额外扩展                  | ❌ 需要 intl 扩展               |
| **使用场景**  | 简单格式化、性能敏感             | 多语言应用、复杂格式化           |

## 练习任务

1. **日期格式化工具类**：创建一个 `DateFormatter` 类，封装常用格式化模式（ISO 8601、数据库格式、可读格式等）。

2. **多格式日期解析**：编写一个函数，解析多种日期格式（如 `Y-m-d`、`m/d/Y`、`d-m-Y`），自动尝试多种格式直到成功。

3. **多语言日期格式化**：实现一个多语言日期格式化函数，支持中英文切换，使用 `IntlDateFormatter`（如果可用）。

4. **相对时间格式化**：创建一个相对时间格式化函数，支持"刚刚"、"2小时前"、"3天前"等格式，并处理未来时间。

5. **日期格式验证**：编写一个函数，验证日期字符串是否符合指定格式，返回布尔值。

## 相关章节

- **[4.1.1 DateTime 基础](section-01-datetime-basics.md)**：了解 DateTime 类的基础用法
- **[4.1.2 时区处理](section-02-timezone.md)**：了解时区对格式化的影响
- **[4.1.4 时间计算与比较](section-04-calculations-comparisons.md)**：学习时间的计算和比较操作
