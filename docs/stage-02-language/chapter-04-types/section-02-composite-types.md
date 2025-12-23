# 2.4.2 复合类型

## 概述

复合类型（Composite Types）是可以包含多个值或具有复杂结构的类型，包括数组（`array`）、对象（`object`）和可调用类型（`callable`）。

## 数组类型（array）

### 基本概念

PHP 中的数组是一个有序映射（ordered map），可以同时包含数字键和字符串键。数组可以表示多种数据结构：列表（索引数组）、字典（关联数组）、栈、队列等。

### 创建数组

#### 使用方括号语法（PHP 5.4+）

```php
<?php
declare(strict_types=1);

// 索引数组
$numbers = [1, 2, 3, 4, 5];

// 关联数组
$user = [
    'name' => 'Alice',
    'age' => 25,
    'email' => 'alice@example.com'
];

// 混合数组
$mixed = [
    0 => 'first',
    'key' => 'value',
    1 => 'second'
];
```

#### 使用 array() 语法

```php
<?php
declare(strict_types=1);

$numbers = array(1, 2, 3, 4, 5);
$user = array(
    'name' => 'Alice',
    'age' => 25
);
```

### 类型检测

#### `is_array()`

**语法**：`is_array(mixed $value): bool`

**参数**：
- `$value`：要检测的值

**返回值**：如果 `$value` 是数组类型，返回 `true`；否则返回 `false`。

```php
<?php
declare(strict_types=1);

var_dump(is_array([1, 2, 3]));      // bool(true)
var_dump(is_array(['key' => 'value'])); // bool(true)
var_dump(is_array("string"));        // bool(false)
var_dump(is_array(null));            // bool(false)
```

### 数组操作

#### 访问元素

```php
<?php
declare(strict_types=1);

$arr = ['a' => 1, 'b' => 2, 'c' => 3];
echo $arr['a'] . "\n";  // 1

// 检查键是否存在
if (isset($arr['a'])) {
    echo "Key 'a' exists\n";
}

// 使用 null 合并运算符
$value = $arr['d'] ?? 'default';  // 'default'
```

#### 添加元素

```php
<?php
declare(strict_types=1);

$arr = [];
$arr[] = 'first';        // 自动索引
$arr[] = 'second';
$arr['key'] = 'value';   // 指定键

print_r($arr);
// 输出：
// Array
// (
//     [0] => first
//     [1] => second
//     [key] => value
// )
```

#### 删除元素

```php
<?php
declare(strict_types=1);

$arr = ['a' => 1, 'b' => 2, 'c' => 3];
unset($arr['b']);

print_r($arr);
// 输出：
// Array
// (
//     [a] => 1
//     [c] => 3
// )
```

### 数组遍历

#### foreach 循环

```php
<?php
declare(strict_types=1);

$arr = ['apple' => 'red', 'banana' => 'yellow', 'orange' => 'orange'];

// 遍历键值对
foreach ($arr as $key => $value) {
    echo "{$key}: {$value}\n";
}

// 只遍历值
foreach ($arr as $value) {
    echo "{$value}\n";
}
```

### 数组高阶函数

#### `array_map()` - 映射

**语法**：`array_map(?callable $callback, array $array, array ...$arrays): array`

**参数**：
- `$callback`：回调函数，对每个元素执行
- `$array`：要处理的数组
- `...$arrays`：可选，多个数组时，回调函数接收多个参数

**返回值**：返回处理后的新数组。

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

// 将每个元素乘以 2
$doubled = array_map(fn($n) => $n * 2, $numbers);
print_r($doubled);  // [2, 4, 6, 8, 10]

// 使用多个数组
$arr1 = [1, 2, 3];
$arr2 = [10, 20, 30];
$sum = array_map(fn($a, $b) => $a + $b, $arr1, $arr2);
print_r($sum);  // [11, 22, 33]
```

#### `array_filter()` - 过滤

**语法**：`array_filter(array $array, ?callable $callback = null, int $mode = 0): array`

**参数**：
- `$array`：要过滤的数组
- `$callback`：可选，过滤函数，返回 `true` 保留元素
- `$mode`：可选，`ARRAY_FILTER_USE_KEY` 或 `ARRAY_FILTER_USE_BOTH`

**返回值**：返回过滤后的数组。

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 过滤偶数
$evens = array_filter($numbers, fn($n) => $n % 2 === 0);
print_r($evens);  // [2, 4, 6, 8, 10]

// 过滤空值
$data = ['a' => 1, 'b' => 0, 'c' => '', 'd' => null, 'e' => 2];
$filtered = array_filter($data);
print_r($filtered);  // ['a' => 1, 'e' => 2]
```

#### `array_reduce()` - 归约

**语法**：`array_reduce(array $array, callable $callback, mixed $initial = null): mixed`

**参数**：
- `$array`：要归约的数组
- `$callback`：归约函数，接收累积值和当前值
- `$initial`：可选，初始值

**返回值**：返回归约结果。

```php
<?php
declare(strict_types=1);

$numbers = [1, 2, 3, 4, 5];

// 求和
$sum = array_reduce($numbers, fn($carry, $n) => $carry + $n, 0);
echo $sum . "\n";  // 15

// 求积
$product = array_reduce($numbers, fn($carry, $n) => $carry * $n, 1);
echo $product . "\n";  // 120

// 连接字符串
$words = ['Hello', 'World', 'PHP'];
$sentence = array_reduce($words, fn($carry, $word) => $carry . ' ' . $word, '');
echo trim($sentence) . "\n";  // Hello World PHP
```

### 数组排序

#### `sort()` - 按值排序（重新索引）

**语法**：`sort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：
- `$array`：要排序的数组（按引用传递，会被修改）
- `$flags`：可选，排序标志，默认为 `SORT_REGULAR`

**返回值**：成功返回 `true`，失败返回 `false`。

```php
<?php
declare(strict_types=1);

$numbers = [3, 1, 4, 1, 5, 9, 2, 6];
sort($numbers);
print_r($numbers);  // [1, 1, 2, 3, 4, 5, 6, 9]
```

#### `rsort()` - 按值逆序排序

**语法**：`rsort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：
- `$array`：要排序的数组（按引用传递，会被修改）
- `$flags`：可选，排序标志，默认为 `SORT_REGULAR`

**返回值**：成功返回 `true`，失败返回 `false`。

#### `asort()` - 按值排序（保留键）

**语法**：`asort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：
- `$array`：要排序的数组（按引用传递，会被修改）
- `$flags`：可选，排序标志，默认为 `SORT_REGULAR`

**返回值**：成功返回 `true`，失败返回 `false`。

```php
<?php
declare(strict_types=1);

$fruits = ['d' => 'lemon', 'a' => 'orange', 'b' => 'banana', 'c' => 'apple'];
asort($fruits);
print_r($fruits);
// 输出：
// Array
// (
//     [c] => apple
//     [b] => banana
//     [d] => lemon
//     [a] => orange
// )
```

#### `ksort()` - 按键排序

**语法**：`ksort(array &$array, int $flags = SORT_REGULAR): bool`

**参数**：
- `$array`：要排序的数组（按引用传递，会被修改）
- `$flags`：可选，排序标志，默认为 `SORT_REGULAR`

**返回值**：成功返回 `true`，失败返回 `false`。

#### `usort()` - 自定义排序

**语法**：`usort(array &$array, callable $callback): bool`

**参数**：
- `$array`：要排序的数组（按引用传递，会被修改）
- `$callback`：比较函数，接收两个参数，返回 `< 0`、`0` 或 `> 0`

**返回值**：成功返回 `true`，失败返回 `false`。

```php
<?php
declare(strict_types=1);

$users = [
    ['name' => 'Alice', 'age' => 25],
    ['name' => 'Bob', 'age' => 30],
    ['name' => 'Charlie', 'age' => 20]
];

// 按年龄排序
usort($users, fn($a, $b) => $a['age'] <=> $b['age']);
print_r($users);
```

### 完整示例

```php
<?php
declare(strict_types=1);

// 数组创建和操作
function createUserArray(string $name, int $age, string $email): array
{
    return [
        'name' => $name,
        'age' => $age,
        'email' => $email,
        'created_at' => date('Y-m-d H:i:s')
    ];
}

$user = createUserArray('Alice', 25, 'alice@example.com');
print_r($user);

// 数组处理函数
function processNumbers(array $numbers): array
{
    return [
        'original' => $numbers,
        'doubled' => array_map(fn($n) => $n * 2, $numbers),
        'evens' => array_filter($numbers, fn($n) => $n % 2 === 0),
        'sum' => array_reduce($numbers, fn($carry, $n) => $carry + $n, 0),
        'max' => max($numbers),
        'min' => min($numbers),
        'average' => count($numbers) > 0 ? array_sum($numbers) / count($numbers) : 0
    ];
}

$result = processNumbers([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);
print_r($result);
```

## 对象类型（object）

### 基本概念

对象是类的实例，包含属性和方法。PHP 是面向对象语言，支持类、对象、继承、多态等特性。

### 创建对象

```php
<?php
declare(strict_types=1);

class User
{
    public string $name;
    public int $age;
    
    public function __construct(string $name, int $age)
    {
        $this->name = $name;
        $this->age = $age;
    }
    
    public function greet(): string
    {
        return "Hello, I'm {$this->name}";
    }
}

$user = new User('Alice', 25);
echo $user->greet() . "\n";  // Hello, I'm Alice
```

### 类型检测

#### `is_object()`

**语法**：`is_object(mixed $value): bool`

**参数**：
- `$value`：要检测的值

**返回值**：如果 `$value` 是对象类型，返回 `true`；否则返回 `false`。

```php
<?php
declare(strict_types=1);

$user = new User('Alice', 25);
var_dump(is_object($user));  // bool(true)
var_dump(is_object([]));     // bool(false)
```

#### `instanceof` - 类型检查

```php
<?php
declare(strict_types=1);

$user = new User('Alice', 25);
var_dump($user instanceof User);  // bool(true)

// 检查是否实现接口
interface Greetable
{
    public function greet(): string;
}

class Person implements Greetable
{
    public function greet(): string
    {
        return "Hello";
    }
}

$person = new Person();
var_dump($person instanceof Greetable);  // bool(true)
```

### 对象克隆

```php
<?php
declare(strict_types=1);

$user1 = new User('Alice', 25);
$user2 = clone $user1;  // 浅拷贝

$user2->name = 'Bob';
echo $user1->name . "\n";  // Alice（不受影响）
```

### 完整示例

```php
<?php
declare(strict_types=1);

class Product
{
    public function __construct(
        public string $name,
        public float $price,
        public int $stock = 0
    ) {}
    
    public function isAvailable(): bool
    {
        return $this->stock > 0;
    }
    
    public function getPriceWithTax(float $taxRate = 0.1): float
    {
        return $this->price * (1 + $taxRate);
    }
}

$product = new Product('Laptop', 999.99, 10);
echo "Product: {$product->name}\n";
echo "Price: \${$product->price}\n";
echo "Available: " . ($product->isAvailable() ? 'Yes' : 'No') . "\n";
echo "Price with tax: \${$product->getPriceWithTax()}\n";
```

## 可调用类型（callable）

### 基本概念

可调用类型表示可以作为函数调用的值，包括函数名、方法、匿名函数、实现了 `__invoke()` 方法的对象等。

### 可调用类型的形式

#### 1. 函数名字符串

```php
<?php
declare(strict_types=1);

function greet(string $name): string
{
    return "Hello, {$name}";
}

$callable = 'greet';
echo $callable('Alice') . "\n";  // Hello, Alice
```

#### 2. 对象方法数组

```php
<?php
declare(strict_types=1);

class Greeter
{
    public function greet(string $name): string
    {
        return "Hello, {$name}";
    }
}

$greeter = new Greeter();
$callable = [$greeter, 'greet'];
echo $callable('Bob') . "\n";  // Hello, Bob

// 静态方法
$callable = [Greeter::class, 'greet'];
```

#### 3. 匿名函数（闭包）

```php
<?php
declare(strict_types=1);

$callable = function(string $name): string {
    return "Hello, {$name}";
};

echo $callable('Charlie') . "\n";  // Hello, Charlie
```

#### 4. 箭头函数（PHP 7.4+）

```php
<?php
declare(strict_types=1);

$callable = fn(string $name): string => "Hello, {$name}";
echo $callable('David') . "\n";  // Hello, David
```

#### 5. 实现了 __invoke() 的对象

```php
<?php
declare(strict_types=1);

class CallableClass
{
    public function __invoke(string $name): string
    {
        return "Hello, {$name}";
    }
}

$callable = new CallableClass();
echo $callable('Eve') . "\n";  // Hello, Eve
```

### 类型检测

#### `is_callable()`

**语法**：`is_callable(mixed $value, bool $syntax_only = false, string &$callable_name = null): bool`

**参数**：
- `$value`：要检测的值
- `$syntax_only`：如果为 `true`，只检查语法，不检查是否可调用
- `$callable_name`：可选，接收可调用名称

**返回值**：如果值可调用，返回 `true`；否则返回 `false`。

```php
<?php
declare(strict_types=1);

function test(): void {}

var_dump(is_callable('test'));           // bool(true)
var_dump(is_callable('nonexistent'));    // bool(false)
var_dump(is_callable(fn() => null));     // bool(true)
var_dump(is_callable([new Greeter(), 'greet'])); // bool(true)
```

### 调用可调用类型

#### `call_user_func()`

**语法**：`call_user_func(callable $callback, mixed ...$args): mixed`

**参数**：
- `$callback`：要调用的函数或方法
- `...$args`：可变参数，传递给回调函数的参数

**返回值**：返回回调函数的返回值。

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

$result = call_user_func('add', 5, 3);
echo $result . "\n";  // 8
```

#### `call_user_func_array()`

**语法**：`call_user_func_array(callable $callback, array $args): mixed`

**参数**：
- `$callback`：要调用的函数或方法
- `$args`：传递给回调函数的参数数组

**返回值**：返回回调函数的返回值。

```php
<?php
declare(strict_types=1);

function add(int $a, int $b): int
{
    return $a + $b;
}

$result = call_user_func_array('add', [5, 3]);
echo $result . "\n";  // 8
```

### 完整示例

```php
<?php
declare(strict_types=1);

// 高阶函数示例
function applyOperation(array $numbers, callable $operation): array
{
    return array_map($operation, $numbers);
}

$numbers = [1, 2, 3, 4, 5];

// 使用不同的操作
$doubled = applyOperation($numbers, fn($n) => $n * 2);
$squared = applyOperation($numbers, fn($n) => $n ** 2);

print_r($doubled);  // [2, 4, 6, 8, 10]
print_r($squared);  // [1, 4, 9, 16, 25]

// 事件系统示例
class EventDispatcher
{
    private array $listeners = [];
    
    public function on(string $event, callable $listener): void
    {
        $this->listeners[$event][] = $listener;
    }
    
    public function dispatch(string $event, mixed $data = null): void
    {
        if (isset($this->listeners[$event])) {
            foreach ($this->listeners[$event] as $listener) {
                $listener($data);
            }
        }
    }
}

$dispatcher = new EventDispatcher();
$dispatcher->on('user.created', fn($user) => echo "User created: {$user}\n");
$dispatcher->dispatch('user.created', 'Alice');
```

## 注意事项

1. **数组键类型**：数组键可以是整数或字符串，浮点数会被转换为整数。

2. **对象引用**：对象赋值是引用传递，使用 `clone` 进行拷贝。

3. **可调用验证**：在调用前使用 `is_callable()` 验证，避免运行时错误。

4. **类型提示**：函数参数可以使用 `array`、`object`、`callable` 作为类型提示。

## 练习

1. 实现一个函数 `arrayGroupBy(array $array, callable $keyFunction): array`，根据键函数对数组进行分组。

2. 创建一个 `Stack` 类，使用数组实现栈数据结构（push、pop、peek、isEmpty 方法）。

3. 编写一个函数 `invoke(callable $callback, array $args = []): mixed`，支持调用各种形式的可调用类型。

4. 实现一个简单的 `Collection` 类，封装数组操作，提供 `map`、`filter`、`reduce` 等方法。

5. 创建一个事件系统，使用可调用类型实现事件的注册和触发机制。
