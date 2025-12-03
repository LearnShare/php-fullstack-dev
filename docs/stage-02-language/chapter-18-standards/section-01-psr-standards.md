# 2.18.1 PSR 标准

## 概述

PSR（PHP Standards Recommendations）是 PHP 社区制定的编码标准。遵循 PSR 标准可以提高代码的可读性、可维护性和互操作性。

## PSR-1：基本编码标准

### 文件要求

- 文件必须使用 `<?php` 或 `<?=` 标签
- 文件必须使用 UTF-8 无 BOM 编码
- 文件应该只声明符号（类、函数、常量等）或产生副作用，不应该同时做两件事

### 类命名

- 类名必须使用 `StudlyCaps`（大驼峰）
- 文件名必须与类名相同

```php
<?php
// UserService.php
declare(strict_types=1);

namespace App\Services;

class UserService
{
    // ...
}
```

### 方法命名

- 方法名必须使用 `camelCase`（小驼峰）

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
}
```

## PSR-4：自动加载标准

### 基本规则

- 完全限定的类名格式：`\<NamespaceName>(\<SubNamespaceNames>)*\<ClassName>`
- 完全限定的类名必须有一个顶级命名空间（"Vendor namespace"）
- 完全限定的类名可以有多个子命名空间
- 从完全限定的类名中去除最前面的命名空间，对应"命名空间前缀"
- 命名空间前缀后的子命名空间名称对应目录结构
- 类名对应文件名

### 示例

```php
<?php
// src/Controllers/UserController.php
declare(strict_types=1);

namespace App\Controllers;

class UserController
{
    // ...
}
```

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

## PSR-12：编码风格扩展

### 基本规则

- 代码必须使用 4 个空格缩进，不能使用制表符
- 行长度没有硬性限制，但建议不超过 120 个字符
- PHP 文件必须以一个空行结束
- 每行代码不能有尾随空格

### 关键字

- PHP 关键字必须小写
- `true`、`false`、`null` 必须小写

```php
<?php
declare(strict_types=1);

if ($condition === true) {
    return null;
}
```

### 类和方法的声明

```php
<?php
declare(strict_types=1);

namespace App\Services;

class UserService
{
    public function getUser(int $id): ?User
    {
        // ...
    }
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

namespace App\Services;

use App\Models\User;
use App\Repositories\UserRepository;

class UserService
{
    private UserRepository $repository;
    
    public function __construct(UserRepository $repository)
    {
        $this->repository = $repository;
    }
    
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
        if (empty($data['name'])) {
            throw new InvalidArgumentException('Name is required');
        }
    }
}
```

## 注意事项

1. **一致性**：在整个项目中保持一致的编码风格。

2. **工具支持**：使用 PHP CS Fixer 或 PHP_CodeSniffer 自动检查。

3. **团队协作**：在团队中统一使用 PSR 标准。

4. **逐步采用**：对于现有项目，可以逐步采用 PSR 标准。

5. **文档说明**：在项目文档中说明使用的 PSR 标准。

## 练习

1. 配置 PHP CS Fixer，使用 PSR-12 标准格式化代码。

2. 编写一个脚本，检查项目代码是否符合 PSR 标准。

3. 重构现有代码，使其符合 PSR-12 标准。

4. 创建一个代码审查清单，包含 PSR 标准检查项。

5. 编写文档，说明项目中使用的 PSR 标准。
