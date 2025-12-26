# 2.13.2 PHPDoc

## 概述

PHPDoc 是 PHP 的文档注释标准。本节详细介绍 PHPDoc 基本语法、常用标签（`@param`、`@return`、`@throws`、`@var`、`@deprecated`）、类和方法文档、完整示例。

理解 PHPDoc 的使用对于编写规范的代码文档非常重要。掌握 PHPDoc 标签可以帮助生成 API 文档、提供 IDE 代码提示、支持静态分析工具。

## 特性

- **文档生成**：可以生成 API 文档
- **IDE 支持**：IDE 可以提供代码提示和类型信息
- **类型提示**：提供类型信息，帮助静态分析工具
- **标准化**：遵循 PHPDoc 标准，工具广泛支持

## 语法/定义

### PHPDoc 语法

**基本语法**：`/** ... */`

**位置**：函数、类、属性、常量之前

**格式**：
- 以 `/**` 开始
- 以 `*/` 结束
- 每行以 `*` 开头（可选，但推荐）

### 常用标签

#### @param - 参数说明

**语法**：`@param type $paramName description`

**功能**：说明函数/方法的参数类型、名称和描述

**示例**：
```php
/**
 * @param int $age 用户年龄
 * @param string|null $name 用户名称，可选
 */
```

#### @return - 返回值说明

**语法**：`@return type description`

**功能**：说明函数/方法的返回值类型和描述

**示例**：
```php
/**
 * @return string 用户名称
 * @return array<string, mixed> 配置数组
 */
```

#### @throws - 异常说明

**语法**：`@throws ExceptionType description`

**功能**：说明函数/方法可能抛出的异常

**示例**：
```php
/**
 * @throws InvalidArgumentException 当参数无效时
 * @throws RuntimeException 当操作失败时
 */
```

#### @var - 变量类型

**语法**：`@var type description`

**功能**：说明变量、属性的类型

**示例**：
```php
/**
 * @var string 用户名称
 * @var array<int, User> 用户列表
 */
```

#### @deprecated - 已废弃标记

**语法**：`@deprecated version description`

**功能**：标记已废弃的函数/方法

**示例**：
```php
/**
 * @deprecated 2.0.0 使用新方法代替
 */
```

## 基本用法

### 示例 1：函数文档

```php
<?php
declare(strict_types=1);

/**
 * 计算两个数的和
 * 
 * @param int $a 第一个数
 * @param int $b 第二个数
 * @return int 两数之和
 * @throws InvalidArgumentException 当参数无效时
 */
function add(int $a, int $b): int
{
    if ($a < 0 || $b < 0) {
        throw new InvalidArgumentException("Parameters must be non-negative");
    }
    
    return $a + $b;
}
```

### 示例 2：类文档

```php
<?php
declare(strict_types=1);

/**
 * 用户服务类
 * 
 * 提供用户相关的业务逻辑处理
 * 
 * @package App\Service
 * @author John Doe
 * @since 1.0.0
 */
class UserService
{
    /**
     * 用户名称
     * 
     * @var string
     */
    private string $name;
    
    /**
     * 获取用户名称
     * 
     * @return string 用户名称
     */
    public function getName(): string
    {
        return $this->name;
    }
}
```

### 示例 3：方法文档

```php
<?php
declare(strict_types=1);

class UserRepository
{
    /**
     * 根据 ID 查找用户
     * 
     * @param int $id 用户 ID
     * @return User|null 用户对象，如果不存在返回 null
     * @throws DatabaseException 当数据库操作失败时
     */
    public function findById(int $id): ?User
    {
        // 实现逻辑
    }
    
    /**
     * 查找所有用户
     * 
     * @param int $limit 限制数量，默认 10
     * @param int $offset 偏移量，默认 0
     * @return array<int, User> 用户数组
     */
    public function findAll(int $limit = 10, int $offset = 0): array
    {
        // 实现逻辑
    }
}
```

## 完整代码示例

### 示例 1：完整的类文档

```php
<?php
declare(strict_types=1);

namespace App\Service;

use App\Model\User;
use App\Exception\UserNotFoundException;

/**
 * 用户服务类
 * 
 * 提供用户相关的业务逻辑处理，包括用户查找、创建、更新等操作
 * 
 * @package App\Service
 * @author John Doe
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
     * @return User 用户对象
     * @throws UserNotFoundException 当用户不存在时
     */
    public function getUserById(int $id): User
    {
        $user = $this->repository->find($id);
        
        if ($user === null) {
            throw new UserNotFoundException("User not found: {$id}");
        }
        
        return $user;
    }
    
    /**
     * 创建用户
     * 
     * @param string $name 用户名称
     * @param string $email 用户邮箱
     * @param int $age 用户年龄
     * @return User 创建的用户对象
     * @throws InvalidArgumentException 当参数无效时
     */
    public function createUser(string $name, string $email, int $age): User
    {
        if (empty($name) || empty($email)) {
            throw new InvalidArgumentException("Name and email are required");
        }
        
        if ($age < 0) {
            throw new InvalidArgumentException("Age must be non-negative");
        }
        
        return $this->repository->create($name, $email, $age);
    }
}
```

### 示例 2：复杂类型文档

```php
<?php
declare(strict_types=1);

/**
 * 处理用户数据
 * 
 * @param array<string, mixed> $userData 用户数据数组
 * @param callable(string, int): bool $validator 验证函数
 * @return array{name: string, age: int, email: string} 处理后的用户数据
 * @throws InvalidArgumentException 当数据无效时
 */
function processUserData(array $userData, callable $validator): array
{
    // 实现逻辑
}

/**
 * 获取配置值
 * 
 * @param array<string, mixed> $config 配置数组
 * @param string $key 配置键
 * @param mixed $default 默认值
 * @return mixed 配置值
 */
function getConfig(array $config, string $key, mixed $default = null): mixed
{
    return $config[$key] ?? $default;
}
```

### 示例 3：已废弃方法文档

```php
<?php
declare(strict_types=1);

class UserService
{
    /**
     * 获取用户信息（已废弃）
     * 
     * @param int $id 用户 ID
     * @return array<string, mixed> 用户信息数组
     * @deprecated 2.0.0 使用 getUserById() 代替
     * @see UserService::getUserById()
     */
    public function getUserInfo(int $id): array
    {
        // 实现逻辑
    }
    
    /**
     * 根据 ID 获取用户
     * 
     * @param int $id 用户 ID
     * @return User 用户对象
     * @since 2.0.0
     */
    public function getUserById(int $id): User
    {
        // 实现逻辑
    }
}
```

## 使用场景

### API 文档生成

- **自动生成**：使用工具自动生成 API 文档
- **在线文档**：生成在线 API 文档
- **文档维护**：通过代码注释维护文档

### IDE 支持

- **代码提示**：IDE 可以提供参数和返回值提示
- **类型检查**：IDE 可以进行类型检查
- **自动完成**：IDE 可以提供自动完成功能

### 静态分析

- **类型检查**：静态分析工具使用 PHPDoc 进行类型检查
- **错误检测**：帮助发现类型错误
- **代码质量**：提高代码质量

## 注意事项

### 格式规范

- **遵循标准**：遵循 PHPDoc 格式规范
- **标签顺序**：按照标准顺序排列标签
- **描述完整**：提供完整的参数和返回值描述

### 及时更新

- **代码变更**：代码变更时及时更新文档
- **保持同步**：确保文档与代码同步
- **版本管理**：使用 `@since`、`@deprecated` 管理版本

### 完整性

- **公共 API**：为公共 API 编写完整文档
- **复杂逻辑**：为复杂逻辑添加文档
- **参数说明**：为所有参数提供说明

## 常见问题

### 问题 1：文档格式不规范

**症状**：IDE 无法识别文档注释

**原因**：不符合 PHPDoc 格式规范

**错误示例**：

```php
<?php
declare(strict_types=1);

// 错误：使用单行注释
// @param int $a 第一个数
function add(int $a, int $b): int
{
    return $a + $b;
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

/**
 * 计算两个数的和
 * 
 * @param int $a 第一个数
 * @param int $b 第二个数
 * @return int 两数之和
 */
function add(int $a, int $b): int
{
    return $a + $b;
}
```

### 问题 2：文档过期

**症状**：文档与代码不一致

**原因**：代码变更后未更新文档

**解决方法**：
- 代码变更时同步更新文档
- 使用工具检查文档一致性
- 代码审查时检查文档

### 问题 3：类型声明不准确

**症状**：PHPDoc 类型与代码类型不一致

**原因**：类型声明错误或未更新

**解决方法**：

```php
<?php
declare(strict_types=1);

/**
 * 获取用户列表
 * 
 * @param int $limit 限制数量
 * @return array<int, User> 用户数组（类型必须与代码一致）
 */
function getUsers(int $limit): array
{
    // 返回 User 对象数组
    return [];
}
```

## 最佳实践

### 文档编写

- **完整描述**：为函数/方法提供完整描述
- **参数说明**：为所有参数提供类型和描述
- **返回值说明**：说明返回值的类型和含义
- **异常说明**：说明可能抛出的异常

### 类型声明

- **准确类型**：使用准确的类型声明
- **复杂类型**：使用 PHPDoc 类型语法声明复杂类型
- **与代码一致**：确保 PHPDoc 类型与代码类型一致

### 工具使用

- **文档生成**：使用工具生成 API 文档
- **类型检查**：使用静态分析工具检查类型
- **格式检查**：使用工具检查文档格式

## 对比分析

### PHPDoc vs 类型声明

| 特性 | PHPDoc | 类型声明 |
|:-----|:-------|:---------|
| 运行时检查 | 否 | 是（PHP 7.0+） |
| IDE 支持 | 是 | 是 |
| 文档生成 | 是 | 否 |
| 推荐度 | 推荐（补充） | 推荐（必需） |

**选择建议**：
- **类型声明**：优先使用类型声明（PHP 7.0+）
- **PHPDoc**：作为补充，提供更详细的文档

## 相关章节

- **2.4.5 类型声明**：了解类型声明
- **2.13.1 PSR 标准**：了解编码标准
- **2.13.3 静态代码分析**：了解静态分析工具

## 练习任务

1. **PHPDoc 编写练习**：
   - 为函数添加 PHPDoc 注释
   - 为类添加 PHPDoc 注释
   - 为方法添加 PHPDoc 注释
   - 测试 IDE 代码提示

2. **类型声明练习**：
   - 使用 PHPDoc 声明复杂类型
   - 使用数组类型语法
   - 使用联合类型语法
   - 测试类型检查

3. **文档生成练习**：
   - 使用工具生成 API 文档
   - 查看生成的文档
   - 测试文档链接
   - 验证文档完整性

4. **实际应用练习**：
   - 为项目添加完整的 PHPDoc 注释
   - 生成 API 文档
   - 使用静态分析工具检查
   - 测试 IDE 支持

5. **综合练习**：
   - 创建一个包含完整 PHPDoc 的项目
   - 生成 API 文档
   - 使用工具检查文档
   - 进行代码审查
