# 1.5.4 配置与优化实践

## 概述

合理的配置和优化可以显著提升 PHP 应用的性能。本章介绍 Nginx 和 PHP-FPM 的优化实践。

## Nginx 优化

### 基本优化

```nginx
# 工作进程数（通常等于 CPU 核心数）
worker_processes auto;

# 每个工作进程的最大连接数
events {
    worker_connections 1024;
    use epoll;  # Linux 使用 epoll
}

# 启用 gzip 压缩
gzip on;
gzip_vary on;
gzip_min_length 1000;
gzip_types text/plain text/css application/json application/javascript;

# 静态文件缓存
location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

### FastCGI 优化

```nginx
location ~ \.php$ {
    fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    
    # 缓存
    fastcgi_cache_path /var/cache/nginx levels=1:2 keys_zone=php_cache:10m max_size=100m inactive=60m;
    fastcgi_cache php_cache;
    fastcgi_cache_valid 200 60m;
    
    # 超时设置
    fastcgi_read_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_connect_timeout 60;
    
    # 缓冲区
    fastcgi_buffer_size 128k;
    fastcgi_buffers 4 256k;
    fastcgi_busy_buffers_size 256k;
}
```

## PHP-FPM 优化

### 进程池优化

```ini
; 根据服务器配置调整
; 单个进程内存占用约 20-50MB

; 2GB 内存服务器
pm = dynamic
pm.max_children = 20
pm.start_servers = 5
pm.min_spare_servers = 3
pm.max_spare_servers = 8
pm.max_requests = 500

; 4GB 内存服务器
pm = dynamic
pm.max_children = 40
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 15
pm.max_requests = 500

; 8GB 内存服务器
pm = dynamic
pm.max_children = 80
pm.start_servers = 20
pm.min_spare_servers = 10
pm.max_spare_servers = 30
pm.max_requests = 500
```

### 性能优化配置

```ini
; 请求超时
request_terminate_timeout = 300

; 慢日志
slowlog = /var/log/php8.2-fpm-slow.log
request_slowlog_timeout = 10s

; 进程管理
pm.process_idle_timeout = 10s
pm.max_requests = 500
```

## OPcache 优化

### 配置

```ini
; 启用 OPcache
opcache.enable=1

; 内存配置
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000

; 验证配置
opcache.validate_timestamps=0  ; 生产环境
opcache.revalidate_freq=0      ; 生产环境

; 其他优化
opcache.fast_shutdown=1
opcache.enable_cli=0
```

### 验证 OPcache

```php
<?php
declare(strict_types=1);

if (function_exists('opcache_get_status')) {
    $status = opcache_get_status();
    echo "OPcache enabled: " . ($status['opcache_enabled'] ? 'Yes' : 'No') . "\n";
    echo "Cache hits: " . $status['opcache_statistics']['hits'] . "\n";
    echo "Cache misses: " . $status['opcache_statistics']['misses'] . "\n";
}
```

## 性能监控

### PHP-FPM 状态监控

```ini
; 启用状态页面
pm.status_path = /status
ping.path = /ping
```

```nginx
location ~ ^/(status|ping)$ {
    access_log off;
    allow 127.0.0.1;
    deny all;
    fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}
```

### 监控脚本

```php
<?php
declare(strict_types=1);

class PerformanceMonitor
{
    public static function getFpmStatus(): array
    {
        $status = file_get_contents('http://127.0.0.1/status?json');
        return json_decode($status, true);
    }
    
    public static function getMetrics(): array
    {
        return [
            'fpm' => self::getFpmStatus(),
            'opcache' => opcache_get_status(),
            'memory' => [
                'usage' => memory_get_usage(true),
                'peak' => memory_get_peak_usage(true),
                'limit' => ini_get('memory_limit')
            ]
        ];
    }
}
```

## 慢日志分析

### 启用慢日志

```ini
; PHP-FPM 配置
slowlog = /var/log/php8.2-fpm-slow.log
request_slowlog_timeout = 10s
```

### 分析慢日志

```bash
# 查看慢日志
tail -f /var/log/php8.2-fpm-slow.log

# 分析慢请求
grep "script_filename" /var/log/php8.2-fpm-slow.log | \
    awk '{print $NF}' | \
    sort | uniq -c | sort -rn | head -10
```

## 完整优化配置

### Nginx 完整配置

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # 日志格式
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log;
    
    # 基本优化
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    
    # Gzip 压缩
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    
    # FastCGI 缓存
    fastcgi_cache_path /var/cache/nginx levels=1:2 keys_zone=php_cache:10m max_size=100m inactive=60m;
    
    server {
        listen 80;
        server_name example.com;
        root /var/www/example.com/public;
        index index.php;
        
        # 静态文件
        location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
            access_log off;
        }
        
        # PHP 处理
        location ~ \.php$ {
            try_files $uri =404;
            fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
            
            # 缓存
            fastcgi_cache php_cache;
            fastcgi_cache_valid 200 60m;
            fastcgi_cache_bypass $http_pragma $http_authorization;
            fastcgi_no_cache $http_pragma $http_authorization;
            
            # 超时
            fastcgi_read_timeout 300;
            fastcgi_send_timeout 300;
        }
    }
}
```

### PHP-FPM 完整配置

```ini
[www]
user = www-data
group = www-data

listen = /var/run/php/php8.2-fpm.sock
listen.owner = www-data
listen.group = www-data
listen.mode = 0660

; 进程管理
pm = dynamic
pm.max_children = 50
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 20
pm.max_requests = 500
pm.process_idle_timeout = 10s

; 性能
request_terminate_timeout = 300
request_slowlog_timeout = 10s
slowlog = /var/log/php8.2-fpm-slow.log

; 状态
pm.status_path = /status
ping.path = /ping
```

## 性能测试

### 使用 Apache Bench

```bash
# 基本测试
ab -n 1000 -c 10 http://localhost/

# 详细测试
ab -n 10000 -c 100 -k http://localhost/
```

### 使用 wrk

```bash
# 安装
brew install wrk  # macOS
# 或从源码编译

# 测试
wrk -t4 -c100 -d30s http://localhost/
```

## 注意事项

1. **资源监控**：定期监控 CPU、内存、磁盘 I/O 使用情况。

2. **日志管理**：定期清理日志文件，避免占用过多磁盘空间。

3. **缓存策略**：合理使用缓存，但要注意缓存失效策略。

4. **安全配置**：不要在生产环境暴露状态页面和调试信息。

5. **渐进优化**：不要一次性优化所有配置，逐步调整并观察效果。

## 练习

1. 配置 Nginx 和 PHP-FPM，实现基本的性能优化。

2. 启用 OPcache，对比启用前后的性能差异。

3. 配置慢日志，分析并优化慢请求。

4. 使用性能测试工具，评估优化效果。

5. 实现性能监控脚本，实时监控服务器状态。
