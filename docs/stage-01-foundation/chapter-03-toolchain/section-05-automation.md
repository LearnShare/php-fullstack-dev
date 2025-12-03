# 1.3.5 自动化脚本

## 概述

自动化脚本可以简化重复性任务，提高开发效率。本章介绍如何使用 Composer Scripts、Makefile 等工具实现自动化。

## Composer Scripts

### 基本配置

在 `composer.json` 中添加 `scripts` 部分：

```json
{
    "scripts": {
        "post-install-cmd": [
            "@php artisan key:generate"
        ],
        "post-update-cmd": [
            "@php artisan migrate"
        ],
        "lint": "php-cs-fixer fix --dry-run --diff",
        "lint:fix": "php-cs-fixer fix",
        "stan": "phpstan analyse",
        "test": "phpunit",
        "test:coverage": "phpunit --coverage-html coverage/"
    }
}
```

### 事件钩子

| 事件 | 触发时机 | 示例 |
| :--- | :------- | :--- |
| `pre-install-cmd` | `composer install` 之前 | 检查 PHP 版本 |
| `post-install-cmd` | `composer install` 之后 | 生成配置文件 |
| `pre-update-cmd` | `composer update` 之前 | 备份数据库 |
| `post-update-cmd` | `composer update` 之后 | 运行迁移 |
| `pre-package-install` | 安装包之前 | 检查依赖 |
| `post-package-install` | 安装包之后 | 清理缓存 |

### 调用其他脚本

```json
{
    "scripts": {
        "test": "phpunit",
        "lint": "php-cs-fixer fix --dry-run",
        "check": [
            "@lint",
            "@test"
        ]
    }
}
```

### 使用示例

```bash
# 运行脚本
composer run lint
composer run stan
composer run test

# 查看所有脚本
composer run-script --list
```

## Makefile

### 基本语法

```makefile
target: dependencies
	command
```

### 示例

```makefile
.PHONY: setup test lint stan clean help

help: ## 显示帮助信息
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'

setup: ## 初始化项目
	composer install
	cp .env.example .env
	php artisan key:generate

test: ## 运行测试
	composer run test

lint: ## 检查代码风格
	composer run lint

lint:fix: ## 修复代码风格
	composer run lint:fix

stan: ## 运行静态分析
	composer run stan

check: lint stan test ## 运行所有检查

clean: ## 清理生成的文件
	rm -rf vendor/
	rm -rf node_modules/
	rm -rf coverage/

install: ## 安装依赖
	composer install
	npm install
```

### 使用

```bash
# 显示帮助
make help

# 运行目标
make setup
make test
make lint

# 运行多个目标
make lint stan test
```

## Justfile

### 安装

```bash
# macOS
brew install just

# Linux
# 参考 https://github.com/casey/just
```

### 基本语法

```just
recipe:
    command
```

### 示例

```just
# 默认任务
default:
    just --list

# 初始化项目
setup:
    composer install
    cp .env.example .env
    php artisan key:generate

# 运行测试
test:
    composer run test

# 检查代码风格
lint:
    composer run lint

# 修复代码风格
lint-fix:
    composer run lint:fix

# 静态分析
stan:
    composer run stan

# 所有检查
check: lint stan test
    echo "All checks passed"
```

### 使用

```bash
# 显示所有任务
just

# 运行任务
just setup
just test
just lint

# 运行多个任务
just lint stan test
```

## Taskfile

### 安装

```bash
# macOS
brew install go-task/tap/go-task

# Linux
# 参考 https://taskfile.dev/installation/
```

### 基本语法

```yaml
version: '3'

tasks:
  task-name:
    desc: Task description
    cmds:
      - command1
      - command2
```

### 示例

```yaml
version: '3'

tasks:
  setup:
    desc: Setup project
    cmds:
      - composer install
      - cp .env.example .env
      - php artisan key:generate

  test:
    desc: Run tests
    cmds:
      - composer run test

  lint:
    desc: Check code style
    cmds:
      - composer run lint

  lint:fix:
    desc: Fix code style
    cmds:
      - composer run lint:fix

  stan:
    desc: Run static analysis
    cmds:
      - composer run stan

  check:
    desc: Run all checks
    deps: [lint, stan, test]
    cmds:
      - echo "All checks passed"
```

### 使用

```bash
# 显示所有任务
task --list

# 运行任务
task setup
task test
task lint

# 运行多个任务
task lint stan test
```

## CI/CD 集成

### GitHub Actions

创建 `.github/workflows/ci.yml`：

```yaml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
          extensions: mbstring, pdo_mysql
      
      - name: Install dependencies
        run: composer install --prefer-dist --no-progress
      
      - name: Validate composer.json
        run: composer validate --strict
      
      - name: Run lint
        run: composer run lint
      
      - name: Run static analysis
        run: composer run stan
      
      - name: Run tests
        run: composer run test
```

### GitLab CI

创建 `.gitlab-ci.yml`：

```yaml
stages:
  - lint
  - test

lint:
  stage: lint
  image: php:8.2
  script:
    - composer install
    - composer run lint
    - composer run stan

test:
  stage: test
  image: php:8.2
  script:
    - composer install
    - composer run test
```

## 完整示例

### 项目配置

```
my-project/
├── composer.json
├── Makefile
├── justfile
├── Taskfile.yml
└── .github/
    └── workflows/
        └── ci.yml
```

### composer.json

```json
{
    "scripts": {
        "post-install-cmd": [
            "@php -r \"file_exists('.env') || copy('.env.example', '.env');\""
        ],
        "lint": "php-cs-fixer fix --dry-run --diff",
        "lint:fix": "php-cs-fixer fix",
        "stan": "phpstan analyse",
        "test": "phpunit",
        "check": [
            "@lint",
            "@stan",
            "@test"
        ]
    }
}
```

### Makefile

```makefile
.PHONY: help setup test lint stan check clean

help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'

setup: ## 初始化项目
	composer install
	cp .env.example .env

test: ## 运行测试
	composer run test

lint: ## 检查代码风格
	composer run lint

lint:fix: ## 修复代码风格
	composer run lint:fix

stan: ## 运行静态分析
	composer run stan

check: lint stan test ## 运行所有检查

clean: ## 清理生成的文件
	rm -rf vendor/ coverage/
```

## 注意事项

1. **脚本顺序**：确保脚本按正确顺序执行，处理依赖关系。

2. **错误处理**：脚本失败时应该退出，避免继续执行。

3. **跨平台**：考虑 Windows、macOS、Linux 的兼容性。

4. **文档说明**：在 README 中说明如何使用自动化脚本。

5. **CI 集成**：在 CI 流程中使用相同的脚本，确保一致性。

## 练习

1. 创建 Composer Scripts，封装常用的开发任务。

2. 创建 Makefile，提供项目管理的常用命令。

3. 配置 GitHub Actions 或 GitLab CI，自动化代码检查。

4. 创建安装脚本，一键初始化项目环境。

5. 实现代码检查的 pre-commit 钩子，防止提交错误代码。
