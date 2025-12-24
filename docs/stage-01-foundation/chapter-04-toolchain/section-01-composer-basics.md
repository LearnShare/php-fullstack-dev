# 1.3.1 Composer 基础

## 概述

Composer 是 PHP 的依赖管理工具，类似于 Node.js 的 npm 或 Python 的 pip。它不仅可以管理项目依赖，还提供了强大的自动加载功能。掌握 Composer 的使用是现代 PHP 开发的基础。

## 什么是 Composer

### Composer 的定义

**Composer** 是 PHP 的依赖管理工具，主要功能：

- **依赖管理**：安装和管理第三方库
- **自动加载**：自动加载类和文件
- **版本控制**：管理依赖的版本
- **包管理**：发布和分享自己的包

### 为什么需要 Composer

**传统方式的问题**：

```php
<?php
// 传统方式：手动下载和包含
require 'vendor/library1/Class1.php';
require 'vendor/library2/Class2.php';
require 'vendor/library3/Class3.php';
// ... 需要管理大量文件
```

**使用 Composer 的优势**：

```php
<?php
// 现代方式：自动加载
require __DIR__ . '/vendor/autoload.php';

use Library1\Class1;
use Library2\Class2;

$obj1 = new Class1();  // 自动加载
$obj2 = new Class2();  // 自动加载
```

## 安装 Composer

### Windows 安装

**方法 1：使用安装程序（推荐）**

1. 下载 [Composer-Setup.exe](https://getcomposer.org/Composer-Setup.exe)
2. 运行安装程序，按提示完成安装
3. 验证安装：`composer --version`

**方法 2：手动安装**

```bash
# 下载 composer.phar
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"

# 移动到全局位置（可选）
# 将 composer.phar 复制到 PATH 环境变量中的目录
```

### Linux/macOS 安装

**全局安装**：

```bash
# 下载并安装
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"

# 移动到全局位置
sudo mv composer.phar /usr/local/bin/composer

# 验证安装
composer --version
```

**本地安装**：

```bash
# 下载到项目目录
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"

# 使用本地 composer.phar
php composer.phar --version
```

### 验证安装

```bash
# 检查版本
composer --version

# 查看帮助
composer help

# 查看命令列表
composer list
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

## composer.json 文件

### 基本结构

**`composer.json`** 是 Composer 的配置文件，定义项目的依赖和自动加载规则。

**基本结构**：

```json
{
    "name": "vendor/project-name",
    "description": "项目描述",
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

### 字段说明

| 字段 | 说明 | 示例 |
| :--- | :--- | :--- |
| `name` | 包名，格式为 `vendor/package` | `"myvendor/myapp"` |
| `description` | 项目描述 | `"My awesome application"` |
| `type` | 包类型 | `"project"`, `"library"` |
| `require` | 生产环境依赖 | `{"php": "^8.2"}` |
| `require-dev` | 开发环境依赖 | `{"phpunit/phpunit": "^10.0"}` |
| `autoload` | 自动加载配置 | 见 [1.3.2 Composer 自动加载](section-02-composer-autoload.md) |
| `license` | 许可证 | `"MIT"`, `"Apache-2.0"` |
| `authors` | 作者信息 | `[{"name": "John", "email": "john@example.com"}]` |

### 创建 composer.json

**方法 1：使用命令行**

```bash
# 交互式创建
composer init

# 会提示输入：
# - Package name
# - Description
# - Author
# - Minimum stability
# - License
# - Dependencies
```

**方法 2：手动创建**

```bash
# 创建文件
touch composer.json

# 编辑文件，添加基本配置
```

## 如何安装和导入第三方模块

### 查找包

**使用 Packagist**（PHP 包仓库）：

- 访问 [packagist.org](https://packagist.org)
- 搜索需要的包（如 `monolog/monolog`、`guzzlehttp/guzzle`）
- 查看包的文档、版本、依赖关系

**使用命令行搜索**：

```bash
# 搜索包
composer search monolog

# 显示包信息
composer show monolog/monolog
```

### 安装包

**使用 `composer require` 命令**：

```bash
# 安装单个包
composer require monolog/monolog

# 安装指定版本
composer require monolog/monolog:^2.0

# 安装多个包
composer require monolog/monolog guzzlehttp/guzzle

# 安装开发依赖
composer require --dev phpunit/phpunit

# 安装并更新 composer.lock
composer require monolog/monolog --update-with-dependencies
```

**安装过程**：

1. Composer 会更新 `composer.json`，添加依赖
2. 下载依赖包到 `vendor/` 目录
3. 生成 `composer.lock` 文件（锁定版本）
4. 更新 `vendor/autoload.php` 自动加载文件

### 使用第三方包

**步骤 1：引入自动加载文件**

```php
<?php
// index.php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';
```

**步骤 2：使用包中的类**

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

// 使用 use 导入类
use Monolog\Logger;
use Monolog\Handler\StreamHandler;

// 创建 Logger 实例
$logger = new Logger('my_app');
$logger->pushHandler(new StreamHandler('app.log', Logger::WARNING));

// 使用
$logger->warning('This is a warning');
```

### 常用第三方包示例

#### 1. Monolog（日志库）

```bash
composer require monolog/monolog
```

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Handler\RotatingFileHandler;

$logger = new Logger('app');
$logger->pushHandler(new StreamHandler('php://stdout', Logger::DEBUG));
$logger->pushHandler(new RotatingFileHandler('app.log', 7, Logger::WARNING));

$logger->info('Application started');
$logger->warning('This is a warning');
$logger->error('This is an error');
```

#### 2. Guzzle（HTTP 客户端）

```bash
composer require guzzlehttp/guzzle
```

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use GuzzleHttp\Client;

$client = new Client();
$response = $client->request('GET', 'https://api.example.com/data');
$body = $response->getBody()->getContents();
echo $body;
```

#### 3. Carbon（日期时间库）

```bash
composer require nesbot/carbon
```

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Carbon\Carbon;

$now = Carbon::now();
echo $now->format('Y-m-d H:i:s') . "\n";

$tomorrow = Carbon::tomorrow();
echo $tomorrow->diffForHumans() . "\n";
```

### 更新依赖

```bash
# 更新所有依赖到最新版本（在 composer.json 约束内）
composer update

# 更新特定包
composer update monolog/monolog

# 更新依赖并更新 composer.lock
composer update --lock

# 只更新 composer.lock，不更新包
composer update --lock --no-install
```

### 移除依赖

```bash
# 移除包
composer remove monolog/monolog

# 移除开发依赖
composer remove --dev phpunit/phpunit

# 移除多个包
composer remove monolog/monolog guzzlehttp/guzzle
```

### 安装现有项目的依赖

```bash
# 根据 composer.lock 安装（推荐生产环境）
composer install

# 安装时不包含开发依赖
composer install --no-dev

# 安装并优化自动加载
composer install --optimize-autoloader
```

## 版本约束

### 版本约束语法

| 约束 | 说明 | 示例 |
| :--- | :--- | :--- |
| `^1.2.3` | 兼容版本（推荐） | `>=1.2.3 <2.0.0` |
| `~1.2.3` | 近似版本 | `>=1.2.3 <1.3.0` |
| `>=1.2.3` | 大于等于 | `>=1.2.3` |
| `<=1.2.3` | 小于等于 | `<=1.2.3` |
| `1.2.3` | 精确版本 | `=1.2.3` |
| `*` | 任意版本 | 任何版本 |
| `dev-main` | 开发分支 | `main` 分支 |

### 版本约束示例

```json
{
    "require": {
        "php": "^8.2",
        "monolog/monolog": "^3.0",
        "guzzlehttp/guzzle": "~7.0",
        "symfony/console": ">=5.0 <6.0"
    }
}
```

## composer.lock 文件

### 什么是 composer.lock

**`composer.lock`** 文件记录了所有依赖的确切版本，确保在不同环境中安装相同版本的包。

### 为什么需要 composer.lock

- **版本一致性**：确保所有环境使用相同版本
- **可重现构建**：可以精确重现安装过程
- **安全**：避免意外升级到不兼容版本

### 使用 composer.lock

```bash
# 根据 composer.lock 安装（生产环境推荐）
composer install

# 更新 composer.lock（开发环境）
composer update
```

### 版本控制

- **提交 `composer.lock`**：确保团队使用相同版本
- **不提交 `vendor/`**：添加到 `.gitignore`

## 常用命令

### 基本命令

```bash
# 初始化项目
composer init

# 安装依赖
composer install

# 更新依赖
composer update

# 添加依赖
composer require package/name

# 移除依赖
composer remove package/name

# 显示包信息
composer show package/name

# 搜索包
composer search keyword
```

### 信息命令

```bash
# 显示已安装的包
composer show

# 显示包的详细信息
composer show monolog/monolog

# 显示包的依赖树
composer show --tree

# 显示过时的包
composer outdated
```

### 诊断命令

```bash
# 验证 composer.json
composer validate

# 诊断问题
composer diagnose

# 检查安全漏洞
composer audit
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

## 最佳实践

### 1. 版本控制

```bash
# .gitignore
vendor/
composer.lock  # 对于库项目，不提交 lock 文件
```

**项目 vs 库**：
- **项目**：提交 `composer.lock`
- **库**：不提交 `composer.lock`

### 2. 环境区分

```json
{
    "require": {
        "php": "^8.2",
        "monolog/monolog": "^3.0"
    },
    "require-dev": {
        "phpunit/phpunit": "^10.0",
        "phpstan/phpstan": "^1.0"
    }
}
```

### 3. 性能优化

```bash
# 生产环境安装
composer install --no-dev --optimize-autoloader

# 开发环境安装
composer install
```

### 4. 安全性

```bash
# 定期检查安全漏洞
composer audit

# 更新有安全问题的包
composer update vulnerable/package
```

## 完整示例

```php
<?php
// composer.json
{
    "name": "myapp/app",
    "description": "My Application",
    "type": "project",
    "require": {
        "php": "^8.2",
        "monolog/monolog": "^3.0",
        "guzzlehttp/guzzle": "^7.0"
    },
    "require-dev": {
        "phpunit/phpunit": "^10.0"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

```php
<?php
// index.php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use App\Controllers\UserController;
use Monolog\Logger;
use Monolog\Handler\StreamHandler;

// 使用第三方库
$logger = new Logger('app');
$logger->pushHandler(new StreamHandler('app.log', Logger::WARNING));
$logger->warning('Application started');

// 使用自己的类
$controller = new UserController();
$controller->index();
```

## 注意事项

1. **版本控制**：
   - 将 `vendor/` 目录添加到 `.gitignore`
   - 项目提交 `composer.lock`，库不提交

2. **更新依赖**：
   - 开发环境：`composer update`
   - 生产环境：`composer install --no-dev --optimize-autoloader`

3. **性能优化**：
   - 生产环境使用 `--optimize-autoloader`
   - 使用 `--classmap-authoritative` 获得最佳性能

4. **安全性**：
   - 定期更新依赖包
   - 检查包的维护状态
   - 使用 `composer audit` 检查安全漏洞

5. **版本约束**：
   - 使用 `^` 允许小版本更新
   - 避免使用 `*` 或过宽的约束

## 相关章节

- **[1.3.2 Composer 自动加载](section-02-composer-autoload.md)**：深入学习 Composer 的自动加载机制
- **[1.3.3 编写和发布 Composer 包](section-03-composer-packages.md)**：学习如何创建和发布自己的包

## 练习

1. 安装 Composer 并验证安装。

2. 创建一个新项目，使用 Composer 安装 `monolog/monolog`，并编写日志记录代码。

3. 安装多个第三方包（如 Guzzle、Carbon），并在代码中使用它们。

4. 练习使用 `composer require`、`composer update` 和 `composer remove` 命令。

5. 配置 `.gitignore`，正确处理 `vendor/` 和 `composer.lock`。
