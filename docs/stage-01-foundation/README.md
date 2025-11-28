
# 阶段一：环境、运行时与工具链总览

本阶段帮助零基础学员搭建现代 PHP + MySQL 开发所需的基础设施，确保在 Windows、macOS、Linux 等环境都能快速获得一致的运行体验。每个章节都提供操作步骤、验证方法与常见陷阱说明：

1. [`chapter-01-runtime`](chapter-01-runtime/README.md)：逐步完成 PHP 的安装、版本切换与运行模式验证。
2. [`chapter-02-mysql`](chapter-02-mysql/README.md)：通过 Docker 与本地安装方案搭建 MySQL 8，并熟悉 GUI/CLI 工具。
3. [`chapter-03-toolchain`](chapter-03-toolchain/README.md)：配置 Composer、IDE、Git、容器与自动化脚本，形成统一工具链。
4. [`chapter-04-config-debug`](chapter-04-config-debug/README.md)：掌握 php.ini、扩展安装、Xdebug 调试与配置差异排查。
5. [`chapter-05-web-architecture`](chapter-05-web-architecture/README.md)：PHP 执行模式与 Web 架构（Nginx + PHP-FPM 工作原理、Shared Nothing 架构）。
6. [`chapter-06-runtime`](chapter-06-runtime/README.md)：现代 PHP 运行时与新生态（RoadRunner、FrankenPHP、OpenSwoole、Laravel Octane）。
7. [`chapter-07-workers`](chapter-07-workers/README.md)：Worker、协程、Event Loop（Worker 模式、协程、PHP 8 Fibers）。

完成本阶段后，你将具备：

- 能够在 CLI、FPM、Docker 之间切换 PHP 版本，并保留安装脚本以便复现环境。
- 独立部署 MySQL 8，配置安全访问策略，熟悉数据备份与恢复流程。
- 使用 Composer、IDE、Git Hook、容器脚本构建可复制的开发基线。
- 明确 php.ini、扩展、Xdebug 的配置路径，遇到问题能主动排查并记录结果。
