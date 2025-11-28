# 3.5 命名空间与自动加载

## 目标

- 理解命名空间（Namespace）的作用与语法，掌握如何避免类名冲突。
- 熟悉 `use` 语句的使用方式，包括别名、分组导入等。
- 掌握 PSR-4 自动加载标准，理解 Composer 自动加载的工作原理。
- 能够组织大型项目的目录结构，实现模块化的代码组织。

## 命名空间基础

### 为什么需要命名空间

- 解决类名冲突问题。
- 组织代码结构，提高可维护性。
- 支持自动加载机制。

```php
// 没有命名空间时，类名冲突
class User {} // 来自框架
class User {} // 来自第三方库 - 冲突！

// 使用命名空间后
namespace App\Models;
class User {}

namespace Vendor\Library;
class User {}
```

### 命名空间语法

- **语法**：`namespace NamespaceName;`
- 命名空间声明必须在文件顶部（`<?php` 之后，其他代码之前）。
- 命名空间使用反斜杠 `\` 分隔层级。

```php
<?php
declare(strict_types=1);

namespace App\Models;

class User
{
    public function __construct(
        public int $id,
        public string $name
    ) {
    }
}
```

### 子命名空间

- 命名空间可以嵌套，形成层次结构。

```php
namespace App\Http\Controllers;

class UserController
{
    // ...
}

namespace App\Services;

class UserService
{
    // ...
}

namespace App\Repositories;

class UserRepository
{
    // ...
}
```

### 全局命名空间

- 没有命名空间的类属于全局命名空间。
- 使用 `\` 前缀访问全局命名空间。

```php
namespace App\Utils;

class Helper
{
    public function formatDate(\DateTime $date): string
    {
        return $date->format('Y-m-d');
    }

    public function jsonEncode(mixed $data): string
    {
        return \json_encode($data); // 调用全局函数
    }
}
```

## use 语句

### 基础导入

- **语法**：`use Namespace\ClassName;`
- 导入后可以直接使用类名，无需完整命名空间路径。

```php
namespace App\Controllers;

use App\Models\User;
use App\Services\UserService;

class UserController
{
    public function __construct(
        private UserService $userService
    ) {
    }

    public function show(int $id): User
    {
        return $this->userService->find($id);
    }
}
```

### 别名（Alias）

- 使用 `as` 关键字创建别名，解决类名冲突。

```php
namespace App\Services;

use App\Models\User as AppUser;
use Vendor\Library\User as VendorUser;

class UserService
{
    public function createAppUser(): AppUser
    {
        return new AppUser(1, 'Alice');
    }

    public function createVendorUser(): VendorUser
    {
        return new VendorUser();
    }
}
```

### 分组导入（PHP 7.0+）

- 从同一命名空间导入多个类。

```php
use App\Models\{User, Product, Order};
use App\Services\{UserService, ProductService, OrderService};

// 等价于：
use App\Models\User;
use App\Models\Product;
use App\Models\Order;
use App\Services\UserService;
use App\Services\ProductService;
use App\Services\OrderService;
```

### 函数和常量导入（PHP 5.6+）

- 可以导入函数和常量。

```php
namespace App\Utils;

use function Vendor\Library\helperFunction;
use const Vendor\Library\CONSTANT_VALUE;

class Helper
{
    public function doSomething(): void
    {
        helperFunction();
        echo CONSTANT_VALUE;
    }
}
```

## 命名空间解析规则

### 完全限定名称（Fully Qualified Name）

- 以 `\` 开头的类名，从全局命名空间开始解析。

```php
namespace App\Controllers;

$user = new \App\Models\User(1, 'Alice');
```

### 限定名称（Qualified Name）

- 包含命名空间前缀的类名，从当前命名空间开始解析。

```php
namespace App\Http;

use App\Models\User;

$user = new Models\User(1, 'Alice'); // 解析为 App\Models\User
```

### 非限定名称（Unqualified Name）

- 不包含命名空间的类名，从当前命名空间开始解析。

```php
namespace App\Models;

class User {}

namespace App\Controllers;

use App\Models\User;

$user = new User(); // 解析为 App\Models\User
```

## 自动加载基础

### 传统方式的问题

```php
// 需要手动引入每个文件
require_once 'App/Models/User.php';
require_once 'App/Models/Product.php';
require_once 'App/Services/UserService.php';
// ... 更多文件
```

### `spl_autoload_register`

- PHP 提供自动加载机制，当类未定义时触发。

```php
spl_autoload_register(function ($className) {
    // 将命名空间分隔符转换为目录分隔符
    $file = __DIR__ . '/' . str_replace('\\', '/', $className) . '.php';
    
    if (file_exists($file)) {
        require_once $file;
    }
});

// 现在可以直接使用类，无需手动 require
use App\Models\User;
$user = new User(1, 'Alice');
```

### 简单自动加载器示例

```php
class Autoloader
{
    private array $prefixes = [];

    public function addNamespace(string $prefix, string $baseDir): void
    {
        $prefix = trim($prefix, '\\') . '\\';
        $baseDir = rtrim($baseDir, DIRECTORY_SEPARATOR) . '/';
        $this->prefixes[$prefix] = $baseDir;
    }

    public function register(): void
    {
        spl_autoload_register([$this, 'loadClass']);
    }

    public function loadClass(string $class): bool
    {
        $prefix = $class;
        
        while (false !== $pos = strrpos($prefix, '\\')) {
            $prefix = substr($class, 0, $pos + 1);
            $relativeClass = substr($class, $pos + 1);
            
            $mappedFile = $this->loadMappedFile($prefix, $relativeClass);
            if ($mappedFile) {
                return $mappedFile;
            }
            
            $prefix = rtrim($prefix, '\\');
        }
        
        return false;
    }

    private function loadMappedFile(string $prefix, string $relativeClass): bool
    {
        if (!isset($this->prefixes[$prefix])) {
            return false;
        }
        
        $file = $this->prefixes[$prefix]
              . str_replace('\\', '/', $relativeClass)
              . '.php';
        
        if (file_exists($file)) {
            require $file;
            return true;
        }
        
        return false;
    }
}

// 使用
$loader = new Autoloader();
$loader->addNamespace('App', __DIR__ . '/src');
$loader->register();
```

## PSR-4 自动加载标准

### PSR-4 规则

1. **命名空间前缀**：完全限定类名的顶级命名空间。
2. **基础目录**：命名空间前缀对应的文件系统目录。
3. **子命名空间**：对应子目录。
4. **类名**：对应文件名（不含 `.php` 扩展名）。

```
命名空间：App\Models\User
基础目录：src/
文件路径：src/Models/User.php
```

### Composer 自动加载配置

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

### 生成自动加载文件

```bash
# 生成或更新自动加载文件
composer dump-autoload

# 开发时使用优化版本（生成类映射，提升性能）
composer dump-autoload --optimize
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

### 避免命名空间污染

```php
// 不推荐：在全局命名空间定义类
class Helper {}

// 推荐：使用命名空间
namespace App\Utils;

class Helper {}
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

## 练习

1. 创建一个项目结构，使用命名空间 `App\Models`、`App\Services`、`App\Controllers`，配置 Composer PSR-4 自动加载。

2. 实现一个简单的自动加载器，支持命名空间到目录的映射，不使用 Composer。

3. 创建一个 `App\Utils` 命名空间，包含多个工具类（`StringHelper`、`ArrayHelper`、`DateHelper`），使用 `use` 语句导入并使用这些类。

4. 设计一个多模块项目结构，每个模块有自己的命名空间（如 `App\Blog`、`App\Shop`），配置相应的自动加载规则。

5. 创建一个命名空间冲突的场景（两个不同命名空间的同名类），使用别名解决冲突。

6. 实现一个自动加载器，支持多个命名空间前缀映射到不同的基础目录，并处理命名空间解析优先级。
