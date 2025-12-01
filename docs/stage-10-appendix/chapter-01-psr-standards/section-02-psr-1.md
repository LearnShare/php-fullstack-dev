# 10.1.2 PSR-1 基础编码标准

## 概述

PSR-1（PHP Standards Recommendation 1）是 PHP-FIG 制定的基础编码标准，定义了 PHP 代码的基本规范。所有其他 PSR 标准都基于 PSR-1。

## 官方文档

- **标准名称**：Basic Coding Standard
- **状态**：已接受（Accepted）
- **版本**：1.0.0
- **官方链接**：https://www.php-fig.org/psr/psr-1/

## 核心要求

### 1. 文件

#### 1.1 PHP 标签

PHP 代码必须使用 `<?php ?>` 标签或 `<?=` 短输出标签；不得使用其他标签变体。

**正确示例**：

```php
<?php
echo "Hello, World!";
```

```php
<?= "Hello, World!" ?>
```

**错误示例**：

```php
<?
// 错误：短标签（需要 short_open_tag=On）
echo "Hello";
```

```php
<%
// 错误：ASP 风格标签（已废弃）
echo "Hello";
%>
```

#### 1.2 字符编码

PHP 代码必须使用 UTF-8 编码（无 BOM）。

**检查方法**：

```bash
# Linux/macOS
file -I file.php
# 输出：file.php: text/plain; charset=utf-8

# 检查 BOM
head -c 3 file.php | od -An -tx1
# 如果输出包含 ef bb bf，说明有 BOM
```

#### 1.3 副作用

一个文件应该只声明符号（类、函数、常量等）或产生副作用（输出、修改 ini 设置等），不应同时做两件事。

**正确示例**：

```php
<?php
declare(strict_types=1);

namespace App\Services;

// 只声明类
class UserService
{
    public function getUser(int $id): ?User
    {
        // ...
    }
}
```

```php
<?php
// 只产生副作用（引导文件）
require 'vendor/autoload.php';

$app = new Application();
$app->run();
```

**错误示例**：

```php
<?php
// 错误：同时声明类和产生副作用
class UserService
{
    // ...
}

// 产生副作用
echo "UserService loaded";
```

### 2. 命名空间和类名

#### 2.1 命名空间

命名空间和类必须遵循自动加载标准（PSR-4）。

**要求**：
- 完全限定的类名格式：`\<NamespaceName>(\<SubNamespaceNames>)*\<ClassName>`
- 完全限定的类名必须有一个顶级命名空间（"Vendor namespace"）
- 完全限定的类名可以有多个子命名空间
- 命名空间前缀与基础目录一一对应

**示例**：

```php
<?php
declare(strict_types=1);

namespace App\Http\Controllers;

class UserController
{
    // ...
}
```

#### 2.2 类名

类名必须使用 `StudlyCaps`（大驼峰）命名。

**正确示例**：

```php
<?php
declare(strict_types=1);

namespace App\Models;

class UserProfile
{
    // ...
}

class OrderItem
{
    // ...
}
```

**错误示例**：

```php
<?php
// 错误：使用下划线
class user_profile
{
    // ...
}

// 错误：使用小写
class user
{
    // ...
}
```

### 3. 类常量、属性和方法

#### 3.1 类常量

类常量必须使用全大写字母，单词间用下划线分隔。

**正确示例**：

```php
<?php
declare(strict_types=1);

class Status
{
    public const PENDING = 'pending';
    public const APPROVED = 'approved';
    public const REJECTED = 'rejected';
    
    public const MAX_RETRY_COUNT = 3;
    public const DEFAULT_TIMEOUT = 30;
}
```

**错误示例**：

```php
<?php
// 错误：使用小驼峰
class Status
{
    public const pending = 'pending';
    public const maxRetryCount = 3;
}

// 错误：使用大驼峰
class Status
{
    public const Pending = 'pending';
    public const MaxRetryCount = 3;
}
```

#### 3.2 属性名

本规范对属性名不做强制要求，但建议使用一致的命名风格。

**推荐风格**：

```php
<?php
declare(strict_types=1);

class User
{
    // 小驼峰（推荐）
    private string $firstName;
    private string $lastName;
    
    // 或下划线前缀（也常见）
    private string $_firstName;
    private string $_lastName;
}
```

#### 3.3 方法名

方法名必须使用 `camelCase`（小驼峰）命名。

**正确示例**：

```php
<?php
declare(strict_types=1);

class UserService
{
    public function getUserById(int $id): ?User
    {
        // ...
    }
    
    public function createUser(array $data): User
    {
        // ...
    }
    
    public function updateUserProfile(int $userId, array $profile): void
    {
        // ...
    }
}
```

**错误示例**：

```php
<?php
// 错误：使用下划线
class UserService
{
    public function get_user_by_id(int $id): ?User
    {
        // ...
    }
}

// 错误：使用大驼峰
class UserService
{
    public function GetUserById(int $id): ?User
    {
        // ...
    }
}
```

## 完整示例

### 符合 PSR-1 的类示例

```php
<?php
declare(strict_types=1);

namespace App\Services;

use App\Models\User;
use App\Repositories\UserRepository;

/**
 * 用户服务类
 */
class UserService
{
    // 类常量
    public const DEFAULT_PAGE_SIZE = 20;
    public const MAX_LOGIN_ATTEMPTS = 5;
    
    // 属性
    private UserRepository $repository;
    
    // 构造函数
    public function __construct(UserRepository $repository)
    {
        $this->repository = $repository;
    }
    
    // 方法
    public function getUserById(int $id): ?User
    {
        return $this->repository->find($id);
    }
    
    public function createUser(array $data): User
    {
        // 验证数据
        $this->validateUserData($data);
        
        // 创建用户
        return $this->repository->create($data);
    }
    
    private function validateUserData(array $data): void
    {
        // 验证逻辑
    }
}
```

## 常见错误与修正

### 错误 1：使用短标签

**错误代码**：

```php
<?
echo "Hello";
```

**修正**：

```php
<?php
echo "Hello";
```

### 错误 2：类名使用下划线

**错误代码**：

```php
class user_profile
{
    // ...
}
```

**修正**：

```php
class UserProfile
{
    // ...
}
```

### 错误 3：方法名使用下划线

**错误代码**：

```php
class UserService
{
    public function get_user_by_id(int $id): ?User
    {
        // ...
    }
}
```

**修正**：

```php
class UserService
{
    public function getUserById(int $id): ?User
    {
        // ...
    }
}
```

### 错误 4：类常量使用小写

**错误代码**：

```php
class Status
{
    public const pending = 'pending';
}
```

**修正**：

```php
class Status
{
    public const PENDING = 'pending';
}
```

### 错误 5：文件同时声明类和产生副作用

**错误代码**：

```php
<?php
class UserService
{
    // ...
}

echo "UserService loaded";
```

**修正**：

```php
<?php
// 文件 1: UserService.php
class UserService
{
    // ...
}

// 文件 2: bootstrap.php
require 'UserService.php';
echo "UserService loaded";
```

## 检查工具

### PHP CodeSniffer

```bash
# 安装
composer require --dev squizlabs/php_codesniffer

# 检查 PSR-1
vendor/bin/phpcs --standard=PSR1 src/
```

### PHP CS Fixer

```bash
# 安装
composer require --dev friendsofphp/php-cs-fixer

# 配置 .php-cs-fixer.php
<?php
$config = new PhpCsFixer\Config();
return $config
    ->setRules([
        '@PSR1' => true,
    ]);

# 检查
vendor/bin/php-cs-fixer fix --dry-run

# 修复
vendor/bin/php-cs-fixer fix
```

## 练习

1. 创建一个符合 PSR-1 的类，包含：
   - 正确的命名空间
   - 使用 `StudlyCaps` 的类名
   - 使用全大写的类常量
   - 使用 `camelCase` 的方法名

2. 检查现有代码，找出不符合 PSR-1 的地方并修正。

3. 配置 PHP CodeSniffer 或 PHP CS Fixer，自动检查代码是否符合 PSR-1。

## 总结

PSR-1 是 PHP 编码标准的基础，所有其他 PSR 标准都基于它。遵循 PSR-1 可以：

- 提高代码的可读性和一致性
- 减少团队协作中的争议
- 为后续学习其他 PSR 标准打下基础
- 使代码更容易被其他开发者理解和维护
