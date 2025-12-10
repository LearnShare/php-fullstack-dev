# 1.4.1 php.ini 配置

## 概述

`php.ini` 是 PHP 的主配置文件，控制 PHP 的行为和功能。理解如何配置 `php.ini` 对于开发和生产环境都至关重要。

## 定位 php.ini

### 查找配置文件

```bash
# 查看加载的配置文件
php --ini

# 输出示例：
# Configuration File (php.ini) Path: /usr/local/etc/php/8.2
# Loaded Configuration File:         /usr/local/etc/php/8.2/php.ini
# Scan for additional .ini files in: /usr/local/etc/php/8.2/conf.d
```

### 不同运行模式的配置

- **CLI 模式**：使用 `php --ini` 查看的配置文件
- **PHP-FPM 模式**：通常有独立的配置文件
- **Apache 模块**：使用 Apache 的配置文件

### 验证配置

```bash
# 查看所有配置
php -i

# 查看特定配置
php -i | grep memory_limit

# 在代码中查看
phpinfo();
```

## 常用配置项

### 错误报告

| 配置项 | 开发值 | 生产值 | 说明 |
| :----- | :----- | :----- | :--- |
| `display_errors` | `On` | `Off` | 是否在页面显示错误 |
| `display_startup_errors` | `On` | `Off` | 是否显示启动错误 |
| `error_reporting` | `E_ALL` | `E_ALL & ~E_DEPRECATED` | 错误报告级别 |
| `log_errors` | `On` | `On` | 是否记录错误日志 |
| `error_log` | `/path/to/error.log` | `/path/to/error.log` | 错误日志路径 |

### 内存和性能

| 配置项 | 推荐值 | 说明 |
| :----- | :----- | :--- |
| `memory_limit` | `256M` 或 `512M` | 脚本内存限制 |
| `max_execution_time` | `30`（开发）`300`（生产） | 最大执行时间（秒） |
| `max_input_time` | `60` | 最大输入时间（秒） |
| `post_max_size` | `8M` 或更大 | POST 数据最大大小 |
| `upload_max_filesize` | `2M` 或更大 | 上传文件最大大小 |

### OPcache（生产环境必需）

| 配置项 | 推荐值 | 说明 |
| :----- | :----- | :--- |
| `opcache.enable` | `1` | 启用 OPcache |
| `opcache.memory_consumption` | `128` | OPcache 内存大小（MB） |
| `opcache.interned_strings_buffer` | `8` | 字符串缓存大小（MB） |
| `opcache.max_accelerated_files` | `10000` | 最大缓存文件数 |
| `opcache.validate_timestamps` | `0`（生产）`1`（开发） | 是否验证文件时间戳 |

### 时区

| 配置项 | 推荐值 | 说明 |
| :----- | :----- | :--- |
| `date.timezone` | `Asia/Shanghai` | 默认时区 |

## 配置示例

### 开发环境配置

```ini
; php.ini.dev

; 错误报告
display_errors = On
display_startup_errors = On
error_reporting = E_ALL
log_errors = On
error_log = /var/log/php/error.log

; 内存和性能
memory_limit = 512M
max_execution_time = 30
max_input_time = 60
post_max_size = 8M
upload_max_filesize = 2M

; OPcache
opcache.enable = 1
opcache.validate_timestamps = 1

; 时区
date.timezone = Asia/Shanghai
```

### 生产环境配置

```ini
; php.ini.prod

; 错误报告
display_errors = Off
display_startup_errors = Off
error_reporting = E_ALL & ~E_DEPRECATED
log_errors = On
error_log = /var/log/php/error.log

; 内存和性能
memory_limit = 256M
max_execution_time = 300
max_input_time = 60
post_max_size = 8M
upload_max_filesize = 2M

; OPcache
opcache.enable = 1
opcache.memory_consumption = 128
opcache.interned_strings_buffer = 8
opcache.max_accelerated_files = 10000
opcache.validate_timestamps = 0
opcache.revalidate_freq = 0

; 时区
date.timezone = Asia/Shanghai
```

## 运行时配置

### ini_set() - 设置配置

**语法**：`ini_set(string $option, string|int|float|bool $value): string|false`

**参数**：
- `$option`：配置项名称
- `$value`：配置值

**返回值**：成功返回旧值，失败返回 `false`。

**示例**：

```php
<?php
declare(strict_types=1);

// 设置内存限制
ini_set('memory_limit', '512M');

// 设置错误报告
ini_set('display_errors', '1');
ini_set('error_reporting', E_ALL);

// 设置时区
ini_set('date.timezone', 'Asia/Shanghai');
```

### ini_get() - 获取配置

**语法**：`ini_get(string $option): string|false`

**示例**：

```php
<?php
declare(strict_types=1);

$memoryLimit = ini_get('memory_limit');
$timezone = ini_get('date.timezone');
$displayErrors = ini_get('display_errors');

echo "Memory Limit: {$memoryLimit}\n";
echo "Timezone: {$timezone}\n";
echo "Display Errors: {$displayErrors}\n";
```

### ini_get_all() - 获取所有配置

**语法**：`ini_get_all(?string $extension = null, bool $details = true): array`

**示例**：

```php
<?php
declare(strict_types=1);

// 获取所有配置
$all = ini_get_all();

// 获取特定扩展的配置
$opcache = ini_get_all('opcache');
```

## 环境相关配置

### 根据环境切换配置

```php
<?php
declare(strict_types=1);

$env = getenv('APP_ENV') ?: 'production';

if ($env === 'development') {
    ini_set('display_errors', '1');
    ini_set('error_reporting', E_ALL);
    ini_set('opcache.validate_timestamps', '1');
} else {
    ini_set('display_errors', '0');
    ini_set('error_reporting', E_ALL & ~E_DEPRECATED);
    ini_set('opcache.validate_timestamps', '0');
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class PhpConfig
{
    public static function setup(string $env = 'production'): void
    {
        // 时区
        ini_set('date.timezone', 'Asia/Shanghai');
        
        // 错误报告
        if ($env === 'development') {
            ini_set('display_errors', '1');
            ini_set('display_startup_errors', '1');
            error_reporting(E_ALL);
        } else {
            ini_set('display_errors', '0');
            ini_set('display_startup_errors', '0');
            error_reporting(E_ALL & ~E_DEPRECATED);
        }
        
        // 日志
        ini_set('log_errors', '1');
        ini_set('error_log', __DIR__ . '/logs/php-errors.log');
        
        // 内存和性能
        ini_set('memory_limit', $env === 'development' ? '512M' : '256M');
        ini_set('max_execution_time', $env === 'development' ? '30' : '300');
        
        // OPcache
        if (function_exists('opcache_reset')) {
            ini_set('opcache.enable', '1');
            ini_set('opcache.validate_timestamps', $env === 'development' ? '1' : '0');
        }
    }
    
    public static function getConfig(string $key): string|false
    {
        return ini_get($key);
    }
    
    public static function setConfig(string $key, string $value): bool
    {
        return ini_set($key, $value) !== false;
    }
}

// 使用
PhpConfig::setup(getenv('APP_ENV') ?: 'production');
```

## 注意事项

1. **配置文件位置**：CLI 和 FPM 可能使用不同的配置文件，需要分别配置。

2. **配置优先级**：`ini_set()` 设置的配置优先级高于 `php.ini`。

3. **生产环境**：生产环境必须关闭 `display_errors`，只记录日志。

4. **OPcache**：生产环境必须启用 OPcache，但关闭时间戳验证。

5. **配置验证**：修改配置后使用 `php -i` 验证是否生效。

## 练习

1. 创建开发和生产环境的 `php.ini` 配置文件。

2. 编写脚本，根据环境自动设置 PHP 配置。

3. 使用 `ini_get()` 和 `ini_set()` 动态管理配置。

4. 配置 OPcache，对比启用前后的性能差异。

5. 设置错误日志，验证错误记录是否正常工作。
