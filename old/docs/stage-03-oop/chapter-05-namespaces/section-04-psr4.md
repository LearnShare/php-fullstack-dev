# 3.5.4 PSR-4 自动加载标准

## 概述

PSR-4 是 PHP 标准的自动加载规范，定义了命名空间与文件系统目录的映射关系。Composer 实现了 PSR-4 标准，是现代 PHP 项目的标准自动加载方式。

## PSR-4 规则

### 基本规则

1. **命名空间前缀**：完全限定类名的顶级命名空间。
2. **基础目录**：命名空间前缀对应的文件系统目录。
3. **子命名空间**：对应子目录。
4. **类名**：对应文件名（不含 `.php` 扩展名）。

```
命名空间：App\Models\User
基础目录：src/
文件路径：src/Models/User.php
```

### 映射示例

```
命名空间：App\Domain\User
基础目录：src/
文件路径：src/Domain/User.php

命名空间：App\Application\Services\UserService
基础目录：src/
文件路径：src/Application/Services/UserService.php
```

## Composer 自动加载配置

### composer.json 配置

在 `composer.json` 中配置：

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/",
            "App\\Tests\\": "tests/"
        }
    }
}
```

### 目录结构示例

```
project/
├── composer.json
├── src/
│   ├── Models/
│   │   └── User.php
│   ├── Services/
│   │   └── UserService.php
│   └── Controllers/
│       └── UserController.php
└── vendor/
    └── autoload.php
```

### 使用 Composer 自动加载

```php
<?php
// 在项目入口文件引入
require __DIR__ . '/vendor/autoload.php';

// 现在可以直接使用命名空间类
use App\Models\User;
use App\Services\UserService;

$user = new User(1, 'Alice');
$service = new UserService();
```

## 生成自动加载文件

### composer dump-autoload

```bash
# 生成或更新自动加载文件
composer dump-autoload

# 开发时使用优化版本（生成类映射，提升性能）
composer dump-autoload --optimize

# 生产环境使用优化版本
composer dump-autoload --optimize --classmap-authoritative
```

## 自动加载优化

### 类映射（Class Map）

- Composer 可以生成类映射文件，提升加载性能。

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        },
        "classmap": [
            "src/Legacy/"
        ]
    }
}
```

### 文件自动加载

- 对于非 PSR-4 标准的文件，可以使用 `files` 自动加载。

```json
{
    "autoload": {
        "files": [
            "src/helpers.php"
        ]
    }
}
```

## 实际项目示例

### 完整的项目结构

```
myapp/
├── composer.json
├── public/
│   └── index.php
├── src/
│   ├── Domain/
│   │   └── User/
│   │       ├── User.php
│   │       └── UserId.php
│   ├── Application/
│   │   └── UserService.php
│   ├── Infrastructure/
│   │   └── Database/
│   │       └── UserRepository.php
│   └── Http/
│       └── Controllers/
│           └── UserController.php
└── tests/
    └── Unit/
        └── UserTest.php
```

### composer.json 配置

```json
{
    "name": "myapp/app",
    "autoload": {
        "psr-4": {
            "App\\": "src/",
            "App\\Tests\\": "tests/"
        }
    },
    "require": {
        "php": "^8.2"
    }
}
```

### 入口文件

```php
<?php
declare(strict_types=1);

require __DIR__ . '/../vendor/autoload.php';

use App\Http\Controllers\UserController;

$controller = new UserController();
// ...
```

## 调试自动加载

### 查看自动加载信息

```php
$loader = require 'vendor/autoload.php';

// 获取类映射
$classMap = $loader->getClassMap();
print_r($classMap);

// 检查类是否存在
if (class_exists('App\Models\User')) {
    echo "User class is autoloadable\n";
}
```

### 常见问题排查

1. **类找不到**：检查命名空间和目录结构是否匹配。
2. **自动加载未注册**：确保 `vendor/autoload.php` 已引入。
3. **路径问题**：使用 `__DIR__` 而非相对路径。

```php
// 错误：相对路径可能不稳定
require 'vendor/autoload.php';

// 正确：使用绝对路径
require __DIR__ . '/vendor/autoload.php';
```

## 命名空间最佳实践

### 目录结构组织

```
src/
├── Domain/              # 领域层
│   ├── User/
│   │   ├── User.php
│   │   └── UserRepository.php
│   └── Product/
├── Application/         # 应用层
│   ├── Services/
│   └── DTOs/
├── Infrastructure/      # 基础设施层
│   ├── Database/
│   └── Cache/
└── Http/                # 表现层
    ├── Controllers/
    └── Middleware/
```

### 命名空间约定

- 使用 `Vendor\Package` 作为顶级命名空间。
- 命名空间与目录结构保持一致。
- 使用 `StudlyCase` 命名命名空间部分。

```php
namespace App\Domain\User;

class User {}

namespace App\Application\Services;

class UserService {}

namespace App\Infrastructure\Database;

class UserRepository {}
```

## 完整示例

```php
<?php
declare(strict_types=1);

// composer.json
// {
//     "autoload": {
//         "psr-4": {
//             "App\\": "src/"
//         }
//     }
// }

// src/Domain/User/User.php
namespace App\Domain\User;

class User
{
    public function __construct(
        public int $id,
        public string $name
    ) {
    }
}

// src/Application/UserService.php
namespace App\Application;

use App\Domain\User\User;

class UserService
{
    public function findUser(int $id): ?User
    {
        // 查找用户逻辑
        return null;
    }
}

// public/index.php
require __DIR__ . '/../vendor/autoload.php';

use App\Application\UserService;

$service = new UserService();
```

## 注意事项

1. **命名空间一致性**：确保命名空间与目录结构完全一致。

2. **PSR-4 标准**：遵循 PSR-4 标准，确保与 Composer 兼容。

3. **自动加载优化**：生产环境使用 `--optimize` 选项提升性能。

4. **类映射**：对于非 PSR-4 标准的代码，使用 `classmap` 自动加载。

5. **文件自动加载**：对于辅助函数文件，使用 `files` 自动加载。

## 练习

1. 创建一个项目结构，使用命名空间 `App\Models`、`App\Services`、`App\Controllers`，配置 Composer PSR-4 自动加载。

2. 设计一个多模块项目结构，每个模块有自己的命名空间（如 `App\Blog`、`App\Shop`），配置相应的自动加载规则。

3. 实现一个自动加载器，支持多个命名空间前缀映射到不同的基础目录，并处理命名空间解析优先级。

4. 配置 Composer 自动加载，使用类映射优化性能。

5. 创建一个完整的项目，演示 PSR-4 自动加载的完整流程。
