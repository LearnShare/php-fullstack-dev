# 2.4.4 类型检测

## 概述

类型检测是确定变量类型的重要方法。本节详细介绍 `gettype()`、`is_*()` 系列函数、`instanceof` 运算符、`var_dump()` 和 `print_r()` 的使用，以及类型检测的最佳实践。

掌握类型检测方法可以帮助编写更健壮的代码，在运行时验证数据类型，避免类型相关的错误。

## 特性

- **gettype()**：返回类型的字符串表示，性能较差
- **is_*() 系列**：针对特定类型的检测函数，性能好，推荐使用
- **instanceof**：检测对象是否属于某个类或其子类
- **var_dump()**：显示变量的类型和值，用于调试
- **print_r()**：以可读格式显示变量，用于调试

## 语法/定义

### gettype() - 获取类型字符串

**语法**：`gettype(mixed $value): string`

**参数**：
- `$value`：要检测的值（任意类型）

**返回值**：返回类型的字符串表示，可能的值：
- `"boolean"`：布尔类型
- `"integer"`：整数类型
- `"double"`：浮点数类型（注意：不是 "float"）
- `"string"`：字符串类型
- `"array"`：数组类型
- `"object"`：对象类型
- `"resource"`：资源类型（PHP 7.x）
- `"resource (closed)"`：已关闭的资源
- `"NULL"`：null 类型
- `"unknown type"`：未知类型

**特点**：
- 返回类型字符串，便于显示和比较
- 性能较差，不推荐用于性能敏感的场景
- 适合调试和日志记录

### is_*() 系列函数

**常用函数**：

| 函数 | 检测类型 | 返回值 |
|:-----|:---------|:-------|
| `is_bool($value)` | 布尔类型 | `bool` |
| `is_int($value)` | 整数类型 | `bool` |
| `is_float($value)` | 浮点数类型 | `bool` |
| `is_string($value)` | 字符串类型 | `bool` |
| `is_array($value)` | 数组类型 | `bool` |
| `is_object($value)` | 对象类型 | `bool` |
| `is_null($value)` | null 类型 | `bool` |
| `is_resource($value)` | 资源类型 | `bool` |
| `is_callable($value)` | 可调用类型 | `bool` |
| `is_numeric($value)` | 数字字符串或数字 | `bool` |
| `is_scalar($value)` | 标量类型 | `bool` |

**语法**：`is_type(mixed $value): bool`

**参数**：
- `$value`：要检测的值（任意类型）

**返回值**：如果 `$value` 是指定类型返回 `true`，否则返回 `false`

**特点**：
- 性能好，推荐使用
- 不进行类型转换，严格检测
- 返回布尔值，适合条件判断

### instanceof - 对象类型检测

**语法**：`$object instanceof ClassName`

**参数**：
- `$object`：要检测的对象
- `ClassName`：类名（可以是字符串或 `::class` 常量）

**返回值**：如果 `$object` 是 `ClassName` 的实例或其子类的实例返回 `true`，否则返回 `false`

**特点**：
- 检测对象是否属于某个类
- 支持继承关系检测
- 支持接口检测

### var_dump() - 详细输出

**语法**：`var_dump(mixed ...$values): void`

**参数**：
- `...$values`：可变参数，要输出的变量（可传入多个变量）

**返回值**：无返回值，直接输出到标准输出

**特点**：
- 显示变量的类型和值
- 递归显示数组和对象
- 输出格式详细但不易阅读
- 适合调试

### print_r() - 可读输出

**语法**：`print_r(mixed $value, bool $return = false): string|bool`

**参数**：
- `$value`：要输出的变量（任意类型）
- `$return`：可选，如果为 `true`，返回字符串而不是直接输出

**返回值**：
- 如果 `$return` 为 `false`：返回 `true`
- 如果 `$return` 为 `true`：返回格式化的字符串

**特点**：
- 输出格式更简洁易读
- 适合查看数组和对象结构
- 可以返回字符串用于后续处理

## 基本用法

### 示例 1：gettype() 使用

```php
<?php
declare(strict_types=1);

// 不同类型
var_dump(gettype(true));      // string(7) "boolean"
var_dump(gettype(42));        // string(7) "integer"
var_dump(gettype(99.99));     // string(6) "double"
var_dump(gettype("Hello"));   // string(6) "string"
var_dump(gettype([1, 2]));    // string(5) "array"
var_dump(gettype(null));      // string(4) "NULL"

// 对象
class User {}
$user = new User();
var_dump(gettype($user));     // string(6) "object"
```

**执行**：

```bash
php gettype-example.php
```

**输出**：

```
string(7) "boolean"
string(7) "integer"
string(6) "double"
string(6) "string"
string(5) "array"
string(4) "NULL"
string(6) "object"
```

### 示例 2：is_*() 系列函数

```php
<?php
declare(strict_types=1);

$values = [
    true,
    42,
    99.99,
    "Hello",
    [1, 2],
    null,
    new stdClass(),
];

foreach ($values as $value) {
    echo "Value: ";
    var_dump($value);
    echo "is_bool: " . (is_bool($value) ? 'true' : 'false') . "\n";
    echo "is_int: " . (is_int($value) ? 'true' : 'false') . "\n";
    echo "is_float: " . (is_float($value) ? 'true' : 'false') . "\n";
    echo "is_string: " . (is_string($value) ? 'true' : 'false') . "\n";
    echo "is_array: " . (is_array($value) ? 'true' : 'false') . "\n";
    echo "is_null: " . (is_null($value) ? 'true' : 'false') . "\n";
    echo "is_object: " . (is_object($value) ? 'true' : 'false') . "\n";
    echo "is_scalar: " . (is_scalar($value) ? 'true' : 'false') . "\n";
    echo "---\n";
}
```

### 示例 3：instanceof 使用

```php
<?php
declare(strict_types=1);

class Animal {}
class Dog extends Animal {}
class Cat extends Animal {}

$dog = new Dog();
$cat = new Cat();

// 检测对象类型
var_dump($dog instanceof Dog);    // bool(true)
var_dump($dog instanceof Animal); // bool(true) - 继承关系
var_dump($dog instanceof Cat);   // bool(false)

// 使用 ::class 常量
var_dump($dog instanceof Dog::class);    // bool(true)
var_dump($dog instanceof Animal::class); // bool(true)

// 接口检测
interface Flyable {}
class Bird extends Animal implements Flyable {}

$bird = new Bird();
var_dump($bird instanceof Flyable); // bool(true)
```

### 示例 4：var_dump() 和 print_r()

```php
<?php
declare(strict_types=1);

$data = [
    'name' => 'John',
    'age' => 25,
    'hobbies' => ['reading', 'coding'],
];

// var_dump：详细输出
echo "=== var_dump ===\n";
var_dump($data);

// print_r：可读输出
echo "\n=== print_r ===\n";
print_r($data);

// print_r 返回字符串
echo "\n=== print_r (return) ===\n";
$output = print_r($data, true);
echo $output;
```

## 完整代码示例

### 示例 1：类型检测工具函数

```php
<?php
declare(strict_types=1);

function getTypeInfo(mixed $value): array
{
    return [
        'value' => $value,
        'type' => gettype($value),
        'is_bool' => is_bool($value),
        'is_int' => is_int($value),
        'is_float' => is_float($value),
        'is_string' => is_string($value),
        'is_array' => is_array($value),
        'is_object' => is_object($value),
        'is_null' => is_null($value),
        'is_scalar' => is_scalar($value),
        'is_numeric' => is_numeric($value),
    ];
}

$values = [true, 42, 99.99, "Hello", [1, 2], null];

foreach ($values as $value) {
    $info = getTypeInfo($value);
    echo "Value: ";
    var_dump($value);
    echo "Type: {$info['type']}\n";
    echo "Is scalar: " . ($info['is_scalar'] ? 'yes' : 'no') . "\n";
    echo "---\n";
}
```

### 示例 2：类型安全函数

```php
<?php
declare(strict_types=1);

function addNumbers(mixed $a, mixed $b): int
{
    // 类型检测
    if (!is_numeric($a) || !is_numeric($b)) {
        throw new TypeError("Arguments must be numeric");
    }
    
    // 转换为整数
    return (int) $a + (int) $b;
}

try {
    echo addNumbers(10, 20) . "\n";        // 30
    echo addNumbers("10", "20") . "\n";     // 30
    echo addNumbers(10.5, 20.7) . "\n";     // 30
    echo addNumbers("10", "abc") . "\n";    // TypeError
} catch (TypeError $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
```

### 示例 3：对象类型检测

```php
<?php
declare(strict_types=1);

interface PaymentMethod {}
class CreditCard implements PaymentMethod {}
class PayPal implements PaymentMethod {}

function processPayment(PaymentMethod $method, float $amount): void
{
    // 使用 instanceof 检测具体类型
    if ($method instanceof CreditCard) {
        echo "Processing credit card payment: \${$amount}\n";
    } elseif ($method instanceof PayPal) {
        echo "Processing PayPal payment: \${$amount}\n";
    } else {
        echo "Processing unknown payment method: \${$amount}\n";
    }
}

$card = new CreditCard();
$paypal = new PayPal();

processPayment($card, 100.00);
processPayment($paypal, 50.00);
```

## 使用场景

### gettype()

- **调试和日志**：记录变量的类型信息
- **类型字符串比较**：需要类型字符串时使用
- **不推荐用于性能敏感场景**：性能较差

### is_*() 系列函数

- **类型检查**：在函数中检查参数类型
- **条件判断**：根据类型执行不同逻辑
- **类型验证**：验证用户输入或外部数据
- **推荐使用**：性能好，代码清晰

### instanceof

- **对象类型检测**：检测对象是否属于某个类
- **多态处理**：根据对象类型执行不同逻辑
- **接口检测**：检测对象是否实现了某个接口
- **设计模式**：实现策略模式、工厂模式等

### var_dump() 和 print_r()

- **调试输出**：查看变量的类型和值
- **开发调试**：快速了解变量内容
- **CLI 环境**：在命令行环境下使用
- **详细说明**：见 [2.2.3 调试函数和策略](../chapter-02-output/section-03-debugging.md)

## 注意事项

### 性能考虑

- **优先使用 is_*()**：`is_*()` 函数性能好，推荐使用
- **避免 gettype()**：`gettype()` 性能较差，除非确实需要类型字符串
- **缓存结果**：如果多次使用类型检测结果，可以缓存

### 类型转换

- **不进行类型转换**：类型检测函数不进行类型转换，严格检测
- **理解区别**：`is_numeric()` 会检测数字字符串，`is_int()` 只检测整数类型
- **严格模式**：在严格模式下，类型转换受到限制

### 对象检测

- **使用 instanceof**：检测对象类型时使用 `instanceof`
- **支持继承**：`instanceof` 支持继承关系检测
- **接口检测**：可以使用 `instanceof` 检测接口

## 常见问题

### 问题 1：gettype() 性能问题

**症状**：代码执行缓慢

**原因**：频繁使用 `gettype()` 导致性能问题

**错误示例**：

```php
<?php
declare(strict_types=1);

function processValue(mixed $value): void
{
    $type = gettype($value);  // 性能较差
    if ($type === "string") {
        // 处理字符串
    }
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

function processValue(mixed $value): void
{
    if (is_string($value)) {  // 性能好
        // 处理字符串
    }
}
```

### 问题 2：类型检测不准确

**症状**：类型检测结果不符合预期

**原因**：不理解类型检测的规则

**示例**：

```php
<?php
declare(strict_types=1);

$value = "42";
var_dump(is_int($value));     // bool(false) - 字符串不是整数
var_dump(is_numeric($value)); // bool(true) - 是数字字符串
var_dump((int) $value);       // int(42) - 可以转换为整数
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$value = "42";

// 检测是否为整数类型
if (is_int($value)) {
    // 处理整数
}

// 检测是否为数字（包括数字字符串）
if (is_numeric($value)) {
    $intValue = (int) $value;
    // 处理数字
}
```

### 问题 3：instanceof 使用错误

**症状**：对象类型检测失败

**原因**：类名拼写错误或命名空间问题

**错误示例**：

```php
<?php
declare(strict_types=1);

namespace App;

class User {}

$user = new User();
var_dump($user instanceof User);  // 可能失败，取决于上下文
```

**解决方法**：

```php
<?php
declare(strict_types=1);

namespace App;

class User {}

$user = new User();

// 方法1：使用完全限定类名
var_dump($user instanceof \App\User);

// 方法2：使用 ::class 常量
var_dump($user instanceof User::class);

// 方法3：在当前命名空间内
var_dump($user instanceof User);  // 如果当前在 App 命名空间内
```

## 最佳实践

### 类型检测选择

- **优先使用 is_*()**：性能好，代码清晰
- **避免 gettype()**：除非确实需要类型字符串
- **使用 instanceof**：检测对象类型
- **组合使用**：根据需要组合使用多种检测方法

### 性能优化

- **缓存结果**：如果多次使用，缓存类型检测结果
- **提前返回**：使用类型检测提前返回，避免不必要的处理
- **避免重复检测**：避免在循环中重复检测相同变量

### 代码可读性

- **使用有意义的变量名**：类型检测结果存储在有意义的变量中
- **添加注释**：为复杂的类型检测逻辑添加注释
- **提取函数**：将类型检测逻辑提取到函数中

## 对比分析

### gettype() vs is_*()

| 特性 | gettype() | is_*() |
|:-----|:----------|:-------|
| 返回值 | 字符串 | 布尔值 |
| 性能 | 较差 | 好 |
| 使用场景 | 调试、日志 | 类型检查 |
| 推荐度 | 不推荐 | 推荐 |

**选择建议**：
- **类型检查**：使用 `is_*()` 函数
- **调试日志**：可以使用 `gettype()`，但不推荐用于性能敏感场景

### instanceof vs is_object()

| 特性 | instanceof | is_object() |
|:-----|:-----------|:------------|
| 检测内容 | 对象类型 | 是否为对象 |
| 支持继承 | 是 | 否 |
| 支持接口 | 是 | 否 |
| 推荐度 | 推荐（对象） | 推荐（通用） |

**选择建议**：
- **检测对象类型**：使用 `instanceof`
- **检测是否为对象**：使用 `is_object()`

## 相关章节

- **2.4.1 标量类型**：了解基本数据类型
- **2.4.2 复合类型**：了解复合数据类型
- **2.2.3 调试函数和策略**：详细了解 var_dump 和 print_r
- **2.5 类型转换与比较**：了解类型转换规则

## 练习任务

1. **类型检测函数练习**：
   - 使用 `is_*()` 系列函数检测不同类型
   - 使用 `gettype()` 获取类型字符串
   - 理解不同检测函数的区别
   - 测试性能差异

2. **instanceof 练习**：
   - 使用 `instanceof` 检测对象类型
   - 测试继承关系的检测
   - 测试接口检测
   - 理解命名空间的影响

3. **类型安全函数练习**：
   - 创建类型安全的函数
   - 使用类型检测验证参数
   - 处理类型错误
   - 提供清晰的错误信息

4. **调试工具练习**：
   - 使用 `var_dump()` 和 `print_r()` 调试
   - 创建类型检测工具函数
   - 实现类型信息输出函数
   - 测试不同场景下的输出

5. **综合练习**：
   - 创建一个类型检测工具类
   - 实现多种类型检测方法
   - 优化类型检测性能
   - 进行代码审查，确保类型检测正确
