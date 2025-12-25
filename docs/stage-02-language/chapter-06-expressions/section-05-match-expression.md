# 2.6.5 match 表达式

## 概述

`match` 表达式是 PHP 8.0 引入的新特性，是 `switch` 语句的改进版本。本节介绍 `match` 表达式的语法、与 `switch` 的对比、严格比较、返回值、表达式模式及完整示例。

**章节类型**：语法性章节

**主要内容**：
- `match` 表达式的语法
- 与 `switch` 语句的对比
- 严格比较特性
- 返回值特性
- 表达式模式
- 使用场景和最佳实践

## 特性

- **严格比较**：使用 `===` 进行比较
- **返回值**：可以直接返回值
- **表达式模式**：支持更复杂的匹配模式
- **无贯穿**：不需要 `break`

## 语法/定义

### match 表达式语法

**语法**：`match ($value) { pattern1 => result1, pattern2 => result2, default => default_result }`
**特点**：严格比较、返回值、无贯穿

### 与 switch 对比

| 特性 | `switch` | `match` |
| :--- | :------- | :------ |
| 比较方式 | 宽松比较（`==`） | 严格比较（`===`） |
| 返回值 | 不能直接返回值 | 可以返回值 |
| break | 需要 `break` | 不需要 |
| 默认分支 | `default:` | `default =>` |

## 基本用法

### 基础示例

```php
<?php
declare(strict_types=1);

$status = match($code) {
    200 => "OK",
    404 => "Not Found",
    500 => "Server Error",
    default => "Unknown"
};
```

### 多个值匹配示例

```php
<?php
declare(strict_types=1);

$result = match($value) {
    1, 2, 3 => "Small",
    4, 5, 6 => "Medium",
    7, 8, 9 => "Large",
    default => "Unknown"
};
```

### 表达式模式示例

```php
<?php
declare(strict_types=1);

$result = match(true) {
    $value < 0 => "Negative",
    $value == 0 => "Zero",
    $value > 0 => "Positive"
};
```

## 使用场景

- **状态码处理**：处理 HTTP 状态码
- **类型判断**：根据类型执行不同逻辑
- **值映射**：将值映射到其他值

## 注意事项

- **严格比较**：使用 `===` 进行比较
- **必须匹配**：如果没有 `default` 且没有匹配，会抛出异常
- **表达式模式**：可以使用表达式作为模式

## 常见问题

### 问题 1：未匹配值异常
- **原因**：没有 `default` 且没有匹配的值
- **解决**：添加 `default` 分支

### 问题 2：与 switch 混淆
- **原因**：不理解 `match` 和 `switch` 的区别
- **解决**：理解严格比较和返回值的特性

## 最佳实践

- 使用 `match` 替代简单的 `switch` 语句
- 利用严格比较提高代码安全性
- 使用返回值简化代码
- 理解表达式模式的使用
