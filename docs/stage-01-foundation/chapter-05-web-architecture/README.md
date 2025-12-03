# 1.5 PHP 执行模式与 Web 架构

## 目标

- 理解 PHP 的不同执行模式（CLI、FPM、内置服务器）。
- 深入理解 Nginx + PHP-FPM 的工作原理。
- 掌握 FastCGI 协议和 FPM Pool 配置。
- 理解 Shared Nothing 架构的设计理念。
- 掌握配置优化和性能调优方法。

## 章节内容

本章分为四个独立小节，每节提供详细的概念解释、语法说明、参数列表和完整示例：

1. **[PHP 执行模式](section-01-execution-modes.md)**：执行模式对比、CLI 模式、内置 Web 服务器、PHP-FPM 模式、常驻运行时、选择建议及完整示例。

2. **[Nginx + PHP-FPM 架构](section-02-nginx-fpm.md)**：请求处理流程、FastCGI 协议、Nginx 配置、PHP-FPM 配置、进程管理、监控调试、最佳实践及完整示例。

3. **[Shared Nothing 架构](section-03-shared-nothing.md)**：核心概念、请求生命周期、状态管理、高并发处理、优势、注意事项及完整示例。

4. **[配置与优化实践](section-04-optimization.md)**：Nginx 优化、PHP-FPM 优化、OPcache 优化、性能监控、慢日志分析、完整配置示例及完整示例。

## 执行模式概览

- **CLI**：命令行执行
- **内置服务器**：开发测试
- **PHP-FPM**：生产环境标准
- **常驻运行时**：高性能场景

## 学习建议

1. **按顺序学习**：按顺序学习四个小节，理解 PHP Web 架构的完整体系。

2. **重点掌握**：
   - 不同执行模式的特点和适用场景
   - Nginx + PHP-FPM 的工作原理
   - Shared Nothing 架构的设计理念
   - 配置优化和性能调优方法

3. **实践练习**：
   - 完成每小节后的练习题目
   - 配置完整的 Nginx + PHP-FPM 环境
   - 实现无状态的 Web 应用
   - 进行性能优化和测试

## 完成本章后

- 能够理解 PHP 的不同执行模式并选择合适的模式。
- 能够配置 Nginx + PHP-FPM 架构。
- 理解 Shared Nothing 架构并编写无状态应用。
- 能够进行性能优化和监控。

## 相关章节

- **1.1 PHP 安装与运行基础**：了解 PHP 运行环境。
- **1.6 现代 PHP 运行时与新生态**：了解常驻运行时。
- **1.7 Worker、协程、Event Loop**：了解 Worker 模式。
