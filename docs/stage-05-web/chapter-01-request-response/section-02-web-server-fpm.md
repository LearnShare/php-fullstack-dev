# 5.1.2 Web Server 与 PHP-FPM 协作

## 概述

Web Server（Web 服务器）和 PHP-FPM（PHP FastCGI Process Manager）的协作是 PHP Web 应用运行的基础架构。理解这种协作机制对于部署、优化和排查 PHP Web 应用至关重要。本节详细介绍 Web Server 的作用、PHP-FPM 的作用、FastCGI 协议、完整的协作流程、配置优化等内容，帮助零基础学员理解 PHP Web 应用的运行架构。

在典型的 PHP Web 应用架构中，Web Server（如 Nginx、Apache）负责接收 HTTP 请求、处理静态文件、将 PHP 请求转发给 PHP-FPM。PHP-FPM 负责管理 PHP 进程、执行 PHP 代码、返回处理结果。两者通过 FastCGI 协议进行通信，实现高效的请求处理。

理解 Web Server 与 PHP-FPM 的协作机制有助于：
- 正确配置 Web 服务器和 PHP-FPM
- 优化应用性能
- 排查运行问题
- 理解 PHP Web 应用的运行原理

**主要内容**：
- Web Server 的作用和常见类型（Nginx、Apache）
- PHP-FPM 的作用和进程管理机制
- FastCGI 协议的原理和优势
- Web Server 与 PHP-FPM 的完整协作流程
- Nginx 和 Apache 的配置方法
- PHP-FPM 的配置和优化
- 性能调优和故障排查
- 实际应用示例和最佳实践

## 特性

- **职责分离**：Web Server 处理静态文件和请求转发，PHP-FPM 处理 PHP 代码执行
- **进程复用**：FastCGI 协议支持进程复用，提高性能
- **高并发**：PHP-FPM 进程池可以处理大量并发请求
- **资源隔离**：每个 PHP 进程独立运行，互不干扰
- **灵活配置**：可以根据需求调整进程数量、超时时间等参数
- **易于扩展**：可以水平扩展 PHP-FPM 服务器

## Web Server 的作用

Web Server 是接收和处理 HTTP 请求的服务器程序，在 PHP Web 应用中承担以下职责：

### 接收 HTTP 请求

Web Server 监听指定端口（通常是 80 或 443），接收来自客户端的 HTTP 请求。

**工作流程**：
1. 监听网络端口
2. 接收 TCP 连接
3. 解析 HTTP 请求
4. 根据配置处理请求

### 处理静态文件

Web Server 可以直接处理静态文件（HTML、CSS、JavaScript、图片等），无需经过 PHP 处理。

**优势**：
- 性能高：直接返回文件内容，无需执行 PHP 代码
- 资源消耗低：不占用 PHP 进程
- 响应速度快：减少处理环节

**示例**（Nginx 配置）：
```nginx
location /static/ {
    alias /var/www/static/;
    expires 30d;
    add_header Cache-Control "public, immutable";
}
```

### 请求转发

对于 PHP 文件请求，Web Server 将请求转发给 PHP-FPM 处理。

**转发方式**：
- **FastCGI**：通过 FastCGI 协议转发（推荐）
- **mod_php**：Apache 模块方式（已较少使用）

### 负载均衡

Web Server 可以将请求分发到多个 PHP-FPM 服务器，实现负载均衡。

**优势**：
- 提高处理能力
- 提高可用性
- 水平扩展

### 常见 Web Server

#### Nginx

**特点**：
- 高性能、低内存占用
- 事件驱动、异步非阻塞
- 适合高并发场景
- 配置简单

**使用场景**：
- 高并发 Web 应用
- 反向代理
- 负载均衡
- 静态文件服务

#### Apache

**特点**：
- 功能丰富、模块化
- 支持多种运行模式（prefork、worker、event）
- 配置灵活
- 社区活跃

**使用场景**：
- 传统 Web 应用
- 需要复杂配置的场景
- .htaccess 文件支持

## PHP-FPM 的作用

PHP-FPM（PHP FastCGI Process Manager）是 PHP 的 FastCGI 进程管理器，负责管理 PHP 进程、执行 PHP 代码。

### PHP 进程管理

PHP-FPM 管理一个进程池（Process Pool），包含多个 PHP 工作进程（Worker Process）。

**进程类型**：
- **Master 进程**：管理进程，负责创建和管理 Worker 进程
- **Worker 进程**：工作进程，负责执行 PHP 代码

**进程管理**：
- 动态创建和销毁 Worker 进程
- 根据负载调整进程数量
- 监控进程状态
- 处理进程异常

### FastCGI 协议处理

PHP-FPM 通过 FastCGI 协议与 Web Server 通信。

**通信流程**：
1. 接收 Web Server 的 FastCGI 请求
2. 解析请求参数
3. 执行 PHP 代码
4. 返回处理结果

### 请求执行

PHP-FPM 的 Worker 进程执行 PHP 代码，处理业务逻辑。

**执行流程**：
1. 接收请求参数
2. 加载 PHP 文件
3. 执行 PHP 代码
4. 生成响应内容
5. 返回响应

### 资源管理

PHP-FPM 管理 PHP 进程的资源使用。

**管理内容**：
- 内存使用
- CPU 使用
- 文件句柄
- 数据库连接

## FastCGI 协议

FastCGI（Fast Common Gateway Interface）是 CGI 的改进版本，解决了 CGI 的性能问题。

### CGI 的问题

**传统 CGI**：
- 每个请求创建一个新进程
- 进程创建和销毁开销大
- 无法处理高并发
- 资源消耗高

### FastCGI 的优势

**FastCGI**：
- 进程复用：Worker 进程可以处理多个请求
- 减少进程创建开销
- 提高并发处理能力
- 降低资源消耗

### FastCGI 协议原理

FastCGI 协议定义了 Web Server 和 FastCGI 应用之间的通信规范。

**通信方式**：
- 通过 TCP 或 Unix Socket 通信
- 使用二进制协议（比 HTTP 更高效）
- 支持请求复用和流水线处理

**协议特点**：
- 二进制协议：比文本协议更高效
- 请求复用：一个连接可以处理多个请求
- 异步处理：支持异步请求处理

### FastCGI 与 HTTP 的区别

| 特性 | HTTP | FastCGI |
|:-----|:-----|:--------|
| 协议类型 | 文本协议 | 二进制协议 |
| 通信对象 | 客户端-服务器 | Web Server-FastCGI 应用 |
| 连接复用 | 支持（HTTP/1.1+） | 支持 |
| 性能 | 较低 | 较高 |

## 协作流程

Web Server 与 PHP-FPM 的完整协作流程如下：

### 1. 客户端发送 HTTP 请求

客户端（浏览器）发送 HTTP 请求到 Web Server。

**示例请求**：
```http
GET /index.php HTTP/1.1
Host: example.com
```

### 2. Web Server 接收请求

Web Server 接收并解析 HTTP 请求。

**处理步骤**：
- 解析请求行
- 解析请求头
- 确定请求的文件类型

### 3. Web Server 判断请求类型

Web Server 根据请求的文件类型决定处理方式。

**判断逻辑**：
- **静态文件**（.html、.css、.js 等）：直接返回文件内容
- **PHP 文件**（.php）：转发给 PHP-FPM

### 4. Web Server 转发 PHP 请求

对于 PHP 文件请求，Web Server 通过 FastCGI 协议转发给 PHP-FPM。

**转发内容**：
- 请求方法（GET、POST 等）
- 请求 URI
- 请求头
- 请求体（如果有）
- 服务器环境变量

### 5. PHP-FPM 接收请求

PHP-FPM 的 Master 进程接收 FastCGI 请求，分配给空闲的 Worker 进程。

**分配策略**：
- 选择空闲的 Worker 进程
- 如果所有进程都忙碌，等待或创建新进程
- 根据配置决定是否创建新进程

### 6. Worker 进程执行 PHP 代码

Worker 进程执行 PHP 代码，处理业务逻辑。

**执行步骤**：
1. 解析请求参数
2. 设置环境变量（`$_SERVER`、`$_GET`、`$_POST` 等）
3. 加载和执行 PHP 文件
4. 生成响应内容

### 7. Worker 进程返回响应

Worker 进程将处理结果返回给 PHP-FPM Master 进程。

**返回内容**：
- 响应状态码
- 响应头
- 响应体

### 8. PHP-FPM 返回响应给 Web Server

PHP-FPM 通过 FastCGI 协议将响应返回给 Web Server。

### 9. Web Server 返回响应给客户端

Web Server 将响应转换为 HTTP 响应，返回给客户端。

**HTTP 响应**：
```http
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234

<html>...</html>
```

### 10. 连接管理

根据配置决定是否保持连接：
- **HTTP/1.0**：立即关闭连接
- **HTTP/1.1**：根据 `Connection` 头决定（Keep-Alive 保持连接）

## Nginx 配置

Nginx 通过 `fastcgi_pass` 指令将 PHP 请求转发给 PHP-FPM。

### 基本配置

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.php index.html;

    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

**配置说明**：
- `fastcgi_pass`：PHP-FPM 的地址和端口（TCP）或 Unix Socket 路径
- `fastcgi_index`：默认索引文件
- `fastcgi_param`：设置 FastCGI 参数
- `include fastcgi_params`：包含标准 FastCGI 参数

### 使用 Unix Socket

```nginx
location ~ \.php$ {
    fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}
```

**优势**：
- 性能更好（无需网络开销）
- 更安全（仅本地访问）
- 适合单服务器部署

### 完整配置示例

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.php index.html;

    # 静态文件处理
    location /static/ {
        alias /var/www/static/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # PHP 文件处理
    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param REQUEST_URI $request_uri;
        fastcgi_param QUERY_STRING $query_string;
        fastcgi_param REQUEST_METHOD $request_method;
        fastcgi_param CONTENT_TYPE $content_type;
        fastcgi_param CONTENT_LENGTH $content_length;
        include fastcgi_params;

        # 超时设置
        fastcgi_connect_timeout 60s;
        fastcgi_send_timeout 60s;
        fastcgi_read_timeout 60s;

        # 缓冲区设置
        fastcgi_buffer_size 64k;
        fastcgi_buffers 4 64k;
        fastcgi_busy_buffers_size 128k;
    }

    # 禁止访问隐藏文件
    location ~ /\. {
        deny all;
    }
}
```

## Apache 配置

Apache 可以通过 `mod_proxy_fcgi` 模块将 PHP 请求转发给 PHP-FPM。

### 基本配置

```apache
<VirtualHost *:80>
    ServerName example.com
    DocumentRoot /var/www/html

    <FilesMatch \.php$>
        SetHandler "proxy:fcgi://127.0.0.1:9000"
    </FilesMatch>
</VirtualHost>
```

### 使用 Unix Socket

```apache
<FilesMatch \.php$>
    SetHandler "proxy:unix:/var/run/php/php8.2-fpm.sock|fcgi://localhost"
</FilesMatch>
```

## PHP-FPM 配置

PHP-FPM 的配置文件通常是 `/etc/php/8.2/fpm/pool.d/www.conf`（路径可能因系统而异）。

### 进程池配置

```ini
[www]
user = www-data
group = www-data
listen = 127.0.0.1:9000
listen.owner = www-data
listen.group = www-data
listen.mode = 0660
```

**配置说明**：
- `user`、`group`：运行 PHP-FPM 进程的用户和组
- `listen`：监听地址和端口（TCP）或 Unix Socket 路径
- `listen.owner`、`listen.group`：Socket 文件的所有者和组
- `listen.mode`：Socket 文件的权限

### 进程管理配置

```ini
pm = dynamic
pm.max_children = 50
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 20
pm.max_requests = 500
```

**配置说明**：
- `pm`：进程管理方式（`static`、`dynamic`、`ondemand`）
- `pm.max_children`：最大子进程数
- `pm.start_servers`：启动时的子进程数（dynamic 模式）
- `pm.min_spare_servers`：最小空闲进程数（dynamic 模式）
- `pm.max_spare_servers`：最大空闲进程数（dynamic 模式）
- `pm.max_requests`：每个子进程处理的最大请求数（达到后重启）

### 进程管理方式

#### static（静态）

**特点**：
- 固定数量的子进程
- 进程数不变
- 适合负载稳定的场景

**配置**：
```ini
pm = static
pm.max_children = 50
```

#### dynamic（动态）

**特点**：
- 根据负载动态调整进程数
- 在最小和最大进程数之间调整
- 适合负载变化的场景（推荐）

**配置**：
```ini
pm = dynamic
pm.max_children = 50
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 20
```

#### ondemand（按需）

**特点**：
- 按需创建进程
- 空闲时销毁进程
- 适合低负载场景

**配置**：
```ini
pm = ondemand
pm.max_children = 50
pm.process_idle_timeout = 10s
```

### 超时配置

```ini
request_terminate_timeout = 60s
request_slowlog_timeout = 10s
```

**配置说明**：
- `request_terminate_timeout`：请求超时时间（超过后终止进程）
- `request_slowlog_timeout`：慢请求日志阈值（超过后记录日志）

### 日志配置

```ini
access.log = /var/log/php8.2-fpm.log
access.format = "%R - %u %t \"%m %r%Q%q\" %s %f %{mili}d %{kilo}M %C%%"
slowlog = /var/log/php8.2-fpm-slow.log
```

**配置说明**：
- `access.log`：访问日志路径
- `access.format`：访问日志格式
- `slowlog`：慢请求日志路径

## 性能优化

### 进程池优化

**根据服务器资源调整进程数**：
```ini
# 计算最大进程数
# max_children = (总内存 - 系统内存) / 单个进程内存
# 例如：8GB 内存，每个进程 50MB
# max_children = (8192 - 2048) / 50 ≈ 120

pm.max_children = 120
pm.start_servers = 20
pm.min_spare_servers = 10
pm.max_spare_servers = 30
```

### 使用 Unix Socket

Unix Socket 比 TCP 连接性能更好：

**Nginx 配置**：
```nginx
fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
```

**PHP-FPM 配置**：
```ini
listen = /var/run/php/php8.2-fpm.sock
```

### 启用 OPcache

OPcache 可以缓存 PHP 字节码，提高性能：

**php.ini 配置**：
```ini
opcache.enable=1
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.revalidate_freq=2
opcache.fast_shutdown=1
```

### 调整缓冲区大小

**Nginx 配置**：
```nginx
fastcgi_buffer_size 128k;
fastcgi_buffers 4 256k;
fastcgi_busy_buffers_size 256k;
```

## 使用场景

### Web 应用部署

- 部署 PHP Web 应用
- 配置 Web Server 和 PHP-FPM
- 优化应用性能

### 性能优化

- 调整进程池配置
- 优化 FastCGI 参数
- 启用缓存和优化

### 架构理解

- 理解 PHP Web 应用的运行架构
- 理解请求处理流程
- 理解性能瓶颈

### 故障排查

- 排查请求处理问题
- 排查性能问题
- 排查配置问题

## 注意事项

### FastCGI 配置正确性

- 确保 `fastcgi_pass` 地址正确
- 确保 `SCRIPT_FILENAME` 参数正确
- 确保包含必要的 FastCGI 参数

### 进程池配置

- 根据服务器资源合理配置进程数
- 避免进程数过多导致内存不足
- 避免进程数过少导致性能问题

### 超时设置

- 设置合理的超时时间
- 避免超时时间过短导致请求失败
- 避免超时时间过长导致资源浪费

### 资源限制

- 监控内存使用
- 监控 CPU 使用
- 监控文件句柄数量
- 监控数据库连接数

### 安全性

- 使用 Unix Socket 提高安全性
- 设置正确的文件权限
- 限制 PHP-FPM 进程的权限
- 防止未授权访问

## 常见问题

### Web Server 和 PHP-FPM 如何协作？

1. Web Server 接收 HTTP 请求
2. 判断是否为 PHP 文件请求
3. 通过 FastCGI 协议转发给 PHP-FPM
4. PHP-FPM 执行 PHP 代码
5. 返回处理结果给 Web Server
6. Web Server 返回 HTTP 响应给客户端

### FastCGI 协议的作用？

FastCGI 协议定义了 Web Server 和 FastCGI 应用（如 PHP-FPM）之间的通信规范，支持进程复用，提高性能。

### 如何优化 PHP-FPM 性能？

1. **调整进程池配置**：根据服务器资源合理配置进程数
2. **使用 Unix Socket**：比 TCP 连接性能更好
3. **启用 OPcache**：缓存 PHP 字节码
4. **调整缓冲区大小**：优化数据传输
5. **监控和调优**：根据实际负载调整配置

### 如何排查协作问题？

1. **检查配置**：确保 Web Server 和 PHP-FPM 配置正确
2. **查看日志**：检查 Web Server 和 PHP-FPM 的日志
3. **测试连接**：使用工具测试 FastCGI 连接
4. **监控资源**：检查内存、CPU、文件句柄等资源使用
5. **逐步排查**：从简单到复杂，逐步定位问题

### TCP 和 Unix Socket 的区别？

| 特性 | TCP | Unix Socket |
|:-----|:----|:------------|
| 通信方式 | 网络通信 | 本地文件系统通信 |
| 性能 | 较低（有网络开销） | 较高（无网络开销） |
| 适用场景 | 跨服务器通信 | 同服务器通信 |
| 安全性 | 需要网络安全措施 | 仅本地访问 |

## 最佳实践

### 合理配置进程池

- 根据服务器资源计算最大进程数
- 使用 dynamic 模式适应负载变化
- 设置合理的进程数范围

### 优化 FastCGI 参数

- 使用 Unix Socket 提高性能
- 调整缓冲区大小
- 设置合理的超时时间

### 监控资源使用

- 监控内存使用
- 监控 CPU 使用
- 监控进程数量
- 监控请求处理时间

### 理解协作机制

- 理解请求处理流程
- 理解 FastCGI 协议
- 理解进程管理机制
- 理解性能优化方法

### 安全性考虑

- 使用 Unix Socket 提高安全性
- 设置正确的文件权限
- 限制 PHP-FPM 进程的权限
- 定期更新和打补丁

## 相关章节

- **[5.1.1 HTTP 请求响应流程](section-01-http-flow.md)**：了解 HTTP 协议基础
- **[5.1.3 Shared Nothing 架构](section-03-shared-nothing.md)**：了解 PHP Web 应用的架构特点
- **[1.2.2 运行模式与验证](../../../stage-01-foundation/chapter-02-runtime/section-02-execution-modes.md)**：了解 PHP 的不同运行模式

## 练习任务

1. **配置 Nginx 和 PHP-FPM**
   - 安装和配置 Nginx
   - 安装和配置 PHP-FPM
   - 测试 PHP 文件访问

2. **调整进程池配置**
   - 修改 PHP-FPM 进程池配置
   - 观察进程数量变化
   - 测试性能影响

3. **使用 Unix Socket**
   - 配置 PHP-FPM 使用 Unix Socket
   - 配置 Nginx 使用 Unix Socket
   - 测试性能提升

4. **监控和优化**
   - 监控 PHP-FPM 进程状态
   - 查看访问日志和慢请求日志
   - 根据实际情况优化配置

5. **故障排查**
   - 模拟常见问题（配置错误、进程不足等）
   - 使用日志和工具排查问题
   - 解决问题并验证
