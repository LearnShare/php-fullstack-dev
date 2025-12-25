# 阶段一：基础入门（Foundation）

本阶段帮助零基础学员搭建现代 PHP + MySQL 开发所需的基础设施，确保在 Windows、macOS、Linux 等环境都能快速获得一致的运行体验。每个章节都提供操作步骤、验证方法与常见陷阱说明。

## 定位

环境搭建、工具配置、基础概念

## 前置知识要求

在开始本阶段学习之前，请确保你已经：

### 必须掌握（根据 README 说明）

- **网页开发基础**：HTML、CSS、JavaScript/TypeScript 基础
- **计算机基础**：文件管理、命令行使用、操作系统基本操作

### 不需要掌握

以下知识**不需要**提前掌握，会在学习过程中逐步学习：

- PHP 语言（从零开始学习）
- MySQL 数据库（从零开始学习）
- 后端开发经验（本指南会教授）

### 如何检查

完成以下检查点，确认可以开始本阶段：

1. **基础检查**
   - [ ] 能够编写基本的 HTML 页面
   - [ ] 了解基本的 CSS 样式设置
   - [ ] 了解 JavaScript 的基本概念（变量、函数、条件语句）
   - [ ] 能够使用命令行（Windows CMD/PowerShell、macOS Terminal、Linux Shell）

2. **学习准备**
   - [ ] 准备一台电脑（Windows/macOS/Linux 均可）
   - [ ] 准备稳定的网络连接
   - [ ] 准备学习时间（每天至少 1-2 小时）

**如果以上检查点都通过，可以开始阶段一的学习。**

**如果某些检查点未通过，建议：**
- 先学习 HTML/CSS/JS 基础
- 学习基本的命令行操作
- 参考[全局学习指南](../old/docs/learning-guide.md)了解更多信息

## 学习时间估算

**建议学习时间：** 2-3 周

- 每天学习 2-3 小时
- 每周学习 5-6 天
- 包含环境搭建和验证时间

**时间分配建议：**
- PHP 发展史（1.1）：1 天
- PHP 和 MySQL 安装（1.2-1.3）：3-5 天
- 工具链配置（1.4）：3-5 天
- 配置和调试（1.5）：2-3 天
- 环境验证和练习：2-3 天

## 练习建议

### 基础练习（每章完成后）

1. **环境验证**
   - 编写 Hello World 程序
   - 验证 PHP 和 MySQL 安装
   - 测试 Composer 使用

2. **工具使用**
   - 使用 Git 管理代码
   - 使用 IDE 编写代码
   - 配置调试环境

### 综合练习（阶段完成后）

3. **环境搭建**
   - 在另一台电脑上复现环境
   - 编写环境搭建文档
   - 解决环境问题

## 章节内容

1. **[1.1 PHP 发展史](chapter-01-history/readme.md)**：PHP 的起源、发展历程、版本演进和功能特性发展。
   - [1.1.1 PHP 起源与发展历程](chapter-01-history/section-01-history.md)
   - [1.1.2 PHP 生态系统](chapter-01-history/section-02-ecosystem.md)

2. **[1.2 PHP 安装与运行基础](chapter-02-runtime/readme.md)**：逐步完成 PHP 的安装、版本切换与运行模式验证。
   - [1.2.1 PHP 安装与版本管理](chapter-02-runtime/section-01-installation.md)
   - [1.2.2 运行模式与验证](chapter-02-runtime/section-02-execution-modes.md)

3. **[1.3 MySQL 环境搭建与工具](chapter-03-mysql/readme.md)**：通过 Docker 与本地安装方案搭建 MySQL 8，并熟悉 GUI/CLI 工具。
   - [1.3.1 MySQL 安装与配置](chapter-03-mysql/section-01-installation.md)
   - [1.3.2 客户端工具与命令](chapter-03-mysql/section-02-tools-commands.md)

4. **[1.4 核心工具链](chapter-04-toolchain/readme.md)**：配置 Composer、IDE、Git、容器与自动化脚本，形成统一工具链。
   - [1.4.1 Composer 基础](chapter-04-toolchain/section-01-composer-basics.md)
   - [1.4.2 IDE 与扩展配置](chapter-04-toolchain/section-02-ide-extensions.md)
   - [1.4.3 Git 与任务管理](chapter-04-toolchain/section-03-git-tasks.md)
   - [1.4.4 容器与环境管理](chapter-04-toolchain/section-04-docker-compose.md)
   - [1.4.5 自动化脚本](chapter-04-toolchain/section-05-automation.md)

5. **[1.5 配置、扩展与调试基础](chapter-05-config-debug/readme.md)**：掌握 php.ini、扩展安装、调试基础与配置差异排查。
   - [1.5.1 php.ini 配置](chapter-05-config-debug/section-01-php-ini.md)
   - [1.5.2 扩展管理](chapter-05-config-debug/section-02-extensions.md)
   - [1.5.3 调试基础介绍](chapter-05-config-debug/section-03-debugging-basics.md)

完成本阶段后，你将具备：

- 能够在 CLI、FPM、Docker 之间切换 PHP 版本，并保留安装脚本以便复现环境。
- 独立部署 MySQL 8，配置安全访问策略，熟悉数据备份与恢复流程。
- 使用 Composer、IDE、Git Hook、容器脚本构建可复制的开发基线。
- 明确 php.ini、扩展、调试工具的配置路径，遇到问题能主动排查并记录结果。

## 相关章节

- **阶段二：PHP 语言特性**：学习 PHP 语言基础
- **阶段五：Web/API 开发**：学习 Web 开发基础（包含从本阶段移出的 Web 架构内容）
- **阶段七：性能优化与安全**：学习现代 PHP 运行时（包含从本阶段移出的运行时内容）
