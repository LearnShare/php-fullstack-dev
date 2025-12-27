# 4.4.6 环境变量与系统信息

## 概述

环境变量和系统信息是 CLI 程序中的重要数据源。环境变量用于配置应用程序的行为，系统信息用于了解运行环境。PHP 提供了多种方式获取和设置环境变量，以及获取系统信息。理解这些功能的使用方法，对于开发健壮的 CLI 应用至关重要。

在实际应用中，我们经常需要根据环境变量配置应用、检测操作系统类型、获取系统信息等。这些功能在 CLI 程序中尤其重要。

**主要内容**：
- 环境变量概述和作用
- `$_ENV` 超全局变量
- `getenv()` 函数（获取环境变量）
- `putenv()` 函数（设置环境变量）
- `php_uname()` 函数（获取系统信息）
- `PHP_OS_FAMILY` 常量（操作系统检测）
- PHP 配置信息获取
- 平台检测方法
- 实际应用场景和最佳实践

## 特性

- **环境变量访问**：支持获取和设置环境变量
- **系统信息**：支持获取操作系统、主机名等信息
- **平台检测**：支持检测操作系统类型
- **配置信息**：支持获取 PHP 配置信息
- **跨平台**：支持 Windows 和类 Unix 系统

## 语法/定义

### getenv() 函数

**语法**：`getenv(string $name, bool $local_only = false): string|false`

**参数**：
- `$name`：环境变量名称
- `$local_only`：可选，是否只从本地环境获取（不包括 `$_SERVER`）

**返回值**：成功返回环境变量的值，失败返回 `false`。

### putenv() 函数

**语法**：`putenv(string $assignment): bool`

**参数**：
- `$assignment`：环境变量设置字符串，格式为 `NAME=VALUE`

**返回值**：成功返回 `true`，失败返回 `false`。

### php_uname() 函数

**语法**：`php_uname(?string $mode = null): string`

**参数**：
- `$mode`：可选，返回信息的模式（`'a'` 全部，`'s'` 操作系统名称，`'n'` 主机名，`'r'` 版本，`'v'` 版本信息，`'m'` 机器类型）

**返回值**：返回系统信息字符串。

## 基本用法

### 示例 1：获取环境变量

```php
<?php
declare(strict_types=1);

// 方法1：使用 getenv()
$path = getenv('PATH');
if ($path !== false) {
    echo "PATH: {$path}\n";
} else {
    echo "PATH 环境变量不存在\n";
}

// 方法2：使用 $_ENV 超全局变量
$home = $_ENV['HOME'] ?? null;
if ($home !== null) {
    echo "HOME: {$home}\n";
}

// 方法3：使用 $_SERVER（在 Web 环境可用）
$serverName = $_SERVER['SERVER_NAME'] ?? null;
if ($serverName !== null) {
    echo "SERVER_NAME: {$serverName}\n";
}
```

**说明**：
- `getenv()` 是获取环境变量的标准方法
- `$_ENV` 需要 `variables_order` 配置包含 `E`
- `$_SERVER` 在 Web 环境包含环境变量

### 示例 2：设置环境变量

```php
<?php
declare(strict_types=1);

// 设置环境变量
putenv('MY_APP_ENV=production');
putenv('MY_APP_DEBUG=false');

// 获取设置的环境变量
$env = getenv('MY_APP_ENV');
$debug = getenv('MY_APP_DEBUG');

echo "环境: {$env}\n";
echo "调试: {$debug}\n";

// 注意：putenv() 设置的变量只在当前进程中有效
```

**说明**：
- `putenv()` 设置的环境变量只在当前进程中有效
- 格式为 `NAME=VALUE`
- 设置后可以使用 `getenv()` 获取

### 示例 3：获取系统信息

```php
<?php
declare(strict_types=1);

// 获取全部系统信息
$uname = php_uname();
echo "系统信息: {$uname}\n";

// 获取特定信息
$os = php_uname('s');  // 操作系统名称
$hostname = php_uname('n');  // 主机名
$release = php_uname('r');  // 版本
$machine = php_uname('m');  // 机器类型

echo "操作系统: {$os}\n";
echo "主机名: {$hostname}\n";
echo "版本: {$release}\n";
echo "机器类型: {$machine}\n";
```

**说明**：
- `php_uname()` 获取系统信息
- 可以使用参数获取特定信息

### 示例 4：检测操作系统

```php
<?php
declare(strict_types=1);

// 方法1：使用 PHP_OS_FAMILY 常量（PHP 7.2+）
$osFamily = PHP_OS_FAMILY;

match ($osFamily) {
    'Windows' => echo "Windows 系统\n",
    'Linux' => echo "Linux 系统\n",
    'Darwin' => echo "macOS 系统\n",
    default => echo "其他系统: {$osFamily}\n",
};

// 方法2：使用 php_uname()
$os = php_uname('s');
if (str_starts_with($os, 'Windows')) {
    echo "Windows 系统\n";
} elseif ($os === 'Linux') {
    echo "Linux 系统\n";
} elseif ($os === 'Darwin') {
    echo "macOS 系统\n";
}
```

**说明**：
- `PHP_OS_FAMILY` 常量提供操作系统家族信息
- `php_uname('s')` 返回详细的操作系统名称

### 示例 5：获取 PHP 配置信息

```php
<?php
declare(strict_types=1);

// 获取 PHP 版本
$version = PHP_VERSION;
echo "PHP 版本: {$version}\n";

// 获取 PHP 版本 ID
$versionId = PHP_VERSION_ID;
echo "PHP 版本 ID: {$versionId}\n";

// 获取 SAPI 类型
$sapi = php_sapi_name();
echo "SAPI: {$sapi}\n";

// 获取配置值
$memoryLimit = ini_get('memory_limit');
echo "内存限制: {$memoryLimit}\n";

// 获取所有配置
$allConfig = ini_get_all();
// print_r($allConfig);
```

**说明**：
- `PHP_VERSION` 常量包含 PHP 版本字符串
- `PHP_VERSION_ID` 包含版本数字 ID
- `php_sapi_name()` 获取 SAPI 类型
- `ini_get()` 获取配置值

### 示例 6：环境变量配置管理

```php
<?php
declare(strict_types=1);

class Config
{
    public static function get(string $key, ?string $default = null): ?string
    {
        // 1. 检查环境变量
        $value = getenv($key);
        if ($value !== false) {
            return $value;
        }
        
        // 2. 检查 $_ENV
        if (isset($_ENV[$key])) {
            return $_ENV[$key];
        }
        
        // 3. 返回默认值
        return $default;
    }
    
    public static function getBool(string $key, bool $default = false): bool
    {
        $value = self::get($key);
        if ($value === null) {
            return $default;
        }
        
        return in_array(strtolower($value), ['1', 'true', 'on', 'yes'], true);
    }
    
    public static function getInt(string $key, int $default = 0): int
    {
        $value = self::get($key);
        if ($value === null) {
            return $default;
        }
        
        return (int)$value;
    }
}

// 使用
$dbHost = Config::get('DB_HOST', 'localhost');
$dbPort = Config::getInt('DB_PORT', 3306);
$debug = Config::getBool('DEBUG', false);

echo "数据库主机: {$dbHost}\n";
echo "数据库端口: {$dbPort}\n";
echo "调试模式: " . ($debug ? '是' : '否') . "\n";
```

**说明**：
- 封装环境变量获取逻辑
- 支持默认值和类型转换

### 示例 7：跨平台路径处理

```php
<?php
declare(strict_types=1);

function getHomeDirectory(): string
{
    // Unix/Linux/macOS
    if (PHP_OS_FAMILY !== 'Windows') {
        $home = getenv('HOME');
        if ($home !== false) {
            return $home;
        }
    } else {
        // Windows
        $home = getenv('USERPROFILE');
        if ($home !== false) {
            return $home;
        }
        
        $home = getenv('HOMEDRIVE') . getenv('HOMEPATH');
        if ($home !== false) {
            return $home;
        }
    }
    
    return '/';
}

$homeDir = getHomeDirectory();
echo "用户主目录: {$homeDir}\n";
```

**说明**：
- 根据操作系统获取用户主目录
- Windows 和 Unix 系统使用不同的环境变量

## 使用场景

### 场景 1：配置管理

使用环境变量配置应用程序。

**示例**：

```php
<?php
declare(strict_types=1);

// 从环境变量读取配置
$dbHost = getenv('DB_HOST') ?: 'localhost';
$dbPort = (int)(getenv('DB_PORT') ?: '3306');
$dbName = getenv('DB_NAME') ?: 'myapp';

// 使用配置
echo "数据库配置:\n";
echo "  主机: {$dbHost}\n";
echo "  端口: {$dbPort}\n";
echo "  数据库: {$dbName}\n";
```

### 场景 2：环境检测

检测运行环境并执行相应逻辑。

**示例**：

```php
<?php
declare(strict_types=1);

$environment = getenv('APP_ENV') ?: 'production';

match ($environment) {
    'development' => {
        error_reporting(E_ALL);
        ini_set('display_errors', '1');
    },
    'production' => {
        error_reporting(0);
        ini_set('display_errors', '0');
    },
    default => {
        // 默认配置
    },
};
```

## 注意事项

### 环境变量的可用性

环境变量可能不存在，应该提供默认值。

**示例**：

```php
<?php
declare(strict_types=1);

// ✅ 正确：提供默认值
$timeout = (int)(getenv('TIMEOUT') ?: '30');

// ❌ 错误：假设环境变量存在
$timeout = (int)getenv('TIMEOUT');  // 如果不存在，返回 false
```

### 跨平台差异

不同平台的环境变量名称可能不同。

**示例**：见"示例 7：跨平台路径处理"

### 安全性考虑

环境变量可能包含敏感信息，注意不要泄露。

**示例**：

```php
<?php
declare(strict_types=1);

// 不要在生产环境输出敏感信息
$password = getenv('DB_PASSWORD');
// echo $password;  // 危险！
```

## 常见问题

### 问题 1：如何获取环境变量？

**回答**：使用 `getenv()` 函数或 `$_ENV` 超全局变量。

**示例**：

```php
<?php
declare(strict_types=1);

$value = getenv('VAR_NAME');
// 或
$value = $_ENV['VAR_NAME'] ?? null;
```

### 问题 2：环境变量和配置文件的区别？

**回答**：
- 环境变量：适合敏感信息、环境特定配置，不在代码中硬编码
- 配置文件：适合复杂配置、需要版本控制的配置

### 问题 3：如何检测操作系统？

**回答**：使用 `PHP_OS_FAMILY` 常量或 `php_uname('s')` 函数。

**示例**：见"示例 4：检测操作系统"

### 问题 4：如何获取 PHP 配置信息？

**回答**：使用 `ini_get()` 函数获取配置值，使用 `phpinfo()` 查看所有配置。

**示例**：

```php
<?php
declare(strict_types=1);

$memoryLimit = ini_get('memory_limit');
$maxExecutionTime = ini_get('max_execution_time');
```

## 最佳实践

### 1. 使用 getenv() 获取环境变量

`getenv()` 是获取环境变量的标准方法。

**示例**：

```php
<?php
declare(strict_types=1);

$value = getenv('VAR_NAME') ?: 'default_value';
```

### 2. 提供合理的默认值

对于可能不存在的环境变量，提供默认值。

**示例**：见"注意事项"部分

### 3. 检测平台差异

根据不同平台使用不同的环境变量或逻辑。

**示例**：见"示例 7：跨平台路径处理"

### 4. 注意环境变量的安全性

不要在生产环境输出或记录敏感的环境变量。

**示例**：见"安全性考虑"部分

### 5. 使用配置管理类

封装环境变量获取逻辑，提供统一的接口。

**示例**：见"示例 6：环境变量配置管理"

## 对比分析

### getenv() vs $_ENV vs $_SERVER

| 特性         | getenv()                    | $_ENV                       | $_SERVER                    |
|:-------------|:----------------------------|:----------------------------|:----------------------------|
| **可用性**   | ✅ 总是可用                 | ⚠️ 需要配置                 | ⚠️ 主要 Web 环境            |
| **性能**     | ✅ 直接系统调用             | ⚠️ 需要解析                 | ⚠️ 需要解析                 |
| **推荐使用** | ✅ CLI 环境推荐             | ⚠️ 需要配置                 | Web 环境                    |

### PHP_OS_FAMILY vs php_uname()

| 特性         | PHP_OS_FAMILY               | php_uname()                 |
|:-------------|:----------------------------|:----------------------------|
| **信息类型** | 操作系统家族（Windows/Linux/Darwin）| 详细信息（完整系统信息）   |
| **可用性**   | PHP 7.2+                    | 所有版本                    |
| **使用场景** | 简单的平台检测              | 需要详细信息                |

## 练习任务

1. **环境变量管理工具类**：创建一个工具类，封装环境变量的获取和设置功能。

2. **系统信息工具**：实现一个工具，收集和显示系统信息。

3. **跨平台配置管理**：创建一个配置管理类，处理不同平台的环境变量差异。

4. **环境检测工具**：编写一个工具，检测运行环境并输出环境信息。

5. **配置验证工具**：实现一个工具，验证必需的环境变量是否存在。

## 相关章节

- **[4.4.1 CLI 基础与参数处理](section-01-cli-basics.md)**：了解 CLI 编程基础
- **[4.4.7 CLI 最佳实践](section-07-best-practices.md)**：了解 CLI 最佳实践
- **[4.2.12 路径处理](../chapter-02-filesystem/section-12-path-handling.md)**：了解路径处理的相关内容
