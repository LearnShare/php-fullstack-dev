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

## Carbon（可选）

- `nesbot/carbon` 扩展了 `DateTime`，提供链式 API。

```php
use Carbon\CarbonImmutable;

$nextWeek = CarbonImmutable::now()->addWeek();
echo $nextWeek->diffForHumans(); // “1周后”
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
