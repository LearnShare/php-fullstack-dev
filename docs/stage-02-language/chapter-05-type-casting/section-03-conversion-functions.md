# 2.5.3 转换函数

## 概述

PHP 提供了专门的转换函数进行类型转换，这些函数比类型转换操作符更灵活，支持更多选项。本节详细介绍 `intval()`、`floatval()`、`boolval()`、`strval()` 的语法、参数、用法及与类型转换操作符的对比。

转换函数提供了比类型转换操作符更多的控制选项，适合需要特定转换行为的场景。理解这些函数的特点可以帮助选择合适的转换方式。

## 特性

- **灵活性**：转换函数提供更多选项和控制
- **可读性**：函数名明确表达转换意图
- **参数控制**：某些函数支持参数控制转换行为
- **兼容性**：与类型转换操作符功能相似，但提供更多选项

## 语法/定义

### intval() - 转换为整数

**语法**：`intval(mixed $value, int $base = 10): int`

**参数**：
- `$value`：要转换的值（任意类型）
- `$base`：可选，进制（2-36），默认为 10

**返回值**：返回转换后的整数值（类型为 `int`）

**特点**：
- 支持不同进制的转换
- 与 `(int)` 操作符功能相似
- 提供进制参数，更灵活

### floatval() - 转换为浮点数

**语法**：`floatval(mixed $value): float`

**参数**：
- `$value`：要转换的值（任意类型）

**返回值**：返回转换后的浮点数值（类型为 `float`）

**别名**：`doubleval()`（功能完全相同）

**特点**：
- 与 `(float)` 操作符功能相似
- 不支持额外参数
- 适合需要函数调用的场景

### boolval() - 转换为布尔值

**语法**：`boolval(mixed $value): bool`

**参数**：
- `$value`：要转换的值（任意类型）

**返回值**：返回转换后的布尔值（类型为 `bool`）

**要求**：PHP 5.5+

**特点**：
- 与 `(bool)` 操作符功能相同
- 提供函数形式，适合函数式编程
- 不支持额外参数

### strval() - 转换为字符串

**语法**：`strval(mixed $value): string`

**参数**：
- `$value`：要转换的值（任意类型）

**返回值**：返回转换后的字符串（类型为 `string`）

**特点**：
- 与 `(string)` 操作符功能相似
- 提供函数形式，适合函数式编程
- 不支持额外参数

## 基本用法

### 示例 1：intval() 基本使用

```php
<?php
declare(strict_types=1);

// 基本转换
$value1 = intval(3.14);
echo "3.14 -> int: {$value1}\n";  // 3.14 -> int: 3

$value2 = intval("123");
echo "\"123\" -> int: {$value2}\n";  // "123" -> int: 123

$value3 = intval("123abc");
echo "\"123abc\" -> int: {$value3}\n";  // "123abc" -> int: 123

// 不同进制
$value4 = intval("1010", 2);  // 二进制
echo "\"1010\" (binary) -> int: {$value4}\n";  // "1010" (binary) -> int: 10

$value5 = intval("FF", 16);  // 十六进制
echo "\"FF\" (hex) -> int: {$value5}\n";  // "FF" (hex) -> int: 255

$value6 = intval("52", 8);  // 八进制
echo "\"52\" (octal) -> int: {$value6}\n";  // "52" (octal) -> int: 42
```

**执行**：

```bash
php intval-example.php
```

**输出**：

```
3.14 -> int: 3
"123" -> int: 123
"123abc" -> int: 123
"1010" (binary) -> int: 10
"FF" (hex) -> int: 255
"52" (octal) -> int: 42
```

### 示例 2：floatval() 基本使用

```php
<?php
declare(strict_types=1);

// 基本转换
$value1 = floatval(42);
echo "42 -> float: {$value1}\n";  // 42 -> float: 42

$value2 = floatval("3.14");
echo "\"3.14\" -> float: {$value2}\n";  // "3.14" -> float: 3.14

$value3 = floatval("1.23e4");
echo "\"1.23e4\" -> float: {$value3}\n";  // "1.23e4" -> float: 12300

$value4 = floatval("3.14abc");
echo "\"3.14abc\" -> float: {$value4}\n";  // "3.14abc" -> float: 3.14
```

### 示例 3：boolval() 基本使用

```php
<?php
declare(strict_types=1);

// 转换为 false 的值
$falseValues = [false, 0, 0.0, "", "0", [], null];

foreach ($falseValues as $value) {
    $bool = boolval($value);
    echo "Value: ";
    var_dump($value);
    echo "Bool: " . ($bool ? 'true' : 'false') . "\n";
    echo "---\n";
}

// 转换为 true 的值
$trueValues = [true, 1, -1, 0.1, "1", "hello", [1]];

foreach ($trueValues as $value) {
    $bool = boolval($value);
    echo "Value: ";
    var_dump($value);
    echo "Bool: " . ($bool ? 'true' : 'false') . "\n";
    echo "---\n";
}
```

### 示例 4：strval() 基本使用

```php
<?php
declare(strict_types=1);

// 基本转换
$value1 = strval(42);
echo "42 -> string: '{$value1}'\n";  // 42 -> string: '42'

$value2 = strval(3.14);
echo "3.14 -> string: '{$value2}'\n";  // 3.14 -> string: '3.14'

$value3 = strval(true);
echo "true -> string: '{$value3}'\n";  // true -> string: '1'

$value4 = strval(false);
echo "false -> string: '{$value4}'\n";  // false -> string: ''（空字符串）

$value5 = strval(null);
echo "null -> string: '{$value5}'\n";  // null -> string: ''（空字符串）
```

## 完整代码示例

### 示例 1：转换函数对比

```php
<?php
declare(strict_types=1);

function compareConversion(mixed $value): void
{
    echo "Original value: ";
    var_dump($value);
    
    // 类型转换操作符
    $int1 = (int) $value;
    $float1 = (float) $value;
    $bool1 = (bool) $value;
    $str1 = (string) $value;
    
    // 转换函数
    $int2 = intval($value);
    $float2 = floatval($value);
    $bool2 = boolval($value);
    $str2 = strval($value);
    
    echo "Operator: int={$int1}, float={$float1}, bool=" . ($bool1 ? 'true' : 'false') . ", str='{$str1}'\n";
    echo "Function: int={$int2}, float={$float2}, bool=" . ($bool2 ? 'true' : 'false') . ", str='{$str2}'\n";
    echo "Match: " . ($int1 === $int2 && $float1 === $float2 && $bool1 === $bool2 && $str1 === $str2 ? 'Yes' : 'No') . "\n";
    echo "---\n";
}

$values = [42, 3.14, "123", true, false, null];

foreach ($values as $value) {
    compareConversion($value);
}
```

### 示例 2：intval() 进制转换

```php
<?php
declare(strict_types=1);

// 不同进制的转换
function convertFromBase(string $value, int $base): void
{
    $decimal = intval($value, $base);
    echo "Base {$base}: '{$value}' -> Decimal: {$decimal}\n";
}

// 二进制
convertFromBase("1010", 2);    // 10
convertFromBase("1111", 2);    // 15

// 八进制
convertFromBase("52", 8);      // 42
convertFromBase("100", 8);     // 64

// 十六进制
convertFromBase("FF", 16);     // 255
convertFromBase("A0", 16);     // 160

// 三十六进制（0-9, a-z）
convertFromBase("Z", 36);      // 35
convertFromBase("10", 36);     // 36
```

### 示例 3：实际应用场景

```php
<?php
declare(strict_types=1);

// 用户输入处理
function processUserInput(mixed $input): array
{
    return [
        'int' => intval($input),
        'float' => floatval($input),
        'bool' => boolval($input),
        'string' => strval($input),
    ];
}

// API 数据处理
function processApiData(array $data): array
{
    return [
        'id' => intval($data['id'] ?? 0),
        'price' => floatval($data['price'] ?? 0.0),
        'active' => boolval($data['active'] ?? false),
        'name' => strval($data['name'] ?? ''),
    ];
}

// 使用
$userInput = "123";
$processed = processUserInput($userInput);
print_r($processed);

$apiData = ['id' => '456', 'price' => '99.99', 'active' => '1', 'name' => 'Product'];
$processed = processApiData($apiData);
print_r($processed);
```

## 使用场景

### intval() 使用场景

- **进制转换**：需要不同进制转换时使用 `intval()` 的 `$base` 参数
- **用户输入**：将用户输入的字符串转换为整数
- **API 数据**：处理来自 API 的字符串数字

### floatval() 使用场景

- **数值计算**：确保数值为浮点数类型
- **价格处理**：处理价格等需要小数的数据
- **科学计算**：处理科学计数法格式的字符串

### boolval() 使用场景

- **函数式编程**：需要函数形式时使用
- **条件判断**：将值转换为布尔值进行判断
- **配置处理**：处理配置选项的布尔值

### strval() 使用场景

- **函数式编程**：需要函数形式时使用
- **数组处理**：在 `array_map()` 等函数中使用
- **日志记录**：将值转换为字符串记录

## 注意事项

### 性能考虑

- **转换函数稍慢**：转换函数比类型转换操作符稍慢，但差异很小
- **优先考虑可读性**：在大多数情况下，性能差异可以忽略
- **性能敏感场景**：在性能敏感的场景中，可以使用类型转换操作符

### 参数使用

- **intval() 的 base 参数**：`intval()` 支持 `$base` 参数，这是类型转换操作符无法提供的
- **其他函数无参数**：`floatval()`、`boolval()`、`strval()` 不支持额外参数
- **选择建议**：需要参数时使用函数，否则可以使用操作符

### 功能对比

- **功能相似**：转换函数与类型转换操作符功能相似
- **intval() 特殊**：`intval()` 支持进制参数，功能更强大
- **其他函数**：其他函数与操作符功能相同

## 常见问题

### 问题 1：何时使用转换函数

**问题**：什么时候使用转换函数，什么时候使用类型转换操作符？

**解答**：

**使用转换函数的情况**：
- 需要 `intval()` 的 `$base` 参数进行进制转换
- 需要函数形式（如函数式编程、回调函数）
- 团队偏好使用函数形式

**使用类型转换操作符的情况**：
- 基本类型转换
- 性能敏感的场景
- 代码简洁性优先

**示例**：

```php
<?php
declare(strict_types=1);

// 需要进制转换：使用 intval()
$binary = intval("1010", 2);  // 10

// 基本转换：可以使用操作符
$int = (int) 3.14;  // 3

// 函数式编程：使用函数
$numbers = array_map('intval', ["1", "2", "3"]);
```

### 问题 2：性能差异

**问题**：转换函数和类型转换操作符的性能差异有多大？

**解答**：

- **差异很小**：性能差异通常可以忽略
- **优先考虑可读性**：在大多数情况下，可读性比性能更重要
- **性能敏感场景**：在性能敏感的场景中，可以使用类型转换操作符

**测试示例**：

```php
<?php
declare(strict_types=1);

$value = 3.14;
$iterations = 1000000;

// 测试类型转换操作符
$start = microtime(true);
for ($i = 0; $i < $iterations; $i++) {
    $result = (int) $value;
}
$time1 = microtime(true) - $start;

// 测试转换函数
$start = microtime(true);
for ($i = 0; $i < $iterations; $i++) {
    $result = intval($value);
}
$time2 = microtime(true) - $start;

echo "Operator: {$time1}s\n";
echo "Function: {$time2}s\n";
echo "Difference: " . (($time2 - $time1) / $time1 * 100) . "%\n";
```

## 最佳实践

### 选择转换方式

- **基本转换**：优先使用类型转换操作符
- **需要参数**：使用 `intval()` 的 `$base` 参数
- **函数式编程**：使用转换函数
- **团队偏好**：根据团队偏好选择

### 代码可读性

- **明确意图**：使用显式转换明确代码意图
- **一致性**：在整个项目中保持一致的转换方式
- **文档说明**：为复杂的转换逻辑添加注释

### 错误处理

- **验证结果**：检查转换结果是否符合预期
- **处理异常**：处理转换可能产生的异常
- **提供默认值**：转换失败时提供默认值

## 对比分析

### 转换函数 vs 类型转换操作符

| 特性 | 转换函数 | 类型转换操作符 |
|:-----|:---------|:---------------|
| 语法 | `intval($value)` | `(int) $value` |
| 性能 | 略差 | 略好 |
| 参数支持 | 是（intval） | 否 |
| 可读性 | 高 | 中 |
| 函数式编程 | 适合 | 不适合 |
| 推荐度 | 按需使用 | 推荐（基本） |

**选择建议**：
- **基本转换**：使用类型转换操作符
- **需要参数**：使用 `intval()` 的 `$base` 参数
- **函数式编程**：使用转换函数
- **团队偏好**：根据团队偏好选择

### 各转换函数对比

| 函数 | 对应操作符 | 特殊功能 | 推荐场景 |
|:-----|:-----------|:---------|:---------|
| `intval()` | `(int)` | 支持进制参数 | 进制转换 |
| `floatval()` | `(float)` | 无 | 函数式编程 |
| `boolval()` | `(bool)` | 无 | 函数式编程 |
| `strval()` | `(string)` | 无 | 函数式编程 |

**选择建议**：
- **intval()**：需要进制转换时使用
- **其他函数**：根据团队偏好和场景选择

## 相关章节

- **2.5.1 隐式转换**：了解隐式转换规则
- **2.5.2 显式转换**：了解类型转换操作符
- **2.5.4 比较运算符与函数**：了解比较运算符
- **2.4.1 标量类型**：了解基本数据类型

## 练习任务

1. **基本转换函数练习**：
   - 练习使用 `intval()`、`floatval()`、`boolval()`、`strval()`
   - 理解各函数的功能和特点
   - 对比转换函数和类型转换操作符
   - 测试转换结果

2. **intval() 进制转换练习**：
   - 练习使用 `intval()` 的 `$base` 参数
   - 转换不同进制的数字
   - 理解进制转换的规则
   - 测试各种进制转换

3. **函数式编程练习**：
   - 在 `array_map()` 中使用转换函数
   - 在回调函数中使用转换函数
   - 理解函数式编程的优势
   - 测试函数式编程场景

4. **实际应用练习**：
   - 实现用户输入处理函数
   - 实现 API 数据处理函数
   - 使用转换函数处理数据
   - 测试不同场景下的转换

5. **综合练习**：
   - 创建一个类型转换工具类
   - 实现多种转换方法
   - 对比不同转换方式的性能
   - 进行代码审查，确保转换正确
