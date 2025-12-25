# 2.12.5 路径处理和返回值

## 概述

在模块化开发中，正确处理文件路径和返回值是至关重要的。本节详细介绍路径处理的最佳实践以及如何使用导入文件的返回值。

## 路径处理最佳实践

### 使用 __DIR__

**推荐**：始终使用 `__DIR__` 构建相对路径

```php
<?php
declare(strict_types=1);

// 推荐：使用 __DIR__
require __DIR__ . '/config/app.php';
require __DIR__ . '/src/User.php';

// 不推荐：依赖当前工作目录
// require 'config/app.php';  // 可能找不到文件
```

**为什么使用 `__DIR__`**：
- `__DIR__` 是当前文件所在的目录
- 不依赖脚本的执行位置
- 更可靠和可预测
- 适合在不同环境中运行

### __DIR__ vs getcwd()

```php
<?php
// 当前文件：/var/www/app/src/main.php
// 执行命令：cd /tmp && php /var/www/app/src/main.php

// __DIR__：始终是文件所在目录
echo __DIR__;  // /var/www/app/src

// getcwd()：当前工作目录
echo getcwd();  // /tmp（取决于执行位置）
```

### 相对路径示例

**当前目录下的文件**：

```php
<?php
declare(strict_types=1);

// 当前目录
require __DIR__ . '/file.php';

// 子目录
require __DIR__ . '/subdir/file.php';

// 多个子目录
require __DIR__ . '/src/Models/User.php';
```

**上一级目录**：

```php
<?php
declare(strict_types=1);

// 上一级目录
require dirname(__DIR__) . '/file.php';

// 上两级目录
require dirname(__DIR__, 2) . '/file.php';

// 上 N 级目录
require dirname(__DIR__, 3) . '/file.php';
```

**组合路径**：

```php
<?php
declare(strict_types=1);

// 使用 dirname() 和 __DIR__ 组合
$baseDir = dirname(__DIR__);
require $baseDir . '/config/app.php';
require $baseDir . '/src/User.php';
```

### 绝对路径

**使用绝对路径（不推荐，降低可移植性）**：

```php
<?php
declare(strict_types=1);

// 不推荐：硬编码绝对路径
require '/var/www/app/config.php';
```

**动态获取项目根目录**：

```php
<?php
// bootstrap.php
define('ROOT_DIR', dirname(__DIR__));

// 其他文件
require ROOT_DIR . '/config/app.php';
```

### 路径规范化

**使用 realpath() 规范化路径**：

```php
<?php
declare(strict_types=1);

$path = __DIR__ . '/../config/app.php';
$realPath = realpath($path);

if ($realPath === false) {
    throw new RuntimeException("文件不存在: {$path}");
}

require $realPath;
```

**处理符号链接**：

```php
<?php
declare(strict_types=1);

$path = __DIR__ . '/config.php';
$realPath = realpath($path);

// realpath() 会解析符号链接
echo "原始路径: {$path}\n";
echo "实际路径: {$realPath}\n";
```

### 跨平台路径处理

**使用 DIRECTORY_SEPARATOR**：

```php
<?php
declare(strict_types=1);

$ds = DIRECTORY_SEPARATOR;  // Windows: \, Unix: /
$path = __DIR__ . $ds . 'config' . $ds . 'app.php';
require $path;
```

**或使用正斜杠（PHP 会自动处理）**：

```php
<?php
declare(strict_types=1);

// PHP 在 Windows 和 Unix 上都支持正斜杠
require __DIR__ . '/config/app.php';
```

### 路径验证

**验证路径安全性**：

```php
<?php
declare(strict_types=1);

function safeRequire(string $file): void
{
    // 1. 规范化路径
    $realPath = realpath($file);
    if ($realPath === false) {
        throw new RuntimeException("文件不存在: {$file}");
    }
    
    // 2. 检查是否在允许的目录内
    $allowedDir = __DIR__ . '/modules';
    $allowedRealPath = realpath($allowedDir);
    
    if ($allowedRealPath === false) {
        throw new RuntimeException("允许的目录不存在: {$allowedDir}");
    }
    
    if (strpos($realPath, $allowedRealPath) !== 0) {
        throw new SecurityException("不允许访问目录外的文件");
    }
    
    // 3. 检查文件扩展名
    if (pathinfo($realPath, PATHINFO_EXTENSION) !== 'php') {
        throw new SecurityException("只允许包含 PHP 文件");
    }
    
    require $realPath;
}
```

## 返回值的使用

### 配置文件返回数组

**基本用法**：

```php
<?php
// config/app.php
declare(strict_types=1);

return [
    'app_name' => 'My Application',
    'version' => '1.0.0',
    'debug' => true
];
```

```php
<?php
// main.php
declare(strict_types=1);

$config = require __DIR__ . '/config/app.php';
echo $config['app_name'] . "\n";
```

### return 语句的行为

**重要**：如果导入文件有 `return` 语句，`return` 之后的代码不会执行：

```php
<?php
// file.php
echo "这行会执行\n";
return '返回值';
echo "这行不会执行\n";  // 不会执行
```

**使用 return 提前退出**：

```php
<?php
// config.php
declare(strict_types=1);

// 检查环境
if (!file_exists(__DIR__ . '/.env')) {
    return [];  // 提前返回，后续代码不执行
}

// 加载环境变量
$env = parse_ini_file(__DIR__ . '/.env');
return $env;
```

### 条件返回

**根据条件返回不同配置**：

```php
<?php
// config.php
declare(strict_types=1);

if (php_sapi_name() === 'cli') {
    return [
        'env' => 'development',
        'debug' => true,
        'log_level' => 'debug'
    ];
} else {
    return [
        'env' => 'production',
        'debug' => false,
        'log_level' => 'error'
    ];
}
```

**根据环境变量返回**：

```php
<?php
// config.php
declare(strict_types=1);

$env = getenv('APP_ENV') ?: 'production';

return match($env) {
    'development' => [
        'debug' => true,
        'cache' => false
    ],
    'staging' => [
        'debug' => true,
        'cache' => true
    ],
    'production' => [
        'debug' => false,
        'cache' => true
    ],
    default => throw new InvalidArgumentException("未知环境: {$env}")
};
```

### 返回函数

**返回闭包或可调用对象**：

```php
<?php
// factory.php
declare(strict_types=1);

return function(string $type): object {
    return match($type) {
        'user' => new User(),
        'product' => new Product(),
        'order' => new Order(),
        default => throw new InvalidArgumentException("Unknown type: {$type}")
    };
};
```

```php
<?php
// main.php
declare(strict_types=1);

$factory = require __DIR__ . '/factory.php';
$user = $factory('user');
```

**返回配置对象**：

```php
<?php
// config.php
declare(strict_types=1);

class Config
{
    private array $data;
    
    public function __construct(array $data)
    {
        $this->data = $data;
    }
    
    public function get(string $key, mixed $default = null): mixed
    {
        return $this->data[$key] ?? $default;
    }
}

return new Config([
    'app_name' => 'My Application',
    'version' => '1.0.0'
]);
```

```php
<?php
// main.php
declare(strict_types=1);

$config = require __DIR__ . '/config.php';
echo $config->get('app_name') . "\n";
```

### 返回多个值

**返回数组包含多个值**：

```php
<?php
// init.php
declare(strict_types=1);

$db = new Database();
$cache = new Cache();
$logger = new Logger();

return [
    'database' => $db,
    'cache' => $cache,
    'logger' => $logger
];
```

```php
<?php
// main.php
declare(strict_types=1);

$services = require __DIR__ . '/init.php';
$db = $services['database'];
$cache = $services['cache'];
```

### 链式配置加载

**合并多个配置文件**：

```php
<?php
// config/base.php
declare(strict_types=1);

return [
    'app_name' => 'My Application',
    'version' => '1.0.0'
];
```

```php
<?php
// config/database.php
declare(strict_types=1);

return [
    'database' => [
        'host' => 'localhost',
        'port' => 3306
    ]
];
```

```php
<?php
// main.php
declare(strict_types=1);

$baseConfig = require __DIR__ . '/config/base.php';
$dbConfig = require __DIR__ . '/config/database.php';

$config = array_merge($baseConfig, $dbConfig);
```

**递归合并嵌套数组**：

```php
<?php
declare(strict_types=1);

function mergeConfig(array $base, array $override): array
{
    foreach ($override as $key => $value) {
        if (isset($base[$key]) && is_array($base[$key]) && is_array($value)) {
            $base[$key] = mergeConfig($base[$key], $value);
        } else {
            $base[$key] = $value;
        }
    }
    return $base;
}

$base = require __DIR__ . '/config/base.php';
$override = require __DIR__ . '/config/override.php';
$config = mergeConfig($base, $override);
```

### 配置验证

**验证返回的配置**：

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

// 验证必需字段
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

## 完整示例

### 路径处理和返回值综合示例

```php
<?php
// config/loader.php
declare(strict_types=1);

class ConfigLoader
{
    private string $configDir;
    
    public function __construct(string $configDir)
    {
        $this->configDir = realpath($configDir);
        if ($this->configDir === false) {
            throw new RuntimeException("配置目录不存在: {$configDir}");
        }
    }
    
    public function load(string $file): array
    {
        $path = $this->configDir . DIRECTORY_SEPARATOR . $file;
        $realPath = realpath($path);
        
        if ($realPath === false) {
            throw new RuntimeException("配置文件不存在: {$file}");
        }
        
        // 确保文件在配置目录内
        if (strpos($realPath, $this->configDir) !== 0) {
            throw new SecurityException("不允许访问配置目录外的文件");
        }
        
        $config = require $realPath;
        
        if (!is_array($config)) {
            throw new RuntimeException("配置文件必须返回数组: {$file}");
        }
        
        return $config;
    }
    
    public function loadAll(): array
    {
        $config = [];
        $files = glob($this->configDir . '/*.php');
        
        foreach ($files as $file) {
            $name = basename($file, '.php');
            $config[$name] = $this->load(basename($file));
        }
        
        return $config;
    }
}
```

```php
<?php
// main.php
declare(strict_types=1);

$loader = new ConfigLoader(__DIR__ . '/config');
$config = $loader->loadAll();

echo "应用: {$config['app']['app_name']}\n";
echo "数据库: {$config['database']['host']}\n";
```

## 最佳实践

1. **始终使用 `__DIR__`**：不依赖当前工作目录
2. **验证路径**：检查文件是否存在和可访问
3. **规范化路径**：使用 `realpath()` 处理符号链接
4. **安全验证**：防止路径遍历攻击
5. **返回值验证**：验证返回的数据类型和结构
6. **错误处理**：提供清晰的错误信息

## 注意事项

1. **路径分隔符**：PHP 自动处理正斜杠，但可以使用 `DIRECTORY_SEPARATOR`
2. **符号链接**：使用 `realpath()` 解析符号链接
3. **相对路径**：相对于当前文件，不是当前工作目录
4. **返回值类型**：验证返回值的类型和结构
5. **安全性**：验证路径，防止目录遍历攻击

## 练习

1. 创建一个配置加载器类，安全地加载配置文件。

2. 编写代码，演示如何使用 `__DIR__` 和 `dirname()` 构建路径。

3. 实现一个函数，验证文件路径是否在允许的目录内。

4. 创建一个配置系统，支持环境变量和配置文件合并。

5. 编写代码，演示配置文件的链式加载和合并。
