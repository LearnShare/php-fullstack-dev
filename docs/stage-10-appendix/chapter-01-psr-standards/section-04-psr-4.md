# 10.1.4 PSR-4 自动加载标准

## 概述

PSR-4（PHP Standards Recommendation 4）定义了自动加载的标准，是现代 PHP 项目的基础。它替代了已废弃的 PSR-0，提供了更简洁和高效的自动加载机制。

## 官方文档

- **标准名称**：Autoloader
- **状态**：已接受（Accepted）
- **版本**：1.0.0
- **官方链接**：https://www.php-fig.org/psr/psr-4/

## 核心要求

### 1. 完全限定的类名

完全限定的类名格式：

```
\<NamespaceName>(\<SubNamespaceNames>)*\<ClassName>
```

**要求**：
- 完全限定的类名必须有一个顶级命名空间（"Vendor namespace"）
- 完全限定的类名可以有多个子命名空间
- 完全限定的类名必须以类名结束
- 下划线在完全限定的类名中没有任何特殊含义
- 字母大小写在自动加载时是大小写敏感的

**示例**：

```php
<?php
// 完全限定的类名
\App\Http\Controllers\UserController
\App\Services\UserService
\Vendor\Package\SubPackage\ClassName
```

### 2. 命名空间前缀

命名空间前缀与基础目录的映射关系：

- 完全限定的命名空间前缀必须有一个对应的"基础目录"
- 命名空间前缀后的子命名空间名称对应"基础目录"下的子目录，命名空间分隔符 `\` 对应目录分隔符 `/`
- 类名对应以 `.php` 结尾的文件名

**映射规则**：

```
命名空间前缀: App\Http\Controllers
基础目录: /path/to/project/src
类名: UserController

完整路径: /path/to/project/src/Http/Controllers/UserController.php
```

### 3. 实现要求

#### 3.1 自动加载器实现

自动加载器实现必须：

- 抛出异常（实现 `\Throwable`），不能触发任何级别的错误
- 抛出异常时不能返回值

**示例实现**：

```php
<?php
spl_autoload_register(function ($class) {
    // 命名空间前缀
    $prefix = 'App\\';
    
    // 基础目录
    $baseDir = __DIR__ . '/src/';
    
    // 检查类是否使用命名空间前缀
    $len = strlen($prefix);
    if (strncmp($prefix, $class, $len) !== 0) {
        return;
    }
    
    // 获取相对类名
    $relativeClass = substr($class, $len);
    
    // 将命名空间分隔符替换为目录分隔符
    $file = $baseDir . str_replace('\\', '/', $relativeClass) . '.php';
    
    // 如果文件存在，加载它
    if (file_exists($file)) {
        require $file;
    }
});
```

## Composer 实现

### 1. composer.json 配置

在 `composer.json` 中配置自动加载：

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

**配置说明**：
- `"App\\"`：命名空间前缀
- `"src/"`：对应的基础目录
- 多个命名空间前缀可以映射到不同的目录

### 2. 生成自动加载文件

```bash
# 生成自动加载文件
composer dump-autoload

# 或使用 -o 优化（生产环境）
composer dump-autoload -o
```

### 3. 使用自动加载

在项目入口文件中引入：

```php
<?php
require 'vendor/autoload.php';

// 现在可以直接使用类
use App\Http\Controllers\UserController;
$controller = new UserController();
```

## 实际应用示例

### 示例 1：基础项目结构

**目录结构**：

```
project/
├── composer.json
├── src/
│   ├── Http/
│   │   └── Controllers/
│   │       └── UserController.php
│   ├── Services/
│   │   └── UserService.php
│   └── Models/
│       └── User.php
└── vendor/
    └── autoload.php
```

**composer.json**：

```json
{
    "name": "mycompany/myproject",
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

**UserController.php**：

```php
<?php
declare(strict_types=1);

namespace App\Http\Controllers;

use App\Services\UserService;
use App\Models\User;

class UserController
{
    public function __construct(
        private UserService $userService
    ) {}
    
    public function show(int $id): User
    {
        return $this->userService->getUserById($id);
    }
}
```

### 示例 2：多命名空间前缀

**目录结构**：

```
project/
├── composer.json
├── src/
│   └── App/
│       └── Services/
│           └── UserService.php
├── tests/
│   └── App/
│       └── Tests/
│           └── UserServiceTest.php
└── vendor/
```

**composer.json**：

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/App/",
            "App\\Tests\\": "tests/App/Tests/"
        }
    }
}
```

### 示例 3：第三方包

**包结构**：

```
mypackage/
├── composer.json
├── src/
│   └── MyPackage/
│       └── Service.php
└── README.md
```

**composer.json**：

```json
{
    "name": "vendor/mypackage",
    "autoload": {
        "psr-4": {
            "Vendor\\MyPackage\\": "src/MyPackage/"
        }
    }
}
```

**在其他项目中使用**：

```bash
composer require vendor/mypackage
```

```php
<?php
require 'vendor/autoload.php';

use Vendor\MyPackage\Service;
$service = new Service();
```

## 最佳实践

### 1. 命名空间与目录结构一致

**推荐**：

```
src/
└── App/
    ├── Http/
    │   └── Controllers/
    │       └── UserController.php
    └── Services/
        └── UserService.php
```

命名空间：
- `App\Http\Controllers\UserController`
- `App\Services\UserService`

### 2. 使用有意义的命名空间前缀

**推荐**：

```json
{
    "autoload": {
        "psr-4": {
            "MyCompany\\MyProject\\": "src/"
        }
    }
}
```

**不推荐**：

```json
{
    "autoload": {
        "psr-4": {
            "A\\": "src/"
        }
    }
}
```

### 3. 避免过深的目录结构

**推荐**：3-4 层

```
App\Http\Controllers\UserController
```

**不推荐**：过深

```
App\Http\Controllers\Admin\Users\Management\UserController
```

### 4. 测试代码的自动加载

**composer.json**：

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "App\\Tests\\": "tests/"
        }
    }
}
```

## 常见错误与修正

### 错误 1：命名空间与目录不匹配

**错误**：

```php
<?php
// 文件位置: src/UserController.php
namespace App\Http\Controllers;

class UserController
{
    // ...
}
```

**修正**：

```php
<?php
// 文件位置: src/Http/Controllers/UserController.php
namespace App\Http\Controllers;

class UserController
{
    // ...
}
```

### 错误 2：composer.json 配置错误

**错误**：

```json
{
    "autoload": {
        "psr-4": {
            "App": "src/"
        }
    }
}
```

**修正**：

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

注意：命名空间前缀必须以 `\\` 结尾。

### 错误 3：类名与文件名不匹配

**错误**：

```php
<?php
// 文件名: usercontroller.php
namespace App\Http\Controllers;

class UserController
{
    // ...
}
```

**修正**：

```php
<?php
// 文件名: UserController.php
namespace App\Http\Controllers;

class UserController
{
    // ...
}
```

注意：类名必须与文件名完全匹配（大小写敏感）。

## 与 PSR-0 的区别

### PSR-0（已废弃）

- 下划线有特殊含义（对应目录分隔符）
- 命名空间前缀不需要以 `\` 结尾
- 目录结构更复杂

**示例**：

```
类名: Vendor_Package_ClassName
文件: vendor/package/class_name.php
```

### PSR-4（当前标准）

- 下划线没有特殊含义
- 命名空间前缀必须以 `\` 结尾
- 目录结构更简洁

**示例**：

```
类名: Vendor\Package\ClassName
文件: vendor/package/ClassName.php
```

## 性能优化

### 1. 使用优化的自动加载

```bash
composer dump-autoload -o
```

这会生成类映射文件，提高自动加载性能。

### 2. 生产环境优化

```bash
composer install --no-dev --optimize-autoloader
```

这会：
- 移除开发依赖
- 优化自动加载器
- 生成类映射文件

## 练习

1. 创建一个符合 PSR-4 的项目结构，包含多个命名空间。

2. 配置 `composer.json` 的自动加载，并测试类加载。

3. 创建一个第三方包，配置 PSR-4 自动加载，并在其他项目中使用。

4. 对比 PSR-0 和 PSR-4 的区别，理解为什么 PSR-4 更好。

## 总结

PSR-4 是现代 PHP 项目的基础，它提供了：

- 简洁的自动加载机制
- 清晰的命名空间与目录映射关系
- 高效的类加载性能
- 与 Composer 的完美集成
- 跨框架和库的互操作性

遵循 PSR-4 标准，可以：

- 提高项目的可维护性
- 简化类的组织和管理
- 提高自动加载的性能
- 使代码更容易被其他开发者理解和使用
