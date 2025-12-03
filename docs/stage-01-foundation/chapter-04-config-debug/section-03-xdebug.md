# 1.4.3 Xdebug 调试配置

## 概述

Xdebug 是 PHP 的调试和性能分析工具。它提供了断点调试、堆栈跟踪、代码覆盖率等功能，是 PHP 开发的重要工具。

## 为什么需要 Xdebug

### 主要功能

1. **断点调试**：在代码中设置断点，逐步执行，查看变量值
2. **堆栈跟踪**：查看函数调用链，定位问题
3. **性能分析**：分析代码执行时间，找出性能瓶颈
4. **代码覆盖率**：测试代码覆盖率分析

### 与 var_dump 对比

| 特性 | var_dump | Xdebug |
| :--- | :------- | :----- |
| 断点调试 | 否 | 是 |
| 逐步执行 | 否 | 是 |
| 变量监视 | 基础 | 完整 |
| 性能分析 | 否 | 是 |
| 代码覆盖率 | 否 | 是 |

## 安装 Xdebug

### 方式一：PECL 安装

```bash
# 安装 Xdebug
pecl install xdebug

# 验证安装
php -m | grep xdebug
```

### 方式二：Docker 环境

```dockerfile
# Dockerfile
FROM php:8.2-fpm

# 安装 Xdebug
RUN pecl install xdebug && \
    docker-php-ext-enable xdebug

# 配置 Xdebug
RUN echo "xdebug.mode=debug" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
    echo "xdebug.client_host=host.docker.internal" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini && \
    echo "xdebug.client_port=9003" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
```

### 方式三：包管理器

```bash
# Ubuntu/Debian
sudo apt install php-xdebug

# macOS
brew install php-xdebug
```

## 配置 Xdebug

### 基础配置

创建 `xdebug.ini` 或添加到 `php.ini`：

```ini
; Xdebug 配置
zend_extension=xdebug

; 调试模式
xdebug.mode=debug

; 自动启动（开发环境）
xdebug.start_with_request=yes

; 客户端配置
xdebug.client_host=127.0.0.1
xdebug.client_port=9003

; IDE 标识
xdebug.idekey=PHPSTORM

; 日志（可选）
xdebug.log=/var/log/php/xdebug.log
xdebug.log_level=0
```

### 配置参数说明

| 参数 | 说明 | 推荐值 |
| :--- | :--- | :----- |
| `xdebug.mode` | 调试模式 | `debug`（开发）、`off`（生产） |
| `xdebug.start_with_request` | 自动启动调试 | `yes`（开发）、`no`（生产） |
| `xdebug.client_host` | IDE 所在主机 | `127.0.0.1` 或 `host.docker.internal` |
| `xdebug.client_port` | IDE 监听端口 | `9003`（Xdebug 3.x） |
| `xdebug.idekey` | IDE 标识 | `PHPSTORM` 或自定义 |

### 开发环境配置

```ini
; 开发环境
xdebug.mode=debug,develop
xdebug.start_with_request=yes
xdebug.client_host=127.0.0.1
xdebug.client_port=9003
xdebug.idekey=PHPSTORM
```

### 生产环境配置

```ini
; 生产环境（禁用 Xdebug）
xdebug.mode=off
```

### Docker 环境配置

```ini
; Docker 环境
xdebug.mode=debug
xdebug.start_with_request=yes
xdebug.client_host=host.docker.internal
xdebug.client_port=9003
xdebug.idekey=PHPSTORM
```

## IDE 配置

### PhpStorm 配置

#### 1. 设置 PHP 解释器

1. File → Settings → PHP
2. 选择 PHP 解释器路径
3. 设置 PHP 版本

#### 2. 配置服务器

1. File → Settings → PHP → Servers
2. 添加服务器：
   - Name: `localhost`
   - Host: `localhost`
   - Port: `80`
   - Debugger: `Xdebug`
   - Path mappings: 项目路径映射

#### 3. 启用监听

1. 点击工具栏的 "Start Listening for PHP Debug Connections" 按钮
2. 或使用快捷键（需配置）

#### 4. 设置断点

在代码行号左侧点击，设置断点。

### VS Code 配置

#### 1. 安装扩展

安装 "PHP Debug" 扩展：
```bash
code --install-extension xdebug.php-debug
```

#### 2. 创建 launch.json

创建 `.vscode/launch.json`：

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Listen for Xdebug",
            "type": "php",
            "request": "launch",
            "port": 9003,
            "pathMappings": {
                "/var/www/html": "${workspaceFolder}"
            },
            "log": true
        }
    ]
}
```

#### 3. 启动调试

1. 按 F5 或点击调试按钮
2. 选择 "Listen for Xdebug"
3. 设置断点
4. 访问页面触发调试

## 断点调试流程

### 步骤 1：设置断点

```php
<?php
declare(strict_types=1);

function calculateTotal(array $items): float
{
    $total = 0.0;
    
    foreach ($items as $item) {  // 在此行设置断点
        $total += $item['price'] * $item['quantity'];
    }
    
    return $total;  // 或在此行设置断点
}
```

### 步骤 2：启动调试

1. IDE 中点击 "Start Listening"
2. 浏览器访问页面（或使用 Xdebug helper 插件）
3. IDE 会自动停在断点处

### 步骤 3：调试操作

- **Step Over (F10)**：执行当前行，不进入函数
- **Step Into (F11)**：进入函数内部
- **Step Out (Shift+F11)**：跳出当前函数
- **Continue (F5)**：继续执行到下一个断点
- **Stop**：停止调试

### 步骤 4：查看变量

- **Variables 面板**：查看当前作用域的所有变量
- **Watch 面板**：监视特定表达式
- **Call Stack**：查看函数调用栈

## 调试技巧

### 条件断点

在 IDE 中设置断点时，添加条件：

```php
<?php
// 只在特定条件下触发断点
// 在 IDE 中设置断点时，添加条件：$userId === 1
if ($userId === 1) {
    // 断点会在这里触发
}
```

### 日志断点

不停止执行，只记录日志：

```php
<?php
// 在 IDE 中设置日志断点，输出：User ID: {$userId}
```

### 临时断点

只触发一次的断点：

```php
<?php
// 在 IDE 中设置临时断点
```

### 异常断点

在抛出异常时自动停止：

```php
<?php
// 在 IDE 中配置异常断点
```

## 常见问题排查

### 问题 1：断点不触发

**原因**：
- Xdebug 未正确安装或启用
- IDE 未监听调试连接
- 路径映射不正确

**解决方案**：

```bash
# 检查 Xdebug 是否加载
php -m | grep xdebug

# 检查 Xdebug 配置
php --ri xdebug

# 验证连接
# 在代码中添加
xdebug_break(); // 强制断点
```

### 问题 2：Docker 环境连接失败

**原因**：
- `xdebug.client_host` 配置错误
- 防火墙阻止连接

**解决方案**：

```ini
; Windows/macOS Docker
xdebug.client_host=host.docker.internal

; Linux Docker
xdebug.client_host=172.17.0.1  ; 或宿主机 IP

; 验证连接
xdebug.client_port=9003
xdebug.log=/var/log/php/xdebug.log
xdebug.log_level=0
```

### 问题 3：性能下降

**原因**：
- Xdebug 在每次请求时都启动
- 开启了性能分析模式

**解决方案**：

```ini
; 生产环境禁用
xdebug.mode=off

; 开发环境按需启动
xdebug.start_with_request=trigger  ; 需要 XDEBUG_SESSION cookie
```

## 性能分析

### 启用性能分析

```ini
; xdebug.ini
xdebug.mode=profile
xdebug.output_dir=/tmp/xdebug
```

### 分析结果

```bash
# 生成分析文件
# 使用工具分析：
# - KCacheGrind (Linux)
# - WinCacheGrind (Windows)
# - WebGrind (Web)
```

## 代码覆盖率

### 启用覆盖率分析

```ini
xdebug.mode=coverage
```

### 使用示例

```php
<?php
declare(strict_types=1);

use PHPUnit\Framework\TestCase;

class CoverageTest extends TestCase
{
    public function testFunction(): void
    {
        // 运行测试
        $result = myFunction();
        
        // 生成覆盖率报告
        // phpunit --coverage-html coverage/
    }
}
```

## 完整示例

### 配置脚本

```php
<?php
declare(strict_types=1);

class XdebugConfig
{
    public static function setup(string $env = 'development'): void
    {
        if ($env === 'development') {
            ini_set('xdebug.mode', 'debug,develop');
            ini_set('xdebug.start_with_request', 'yes');
            ini_set('xdebug.client_host', '127.0.0.1');
            ini_set('xdebug.client_port', '9003');
        } else {
            ini_set('xdebug.mode', 'off');
        }
    }
    
    public static function check(): bool
    {
        return extension_loaded('xdebug');
    }
}

// 使用
if (XdebugConfig::check()) {
    XdebugConfig::setup(getenv('APP_ENV') ?: 'development');
}
```

## 注意事项

1. **性能影响**：Xdebug 会显著影响性能，生产环境必须禁用。

2. **端口配置**：Xdebug 3.x 默认使用 9003 端口，确保 IDE 监听正确端口。

3. **路径映射**：Docker 环境必须正确配置路径映射。

4. **触发方式**：可以配置为自动启动或按需启动（通过 cookie）。

5. **版本兼容**：确保 Xdebug 版本与 PHP 版本兼容。

## 练习

1. 安装和配置 Xdebug，验证是否正常工作。

2. 在 PhpStorm 或 VS Code 中配置 Xdebug 调试。

3. 设置断点，调试一个简单的 PHP 函数。

4. 配置条件断点和日志断点。

5. 使用 Xdebug 进行性能分析，找出代码瓶颈。
