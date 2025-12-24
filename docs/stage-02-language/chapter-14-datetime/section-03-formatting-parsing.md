# 2.14.3 格式化与解析

## 概述

日期时间的格式化与解析是日常开发中的高频操作。本节详细介绍 `format()` 方法、格式化字符串完整列表、`createFromFormat()` 解析、多语言格式化，以及相对时间格式化。掌握这些内容对于处理各种日期时间格式至关重要。

## 特性

- **丰富的格式化字符**：支持 40+ 种格式化字符，满足各种格式化需求
- **灵活的解析功能**：支持从任意格式字符串解析日期时间
- **多语言支持**：支持使用 `IntlDateFormatter` 进行多语言格式化
- **相对时间格式化**：支持将日期时间格式化为相对时间（如"2小时前"）
- **错误处理**：提供完善的错误处理机制

## format() 方法

### format() - 格式化日期时间

**语法**：`format(string $format): string`

**参数**：
- `$format`：格式化字符串，使用特定字符表示日期时间各部分。格式化字符区分大小写，特殊字符需要使用反斜杠转义。

**返回值**：返回格式化后的日期时间字符串。

### 基本用法

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15 14:30:45');
echo $date->format('Y-m-d H:i:s') . "\n";  // 2024-01-15 14:30:45
```

### 格式化字符串完整列表

#### 年份格式化字符

| 字符 | 说明                           | 示例   | 范围           |
|:-----|:-------------------------------|:-------|:---------------|
| `Y`  | 4 位数字年份                   | 2024   | 0000-9999      |
| `y`  | 2 位数字年份                   | 24     | 00-99          |
| `o`  | ISO-8601 周数年份              | 2024   | 0000-9999      |
| `L`  | 是否为闰年（1=是，0=否）        | 1      | 0-1            |
| `B`  | Swatch Internet Time（.beat） | 500    | 000-999        |

#### 月份格式化字符

| 字符 | 说明                    | 示例        | 范围      |
|:-----|:------------------------|:------------|:----------|
| `m`  | 带前导零的月份          | 01          | 01-12     |
| `n`  | 不带前导零的月份        | 1           | 1-12      |
| `M`  | 3 字母月份缩写（英文）   | Jan         | Jan-Dec   |
| `F`  | 完整月份名称（英文）     | January     | January-December |
| `t`  | 月份天数                | 31          | 28-31     |

#### 日期格式化字符

| 字符 | 说明                    | 示例        | 范围           |
|:-----|:------------------------|:------------|:---------------|
| `d`  | 带前导零的日期          | 01          | 01-31          |
| `j`  | 不带前导零的日期        | 1           | 1-31           |
| `S`  | 日期后缀（英文）        | st, nd, rd, th | st, nd, rd, th |
| `z`  | 一年中的第几天          | 15          | 0-365          |
| `W`  | ISO-8601 周数           | 03          | 01-53          |
| `w`  | 星期几（数字，0=周日）   | 1           | 0-6            |
| `N`  | ISO-8601 星期几（1=周一）| 1           | 1-7            |
| `D`  | 3 字母星期缩写（英文）   | Mon         | Mon-Sun         |
| `l`  | 完整星期名称（英文）     | Monday      | Monday-Sunday   |

#### 时间格式化字符

| 字符 | 说明                    | 示例        | 范围           |
|:-----|:------------------------|:------------|:---------------|
| `H`  | 24 小时制（带前导零）    | 14          | 00-23          |
| `G`  | 24 小时制（不带前导零）  | 14          | 0-23           |
| `h`  | 12 小时制（带前导零）    | 02          | 01-12          |
| `g`  | 12 小时制（不带前导零）  | 2           | 1-12           |
| `i`  | 分钟（带前导零）         | 30          | 00-59          |
| `s`  | 秒（带前导零）           | 45          | 00-59          |
| `u`  | 微秒                     | 123456      | 000000-999999  |
| `v`  | 毫秒                     | 123         | 000-999        |
| `a`  | 小写 am/pm               | pm          | am/pm          |
| `A`  | 大写 AM/PM               | PM          | AM/PM          |

#### 时区格式化字符

| 字符 | 说明                    | 示例            | 范围           |
|:-----|:------------------------|:----------------|:---------------|
| `e`  | 时区标识符              | Asia/Shanghai  | 时区标识符      |
| `T`  | 时区缩写                | CST             | 时区缩写        |
| `O`  | 时区偏移（小时，无冒号） | +0800           | ±1200          |
| `P`  | 时区偏移（小时，带冒号） | +08:00          | ±12:00         |
| `Z`  | 时区偏移（秒）           | 28800           | -43200 到 50400 |
| `I`  | 是否为夏令时（1=是，0=否）| 0              | 0-1            |

#### 其他格式化字符

| 字符 | 说明                    | 示例                        | 范围           |
|:-----|:------------------------|:----------------------------|:---------------|
| `c`  | ISO-8601 日期            | 2024-01-15T14:30:45+08:00  | ISO-8601 格式  |
| `r`  | RFC 2822 日期            | Mon, 15 Jan 2024 14:30:45 +0800 | RFC 2822 格式 |
| `U`  | Unix 时间戳（秒）        | 1705294245                  | 整数           |
| `\`  | 转义字符                 | \Y\e\a\r                    | 转义特殊字符   |

### 转义字符

在格式化字符串中，如果需要输出格式化字符本身，需要使用反斜杠 `\` 进行转义：

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15');
echo $date->format('\Y\e\a\r: Y') . "\n";  // Year: 2024
echo $date->format('Y-m-d \a\t H:i:s') . "\n";  // 2024-01-15 at 14:30:45
```

### 常用格式化模式

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15 14:30:45');

// ISO 8601
echo $date->format('c') . "\n";  // 2024-01-15T14:30:45+08:00

// 数据库格式
echo $date->format('Y-m-d H:i:s') . "\n";  // 2024-01-15 14:30:45

// 可读格式
echo $date->format('l, F j, Y') . "\n";  // Monday, January 15, 2024

// 简短格式
echo $date->format('M j, Y') . "\n";  // Jan 15, 2024

// 时间格式
echo $date->format('g:i A') . "\n";  // 2:30 PM
```

## createFromFormat() 解析

### createFromFormat() - 从格式字符串创建

**语法**：`static createFromFormat(string $format, string $datetime, ?DateTimeZone $timezone = null): DateTime|false`

**参数**：
- `$format`：日期时间格式字符串，使用格式化字符定义输入格式。格式字符串必须与 `$datetime` 参数完全匹配。
- `$datetime`：要解析的日期时间字符串，必须与 `$format` 参数定义的格式完全匹配。
- `$timezone`：可选，`DateTimeZone` 对象，指定时区。如果为 `null`，使用默认时区。

**返回值**：成功返回 `DateTime` 对象，失败返回 `false`。可以使用 `DateTime::getLastErrors()` 获取错误信息。

**注意**：
- 格式字符串必须与输入字符串完全匹配
- 如果格式不匹配，返回 `false`
- 解析时忽略格式字符串中未定义的字符
- 可以使用 `!` 前缀重置未指定的部分为 Unix 纪元（1970-01-01 00:00:00）

### 基本用法

```php
<?php
declare(strict_types=1);

// 从字符串解析
$date = DateTime::createFromFormat('Y-m-d H:i:s', '2024-01-15 14:30:45');
if ($date !== false) {
    echo $date->format('Y-m-d H:i:s') . "\n";  // 2024-01-15 14:30:45
}

// 从自定义格式解析
$date = DateTime::createFromFormat('d/m/Y', '15/01/2024');
if ($date !== false) {
    echo $date->format('Y-m-d') . "\n";  // 2024-01-15
}
```

### 常见解析场景

```php
<?php
declare(strict_types=1);

// 解析美国日期格式（月/日/年）
$date = DateTime::createFromFormat('m/d/Y', '01/15/2024');
if ($date !== false) {
    echo $date->format('Y-m-d') . "\n";  // 2024-01-15
}

// 解析带时间的格式
$date = DateTime::createFromFormat('Y-m-d H:i:s.u', '2024-01-15 14:30:45.123456');
if ($date !== false) {
    echo $date->format('Y-m-d H:i:s.u') . "\n";
}

// 解析只有日期的格式
$date = DateTime::createFromFormat('Y-m-d', '2024-01-15');
if ($date !== false) {
    echo $date->format('Y-m-d H:i:s') . "\n";  // 2024-01-15 00:00:00
}

// 解析带时区的格式
$date = DateTime::createFromFormat('Y-m-d H:i:s T', '2024-01-15 14:30:45 CST');
if ($date !== false) {
    echo $date->format('Y-m-d H:i:s T') . "\n";
}
```

### 使用 ! 前缀重置未指定部分

使用 `!` 前缀可以重置未指定的部分为 Unix 纪元（1970-01-01 00:00:00）：

```php
<?php
declare(strict_types=1);

// 不使用 ! 前缀：未指定的部分使用当前时间
$date = DateTime::createFromFormat('Y-m-d', '2024-01-15');
echo $date->format('Y-m-d H:i:s') . "\n";  // 2024-01-15 当前时间

// 使用 ! 前缀：未指定的部分重置为 00:00:00
$date = DateTime::createFromFormat('!Y-m-d', '2024-01-15');
echo $date->format('Y-m-d H:i:s') . "\n";  // 2024-01-15 00:00:00
```

### 错误处理

```php
<?php
declare(strict_types=1);

$date = DateTime::createFromFormat('Y-m-d', 'invalid-date');
if ($date === false) {
    $errors = DateTime::getLastErrors();
    if ($errors !== false) {
        echo "Parse errors:\n";
        foreach ($errors['errors'] as $error) {
            echo "- " . $error . "\n";
        }
        echo "\nWarnings:\n";
        foreach ($errors['warnings'] as $warning) {
            echo "- " . $warning . "\n";
        }
    }
}
```

### getLastErrors() - 获取最后错误

**语法**：`static getLastErrors(): array|false`

**参数**：无

**返回值**：返回包含 `errors` 和 `warnings` 键的数组，如果没有错误返回 `false`。

**返回数组结构**：
- `errors`：解析错误数组
- `warnings`：解析警告数组

## 多语言格式化

### 使用 setlocale()

```php
<?php
declare(strict_types=1);

// 设置中文环境
setlocale(LC_TIME, 'zh_CN.UTF-8', 'zh_CN', 'chinese');

$date = new DateTime('2024-01-15');
echo strftime('%Y年%m月%d日', $date->getTimestamp()) . "\n";
```

### 使用 IntlDateFormatter（推荐）

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15 14:30:45');
$formatter = new IntlDateFormatter(
    'zh_CN',
    IntlDateFormatter::FULL,
    IntlDateFormatter::FULL,
    'Asia/Shanghai',
    IntlDateFormatter::GREGORIAN,
    'yyyy年MM月dd日 HH:mm:ss'
);
echo $formatter->format($date) . "\n";  // 2024年01月15日 14:30:45
```

## 相对时间格式化

### 自定义相对时间函数

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
            return $diff->y . '年前';
        } elseif ($diff->m > 0) {
            return $diff->m . '个月前';
        } elseif ($diff->d > 0) {
            return $diff->d . '天前';
        } elseif ($diff->h > 0) {
            return $diff->h . '小时前';
        } elseif ($diff->i > 0) {
            return $diff->i . '分钟前';
        } else {
            return '刚刚';
        }
    } else {
        // 未来时间
        if ($diff->y > 0) {
            return $diff->y . '年后';
        } elseif ($diff->m > 0) {
            return $diff->m . '个月后';
        } elseif ($diff->d > 0) {
            return $diff->d . '天后';
        } elseif ($diff->h > 0) {
            return $diff->h . '小时后';
        } elseif ($diff->i > 0) {
            return $diff->i . '分钟后';
        } else {
            return '即将';
        }
    }
}

$date = new DateTimeImmutable('-2 hours');
echo formatRelativeTime($date) . "\n";  // 2小时前
```

## 完整示例

```php
<?php
declare(strict_types=1);

class FormattingDemo
{
    public static function demonstrate(): void
    {
        $date = new DateTime('2024-01-15 14:30:45');
        
        echo "=== 基本格式化 ===\n";
        echo "ISO 8601: " . $date->format('c') . "\n";
        echo "数据库格式: " . $date->format('Y-m-d H:i:s') . "\n";
        echo "可读格式: " . $date->format('l, F j, Y') . "\n";
        
        echo "\n=== 解析 ===\n";
        $parsed = DateTime::createFromFormat('d/m/Y', '15/01/2024');
        echo "解析结果: " . $parsed->format('Y-m-d') . "\n";
        
        echo "\n=== 相对时间 ===\n";
        $relative = new DateTimeImmutable('-2 hours');
        echo formatRelativeTime($relative) . "\n";
    }
}

FormattingDemo::demonstrate();
```

## 常见问题

### 问题 1：格式化字符串区分大小写吗？

**回答**：是的，格式化字符串严格区分大小写。例如：
- `Y` 表示 4 位数字年份（2024）
- `y` 表示 2 位数字年份（24）
- `H` 表示 24 小时制小时（00-23）
- `h` 表示 12 小时制小时（01-12）

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15 14:30:45');
echo $date->format('Y') . "\n";  // 2024
echo $date->format('y') . "\n";  // 24
```

### 问题 2：如何输出格式化字符本身？

**回答**：使用反斜杠 `\` 转义格式化字符：

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15');
echo $date->format('\Y\e\a\r: Y') . "\n";  // Year: 2024
echo $date->format('Y-m-d \a\t H:i:s') . "\n";  // 2024-01-15 at 14:30:45
```

### 问题 3：createFromFormat() 解析失败怎么办？

**回答**：`createFromFormat()` 失败时返回 `false`，可以使用 `DateTime::getLastErrors()` 获取详细错误信息：

```php
<?php
declare(strict_types=1);

$date = DateTime::createFromFormat('Y-m-d', 'invalid-date');
if ($date === false) {
    $errors = DateTime::getLastErrors();
    if ($errors !== false) {
        print_r($errors);
    }
}
```

### 问题 4：如何解析多种日期格式？

**回答**：可以尝试多种格式，直到解析成功：

```php
<?php
declare(strict_types=1);

function parseDate(string $dateString): ?DateTime
{
    $formats = ['Y-m-d', 'm/d/Y', 'd-m-Y', 'Y/m/d'];
    foreach ($formats as $format) {
        $date = DateTime::createFromFormat($format, $dateString);
        if ($date !== false) {
            return $date;
        }
    }
    return null;
}

$date = parseDate('2024-01-15');
if ($date !== null) {
    echo $date->format('Y-m-d') . "\n";
}
```

## 最佳实践

### 1. 使用标准格式存储

在数据库中存储日期时间时，使用标准格式（ISO 8601 或 `Y-m-d H:i:s`）：

```php
<?php
declare(strict_types=1);

// 推荐：使用标准格式
$date = new DateTimeImmutable('now', new DateTimeZone('UTC'));
$dbValue = $date->format('Y-m-d H:i:s');
```

### 2. 明确格式化字符含义

使用格式化字符时，明确其含义，避免混淆：

```php
<?php
declare(strict_types=1);

// 推荐：使用明确的格式化字符
$date = new DateTime('2024-01-15 14:30:45');
echo $date->format('Y-m-d H:i:s') . "\n";  // 明确：4位年份，24小时制

// 不推荐：使用容易混淆的字符
echo $date->format('y-m-d h:i:s') . "\n";  // 容易混淆：2位年份，12小时制
```

### 3. 错误处理

解析日期时间时，始终进行错误处理：

```php
<?php
declare(strict_types=1);

$date = DateTime::createFromFormat('Y-m-d', $userInput);
if ($date === false) {
    // 处理错误
    throw new InvalidArgumentException('Invalid date format');
}
```

### 4. 缓存格式化结果

如果频繁格式化相同的日期时间，考虑缓存结果：

```php
<?php
declare(strict_types=1);

class DateFormatter
{
    private static array $cache = [];

    public static function format(DateTimeImmutable $date, string $format): string
    {
        $key = $date->getTimestamp() . '|' . $format;
        if (!isset(self::$cache[$key])) {
            self::$cache[$key] = $date->format($format);
        }
        return self::$cache[$key];
    }
}
```

### 5. 使用 IntlDateFormatter 进行多语言格式化

对于多语言应用，使用 `IntlDateFormatter` 而不是 `format()`：

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15 14:30:45');
$formatter = new IntlDateFormatter(
    'zh_CN',
    IntlDateFormatter::FULL,
    IntlDateFormatter::FULL,
    'Asia/Shanghai'
);
echo $formatter->format($date) . "\n";  // 中文格式
```

## 对比分析

### format() vs createFromFormat()

| 特性                 | format()                    | createFromFormat()         |
|:---------------------|:----------------------------|:---------------------------|
| **用途**             | 将 DateTime 格式化为字符串  | 从字符串解析为 DateTime    |
| **方向**             | DateTime → 字符串           | 字符串 → DateTime          |
| **参数**             | 格式化字符串                | 格式字符串 + 输入字符串    |
| **返回值**           | 字符串                      | DateTime\|false            |
| **错误处理**         | 不返回错误（总是返回字符串）| 返回 false 表示失败        |

### 格式化字符 vs IntlDateFormatter

| 特性                 | 格式化字符（format()）      | IntlDateFormatter          |
|:---------------------|:----------------------------|:---------------------------|
| **多语言支持**       | 不支持（仅英文）            | 完整支持                   |
| **性能**             | 更快                        | 较慢                       |
| **功能丰富度**       | 基础功能                    | 更丰富（本地化、时区等）   |
| **依赖**             | 无需额外扩展                | 需要 intl 扩展             |
| **使用场景**         | 简单格式化、性能敏感        | 多语言应用、复杂格式化     |

## 注意事项

1. **格式化字符串区分大小写**：`Y`（4位年份）和 `y`（2位年份）不同，使用时需注意。

2. **转义字符**：使用反斜杠 `\` 转义特殊字符，如 `format('\Y\e\a\r')`。

3. **时区影响**：格式化时会考虑时区，确保时区设置正确。

4. **性能考虑**：频繁格式化可考虑缓存格式化结果，提高性能。

5. **错误处理**：`createFromFormat()` 可能返回 `false`，必须进行错误处理。

6. **格式匹配**：`createFromFormat()` 要求格式字符串与输入字符串完全匹配。

7. **未指定部分**：使用 `!` 前缀可以重置未指定的部分为 Unix 纪元。

8. **多语言支持**：`format()` 仅支持英文，多语言应用应使用 `IntlDateFormatter`。

## 练习

1. 创建一个日期格式化工具类，封装常用格式化模式（ISO 8601、数据库格式、可读格式等）。

2. 编写一个函数，解析多种日期格式（如 `Y-m-d`、`m/d/Y`、`d-m-Y`），自动尝试多种格式直到成功。

3. 实现一个多语言日期格式化函数，支持中英文切换，使用 `IntlDateFormatter`。

4. 创建一个相对时间格式化函数，支持"刚刚"、"2小时前"、"3天前"等格式，并处理未来时间。

5. 编写一个函数，验证日期字符串是否符合指定格式，返回布尔值。
