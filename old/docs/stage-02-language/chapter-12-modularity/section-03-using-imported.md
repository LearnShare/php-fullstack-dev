# 2.12.3 使用导入的内容

## 概述

导入文件后，需要知道如何使用导入的函数、类、配置和常量。本节详细介绍如何使用导入的各种内容，以及相关的注意事项。

## 使用导入的函数

### 基本使用

导入的函数在全局作用域可用，可以直接调用：

```php
<?php
// utils/math.php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

function multiply(int $a, int $b): int
{
    return $a * $b;
}
```

```php
<?php
// main.php
declare(strict_types=1);

// 导入函数模块
require __DIR__ . '/utils/math.php';

// 直接使用函数（函数在全局作用域）
$result = add(10, 20);
echo $result . "\n";  // 30

// 可以调用多个函数
$sum = add(5, 3);
$product = multiply(4, 6);
echo "Sum: {$sum}, Product: {$product}\n";
```

### 检查函数是否存在

在使用函数前，可以检查函数是否已定义：

```php
<?php
declare(strict_types=1);

// 可选导入
include __DIR__ . '/utils/math.php';

// 检查函数是否存在
if (function_exists('add')) {
    $result = add(5, 3);
    echo $result . "\n";
} else {
    echo "函数 add 不存在\n";
}
```

### 函数名冲突处理

如果函数名可能冲突，使用命名空间：

```php
<?php
// utils/math.php
namespace Utils;

function add(int $a, int $b): int
{
    return $a + $b;
}
```

```php
<?php
// main.php
declare(strict_types=1);

require __DIR__ . '/utils/math.php';

// 使用完全限定名
$result = \Utils\add(5, 3);

// 或使用 use 导入
use function Utils\add;
$result = add(5, 3);
```

### 注意事项

- 函数名不能与已存在的函数冲突
- 函数在全局作用域可用（除非使用命名空间）
- 如果函数已定义，再次包含会报错（除非使用 `_once`）
- 函数定义在导入后立即生效

## 使用导入的类

### 基本使用

导入类后，可以创建对象并调用方法：

```php
<?php
// src/User.php
declare(strict_types=1);

class User
{
    public function __construct(
        private string $name,
        private int $age
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
}
```

```php
<?php
// main.php
declare(strict_types=1);

// 导入类文件
require __DIR__ . '/src/User.php';

// 创建对象
$user = new User('Bob', 30);

// 调用方法
echo $user->getName() . "\n";  // Bob
echo $user->getAge() . "\n";   // 30
```

### 检查类是否存在

在使用类前，可以检查类是否已定义：

```php
<?php
declare(strict_types=1);

require __DIR__ . '/src/User.php';

// 检查类是否存在
if (class_exists('User')) {
    $user = new User('Alice', 25);
    echo $user->getName() . "\n";
} else {
    echo "类 User 不存在\n";
}
```

### 类名冲突处理

如果类名可能冲突，使用命名空间：

```php
<?php
// database/Connection.php
namespace Database;

class Connection
{
    // ...
}
```

```php
<?php
// cache/Connection.php
namespace Cache;

class Connection
{
    // ...
}
```

```php
<?php
// main.php
declare(strict_types=1);

require __DIR__ . '/database/Connection.php';
require __DIR__ . '/cache/Connection.php';

// 使用别名避免冲突
use Database\Connection as DatabaseConnection;
use Cache\Connection as CacheConnection;

$db = new DatabaseConnection();
$cache = new CacheConnection();
```

### 静态方法和属性

```php
<?php
// utils/Formatter.php
declare(strict_types=1);

class Formatter
{
    public static function formatCurrency(float $amount): string
    {
        return '¥' . number_format($amount, 2);
    }
    
    public static function formatDate(string $date): string
    {
        return date('Y-m-d', strtotime($date));
    }
}
```

```php
<?php
// main.php
declare(strict_types=1);

require __DIR__ . '/utils/Formatter.php';

// 调用静态方法
echo Formatter::formatCurrency(1234.56) . "\n";  // ¥1,234.56
echo Formatter::formatDate('2024-01-01') . "\n";  // 2024-01-01
```

### 注意事项

- 类名不能与已存在的类冲突
- 类在全局命名空间（如果没有命名空间）
- 如果类已定义，再次包含会报错（除非使用 `_once`）
- 类定义在导入后立即生效

## 使用导入的配置

### 基本使用

配置文件通常返回数组：

```php
<?php
// config/app.php
declare(strict_types=1);

return [
    'app_name' => 'My Application',
    'version' => '1.0.0',
    'debug' => true,
    'database' => [
        'host' => 'localhost',
        'port' => 3306,
        'name' => 'mydb'
    ]
];
```

```php
<?php
// main.php
declare(strict_types=1);

// 导入配置文件（返回数组）
$config = require __DIR__ . '/config/app.php';

// 使用配置
echo $config['app_name'] . "\n";  // My Application
echo $config['version'] . "\n";   // 1.0.0

// 访问嵌套配置
echo $config['database']['host'] . "\n";  // localhost
echo $config['database']['port'] . "\n";  // 3306
```

### 配置使用模式

**模式 1：直接使用返回值**

```php
<?php
declare(strict_types=1);

$config = require __DIR__ . '/config/app.php';
echo $config['app_name'] . "\n";
```

**模式 2：合并多个配置**

```php
<?php
declare(strict_types=1);

// 基础配置
$baseConfig = require __DIR__ . '/config/base.php';

// 环境配置
$envConfig = require __DIR__ . '/config/env.php';

// 合并配置（环境配置覆盖基础配置）
$config = array_merge($baseConfig, $envConfig);
```

**模式 3：条件加载**

```php
<?php
declare(strict_types=1);

// 根据环境加载不同配置
$env = getenv('APP_ENV') ?: 'production';
$config = require __DIR__ . "/config/{$env}.php";
```

**模式 4：配置类封装**

```php
<?php
// config/Config.php
declare(strict_types=1);

class Config
{
    private static array $config = [];
    
    public static function load(string $file): void
    {
        self::$config = require $file;
    }
    
    public static function get(string $key, mixed $default = null): mixed
    {
        $keys = explode('.', $key);
        $value = self::$config;
        
        foreach ($keys as $k) {
            if (!isset($value[$k])) {
                return $default;
            }
            $value = $value[$k];
        }
        
        return $value;
    }
}
```

```php
<?php
// main.php
declare(strict_types=1);

require __DIR__ . '/config/Config.php';

Config::load(__DIR__ . '/config/app.php');

// 使用点号访问嵌套配置
echo Config::get('app_name') . "\n";
echo Config::get('database.host') . "\n";
```

### 配置验证

```php
<?php
// config/app.php
declare(strict_types=1);

$config = [
    'app_name' => 'My Application',
    'version' => '1.0.0',
    'database' => [
        'host' => 'localhost',
        'port' => 3306
    ]
];

// 验证必需配置
$required = ['app_name', 'version', 'database.host'];
foreach ($required as $key) {
    $keys = explode('.', $key);
    $value = $config;
    foreach ($keys as $k) {
        if (!isset($value[$k])) {
            throw new RuntimeException("缺少必需配置: {$key}");
        }
        $value = $value[$k];
    }
}

return $config;
```

### 注意事项

- 配置文件应该只返回数据，避免执行代码
- 使用 `return` 语句返回值
- 可以嵌套数组组织复杂配置
- 验证配置的完整性和有效性

## 使用导入的常量

### 基本使用

```php
<?php
// constants.php
declare(strict_types=1);

const MAX_USERS = 100;
const DEFAULT_LANGUAGE = 'zh-CN';
const STATUS_ACTIVE = 1;
const STATUS_INACTIVE = 0;
```

```php
<?php
// main.php
declare(strict_types=1);

// 导入常量
require __DIR__ . '/constants.php';

// 使用常量
echo MAX_USERS . "\n";  // 100
echo DEFAULT_LANGUAGE . "\n";  // zh-CN

// 在条件判断中使用
$status = STATUS_ACTIVE;
if ($status === STATUS_ACTIVE) {
    echo "用户已激活\n";
}
```

### 常量数组（PHP 5.6+）

```php
<?php
// constants.php
declare(strict_types=1);

const CONFIG = [
    'max_users' => 100,
    'default_language' => 'zh-CN'
];
```

```php
<?php
// main.php
declare(strict_types=1);

require __DIR__ . '/constants.php';

echo CONFIG['max_users'] . "\n";
echo CONFIG['default_language'] . "\n";
```

### 常量名冲突处理

如果常量名可能冲突，使用命名空间：

```php
<?php
// config1.php
namespace Config1;

const MAX_SIZE = 100;
```

```php
<?php
// config2.php
namespace Config2;

const MAX_SIZE = 200;
```

```php
<?php
// main.php
declare(strict_types=1);

require __DIR__ . '/config1.php';
require __DIR__ . '/config2.php';

// 使用完全限定名
echo \Config1\MAX_SIZE . "\n";  // 100
echo \Config2\MAX_SIZE . "\n";  // 200

// 或使用 use const
use const Config1\MAX_SIZE as CONFIG1_MAX_SIZE;
use const Config2\MAX_SIZE as CONFIG2_MAX_SIZE;

echo CONFIG1_MAX_SIZE . "\n";  // 100
echo CONFIG2_MAX_SIZE . "\n";  // 200
```

### 检查常量是否存在

```php
<?php
declare(strict_types=1);

require __DIR__ . '/constants.php';

// 检查常量是否存在
if (defined('MAX_USERS')) {
    echo MAX_USERS . "\n";
} else {
    echo "常量 MAX_USERS 不存在\n";
}
```

### 注意事项

- 常量名不能与已存在的常量冲突
- 常量在全局作用域可用（除非使用命名空间）
- 常量一旦定义不能修改
- 使用 `const` 定义的常量在编译时确定

## 使用导入的变量

### 基本使用

```php
<?php
// config.php
$appName = 'My App';
$version = '1.0.0';
```

```php
<?php
// main.php
declare(strict_types=1);

require __DIR__ . '/config.php';

// 直接使用导入的变量
echo $appName . "\n";  // My App
echo $version . "\n";  // 1.0.0
```

### 在函数中使用

```php
<?php
// config.php
$databaseHost = 'localhost';
```

```php
<?php
// main.php
declare(strict_types=1);

require __DIR__ . '/config.php';

function connectDatabase(): void
{
    // 需要使用 global 关键字
    global $databaseHost;
    echo "连接到: {$databaseHost}\n";
}

connectDatabase();
```

### 注意事项

- 变量在导入后立即可用（如果在全局作用域）
- 在函数内部需要使用 `global` 关键字
- 变量可以被修改，可能影响其他使用该变量的代码
- 建议使用配置数组而不是全局变量

## 综合示例

### 完整的模块使用示例

```php
<?php
// utils/math.php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}
```

```php
<?php
// src/User.php
declare(strict_types=1);

class User
{
    public function __construct(
        private string $name,
        private int $age
    ) {
    }
    
    public function getName(): string
    {
        return $this->name;
    }
}
```

```php
<?php
// config/app.php
declare(strict_types=1);

return [
    'app_name' => 'My Application',
    'version' => '1.0.0'
];
```

```php
<?php
// constants.php
declare(strict_types=1);

const MAX_USERS = 100;
```

```php
<?php
// main.php
declare(strict_types=1);

// 1. 导入配置
$config = require __DIR__ . '/config/app.php';

// 2. 导入常量
require __DIR__ . '/constants.php';

// 3. 导入函数
require __DIR__ . '/utils/math.php';

// 4. 导入类
require __DIR__ . '/src/User.php';

// 5. 使用导入的内容
echo "应用: {$config['app_name']}\n";
echo "版本: {$config['version']}\n";
echo "最大用户数: " . MAX_USERS . "\n";

$sum = add(5, 3);
echo "5 + 3 = {$sum}\n";

$user = new User('Alice', 25);
echo "用户: {$user->getName()}\n";
```

## 最佳实践

1. **使用配置数组**：而不是全局变量
2. **验证导入的内容**：检查函数、类、常量是否存在
3. **使用命名空间**：避免名称冲突
4. **封装配置访问**：使用配置类管理配置
5. **文档化**：为导入的内容添加文档注释

## 注意事项

1. **作用域**：函数、类、常量在全局作用域可用
2. **执行顺序**：导入后立即可用
3. **名称冲突**：使用命名空间避免冲突
4. **错误处理**：检查内容是否存在
5. **性能**：避免在循环中重复导入

## 练习

1. 创建一个配置模块，返回嵌套数组，在主文件中读取和使用配置。

2. 编写代码，演示如何使用导入的函数、类和常量。

3. 实现一个配置管理类，封装配置的加载和访问。

4. 创建两个同名的函数，使用命名空间避免冲突。

5. 编写代码，检查导入的函数、类和常量是否存在。
