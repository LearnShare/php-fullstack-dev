# 2.3.5 最佳实践与综合练习

## 概述

本章节总结了变量与常量使用的最佳实践，并提供了综合性的练习题目，帮助巩固所学知识。遵循这些最佳实践可以提高代码质量、可维护性和性能。

## 最佳实践

### 1. 严格类型声明

始终在文件开头使用 `declare(strict_types=1);`，这可以：

- 减少隐式类型转换带来的混乱
- 提高代码可读性
- 在开发阶段捕获类型错误
- 提高性能（PHP 7.0+）

```php
<?php
declare(strict_types=1);

function calculateTotal(float $price, int $quantity): float
{
    return $price * $quantity;
}

// 在严格模式下，类型不匹配会报错
// calculateTotal("10.5", 2);  // 类型错误
calculateTotal(10.5, 2);  // 正确：21.0
```

### 2. 明确的变量命名

使用清晰、描述性的变量名，遵循一致的命名规范：

```php
<?php
declare(strict_types=1);

// 推荐：清晰、描述性
$userName        = 'Alice';
$orderTotal      = 99.99;
$isAuthenticated = true;
$maxRetryCount   = 3;

// 不推荐：模糊、缩写
$un = 'Alice';           // 不清楚
$ot = 99.99;            // 不清楚
$auth = true;           // 不够明确
$mrc = 3;               // 不清楚
```

### 3. 类型提示的使用

在函数参数和返回值中使用类型提示：

```php
<?php
declare(strict_types=1);

// 推荐：使用类型提示
function processUser(string $name, int $age, bool $isActive): array
{
    return [
        'name' => $name,
        'age' => $age,
        'is_active' => $isActive
    ];
}

// PHP 7.4+ 支持属性类型声明
class User
{
    public string $name;
    public int $age;
    public bool $isActive;
    
    public function __construct(string $name, int $age, bool $isActive)
    {
        $this->name = $name;
        $this->age = $age;
        $this->isActive = $isActive;
    }
}
```

### 4. 避免使用全局变量

优先使用函数参数、配置类或依赖注入：

```php
<?php
declare(strict_types=1);

// 不推荐：使用全局变量
$config = ['debug' => true];

function logMessage(string $message): void
{
    global $config;
    if ($config['debug']) {
        echo $message . "\n";
    }
}

// 推荐：通过参数传递
function logMessageBetter(string $message, array $config): void
{
    if ($config['debug'] ?? false) {
        echo $message . "\n";
    }
}

// 推荐：使用配置类
class Config
{
    private static array $settings = [];
    
    public static function set(string $key, mixed $value): void
    {
        self::$settings[$key] = $value;
    }
    
    public static function get(string $key, mixed $default = null): mixed
    {
        return self::$settings[$key] ?? $default;
    }
}
```

### 5. 谨慎使用引用

只在确实需要修改外部变量时使用引用：

```php
<?php
declare(strict_types=1);

// 合理使用：交换变量
function swap(&$a, &$b): void
{
    $temp = $a;
    $a = $b;
    $b = $temp;
}

// 合理使用：在循环中修改数组
function processItems(array &$items): void
{
    foreach ($items as &$item) {
        $item['processed'] = true;
    }
    unset($item);  // 重要：取消引用
}

// 不推荐：不必要的引用
function addOne(int &$value): void
{
    $value++;  // 如果不需要修改原值，应该返回值
}

// 推荐：返回值而不是引用
function addOneBetter(int $value): int
{
    return $value + 1;
}
```

### 6. 避免可变变量

优先使用数组或对象属性：

```php
<?php
declare(strict_types=1);

// 不推荐：使用可变变量
$varName = 'userName';
$userName = 'Alice';
echo $$varName . "\n";

// 推荐：使用数组
$data = ['userName' => 'Alice'];
echo $data['userName'] . "\n";

// 推荐：使用对象属性
class User
{
    public string $userName;
}

$user = new User();
$user->userName = 'Alice';
echo $user->userName . "\n";
```

### 7. 常量命名规范

使用全大写字母和下划线分隔：

```php
<?php
declare(strict_types=1);

// 推荐
const MAX_FILE_SIZE = 10485760;
const DEFAULT_TIMEOUT = 30;
const API_BASE_URL = 'https://api.example.com';

// 不推荐
const maxFileSize = 10485760;
const defaultTimeout = 30;
const apiBaseUrl = 'https://api.example.com';
```

### 8. 优先使用 const 而非 define

对于文件级别的常量，优先使用 `const`（性能更好，语法更简洁）：

```php
<?php
declare(strict_types=1);

// 推荐：使用 const
const APP_NAME = 'My Application';
const APP_VERSION = '1.0.0';

// 仅在需要条件定义时使用 define
if (php_sapi_name() === 'cli') {
    define('ENVIRONMENT', 'development');
} else {
    define('ENVIRONMENT', 'production');
}
```

### 9. 变量存在性检查

始终检查变量是否存在，避免未定义变量警告：

```php
<?php
declare(strict_types=1);

// 推荐：检查后再使用
if (isset($user['name'])) {
    echo $user['name'];
}

// PHP 7.0+ 推荐：使用 null 合并运算符
$name = $user['name'] ?? 'Unknown';

// PHP 7.4+ 推荐：使用 null 合并赋值运算符
$user['name'] ??= 'Unknown';
```

### 10. 静态变量的合理使用

静态变量适用于函数内部状态保持，但要避免过度使用：

```php
<?php
declare(strict_types=1);

// 合理使用：计数器
function getNextId(): int
{
    static $id = 0;
    return ++$id;
}

// 合理使用：缓存
function loadConfig(): array
{
    static $config = null;
    
    if ($config === null) {
        $config = parse_ini_file('config.ini');
    }
    
    return $config;
}

// 不推荐：过度使用静态变量
function complexFunction(): void
{
    static $state1 = null;
    static $state2 = null;
    static $state3 = null;
    // ... 太多静态变量，应该使用类
}
```

## 综合练习

### 练习 1：实现 Counter 类

创建一个 `Counter` 类，使用静态属性记录实例数量，并提供 `getCount()` 方法。

**要求**：
- 使用静态属性 `$instanceCount` 记录创建的实例数量
- 每次创建新实例时，计数器自动递增
- 提供静态方法 `getCount()` 返回当前实例数量
- 提供实例方法 `getId()` 返回当前实例的唯一 ID

**参考实现**：

```php
<?php
declare(strict_types=1);

class Counter
{
    private static int $instanceCount = 0;
    private int $id;
    
    public function __construct()
    {
        self::$instanceCount++;
        $this->id = self::$instanceCount;
    }
    
    public static function getCount(): int
    {
        return self::$instanceCount;
    }
    
    public function getId(): int
    {
        return $this->id;
    }
}

// 测试
$counter1 = new Counter();
echo "Instance count: " . Counter::getCount() . "\n";  // 1
echo "Counter1 ID: " . $counter1->getId() . "\n";     // 1

$counter2 = new Counter();
echo "Instance count: " . Counter::getCount() . "\n";  // 2
echo "Counter2 ID: " . $counter2->getId() . "\n";     // 2

$counter3 = new Counter();
echo "Instance count: " . Counter::getCount() . "\n";  // 3
echo "Counter3 ID: " . $counter3->getId() . "\n";     // 3
```

### 练习 2：实现 swap 函数

创建一个函数 `swap(&$a, &$b)`，通过引用交换两个变量的值。

**要求**：
- 使用引用参数
- 支持不同类型的变量（int、string、array 等）
- 添加类型提示（PHP 8.0+ 可以使用 `mixed` 类型）

**参考实现**：

```php
<?php
declare(strict_types=1);

function swap(mixed &$a, mixed &$b): void
{
    $temp = $a;
    $a = $b;
    $b = $temp;
}

// 测试整数
$x = 10;
$y = 20;
swap($x, $y);
echo "x = {$x}, y = {$y}\n";  // x = 20, y = 10

// 测试字符串
$str1 = 'Hello';
$str2 = 'World';
swap($str1, $str2);
echo "str1 = {$str1}, str2 = {$str2}\n";  // str1 = World, str2 = Hello

// 测试数组
$arr1 = [1, 2, 3];
$arr2 = [4, 5, 6];
swap($arr1, $arr2);
print_r($arr1);  // [4, 5, 6]
print_r($arr2);  // [1, 2, 3]
```

### 练习 3：使用魔术常量记录日志

创建一个日志函数，使用魔术常量记录每次函数调用的来源文件与行号，输出到日志文件。

**要求**：
- 使用 `__FILE__`、`__LINE__`、`__FUNCTION__` 等魔术常量
- 记录时间戳、消息、文件、行号、函数名
- 支持不同的日志级别（info、warning、error）
- 将日志写入文件

**参考实现**：

```php
<?php
declare(strict_types=1);

class Logger
{
    private string $logFile;
    
    public function __construct(string $logFile = 'app.log')
    {
        $this->logFile = $logFile;
    }
    
    private function writeLog(
        string $level,
        string $message,
        string $file,
        int $line,
        string $function
    ): void {
        $timestamp = date('Y-m-d H:i:s');
        $logEntry = sprintf(
            "[%s] [%s] %s | File: %s, Line: %d, Function: %s\n",
            $timestamp,
            strtoupper($level),
            $message,
            basename($file),
            $line,
            $function
        );
        
        file_put_contents($this->logFile, $logEntry, FILE_APPEND);
        echo $logEntry;
    }
    
    public function info(string $message): void
    {
        $backtrace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 1);
        $caller = $backtrace[0] ?? [];
        
        $this->writeLog(
            'info',
            $message,
            $caller['file'] ?? __FILE__,
            $caller['line'] ?? __LINE__,
            $caller['function'] ?? __FUNCTION__
        );
    }
    
    public function warning(string $message): void
    {
        $backtrace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 1);
        $caller = $backtrace[0] ?? [];
        
        $this->writeLog(
            'warning',
            $message,
            $caller['file'] ?? __FILE__,
            $caller['line'] ?? __LINE__,
            $caller['function'] ?? __FUNCTION__
        );
    }
    
    public function error(string $message): void
    {
        $backtrace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 1);
        $caller = $backtrace[0] ?? [];
        
        $this->writeLog(
            'error',
            $message,
            $caller['file'] ?? __FILE__,
            $caller['line'] ?? __LINE__,
            $caller['function'] ?? __FUNCTION__
        );
    }
}

// 使用示例
$logger = new Logger();

function processOrder(int $orderId, Logger $logger): void
{
    $logger->info("Processing order #{$orderId}");
    // 处理订单逻辑...
    $logger->info("Order #{$orderId} processed successfully");
}

function handleError(string $message, Logger $logger): void
{
    $logger->error("Error occurred: {$message}");
}

processOrder(12345, $logger);
handleError("Database connection failed", $logger);
```

### 练习 4：配置管理器

实现一个配置管理器，比较使用可变变量和数组两种方式，并说明为什么数组方式更优。

**要求**：
- 实现两种配置管理方式
- 比较可读性、安全性、可维护性
- 提供使用示例

**参考实现**：

```php
<?php
declare(strict_types=1);

// 方式 1：使用可变变量（不推荐）
class ConfigManagerVariables
{
    public static function load(array $config): void
    {
        foreach ($config as $key => $value) {
            $varName = strtoupper($key);
            $$varName = $value;  // 可变变量
        }
    }
}

// 方式 2：使用数组（推荐）
class ConfigManagerArray
{
    private static array $config = [];
    
    public static function load(array $config): void
    {
        self::$config = array_merge(self::$config, $config);
    }
    
    public static function get(string $key, mixed $default = null): mixed
    {
        return self::$config[$key] ?? $default;
    }
    
    public static function set(string $key, mixed $value): void
    {
        self::$config[$key] = $value;
    }
    
    public static function has(string $key): bool
    {
        return isset(self::$config[$key]);
    }
    
    public static function all(): array
    {
        return self::$config;
    }
}

// 使用示例
$config = [
    'app_name' => 'My Application',
    'app_version' => '1.0.0',
    'debug' => true,
    'database' => [
        'host' => 'localhost',
        'port' => 3306
    ]
];

// 数组方式（推荐）
ConfigManagerArray::load($config);

echo ConfigManagerArray::get('app_name') . "\n";
echo ConfigManagerArray::get('app_version') . "\n";
echo ConfigManagerArray::get('debug') ? 'true' : 'false' . "\n";
echo ConfigManagerArray::get('database.host', 'default') . "\n";

// 数组方式的优势：
// 1. 可读性：代码意图清晰，易于理解
// 2. 安全性：不会意外覆盖其他变量
// 3. 可维护性：易于添加新功能（如嵌套配置、验证等）
// 4. 可测试性：易于单元测试
// 5. IDE 支持：IDE 可以提供代码补全和类型检查
```

### 练习 5：记忆化函数

实现一个简单的记忆化（Memoization）函数，使用静态变量缓存函数调用的结果。

**要求**：
- 创建一个 `memoize()` 函数，接受一个可调用对象作为参数
- 使用静态变量缓存函数调用的结果
- 对于相同的参数，直接返回缓存的结果
- 支持不同参数类型的函数

**参考实现**：

```php
<?php
declare(strict_types=1);

function memoize(callable $fn): callable
{
    static $cache = [];
    
    return function (...$args) use ($fn, &$cache) {
        // 生成缓存键
        $key = serialize($args);
        
        // 检查缓存
        if (isset($cache[$key])) {
            echo "Cache hit for key: {$key}\n";
            return $cache[$key];
        }
        
        // 计算并缓存结果
        echo "Cache miss, computing...\n";
        $result = $fn(...$args);
        $cache[$key] = $result;
        
        return $result;
    };
}

// 示例：记忆化斐波那契函数
function fibonacci(int $n): int
{
    if ($n <= 1) {
        return $n;
    }
    
    static $memoized = null;
    
    if ($memoized === null) {
        $memoized = memoize(function (int $n): int {
            if ($n <= 1) {
                return $n;
            }
            
            $memoized = $GLOBALS['fib_memoized'] ?? null;
            return $memoized($n - 1) + $memoized($n - 2);
        });
        
        $GLOBALS['fib_memoized'] = $memoized;
    }
    
    return $memoized($n);
}

// 更简单的实现
function fibonacciSimple(int $n): int
{
    static $cache = [];
    
    if (isset($cache[$n])) {
        return $cache[$n];
    }
    
    if ($n <= 1) {
        return $n;
    }
    
    $result = fibonacciSimple($n - 1) + fibonacciSimple($n - 2);
    $cache[$n] = $result;
    
    return $result;
}

// 测试
echo "fibonacciSimple(10) = " . fibonacciSimple(10) . "\n";
echo "fibonacciSimple(10) = " . fibonacciSimple(10) . "\n";  // 使用缓存
```

### 练习 6：变量作用域综合应用

创建一个综合示例，演示变量作用域、全局变量、静态变量的使用场景。

**要求**：
- 创建全局变量、局部变量、静态变量
- 演示在不同作用域中访问变量的方式
- 说明各种方式的优缺点

**参考实现**：

```php
<?php
declare(strict_types=1);

// 全局变量
$globalCounter = 0;
$globalConfig = ['debug' => true];

// 使用全局变量的函数（不推荐）
function incrementGlobal(): void
{
    global $globalCounter;
    $globalCounter++;
    echo "Global counter: {$globalCounter}\n";
}

// 使用静态变量的函数（推荐用于函数内部状态）
function incrementStatic(): int
{
    static $count = 0;
    $count++;
    echo "Static counter: {$count}\n";
    return $count;
}

// 使用参数传递的函数（推荐）
function incrementWithParam(int &$counter): void
{
    $counter++;
    echo "Parameter counter: {$counter}\n";
}

// 配置类（推荐替代全局变量）
class AppConfig
{
    private static array $config = [];
    
    public static function set(string $key, mixed $value): void
    {
        self::$config[$key] = $value;
    }
    
    public static function get(string $key, mixed $default = null): mixed
    {
        return self::$config[$key] ?? $default;
    }
}

// 使用示例
echo "=== 全局变量方式 ===\n";
incrementGlobal();
incrementGlobal();

echo "\n=== 静态变量方式 ===\n";
incrementStatic();
incrementStatic();

echo "\n=== 参数传递方式 ===\n";
$localCounter = 0;
incrementWithParam($localCounter);
incrementWithParam($localCounter);

echo "\n=== 配置类方式 ===\n";
AppConfig::set('debug', true);
AppConfig::set('version', '1.0.0');
echo "Debug: " . (AppConfig::get('debug') ? 'true' : 'false') . "\n";
echo "Version: " . AppConfig::get('version') . "\n";
```

## 自检清单

完成本章学习后，请检查是否掌握以下内容：

- [ ] 理解 PHP 变量的命名规则和作用域
- [ ] 掌握变量的常用操作（赋值、检查、销毁）
- [ ] 理解引用赋值和引用参数的使用场景
- [ ] 了解可变变量的语法和限制
- [ ] 掌握 `const` 和 `define()` 的区别和使用场景
- [ ] 熟悉魔术常量的含义和用法
- [ ] 理解全局变量和静态变量的区别
- [ ] 能够根据场景选择合适的作用域管理方式
- [ ] 遵循变量和常量的命名规范
- [ ] 能够识别和避免常见的反模式

## 扩展阅读

1. **PHP 官方文档**：
   - [Variables](https://www.php.net/manual/en/language.variables.php)
   - [Constants](https://www.php.net/manual/en/language.constants.php)
   - [Variable scope](https://www.php.net/manual/en/language.variables.scope.php)

2. **PSR 标准**：
   - PSR-1: Basic Coding Standard
   - PSR-12: Extended Coding Style Guide

3. **相关主题**：
   - 类型系统（下一章）
   - 函数和作用域（后续章节）
   - 面向对象编程中的静态属性和方法
