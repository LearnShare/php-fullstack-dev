# 2.12.1 isset、empty 与 is_null

## 概述

`isset`、`empty` 和 `is_null` 是检查变量状态的常用函数。本节详细介绍这三个函数的语法、返回值、使用场景、函数对比及完整示例。

理解这三个函数的区别和适用场景对于正确处理变量状态非常重要。掌握它们的差异可以帮助避免常见的空值判断错误。

## 特性

- **isset**：检查变量是否已设置且不为 `null`，不会触发 Notice
- **empty**：检查变量是否为空（falsy 值），包括 `null`、`false`、`0`、`""`、`[]` 等
- **is_null**：检查变量是否为 `null`，等价于 `=== null`

## 语法/定义

### isset() - 检查变量是否已设置

**语法**：`isset(mixed ...$vars): bool`

**参数**：
- `...$vars`：可变参数，可以检查多个变量

**返回值**：
- 如果所有变量都已设置且不为 `null`，返回 `true`
- 如果任何一个变量未设置或为 `null`，返回 `false`

**特点**：
- 不会触发 `E_NOTICE` 或 `E_WARNING`
- 可以检查多个变量
- 只检查变量是否存在且不为 `null`

### empty() - 检查变量是否为空

**语法**：`empty(mixed $var): bool`

**参数**：
- `$var`：要检查的变量

**返回值**：
- 如果变量为空（falsy 值），返回 `true`
- 否则返回 `false`

**特点**：
- 等价于 `!isset($var) || $var == false`
- 以下值被认为是空：`null`、`false`、`0`、`"0"`、`""`、`[]`、未定义变量
- 不会触发 `E_NOTICE` 或 `E_WARNING`

### is_null() - 检查变量是否为 null

**语法**：`is_null(mixed $value): bool`

**参数**：
- `$value`：要检查的值

**返回值**：
- 如果值为 `null`，返回 `true`
- 否则返回 `false`

**特点**：
- 等价于 `$value === null`
- 如果变量未定义，会触发 `E_NOTICE`
- 只检查是否为 `null`，不检查其他 falsy 值

## 基本用法

### 示例 1：isset() 基本使用

```php
<?php
declare(strict_types=1);

// 检查单个变量
$name = "John";
var_dump(isset($name));  // bool(true)

$age = null;
var_dump(isset($age));   // bool(false)

// 未定义变量不会触发 Notice
var_dump(isset($undefined));  // bool(false)

// 检查多个变量
var_dump(isset($name, $age));  // bool(false)（因为 $age 为 null）
```

### 示例 2：empty() 基本使用

```php
<?php
declare(strict_types=1);

// 空值
var_dump(empty(null));    // bool(true)
var_dump(empty(false));   // bool(true)
var_dump(empty(0));       // bool(true)
var_dump(empty("0"));     // bool(true) - 注意！
var_dump(empty(""));      // bool(true)
var_dump(empty([]));      // bool(true)

// 非空值
var_dump(empty(1));       // bool(false)
var_dump(empty("hello")); // bool(false)
var_dump(empty([1, 2]));  // bool(false)

// 未定义变量不会触发 Notice
var_dump(empty($undefined));  // bool(true)
```

### 示例 3：is_null() 基本使用

```php
<?php
declare(strict_types=1);

// null 值
$value = null;
var_dump(is_null($value));  // bool(true)

// 非 null 值
$value = 0;
var_dump(is_null($value));  // bool(false)

$value = false;
var_dump(is_null($value));  // bool(false)

// 未定义变量会触发 Notice
// var_dump(is_null($undefined));  // Notice + bool(true)
```

## 完整代码示例

### 示例 1：变量状态检查工具

```php
<?php
declare(strict_types=1);

class VariableChecker
{
    public static function isSetAndNotNull(mixed $var): bool
    {
        return isset($var);
    }
    
    public static function isEmpty(mixed $var): bool
    {
        return empty($var);
    }
    
    public static function isNull(mixed $var): bool
    {
        return is_null($var);
    }
    
    public static function hasValue(mixed $var): bool
    {
        return isset($var) && $var !== null && $var !== '';
    }
}

// 使用
$name = "John";
var_dump(VariableChecker::isSetAndNotNull($name));  // bool(true)
var_dump(VariableChecker::isEmpty($name));          // bool(false)

$age = null;
var_dump(VariableChecker::isNull($age));            // bool(true)
```

### 示例 2：表单数据处理

```php
<?php
declare(strict_types=1);

function processFormData(array $data): array
{
    $result = [];
    
    // 使用 isset 检查字段是否存在
    if (isset($data['name'])) {
        $result['name'] = $data['name'];
    }
    
    // 使用 empty 检查字段是否为空（包括空字符串）
    if (!empty($data['email'])) {
        $result['email'] = $data['email'];
    }
    
    // 使用 is_null 检查是否为 null（不包括空字符串）
    if (!is_null($data['age'] ?? null)) {
        $result['age'] = (int) $data['age'];
    }
    
    return $result;
}

// 使用
$formData = [
    'name' => 'John',
    'email' => '',
    'age' => null
];

$processed = processFormData($formData);
print_r($processed);
// Array
// (
//     [name] => John
// )
```

### 示例 3：配置读取

```php
<?php
declare(strict_types=1);

function getConfig(array $config, string $key, mixed $default = null): mixed
{
    // 使用 isset 检查键是否存在
    if (isset($config[$key])) {
        return $config[$key];
    }
    
    return $default;
}

function getConfigOrEmpty(array $config, string $key, mixed $default = null): mixed
{
    // 使用 empty 检查键是否存在且不为空
    if (!empty($config[$key])) {
        return $config[$key];
    }
    
    return $default;
}

// 使用
$config = [
    'host' => 'localhost',
    'port' => 0,  // 注意：0 会被 empty 认为是空
    'debug' => null
];

echo getConfig($config, 'host', 'default') . "\n";      // localhost
echo getConfig($config, 'port', 3306) . "\n";            // 0
echo getConfigOrEmpty($config, 'port', 3306) . "\n";      // 3306（因为 0 被认为是空）
```

## 使用场景

### isset() 使用场景

- **检查变量是否存在**：检查变量是否已定义且不为 `null`
- **数组键检查**：检查数组键是否存在
- **多个变量检查**：同时检查多个变量

### empty() 使用场景

- **表单验证**：检查表单字段是否为空
- **字符串检查**：检查字符串是否为空（包括 `"0"`）
- **数组检查**：检查数组是否为空

### is_null() 使用场景

- **null 检查**：明确检查值是否为 `null`
- **类型检查**：在类型检查中使用
- **可选参数**：检查可选参数是否为 `null`

## 注意事项

### empty("0") 陷阱

- **字符串 "0"**：`empty("0")` 返回 `true`，这可能不符合预期
- **解决方法**：使用 `=== ""` 或 `=== null` 检查

### isset vs is_null

- **未定义变量**：`isset()` 对未定义变量返回 `false`，不会触发 Notice
- **is_null**：`is_null()` 对未定义变量会触发 Notice
- **选择**：检查变量是否存在使用 `isset()`，检查是否为 `null` 使用 `is_null()` 或 `=== null`

### 性能考虑

- **isset() 最快**：`isset()` 性能最好
- **empty() 次之**：`empty()` 性能稍差
- **is_null() 最慢**：`is_null()` 性能最差，但差异很小

## 常见问题

### 问题 1：empty("0") 陷阱

**症状**：字符串 "0" 被认为是空

**原因**：`empty()` 将 `"0"` 视为 falsy 值

**错误示例**：

```php
<?php
declare(strict_types=1);

$value = "0";
if (empty($value)) {
    echo "Value is empty\n";  // 会输出，但 "0" 不是空
}
```

**解决方法**：

```php
<?php
declare(strict_types=1);

$value = "0";

// 方法1：使用 === 检查
if ($value === "" || $value === null) {
    echo "Value is empty\n";
} else {
    echo "Value is not empty\n";  // 会输出
}

// 方法2：使用 trim 检查（如果允许空白）
if (trim($value) === "") {
    echo "Value is empty\n";
}

// 方法3：明确检查
if ($value === null || $value === false || $value === "") {
    echo "Value is empty\n";
}
```

### 问题 2：isset vs is_null 混淆

**症状**：不理解 `isset()` 和 `is_null()` 的区别

**原因**：不理解两个函数的行为差异

**解决方法**：

```php
<?php
declare(strict_types=1);

$value = null;

// isset()：检查变量是否存在且不为 null
var_dump(isset($value));  // bool(false)

// is_null()：只检查是否为 null
var_dump(is_null($value));  // bool(true)

// 未定义变量
// isset() 不会触发 Notice
var_dump(isset($undefined));  // bool(false)

// is_null() 会触发 Notice
// var_dump(is_null($undefined));  // Notice + bool(true)

// 推荐：使用 === null（不会触发 Notice，如果变量未定义）
// var_dump($undefined === null);  // Notice + bool(true)
```

### 问题 3：数组键检查

**症状**：使用 `empty()` 检查数组键时不符合预期

**原因**：`empty()` 会将 `0`、`false` 等视为空

**解决方法**：

```php
<?php
declare(strict_types=1);

$data = [
    'count' => 0,
    'enabled' => false,
    'name' => ''
];

// 错误：empty() 会将 0 和 false 视为空
if (empty($data['count'])) {
    echo "Count is empty\n";  // 会输出，但 count 是 0，不是空
}

// 正确：使用 isset() 或 array_key_exists()
if (isset($data['count'])) {
    echo "Count exists: {$data['count']}\n";  // 会输出：Count exists: 0
}

// 或使用 array_key_exists()
if (array_key_exists('count', $data)) {
    echo "Count exists: {$data['count']}\n";
}
```

## 最佳实践

### 变量检查

- **检查存在**：使用 `isset()` 检查变量是否存在
- **检查空值**：根据需求选择 `empty()` 或显式检查
- **检查 null**：使用 `is_null()` 或 `=== null` 检查是否为 `null`

### 数组操作

- **键存在检查**：使用 `isset()` 或 `array_key_exists()`
- **值检查**：根据需求选择检查方式
- **避免 empty()**：对于可能为 `0` 或 `false` 的值，避免使用 `empty()`

### 性能优化

- **优先使用 isset()**：性能最好
- **避免过度使用**：不要过度使用这些函数
- **缓存结果**：如果需要多次检查，缓存结果

## 对比分析

### isset vs empty vs is_null

| 特性 | isset() | empty() | is_null() |
|:-----|:--------|:--------|:----------|
| 未定义变量 | 返回 false，不触发 Notice | 返回 true，不触发 Notice | 触发 Notice |
| null | 返回 false | 返回 true | 返回 true |
| false | 返回 true | 返回 true | 返回 false |
| 0 | 返回 true | 返回 true | 返回 false |
| "0" | 返回 true | 返回 true | 返回 false |
| "" | 返回 true | 返回 true | 返回 false |
| [] | 返回 true | 返回 true | 返回 false |
| 性能 | 最好 | 中等 | 最差 |
| 推荐度 | 推荐（检查存在） | 推荐（检查空值） | 推荐（检查 null） |

**选择建议**：
- **检查变量是否存在**：使用 `isset()`
- **检查变量是否为空**：根据需求选择 `empty()` 或显式检查
- **检查变量是否为 null**：使用 `is_null()` 或 `=== null`

## 相关章节

- **2.3.1 变量基础**：了解变量的基本概念
- **2.4.3 特殊类型**：了解 `null` 类型
- **2.12.2 空合并运算符**：了解空合并运算符

## 练习任务

1. **函数使用练习**：
   - 练习使用 `isset()`、`empty()`、`is_null()`
   - 理解三个函数的区别
   - 测试各种变量状态
   - 观察函数行为

2. **实际应用练习**：
   - 实现表单数据处理功能
   - 实现配置读取功能
   - 实现变量状态检查工具
   - 测试各种应用场景

3. **陷阱避免练习**：
   - 测试 `empty("0")` 的行为
   - 测试未定义变量的处理
   - 测试数组键检查
   - 避免常见陷阱

4. **综合练习**：
   - 创建一个使用这些函数的程序
   - 实现各种变量检查功能
   - 处理各种边界情况
   - 进行代码审查，确保正确性
