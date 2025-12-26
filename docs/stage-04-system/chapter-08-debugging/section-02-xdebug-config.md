# 4.8.2 Xdebug 配置与使用

## 概述

Xdebug 是 PHP 最强大的调试工具之一。本节介绍 Xdebug 的安装、配置和使用，包括 IDE 集成、断点调试、变量查看等功能，帮助零基础学员掌握专业的调试方法。

**章节类型**：工具性章节

**主要内容**：
- Xdebug 概述
- Xdebug 安装
- php.ini 配置
- IDE 集成（PhpStorm、VS Code）
- 断点调试
- 变量查看和监视
- 堆栈跟踪
- 性能分析
- 完整示例

## 核心内容

### Xdebug 概述

- Xdebug 的功能
- 调试协议
- 性能分析功能

### 安装配置

- 扩展安装
- php.ini 配置
- 配置参数说明

### IDE 集成

- PhpStorm 配置
- VS Code 配置
- 调试会话启动
- 远程调试

### 断点调试

- 设置断点
- 条件断点
- 断点管理
- 执行控制（步进、步出、继续）

### 变量查看

- 变量监视
- 表达式求值
- 调用堆栈查看

## 基本用法

### Xdebug 配置示例

```ini
; php.ini
[xdebug]
zend_extension=xdebug
xdebug.mode=debug
xdebug.start_with_request=yes
xdebug.client_host=127.0.0.1
xdebug.client_port=9003
```

## 使用场景

- 复杂问题调试
- 代码逻辑分析
- 性能问题排查
- 学习代码执行流程

## 注意事项

- 性能影响
- 生产环境禁用
- 配置正确性
- IDE 兼容性

## 常见问题

- 如何安装 Xdebug？
- 如何配置 IDE 调试？
- 断点不生效怎么办？
- 如何远程调试？

## 最佳实践

- 开发环境启用 Xdebug
- 生产环境禁用 Xdebug
- 使用条件断点提高效率
- 结合日志进行调试
