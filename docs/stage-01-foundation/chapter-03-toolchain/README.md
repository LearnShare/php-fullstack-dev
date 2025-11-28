# 1.3 核心工具链

## Composer

- 作用：依赖管理、自动加载生成、发布私有包，是现代 PHP 项目的"包管理器"。
- 安装：访问 [getcomposer.org](https://getcomposer.org)，下载 `composer.phar` 并移动到 PATH；Windows 用户运行官方安装器即可。

### 与 npm 对比

如果你熟悉 Node.js 的 npm，Composer 是 PHP 的包管理器，功能类似：

| 功能 | npm (Node.js) | Composer (PHP) |
| :--- | :------------ | :------------- |
| 包管理 | `npm install` | `composer install` |
| 添加依赖 | `npm install package` | `composer require package` |
| 移除依赖 | `npm uninstall package` | `composer remove package` |
| 更新依赖 | `npm update` | `composer update` |
| 配置文件 | `package.json` | `composer.json` |
| 锁定文件 | `package-lock.json` | `composer.lock` |
| 依赖目录 | `node_modules/` | `vendor/` |
| 脚本命令 | `npm run script` | `composer run-script script` |
| 全局安装 | `npm install -g` | `composer global require` |

**关键差异**：

1. **自动加载**：
   - npm：需要手动 `require()` 或 `import`
   - Composer：自动生成 `vendor/autoload.php`，自动加载类

2. **命名空间**：
   - npm：使用模块路径（`require('./module')`）
   - Composer：使用命名空间（`use Vendor\Package\Class`）

3. **版本约束**：
   - npm：`^1.0.0`、`~1.0.0`、`>=1.0.0`
   - Composer：`^1.0`、`~1.0`、`>=1.0`（语义相同）

**示例对比**：

```javascript
// npm/Node.js
// package.json
{
  "dependencies": {
    "express": "^4.18.0"
  }
}

// 使用
const express = require('express');
const app = express();
```

```php
// Composer/PHP
// composer.json
{
  "require": {
    "monolog/monolog": "^3.0"
  }
}

// 使用（自动加载）
require 'vendor/autoload.php';
use Monolog\Logger;
$logger = new Logger('app');
```
- 关键指令：
  - `composer init`：创建 `composer.json`，引导式填写包名、版本、PSR-4 命名空间。
  - `composer install --no-dev`：生产环境安装依赖。
  - `composer update vendor/package`：按需升级单个依赖，避免无脑全量更新。
  - `composer dump-autoload -o`：生成优化过的 PSR-4 map，提升自动加载性能。
- 建议配置：
  ```json
  {
    "config": {
      "preferred-install": "dist",
      "sort-packages": true
    }
  }
  ```
- 镜像：国内网络可通过 `composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/` 加速；如需恢复官方源，执行 `composer config -g --unset repos.packagist`.
- 安全审计：使用 `composer audit` 检查依赖包的安全漏洞（Composer 2.4+）。
- 新手练习：创建 `hello-composer` 目录，执行 `composer init`，添加 `psr-4` 自动加载，编写 `src/Greeter.php` 并通过 `vendor/bin/phpunit` 运行第一条测试。

### Composer 常用命令速查

#### `composer install`

- **语法**：`composer install [--no-dev] [--prefer-dist]`
- **关键参数**：
  - `--no-dev`：生产环境跳过 dev 依赖。
  - `--prefer-dist`：下载压缩包，安装更快。
- **示例**：
  ```bash
  composer install --no-dev --prefer-dist
  ```

#### `composer update`

- **语法**：`composer update [vendor/package] [--with-all-dependencies]`
- **关键参数**：
  - 指定包名可避免全量升级。
  - `--with-all-dependencies`：同时升级依赖树。
- **示例**：
  ```bash
  composer update symfony/console --with-all-dependencies
  ```

#### `composer require`

- **语法**：`composer require vendor/package:^version [--dev]`
- **关键参数**：
  - `--dev`：写入 `require-dev` 区块。
  - `--with-all-dependencies`：解决依赖冲突。
- **示例**：
  ```bash
  composer require pestphp/pest --dev
  composer require phpunit/phpunit:^11.0 --dev
  ```

#### `composer remove`

- **语法**：`composer remove vendor/package [--dev]`
- **作用**：自动从 `composer.json`、`composer.lock`、`vendor/` 清理目标依赖。
- **示例**：
  ```bash
  composer remove monolog/monolog
  ```

#### `composer dump-autoload`

- **语法**：`composer dump-autoload [-o|--optimize]`
- **说明**：`-o` 生成 classmap，提高生产环境加载性能。
- **示例**：
  ```bash
  composer dump-autoload -o
  ```

#### `composer config`

- **语法**：`composer config [--global] key value`
- **说明**：`--global` 修改全局配置，如镜像源。
- **示例**：
  ```bash
  composer config -g repos.packagist composer https://mirrors.aliyun.com/composer/
  ```

#### `composer audit`

- **语法**：`composer audit [--format=json]`
- **说明**：检查已安装依赖包的安全漏洞（Composer 2.4+）。
- **示例**：
  ```bash
  composer audit
  composer audit --format=json
  ```
- **建议**：在 CI/CD 流程中集成 `composer audit`，及时发现依赖漏洞。

## IDE 与扩展

| IDE      | 核心功能               | 推荐扩展                                   |
| :------- | :--------------------- | :----------------------------------------- |
| PhpStorm | 即时类型推断、调试、重构 | Psalm/PHPStan 集成、Laravel Idea           |
| VS Code  | 轻量、跨平台           | PHP Intelephense、PHP CS Fixer、Pest、Blade Syntax |

最佳实践：

- 启用 EditorConfig 和统一的代码样式（PSR-12），初学者可直接复制 `.editorconfig` 模板。
- 配置“保存时格式化 + 代码静态分析”，VS Code 用户可安装 PHP CS Fixer、PHPStan 插件。
- 利用 IDE Live Templates 生成 Controller、Repository 等骨架，减少重复输入。
- 将 IDE 设置导出为 `.icls` 或 `settings.zip`，放入仓库的 `docs/ide-config/` 中方便团队共享。
- `.env.example`：列出必须的环境变量，结合 `phpdotenv` 加载，确保新成员复制后即可运行。
- `Makefile`/`justfile` 示例：
  ```Makefile
  setup:
  	composer install
  	cp .env.example .env

  test:
  	composer run lint && composer run stan && phpunit
  ```
  初学者只需记住 `make setup`、`make test` 两条命令即可完成常用操作。

## Git 与任务管理

- 强制启用 `pre-commit` 钩子（lint、tests）。
- 通过 Conventional Commits 规范提交信息。
- 在多仓库项目中使用 `git worktree` 或 `gh` CLI 管理分支。

### `git switch`

- **语法**：`git switch <branch>`
- **说明**：搭配 `-c` 可创建并切换到新分支。
- **示例**：
  ```bash
  git switch -c feature/runtime-guide
  ```

### `git worktree`

- **语法**：`git worktree add <path> <branch>`
- **说明**：在同一仓库同时检出多个目录，便于多分支并行开发。
- **示例**：
  ```bash
  git worktree add ../proj-docs docs-update
  ```

### `git commit`

- **语法**：`git commit -m "type(scope): message"`
- **说明**：遵循 Conventional Commits，便于生成变更日志。
- **示例**：
  ```bash
  git commit -m "docs(runtime): add command table"
  ```

### `gh pr create`

- **语法**：`gh pr create --fill`
- **说明**：GitHub CLI 一键创建 PR，并用最近一次提交填充标题和描述。
- **示例**：
  ```bash
  gh pr create --fill --base main
  ```

## 容器与环境管理

- `docker compose`：编排 PHP-FPM、Nginx、MySQL、Redis、Mailpit。
- `just`、`make` 或 `taskfile`：封装常用命令，保证团队一致性。
- `.env.example`：列出必须的环境变量，结合 `phpdotenv` 加载。

### docker compose 命令语法

#### 启动

- **语法**：`docker compose up [service] [-d]`
- **参数**：
  - `-d`：后台运行。
  - `--build`：在启动前重新构建镜像。
- **示例**：
  ```bash
  docker compose up php-fpm nginx -d
  ```

#### 停止

- **语法**：`docker compose down [--volumes]`
- **参数**：
  - `--volumes`：连同数据卷一起删除。
- **示例**：
  ```bash
  docker compose down --volumes
  ```

#### 查看日志

- **语法**：`docker compose logs [-f] [service]`
- **参数**：
  - `-f`：实时跟随输出。
- **示例**：
  ```bash
  docker compose logs -f php-fpm
  ```

#### 进入容器

- **语法**：`docker compose exec service bash`
- **参数**：
  - `-T`：关闭 pseudo-tty，适合 CI。
- **示例**：
  ```bash
  docker compose exec php-fpm bash
  ```

## 自动化脚本

- `composer scripts` 示例：
  ```json
  "scripts": {
    "post-install-cmd": [
      "@php artisan key:generate"
    ],
    "lint": "phpcs --standard=psr12 src",
    "stan": "phpstan analyse src"
  }
  ```
- 在 CI 中使用 `composer validate --strict` 保障配置正确性；可配合 GitHub Actions、GitLab CI。
- 将 lint、test、build、deploy 步骤写入 `composer scripts` 或 `justfile`，结合 `pre-commit` 钩子执行，减少低级错误。

掌握本章内容后，你可以快速搭建带有统一依赖、IDE 配置、任务脚本与容器编排的现代 PHP 项目基础设施。记得在团队 wiki 中记录每个工具的安装步骤，让后来者“照着做就能成功”。
