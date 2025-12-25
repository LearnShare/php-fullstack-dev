# 2.2.3 调试函数和策略

## 目标

- 掌握 `var_dump`、`print_r`、`var_export`、`debug_backtrace` 等调试函数的用法。
- 理解开发环境与生产环境的调试策略差异。
- 学会使用 PSR-3 日志接口（Monolog）进行结构化日志记录。
- 构建完整的错误追踪与调试工作流。

## 基础调试函数

### `var_dump`

#### 语法

```php
var_dump(mixed ...$values): void
```

#### 参数说明

- `...$values`：可变参数，可传入多个值进行调试。
- 支持任意类型：标量、数组、对象、资源等。

#### 返回值

- 无返回值（`void`），直接输出到标准输出。

#### 输出特点

- 显示变量的类型、长度、层级结构。
- 递归显示数组和对象的完整内容。
- 输出格式详细但不易阅读。

#### 详细示例

```php
<?php
declare(strict_types=1);

// 标量类型
$name = "张三";
$age = 25;
$price = 99.99;
$active = true;

var_dump($name);   // string(6) "张三"
var_dump($age);    // int(25)
var_dump($price);  // float(99.99)
var_dump($active); // bool(true)

// 数组
$user = [
    'id' => 1,
    'name' => '张三',
    'tags' => ['php', 'mysql'],
];

var_dump($user);
// array(3) {
//   ["id"]=>
//   int(1)
//   ["name"]=>
//   string(6) "张三"
//   ["tags"]=>
//   array(2) {
//     [0]=>
//     string(3) "php"
//     [1]=>
//     string(5) "mysql"
//   }
// }

// 多个值
var_dump($name, $age, $price);
// string(6) "张三"
// int(25)
// float(99.99)

// 对象
class User
{
    public function __construct(
        private int $id,
        private string $name
    ) {}
}

$userObj = new User(1, '张三');
var_dump($userObj);
// object(User)#1 (2) {
//   ["id":"User":private]=>
//   int(1)
//   ["name":"User":private]=>
//   string(6) "张三"
// }

// NULL 值
$null = null;
var_dump($null);  // NULL
```

#### 使用场景

- 快速查看变量的完整结构和类型。
- 调试复杂数据结构。
- CLI 环境下的调试（Web 环境可能破坏 HTML 结构）。

#### 注意事项

- Web 环境中输出会破坏 HTML 结构，建议使用 `<pre>` 标签包裹。
- 输出内容较多时，建议使用 `ob_start()` 捕获后处理。

```php
// Web 环境安全使用
echo '<pre>';
var_dump($data);
echo '</pre>';
```

### `print_r`

#### 语法

```php
print_r(mixed $value, bool $return = false): string|bool
```

#### 参数说明

- `$value`：要输出的值（任意类型）。
- `$return`：如果为 `true`，返回字符串而不直接输出。

#### 返回值

- `$return = false`：返回 `true`（成功）。
- `$return = true`：返回格式化的字符串。

#### 输出特点

- 输出格式更简洁易读。
- 适合查看数组和对象的基本结构。
- 不显示类型信息。

#### 详细示例

```php
<?php
declare(strict_types=1);

$data = [
    'id' => 1,
    'name' => '张三',
    'tags' => ['php', 'mysql'],
];

// 直接输出
print_r($data);
// Array
// (
//     [id] => 1
//     [name] => 张三
//     [tags] => Array
//         (
//             [0] => php
//             [1] => mysql
//         )
// )

// 返回字符串
$output = print_r($data, true);
file_put_contents('debug.log', $output);

// 对象
class Product
{
    public function __construct(
        public int $id,
        public string $name,
        public float $price
    ) {}
}

$product = new Product(1, 'iPhone', 5999.00);
print_r($product);
// Product Object
// (
//     [id] => 1
//     [name] => iPhone
//     [price] => 5999
// )

// 与日志结合
$logMessage = print_r($data, true);
error_log($logMessage);
```

#### 使用场景

- 快速查看数组和对象结构。
- 需要将调试信息写入日志文件时。
- 需要格式化输出但不需要类型信息时。

### `var_export`

#### 语法

```php
var_export(mixed $value, bool $return = false): string|bool
```

#### 参数说明

- `$value`：要导出的值。
- `$return`：如果为 `true`，返回字符串而不直接输出。

#### 返回值

- `$return = false`：返回 `true`（成功）。
- `$return = true`：返回可执行的 PHP 代码字符串。

#### 输出特点

- 生成可直接 `eval()` 的 PHP 表达式。
- 适合生成配置文件、缓存数据。

#### 详细示例

```php
<?php
declare(strict_types=1);

$config = [
    'database' => [
        'host' => 'localhost',
        'port' => 3306,
    ],
    'cache' => [
        'enabled' => true,
        'ttl' => 3600,
    ],
];

// 生成 PHP 代码
$phpCode = var_export($config, true);
echo $phpCode;
// array (
//   'database' => 
//   array (
//     'host' => 'localhost',
//     'port' => 3306,
//   ),
//   'cache' => 
//   array (
//     'enabled' => true,
//     'ttl' => 3600,
//   ),
// )

// 保存为 PHP 文件
file_put_contents('config.php', "<?php\nreturn " . $phpCode . ";\n");

// 从文件加载
$loaded = include 'config.php';
print_r($loaded);

// 对象导出（需要实现 __set_state）
class Config
{
    public function __construct(
        public string $host,
        public int $port
    ) {}
    
    public static function __set_state(array $properties): self
    {
        return new self($properties['host'], $properties['port']);
    }
}

$configObj = new Config('localhost', 3306);
$exported = var_export($configObj, true);
echo $exported;
// Config::__set_state(array(
//    'host' => 'localhost',
//    'port' => 3306,
// ))
```

#### 使用场景

- 生成配置文件。
- 缓存复杂数据结构。
- 代码生成工具。

### `debug_backtrace`

#### 语法

```php
debug_backtrace(int $options = DEBUG_BACKTRACE_PROVIDE_OBJECT, int $limit = 0): array
```

#### 参数说明

- `$options`：控制返回信息的详细程度。
  - `DEBUG_BACKTRACE_PROVIDE_OBJECT`：包含对象信息。
  - `DEBUG_BACKTRACE_IGNORE_ARGS`：忽略函数参数（提升性能）。
  - `DEBUG_BACKTRACE_PROVIDE_OBJECT | DEBUG_BACKTRACE_IGNORE_ARGS`：组合使用。
- `$limit`：限制返回的调用栈层级，0 表示无限制。

#### 返回值

- 返回调用栈数组，每个元素包含文件、行号、函数名等信息。

#### 详细示例

```php
<?php
declare(strict_types=1);

function level1(): void
{
    level2();
}

function level2(): void
{
    level3();
}

function level3(): void
{
    $trace = debug_backtrace();
    print_r($trace);
}

level1();

// 输出调用栈：
// Array
// (
//     [0] => Array
//         (
//             [file] => /path/to/file.php
//             [line] => 15
//             [function] => level3
//         )
//     [1] => Array
//         (
//             [file] => /path/to/file.php
//             [line] => 10
//             [function] => level2
//         )
//     [2] => Array
//         (
//             [file] => /path/to/file.php
//             [line] => 5
//             [function] => level1
//         )
// )

// 忽略参数（提升性能）
$trace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS);
print_r($trace);

// 限制层级
$trace = debug_backtrace(DEBUG_BACKTRACE_PROVIDE_OBJECT, 2);
print_r($trace);  // 只返回最近 2 层

// 实际应用：记录错误位置
function logError(string $message): void
{
    $trace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 1);
    $caller = $trace[0] ?? [];
    
    error_log(sprintf(
        "%s in %s:%d",
        $message,
        $caller['file'] ?? 'unknown',
        $caller['line'] ?? 0
    ));
}

function processData(array $data): void
{
    if (empty($data)) {
        logError("Empty data provided");
        return;
    }
    // ...
}
```

#### 使用场景

- 追踪函数调用路径。
- 记录错误发生位置。
- 调试递归函数。
- 性能分析工具。

### `debug_zval_dump`

#### 语法

```php
debug_zval_dump(mixed ...$values): void
```

#### 参数说明

- `...$values`：可变参数，可传入多个值。

#### 返回值

- 无返回值，直接输出。

#### 输出特点

- 显示 zval 引用计数信息。
- 帮助理解 PHP 的引用机制和内存管理。

#### 详细示例

```php
<?php
declare(strict_types=1);

$a = "Hello";
debug_zval_dump($a);
// string(5) "Hello" refcount(2)

$b = $a;
debug_zval_dump($a, $b);
// string(5) "Hello" refcount(3)
// string(5) "Hello" refcount(3)

$c = &$a;
debug_zval_dump($a, $c);
// string(5) "Hello" refcount(2) is_ref(1)
// string(5) "Hello" refcount(2) is_ref(1)
```

#### 使用场景

- 学习 PHP 内部机制。
- 调试引用相关问题。
- 内存泄漏排查。

### `dump`（Symfony VarDumper）

#### 安装

```bash
composer require symfony/var-dumper
```

#### 语法

```php
dump(mixed ...$vars): void
```

#### 特点

- 更友好的 CLI 和 HTML 输出。
- 支持语法高亮。
- 可折叠的复杂结构。
- 自动检测运行环境。

#### 详细示例

```php
<?php
declare(strict_types=1);

require 'vendor/autoload.php';

$data = [
    'id' => 1,
    'name' => '张三',
    'tags' => ['php', 'mysql'],
];

dump($data);
// 输出格式化的、可折叠的结构

// 多个值
dump($data, $data['name'], $data['tags']);

// 对象
class User
{
    public function __construct(
        private int $id,
        private string $name
    ) {}
}

$user = new User(1, '张三');
dump($user);
```

## 高级调试工具

### Xdebug

Xdebug 是 PHP 最强大的调试工具，提供：

- 断点调试。
- 变量检查。
- 调用栈分析。
- 性能分析。

#### 安装

```bash
pecl install xdebug
```

#### 配置

```ini
zend_extension=xdebug
xdebug.mode=debug,develop
xdebug.start_with_request=yes
xdebug.client_host=127.0.0.1
xdebug.client_port=9003
```

#### IDE 配置

- PhpStorm：Preferences → PHP → Servers
- VS Code：安装 PHP Debug 扩展

### Monolog（PSR-3 日志）

#### 安装

```bash
composer require monolog/monolog
```

#### 基础使用

```php
<?php
declare(strict_types=1);

require 'vendor/autoload.php';

use Monolog\Logger;
use Monolog\Handler\StreamHandler;

// 创建日志实例
$logger = new Logger('app');
$logger->pushHandler(new StreamHandler('app.log', Logger::DEBUG));

// 不同级别日志
$logger->debug('调试信息', ['user_id' => 123]);
$logger->info('用户登录', ['user_id' => 123]);
$logger->warning('警告信息', ['order_id' => 456]);
$logger->error('错误信息', ['exception' => $e]);
$logger->critical('严重错误', ['system' => 'payment']);
```

#### 处理器类型

| 处理器 | 说明 | 使用场景 |
| :----- | :--- | :--- |
| `StreamHandler` | 写入文件 | 本地日志文件 |
| `RotatingFileHandler` | 按日期轮转 | 生产环境日志 |
| `SlackHandler` | 发送到 Slack | 错误通知 |
| `EmailHandler` | 发送邮件 | 严重错误通知 |
| `SyslogHandler` | 系统日志 | 服务器日志 |

#### 完整示例

```php
<?php
declare(strict_types=1);

require 'vendor/autoload.php';

use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Handler\RotatingFileHandler;
use Monolog\Formatter\JsonFormatter;

class AppLogger
{
    private Logger $logger;
    
    public function __construct()
    {
        $this->logger = new Logger('app');
        
        // 文件日志（按日期轮转）
        $fileHandler = new RotatingFileHandler(
            'logs/app.log',
            30,  // 保留 30 天
            Logger::DEBUG
        );
        $fileHandler->setFormatter(new JsonFormatter());
        $this->logger->pushHandler($fileHandler);
        
        // 错误日志（单独文件）
        $errorHandler = new StreamHandler('logs/error.log', Logger::ERROR);
        $errorHandler->setFormatter(new JsonFormatter());
        $this->logger->pushHandler($errorHandler);
    }
    
    public function log(string $level, string $message, array $context = []): void
    {
        $this->logger->log($level, $message, $context);
    }
}

$logger = new AppLogger();
$logger->log('info', '用户登录', [
    'user_id' => 123,
    'ip' => '192.168.1.1',
    'timestamp' => time(),
]);
```

## 调试策略

### 开发环境

#### 配置

```php
<?php
declare(strict_types=1);

// 显示所有错误
error_reporting(E_ALL);
ini_set('display_errors', '1');
ini_set('display_startup_errors', '1');

// 记录错误
ini_set('log_errors', '1');
ini_set('error_log', 'logs/php-errors.log');
```

#### 调试流程

1. **快速调试**：使用 `var_dump` 或 `dump` 查看变量。
2. **断点调试**：使用 Xdebug 在 IDE 中设置断点。
3. **日志记录**：使用 Monolog 记录关键步骤。
4. **清理代码**：提交前移除所有调试代码。

### 生产环境

#### 配置

```php
<?php
declare(strict_types=1);

// 隐藏错误显示
ini_set('display_errors', '0');
ini_set('display_startup_errors', '0');

// 记录所有错误
error_reporting(E_ALL);
ini_set('log_errors', '1');
ini_set('error_log', 'logs/php-errors.log');
```

#### 最佳实践

1. **结构化日志**：使用 JSON 格式记录上下文信息。
2. **错误通知**：严重错误发送到 Slack/Email。
3. **敏感数据脱敏**：日志中不记录密码、token 等。
4. **性能监控**：记录慢查询、慢请求。

### 调试工作流示例

```php
<?php
declare(strict_types=1);

require 'vendor/autoload.php';

use Monolog\Logger;
use Monolog\Handler\StreamHandler;

class DebugHelper
{
    private Logger $logger;
    private bool $isDevelopment;
    
    public function __construct(bool $isDevelopment = false)
    {
        $this->isDevelopment = $isDevelopment;
        $this->logger = new Logger('app');
        $this->logger->pushHandler(
            new StreamHandler('logs/app.log', Logger::DEBUG)
        );
    }
    
    public function dump(mixed $value, string $label = ''): void
    {
        if ($this->isDevelopment) {
            if ($label) {
                echo "<strong>$label:</strong><br>";
            }
            echo '<pre>';
            var_dump($value);
            echo '</pre>';
        } else {
            $this->logger->debug($label, ['value' => $value]);
        }
    }
    
    public function trace(string $message): void
    {
        $trace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 5);
        $this->logger->debug($message, ['trace' => $trace]);
    }
    
    public function log(string $level, string $message, array $context = []): void
    {
        $this->logger->log($level, $message, $context);
    }
}

// 使用
$debug = new DebugHelper(true);  // 开发环境
$debug->dump($user, 'User Data');
$debug->trace('Function called');
$debug->log('info', 'User login', ['user_id' => 123]);
```

## 常见错误与注意事项

1. **调试代码泄漏**：提交前检查并移除所有 `var_dump`、`dd()` 等。

```php
// 使用 CI 检查
// .github/workflows/lint.yml
- name: Check for debug code
  run: |
    if grep -r "var_dump\|dd()\|dump(" src/; then
      echo "Found debug code!"
      exit 1
    fi
```

2. **敏感信息泄露**：日志中不记录密码、token、信用卡号等。

```php
// 错误示例
$logger->info('User login', [
    'password' => $password,  // 危险！
    'token' => $token,       // 危险！
]);

// 正确示例
$logger->info('User login', [
    'user_id' => $userId,
    'ip' => $ip,
]);
```

3. **性能影响**：生产环境避免使用 `var_dump`、`debug_backtrace` 等。

```php
// 条件调试
if (getenv('APP_DEBUG') === 'true') {
    var_dump($data);
}
```

## 练习

1. 编写一个调试工具类，根据环境（开发/生产）选择不同的输出方式。

2. 使用 Monolog 实现一个日志系统，支持文件日志、错误通知和日志轮转。

3. 实现一个函数，使用 `debug_backtrace` 记录函数调用路径，并写入日志。

4. 创建一个调试中间件，在开发环境下自动捕获并格式化输出错误信息。

5. 使用 `var_export` 实现一个配置缓存系统，将配置数组保存为 PHP 文件。
