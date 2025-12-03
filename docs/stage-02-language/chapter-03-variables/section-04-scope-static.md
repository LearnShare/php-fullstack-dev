# 2.3.4 全局与静态

## 概述

在 PHP 中，变量的作用域决定了变量的可见性和生命周期。**全局变量**（Global Variables）在脚本级别定义，但在函数内部需要使用特殊方式访问。**静态变量**（Static Variables）在函数调用之间保持其值，提供了一种在函数内部维护状态的方式，而无需使用全局变量。

## 全局变量（Global Variables）

### 基本概念

在函数外部定义的变量具有全局作用域，可以在脚本的任何地方访问。但在函数内部，不能直接访问全局变量，需要使用 `global` 关键字或 `$GLOBALS` 超全局数组。

### 使用 global 关键字

#### 语法

```php
global $variable1, $variable2, ...;
```

#### 基本示例

```php
<?php
declare(strict_types=1);

$counter = 0;  // 全局变量

function increment(): void
{
    global $counter;  // 声明使用全局变量
    $counter++;
}

function decrement(): void
{
    global $counter;
    $counter--;
}

increment();
echo $counter . "\n";  // 输出：1

increment();
echo $counter . "\n";  // 输出：2

decrement();
echo $counter . "\n";  // 输出：1
```

#### 多个全局变量

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$age  = 25;
$city = 'Beijing';

function displayInfo(): void
{
    global $name, $age, $city;  // 声明多个全局变量
    
    echo "Name: {$name}\n";
    echo "Age: {$age}\n";
    echo "City: {$city}\n";
}

displayInfo();
```

### 使用 $GLOBALS 超全局数组

`$GLOBALS` 是一个超全局数组，包含了所有全局变量的引用。

#### 语法

```php
$GLOBALS['variable_name']
```

#### 基本示例

```php
<?php
declare(strict_types=1);

$counter = 0;

function increment(): void
{
    $GLOBALS['counter']++;  // 使用 $GLOBALS 访问全局变量
}

function decrement(): void
{
    $GLOBALS['counter']--;
}

increment();
echo $counter . "\n";  // 输出：1

increment();
echo $counter . "\n";  // 输出：2
```

#### global 与 $GLOBALS 的区别

```php
<?php
declare(strict_types=1);

$var = 10;

function testGlobal(): void
{
    global $var;
    $var = 20;  // 修改全局变量
}

function testGlobals(): void
{
    $GLOBALS['var'] = 30;  // 修改全局变量
}

echo "Before: {$var}\n";  // 输出：Before: 10

testGlobal();
echo "After global: {$var}\n";  // 输出：After global: 20

testGlobals();
echo "After GLOBALS: {$var}\n";  // 输出：After GLOBALS: 30
```

**区别说明**：
- `global $var` 创建一个指向全局变量的局部引用
- `$GLOBALS['var']` 直接访问全局变量数组

### 全局变量的限制和问题

#### 1. 可读性和维护性

全局变量使代码难以理解和维护，因为函数的行为依赖于外部状态。

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
    if ($config['debug']) {
        echo $message . "\n";
    }
}
```

#### 2. 测试困难

全局变量使单元测试变得困难，因为测试需要设置和清理全局状态。

#### 3. 命名冲突

在大型项目中，全局变量名可能发生冲突。

#### 4. 线程安全

在多线程环境中（如某些 CLI 应用），全局变量可能导致竞态条件。

### 全局变量的替代方案

#### 1. 使用函数参数

```php
<?php
declare(strict_types=1);

function processData(array $data, array $config): array
{
    // 使用参数而不是全局变量
    if ($config['validate']) {
        // 验证数据
    }
    // 处理数据...
    return $data;
}
```

#### 2. 使用配置类

```php
<?php
declare(strict_types=1);

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

Config::set('debug', true);
Config::set('timezone', 'Asia/Shanghai');

function logMessage(string $message): void
{
    if (Config::get('debug', false)) {
        echo $message . "\n";
    }
}
```

#### 3. 使用依赖注入

```php
<?php
declare(strict_types=1);

class Logger
{
    private bool $debug;
    
    public function __construct(bool $debug = false)
    {
        $this->debug = $debug;
    }
    
    public function log(string $message): void
    {
        if ($this->debug) {
            echo $message . "\n";
        }
    }
}

$logger = new Logger(true);
$logger->log('Debug message');
```

## 静态变量（Static Variables）

### 基本概念

静态变量在函数调用之间保持其值，但只能在定义它的函数内部访问。静态变量在函数第一次调用时初始化，之后的值会保留到下次调用。

### 语法

```php
static $variable = initial_value;
```

### 基本示例

```php
<?php
declare(strict_types=1);

function counter(): int
{
    static $count = 0;  // 只在第一次调用时初始化为 0
    $count++;
    return $count;
}

echo counter() . "\n";  // 输出：1
echo counter() . "\n";  // 输出：2
echo counter() . "\n";  // 输出：3
```

### 静态变量的初始化

静态变量的初始化表达式只在函数第一次调用时执行：

```php
<?php
declare(strict_types=1);

function testStatic(): void
{
    static $count = 0;
    static $timestamp = null;
    
    if ($timestamp === null) {
        $timestamp = time();  // 只在第一次调用时设置
        echo "First call at: " . date('Y-m-d H:i:s', $timestamp) . "\n";
    }
    
    $count++;
    echo "Call count: {$count}\n";
}

testStatic();  // First call at: ..., Call count: 1
sleep(1);
testStatic();  // Call count: 2（时间戳不变）
```

### 静态变量与局部变量的对比

```php
<?php
declare(strict_types=1);

function withLocal(): void
{
    $count = 0;  // 每次调用都重新初始化为 0
    $count++;
    echo "Local count: {$count}\n";
}

function withStatic(): void
{
    static $count = 0;  // 只在第一次调用时初始化为 0
    $count++;
    echo "Static count: {$count}\n";
}

echo "=== Local variable ===\n";
withLocal();  // Local count: 1
withLocal();  // Local count: 1
withLocal();  // Local count: 1

echo "\n=== Static variable ===\n";
withStatic();  // Static count: 1
withStatic();  // Static count: 2
withStatic();  // Static count: 3
```

### 静态变量的应用场景

#### 1. 计数器

```php
<?php
declare(strict_types=1);

function getNextId(): int
{
    static $id = 0;
    return ++$id;
}

echo getNextId() . "\n";  // 输出：1
echo getNextId() . "\n";  // 输出：2
echo getNextId() . "\n";  // 输出：3
```

#### 2. 缓存计算结果

```php
<?php
declare(strict_types=1);

function fibonacci(int $n): int
{
    static $cache = [];
    
    if (isset($cache[$n])) {
        return $cache[$n];
    }
    
    if ($n <= 1) {
        return $n;
    }
    
    $result = fibonacci($n - 1) + fibonacci($n - 2);
    $cache[$n] = $result;
    
    return $result;
}

echo fibonacci(10) . "\n";  // 使用缓存加速计算
```

#### 3. 单例模式（简化版）

```php
<?php
declare(strict_types=1);

class Database
{
    private static ?Database $instance = null;
    
    private function __construct()
    {
        // 私有构造函数
    }
    
    public static function getInstance(): Database
{
        if (self::$instance === null) {
            self::$instance = new self();
        }
        return self::$instance;
    }
}

$db1 = Database::getInstance();
$db2 = Database::getInstance();

var_dump($db1 === $db2);  // bool(true) - 同一个实例
```

#### 4. 记录函数调用历史

```php
<?php
declare(strict_types=1);

function logCall(string $message): void
{
    static $callHistory = [];
    static $callCount = 0;
    
    $callCount++;
    $callHistory[] = [
        'count' => $callCount,
        'message' => $message,
        'time' => date('Y-m-d H:i:s')
    ];
    
    echo "Call #{$callCount}: {$message}\n";
}

logCall('First call');
logCall('Second call');
logCall('Third call');
```

### 静态变量的限制

#### 1. 作用域限制

静态变量只能在定义它的函数内部访问：

```php
<?php
declare(strict_types=1);

function test(): void
{
    static $count = 0;
    $count++;
}

test();
// echo $count;  // 错误：未定义变量
```

#### 2. 不能直接重置

静态变量不能像普通变量那样直接重置，需要特殊处理：

```php
<?php
declare(strict_types=1);

function counter(): int
{
    static $count = 0;
    return ++$count;
}

function resetCounter(): void
{
    // 无法直接重置静态变量
    // 但可以通过其他方式实现
}

// 解决方案：使用函数参数控制
function counterWithReset(bool $reset = false): int
{
    static $count = 0;
    
    if ($reset) {
        $count = 0;
    }
    
    return ++$count;
}

echo counterWithReset() . "\n";  // 1
echo counterWithReset() . "\n";  // 2
echo counterWithReset(true) . "\n";  // 1（重置后）
```

#### 3. 线程安全

在多线程环境中，静态变量可能导致竞态条件（但在 PHP 的典型使用场景中通常不是问题）。

### 静态变量与全局变量的对比

| 特性           | 全局变量                 | 静态变量                 |
| :------------- | :----------------------- | :----------------------- |
| 作用域         | 脚本级别                 | 函数级别                 |
| 访问方式       | 需要 `global` 或 `$GLOBALS` | 直接在函数内访问         |
| 可见性         | 整个脚本可见             | 仅在函数内可见           |
| 状态保持       | 是                       | 是                       |
| 命名冲突风险   | 高                       | 低（函数作用域内）       |
| 可测试性       | 差                       | 较好                     |
| 推荐使用场景   | 配置、单例（不推荐）     | 函数内部状态保持         |

## 完整示例

### 示例 1：使用全局变量实现简单计数器（不推荐，仅作演示）

```php
<?php
declare(strict_types=1);

$globalCounter = 0;

function incrementGlobal(): void
{
    global $globalCounter;
    $globalCounter++;
    echo "Global counter: {$globalCounter}\n";
}

function resetGlobalCounter(): void
{
    global $globalCounter;
    $globalCounter = 0;
    echo "Counter reset\n";
}

incrementGlobal();  // Global counter: 1
incrementGlobal();  // Global counter: 2
incrementGlobal();  // Global counter: 3
resetGlobalCounter();  // Counter reset
incrementGlobal();  // Global counter: 1
```

### 示例 2：使用静态变量实现计数器（推荐）

```php
<?php
declare(strict_types=1);

function getCounter(): int
{
    static $count = 0;
    return ++$count;
}

function resetCounter(): int
{
    static $count = 0;
    $count = 0;
    return $count;
}

echo getCounter() . "\n";  // 1
echo getCounter() . "\n";  // 2
echo getCounter() . "\n";  // 3
resetCounter();
echo getCounter() . "\n";  // 1
```

### 示例 3：使用静态变量实现请求 ID 生成器

```php
<?php
declare(strict_types=1);

function generateRequestId(): string
{
    static $counter = 0;
    static $prefix = null;
    
    if ($prefix === null) {
        $prefix = date('YmdHis') . '_';
    }
    
    $counter++;
    return $prefix . str_pad((string)$counter, 4, '0', STR_PAD_LEFT);
}

echo generateRequestId() . "\n";  // 20240115103045_0001
echo generateRequestId() . "\n";  // 20240115103045_0002
echo generateRequestId() . "\n";  // 20240115103045_0003
```

### 示例 4：使用静态变量缓存配置加载

```php
<?php
declare(strict_types=1);

function loadConfig(): array
{
    static $config = null;
    
    if ($config === null) {
        // 模拟从文件加载配置（实际项目中应从配置文件读取）
        $config = [
            'app_name' => 'My Application',
            'version' => '1.0.0',
            'debug' => true
        ];
        echo "Config loaded from file\n";
    }
    
    return $config;
}

// 第一次调用：加载配置
$config1 = loadConfig();  // Config loaded from file

// 后续调用：使用缓存的配置
$config2 = loadConfig();  // 不再输出 "Config loaded from file"
$config3 = loadConfig();  // 不再输出 "Config loaded from file"

var_dump($config1 === $config2);  // bool(true) - 同一个数组
```

### 示例 5：改进的配置管理（避免全局变量）

```php
<?php
declare(strict_types=1);

class AppConfig
{
    private static array $config = [];
    private static bool $loaded = false;
    
    public static function load(array $config): void
    {
        if (!self::$loaded) {
            self::$config = $config;
            self::$loaded = true;
        }
    }
    
    public static function get(string $key, mixed $default = null): mixed
    {
        return self::$config[$key] ?? $default;
    }
    
    public static function set(string $key, mixed $value): void
    {
        self::$config[$key] = $value;
    }
    
    public static function reset(): void
    {
        self::$config = [];
        self::$loaded = false;
    }
}

// 初始化配置
AppConfig::load([
    'app_name' => 'My Application',
    'version' => '1.0.0',
    'debug' => true
]);

// 使用配置
echo AppConfig::get('app_name') . "\n";
echo AppConfig::get('version') . "\n";
```

## 最佳实践

1. **避免使用全局变量**：
   - 优先使用函数参数传递数据
   - 使用配置类或依赖注入
   - 如果必须使用，添加详细注释说明原因

2. **合理使用静态变量**：
   - 用于函数内部状态保持
   - 用于缓存计算结果
   - 避免过度使用，保持函数纯净性

3. **状态管理**：
   - 对于复杂的状态管理，使用类属性而不是静态变量
   - 考虑使用单例模式或服务容器

4. **测试友好**：
   - 避免使用全局变量，使函数易于测试
   - 如果使用静态变量，提供重置机制

5. **文档说明**：
   - 在使用全局变量或静态变量时，添加注释说明其用途和生命周期

## 练习

1. 编写一个函数 `getUniqueId()`，使用静态变量生成唯一的递增 ID。

2. 创建一个函数 `memoize(callable $fn)`，使用静态变量缓存函数调用的结果（简单的记忆化实现）。

3. 实现一个配置管理器类，使用静态属性存储配置，避免使用全局变量。

4. 编写一个函数 `trackCalls(string $functionName)`，使用静态变量记录每个函数的调用次数。

5. 创建一个简单的日志系统，比较使用全局变量和静态变量的实现方式，并说明各自的优缺点。
