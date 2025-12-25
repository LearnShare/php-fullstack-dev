# 2.9.2 循环结构

## 概述

循环结构用于重复执行代码。本节介绍 `while`、`do-while`、`for`、`foreach` 循环的语法、用法、适用场景、嵌套循环及完整示例。

**章节类型**：语法性章节

**主要内容**：
- `while` 循环的语法和用法
- `do-while` 循环的语法和用法
- `for` 循环的语法和用法
- `foreach` 循环的语法和用法
- 各种循环的适用场景
- 嵌套循环的使用
- 循环控制（`break`、`continue`）

## 特性

- **while**：条件未知的循环
- **do-while**：至少执行一次的循环
- **for**：已知次数的循环
- **foreach**：遍历数组或可迭代对象

## 语法/定义

### while 循环

**语法**：`while (condition) { ... }`
**特点**：先检查条件，可能不执行

### do-while 循环

**语法**：`do { ... } while (condition);`
**特点**：先执行后检查，至少执行一次

### for 循环

**语法**：`for (init; condition; increment) { ... }`
**特点**：适合计数循环

### foreach 循环

**语法**：`foreach ($array as $value)` 或 `foreach ($array as $key => $value)`
**特点**：最高效的数组遍历方式

## 基本用法

### while 示例

```php
<?php
declare(strict_types=1);

$i = 0;
while ($i < 5) {
    echo $i . "\n";
    $i++;
}
```

### do-while 示例

```php
<?php
declare(strict_types=1);

$i = 0;
do {
    echo $i . "\n";
    $i++;
} while ($i < 5);
```

### for 示例

```php
<?php
declare(strict_types=1);

for ($i = 0; $i < 5; $i++) {
    echo $i . "\n";
}
```

### foreach 示例

```php
<?php
declare(strict_types=1);

$arr = [1, 2, 3];
foreach ($arr as $value) {
    echo $value . "\n";
}
```

## 使用场景

- **while**：条件未知的循环
- **do-while**：至少执行一次的循环
- **for**：已知次数的循环
- **foreach**：遍历数组

## 注意事项

- **无限循环**：注意避免无限循环
- **性能**：`foreach` 性能最好
- **嵌套循环**：注意嵌套循环的性能

## 常见问题

### 问题 1：无限循环
- **原因**：循环条件永远为真
- **解决**：确保循环条件会改变

### 问题 2：循环变量作用域
- **原因**：不理解循环变量的作用域
- **解决**：理解变量的作用域规则

## 最佳实践

- 遍历数组使用 `foreach`
- 计数循环使用 `for`
- 条件未知使用 `while`
- 避免无限循环
