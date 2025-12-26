# 2.3.5 最佳实践与综合练习

## 概述

变量和常量的使用需要遵循最佳实践，以提高代码质量和可维护性。本节总结变量和常量的最佳实践，包括命名规范、作用域管理、常量使用、引用使用等，并提供综合练习和代码审查清单，帮助巩固所学知识。

通过本节的学习，学员将掌握变量和常量的最佳实践，能够编写高质量、可维护的代码，并建立代码审查意识。

## 实践建议/原则

### 变量命名规范

**命名原则**：
1. **使用有意义的变量名**：变量名应该清晰表达变量的用途
2. **遵循命名约定**：使用驼峰命名（`camelCase`）或蛇形命名（`snake_case`），保持一致性
3. **避免缩写**：除非是通用缩写（如 `id`、`url`），避免使用缩写
4. **避免单字母变量**：除非是循环计数器（`$i`、`$j`、`$k`），避免使用单字母变量名
5. **布尔值使用前缀**：布尔变量使用 `is`、`has`、`can`、`should` 等前缀
6. **数组使用复数形式**：数组变量使用复数形式（`$users`、`$items`）

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：有意义的变量名
$userName = "Alice";
$userAge = 25;
$isActive = true;
$hasPermission = false;
$canEdit = true;
$users = ["Alice", "Bob"];

// 不推荐：无意义的变量名
$n = "Alice";
$x = 25;
$flag = true;
$arr = ["Alice", "Bob"];
```

### 变量作用域最佳实践

**最佳实践**：
1. **优先使用局部变量**：优先使用局部变量，避免全局变量
2. **使用参数传递**：通过函数参数传递数据，而不是使用全局变量
3. **使用依赖注入**：使用依赖注入替代全局变量
4. **理解作用域规则**：理解局部变量和全局变量的作用域规则
5. **避免变量污染**：避免在全局作用域创建过多变量

**示例**：

```php
<?php
declare(strict_types=1);

// 不推荐：使用全局变量
$config = [];

function loadConfig(): void
{
    global $config;
    $config = ['key' => 'value'];
}

// 推荐：使用参数传递
function loadConfig(): array
{
    return ['key' => 'value'];
}

$config = loadConfig();

// 推荐：使用依赖注入
class ConfigLoader
{
    public function __construct(
        private array $config = []
    ) {}
    
    public function load(): array
    {
        return $this->config;
    }
}
```

### 常量使用最佳实践

**最佳实践**：
1. **使用常量存储配置值**：将配置值存储在常量中，避免硬编码
2. **遵循命名规范**：常量名使用大写字母和下划线（`UPPER_SNAKE_CASE`）
3. **使用类常量组织**：将相关常量组织在类中，提高可维护性
4. **避免硬编码**：避免在代码中硬编码值，使用常量替代
5. **集中管理**：将常量集中管理，便于维护

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：使用类常量组织
class AppConfig
{
    public const APP_NAME = "My Application";
    public const APP_VERSION = "1.0.0";
    public const MAX_USERS = 100;
    public const DEFAULT_TIMEOUT = 30;
}

// 使用常量
echo AppConfig::APP_NAME . "\n";
echo AppConfig::MAX_USERS . "\n";

// 不推荐：硬编码
function processUser(int $userId): void
{
    if ($userId > 100) {  // 硬编码
        // ...
    }
}

// 推荐：使用常量
function processUser(int $userId): void
{
    if ($userId > AppConfig::MAX_USERS) {
        // ...
    }
}
```

### 引用使用最佳实践

**最佳实践**：
1. **谨慎使用引用**：只在必要时使用引用（如避免大数组复制）
2. **明确标注**：使用引用时明确标注，添加注释说明
3. **避免循环引用**：注意避免循环引用导致的内存泄漏
4. **及时取消引用**：使用 `unset()` 及时取消不需要的引用

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐：明确标注引用
/**
 * 处理大数组（按引用传递，避免复制）
 *
 * @param array $data 要处理的数据（按引用传递，会被修改）
 */
function processLargeArray(array &$data): void
{
    $data[0] = 999;
}

// 不推荐：未标注的引用
function processArray(array &$data): void
{
    $data[0] = 999;  // 可能意外修改外部变量
}
```

### 可变变量和可变函数最佳实践

**最佳实践**：
1. **避免过度使用**：可变变量和可变函数会降低代码可读性
2. **使用数组替代**：通常使用数组比可变变量更清晰
3. **验证函数存在**：使用可变函数时，使用 `function_exists()` 检查
4. **避免用户输入**：不要直接使用用户输入作为函数名

**示例**：

```php
<?php
declare(strict_types=1);

// 不推荐：可变变量
$name = "user";
$$name = "John";  // 难以理解

// 推荐：使用数组
$data = [];
$data['user'] = "John";  // 更清晰

// 推荐：验证函数存在
$func = "strlen";
if (function_exists($func)) {
    $result = $func("Hello");
}
```

## 完整代码示例

### 示例 1：良好的变量命名

```php
<?php
declare(strict_types=1);

// 用户信息
$userName = "Alice";
$userEmail = "alice@example.com";
$userAge = 25;

// 布尔值使用前缀
$isActive = true;
$hasPermission = false;
$canEdit = true;
$shouldNotify = true;

// 数组使用复数形式
$users = ["Alice", "Bob", "Charlie"];
$items = [1, 2, 3, 4, 5];

// 配置信息
$maxRetries = 3;
$timeoutSeconds = 30;
$apiEndpoint = "https://api.example.com";

// 函数使用有意义的参数名
function sendEmail(string $recipientEmail, string $subject, string $body): void
{
    // 实现发送邮件逻辑
}
```

### 示例 2：避免全局变量

```php
<?php
declare(strict_types=1);

// 不推荐：使用全局变量
$databaseConnection = null;

function getConnection()
{
    global $databaseConnection;
    if ($databaseConnection === null) {
        $databaseConnection = new PDO("mysql:host=localhost;dbname=test", "user", "pass");
    }
    return $databaseConnection;
}

// 推荐：使用依赖注入
class DatabaseService
{
    public function __construct(
        private ?PDO $connection = null
    ) {}
    
    public function getConnection(): PDO
    {
        if ($this->connection === null) {
            $this->connection = new PDO("mysql:host=localhost;dbname=test", "user", "pass");
        }
        return $this->connection;
    }
}
```

### 示例 3：使用常量替代硬编码

```php
<?php
declare(strict_types=1);

// 定义常量
class HttpStatus
{
    public const OK = 200;
    public const NOT_FOUND = 404;
    public const INTERNAL_ERROR = 500;
}

class UserRole
{
    public const ADMIN = "admin";
    public const USER = "user";
    public const GUEST = "guest";
}

// 使用常量
function checkAccess(string $userRole): bool
{
    return $userRole === UserRole::ADMIN;
}

function sendResponse(int $statusCode): void
{
    if ($statusCode === HttpStatus::OK) {
        // 处理成功响应
    } elseif ($statusCode === HttpStatus::NOT_FOUND) {
        // 处理未找到响应
    }
}
```

## 代码审查清单

完成本章学习后，使用以下清单审查代码：

### 变量命名

- [ ] 变量名是否有意义，清晰表达变量用途
- [ ] 是否遵循命名约定（驼峰命名或蛇形命名）
- [ ] 是否避免使用缩写和单字母变量名
- [ ] 布尔变量是否使用 `is`、`has`、`can` 等前缀
- [ ] 数组变量是否使用复数形式

### 变量作用域

- [ ] 是否避免使用全局变量
- [ ] 是否优先使用局部变量和参数传递
- [ ] 是否使用依赖注入替代全局变量
- [ ] 是否理解变量作用域规则

### 常量使用

- [ ] 是否使用常量存储配置值
- [ ] 常量名是否遵循命名规范（`UPPER_SNAKE_CASE`）
- [ ] 是否使用类常量组织相关常量
- [ ] 是否避免硬编码值

### 引用使用

- [ ] 是否谨慎使用引用
- [ ] 使用引用时是否明确标注
- [ ] 是否避免循环引用
- [ ] 是否及时取消不需要的引用

### 代码质量

- [ ] 代码是否清晰易懂
- [ ] 是否添加必要的注释
- [ ] 是否遵循 PSR 编码规范
- [ ] 是否通过语法检查

## 综合练习

### 练习 1：重构变量命名

**任务**：重构以下代码，改进变量命名

```php
<?php
declare(strict_types=1);

// 原始代码（需要重构）
$n = "John";
$a = 25;
$e = "john@example.com";
$f = true;
$arr = [1, 2, 3];

function p($d)
{
    echo $d;
}
```

**要求**：
- 使用有意义的变量名
- 遵循命名规范
- 布尔变量使用前缀
- 数组使用复数形式

### 练习 2：消除全局变量

**任务**：重构以下代码，消除全局变量的使用

```php
<?php
declare(strict_types=1);

// 原始代码（需要重构）
$config = [];

function loadConfig(): void
{
    global $config;
    $config = ['host' => 'localhost', 'port' => 3306];
}

function getConfig(): array
{
    global $config;
    return $config;
}
```

**要求**：
- 使用参数传递或返回值
- 使用依赖注入
- 避免全局变量

### 练习 3：使用常量替代硬编码

**任务**：重构以下代码，使用常量替代硬编码值

```php
<?php
declare(strict_types=1);

// 原始代码（需要重构）
function checkUser(int $userId): bool
{
    if ($userId > 100) {
        return false;
    }
    return true;
}

function processOrder(string $status): void
{
    if ($status === "pending") {
        // 处理待处理订单
    } elseif ($status === "completed") {
        // 处理已完成订单
    }
}
```

**要求**：
- 定义常量替代硬编码值
- 使用类常量组织相关常量
- 提高代码可维护性

### 练习 4：实现静态变量计数器

**任务**：使用静态变量实现函数调用计数器

**要求**：
- 统计不同函数的调用次数
- 提供获取统计信息的方法
- 使用有意义的变量名

### 练习 5：创建配置管理类

**任务**：创建一个配置管理类，使用类常量存储配置

**要求**：
- 使用类常量组织配置
- 提供获取配置的方法
- 遵循命名规范
- 添加文档注释

## 最佳实践总结

### 变量命名

- ✅ 使用有意义的变量名
- ✅ 遵循命名约定（camelCase 或 snake_case）
- ✅ 布尔值使用 `is`、`has`、`can` 前缀
- ✅ 数组使用复数形式
- ❌ 避免缩写和单字母变量名

### 变量作用域

- ✅ 优先使用局部变量
- ✅ 使用参数传递数据
- ✅ 使用依赖注入
- ❌ 避免全局变量

### 常量使用

- ✅ 使用常量存储配置值
- ✅ 遵循命名规范（UPPER_SNAKE_CASE）
- ✅ 使用类常量组织
- ❌ 避免硬编码

### 引用使用

- ✅ 谨慎使用引用
- ✅ 明确标注引用
- ✅ 及时取消引用
- ❌ 避免循环引用

### 代码质量

- ✅ 代码清晰易懂
- ✅ 添加必要注释
- ✅ 遵循 PSR 规范
- ✅ 通过语法检查

## 相关章节

- **2.3.1 变量基础**：了解变量的基本概念
- **2.3.2 引用与可变变量**：了解引用和可变变量的使用
- **2.3.3 常量**：了解常量的定义和使用
- **2.3.4 全局与静态**：了解全局变量和静态变量
- **2.13 代码规范**：深入学习代码规范

## 练习任务

1. **变量命名重构**：
   - 重构代码，改进变量命名
   - 确保变量名有意义且遵循规范
   - 布尔变量使用前缀，数组使用复数形式

2. **消除全局变量**：
   - 将使用全局变量的代码重构为使用参数传递
   - 实现依赖注入替代全局变量
   - 测试重构后的代码

3. **常量替代硬编码**：
   - 识别代码中的硬编码值
   - 定义常量替代硬编码值
   - 使用类常量组织相关常量

4. **静态变量实践**：
   - 使用静态变量实现计数器
   - 使用静态变量实现简单缓存
   - 理解静态变量的使用场景

5. **综合项目**：
   - 创建一个完整的应用
   - 遵循所有最佳实践
   - 进行代码审查，确保代码质量
   - 编写文档说明代码设计
