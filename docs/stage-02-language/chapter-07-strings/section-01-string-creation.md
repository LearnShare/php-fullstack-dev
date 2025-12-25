# 2.7.1 字符串创建方式

## 概述

PHP 提供了四种方式创建字符串：单引号、双引号、Heredoc、Nowdoc。每种方式有不同的特点和适用场景。本节详细介绍这四种方式的语法、特点、变量插值、转义序列及选择建议。

**章节类型**：语法性章节

**主要内容**：
- 单引号字符串的语法和特点
- 双引号字符串的语法和特点
- Heredoc 的语法和特点
- Nowdoc 的语法和特点
- 变量插值的规则
- 转义序列的使用
- 选择建议和最佳实践

## 特性

- **单引号**：不解析转义（除 `\'`、`\\`），不解析变量
- **双引号**：解析转义，解析变量
- **Heredoc**：类似双引号，适合多行字符串
- **Nowdoc**：类似单引号，适合多行字符串

## 语法/定义

### 单引号

**语法**：`'string'`
**特点**：不解析转义（除 `\'`、`\\`），不解析变量
**性能**：性能最好

### 双引号

**语法**：`"string"`
**特点**：解析转义，解析变量
**性能**：需要解析，性能稍慢

### Heredoc

**语法**：`<<<IDENTIFIER ... IDENTIFIER;`
**特点**：类似双引号，可插值，适合多行
**要求**：结束标识符必须单独一行，不能有缩进

### Nowdoc

**语法**：`<<<'IDENTIFIER' ... IDENTIFIER;`
**特点**：类似单引号，不插值，适合多行
**要求**：结束标识符必须单独一行，不能有缩进

## 基本用法

### 单引号示例

```php
<?php
declare(strict_types=1);

$str = 'Hello, World!';
$str = 'It\'s a string';  // 转义单引号
```

### 双引号示例

```php
<?php
declare(strict_types=1);

$name = "World";
$str = "Hello, {$name}!";  // 变量插值
$str = "Line 1\nLine 2";   // 转义序列
```

### Heredoc 示例

```php
<?php
declare(strict_types=1);

$name = "World";
$str = <<<TEXT
Hello, {$name}!
This is a multi-line string.
TEXT;
```

### Nowdoc 示例

```php
<?php
declare(strict_types=1);

$str = <<<'TEXT'
Hello, World!
This is a multi-line string.
No variable interpolation.
TEXT;
```

## 使用场景

- **单引号**：简单字符串，不需要变量插值
- **双引号**：需要变量插值或转义序列
- **Heredoc**：多行字符串，需要变量插值
- **Nowdoc**：多行字符串，不需要变量插值

## 注意事项

- **性能**：单引号性能最好
- **转义**：理解不同方式的转义规则
- **变量插值**：双引号和 Heredoc 支持变量插值
- **标识符**：Heredoc/Nowdoc 的结束标识符必须单独一行

## 常见问题

### 问题 1：单引号内变量不解析
- **原因**：单引号不解析变量
- **解决**：使用双引号或字符串连接

### 问题 2：Heredoc 结束标识符错误
- **原因**：结束标识符格式不正确
- **解决**：确保结束标识符单独一行，无缩进

## 最佳实践

- 简单字符串使用单引号
- 需要变量插值使用双引号
- 多行字符串使用 Heredoc/Nowdoc
- 理解不同方式的性能和特点
