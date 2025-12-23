# 2.14.3 格式化与解析

## 概述

日期时间的格式化与解析是日常开发中的高频操作。本节详细介绍 `format()` 方法、格式化字符串完整列表、`createFromFormat()` 解析、多语言格式化，以及相对时间格式化。

## format() 方法

### 基本用法

```php
<?php
declare(strict_types=1);

$date = new DateTime('2024-01-15 14:30:45');
echo $date->format('Y-m-d H:i:s') . "\n";  // 2024-01-15 14:30:45
```

### 格式化字符串完整列表

#### 年份

| 字符 | 说明 | 示例 |
| :--- | :--- | :--- |
| `Y`  | 4 位数字年份 | 2024 |
| `y`  | 2 位数字年份 | 24 |
| `o`  | ISO-8601 周数年份 | 2024 |

#### 月份

| 字符 | 说明 | 示例 |
| :--- | :--- | :--- |
| `m`  | 带前导零的月份 | 01-12 |
| `n`  | 不带前导零的月份 | 1-12 |
| `M`  | 3 字母月份缩写 | Jan-Dec |
| `F`  | 完整月份名称 | January-December |
| `t`  | 月份天数 | 28-31 |

#### 日期

| 字符 | 说明 | 示例 |
| :--- | :--- | :--- |
| `d`  | 带前导零的日期 | 01-31 |
| `j`  | 不带前导零的日期 | 1-31 |
| `S`  | 日期后缀 | st, nd, rd, th |
| `w`  | 星期几（数字） | 0（周日）-6（周六） |
| `N`  | ISO-8601 星期几 | 1（周一）-7（周日） |
| `D`  | 3 字母星期缩写 | Mon-Sun |
| `l`  | 完整星期名称 | Monday-Sunday |

#### 时间

| 字符 | 说明 | 示例 |
| :--- | :--- | :--- |
| `H`  | 24 小时制（带前导零） | 00-23 |
| `G`  | 24 小时制（不带前导零） | 0-23 |
| `h`  | 12 小时制（带前导零） | 01-12 |
| `g`  | 12 小时制（不带前导零） | 1-12 |
| `i`  | 分钟（带前导零） | 00-59 |
| `s`  | 秒（带前导零） | 00-59 |
| `u`  | 微秒 | 000000-999999 |
| `a`  | 小写 am/pm | am/pm |
| `A`  | 大写 AM/PM | AM/PM |

#### 时区

| 字符 | 说明 | 示例 |
| :--- | :--- | :--- |
| `e`  | 时区标识符 | UTC, Asia/Shanghai |
| `T`  | 时区缩写 | UTC, CST, EST |
| `O`  | 时区偏移（小时） | +0800 |
| `P`  | 时区偏移（带冒号） | +08:00 |
| `Z`  | 时区偏移（秒） | 28800 |

#### 其他

| 字符 | 说明 | 示例 |
| :--- | :--- | :--- |
| `c`  | ISO-8601 日期 | 2024-01-15T14:30:45+08:00 |
| `r`  | RFC 2822 日期 | Mon, 15 Jan 2024 14:30:45 +0800 |
| `U`  | Unix 时间戳 | 1705294245 |

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

### 基本用法

```php
<?php
declare(strict_types=1);

// 从字符串解析
$date = DateTime::createFromFormat('Y-m-d H:i:s', '2024-01-15 14:30:45');
echo $date->format('Y-m-d H:i:s') . "\n";

// 从自定义格式解析
$date = DateTime::createFromFormat('d/m/Y', '15/01/2024');
echo $date->format('Y-m-d') . "\n";  // 2024-01-15
```

### 常见解析场景

```php
<?php
declare(strict_types=1);

// 解析美国日期格式
$date = DateTime::createFromFormat('m/d/Y', '01/15/2024');

// 解析带时间的格式
$date = DateTime::createFromFormat('Y-m-d H:i:s.u', '2024-01-15 14:30:45.123456');

// 解析相对时间
$date = DateTime::createFromFormat('Y-m-d', '2024-01-15');
```

### 错误处理

```php
<?php
declare(strict_types=1);

$date = DateTime::createFromFormat('Y-m-d', 'invalid-date');
if ($date === false) {
    $errors = DateTime::getLastErrors();
    echo "Parse errors:\n";
    foreach ($errors['errors'] as $error) {
        echo "- " . $error . "\n";
    }
}
```

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

## 注意事项

1. **格式化字符串区分大小写**：`Y`（4位年份）和 `y`（2位年份）不同。
2. **转义字符**：使用反斜杠转义特殊字符，如 `format('\Y\e\a\r')`。
3. **时区影响**：格式化时会考虑时区，确保时区设置正确。
4. **性能考虑**：频繁格式化可考虑缓存格式化结果。

## 练习

1. 创建一个日期格式化工具类，封装常用格式化模式。
2. 编写一个函数，解析多种日期格式（如 `Y-m-d`、`m/d/Y`、`d-m-Y`）。
3. 实现一个多语言日期格式化函数，支持中英文切换。
4. 创建一个相对时间格式化函数，支持"刚刚"、"2小时前"等格式。
5. 编写一个函数，验证日期字符串是否符合指定格式。
