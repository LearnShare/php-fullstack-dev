# 2.5.2 显式转换（Cast）

## 概述

显式转换（Type Casting）使用类型转换操作符将值从一种类型转换为另一种类型。与隐式转换不同，显式转换明确表达了开发者的意图，使代码更加清晰和安全。

## 基本语法

```php
(type) $value
```

## 支持的转换类型

| 转换   | 语法示例          | 说明                               |
| :----- | :---------------- | :--------------------------------- |
| `int`  | `(int) $value`    | 等价 `intval($value)`              |
| `float` | `(float) $value` | 等价 `floatval($value)`            |
| `bool` | `(bool) $value`   | 等价 `boolval($value)`             |
| `string` | `(string) $value` | 等价 `strval($value)`            |
| `array` | `(array) $value` | 非数组转换为一元素数组             |
| `object` | `(object) $value` | 数组变对象或包装标量              |

## 转换为整数（int）

### 基本规则

- 浮点数：截断小数部分
- 字符串：从开头解析数字，遇到非数字字符停止
- 布尔值：`true` 转为 `1`，`false` 转为 `0`
- 数组：空数组转为 `0`，非空数组转为 `1`
- 对象：尝试调用 `__toString()`，然后解析
- `null`：转为 `0`

### 示例

```php
<?php
declare(strict_types=1);

echo (int) 3.14 . "\n";        // 3
echo (int) 3.99 . "\n";        // 3（截断，不四舍五入）
echo (int) "42" . "\n";        // 42
echo (int) "42abc" . "\n";     // 42（忽略非数字字符）
echo (int) "abc42" . "\n";     // 0（无法解析）
echo (int) true . "\n";        // 1
echo (int) false . "\n";       // 0
echo (int) [] . "\n";          // 0
echo (int) [1, 2, 3] . "\n";   // 1
echo (int) null . "\n";        // 0
```

## 转换为浮点数（float）

### 基本规则

- 整数：直接转换
- 字符串：从开头解析数字，遇到非数字字符停止
- 布尔值：`true` 转为 `1.0`，`false` 转为 `0.0`
- 数组：空数组转为 `0.0`，非空数组转为 `1.0`
- `null`：转为 `0.0`

### 示例

```php
<?php
declare(strict_types=1);

echo (float) 42 . "\n";        // 42.0
echo (float) "3.14" . "\n";    // 3.14
echo (float) "3.14abc" . "\n"; // 3.14
echo (float) "abc3.14" . "\n"; // 0.0
echo (float) true . "\n";      // 1.0
echo (float) false . "\n";     // 0.0
echo (float) null . "\n";      // 0.0
```

## 转换为布尔值（bool）

### 基本规则

以下值转为 `false`：
- `false` 本身
- `0`（整数）
- `0.0`（浮点数）
- `"0"`（字符串）
- `""`（空字符串）
- `[]`（空数组）
- `null`

所有其他值转为 `true`。

### 示例

```php
<?php
declare(strict_types=1);

var_dump((bool) 0);        // bool(false)
var_dump((bool) 1);        // bool(true)
var_dump((bool) -1);       // bool(true)
var_dump((bool) 0.0);      // bool(false)
var_dump((bool) 0.1);      // bool(true)
var_dump((bool) "0");      // bool(false)
var_dump((bool) "1");      // bool(true)
var_dump((bool) "");       // bool(false)
var_dump((bool) "false");  // bool(true) - 注意：非空字符串
var_dump((bool) []);       // bool(false)
var_dump((bool) [1, 2]);   // bool(true)
var_dump((bool) null);     // bool(false)
```

## 转换为字符串（string）

### 基本规则

- 数字：直接转为字符串形式
- 布尔值：`true` 转为 `"1"`，`false` 转为 `""`
- 数组：转为 `"Array"`
- 对象：尝试调用 `__toString()` 方法
- `null`：转为 `""`

### 示例

```php
<?php
declare(strict_types=1);

echo (string) 42 . "\n";        // "42"
echo (string) 3.14 . "\n";      // "3.14"
echo (string) true . "\n";      // "1"
echo (string) false . "\n";     // ""（空字符串）
echo (string) [1, 2, 3] . "\n"; // "Array"
echo (string) null . "\n";      // ""
```

## 转换为数组（array）

### 基本规则

- 标量值：转为包含一个元素的数组 `[value]`
- 对象：转为关联数组，属性名作为键
- `null`：转为空数组 `[]`

### 示例

```php
<?php
declare(strict_types=1);

print_r((array) 42);        // Array([0] => 42)
print_r((array) "hello");   // Array([0] => hello)
print_r((array) null);      // Array()

// 对象转数组
class User
{
    public string $name = 'Alice';
    private int $age = 25;
}

$user = new User();
print_r((array) $user);
// 输出：
// Array
// (
//     [name] => Alice
//     [Userage] => 25  // 注意：私有属性名会包含类名
// )
```

## 转换为对象（object）

### 基本规则

- 数组：转为 `stdClass` 对象，数组键作为属性名
- 标量值：转为包含 `scalar` 属性的 `stdClass` 对象
- `null`：转为空 `stdClass` 对象

### 示例

```php
<?php
declare(strict_types=1);

$arr = ['name' => 'Alice', 'age' => 25];
$obj = (object) $arr;
echo $obj->name . "\n";  // Alice
echo $obj->age . "\n";   // 25

$scalar = (object) 42;
echo $scalar->scalar . "\n";  // 42

$null = (object) null;
var_dump($null);  // object(stdClass)#1 (0) {}
```

## 完整示例

```php
<?php
declare(strict_types=1);

class TypeConverter
{
    public static function convert(mixed $value, string $type): mixed
    {
        return match ($type) {
            'int' => (int) $value,
            'float' => (float) $value,
            'bool' => (bool) $value,
            'string' => (string) $value,
            'array' => (array) $value,
            'object' => (object) $value,
            default => throw new InvalidArgumentException("Unknown type: {$type}")
        };
    }
    
    public static function demonstrate(): void
    {
        $values = [42, 3.14, true, "hello", [1, 2, 3], null];
        $types = ['int', 'float', 'bool', 'string', 'array', 'object'];
        
        foreach ($values as $value) {
            echo "Original: " . var_export($value, true) . "\n";
            foreach ($types as $type) {
                $converted = self::convert($value, $type);
                echo "  -> {$type}: " . var_export($converted, true) . "\n";
            }
            echo "\n";
        }
    }
}

TypeConverter::demonstrate();
```

## 注意事项

1. **精度损失**：浮点数转整数会截断小数部分，不进行四舍五入。

2. **字符串解析**：字符串转数字时，从开头解析，遇到非数字字符停止。

3. **数组转换**：对象转数组时，私有和受保护属性的名称会包含类名。

4. **布尔转换**：注意 `"0"` 和空字符串在布尔上下文中为 `false`。

5. **显式优于隐式**：始终使用显式转换，避免依赖隐式转换。

## 练习

1. 编写一个函数 `safeCast(mixed $value, string $type): mixed`，安全地进行类型转换，处理边界情况。

2. 创建一个类型转换工具类，提供各种类型转换方法，并包含详细的文档说明。

3. 实现一个函数，将用户输入转换为指定类型，并提供错误处理。

4. 编写测试用例，验证各种类型转换的边界情况。

5. 创建一个类型转换演示程序，展示所有支持的转换类型和规则。
