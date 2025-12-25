# 3.5.2 use 语句

## 概述

`use` 语句用于导入命名空间中的类，简化代码，提高可读性。理解 `use` 语句的各种用法对于编写清晰的代码至关重要。

## 基础导入

### use 语句

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

## 别名（Alias）

### 解决类名冲突

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

### 简化长命名空间

```php
use App\Domain\Entities\User as DomainUser;
use App\Application\DTOs\User as UserDTO;

// 使用别名简化代码
$domainUser = new DomainUser();
$dto = new UserDTO();
```

## 分组导入（PHP 7.0+）

### 从同一命名空间导入多个类

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

## 函数和常量导入（PHP 5.6+）

### 导入函数

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

## 完整示例

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

use App\Models\{User, Product};
use App\Services\UserService as Service;
use App\Repositories\UserRepository;
use function App\Utils\formatDate;
use const App\Config\APP_NAME;

class UserController
{
    public function __construct(
        private Service $userService,
        private UserRepository $userRepository
    ) {
    }

    public function show(int $id): User
    {
        $user = $this->userService->find($id);
        echo formatDate($user->createdAt);
        echo APP_NAME;
        return $user;
    }
}
```

## 注意事项

1. **导入顺序**：通常先导入类，再导入函数，最后导入常量。

2. **别名使用**：只在必要时使用别名，避免过度使用导致代码难以理解。

3. **分组导入**：从同一命名空间导入多个类时，使用分组导入简化代码。

4. **命名冲突**：遇到类名冲突时，使用别名解决。

5. **代码组织**：将 `use` 语句放在命名空间声明之后，类定义之前。

## 练习

1. 创建一个项目，使用 `use` 语句导入多个类，演示基础导入。

2. 实现一个类名冲突的场景，使用别名解决冲突。

3. 使用分组导入从同一命名空间导入多个类。

4. 导入函数和常量，演示函数和常量的导入方式。

5. 创建一个完整的控制器类，使用各种 `use` 语句导入依赖。
