# 2.9.3 跳转语句

## 概述

跳转语句用于改变程序的执行流程。本节介绍 `break`、`continue`、`goto` 的语法、用法、跳出多层循环、使用限制及注意事项。

**章节类型**：语法性章节

**主要内容**：
- `break` 的语法和用法
- `continue` 的语法和用法
- `goto` 的语法和用法
- 跳出多层循环
- 使用限制和注意事项
- 替代方案

## 特性

- **break**：跳出循环或 switch
- **continue**：跳过当前循环迭代
- **goto**：跳转到标签（不推荐使用）

## 语法/定义

### break

**语法**：`break;` 或 `break $levels;`
**功能**：跳出循环或 switch
**参数**：`$levels` 指定跳出层数

### continue

**语法**：`continue;` 或 `continue $levels;`
**功能**：跳过当前循环迭代
**参数**：`$levels` 指定跳过层数

### goto

**语法**：`goto label; ... label:`
**功能**：跳转到标签
**限制**：不能跳入函数、类、循环

## 基本用法

### break 示例

```php
<?php
declare(strict_types=1);

for ($i = 0; $i < 10; $i++) {
    if ($i === 5) {
        break;  // 跳出循环
    }
    echo $i . "\n";
}
```

### continue 示例

```php
<?php
declare(strict_types=1);

for ($i = 0; $i < 10; $i++) {
    if ($i % 2 === 0) {
        continue;  // 跳过偶数
    }
    echo $i . "\n";
}
```

### 跳出多层循环示例

```php
<?php
declare(strict_types=1);

for ($i = 0; $i < 3; $i++) {
    for ($j = 0; $j < 3; $j++) {
        if ($i === 1 && $j === 1) {
            break 2;  // 跳出两层循环
        }
    }
}
```

## 使用场景

- **break**：满足条件时提前退出循环
- **continue**：跳过某些迭代
- **goto**：很少使用，不推荐

## 注意事项

- **goto 限制**：不能跳入函数、类、循环
- **可读性**：过度使用 `goto` 降低可读性
- **替代方案**：使用函数或重构代码替代 `goto`

## 常见问题

### 问题 1：goto 使用限制
- **原因**：不理解 `goto` 的限制
- **解决**：避免使用 `goto`，使用其他方法

### 问题 2：跳出多层循环
- **原因**：需要跳出嵌套循环
- **解决**：使用 `break $levels` 或重构代码

## 最佳实践

- 使用 `break` 和 `continue` 控制循环流程
- 避免使用 `goto`
- 使用函数或重构代码替代 `goto`
- 理解跳转语句的作用范围
