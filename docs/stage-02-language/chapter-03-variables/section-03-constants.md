# 2.3.3 常量

## 概述

常量是不可改变的值，用于存储配置、魔法值等。本节详细介绍常量的定义方法（`const`、`define()`）、常量类型、类常量、魔术常量等，帮助理解常量的使用场景和最佳实践。

理解常量的使用可以帮助避免硬编码，提高代码的可维护性。常量在全局作用域可用，适合存储配置信息和不会改变的值。

## 特性

- **不可改变**：常量一旦定义就不能修改
- **全局作用域**：常量在全局作用域可用（包括函数内部）
- **命名规范**：通常使用大写字母和下划线
- **编译时定义**：`const` 在编译时定义，性能更好
- **运行时定义**：`define()` 在运行时定义，更灵活

## 语法/定义

### const - 定义常量

**语法**：`const CONSTANT_NAME = value;`

**特点**：
- 编译时定义，性能更好
- 只能在类或顶层作用域定义
- 不能使用表达式（PHP 5.6+ 可以使用常量表达式）
- 推荐用于类常量和顶层常量

**限制**：
- 不能在函数内部定义
- 不能在条件语句中定义
- 值必须是常量表达式（PHP 5.6+）

### define() - 定义常量

**语法**：`define(string $name, mixed $value, bool $case_insensitive = false): bool`

**参数**：
- `$name`：常量名（类型为 `string`）
- `$value`：常量的值（可以是标量类型或数组）
- `$case_insensitive`：可选，是否大小写不敏感，默认为 `false`

**返回值**：成功返回 `true`，失败返回 `false`（如果常量已存在）

**特点**：
- 运行时定义，更灵活
- 可以在任何地方定义（包括函数内部、条件语句中）
- 可以使用表达式
- 适合动态定义常量

### 类常量

**语法**：`class ClassName { const CONSTANT_NAME = value; }`

**访问方式**：
- `ClassName::CONSTANT_NAME`：通过类名访问
- `$obj::CONSTANT_NAME`：通过对象访问（PHP 5.3+）
- `self::CONSTANT_NAME`：在类内部使用 `self` 访问
- `static::CONSTANT_NAME`：在类内部使用 `static` 访问（支持后期静态绑定）

**特点**：
- 属于类，不属于实例
- 可以通过类名或对象访问
- 支持可见性修饰符（`public`、`protected`、`private`）

## 基本用法

### 示例 1：const 定义常量

```php
<?php
declare(strict_types=1);

// 顶层常量
const APP_NAME = "My App";
const MAX_USERS = 100;
const VERSION = "1.0.0";

echo APP_NAME . "\n";
echo MAX_USERS . "\n";
echo VERSION . "\n";

// 类常量
class Config
{
    public const DB_HOST = "localhost";
    public const DB_PORT = 3306;
    public const DB_NAME = "mydb";
}

echo Config::DB_HOST . "\n";
echo Config::DB_PORT . "\n";
echo Config::DB_NAME . "\n";
```

**执行**：

```bash
php const-example.php
```

**输出**：

```
My App
100
1.0.0
localhost
3306
mydb
```

### 示例 2：define() 定义常量

```php
<?php
declare(strict_types=1);

// 基本定义
define("APP_NAME", "My App");
define("MAX_USERS", 100);
define("DEBUG_MODE", true);

echo APP_NAME . "\n";
echo MAX_USERS . "\n";
echo DEBUG_MODE ? "true" : "false";
echo "\n";

// 在函数内部定义
function defineConstants(): void
{
    define("FUNCTION_CONSTANT", "Defined in function");
}

defineConstants();
echo FUNCTION_CONSTANT . "\n";

// 条件定义
if (true) {
    define("CONDITIONAL_CONSTANT", "Defined conditionally");
}
echo CONDITIONAL_CONSTANT . "\n";

// 数组常量（PHP 7.0+）
define("CONFIG", [
    'host' => 'localhost',
    'port' => 3306,
]);
echo CONFIG['host'] . "\n";
```

**执行**：

```bash
php define-example.php
```

**输出**：

```
My App
100
true
Defined in function
Defined conditionally
localhost
```

### 示例 3：类常量

```php
<?php
declare(strict_types=1);

class Database
{
    public const DRIVER_MYSQL = "mysql";
    public const DRIVER_POSTGRES = "postgres";
    public const DRIVER_SQLITE = "sqlite";
    
    private const DEFAULT_PORT = 3306;
    
    public function getDefaultPort(): int
    {
        return self::DEFAULT_PORT;
    }
}

// 通过类名访问
echo Database::DRIVER_MYSQL . "\n";
echo Database::DRIVER_POSTGRES . "\n";

// 通过对象访问
$db = new Database();
echo $db::DRIVER_SQLITE . "\n";
echo $db->getDefaultPort() . "\n";

// 在类内部使用
class Connection
{
    public const TIMEOUT = 30;
    
    public function getTimeout(): int
    {
        return self::TIMEOUT;
    }
}
```

**执行**：

```bash
php class-constants.php
```

**输出**：

```
mysql
postgres
sqlite
3306
30
```

### 示例 4：魔术常量

```php
<?php
declare(strict_types=1);

// __LINE__：当前行号
echo "Current line: " . __LINE__ . "\n";

// __FILE__：当前文件路径
echo "Current file: " . __FILE__ . "\n";

// __DIR__：当前文件所在目录
echo "Current directory: " . __DIR__ . "\n";

// __FUNCTION__：当前函数名
function testFunction(): void
{
    echo "Current function: " . __FUNCTION__ . "\n";
}
testFunction();

// __CLASS__：当前类名
class TestClass
{
    public function showClass(): void
    {
        echo "Current class: " . __CLASS__ . "\n";
        echo "Current method: " . __METHOD__ . "\n";
    }
}

$obj = new TestClass();
$obj->showClass();

// __NAMESPACE__：当前命名空间
namespace MyNamespace;
echo "Current namespace: " . __NAMESPACE__ . "\n";
```

**执行**：

```bash
php magic-constants.php
```

**输出**：

```
Current line: 5
Current file: /path/to/magic-constants.php
Current directory: /path/to
Current function: testFunction
Current class: TestClass
Current method: TestClass::showClass
Current namespace: MyNamespace
```

## 完整代码示例

### 示例 1：配置常量

```php
<?php
declare(strict_types=1);

// 应用配置
const APP_NAME = "My Application";
const APP_VERSION = "1.0.0";
const APP_ENV = "development";

// 数据库配置
class DatabaseConfig
{
    public const HOST = "localhost";
    public const PORT = 3306;
    public const NAME = "mydb";
    public const USER = "root";
    public const PASSWORD = "";
}

// 使用配置
echo "Application: " . APP_NAME . " v" . APP_VERSION . "\n";
echo "Database: " . DatabaseConfig::HOST . ":" . DatabaseConfig::PORT . "\n";
```

### 示例 2：枚举常量

```php
<?php
declare(strict_types=1);

// 使用类常量实现枚举
class UserStatus
{
    public const ACTIVE = "active";
    public const INACTIVE = "inactive";
    public const SUSPENDED = "suspended";
}

class OrderStatus
{
    public const PENDING = "pending";
    public const PROCESSING = "processing";
    public const COMPLETED = "completed";
    public const CANCELLED = "cancelled";
}

// 使用枚举常量
$userStatus = UserStatus::ACTIVE;
$orderStatus = OrderStatus::PENDING;

echo "User status: {$userStatus}\n";
echo "Order status: {$orderStatus}\n";
```

**说明**：PHP 8.1+ 支持原生枚举（enum），详见阶段三：面向对象编程基础。

### 示例 3：常量检查

```php
<?php
declare(strict_types=1);

// 定义常量
define("MY_CONSTANT", "value");

// 检查常量是否存在
if (defined("MY_CONSTANT")) {
    echo "MY_CONSTANT is defined: " . MY_CONSTANT . "\n";
} else {
    echo "MY_CONSTANT is not defined\n";
}

// 获取所有已定义的常量
$constants = get_defined_constants();
echo "Total constants: " . count($constants) . "\n";

// 获取用户定义的常量
$userConstants = get_defined_constants(true)['user'] ?? [];
echo "User constants: " . count($userConstants) . "\n";
```

## 使用场景

### 配置值

- **应用配置**：存储应用名称、版本、环境等
- **数据库配置**：存储数据库连接信息
- **API 配置**：存储 API 密钥、端点等

### 魔法值

- **状态值**：存储状态枚举值
- **错误代码**：存储错误代码常量
- **特殊值**：存储不会改变的固定值

### 类常量

- **枚举值**：使用类常量实现枚举（PHP 8.1 前）
- **配置值**：存储类相关的配置
- **默认值**：存储类的默认值

## 注意事项

### 命名规范

- **使用大写字母**：常量名通常使用大写字母
- **使用下划线**：多个单词使用下划线分隔
- **有意义的名称**：常量名应该清晰表达常量的含义

**示例**：

```php
<?php
declare(strict_types=1);

// 推荐
const MAX_RETRY_COUNT = 3;
const DEFAULT_TIMEOUT = 30;

// 不推荐
const maxRetryCount = 3;
const DEFAULTTIMEOUT = 30;
```

### 作用域

- **全局作用域**：常量在全局作用域可用（包括函数内部）
- **类常量作用域**：类常量属于类，可以通过类名或对象访问
- **命名空间**：常量可以定义在命名空间中

### 类型限制

- **标量类型**：常量可以是标量类型（字符串、整数、浮点数、布尔值）
- **数组常量**：PHP 7.0+ 支持数组常量
- **对象常量**：不支持对象常量

### const vs define()

| 特性 | const | define() |
|:-----|:------|:---------|
| 定义时机 | 编译时 | 运行时 |
| 性能 | 更好 | 较差 |
| 定义位置 | 类或顶层 | 任何地方 |
| 表达式 | 常量表达式 | 任意表达式 |
| 推荐度 | 推荐 | 按需使用 |

**选择建议**：
- **类常量**：使用 `const`
- **顶层常量**：优先使用 `const`，需要表达式时使用 `define()`
- **条件定义**：使用 `define()`

## 常见问题

### 问题 1：常量未定义

**症状**：

```
Warning: Use of undefined constant CONSTANT_NAME
```

**原因**：常量未定义就使用

**错误示例**：

```php
<?php
declare(strict_types=1);

// 错误：常量未定义
echo MY_CONSTANT;  // Warning
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：先定义再使用
define("MY_CONSTANT", "value");
echo MY_CONSTANT . "\n";

// 方法2：使用 defined() 检查
if (defined("MY_CONSTANT")) {
    echo MY_CONSTANT . "\n";
} else {
    echo "Constant not defined\n";
}
```

### 问题 2：常量命名冲突

**症状**：常量重复定义或与内置常量冲突

**原因**：常量名重复定义或使用了 PHP 内置常量名

**解决方法**：

1. **使用命名空间**：将常量定义在命名空间中

```php
<?php
declare(strict_types=1);

namespace MyApp;

const APP_NAME = "My App";
echo \MyApp\APP_NAME . "\n";
```

2. **使用类常量**：使用类常量组织相关常量

```php
<?php
declare(strict_types=1);

class Config
{
    public const APP_NAME = "My App";
}

echo Config::APP_NAME . "\n";
```

3. **检查常量是否存在**：使用 `defined()` 检查

```php
<?php
declare(strict_types=1);

if (!defined("MY_CONSTANT")) {
    define("MY_CONSTANT", "value");
}
```

### 问题 3：const 不能在函数内定义

**症状**：

```
Parse error: syntax error, unexpected 'const'
```

**原因**：`const` 不能在函数内部定义

**错误示例**：

```php
<?php
declare(strict_types=1);

function defineConstant(): void
{
    const MY_CONSTANT = "value";  // 错误
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：在函数外部定义
const MY_CONSTANT = "value";

// 方法2：使用 define()
function defineConstant(): void
{
    define("MY_CONSTANT", "value");  // 正确
}
```

### 问题 4：常量值不能是对象

**症状**：尝试将对象赋值给常量

**原因**：常量只能是标量类型或数组（PHP 7.0+）

**解决方法**：

```php
<?php
declare(strict_types=1);

// 错误：不能使用对象
// const MY_CONSTANT = new stdClass();  // 错误

// 正确：使用标量类型或数组
const MY_CONSTANT = "value";
const MY_ARRAY = [1, 2, 3];
```

## 最佳实践

### 常量命名

- **使用大写字母**：常量名使用大写字母
- **使用下划线分隔**：多个单词使用下划线分隔
- **有意义的名称**：常量名应该清晰表达常量的含义
- **避免缩写**：除非是通用缩写，避免使用缩写

### 常量组织

- **使用类常量**：将相关常量组织在类中
- **使用命名空间**：将常量定义在命名空间中
- **分组管理**：按功能分组管理常量

### 常量定义

- **优先使用 const**：优先使用 `const` 定义常量（性能更好）
- **需要表达式时使用 define()**：需要表达式或条件定义时使用 `define()`
- **检查常量是否存在**：使用 `defined()` 检查常量是否存在

### 避免硬编码

- **使用常量替代魔法值**：避免在代码中硬编码值
- **集中管理配置**：将配置值集中管理
- **便于维护**：使用常量便于后续修改和维护

## 对比分析

### const vs define()

| 特性 | const | define() |
|:-----|:------|:---------|
| 定义时机 | 编译时 | 运行时 |
| 性能 | 更好 | 较差 |
| 定义位置 | 类或顶层 | 任何地方 |
| 表达式 | 常量表达式 | 任意表达式 |
| 条件定义 | 不支持 | 支持 |
| 推荐度 | 推荐 | 按需使用 |

**选择建议**：
- **类常量**：使用 `const`
- **顶层常量**：优先使用 `const`，需要表达式时使用 `define()`
- **条件定义**：使用 `define()`

### 常量 vs 变量

| 特性 | 常量 | 变量 |
|:-----|:-----|:------|
| 可变性 | 不可变 | 可变 |
| 作用域 | 全局 | 局部/全局 |
| 命名 | 大写 | 小写 |
| 定义 | const/define() | $variable = value |
| 使用场景 | 配置、魔法值 | 数据存储 |

**选择建议**：
- **不会改变的值**：使用常量
- **会改变的值**：使用变量
- **配置值**：使用常量
- **临时数据**：使用变量

## 相关章节

- **2.3.1 变量基础**：了解变量的基本概念
- **2.3.4 全局与静态**：了解全局变量和静态变量
- **阶段三：面向对象编程基础**：了解类常量和枚举（PHP 8.1+）

## 练习任务

1. **常量定义练习**：
   - 使用 `const` 定义顶层常量
   - 使用 `define()` 定义常量
   - 定义类常量
   - 理解 `const` 和 `define()` 的区别

2. **类常量练习**：
   - 定义类常量
   - 通过类名和对象访问类常量
   - 在类内部使用 `self` 和 `static` 访问常量
   - 实现枚举常量

3. **魔术常量练习**：
   - 使用 `__LINE__`、`__FILE__`、`__DIR__`
   - 使用 `__FUNCTION__`、`__CLASS__`、`__METHOD__`
   - 使用 `__NAMESPACE__`
   - 理解魔术常量的使用场景

4. **常量检查练习**：
   - 使用 `defined()` 检查常量是否存在
   - 使用 `get_defined_constants()` 获取所有常量
   - 区分用户定义常量和内置常量

5. **综合练习**：
   - 创建一个配置类，使用类常量存储配置
   - 实现一个状态枚举，使用类常量
   - 使用常量替代代码中的魔法值
   - 测试不同常量定义方式的性能差异
