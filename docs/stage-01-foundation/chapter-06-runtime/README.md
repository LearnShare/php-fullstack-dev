# 1.6 现代 PHP 运行时与新生态

## 目标

- 了解现代 PHP 运行时的优势和使用场景。
- 掌握 RoadRunner、FrankenPHP、OpenSwoole 的特点。
- 理解 Laravel Octane 的作用和配置。
- 能够选择合适的运行时方案。

## 传统 PHP-FPM 的局限性

### 问题

1. **进程创建开销**：每个请求可能创建新进程
2. **内存无法复用**：代码每次都需要重新加载
3. **连接无法复用**：数据库连接每次重新建立
4. **启动延迟**：首次请求需要加载框架和依赖

### 性能对比

| 指标           | PHP-FPM        | 常驻运行时     |
| :------------- | :------------- | :------------- |
| 请求处理速度   | 基准           | 2-10 倍提升    |
| 内存使用       | 每个请求独立   | 进程间共享     |
| 连接复用       | 不支持         | 支持           |
| 启动时间       | 每次请求       | 仅启动时       |

## RoadRunner

### 特点

- **语言**：Go 语言编写
- **架构**：Worker 进程池 + HTTP 服务器
- **优势**：高性能、低内存占用、易于部署

### 安装和配置

```bash
# 安装 RoadRunner
composer require spiral/roadrunner

# 生成配置文件
vendor/bin/rr get

# 启动
./rr serve
```

### 配置文件

```yaml
# .rr.yaml
version: "3"

http:
  address: 0.0.0.0:8080
  middleware: ["gzip"]
  pool:
    num_workers: 4
    max_jobs: 0
    allocate_timeout: 60s
    destroy_timeout: 60s

server:
  command: "php worker.php"

logs:
  level: error
```

### Worker 文件

```php
<?php
// worker.php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Spiral\RoadRunner\Http\PSR7Worker;
use Spiral\RoadRunner\Worker;
use Nyholm\Psr7\Factory\Psr17Factory;

$worker = Worker::create();
$factory = new Psr17Factory();
$psr7 = new PSR7Worker($worker, $factory, $factory, $factory);

while ($request = $psr7->waitRequest()) {
    try {
        // 处理请求
        $response = handleRequest($request);
        $psr7->respond($response);
    } catch (Throwable $e) {
        $psr7->getWorker()->error((string) $e);
    }
}
```

## FrankenPHP

### 特点

- **语言**：Caddy 服务器 + PHP 集成
- **架构**：内置 HTTP 服务器 + Worker 模式
- **优势**：开箱即用、自动 HTTPS、HTTP/2、HTTP/3

### 安装

```bash
# 使用 Docker
docker run -v $PWD:/app \
  -p 80:80 -p 443:443 \
  dunglas/frankenphp

# 或使用 Homebrew (macOS)
brew install dunglas/frankenphp/frankenphp
```

### 配置

```ini
; php.ini
frankenphp.worker = 1
frankenphp.worker_count = 4
```

### 使用

```bash
# 启动 FrankenPHP
frankenphp serve

# 或使用 Docker
docker run -v $PWD:/app -p 80:80 dunglas/frankenphp
```

## OpenSwoole

### 特点

- **语言**：C 扩展
- **架构**：事件驱动 + 协程
- **优势**：极高性能、支持 WebSocket、异步 I/O

### 安装

```bash
# 使用 PECL
pecl install openswoole

# 或使用包管理器
# Ubuntu/Debian
sudo apt install php-openswoole

# macOS
brew install openswoole
```

### 基础使用

```php
<?php
declare(strict_types=1);

use OpenSwoole\Http\Server;
use OpenSwoole\Http\Request;
use OpenSwoole\Http\Response;

$server = new Server("0.0.0.0", 9501);

$server->on("request", function (Request $request, Response $response) {
    $response->header("Content-Type", "text/plain");
    $response->end("Hello World\n");
});

$server->start();
```

### 协程示例

```php
<?php
declare(strict_types=1);

use OpenSwoole\Coroutine;

Coroutine::create(function () {
    // 协程 1
    $result1 = Coroutine::create(function () {
        // 模拟异步操作
        Coroutine::sleep(1);
        return "Result 1";
    });
    
    // 协程 2
    $result2 = Coroutine::create(function () {
        Coroutine::sleep(1);
        return "Result 2";
    });
    
    // 等待两个协程完成
    echo $result1 . "\n";
    echo $result2 . "\n";
});
```

## Laravel Octane

### 特点

- **框架**：Laravel 专用
- **支持**：RoadRunner、Swoole
- **优势**：框架级优化、保持 Laravel 特性

### 安装

```bash
composer require laravel/octane

php artisan octane:install

# 选择运行时
# 1. RoadRunner
# 2. Swoole
```

### 配置

```php
// config/octane.php
return [
    'server' => env('OCTANE_SERVER', 'roadrunner'),
    
    'servers' => [
        'roadrunner' => [
            'host' => env('OCTANE_HOST', '127.0.0.1'),
            'port' => env('OCTANE_PORT', 8000),
            'workers' => env('OCTANE_WORKERS', 4),
        ],
    ],
];
```

### 启动

```bash
# 开发环境
php artisan octane:serve

# 生产环境
php artisan octane:start --server=roadrunner --workers=4
```

### 注意事项

```php
<?php
// 需要重置的状态
// 1. 单例服务
// 2. 静态属性
// 3. 全局变量

// 使用 Octane 的监听器重置状态
use Laravel\Octane\Events\RequestReceived;

Event::listen(RequestReceived::class, function () {
    // 重置单例
    app()->forgetInstance(SomeService::class);
});
```

## 运行时对比

| 特性           | PHP-FPM      | RoadRunner   | FrankenPHP  | OpenSwoole  |
| :------------- | :----------- | :------------ | :----------- | :----------- |
| 性能           | 基准         | 高            | 高           | 极高         |
| 内存占用       | 高           | 低            | 中           | 低           |
| 部署复杂度     | 低           | 中            | 低           | 高           |
| WebSocket 支持 | 否           | 是            | 是           | 是           |
| HTTP/2 支持    | 需 Nginx     | 是            | 是           | 是           |
| 协程支持       | 否           | 否            | 否           | 是           |
| Laravel 支持   | 原生         | Octane       | Octane       | Octane       |

## 选择建议

### 使用 PHP-FPM 的场景

- 传统 Web 应用
- 低到中等流量
- 简单的部署需求
- 不需要 WebSocket

### 使用 RoadRunner 的场景

- 高并发 API 服务
- 需要性能提升
- Go 生态集成
- Laravel 应用（通过 Octane）

### 使用 FrankenPHP 的场景

- 需要自动 HTTPS
- 需要 HTTP/2/3
- 简单的部署
- Laravel 应用（通过 Octane）

### 使用 OpenSwoole 的场景

- 极高性能要求
- 需要 WebSocket
- 需要协程
- 实时应用

## 最佳实践

### 1. 状态管理

```php
<?php
// 避免在 Worker 中存储状态
// 错误示例
class Counter
{
    private static int $count = 0; // 会在 Worker 间共享
    
    public static function increment(): void
    {
        self::$count++;
    }
}

// 正确做法：使用外部存储
class Counter
{
    public static function increment(): void
    {
        $redis = new Redis();
        $redis->incr('counter');
    }
}
```

### 2. 连接复用

```php
<?php
// 在 Worker 启动时创建连接
// worker.php
$db = new PDO($dsn, $user, $pass);

while ($request = $psr7->waitRequest()) {
    // 复用数据库连接
    $result = $db->query('SELECT * FROM users');
    // ...
}
```

### 3. 内存管理

```php
<?php
// 定期清理内存
use Laravel\Octane\Events\TaskReceived;

Event::listen(TaskReceived::class, function () {
    // 清理缓存
    Cache::flush();
    
    // 重置单例
    app()->forgetInstances();
});
```

## 练习

1. 安装和配置 RoadRunner，创建一个简单的 HTTP 服务器。

2. 使用 FrankenPHP 部署一个 Laravel 应用，对比性能差异。

3. 使用 OpenSwoole 创建一个 WebSocket 服务器。

4. 配置 Laravel Octane，使用 RoadRunner 运行 Laravel 应用。

5. 实现连接池，在常驻运行时中复用数据库连接。

6. 创建一个性能测试，对比不同运行时的性能。

