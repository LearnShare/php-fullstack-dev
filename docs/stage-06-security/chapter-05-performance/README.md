# 6.5 PHP 性能优化

## 目标

- 理解 OPcache 的作用与配置。
- 掌握 PHP-FPM Worker 优化策略。
- 了解 JIT（Just-In-Time）编译的使用场景。
- 熟悉 FrankenPHP、OpenSwoole 等运行时的性能特点。

## OPcache

### 什么是 OPcache

- **OPcache**：操作码缓存，缓存编译后的 PHP 代码。
- 避免重复编译，提升执行速度。

### OPcache 配置

```ini
; php.ini
[opcache]
opcache.enable=1
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.revalidate_freq=2
opcache.fast_shutdown=1
opcache.enable_cli=0
```

### 配置说明

| 配置项                      | 说明                           | 推荐值        |
| :-------------------------- | :----------------------------- | :------------ |
| `opcache.enable`            | 启用 OPcache                   | `1`           |
| `opcache.memory_consumption` | 内存大小（MB）                 | `128` 或 `256` |
| `opcache.max_accelerated_files` | 最大缓存文件数             | `10000`       |
| `opcache.revalidate_freq`   | 检查文件更新频率（秒）         | `2`           |
| `opcache.fast_shutdown`     | 快速关闭                       | `1`           |

### 检查 OPcache 状态

```php
<?php
if (function_exists('opcache_get_status')) {
    $status = opcache_get_status();
    print_r($status);
}
```

## JIT（Just-In-Time）

### JIT 配置

```ini
; php.ini
opcache.enable=1
opcache.jit_buffer_size=256M
opcache.jit=tracing
```

### JIT 模式

- `tracing`：跟踪模式，适合长时间运行的脚本。
- `function`：函数模式，适合函数调用频繁的场景。
- `disable`：禁用 JIT。

### JIT 使用场景

```php
<?php
// JIT 适合的场景：
// 1. 数学计算密集型
function calculate(int $n): int
{
    $sum = 0;
    for ($i = 0; $i < $n; $i++) {
        $sum += $i * $i;
    }
    return $sum;
}

// 2. 循环密集型
function processArray(array $data): array
{
    $result = [];
    foreach ($data as $item) {
        $result[] = $item * 2;
    }
    return $result;
}
```

## PHP-FPM 优化

### Worker 配置

```ini
; php-fpm.conf
[www]
pm = dynamic
pm.max_children = 50
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 20
pm.max_requests = 500
```

### 配置说明

| 配置项              | 说明                           | 推荐值                    |
| :------------------ | :----------------------------- | :------------------------ |
| `pm.max_children`   | 最大 Worker 进程数            | CPU 核心数 × 2 到 4      |
| `pm.start_servers`  | 启动时的进程数                 | `pm.min_spare_servers`   |
| `pm.min_spare_servers` | 最小空闲进程数             | CPU 核心数                |
| `pm.max_spare_servers` | 最大空闲进程数             | `pm.max_children` × 0.4  |
| `pm.max_requests`   | 每个进程处理的最大请求数       | `500` 到 `1000`          |

### 性能监控

```php
<?php
// 获取 FPM 状态
$status = file_get_contents('http://localhost/status?json');
$data = json_decode($status, true);

echo "Active processes: {$data['active-processes']}\n";
echo "Idle processes: {$data['idle-processes']}\n";
echo "Max active processes: {$data['max-active-processes']}\n";
```

## 运行时对比

### 传统 PHP-FPM

- **特点**：每个请求启动新进程。
- **优势**：简单、稳定、隔离性好。
- **劣势**：进程创建开销，内存占用高。

### FrankenPHP

- **特点**：内置 HTTP 服务器，Worker 持久化。
- **优势**：减少进程创建，提升性能。
- **适用**：API 服务、微服务。

### OpenSwoole

- **特点**：协程 Event Loop，异步 I/O。
- **优势**：高并发、低延迟。
- **适用**：实时应用、WebSocket。

## 性能优化技巧

### 1. 减少文件包含

```php
// 不推荐：多次包含
require 'config.php';
require 'functions.php';
require 'database.php';

// 推荐：使用 Composer autoload
require __DIR__ . '/vendor/autoload.php';
```

### 2. 使用连接池

```php
<?php
// 数据库连接池
class ConnectionPool
{
    private array $connections = [];
    private int $maxConnections = 10;
    
    public function getConnection(): PDO
    {
        if (empty($this->connections)) {
            return $this->createConnection();
        }
        
        return array_pop($this->connections);
    }
    
    public function releaseConnection(PDO $connection): void
    {
        if (count($this->connections) < $this->maxConnections) {
            $this->connections[] = $connection;
        }
    }
}
```

### 3. 缓存计算结果

```php
<?php
declare(strict_types=1);

function memoize(callable $fn): callable
{
    static $cache = [];
    
    return function (...$args) use ($fn, &$cache) {
        $key = serialize($args);
        
        if (!isset($cache[$key])) {
            $cache[$key] = $fn(...$args);
        }
        
        return $cache[$key];
    };
}

// 使用
$expensiveFunction = memoize(function ($n) {
    // 复杂计算
    return $n * $n;
});
```

## 性能分析工具

### Xdebug Profiler

#### 启用性能分析

```ini
; xdebug.ini
xdebug.mode=profile
xdebug.output_dir=/tmp/xdebug
xdebug.profiler_output_name=cachegrind.out.%p
```

#### 分析性能数据

```bash
# 生成分析文件后，使用工具分析
# 1. KCacheGrind (Linux)
sudo apt install kcachegrind
kcachegrind /tmp/xdebug/cachegrind.out.*

# 2. WinCacheGrind (Windows)
# 下载：https://sourceforge.net/projects/wincachegrind/

# 3. WebGrind (Web 界面)
composer require jokkedk/webgrind
# 访问 http://localhost/webgrind
```

#### 分析结果解读

- **Self Time**：函数自身执行时间
- **Cumulative Time**：函数及其调用子函数的总时间
- **Call Count**：函数调用次数
- **Memory Usage**：内存使用量

#### 实际应用

```php
<?php
declare(strict_types=1);

// 在需要分析的代码前后添加标记
xdebug_start_trace('/tmp/trace');

// 要分析的代码
processLargeDataset();

xdebug_stop_trace();

// 分析 trace 文件
```

### Blackfire

#### 什么是 Blackfire

- **Blackfire**：专业的 PHP 性能分析工具
- **特点**：低开销、生产环境可用、详细的性能报告
- **功能**：CPU 时间、内存使用、I/O 操作、SQL 查询分析

#### 安装和配置

```bash
# 安装 Blackfire Probe
# macOS
brew tap blackfireio/homebrew-blackfire
brew install blackfire-php

# Linux
wget -O - https://packagecloud.io/blackfire/blackfire/gpgkey | sudo apt-key add -
echo "deb https://packagecloud.io/blackfire/blackfire/ any main" | sudo tee /etc/apt/sources.list.d/blackfire.list
sudo apt-get update
sudo apt-get install blackfire-php

# 配置
blackfire config
# 输入 Server ID 和 Server Token
```

#### 使用方式

##### 1. 命令行分析

```bash
# 分析 PHP 脚本
blackfire run php script.php

# 分析 Web 请求
blackfire curl http://localhost/api/users

# 分析特定函数
blackfire run php -r "function test() { /* ... */ } test();"
```

##### 2. 浏览器扩展

```php
<?php
declare(strict_types=1);

// 在代码中触发分析
if (extension_loaded('blackfire')) {
    blackfire_enable();
}

// 要分析的代码
processRequest();

if (extension_loaded('blackfire')) {
    blackfire_disable();
}
```

##### 3. 自动分析

```ini
; php.ini
blackfire.agent_socket = unix:///var/run/blackfire/agent.sock
blackfire.agent_timeout = 0.25
blackfire.server_id = your-server-id
blackfire.server_token = your-server-token
```

#### 分析报告解读

Blackfire 报告包含：

- **Timeline**：请求时间线
- **Call Graph**：函数调用图
- **SQL Queries**：数据库查询分析
- **I/O Operations**：文件 I/O 操作
- **Memory Usage**：内存使用情况

#### 性能优化建议

```php
<?php
declare(strict_types=1);

// Blackfire 会识别以下性能问题：
// 1. N+1 查询问题
// 2. 重复计算
// 3. 内存泄漏
// 4. 慢查询
// 5. 文件 I/O 瓶颈

// 优化前
function getUsersWithOrders(): array
{
    $users = User::all();
    foreach ($users as $user) {
        $user->orders; // N+1 查询
    }
    return $users;
}

// 优化后
function getUsersWithOrders(): array
{
    return User::with('orders')->get(); // 预加载
}
```

### 性能分析最佳实践

#### 1. 建立性能基准

```php
<?php
declare(strict_types=1);

class PerformanceBenchmark
{
    private array $metrics = [];
    
    public function start(string $label): void
    {
        $this->metrics[$label] = [
            'start_time' => microtime(true),
            'start_memory' => memory_get_usage(),
        ];
    }
    
    public function end(string $label): array
    {
        if (!isset($this->metrics[$label])) {
            throw new InvalidArgumentException("Benchmark {$label} not started");
        }
        
        $metric = $this->metrics[$label];
        return [
            'time' => microtime(true) - $metric['start_time'],
            'memory' => memory_get_usage() - $metric['start_memory'],
            'peak_memory' => memory_get_peak_usage(),
        ];
    }
    
    public function report(): void
    {
        foreach ($this->metrics as $label => $metric) {
            $result = $this->end($label);
            echo sprintf(
                "%s: %.4f seconds, %d bytes, peak: %d bytes\n",
                $label,
                $result['time'],
                $result['memory'],
                $result['peak_memory']
            );
        }
    }
}

// 使用
$benchmark = new PerformanceBenchmark();
$benchmark->start('database_query');
// ... 执行查询
$result = $benchmark->end('database_query');
```

#### 2. 定期性能分析

```bash
# 在 CI/CD 中集成性能测试
# .github/workflows/performance.yml
name: Performance Test

on: [push]

jobs:
  performance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
      - run: composer install
      - name: Run Blackfire
        run: blackfire run phpunit
```

#### 3. 生产环境监控

```php
<?php
declare(strict_types=1);

// 在生产环境中记录慢请求
class SlowRequestLogger
{
    private float $threshold = 1.0; // 1 秒
    
    public function logSlowRequest(): void
    {
        $executionTime = microtime(true) - $_SERVER['REQUEST_TIME_FLOAT'];
        
        if ($executionTime > $this->threshold) {
            error_log(sprintf(
                'Slow request: %s took %.2f seconds',
                $_SERVER['REQUEST_URI'],
                $executionTime
            ));
        }
    }
}
```

## 练习

1. 配置 OPcache，优化 PHP 执行性能。

2. 调整 PHP-FPM Worker 配置，根据服务器资源优化进程数。

3. 实现一个性能监控工具，收集和展示 PHP 性能指标。

4. 创建一个缓存层，减少重复计算和数据库查询。

5. 编写性能测试脚本，比较优化前后的性能差异。

6. 设计一个性能分析工具，识别性能瓶颈。

7. **使用 Xdebug Profiler 分析一个复杂函数，找出性能瓶颈。**

8. **使用 Blackfire 分析一个完整的 Web 请求，优化慢查询和重复计算。**

9. **建立性能基准测试，监控应用性能变化。**
