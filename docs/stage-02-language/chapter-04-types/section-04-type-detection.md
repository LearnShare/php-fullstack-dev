# 2.4.4 类型检测

## 概述

PHP 提供了多种方式检测变量的类型。正确使用类型检测函数可以确保代码的健壮性和安全性。

## gettype() 函数

### 基本语法

**语法**：`gettype(mixed $value): string`

**参数**：
- `$value`：要检测的值

**返回值**：返回类型的字符串描述。

### 返回值列表

| 返回值      | 说明           |
| :---------- | :------------- |
| `"boolean"` | 布尔类型       |
| `"integer"` | 整数类型       |
| `"double"`  | 浮点数类型     |
| `"string"`  | 字符串类型     |
| `"array"`  | 数组类型       |
| `"object"`  | 对象类型       |
| `"resource"` | 资源类型（PHP 7.4-） |
| `"resource (closed)"` | 已关闭的资源 |
| `"NULL"`    | null 类型      |
| `"unknown type"` | 未知类型 |

### 示例

```php
<?php
declare(strict_types=1);

echo gettype(true) . "\n";        // boolean
echo gettype(42) . "\n";          // integer
echo gettype(3.14) . "\n";        // double
echo gettype("hello") . "\n";     // string
echo gettype([1, 2, 3]) . "\n";  // array
echo gettype(new stdClass()) . "\n"; // object
echo gettype(null) . "\n";        // NULL
```

### 注意事项

**不推荐使用 `gettype()` 进行类型判断**，原因：

1. 返回字符串，容易拼写错误
2. 性能较差
3. 不够直观

**推荐使用 `is_*()` 系列函数**：

```php
<?php
declare(strict_types=1);

// 不推荐
if (gettype($value) === 'string') {
    // ...
}

// 推荐
if (is_string($value)) {
    // ...
}
```

## is_*() 系列函数

### 标量类型检测

#### `is_bool()`

**语法**：`is_bool(mixed $value): bool`

```php
<?php
declare(strict_types=1);

var_dump(is_bool(true));   // bool(true)
var_dump(is_bool(false));  // bool(true)
var_dump(is_bool(0));      // bool(false)
```

#### `is_int()` / `is_integer()` / `is_long()`

**语法**：`is_int(mixed $value): bool`

```php
<?php
declare(strict_types=1);

var_dump(is_int(42));      // bool(true)
var_dump(is_int(42.0));    // bool(false)
var_dump(is_int("42"));    // bool(false)
```

#### `is_float()` / `is_double()`

**语法**：`is_float(mixed $value): bool`

```php
<?php
declare(strict_types=1);

var_dump(is_float(3.14));  // bool(true)
var_dump(is_float(42));     // bool(false)
```

#### `is_string()`

**语法**：`is_string(mixed $value): bool`

```php
<?php
declare(strict_types=1);

var_dump(is_string("hello")); // bool(true)
var_dump(is_string(42));      // bool(false)
```

### 复合类型检测

#### `is_array()`

**语法**：`is_array(mixed $value): bool`

```php
<?php
declare(strict_types=1);

var_dump(is_array([1, 2, 3]));     // bool(true)
var_dump(is_array(['key' => 'value'])); // bool(true)
var_dump(is_array("string"));     // bool(false)
```

#### `is_object()`

**语法**：`is_object(mixed $value): bool`

```php
<?php
declare(strict_types=1);

var_dump(is_object(new stdClass())); // bool(true)
var_dump(is_object([]));             // bool(false)
```

#### `is_callable()`

**语法**：`is_callable(mixed $value, bool $syntax_only = false, string &$callable_name = null): bool`

```php
<?php
declare(strict_types=1);

function test(): void {}

var_dump(is_callable('test'));              // bool(true)
var_dump(is_callable(fn() => null));        // bool(true)
var_dump(is_callable([new stdClass(), 'method'])); // 取决于方法是否存在
```

### 特殊类型检测

#### `is_null()`

**语法**：`is_null(mixed $value): bool`

```php
<?php
declare(strict_types=1);

var_dump(is_null(null));    // bool(true)
var_dump(is_null(0));       // bool(false)
```

#### `is_resource()`

**语法**：`is_resource(mixed $value): bool`

**注意**：PHP 8.0+ 中许多资源已对象化，此函数可能返回 `false`。

```php
<?php
declare(strict_types=1);

$file = fopen('php://memory', 'r+');
// PHP 7.4-: bool(true)
// PHP 8.0+: bool(false) - 已对象化
var_dump(is_resource($file));
```

### 数值类型检测

#### `is_numeric()`

**语法**：`is_numeric(mixed $value): bool`

检测值是否为数字或数字字符串。

```php
<?php
declare(strict_types=1);

var_dump(is_numeric(42));       // bool(true)
var_dump(is_numeric("42"));     // bool(true)
var_dump(is_numeric("3.14"));   // bool(true)
var_dump(is_numeric("42abc"));  // bool(false)
var_dump(is_numeric("0x1A"));   // bool(false) - 注意：不识别十六进制字符串
```

#### `is_scalar()`

**语法**：`is_scalar(mixed $value): bool`

检测值是否为标量类型（bool、int、float、string）。

```php
<?php
declare(strict_types=1);

var_dump(is_scalar(true));      // bool(true)
var_dump(is_scalar(42));        // bool(true)
var_dump(is_scalar("hello"));   // bool(true)
var_dump(is_scalar([1, 2, 3])); // bool(false)
var_dump(is_scalar(null));      // bool(false)
```

### 迭代类型检测

#### `is_iterable()`

**语法**：`is_iterable(mixed $value): bool`

检测值是否可迭代（数组或实现了 `Traversable` 接口的对象）。

```php
<?php
declare(strict_types=1);

var_dump(is_iterable([1, 2, 3]));           // bool(true)
var_dump(is_iterable(new ArrayIterator())); // bool(true)
var_dump(is_iterable("string"));            // bool(false)
```

#### `is_countable()`

**语法**：`is_countable(mixed $value): bool`

检测值是否可计数（数组或实现了 `Countable` 接口的对象）。

```php
<?php
declare(strict_types=1);

var_dump(is_countable([1, 2, 3]));          // bool(true)
var_dump(is_countable(new ArrayObject()));  // bool(true)
var_dump(is_countable("string"));           // bool(false)
```

## instanceof 运算符

### 基本语法

**语法**：`$object instanceof ClassName`

用于检查对象是否为指定类的实例，或是否实现了指定接口。

```php
<?php
declare(strict_types=1);

class User {}

class Admin extends User {}

interface Greetable {}

class Person implements Greetable {}

$user = new User();
$admin = new Admin();
$person = new Person();

var_dump($user instanceof User);      // bool(true)
var_dump($admin instanceof User);     // bool(true) - 继承关系
var_dump($admin instanceof Admin);    // bool(true)
var_dump($person instanceof Greetable); // bool(true) - 接口实现
```

### 检查多个类型

```php
<?php
declare(strict_types=1);

function process(mixed $value): void
{
    if ($value instanceof User || $value instanceof Admin) {
        echo "User object\n";
    }
}
```

## var_dump() 和 print_r()

### var_dump()

**语法**：`var_dump(mixed ...$values): void`

输出变量的详细信息，包括类型和值。

```php
<?php
declare(strict_types=1);

$var = ['name' => 'Alice', 'age' => 25];
var_dump($var);
// 输出：
// array(2) {
//   ["name"]=>
//   string(5) "Alice"
//   ["age"]=>
//   int(25)
// }
```

### print_r()

**语法**：`print_r(mixed $value, bool $return = false): string|bool`

以更易读的格式输出变量信息。

```php
<?php
declare(strict_types=1);

$var = ['name' => 'Alice', 'age' => 25];
print_r($var);
// 输出：
// Array
// (
//     [name] => Alice
//     [age] => 25
// )
```

## 类型检测最佳实践

### 1. 优先使用 is_*() 函数

```php
<?php
declare(strict_types=1);

// 推荐
if (is_string($value)) {
    // 处理字符串
}

// 不推荐
if (gettype($value) === 'string') {
    // 处理字符串
}
```

### 2. 使用严格比较

```php
<?php
declare(strict_types=1);

// 推荐：严格比较
if ($value === null) {
    // 处理 null
}

// 不推荐：宽松比较
if ($value == null) {
    // 可能与其他值相等
}
```

### 3. 组合类型检测

```php
<?php
declare(strict_types=1);

function processValue(mixed $value): void
{
    if (is_string($value)) {
        echo "String: {$value}\n";
    } elseif (is_int($value)) {
        echo "Integer: {$value}\n";
    } elseif (is_array($value)) {
        echo "Array with " . count($value) . " elements\n";
    } elseif (is_object($value)) {
        echo "Object: " . get_class($value) . "\n";
    } else {
        echo "Unknown type: " . gettype($value) . "\n";
    }
}
```

### 4. 类型守卫模式

```php
<?php
declare(strict_types=1);

function process(int|string $value): int
{
    if (is_string($value)) {
        return (int) $value;
    }
    
    // PHP 8.0+ 类型系统知道这里 $value 是 int
    return $value;
}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class TypeChecker
{
    public static function getTypeInfo(mixed $value): array
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
            'is_iterable' => is_iterable($value),
            'is_countable' => is_countable($value),
        ];
    }
    
    public static function formatTypeInfo(mixed $value): string
    {
        $info = self::getTypeInfo($value);
        $details = [];
        
        foreach ($info as $key => $val) {
            if ($key !== 'value' && $val === true) {
                $details[] = $key;
            }
        }
        
        return sprintf(
            "Value: %s | Type: %s | Details: %s",
            var_export($info['value'], true),
            $info['type'],
            implode(', ', $details) ?: 'none'
        );
    }
}

// 使用示例
$values = [
    true,
    42,
    3.14,
    "hello",
    [1, 2, 3],
    new stdClass(),
    null
];

foreach ($values as $value) {
    echo TypeChecker::formatTypeInfo($value) . "\n";
}
```

## 注意事项

1. **性能考虑**：`is_*()` 函数比 `gettype()` 更快。

2. **类型转换**：类型检测不会改变变量的类型，只是检查。

3. **资源类型变化**：PHP 8.0+ 中资源已对象化，注意使用 `instanceof` 检查。

4. **null 检查**：使用 `=== null` 或 `is_null()`，不要使用 `== null`。

5. **类型提示**：在函数参数中使用类型提示，减少运行时类型检测的需要。

## 练习

1. 编写一个函数 `getDetailedType(mixed $value): array`，返回值的详细类型信息（类型、是否为标量、是否可迭代等）。

2. 创建一个 `TypeValidator` 类，提供各种类型验证方法，支持链式调用。

3. 实现一个函数 `assertType(mixed $value, string $expectedType): void`，如果类型不匹配则抛出异常。

4. 编写一个函数，检查值是否为数值类型（int 或 float 或数字字符串），并返回相应的数值。

5. 创建一个类型检测工具函数库，包含常用的类型检测和转换函数。
