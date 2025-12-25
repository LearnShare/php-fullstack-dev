# 2.3.3 常量

## 概述

常量是 PHP 中用于存储不变值的标识符。与变量不同，常量一旦定义就不能被修改或取消定义。PHP 提供了两种定义常量的方式：`const` 关键字和 `define()` 函数。此外，PHP 还提供了一系列**魔术常量**（Magic Constants），它们在运行时自动解析为当前上下文的相关信息。

## const 关键字

### 基本语法

```php
const CONSTANT_NAME = value;
```

### 特点

1. **编译期常量**：`const` 定义的常量在编译时解析，性能更好。
2. **作用域限制**：只能在顶级作用域（文件级别）或类内部使用。
3. **语法限制**：只能使用标识符作为常量名（不能使用表达式或变量）。
4. **类型支持**：可以定义标量类型（string、int、float、bool）和数组常量（PHP 5.6+）。

### 基本示例

```php
<?php
declare(strict_types=1);

// 文件级别常量
const SITE_NAME = 'My Website';
const MAX_USERS = 1000;
const PI = 3.14159;
const IS_PRODUCTION = false;

echo SITE_NAME . "\n";      // 输出：My Website
echo MAX_USERS . "\n";      // 输出：1000
echo PI . "\n";             // 输出：3.14159
```

### 类常量

```php
<?php
declare(strict_types=1);

class Config
{
    const DB_HOST = 'localhost';
    const DB_NAME = 'myapp';
    const MAX_CONNECTIONS = 100;
    
    public function getDbHost(): string
    {
        return self::DB_HOST;  // 使用 self:: 访问类常量
    }
}

// 访问类常量
echo Config::DB_HOST . "\n";           // 输出：localhost
echo Config::MAX_CONNECTIONS . "\n";   // 输出：100

$config = new Config();
echo $config->getDbHost() . "\n";      // 输出：localhost
```

### 数组常量（PHP 5.6+）

```php
<?php
declare(strict_types=1);

const COLORS = ['red', 'green', 'blue'];
const CONFIG = [
    'host' => 'localhost',
    'port' => 3306
];

echo COLORS[0] . "\n";           // 输出：red
echo CONFIG['host'] . "\n";      // 输出：localhost
```

### const 的限制

1. **不能在条件语句中定义**：
```php
<?php
// 错误：不能在 if 语句中使用 const
if (true) {
    // const MY_CONST = 'value';  // 语法错误
}
```

2. **不能使用表达式**：
```php
<?php
// 错误：不能使用表达式
// const PREFIX = 'APP_' . 'VERSION';  // 语法错误
```

## define() 函数

### 基本语法

```php
define(string $name, mixed $value, bool $case_insensitive = false): bool
```

### 参数说明

- `$name`：常量名称（字符串），遵循与变量相同的命名规则，但不需要 `$` 前缀。
- `$value`：常量的值，可以是标量类型或数组。
- `$case_insensitive`：可选，如果设置为 `true`，常量名将不区分大小写（不推荐使用）。

### 返回值

成功定义常量返回 `true`，如果常量已存在则返回 `false`。

### 特点

1. **运行期常量**：`define()` 定义的常量在运行时解析。
2. **全局作用域**：定义的常量在全局命名空间中可用。
3. **灵活性**：可以在条件语句中定义，可以使用表达式。
4. **动态定义**：可以使用变量或表达式构建常量名。

### 基本示例

```php
<?php
declare(strict_types=1);

define('SITE_NAME', 'My Website');
define('MAX_USERS', 1000);
define('PI', 3.14159);
define('IS_PRODUCTION', false);

echo SITE_NAME . "\n";      // 输出：My Website
echo MAX_USERS . "\n";      // 输出：1000
```

### 条件定义

```php
<?php
declare(strict_types=1);

if (php_sapi_name() === 'cli') {
    define('ENVIRONMENT', 'development');
} else {
    define('ENVIRONMENT', 'production');
}

echo ENVIRONMENT . "\n";
```

### 使用表达式定义

```php
<?php
declare(strict_types=1);

$prefix = 'APP_';
define($prefix . 'VERSION', '1.0.0');
define($prefix . 'NAME', 'My Application');

echo APP_VERSION . "\n";    // 输出：1.0.0
echo APP_NAME . "\n";       // 输出：My Application
```

### 数组常量

```php
<?php
declare(strict_types=1);

define('COLORS', ['red', 'green', 'blue']);
define('CONFIG', [
    'host' => 'localhost',
    'port' => 3306
]);

echo COLORS[0] . "\n";           // 输出：red
echo CONFIG['host'] . "\n";      // 输出：localhost
```

### 检查常量是否已定义

使用 `defined()` 函数检查常量是否已定义：

```php
<?php
declare(strict_types=1);

if (!defined('APP_VERSION')) {
    define('APP_VERSION', '1.0.0');
}

echo APP_VERSION . "\n";
```

## const 与 define 的对比

| 特性           | `const`                          | `define()`                       |
| :------------- | :------------------------------- | :------------------------------- |
| 定义时机       | 编译期                           | 运行期                           |
| 作用域         | 文件级别或类内部                 | 全局命名空间                     |
| 语法支持       | 仅标识符                         | 可以使用表达式或变量             |
| 条件定义       | 不支持                           | 支持                             |
| 性能           | 稍快（编译期解析）               | 稍慢（运行期解析）               |
| 数组常量       | PHP 5.6+ 支持                    | PHP 7.0+ 支持                    |
| 类常量         | 支持                             | 不支持（在类外部定义）           |
| 命名空间       | 遵循命名空间规则                 | 始终在全局命名空间               |

### 选择建议

- **类常量**：必须使用 `const`。
- **文件级别常量**：如果不需要条件定义，优先使用 `const`（性能更好，语法更简洁）。
- **条件定义或动态定义**：使用 `define()`。
- **命名空间常量**：使用 `const`（遵循命名空间规则）。

## 常量命名规范

### 推荐规范

1. **全大写字母**：常量名应全部使用大写字母。
2. **下划线分隔**：多个单词使用下划线（`_`）分隔。
3. **描述性名称**：使用清晰、描述性的名称。

### 示例

```php
<?php
declare(strict_types=1);

// 推荐
const MAX_FILE_SIZE = 10485760;      // 10MB
const DEFAULT_TIMEOUT = 30;
const API_BASE_URL = 'https://api.example.com';

// 不推荐
const maxFileSize = 10485760;        // 应使用大写
const defaultTimeout = 30;           // 应使用大写
const apiBaseUrl = 'https://api.example.com';  // 应使用大写
```

### 命名空间常量

在命名空间中，`const` 定义的常量遵循命名空间规则：

```php
<?php
declare(strict_types=1);

namespace App\Config;

const DB_HOST = 'localhost';
const DB_NAME = 'myapp';

// 访问方式
echo \App\Config\DB_HOST . "\n";     // 完整路径
echo DB_HOST . "\n";                  // 当前命名空间内
```

## 魔术常量（Magic Constants）

魔术常量是 PHP 预定义的常量，它们在运行时根据使用位置自动解析为相应的值。所有魔术常量都以双下划线开头和结尾。

### 文件相关常量

#### `__FILE__`

当前文件的完整路径和文件名。

```php
<?php
declare(strict_types=1);

echo __FILE__ . "\n";
// 输出：/path/to/your/script.php
```

#### `__DIR__`

当前文件所在的目录（等同于 `dirname(__FILE__)`）。

```php
<?php
declare(strict_types=1);

echo __DIR__ . "\n";
// 输出：/path/to/your
```

#### `__LINE__`

当前行号。

```php
<?php
declare(strict_types=1);

echo "Current line: " . __LINE__ . "\n";  // 输出：Current line: 5
echo "Current line: " . __LINE__ . "\n";  // 输出：Current line: 6
```

### 函数和方法相关常量

#### `__FUNCTION__`

当前函数名（在函数内部使用）。

```php
<?php
declare(strict_types=1);

function myFunction(): void
{
    echo "Function: " . __FUNCTION__ . "\n";
}

myFunction();  // 输出：Function: myFunction
```

#### `__METHOD__`

当前方法名（在类方法内部使用），包含类名。

```php
<?php
declare(strict_types=1);

class MyClass
{
    public function myMethod(): void
    {
        echo "Method: " . __METHOD__ . "\n";
    }
}

$obj = new MyClass();
$obj->myMethod();  // 输出：Method: MyClass::myMethod
```

### 类相关常量

#### `__CLASS__`

当前类名（在类内部使用）。

```php
<?php
declare(strict_types=1);

class MyClass
{
    public function showClass(): void
    {
        echo "Class: " . __CLASS__ . "\n";
    }
}

$obj = new MyClass();
$obj->showClass();  // 输出：Class: MyClass
```

#### `__TRAIT__`

当前 trait 名（在 trait 内部使用）。

```php
<?php
declare(strict_types=1);

trait MyTrait
{
    public function showTrait(): void
    {
        echo "Trait: " . __TRAIT__ . "\n";
    }
}

class MyClass
{
    use MyTrait;
}

$obj = new MyClass();
$obj->showTrait();  // 输出：Trait: MyTrait
```

#### `__NAMESPACE__`

当前命名空间名。

```php
<?php
declare(strict_types=1);

namespace App\Controllers;

echo "Namespace: " . __NAMESPACE__ . "\n";
// 输出：Namespace: App\Controllers
```

### 完整示例：使用魔术常量记录日志

```php
<?php
declare(strict_types=1);

function logMessage(string $message): void
{
    $logEntry = sprintf(
        "[%s] %s (File: %s, Line: %d, Function: %s)\n",
        date('Y-m-d H:i:s'),
        $message,
        __FILE__,
        __LINE__,
        __FUNCTION__
    );
    
    file_put_contents('app.log', $logEntry, FILE_APPEND);
    echo $logEntry;
}

function processOrder(int $orderId): void
{
    logMessage("Processing order #{$orderId}");
    // 处理订单逻辑...
}

processOrder(12345);
```

**输出示例**：
```
[2024-01-15 10:30:45] Processing order #12345 (File: /path/to/script.php, Line: 18, Function: processOrder)
```

## 获取所有已定义的常量

### get_defined_constants()

`get_defined_constants()` 函数返回所有已定义的常量数组。

```php
<?php
declare(strict_types=1);

const MY_CONST = 'value';
define('ANOTHER_CONST', 42);

$constants = get_defined_constants(true);  // true 表示按扩展分组

// 查看用户定义的常量
if (isset($constants['user'])) {
    print_r($constants['user']);
}
```

## 完整示例

### 示例 1：应用配置常量

```php
<?php
declare(strict_types=1);

// 使用 const 定义配置常量
const APP_NAME = 'My Application';
const APP_VERSION = '1.0.0';
const APP_ENV = 'development';

// 使用 define 定义条件常量
if (APP_ENV === 'production') {
    define('DEBUG_MODE', false);
    define('LOG_LEVEL', 'error');
} else {
    define('DEBUG_MODE', true);
    define('LOG_LEVEL', 'debug');
}

class Config
{
    const DB_HOST = 'localhost';
    const DB_PORT = 3306;
    const DB_NAME = 'myapp';
    
    public static function getDbConfig(): array
    {
        return [
            'host' => self::DB_HOST,
            'port' => self::DB_PORT,
            'name' => self::DB_NAME
        ];
    }
}

echo "App: " . APP_NAME . " v" . APP_VERSION . "\n";
echo "Environment: " . APP_ENV . "\n";
echo "Debug Mode: " . (DEBUG_MODE ? 'ON' : 'OFF') . "\n";
print_r(Config::getDbConfig());
```

### 示例 2：使用魔术常量进行调试

```php
<?php
declare(strict_types=1);

class Logger
{
    public static function log(string $level, string $message): void
    {
        $backtrace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 1);
        $caller = $backtrace[0] ?? [];
        
        $logEntry = sprintf(
            "[%s] [%s] %s | File: %s, Line: %d\n",
            date('Y-m-d H:i:s'),
            strtoupper($level),
            $message,
            $caller['file'] ?? __FILE__,
            $caller['line'] ?? __LINE__
        );
        
        echo $logEntry;
    }
}

function processData(array $data): void
{
    Logger::log('info', 'Processing data: ' . count($data) . ' items');
    // 处理数据...
}

processData([1, 2, 3, 4, 5]);
```

### 示例 3：命名空间常量

```php
<?php
declare(strict_types=1);

namespace App\Config;

const API_VERSION = 'v1';
const API_BASE_URL = 'https://api.example.com';

class ApiClient
{
    public function getBaseUrl(): string
    {
        return \App\Config\API_BASE_URL . '/' . \App\Config\API_VERSION;
    }
    
    public function getCurrentFile(): string
    {
        return __FILE__;
    }
    
    public function getCurrentNamespace(): string
    {
        return __NAMESPACE__;
    }
}

$client = new ApiClient();
echo "Base URL: " . $client->getBaseUrl() . "\n";
echo "File: " . $client->getCurrentFile() . "\n";
echo "Namespace: " . $client->getCurrentNamespace() . "\n";
```

## 注意事项

1. **常量不能重新定义**：一旦定义，常量就不能被修改或重新定义。

2. **常量名区分大小写**：虽然 `define()` 支持 `$case_insensitive` 参数，但不推荐使用。

3. **常量作用域**：`const` 定义的常量遵循命名空间规则，`define()` 定义的常量始终在全局命名空间。

4. **性能考虑**：`const` 在编译期解析，性能略好于 `define()`，但在大多数应用中差异可以忽略。

5. **魔术常量的解析**：魔术常量在运行时根据使用位置解析，在不同位置使用会得到不同的值。

6. **类常量访问**：类常量使用 `ClassName::CONSTANT` 或 `self::CONSTANT`（在类内部）访问。

## 练习

1. 创建一个配置类，使用 `const` 定义数据库连接相关的常量（主机、端口、数据库名、用户名、密码）。

2. 编写一个函数，使用 `define()` 根据环境变量动态定义应用配置常量（开发环境、生产环境）。

3. 创建一个日志记录函数，使用魔术常量（`__FILE__`、`__LINE__`、`__FUNCTION__`）记录函数调用的位置信息。

4. 实现一个简单的配置管理器，比较使用 `const` 和 `define()` 两种方式的优缺点。

5. 编写一个脚本，使用 `get_defined_constants()` 列出所有用户定义的常量，并格式化输出。
