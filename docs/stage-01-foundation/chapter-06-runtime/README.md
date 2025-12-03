# 1.6 现代 PHP 运行时与新生态

## 目标

- 了解现代 PHP 运行时的优势和使用场景。
- 掌握 RoadRunner、FrankenPHP、OpenSwoole 的特点和配置。
- 理解 Laravel Octane 的作用和配置。
- 能够选择合适的运行时方案。

## 章节内容

本章分为五个独立小节，每节提供详细的概念解释、语法说明、参数列表和完整示例：

1. **[RoadRunner](section-01-roadrunner.md)**：RoadRunner 特点、安装方法、配置参数、Worker 文件、启动停止、性能优化及完整示例。

2. **[FrankenPHP](section-02-frankenphp.md)**：FrankenPHP 特点、安装方法、配置（Caddyfile、PHP 配置）、Worker 模式、自动 HTTPS、HTTP/2/3 支持及完整示例。

3. **[OpenSwoole](section-03-openswoole.md)**：OpenSwoole 特点、安装方法、HTTP 服务器、WebSocket 服务器、协程、事件循环及完整示例。

4. **[Laravel Octane](section-04-laravel-octane.md)**：Octane 特点、安装方法、配置、启动方式、状态管理、连接复用、性能对比及完整示例。

5. **[运行时对比与选择](section-05-comparison.md)**：功能对比、性能对比、选择建议、决策流程、迁移建议、性能测试及完整示例。

## 运行时概览

- **PHP-FPM**：传统标准，稳定可靠
- **RoadRunner**：Go 语言编写，高性能
- **FrankenPHP**：自动 HTTPS，开箱即用
- **OpenSwoole**：C 扩展，极高性能

## 学习建议

1. **按顺序学习**：按顺序学习五个小节，理解各种运行时的特点和适用场景。

2. **重点掌握**：
   - 各种运行时的特点和优势
   - 安装和配置方法
   - 状态管理和连接复用
   - 如何选择合适的运行时

3. **实践练习**：
   - 完成每小节后的练习题目
   - 安装和配置不同的运行时
   - 进行性能对比测试

## 完成本章后

- 能够理解各种现代 PHP 运行时的特点和优势。
- 能够安装和配置 RoadRunner、FrankenPHP、OpenSwoole。
- 能够使用 Laravel Octane 提升 Laravel 应用性能。
- 能够根据项目需求选择合适的运行时方案。

## 相关章节

- **1.5 PHP 执行模式与 Web 架构**：了解传统 PHP-FPM 架构。
- **1.7 Worker、协程、Event Loop**：深入学习 Worker 模式和协程。
