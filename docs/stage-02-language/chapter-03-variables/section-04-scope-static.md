# 2.3.4 全局与静态

## 概述

全局变量和静态变量是 PHP 中两种特殊的变量类型，它们有不同的作用域和生命周期。本节详细介绍全局变量的访问方式（`global` 关键字、`$GLOBALS` 数组）、静态变量的定义和使用，以及它们的使用场景和注意事项。

理解全局变量和静态变量的区别，可以帮助在合适的场景选择合适的变量类型。但需要注意，过度使用全局变量会导致代码耦合，应该优先使用参数传递和依赖注入。

## 特性

- **全局变量**：在脚本顶级作用域定义的变量，可以在函数内部通过 `global` 关键字或 `$GLOBALS` 数组访问
- **静态变量**：在函数内部定义的变量，在多次函数调用间保持值不变
- **生命周期**：全局变量在脚本执行期间存在，静态变量在函数调用间保持值
- **作用域**：全局变量在全局作用域可见，静态变量只在函数内部可见

## 语法/定义

### global 关键字

**语法**：`global $variable_name1, $variable_name2, ...;`

**作用**：
- 在函数内部声明全局变量
- 允许在函数内部访问和修改全局变量
- 可以同时声明多个全局变量

**特点**：
- 必须在函数内部使用
- 声明后可以直接使用全局变量
- 修改会影响全局变量

### $GLOBALS 数组

**语法**：`$GLOBALS['variable_name']`

**作用**：
- 直接访问全局变量，不需要 `global` 关键字
- 可以在任何作用域访问
- 是超全局变量之一

**特点**：
- 是关联数组，键是变量名（不含 `$`）
- 修改 `$GLOBALS['variable_name']` 会影响全局变量
- 可以直接赋值和读取

### static 关键字

**语法**：`static $variable_name = initial_value;`

**作用**：
- 在函数内部定义静态变量
- 静态变量只在第一次调用时初始化
- 后续调用保持上次的值

**特点**：
- 只在函数内部可见
- 生命周期跨越多次函数调用
- 初始化值必须是常量表达式

## 基本用法

### 示例 1：global 关键字

```php
<?php
declare(strict_types=1);

// 全局变量
$globalCounter = 0;
$globalName = "World";

function incrementCounter(): void
{
    // 声明全局变量
    global $globalCounter, $globalName;
    
    // 使用全局变量
    $globalCounter++;
    echo "Counter: {$globalCounter}, Name: {$globalName}\n";
}

incrementCounter();  // Counter: 1, Name: World
incrementCounter();  // Counter: 2, Name: World

// 修改全局变量
$globalName = "PHP";
incrementCounter();  // Counter: 3, Name: PHP
```

**执行**：

```bash
php global-example.php
```

**输出**：

```
Counter: 1, Name: World
Counter: 2, Name: World
Counter: 3, Name: PHP
```

### 示例 2：$GLOBALS 数组

```php
<?php
declare(strict_types=1);

// 全局变量
$config = ['key' => 'value'];
$counter = 0;

function accessGlobals(): void
{
    // 使用 $GLOBALS 访问全局变量
    echo "Config: " . json_encode($GLOBALS['config']) . "\n";
    
    // 修改全局变量
    $GLOBALS['counter']++;
    echo "Counter: {$GLOBALS['counter']}\n";
    
    // 添加新的全局变量
    $GLOBALS['newVar'] = "New Value";
}

accessGlobals();
echo "New var: {$newVar}\n";  // New var: New Value
```

**执行**：

```bash
php globals-example.php
```

**输出**：

```
Config: {"key":"value"}
Counter: 1
New var: New Value
```

### 示例 3：静态变量

```php
<?php
declare(strict_types=1);

// 计数器函数
function counter(): int
{
    static $count = 0;  // 只在第一次调用时初始化
    $count++;
    return $count;
}

echo counter() . "\n";  // 1
echo counter() . "\n";  // 2
echo counter() . "\n";  // 3

// 多个静态变量
function multiStatic(): void
{
    static $var1 = 0;
    static $var2 = "initial";
    
    $var1++;
    $var2 .= " updated";
    
    echo "var1: {$var1}, var2: {$var2}\n";
}

multiStatic();  // var1: 1, var2: initial updated
multiStatic();  // var1: 2, var2: initial updated updated
```

**执行**：

```bash
php static-example.php
```

**输出**：

```
1
2
3
var1: 1, var2: initial updated
var1: 2, var2: initial updated updated
```

### 示例 4：静态变量实现单例模式

```php
<?php
declare(strict_types=1);

class Database
{
    private function __construct()
    {
        // 私有构造函数
    }
    
    public static function getInstance(): self
    {
        static $instance = null;
        
        if ($instance === null) {
            $instance = new self();
        }
        
        return $instance;
    }
}

$db1 = Database::getInstance();
$db2 = Database::getInstance();

var_dump($db1 === $db2);  // bool(true) - 同一个实例
```

## 完整代码示例

### 示例 1：全局变量 vs 参数传递

```php
<?php
declare(strict_types=1);

// 不推荐：使用全局变量
$globalCounter = 0;

function incrementGlobal(): void
{
    global $globalCounter;
    $globalCounter++;
}

incrementGlobal();
echo "Global counter: {$globalCounter}\n";

// 推荐：使用参数传递
function incrementByReference(int &$counter): void
{
    $counter++;
}

$localCounter = 0;
incrementByReference($localCounter);
echo "Local counter: {$localCounter}\n";

// 推荐：使用返回值
function incrementByReturn(int $counter): int
{
    return $counter + 1;
}

$localCounter = incrementByReturn($localCounter);
echo "Local counter: {$localCounter}\n";
```

### 示例 2：静态变量实现缓存

```php
<?php
declare(strict_types=1);

function expensiveCalculation(int $input): int
{
    static $cache = [];
    
    // 检查缓存
    if (isset($cache[$input])) {
        echo "Cache hit for {$input}\n";
        return $cache[$input];
    }
    
    // 执行计算
    echo "Calculating for {$input}\n";
    $result = $input * $input;  // 模拟耗时计算
    
    // 存储到缓存
    $cache[$input] = $result;
    
    return $result;
}

echo expensiveCalculation(5) . "\n";   // Calculating for 5, 25
echo expensiveCalculation(5) . "\n";   // Cache hit for 5, 25
echo expensiveCalculation(10) . "\n";  // Calculating for 10, 100
echo expensiveCalculation(10) . "\n";  // Cache hit for 10, 100
```

### 示例 3：静态变量实现函数调用统计

```php
<?php
declare(strict_types=1);

function trackFunctionCall(string $functionName): void
{
    static $callCounts = [];
    
    if (!isset($callCounts[$functionName])) {
        $callCounts[$functionName] = 0;
    }
    
    $callCounts[$functionName]++;
    
    echo "Function '{$functionName}' called {$callCounts[$functionName]} times\n";
}

function functionA(): void
{
    trackFunctionCall(__FUNCTION__);
}

function functionB(): void
{
    trackFunctionCall(__FUNCTION__);
}

functionA();  // Function 'functionA' called 1 times
functionA();  // Function 'functionA' called 2 times
functionB();  // Function 'functionB' called 1 times
functionA();  // Function 'functionA' called 3 times
```

## 使用场景

### 全局变量使用场景

- **配置信息**：存储应用配置（不推荐，应使用配置类或依赖注入）
- **共享状态**：在多个函数间共享状态（不推荐，应使用参数传递）
- **遗留代码**：维护遗留代码时可能需要使用

**注意**：全局变量应尽量避免使用，优先使用参数传递、依赖注入或配置类。

### 静态变量使用场景

- **计数器**：统计函数调用次数
- **缓存**：在函数内部实现简单缓存
- **单例模式**：实现单例模式
- **状态保持**：在多次调用间保持状态

### 超全局变量简介

**超全局变量**：PHP 预定义的全局变量，在任何作用域都可以直接访问。

**常见超全局变量**：
- `$_GET`、`$_POST`、`$_REQUEST`：HTTP 请求数据
- `$_SERVER`：服务器和环境信息
- `$_SESSION`、`$_COOKIE`：会话和 Cookie 数据
- `$_FILES`：文件上传数据
- `$_ENV`：环境变量
- `$GLOBALS`：全局变量数组

**详细内容**：将在阶段五（Web/API 开发）中详细介绍。

## 注意事项

### 全局变量注意事项

- **避免使用全局变量**：全局变量会导致代码耦合，难以测试和维护
- **使用参数传递**：优先使用函数参数传递数据
- **使用依赖注入**：使用依赖注入替代全局变量
- **明确标注**：如果必须使用全局变量，应该明确标注并添加注释

### 静态变量注意事项

- **初始化时机**：静态变量只在第一次调用时初始化
- **作用域限制**：静态变量只在函数内部可见
- **线程安全**：静态变量在多线程环境下可能有问题（PHP 通常单线程）
- **内存管理**：静态变量会一直占用内存，直到脚本结束

### 性能考虑

- **全局变量访问**：访问全局变量有轻微性能开销
- **静态变量访问**：访问静态变量性能较好
- **缓存效果**：静态变量缓存可以提升性能，但要注意内存占用

## 常见问题

### 问题 1：全局变量未声明

**症状**：在函数内部无法访问全局变量

**原因**：未使用 `global` 关键字声明

**错误示例**：

```php
<?php
declare(strict_types=1);

$globalVar = "Global";

function test(): void
{
    echo $globalVar;  // Warning: Undefined variable
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$globalVar = "Global";

function test(): void
{
    // 方法1：使用 global 关键字
    global $globalVar;
    echo $globalVar . "\n";
    
    // 方法2：使用 $GLOBALS
    echo $GLOBALS['globalVar'] . "\n";
}
```

### 问题 2：静态变量初始化问题

**症状**：静态变量初始化不符合预期

**原因**：静态变量只在第一次调用时初始化

**示例**：

```php
<?php
declare(strict_types=1);

function test(int $value): void
{
    static $counter = $value;  // 错误：不能使用变量初始化
    
    $counter++;
    echo "Counter: {$counter}\n";
}

// 正确：使用常量表达式初始化
function testCorrect(): void
{
    static $counter = 0;  // 正确：使用常量
    
    $counter++;
    echo "Counter: {$counter}\n";
}
```

### 问题 3：全局变量导致代码耦合

**症状**：代码难以测试和维护

**原因**：过度使用全局变量

**解决方法**：

1. **使用参数传递**：通过函数参数传递数据

```php
<?php
declare(strict_types=1);

// 不推荐
$config = ['key' => 'value'];

function process(): void
{
    global $config;
    // 使用 $config
}

// 推荐
function process(array $config): void
{
    // 使用 $config
}
```

2. **使用依赖注入**：通过构造函数或方法注入依赖

```php
<?php
declare(strict_types=1);

class Processor
{
    public function __construct(
        private array $config
    ) {}
    
    public function process(): void
    {
        // 使用 $this->config
    }
}
```

## 最佳实践

### 全局变量

- **避免使用**：尽量避免使用全局变量
- **使用参数传递**：优先使用函数参数传递数据
- **使用依赖注入**：使用依赖注入替代全局变量
- **明确标注**：如果必须使用，明确标注并添加注释

### 静态变量

- **合理使用**：在合适的场景使用静态变量（计数器、缓存等）
- **注意内存**：注意静态变量的内存占用
- **避免滥用**：不要过度使用静态变量
- **文档说明**：为使用静态变量的函数添加文档说明

### 代码组织

- **优先使用局部变量**：优先使用局部变量和参数传递
- **使用配置类**：使用配置类管理配置信息
- **使用依赖注入**：使用依赖注入管理依赖关系

## 对比分析

### 全局变量 vs 参数传递

| 特性 | 全局变量 | 参数传递 |
|:-----|:---------|:---------|
| 代码耦合 | 高 | 低 |
| 可测试性 | 低 | 高 |
| 可维护性 | 低 | 高 |
| 性能 | 略差 | 略好 |
| 推荐度 | 不推荐 | 推荐 |

**选择建议**：
- **优先使用参数传递**：提高代码可维护性和可测试性
- **避免全局变量**：除非确实必要，避免使用全局变量

### 静态变量 vs 全局变量

| 特性 | 静态变量 | 全局变量 |
|:-----|:---------|:---------|
| 作用域 | 函数内部 | 全局 |
| 可见性 | 低 | 高 |
| 生命周期 | 函数调用间 | 脚本执行期间 |
| 使用场景 | 计数器、缓存 | 配置、共享状态 |
| 推荐度 | 按需使用 | 不推荐 |

**选择建议**：
- **函数内部状态**：使用静态变量
- **全局配置**：使用配置类或依赖注入，不使用全局变量

## 相关章节

- **2.3.1 变量基础**：了解变量的基本概念
- **2.3.2 引用与可变变量**：了解引用传递
- **2.10 函数与作用域**：深入学习函数和作用域
- **阶段五：Web/API 开发**：学习超全局变量

## 练习任务

1. **全局变量练习**：
   - 使用 `global` 关键字访问全局变量
   - 使用 `$GLOBALS` 数组访问全局变量
   - 理解全局变量的作用域和生命周期
   - 练习将全局变量改为参数传递

2. **静态变量练习**：
   - 定义静态变量实现计数器
   - 使用静态变量实现简单缓存
   - 使用静态变量实现单例模式
   - 理解静态变量的初始化时机

3. **代码重构练习**：
   - 将使用全局变量的代码重构为使用参数传递
   - 将硬编码的配置改为配置类
   - 实现依赖注入替代全局变量
   - 测试重构后的代码

4. **最佳实践练习**：
   - 创建一个配置类管理配置信息
   - 实现一个使用静态变量的工具函数
   - 编写代码审查清单，检查全局变量的使用
   - 进行代码审查，确保遵循最佳实践

5. **综合练习**：
   - 创建一个完整的应用，避免使用全局变量
   - 使用参数传递和依赖注入管理数据
   - 在合适的地方使用静态变量
   - 测试代码的可维护性和可测试性
