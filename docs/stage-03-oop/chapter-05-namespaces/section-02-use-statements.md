# 3.5.2 use 语句

## 概述

`use` 语句用于导入命名空间中的类、函数和常量，简化代码，避免使用完全限定名。`use` 语句是使用命名空间的关键机制，它允许在当前命名空间中使用其他命名空间的元素，而无需每次都写完整的命名空间路径。

理解 `use` 语句对于编写清晰的代码非常重要。通过 `use` 语句，可以简化代码，提高可读性。`use` 语句支持多种用法：导入类、使用别名、分组导入、导入函数和常量等。

`use` 语句有多种形式，从基础的类导入到高级的分组导入和函数/常量导入。掌握这些用法可以帮助编写更加简洁和可维护的代码。

**主要内容**：
- `use` 语句的基础语法
- 导入类的基础用法
- 别名（Alias）的使用
- 分组导入（PHP 7.0+）
- 导入函数（PHP 5.6+）
- 导入常量（PHP 5.6+）
- `use` 语句的最佳实践

## 特性

- **简化代码**：避免使用完全限定名，提高代码可读性
- **别名支持**：可以给类、函数、常量起别名，解决名称冲突
- **分组导入**：可以一次导入多个类（PHP 7.0+）
- **函数常量导入**：可以导入函数和常量（PHP 5.6+）
- **代码组织**：帮助组织导入语句

## 语法/定义

### 基础导入

**语法**：`use Namespace\ClassName;`

**组成部分**：
- `use` 关键字：导入语句
- `Namespace\ClassName`：完全限定类名
- `;` 分号：语句结束

**特点**：
- 导入后可以直接使用类名
- 无需使用完全限定名
- 通常在命名空间声明之后使用

### 别名（Alias）

**语法**：`use Namespace\ClassName as Alias;`

**组成部分**：
- `as` 关键字：创建别名
- `Alias`：别名名称

**特点**：
- 可以给类起别名
- 用于解决名称冲突
- 用于简化长名称

### 分组导入（PHP 7.0+）

**语法**：`use Namespace\{Class1, Class2, Class3};`

**特点**：
- PHP 7.0+ 引入的特性
- 从同一命名空间导入多个类
- 简化导入语句

### 函数导入（PHP 5.6+）

**语法**：`use function Namespace\functionName;`

**特点**：
- PHP 5.6+ 引入的特性
- 导入命名空间中的函数
- 使用 `use function` 关键字

### 常量导入（PHP 5.6+）

**语法**：`use const Namespace\CONSTANT_NAME;`

**特点**：
- PHP 5.6+ 引入的特性
- 导入命名空间中的常量
- 使用 `use const` 关键字

## 基本用法

### 示例 1：基础类导入

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

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

**说明**：
- 使用 `use` 语句导入类
- 导入后可以直接使用类名 `User` 和 `UserService`
- 无需使用完全限定名 `\App\Models\User`

### 示例 2：使用别名解决冲突

```php
<?php
declare(strict_types=1);

namespace App\Services;

use App\Models\User as AppUser;
use Vendor\Library\User as VendorUser;

class UserService
{
    public function createAppUser(): AppUser
    {
        return new AppUser(1, "John");
    }
    
    public function createVendorUser(): VendorUser
    {
        return new VendorUser();
    }
}
```

**说明**：
- 两个不同的命名空间都有 `User` 类
- 使用 `as` 关键字创建别名
- 通过别名区分不同的类

### 示例 3：使用别名简化长名称

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

use App\Domain\Entities\User as DomainUser;
use App\Application\DTOs\User as UserDTO;
use App\Infrastructure\Repositories\UserRepository as UserRepo;

class UserController
{
    public function show(int $id): DomainUser
    {
        $repo = new UserRepo();
        $domainUser = $repo->find($id);
        $dto = UserDTO::fromDomain($domainUser);
        return $domainUser;
    }
}
```

**说明**：
- 使用别名简化长的命名空间路径
- 提高代码可读性
- 保持代码简洁

### 示例 4：分组导入（PHP 7.0+）

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

// 分组导入：从同一命名空间导入多个类
use App\Models\{User, Product, Order};
use App\Services\{UserService, ProductService, OrderService};

// 等价于：
// use App\Models\User;
// use App\Models\Product;
// use App\Models\Order;
// use App\Services\UserService;
// use App\Services\ProductService;
// use App\Services\OrderService;

class OrderController
{
    public function __construct(
        private UserService $userService,
        private ProductService $productService,
        private OrderService $orderService
    ) {}
    
    public function create(int $userId, int $productId): Order
    {
        $user = $this->userService->findUser($userId);
        $product = $this->productService->findProduct($productId);
        return $this->orderService->createOrder($user, $product);
    }
}
```

**说明**：
- 使用分组导入从同一命名空间导入多个类
- 简化导入语句
- 提高代码可读性

### 示例 5：分组导入与别名结合

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

use App\Models\{User, Product as ProductModel};
use App\Services\{UserService as UService, ProductService};

class Controller
{
    public function __construct(
        private UService $userService,
        private ProductService $productService
    ) {}
    
    public function getProduct(int $id): ProductModel
    {
        return $this->productService->findProduct($id);
    }
}
```

**说明**：
- 分组导入可以与别名结合使用
- 可以给部分类起别名
- 提供灵活的导入方式

### 示例 6：导入函数（PHP 5.6+）

```php
<?php
declare(strict_types=1);

namespace App\Utils;

function formatDate(int $timestamp): string
{
    return date('Y-m-d H:i:s', $timestamp);
}

function formatCurrency(float $amount): string
{
    return '$' . number_format($amount, 2);
}

// 另一个文件
namespace App\Controllers;

use function App\Utils\formatDate;
use function App\Utils\formatCurrency;

class OrderController
{
    public function show(int $id): void
    {
        $order = $this->getOrder($id);
        echo "Date: " . formatDate($order->createdAt) . "\n";
        echo "Total: " . formatCurrency($order->total) . "\n";
    }
}
```

**说明**：
- 使用 `use function` 导入函数
- 导入后可以直接调用函数
- 避免使用完全限定名

### 示例 7：导入常量（PHP 5.6+）

```php
<?php
declare(strict_types=1);

namespace App\Config;

const MAX_LENGTH = 255;
const MIN_LENGTH = 1;
const DEFAULT_TIMEOUT = 30;

// 另一个文件
namespace App\Services;

use const App\Config\MAX_LENGTH;
use const App\Config\MIN_LENGTH;
use const App\Config\DEFAULT_TIMEOUT;

class ValidationService
{
    public function validateLength(string $str): bool
    {
        $length = strlen($str);
        return $length >= MIN_LENGTH && $length <= MAX_LENGTH;
    }
    
    public function getTimeout(): int
    {
        return DEFAULT_TIMEOUT;
    }
}
```

**说明**：
- 使用 `use const` 导入常量
- 导入后可以直接使用常量
- 避免使用完全限定名

### 示例 8：混合导入（类、函数、常量）

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

// 导入类
use App\Models\{User, Product};
use App\Services\UserService;

// 导入函数
use function App\Utils\formatDate;
use function App\Utils\formatCurrency;

// 导入常量
use const App\Config\APP_NAME;
use const App\Config\MAX_ITEMS;

class DashboardController
{
    public function index(): void
    {
        $users = $this->getUsers();
        $products = $this->getProducts();
        
        echo "App: " . APP_NAME . "\n";
        echo "Max items: " . MAX_ITEMS . "\n";
        echo "Date: " . formatDate(time()) . "\n";
        echo "Total: " . formatCurrency(1000.0) . "\n";
    }
}
```

**说明**：
- 可以混合导入类、函数和常量
- 建议按类型分组组织导入语句
- 提高代码可读性

### 示例 9：导入命名空间（命名空间前缀）

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

// 导入命名空间（不是类）
use App\Models;

// 使用命名空间前缀
$user = new Models\User(1, "John");
$product = new Models\Product("Laptop", 999.99);

class UserController
{
    public function show(int $id): Models\User
    {
        return new Models\User($id, "John");
    }
}
```

**说明**：
- 可以导入命名空间本身
- 使用命名空间前缀访问类
- 适用于需要从同一命名空间导入多个类的情况

### 示例 10：命名空间别名

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

// 给命名空间起别名
use App\Domain\Entities as Entities;
use App\Application\DTOs as DTOs;

class UserController
{
    public function show(int $id): Entities\User
    {
        $entity = new Entities\User($id, "John");
        $dto = DTOs\User::fromEntity($entity);
        return $entity;
    }
}
```

**说明**：
- 可以给命名空间起别名
- 使用 `as` 关键字创建命名空间别名
- 简化长的命名空间路径

## 使用场景

### 场景 1：简化代码

使用 `use` 语句简化代码，避免使用完全限定名。

**示例**：

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

use App\Models\User;
use App\Services\UserService;

// 使用 use 语句后，代码更简洁
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

// 不使用 use 语句，需要使用完全限定名
// class UserController
// {
//     public function show(int $id): ?\App\Models\User
//     {
//         $service = new \App\Services\UserService();
//         return $service->findUser($id);
//     }
// }
```

### 场景 2：解决名称冲突

使用别名解决类名冲突。

**示例**：见"示例 2：使用别名解决冲突"

## 注意事项

### use 语句的位置

- **位置要求**：`use` 语句通常在命名空间声明之后
- **顺序要求**：`namespace` -> `use` -> 其他代码
- **建议**：将所有 `use` 语句放在文件顶部

**示例**：

```php
<?php
declare(strict_types=1);

namespace App\Controllers;  // 1. 命名空间声明

use App\Models\User;        // 2. use 语句
use App\Services\UserService;

class UserController        // 3. 类定义
{
    // ...
}
```

### 导入顺序建议

- **建议顺序**：类导入 -> 函数导入 -> 常量导入
- **组织方式**：按字母顺序组织导入
- **分组方式**：按命名空间分组

**示例**：

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

// 类导入（按字母顺序）
use App\Models\Order;
use App\Models\Product;
use App\Models\User;
use App\Services\OrderService;
use App\Services\UserService;

// 函数导入
use function App\Utils\formatDate;
use function App\Utils\formatCurrency;

// 常量导入
use const App\Config\APP_NAME;
```

### 别名使用建议

- **避免过度使用**：只在必要时使用别名
- **有意义的别名**：使用有意义的别名名称
- **文档说明**：如果使用别名，在文档中说明原因

## 常见问题

### 问题 1：类名冲突错误

**错误信息**：`Cannot use App\Models\User as User because the name is already in use`

**原因**：导入的类名与当前命名空间的类名冲突

**解决方案**：使用别名

**示例**：

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

use App\Models\User;  // 错误：如果当前命名空间也有 User 类

// 正确：使用别名
use App\Models\User as UserModel;

class UserController
{
    public function show(int $id): UserModel
    {
        return new UserModel($id, "John");
    }
}
```

### 问题 2：未找到类错误

**错误信息**：`Class 'X' not found`

**原因**：未正确导入类或类不存在

**解决方案**：
- 检查 `use` 语句是否正确
- 检查类是否存在
- 检查命名空间是否正确

### 问题 3：函数调用问题

**症状**：在命名空间中调用导入的函数时出错

**原因**：未使用 `use function` 导入函数

**解决方案**：使用 `use function` 导入函数

**示例**：

```php
<?php
declare(strict_types=1);

namespace App\Utils;

function helper(): void {}

namespace App\Controllers;

// 错误：不能直接 use 函数（需要 use function）
// use App\Utils\helper;

// 正确：使用 use function
use function App\Utils\helper;

helper();  // 正确
```

## 最佳实践

### 1. 按类型组织 use 语句

将类、函数、常量的导入分组组织。

**示例**：

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

// 类导入
use App\Models\User;
use App\Services\UserService;

// 函数导入
use function App\Utils\formatDate;

// 常量导入
use const App\Config\APP_NAME;
```

### 2. 按字母顺序排序

按字母顺序组织导入，便于查找和维护。

**示例**：

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

// 按字母顺序
use App\Models\Order;
use App\Models\Product;
use App\Models\User;
use App\Services\OrderService;
use App\Services\UserService;
```

### 3. 使用分组导入

从同一命名空间导入多个类时，使用分组导入。

**示例**：

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

// 好的做法：使用分组导入
use App\Models\{Order, Product, User};

// 不好的做法：分开导入
// use App\Models\Order;
// use App\Models\Product;
// use App\Models\User;
```

### 4. 合理使用别名

只在必要时使用别名，避免过度使用。

**示例**：

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

// 好的做法：只在必要时使用别名
use App\Models\User as UserModel;  // 解决冲突
use Vendor\VeryLong\Namespace\ClassName as ShortName;  // 简化长名称

// 不好的做法：过度使用别名
// use App\Models\User as U;  // 不清晰
// use App\Services\UserService as US;  // 不清晰
```

### 5. 避免导入未使用的类

只导入实际使用的类，保持代码清晰。

**示例**：

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

// 好的做法：只导入使用的类
use App\Models\User;  // 实际使用
use App\Services\UserService;  // 实际使用

// 不好的做法：导入未使用的类
// use App\Models\Product;  // 未使用
// use App\Models\Order;    // 未使用
```

## 对比分析

### use 语句 vs 完全限定名

| 特性         | use 语句                           | 完全限定名                        |
|:-------------|:--------------------------------|:--------------------------------|
| **代码长度**  | 较短（导入后直接使用类名）          | 较长（每次使用完整路径）           |
| **可读性**    | ✅ 更好                           | ⚠️ 较差                           |
| **维护性**    | ✅ 更容易维护                      | ⚠️ 较难维护                        |
| **性能**      | 相同（编译后等价）                 | 相同                              |
| **适用场景**  | 频繁使用的类                       | 偶尔使用的类                      |

### 分组导入 vs 分开导入

| 特性         | 分组导入（PHP 7.0+）               | 分开导入                          |
|:-------------|:--------------------------------|:--------------------------------|
| **代码量**    | 更少                            | 更多                            |
| **可读性**    | ✅ 更好（从同一命名空间）           | ⚠️ 较差（重复命名空间）            |
| **维护性**    | ✅ 更容易维护                      | ⚠️ 较难维护                        |
| **版本要求**  | PHP 7.0+                        | 所有 PHP 版本                    |

## 练习任务

1. **基础导入练习**：创建一个控制器类，使用 `use` 语句导入多个模型和服务类。

2. **别名练习**：创建两个不同命名空间的同名类，使用别名解决冲突。

3. **分组导入练习**：从同一命名空间导入多个类，使用分组导入语法。

4. **函数常量导入**：创建命名空间中的函数和常量，使用 `use function` 和 `use const` 导入它们。

5. **完整示例**：创建一个完整的控制器类，使用各种 `use` 语句导入类、函数和常量。

## 相关章节

- **[3.5.1 命名空间基础](section-01-basics.md)**：了解命名空间的基础知识
- **[3.5.3 自动加载基础](section-03-autoloading-basics.md)**：了解自动加载机制
- **[3.5.4 Composer 自动加载与 PSR-4](section-04-composer-psr4.md)**：了解 PSR-4 标准
