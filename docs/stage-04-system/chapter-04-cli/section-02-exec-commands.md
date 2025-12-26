# 4.4.2 执行外部命令

## 概述

执行外部命令是 CLI 编程中的常见需求。本节介绍 PHP 中执行外部命令的多种方法，包括 exec()、shell_exec()、system()、passthru()、proc_open() 等，以及安全注意事项，帮助零基础学员掌握命令执行操作。

**章节类型**：语法性章节

**主要内容**：
- 命令执行函数概述
- exec() 函数
- shell_exec() 函数
- system() 函数
- passthru() 函数
- proc_open() 函数
- 命令执行安全
- 完整示例

## 核心内容

### 命令执行函数

- exec()：执行命令并返回最后一行
- shell_exec()：执行命令并返回完整输出
- system()：执行命令并直接输出
- passthru()：执行命令并直接输出二进制数据
- proc_open()：高级进程控制

### exec() 函数

- 语法和参数
- 返回值处理
- 输出捕获

### proc_open() 函数

- 进程创建
- 标准输入输出处理
- 进程通信
- 进程管理

### 命令执行安全

- 命令注入风险
- 参数转义
- 白名单验证
- 权限控制

## 基本用法

### exec() 示例

```php
<?php
declare(strict_types=1);

// 执行命令
exec('ls -la', $output, $returnCode);
print_r($output);
```

### proc_open() 示例

```php
<?php
declare(strict_types=1);

$descriptors = [
    0 => ['pipe', 'r'], // stdin
    1 => ['pipe', 'w'], // stdout
    2 => ['pipe', 'w']  // stderr
];

$process = proc_open('command', $descriptors, $pipes);
// 处理进程...
proc_close($process);
```

## 使用场景

- 系统命令调用
- 脚本执行
- 进程管理
- 系统集成

## 注意事项

- 命令注入安全风险
- 参数转义
- 错误处理
- 超时控制

## 常见问题

- exec() 和 system() 的区别？
- 如何安全地执行外部命令？
- 如何处理命令输出？
- proc_open() 的优势是什么？

## 最佳实践

- 使用 escapeshellarg() 转义参数
- 验证命令白名单
- 使用 proc_open() 进行高级控制
- 设置执行超时
