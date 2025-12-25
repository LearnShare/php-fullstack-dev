# 2.12.1 isset、empty 与 is_null

## 概述

`isset`、`empty` 和 `is_null` 是检查变量状态的常用函数。本节介绍这三个函数的语法、返回值、使用场景、函数对比及完整示例。

**章节类型**：工具性章节

**主要内容**：
- `isset()` 的语法和用法
- `empty()` 的语法和用法
- `is_null()` 的语法和用法
- 三个函数的对比
- 使用场景和选择建议
- 常见陷阱和注意事项

## 特性

- **isset**：检查变量是否已设置且不为 `null`
- **empty**：检查变量是否为空（falsy 值）
- **is_null**：检查变量是否为 `null`

## 基本用法/命令

### isset()

**语法**：`isset(mixed ...$vars): bool`
**返回值**：变量已设置且不为 `null` 返回 `true`
**特点**：不会触发 Notice

### empty()

**语法**：`empty(mixed $var): bool`
**返回值**：变量为空（falsy 值）返回 `true`
**特点**：等价于 `!isset($var) || $var == false`

### is_null()

**语法**：`is_null(mixed $value): bool`
**返回值**：变量为 `null` 返回 `true`
**特点**：等价于 `$value === null`

## 使用场景

- **isset**：检查变量是否存在
- **empty**：检查变量是否为空
- **is_null**：检查变量是否为 `null`

## 注意事项

- **empty("0")**：字符串 "0" 被认为是空
- **isset vs is_null**：`isset()` 对未定义变量返回 `false`，`is_null()` 会触发 Notice
- **性能**：`isset()` 性能最好

## 常见问题

### 问题 1：empty("0") 陷阱
- **原因**：字符串 "0" 被认为是空
- **解决**：使用 `=== ""` 或 `=== null` 检查

### 问题 2：isset vs is_null
- **区别**：`isset()` 对未定义变量返回 `false`，`is_null()` 会触发 Notice
- **选择**：检查变量是否存在使用 `isset()`，检查是否为 `null` 使用 `is_null()`

## 最佳实践

- 检查变量是否存在使用 `isset()`
- 检查变量是否为空使用 `empty()` 或显式检查
- 检查变量是否为 `null` 使用 `is_null()` 或 `=== null`
- 理解三个函数的区别和适用场景
