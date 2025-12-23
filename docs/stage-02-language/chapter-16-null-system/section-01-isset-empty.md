# 2.16.1 isset、empty 与 is_null

## 概述

`isset()`、`empty()` 和 `is_null()` 是 PHP 中用于检查变量状态的函数。理解它们的区别对于编写正确的代码至关重要。

## isset() - 检查变量是否设置

### 语法

**语法**：`isset(mixed $var, mixed ...$vars): bool`

**参数**：
- `$var, ...$vars`：要检查的变量（可变参数，可传入多个变量）

**返回值**：如果所有变量都存在且不为 `null`，返回 `true`；否则返回 `false`。

### 基本用法

```php
<?php
declare(strict_types=1);

$name = 'Alice';
var_dump(isset($name));  // bool(true)

$age = null;
var_dump(isset($age));   // bool(false)

// 未定义的变量
var_dump(isset($undefined));  // bool(false)
```

### 检查多个变量

```php
<?php
declare(strict_types=1);

$name = 'Alice';
$age = 25;
$email = null;

// 所有变量都必须存在且不为 null
var_dump(isset($name, $age));     // bool(true)
var_dump(isset($name, $age, $email)); // bool(false)
```

### 检查数组键

```php
<?php
declare(strict_types=1);

$user = ['name' => 'Alice', 'age' => 25];

var_dump(isset($user['name']));  // bool(true)
var_dump(isset($user['email'])); // bool(false)

// 检查嵌套键
$data = ['user' => ['name' => 'Alice']];
var_dump(isset($data['user']['name']));  // bool(true)
```

## empty() - 检查是否为空

### 语法

**语法**：`empty(mixed $var): bool`

**参数**：
- `$var`：要检查的变量

**返回值**：如果变量被认为是"空"的，返回 `true`；否则返回 `false`。

### 被认为是"空"的值

以下值在 `empty()` 中返回 `true`：
- `""`（空字符串）
- `0`（整数 0）
- `0.0`（浮点数 0.0）
- `"0"`（字符串 "0"）
- `null`
- `false`
- `[]`（空数组）

### 基本用法

```php
<?php
declare(strict_types=1);

var_dump(empty(""));      // bool(true)
var_dump(empty(0));       // bool(true)
var_dump(empty(0.0));     // bool(true)
var_dump(empty("0"));     // bool(true) - 注意！
var_dump(empty(null));    // bool(true)
var_dump(empty(false));   // bool(true)
var_dump(empty([]));      // bool(true)
var_dump(empty("hello")); // bool(false)
var_dump(empty(42));      // bool(false)
```

### 常见陷阱

```php
<?php
declare(strict_types=1);

// 陷阱：字符串 "0" 被认为是空
$value = "0";
if (empty($value)) {
    echo "Empty\n";  // 会输出，但 "0" 可能是有意义的值
}

// 正确的方式
if ($value === "" || $value === null) {
    echo "Empty\n";
}
```

## is_null() - 检查是否为 null

### 语法

**语法**：`is_null(mixed $value): bool`

**参数**：
- `$value`：要检查的值

**返回值**：如果值为 `null`，返回 `true`；否则返回 `false`。

### 基本用法

```php
<?php
declare(strict_types=1);

var_dump(is_null(null));    // bool(true)
var_dump(is_null(0));       // bool(false)
var_dump(is_null(""));       // bool(false)
var_dump(is_null(false));    // bool(false)
```

### 与 === null 等价

```php
<?php
declare(strict_types=1);

$value = null;

// 两种方式等价
var_dump(is_null($value));  // bool(true)
var_dump($value === null);  // bool(true)
```

## 函数对比

| 函数      | `null` | `0` | `""` | `"0"` | `false` | `[]` | 未定义 |
| :-------- | :----- | :-- | :--- | :---- | :------ | :--- | :----- |
| `isset()` | `false` | `true` | `true` | `true` | `true` | `true` | `false` |
| `empty()` | `true` | `true` | `true` | `true` | `true` | `true` | `true` |
| `is_null()` | `true` | `false` | `false` | `false` | `false` | `false` | 警告 |

## 完整示例

```php
<?php
declare(strict_types=1);

class NullChecks
{
    public static function demonstrate(): void
    {
        $values = [
            'null' => null,
            'zero' => 0,
            'empty_string' => '',
            'zero_string' => '0',
            'false' => false,
            'empty_array' => [],
            'value' => 'hello'
        ];
        
        foreach ($values as $name => $value) {
            echo "=== {$name} ===\n";
            echo "isset: " . (isset($value) ? 'true' : 'false') . "\n";
            echo "empty: " . (empty($value) ? 'true' : 'false') . "\n";
            echo "is_null: " . (is_null($value) ? 'true' : 'false') . "\n";
            echo "\n";
        }
    }
}

NullChecks::demonstrate();
```

## 注意事项

1. **isset() 不产生警告**：检查未定义变量时不会产生警告。

2. **empty() 的陷阱**：`empty("0")` 返回 `true`，处理数字字符串时要小心。

3. **is_null() 会产生警告**：检查未定义变量会产生警告。

4. **推荐使用**：优先使用 `isset()` 和 `=== null`，避免使用 `empty()`。

5. **类型安全**：在严格模式下，使用类型提示和 null 合并运算符。

## 练习

1. 创建一个函数，安全地检查变量是否存在且不为空。

2. 编写一个函数，比较 `isset()`、`empty()` 和 `is_null()` 的行为。

3. 实现一个函数，处理可能为 `null` 或未定义的变量。

4. 创建一个函数，验证表单字段，避免 `empty("0")` 的陷阱。

5. 编写一个函数，使用 `isset()` 检查嵌套数组的键。
