# 2.8.3 数组操作函数

## 概述

PHP 提供了丰富的数组操作函数。本节介绍查找函数、合并函数、高阶函数（`array_map()`、`array_filter()`、`array_reduce()`）、切片函数、PHP 8.4+ 新函数（`array_first()`、`array_last()`）等常用数组操作函数。

**章节类型**：参考性章节

**主要内容**：
- 查找函数（`array_key_exists()`、`in_array()`、`array_search()`、`array_keys()`、`array_values()`）
- 合并函数（`array_merge()`、`array_merge_recursive()`）
- 高阶函数（`array_map()`、`array_filter()`、`array_reduce()`）
- 切片函数（`array_slice()`）
- PHP 8.4+ 新函数（`array_first()`、`array_last()`）
- 函数式编程风格

## 特性

- **函数丰富**：提供大量数组操作函数
- **函数式编程**：支持函数式编程风格
- **性能优化**：选择合适的函数提高性能

## 基本用法/命令

### 查找函数

**array_key_exists()**：检查键是否存在
**in_array()**：检查值是否存在
**array_search()**：查找值的键
**array_keys()**：获取所有键
**array_values()**：获取所有值

### 合并函数

**array_merge()**：合并数组
**array_merge_recursive()**：递归合并数组

### 高阶函数

**array_map()**：对每个元素应用函数
**array_filter()**：过滤数组元素
**array_reduce()**：归约数组为单个值

### 切片函数

**array_slice()**：提取数组片段

### PHP 8.4+ 新函数

**array_first()**：获取第一个元素
**array_last()**：获取最后一个元素

## 使用场景

- **数据处理**：查找、过滤、转换数组数据
- **函数式编程**：使用高阶函数处理数据
- **数组操作**：合并、切片、提取数组

## 注意事项

- **性能考虑**：选择合适的函数提高性能
- **函数式编程**：理解高阶函数的使用
- **版本要求**：注意新函数的版本要求

## 常见问题

### 问题 1：array_key_exists vs isset
- **区别**：`isset()` 对 `null` 值返回 `false`
- **选择**：根据需求选择合适的函数

### 问题 2：高阶函数性能
- **性能**：高阶函数可能比循环慢
- **建议**：在可读性和性能间平衡

## 最佳实践

- 理解各种查找函数的区别
- 使用高阶函数提高代码可读性
- 注意函数的版本要求
- 在性能和可读性间平衡
