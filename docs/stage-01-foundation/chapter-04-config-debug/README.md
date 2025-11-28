# 1.4 配置、扩展与调试

## php.ini

- 使用 `php --ini` 定位“Loaded Configuration File”，CLI 与 FPM 常见不一致，新手可在 `phpinfo()` 页面确认。
- 建议在仓库中保存一份 `php.ini.dev` 模板，标注关键配置与注释来源。
- 常用配置：

| 项目             | 开发值           | 生产值                  | 说明                         |
| :--------------- | :--------------- | :---------------------- | :--------------------------- |
| `display_errors` | On               | Off                     | 线上禁止直接输出错误。       |
| `error_reporting` | `E_ALL`         | `E_ALL & ~E_DEPRECATED` | 开发阶段捕获所有错误。       |
| `memory_limit`   | 512M             | 256M                    | 根据业务调整，避免无限制。   |
| `opcache.enable` | 1                | 1                       | 生产必须开启。               |
| `date.timezone`  | `Asia/Shanghai`  | `Asia/Shanghai`         | 如未设置将触发 Warning。     |

## 扩展管理

- 通过 `php -m` 或 `php --ri xdebug` 检查状态。
- 核心扩展：
  - `pdo_mysql`、`mysqli`（数据库连接）
  - `mbstring`（多字节字符串）
  - `curl`（HTTP 调用）
  - `intl`（国际化）
  - `gd` 或 `imagick`（图像处理）
- 安装方式：
  - Windows：在 `php.ini` 中取消注释 `extension=pdo_mysql` 等条目。
  - macOS/Linux：使用包管理器，如 `brew install php@8.3-imagick` 或 `apt install php8.2-mysql`.
  - 在容器中使用 `docker-php-ext-install`，例如：
  ```bash
  docker-php-ext-install pdo_mysql intl opcache
  ```
  - 如需 PECL 扩展（redis、xdebug），执行 `pecl install redis` 并在 `php.ini` 中通过 `extension=redis` 启用。

## Xdebug 调试

### 为什么需要 Xdebug

- **断点调试**：在代码中设置断点，逐步执行，查看变量值。
- **堆栈跟踪**：查看函数调用链，定位问题。
- **性能分析**：分析代码执行时间，找出性能瓶颈。
- **代码覆盖率**：测试代码覆盖率分析。

### 安装 Xdebug

#### 方式一：PECL 安装

```bash
# 安装 Xdebug
pecl install xdebug

# 验证安装
php -m | grep xdebug
```

#### 方式二：Docker 环境

```dockerfile
# Dockerfile
FROM php:8.2-fpm

# 安装 Xdebug
RUN pecl install xdebug && \
    docker-php-ext-enable xdebug
```

#### 方式三：包管理器

```bash
# Ubuntu/Debian
sudo apt install php-xdebug

# macOS
brew install php-xdebug
```

### 配置 Xdebug

#### 基础配置

```ini
; xdebug.ini
zend_extension=xdebug

; 调试模式
xdebug.mode=debug

; 自动启动（开发环境）
xdebug.start_with_request=yes

; 客户端配置
xdebug.client_host=host.docker.internal
xdebug.client_port=9003

; 日志（可选）
xdebug.log=/var/log/php/xdebug.log
xdebug.log_level=0
```

#### 配置参数说明

| 参数                      | 说明                           | 推荐值                    |
| :------------------------ | :----------------------------- | :------------------------ |
| `xdebug.mode`             | 调试模式                       | `debug`（开发）、`off`（生产） |
| `xdebug.start_with_request` | 自动启动调试                | `yes`（开发）、`no`（生产） |
| `xdebug.client_host`      | IDE 所在主机                   | `127.0.0.1` 或 `host.docker.internal` |
| `xdebug.client_port`      | IDE 监听端口                   | `9003`（Xdebug 3.x）      |
| `xdebug.idekey`           | IDE 标识                       | `PHPSTORM` 或自定义       |

### IDE 配置

#### PhpStorm 配置

1. **设置 PHP 解释器**
   - File → Settings → PHP
   - 选择 PHP 解释器路径

2. **配置服务器**
   - File → Settings → PHP → Servers
   - 添加服务器：
     - Name: `localhost`
     - Host: `localhost`
     - Port: `80`
     - Debugger: `Xdebug`
     - Path mappings: 项目路径映射

3. **启用监听**
   - 点击工具栏的 "Start Listening for PHP Debug Connections" 按钮
   - 或使用快捷键（默认：无，需配置）

#### VS Code 配置

1. **安装扩展**
   - 安装 "PHP Debug" 扩展

2. **创建 launch.json**
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
         }
       }
     ]
   }
   ```

3. **启动调试**
   - 按 F5 或点击调试按钮
   - 选择 "Listen for Xdebug"

### 断点调试流程

#### 步骤 1：设置断点

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

#### 步骤 2：启动调试

1. IDE 中点击 "Start Listening"
2. 浏览器访问页面（或使用 Xdebug helper 插件）
3. IDE 会自动停在断点处

#### 步骤 3：调试操作

- **Step Over (F10)**：执行当前行，不进入函数
- **Step Into (F11)**：进入函数内部
- **Step Out (Shift+F11)**：跳出当前函数
- **Continue (F5)**：继续执行到下一个断点
- **Stop**：停止调试

#### 步骤 4：查看变量

- **Variables 面板**：查看当前作用域的所有变量
- **Watch 面板**：监视特定表达式
- **Call Stack**：查看函数调用栈

### 调试技巧

#### 1. 条件断点

```php
<?php
// 只在特定条件下触发断点
// 在 IDE 中设置断点时，添加条件：$userId === 1
if ($userId === 1) {
    // 断点会在这里触发
}
```

#### 2. 日志断点

```php
<?php
// 不停止执行，只记录日志
// 在 IDE 中设置日志断点，输出：User ID: {$userId}
```

#### 3. 临时断点

```php
<?php
// 只触发一次的断点
// 在 IDE 中设置临时断点
```

#### 4. 异常断点

```php
<?php
// 在抛出异常时自动停止
// 在 IDE 中配置异常断点
```

### 常见问题排查

#### 问题 1：断点不触发

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

#### 问题 2：Docker 环境连接失败

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

#### 问题 3：性能下降

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

### 性能分析

#### 启用性能分析

```ini
; xdebug.ini
xdebug.mode=profile
xdebug.output_dir=/tmp/xdebug
```

#### 分析结果

```bash
# 生成分析文件
# 使用工具分析：
# - KCacheGrind (Linux)
# - WinCacheGrind (Windows)
# - WebGrind (Web)
```

### 代码覆盖率

#### 启用覆盖率分析

```ini
xdebug.mode=coverage
```

#### 使用示例

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

## 调试技巧补充

### 1. 使用 var_dump 和 print_r

```php
<?php
// var_dump：显示类型和值
var_dump($variable);

// print_r：更易读的输出
print_r($variable, true); // 返回字符串

// 格式化输出
echo '<pre>';
print_r($variable);
echo '</pre>';
```

### 2. 使用 error_log

```php
<?php
// 写入错误日志
error_log('Debug: ' . print_r($data, true));

// 带上下文
error_log(sprintf(
    'User %d accessed %s at %s',
    $userId,
    $url,
    date('Y-m-d H:i:s')
));
```

### 3. 使用 debug_backtrace

```php
<?php
function debugTrace(): void
{
    $trace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 5);
    foreach ($trace as $frame) {
        echo sprintf(
            "%s:%d in %s()\n",
            $frame['file'] ?? 'unknown',
            $frame['line'] ?? 0,
            $frame['function'] ?? 'unknown'
        );
    }
}
```

### 4. 使用 Xdebug 函数

```php
<?php
// 强制断点
xdebug_break();

// 获取堆栈跟踪
$trace = xdebug_get_function_stack();

// 获取变量名
$vars = xdebug_get_declared_vars();
```

## 配置差异追踪

- 使用 `php -i | grep php.ini` 确保 CLI/FPM 共用配置。
- 通过 `ini_set()` 在入口脚本中追加临时配置，便于差异对比。
- 生产环境将 `php.ini` 存入版本库（或管理在 Ansible、Chef）确保可追溯。

## 调试技巧

- `xdebug_info()`：快速诊断 Xdebug 是否加载。
- `strace -p <pid>`：分析 FPM 进程阻塞。
- `request_slowlog_timeout` + `slowlog`：捕获慢请求调用栈。

## 常用函数与命令语法

| 名称           | 语法                                                   | 参数说明                            | 示例                              |
| :------------- | :----------------------------------------------------- | :---------------------------------- | :-------------------------------- |
| `ini_get`      | `ini_get(string $option): string|false`                | `$option` 为配置项名称               | `ini_get('memory_limit')`         |
| `ini_set`      | `ini_set(string $option, string|int|float|bool $value): string|false` | 返回旧值；失败时返回 `false` | `ini_set('display_errors', '1');` |
| `xdebug_info`  | `xdebug_info(?string $category = null): array|string`  | `$category` 可为 `mode`、`functions` 等 | `var_dump(xdebug_info('mode'));` |
| `php --ri`     | `php --ri extension`                                   | 显示扩展详细配置                     | `php --ri xdebug`                 |
| `strace`       | `strace -p <pid>`                                      | Linux 下追踪系统调用，需 root 权限   | `sudo strace -p 12345 -tt`        |

示例：在引导文件中根据环境切换配置

```php
<?php
declare(strict_types=1);

if (getenv('APP_ENV') === 'local') {
    ini_set('display_errors', '1');
    ini_set('xdebug.mode', 'debug,develop');
}
```

本章确保你能定位真正生效的配置，管理常用扩展，并熟练使用 Xdebug 完成交互式调试。建议将所有配置、安装命令与故障排查步骤记录在团队 Wiki，形成“环境基线”文档。
