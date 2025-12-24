# 1.3.5 Git 与任务管理

## 概述

Git 是版本控制工具，任务管理工具可以帮助团队协作和自动化工作流。本章介绍 Git 的基本使用和任务管理工具。

## Git 基础

### 初始化仓库

```bash
# 创建新仓库
git init

# 克隆现有仓库
git clone https://github.com/user/repo.git
```

### 基本工作流

```bash
# 1. 查看状态
git status

# 2. 添加文件
git add .

# 3. 提交
git commit -m "feat: add new feature"

# 4. 推送到远程
git push origin main
```

## 分支管理

### 创建和切换分支

```bash
# 创建新分支
git branch feature/new-feature

# 切换到分支
git checkout feature/new-feature

# 或使用 git switch（Git 2.23+）
git switch feature/new-feature

# 创建并切换
git switch -c feature/new-feature
```

### git switch

**语法**：`git switch <branch>`

**参数**：
- `-c <branch>`：创建并切换到新分支
- `-d <branch>`：删除分支

**示例**：

```bash
# 切换到分支
git switch main

# 创建并切换
git switch -c feature/runtime-guide

# 删除分支
git switch -d feature/old-feature
```

### 合并分支

```bash
# 切换到主分支
git switch main

# 合并功能分支
git merge feature/new-feature

# 删除已合并的分支
git branch -d feature/new-feature
```

## 高级功能

### git worktree

**语法**：`git worktree add <path> <branch>`

**说明**：在同一仓库同时检出多个目录，便于多分支并行开发。

**示例**：

```bash
# 添加新的工作树
git worktree add ../proj-docs docs-update

# 列出所有工作树
git worktree list

# 删除工作树
git worktree remove ../proj-docs
```

**使用场景**：
- 同时开发多个功能
- 需要保持不同分支的代码同时可用
- 避免频繁切换分支

### git commit

**语法**：`git commit -m "type(scope): message"`

**说明**：遵循 Conventional Commits，便于生成变更日志。

**提交类型**：

| 类型 | 说明 | 示例 |
| :--- | :--- | :--- |
| `feat` | 新功能 | `feat(auth): add login feature` |
| `fix` | 修复 bug | `fix(api): handle null response` |
| `docs` | 文档更新 | `docs(readme): update installation` |
| `style` | 代码格式 | `style(php): fix indentation` |
| `refactor` | 重构 | `refactor(service): simplify logic` |
| `test` | 测试 | `test(user): add unit tests` |
| `chore` | 构建/工具 | `chore(deps): update composer` |

**示例**：

```bash
git commit -m "feat(user): add user registration"
git commit -m "fix(api): handle authentication error"
git commit -m "docs(readme): update setup instructions"
```

## GitHub CLI

### 安装

```bash
# macOS
brew install gh

# Linux
# 参考 https://cli.github.com/manual/installation
```

### 认证

```bash
gh auth login
```

### 常用命令

#### gh pr create

**语法**：`gh pr create [--fill] [--base <branch>]`

**说明**：创建 Pull Request。

**示例**：

```bash
# 创建 PR，自动填充标题和描述
gh pr create --fill

# 指定基础分支
gh pr create --fill --base main

# 创建草稿 PR
gh pr create --draft
```

#### gh pr list

**语法**：`gh pr list [--state <state>]`

**示例**：

```bash
# 列出所有 PR
gh pr list

# 列出打开的 PR
gh pr list --state open

# 列出已合并的 PR
gh pr list --state merged
```

#### gh pr checkout

**语法**：`gh pr checkout <number>`

**说明**：检出 PR 对应的分支。

**示例**：

```bash
gh pr checkout 123
```

## Pre-commit 钩子

### 安装 pre-commit

```bash
# 安装 pre-commit
pip install pre-commit

# 或使用 Homebrew
brew install pre-commit
```

### 配置

创建 `.pre-commit-config.yaml`：

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.4.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
      - id: check-php-syntax
      - id: php-cs-fixer
        args: [--config=.php-cs-fixer.dist.php]
```

### 安装钩子

```bash
pre-commit install
```

### 手动运行

```bash
# 检查所有文件
pre-commit run --all-files

# 检查特定文件
pre-commit run --files src/App.php
```

## 任务管理工具

### Makefile

**创建 `Makefile`**：

```makefile
.PHONY: setup test lint stan clean

setup:
	composer install
	cp .env.example .env
	php artisan key:generate

test:
	composer run test

lint:
	vendor/bin/php-cs-fixer fix --dry-run --diff

stan:
	vendor/bin/phpstan analyse

clean:
	rm -rf vendor/
	rm -rf node_modules/
```

**使用**：

```bash
make setup
make test
make lint
```

### Justfile

**安装**：

```bash
# macOS
brew install just

# Linux
# 参考 https://github.com/casey/just
```

**创建 `justfile`**：

```just
setup:
    composer install
    cp .env.example .env

test:
    composer run test

lint:
    vendor/bin/php-cs-fixer fix --dry-run --diff

stan:
    vendor/bin/phpstan analyse
```

**使用**：

```bash
just setup
just test
just lint
```

### Taskfile

**安装**：

```bash
# macOS
brew install go-task/tap/go-task

# Linux
# 参考 https://taskfile.dev/installation/
```

**创建 `Taskfile.yml`**：

```yaml
version: '3'

tasks:
  setup:
    desc: Setup project
    cmds:
      - composer install
      - cp .env.example .env

  test:
    desc: Run tests
    cmds:
      - composer run test

  lint:
    desc: Check code style
    cmds:
      - vendor/bin/php-cs-fixer fix --dry-run --diff
```

**使用**：

```bash
task setup
task test
task lint
```

## Composer Scripts

### 配置

在 `composer.json` 中添加：

```json
{
    "scripts": {
        "post-install-cmd": [
            "@php artisan key:generate"
        ],
        "lint": "php-cs-fixer fix --dry-run --diff",
        "lint:fix": "php-cs-fixer fix",
        "stan": "phpstan analyse",
        "test": "phpunit",
        "test:coverage": "phpunit --coverage-html coverage/"
    }
}
```

### 使用

```bash
# 运行脚本
composer run lint
composer run stan
composer run test

# 查看所有脚本
composer run-script --list
```

## 完整示例

### 项目配置

```
my-project/
├── .git/
├── .pre-commit-config.yaml
├── Makefile
├── composer.json
└── .github/
    └── workflows/
        └── ci.yml
```

### 工作流

1. **初始化项目**
   ```bash
   git init
   composer init
   ```

2. **配置 Git**
   ```bash
   git config user.name "Your Name"
   git config user.email "your@email.com"
   ```

3. **创建分支**
   ```bash
   git switch -c feature/new-feature
   ```

4. **开发功能**
   ```bash
   # 编写代码
   # 运行检查
   make lint
   make stan
   ```

5. **提交代码**
   ```bash
   git add .
   git commit -m "feat(feature): add new feature"
   ```

6. **创建 PR**
   ```bash
   git push origin feature/new-feature
   gh pr create --fill
   ```

## 注意事项

1. **提交信息**：使用 Conventional Commits 规范，便于生成变更日志。

2. **分支策略**：根据团队规范选择 Git Flow、GitHub Flow 等。

3. **钩子检查**：在 pre-commit 钩子中运行代码检查，避免提交错误代码。

4. **任务脚本**：将常用命令封装到 Makefile 或 justfile，提高效率。

5. **CI/CD**：在 CI 流程中运行完整的测试和检查。

## 练习

1. 创建一个新项目，配置 Git 仓库和分支策略。

2. 使用 Conventional Commits 规范提交代码。

3. 配置 pre-commit 钩子，自动检查代码格式。

4. 创建 Makefile 或 justfile，封装常用命令。

5. 使用 GitHub CLI 创建和管理 Pull Request。
