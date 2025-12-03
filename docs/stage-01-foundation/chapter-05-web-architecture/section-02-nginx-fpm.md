# 1.5.2 Nginx + PHP-FPM 架构

## 概述

Nginx + PHP-FPM 是 PHP Web 应用的标准架构。理解它们的工作原理对于部署和优化 Web 应用至关重要。

## 请求处理流程

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

### 详细流程

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

## FastCGI 协议

### 基本概念

- **FastCGI**：Fast Common Gateway Interface，CGI 的改进版本
- **特点**：
  - 持久连接：避免每次请求都创建新进程
  - 多路复用：一个连接可以处理多个请求
  - 二进制协议：比 HTTP 更高效

### 与 CGI 对比

| 特性 | CGI | FastCGI |
| :--- | :--- | :------- |
| 连接方式 | 每次请求创建新进程 | 持久连接 |
| 性能 | 低 | 高 |
| 资源消耗 | 高 | 低 |
| 适用场景 | 低流量 | 高流量 |

## Nginx 配置

### 基本配置

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

### 配置说明

| 指令 | 说明 | 示例 |
| :--- | :--- | :--- |
| `fastcgi_pass` | PHP-FPM 地址 | `unix:/var/run/php/php8.2-fpm.sock` 或 `127.0.0.1:9000` |
| `fastcgi_index` | 默认索引文件 | `index.php` |
| `fastcgi_param` | 传递参数 | `SCRIPT_FILENAME $document_root$fastcgi_script_name` |
| `include fastcgi_params` | 包含标准参数 | 包含常用 FastCGI 参数 |

### 完整配置示例

```nginx
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

## PHP-FPM 配置

### 基本配置

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

### 进程管理方式

| 方式 | 说明 | 适用场景 |
| :--- | :--- | :------- |
| `static` | 固定数量的进程 | 流量稳定 |
| `dynamic` | 动态调整进程数（推荐） | 流量波动 |
| `ondemand` | 按需创建进程 | 低流量场景 |

### 配置参数说明

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

### 性能优化配置

```ini
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

## 监控和调试

### 查看 PHP-FPM 状态

```bash
# 查看服务状态
sudo systemctl status php8.2-fpm

# 查看进程
ps aux | grep php-fpm

# 查看 Nginx 错误日志
sudo tail -f /var/log/nginx/error.log

# 查看 PHP-FPM 日志
sudo tail -f /var/log/php8.2-fpm.log
```

### PHP-FPM 状态页面

```ini
; 在 php-fpm.conf 中启用
pm.status_path = /status
ping.path = /ping
```

```nginx
# Nginx 配置
location ~ ^/(status|ping)$ {
    access_log off;
    allow 127.0.0.1;
    deny all;
    fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}
```

访问 `http://localhost/status` 查看状态。

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
fastcgi_send_timeout = 300;
```

### 4. 启用慢日志

```ini
; 记录执行时间超过 10 秒的请求
slowlog = /var/log/php8.2-fpm-slow.log
request_slowlog_timeout = 10s
```

## 完整示例

### docker-compose.yml

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - .:/var/www/html
      - ./docker/nginx/nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - php-fpm

  php-fpm:
    image: php:8.2-fpm
    volumes:
      - .:/var/www/html
    working_dir: /var/www/html
```

### nginx.conf

```nginx
server {
    listen 80;
    root /var/www/html/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass php-fpm:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

## 注意事项

1. **进程数配置**：根据服务器资源合理配置，避免过多或过少。

2. **Socket vs TCP**：优先使用 Unix Socket，性能更好。

3. **超时设置**：设置合理的超时时间，避免请求挂起。

4. **日志管理**：定期清理日志文件，避免占用过多磁盘空间。

5. **安全配置**：限制 PHP-FPM 状态页面的访问，避免泄露信息。

## 练习

1. 配置一个完整的 Nginx + PHP-FPM 环境。

2. 调整 PHP-FPM 进程池配置，观察不同配置对性能的影响。

3. 配置 PHP-FPM 状态页面，监控运行状态。

4. 启用慢日志，分析慢请求的原因。

5. 对比 Unix Socket 和 TCP 的性能差异。
