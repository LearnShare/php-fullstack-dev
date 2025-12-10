# 1.1.2 运行模式与验证

## 概述

PHP 支持多种运行模式，每种模式适用于不同的场景。理解这些模式的差异对于选择合适的部署方案至关重要。

## 运行模式对比

### 基本模式

| 模式 | 用途 | 特点 | 适用场景 |
| :--- | :--- | :--- | :------- |
| CLI | 命令行执行 | 单次执行，脚本结束后进程退出 | 脚本、迁移、队列 |
| 内置服务器 | 开发测试 | 单线程，不适合生产环境 | 快速演示、本地开发 |
| PHP-FPM | Web 应用 | 进程池管理，高性能 | 生产环境标准 |
| 常驻运行时 | 高性能场景 | 进程常驻内存，处理多个请求 | 高并发 API、WebSocket |

## CLI 模式（命令行）

### 特点

- **用途**：执行脚本、运行迁移、处理队列任务
- **特点**：单次执行，脚本结束后进程退出
- **类比**：类似 Node.js 的 CLI 脚本（`node script.php`）

### 基本使用

```php
<?php
// script.php
declare(strict_types=1);

echo "Hello from CLI\n";
```

```bash
# 执行
php script.php
```

### 检测 CLI 模式

```php
<?php
declare(strict_types=1);

if (php_sapi_name() === 'cli') {
    echo "Running in CLI mode\n";
}
```

## 内置 Web 服务器

### 特点

- **用途**：开发测试、快速演示
- **特点**：单线程，不适合生产环境
- **限制**：性能有限，功能简单

### 基本使用

```bash
# 启动内置服务器
php -S localhost:8000 -t public

# 指定路由脚本
php -S localhost:8000 router.php
```

### 路由脚本

```php
<?php
// router.php
declare(strict_types=1);

$path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

// 如果文件存在，直接返回
if (file_exists(__DIR__ . $path)) {
    return false;
}

// 否则路由到 index.php
require __DIR__ . '/index.php';
```

### 使用场景

- 快速测试 API
- 本地开发演示
- 学习 PHP 基础

## PHP-FPM 模式（生产环境）

### 特点

- **用途**：与 Nginx/Apache 配合处理 Web 请求
- **特点**：进程池管理，高性能，适合生产环境
- **架构**：Master-Worker 进程模型

### 启动方式

```bash
# 启动 PHP-FPM
php-fpm

# 或使用 systemd
sudo systemctl start php8.2-fpm

# 查看状态
sudo systemctl status php8.2-fpm
```

### 检测 FPM 模式

```php
<?php
declare(strict_types=1);

$sapi = php_sapi_name();
if (strpos($sapi, 'fpm') !== false) {
    echo "Running in PHP-FPM mode\n";
}
```

## 常驻运行时

### 特点

- **用途**：高并发场景、WebSocket、实时应用
- **特点**：进程常驻内存，可以处理多个请求
- **优势**：性能提升 2-10 倍

### 常见运行时

- **RoadRunner**：Go 语言编写
- **FrankenPHP**：Caddy + PHP
- **OpenSwoole**：C 扩展
- **Laravel Octane**：Laravel 专用

## 运行验证

### 验证脚本

```php
<?php
// verify.php
declare(strict_types=1);

echo "=== PHP Environment Verification ===\n";
echo "PHP Version: " . PHP_VERSION . "\n";
echo "SAPI: " . php_sapi_name() . "\n";
echo "OS: " . PHP_OS . "\n";
echo "Loaded Config: " . php_ini_loaded_file() . "\n";
echo "Extensions: " . implode(', ', get_loaded_extensions()) . "\n";
echo "Memory Limit: " . ini_get('memory_limit') . "\n";
echo "Max Execution Time: " . ini_get('max_execution_time') . "\n";
```

```bash
# 运行验证
php verify.php
```

### 验证不同模式

```php
<?php
declare(strict_types=1);

function detectMode(): string
{
    $sapi = php_sapi_name();
    
    return match($sapi) {
        'cli' => 'CLI Mode',
        'cli-server' => 'Built-in Server',
        'fpm-fcgi' => 'PHP-FPM',
        default => "Unknown: {$sapi}"
    };
}

echo "Current Mode: " . detectMode() . "\n";
```

## 完整示例

```php
<?php
declare(strict_types=1);

class RuntimeVerification
{
    public static function verify(): void
    {
        echo "=== PHP Runtime Verification ===\n";
        
        // 基本信息
        echo "PHP Version: " . PHP_VERSION . "\n";
        echo "SAPI: " . php_sapi_name() . "\n";
        echo "OS: " . PHP_OS . "\n";
        
        // 配置信息
        echo "\n=== Configuration ===\n";
        echo "Loaded Config: " . php_ini_loaded_file() . "\n";
        echo "Memory Limit: " . ini_get('memory_limit') . "\n";
        echo "Max Execution Time: " . ini_get('max_execution_time') . "\n";
        echo "Date Timezone: " . ini_get('date.timezone') . "\n";
        
        // 扩展信息
        echo "\n=== Extensions ===\n";
        $extensions = get_loaded_extensions();
        echo "Total: " . count($extensions) . "\n";
        echo "List: " . implode(', ', $extensions) . "\n";
        
        // 运行模式
        echo "\n=== Execution Mode ===\n";
        echo "Mode: " . self::detectMode() . "\n";
    }
    
    private static function detectMode(): string
    {
        $sapi = php_sapi_name();
        
        return match($sapi) {
            'cli' => 'CLI Mode',
            'cli-server' => 'Built-in Server',
            'fpm-fcgi' => 'PHP-FPM',
            default => "Unknown: {$sapi}"
        };
    }
}

RuntimeVerification::verify();
```

## 注意事项

1. **模式检测**：使用 `php_sapi_name()` 检测当前运行模式。

2. **性能差异**：不同模式的性能差异很大，选择合适的模式。

3. **内存管理**：常驻运行时需要注意内存泄漏。

4. **状态管理**：不同模式的状态管理方式不同。

5. **部署复杂度**：常驻运行时的部署和维护更复杂。

## 练习

1. 创建一个 CLI 脚本，执行数据库迁移任务。

2. 使用内置服务器启动一个简单的 Web 应用。

3. 检测当前 PHP 运行模式，根据模式执行不同逻辑。

4. 对比不同执行模式的性能差异。

5. 配置 PHP-FPM，处理 Web 请求。
