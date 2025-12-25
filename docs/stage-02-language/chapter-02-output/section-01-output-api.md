# 2.2.1 输出 API

## 概述

PHP 提供了多种输出函数，每种函数有不同的特点和适用场景。本节介绍 `echo`、`print`、`printf`、`sprintf` 及输出缓冲机制，帮助选择合适的输出方式。

**章节类型**：语法性章节

**主要内容**：
- `echo` 的语法和用法
- `print` 的语法和用法
- `printf` 的语法和用法
- `sprintf` 的语法和用法
- 输出缓冲机制
- 函数对比和选择建议

## 特性

- **echo**：语言结构，可以输出多个值，无返回值
- **print**：语言结构，只能输出一个值，返回 1
- **printf**：函数，格式化输出，返回输出字符数
- **sprintf**：函数，格式化字符串，返回格式化后的字符串

## 语法/定义

### echo

**语法**：`echo expression1, expression2, ...`
**特点**：可以输出多个值，无返回值
**用法**：`echo "Hello", " ", "World";`

### print

**语法**：`print expression`
**特点**：只能输出一个值，返回 1
**用法**：`print "Hello, World";`

### printf

**语法**：`printf(string $format, mixed ...$values): int`
**特点**：格式化输出，返回输出字符数
**用法**：`printf("Hello, %s", "World");`

### sprintf

**语法**：`sprintf(string $format, mixed ...$values): string`
**特点**：格式化字符串，返回格式化后的字符串
**用法**：`$result = sprintf("Hello, %s", "World");`

## 基本用法

### echo 示例

```php
<?php
declare(strict_types=1);

echo "Hello, World!\n";
echo "Hello", " ", "World", "\n";
```

### print 示例

```php
<?php
declare(strict_types=1);

print "Hello, World!\n";
```

### printf 示例

```php
<?php
declare(strict_types=1);

printf("Hello, %s!\n", "World");
printf("Number: %d\n", 42);
```

### sprintf 示例

```php
<?php
declare(strict_types=1);

$message = sprintf("Hello, %s!", "World");
echo $message;
```

## 使用场景

- **echo**：简单输出，适合大多数场景
- **print**：需要返回值的场景（较少使用）
- **printf**：格式化输出到标准输出
- **sprintf**：格式化字符串用于后续处理

## 注意事项

- **性能**：`echo` 性能最好，`print` 次之，`printf`/`sprintf` 较慢
- **返回值**：`echo` 无返回值，`print` 返回 1，`printf` 返回字符数，`sprintf` 返回字符串
- **格式化**：`printf` 和 `sprintf` 支持格式化占位符

## 常见问题

### 问题 1：echo 和 print 的区别
- **区别**：`echo` 可以输出多个值，`print` 只能输出一个值
- **选择**：大多数情况下使用 `echo`

### 问题 2：何时使用 sprintf
- **场景**：需要格式化字符串用于后续处理
- **示例**：生成日志格式、订单号等

## 最佳实践

- 大多数情况下使用 `echo`
- 需要格式化时使用 `sprintf`
- 避免使用 `print`（性能较差）
- 理解输出缓冲机制
