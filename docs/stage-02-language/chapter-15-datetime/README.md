# 2.15 时间与日期处理

## 目标

- 掌握基于 `DateTime`、`DateTimeImmutable` 的现代日期处理方式。
- 理解时区、DST、时间戳、格式化字符串。
- 能使用 `Carbon` 等扩展库处理复杂时间逻辑。

## 章节内容

本章分为一个独立小节，提供详细的概念解释、语法说明、参数列表和完整示例：

1. **[DateTime 基础](section-01-datetime-basics.md)**：`DateTime` 和 `DateTimeImmutable` 的创建、格式化、修改、时区处理、时间戳转换、时间差计算及完整示例。

## 时间戳与基础函数

- `time()`：返回当前 Unix 时间戳（秒）。
- `microtime(true)`：返回浮点数，包含微秒。
- `date($format, $timestamp = time())`：格式化时间戳。

## DateTime / DateTimeImmutable

- `DateTime` 可变；`DateTimeImmutable` 每次操作返回新实例。
- 建议在业务逻辑中优先使用不可变对象。

## 学习建议

1. **重点掌握**：
   - `DateTime` 和 `DateTimeImmutable` 的区别
   - 时区处理的方法
   - 时间格式化和计算
   - 时间差的计算

2. **实践练习**：
   - 完成练习题目
   - 实现实际的日期处理功能
   - 处理时区转换

## 完成本章后

- 能够使用 `DateTime` 和 `DateTimeImmutable` 处理日期时间。
- 理解时区的概念和处理方法。
- 掌握时间格式化和计算。
- 能够在实际项目中处理日期时间相关的需求。

## 相关章节

- **2.4 数据类型**：了解时间相关的数据类型。
- **阶段五：数据持久化与日志管理**：了解日志中的时间处理。
