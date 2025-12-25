# 1.4.3 Git 与任务管理

## 概述

Git 是分布式版本控制系统，是现代软件开发的基础工具。掌握 Git 的使用和任务管理工具的使用可以提高开发效率和团队协作能力。

**章节类型**：工具性章节

**主要内容**：
- Git 基础概念和安装
- 基本 Git 命令（init、add、commit、push、pull）
- 分支管理（创建、切换、合并、删除）
- Conventional Commits 规范
- GitHub CLI 使用
- pre-commit 钩子配置
- 任务管理工具（Makefile、justfile、Taskfile）

## 特性

- **分布式版本控制**：每个开发者都有完整的代码历史
- **分支管理**：强大的分支管理能力
- **协作友好**：支持多人协作开发
- **任务自动化**：通过任务管理工具自动化常见任务

## 基本用法/命令

### Git 基础

**初始化仓库**：
- `git init`：初始化本地仓库
- `git clone`：克隆远程仓库

**基本操作**：
- `git add`：添加文件到暂存区
- `git commit`：提交更改
- `git push`：推送到远程仓库
- `git pull`：从远程仓库拉取更新
- `git status`：查看仓库状态
- `git log`：查看提交历史

### 分支管理

**创建和切换分支**：
- `git branch`：查看分支列表
- `git branch branch_name`：创建分支
- `git checkout branch_name`：切换分支
- `git checkout -b branch_name`：创建并切换分支

**合并分支**：
- `git merge branch_name`：合并分支
- `git rebase branch_name`：变基合并

**删除分支**：
- `git branch -d branch_name`：删除本地分支
- `git push origin --delete branch_name`：删除远程分支

### Conventional Commits

**提交格式**：
- `feat:`：新功能
- `fix:`：修复 bug
- `docs:`：文档更新
- `style:`：代码格式调整
- `refactor:`：代码重构
- `test:`：测试相关
- `chore:`：构建过程或辅助工具的变动

**示例**：
- `feat: 添加用户登录功能`
- `fix: 修复登录验证问题`

### GitHub CLI

**安装和配置**：
- 安装方法
- 认证配置

**常用命令**：
- `gh repo clone`：克隆仓库
- `gh issue create`：创建 issue
- `gh pr create`：创建 Pull Request

### pre-commit 钩子

**配置方法**：
- 创建 `.git/hooks/pre-commit` 文件
- 配置检查脚本（代码格式、静态分析等）

**示例脚本**：
- PHP CS Fixer 检查
- PHPStan 检查

### 任务管理工具

**Makefile**：
- 基本语法
- 常用任务示例（install、test、clean）

**justfile**：
- 基本语法
- 常用任务示例

**Taskfile**：
- 基本语法
- 常用任务示例

## 使用场景

- **版本控制**：跟踪代码变更历史
- **分支开发**：使用分支进行功能开发和 bug 修复
- **团队协作**：通过 Pull Request 进行代码审查
- **任务自动化**：使用任务管理工具自动化常见操作

## 注意事项

- **提交信息**：使用清晰的提交信息，遵循 Conventional Commits
- **分支策略**：制定清晰的分支管理策略
- **代码审查**：重要更改应通过 Pull Request 审查
- **冲突处理**：及时处理合并冲突

## 常见问题

### 问题 1：合并冲突
- **原因**：多个分支修改了同一文件的同一部分
- **解决**：手动解决冲突，然后提交

### 问题 2：提交到错误分支
- **原因**：未检查当前分支
- **解决**：使用 `git reset` 撤销提交，切换到正确分支

### 问题 3：远程仓库连接失败
- **原因**：认证问题、网络问题
- **解决**：检查认证配置、网络连接

## 最佳实践

- 使用清晰的分支命名和提交信息
- 遵循 Conventional Commits 规范
- 定期提交代码，避免大提交
- 使用 Pull Request 进行代码审查
- 配置 pre-commit 钩子自动检查代码
- 使用任务管理工具自动化常见操作
