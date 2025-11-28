# 1.5 PHP 执行模式与 Web 架构

## 目标

- 理解 PHP 的不同执行模式（CLI、FPM、内置服务器）。
- 深入理解 Nginx + PHP-FPM 的工作原理。
- 掌握 FastCGI 协议和 FPM Pool 配置。
- 理解 Shared Nothing 架构的设计理念。

## PHP 执行模式

> **提示**：如果你熟悉 Node.js，理解 PHP 执行模式的关键是：PHP 是**请求-响应**模式，而 Node.js 是**事件循环**模式。

### 与 Node.js 对比

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

### CLI 模式（命令行）

- **用途**：执行脚本、运行迁移、处理队列任务。
- **特点**：单次执行，脚本结束后进程退出。
- **类比**：类似 Node.js 的 CLI 脚本（`node script.js`）

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

### 内置 Web 服务器

- **用途**：开发测试、快速演示。
- **特点**：单线程，不适合生产环境。

```bash
# 启动内置服务器
php -S localhost:8000 -t public

# 指定路由脚本
php -S localhost:8000 router.php
```

```php
<?php
// router.php
declare(strict_types=1);

$path = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);

if (file_exists(__DIR__ . $path)) {
    return false; // 直接返回文件
}

// 路由到 index.php
require __DIR__ . '/index.php';
```

### PHP-FPM 模式（生产环境）

- **用途**：与 Nginx/Apache 配合处理 Web 请求。
- **特点**：进程池管理，高性能，适合生产环境。

## Nginx + PHP-FPM 工作原理

### 与 Node.js/Express 对比

**Node.js/Express 请求处理**：
```
1. 客户端请求
   ↓
2. Node.js 事件循环接收请求
   ↓
3. Express 中间件链处理
   ↓
4. 路由处理器执行
   ↓
5. 返回响应（异步）
```

**PHP-FPM 请求处理**：
```
1. 客户端请求
   ↓
2. Nginx 接收请求（类似 Node.js HTTP 服务器）
   ↓
3. Nginx 转发给 PHP-FPM（类似 Express 路由）
   ↓
4. PHP-FPM Worker 进程处理（类似 Node.js 请求处理器）
   ↓
5. 返回响应（同步）
```

**关键差异**：
- **Node.js**：单进程处理所有请求，通过事件循环实现并发
- **PHP-FPM**：多进程处理请求，每个进程处理一个请求

### 请求处理流程

```
1. 客户端请求
   ↓
2. Nginx 接收请求
   ↓
3. Nginx 根据配置决定处理方式
   ↓
4. 对于 PHP 文件，通过 FastCGI 协议转发给 PHP-FPM
   ↓
5. PHP-FPM Master 进程接收请求
   ↓
6. PHP-FPM Master 分配 Worker 进程处理
   ↓
7. Worker 进程执行 PHP 代码
   ↓
8. Worker 返回结果给 Master
   ↓
9. Master 通过 FastCGI 返回给 Nginx
   ↓
10. Nginx 返回响应给客户端
```

### Shared Nothing 架构

**概念**：每个请求都是独立的，不共享状态。

**与 Node.js 对比**：

| 特性 | Node.js | PHP (FPM) |
| :--- | :------ | :-------- |
| 全局变量 | 所有请求共享 | 每个请求独立 |
| 内存状态 | 请求间共享 | 请求间隔离 |
| 并发模型 | 事件循环（单线程） | 多进程（多线程可选） |
| 状态管理 | 需要小心处理全局状态 | 天然隔离，更安全 |

**示例对比**：

```javascript
// Node.js：全局变量在所有请求间共享
let requestCount = 0;  // 危险：所有请求共享

app.get('/api', (req, res) => {
    requestCount++;  // 所有请求都会修改同一个变量
    res.json({ count: requestCount });
});
```

```php
// PHP-FPM：每个请求有独立的内存空间
// 全局变量只在当前请求中有效
$requestCount = 0;  // 安全：每个请求独立

function handleRequest(): array {
    global $requestCount;  // 只在当前请求中有效
    $requestCount++;
    return ['count' => $requestCount];
}
```

### FastCGI 协议

- **FastCGI**：Fast Common Gateway Interface，CGI 的改进版本。
- **特点**：
  - 持久连接：避免每次请求都创建新进程
  - 多路复用：一个连接可以处理多个请求
  - 二进制协议：比 HTTP 更高效

### Nginx 配置

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

### PHP-FPM 配置

```ini
; /etc/php/8.2/fpm/pool.d/www.conf

[www]
user = www-data
group = www-data

; 进程池管理方式
pm = dynamic

; 最大子进程数
pm.max_children = 50

; 启动时的进程数
pm.start_servers = 10

; 最小空闲进程数
pm.min_spare_servers = 5

; 最大空闲进程数
pm.max_spare_servers = 20

; 每个进程处理的最大请求数
pm.max_requests = 500

; 监听方式
listen = /var/run/php/php8.2-fpm.sock
listen.owner = www-data
listen.group = www-data
listen.mode = 0660
```

### FPM Pool 详解

#### 进程管理方式

| 方式      | 说明                           | 适用场景           |
| :-------- | :----------------------------- | :----------------- |
| `static`  | 固定数量的进程                 | 流量稳定           |
| `dynamic` | 动态调整进程数（推荐）         | 流量波动           |
| `ondemand` | 按需创建进程                 | 低流量场景         |

#### 配置参数说明

```ini
; pm.max_children
; 最大子进程数，根据服务器内存计算
; 公式：max_children = (可用内存 - 系统开销) / 单个进程内存占用

; pm.start_servers
; 启动时的进程数，通常设置为 min_spare_servers 和 max_spare_servers 的平均值

; pm.min_spare_servers
; 最小空闲进程数，确保快速响应

; pm.max_spare_servers
; 最大空闲进程数，避免资源浪费

; pm.max_requests
; 每个进程处理的最大请求数，防止内存泄漏
```

### 性能优化建议

```ini
; 根据服务器配置调整
; 2GB 内存服务器示例
pm = dynamic
pm.max_children = 20
pm.start_servers = 5
pm.min_spare_servers = 3
pm.max_spare_servers = 8
pm.max_requests = 500

; 4GB 内存服务器示例
pm = dynamic
pm.max_children = 40
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 15
pm.max_requests = 500
```

## Shared Nothing 架构

### 核心概念

- **Shared Nothing**：每个请求都是独立的，不共享状态。
- **无状态**：服务器不保存客户端状态信息。
- **水平扩展**：可以轻松添加更多服务器。

### 请求生命周期

```
1. 请求到达
   ↓
2. 创建新的 PHP 进程/线程
   ↓
3. 加载 PHP 代码和配置
   ↓
4. 处理请求
   ↓
5. 生成响应
   ↓
6. 进程/线程结束（或返回进程池）
   ↓
7. 内存释放
```

### 状态管理

#### 不共享的状态

- 全局变量（每个请求独立）
- 静态变量（每个进程独立）
- 对象实例（每个请求独立）

#### 需要共享的状态

- **数据库**：持久化存储
- **缓存（Redis/Memcached）**：共享缓存
- **Session 存储**：文件、数据库或 Redis
- **文件系统**：共享存储

### 高并发处理

```php
<?php
declare(strict_types=1);

// 每个请求都是独立的
// 不依赖全局状态

class RequestHandler
{
    public function handle(): void
    {
        // 从外部存储获取状态
        $sessionId = $_COOKIE['session_id'] ?? null;
        $session = $this->getSession($sessionId); // 从 Redis/数据库获取
        
        // 处理请求
        $response = $this->process($session);
        
        // 保存状态到外部存储
        $this->saveSession($session);
        
        // 返回响应
        echo $response;
    }
    
    private function getSession(?string $sessionId): array
    {
        if ($sessionId === null) {
            return [];
        }
        
        // 从 Redis 获取
        $redis = new Redis();
        $redis->connect('127.0.0.1', 6379);
        $data = $redis->get("session:{$sessionId}");
        
        return $data !== false ? json_decode($data, true) : [];
    }
    
    private function saveSession(array $session): void
    {
        $sessionId = $_COOKIE['session_id'] ?? bin2hex(random_bytes(16));
        
        $redis = new Redis();
        $redis->connect('127.0.0.1', 6379);
        $redis->setex("session:{$sessionId}", 3600, json_encode($session));
        
        setcookie('session_id', $sessionId, time() + 3600);
    }
}
```

### 优势

1. **水平扩展**：可以轻松添加更多服务器
2. **故障隔离**：一个请求的故障不影响其他请求
3. **无状态**：简化部署和扩展
4. **负载均衡**：可以轻松实现负载均衡

### 注意事项

1. **不要使用全局变量存储状态**
   ```php
   // 错误示例
   $globalCounter = 0; // 每个请求都是独立的
   
   // 正确做法
   // 使用数据库或缓存存储计数器
   ```

2. **Session 存储**
   ```php
   // 不要依赖文件 Session（多服务器时会有问题）
   // 使用 Redis 或数据库存储 Session
   ```

3. **文件上传**
   ```php
   // 使用共享存储（如 S3、NFS）存储上传的文件
   ```

## 实际应用示例

### 完整的 Nginx + PHP-FPM 配置

```nginx
# /etc/nginx/sites-available/example.com
server {
    listen 80;
    server_name example.com;
    root /var/www/example.com/public;
    index index.php;

    # 日志
    access_log /var/log/nginx/example.com.access.log;
    error_log /var/log/nginx/example.com.error.log;

    # 静态文件
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # PHP 处理
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        
        # 超时设置
        fastcgi_read_timeout 300;
        fastcgi_send_timeout 300;
    }

    # 禁止访问隐藏文件
    location ~ /\. {
        deny all;
    }
}
```

### 监控和调试

```bash
# 查看 PHP-FPM 状态
sudo systemctl status php8.2-fpm

# 查看进程
ps aux | grep php-fpm

# 查看 Nginx 错误日志
sudo tail -f /var/log/nginx/error.log

# 查看 PHP-FPM 日志
sudo tail -f /var/log/php8.2-fpm.log
```

## 最佳实践

### 1. 合理配置进程数

```ini
; 根据服务器内存和 CPU 核心数配置
; 单个进程内存占用约 20-50MB
; max_children = (总内存 - 系统内存) / 单个进程内存
```

### 2. 使用 Unix Socket

```ini
; 推荐使用 Unix Socket（比 TCP 更快）
listen = /var/run/php/php8.2-fpm.sock

; 如果必须使用 TCP
; listen = 127.0.0.1:9000
```

### 3. 设置合理的超时时间

```ini
; PHP-FPM
request_terminate_timeout = 300

; Nginx
fastcgi_read_timeout = 300;
```

### 4. 启用慢日志

```ini
; 记录执行时间超过 10 秒的请求
slowlog = /var/log/php8.2-fpm-slow.log
request_slowlog_timeout = 10s
```

## 练习

1. 配置一个完整的 Nginx + PHP-FPM 环境，处理 PHP 请求。

2. 调整 PHP-FPM 进程池配置，观察不同配置对性能的影响。

3. 实现一个无状态的 Session 管理系统，使用 Redis 存储。

4. 创建一个负载均衡配置，将请求分发到多个 PHP-FPM 服务器。

5. 监控 PHP-FPM 的性能指标，识别瓶颈。

6. 实现一个健康检查端点，用于监控 PHP-FPM 状态。
