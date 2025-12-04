# 6.3.2 环境变量与配置

## 概述

环境变量和配置管理是应用开发的基础。本节介绍环境变量的管理、配置管理的最佳实践、多环境配置的实现，以及配置验证的方法。

## 环境变量管理

### 基础环境变量操作

PHP 提供了多种方式访问环境变量：

```php
<?php
declare(strict_types=1);

// 方式 1: $_ENV 超全局变量
$dbHost = $_ENV['DB_HOST'] ?? 'localhost';

// 方式 2: $_SERVER 超全局变量
$dbHost = $_SERVER['DB_HOST'] ?? 'localhost';

// 方式 3: getenv() 函数
$dbHost = getenv('DB_HOST') ?: 'localhost';
```

### 环境变量管理类

创建一个统一的环境变量管理类，提供类型转换和验证功能：

```php
<?php
declare(strict_types=1);

class Env
{
    /**
     * 获取环境变量
     */
    public static function get(string $key, mixed $default = null): mixed
    {
        $value = $_ENV[$key] ?? $_SERVER[$key] ?? getenv($key);
        return $value !== false ? $value : $default;
    }
    
    /**
     * 获取整数类型的环境变量
     */
    public static function getInt(string $key, int $default = 0): int
    {
        $value = self::get($key);
        if ($value === null) {
            return $default;
        }
        return (int) $value;
    }
    
    /**
     * 获取布尔类型的环境变量
     */
    public static function getBool(string $key, bool $default = false): bool
    {
        $value = self::get($key);
        if ($value === null) {
            return $default;
        }
        
        if (is_bool($value)) {
            return $value;
        }
        
        if (is_string($value)) {
            $value = strtolower($value);
            return in_array($value, ['1', 'true', 'on', 'yes'], true);
        }
        
        return (bool) $value;
    }
    
    /**
     * 获取浮点数类型的环境变量
     */
    public static function getFloat(string $key, float $default = 0.0): float
    {
        $value = self::get($key);
        if ($value === null) {
            return $default;
        }
        return (float) $value;
    }
    
    /**
     * 获取数组类型的环境变量（JSON 格式）
     */
    public static function getArray(string $key, array $default = []): array
    {
        $value = self::get($key);
        if ($value === null) {
            return $default;
        }
        
        if (is_array($value)) {
            return $value;
        }
        
        $decoded = json_decode($value, true);
        if (json_last_error() === JSON_ERROR_NONE) {
            return $decoded;
        }
        
        // 尝试逗号分隔的字符串
        return explode(',', $value);
    }
    
    /**
     * 获取必需的环境变量（不存在时抛出异常）
     */
    public static function require(string $key): string
    {
        $value = self::get($key);
        if ($value === null || $value === '') {
            throw new RuntimeException("Required environment variable {$key} is not set");
        }
        return (string) $value;
    }
    
    /**
     * 获取必需的整数类型环境变量
     */
    public static function requireInt(string $key): int
    {
        $value = self::require($key);
        if (!is_numeric($value)) {
            throw new InvalidArgumentException("Environment variable {$key} must be an integer");
        }
        return (int) $value;
    }
}

// 使用示例
$dbHost = Env::require('DB_HOST');
$dbPort = Env::getInt('DB_PORT', 3306);
$debug = Env::getBool('APP_DEBUG', false);
$maxConnections = Env::getInt('MAX_CONNECTIONS', 100);
```

## 配置管理

### 配置类设计

创建一个配置管理类，支持从多个来源读取配置：

```php
<?php
declare(strict_types=1);

class Config
{
    private static array $config = [];
    private static bool $loaded = false;
    
    /**
     * 加载配置
     */
    public static function load(array $sources = []): void
    {
        if (self::$loaded) {
            return;
        }
        
        // 1. 从环境变量加载
        self::loadFromEnv();
        
        // 2. 从配置文件加载
        if (file_exists(__DIR__ . '/config.php')) {
            self::loadFromFile(__DIR__ . '/config.php');
        }
        
        // 3. 从其他来源加载
        foreach ($sources as $source) {
            if (is_callable($source)) {
                $config = $source();
                self::$config = array_merge(self::$config, $config);
            } elseif (is_array($source)) {
                self::$config = array_merge(self::$config, $source);
            }
        }
        
        self::$loaded = true;
    }
    
    /**
     * 从环境变量加载配置
     */
    private static function loadFromEnv(): void
    {
        // 数据库配置
        self::$config['database'] = [
            'host' => Env::get('DB_HOST', 'localhost'),
            'port' => Env::getInt('DB_PORT', 3306),
            'database' => Env::require('DB_DATABASE'),
            'username' => Env::require('DB_USERNAME'),
            'password' => Env::require('DB_PASSWORD'),
            'charset' => Env::get('DB_CHARSET', 'utf8mb4'),
        ];
        
        // 应用配置
        self::$config['app'] = [
            'name' => Env::get('APP_NAME', 'My App'),
            'env' => Env::get('APP_ENV', 'production'),
            'debug' => Env::getBool('APP_DEBUG', false),
            'url' => Env::get('APP_URL', 'http://localhost'),
        ];
        
        // Redis 配置
        self::$config['redis'] = [
            'host' => Env::get('REDIS_HOST', '127.0.0.1'),
            'port' => Env::getInt('REDIS_PORT', 6379),
            'password' => Env::get('REDIS_PASSWORD'),
            'database' => Env::getInt('REDIS_DATABASE', 0),
        ];
    }
    
    /**
     * 从文件加载配置
     */
    private static function loadFromFile(string $file): void
    {
        $config = require $file;
        if (is_array($config)) {
            self::$config = array_merge(self::$config, $config);
        }
    }
    
    /**
     * 获取配置值
     */
    public static function get(string $key, mixed $default = null): mixed
    {
        if (!self::$loaded) {
            self::load();
        }
        
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
    
    /**
     * 设置配置值
     */
    public static function set(string $key, mixed $value): void
    {
        $keys = explode('.', $key);
        $config = &self::$config;
        
        foreach ($keys as $k) {
            if (!isset($config[$k]) || !is_array($config[$k])) {
                $config[$k] = [];
            }
            $config = &$config[$k];
        }
        
        $config = $value;
    }
    
    /**
     * 检查配置是否存在
     */
    public static function has(string $key): bool
    {
        if (!self::$loaded) {
            self::load();
        }
        
        $keys = explode('.', $key);
        $value = self::$config;
        
        foreach ($keys as $k) {
            if (!isset($value[$k])) {
                return false;
            }
            $value = $value[$k];
        }
        
        return true;
    }
    
    /**
     * 获取所有配置
     */
    public static function all(): array
    {
        if (!self::$loaded) {
            self::load();
        }
        
        return self::$config;
    }
}

// 使用示例
Config::load();

$dbHost = Config::get('database.host');
$appName = Config::get('app.name');
$redisPort = Config::get('redis.port', 6379);
```

## 多环境配置

### 环境检测

```php
<?php
declare(strict_types=1);

class Environment
{
    public const PRODUCTION = 'production';
    public const STAGING = 'staging';
    public const TESTING = 'testing';
    public const DEVELOPMENT = 'development';
    
    private static ?string $current = null;
    
    /**
     * 获取当前环境
     */
    public static function current(): string
    {
        if (self::$current === null) {
            self::$current = Env::get('APP_ENV', self::PRODUCTION);
        }
        
        return self::$current;
    }
    
    /**
     * 检查是否为指定环境
     */
    public static function is(string $environment): bool
    {
        return self::current() === $environment;
    }
    
    /**
     * 检查是否为生产环境
     */
    public static function isProduction(): bool
    {
        return self::is(self::PRODUCTION);
    }
    
    /**
     * 检查是否为开发环境
     */
    public static function isDevelopment(): bool
    {
        return self::is(self::DEVELOPMENT);
    }
    
    /**
     * 检查是否为测试环境
     */
    public static function isTesting(): bool
    {
        return self::is(self::TESTING);
    }
    
    /**
     * 检查是否为本地环境
     */
    public static function isLocal(): bool
    {
        return in_array(self::current(), [self::DEVELOPMENT, self::TESTING], true);
    }
}
```

### 环境特定配置

```php
<?php
declare(strict_types=1);

class ConfigManager
{
    private array $config = [];
    private string $environment;
    
    public function __construct(string $environment)
    {
        $this->environment = $environment;
        $this->loadConfig();
    }
    
    /**
     * 加载配置
     */
    private function loadConfig(): void
    {
        // 1. 加载基础配置
        $baseConfig = $this->loadConfigFile(__DIR__ . '/config/base.php');
        
        // 2. 加载环境特定配置
        $envConfig = $this->loadConfigFile(__DIR__ . "/config/{$this->environment}.php");
        
        // 3. 合并配置（环境配置覆盖基础配置）
        $this->config = array_merge_recursive($baseConfig, $envConfig);
        
        // 4. 从环境变量覆盖
        $this->overrideFromEnv();
    }
    
    /**
     * 加载配置文件
     */
    private function loadConfigFile(string $file): array
    {
        if (!file_exists($file)) {
            return [];
        }
        
        return require $file;
    }
    
    /**
     * 从环境变量覆盖配置
     */
    private function overrideFromEnv(): void
    {
        // 数据库配置
        if (Env::has('DB_HOST')) {
            $this->config['database']['host'] = Env::get('DB_HOST');
        }
        
        // 应用配置
        if (Env::has('APP_DEBUG')) {
            $this->config['app']['debug'] = Env::getBool('APP_DEBUG');
        }
    }
    
    /**
     * 获取配置
     */
    public function get(string $key, mixed $default = null): mixed
    {
        $keys = explode('.', $key);
        $value = $this->config;
        
        foreach ($keys as $k) {
            if (!isset($value[$k])) {
                return $default;
            }
            $value = $value[$k];
        }
        
        return $value;
    }
}

// 配置文件示例
// config/base.php
return [
    'app' => [
        'name' => 'My App',
        'timezone' => 'UTC',
    ],
    'database' => [
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
    ],
];

// config/development.php
return [
    'app' => [
        'debug' => true,
    ],
    'database' => [
        'host' => 'localhost',
        'port' => 3306,
    ],
];

// config/production.php
return [
    'app' => [
        'debug' => false,
    ],
    'database' => [
        'host' => 'db.example.com',
        'port' => 3306,
    ],
];
```

## 配置验证

### 配置验证器

```php
<?php
declare(strict_types=1);

class ConfigValidator
{
    private array $rules = [];
    private array $errors = [];
    
    /**
     * 添加验证规则
     */
    public function rule(string $key, callable $validator, string $message = ''): self
    {
        $this->rules[$key] = [
            'validator' => $validator,
            'message' => $message ?: "Configuration key {$key} is invalid",
        ];
        
        return $this;
    }
    
    /**
     * 验证配置
     */
    public function validate(array $config): bool
    {
        $this->errors = [];
        
        foreach ($this->rules as $key => $rule) {
            $value = $this->getNestedValue($config, $key);
            
            if (!$rule['validator']($value)) {
                $this->errors[$key] = $rule['message'];
            }
        }
        
        return empty($this->errors);
    }
    
    /**
     * 获取嵌套值
     */
    private function getNestedValue(array $config, string $key): mixed
    {
        $keys = explode('.', $key);
        $value = $config;
        
        foreach ($keys as $k) {
            if (!isset($value[$k])) {
                return null;
            }
            $value = $value[$k];
        }
        
        return $value;
    }
    
    /**
     * 获取验证错误
     */
    public function getErrors(): array
    {
        return $this->errors;
    }
    
    /**
     * 检查是否有错误
     */
    public function hasErrors(): bool
    {
        return !empty($this->errors);
    }
}

// 使用示例
$validator = new ConfigValidator();

$validator
    ->rule('database.host', fn($v) => !empty($v), 'Database host is required')
    ->rule('database.port', fn($v) => is_int($v) && $v > 0 && $v < 65536, 'Database port must be between 1 and 65535')
    ->rule('database.username', fn($v) => !empty($v), 'Database username is required')
    ->rule('database.password', fn($v) => !empty($v), 'Database password is required')
    ->rule('app.url', fn($v) => filter_var($v, FILTER_VALIDATE_URL) !== false, 'App URL must be a valid URL');

$config = Config::all();

if (!$validator->validate($config)) {
    $errors = $validator->getErrors();
    foreach ($errors as $key => $message) {
        echo "Error in {$key}: {$message}\n";
    }
    exit(1);
}
```

### 配置验证工具

创建一个命令行工具来验证配置：

```php
<?php
declare(strict_types=1);

// bin/validate-config.php

require __DIR__ . '/../vendor/autoload.php';

use Dotenv\Dotenv;

// 加载环境变量
$dotenv = Dotenv::createImmutable(__DIR__ . '/..');
$dotenv->load();

// 定义必需的配置项
$requiredConfigs = [
    'DB_HOST',
    'DB_DATABASE',
    'DB_USERNAME',
    'DB_PASSWORD',
    'APP_KEY',
    'APP_URL',
];

$missing = [];
$invalid = [];

foreach ($requiredConfigs as $key) {
    $value = Env::get($key);
    
    if ($value === null || $value === '') {
        $missing[] = $key;
    } else {
        // 验证特定配置
        switch ($key) {
            case 'APP_URL':
                if (filter_var($value, FILTER_VALIDATE_URL) === false) {
                    $invalid[$key] = 'Must be a valid URL';
                }
                break;
                
            case 'DB_PORT':
                $port = (int) $value;
                if ($port < 1 || $port > 65535) {
                    $invalid[$key] = 'Must be between 1 and 65535';
                }
                break;
        }
    }
}

// 输出结果
if (!empty($missing)) {
    echo "Missing required configuration:\n";
    foreach ($missing as $key) {
        echo "  - {$key}\n";
    }
    exit(1);
}

if (!empty($invalid)) {
    echo "Invalid configuration:\n";
    foreach ($invalid as $key => $message) {
        echo "  - {$key}: {$message}\n";
    }
    exit(1);
}

echo "Configuration is valid!\n";
exit(0);
```

## 完整示例

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Dotenv\Dotenv;

// 1. 加载环境变量
$dotenv = Dotenv::createImmutable(__DIR__);
$dotenv->load();

// 2. 验证必需的环境变量
$dotenv->required([
    'DB_HOST',
    'DB_DATABASE',
    'DB_USERNAME',
    'DB_PASSWORD',
    'APP_KEY',
]);

// 3. 初始化配置管理器
$environment = Env::get('APP_ENV', 'production');
$config = new ConfigManager($environment);

// 4. 验证配置
$validator = new ConfigValidator();
$validator
    ->rule('database.host', fn($v) => !empty($v))
    ->rule('database.port', fn($v) => is_int($v) && $v > 0)
    ->rule('app.url', fn($v) => filter_var($v, FILTER_VALIDATE_URL) !== false);

if (!$validator->validate($config->all())) {
    $errors = $validator->getErrors();
    foreach ($errors as $key => $message) {
        error_log("Config validation error: {$key} - {$message}");
    }
    throw new RuntimeException('Configuration validation failed');
}

// 5. 使用配置
$dbHost = $config->get('database.host');
$dbPort = $config->get('database.port', 3306);
$appName = $config->get('app.name');
```

## 最佳实践

### 1. 使用 .env.example 作为模板

```bash
# .env.example
APP_NAME=My App
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost

DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=myapp
DB_USERNAME=root
DB_PASSWORD=
```

### 2. 环境特定配置

- 开发环境：使用 `.env.local`
- 测试环境：使用 `.env.testing`
- 生产环境：使用环境变量或密钥管理服务

### 3. 配置验证

- 应用启动时验证所有必需配置
- 提供清晰的错误信息
- 使用类型安全的配置访问

### 4. 配置缓存

```php
<?php
declare(strict_types=1);

// 生产环境可以缓存配置以提高性能
if (Environment::isProduction()) {
    $cacheFile = __DIR__ . '/cache/config.php';
    
    if (file_exists($cacheFile)) {
        $config = require $cacheFile;
    } else {
        $config = Config::all();
        file_put_contents($cacheFile, "<?php\nreturn " . var_export($config, true) . ";\n");
    }
}
```

## 注意事项

1. **敏感信息**：不要将敏感配置提交到版本控制
2. **类型安全**：使用类型转换确保配置值的正确类型
3. **默认值**：为可选配置提供合理的默认值
4. **验证**：在应用启动时验证所有必需配置

## 练习

1. 创建一个环境变量管理类，支持类型转换和必需变量验证。

2. 实现一个配置管理类，支持从多个来源（.env、配置文件、数据库）读取配置，并支持配置合并。

3. 设计一个多环境配置系统，支持开发、测试、生产等不同环境的配置。

4. 创建一个配置验证工具，检查所有必需的配置项是否已设置，并验证配置值的有效性。

5. 实现一个配置缓存机制，在生产环境中缓存配置以提高性能。
