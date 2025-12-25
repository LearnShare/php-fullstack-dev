# 2.12.1 模块基础

## 概述

模块化是代码组织的重要方式。通过将代码拆分为独立的文件/模块，可以提高代码的可维护性、可复用性和可测试性。本节将详细介绍模块的概念、优势以及如何编写独立的模块文件。

## 什么是模块

### 模块的定义

**模块**（Module）是一个独立的代码单元，通常包含：
- 相关的函数定义
- 类定义
- 常量定义
- 配置数据
- 可执行的初始化代码

### 模块的优势

**1. 代码复用**
- 一次编写，多处使用
- 避免重复代码
- 提高开发效率

**2. 代码组织**
- 按功能拆分，结构清晰
- 易于理解和导航
- 符合单一职责原则

**3. 易于维护**
- 修改一处，影响范围可控
- 便于定位和修复问题
- 降低维护成本

**4. 团队协作**
- 不同开发者可以并行工作
- 减少代码冲突
- 便于代码审查

**5. 可测试性**
- 模块可以独立测试
- 便于单元测试
- 提高代码质量

### 模块化的重要性

在现代软件开发中，模块化是构建大型应用的基础：

- **可扩展性**：新功能可以作为新模块添加
- **可维护性**：问题可以快速定位到特定模块
- **可重用性**：模块可以在不同项目中复用
- **可测试性**：模块可以独立测试和验证

## 如何编写独立的文件/模块

### 函数模块

创建一个包含相关函数的文件：

```php
<?php
// utils/math.php
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

/**
 * 计算两个数的差
 * 
 * @param int $a 被减数
 * @param int $b 减数
 * @return int 两数之差
 */
function subtract(int $a, int $b): int
{
    return $a - $b;
}

/**
 * 计算两个数的乘积
 * 
 * @param int $a 第一个数
 * @param int $b 第二个数
 * @return int 两数之积
 */
function multiply(int $a, int $b): int
{
    return $a * $b;
}

/**
 * 计算两个数的商
 * 
 * @param int $a 被除数
 * @param int $b 除数
 * @return float 两数之商
 * @throws DivisionByZeroError 当除数为 0 时抛出
 */
function divide(int $a, int $b): float
{
    if ($b === 0) {
        throw new DivisionByZeroError('Division by zero');
    }
    return $a / $b;
}
```

**函数模块编写要点**：
- 文件顶部必须有 `<?php` 标签
- 使用 `declare(strict_types=1)` 启用严格类型
- 函数应该有清晰的命名和文档注释（PHPDoc）
- 文件应该只包含相关的功能
- 函数应该处理错误情况（如除零错误）

### 类模块

创建一个包含类定义的文件：

```php
<?php
// src/User.php
declare(strict_types=1);

/**
 * 用户类
 * 
 * 表示系统中的用户实体
 */
class User
{
    private string $name;
    private int $age;
    private string $email;
    
    /**
     * 构造函数
     * 
     * @param string $name 用户名
     * @param int $age 年龄
     * @param string $email 邮箱地址
     */
    public function __construct(string $name, int $age, string $email)
    {
        $this->name = $name;
        $this->age = $age;
        $this->email = $email;
    }
    
    /**
     * 获取用户名
     * 
     * @return string 用户名
     */
    public function getName(): string
    {
        return $this->name;
    }
    
    /**
     * 获取年龄
     * 
     * @return int 年龄
     */
    public function getAge(): int
    {
        return $this->age;
    }
    
    /**
     * 获取邮箱
     * 
     * @return string 邮箱地址
     */
    public function getEmail(): string
    {
        return $this->email;
    }
    
    /**
     * 验证用户信息
     * 
     * @return bool 验证是否通过
     */
    public function validate(): bool
    {
        return !empty($this->name) 
            && $this->age > 0 
            && filter_var($this->email, FILTER_VALIDATE_EMAIL) !== false;
    }
}
```

**类模块编写要点**：
- 一个文件通常只包含一个类
- 文件名应该与类名相同（区分大小写）
- 类应该有完整的类型声明
- 使用 PHPDoc 注释文档化类和方法
- 遵循 PSR-1 和 PSR-12 编码规范

### 配置模块

创建一个配置文件：

```php
<?php
// config/app.php
declare(strict_types=1);

return [
    'app_name' => 'My Application',
    'version' => '1.0.0',
    'debug' => true,
    'timezone' => 'Asia/Shanghai',
    'database' => [
        'host' => 'localhost',
        'port' => 3306,
        'name' => 'mydb',
        'user' => 'root',
        'password' => '',
        'charset' => 'utf8mb4'
    ],
    'cache' => [
        'driver' => 'file',
        'ttl' => 3600
    ]
];
```

**配置模块编写要点**：
- 使用 `return` 语句返回配置数组
- 配置应该是只读的，避免在配置文件中执行代码
- 可以嵌套数组组织复杂配置
- 使用有意义的键名
- 可以按环境分离配置（development.php, production.php）

**环境配置示例**：

```php
<?php
// config/development.php
declare(strict_types=1);

return [
    'debug' => true,
    'error_reporting' => E_ALL,
    'display_errors' => true,
    'log_errors' => true
];
```

```php
<?php
// config/production.php
declare(strict_types=1);

return [
    'debug' => false,
    'error_reporting' => E_ALL & ~E_DEPRECATED & ~E_STRICT,
    'display_errors' => false,
    'log_errors' => true
];
```

### 常量模块

创建一个包含常量的文件：

```php
<?php
// constants.php
declare(strict_types=1);

// 应用常量
const APP_NAME = 'My Application';
const APP_VERSION = '1.0.0';

// 状态常量
const STATUS_ACTIVE = 1;
const STATUS_INACTIVE = 0;
const STATUS_PENDING = 2;

// 错误代码
const ERROR_NOT_FOUND = 404;
const ERROR_UNAUTHORIZED = 401;
const ERROR_FORBIDDEN = 403;

// 时间常量（秒）
const ONE_MINUTE = 60;
const ONE_HOUR = 3600;
const ONE_DAY = 86400;
const ONE_WEEK = 604800;
```

**常量模块编写要点**：
- 使用 `const` 关键字定义常量
- 常量名使用大写字母和下划线
- 相关常量分组组织
- 添加注释说明常量的用途

### 混合模块

一个文件可以包含多种内容（不推荐，但在某些场景下有用）：

```php
<?php
// helpers.php
declare(strict_types=1);

// 常量定义
const MAX_RETRY_COUNT = 3;
const DEFAULT_TIMEOUT = 30;

/**
 * 格式化日期
 * 
 * @param string $date 日期字符串
 * @return string 格式化后的日期
 */
function formatDate(string $date): string
{
    return date('Y-m-d', strtotime($date));
}

/**
 * 格式化货币
 * 
 * @param float $amount 金额
 * @return string 格式化后的货币字符串
 */
function formatCurrency(float $amount): string
{
    return '¥' . number_format($amount, 2);
}

/**
 * 格式化类
 */
class Formatter
{
    /**
     * 格式化百分比
     * 
     * @param float $value 值
     * @param int $decimals 小数位数
     * @return string 格式化后的百分比
     */
    public static function formatPercent(float $value, int $decimals = 2): string
    {
        return number_format($value * 100, $decimals) . '%';
    }
}
```

**混合模块使用场景**：
- 小型项目的辅助函数
- 工具类集合
- 临时代码组织

**注意**：在大型项目中，建议将不同类型的内容分离到不同的文件中。

## 文件组织和目录结构

### 推荐的目录结构

**小型项目**：

```
project/
├── index.php
├── config/
│   └── app.php
├── utils/
│   └── helpers.php
└── src/
    └── User.php
```

**中型项目**：

```
project/
├── index.php
├── config/
│   ├── app.php
│   ├── database.php
│   └── cache.php
├── src/
│   ├── Models/
│   │   └── User.php
│   ├── Controllers/
│   │   └── UserController.php
│   └── Services/
│       └── UserService.php
├── utils/
│   ├── math.php
│   └── string.php
└── constants.php
```

**大型项目（使用命名空间和 Composer）**：

```
project/
├── composer.json
├── index.php
├── config/
│   ├── app.php
│   └── database.php
└── src/
    ├── App/
    │   ├── Models/
    │   │   └── User.php
    │   ├── Controllers/
    │   │   └── UserController.php
    │   └── Services/
    │       └── UserService.php
    └── Utils/
        ├── Math.php
        └── StringHelper.php
```

### 文件命名规范

**函数和类文件**：
- 使用小写字母和下划线：`user_service.php`
- 或使用大驼峰：`UserService.php`（推荐，符合 PSR-1）
- 文件名应该与主要内容相关

**配置文件**：
- 使用小写字母和下划线：`app.php`, `database.php`
- 文件名应该描述配置内容

**常量文件**：
- 使用小写字母：`constants.php`
- 或使用描述性名称：`app_constants.php`

## 模块设计原则

### 单一职责原则

每个模块应该只负责一个功能或一组相关的功能：

```php
<?php
// ✅ 好的设计：单一职责
// utils/math.php - 只包含数学函数
function add(int $a, int $b): int { ... }
function subtract(int $a, int $b): int { ... }

// ❌ 不好的设计：混合职责
// utils/mixed.php - 包含数学、字符串、日期等多种功能
function add(int $a, int $b): int { ... }
function formatString(string $str): string { ... }
function formatDate(string $date): string { ... }
```

### 高内聚低耦合

- **高内聚**：模块内部的功能应该紧密相关
- **低耦合**：模块之间的依赖应该尽可能少

```php
<?php
// ✅ 好的设计：高内聚
// utils/string.php - 所有字符串相关函数
function trim(string $str): string { ... }
function upper(string $str): string { ... }
function lower(string $str): string { ... }

// ❌ 不好的设计：低内聚
// utils/misc.php - 各种不相关的函数
function trim(string $str): string { ... }
function add(int $a, int $b): int { ... }
function formatDate(string $date): string { ... }
```

### 接口设计

模块应该提供清晰的接口（函数签名、参数、返回值）：

```php
<?php
// ✅ 好的接口设计：清晰的参数和返回值
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

// ❌ 不好的接口设计：缺少类型和文档
function add($a, $b)
{
    return $a + $b;
}
```

## 完整示例

### 示例：用户管理模块

**目录结构**：

```
user-module/
├── User.php
├── UserRepository.php
└── UserService.php
```

**User.php**：

```php
<?php
// User.php
declare(strict_types=1);

class User
{
    public function __construct(
        private string $name,
        private int $age,
        private string $email
    ) {
    }
    
    public function getName(): string
    {
        return $this->name;
    }
    
    public function getAge(): int
    {
        return $this->age;
    }
    
    public function getEmail(): string
    {
        return $this->email;
    }
}
```

**UserRepository.php**：

```php
<?php
// UserRepository.php
declare(strict_types=1);

class UserRepository
{
    private array $users = [];
    
    public function save(User $user): void
    {
        $this->users[] = $user;
    }
    
    public function findByName(string $name): ?User
    {
        foreach ($this->users as $user) {
            if ($user->getName() === $name) {
                return $user;
            }
        }
        return null;
    }
    
    public function getAll(): array
    {
        return $this->users;
    }
}
```

**UserService.php**：

```php
<?php
// UserService.php
declare(strict_types=1);

class UserService
{
    public function __construct(
        private UserRepository $repository
    ) {
    }
    
    public function createUser(string $name, int $age, string $email): User
    {
        $user = new User($name, $age, $email);
        $this->repository->save($user);
        return $user;
    }
    
    public function getUserByName(string $name): ?User
    {
        return $this->repository->findByName($name);
    }
}
```

## 注意事项

1. **文件组织**：
   - 一个文件一个主要功能
   - 使用有意义的文件名
   - 保持目录结构清晰

2. **代码质量**：
   - 使用类型声明
   - 添加文档注释
   - 遵循编码规范

3. **依赖管理**：
   - 明确模块之间的依赖关系
   - 避免循环依赖
   - 使用接口降低耦合

4. **错误处理**：
   - 验证输入参数
   - 处理异常情况
   - 提供清晰的错误信息

## 练习

1. 创建一个数学工具模块，包含加、减、乘、除函数，并添加适当的错误处理。

2. 编写一个用户管理模块（User 类），包含完整的属性和方法，并添加验证功能。

3. 创建一个配置模块，支持开发和生产环境的配置分离。

4. 设计一个文件组织方案，将不同类型的模块组织到合适的目录中。

5. 实现一个简单的日志模块，包含日志记录和日志读取功能。
