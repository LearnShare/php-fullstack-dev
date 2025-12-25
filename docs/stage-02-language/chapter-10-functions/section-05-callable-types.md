# 2.10.5 可调用类型与内置函数

## 概述

可调用类型表示可以调用的值。本节介绍可调用类型的各种形式、`is_callable()`、`call_user_func()`、`call_user_func_array()`、内置函数工具等。

**章节类型**：参考性章节

**主要内容**：
- 可调用类型的各种形式
- `is_callable()` 函数的使用
- `call_user_func()` 函数的使用
- `call_user_func_array()` 函数的使用
- 内置函数工具
- 使用场景和最佳实践

## 特性

- **多种形式**：函数名、方法、闭包、实现了 `__invoke()` 的对象
- **动态调用**：可以动态调用函数
- **类型检查**：可以检查值是否可调用

## 语法/定义

### 可调用类型

**函数名**：`'function_name'`
**方法**：`[$object, 'method']` 或 `[ClassName::class, 'method']`
**闭包**：`function() { ... }`
**可调用对象**：实现了 `__invoke()` 的对象

### 类型声明

**语法**：`function func(callable $callback)`
**特点**：接受任何可调用值

## 基本用法/命令

### is_callable()

**语法**：`is_callable(mixed $value, bool $syntax_only = false, string &$callable_name = null): bool`
**功能**：检查值是否可调用

### call_user_func()

**语法**：`call_user_func(callable $callback, mixed ...$args): mixed`
**功能**：调用回调函数

### call_user_func_array()

**语法**：`call_user_func_array(callable $callback, array $args): mixed`
**功能**：使用数组参数调用回调函数

## 使用场景

- **动态调用**：根据条件动态调用函数
- **回调处理**：处理各种回调函数
- **函数包装**：包装其他函数

## 注意事项

- **性能**：动态调用有性能开销
- **类型检查**：使用 `is_callable()` 检查
- **错误处理**：处理调用失败的情况

## 常见问题

### 问题 1：可调用类型检查
- **原因**：不确定值是否可调用
- **解决**：使用 `is_callable()` 检查

### 问题 2：动态调用错误
- **原因**：调用的函数不存在
- **解决**：检查函数是否存在

## 最佳实践

- 使用 `is_callable()` 检查可调用性
- 理解各种可调用形式
- 注意动态调用的性能
- 处理调用失败的情况
