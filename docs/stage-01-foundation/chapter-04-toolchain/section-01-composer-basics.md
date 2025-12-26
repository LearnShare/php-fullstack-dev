# 1.4.1 Composer 基础

## 概述

Composer 是 PHP 的依赖管理工具，类似于 Node.js 的 npm 或 Python 的 pip。它不仅可以管理项目依赖，还提供了强大的自动加载功能。掌握 Composer 的使用是现代 PHP 开发的基础。

## 安装 Composer

### Windows

**使用安装程序（推荐）**：

1. 访问 Composer 官网：https://getcomposer.org/download/
2. 下载并运行 `Composer-Setup.exe`
3. 按照安装向导完成安装

**验证安装**：

```bash
composer --version
```

### Linux/macOS

**全局安装**：

```bash
# 下载安装脚本
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

# 运行安装脚本
php composer-setup.php

# 移动到全局位置
sudo mv composer.phar /usr/local/bin/composer

# 验证安装
composer --version
```

## 基本使用

### 初始化项目

**交互式创建**：

```bash
composer init
```

**手动创建 composer.json**：

```json
{
    "name": "vendor/project",
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

### 常用命令

**安装依赖**：

```bash
composer install
```

**添加新包并安装**：

```bash
composer require vendor/package
```

**更新所有依赖**：

```bash
composer update
```

**移除依赖**：

```bash
composer remove vendor/package
```

**显示已安装的包**：

```bash
composer show
```

## 完整示例

### 示例 1：创建新项目

```bash
# 1. 创建项目目录
mkdir myproject
cd myproject

# 2. 初始化 Composer
composer init

# 3. 添加依赖
composer require monolog/monolog

# 4. 查看安装的包
composer show
```

### 示例 2：使用自动加载

创建 `src/App.php`：

```php
<?php
declare(strict_types=1);

namespace App;

class App
{
    public function run(): void
    {
        echo "Hello from App!\n";
    }
}
```

在 `index.php` 中使用：

```php
<?php
declare(strict_types=1);

require 'vendor/autoload.php';

use App\App;

$app = new App();
$app->run();
```

运行：

```bash
php index.php
```

**输出**：

```
Hello from App!
```

## 注意事项

- **composer.lock**：应该提交到版本控制，确保团队使用相同版本
- **版本约束**：合理使用版本约束，避免过于严格或过于宽松
- **开发依赖**：区分生产依赖和开发依赖
- **自动加载**：确保在代码中引入 `vendor/autoload.php`

## 常见问题

### 问题 1：composer install 失败

**原因**：网络问题、版本冲突、权限问题

**解决方法**：

```bash
# 使用国内镜像
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

### 问题 2：自动加载不工作

**原因**：未引入 autoload.php、命名空间配置错误

**解决方法**：

```php
// 确保引入 autoload.php
require 'vendor/autoload.php';

// 检查命名空间配置
// composer.json 中的 autoload 配置必须与代码中的命名空间匹配
```

## 最佳实践

- 提交 composer.lock 到版本控制
- 区分生产依赖和开发依赖
- 使用 `composer validate` 验证 composer.json

## 相关章节

- **2.11 文件引入与模块化**：了解模块化开发的基础概念和 PSR-4 自动加载
- **2.13 代码规范**：了解 PSR-4 自动加载标准的详细规范
- **12.2 Composer 高级应用**：深入学习 Composer 的高级用法，包括版本约束详解、composer.json 完整配置、包发布等

## 练习任务

1. 安装 Composer，并使用 `composer --version` 验证安装。
2. 创建一个新项目，使用 `composer init` 初始化，并添加一个依赖包。
3. 配置自动加载（PSR-4），创建一个类并使用自动加载引入。
4. 使用 `composer require` 安装一个第三方包（如 monolog/monolog），并在代码中使用。
5. 查看 `composer.lock` 文件，了解依赖版本锁定机制。
