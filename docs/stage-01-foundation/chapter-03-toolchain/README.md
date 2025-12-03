# 1.3 核心工具链

## 目标

- 掌握 Composer 依赖管理和自动加载。
- 配置 IDE 和代码格式化工具。
- 使用 Git 进行版本控制和任务管理。
- 使用 Docker Compose 管理开发环境。
- 创建自动化脚本提高开发效率。

## 章节内容

本章分为五个独立小节，每节提供详细的概念解释、语法说明、参数列表和完整示例：

1. **[Composer 依赖管理](section-01-composer.md)**：Composer 基础、与 npm 对比、常用命令、版本约束、自动加载、配置建议及完整示例。

2. **[IDE 与扩展配置](section-02-ide-extensions.md)**：IDE 选择（PhpStorm、VS Code）、扩展推荐、EditorConfig、代码格式化、静态分析工具及完整示例。

3. **[Git 与任务管理](section-03-git-tasks.md)**：Git 基础、分支管理、Conventional Commits、GitHub CLI、pre-commit 钩子、任务管理工具（Makefile、justfile、Taskfile）及完整示例。

4. **[容器与环境管理](section-04-docker-compose.md)**：Docker Compose 基础、常用命令、完整配置示例、环境变量、数据卷管理、网络管理及完整示例。

5. **[自动化脚本](section-05-automation.md)**：Composer Scripts、Makefile、justfile、Taskfile、CI/CD 集成及完整示例。

## 工具链概览

现代 PHP 开发工具链包括：

- **依赖管理**：Composer
- **代码编辑**：PhpStorm / VS Code
- **版本控制**：Git
- **容器化**：Docker Compose
- **自动化**：Makefile / justfile / Taskfile

## 学习建议

1. **按顺序学习**：按顺序学习五个小节，理解每个工具的作用和配置方法。

2. **重点掌握**：
   - Composer 的基本使用和自动加载
   - IDE 的配置和扩展安装
   - Git 工作流和分支管理
   - Docker Compose 的配置和使用
   - 自动化脚本的创建

3. **实践练习**：
   - 完成每小节后的练习题目
   - 配置一个完整的开发环境
   - 创建项目的自动化脚本

## 完成本章后

- 能够使用 Composer 管理项目依赖。
- 能够配置 IDE 和代码格式化工具。
- 能够使用 Git 进行版本控制和协作。
- 能够使用 Docker Compose 管理开发环境。
- 能够创建自动化脚本提高开发效率。

## 相关章节

- **1.1 PHP 安装与运行基础**：了解 PHP 运行环境。
- **1.4 配置、扩展与调试**：了解 Xdebug 调试配置。
- **2.13 文件引入与模块化**：深入学习 Composer autoload。
