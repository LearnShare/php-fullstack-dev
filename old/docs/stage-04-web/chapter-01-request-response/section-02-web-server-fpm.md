# 4.1.2 Web Server 与 PHP-FPM 协作

## 概述

Web Server（Nginx/Apache）与 PHP-FPM 的协作是 PHP Web 应用的核心。本节详细介绍 Nginx/Apache 配置、FastCGI 协议、PHP-FPM 工作原理、进程池配置优化，以及性能调优和调试技巧。

## Nginx 配置

### 基础配置

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.php index.html;

    # PHP 文件处理
    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # 静态文件直接返回
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### 高级配置

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;

    # 请求体大小限制
    client_max_body_size 10M;

    # 超时设置
    fastcgi_connect_timeout 60s;
    fastcgi_send_timeout 60s;
    fastcgi_read_timeout 60s;

    # 缓冲区设置
    fastcgi_buffers 8 16k;
    fastcgi_buffer_size 32k;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;  # 使用 Unix Socket
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        
        # 安全设置
        fastcgi_param PHP_ADMIN_VALUE "open_basedir=/var/www/html:/tmp";
    }
}
```

### Unix Socket vs TCP

**Unix Socket（推荐）**：
- 性能更好（无需网络开销）
- 更安全（仅本地访问）
- 配置：`fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;`

**TCP Socket**：
- 可跨服务器部署
- 配置：`fastcgi_pass 127.0.0.1:9000;`

## FastCGI 协议

### 协议特点

- **持久连接**：避免每次请求都建立新连接
- **多路复用**：一个连接可以处理多个请求
- **进程复用**：PHP 进程可以处理多个请求

### 通信流程

```
1. Web Server 建立 FastCGI 连接
2. 发送请求数据（环境变量、请求体）
3. PHP-FPM 处理请求
4. 返回响应数据
5. 保持连接（等待下一个请求）
```

## PHP-FPM 工作原理

### 进程模型

```
Master 进程
  ├── Worker 进程 1
  ├── Worker 进程 2
  ├── Worker 进程 3
  └── ...
```

- **Master 进程**：管理 Worker 进程，不处理请求
- **Worker 进程**：执行 PHP 代码，处理请求

### 进程池配置

**php-fpm.conf 配置示例：**

```ini
[www]
user = www-data
group = www-data

# 监听方式
listen = /var/run/php/php8.3-fpm.sock
listen.owner = www-data
listen.group = www-data
listen.mode = 0660

# 进程管理方式
pm = dynamic

# 进程数配置
pm.max_children = 50        # 最大进程数
pm.start_servers = 5        # 启动时的进程数
pm.min_spare_servers = 5    # 最小空闲进程数
pm.max_spare_servers = 35   # 最大空闲进程数
pm.max_requests = 500       # 每个进程处理的最大请求数

# 进程超时
request_terminate_timeout = 30s

# 慢日志
slowlog = /var/log/php-fpm/slow.log
request_slowlog_timeout = 10s
```

### 配置说明

| 配置项 | 说明 | 推荐值 |
| :--- | :--- | :--- |
| `pm.max_children` | 最大 Worker 进程数 | 根据服务器内存计算 |
| `pm.start_servers` | 启动时的进程数 | `pm.min_spare_servers` 的值 |
| `pm.min_spare_servers` | 最小空闲进程数 | `pm.max_children * 0.1` |
| `pm.max_spare_servers` | 最大空闲进程数 | `pm.max_children * 0.7` |
| `pm.max_requests` | 每个进程处理的最大请求数 | 500-1000（防止内存泄漏） |

### 进程数计算

```bash
# 计算最大进程数
# 公式：max_children = (可用内存 - 系统预留) / 每个进程内存

# 示例：服务器 4GB 内存，每个 PHP 进程约 50MB
# max_children = (4096 - 512) / 50 ≈ 70
```

## 性能优化

### 1. OPcache 配置

```ini
; php.ini
opcache.enable=1
opcache.memory_consumption=128
opcache.max_accelerated_files=10000
opcache.validate_timestamps=0  # 生产环境关闭
opcache.revalidate_freq=0
```

### 2. 进程池优化

```ini
; 根据服务器资源调整
pm.max_children = 50
pm.start_servers = 10
pm.min_spare_servers = 10
pm.max_spare_servers = 30
```

### 3. 连接优化

```php
<?php
// 使用持久连接（谨慎使用）
$pdo = new PDO(
    'mysql:host=localhost;dbname=app',
    'user',
    'password',
    [PDO::ATTR_PERSISTENT => true]  // 生产环境需评估
);
```

### 4. 静态文件分离

```nginx
# Nginx 直接处理静态文件，不经过 PHP-FPM
location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|svg)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
    access_log off;
}
```

## 调试技巧

### 查看 PHP-FPM 状态

```nginx
# Nginx 配置
location ~ ^/status$ {
    access_log off;
    allow 127.0.0.1;
    deny all;
    fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}
```

```php
<?php
// 访问 http://example.com/status
// 显示进程池状态信息
```

### 慢日志分析

```ini
; php-fpm.conf
slowlog = /var/log/php-fpm/slow.log
request_slowlog_timeout = 10s
```

```bash
# 分析慢日志
tail -f /var/log/php-fpm/slow.log
```

### 进程监控

```bash
# 查看 PHP-FPM 进程
ps aux | grep php-fpm

# 查看进程数
ps aux | grep php-fpm | wc -l

# 监控进程状态
watch -n 1 'ps aux | grep php-fpm | grep -v grep'
```

## 完整示例

### Nginx + PHP-FPM 完整配置

```nginx
# /etc/nginx/sites-available/example.com
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.php index.html;

    # 日志
    access_log /var/log/nginx/example.com.access.log;
    error_log /var/log/nginx/example.com.error.log;

    # 安全头
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # PHP 处理
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
        
        # 超时设置
        fastcgi_read_timeout 60s;
    }

    # 静态文件
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # 禁止访问隐藏文件
    location ~ /\. {
        deny all;
    }
}
```

## 注意事项

1. **进程数配置**：根据服务器资源合理配置，避免内存溢出
2. **超时设置**：设置合理的超时时间，避免长时间占用进程
3. **日志管理**：定期清理日志，避免磁盘空间不足
4. **安全配置**：限制 PHP 执行目录，防止安全漏洞

## 练习

1. 配置本地 Nginx + PHP-FPM 环境，实现基本的 PHP 文件处理。

2. 调整 PHP-FPM 进程池配置，观察不同配置下的性能表现。

3. 实现 PHP-FPM 状态监控页面，显示进程池状态信息。

4. 配置慢日志，分析应用中的性能瓶颈。

5. 优化 Nginx 配置，实现静态文件缓存和 Gzip 压缩。
