# 2.10.5 可调用类型与内置函数

## 概述

可调用类型表示可以调用的值。本节详细介绍可调用类型的各种形式、`is_callable()`、`call_user_func()`、`call_user_func_array()`、内置函数工具等。

理解可调用类型的使用对于编写动态、灵活的代码非常重要。掌握可调用类型的各种形式和动态调用函数可以帮助编写更强大的代码。

## 特性

- **多种形式**：函数名、方法、闭包、实现了 `__invoke()` 的对象
- **动态调用**：可以动态调用函数
- **类型检查**：可以检查值是否可调用
- **灵活性**：提供更大的灵活性

## 语法/定义

### 可调用类型的各种形式

#### 函数名（字符串）

**语法**：`'function_name'`

**特点**：
- 函数名作为字符串
- 必须是已定义的函数
- 可以使用 `function_exists()` 检查

#### 方法（数组）

**实例方法**：`[$object, 'method']`

**静态方法**：`[ClassName::class, 'method']` 或 `'ClassName::method'`

**特点**：
- 数组第一个元素是对象或类名
- 第二个元素是方法名（字符串）

#### 闭包

**语法**：`function($param) { ... }` 或 `fn($param) => expression`

**特点**：
- 匿名函数或箭头函数
- 可以直接调用

#### 可调用对象

**语法**：实现了 `__invoke()` 方法的对象

**特点**：
- 对象可以像函数一样调用
- 必须实现 `__invoke()` 方法

### 类型声明

**语法**：`function func(callable $callback) { ... }`

**特点**：
- 接受任何可调用值
- 不区分可调用形式

### is_callable() - 检查是否可调用

**语法**：`is_callable(mixed $value, bool $syntax_only = false, string &$callable_name = null): bool`

**参数**：
- `$value`：要检查的值
- `$syntax_only`：可选，是否只检查语法，默认为 `false`
- `$callable_name`：可选，引用参数，返回可调用名称

**返回值**：如果值可调用返回 `true`，否则返回 `false`

### call_user_func() - 调用回调函数

**语法**：`call_user_func(callable $callback, mixed ...$args): mixed`

**参数**：
- `$callback`：要调用的回调函数
- `...$args`：传递给回调函数的参数

**返回值**：返回回调函数的返回值

### call_user_func_array() - 使用数组参数调用

**语法**：`call_user_func_array(callable $callback, array $args): mixed`

**参数**：
- `$callback`：要调用的回调函数
- `$args`：包含参数的数组

**返回值**：返回回调函数的返回值

## 基本用法

### 示例 1：可调用类型的各种形式

```php
<?php
declare(strict_types=1);

// 函数名
function greet(string $name): string
{
    return "Hello, {$name}!\n";
}

$callable1 = 'greet';
echo $callable1("World");  // Hello, World!

// 实例方法
class User
{
    public function getName(): string
    {
        return "John";
    }
}

$user = new User();
$callable2 = [$user, 'getName'];
echo $callable2() . "\n";  // John

// 静态方法
class Math
{
    public static function add(int $a, int $b): int
    {
        return $a + $b;
    }
}

$callable3 = [Math::class, 'add'];
echo $callable3(3, 4) . "\n";  // 7

// 闭包
$callable4 = function($x) {
    return $x * 2;
};
echo $callable4(5) . "\n";  // 10

// 可调用对象
class Multiplier
{
    public function __construct(private int $factor) {}
    
    public function __invoke(int $x): int
    {
        return $x * $this->factor;
    }
}

$callable5 = new Multiplier(3);
echo $callable5(5) . "\n";  // 15
```

### 示例 2：is_callable()

```php
<?php
declare(strict_types=1);

// 检查函数名
var_dump(is_callable('strlen'));  // bool(true)
var_dump(is_callable('nonexistent'));  // bool(false)

// 检查方法
class MyClass
{
    public function method(): void {}
}

$obj = new MyClass();
var_dump(is_callable([$obj, 'method']));  // bool(true)
var_dump(is_callable([$obj, 'nonexistent']));  // bool(false)

// 检查闭包
$closure = function() {};
var_dump(is_callable($closure));  // bool(true)

// 检查可调用对象
class CallableClass
{
    public function __invoke(): void {}
}

$callable = new CallableClass();
var_dump(is_callable($callable));  // bool(true)

// 获取可调用名称
$name = '';
is_callable([$obj, 'method'], false, $name);
echo $name . "\n";  // MyClass::method
```

### 示例 3：call_user_func()

```php
<?php
declare(strict_types=1);

// 调用函数
function add(int $a, int $b): int
{
    return $a + $b;
}

$result = call_user_func('add', 3, 4);
echo $result . "\n";  // 7

// 调用方法
class Calculator
{
    public function multiply(int $a, int $b): int
    {
        return $a * $b;
    }
}

$calc = new Calculator();
$result = call_user_func([$calc, 'multiply'], 3, 4);
echo $result . "\n";  // 12

// 调用闭包
$closure = function($x) {
    return $x * 2;
};
$result = call_user_func($closure, 5);
echo $result . "\n";  // 10
```

### 示例 4：call_user_func_array()

```php
<?php
declare(strict_types=1);

// 使用数组参数调用
function sum(int $a, int $b, int $c): int
{
    return $a + $b + $c;
}

$args = [1, 2, 3];
$result = call_user_func_array('sum', $args);
echo $result . "\n";  // 6

// 动态调用方法
class Processor
{
    public function process(string $operation, int $a, int $b): int
    {
        return match($operation) {
            'add' => $a + $b,
            'multiply' => $a * $b,
            default => 0
        };
    }
}

$processor = new Processor();
$args = ['add', 3, 4];
$result = call_user_func_array([$processor, 'process'], $args);
echo $result . "\n";  // 7
```

## 完整代码示例

### 示例 1：事件系统

```php
<?php
declare(strict_types=1);

class EventDispatcher
{
    private array $listeners = [];
    
    public function on(string $event, callable $listener): void
    {
        $this->listeners[$event][] = $listener;
    }
    
    public function emit(string $event, ...$args): void
    {
        if (isset($this->listeners[$event])) {
            foreach ($this->listeners[$event] as $listener) {
                if (is_callable($listener)) {
                    call_user_func($listener, ...$args);
                }
            }
        }
    }
}

// 使用
$dispatcher = new EventDispatcher();

// 注册不同类型的监听器
$dispatcher->on('user.created', function($user) {
    echo "User created: {$user['name']}\n";
});

$dispatcher->on('user.created', [new class {
    public function handle($user): void
    {
        echo "Sending email to {$user['email']}\n";
    }
}, 'handle']);

$dispatcher->emit('user.created', ['name' => 'John', 'email' => 'john@example.com']);
```

### 示例 2：函数路由

```php
<?php
declare(strict_types=1);

class FunctionRouter
{
    private array $routes = [];
    
    public function register(string $name, callable $handler): void
    {
        $this->routes[$name] = $handler;
    }
    
    public function call(string $name, array $args = []): mixed
    {
        if (!isset($this->routes[$name])) {
            throw new InvalidArgumentException("Route '{$name}' not found");
        }
        
        $handler = $this->routes[$name];
        if (!is_callable($handler)) {
            throw new RuntimeException("Handler for '{$name}' is not callable");
        }
        
        return call_user_func_array($handler, $args);
    }
}

// 使用
$router = new FunctionRouter();

$router->register('add', fn($a, $b) => $a + $b);
$router->register('multiply', function($a, $b) {
    return $a * $b;
});

echo $router->call('add', [3, 4]) . "\n";        // 7
echo $router->call('multiply', [3, 4]) . "\n";  // 12
```

### 示例 3：可调用对象工厂

```php
<?php
declare(strict_types=1);

class CallableFactory
{
    public static function createValidator(string $type): callable
    {
        return match($type) {
            'email' => function(string $value): bool {
                return filter_var($value, FILTER_VALIDATE_EMAIL) !== false;
            },
            'int' => fn(mixed $value): bool => is_int($value),
            'string' => fn(mixed $value): bool => is_string($value),
            default => fn(mixed $value): bool => true
        };
    }
    
    public static function createFormatter(string $format): callable
    {
        return new class($format) {
            public function __construct(private string $format) {}
            
            public function __invoke(string $value): string
            {
                return match($this->format) {
                    'upper' => strtoupper($value),
                    'lower' => strtolower($value),
                    'capitalize' => ucfirst($value),
                    default => $value
                };
            }
        };
    }
}

// 使用
$emailValidator = CallableFactory::createValidator('email');
var_dump($emailValidator('test@example.com'));  // bool(true)

$upperFormatter = CallableFactory::createFormatter('upper');
echo $upperFormatter('hello') . "\n";  // HELLO
```

### 示例 4：动态方法调用

```php
<?php
declare(strict_types=1);

class ApiClient
{
    private array $endpoints = [];
    
    public function registerEndpoint(string $method, string $path, callable $handler): void
    {
        $this->endpoints["{$method}:{$path}"] = $handler;
    }
    
    public function handle(string $method, string $path, array $params = []): mixed
    {
        $key = "{$method}:{$path}";
        if (!isset($this->endpoints[$key])) {
            throw new RuntimeException("Endpoint not found: {$key}");
        }
        
        $handler = $this->endpoints[$key];
        if (!is_callable($handler)) {
            throw new RuntimeException("Handler is not callable");
        }
        
        return call_user_func_array($handler, $params);
    }
}

// 使用
$client = new ApiClient();

$client->registerEndpoint('GET', '/users', function(int $id) {
    return ['id' => $id, 'name' => 'John'];
});

$client->registerEndpoint('POST', '/users', function(string $name, string $email) {
    return ['name' => $name, 'email' => $email, 'created' => true];
});

$user = $client->handle('GET', '/users', [1]);
print_r($user);

$result = $client->handle('POST', '/users', ['Jane', 'jane@example.com']);
print_r($result);
```

## 使用场景

### 动态调用

- **函数路由**：根据条件动态调用函数
- **事件处理**：处理各种事件回调
- **插件系统**：实现插件系统

### 函数式编程

- **高阶函数**：作为高阶函数的参数
- **函数组合**：组合多个函数
- **函数工厂**：创建函数工厂

### 框架开发

- **路由系统**：实现路由系统
- **中间件**：实现中间件系统
- **事件系统**：实现事件系统

## 注意事项

### 类型检查

- **使用 is_callable()**：调用前检查是否可调用
- **错误处理**：处理调用失败的情况
- **类型安全**：使用类型声明提高类型安全

### 性能考虑

- **动态调用开销**：动态调用有性能开销
- **缓存结果**：缓存可调用对象
- **合理使用**：合理使用，避免过度使用

### 安全性

- **验证输入**：验证可调用值
- **防止注入**：防止代码注入攻击
- **权限检查**：检查调用权限

## 常见问题

### 问题 1：可调用类型检查

**症状**：调用失败或类型错误

**原因**：没有检查值是否可调用

**错误示例**：

```php
<?php
declare(strict_types=1);

$callback = 'nonexistent_function';
$callback();  // Fatal error: Call to undefined function
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$callback = 'nonexistent_function';

// 检查是否可调用
if (is_callable($callback)) {
    $callback();
} else {
    echo "Callback is not callable\n";
}

// 或使用 call_user_func
if (is_callable($callback)) {
    call_user_func($callback);
}
```

### 问题 2：动态调用错误

**症状**：动态调用失败

**原因**：调用的函数或方法不存在

**解决方法**：

```php
<?php
declare(strict_types=1);

function safeCall(callable $callback, ...$args): mixed
{
    if (!is_callable($callback)) {
        throw new InvalidArgumentException("Callback is not callable");
    }
    
    try {
        return call_user_func($callback, ...$args);
    } catch (Throwable $e) {
        error_log("Call failed: " . $e->getMessage());
        throw $e;
    }
}

// 使用
try {
    $result = safeCall('strlen', 'hello');
    echo $result . "\n";  // 5
} catch (Exception $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

### 问题 3：方法调用权限

**症状**：无法调用私有或受保护方法

**原因**：方法访问权限限制

**解决方法**：

```php
<?php
declare(strict_types=1);

class MyClass
{
    private function privateMethod(): string
    {
        return "Private method called";
    }
    
    public function callPrivate(): string
    {
        // 在类内部可以调用私有方法
        return $this->privateMethod();
    }
}

$obj = new MyClass();
echo $obj->callPrivate() . "\n";  // Private method called

// 外部无法直接调用私有方法
// $obj->privateMethod();  // 错误
```

## 最佳实践

### 可调用类型使用

- **类型检查**：使用 `is_callable()` 检查
- **错误处理**：处理调用失败的情况
- **类型声明**：使用 `callable` 类型声明

### 动态调用

- **合理使用**：合理使用动态调用
- **性能考虑**：注意动态调用的性能开销
- **安全性**：确保动态调用的安全性

### 代码组织

- **函数路由**：使用函数路由组织代码
- **事件系统**：使用事件系统解耦代码
- **插件系统**：使用插件系统扩展功能

## 对比分析

### call_user_func vs 直接调用

| 特性 | call_user_func | 直接调用 |
|:-----|:---------------|:---------|
| 灵活性 | 高 | 低 |
| 性能 | 稍差 | 稍好 |
| 动态性 | 是 | 否 |
| 推荐度 | 推荐（动态） | 推荐（静态） |

**选择建议**：
- **动态调用**：使用 `call_user_func`
- **静态调用**：直接调用

### 可调用类型 vs 具体类型

| 特性 | callable | 具体类型 |
|:-----|:---------|:---------|
| 灵活性 | 高 | 低 |
| 类型安全 | 低 | 高 |
| 推荐度 | 推荐（动态） | 推荐（静态） |

**选择建议**：
- **动态场景**：使用 `callable`
- **静态场景**：使用具体类型

## 相关章节

- **2.10.1 函数基础**：了解函数基础
- **2.10.4 匿名函数**：了解匿名函数
- **2.8.3 数组操作函数**：了解数组函数

## 练习任务

1. **可调用类型练习**：
   - 练习使用各种可调用类型
   - 理解不同形式的区别
   - 测试各种可调用场景
   - 观察调用行为

2. **动态调用练习**：
   - 练习使用 `call_user_func` 和 `call_user_func_array`
   - 实现函数路由系统
   - 实现事件系统
   - 测试各种动态调用场景

3. **可调用对象练习**：
   - 实现 `__invoke()` 方法
   - 创建可调用对象
   - 在动态调用中使用可调用对象
   - 测试各种可调用对象场景

4. **实际应用练习**：
   - 实现事件分发系统
   - 实现函数路由系统
   - 实现插件系统
   - 测试各种应用场景

5. **综合练习**：
   - 创建一个使用可调用类型的程序
   - 实现各种高级功能
   - 理解动态调用的机制
   - 进行代码审查，确保正确性
