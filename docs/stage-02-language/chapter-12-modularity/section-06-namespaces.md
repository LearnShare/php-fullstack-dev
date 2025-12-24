# 2.12.6 命名空间与模块组织

## 概述

命名空间（Namespace）是 PHP 5.3+ 引入的重要特性，用于组织代码和避免名称冲突。在模块化开发中，命名空间是组织和管理代码的关键工具。

## 命名空间的基本概念

### 什么是命名空间

**命名空间**（Namespace）是一个逻辑容器，用于组织相关的代码元素（类、函数、常量）。它类似于文件系统中的目录，可以将代码分组管理。

**类比理解**：
- 文件系统：`/home/user/documents/file.txt`
- 命名空间：`App\Controllers\UserController`

### 为什么需要命名空间

**问题场景**：两个不同的库都定义了 `User` 类

```php
<?php
// 库 A
class User
{
    // ...
}
```

```php
<?php
// 库 B
class User  // 错误：类 User 已定义
{
    // ...
}
```

**解决方案**：使用命名空间

```php
<?php
// 库 A
namespace LibraryA;

class User
{
    // ...
}
```

```php
<?php
// 库 B
namespace LibraryB;

class User  // 可以，因为命名空间不同
{
    // ...
}
```

## 如何定义命名空间

### 基本语法

```php
<?php
declare(strict_types=1);

namespace NamespaceName;

// 类、函数、常量定义
```

### 单层命名空间

```php
<?php
// src/User.php
declare(strict_types=1);

namespace App;

class User
{
    public function __construct(
        private string $name
    ) {
    }
    
    public function getName(): string
    {
        return $this->name;
    }
}
```

### 多层命名空间

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
    public function __construct(
        private string $name
    ) {
    }
}
```

### 全局命名空间

**没有命名空间的代码属于全局命名空间**：

```php
<?php
// functions.php
declare(strict_types=1);

// 全局命名空间
function globalFunction(): void
{
    echo "Global function\n";
}

class GlobalClass
{
    // ...
}
```

**在命名空间中访问全局命名空间**：

```php
<?php
namespace App;

// 使用 \ 前缀访问全局命名空间
\globalFunction();  // 调用全局函数

$obj = new \GlobalClass();  // 使用全局类
```

## 如何使用命名空间

### 完全限定名

**完全限定名**（Fully Qualified Name）包含完整的命名空间路径：

```php
<?php
declare(strict_types=1);

require __DIR__ . '/src/Controllers/UserController.php';
require __DIR__ . '/src/Models/User.php';

// 使用完全限定名
$controller = new \App\Controllers\UserController();
$user = new \App\Models\User('Alice');
```

### use 语句导入

**`use` 语句**用于导入命名空间或类，简化代码：

```php
<?php
declare(strict_types=1);

require __DIR__ . '/src/Controllers/UserController.php';
require __DIR__ . '/src/Models/User.php';

// 导入类
use App\Controllers\UserController;
use App\Models\User;

// 直接使用类名
$controller = new UserController();
$user = new User('Alice');
```

### 导入命名空间

**导入整个命名空间**：

```php
<?php
declare(strict_types=1);

require __DIR__ . '/src/Controllers/UserController.php';
require __DIR__ . '/src/Controllers/ProductController.php';

// 导入命名空间
use App\Controllers;

// 使用命名空间前缀
$userController = new Controllers\UserController();
$productController = new Controllers\ProductController();
```

### 使用别名

**为类或命名空间创建别名**，避免名称冲突或简化长名称：

```php
<?php
declare(strict_types=1);

require __DIR__ . '/src/Controllers/UserController.php';
require __DIR__ . '/src/Models/User.php';

// 使用别名
use App\Controllers\UserController as UC;
use App\Models\User as UserModel;

$controller = new UC();
$user = new UserModel('Alice');
```

**为命名空间创建别名**：

```php
<?php
declare(strict_types=1);

// 导入并创建别名
use App\Controllers as Controllers;
use App\Models as Models;

$controller = new Controllers\UserController();
$user = new Models\User('Alice');
```

## 命名空间在模块化中的应用

### 按功能组织模块

**目录结构**：

```
src/
├── Controllers/
│   └── UserController.php  (namespace App\Controllers)
├── Models/
│   └── User.php  (namespace App\Models)
├── Services/
│   └── UserService.php  (namespace App\Services)
└── Utils/
    └── Helper.php  (namespace App\Utils)
```

**文件内容**：

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
}
```

```php
<?php
// src/Services/UserService.php
declare(strict_types=1);

namespace App\Services;

use App\Models\User;

class UserService
{
    public function getAllUsers(): array
    {
        // ...
        return [];
    }
}
```

### 按模块组织

**目录结构**：

```
src/
├── User/
│   ├── UserController.php  (namespace App\User)
│   ├── UserModel.php  (namespace App\User)
│   └── UserService.php  (namespace App\User)
└── Product/
    ├── ProductController.php  (namespace App\Product)
    └── ProductModel.php  (namespace App\Product)
```

**优势**：
- 相关功能集中在一起
- 模块之间边界清晰
- 便于模块的独立开发和测试

### 第三方库的命名空间

**Composer 包的命名空间通常遵循 PSR-4 标准**：

```
vendor/
└── monolog/
    └── monolog/
        └── src/
            └── Monolog/
                └── Logger.php  (namespace Monolog)
```

**使用第三方库**：

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Monolog\Logger;
use Monolog\Handler\StreamHandler;

$logger = new Logger('name');
$logger->pushHandler(new StreamHandler('path/to/your.log', Logger::WARNING));
```

## 命名空间中的函数和常量

### 函数

**定义命名空间函数**：

```php
<?php
// utils/helpers.php
declare(strict_types=1);

namespace Utils;

function formatDate(string $date): string
{
    return date('Y-m-d', strtotime($date));
}

function formatCurrency(float $amount): string
{
    return '¥' . number_format($amount, 2);
}
```

**使用命名空间函数**：

```php
<?php
declare(strict_types=1);

require __DIR__ . '/utils/helpers.php';

// 使用完全限定名
echo \Utils\formatDate('2024-01-01') . "\n";

// 或使用 use function
use function Utils\formatDate;
use function Utils\formatCurrency;

echo formatDate('2024-01-01') . "\n";
echo formatCurrency(1234.56) . "\n";
```

### 常量

**定义命名空间常量**：

```php
<?php
// config/constants.php
declare(strict_types=1);

namespace Config;

const MAX_USERS = 100;
const DEFAULT_LANGUAGE = 'zh-CN';
```

**使用命名空间常量**：

```php
<?php
declare(strict_types=1);

require __DIR__ . '/config/constants.php';

// 使用完全限定名
echo \Config\MAX_USERS . "\n";

// 或使用 use const
use const Config\MAX_USERS;
use const Config\DEFAULT_LANGUAGE;

echo MAX_USERS . "\n";
echo DEFAULT_LANGUAGE . "\n";
```

## 命名空间的嵌套和子命名空间

### 嵌套命名空间

**命名空间可以嵌套**：

```php
<?php
// src/App/Controllers/Admin/UserController.php
declare(strict_types=1);

namespace App\Controllers\Admin;

class UserController
{
    public function index(): void
    {
        echo "Admin user list\n";
    }
}
```

```php
<?php
// src/App/Controllers/User/UserController.php
declare(strict_types=1);

namespace App\Controllers\User;

class UserController
{
    public function index(): void
    {
        echo "User list\n";
    }
}
```

**使用嵌套命名空间**：

```php
<?php
declare(strict_types=1);

require __DIR__ . '/src/App/Controllers/Admin/UserController.php';
require __DIR__ . '/src/App/Controllers/User/UserController.php';

use App\Controllers\Admin\UserController as AdminUserController;
use App\Controllers\User\UserController as UserUserController;

$adminController = new AdminUserController();
$userController = new UserUserController();
```

## 命名空间与自动加载

### PSR-4 自动加载

**命名空间与文件路径的映射关系**：

```
命名空间：App\Controllers\UserController
文件路径：src/Controllers/UserController.php

规则：
- App\ 对应 src/
- Controllers\ 对应 Controllers/
- UserController 对应 UserController.php
```

**composer.json 配置**：

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

**使用自动加载**：

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

// 自动加载，无需手动 require
use App\Controllers\UserController;
use App\Models\User;

$controller = new UserController();
$user = new User('Alice');
```

## 命名空间的最佳实践

### 1. 命名规范

- **使用大驼峰**：`App\Controllers\UserController`
- **使用有意义的名称**：避免过短或过长的命名空间
- **遵循 PSR-4**：命名空间与目录结构一致

### 2. 组织方式

- **按功能组织**：`App\Controllers`、`App\Models`、`App\Services`
- **按模块组织**：`App\User`、`App\Product`、`App\Order`
- **避免过深嵌套**：通常不超过 3-4 层

### 3. 导入规范

- **按字母顺序排列** `use` 语句
- **使用有意义的别名**：避免过短的别名
- **避免使用通配符导入**：`use App\*;`（PHP 不支持）

### 4. 避免冲突

- **使用唯一的顶级命名空间**：如公司名或项目名
- **第三方库使用其官方命名空间**
- **使用别名解决冲突**

## 完整示例

```php
<?php
// src/Utils/Math.php
declare(strict_types=1);

namespace App\Utils;

class Math
{
    public static function add(int $a, int $b): int
    {
        return $a + $b;
    }
    
    public static function multiply(int $a, int $b): int
    {
        return $a * $b;
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

```php
<?php
// src/Services/UserService.php
declare(strict_types=1);

namespace App\Services;

use App\Models\User;

class UserService
{
    public function createUser(string $name, int $age): User
    {
        return new User($name, $age);
    }
}
```

```php
<?php
// main.php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use App\Utils\Math;
use App\Models\User;
use App\Services\UserService;

// 使用工具类
echo "Sum: " . Math::add(5, 3) . "\n";

// 使用服务
$userService = new UserService();
$user = $userService->createUser('Alice', 25);
echo "User: {$user->getName()}\n";
```

## 注意事项

1. **命名空间声明**：必须在文件的第一行（`<?php` 之后），`declare` 之前

2. **全局命名空间**：使用 `\` 前缀访问全局命名空间

3. **文件路径**：使用 PSR-4 时，文件路径必须与命名空间匹配

4. **大小写敏感**：命名空间和类名都区分大小写

5. **避免循环依赖**：使用命名空间时也要注意避免循环依赖

## 练习

1. 创建一个工具类模块，使用命名空间 `App\Utils`，包含多个静态方法。

2. 实现两个同名的类（如 `User`），使用不同的命名空间避免冲突。

3. 创建一个多层命名空间结构（如 `App\Controllers\Admin`），并正确使用。

4. 编写代码，演示 `use` 语句的不同用法（导入类、导入命名空间、使用别名）。

5. 配置 PSR-4 自动加载，验证命名空间与文件路径的映射关系。

## 相关章节

- **[1.3.2 Composer 自动加载](../../stage-01-foundation/chapter-04-toolchain/section-02-composer-autoload.md)**：深入学习 Composer 的 PSR-4 自动加载机制
- **[2.16.1 PSR 标准](../chapter-17-standards/section-01-psr-standards.md)**：了解 PSR-4 标准的详细规范
