# 2.17.2 PHPDoc

## 概述

PHPDoc 是 PHP 的文档注释标准，用于为代码添加文档说明。PHPDoc 注释可以帮助 IDE 提供代码补全和类型检查，也可以生成 API 文档。

## 基本语法

### 函数文档

```php
<?php
declare(strict_types=1);

/**
 * 计算两个数的和
 *
 * @param int $a 第一个数
 * @param int $b 第二个数
 * @return int 两个数的和
 */
function add(int $a, int $b): int
{
    return $a + $b;
}
```

### 类文档

```php
<?php
declare(strict_types=1);

/**
 * 用户服务类
 *
 * 提供用户相关的业务逻辑处理
 *
 * @package App\Services
 * @author Your Name
 * @since 1.0.0
 */
class UserService
{
    // ...
}
```

## 常用标签

### @param - 参数说明

```php
<?php
declare(strict_types=1);

/**
 * @param string $name 用户名
 * @param int $age 年龄
 * @param string|null $email 邮箱（可选）
 */
function createUser(string $name, int $age, ?string $email = null): void
{
    // ...
}
```

### @return - 返回值说明

```php
<?php
declare(strict_types=1);

/**
 * @return User|null 用户对象，如果不存在返回 null
 */
function findUser(int $id): ?User
{
    // ...
}
```

### @throws - 异常说明

```php
<?php
declare(strict_types=1);

/**
 * @throws InvalidArgumentException 当参数无效时
 * @throws RuntimeException 当操作失败时
 */
function processData(array $data): void
{
    if (empty($data)) {
        throw new InvalidArgumentException('Data cannot be empty');
    }
    // ...
}
```

### @var - 变量类型

```php
<?php
declare(strict_types=1);

class UserService
{
    /** @var UserRepository */
    private $repository;
    
    /** @var Logger */
    private $logger;
}
```

### @deprecated - 废弃说明

```php
<?php
declare(strict_types=1);

/**
 * @deprecated 使用 newMethod() 替代
 * @see newMethod()
 */
function oldMethod(): void
{
    // ...
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

namespace App\Services;

use App\Models\User;
use App\Repositories\UserRepository;
use InvalidArgumentException;
use RuntimeException;

/**
 * 用户服务类
 *
 * 提供用户相关的业务逻辑处理，包括用户的创建、查询、更新等操作。
 *
 * @package App\Services
 * @author Your Name
 * @since 1.0.0
 */
class UserService
{
    /**
     * 用户仓库
     *
     * @var UserRepository
     */
    private UserRepository $repository;
    
    /**
     * 构造函数
     *
     * @param UserRepository $repository 用户仓库实例
     */
    public function __construct(UserRepository $repository)
    {
        $this->repository = $repository;
    }
    
    /**
     * 根据 ID 获取用户
     *
     * @param int $id 用户 ID
     * @return User|null 用户对象，如果不存在返回 null
     */
    public function getUserById(int $id): ?User
    {
        return $this->repository->find($id);
    }
    
    /**
     * 创建用户
     *
     * @param array<string, mixed> $data 用户数据
     * @return User 创建的用户对象
     * @throws InvalidArgumentException 当数据无效时
     * @throws RuntimeException 当创建失败时
     */
    public function createUser(array $data): User
    {
        $this->validateUserData($data);
        return $this->repository->create($data);
    }
    
    /**
     * 验证用户数据
     *
     * @param array<string, mixed> $data 用户数据
     * @return void
     * @throws InvalidArgumentException 当数据无效时
     */
    private function validateUserData(array $data): void
    {
        if (empty($data['name'])) {
            throw new InvalidArgumentException('Name is required');
        }
    }
}
```

## 注意事项

1. **完整性**：为所有公共方法和类添加 PHPDoc 注释。

2. **准确性**：确保注释与实际代码一致。

3. **类型提示**：使用类型提示时，PHPDoc 可以补充更详细的信息。

4. **工具支持**：使用 IDE 插件自动生成 PHPDoc 注释。

5. **文档生成**：可以使用 phpDocumentor 等工具生成 API 文档。

## 练习

1. 为现有代码添加完整的 PHPDoc 注释。

2. 使用 phpDocumentor 生成项目的 API 文档。

3. 创建一个 PHPDoc 模板，统一注释格式。

4. 编写一个脚本，检查代码是否缺少 PHPDoc 注释。

5. 创建一个文档，说明项目中使用的 PHPDoc 规范。
