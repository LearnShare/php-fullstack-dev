# 2.10.4 匿名函数

## 概述

匿名函数是没有名称的函数，可以作为值传递。本节详细介绍匿名函数的定义、调用、类型声明、在数组函数中使用、立即执行函数（IIFE）、作为参数/返回值等。

理解匿名函数的使用对于编写函数式风格的代码非常重要。掌握匿名函数的定义、调用和使用场景可以帮助编写更灵活、更强大的代码。

## 特性

- **无名称**：函数没有名称，只能通过变量调用
- **作为值**：可以作为变量值、参数、返回值
- **灵活性**：提供更大的灵活性
- **作用域**：有自己的作用域，可以捕获外部变量

## 语法/定义

### 匿名函数定义

**基本语法**：`$func = function(Type $param): ReturnType { ... };`

**特点**：
- 函数赋值给变量
- 支持类型声明
- 可以捕获外部变量（使用 `use`）

### 立即执行函数（IIFE）

**基本语法**：`(function($param) { ... })($value);`

**特点**：
- 定义后立即执行
- 创建独立作用域
- 可以返回值

### 类型声明

**语法**：`$func = function(Type $param): ReturnType { ... };`

**特点**：
- 支持参数类型声明
- 支持返回值类型声明
- 提高代码质量

## 基本用法

### 示例 1：基本匿名函数

```php
<?php
declare(strict_types=1);

// 基本匿名函数
$greet = function(string $name): string {
    return "Hello, {$name}!\n";
};

echo $greet("World");  // Hello, World!

// 赋值给变量
$add = function(int $a, int $b): int {
    return $a + $b;
};

echo $add(3, 4) . "\n";  // 7
```

### 示例 2：在数组函数中使用

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

// 在 array_map 中使用
$doubled = array_map(function($n) {
    return $n * 2;
}, $numbers);
print_r($doubled);  // Array ( [0] => 2 [1] => 4 [2] => 6 [3] => 8 [4] => 10 )

// 在 array_filter 中使用
$evens = array_filter($numbers, function($n) {
    return $n % 2 === 0;
});
print_r($evens);  // Array ( [1] => 2 [3] => 4 )

// 在 array_reduce 中使用
$sum = array_reduce($numbers, function($carry, $item) {
    return $carry + $item;
}, 0);
echo $sum . "\n";  // 15
```

### 示例 3：立即执行函数（IIFE）

```php
<?php
declare(strict_types=1);

// 基本 IIFE
$result = (function($x, $y) {
    return $x + $y;
})(3, 4);
echo $result . "\n";  // 7

// IIFE 创建独立作用域
$value = 10;
$result = (function() {
    $value = 20;  // 独立作用域，不影响外部
    return $value;
})();
echo $result . "\n";  // 20
echo $value . "\n";   // 10（未改变）

// IIFE 用于初始化
$config = (function() {
    $data = [];
    $data['host'] = 'localhost';
    $data['port'] = 3306;
    return $data;
})();
print_r($config);
```

### 示例 4：作为参数和返回值

```php
<?php
declare(strict_types=1);

// 作为参数
function process(callable $callback, mixed $value): mixed
{
    return $callback($value);
}

$result = process(function($x) {
    return $x * 2;
}, 5);
echo $result . "\n";  // 10

// 作为返回值
function createMultiplier(int $factor): Closure
{
    return function($x) use ($factor) {
        return $x * $factor;
    };
}

$double = createMultiplier(2);
echo $double(5) . "\n";  // 10
```

### 示例 5：变量捕获

```php
<?php
declare(strict_types=1);

// 捕获外部变量
$prefix = "Hello";
$suffix = "!";
$greet = function($name) use ($prefix, $suffix) {
    return "{$prefix}, {$name}{$suffix}";
};

echo $greet("World") . "\n";  // Hello, World!

// 按引用捕获
$count = 0;
$increment = function() use (&$count) {
    $count++;
    return $count;
};

echo $increment() . "\n";  // 1
echo $increment() . "\n";  // 2
echo $count . "\n";        // 2
```

## 完整代码示例

### 示例 1：函数工厂

```php
<?php
declare(strict_types=1);

class FunctionFactory
{
    public static function createValidator(string $type): Closure
    {
        return match($type) {
            'email' => function(string $value): bool {
                return filter_var($value, FILTER_VALIDATE_EMAIL) !== false;
            },
            'url' => function(string $value): bool {
                return filter_var($value, FILTER_VALIDATE_URL) !== false;
            },
            'int' => function(mixed $value): bool {
                return is_int($value);
            },
            default => function(mixed $value): bool {
                return true;
            }
        };
    }
    
    public static function createFormatter(string $format): Closure
    {
        return match($format) {
            'uppercase' => fn(string $s) => strtoupper($s),
            'lowercase' => fn(string $s) => strtolower($s),
            'capitalize' => fn(string $s) => ucfirst($s),
            default => fn(string $s) => $s
        };
    }
}

// 使用
$emailValidator = FunctionFactory::createValidator('email');
var_dump($emailValidator('test@example.com'));  // bool(true)

$upperFormatter = FunctionFactory::createFormatter('uppercase');
echo $upperFormatter('hello') . "\n";  // HELLO
```

### 示例 2：中间件系统

```php
<?php
declare(strict_types=1);

class MiddlewarePipeline
{
    private array $middlewares = [];
    
    public function add(callable $middleware): void
    {
        $this->middlewares[] = $middleware;
    }
    
    public function execute(mixed $input): mixed
    {
        $result = $input;
        foreach ($this->middlewares as $middleware) {
            $result = $middleware($result);
        }
        return $result;
    }
}

// 使用
$pipeline = new MiddlewarePipeline();

$pipeline->add(function($data) {
    return $data * 2;
});

$pipeline->add(function($data) {
    return $data + 10;
});

$pipeline->add(function($data) {
    return $data / 2;
});

$result = $pipeline->execute(5);
echo $result . "\n";  // 10 ((5 * 2 + 10) / 2)
```

### 示例 3：配置构建器

```php
<?php
declare(strict_types=1);

class ConfigBuilder
{
    private array $config = [];
    
    public function set(string $key, mixed $value): self
    {
        $this->config[$key] = $value;
        return $this;
    }
    
    public function transform(callable $transformer): self
    {
        $this->config = $transformer($this->config);
        return $this;
    }
    
    public function build(): array
    {
        return $this->config;
    }
}

// 使用
$config = (new ConfigBuilder())
    ->set('host', 'localhost')
    ->set('port', 3306)
    ->transform(function($config) {
        $config['dsn'] = "mysql:host={$config['host']};port={$config['port']}";
        return $config;
    })
    ->build();

print_r($config);
```

## 使用场景

### 回调函数

- **事件处理**：作为事件回调函数
- **数组操作**：在数组函数中使用
- **异步处理**：作为异步处理回调

### 函数式编程

- **高阶函数**：作为高阶函数的参数
- **函数组合**：组合多个函数
- **函数工厂**：创建函数工厂

### 作用域隔离

- **IIFE**：使用 IIFE 创建独立作用域
- **变量封装**：封装变量避免污染全局作用域
- **初始化代码**：执行初始化代码

## 注意事项

### 作用域

- **独立作用域**：匿名函数有自己的作用域
- **变量捕获**：使用 `use` 子句捕获外部变量
- **全局变量**：可以访问全局变量（不推荐）

### 性能考虑

- **函数调用开销**：匿名函数有函数调用开销
- **变量捕获**：变量捕获有性能开销
- **合理使用**：合理使用，避免过度使用

### 可读性

- **简单逻辑**：简单逻辑使用匿名函数
- **复杂逻辑**：复杂逻辑提取为命名函数
- **命名变量**：使用有意义的变量名

## 常见问题

### 问题 1：变量作用域

**症状**：匿名函数无法访问外部变量

**原因**：没有使用 `use` 子句捕获变量

**错误示例**：

```php
<?php
declare(strict_types=1);

$factor = 2;
$multiply = function($x) {
    return $x * $factor;  // 错误：$factor 未定义
};
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$factor = 2;

// 方法1：使用 use 子句
$multiply = function($x) use ($factor) {
    return $x * $factor;
};

// 方法2：使用箭头函数（自动捕获）
$multiply = fn($x) => $x * $factor;

echo $multiply(5) . "\n";  // 10
```

### 问题 2：IIFE 返回值

**症状**：IIFE 返回值处理错误

**原因**：不理解 IIFE 的返回值

**解决方法**：

```php
<?php
declare(strict_types=1);

// IIFE 可以返回值
$result = (function($x, $y) {
    return $x + $y;
})(3, 4);
echo $result . "\n";  // 7

// IIFE 可以赋值
$config = (function() {
    return ['host' => 'localhost', 'port' => 3306];
})();
print_r($config);
```

### 问题 3：匿名函数类型

**症状**：类型检查错误

**原因**：匿名函数类型声明不正确

**解决方法**：

```php
<?php
declare(strict_types=1);

// 正确：使用类型声明
$add = function(int $a, int $b): int {
    return $a + $b;
};

// 正确：作为 callable 类型
function process(callable $callback, mixed $value): mixed
{
    return $callback($value);
}

$result = process($add, [3, 4]);  // 需要展开
$result = process(fn($a, $b) => $a + $b, [3, 4]);  // 需要展开或使用包装
```

## 最佳实践

### 匿名函数使用

- **简单回调**：简单回调使用匿名函数
- **数组操作**：在数组函数中使用匿名函数
- **函数式编程**：函数式编程风格使用匿名函数

### 代码组织

- **提取函数**：复杂逻辑提取为命名函数
- **使用箭头函数**：简单场景使用箭头函数
- **提高可读性**：使用有意义的变量名

### 性能优化

- **合理使用**：合理使用匿名函数
- **避免过度**：避免过度使用匿名函数
- **缓存结果**：缓存重复使用的匿名函数

## 对比分析

### 匿名函数 vs 命名函数

| 特性 | 匿名函数 | 命名函数 |
|:-----|:---------|:---------|
| 名称 | 无 | 有 |
| 重用性 | 低 | 高 |
| 可读性 | 中 | 高 |
| 推荐度 | 推荐（回调） | 推荐（重用） |

**选择建议**：
- **回调函数**：使用匿名函数
- **可重用逻辑**：使用命名函数

### 匿名函数 vs 箭头函数

| 特性 | 匿名函数 | 箭头函数 |
|:-----|:---------|:---------|
| 语法 | 完整 | 简洁 |
| 表达式 | 多条语句 | 单表达式 |
| 变量捕获 | 手动 use | 自动 |
| 推荐度 | 推荐（复杂） | 推荐（简单） |

**选择建议**：
- **简单场景**：使用箭头函数
- **复杂逻辑**：使用匿名函数

## 相关章节

- **2.10.1 函数基础**：了解函数基础
- **2.10.3 引用、箭头函数与闭包**：了解闭包
- **2.8.3 数组操作函数**：了解数组函数

## 练习任务

1. **匿名函数练习**：
   - 练习定义和使用匿名函数
   - 理解作用域和变量捕获
   - 在数组函数中使用匿名函数
   - 测试各种匿名函数场景

2. **IIFE 练习**：
   - 练习使用立即执行函数
   - 理解独立作用域的概念
   - 实现配置初始化
   - 测试各种 IIFE 场景

3. **函数式编程练习**：
   - 实现函数工厂
   - 实现中间件系统
   - 实现函数组合
   - 测试各种函数式编程场景

4. **实际应用练习**：
   - 实现事件处理系统
   - 实现数据转换管道
   - 实现配置构建器
   - 测试各种应用场景

5. **综合练习**：
   - 创建一个使用匿名函数的程序
   - 实现各种高级功能
   - 理解作用域和变量捕获
   - 进行代码审查，确保正确性
