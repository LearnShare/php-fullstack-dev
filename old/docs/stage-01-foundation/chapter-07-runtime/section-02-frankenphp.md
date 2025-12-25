# 1.6.2 FrankenPHP

## 概述

FrankenPHP 是基于 Caddy 服务器的 PHP 集成方案。它提供了开箱即用的 HTTP 服务器、自动 HTTPS、HTTP/2 和 HTTP/3 支持。

## 特点

- **语言**：Caddy 服务器 + PHP 集成
- **架构**：内置 HTTP 服务器 + Worker 模式
- **优势**：开箱即用、自动 HTTPS、HTTP/2、HTTP/3
- **适用场景**：需要自动 HTTPS、简单部署、Laravel 应用（通过 Octane）

## 安装

### 方式一：Docker

```bash
# 使用 Docker
docker run -v $PWD:/app \
  -p 80:80 -p 443:443 \
  dunglas/frankenphp

# 或使用 docker-compose
```

### 方式二：Homebrew (macOS)

```bash
brew install dunglas/frankenphp/frankenphp
```

### 方式三：直接下载

```bash
# 下载二进制文件
# 参考 https://frankenphp.dev/docs/install/
```

## 配置

### 基本配置（Caddyfile）

```caddy
localhost {
    root * /app/public
    php_server
    file_server
}
```

### PHP 配置

```ini
; php.ini
frankenphp.worker = 1
frankenphp.worker_count = 4
```

### Worker 模式

```ini
; 启用 Worker 模式
frankenphp.worker = 1
frankenphp.worker_count = 4
frankenphp.worker_max_requests = 1000
```

## 使用

### 基本启动

```bash
# 启动 FrankenPHP
frankenphp serve

# 或使用 Docker
docker run -v $PWD:/app -p 80:80 dunglas/frankenphp
```

### 使用 Caddyfile

```bash
# 使用自定义 Caddyfile
frankenphp serve --config Caddyfile
```

## 完整配置示例

### Caddyfile

```caddy
localhost {
    root * /app/public
    encode zstd gzip
    
    # PHP 处理
    php_server
    
    # 静态文件
    file_server
    
    # 日志
    log {
        output file /var/log/frankenphp/access.log
        format json
    }
    
    # 错误处理
    handle_errors {
        rewrite * /error.html
        file_server
    }
}
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  frankenphp:
    image: dunglas/frankenphp
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - .:/app
      - ./Caddyfile:/etc/caddy/Caddyfile
    environment:
      SERVER_NAME: localhost
```

## 自动 HTTPS

### 配置

```caddy
example.com {
    root * /app/public
    php_server
    file_server
}
```

FrankenPHP 会自动：
- 获取 Let's Encrypt 证书
- 配置 HTTPS
- 自动续期证书

## HTTP/2 和 HTTP/3

### 自动支持

FrankenPHP 自动支持：
- HTTP/2：多路复用、服务器推送
- HTTP/3：基于 QUIC 协议

无需额外配置。

## Worker 模式

### 启用 Worker 模式

```ini
; php.ini
frankenphp.worker = 1
frankenphp.worker_count = 4
```

### Worker 配置

```ini
; Worker 数量
frankenphp.worker_count = 4

; 每个 Worker 最大请求数
frankenphp.worker_max_requests = 1000

; Worker 重启时间
frankenphp.worker_restart_after = 3600
```

## 完整示例

### 项目结构

```
my-project/
├── Caddyfile
├── docker-compose.yml
├── public/
│   └── index.php
└── composer.json
```

### Caddyfile

```caddy
localhost {
    root * /app/public
    encode zstd gzip
    
    php_server
    
    file_server
    
    log {
        output stdout
        format console
    }
}
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    image: dunglas/frankenphp
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - .:/app
      - ./Caddyfile:/etc/caddy/Caddyfile
    environment:
      SERVER_NAME: localhost
      CADDY_GLOBAL_OPTIONS: admin off
```

### public/index.php

```php
<?php
declare(strict_types=1);

echo "Hello from FrankenPHP\n";
echo "PHP Version: " . PHP_VERSION . "\n";
echo "SAPI: " . php_sapi_name() . "\n";
```

## 注意事项

1. **自动 HTTPS**：生产环境会自动获取证书，确保域名正确配置。

2. **Worker 模式**：启用 Worker 模式可以显著提升性能。

3. **配置管理**：使用 Caddyfile 管理配置，便于版本控制。

4. **日志管理**：配置日志输出，便于问题排查。

5. **性能监控**：监控 Worker 状态和请求处理时间。

## 练习

1. 使用 Docker 部署 FrankenPHP，创建一个简单的 Web 应用。

2. 配置自动 HTTPS，验证证书自动获取。

3. 启用 Worker 模式，对比性能差异。

4. 配置 HTTP/2 和 HTTP/3，验证协议支持。

5. 使用 FrankenPHP 部署一个 Laravel 应用（通过 Octane）。
