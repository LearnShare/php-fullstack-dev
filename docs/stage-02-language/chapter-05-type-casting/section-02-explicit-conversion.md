# 2.5.2 显式转换

## 概述

显式转换是使用类型转换操作符主动进行的类型转换。本节详细介绍类型转换操作符的语法、各种类型的转换规则（int、float、bool、string、array、object）、转换示例及注意事项。

显式转换提供了对类型转换的明确控制，提高了代码的可读性和可维护性。理解各种类型的转换规则可以帮助编写更可靠的代码。

## 特性

- **明确控制**：明确指定要转换的目标类型
- **可读性**：代码意图更清晰，易于理解
- **类型安全**：避免隐式转换的陷阱
- **灵活性**：支持多种类型的转换

## 语法/定义

### 类型转换操作符

**基本语法**：`(type) $value`

**支持的类型**：
- `(int)` 或 `(integer)`：转换为整数
- `(float)` 或 `(double)` 或 `(real)`：转换为浮点数
- `(bool)` 或 `(boolean)`：转换为布尔值
- `(string)`：转换为字符串
- `(array)`：转换为数组
- `(object)`：转换为对象

**特点**：
- 是语言结构，不是函数
- 优先级较高
- 可以嵌套使用

### 整数转换（(int) 或 (integer)）

**转换规则**：
- **浮点数**：向下取整（使用 `floor()`）
- **字符串**：从开头解析数字，遇到非数字字符停止
- **布尔值**：`true` → 1，`false` → 0
- **null**：转换为 0
- **数组**：空数组转换为 0，非空数组转换为 1
- **对象**：如果对象实现了 `__toString()`，先转换为字符串再解析；否则会产生警告并返回 1

### 浮点数转换（(float) 或 (double) 或 (real)）

**转换规则**：
- **整数**：直接转换为浮点数（如 `42` → `42.0`）
- **字符串**：从开头解析数字，支持小数点和科学计数法
- **布尔值**：`true` → 1.0，`false` → 0.0
- **null**：转换为 0.0
- **数组**：空数组转换为 0.0，非空数组转换为 1.0
- **对象**：如果对象实现了 `__toString()`，先转换为字符串再解析；否则会产生警告并返回 1.0

### 布尔转换（(bool) 或 (boolean)）

**转换规则**：
- **转换为 false**：`false`、`0`、`0.0`、`""`、`"0"`、`[]`、`null`
- **转换为 true**：其他所有值

### 字符串转换（(string)）

**转换规则**：
- **整数**：转换为数字字符串（如 `42` → `"42"`）
- **浮点数**：转换为数字字符串，可能使用科学计数法
- **布尔值**：`true` → `"1"`，`false` → `""`（空字符串）
- **null**：转换为 `""`（空字符串）
- **数组**：转换为 `"Array"`（无法获取数组内容）
- **对象**：如果对象实现了 `__toString()`，调用该方法；否则会产生警告并返回类名

### 数组转换（(array)）

**转换规则**：
- **标量类型**：转换为包含一个元素的数组（如 `42` → `[42]`）
- **null**：转换为空数组 `[]`
- **对象**：将对象的属性转换为数组的键值对
- **数组**：保持不变

### 对象转换（(object)）

**转换规则**：
- **数组**：转换为 `stdClass` 对象，数组的键成为对象的属性
- **标量类型**：转换为包含 `scalar` 属性的对象
- **null**：转换为空对象
- **对象**：保持不变

## 基本用法

### 示例 1：整数转换

```php
<?php
declare(strict_types=1);

// 浮点数转整数
$float = 3.14;
$int = (int) $float;
echo "Float: {$float}, Int: {$int}\n";  // Float: 3.14, Int: 3

$float = 3.99;
$int = (int) $float;
echo "Float: {$float}, Int: {$int}\n";  // Float: 3.99, Int: 3（向下取整）

// 字符串转整数
$str = "123";
$int = (int) $str;
echo "String: {$str}, Int: {$int}\n";  // String: 123, Int: 123

$str = "123abc";
$int = (int) $str;
echo "String: {$str}, Int: {$int}\n";  // String: 123abc, Int: 123

$str = "abc123";
$int = (int) $str;
echo "String: {$str}, Int: {$int}\n";  // String: abc123, Int: 0

// 布尔值转整数
$bool = true;
$int = (int) $bool;
echo "Bool: " . ($bool ? 'true' : 'false') . ", Int: {$int}\n";  // Bool: true, Int: 1

$bool = false;
$int = (int) $bool;
echo "Bool: " . ($bool ? 'true' : 'false') . ", Int: {$int}\n";  // Bool: false, Int: 0
```

**执行**：

```bash
php int-conversion.php
```

**输出**：

```
Float: 3.14, Int: 3
Float: 3.99, Int: 3
String: 123, Int: 123
String: 123abc, Int: 123
String: abc123, Int: 0
Bool: true, Int: 1
Bool: false, Int: 0
```

### 示例 2：浮点数转换

```php
<?php
declare(strict_types=1);

// 整数转浮点数
$int = 42;
$float = (float) $int;
echo "Int: {$int}, Float: {$float}\n";  // Int: 42, Float: 42

// 字符串转浮点数
$str = "3.14";
$float = (float) $str;
echo "String: {$str}, Float: {$float}\n";  // String: 3.14, Float: 3.14

$str = "1.23e4";
$float = (float) $str;
echo "String: {$str}, Float: {$float}\n";  // String: 1.23e4, Float: 12300

// 布尔值转浮点数
$bool = true;
$float = (float) $bool;
echo "Bool: true, Float: {$float}\n";  // Bool: true, Float: 1

$bool = false;
$float = (float) $bool;
echo "Bool: false, Float: {$float}\n";  // Bool: false, Float: 0
```

### 示例 3：布尔转换

```php
<?php
declare(strict_types=1);

// 转换为 false 的值
$falseValues = [false, 0, 0.0, "", "0", [], null];

foreach ($falseValues as $value) {
    $bool = (bool) $value;
    echo "Value: ";
    var_dump($value);
    echo "Boolean: " . ($bool ? 'true' : 'false') . "\n";
    echo "---\n";
}

// 转换为 true 的值
$trueValues = [true, 1, -1, 0.1, "1", "hello", [1], new stdClass()];

foreach ($trueValues as $value) {
    $bool = (bool) $value;
    echo "Value: ";
    var_dump($value);
    echo "Boolean: " . ($bool ? 'true' : 'false') . "\n";
    echo "---\n";
}
```

### 示例 4：字符串转换

```php
<?php
declare(strict_types=1);

// 整数转字符串
$int = 42;
$str = (string) $int;
echo "Int: {$int}, String: '{$str}'\n";  // Int: 42, String: '42'

// 浮点数转字符串
$float = 3.14;
$str = (string) $float;
echo "Float: {$float}, String: '{$str}'\n";  // Float: 3.14, String: '3.14'

// 布尔值转字符串
$bool = true;
$str = (string) $bool;
echo "Bool: true, String: '{$str}'\n";  // Bool: true, String: '1'

$bool = false;
$str = (string) $bool;
echo "Bool: false, String: '{$str}'\n";  // Bool: false, String: ''（空字符串）

// 数组转字符串
$arr = [1, 2, 3];
$str = (string) $arr;
echo "Array: " . json_encode($arr) . ", String: '{$str}'\n";  // Array: [1,2,3], String: 'Array'
```

### 示例 5：数组和对象转换

```php
<?php
declare(strict_types=1);

// 标量转数组
$int = 42;
$arr = (array) $int;
print_r($arr);  // Array ( [0] => 42 )

$str = "hello";
$arr = (array) $str;
print_r($arr);  // Array ( [0] => hello )

// null 转数组
$null = null;
$arr = (array) $null;
print_r($arr);  // Array ( )

// 对象转数组
class User
{
    public string $name = "Alice";
    public int $age = 25;
}

$user = new User();
$arr = (array) $user;
print_r($arr);  // Array ( [name] => Alice [age] => 25 )

// 数组转对象
$arr = ['name' => 'Bob', 'age' => 30];
$obj = (object) $arr;
echo "Name: {$obj->name}, Age: {$obj->age}\n";  // Name: Bob, Age: 30
```

## 完整代码示例

### 示例 1：类型转换工具函数

```php
<?php
declare(strict_types=1);

class TypeConverter
{
    public static function toInt(mixed $value): int
    {
        return (int) $value;
    }
    
    public static function toFloat(mixed $value): float
    {
        return (float) $value;
    }
    
    public static function toBool(mixed $value): bool
    {
        return (bool) $value;
    }
    
    public static function toString(mixed $value): string
    {
        if (is_array($value)) {
            return json_encode($value);  // 数组使用 JSON 编码
        }
        return (string) $value;
    }
    
    public static function toArray(mixed $value): array
    {
        return (array) $value;
    }
    
    public static function toObject(mixed $value): object
    {
        return (object) $value;
    }
}

// 使用
$values = [42, 3.14, "123", true, [1, 2, 3]];

foreach ($values as $value) {
    echo "Original: ";
    var_dump($value);
    echo "To int: " . TypeConverter::toInt($value) . "\n";
    echo "To float: " . TypeConverter::toFloat($value) . "\n";
    echo "To string: '" . TypeConverter::toString($value) . "'\n";
    echo "---\n";
}
```

### 示例 2：安全的类型转换

```php
<?php
declare(strict_types=1);

function safeIntConversion(mixed $value, ?int $default = null): ?int
{
    if (is_int($value)) {
        return $value;
    }
    
    if (is_float($value)) {
        return (int) $value;
    }
    
    if (is_string($value) && is_numeric($value)) {
        return (int) $value;
    }
    
    return $default;
}

function safeFloatConversion(mixed $value, ?float $default = null): ?float
{
    if (is_float($value) || is_int($value)) {
        return (float) $value;
    }
    
    if (is_string($value) && is_numeric($value)) {
        return (float) $value;
    }
    
    return $default;
}

// 使用
$values = ["42", "3.14", "abc", 100, 99.99];

foreach ($values as $value) {
    $int = safeIntConversion($value);
    $float = safeFloatConversion($value);
    echo "Value: ";
    var_dump($value);
    echo "Safe int: ";
    var_dump($int);
    echo "Safe float: ";
    var_dump($float);
    echo "---\n";
}
```

## 使用场景

### 数据验证和清理

- **用户输入**：将用户输入转换为期望类型
- **API 数据**：处理来自 API 的数据，转换为期望类型
- **数据库数据**：处理数据库返回的数据

### 计算准备

- **数值计算**：确保计算前类型正确
- **类型统一**：统一数据类型进行计算
- **精度控制**：控制数值精度

### 格式化输出

- **字符串格式化**：将数据转换为字符串格式
- **日志记录**：将各种类型转换为字符串记录
- **数据展示**：将数据转换为可展示的格式

## 注意事项

### 精度丢失

- **浮点数转整数**：会丢失小数部分，使用向下取整
- **大数转换**：超出整数范围会转换为浮点数
- **字符串解析**：字符串解析可能丢失精度

### 字符串解析

- **从开头解析**：字符串转数字从开头解析
- **非数字字符**：遇到非数字字符停止
- **科学计数法**：支持科学计数法格式

### 数组转换

- **数组转字符串**：结果固定为 `"Array"`，无法获取内容
- **使用 JSON**：需要数组内容时使用 `json_encode()`
- **对象转数组**：对象的属性转换为数组的键值对

### 对象转换

- **__toString()**：对象转字符串需要实现 `__toString()` 方法
- **数组转对象**：数组的键成为对象的属性
- **stdClass**：数组转对象使用 `stdClass`

## 常见问题

### 问题 1：浮点数转整数精度丢失

**症状**：浮点数转整数后丢失小数部分

**原因**：整数转换使用向下取整

**错误示例**：

```php
<?php
declare(strict_types=1);

$price = 99.99;
$intPrice = (int) $price;
echo "Price: {$intPrice}\n";  // Price: 99（丢失了 0.99）
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$price = 99.99;

// 方法1：使用 round() 四舍五入
$intPrice = (int) round($price);
echo "Price: {$intPrice}\n";  // Price: 100

// 方法2：使用 ceil() 向上取整
$intPrice = (int) ceil($price);
echo "Price: {$intPrice}\n";  // Price: 100

// 方法3：保持浮点数
$floatPrice = (float) $price;
echo "Price: {$floatPrice}\n";  // Price: 99.99
```

### 问题 2：数组转字符串

**症状**：数组转换为字符串后只显示 "Array"

**原因**：数组无法直接转换为有意义的字符串

**错误示例**：

```php
<?php
declare(strict_types=1);

$arr = [1, 2, 3];
$str = (string) $arr;
echo "String: {$str}\n";  // String: Array（无意义）
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$arr = [1, 2, 3];

// 方法1：使用 json_encode()
$str = json_encode($arr);
echo "String: {$str}\n";  // String: [1,2,3]

// 方法2：使用 serialize()
$str = serialize($arr);
echo "String: {$str}\n";  // String: a:3:{i:0;i:1;i:1;i:2;i:2;i:3;}

// 方法3：使用 implode()（仅适用于索引数组）
$str = implode(', ', $arr);
echo "String: {$str}\n";  // String: 1, 2, 3
```

### 问题 3：对象转字符串

**症状**：对象转换为字符串时产生警告

**原因**：对象未实现 `__toString()` 方法

**错误示例**：

```php
<?php
declare(strict_types=1);

class User
{
    public string $name = "Alice";
}

$user = new User();
$str = (string) $user;  // Warning: Object of class User could not be converted to string
```

**解决方法**：

```php
<?php
declare(strict_types=1);

class User
{
    public function __construct(
        public string $name
    ) {}
    
    public function __toString(): string
    {
        return $this->name;
    }
}

$user = new User("Alice");
$str = (string) $user;
echo "String: {$str}\n";  // String: Alice
```

## 最佳实践

### 显式转换

- **明确意图**：使用显式转换明确代码意图
- **类型安全**：显式转换比隐式转换更安全
- **代码可读性**：显式转换提高代码可读性

### 转换规则

- **理解规则**：理解各种类型的转换规则
- **测试验证**：不确定时进行测试验证
- **文档参考**：参考官方文档了解详细规则

### 错误处理

- **检查结果**：检查转换结果是否符合预期
- **处理异常**：处理转换可能产生的异常
- **提供默认值**：转换失败时提供默认值

## 对比分析

### 类型转换操作符 vs 转换函数

| 特性 | 类型转换操作符 | 转换函数 |
|:-----|:---------------|:---------|
| 语法 | `(type) $value` | `typeval($value)` |
| 性能 | 更好 | 略差 |
| 灵活性 | 低 | 高（支持参数） |
| 可读性 | 中 | 高 |
| 推荐度 | 推荐（基本） | 推荐（需要参数时） |

**选择建议**：
- **基本转换**：使用类型转换操作符
- **需要参数**：使用转换函数（如 `intval()` 的 `$base` 参数）
- **可读性优先**：根据团队偏好选择

### 显式转换 vs 隐式转换

| 特性 | 显式转换 | 隐式转换 |
|:-----|:---------|:---------|
| 明确性 | 高 | 低 |
| 可读性 | 高 | 低 |
| 安全性 | 高 | 低 |
| 推荐度 | 推荐 | 不推荐 |

**选择建议**：
- **优先使用显式转换**：提高代码质量
- **避免隐式转换**：除非确实需要
- **启用严格模式**：限制隐式转换

## 相关章节

- **2.5.1 隐式转换**：了解隐式转换规则
- **2.5.3 转换函数**：了解转换函数的使用
- **2.4.1 标量类型**：了解基本数据类型
- **2.5.4 比较运算符与函数**：了解比较运算符

## 练习任务

1. **基本转换练习**：
   - 练习各种类型的显式转换
   - 理解不同类型的转换规则
   - 测试转换结果
   - 记录转换规则

2. **精度处理练习**：
   - 处理浮点数转整数的精度问题
   - 使用 `round()`、`ceil()`、`floor()` 函数
   - 理解不同取整方式的区别
   - 测试精度处理

3. **数组和对象转换练习**：
   - 练习数组和对象的相互转换
   - 理解转换规则
   - 处理对象转字符串（实现 `__toString()`）
   - 测试转换结果

4. **安全转换练习**：
   - 实现安全的类型转换函数
   - 处理转换失败的情况
   - 提供默认值
   - 测试各种场景

5. **综合练习**：
   - 创建一个类型转换工具类
   - 实现各种类型的转换方法
   - 添加错误处理和验证
   - 进行代码审查，确保转换正确
