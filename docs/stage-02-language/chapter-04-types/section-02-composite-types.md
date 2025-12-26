# 2.4.2 复合类型

## 概述

复合类型是可以包含多个元素或方法的数据类型，包括数组类型（`array`）、对象类型（`object`）、可调用类型（`callable`）。本节详细介绍这三种复合类型的语法、特性、操作方法和使用场景。

理解复合类型是学习 PHP 的重要部分。数组是 PHP 中最常用的数据结构，对象是面向对象编程的基础，可调用类型提供了灵活的回调机制。

## 特性

- **数组类型（array）**：有序映射，可同时包含数字键与字符串键，是 PHP 的核心数据结构
- **对象类型（object）**：类的实例，包含属性和方法，支持面向对象编程
- **可调用类型（callable）**：函数、方法、闭包等可调用的值，支持回调机制

## 语法/定义

### 数组类型（array）

**创建方式**：
- **短数组语法**：`[]`（PHP 5.4+，推荐）
- **长数组语法**：`array()`（兼容旧版本）

**类型检测**：`is_array(mixed $value): bool`

**参数**：
- `$value`：要检测的值（任意类型）

**返回值**：如果 `$value` 是数组类型返回 `true`，否则返回 `false`

**特点**：
- 有序映射，键可以是整数或字符串
- 值可以是任意类型
- 支持多维数组
- 详细说明见 [2.8 数组完整指南](../chapter-08-arrays/readme.md)

### 对象类型（object）

**创建方式**：通过 `new` 关键字创建类的实例

**类型检测**：
- `is_object(mixed $value): bool`：检测是否为对象类型
- `$object instanceof ClassName`：检测对象是否属于某个类

**特点**：
- 包含属性和方法
- 支持继承和多态
- 对象赋值是引用赋值
- 详细说明见阶段三：面向对象编程基础

### 可调用类型（callable）

**形式**：
- **函数名**：`'function_name'`（字符串）
- **方法**：`[$object, 'method']` 或 `[ClassName::class, 'method']`（数组）
- **静态方法**：`'ClassName::method'`（字符串，PHP 5.2.3+）
- **闭包**：`function() { ... }` 或 `fn() => ...`（PHP 7.4+）
- **实现了 `__invoke()` 的对象**：任何实现了 `__invoke()` 魔术方法的对象

**类型检测**：`is_callable(mixed $value, bool $syntax_only = false, string &$callable_name = null): bool`

**参数**：
- `$value`：要检测的值（任意类型）
- `$syntax_only`：可选，如果为 `true`，只检查语法，不检查函数是否存在
- `$callable_name`：可选，如果提供，会接收可调用名称

**返回值**：如果 `$value` 是可调用类型返回 `true`，否则返回 `false`

## 基本用法

### 示例 1：数组类型

```php
<?php
declare(strict_types=1);

// 索引数组
$indexed = [1, 2, 3];
echo "Indexed: " . json_encode($indexed) . "\n";

// 关联数组
$associative = ['name' => 'John', 'age' => 25];
echo "Associative: " . json_encode($associative) . "\n";

// 混合数组
$mixed = [0 => 'first', 'key' => 'value', 1 => 'second'];
echo "Mixed: " . json_encode($mixed) . "\n";

// 多维数组
$multi = [
    'user' => [
        'name' => 'John',
        'age' => 25,
        'hobbies' => ['reading', 'coding']
    ]
];
echo "Multi: " . json_encode($multi) . "\n";

// 类型检测
var_dump(is_array($indexed));      // bool(true)
var_dump(is_array("not array"));    // bool(false)
```

**执行**：

```bash
php array-example.php
```

**输出**：

```
Indexed: [1,2,3]
Associative: {"name":"John","age":25}
Mixed: {"0":"first","key":"value","1":"second"}
Multi: {"user":{"name":"John","age":25,"hobbies":["reading","coding"]}}
bool(true)
bool(false)
```

### 示例 2：对象类型

```php
<?php
declare(strict_types=1);

// 定义类
class User
{
    public function __construct(
        public string $name,
        public int $age
    ) {}
    
    public function greet(): string
    {
        return "Hello, I'm {$this->name}";
    }
}

// 创建对象
$user = new User("John", 25);
echo $user->greet() . "\n";

// 类型检测
var_dump(is_object($user));              // bool(true)
var_dump($user instanceof User);         // bool(true)
var_dump(is_object("not object"));       // bool(false)
```

**执行**：

```bash
php object-example.php
```

**输出**：

```
Hello, I'm John
bool(true)
bool(true)
bool(false)
```

### 示例 3：可调用类型

```php
<?php
declare(strict_types=1);

// 函数名
$func1 = 'strlen';
var_dump(is_callable($func1));  // bool(true)
echo $func1("Hello") . "\n";    // 5

// 方法
class Calculator
{
    public function add(int $a, int $b): int
    {
        return $a + $b;
    }
    
    public static function multiply(int $a, int $b): int
    {
        return $a * $b;
    }
}

$calc = new Calculator();
$method1 = [$calc, 'add'];
var_dump(is_callable($method1));  // bool(true)
echo $method1(10, 20) . "\n";     // 30

// 静态方法
$method2 = [Calculator::class, 'multiply'];
var_dump(is_callable($method2));  // bool(true)
echo $method2(10, 20) . "\n";    // 200

// 闭包
$closure = function(int $x): int {
    return $x * 2;
};
var_dump(is_callable($closure));  // bool(true)
echo $closure(5) . "\n";          // 10

// 实现了 __invoke() 的对象
class Multiplier
{
    public function __construct(
        private int $factor
    ) {}
    
    public function __invoke(int $x): int
    {
        return $x * $this->factor;
    }
}

$multiplier = new Multiplier(3);
var_dump(is_callable($multiplier));  // bool(true)
echo $multiplier(5) . "\n";          // 15
```

**执行**：

```bash
php callable-example.php
```

**输出**：

```
bool(true)
5
bool(true)
30
bool(true)
200
bool(true)
10
bool(true)
15
```

## 完整代码示例

### 示例 1：数组操作

```php
<?php
declare(strict_types=1);

// 创建数组
$users = [
    ['id' => 1, 'name' => 'Alice', 'age' => 25],
    ['id' => 2, 'name' => 'Bob', 'age' => 30],
    ['id' => 3, 'name' => 'Charlie', 'age' => 28],
];

// 数组操作
echo "Count: " . count($users) . "\n";
echo "First user: " . json_encode($users[0]) . "\n";

// 添加元素
$users[] = ['id' => 4, 'name' => 'David', 'age' => 32];
echo "After add: " . count($users) . "\n";

// 遍历数组
foreach ($users as $user) {
    echo "User: {$user['name']}, Age: {$user['age']}\n";
}

// 数组函数
$names = array_column($users, 'name');
echo "Names: " . implode(', ', $names) . "\n";
```

### 示例 2：对象操作

```php
<?php
declare(strict_types=1);

class Product
{
    public function __construct(
        public int $id,
        public string $name,
        public float $price
    ) {}
    
    public function getFormattedPrice(): string
    {
        return sprintf("$%.2f", $this->price);
    }
}

// 创建对象
$product = new Product(1, "iPhone", 999.99);
echo "Product: {$product->name}\n";
echo "Price: {$product->getFormattedPrice()}\n";

// 对象克隆
$product2 = clone $product;
$product2->name = "iPad";
echo "Product 2: {$product2->name}\n";
echo "Original: {$product->name}\n";
```

### 示例 3：可调用类型应用

```php
<?php
declare(strict_types=1);

// 使用可调用类型实现策略模式
function processNumbers(array $numbers, callable $processor): array
{
    return array_map($processor, $numbers);
}

$numbers = [1, 2, 3, 4, 5];

// 使用函数
$doubled = processNumbers($numbers, fn($x) => $x * 2);
echo "Doubled: " . implode(', ', $doubled) . "\n";

// 使用闭包
$squared = processNumbers($numbers, function($x) {
    return $x * $x;
});
echo "Squared: " . implode(', ', $squared) . "\n";

// 使用类方法
class Math
{
    public static function cube(int $x): int
    {
        return $x * $x * $x;
    }
}

$cubed = processNumbers($numbers, [Math::class, 'cube']);
echo "Cubed: " . implode(', ', $cubed) . "\n";
```

## 使用场景

### 数组类型

- **数据集合**：存储多个相关数据
- **配置信息**：存储应用配置
- **映射关系**：键值对映射
- **数据结构**：实现栈、队列等数据结构

### 对象类型

- **面向对象编程**：封装数据和行为
- **数据模型**：表示业务实体
- **设计模式**：实现各种设计模式
- **代码组织**：组织相关代码

### 可调用类型

- **回调函数**：事件处理、异步操作
- **策略模式**：根据条件选择不同处理方式
- **函数式编程**：高阶函数、函数组合
- **事件系统**：事件监听和处理

## 注意事项

### 数组类型

- **键类型**：键可以是整数或字符串，整数键会被重新索引
- **有序性**：数组是有序的，插入顺序会被保留
- **性能**：大数组操作可能影响性能
- **详细说明**：见 [2.8 数组完整指南](../chapter-08-arrays/readme.md)

### 对象类型

- **引用赋值**：对象赋值是引用赋值，使用 `clone` 创建副本
- **类型检测**：使用 `instanceof` 检测对象类型
- **详细说明**：见阶段三：面向对象编程基础

### 可调用类型

- **验证可调用性**：使用 `is_callable()` 检查是否可调用
- **函数存在性**：使用 `function_exists()` 检查函数是否存在
- **方法存在性**：使用 `method_exists()` 检查方法是否存在

## 常见问题

### 问题 1：数组键类型混淆

**症状**：数组行为不符合预期

**原因**：不理解数组键的类型转换规则

**示例**：

```php
<?php
declare(strict_types=1);

$arr = [];
$arr[1] = "one";
$arr["1"] = "one string";  // 会覆盖 $arr[1]
$arr[01] = "octal";         // 等同于 $arr[1]

print_r($arr);  // 只包含一个元素
```

**解决方法**：

```php
<?php
declare(strict_types=1);

// 理解键的类型转换
$arr = [];
$arr[1] = "one";
$arr["2"] = "two";  // 字符串键 "2" 和整数键 2 不同
$arr[2] = "two int";

print_r($arr);  // 包含三个元素
```

### 问题 2：对象引用问题

**症状**：修改一个对象，另一个对象也被修改

**原因**：对象赋值是引用赋值

**错误示例**：

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(public string $name) {}
}

$user1 = new User("Alice");
$user2 = $user1;  // 引用赋值
$user2->name = "Bob";

echo $user1->name;  // "Bob"（也被修改了）
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$user1 = new User("Alice");
$user2 = clone $user1;  // 克隆对象
$user2->name = "Bob";

echo $user1->name;  // "Alice"（未被修改）
```

### 问题 3：可调用类型不存在

**症状**：调用可调用类型时报错

**原因**：函数或方法不存在

**解决方法**：

```php
<?php
declare(strict_types=1);

$func = "nonExistentFunction";

// 检查是否可调用
if (is_callable($func)) {
    $result = $func();
} else {
    echo "Function is not callable\n";
}

// 检查函数是否存在
if (function_exists($func)) {
    $result = $func();
} else {
    echo "Function does not exist\n";
}
```

## 最佳实践

### 数组类型

- **使用短数组语法**：优先使用 `[]` 而不是 `array()`
- **理解数组本质**：数组是有序映射，不是简单的列表
- **使用数组函数**：利用 PHP 提供的数组函数提高效率
- **类型声明**：为数组参数添加类型声明

### 对象类型

- **使用类型声明**：为对象参数添加类型声明
- **使用 instanceof**：检测对象类型
- **理解引用和克隆**：理解对象赋值和克隆的区别
- **封装原则**：遵循面向对象的封装原则

### 可调用类型

- **验证可调用性**：使用 `is_callable()` 检查
- **使用类型声明**：为可调用参数添加 `callable` 类型声明
- **文档说明**：为可调用参数添加文档说明
- **错误处理**：处理可调用不存在的情况

## 对比分析

### 数组 vs 对象

| 特性 | 数组 | 对象 |
|:-----|:-----|:-----|
| 访问方式 | `$arr['key']` | `$obj->property` |
| 方法 | 不支持 | 支持 |
| 类型安全 | 弱 | 强 |
| 性能 | 好 | 略差 |
| 适用场景 | 数据集合 | 数据和行为封装 |

**选择建议**：
- **简单数据集合**：使用数组
- **需要方法**：使用对象
- **配置信息**：使用数组或配置类

### 函数 vs 闭包 vs 方法

| 特性 | 函数 | 闭包 | 方法 |
|:-----|:-----|:-----|:------|
| 作用域 | 全局 | 局部 | 类 |
| 状态 | 无 | 有（捕获变量） | 有（对象状态） |
| 性能 | 最好 | 中等 | 中等 |
| 适用场景 | 通用功能 | 回调、高阶函数 | 对象行为 |

**选择建议**：
- **通用功能**：使用函数
- **回调函数**：使用闭包
- **对象行为**：使用方法

## 相关章节

- **2.4.1 标量类型**：了解基本数据类型
- **2.8 数组完整指南**：详细了解数组操作
- **阶段三：面向对象编程基础**：详细了解对象和类
- **2.10 函数与作用域**：详细了解函数和闭包

## 练习任务

1. **数组操作练习**：
   - 创建不同类型的数组（索引、关联、多维）
   - 练习数组的增删改查操作
   - 使用数组函数处理数据
   - 理解数组键的类型

2. **对象操作练习**：
   - 创建类和对象
   - 练习对象的属性和方法访问
   - 练习对象克隆
   - 理解对象引用

3. **可调用类型练习**：
   - 使用函数名作为可调用
   - 使用方法作为可调用
   - 使用闭包作为可调用
   - 实现可调用对象

4. **综合练习**：
   - 创建一个数据处理工具类
   - 使用数组存储数据
   - 使用对象封装逻辑
   - 使用可调用类型实现策略模式

5. **最佳实践练习**：
   - 为函数添加类型声明
   - 验证可调用类型
   - 理解数组和对象的选择
   - 进行代码审查
