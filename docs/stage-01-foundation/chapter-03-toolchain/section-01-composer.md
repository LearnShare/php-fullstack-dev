# 1.3.1 Composer 依赖管理

## 概述

Composer 是 PHP 的依赖管理工具，类似于 Node.js 的 npm。它负责管理项目依赖、生成自动加载文件，是现代 PHP 项目的核心工具。

## Composer 基础

### 作用

- **依赖管理**：管理项目所需的第三方包
- **自动加载**：自动生成 PSR-4 自动加载文件
- **版本控制**：通过 `composer.lock` 锁定依赖版本
- **包发布**：可以发布自己的包到 Packagist

### 安装

#### 方式一：官方安装器（推荐）

```bash
# 下载安装脚本
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"

# 移动到全局路径
sudo mv composer.phar /usr/local/bin/composer
```

#### 方式二：Windows 安装器

访问 [getcomposer.org](https://getcomposer.org)，下载 Windows 安装器并运行。

#### 验证安装

```bash
composer --version
# 输出：Composer version 2.x.x
```

## 与 npm 对比

如果你熟悉 Node.js 的 npm，Composer 功能类似：

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

### 关键差异

1. **自动加载**：
   - npm：需要手动 `require()` 或 `import`
   - Composer：自动生成 `vendor/autoload.php`，自动加载类

2. **命名空间**：
   - npm：使用模块路径（`require('./module')`）
   - Composer：使用命名空间（`use Vendor\Package\Class`）

3. **版本约束**：
   - npm：`^1.0.0`、`~1.0.0`、`>=1.0.0`
   - Composer：`^1.0`、`~1.0`、`>=1.0`（语义相同）

## 基本使用

### 初始化项目

```bash
# 创建新项目
composer init

# 交互式填写信息
# Package name: vendor/package-name
# Description: Project description
# Author: Your Name <email@example.com>
# Minimum Stability: stable
# License: MIT
```

### composer.json 结构

```json
{
    "name": "vendor/package-name",
    "description": "Project description",
    "type": "project",
    "require": {
        "php": "^8.2"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

## 常用命令

### composer install

**语法**：`composer install [--no-dev] [--prefer-dist]`

**参数**：
- `--no-dev`：生产环境跳过 dev 依赖
- `--prefer-dist`：下载压缩包，安装更快

**示例**：

```bash
# 安装所有依赖
composer install

# 生产环境安装
composer install --no-dev --prefer-dist
```

### composer require

**语法**：`composer require vendor/package:^version [--dev]`

**参数**：
- `--dev`：写入 `require-dev` 区块
- `--with-all-dependencies`：解决依赖冲突

**示例**：

```bash
# 添加依赖
composer require monolog/monolog

# 添加开发依赖
composer require phpunit/phpunit --dev

# 指定版本
composer require symfony/console:^6.0
```

### composer update

**语法**：`composer update [vendor/package] [--with-all-dependencies]`

**参数**：
- 指定包名可避免全量升级
- `--with-all-dependencies`：同时升级依赖树

**示例**：

```bash
# 更新所有依赖
composer update

# 更新指定包
composer update symfony/console

# 更新并包含依赖
composer update symfony/console --with-all-dependencies
```

### composer remove

**语法**：`composer remove vendor/package [--dev]`

**作用**：自动从 `composer.json`、`composer.lock`、`vendor/` 清理目标依赖。

**示例**：

```bash
composer remove monolog/monolog
```

### composer dump-autoload

**语法**：`composer dump-autoload [-o|--optimize]`

**说明**：`-o` 生成 classmap，提高生产环境加载性能。

**示例**：

```bash
# 重新生成自动加载
composer dump-autoload

# 优化自动加载
composer dump-autoload -o
```

### composer config

**语法**：`composer config [--global] key value`

**说明**：`--global` 修改全局配置，如镜像源。

**示例**：

```bash
# 设置镜像源
composer config -g repos.packagist composer https://mirrors.aliyun.com/composer/

# 查看配置
composer config -g repos.packagist

# 恢复官方源
composer config -g --unset repos.packagist
```

### composer audit

**语法**：`composer audit [--format=json]`

**说明**：检查已安装依赖包的安全漏洞（Composer 2.4+）。

**示例**：

```bash
# 检查安全漏洞
composer audit

# JSON 格式输出
composer audit --format=json
```

## 版本约束

### 语义版本

| 约束 | 说明 | 示例 |
| :--- | :--- | :--- |
| `^1.2.3` | 兼容版本（推荐） | `>=1.2.3 <2.0.0` |
| `~1.2.3` | 近似版本 | `>=1.2.3 <1.3.0` |
| `>=1.2.3` | 最小版本 | `>=1.2.3` |
| `1.2.3` | 精确版本 | `=1.2.3` |
| `*` | 任意版本 | 任何版本 |

### 示例

```json
{
    "require": {
        "php": "^8.2",
        "monolog/monolog": "^3.0",
        "symfony/console": "~6.0",
        "guzzlehttp/guzzle": ">=7.0 <8.0"
    }
}
```

## 自动加载

### PSR-4 自动加载

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/",
            "App\\Controllers\\": "src/Controllers/"
        }
    }
}
```

### 使用自动加载

```php
<?php
// 引入自动加载文件
require __DIR__ . '/vendor/autoload.php';

// 使用命名空间
use App\Controllers\UserController;

$controller = new UserController();
```

## 配置建议

### 推荐配置

```json
{
    "config": {
        "preferred-install": "dist",
        "sort-packages": true,
        "optimize-autoloader": true
    }
}
```

### 镜像配置

```bash
# 使用阿里云镜像（国内推荐）
composer config -g repos.packagist composer https://mirrors.aliyun.com/composer/

# 使用腾讯云镜像
composer config -g repos.packagist composer https://mirrors.cloud.tencent.com/composer/
```

## 完整示例

```bash
# 1. 创建新项目
mkdir my-project
cd my-project

# 2. 初始化 Composer
composer init

# 3. 添加依赖
composer require monolog/monolog

# 4. 配置自动加载
# 编辑 composer.json，添加 autoload 配置

# 5. 生成自动加载
composer dump-autoload

# 6. 使用
# 在代码中 require 'vendor/autoload.php'
```

## 注意事项

1. **版本锁定**：`composer.lock` 应该提交到版本控制，确保团队使用相同版本。

2. **生产环境**：使用 `composer install --no-dev --prefer-dist` 安装。

3. **安全审计**：定期运行 `composer audit` 检查安全漏洞。

4. **性能优化**：生产环境使用 `composer dump-autoload -o` 优化自动加载。

5. **镜像源**：国内用户建议使用镜像源加速下载。

## 练习

1. 创建一个新项目，使用 Composer 初始化并添加依赖。

2. 配置 PSR-4 自动加载，创建类并验证自动加载是否正常工作。

3. 使用 `composer audit` 检查项目依赖的安全漏洞。

4. 配置镜像源，对比使用镜像前后的下载速度。

5. 创建一个包含多个依赖的项目，理解版本约束的作用。
