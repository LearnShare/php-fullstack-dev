# 1.2.2 运行模式与验证

## 概述

PHP 支持多种运行模式，每种模式适用于不同的场景。理解这些模式的差异对于选择合适的部署方案至关重要。本节介绍 PHP 的基础运行模式、特点、使用方法和验证方式。

## 运行模式对比

| 模式 | 用途 | 特点 | 适用场景 |
|:-----|:-----|:-----|:---------|
| CLI | 命令行执行 | 单次执行，脚本结束后进程退出 | 脚本、迁移、队列 |
| 内置服务器 | 开发测试 | 单线程，不适合生产环境 | 快速演示、本地开发 |
| PHP-FPM | Web 应用 | 进程池管理，高性能 | 生产环境标准 |

## CLI 模式（命令行）

### 特点

- **用途**：执行脚本、运行迁移、处理队列任务
- **特点**：单次执行，脚本结束后进程退出
- **类比**：类似 Node.js 的 CLI 脚本

### 基本使用

**执行脚本**：

```bash
php script.php
```

**执行单行代码**：

```bash
php -r "echo 'Hello World';"
```

**输出**：

```
Hello World
```

### 代码示例

**示例 1：简单的 CLI 脚本**

创建文件 `hello.php`：

```php
<?php
echo "Hello from CLI!\n";
echo "Current time: " . date('Y-m-d H:i:s') . "\n";
```

运行：

```bash
php hello.php
```

**输出**：

```
Hello from CLI!
Current time: 2024-01-15 10:30:00
```

**示例 2：CLI 模式检测**

创建文件 `check-mode.php`：

```php
<?php
if (php_sapi_name() === 'cli') {
    echo "Running in CLI mode\n";
} else {
    echo "Running in Web mode\n";
}
```

运行：

```bash
php check-mode.php
```

**输出**：

```
Running in CLI mode
```

## 内置 Web 服务器

### 特点

- **用途**：开发测试、快速演示
- **特点**：单线程，不适合生产环境
- **限制**：性能有限，功能简单

### 基本使用

**启动服务器**：

```bash
php -S localhost:8000 -t public
```

**指定路由脚本**：

```bash
php -S localhost:8000 router.php
```

### 完整示例

**示例 1：启动内置服务器**

```bash
# 在项目根目录启动
php -S localhost:8000

# 输出
PHP 8.3.0 Development Server (http://localhost:8000) started
```

在浏览器中访问 `http://localhost:8000` 即可查看页面。

**示例 2：使用路由脚本**

创建文件 `router.php`：

```php
<?php
// 检查文件是否存在
if (file_exists(__DIR__ . $_SERVER['REQUEST_URI'])) {
    return false; // 直接返回文件
}

// 路由处理
if ($_SERVER['REQUEST_URI'] === '/') {
    echo "Welcome to PHP built-in server!";
} else {
    http_response_code(404);
    echo "404 Not Found";
}
```

启动服务器：

```bash
php -S localhost:8000 router.php
```

**示例 3：指定文档根目录**

```bash
# 指定 public 目录为文档根目录
php -S localhost:8000 -t public

# 访问 http://localhost:8000/index.php
```

## PHP-FPM 模式

### 特点

- **用途**：生产环境 Web 应用
- **特点**：进程池管理，高性能
- **配置**：需要 Nginx 或 Apache 配合

### 基本配置

**PHP-FPM 配置文件位置**：

- Linux：`/etc/php/8.3/fpm/php-fpm.conf`
- macOS：`/opt/homebrew/etc/php/8.3/php-fpm.conf`

**进程池配置示例**：

```ini
[www]
user = www-data
group = www-data
listen = 127.0.0.1:9000
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
```

**与 Nginx 的配合**：

```nginx
location ~ \.php$ {
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}
```

### 验证方法

**检查 PHP-FPM 进程**：

```bash
ps aux | grep php-fpm
```

**输出示例**：

```
www-data  1234  0.0  0.5  123456  5678 ?  S    10:00   0:00 php-fpm: master process
www-data  1235  0.0  0.3  123456  3456 ?  S    10:00   0:00 php-fpm: pool www
```

**测试 PHP-FPM**：

创建文件 `info.php`：

```php
<?php
phpinfo();
```

通过浏览器访问，查看 PHP 信息。

## 高性能运行时

除了上述基础运行模式外，PHP 还支持高性能运行时（如 RoadRunner、FrankenPHP、OpenSwoole），这些运行时通过常驻内存等方式提升性能，适合高并发场景。详细内容见**阶段八：性能优化与安全**。

## 运行验证

### 验证方法

**检测运行模式**：

```php
<?php
echo php_sapi_name();
```

**输出示例**：

- CLI 模式：`cli`
- 内置服务器：`cli-server`
- PHP-FPM：`fpm-fcgi`
- Apache：`apache2handler`

**查看配置信息**：

```bash
php -i
```

**查看已加载的扩展**：

```bash
php -m
```

**检查进程**：

```bash
# Linux/macOS
ps aux | grep php

# Windows
tasklist | findstr php
```

### 完整示例

**示例 1：CLI 模式验证**

创建文件 `verify-cli.php`：

```php
<?php
echo "SAPI Name: " . php_sapi_name() . "\n";
echo "Is CLI: " . (php_sapi_name() === 'cli' ? 'Yes' : 'No') . "\n";
```

运行：

```bash
php verify-cli.php
```

**输出**：

```
SAPI Name: cli
Is CLI: Yes
```

**示例 2：Web 模式验证**

创建文件 `verify-web.php`：

```php
<?php
echo "SAPI Name: " . php_sapi_name() . "\n";
echo "Is CLI: " . (php_sapi_name() === 'cli' ? 'Yes' : 'No') . "\n";
```

通过内置服务器访问：

```bash
php -S localhost:8000
```

在浏览器中访问 `http://localhost:8000/verify-web.php`。

**输出**：

```
SAPI Name: cli-server
Is CLI: No
```

**示例 3：模式检测函数**

创建文件 `detect-mode.php`：

```php
<?php
function getRunMode(): string
{
    $sapi = php_sapi_name();
    
    return match ($sapi) {
        'cli' => 'CLI Mode',
        'cli-server' => 'Built-in Server',
        'fpm-fcgi' => 'PHP-FPM',
        'apache2handler' => 'Apache',
        default => 'Unknown: ' . $sapi,
    };
}

echo "Current mode: " . getRunMode() . "\n";
```

## 注意事项

- **生产环境**：不要使用内置 Web 服务器，应使用 PHP-FPM
- **模式检测**：在代码中检测运行模式，避免在 CLI 模式下执行 Web 相关代码
- **配置差异**：不同模式的配置可能不同，需要注意区分

## 常见问题

### 问题 1：内置服务器无法访问

**原因**：端口被占用或防火墙阻止

**解决方法**：

1. 检查端口占用：`netstat -an | grep 8000`（Linux/macOS）或 `netstat -an | findstr 8000`（Windows）
2. 更换端口：`php -S localhost:8080`
3. 检查防火墙设置

### 问题 2：PHP-FPM 无法启动

**原因**：配置文件错误或权限问题

**解决方法**：

1. 检查配置文件语法：`php-fpm -t`
2. 检查日志文件：`/var/log/php-fpm.log`
3. 确保有执行权限：`chmod +x /usr/sbin/php-fpm`

### 问题 3：CLI 和 Web 模式行为不一致

**原因**：配置不同或环境变量不同

**解决方法**：

1. 统一配置文件
2. 检查环境变量差异
3. 使用 `php -i` 对比配置信息

## 最佳实践

- 开发环境使用内置服务器或 PHP-FPM
- 生产环境必须使用 PHP-FPM
- 在代码中检测运行模式，避免模式相关的错误
- 记录运行模式配置，便于问题排查

## 相关章节

- **阶段八：性能优化与安全**：深入学习高性能运行时（RoadRunner、FrankenPHP、OpenSwoole）

## 练习任务

1. 创建一个 CLI 脚本 `greet.php`，接受命令行参数并输出问候语（如 `php greet.php John` 输出 "Hello, John!"）。
2. 使用内置服务器启动一个简单的 PHP 页面，并在浏览器中访问。
3. 创建一个脚本检测当前运行模式，分别在 CLI 和 Web 模式下运行并查看输出。
4. 使用 `php_sapi_name()` 函数检测运行模式，并编写一个函数根据模式执行不同的逻辑。
5. 了解 PHP-FPM 的基本配置，并查看本地 PHP-FPM 配置文件（如果已安装）。
