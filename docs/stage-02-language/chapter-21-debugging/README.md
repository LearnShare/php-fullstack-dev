# 2.21 常见错误与调试技巧

## 目标

- 识别和解决常见的 PHP 错误。
- 掌握有效的调试方法和工具。
- 理解错误信息的含义。
- 能够快速定位和修复问题。

## 常见错误类型

### 语法错误（Parse Error）

#### 错误示例

```php
<?php
// 错误：缺少分号
$name = "Alice"
echo $name;

// 错误：未闭合的括号
if ($condition {
    // ...
}

// 错误：未闭合的引号
$text = "Hello World;
```

#### 解决方法

```bash
# 使用 PHP 语法检查
php -l file.php

# 输出示例：
# PHP Parse error: syntax error, unexpected 'echo' (T_ECHO) in file.php on line 2
```

### 类型错误（Type Error）

#### 错误示例

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

// 错误：传递了字符串
add("5", 3); // TypeError: Argument #1 must be of type int, string given
```

#### 解决方法

```php
<?php
declare(strict_types=1);

// 方法 1：类型转换
add((int) "5", 3);

// 方法 2：验证输入
function safeAdd(mixed $a, mixed $b): int
{
    if (!is_int($a) || !is_int($b)) {
        throw new TypeError('Arguments must be integers');
    }
    return $a + $b;
}
```

### 未定义变量（Undefined Variable）

#### 错误示例

```php
<?php
// 错误：使用未定义的变量
echo $undefinedVariable; // Warning: Undefined variable $undefinedVariable
```

#### 解决方法

```php
<?php
// 方法 1：初始化变量
$variable = null;
echo $variable ?? 'default';

// 方法 2：检查变量是否存在
if (isset($variable)) {
    echo $variable;
}

// 方法 3：使用空合并运算符
echo $variable ?? 'default';
```

### 未定义索引（Undefined Index）

#### 错误示例

```php
<?php
// 错误：访问不存在的数组索引
$data = ['name' => 'Alice'];
echo $data['email']; // Warning: Undefined array key "email"
```

#### 解决方法

```php
<?php
// 方法 1：使用 isset 检查
if (isset($data['email'])) {
    echo $data['email'];
}

// 方法 2：使用空合并运算符
echo $data['email'] ?? 'N/A';

// 方法 3：使用 array_key_exists
if (array_key_exists('email', $data)) {
    echo $data['email'];
}
```

### 调用未定义函数（Call to Undefined Function）

#### 错误示例

```php
<?php
// 错误：调用不存在的函数
unknownFunction(); // Fatal error: Uncaught Error: Call to undefined function unknownFunction()
```

#### 解决方法

```php
<?php
// 方法 1：检查函数是否存在
if (function_exists('unknownFunction')) {
    unknownFunction();
}

// 方法 2：使用 is_callable
if (is_callable('unknownFunction')) {
    unknownFunction();
}
```

### 内存耗尽（Memory Exhausted）

#### 错误示例

```php
<?php
// 错误：无限循环或大量数据处理
$data = [];
while (true) {
    $data[] = str_repeat('x', 1000000);
}
// Fatal error: Allowed memory size of 134217728 bytes exhausted
```

#### 解决方法

```php
<?php
// 方法 1：增加内存限制
ini_set('memory_limit', '512M');

// 方法 2：使用生成器处理大量数据
function generateData(): Generator
{
    for ($i = 0; $i < 1000000; $i++) {
        yield str_repeat('x', 1000);
    }
}

// 方法 3：分批处理
$batchSize = 1000;
$offset = 0;
while ($batch = getBatch($offset, $batchSize)) {
    processBatch($batch);
    $offset += $batchSize;
}
```

## 调试技巧

### 1. 错误报告配置

```php
<?php
// 开发环境
error_reporting(E_ALL);
ini_set('display_errors', '1');
ini_set('display_startup_errors', '1');

// 生产环境
error_reporting(E_ALL & ~E_DEPRECATED & ~E_STRICT);
ini_set('display_errors', '0');
ini_set('log_errors', '1');
ini_set('error_log', '/var/log/php/errors.log');
```

### 2. 异常处理

```php
<?php
declare(strict_types=1);

// 全局异常处理
set_exception_handler(function (Throwable $e): void {
    error_log(sprintf(
        "Uncaught exception: %s in %s:%d\nStack trace:\n%s",
        $e->getMessage(),
        $e->getFile(),
        $e->getLine(),
        $e->getTraceAsString()
    ));
    
    if (getenv('APP_ENV') === 'development') {
        echo "<pre>{$e}</pre>";
    } else {
        http_response_code(500);
        echo json_encode(['error' => 'Internal server error']);
    }
});

// 错误处理
set_error_handler(function (int $severity, string $message, string $file, int $line): bool {
    if (!(error_reporting() & $severity)) {
        return false;
    }
    
    throw new ErrorException($message, 0, $severity, $file, $line);
});
```

### 3. 日志调试

```php
<?php
declare(strict_types=1);

class DebugLogger
{
    private string $logFile;
    
    public function __construct(string $logFile = '/tmp/debug.log')
    {
        $this->logFile = $logFile;
    }
    
    public function log(string $message, array $context = []): void
    {
        $timestamp = date('Y-m-d H:i:s');
        $contextStr = !empty($context) ? ' ' . json_encode($context) : '';
        $logMessage = "[{$timestamp}] {$message}{$contextStr}\n";
        
        file_put_contents($this->logFile, $logMessage, FILE_APPEND);
    }
    
    public function dump(mixed $variable, string $label = 'DEBUG'): void
    {
        $this->log("{$label}: " . print_r($variable, true));
    }
}

// 使用
$logger = new DebugLogger();
$logger->log('User logged in', ['user_id' => 1, 'ip' => '192.168.1.1']);
$logger->dump($complexObject, 'User Object');
```

### 4. 性能调试

```php
<?php
declare(strict_types=1);

class PerformanceProfiler
{
    private array $timers = [];
    
    public function start(string $label): void
    {
        $this->timers[$label] = [
            'start' => microtime(true),
            'memory' => memory_get_usage(),
        ];
    }
    
    public function end(string $label): array
    {
        if (!isset($this->timers[$label])) {
            throw new InvalidArgumentException("Timer {$label} not started");
        }
        
        $timer = $this->timers[$label];
        $endTime = microtime(true);
        $endMemory = memory_get_usage();
        
        return [
            'time' => $endTime - $timer['start'],
            'memory' => $endMemory - $timer['memory'],
            'peak_memory' => memory_get_peak_usage(),
        ];
    }
    
    public function report(): void
    {
        foreach ($this->timers as $label => $timer) {
            $result = $this->end($label);
            echo sprintf(
                "%s: %.4f seconds, %d bytes\n",
                $label,
                $result['time'],
                $result['memory']
            );
        }
    }
}

// 使用
$profiler = new PerformanceProfiler();
$profiler->start('database_query');
// ... 执行数据库查询
$result = $profiler->end('database_query');
echo "Query took: {$result['time']} seconds\n";
```

### 5. 数据库查询调试

```php
<?php
declare(strict_types=1);

class QueryLogger
{
    private array $queries = [];
    
    public function log(string $sql, array $params = [], float $time = 0): void
    {
        $this->queries[] = [
            'sql' => $sql,
            'params' => $params,
            'time' => $time,
            'trace' => debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 5),
        ];
    }
    
    public function getQueries(): array
    {
        return $this->queries;
    }
    
    public function getTotalTime(): float
    {
        return array_sum(array_column($this->queries, 'time'));
    }
    
    public function report(): void
    {
        echo "Total queries: " . count($this->queries) . "\n";
        echo "Total time: " . $this->getTotalTime() . " seconds\n";
        
        foreach ($this->queries as $i => $query) {
            echo sprintf(
                "\nQuery %d (%.4fs):\n%s\nParams: %s\n",
                $i + 1,
                $query['time'],
                $query['sql'],
                json_encode($query['params'])
            );
        }
    }
}
```

## 调试工具

### 1. Xdebug

```php
<?php
// 强制断点
xdebug_break();

// 获取堆栈跟踪
$trace = xdebug_get_function_stack();

// 获取变量信息
xdebug_var_dump($variable);
```

### 2. Symfony VarDumper

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use Symfony\Component\VarDumper\VarDumper;

// 更美观的变量输出
VarDumper::dump($variable);

// 或使用全局函数
dump($variable);
```

### 3. Whoops（错误页面）

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use Whoops\Run;
use Whoops\Handler\PrettyPageHandler;

$whoops = new Run();
$whoops->pushHandler(new PrettyPageHandler());
$whoops->register();
```

## 常见问题排查清单

### 1. 代码不执行

- [ ] 检查 PHP 标签是否正确
- [ ] 检查文件是否被包含
- [ ] 检查条件判断逻辑
- [ ] 检查错误是否被抑制（@）

### 2. 变量值为空

- [ ] 检查变量是否初始化
- [ ] 检查作用域（局部/全局）
- [ ] 检查是否被 unset
- [ ] 检查类型转换

### 3. 数据库查询失败

- [ ] 检查连接是否建立
- [ ] 检查 SQL 语法
- [ ] 检查参数绑定
- [ ] 检查权限
- [ ] 查看数据库错误日志

### 4. 性能问题

- [ ] 检查 N+1 查询
- [ ] 检查循环嵌套
- [ ] 检查内存使用
- [ ] 使用性能分析工具

## 最佳实践

### 1. 防御性编程

```php
<?php
declare(strict_types=1);

// 总是验证输入
function processUser(array $data): User
{
    if (empty($data['name'])) {
        throw new InvalidArgumentException('Name is required');
    }
    
    if (!filter_var($data['email'] ?? '', FILTER_VALIDATE_EMAIL)) {
        throw new InvalidArgumentException('Invalid email');
    }
    
    return new User($data);
}
```

### 2. 详细的错误信息

```php
<?php
declare(strict_types=1);

try {
    riskyOperation();
} catch (Exception $e) {
    error_log(sprintf(
        'Error in %s:%d: %s\nStack trace:\n%s',
        $e->getFile(),
        $e->getLine(),
        $e->getMessage(),
        $e->getTraceAsString()
    ));
    throw $e;
}
```

### 3. 使用类型提示

```php
<?php
declare(strict_types=1);

// 类型提示可以帮助发现错误
function calculateTotal(array $items): float
{
    $total = 0.0;
    foreach ($items as $item) {
        // 如果 $item 不是数组，会立即报错
        $total += $item['price'] * $item['quantity'];
    }
    return $total;
}
```

## 练习

1. 创建一个错误处理类，统一处理所有错误和异常。

2. 实现一个查询日志系统，记录所有数据库查询及其执行时间。

3. 创建一个性能分析工具，测量代码执行时间和内存使用。

4. 编写一个调试辅助函数库，包含常用的调试方法。

5. 实现一个错误报告系统，自动收集和报告错误信息。

6. 创建一个调试检查清单，用于排查常见问题。

