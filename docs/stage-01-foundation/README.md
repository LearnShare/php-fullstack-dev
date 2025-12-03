
# 阶段一：环境、运行时与工具链总览

本阶段帮助零基础学员搭建现代 PHP + MySQL 开发所需的基础设施，确保在 Windows、macOS、Linux 等环境都能快速获得一致的运行体验。每个章节都提供操作步骤、验证方法与常见陷阱说明：

1. **[PHP 安装与运行基础](chapter-01-runtime/README.md)**：逐步完成 PHP 的安装、版本切换与运行模式验证。
   - [1.1.1 PHP 安装与版本管理](chapter-01-runtime/section-01-installation.md)
   - [1.1.2 运行模式与验证](chapter-01-runtime/section-02-execution-modes.md)

2. **[MySQL 环境搭建与工具](chapter-02-mysql/README.md)**：通过 Docker 与本地安装方案搭建 MySQL 8，并熟悉 GUI/CLI 工具。
   - [1.2.1 MySQL 安装与配置](chapter-02-mysql/section-01-installation.md)
   - [1.2.2 客户端工具与命令](chapter-02-mysql/section-02-tools-commands.md)

3. **[核心工具链](chapter-03-toolchain/README.md)**：配置 Composer、IDE、Git、容器与自动化脚本，形成统一工具链。
   - [1.3.1 Composer 依赖管理](chapter-03-toolchain/section-01-composer.md)
   - [1.3.2 IDE 与扩展配置](chapter-03-toolchain/section-02-ide-extensions.md)
   - [1.3.3 Git 与任务管理](chapter-03-toolchain/section-03-git-tasks.md)
   - [1.3.4 容器与环境管理](chapter-03-toolchain/section-04-docker-compose.md)
   - [1.3.5 自动化脚本](chapter-03-toolchain/section-05-automation.md)

4. **[配置、扩展与调试](chapter-04-config-debug/README.md)**：掌握 php.ini、扩展安装、Xdebug 调试与配置差异排查。
   - [1.4.1 php.ini 配置](chapter-04-config-debug/section-01-php-ini.md)
   - [1.4.2 扩展管理](chapter-04-config-debug/section-02-extensions.md)
   - [1.4.3 Xdebug 调试配置](chapter-04-config-debug/section-03-xdebug.md)
   - [1.4.4 调试技巧与实践](chapter-04-config-debug/section-04-debugging-tips.md)

5. **[PHP 执行模式与 Web 架构](chapter-05-web-architecture/README.md)**：PHP 执行模式与 Web 架构（Nginx + PHP-FPM 工作原理、Shared Nothing 架构）。
   - [1.5.1 PHP 执行模式](chapter-05-web-architecture/section-01-execution-modes.md)
   - [1.5.2 Nginx + PHP-FPM 架构](chapter-05-web-architecture/section-02-nginx-fpm.md)
   - [1.5.3 Shared Nothing 架构](chapter-05-web-architecture/section-03-shared-nothing.md)
   - [1.5.4 配置与优化实践](chapter-05-web-architecture/section-04-optimization.md)

6. **[现代 PHP 运行时与新生态](chapter-06-runtime/README.md)**：现代 PHP 运行时与新生态（RoadRunner、FrankenPHP、OpenSwoole、Laravel Octane）。
   - [1.6.1 RoadRunner](chapter-06-runtime/section-01-roadrunner.md)
   - [1.6.2 FrankenPHP](chapter-06-runtime/section-02-frankenphp.md)
   - [1.6.3 OpenSwoole](chapter-06-runtime/section-03-openswoole.md)
   - [1.6.4 Laravel Octane](chapter-06-runtime/section-04-laravel-octane.md)
   - [1.6.5 运行时对比与选择](chapter-06-runtime/section-05-comparison.md)

7. **[Worker、协程、Event Loop](chapter-07-workers/README.md)**：Worker、协程、Event Loop（Worker 模式、协程、PHP 8 Fibers）。
   - [1.7.1 Worker 模式](chapter-07-workers/section-01-worker-mode.md)
   - [1.7.2 协程（Coroutine）](chapter-07-workers/section-02-coroutines.md)
   - [1.7.3 Event Loop（事件循环）](chapter-07-workers/section-03-event-loop.md)
   - [1.7.4 PHP 8 Fibers](chapter-07-workers/section-04-fibers.md)
   - [1.7.5 实际应用与最佳实践](chapter-07-workers/section-05-best-practices.md)

完成本阶段后，你将具备：

- 能够在 CLI、FPM、Docker 之间切换 PHP 版本，并保留安装脚本以便复现环境。
- 独立部署 MySQL 8，配置安全访问策略，熟悉数据备份与恢复流程。
- 使用 Composer、IDE、Git Hook、容器脚本构建可复制的开发基线。
- 明确 php.ini、扩展、Xdebug 的配置路径，遇到问题能主动排查并记录结果。
