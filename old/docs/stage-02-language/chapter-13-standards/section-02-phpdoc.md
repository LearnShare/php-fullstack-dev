# 2.13.2 PHPDoc

## 概述

PHPDoc 是 PHP 的文档注释标准，用于为代码添加文档说明。PHPDoc 注释可以帮助 IDE 提供代码补全和类型检查，也可以生成 API 文档。PHPDoc 基于 JavaDoc 风格，使用特殊的注释块和标签来描述代码元素。

## 特性

- **IDE 支持**：提供代码补全、类型检查、参数提示等功能
- **文档生成**：可以使用 phpDocumentor 等工具自动生成 API 文档
- **类型增强**：在 PHP 7.4 之前，PHPDoc 是唯一的方式为属性添加类型提示
- **标准格式**：遵循统一的注释格式，提高代码可读性
- **工具集成**：与静态分析工具（如 PHPStan、Psalm）集成，提供类型检查

## 基本语法

### PHPDoc 注释格式

PHPDoc 注释以 `/**` 开始，以 `*/` 结束，每行以 `*` 开头。注释块必须紧邻被注释的代码元素。

**基本格式**：

```php
<?php
declare(strict_types=1);

/**
 * 注释内容
 *
 * 详细描述（可选）
 *
 * @tag 标签内容
 */
```

**规则**：
- 注释块以 `/**` 开始（两个星号）
- 每行以 `*` 开头，后跟一个空格
- 注释块以 `*/` 结束
- 第一行是简短描述，不能为空
- 详细描述和标签之间用空行分隔
- 标签以 `@` 开头

### 注释块结构

PHPDoc 注释块的标准结构：

```
/**
 * 简短描述（必需，一行）
 *
 * 详细描述（可选，多行）
 * 可以包含多行内容
 *
 * @tag1 标签1的内容
 * @tag2 标签2的内容
 */
```

**说明**：
- **简短描述**：第一行，简洁描述代码元素的功能
- **详细描述**：可选，提供更详细的说明，可以多行
- **标签**：以 `@` 开头的特殊标记，提供元数据信息

### 函数/方法文档

**语法格式**：

```php
/**
 * 简短描述
 *
 * 详细描述（可选）
 *
 * @param type $paramName 参数说明
 * @return type 返回值说明
 * @throws ExceptionClass 异常说明
 */
```

**示例**：

```php
<?php
declare(strict_types=1);

/**
 * 计算两个数的和
 *
 * 将两个整数相加并返回结果。如果结果溢出，行为未定义。
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

**语法格式**：

```php
/**
 * 简短描述
 *
 * 详细描述（可选）
 *
 * @package PackageName
 * @author Author Name
 * @since version
 * @see RelatedClass
 */
```

**示例**：

```php
<?php
declare(strict_types=1);

/**
 * 用户服务类
 *
 * 提供用户相关的业务逻辑处理，包括用户的创建、查询、更新、删除等操作。
 * 本类封装了用户相关的业务规则，与数据访问层解耦。
 *
 * @package App\Services
 * @author Your Name <your.email@example.com>
 * @since 1.0.0
 * @see \App\Models\User
 * @see \App\Repositories\UserRepository
 */
class UserService
{
    // ...
}
```

### 属性文档

**语法格式**：

```php
/**
 * 属性描述
 *
 * @var type 详细说明（可选）
 */
```

**示例**：

```php
<?php
declare(strict_types=1);

class UserService
{
    /**
     * 用户仓库实例
     *
     * 用于访问用户数据
     *
     * @var UserRepository
     */
    private UserRepository $repository;
    
    /**
     * 日志记录器
     *
     * @var Logger|null
     */
    private ?Logger $logger = null;
}
```

## 类型系统

### 基本类型

PHPDoc 支持以下基本类型：

| 类型      | 说明           | 示例              |
|:----------|:---------------|:------------------|
| `int`     | 整数           | `int`             |
| `float`   | 浮点数         | `float`           |
| `string`  | 字符串         | `string`          |
| `bool`    | 布尔值         | `bool`             |
| `array`   | 数组           | `array`            |
| `object`  | 对象           | `object`           |
| `resource`| 资源           | `resource`         |
| `callable`| 可调用类型     | `callable`         |
| `iterable`| 可迭代类型     | `iterable`         |
| `void`    | 无返回值       | `void`             |
| `mixed`   | 混合类型       | `mixed`            |
| `null`    | 空值           | `null`             |

### 复合类型

**联合类型**：使用 `|` 分隔多个类型

```php
/**
 * @param int|string $id 用户 ID（整数或字符串）
 * @return User|false 用户对象或 false
 */
```

**可空类型**：使用 `?` 或 `|null`

```php
/**
 * @param string|null $name 用户名（可选）
 * @return ?User 用户对象或 null
 */
```

**数组类型**：使用 `array<KeyType, ValueType>` 或 `Type[]`

```php
/**
 * @param array<string, int> $scores 分数数组（键为字符串，值为整数）
 * @param string[] $names 名称数组（字符串数组）
 * @return array<int, User> 用户数组（键为整数，值为 User 对象）
 */
```

**泛型类型**：使用尖括号指定类型参数

```php
/**
 * @param Collection<User> $users 用户集合
 * @return array<string, array<int, mixed>> 嵌套数组
 */
```

### 类类型

**完全限定类名**：使用完整的命名空间路径

```php
/**
 * @param \App\Models\User $user 用户对象
 * @return \App\Repositories\UserRepository 用户仓库
 */
```

**相对类名**：在当前命名空间下使用相对路径

```php
<?php
namespace App\Services;

/**
 * @param User $user 用户对象（等同于 \App\Models\User）
 */
```

## 常用标签详解

### @param - 参数说明

**语法**：`@param [Type] $paramName 参数说明`

**参数**：
- `Type`：参数类型（可选，但推荐使用）
- `$paramName`：参数名称（必需）
- 参数说明：参数的详细描述（可选）

**说明**：
- 每个参数必须单独使用一个 `@param` 标签
- 类型可以是基本类型、类名、联合类型等
- 可选参数可以在说明中标注"可选"或"默认值"

**示例**：

```php
<?php
declare(strict_types=1);

/**
 * 创建用户
 *
 * @param string $name 用户名（必需）
 * @param int $age 年龄（必需）
 * @param string|null $email 邮箱地址（可选）
 * @param array<string, mixed> $metadata 元数据数组（可选，默认为空数组）
 * @return void
 */
function createUser(
    string $name,
    int $age,
    ?string $email = null,
    array $metadata = []
): void {
    // ...
}
```

### @return - 返回值说明

**语法**：`@return Type 返回值说明`

**参数**：
- `Type`：返回值类型（可选，但推荐使用）
- 返回值说明：返回值的详细描述（可选）

**说明**：
- 每个方法只能有一个 `@return` 标签
- 如果方法不返回值，使用 `@return void`
- 如果可能返回多种类型，使用联合类型 `Type1|Type2`

**示例**：

```php
<?php
declare(strict_types=1);

/**
 * 根据 ID 查找用户
 *
 * @param int $id 用户 ID
 * @return \App\Models\User|null 用户对象，如果不存在返回 null
 */
function findUser(int $id): ?\App\Models\User
{
    // ...
}

/**
 * 获取所有用户
 *
 * @return array<int, \App\Models\User> 用户数组，键为用户 ID
 */
function getAllUsers(): array
{
    // ...
}

/**
 * 执行操作（无返回值）
 *
 * @return void
 */
function performAction(): void
{
    // ...
}
```

### @throws - 异常说明

**语法**：`@throws ExceptionClass 异常说明`

**参数**：
- `ExceptionClass`：异常类名（必需）
- 异常说明：异常的描述和触发条件（可选）

**说明**：
- 可以多个 `@throws` 标签，每个标签对应一种可能的异常
- 异常类名可以使用完全限定名称或相对名称
- 应该说明什么情况下会抛出该异常

**示例**：

```php
<?php
declare(strict_types=1);

use InvalidArgumentException;
use RuntimeException;

/**
 * 处理用户数据
 *
 * @param array<string, mixed> $data 用户数据
 * @return void
 * @throws InvalidArgumentException 当数据为空或缺少必需字段时
 * @throws RuntimeException 当数据库操作失败时
 * @throws \PDOException 当数据库连接失败时
 */
function processUserData(array $data): void
{
    if (empty($data)) {
        throw new InvalidArgumentException('Data cannot be empty');
    }
    
    if (!isset($data['name'])) {
        throw new InvalidArgumentException('Name is required');
    }
    
    // ...
}
```

### @var - 变量/属性类型

**语法**：`@var Type 变量说明`

**参数**：
- `Type`：变量类型（可选，但推荐使用）
- 变量说明：变量的详细描述（可选）

**说明**：
- 用于为类属性、局部变量等添加类型提示
- 在 PHP 7.4 之前，这是为属性添加类型提示的唯一方式
- 在 PHP 7.4+ 中，如果属性已有类型声明，`@var` 可以省略，但建议保留用于文档

**示例**：

```php
<?php
declare(strict_types=1);

class UserService
{
    /**
     * 用户仓库实例
     *
     * @var \App\Repositories\UserRepository
     */
    private $repository;
    
    /**
     * 日志记录器（可选）
     *
     * @var \Psr\Log\LoggerInterface|null
     */
    private ?\Psr\Log\LoggerInterface $logger = null;
    
    /**
     * 配置数组
     *
     * @var array<string, mixed>
     */
    private array $config = [];
    
    public function process(): void
    {
        /** @var \App\Models\User $user */
        $user = $this->repository->find(1);
        
        /** @var array<int, string> $names */
        $names = array_map(fn($u) => $u->name, $users);
    }
}
```

### @deprecated - 废弃说明

**语法**：`@deprecated [version] 废弃说明`

**参数**：
- `version`：废弃的版本号（可选）
- 废弃说明：废弃的原因和替代方案（推荐）

**说明**：
- 标记已废弃的代码元素
- 应该说明废弃的原因和推荐的替代方案
- 通常配合 `@see` 标签使用，指向替代方法

**示例**：

```php
<?php
declare(strict_types=1);

/**
 * 获取用户信息（旧方法）
 *
 * @param int $id 用户 ID
 * @return array<string, mixed> 用户信息数组
 * @deprecated 1.2.0 使用 getUserById() 方法替代，该方法返回对象而非数组
 * @see getUserById()
 */
function getOldUser(int $id): array
{
    // ...
}

/**
 * 获取用户信息（新方法）
 *
 * @param int $id 用户 ID
 * @return \App\Models\User|null 用户对象
 */
function getUserById(int $id): ?\App\Models\User
{
    // ...
}
```

### @see - 相关引用

**语法**：`@see Reference 说明`

**参数**：
- `Reference`：引用的目标（类名、方法名、URL 等）
- 说明：引用的说明（可选）

**说明**：
- 用于引用相关的类、方法、文档等
- 可以引用类名、方法名（格式：`ClassName::methodName()`）、URL 等
- 常用于 `@deprecated` 标签中，指向替代方法

**示例**：

```php
<?php
declare(strict_types=1);

/**
 * 用户服务类
 *
 * @see \App\Models\User 用户模型
 * @see \App\Repositories\UserRepository 用户仓库
 * @see https://example.com/docs/user-service 相关文档
 */
class UserService
{
    /**
     * @deprecated 使用 createUser() 方法替代
     * @see createUser()
     */
    public function addUser(): void
    {
        // ...
    }
}
```

### @since - 版本说明

**语法**：`@since version 说明`

**参数**：
- `version`：版本号（必需）
- 说明：版本说明（可选）

**说明**：
- 标记代码元素引入的版本
- 用于 API 版本管理

**示例**：

```php
<?php
declare(strict_types=1);

/**
 * 用户服务类
 *
 * @since 1.0.0
 */
class UserService
{
    /**
     * 获取用户（1.0.0 引入）
     *
     * @since 1.0.0
     */
    public function getUser(): void
    {
        // ...
    }
    
    /**
     * 批量获取用户（1.2.0 引入）
     *
     * @since 1.2.0
     */
    public function getUsers(): void
    {
        // ...
    }
}
```

### @package - 包说明

**语法**：`@package PackageName`

**参数**：
- `PackageName`：包名称（通常使用命名空间）

**说明**：
- 用于组织相关的类
- 通常使用命名空间作为包名

**示例**：

```php
<?php
declare(strict_types=1);

namespace App\Services;

/**
 * 用户服务类
 *
 * @package App\Services
 */
class UserService
{
    // ...
}
```

### @author - 作者说明

**语法**：`@author Name <email>`

**参数**：
- `Name`：作者名称（必需）
- `email`：作者邮箱（可选）

**示例**：

```php
<?php
declare(strict_types=1);

/**
 * 用户服务类
 *
 * @author John Doe <john.doe@example.com>
 * @author Jane Smith
 */
class UserService
{
    // ...
}
```

### @internal - 内部使用

**语法**：`@internal 说明`

**说明**：
- 标记为内部使用的代码元素
- 不应该在外部使用，可能在未来版本中更改或删除

**示例**：

```php
<?php
declare(strict_types=1);

class UserService
{
    /**
     * 内部方法，用于验证用户数据
     *
     * @internal 此方法仅供内部使用，不应在外部调用
     * @param array<string, mixed> $data 用户数据
     * @return void
     */
    private function validateUserData(array $data): void
    {
        // ...
    }
}
```

### @todo - 待办事项

**语法**：`@todo 待办说明`

**说明**：
- 标记需要完成或改进的代码
- 用于代码审查和开发计划

**示例**：

```php
<?php
declare(strict_types=1);

class UserService
{
    /**
     * 获取用户
     *
     * @todo 添加缓存支持
     * @todo 优化查询性能
     * @return \App\Models\User|null
     */
    public function getUser(int $id): ?\App\Models\User
    {
        // ...
    }
}
```

## 其他常用标签

### @api - API 说明

**语法**：`@api`

**说明**：标记为公共 API，表示该代码元素是公开的 API，应该保持向后兼容。

```php
/**
 * 获取用户信息
 *
 * @api 这是公共 API，保持向后兼容
 * @param int $id 用户 ID
 * @return \App\Models\User|null
 */
public function getUser(int $id): ?\App\Models\User
{
    // ...
}
```

### @inheritDoc - 继承文档

**语法**：`@inheritDoc`

**说明**：继承父类或接口的文档注释。

```php
class UserService extends BaseService
{
    /**
     * @inheritDoc
     */
    public function process(): void
    {
        // ...
    }
}
```

### @method - 魔术方法

**语法**：`@method ReturnType methodName(Type $param) 方法说明`

**说明**：为魔术方法（如 `__call()`）添加文档。

```php
/**
 * 用户模型
 *
 * @method \App\Models\User find(int $id) 查找用户
 * @method array<int, \App\Models\User> all() 获取所有用户
 */
class UserRepository
{
    public function __call(string $name, array $arguments)
    {
        // ...
    }
}
```

### @property - 魔术属性

**语法**：`@property Type $propertyName 属性说明`

**说明**：为魔术属性（如 `__get()`）添加文档。

```php
/**
 * 用户模型
 *
 * @property int $id 用户 ID
 * @property string $name 用户名
 * @property string|null $email 邮箱
 */
class User
{
    public function __get(string $name)
    {
        // ...
    }
}
```

### @example - 示例代码

**语法**：`@example [URL] 示例说明`

**说明**：提供使用示例。

```php
/**
 * 计算两个数的和
 *
 * @param int $a 第一个数
 * @param int $b 第二个数
 * @return int 两个数的和
 * @example
 * $result = add(5, 3); // 返回 8
 */
function add(int $a, int $b): int
{
    return $a + $b;
}
```

### @link - 链接

**语法**：`@link URL 链接说明`

**说明**：提供相关链接。

```php
/**
 * 用户服务类
 *
 * @link https://example.com/docs/user-service 用户服务文档
 * @link https://example.com/api/users API 文档
 */
class UserService
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
use Psr\Log\LoggerInterface;

/**
 * 用户服务类
 *
 * 提供用户相关的业务逻辑处理，包括用户的创建、查询、更新、删除等操作。
 * 本类封装了用户相关的业务规则，与数据访问层解耦。
 *
 * @package App\Services
 * @author John Doe <john.doe@example.com>
 * @since 1.0.0
 * @see \App\Models\User
 * @see \App\Repositories\UserRepository
 * @link https://example.com/docs/user-service 相关文档
 */
class UserService
{
    /**
     * 用户仓库实例
     *
     * 用于访问用户数据，提供数据持久化功能。
     *
     * @var \App\Repositories\UserRepository
     */
    private UserRepository $repository;
    
    /**
     * 日志记录器
     *
     * 用于记录操作日志和错误信息。
     *
     * @var \Psr\Log\LoggerInterface|null
     */
    private ?LoggerInterface $logger = null;
    
    /**
     * 配置数组
     *
     * 存储服务配置信息。
     *
     * @var array<string, mixed>
     */
    private array $config = [];
    
    /**
     * 构造函数
     *
     * 初始化用户服务，注入依赖的仓库实例。
     *
     * @param \App\Repositories\UserRepository $repository 用户仓库实例
     * @param \Psr\Log\LoggerInterface|null $logger 日志记录器（可选）
     * @param array<string, mixed> $config 配置数组（可选，默认为空数组）
     */
    public function __construct(
        UserRepository $repository,
        ?LoggerInterface $logger = null,
        array $config = []
    ) {
        $this->repository = $repository;
        $this->logger = $logger;
        $this->config = $config;
    }
    
    /**
     * 根据 ID 获取用户
     *
     * 从数据仓库中查找指定 ID 的用户。如果用户不存在，返回 null。
     *
     * @param int $id 用户 ID（必须大于 0）
     * @return \App\Models\User|null 用户对象，如果不存在返回 null
     * @throws \InvalidArgumentException 当 ID 无效时（小于等于 0）
     */
    public function getUserById(int $id): ?User
    {
        if ($id <= 0) {
            throw new InvalidArgumentException('User ID must be greater than 0');
        }
        
        return $this->repository->find($id);
    }
    
    /**
     * 创建用户
     *
     * 创建新用户并保存到数据库。会验证用户数据的有效性。
     *
     * @param array<string, mixed> $data 用户数据，必须包含以下键：
     *   - `name` (string): 用户名（必需）
     *   - `email` (string): 邮箱地址（必需，必须是有效的邮箱格式）
     *   - `age` (int): 年龄（可选，必须大于 0）
     * @return \App\Models\User 创建的用户对象
     * @throws \InvalidArgumentException 当数据无效时（缺少必需字段或格式不正确）
     * @throws \RuntimeException 当数据库操作失败时
     * @since 1.0.0
     */
    public function createUser(array $data): User
    {
        $this->validateUserData($data);
        
        try {
            $user = $this->repository->create($data);
            
            if ($this->logger !== null) {
                $this->logger->info('User created', ['user_id' => $user->id]);
            }
            
            return $user;
        } catch (\Exception $e) {
            throw new RuntimeException('Failed to create user: ' . $e->getMessage(), 0, $e);
        }
    }
    
    /**
     * 获取所有用户
     *
     * 获取系统中的所有用户列表。
     *
     * @param int $limit 返回数量限制（可选，默认 100，最大 1000）
     * @param int $offset 偏移量（可选，默认 0）
     * @return array<int, \App\Models\User> 用户数组，键为用户 ID
     */
    public function getAllUsers(int $limit = 100, int $offset = 0): array
    {
        $limit = min($limit, 1000);
        
        return $this->repository->findAll($limit, $offset);
    }
    
    /**
     * 验证用户数据
     *
     * 验证用户数据的有效性和完整性。
     *
     * @param array<string, mixed> $data 用户数据
     * @return void
     * @throws \InvalidArgumentException 当数据无效时
     * @internal 此方法仅供内部使用，不应在外部调用
     */
    private function validateUserData(array $data): void
    {
        if (empty($data['name'])) {
            throw new InvalidArgumentException('Name is required');
        }
        
        if (empty($data['email'])) {
            throw new InvalidArgumentException('Email is required');
        }
        
        if (!filter_var($data['email'], FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException('Invalid email format');
        }
        
        if (isset($data['age']) && $data['age'] <= 0) {
            throw new InvalidArgumentException('Age must be greater than 0');
        }
    }
}
```

## 输出结果说明

使用 phpDocumentor 等工具可以基于 PHPDoc 注释生成 HTML 格式的 API 文档，包含：

- 类和方法的结构化文档
- 参数和返回值的详细说明
- 异常说明
- 使用示例
- 相关链接和引用

## 常见问题

### 问题 1：PHPDoc 和类型声明有什么区别？

**回答**：
- **类型声明**（PHP 7.0+）：在运行时进行类型检查，如果类型不匹配会抛出错误
- **PHPDoc**：仅在开发时提供类型提示，IDE 和静态分析工具使用，不影响运行时

```php
<?php
declare(strict_types=1);

// 类型声明（运行时检查）
function add(int $a, int $b): int
{
    return $a + $b;
}

// PHPDoc（仅开发时提示）
/**
 * @param int $a
 * @param int $b
 * @return int
 */
function addOld($a, $b)
{
    return $a + $b;
}
```

### 问题 2：什么时候需要 PHPDoc，什么时候不需要？

**回答**：
- **需要 PHPDoc**：
  - 公共 API 方法
  - 复杂的方法（参数多、逻辑复杂）
  - 可能抛出异常的方法
  - 返回类型不明确的方法
  - PHP 7.4 之前的类属性

- **可以省略 PHPDoc**：
  - 简单的 getter/setter 方法（如果类型声明已足够清晰）
  - 私有方法（如果代码已足够清晰）
  - PHP 7.4+ 已有类型声明的属性

### 问题 3：如何处理泛型和复杂类型？

**回答**：使用 PHPDoc 的类型语法：

```php
/**
 * @param array<string, int> $scores 分数数组
 * @param array<int, \App\Models\User> $users 用户数组
 * @param \Iterator<\App\Models\User> $userIterator 用户迭代器
 * @return array<string, array<int, mixed>> 嵌套数组
 */
function processData(
    array $scores,
    array $users,
    \Iterator $userIterator
): array {
    // ...
}
```

### 问题 4：PHPDoc 中的类型和实际类型声明不一致怎么办？

**回答**：应该保持一致。如果类型声明和 PHPDoc 不一致，应该以类型声明为准，并更新 PHPDoc：

```php
// ❌ 错误：类型不一致
/**
 * @param string $id
 */
function getUser(int $id): ?User  // 实际是 int，不是 string
{
    // ...
}

// ✅ 正确：保持一致
/**
 * @param int $id
 */
function getUser(int $id): ?User
{
    // ...
}
```

## 最佳实践

### 1. 保持简洁但完整

PHPDoc 应该简洁但包含必要信息：

```php
// ✅ 好的示例
/**
 * 根据 ID 获取用户
 *
 * @param int $id 用户 ID
 * @return \App\Models\User|null 用户对象
 * @throws \InvalidArgumentException 当 ID 无效时
 */
public function getUserById(int $id): ?User

// ❌ 过于简单
/**
 * 获取用户
 */
public function getUserById(int $id): ?User

// ❌ 过于冗长
/**
 * 根据 ID 获取用户
 *
 * 这个方法会从数据库中查询指定 ID 的用户。
 * 如果用户存在，返回用户对象；如果不存在，返回 null。
 * 如果 ID 无效（小于等于 0），会抛出异常。
 * 这个方法使用了缓存机制，提高查询性能。
 * 
 * @param int $id 用户 ID，必须是大于 0 的整数
 * @return \App\Models\User|null 用户对象，如果不存在返回 null
 * @throws \InvalidArgumentException 当 ID 无效时（小于等于 0）
 */
public function getUserById(int $id): ?User
```

### 2. 使用完全限定类名

在 PHPDoc 中使用完全限定类名，避免命名空间问题：

```php
// ✅ 推荐
/**
 * @param \App\Models\User $user
 * @return \App\Repositories\UserRepository
 */

// ⚠️ 可以使用（如果当前命名空间明确）
namespace App\Services;
/**
 * @param User $user  // 等同于 \App\Models\User（如果已 use）
 */
```

### 3. 为复杂类型提供详细说明

对于复杂类型，提供详细的说明：

```php
/**
 * 处理用户数据
 *
 * @param array<string, mixed> $data 用户数据，必须包含以下键：
 *   - `name` (string): 用户名（必需）
 *   - `email` (string): 邮箱（必需）
 *   - `age` (int): 年龄（可选）
 * @return array<string, mixed> 处理后的数据
 */
```

### 4. 使用工具自动生成和检查

使用 IDE 插件和工具自动生成和检查 PHPDoc：

- **PHPStorm**：自动生成 PHPDoc
- **phpDocumentor**：生成 API 文档
- **PHPStan / Psalm**：静态分析，检查类型一致性

### 5. 保持文档和代码同步

当代码更改时，及时更新 PHPDoc：

```php
// 代码更改前
/**
 * @param string $id
 */
function getUser(string $id): ?User

// 代码更改后（类型改为 int）
/**
 * @param int $id  // 必须更新
 */
function getUser(int $id): ?User
```

## 对比分析

### PHPDoc vs 类型声明

| 特性                 | PHPDoc                    | 类型声明                  |
|:---------------------|:--------------------------|:--------------------------|
| **运行时检查**       | 否                        | 是                        |
| **IDE 支持**         | 是                        | 是                        |
| **文档生成**         | 是                        | 否                        |
| **复杂类型**         | 支持（泛型、联合类型等）  | 有限支持                  |
| **向后兼容**         | 是（PHP 5.6+）            | 否（PHP 7.0+）            |
| **推荐使用**         | 补充类型声明，提供文档    | 必需（PHP 7.0+）          |

### PHPDoc vs 普通注释

| 特性                 | PHPDoc                    | 普通注释                  |
|:---------------------|:--------------------------|:--------------------------|
| **结构化**           | 是（标签系统）            | 否（自由文本）            |
| **工具支持**         | 是（IDE、文档生成工具）   | 否                        |
| **类型检查**         | 是（静态分析工具）        | 否                        |
| **标准化**           | 是（PHPDoc 标准）         | 否                        |
| **推荐使用**         | 代码元素文档              | 代码逻辑说明              |

## 注意事项

1. **保持一致性**：在整个项目中保持 PHPDoc 格式的一致性。

2. **及时更新**：当代码更改时，及时更新 PHPDoc 注释。

3. **准确性**：确保 PHPDoc 中的类型和说明与实际代码一致。

4. **完整性**：为所有公共 API 添加完整的 PHPDoc 注释。

5. **简洁性**：保持注释简洁，避免冗余信息。

6. **工具使用**：使用 IDE 和工具自动生成和检查 PHPDoc。

7. **类型优先**：优先使用类型声明，PHPDoc 作为补充。

8. **文档生成**：定期使用 phpDocumentor 等工具生成和检查文档。

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
