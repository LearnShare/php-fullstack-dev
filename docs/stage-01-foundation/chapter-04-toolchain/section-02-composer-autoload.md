# 1.3.2 Composer 自动加载

## 概述

Composer 提供了强大的自动加载功能，可以根据命名空间自动加载类文件，无需手动 `require`。本节详细介绍 PSR-4 自动加载、类映射和文件自动加载的配置和使用。

## PSR-4 自动加载

> **标准规范**：关于 PSR-4 标准的详细规范要求，请参考 [2.16.1 PSR 标准](../../stage-02-language/chapter-17-standards/section-01-psr-standards.md)。

### 基本概念

**PSR-4** 是 PHP 标准建议（PHP Standards Recommendation）的自动加载标准。它定义了命名空间与文件路径的映射关系。

### 配置 PSR-4

**composer.json 配置**：

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

**映射规则**：
- `App\` 对应 `src/` 目录
- `App\Controllers\UserController` 对应 `src/Controllers/UserController.php`
- `App\Models\User` 对应 `src/Models/User.php`

### 目录结构示例

```
project/
├── composer.json
├── vendor/
│   └── autoload.php
└── src/
    ├── Controllers/
    │   └── UserController.php  (namespace App\Controllers)
    ├── Models/
    │   └── User.php  (namespace App\Models)
    └── Services/
        └── UserService.php  (namespace App\Services)
```

### 类文件示例

```php
<?php
// src/Controllers/UserController.php
declare(strict_types=1);

namespace App\Controllers;

use App\Models\User;
use App\Services\UserService;

class UserController
{
    public function __construct(
        private UserService $userService
    ) {
    }
    
    public function index(): void
    {
        $users = $this->userService->getAllUsers();
        // ...
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
    public function __construct(
        private string $name,
        private int $age
    ) {
    }
    
    public function getName(): string
    {
        return $this->name;
    }
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

// 自动加载，无需手动 require
$controller = new UserController();
$user = new User('Alice', 25);
```

### 生成自动加载文件

```bash
# 生成自动加载文件
composer dump-autoload

# 优化自动加载（生产环境）
composer dump-autoload --optimize

# 优化并生成类映射（最佳性能）
composer dump-autoload --optimize --classmap-authoritative
```

### PSR-4 规则详解

**规则 1：命名空间前缀必须包含尾部反斜杠**

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"  // ✅ 正确：有尾部反斜杠
        }
    }
}
```

```json
{
    "autoload": {
        "psr-4": {
            "App": "src/"  // ❌ 错误：缺少尾部反斜杠
        }
    }
}
```

**规则 2：目录路径末尾可以有斜杠，也可以没有**

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/",   // ✅ 正确
            "App\\": "src"     // ✅ 也正确
        }
    }
}
```

**规则 3：类名必须与文件名匹配**

```php
<?php
// 命名空间：App\Controllers
// 类名：UserController
// 文件名必须是：UserController.php（区分大小写）
namespace App\Controllers;

class UserController  // 类名必须与文件名匹配
{
    // ...
}
```

### 多个命名空间前缀

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/",
            "App\\Controllers\\": "src/Controllers/",
            "App\\Models\\": "src/Models/",
            "Vendor\\Package\\": "vendor-package/src/"
        }
    }
}
```

**注意**：更具体的命名空间前缀应该放在前面，因为 Composer 会按顺序匹配。

## 类映射（Classmap）自动加载

### 配置类映射

**适用于不符合 PSR-4 的遗留代码**：

```json
{
    "autoload": {
        "classmap": [
            "src/Legacy/",
            "src/Helpers/",
            "src/Singleton.php"
        ]
    }
}
```

### 使用场景

- 遗留代码（不符合 PSR-4）
- 辅助函数类
- 第三方库（不符合 PSR-4）
- 动态生成的类

### 示例

```php
<?php
// src/Legacy/OldClass.php
// 没有命名空间，不符合 PSR-4

class OldClass
{
    public function doSomething(): void
    {
        echo "Legacy class\n";
    }
}
```

```php
<?php
// index.php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

// 自动加载，无需手动 require
$obj = new OldClass();
$obj->doSomething();
```

### 类映射的工作原理

1. Composer 扫描指定目录
2. 查找所有 PHP 文件
3. 解析文件中的类定义
4. 生成类名到文件路径的映射
5. 存储在 `vendor/composer/autoload_classmap.php`

### 优化类映射

```bash
# 生成优化的类映射
composer dump-autoload --classmap-authoritative

# 这会：
# 1. 只使用类映射，不使用 PSR-4
# 2. 如果类不在映射中，直接失败（不尝试 PSR-4）
# 3. 性能最佳，但需要重新生成映射
```

## 文件自动加载

### 配置文件自动加载

**用于自动加载函数和常量文件**：

```json
{
    "autoload": {
        "files": [
            "src/helpers.php",
            "src/functions.php",
            "src/constants.php"
        ]
    }
}
```

### 使用场景

- 全局函数
- 常量定义
- 配置加载
- 初始化代码

### 示例

```php
<?php
// src/helpers.php
declare(strict_types=1);

function formatDate(string $date): string
{
    return date('Y-m-d', strtotime($date));
}

function formatCurrency(float $amount): string
{
    return '¥' . number_format($amount, 2);
}
```

```php
<?php
// index.php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

// 函数自动加载，无需手动 require
echo formatDate('2024-01-01') . "\n";
echo formatCurrency(1234.56) . "\n";
```

### 文件自动加载的时机

**文件会在 `require 'vendor/autoload.php'` 时立即执行**：

```php
<?php
// src/init.php
declare(strict_types=1);

echo "初始化代码执行\n";
$initialized = true;
```

```php
<?php
// index.php
require __DIR__ . '/vendor/autoload.php';
// 输出：初始化代码执行

// $initialized 变量立即可用
```

## 混合使用

### 同时使用多种自动加载方式

```json
{
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

### 加载顺序

1. **files**：首先加载（立即执行）
2. **classmap**：其次加载（生成映射）
3. **psr-4**：最后加载（按需加载）

## 自动加载优化

### 开发环境

```bash
# 标准自动加载（开发环境）
composer dump-autoload

# 特点：
# - 按需加载类
# - 支持新添加的类（无需重新生成）
# - 性能较低
```

### 生产环境

```bash
# 优化自动加载
composer dump-autoload --optimize

# 特点：
# - 生成优化的类映射
# - 性能提升
# - 需要重新生成映射
```

### 最佳性能

```bash
# 权威类映射
composer dump-autoload --optimize --classmap-authoritative

# 特点：
# - 只使用类映射
# - 性能最佳
# - 不支持动态添加类
```

### 性能对比

| 方式 | 性能 | 灵活性 | 适用场景 |
| :--- | :--- | :--- | :--- |
| 标准 | 低 | 高 | 开发环境 |
| 优化 | 中 | 中 | 生产环境 |
| 权威 | 高 | 低 | 生产环境（稳定代码） |

## 调试自动加载

### 查看自动加载信息

```bash
# 显示自动加载的类
composer dump-autoload --verbose

# 显示类映射
cat vendor/composer/autoload_classmap.php
```

### 检查类是否可加载

```php
<?php
require __DIR__ . '/vendor/autoload.php';

// 检查类是否存在
if (class_exists('App\\Controllers\\UserController')) {
    echo "类存在\n";
} else {
    echo "类不存在\n";
}

// 检查函数是否存在
if (function_exists('formatDate')) {
    echo "函数存在\n";
}
```

### 调试自动加载问题

```php
<?php
// 启用自动加载调试
spl_autoload_register(function ($class) {
    echo "尝试加载类: {$class}\n";
    // 返回 false 让其他自动加载器尝试
    return false;
});

require __DIR__ . '/vendor/autoload.php';
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
        "php": "^8.2"
    },
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
// src/helpers.php
declare(strict_types=1);

function helper(): void
{
    echo "Helper function\n";
}
```

```php
<?php
// src/Legacy/OldClass.php
class OldClass
{
    public function doSomething(): void
    {
        echo "Legacy class\n";
    }
}
```

```php
<?php
// src/Controllers/UserController.php
declare(strict_types=1);

namespace App\Controllers;

class UserController
{
    public function index(): void
    {
        echo "User Controller\n";
    }
}
```

```php
<?php
// index.php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

// 使用文件自动加载的函数
helper();

// 使用类映射自动加载的类
$old = new OldClass();
$old->doSomething();

// 使用 PSR-4 自动加载的类
use App\Controllers\UserController;
$controller = new UserController();
$controller->index();
```

## 最佳实践

1. **使用 PSR-4**：新代码使用 PSR-4 自动加载
2. **类映射用于遗留代码**：不符合 PSR-4 的代码使用类映射
3. **文件自动加载谨慎使用**：只在必要时使用
4. **生产环境优化**：使用 `--optimize` 标志
5. **命名空间规范**：遵循 PSR-4 命名规范

## 注意事项

1. **命名空间**：类名必须与命名空间和文件路径匹配
2. **文件命名**：文件名必须与类名相同（区分大小写）
3. **更新自动加载**：添加新类后运行 `composer dump-autoload`
4. **性能优化**：生产环境使用 `--optimize` 标志
5. **版本控制**：将 `vendor/` 目录添加到 `.gitignore`

## 练习

1. 配置 PSR-4 自动加载，创建多个类并验证自动加载是否正常工作。

2. 编写一个辅助函数文件，使用 `files` 自动加载，在主文件中使用这些函数。

3. 创建一个遗留代码适配器，使用 `classmap` 自动加载。

4. 编写脚本，比较优化前后的自动加载性能。

5. 实现一个混合自动加载配置，同时使用 PSR-4、类映射和文件自动加载。

## 相关章节

- **[1.3.1 Composer 基础](section-01-composer-basics.md)**：了解 Composer 的基本使用
- **[1.3.3 编写和发布 Composer 包](section-03-composer-packages.md)**：学习如何创建和发布自己的包
- **[2.16.1 PSR 标准](../../stage-02-language/chapter-17-standards/section-01-psr-standards.md)**：了解 PSR-4 标准的详细规范
