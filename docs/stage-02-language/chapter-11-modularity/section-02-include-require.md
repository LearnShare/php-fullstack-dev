# 2.11.2 include 与 require

## 概述

`include` 和 `require` 是导入其他文件的基本方法。本节介绍 `include`、`require`、`include_once`、`require_once` 的区别、工作原理、导入位置的影响、执行时机、作用域、错误处理和性能优化。

**章节类型**：语法性章节

**主要内容**：
- `include` 和 `require` 的区别
- `include_once` 和 `require_once` 的作用
- 导入位置的影响
- 执行时机和作用域
- 错误处理
- 性能优化
- 使用场景和最佳实践

## 特性

- **include**：失败时发出警告，脚本继续执行
- **require**：失败时抛出致命错误，脚本终止
- **once 版本**：避免重复导入

## 语法/定义

### include

**语法**：`include 'file.php';`
**行为**：失败时发出警告，脚本继续执行
**返回值**：成功返回 1，失败返回 `false`

### require

**语法**：`require 'file.php';`
**行为**：失败时抛出致命错误，脚本终止
**返回值**：成功返回 1，失败不返回（脚本终止）

### include_once / require_once

**语法**：`include_once 'file.php';` 或 `require_once 'file.php';`
**行为**：仅在第一次导入时执行，避免重复定义

## 基本用法

### include 示例

```php
<?php
declare(strict_types=1);

// 导入配置文件
include 'config.php';

// 导入函数文件
include 'functions.php';
```

### require 示例

```php
<?php
declare(strict_types=1);

// 导入必需的类文件
require 'User.php';
require 'Database.php';
```

### include_once 示例

```php
<?php
declare(strict_types=1);

// 避免重复导入
include_once 'config.php';
include_once 'config.php';  // 不会重复执行
```

## 使用场景

- **include**：可选模块、模板文件
- **require**：必需的依赖、类文件
- **once 版本**：避免重复定义

## 注意事项

- **路径处理**：注意相对路径和绝对路径
- **错误处理**：检查返回值
- **性能**：`once` 版本有性能开销

## 常见问题

### 问题 1：路径错误
- **原因**：文件路径不正确
- **解决**：使用 `__DIR__` 或绝对路径

### 问题 2：重复导入
- **原因**：多次导入同一文件
- **解决**：使用 `include_once` 或 `require_once`

## 最佳实践

- 关键依赖使用 `require_once`
- 可选模块使用 `include`
- 使用 `__DIR__` 处理路径
- 检查返回值
