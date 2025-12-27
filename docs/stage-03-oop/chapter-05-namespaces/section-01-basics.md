# 3.5.1 命名空间基础

## 概述

命名空间（Namespace）是 PHP 5.3+ 引入的特性，用于解决类名冲突问题和组织代码结构。命名空间提供了逻辑分组机制，可以将相关的类、函数和常量组织在一起，形成一个逻辑容器。

理解命名空间对于组织大型项目至关重要。命名空间类似于文件系统中的目录，可以将代码分组管理，避免名称冲突。通过命名空间，可以在不同的命名空间中定义同名的类，而不会产生冲突。

命名空间是现代 PHP 项目的基础，几乎所有的 PHP 框架和库都使用命名空间来组织代码。掌握命名空间是理解自动加载、PSR-4 标准以及现代 PHP 项目结构的前提。

**主要内容**：
- 为什么需要命名空间
- 命名空间的语法和定义
- 子命名空间的定义和使用
- 全局命名空间的访问
- 命名空间解析规则（完全限定名、限定名、非限定名）
- 命名空间与目录结构的关系
- 命名空间的最佳实践

## 特性

- **避免冲突**：解决类名、函数名、常量名冲突问题
- **代码组织**：按功能和模块组织代码
- **模块化**：支持模块化开发
- **逻辑分组**：将相关代码元素组织在一起
- **自动加载支持**：为自动加载机制提供基础

## 语法/定义

### 命名空间定义

**语法**：`namespace NamespaceName;`

**组成部分**：
- `namespace` 关键字：声明命名空间
- `NamespaceName`：命名空间名称
- `;` 分号：语句结束

**特点**：
- 命名空间声明必须在文件顶部，`<?php` 之后
- 一个文件通常只包含一个命名空间
- 命名空间名称使用反斜杠 `\` 分隔层级
- 遵循 `StudlyCase` 命名规范

### 子命名空间

**语法**：`namespace ParentNamespace\ChildNamespace;`

**特点**：
- 使用反斜杠 `\` 分隔命名空间层级
- 可以有多层嵌套
- 子命名空间是独立的命名空间

### 全局命名空间

**语法**：`\ClassName` 或 `\functionName()`

**特点**：
- 没有命名空间的代码属于全局命名空间
- 使用 `\` 前缀访问全局命名空间的元素
- 全局命名空间是默认命名空间

## 基本用法

### 示例 1：为什么需要命名空间（类名冲突问题）

```php
<?php
declare(strict_types=1);

// 问题场景：没有命名空间时，类名冲突

// 文件 A：框架定义的 User 类
class User
{
    public function __construct(
        private string $name
    ) {}
}

// 文件 B：第三方库定义的 User 类
// class User  // 错误：Cannot redeclare class User
// {
//     public function __construct(
//         private int $id
//     ) {}
// }

// 解决方案：使用命名空间
namespace App\Models;

class User
{
    public function __construct(
        private string $name
    ) {}
}

namespace Vendor\Library;

class User  // 可以，因为命名空间不同
{
    public function __construct(
        private int $id
    ) {}
}
```

**说明**：
- 没有命名空间时，同名类会产生冲突
- 使用命名空间后，不同命名空间的同名类不会冲突
- 命名空间解决了类名冲突问题

### 示例 2：基础命名空间使用

```php
<?php
declare(strict_types=1);

namespace App\Models;

class User
{
    public function __construct(
        private int $id,
        private string $name
    ) {}
    
    public function getId(): int
    {
        return $this->id;
    }
    
    public function getName(): string
    {
        return $this->name;
    }
}

// 使用完全限定名
$user = new \App\Models\User(1, "John");
echo "User: {$user->getName()}\n";
```

**输出**：

```
User: John
```

**说明**：
- 使用 `namespace App\Models;` 声明命名空间
- 类 `User` 属于 `App\Models` 命名空间
- 使用完全限定名 `\App\Models\User` 访问类

### 示例 3：子命名空间

```php
<?php
declare(strict_types=1);

namespace App\Http\Controllers;

class UserController
{
    public function index(): array
    {
        return [];
    }
}

// 另一个文件
namespace App\Services;

class UserService
{
    public function findUser(int $id): ?\App\Models\User
    {
        // 查找用户逻辑
        return null;
    }
}

// 另一个文件
namespace App\Repositories;

class UserRepository
{
    public function save(\App\Models\User $user): void
    {
        // 保存用户逻辑
    }
}
```

**说明**：
- `App\Http\Controllers` 是 `App\Http` 的子命名空间
- `App\Services` 和 `App\Repositories` 是 `App` 的子命名空间
- 不同子命名空间的类可以同名

### 示例 4：全局命名空间访问

```php
<?php
declare(strict_types=1);

namespace App\Utils;

class Helper
{
    // 使用全局命名空间的 DateTime 类
    public function formatDate(\DateTime $date): string
    {
        return $date->format('Y-m-d');
    }
    
    // 调用全局函数
    public function jsonEncode(mixed $data): string
    {
        return \json_encode($data);  // 使用 \ 前缀调用全局函数
    }
    
    // 访问全局常量
    public function getPi(): float
    {
        return \M_PI;  // 使用 \ 前缀访问全局常量
    }
}

$helper = new Helper();
$date = new \DateTime();  // 使用全局命名空间的 DateTime
echo $helper->formatDate($date) . "\n";
echo $helper->jsonEncode(['key' => 'value']) . "\n";
```

**输出**：

```
2025-01-15
{"key":"value"}
```

**说明**：
- 使用 `\` 前缀访问全局命名空间的类、函数、常量
- `\DateTime` 表示全局命名空间的 `DateTime` 类
- `\json_encode()` 表示调用全局函数
- `\M_PI` 表示访问全局常量

### 示例 5：命名空间解析规则（完全限定名）

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

// 完全限定名：以 \ 开头，从全局命名空间开始解析
$user = new \App\Models\User(1, "John");  // 解析为全局命名空间的 App\Models\User

class UserController
{
    public function show(): void
    {
        // 使用完全限定名
        $user = new \App\Models\User(1, "John");
        echo "User ID: {$user->getId()}\n";
    }
}
```

**说明**：
- 完全限定名（Fully Qualified Name）以 `\` 开头
- 从全局命名空间开始解析
- 不受当前命名空间影响

### 示例 6：命名空间解析规则（限定名和非限定名）

```php
<?php
declare(strict_types=1);

namespace App\Http;

// 限定名：包含命名空间前缀，从当前命名空间开始解析
use App\Models\User;

// 在 App\Http 命名空间中
// Models\User 会被解析为 App\Http\Models\User（如果存在）
// 但这里使用了 use 语句，所以解析为 App\Models\User

$user = new Models\User(1, "John");  // 限定名

// 非限定名：不包含命名空间，从当前命名空间开始解析
namespace App\Models;

class User
{
    public function __construct(
        private int $id,
        private string $name
    ) {}
}

// 在 App\Models 命名空间中
$user = new User(1, "John");  // 非限定名，解析为 App\Models\User
```

**说明**：
- **限定名**（Qualified Name）：包含命名空间前缀，从当前命名空间开始解析
- **非限定名**（Unqualified Name）：不包含命名空间，从当前命名空间开始解析
- 使用 `use` 语句可以改变解析行为

### 示例 7：命名空间中的类、函数、常量

```php
<?php
declare(strict_types=1);

namespace App\Utils;

// 命名空间中的类
class StringHelper
{
    public static function capitalize(string $str): string
    {
        return ucfirst(strtolower($str));
    }
}

// 命名空间中的函数
function formatCurrency(float $amount): string
{
    return '$' . number_format($amount, 2);
}

// 命名空间中的常量
const MAX_LENGTH = 255;

// 使用命名空间中的元素
$helper = new StringHelper();
echo StringHelper::capitalize("hello") . "\n";  // Hello

echo formatCurrency(1234.56) . "\n";  // $1,234.56

echo "Max length: " . MAX_LENGTH . "\n";  // Max length: 255
```

**输出**：

```
Hello
$1,234.56
Max length: 255
```

**说明**：
- 命名空间可以包含类、函数、常量
- 它们都属于该命名空间
- 在命名空间内部可以直接使用

### 示例 8：命名空间与目录结构

```php
<?php
// 文件：src/App/Models/User.php
declare(strict_types=1);

namespace App\Models;

class User
{
    // ...
}
```

```php
<?php
// 文件：src/App/Http/Controllers/UserController.php
declare(strict_types=1);

namespace App\Http\Controllers;

class UserController
{
    // ...
}
```

**说明**：
- 命名空间通常与目录结构对应
- `App\Models\User` 对应 `src/App/Models/User.php`
- `App\Http\Controllers\UserController` 对应 `src/App/Http/Controllers/UserController.php`
- 这种对应关系是 PSR-4 自动加载标准的基础

## 使用场景

### 场景 1：组织大型项目

命名空间用于组织大型项目的代码结构。

**示例**：MVC 架构

```php
<?php
// App\Models\User.php
namespace App\Models;

class User {}

// App\Controllers\UserController.php
namespace App\Controllers;

class UserController {}

// App\Services\UserService.php
namespace App\Services;

class UserService {}
```

### 场景 2：第三方库集成

命名空间用于避免第三方库的类名冲突。

**示例**：

```php
<?php
// 应用代码
namespace App\Models;
class User {}

// 第三方库
namespace Vendor\Package\Models;
class User {}  // 不会冲突

// 使用
$appUser = new \App\Models\User();
$vendorUser = new \Vendor\Package\Models\User();
```

## 注意事项

### 命名空间声明位置

- **必须位置**：命名空间声明必须在文件顶部，`<?php` 之后
- **唯一声明**：一个文件通常只有一个命名空间声明
- **顺序要求**：`<?php` -> `declare(strict_types=1);` -> `namespace ...;`

**示例**：

```php
<?php
declare(strict_types=1);  // declare 必须在 namespace 之前

namespace App\Models;      // 命名空间声明

class User {}             // 其他代码
```

### 命名空间与目录结构

- **建议对应**：命名空间应该与目录结构对应
- **PSR-4 标准**：遵循 PSR-4 自动加载标准
- **清晰组织**：有助于代码组织和自动加载

### 完全限定名的使用

- **明确性**：使用完全限定名可以避免歧义
- **性能**：完全限定名解析速度最快
- **可读性**：完全限定名可能较长，影响可读性

## 常见问题

### 问题 1：命名空间声明位置错误

**错误信息**：`Fatal error: Namespace declaration statement has to be the very first statement`

**原因**：命名空间声明不在文件顶部

**解决方案**：
- 确保命名空间声明在 `<?php` 之后
- 确保 `declare(strict_types=1);` 在命名空间之前

**示例**：

```php
<?php
// 正确：命名空间在顶部
declare(strict_types=1);
namespace App\Models;

// 错误：命名空间不在顶部
// echo "Something";
// namespace App\Models;  // 错误
```

### 问题 2：类名未找到

**错误信息**：`Class 'X' not found`

**原因**：未正确使用命名空间或未导入类

**解决方案**：
- 使用完全限定名
- 使用 `use` 语句导入类
- 检查命名空间是否正确

**示例**：

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

// 错误：类未找到
// $user = new User();  // 错误：App\Controllers\User 不存在

// 正确：使用完全限定名
$user = new \App\Models\User(1, "John");

// 或使用 use 语句
use App\Models\User;
$user = new User(1, "John");
```

### 问题 3：全局函数调用问题

**症状**：在命名空间中调用全局函数时出错

**原因**：PHP 会先在当前命名空间中查找函数

**解决方案**：使用 `\` 前缀调用全局函数

**示例**：

```php
<?php
declare(strict_types=1);

namespace App;

// 如果当前命名空间有同名函数，会调用当前命名空间的函数
function strlen(string $str): int
{
    return 999;  // 自定义实现
}

// 使用 \ 前缀调用全局函数
$length = \strlen("hello");  // 调用全局 strlen
echo "Length: {$length}\n";  // Length: 5
```

## 最佳实践

### 1. 命名空间与目录结构对应

命名空间应该与目录结构保持一致。

**示例**：

```
src/
  App/
    Models/
      User.php          -> namespace App\Models;
    Controllers/
      UserController.php -> namespace App\Controllers;
```

### 2. 使用有意义的命名空间名称

命名空间名称应该清晰描述其用途。

**示例**：

```php
<?php
// 好的命名
namespace App\Models;
namespace App\Http\Controllers;
namespace App\Services;

// 不好的命名
namespace A;
namespace App\C;
namespace MyApp\Stuff;
```

### 3. 遵循 PSR-4 标准

遵循 PSR-4 自动加载标准，命名空间与目录结构严格对应。

**示例**：

```
src/                              -> 命名空间前缀：App
  App/
    Models/User.php               -> namespace App\Models;
    Controllers/UserController.php -> namespace App\Controllers;
```

### 4. 使用 use 语句简化代码

使用 `use` 语句导入类，避免使用完全限定名。

**示例**：

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

// 使用 use 语句导入
use App\Models\User;
use App\Services\UserService;

class UserController
{
    public function __construct(
        private UserService $userService
    ) {}
    
    public function show(int $id): ?User
    {
        return $this->userService->findUser($id);
    }
}
```

### 5. 明确使用全局命名空间

当需要访问全局命名空间的元素时，使用 `\` 前缀明确表示。

**示例**：

```php
<?php
declare(strict_types=1);

namespace App;

// 明确使用全局函数
$json = \json_encode($data);
$array = \json_decode($json, true);

// 明确使用全局类
$date = new \DateTime();
```

## 对比分析

### 命名空间 vs 没有命名空间

| 特性         | 使用命名空间                       | 不使用命名空间                     |
|:-------------|:--------------------------------|:--------------------------------|
| **类名冲突**  | ✅ 可以避免                       | ❌ 容易冲突                       |
| **代码组织**  | ✅ 按功能组织                     | ❌ 难以组织                       |
| **自动加载**  | ✅ 支持标准自动加载                | ❌ 需要手动引入                   |
| **可维护性**  | ✅ 易于维护                       | ❌ 难以维护                       |
| **适用场景**  | 大型项目、库、框架                 | 简单脚本                         |

### 完全限定名 vs 非限定名 vs 限定名

| 类型         | 语法示例                        | 解析方式                         |
|:-------------|:-------------------------------|:--------------------------------|
| **完全限定名** | `\App\Models\User`              | 从全局命名空间开始                |
| **限定名**    | `Models\User`（当前在 App）     | 从当前命名空间开始                |
| **非限定名**  | `User`（当前在 App\Models）     | 从当前命名空间开始                |

## 练习任务

1. **创建命名空间结构**：创建 `App\Models`、`App\Controllers`、`App\Services` 三个命名空间，每个命名空间包含相应的类。

2. **演示类名冲突解决**：在两个不同的命名空间中创建同名类，演示命名空间如何解决冲突。

3. **全局命名空间访问**：在一个命名空间中访问全局命名空间的类、函数和常量。

4. **命名空间解析规则**：演示完全限定名、限定名和非限定名的解析方式。

5. **命名空间与目录结构**：创建一个项目，命名空间与目录结构对应，理解它们的关系。

## 相关章节

- **[3.5.2 use 语句](section-02-use-statements.md)**：了解如何使用 use 语句导入命名空间
- **[3.5.3 自动加载基础](section-03-autoloading-basics.md)**：了解自动加载机制
- **[3.5.4 Composer 自动加载与 PSR-4](section-04-composer-psr4.md)**：了解 PSR-4 标准
