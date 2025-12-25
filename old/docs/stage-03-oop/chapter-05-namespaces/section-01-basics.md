# 3.5.1 命名空间基础

## 概述

命名空间（Namespace）用于解决类名冲突问题，组织代码结构，提高可维护性。理解命名空间是组织大型项目的基础。

## 为什么需要命名空间

### 类名冲突问题

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

### 优势

- 解决类名冲突问题
- 组织代码结构，提高可维护性
- 支持自动加载机制

## 命名空间语法

### 基本语法

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

## 全局命名空间

### 访问全局命名空间

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

## 完整示例

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

namespace App\Services;

use App\Models\User;

class UserService
{
    public function findUser(int $id): ?User
    {
        // 查找用户逻辑
        return null;
    }
}

namespace App\Controllers;

use App\Models\User;
use App\Services\UserService;

class UserController
{
    public function __construct(
        private UserService $userService
    ) {
    }

    public function show(int $id): ?User
    {
        return $this->userService->findUser($id);
    }
}
```

## 注意事项

1. **文件顶部**：命名空间声明必须在文件顶部，`<?php` 之后。

2. **一个文件一个命名空间**：一个文件通常只包含一个命名空间。

3. **全局命名空间**：使用 `\` 前缀访问全局命名空间的类和函数。

4. **命名规范**：使用 `StudlyCase` 命名命名空间部分。

5. **目录结构**：命名空间应该与目录结构保持一致。

## 练习

1. 创建多个命名空间，演示命名空间如何解决类名冲突。

2. 创建一个 `App\Utils` 命名空间，包含工具类，演示全局命名空间的访问。

3. 实现命名空间的三种解析方式（完全限定、限定、非限定）。

4. 创建一个多模块项目，每个模块有自己的命名空间。

5. 演示命名空间与目录结构的关系。
