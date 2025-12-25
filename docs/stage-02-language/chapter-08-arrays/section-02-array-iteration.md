# 2.8.2 数组遍历

## 概述

数组遍历是处理数组数据的基本操作。本节介绍 `foreach` 循环、引用遍历、嵌套数组遍历、数组指针操作（`reset()`、`next()`、`prev()`、`end()`、`current()`、`key()`）及注意事项。

**章节类型**：语法性章节

**主要内容**：
- `foreach` 循环的语法和用法
- 引用遍历的使用
- 嵌套数组的遍历
- 数组指针操作函数
- 遍历性能考虑
- 遍历最佳实践

## 特性

- **foreach**：最高效的数组遍历方式
- **引用遍历**：可以修改数组元素
- **指针操作**：可以手动控制遍历位置

## 语法/定义

### foreach 循环

**值遍历**：`foreach ($array as $value)`
**键值遍历**：`foreach ($array as $key => $value)`
**引用遍历**：`foreach ($array as &$value)`

### 数组指针操作

**reset()**：重置指针到第一个元素
**next()**：移动指针到下一个元素
**prev()**：移动指针到上一个元素
**end()**：移动指针到最后一个元素
**current()**：获取当前元素值
**key()**：获取当前元素键

## 基本用法

### foreach 值遍历示例

```php
<?php
declare(strict_types=1);

$arr = [1, 2, 3];
foreach ($arr as $value) {
    echo $value . "\n";
}
```

### foreach 键值遍历示例

```php
<?php
declare(strict_types=1);

$arr = ['a' => 1, 'b' => 2, 'c' => 3];
foreach ($arr as $key => $value) {
    echo "{$key}: {$value}\n";
}
```

### 引用遍历示例

```php
<?php
declare(strict_types=1);

$arr = [1, 2, 3];
foreach ($arr as &$value) {
    $value *= 2;  // 修改数组元素
}
unset($value);  // 重要：解除引用
```

### 嵌套数组遍历示例

```php
<?php
declare(strict_types=1);

$users = [
    ['name' => 'John', 'age' => 25],
    ['name' => 'Jane', 'age' => 30]
];

foreach ($users as $user) {
    foreach ($user as $key => $value) {
        echo "{$key}: {$value}\n";
    }
}
```

## 使用场景

- **数据处理**：遍历数组处理数据
- **数据转换**：将数组转换为其他格式
- **数据筛选**：筛选符合条件的元素

## 注意事项

- **引用遍历**：遍历后必须 `unset()` 引用变量
- **性能**：`foreach` 性能最好
- **指针操作**：指针操作可能影响 `foreach` 的行为

## 常见问题

### 问题 1：引用遍历后未 unset
- **原因**：引用变量在循环后仍然存在
- **解决**：循环后使用 `unset()` 解除引用

### 问题 2：修改数组结构
- **原因**：遍历时修改数组结构可能导致问题
- **解决**：遍历数组副本或使用其他方法

## 最佳实践

- 使用 `foreach` 遍历数组
- 引用遍历后记得 `unset()`
- 避免在遍历时修改数组结构
- 理解数组指针操作的影响
