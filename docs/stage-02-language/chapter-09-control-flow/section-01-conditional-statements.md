# 2.9.1 条件语句

## 概述

条件语句用于根据条件执行不同的代码。本节介绍 `if/elseif/else`、`switch`、`match` 表达式的语法、用法、比较方式、使用场景及完整示例。

**章节类型**：语法性章节

**主要内容**：
- `if/elseif/else` 的语法和用法
- `switch` 的语法和用法
- `match` 表达式的语法和用法
- 三种方式的比较方式差异
- 使用场景和选择建议
- 最佳实践

## 特性

- **if/elseif/else**：灵活的条件判断
- **switch**：多个固定值匹配
- **match**：严格比较、返回值、无贯穿

## 语法/定义

### if/elseif/else

**语法**：`if (condition) { ... } elseif (condition) { ... } else { ... }`
**特点**：按顺序检查条件，灵活

### switch

**语法**：`switch ($value) { case value1: ... break; default: ... }`
**特点**：宽松比较，需要 `break`

### match

**语法**：`match ($value) { pattern1 => result1, default => default_result }`
**特点**：严格比较，返回值，无贯穿

## 基本用法

### if/elseif/else 示例

```php
<?php
declare(strict_types=1);

if ($age >= 18) {
    echo "Adult\n";
} elseif ($age >= 13) {
    echo "Teenager\n";
} else {
    echo "Child\n";
}
```

### switch 示例

```php
<?php
declare(strict_types=1);

switch ($status) {
    case 'active':
        echo "Active\n";
        break;
    case 'inactive':
        echo "Inactive\n";
        break;
    default:
        echo "Unknown\n";
}
```

### match 示例

```php
<?php
declare(strict_types=1);

$result = match($status) {
    'active' => "Active",
    'inactive' => "Inactive",
    default => "Unknown"
};
```

## 使用场景

- **if/elseif/else**：复杂条件、简单判断
- **switch**：多个固定值匹配
- **match**：多个固定值匹配、需要返回值

## 注意事项

- **比较方式**：`if` 和 `match` 使用严格比较，`switch` 使用宽松比较
- **break**：`switch` 需要 `break`，`match` 不需要
- **返回值**：`match` 可以返回值

## 常见问题

### 问题 1：switch 缺少 break
- **原因**：忘记添加 `break`
- **解决**：每个 `case` 后添加 `break`，或使用 `match`

### 问题 2：switch 宽松比较陷阱
- **原因**：`switch` 使用宽松比较
- **解决**：使用 `match` 或严格比较

## 最佳实践

- 简单条件使用 `if`
- 多个固定值使用 `match`
- 避免使用 `switch`（推荐使用 `match`）
- 理解不同方式的比较规则
