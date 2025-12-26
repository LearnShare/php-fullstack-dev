# 2.1.4 文件执行方式

## 概述

PHP 程序可以通过不同的方式执行，每种方式有不同的执行环境和特点。本节详细介绍 CLI 模式、Web 模式、执行环境差异，以及如何检测当前执行模式，帮助理解不同执行方式的区别和适用场景。

理解不同执行方式的差异对于选择合适的开发方式和部署方案至关重要。CLI 模式适合脚本和命令行工具，Web 模式适合 Web 应用开发。

## 特性

- **多种执行方式**：支持 CLI（命令行）和 Web 两种主要执行方式
- **环境差异**：不同执行方式有不同的执行环境和特点
- **模式检测**：可以使用 `php_sapi_name()` 函数检测当前执行模式
- **灵活选择**：可以根据场景选择合适的执行方式

## 核心内容

### CLI 模式（命令行）

**特点**：
- 直接执行 PHP 脚本，不依赖 Web 服务器
- 输出直接显示在终端/命令行窗口中
- 适合脚本、迁移、队列任务、命令行工具
- 单次执行，脚本结束后进程退出
- 使用系统环境变量

**使用方式**：

```bash
# 执行脚本文件
php script.php

# 执行单行代码
php -r "echo 'Hello, World!';"

# 启动内置服务器
php -S localhost:8000
```

**详细说明**：见 [1.2.2 运行模式与验证](../../stage-01-foundation/chapter-02-runtime/section-02-execution-modes.md)

### Web 模式

**特点**：
- 通过 Web 服务器执行（Apache、Nginx、PHP-FPM 等）
- 输出发送到浏览器，作为 HTTP 响应
- 适合 Web 应用、API 服务
- 每次请求独立执行
- 使用 Web 服务器配置的环境变量

**使用方式**：
- 通过浏览器访问 PHP 文件：`http://localhost/file.php`
- 需要 Web 服务器支持 PHP
- 需要正确配置 Web 服务器

**详细说明**：见 [1.2.2 运行模式与验证](../../stage-01-foundation/chapter-02-runtime/section-02-execution-modes.md)

### 执行环境差异

#### 环境变量

**CLI 模式**：
- 使用系统环境变量
- 可以通过 `$_ENV` 或 `getenv()` 访问
- 环境变量在脚本执行期间保持不变

**Web 模式**：
- 使用 Web 服务器配置的环境变量
- 可以通过 `$_SERVER`、`$_ENV` 或 `getenv()` 访问
- 环境变量可能因请求而异

#### 输出方式

**CLI 模式**：
- 输出到标准输出（stdout）或标准错误（stderr）
- 支持换行符 `\n`、制表符 `\t` 等转义字符
- 支持 ANSI 转义码（颜色、格式等）

**Web 模式**：
- 输出到 HTTP 响应体
- 换行符 `\n` 在 HTML 中无效，需要使用 `<br>` 标签
- 需要设置正确的 HTTP 头（Content-Type 等）

#### 错误处理

**CLI 模式**：
- 错误直接显示在终端
- 可以使用 `error_log()` 写入日志文件
- 退出码（exit code）有意义

**Web 模式**：
- 错误可能显示在浏览器或日志文件中
- 根据配置，错误可能被隐藏或记录
- 退出码通常不重要

#### 输入方式

**CLI 模式**：
- 可以从标准输入（stdin）读取
- 可以读取命令行参数（`$argv`、`$argc`）
- 可以交互式输入

**Web 模式**：
- 从 HTTP 请求获取输入
- 通过 `$_GET`、`$_POST`、`$_REQUEST` 等超全局变量访问
- 不能直接读取标准输入

## 检测执行模式

### php_sapi_name() - 获取 SAPI 名称

**语法**：`php_sapi_name(): string`

**参数**：无参数

**返回值**：返回当前 PHP 运行环境的 SAPI（Server API）名称，类型为 `string`。常见返回值：
- `'cli'`：命令行模式
- `'cli-server'`：PHP 内置服务器
- `'fpm-fcgi'`：PHP-FPM
- `'apache2handler'`：Apache 模块
- `'cgi-fcgi'`：CGI/FastCGI
- 其他：其他 SAPI 名称

**示例**：

```php
<?php
declare(strict_types=1);

$sapi = php_sapi_name();

echo "Current SAPI: {$sapi}\n";

if ($sapi === 'cli') {
    echo "Running in CLI mode\n";
} else {
    echo "Running in Web mode: {$sapi}\n";
}
```

**执行**（CLI 模式）：

```bash
php check-mode.php
```

**输出**：

```
Current SAPI: cli
Running in CLI mode
```

**执行**（Web 模式）：

通过浏览器访问，输出：

```
Current SAPI: fpm-fcgi
Running in Web mode: fpm-fcgi
```

**使用场景**：
- 检测当前运行环境，根据环境执行不同逻辑
- 调试和日志记录，记录运行环境信息
- 条件执行，在 CLI 和 Web 模式下执行不同代码

**注意事项**：
- 返回值是字符串，比较时应使用严格比较（`===`）
- 不同运行环境的返回值可能不同，应使用常量或配置管理
- 详细说明见 [1.2.2 运行模式与验证](../../stage-01-foundation/chapter-02-runtime/section-02-execution-modes.md)

### 检测函数示例

**示例 1：简单的模式检测**

```php
<?php
declare(strict_types=1);

function isCli(): bool
{
    return php_sapi_name() === 'cli';
}

if (isCli()) {
    echo "CLI mode\n";
} else {
    echo "Web mode\n";
}
```

**示例 2：根据模式执行不同逻辑**

```php
<?php
declare(strict_types=1);

function getOutputNewline(): string
{
    return php_sapi_name() === 'cli' ? "\n" : "<br>\n";
}

$newline = getOutputNewline();
echo "Line 1{$newline}Line 2{$newline}Line 3{$newline}";
```

**示例 3：详细的模式检测**

```php
<?php
declare(strict_types=1);

function getExecutionMode(): string
{
    $sapi = php_sapi_name();
    
    return match ($sapi) {
        'cli' => 'CLI Mode',
        'cli-server' => 'Built-in Server',
        'fpm-fcgi' => 'PHP-FPM',
        'apache2handler' => 'Apache Module',
        default => "Unknown: {$sapi}",
    };
}

echo "Execution mode: " . getExecutionMode() . "\n";
```

## 完整代码示例

### 示例 1：CLI 模式示例

**文件**：`cli-example.php`

```php
<?php
declare(strict_types=1);

// CLI 模式：输出到终端
echo "Hello from CLI!\n";
echo "Current time: " . date('Y-m-d H:i:s') . "\n";

// 读取命令行参数
if (isset($argv[1])) {
    echo "Argument: {$argv[1]}\n";
}

// 使用颜色输出（ANSI 转义码）
echo "\033[32mGreen text\033[0m\n";
echo "\033[31mRed text\033[0m\n";
```

**执行**：

```bash
php cli-example.php "test"
```

**输出**：

```
Hello from CLI!
Current time: 2024-01-15 10:30:00
Argument: test
Green text
Red text
```

### 示例 2：Web 模式示例

**文件**：`web-example.php`

```php
<?php
declare(strict_types=1);

// Web 模式：设置 HTTP 头
header('Content-Type: text/html; charset=UTF-8');

// 输出 HTML
echo "<!DOCTYPE html>\n";
echo "<html>\n";
echo "<head><title>Web Example</title></head>\n";
echo "<body>\n";
echo "<h1>Hello from Web!</h1>\n";
echo "<p>Current time: " . date('Y-m-d H:i:s') . "</p>\n";
echo "</body>\n";
echo "</html>\n";
```

**访问**：`http://localhost/web-example.php`

**输出**（浏览器显示）：

```html
<!DOCTYPE html>
<html>
<head><title>Web Example</title></head>
<body>
<h1>Hello from Web!</h1>
<p>Current time: 2024-01-15 10:30:00</p>
</body>
</html>
```

### 示例 3：兼容两种模式的代码

**文件**：`compatible-example.php`

```php
<?php
declare(strict_types=1);

function isCliMode(): bool
{
    return php_sapi_name() === 'cli';
}

// 根据模式选择输出方式
if (isCliMode()) {
    // CLI 模式：使用换行符
    $newline = "\n";
    echo "Running in CLI mode{$newline}";
} else {
    // Web 模式：设置 HTTP 头，使用 HTML 标签
    header('Content-Type: text/html; charset=UTF-8');
    $newline = "<br>\n";
    echo "<!DOCTYPE html><html><body>";
    echo "Running in Web mode{$newline}";
}

echo "Current time: " . date('Y-m-d H:i:s') . $newline;

if (!isCliMode()) {
    echo "</body></html>";
}
```

**执行**（CLI 模式）：

```bash
php compatible-example.php
```

**输出**：

```
Running in CLI mode
Current time: 2024-01-15 10:30:00
```

**访问**（Web 模式）：`http://localhost/compatible-example.php`

**输出**（浏览器显示）：

```html
<!DOCTYPE html><html><body>Running in Web mode<br>
Current time: 2024-01-15 10:30:00<br>
</body></html>
```

## 使用场景

### CLI 模式适用场景

- **命令行工具**：开发命令行工具和脚本
- **数据迁移**：执行数据库迁移、数据导入导出
- **定时任务**：执行定时任务、计划任务（cron）
- **队列处理**：处理后台任务、队列任务
- **系统管理**：系统管理脚本、自动化任务

### Web 模式适用场景

- **Web 应用**：开发 Web 应用、网站
- **API 服务**：提供 RESTful API、GraphQL API
- **动态内容**：生成动态 HTML 内容
- **表单处理**：处理表单提交、文件上传

### 选择建议

- **学习阶段**：建议使用 CLI 模式，输出更直观，便于调试
- **Web 开发**：使用 Web 模式，模拟实际运行环境
- **工具开发**：使用 CLI 模式，适合命令行工具
- **混合场景**：可以开发兼容两种模式的代码

## 注意事项

### 环境差异

- **环境变量**：CLI 和 Web 模式的环境变量可能不同
- **输出格式**：CLI 使用换行符，Web 使用 HTML 标签
- **错误处理**：两种模式的错误处理方式不同
- **输入方式**：CLI 从命令行读取，Web 从 HTTP 请求读取

### 模式检测

- **使用严格比较**：`php_sapi_name()` 返回字符串，应使用 `===` 比较
- **处理未知模式**：代码应能处理未知的 SAPI 名称
- **不要硬编码**：不要硬编码特定的 SAPI 名称，使用常量或配置

### 代码兼容性

- **避免混合使用**：尽量避免在同一个文件中混合 CLI 和 Web 逻辑
- **使用抽象层**：可以创建抽象层，统一处理不同模式的差异
- **测试两种模式**：确保代码在两种模式下都能正常工作

## 常见问题

### 问题 1：Web 模式下换行符无效

**症状**：在 Web 模式下，`\n` 换行符在浏览器中不显示换行

**原因**：HTML 中换行符会被忽略，需要使用 HTML 标签

**解决方法**：

```php
<?php
declare(strict_types=1);

if (php_sapi_name() === 'cli') {
    $newline = "\n";
} else {
    $newline = "<br>\n";
}

echo "Line 1{$newline}Line 2{$newline}";
```

### 问题 2：环境变量不一致

**症状**：CLI 和 Web 模式下环境变量值不同

**原因**：两种模式使用不同的环境变量源

**解决方法**：

1. **统一配置**：使用配置文件统一管理环境变量
2. **检测模式**：根据执行模式使用不同的配置
3. **使用 .env 文件**：使用 `.env` 文件管理环境变量

### 问题 3：HTTP 头设置失败

**症状**：在 Web 模式下，`header()` 函数报错

**原因**：在设置 HTTP 头之前已有输出（包括空格、BOM 等）

**解决方法**：

1. **检查文件编码**：确保文件使用 UTF-8 无 BOM 编码
2. **检查输出**：确保在 `header()` 之前没有任何输出
3. **使用输出缓冲**：使用 `ob_start()` 开启输出缓冲

**详细说明**：见 [2.1.2 文件结构与编码](section-02-file-structure.md)

## 最佳实践

### 模式检测

- **使用函数封装**：创建 `isCli()` 等函数封装模式检测逻辑
- **使用常量**：定义常量存储执行模式信息
- **统一处理**：在应用入口统一检测模式，设置相应配置

### 代码组织

- **分离 CLI 和 Web 代码**：尽量将 CLI 和 Web 代码分离到不同文件
- **使用抽象层**：创建抽象层统一处理不同模式的差异
- **配置管理**：使用配置文件管理不同模式的设置

### 开发流程

- **先 CLI 后 Web**：建议先在 CLI 模式下开发和测试，再在 Web 模式下验证
- **测试两种模式**：确保代码在两种模式下都能正常工作
- **文档说明**：在代码注释中说明适用的执行模式

## 对比分析

### CLI vs Web 模式对比

| 特性 | CLI 模式 | Web 模式 |
|:-----|:--------|:---------|
| 执行环境 | 命令行 | Web 服务器 |
| 输出目标 | 终端 | 浏览器 |
| 换行符 | `\n` 有效 | `\n` 无效，需使用 `<br>` |
| 环境变量 | 系统环境变量 | Web 服务器配置 |
| 输入方式 | 命令行参数、stdin | HTTP 请求 |
| 错误显示 | 终端 | 浏览器或日志 |
| 适用场景 | 脚本、工具 | Web 应用、API |
| 退出码 | 有意义 | 通常不重要 |

**选择建议**：
- **学习阶段**：使用 CLI 模式，输出更直观
- **Web 开发**：使用 Web 模式，模拟实际环境
- **工具开发**：使用 CLI 模式
- **混合场景**：开发兼容两种模式的代码

## 相关章节

- **2.1.1 第一个 PHP 程序：Hello World**：了解 PHP 程序的基本执行方式
- **2.1.2 文件结构与编码**：了解文件结构对执行的影响
- **1.2.2 运行模式与验证**：详细了解不同运行模式的配置和使用
- **阶段四：系统编程**：深入学习 CLI 编程和系统交互
- **阶段五：Web/API 开发**：深入学习 Web 开发和 API 开发

## 练习任务

1. **模式检测练习**：
   - 创建一个脚本，使用 `php_sapi_name()` 检测当前执行模式
   - 在 CLI 和 Web 两种模式下执行，观察输出差异
   - 根据模式执行不同的逻辑

2. **兼容性练习**：
   - 创建一个兼容 CLI 和 Web 两种模式的脚本
   - 在 CLI 模式下使用换行符，在 Web 模式下使用 HTML 标签
   - 确保两种模式下都能正常显示

3. **环境变量练习**：
   - 在 CLI 和 Web 模式下分别读取环境变量
   - 观察两种模式下环境变量的差异
   - 使用配置文件统一管理环境变量

4. **输出格式练习**：
   - 创建一个脚本，在 CLI 模式下输出带颜色的文本
   - 在 Web 模式下输出格式化的 HTML
   - 确保两种模式下输出都正确

5. **综合练习**：
   - 创建一个完整的应用，支持 CLI 和 Web 两种模式
   - 实现模式检测、环境配置、输出格式化等功能
   - 测试两种模式下的功能是否正常
