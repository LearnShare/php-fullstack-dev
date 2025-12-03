# 1.6.1 RoadRunner

## 概述

RoadRunner 是用 Go 语言编写的高性能 PHP 应用服务器。它通过 Worker 进程池和 HTTP 服务器提供比传统 PHP-FPM 更高的性能。

## 特点

- **语言**：Go 语言编写
- **架构**：Worker 进程池 + HTTP 服务器
- **优势**：高性能、低内存占用、易于部署
- **适用场景**：高并发 API 服务、Laravel 应用（通过 Octane）

## 安装

### 方式一：Composer

```bash
# 安装 RoadRunner
composer require spiral/roadrunner

# 生成配置文件
vendor/bin/rr get

# 下载 RoadRunner 二进制
vendor/bin/rr get-binary
```

### 方式二：直接下载

```bash
# 下载二进制文件
wget https://github.com/roadrunner-server/roadrunner/releases/download/v2023.3.0/roadrunner-2023.3.0-linux-amd64.tar.gz
tar -xzf roadrunner-2023.3.0-linux-amd64.tar.gz
sudo mv rr /usr/local/bin/
```

## 配置

### 基本配置（.rr.yaml）

```yaml
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

### 配置参数说明

| 参数 | 说明 | 默认值 |
| :--- | :--- | :----- |
| `http.address` | HTTP 服务器地址 | `0.0.0.0:8080` |
| `http.middleware` | 中间件列表 | `[]` |
| `pool.num_workers` | Worker 进程数 | `2` |
| `pool.max_jobs` | 每个 Worker 最大任务数 | `0`（无限制） |
| `pool.allocate_timeout` | Worker 分配超时 | `60s` |
| `pool.destroy_timeout` | Worker 销毁超时 | `60s` |

## Worker 文件

### 基本 Worker

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

function handleRequest($request)
{
    // 处理逻辑
    return new \Nyholm\Psr7\Response(200, [], 'Hello from RoadRunner');
}
```

### 使用框架

```php
<?php
// worker.php (Laravel)
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Illuminate\Http\Request;
use Spiral\RoadRunner\Http\PSR7Worker;
use Spiral\RoadRunner\Worker;
use Nyholm\Psr7\Factory\Psr17Factory;

$app = require_once __DIR__ . '/bootstrap/app.php';

$worker = Worker::create();
$factory = new Psr17Factory();
$psr7 = new PSR7Worker($worker, $factory, $factory, $factory);

while ($request = $psr7->waitRequest()) {
    try {
        $laravelRequest = Request::createFromPsr7($request);
        $response = $app->handle($laravelRequest);
        $psr7->respond($response->toPsr7());
    } catch (Throwable $e) {
        $psr7->getWorker()->error((string) $e);
    }
}
```

## 启动和停止

### 启动

```bash
# 启动 RoadRunner
./rr serve

# 或使用二进制
rr serve

# 后台运行
rr serve -d
```

### 停止

```bash
# 发送 SIGTERM 信号
kill -TERM <pid>

# 或使用 Ctrl+C
```

## 性能优化

### Worker 数量配置

```yaml
http:
  pool:
    num_workers: 8  # 根据 CPU 核心数调整
    max_jobs: 1000   # 防止内存泄漏
```

### 内存管理

```php
<?php
// 在 Worker 中定期清理
while ($request = $psr7->waitRequest()) {
    try {
        // 处理请求
        $response = handleRequest($request);
        $psr7->respond($response);
        
        // 定期清理（每 100 个请求）
        if (++$requestCount % 100 === 0) {
            gc_collect_cycles();
        }
    } catch (Throwable $e) {
        $psr7->getWorker()->error((string) $e);
    }
}
```

## 完整示例

### 项目结构

```
my-project/
├── .rr.yaml
├── worker.php
├── composer.json
└── src/
    └── App.php
```

### .rr.yaml

```yaml
version: "3"

http:
  address: 0.0.0.0:8080
  middleware: ["gzip"]
  pool:
    num_workers: 4
    max_jobs: 1000
    allocate_timeout: 60s
    destroy_timeout: 60s

server:
  command: "php worker.php"

logs:
  level: error
```

### worker.php

```php
<?php
declare(strict_types=1);

require __DIR__ . '/vendor/autoload.php';

use Spiral\RoadRunner\Http\PSR7Worker;
use Spiral\RoadRunner\Worker;
use Nyholm\Psr7\Factory\Psr17Factory;
use Nyholm\Psr7\Response;

$worker = Worker::create();
$factory = new Psr17Factory();
$psr7 = new PSR7Worker($worker, $factory, $factory, $factory);

while ($request = $psr7->waitRequest()) {
    try {
        $uri = $request->getUri()->getPath();
        $method = $request->getMethod();
        
        // 路由处理
        $response = match($uri) {
            '/' => new Response(200, [], 'Hello from RoadRunner'),
            '/api/users' => handleUsers($request),
            default => new Response(404, [], 'Not Found')
        };
        
        $psr7->respond($response);
    } catch (Throwable $e) {
        $psr7->getWorker()->error((string) $e);
        $psr7->respond(new Response(500, [], 'Internal Server Error'));
    }
}

function handleUsers($request): Response
{
    $users = [
        ['id' => 1, 'name' => 'Alice'],
        ['id' => 2, 'name' => 'Bob']
    ];
    
    return new Response(
        200,
        ['Content-Type' => 'application/json'],
        json_encode($users)
    );
}
```

## 注意事项

1. **状态管理**：Worker 进程常驻内存，避免在 Worker 中存储请求间共享的状态。

2. **内存泄漏**：设置 `max_jobs` 限制每个 Worker 处理的任务数，防止内存泄漏。

3. **错误处理**：妥善处理异常，避免 Worker 崩溃。

4. **连接复用**：在 Worker 启动时创建数据库连接等资源，在请求间复用。

5. **性能监控**：监控 Worker 的内存使用和请求处理时间。

## 练习

1. 安装和配置 RoadRunner，创建一个简单的 HTTP 服务器。

2. 实现一个基本的 Worker，处理 HTTP 请求。

3. 配置 Worker 数量，观察不同配置对性能的影响。

4. 实现连接复用，在 Worker 中复用数据库连接。

5. 对比 RoadRunner 和 PHP-FPM 的性能差异。
