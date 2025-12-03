# 1.6.4 Laravel Octane

## 概述

Laravel Octane 是 Laravel 框架专用的高性能运行时。它支持 RoadRunner 和 Swoole，可以在保持 Laravel 特性的同时获得显著的性能提升。

## 特点

- **框架**：Laravel 专用
- **支持**：RoadRunner、Swoole
- **优势**：框架级优化、保持 Laravel 特性
- **适用场景**：Laravel 应用、需要性能提升

## 安装

### 基本安装

```bash
# 安装 Octane
composer require laravel/octane

# 安装 Octane
php artisan octane:install

# 选择运行时
# 1. RoadRunner
# 2. Swoole
```

### 安装 RoadRunner

```bash
# 如果选择 RoadRunner
composer require spiral/roadrunner
vendor/bin/rr get-binary
```

### 安装 Swoole

```bash
# 如果选择 Swoole
pecl install swoole
# 或在 php.ini 中启用
# extension=swoole.so
```

## 配置

### 基本配置（config/octane.php）

```php
<?php

return [
    'server' => env('OCTANE_SERVER', 'roadrunner'),
    
    'servers' => [
        'roadrunner' => [
            'host' => env('OCTANE_HOST', '127.0.0.1'),
            'port' => env('OCTANE_PORT', 8000),
            'workers' => env('OCTANE_WORKERS', 4),
        ],
        
        'swoole' => [
            'host' => env('OCTANE_HOST', '127.0.0.1'),
            'port' => env('OCTANE_PORT', 8000),
            'workers' => env('OCTANE_WORKERS', 4),
            'task_workers' => env('OCTANE_TASK_WORKERS', 4),
        ],
    ],
];
```

### 环境变量（.env）

```env
OCTANE_SERVER=roadrunner
OCTANE_HOST=127.0.0.1
OCTANE_PORT=8000
OCTANE_WORKERS=4
```

## 启动

### 开发环境

```bash
# 启动 Octane
php artisan octane:serve

# 指定服务器
php artisan octane:serve --server=roadrunner

# 指定端口
php artisan octane:serve --port=8000
```

### 生产环境

```bash
# 启动 Octane
php artisan octane:start --server=roadrunner --workers=4

# 后台运行
php artisan octane:start --server=roadrunner --workers=4 --daemon
```

## 状态管理

### 需要重置的状态

在常驻运行时中，以下状态需要在每个请求后重置：

- **单例服务**：Laravel 容器中的单例
- **静态属性**：类的静态属性
- **全局变量**：全局变量

### 使用监听器重置

```php
<?php
// app/Providers/OctaneServiceProvider.php

use Laravel\Octane\Events\RequestReceived;
use Illuminate\Support\Facades\Event;

public function boot(): void
{
    Event::listen(RequestReceived::class, function () {
        // 重置单例
        app()->forgetInstance(SomeService::class);
        
        // 重置静态属性
        SomeClass::resetStatic();
    });
}
```

### 自动重置

Laravel Octane 会自动重置：
- 请求状态
- 认证状态
- Session 数据
- 缓存（如果使用文件缓存）

## 连接复用

### 数据库连接

```php
<?php
// app/Providers/OctaneServiceProvider.php

use Laravel\Octane\Events\RequestReceived;

public function boot(): void
{
    Event::listen(RequestReceived::class, function () {
        // 复用数据库连接
        // Octane 会自动处理
    });
}
```

### Redis 连接

```php
<?php
// Redis 连接会自动复用
// 无需额外配置
```

## 完整示例

### 项目配置

```
laravel-app/
├── config/
│   └── octane.php
├── .env
└── app/
    └── Providers/
        └── OctaneServiceProvider.php
```

### config/octane.php

```php
<?php

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

### app/Providers/OctaneServiceProvider.php

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Octane\Events\RequestReceived;
use Illuminate\Support\Facades\Event;

class OctaneServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Event::listen(RequestReceived::class, function () {
            // 重置单例服务
            app()->forgetInstance(SomeService::class);
        });
    }
}
```

## 性能对比

### 与 PHP-FPM 对比

| 指标 | PHP-FPM | Laravel Octane |
| :--- | :------ | :------------- |
| 请求/秒 | 基准 | 2-10 倍提升 |
| 内存使用 | 每个请求独立 | 进程间共享 |
| 启动时间 | 每次请求 | 仅启动时 |
| 连接复用 | 不支持 | 支持 |

## 注意事项

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
        Cache::increment('counter');
    }
}
```

### 2. 单例服务

```php
<?php
// 使用监听器重置单例
Event::listen(RequestReceived::class, function () {
    app()->forgetInstance(SomeService::class);
});
```

### 3. 静态属性

```php
<?php
// 避免使用静态属性存储请求间共享的状态
// 使用外部存储（数据库、缓存）代替
```

### 4. 文件缓存

```php
<?php
// 不要使用文件缓存（多进程会有问题）
// 使用 Redis 或 Memcached
```

## 最佳实践

### 1. Worker 数量配置

```php
// config/octane.php
'workers' => env('OCTANE_WORKERS', 4),  // 根据 CPU 核心数调整
```

### 2. 内存管理

```php
<?php
use Laravel\Octane\Events\TaskReceived;

Event::listen(TaskReceived::class, function () {
    // 定期清理内存
    Cache::flush();
    app()->forgetInstances();
});
```

### 3. 错误处理

```php
<?php
use Laravel\Octane\Events\RequestTerminated;

Event::listen(RequestTerminated::class, function ($event) {
    if ($event->exception) {
        // 记录错误
        Log::error($event->exception);
    }
});
```

## 完整示例

### 启动脚本

```bash
#!/bin/bash
# start-octane.sh

php artisan octane:start \
    --server=roadrunner \
    --workers=4 \
    --host=0.0.0.0 \
    --port=8000
```

### 监控脚本

```php
<?php
// routes/web.php

Route::get('/octane/status', function () {
    return [
        'server' => config('octane.server'),
        'workers' => config('octane.servers.roadrunner.workers'),
        'memory' => memory_get_usage(true),
    ];
})->middleware('auth');
```

## 注意事项

1. **状态重置**：确保每个请求后重置需要重置的状态。

2. **连接复用**：利用连接复用提升性能，但要注意连接管理。

3. **错误处理**：妥善处理异常，避免 Worker 崩溃。

4. **性能监控**：监控 Worker 状态和内存使用。

5. **部署复杂度**：Octane 的部署和维护比传统 PHP-FPM 更复杂。

## 练习

1. 在 Laravel 项目中安装和配置 Octane。

2. 使用 RoadRunner 运行 Laravel 应用，对比性能差异。

3. 实现状态重置逻辑，确保应用正常运行。

4. 配置连接复用，提升数据库和缓存性能。

5. 实现性能监控，跟踪 Octane 的运行状态。
