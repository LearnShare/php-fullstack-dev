# 2.13.2 Composer Autoload

## 概述

Composer 是 PHP 的依赖管理工具，同时提供了强大的自动加载功能。使用 Composer autoload 可以自动加载类文件，无需手动 `require`。

## Composer 基础

### 安装 Composer

```bash
# 下载安装脚本
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

### composer.json

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

## PSR-4 自动加载

### 基本配置

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/",
            "App\\Controllers\\": "src/Controllers/",
            "App\\Models\\": "src/Models/"
        }
    }
}
```

### 目录结构

```
project/
├── composer.json
├── vendor/
│   └── autoload.php
└── src/
    ├── Controllers/
    │   └── UserController.php
    ├── Models/
    │   └── User.php
    └── Services/
        └── UserService.php
```

### 类文件示例

```php
<?php
// src/Controllers/UserController.php
declare(strict_types=1);

namespace App\Controllers;

class UserController
{
    public function index(): void
    {
        echo "User list\n";
    }
}
```

```php
<?php
// src/Models/User.php
declare(strict_types=1);

namespace App\Models;

class User
{
    public string $name;
    public int $age;
}
```

### 使用自动加载

```php
<?php
// index.php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use App\Controllers\UserController;
use App\Models\User;

$controller = new UserController();
$user = new User();
```

## 生成自动加载文件

### composer dump-autoload

```bash
# 生成自动加载文件
composer dump-autoload

# 优化自动加载（生产环境）
composer dump-autoload --optimize

# 优化并生成类映射（最佳性能）
composer dump-autoload --optimize --classmap-authoritative
```

## 类映射（Classmap）

### 配置

```json
{
    "autoload": {
        "classmap": [
            "src/Legacy/",
            "src/Helpers/"
        ]
    }
}
```

### 使用场景

- 遗留代码（不符合 PSR-4）
- 辅助函数类
- 第三方库

## 文件自动加载

### 配置

```json
{
    "autoload": {
        "files": [
            "src/helpers.php",
            "src/functions.php"
        ]
    }
}
```

### 使用场景

- 全局函数
- 常量定义
- 配置加载

## 完整示例

```php
<?php
// composer.json
{
    "name": "myapp/app",
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        },
        "classmap": [
            "src/Legacy/"
        ],
        "files": [
            "src/helpers.php"
        ]
    }
}
```

```php
<?php
// index.php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use App\Controllers\UserController;

$controller = new UserController();
$controller->index();
```

## 注意事项

1. **命名空间**：类名必须与命名空间和文件路径匹配。

2. **文件命名**：文件名必须与类名相同（区分大小写）。

3. **更新自动加载**：添加新类后运行 `composer dump-autoload`。

4. **性能优化**：生产环境使用 `--optimize` 标志。

5. **版本控制**：将 `vendor/` 目录添加到 `.gitignore`。

## 练习

1. 创建一个项目，配置 PSR-4 自动加载。

2. 编写一个类，验证自动加载是否正常工作。

3. 实现一个辅助函数文件，使用 `files` 自动加载。

4. 创建一个遗留代码适配器，使用 `classmap` 自动加载。

5. 编写脚本，比较优化前后的自动加载性能。
