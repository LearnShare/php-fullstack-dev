# 2.13 文件引入与模块化

## 目标

- 理解 `include`、`require` 及其 `_once` 变体的差异与适用场景。
- 掌握基于 Composer autoload 的现代模块化方式。
- 学会组织项目结构、拆分配置、模板、业务逻辑文件。

## 章节内容

本章分为两个独立小节，每节提供详细的概念解释、语法说明、参数列表和完整示例：

1. **[include 与 require](section-01-include-require.md)**：`include`、`require`、`include_once`、`require_once` 的区别、使用场景、路径处理、返回值及完整示例。

2. **[Composer Autoload](section-02-composer-autoload.md)**：Composer 基础、PSR-4 自动加载、类映射、文件自动加载、优化选项及完整示例。

## include / require

| 关键字      | 失败时行为         |
| :---------- | :----------------- |
| `include`   | 发出警告 (`E_WARNING`)，脚本继续执行 |
| `require`   | 抛出致命错误 (`E_ERROR`)，脚本终止 |
| `include_once` | 仅在第一次包含文件时执行，避免重复定义 |
| `require_once` | 同上，但致命错误行为不变 |

> **建议**：对关键依赖使用 `require_once`，模板或可选模块用 `include`。

## 学习建议

1. **按顺序学习**：先理解 `include`/`require`，再学习 Composer autoload。

2. **重点掌握**：
   - `include` 和 `require` 的区别
   - 路径处理的最佳实践
   - PSR-4 自动加载的配置和使用
   - Composer autoload 的优化

3. **实践练习**：
   - 完成每小节后的练习题目
   - 配置一个使用 Composer autoload 的项目
   - 组织代码结构

## 完成本章后

- 能够正确使用 `include` 和 `require`。
- 理解路径处理的最佳实践。
- 掌握 Composer autoload 的配置和使用。
- 能够组织模块化的项目结构。

## 相关章节

- **2.10 函数与作用域**：了解函数的作用域。
- **阶段三：面向对象、架构与设计模式**：深入学习命名空间和自动加载。
