# 2.1.1 第一个 PHP 程序：Hello World

## 概述

创建并执行第一个 PHP 程序是学习 PHP 的第一步。本节介绍如何创建 PHP 文件、使用 PHP 起始标签、执行 PHP 程序（CLI 和 Web 两种方式），帮助零基础学员完成第一个 PHP 程序。

**章节类型**：实践性章节

**主要内容**：
- 创建 PHP 文件
- PHP 起始标签的使用
- CLI 方式执行 PHP 程序
- Web 方式执行 PHP 程序
- 带严格类型的 Hello World 示例
- 常见问题和解决方案

## 实践步骤

### 创建文件
- 创建 `hello.php` 文件
- 文件命名规范
- 文件扩展名要求

### 编写代码
- PHP 起始标签 `<?php`
- 使用 `echo` 输出
- 换行符的使用

### 执行程序
- CLI 方式：`php hello.php`
- Web 方式：通过浏览器访问
- 两种方式的区别

### 完整示例
- 带严格类型的 Hello World
- 函数定义和使用
- 类型声明

## 完整代码示例

### 基础示例
```php
<?php
echo "Hello, World!\n";
```

### 带严格类型示例
```php
<?php
declare(strict_types=1);

function greet(string $name): string
{
    return "Hello, {$name}!\n";
}

echo greet("World");
```

## 输出结果说明

**CLI 方式输出**：
```
Hello, World!
```

**Web 方式输出**：
浏览器显示：`Hello, World!`

## 注意事项

- 文件必须使用 UTF-8 编码（无 BOM）
- PHP 起始标签必须使用 `<?php`
- CLI 和 Web 方式的执行环境不同
- 严格类型声明的作用

## 常见问题

### 问题 1：找不到 php 命令
- **原因**：PHP 未添加到系统 PATH
- **解决**：检查环境变量配置

### 问题 2：浏览器显示源代码
- **原因**：Web 服务器未配置 PHP
- **解决**：检查 Web 服务器配置

### 问题 3：编码问题
- **原因**：文件编码不正确
- **解决**：使用 UTF-8 无 BOM 编码

## 练习任务

1. 创建并执行第一个 PHP 程序
2. 尝试使用不同的输出方式
3. 添加函数定义和类型声明
4. 在 CLI 和 Web 两种方式下执行程序
