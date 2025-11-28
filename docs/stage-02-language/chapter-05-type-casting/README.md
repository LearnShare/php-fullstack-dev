# 2.5 类型转换与比较

## 目标

- 区分隐式转换、显式转换、函数转换三种方式。
- 理解宽松比较（`==`）与严格比较（`===`）的差异及隐患。
- 掌握常见转换函数（`intval`、`floatval`、`boolval`、`strval`）的语法。

## 隐式转换（Type Juggling）

- 在算术、字符串连接、比较时，PHP 会根据上下文自动转换类型。
- 典型规则：
  - 字符串与数字运算时，字符串被转换为数字，无法解析时为 `0`。
  - 布尔上下文（如 `if`）会将空字符串、`"0"`、`0`、`0.0`、`[]`、`null` 转为 `false`。

示例：

```php
echo "5" + 3;   // 8
echo "5a" + 3; // 8，因为 "5a" 解析为 5
```

> **提示**：隐式转换易导致逻辑漏洞，除非十分确定规则，否则尽量使用显式转换与严格比较。

## 显式转换（Cast）

语法：`(type) $value`

| 转换   | 语法示例          | 说明                               |
| :----- | :---------------- | :--------------------------------- |
| `int`  | `(int) $value`    | 等价 `intval($value)`              |
| `float` | `(float) $value` | 等价 `floatval($value)`            |
| `bool` | `(bool) $value`   | 等价 `boolval($value)`             |
| `string` | `(string) $value` | 等价 `strval($value)`            |
| `array` | `(array) $value` | 非数组转换为一元素数组             |
| `object` | `(object) $value` | 数组变对象或包装标量              |

示例：

```php
$value = "42";
$int = (int) $value;    // 42
$bool = (bool) $value;  // true
```

## 转换函数

### `intval`

- **语法**：`intval(mixed $value, int $base = 10): int`
- **说明**：支持指定基数，默认十进制。
- **示例**：`intval('0xff', 16); // 255`

### `floatval`

- **语法**：`floatval(mixed $value): float`
- **说明**：会忽略无法解析的字符。
- **示例**：`floatval('19.95USD'); // 19.95`

### `boolval`

- **语法**：`boolval(mixed $value): bool`
- **说明**：常配合用户输入转换为布尔值。
- **示例**：`boolval('false'); // true，因为非空字符串`

### `strval`

- **语法**：`strval(mixed $value): string`
- **说明**：将可打印的值转换为字符串。
- **示例**：`strval(123); // "123"`

## 宽松比较 vs 严格比较

| 操作符 | 行为                     | 示例                             |
| :----- | :----------------------- | :------------------------------- |
| `==`   | 比较前先进行类型转换     | `"0" == 0` 为 `true`            |
| `===`  | 比较值与类型（不转换）  | `"0" === 0` 为 `false`          |
| `!=`   | 与 `==` 相反             |                                 |
| `!==`  | 与 `===` 相反            |                                 |

### 常见陷阱

```php
var_dump("0e123" == "0");      // true
var_dump("0" == false);        // true
var_dump("foo" == 0);          // true (PHP 将 "foo" 转为 0)
var_dump([] == false);         // true
```

> 建议始终使用 `===` 和 `!==`，特别在验证密码哈希、token、数字字符串时。

## 比较函数

- `strcmp($a, $b)`：按字典序比较，区分大小写。
- `strcasecmp($a, $b)`：不区分大小写。
- `bccomp($a, $b, $scale)`：高精度比较，处理大数或金额。
- `version_compare($v1, $v2, $operator = null)`：用于语义化版本。

示例：

```php
if (version_compare(PHP_VERSION, '8.2.0', '>=')) {
    // 使用 PHP 8.2 特性
}
```

## `filter_var` 与数据清洗

- `filter_var($value, FILTER_VALIDATE_INT)`：验证是否为整数。
- `filter_var($value, FILTER_SANITIZE_NUMBER_FLOAT, FILTER_FLAG_ALLOW_FRACTION)`：清理字符串，只保留数字与小数点。
- 对布尔、邮箱、URL 也有对应过滤器，适合处理表单输入。

## 自定义转换逻辑

场景：从配置文件读取值，需要根据类型标签转换。

```php
function castValue(string $value, string $type): mixed
{
    return match ($type) {
        'int' => (int) $value,
        'float' => (float) $value,
        'bool' => filter_var($value, FILTER_VALIDATE_BOOLEAN),
        'array' => json_decode($value, true, 512, JSON_THROW_ON_ERROR),
        default => $value,
    };
}
```

## 练习

1. 实现 `toNumber(string $value): ?float`，要求能识别千位分隔符、百分号，并返回 `float` 或 `null`。
2. 利用 `filter_var` 编写一个函数，验证来自表单的布尔字段（`"yes"`, `"no"`, `"true"`, `"false"`），并给出错误提示。
3. 分析以下表达式的结果并解释原因：`"10a" == 10`、`"10a" === 10`、`0 == "0e1234"`、`0 === "0e1234"`。
