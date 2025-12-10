# 1.5.1 PHP 执行模式

## 概述

PHP 支持多种执行模式，每种模式适用于不同的场景。理解这些模式的差异对于选择合适的部署方案至关重要。

## 执行模式对比

### 与 Node.js 对比

如果你熟悉 Node.js，理解 PHP 执行模式的关键是：PHP 是**请求-响应**模式，而 Node.js 是**事件循环**模式。

**Node.js 执行模式**：
- 单进程事件循环
- 常驻内存，处理多个请求
- 异步 I/O，非阻塞操作

**PHP 执行模式**：
- 请求-响应模式（传统 FPM）
- 每个请求独立进程/线程
- 同步 I/O，阻塞操作（除非使用协程）

| 特性 | Node.js | PHP (FPM) | PHP (常驻运行时) |
| :--- | :------ | :-------- | :--------------- |
| 进程模型 | 单进程事件循环 | 多进程池 | 单进程/多进程可选 |
| 内存共享 | 是（全局变量） | 否（每个请求独立） | 是（常驻内存） |
| 启动开销 | 一次 | 每次请求 | 一次 |
| 适用场景 | I/O 密集型 | CPU 密集型 | 高并发场景 |

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

### 实际应用

```php
<?php
// migrate.php
declare(strict_types=1);

// 数据库迁移脚本
$pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');
$pdo->exec('CREATE TABLE users ...');
echo "Migration completed\n";
```

```bash
php migrate.php
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

### 进程模型

```
Master 进程
├── Worker 1 (处理请求)
├── Worker 2 (处理请求)
├── Worker 3 (处理请求)
└── ...
```

### 启动方式

```bash
# 启动 PHP-FPM
php-fpm

# 或使用 systemd
sudo systemctl start php8.2-fpm

# 查看状态
sudo systemctl status php8.2-fpm
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

## 完整示例

```php
<?php
declare(strict_types=1);

class ExecutionModes
{
    public static function demonstrateCLI(): void
    {
        echo "=== CLI Mode ===\n";
        echo "This is running in CLI mode\n";
        echo "PHP Version: " . PHP_VERSION . "\n";
        echo "SAPI: " . php_sapi_name() . "\n";
    }
    
    public static function detectMode(): string
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

// CLI 模式
if (php_sapi_name() === 'cli') {
    ExecutionModes::demonstrateCLI();
    echo "Mode: " . ExecutionModes::detectMode() . "\n";
}
```

## 选择建议

### 使用 CLI 的场景

- 命令行工具
- 数据库迁移
- 队列处理
- 定时任务

### 使用内置服务器的场景

- 本地开发
- 快速测试
- 学习演示

### 使用 PHP-FPM 的场景

- 传统 Web 应用
- 中等流量网站
- 需要稳定性的生产环境

### 使用常驻运行时的场景

- 高并发 API
- WebSocket 应用
- 实时应用
- 需要极致性能的场景

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
