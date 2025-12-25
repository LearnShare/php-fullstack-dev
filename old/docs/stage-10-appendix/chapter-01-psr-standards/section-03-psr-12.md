# 10.1.3 PSR-12 扩展编码风格指南

## 概述

PSR-12（PHP Standards Recommendation 12）扩展了 PSR-1，提供了详细的代码风格规范。它替代了已废弃的 PSR-2，是当前 PHP 社区最广泛采用的编码风格标准。

## 官方文档

- **标准名称**：Extended Coding Style Guide
- **状态**：已接受（Accepted）
- **版本**：1.0.0
- **官方链接**：https://www.php-fig.org/psr/psr-12/

## 核心要求

### 1. 基本规则

#### 1.1 缩进

代码必须使用 4 个空格缩进，不能使用 Tab。

**正确示例**：

```php
<?php
declare(strict_types=1);

class UserService
{
    public function getUser(int $id): ?User
    {
        if ($id > 0) {
            return $this->repository->find($id);
        }
        
        return null;
    }
}
```

**错误示例**：

```php
<?php
// 错误：使用 Tab 缩进
class UserService
{
	public function getUser(int $id): ?User
	{
		// ...
	}
}
```

#### 1.2 行长度

每行代码长度不应超过 120 个字符；软限制为 80 个字符。

**正确示例**：

```php
<?php
declare(strict_types=1);

// 短行（推荐）
$user = $this->repository->find($id);

// 长行需要换行
$result = $this->service->processData(
    $data,
    $options,
    $callback
);
```

#### 1.3 文件末尾

文件必须以空行结束。

**正确示例**：

```php
<?php
declare(strict_types=1);

class UserService
{
    // ...
}

```

**错误示例**：

```php
<?php
declare(strict_types=1);

class UserService
{
    // ...
}
// 错误：文件末尾没有空行
```

### 2. 关键字和类型

#### 2.1 关键字

所有 PHP 关键字必须小写。

**正确示例**：

```php
<?php
declare(strict_types=1);

if ($condition) {
    // ...
}

foreach ($items as $item) {
    // ...
}

class UserService
{
    public function method(): void
    {
        // ...
    }
}
```

**错误示例**：

```php
<?php
// 错误：关键字大写
IF ($condition) {
    // ...
}

Class UserService
{
    // ...
}
```

#### 2.2 类型声明

`true`、`false`、`null` 必须小写。

**正确示例**：

```php
<?php
declare(strict_types=1);

$isActive = true;
$isDeleted = false;
$user = null;

if ($user === null) {
    // ...
}
```

**错误示例**：

```php
<?php
// 错误：类型大写
$isActive = TRUE;
$isDeleted = FALSE;
$user = NULL;
```

### 3. 命名空间和 use 声明

#### 3.1 命名空间声明

命名空间声明后必须有一个空行。

**正确示例**：

```php
<?php
declare(strict_types=1);

namespace App\Services;

use App\Models\User;
use App\Repositories\UserRepository;

class UserService
{
    // ...
}
```

#### 3.2 use 声明

- `use` 声明必须在 `namespace` 声明之后
- 每个 `use` 声明必须只声明一个名称
- `use` 块后必须有一个空行

**正确示例**：

```php
<?php
declare(strict_types=1);

namespace App\Services;

use App\Models\User;
use App\Repositories\UserRepository;
use App\Exceptions\UserNotFoundException;

class UserService
{
    // ...
}
```

**错误示例**：

```php
<?php
// 错误：多个 use 声明在一行
use App\Models\User, App\Repositories\UserRepository;

// 错误：use 块后没有空行
use App\Models\User;
class UserService
{
    // ...
}
```

### 4. 类、属性和方法

#### 4.1 类的开始和结束花括号

类的开始花括号必须写在下一行，结束花括号必须写在主体后单独一行。

**正确示例**：

```php
<?php
declare(strict_types=1);

namespace App\Services;

class UserService
{
    // ...
}
```

**错误示例**：

```php
<?php
// 错误：开始花括号在同一行
class UserService {
    // ...
}

// 错误：结束花括号位置不对
class UserService
{
    // ...
    }
```

#### 4.2 方法

- 方法的开始花括号必须写在函数声明后下一行
- 结束花括号必须写在函数主体后单独一行
- 方法参数列表可以分成多行，每个参数一行

**正确示例**：

```php
<?php
declare(strict_types=1);

class UserService
{
    public function getUser(int $id): ?User
    {
        return $this->repository->find($id);
    }
    
    public function createUser(
        string $name,
        string $email,
        array $options = []
    ): User {
        // ...
    }
}
```

**错误示例**：

```php
<?php
// 错误：开始花括号在同一行
class UserService
{
    public function getUser(int $id): ?User {
        // ...
    }
}
```

#### 4.3 属性

- 所有属性必须声明可见性
- `var` 关键字不能用于声明属性
- 每个属性声明不能超过一个属性
- 属性名不应使用单个下划线前缀表示 protected 或 private 可见性

**正确示例**：

```php
<?php
declare(strict_types=1);

class User
{
    public string $name;
    protected string $email;
    private int $id;
}
```

**错误示例**：

```php
<?php
// 错误：使用 var
class User
{
    var $name;
}

// 错误：没有可见性声明
class User
{
    $name;
}

// 错误：多个属性声明
class User
{
    public $name, $email;
}
```

### 5. 控制结构

#### 5.1 基本规则

- 控制结构关键字后必须有一个空格
- 左括号后不能有空格
- 右括号前不能有空格
- 右括号和开始花括号之间必须有一个空格
- 结构体主体必须缩进一次
- 结束花括号必须在主体后单独一行

#### 5.2 if、elseif、else

**正确示例**：

```php
<?php
declare(strict_types=1);

if ($condition) {
    // ...
} elseif ($otherCondition) {
    // ...
} else {
    // ...
}
```

**错误示例**：

```php
<?php
// 错误：关键字后没有空格
if($condition) {
    // ...
}

// 错误：左括号后有空格
if ( $condition ) {
    // ...
}

// 错误：开始花括号在同一行
if ($condition)
{
    // ...
}
```

#### 5.3 switch、case

**正确示例**：

```php
<?php
declare(strict_types=1);

switch ($value) {
    case 1:
        // ...
        break;
    
    case 2:
        // ...
        break;
    
    default:
        // ...
        break;
}
```

#### 5.4 while、do-while

**正确示例**：

```php
<?php
declare(strict_types=1);

while ($condition) {
    // ...
}

do {
    // ...
} while ($condition);
```

#### 5.5 for

**正确示例**：

```php
<?php
declare(strict_types=1);

for ($i = 0; $i < 10; $i++) {
    // ...
}
```

#### 5.6 foreach

**正确示例**：

```php
<?php
declare(strict_types=1);

foreach ($items as $item) {
    // ...
}

foreach ($items as $key => $value) {
    // ...
}
```

#### 5.7 try-catch

**正确示例**：

```php
<?php
declare(strict_types=1);

try {
    // ...
} catch (FirstExceptionType $e) {
    // ...
} catch (OtherExceptionType $e) {
    // ...
}
```

### 6. 闭包

#### 6.1 声明

闭包声明时，`function` 关键字后以及 `use` 关键字前后必须有一个空格。

**正确示例**：

```php
<?php
declare(strict_types=1);

$closure = function ($arg) use ($var1, $var2) {
    // ...
};

$result = array_map(function ($item) {
    return $item * 2;
}, $array);
```

**错误示例**：

```php
<?php
// 错误：function 关键字后没有空格
$closure = function($arg) {
    // ...
};

// 错误：use 关键字前后没有空格
$closure = function ($arg)use($var1) {
    // ...
};
```

### 7. 数组

#### 7.1 短数组语法

推荐使用短数组语法 `[]` 而不是 `array()`。

**正确示例**：

```php
<?php
declare(strict_types=1);

$array = [1, 2, 3];
$assoc = ['key' => 'value'];
```

**不推荐**：

```php
<?php
$array = array(1, 2, 3);
$assoc = array('key' => 'value');
```

## 完整示例

### 符合 PSR-12 的完整类示例

```php
<?php
declare(strict_types=1);

namespace App\Services;

use App\Models\User;
use App\Repositories\UserRepository;
use App\Exceptions\UserNotFoundException;

/**
 * 用户服务类
 */
class UserService
{
    private UserRepository $repository;
    
    public function __construct(UserRepository $repository)
    {
        $this->repository = $repository;
    }
    
    /**
     * 根据 ID 获取用户
     *
     * @param int $id 用户 ID
     * @return User|null 用户对象或 null
     * @throws UserNotFoundException 当用户不存在时
     */
    public function getUserById(int $id): ?User
    {
        if ($id <= 0) {
            return null;
        }
        
        $user = $this->repository->find($id);
        
        if ($user === null) {
            throw new UserNotFoundException("User with ID {$id} not found");
        }
        
        return $user;
    }
    
    /**
     * 创建用户
     *
     * @param string $name 用户名
     * @param string $email 邮箱
     * @param array $options 可选参数
     * @return User 创建的用户对象
     */
    public function createUser(
        string $name,
        string $email,
        array $options = []
    ): User {
        $data = array_merge([
            'name' => $name,
            'email' => $email,
        ], $options);
        
        return $this->repository->create($data);
    }
    
    /**
     * 批量处理用户
     *
     * @param array $userIds 用户 ID 数组
     * @return array 处理结果
     */
    public function processUsers(array $userIds): array
    {
        $results = [];
        
        foreach ($userIds as $userId) {
            try {
                $user = $this->getUserById($userId);
                $results[] = [
                    'id' => $userId,
                    'status' => 'success',
                    'user' => $user,
                ];
            } catch (UserNotFoundException $e) {
                $results[] = [
                    'id' => $userId,
                    'status' => 'error',
                    'message' => $e->getMessage(),
                ];
            }
        }
        
        return $results;
    }
}
```

## 工具配置

### PHP CS Fixer 配置

创建 `.php-cs-fixer.php`：

```php
<?php

$config = new PhpCsFixer\Config();

return $config
    ->setRules([
        '@PSR12' => true,
        'array_syntax' => ['syntax' => 'short'],
        'ordered_imports' => ['sort_algorithm' => 'alpha'],
        'no_unused_imports' => true,
        'not_operator_with_successor_space' => true,
        'trailing_comma_in_multiline' => true,
        'phpdoc_scalar' => true,
        'unary_operator_spaces' => true,
        'binary_operator_spaces' => true,
        'blank_line_before_statement' => [
            'statements' => ['break', 'continue', 'declare', 'return', 'throw', 'try'],
        ],
        'phpdoc_single_line_var_spacing' => true,
        'phpdoc_var_without_name' => true,
    ])
    ->setFinder(
        PhpCsFixer\Finder::create()
            ->in(__DIR__ . '/src')
            ->in(__DIR__ . '/tests')
            ->name('*.php')
            ->ignoreDotFiles(true)
            ->ignoreVCS(true)
    );
```

### PHP CodeSniffer 配置

创建 `phpcs.xml`：

```xml
<?xml version="1.0"?>
<ruleset name="PSR12">
    <description>PSR-12 Coding Standard</description>
    
    <file>./src</file>
    <file>./tests</file>
    
    <rule ref="PSR12"/>
    
    <exclude-pattern>*/vendor/*</exclude-pattern>
    <exclude-pattern>*/node_modules/*</exclude-pattern>
</ruleset>
```

## 常见错误与修正

### 错误 1：使用 Tab 缩进

**错误**：使用 Tab 而不是 4 个空格

**修正**：配置编辑器将 Tab 转换为 4 个空格

### 错误 2：关键字大写

**错误**：`IF`、`CLASS`、`FUNCTION` 等

**修正**：全部改为小写

### 错误 3：花括号位置

**错误**：开始花括号在同一行

**修正**：开始花括号写在下一行

### 错误 4：控制结构空格

**错误**：`if($condition)` 或 `if ( $condition )`

**修正**：`if ($condition)`

## 练习

1. 检查现有代码，找出不符合 PSR-12 的地方并修正。

2. 配置 PHP CS Fixer，自动修复代码风格问题。

3. 编写一个符合 PSR-12 的完整类，包含：
   - 命名空间和 use 声明
   - 类、属性和方法
   - 各种控制结构
   - 闭包

## 总结

PSR-12 是当前 PHP 社区最广泛采用的编码风格标准。遵循 PSR-12 可以：

- 提高代码的一致性和可读性
- 减少代码审查时的争议
- 使代码更容易被其他开发者理解和维护
- 与 PHP 生态系统中的其他项目保持一致
