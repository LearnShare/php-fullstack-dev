# 6.5.1 PHP 性能优化

## 概述

PHP 性能优化是提升应用响应速度和吞吐量的关键。本节介绍性能优化的基本原则、代码优化技巧、内存优化方法，以及性能测试工具。

## 性能优化原则

### 1. 测量优先

在优化之前，先测量性能瓶颈：

```php
<?php
declare(strict_types=1);

// 使用 microtime 测量执行时间
$start = microtime(true);

// 执行代码
for ($i = 0; $i < 1000000; $i++) {
    // ...
}

$end = microtime(true);
$executionTime = $end - $start;
echo "Execution time: {$executionTime} seconds\n";
```

### 2. 优化热点代码

使用 Xdebug 或 Xhprof 找出性能瓶颈：

```php
<?php
declare(strict_types=1);

// 使用 Xhprof
xhprof_enable(XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY);

// 执行代码
// ...

$data = xhprof_disable();
include_once '/path/to/xhprof_lib/utils/xhprof_lib.php';
include_once '/path/to/xhprof_lib/utils/xhprof_runs.php';

$xhprof_runs = new XHProfRuns_Default();
$run_id = $xhprof_runs->save_run($data, "xhprof_test");
```

### 3. 避免过早优化

只优化真正影响性能的代码。

## 代码优化

### 1. 使用合适的函数

```php
<?php
declare(strict_types=1);

// 不推荐：使用 count() 在循环中
for ($i = 0; $i < count($array); $i++) {
    // ...
}

// 推荐：预先计算
$count = count($array);
for ($i = 0; $i < $count; $i++) {
    // ...
}

// 更推荐：使用 foreach
foreach ($array as $item) {
    // ...
}
```

### 2. 避免不必要的函数调用

```php
<?php
declare(strict_types=1);

// 不推荐：重复调用函数
if (strlen($string) > 10 && strlen($string) < 100) {
    // ...
}

// 推荐：缓存结果
$length = strlen($string);
if ($length > 10 && $length < 100) {
    // ...
}
```

### 3. 使用类型提示

```php
<?php
declare(strict_types=1);

// 类型提示帮助 PHP 优化
function processUser(User $user): void
{
    // PHP 不需要类型检查
}
```

### 4. 避免使用魔术方法

```php
<?php
declare(strict_types=1);

// 不推荐：使用 __get
class User
{
    private array $data = [];
    
    public function __get(string $name): mixed
    {
        return $this->data[$name] ?? null;
    }
}

// 推荐：直接访问属性
class User
{
    public function __construct(
        public string $name,
        public string $email,
    ) {}
}
```

### 5. 使用生成器处理大数组

```php
<?php
declare(strict_types=1);

// 不推荐：加载所有数据到内存
function getAllUsers(): array
{
    $users = [];
    $stmt = $pdo->query("SELECT * FROM users");
    while ($row = $stmt->fetch()) {
        $users[] = $row;
    }
    return $users;
}

// 推荐：使用生成器
function getAllUsers(): Generator
{
    $stmt = $pdo->query("SELECT * FROM users");
    while ($row = $stmt->fetch()) {
        yield $row;
    }
}

// 使用
foreach (getAllUsers() as $user) {
    // 处理单个用户，内存占用小
}
```

## 内存优化

### 1. 及时释放大变量

```php
<?php
declare(strict_types=1);

// 处理大文件
$largeData = file_get_contents('large_file.txt');
processData($largeData);
unset($largeData); // 及时释放内存
```

### 2. 使用流处理大文件

```php
<?php
declare(strict_types=1);

// 不推荐：一次性读取
$content = file_get_contents('large_file.txt');

// 推荐：流式处理
$handle = fopen('large_file.txt', 'r');
while (($line = fgets($handle)) !== false) {
    processLine($line);
}
fclose($handle);
```

### 3. 避免循环引用

```php
<?php
declare(strict_types=1);

// 避免循环引用
class Node
{
    public ?Node $parent = null;
    public array $children = [];
    
    public function addChild(Node $child): void
    {
        $this->children[] = $child;
        $child->parent = $this;
    }
    
    public function __destruct()
    {
        // 清理引用
        $this->parent = null;
        $this->children = [];
    }
}
```

## 服务配置优化

### PHP-FPM 优化

#### 进程池配置

根据服务器资源和应用负载选择合适的进程管理模式：

```ini
; php-fpm.conf
[www]
; 进程管理模式：static（静态）、dynamic（动态）、ondemand（按需）
pm = dynamic

; 最大子进程数（根据内存计算：可用内存 / 单个进程内存占用）
pm.max_children = 50

; 启动时的子进程数
pm.start_servers = 10

; 空闲时最小子进程数
pm.min_spare_servers = 5

; 空闲时最大子进程数
pm.max_spare_servers = 20

; 每个子进程处理请求数后重启（避免内存泄漏）
pm.max_requests = 1000

; 进程空闲超时时间（仅 ondemand 模式）
pm.process_idle_timeout = 10s
```

#### 进程池配置计算

```php
<?php
declare(strict_types=1);

/**
 * 计算 PHP-FPM 进程池配置
 * 
 * @param int $availableMemory 可用内存（MB）
 * @param int $processMemory 单个进程内存占用（MB）
 * @param float $cpuCores CPU 核心数
 * @return array 推荐的配置参数
 */
function calculateFpmConfig(int $availableMemory, int $processMemory, float $cpuCores): array
{
    // 最大子进程数 = 可用内存 / 单个进程内存占用 * 0.8（保留 20% 缓冲）
    $maxChildren = (int)($availableMemory / $processMemory * 0.8);
    
    // 启动进程数 = CPU 核心数 * 2
    $startServers = (int)($cpuCores * 2);
    
    // 最小空闲进程 = CPU 核心数
    $minSpareServers = (int)$cpuCores;
    
    // 最大空闲进程 = CPU 核心数 * 2
    $maxSpareServers = (int)($cpuCores * 2);
    
    return [
        'pm.max_children' => $maxChildren,
        'pm.start_servers' => $startServers,
        'pm.min_spare_servers' => $minSpareServers,
        'pm.max_spare_servers' => $maxSpareServers,
    ];
}

// 使用示例：8GB 内存，每个进程 50MB，4 核 CPU
$config = calculateFpmConfig(8192, 50, 4);
print_r($config);
```

#### 慢日志配置

```ini
; php-fpm.conf
; 慢日志文件路径
slowlog = /var/log/php-fpm/slow.log

; 慢请求超时时间（超过此时间的请求会记录到慢日志）
request_slowlog_timeout = 5s

; 请求终止超时时间
request_terminate_timeout = 30s
```

#### PHP-FPM 状态监控

```ini
; php-fpm.conf
; 启用状态页面
pm.status_path = /status

; 启用 ping 页面（用于健康检查）
ping.path = /ping
```

访问状态页面查看进程池状态：

```bash
curl http://localhost/status?full
```

### Nginx 配置优化

#### 连接数优化

```nginx
# nginx.conf
http {
    # 工作进程数（通常等于 CPU 核心数）
    worker_processes auto;
    
    # 每个工作进程的最大连接数
    events {
        worker_connections 1024;
        use epoll;  # Linux 使用 epoll
    }
    
    # 连接超时设置
    keepalive_timeout 65;
    keepalive_requests 100;
    
    # 客户端请求体大小限制
    client_max_body_size 10m;
    
    # 缓冲区大小
    client_body_buffer_size 128k;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 4k;
}
```

#### 静态文件缓存

```nginx
# nginx.conf
server {
    # 静态文件缓存配置
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
        access_log off;
    }
    
    # 静态文件直接服务，不经过 PHP
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        root /var/www/html/public;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

#### Gzip 压缩

```nginx
# nginx.conf
http {
    # 启用 Gzip 压缩
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml text/javascript 
               application/json application/javascript application/xml+rss 
               application/rss+xml font/truetype font/opentype 
               application/vnd.ms-fontobject image/svg+xml;
    gzip_min_length 1000;
    gzip_disable "msie6";
}
```

#### FastCGI 缓存

```nginx
# nginx.conf
http {
    # FastCGI 缓存配置
    fastcgi_cache_path /var/cache/nginx levels=1:2 keys_zone=php_cache:10m 
                      max_size=100m inactive=60m;
    fastcgi_cache_key "$scheme$request_method$host$request_uri";
    
    server {
        location ~ \.php$ {
            fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
            
            # 启用缓存
            fastcgi_cache php_cache;
            fastcgi_cache_valid 200 60m;
            fastcgi_cache_valid 404 1m;
            fastcgi_cache_bypass $http_pragma $http_authorization;
            fastcgi_no_cache $http_pragma $http_authorization;
            add_header X-Cache-Status $upstream_cache_status;
        }
    }
}
```

### 系统参数优化

#### Linux 系统参数

```bash
# /etc/sysctl.conf

# TCP 连接优化
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 300

# 文件描述符限制
fs.file-max = 65535

# 内存优化
vm.swappiness = 10
vm.overcommit_memory = 1
```

应用配置：

```bash
sudo sysctl -p
```

#### 文件描述符限制

```bash
# /etc/security/limits.conf
* soft nofile 65535
* hard nofile 65535
```

### PHP 配置优化

#### php.ini 基础优化

```ini
; php.ini

; 内存限制
memory_limit = 256M

; 最大执行时间
max_execution_time = 30
max_input_time = 60

; 文件上传
upload_max_filesize = 10M
post_max_size = 10M

; 错误报告（生产环境）
display_errors = Off
log_errors = On
error_log = /var/log/php/error.log

; 时区
date.timezone = Asia/Shanghai

; 禁用危险函数（生产环境）
disable_functions = exec,passthru,shell_exec,system,proc_open,popen
```

#### JIT 编译配置

PHP 8.0+ 支持 JIT（Just-In-Time）编译，可以将热点代码编译为机器码：

```ini
; php.ini

; 启用 OPcache（JIT 需要 OPcache）
opcache.enable=1

; JIT 配置
opcache.jit_buffer_size=256M
opcache.jit=tracing
```

JIT 模式说明：

- `tracing`：追踪模式，适合长时间运行的脚本（CLI、Worker 模式）
- `function`：函数模式，适合 Web 应用
- `1255`：完整优化（推荐用于生产环境）

```php
<?php
declare(strict_types=1);

/**
 * 检查 JIT 是否启用
 */
function checkJitStatus(): array
{
    if (!function_exists('opcache_get_status')) {
        return ['enabled' => false, 'message' => 'OPcache not available'];
    }
    
    $status = opcache_get_status();
    $config = opcache_get_configuration();
    
    return [
        'enabled' => $status['opcache_enabled'] ?? false,
        'jit_enabled' => $config['directives']['opcache.jit_buffer_size'] > 0,
        'jit_buffer_size' => $config['directives']['opcache.jit_buffer_size'] ?? 0,
        'jit_mode' => $config['directives']['opcache.jit'] ?? 'disabled',
    ];
}

$jitStatus = checkJitStatus();
print_r($jitStatus);
```

### 现代运行时选择

#### PHP-FPM（传统模式）

- **适用场景**：传统 Web 应用、共享主机
- **特点**：稳定、成熟、资源隔离好
- **性能**：中等，适合大多数场景

#### FrankenPHP

- **适用场景**：现代 PHP 应用、需要 HTTP/2 Server Push
- **特点**：内置 Web 服务器、支持协程、自动 HTTPS
- **性能**：高，接近 Go 性能

#### OpenSwoole

- **适用场景**：高并发应用、实时通信、长连接
- **特点**：协程、异步 I/O、WebSocket 支持
- **性能**：极高，适合高并发场景

#### RoadRunner

- **适用场景**：微服务、容器化部署
- **特点**：Go 编写、高性能、支持多种协议
- **性能**：高，资源占用低

运行时选择建议：

```php
<?php
declare(strict_types=1);

/**
 * 运行时选择建议
 */
function recommendRuntime(array $requirements): string
{
    $concurrency = $requirements['concurrency'] ?? 0;
    $longConnection = $requirements['long_connection'] ?? false;
    $websocket = $requirements['websocket'] ?? false;
    $container = $requirements['container'] ?? false;
    
    if ($websocket || $longConnection) {
        return 'OpenSwoole';
    }
    
    if ($container || $requirements['microservice'] ?? false) {
        return 'RoadRunner';
    }
    
    if ($concurrency > 1000) {
        return 'FrankenPHP or OpenSwoole';
    }
    
    return 'PHP-FPM';
}

// 使用示例
$recommendation = recommendRuntime([
    'concurrency' => 500,
    'long_connection' => false,
    'websocket' => false,
    'container' => true,
]);
echo "推荐运行时: {$recommendation}\n";
```

## 性能测试

性能测试是验证优化效果和发现性能瓶颈的重要手段。本节介绍基准测试、压力测试和负载测试的方法和工具。

### 基准测试（Benchmark）

基准测试用于测量单个操作的执行时间，比较不同实现方式的性能差异。

#### 基准测试类

```php
<?php
declare(strict_types=1);

class Benchmark
{
    private array $results = [];
    private int $iterations;
    
    public function __construct(int $iterations = 1000)
    {
        $this->iterations = $iterations;
    }
    
    /**
     * 测量单个操作的执行时间
     * 
     * @param string $name 测试名称
     * @param callable $callback 要测试的代码
     * @param int $iterations 迭代次数
     * @return float 平均执行时间（秒）
     */
    public function measure(string $name, callable $callback, ?int $iterations = null): float
    {
        $iterations = $iterations ?? $this->iterations;
        $start = microtime(true);
        $startMemory = memory_get_usage(true);
        
        // 预热
        $callback();
        
        // 正式测试
        $start = microtime(true);
        for ($i = 0; $i < $iterations; $i++) {
            $callback();
        }
        $end = microtime(true);
        $endMemory = memory_get_usage(true);
        
        $totalTime = $end - $start;
        $avgTime = $totalTime / $iterations;
        $memory = $endMemory - $startMemory;
        
        $this->results[$name] = [
            'iterations' => $iterations,
            'total_time' => $totalTime,
            'avg_time' => $avgTime,
            'memory' => $memory,
            'ops_per_sec' => $iterations / $totalTime,
        ];
        
        return $avgTime;
    }
    
    /**
     * 比较多个实现方式
     */
    public function compare(array $tests): void
    {
        foreach ($tests as $name => $callback) {
            $this->measure($name, $callback);
        }
        
        $this->report();
    }
    
    /**
     * 生成测试报告
     */
    public function report(): void
    {
        echo "Benchmark Results:\n";
        echo str_repeat("=", 80) . "\n";
        
        // 按平均执行时间排序
        uasort($this->results, fn($a, $b) => $a['avg_time'] <=> $b['avg_time']);
        
        $fastest = reset($this->results);
        $fastestTime = $fastest['avg_time'];
        
        foreach ($this->results as $name => $result) {
            $speedup = $fastestTime > 0 ? $result['avg_time'] / $fastestTime : 1;
            
            printf(
                "%-30s | Iterations: %6d | Avg: %10.6fs | Ops/sec: %10.2f | Memory: %10s | Speedup: %.2fx\n",
                $name,
                $result['iterations'],
                $result['avg_time'],
                $result['ops_per_sec'],
                $this->formatBytes($result['memory']),
                $speedup
            );
        }
        
        echo str_repeat("=", 80) . "\n";
    }
    
    /**
     * 获取结果
     */
    public function getResults(): array
    {
        return $this->results;
    }
    
    private function formatBytes(int $bytes): string
    {
        $units = ['B', 'KB', 'MB', 'GB'];
        $i = 0;
        while ($bytes >= 1024 && $i < count($units) - 1) {
            $bytes /= 1024;
            $i++;
        }
        return round($bytes, 2) . ' ' . $units[$i];
    }
}

// 使用示例
$benchmark = new Benchmark(10000);

$benchmark->compare([
    'Array iteration' => function() {
        $array = range(1, 1000);
        foreach ($array as $value) {
            $value * 2;
        }
    },
    'Generator iteration' => function() {
        $generator = function() {
            for ($i = 1; $i <= 1000; $i++) {
                yield $i;
            }
        };
        foreach ($generator() as $value) {
            $value * 2;
        }
    },
    'For loop' => function() {
        for ($i = 1; $i <= 1000; $i++) {
            $i * 2;
        }
    },
]);
```

### 压力测试（Stress Test）

压力测试用于模拟高并发请求，测试系统在极限负载下的表现。

#### Apache Bench (ab) 使用

```bash
# 基本用法
ab -n 10000 -c 100 http://localhost/api/users

# 参数说明：
# -n: 总请求数
# -c: 并发数
# -t: 测试时间（秒）
# -p: POST 数据文件
# -T: Content-Type
# -H: 自定义请求头

# POST 请求示例
ab -n 1000 -c 50 -p post_data.json -T application/json http://localhost/api/users
```

#### 压力测试脚本

```php
<?php
declare(strict_types=1);

/**
 * 压力测试工具
 */
class StressTest
{
    private string $url;
    private int $concurrency;
    private int $totalRequests;
    private array $results = [];
    
    public function __construct(string $url, int $concurrency = 10, int $totalRequests = 1000)
    {
        $this->url = $url;
        $this->concurrency = $concurrency;
        $this->totalRequests = $totalRequests;
    }
    
    /**
     * 执行压力测试
     */
    public function run(): array
    {
        $requestsPerWorker = (int)ceil($this->totalRequests / $this->concurrency);
        $workers = [];
        
        // 创建多个进程/线程进行并发测试
        for ($i = 0; $i < $this->concurrency; $i++) {
            $pid = pcntl_fork();
            
            if ($pid == -1) {
                die("Could not fork\n");
            } elseif ($pid) {
                // 父进程
                $workers[] = $pid;
            } else {
                // 子进程：执行请求
                $this->worker($requestsPerWorker);
                exit(0);
            }
        }
        
        // 等待所有子进程完成
        foreach ($workers as $pid) {
            pcntl_waitpid($pid, $status);
        }
        
        return $this->analyze();
    }
    
    /**
     * 工作进程：执行请求
     */
    private function worker(int $requests): void
    {
        $ch = curl_init();
        curl_setopt_array($ch, [
            CURLOPT_URL => $this->url,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_TIMEOUT => 30,
            CURLOPT_CONNECTTIMEOUT => 5,
        ]);
        
        $startTime = microtime(true);
        
        for ($i = 0; $i < $requests; $i++) {
            $requestStart = microtime(true);
            curl_exec($ch);
            $requestEnd = microtime(true);
            
            $this->results[] = [
                'time' => $requestEnd - $requestStart,
                'timestamp' => $requestStart,
            ];
        }
        
        curl_close($ch);
    }
    
    /**
     * 分析测试结果
     */
    private function analyze(): array
    {
        $times = array_column($this->results, 'time');
        sort($times);
        
        $count = count($times);
        $totalTime = array_sum($times);
        
        return [
            'total_requests' => $count,
            'total_time' => $totalTime,
            'requests_per_sec' => $count / $totalTime,
            'avg_time' => $totalTime / $count,
            'min_time' => min($times),
            'max_time' => max($times),
            'p50' => $times[(int)($count * 0.5)],
            'p95' => $times[(int)($count * 0.95)],
            'p99' => $times[(int)($count * 0.99)],
        ];
    }
}

// 使用示例（需要 pcntl 扩展）
if (function_exists('pcntl_fork')) {
    $test = new StressTest('http://localhost/api/users', 50, 10000);
    $results = $test->run();
    print_r($results);
} else {
    echo "pcntl extension is required for stress testing\n";
}
```

#### 使用 wrk 进行压力测试

```bash
# 安装 wrk（Linux/macOS）
# macOS: brew install wrk
# Linux: 从源码编译或使用包管理器

# 基本用法
wrk -t12 -c400 -d30s http://localhost/api/users

# 参数说明：
# -t: 线程数
# -c: 连接数
# -d: 测试持续时间
# -s: Lua 脚本（用于自定义请求）

# 使用 Lua 脚本进行 POST 请求
wrk -t12 -c400 -d30s -s post.lua http://localhost/api/users
```

`post.lua` 示例：

```lua
wrk.method = "POST"
wrk.body   = '{"name":"test","email":"test@example.com"}'
wrk.headers["Content-Type"] = "application/json"
```

### 负载测试（Load Test）

负载测试用于模拟正常到高峰的负载变化，测试系统在不同负载下的性能表现。

#### 负载测试工具类

```php
<?php
declare(strict_types=1);

/**
 * 负载测试工具
 */
class LoadTest
{
    private string $url;
    private array $loadProfile;
    private array $results = [];
    
    /**
     * @param string $url 测试 URL
     * @param array $loadProfile 负载配置 [['duration' => 60, 'rps' => 10], ...]
     */
    public function __construct(string $url, array $loadProfile)
    {
        $this->url = $url;
        $this->loadProfile = $loadProfile;
    }
    
    /**
     * 执行负载测试
     */
    public function run(): array
    {
        foreach ($this->loadProfile as $stage) {
            $duration = $stage['duration'];
            $targetRps = $stage['rps'];
            
            echo "Stage: {$targetRps} RPS for {$duration} seconds\n";
            
            $this->runStage($duration, $targetRps);
            
            // 阶段间休息
            sleep(5);
        }
        
        return $this->analyze();
    }
    
    /**
     * 执行单个负载阶段
     */
    private function runStage(int $duration, int $targetRps): void
    {
        $interval = 1.0 / $targetRps; // 请求间隔（秒）
        $endTime = microtime(true) + $duration;
        $requestCount = 0;
        
        while (microtime(true) < $endTime) {
            $start = microtime(true);
            
            $ch = curl_init();
            curl_setopt_array($ch, [
                CURLOPT_URL => $this->url,
                CURLOPT_RETURNTRANSFER => true,
                CURLOPT_TIMEOUT => 30,
            ]);
            
            curl_exec($ch);
            $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
            curl_close($ch);
            
            $end = microtime(true);
            
            $this->results[] = [
                'time' => $end - $start,
                'http_code' => $httpCode,
                'timestamp' => $start,
            ];
            
            $requestCount++;
            
            // 控制请求速率
            $sleepTime = $interval - (microtime(true) - $start);
            if ($sleepTime > 0) {
                usleep((int)($sleepTime * 1000000));
            }
        }
        
        $actualRps = $requestCount / $duration;
        echo "Actual RPS: {$actualRps}\n";
    }
    
    /**
     * 分析测试结果
     */
    private function analyze(): array
    {
        $times = array_column($this->results, 'time');
        $httpCodes = array_count_values(array_column($this->results, 'http_code'));
        
        sort($times);
        $count = count($times);
        
        return [
            'total_requests' => $count,
            'http_codes' => $httpCodes,
            'avg_time' => array_sum($times) / $count,
            'min_time' => min($times),
            'max_time' => max($times),
            'p50' => $times[(int)($count * 0.5)],
            'p95' => $times[(int)($count * 0.95)],
            'p99' => $times[(int)($count * 0.99)],
            'success_rate' => ($httpCodes[200] ?? 0) / $count * 100,
        ];
    }
}

// 使用示例：模拟负载逐渐增加
$loadProfile = [
    ['duration' => 60, 'rps' => 10],   // 正常负载
    ['duration' => 60, 'rps' => 50],   // 中等负载
    ['duration' => 60, 'rps' => 100],  // 高负载
    ['duration' => 60, 'rps' => 200],  // 峰值负载
];

$test = new LoadTest('http://localhost/api/users', $loadProfile);
$results = $test->run();
print_r($results);
```

### 性能测试最佳实践

1. **测试环境**：使用与生产环境相似的配置
2. **基准建立**：先建立性能基准，再进行优化
3. **逐步增加**：从低负载开始，逐步增加负载
4. **监控资源**：监控 CPU、内存、网络、磁盘 I/O
5. **记录结果**：保存测试结果，便于对比分析
6. **多次测试**：进行多次测试，取平均值
7. **真实场景**：模拟真实的用户行为模式

## 完整示例

```php
<?php
declare(strict_types=1);

// 性能优化示例：用户列表处理
class UserService
{
    private PDO $db;
    
    public function __construct(PDO $db)
    {
        $this->db = $db;
    }
    
    // 优化前：加载所有用户到内存
    public function getAllUsersOld(): array
    {
        $stmt = $this->db->query("SELECT * FROM users");
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
    
    // 优化后：使用生成器
    public function getAllUsers(): Generator
    {
        $stmt = $this->db->query("SELECT * FROM users");
        while ($user = $stmt->fetch(PDO::FETCH_ASSOC)) {
            yield $user;
        }
    }
    
    // 优化：批量处理
    public function processUsersBatch(int $batchSize = 1000): void
    {
        $offset = 0;
        
        while (true) {
            $stmt = $this->db->prepare("SELECT * FROM users LIMIT ? OFFSET ?");
            $stmt->execute([$batchSize, $offset]);
            $users = $stmt->fetchAll(PDO::FETCH_ASSOC);
            
            if (empty($users)) {
                break;
            }
            
            foreach ($users as $user) {
                $this->processUser($user);
            }
            
            $offset += $batchSize;
            
            // 释放内存
            unset($users);
        }
    }
    
    private function processUser(array $user): void
    {
        // 处理用户
    }
}
```

## 最佳实践

1. **测量优先**：使用工具找出性能瓶颈
2. **优化热点**：只优化真正影响性能的代码
3. **使用生成器**：处理大数据集时使用生成器
4. **及时释放**：及时释放大变量
5. **配置优化**：优化 PHP-FPM 配置

## 注意事项

1. 避免过早优化
2. 保持代码可读性
3. 定期进行性能测试
4. 监控生产环境性能

## 练习

1. 创建一个性能测试工具，比较不同实现方式的性能。

2. 优化一个处理大文件的函数，使用流式处理减少内存占用。

3. 使用生成器重构一个返回大量数据的函数。

4. 配置 PHP-FPM，优化进程池设置。
