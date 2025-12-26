# 2.10.3 引用、箭头函数与闭包

## 概述

引用、箭头函数和闭包是 PHP 的高级特性。本节详细介绍引用传参、箭头函数（PHP 7.4+）、闭包、变量捕获（按值、按引用）、闭包与类、`Closure::bindTo()`、变量生命周期、与 JavaScript 对比等。

理解这些高级特性对于编写函数式风格的代码和利用 PHP 的灵活性非常重要。掌握变量捕获机制、闭包与类的关系可以帮助编写更强大的代码。

## 特性

- **引用传参**：可以修改函数参数，影响原变量
- **箭头函数**：简洁的函数语法，自动捕获外部变量（PHP 7.4+）
- **闭包**：可以捕获外部变量的函数，支持按值和按引用捕获

## 语法/定义

### 引用传参

**基本语法**：`function func(&$param) { ... }`

**特点**：
- 参数前加 `&` 表示引用传递
- 修改参数会影响原变量
- 调用时不需要加 `&`

### 箭头函数（PHP 7.4+）

**基本语法**：`fn($param) => expression`

**特点**：
- 自动捕获外部变量（按值）
- 只能包含单个表达式
- 简洁的语法
- 不能使用 `return` 语句

### 闭包

**基本语法**：`function($param) use ($var) { ... }`

**特点**：
- 可以捕获外部变量
- 支持按值捕获：`use ($var)`
- 支持按引用捕获：`use (&$var)`
- 可以包含多条语句

### Closure::bindTo() - 绑定闭包到对象

**语法**：`Closure::bindTo(?object $newThis, ?string $newScope = "static"): ?Closure`

**参数**：
- `$newThis`：要绑定的对象（`null` 表示不绑定）
- `$newScope`：类作用域（`"static"` 表示保持原作用域）

**返回值**：返回新的闭包实例

## 基本用法

### 示例 1：引用传参

```php
<?php
declare(strict_types=1);

// 基本引用传参
function swap(&$a, &$b): void
{
    $temp = $a;
    $a = $b;
    $b = $temp;
}

$x = 1;
$y = 2;
swap($x, $y);
echo "x: {$x}, y: {$y}\n";  // x: 2, y: 1

// 修改数组元素
function doubleArray(array &$arr): void
{
    foreach ($arr as &$value) {
        $value *= 2;
    }
    unset($value);  // 解除引用
}

$numbers = [1, 2, 3];
doubleArray($numbers);
print_r($numbers);  // Array ( [0] => 2 [1] => 4 [2] => 6 )
```

### 示例 2：箭头函数

```php
<?php
declare(strict_types=1);

// 基本箭头函数
$multiply = fn($x, $y) => $x * $y;
echo $multiply(3, 4) . "\n";  // 12

// 自动捕获外部变量
$factor = 2;
$multiply = fn($x) => $x * $factor;
echo $multiply(5) . "\n";  // 10

// 在数组函数中使用
$numbers = [1, 2, 3, 4, 5];
$doubled = array_map(fn($n) => $n * 2, $numbers);
print_r($doubled);  // Array ( [0] => 2 [1] => 4 [2] => 6 [3] => 8 [4] => 10 )

// 多个参数
$add = fn($a, $b, $c) => $a + $b + $c;
echo $add(1, 2, 3) . "\n";  // 6
```

### 示例 3：闭包

```php
<?php
declare(strict_types=1);

// 基本闭包
$factor = 2;
$multiply = function($x) use ($factor) {
    return $x * $factor;
};

echo $multiply(5) . "\n";  // 10

// 按引用捕获
$count = 0;
$increment = function() use (&$count) {
    $count++;
};

$increment();
$increment();
echo $count . "\n";  // 2

// 捕获多个变量
$prefix = "Hello";
$suffix = "!";
$greet = function($name) use ($prefix, $suffix) {
    return "{$prefix}, {$name}{$suffix}";
};

echo $greet("World") . "\n";  // Hello, World!
```

### 示例 4：闭包与类

```php
<?php
declare(strict_types=1);

class Counter
{
    private int $count = 0;
    
    public function getIncrementor(): Closure
    {
        return function() {
            $this->count++;  // 可以访问 $this
            return $this->count;
        };
    }
}

$counter = new Counter();
$increment = $counter->getIncrementor();
echo $increment() . "\n";  // 1
echo $increment() . "\n";  // 2

// 使用 bindTo 绑定到其他对象
class AnotherCounter
{
    public int $count = 0;
}

$another = new AnotherCounter();
$bound = $increment->bindTo($another, AnotherCounter::class);
echo $bound() . "\n";  // 1
```

## 完整代码示例

### 示例 1：计数器实现

```php
<?php
declare(strict_types=1);

function createCounter(int $initial = 0): Closure
{
    $count = $initial;
    return function() use (&$count) {
        return ++$count;
    };
}

// 使用
$counter1 = createCounter(10);
echo $counter1() . "\n";  // 11
echo $counter1() . "\n";  // 12

$counter2 = createCounter(0);
echo $counter2() . "\n";  // 1
echo $counter2() . "\n";  // 2
```

### 示例 2：函数工厂

```php
<?php
declare(strict_types=1);

function createMultiplier(int $factor): Closure
{
    return function($x) use ($factor) {
        return $x * $factor;
    };
}

// 使用
$double = createMultiplier(2);
$triple = createMultiplier(3);

echo $double(5) . "\n";  // 10
echo $triple(5) . "\n";  // 15

// 箭头函数版本（PHP 7.4+）
function createMultiplierArrow(int $factor): Closure
{
    return fn($x) => $x * $factor;
}
```

### 示例 3：事件系统

```php
<?php
declare(strict_types=1);

class EventEmitter
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
                $listener(...$args);
            }
        }
    }
}

// 使用
$emitter = new EventEmitter();

$emitter->on('user.created', function($user) {
    echo "User created: {$user['name']}\n";
});

$emitter->on('user.created', fn($user) => echo "Sending email to {$user['email']}\n");

$emitter->emit('user.created', ['name' => 'John', 'email' => 'john@example.com']);
```

### 示例 4：数据转换器

```php
<?php
declare(strict_types=1);

class DataTransformer
{
    public static function createTransformer(callable $transform): Closure
    {
        return function(array $data) use ($transform) {
            return array_map($transform, $data);
        };
    }
    
    public static function createFilter(callable $predicate): Closure
    {
        return function(array $data) use ($predicate) {
            return array_filter($data, $predicate);
        };
    }
}

// 使用
$toUpper = DataTransformer::createTransformer(fn($s) => strtoupper($s));
$result = $toUpper(['hello', 'world']);
print_r($result);  // Array ( [0] => HELLO [1] => WORLD )

$isEven = DataTransformer::createFilter(fn($n) => $n % 2 === 0);
$result = $isEven([1, 2, 3, 4, 5]);
print_r($result);  // Array ( [1] => 2 [3] => 4 )
```

## 使用场景

### 引用传参

- **修改参数**：需要修改函数参数
- **交换变量**：交换两个变量的值
- **数组修改**：修改数组元素

### 箭头函数

- **简单回调**：简单的回调函数
- **数组操作**：在数组函数中使用
- **表达式计算**：简单的表达式计算

### 闭包

- **变量捕获**：需要捕获外部变量
- **函数工厂**：创建函数工厂
- **事件处理**：处理事件回调
- **状态保持**：保持函数状态

## 注意事项

### 变量捕获

- **按值捕获**：默认按值捕获，捕获时的值
- **按引用捕获**：使用 `&` 按引用捕获，捕获变量本身
- **理解区别**：理解按值和按引用的区别

### 箭头函数限制

- **单表达式**：只能包含单个表达式
- **自动捕获**：自动按值捕获外部变量
- **不能 return**：不能使用 `return` 语句

### 闭包与类

- **访问 $this**：闭包可以访问 `$this`（在类方法中定义时）
- **bindTo**：可以使用 `bindTo` 绑定到其他对象
- **作用域**：理解闭包的作用域

## 常见问题

### 问题 1：变量捕获方式混淆

**症状**：闭包行为不符合预期

**原因**：不理解按值和按引用的区别

**错误示例**：

```php
<?php
declare(strict_types=1);

$count = 0;
$increment = function() use ($count) {  // 按值捕获
    $count++;
};

$increment();
$increment();
echo $count . "\n";  // 0（不符合预期）
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$count = 0;
$increment = function() use (&$count) {  // 按引用捕获
    $count++;
};

$increment();
$increment();
echo $count . "\n";  // 2（正确）

// 或使用箭头函数（自动按值捕获，但这里需要按引用）
$count = 0;
$increment = function() use (&$count) {
    $count++;
};
```

### 问题 2：箭头函数限制

**症状**：箭头函数无法实现复杂逻辑

**原因**：箭头函数只能包含单个表达式

**错误示例**：

```php
<?php
declare(strict_types=1);

// 错误：箭头函数不能包含多条语句
$process = fn($x) => {
    $result = $x * 2;
    return $result + 1;
};
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 方法1：使用闭包
$process = function($x) {
    $result = $x * 2;
    return $result + 1;
};

// 方法2：简化表达式
$process = fn($x) => $x * 2 + 1;

// 方法3：提取为函数
function process(int $x): int
{
    $result = $x * 2;
    return $result + 1;
}
```

### 问题 3：闭包中的 $this

**症状**：闭包无法访问 `$this`

**原因**：闭包不在类方法中定义，或未正确绑定

**解决方法**：

```php
<?php
declare(strict_types=1);

class MyClass
{
    private int $value = 10;
    
    public function getClosure(): Closure
    {
        // 在类方法中定义的闭包可以访问 $this
        return function() {
            return $this->value;
        };
    }
}

$obj = new MyClass();
$closure = $obj->getClosure();
echo $closure() . "\n";  // 10

// 绑定到其他对象
class AnotherClass
{
    public int $value = 20;
}

$another = new AnotherClass();
$bound = $closure->bindTo($another);
echo $bound() . "\n";  // 20
```

## 最佳实践

### 引用传参

- **谨慎使用**：谨慎使用引用传参
- **明确意图**：使用引用时明确代码意图
- **文档说明**：为引用参数添加文档说明

### 箭头函数

- **简单场景**：简单回调使用箭头函数
- **复杂逻辑**：复杂逻辑使用闭包或普通函数
- **可读性**：优先考虑可读性

### 闭包

- **变量捕获**：理解变量捕获机制
- **生命周期**：注意变量的生命周期
- **性能考虑**：注意闭包的性能开销

## 对比分析

### 箭头函数 vs 闭包

| 特性 | 箭头函数 | 闭包 |
|:-----|:---------|:-----|
| 语法 | 简洁 | 完整 |
| 表达式 | 单表达式 | 多条语句 |
| 变量捕获 | 自动按值 | 手动 use |
| 推荐度 | 推荐（简单） | 推荐（复杂） |

**选择建议**：
- **简单回调**：使用箭头函数
- **复杂逻辑**：使用闭包

### 按值捕获 vs 按引用捕获

| 特性 | 按值捕获 | 按引用捕获 |
|:-----|:---------|:------------|
| 语法 | `use ($var)` | `use (&$var)` |
| 行为 | 捕获值 | 捕获变量 |
| 修改 | 不影响原变量 | 影响原变量 |
| 推荐度 | 推荐（默认） | 按需使用 |

**选择建议**：
- **默认**：使用按值捕获
- **需要修改**：使用按引用捕获

## 相关章节

- **2.3.2 引用**：了解变量引用
- **2.10.1 函数基础**：了解函数基础
- **2.10.4 匿名函数**：了解匿名函数

## 练习任务

1. **引用传参练习**：
   - 练习使用引用传参
   - 理解引用传参的行为
   - 测试各种引用场景
   - 观察参数修改效果

2. **箭头函数练习**：
   - 练习使用箭头函数
   - 理解自动捕获机制
   - 在数组函数中使用箭头函数
   - 测试各种箭头函数场景

3. **闭包练习**：
   - 练习定义和使用闭包
   - 理解按值和按引用捕获
   - 实现函数工厂
   - 测试闭包与类的关系

4. **实际应用练习**：
   - 实现计数器功能
   - 实现事件系统
   - 实现数据转换器
   - 测试各种应用场景

5. **综合练习**：
   - 创建一个使用闭包的程序
   - 实现各种高级功能
   - 理解变量捕获机制
   - 进行代码审查，确保正确性
