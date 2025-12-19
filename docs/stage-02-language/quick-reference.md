# 阶段二：PHP 基础语法速查表

本速查表提供阶段二（PHP 基础语法）中常用函数、方法和操作的快速参考。每个条目包含简要语法、描述和指向详细文档的链接。

## 目录

1. [变量/数据的检测](#1-变量数据的检测)
2. [内容输出、调试相关方法](#2-内容输出调试相关方法)
3. [数据类型的检测和转换](#3-数据类型的检测和转换)
4. [数值运算](#4-数值运算)
5. [字符串操作和常用方法](#5-字符串操作和常用方法)
6. [数组操作和常用方法](#6-数组操作和常用方法)

---

## 1. 变量/数据的检测

### 变量存在性检测

| 函数/运算符 | 描述 | 语法 | 链接 |
| :--------- | :--- | :--- | :--- |
| `isset()` | 检查变量是否存在且不为 `null` | `isset(mixed ...$vars): bool` | [2.16.1](../chapter-16-null-system/section-01-isset-empty.md) |
| `empty()` | 检查变量是否为空（`""`、`0`、`null`、`false`、`[]` 等） | `empty(mixed $var): bool` | [2.16.1](../chapter-16-null-system/section-01-isset-empty.md) |
| `is_null()` | 检查变量是否为 `null` | `is_null(mixed $value): bool` | [2.16.1](../chapter-16-null-system/section-01-isset-empty.md) |
| `??` | 空合并运算符，变量为 `null` 或未定义时返回默认值 | `$var ?? $default` | [2.16.2](../chapter-16-null-system/section-02-null-coalescing.md) |
| `??=` | 空合并赋值运算符，仅在变量为 `null` 或未定义时赋值 | `$var ??= $value` | [2.16.2](../chapter-16-null-system/section-02-null-coalescing.md) |
| `unset()` | 销毁变量 | `unset(mixed $var, mixed ...$vars): void` | [2.3.1](../chapter-03-variables/section-01-variable-basics.md) |

### 类型检测

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `gettype()` | 获取变量的类型名称（不推荐用于类型判断） | `gettype(mixed $value): string` | [2.4.4](../chapter-04-types/section-04-type-detection.md) |
| `is_int()` | 检查是否为整数 | `is_int(mixed $value): bool` | [2.4.4](../chapter-04-types/section-04-type-detection.md) |
| `is_string()` | 检查是否为字符串 | `is_string(mixed $value): bool` | [2.4.4](../chapter-04-types/section-04-type-detection.md) |
| `is_float()` | 检查是否为浮点数 | `is_float(mixed $value): bool` | [2.4.4](../chapter-04-types/section-04-type-detection.md) |
| `is_bool()` | 检查是否为布尔值 | `is_bool(mixed $value): bool` | [2.4.4](../chapter-04-types/section-04-type-detection.md) |
| `is_array()` | 检查是否为数组 | `is_array(mixed $value): bool` | [2.4.4](../chapter-04-types/section-04-type-detection.md) |
| `is_object()` | 检查是否为对象 | `is_object(mixed $value): bool` | [2.4.4](../chapter-04-types/section-04-type-detection.md) |
| `is_null()` | 检查是否为 `null` | `is_null(mixed $value): bool` | [2.4.4](../chapter-04-types/section-04-type-detection.md) |
| `is_numeric()` | 检查是否为数字或数字字符串 | `is_numeric(mixed $value): bool` | [2.4.4](../chapter-04-types/section-04-type-detection.md) |
| `instanceof` | 检查对象是否为指定类的实例 | `$obj instanceof ClassName` | [2.4.4](../chapter-04-types/section-04-type-detection.md) |

---

## 2. 内容输出、调试相关方法

### 输出函数

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `echo` | 输出一个或多个表达式（语言构造，无返回值） | `echo expression [, expression ...]` | [2.2.1](../chapter-02-output/section-01-output-api.md) |
| `print` | 输出一个表达式（语言构造，返回 1） | `print expression` | [2.2.1](../chapter-02-output/section-01-output-api.md) |
| `printf()` | 格式化输出字符串 | `printf(string $format, mixed ...$values): int` | [2.2.1](../chapter-02-output/section-01-output-api.md) |
| `sprintf()` | 格式化字符串并返回（不直接输出） | `sprintf(string $format, mixed ...$values): string` | [2.2.1](../chapter-02-output/section-01-output-api.md) |

### 调试函数

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `var_dump()` | 显示变量的类型、值和结构（详细） | `var_dump(mixed ...$values): void` | [2.2.3](../chapter-02-output/section-03-debugging.md) |
| `print_r()` | 显示变量的可读格式（简洁） | `print_r(mixed $value, bool $return = false): string\|bool` | [2.2.3](../chapter-02-output/section-03-debugging.md) |
| `var_export()` | 输出或返回变量的可执行 PHP 代码 | `var_export(mixed $value, bool $return = false): string\|bool` | [2.2.3](../chapter-02-output/section-03-debugging.md) |
| `debug_backtrace()` | 返回调用栈数组 | `debug_backtrace(int $options, int $limit): array` | [2.2.3](../chapter-02-output/section-03-debugging.md) |
| `debug_zval_dump()` | 显示变量的 zval 引用计数信息 | `debug_zval_dump(mixed ...$values): void` | [2.2.3](../chapter-02-output/section-03-debugging.md) |

### 输出缓冲

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `ob_start()` | 开启输出缓冲 | `ob_start(?callable $callback, int $chunk_size, int $flags): bool` | [2.2.1](../chapter-02-output/section-01-output-api.md) |
| `ob_get_clean()` | 获取缓冲区内容并清空 | `ob_get_clean(): string\|false` | [2.2.1](../chapter-02-output/section-01-output-api.md) |
| `ob_get_contents()` | 获取缓冲区内容（不清空） | `ob_get_contents(): string\|false` | [2.2.1](../chapter-02-output/section-01-output-api.md) |
| `ob_end_flush()` | 输出缓冲区内容并关闭 | `ob_end_flush(): bool` | [2.2.1](../chapter-02-output/section-01-output-api.md) |
| `ob_end_clean()` | 清空缓冲区内容并关闭 | `ob_end_clean(): bool` | [2.2.1](../chapter-02-output/section-01-output-api.md) |

---

## 3. 数据类型的检测和转换

### 类型转换操作符

| 操作符 | 描述 | 语法 | 链接 |
| :---- | :--- | :--- | :--- |
| `(int)` | 转换为整数 | `(int) $value` | [2.5.2](../chapter-05-type-casting/section-02-explicit-conversion.md) |
| `(float)` | 转换为浮点数 | `(float) $value` | [2.5.2](../chapter-05-type-casting/section-02-explicit-conversion.md) |
| `(string)` | 转换为字符串 | `(string) $value` | [2.5.2](../chapter-05-type-casting/section-02-explicit-conversion.md) |
| `(bool)` | 转换为布尔值 | `(bool) $value` | [2.5.2](../chapter-05-type-casting/section-02-explicit-conversion.md) |
| `(array)` | 转换为数组 | `(array) $value` | [2.5.2](../chapter-05-type-casting/section-02-explicit-conversion.md) |
| `(object)` | 转换为对象 | `(object) $value` | [2.5.2](../chapter-05-type-casting/section-02-explicit-conversion.md) |

### 类型转换函数

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `intval()` | 获取变量的整数值 | `intval(mixed $value, int $base = 10): int` | [2.5.3](../chapter-05-type-casting/section-03-conversion-functions.md) |
| `floatval()` | 获取变量的浮点数值 | `floatval(mixed $value): float` | [2.5.3](../chapter-05-type-casting/section-03-conversion-functions.md) |
| `strval()` | 获取变量的字符串值 | `strval(mixed $value): string` | [2.5.3](../chapter-05-type-casting/section-03-conversion-functions.md) |
| `boolval()` | 获取变量的布尔值 | `boolval(mixed $value): bool` | [2.5.3](../chapter-05-type-casting/section-03-conversion-functions.md) |

### JSON 转换

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `json_encode()` | 将值编码为 JSON 字符串 | `json_encode(mixed $value, int $flags = 0, int $depth = 512): string\|false` | [2.4.6](../chapter-04-types/section-06-json.md) |
| `json_decode()` | 解码 JSON 字符串 | `json_decode(string $json, ?bool $associative = null, int $depth = 512, int $flags = 0): mixed` | [2.4.6](../chapter-04-types/section-06-json.md) |

---

## 4. 数值运算

### 算术运算符

| 运算符 | 描述 | 语法 | 链接 |
| :----- | :--- | :--- | :--- |
| `+` | 加法 | `$a + $b` | [2.6.1](../chapter-06-expressions/section-01-arithmetic-operators.md) |
| `-` | 减法 | `$a - $b` | [2.6.1](../chapter-06-expressions/section-01-arithmetic-operators.md) |
| `*` | 乘法 | `$a * $b` | [2.6.1](../chapter-06-expressions/section-01-arithmetic-operators.md) |
| `/` | 除法（返回浮点数） | `$a / $b` | [2.6.1](../chapter-06-expressions/section-01-arithmetic-operators.md) |
| `%` | 取模（整数取余） | `$a % $b` | [2.6.1](../chapter-06-expressions/section-01-arithmetic-operators.md) |
| `**` | 幂运算 | `$a ** $b` | [2.6.1](../chapter-06-expressions/section-01-arithmetic-operators.md) |
| `intdiv()` | 整数除法 | `intdiv(int $dividend, int $divisor): int` | [2.6.1](../chapter-06-expressions/section-01-arithmetic-operators.md) |

### 数学函数

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `abs()` | 绝对值 | `abs(int\|float $num): int\|float` | [2.6.1](../chapter-06-expressions/section-01-arithmetic-operators.md) |
| `round()` | 四舍五入 | `round(float $num, int $precision = 0, int $mode = PHP_ROUND_HALF_UP): float` | [2.4.1](../chapter-04-types/section-01-scalar-types.md) |
| `floor()` | 向下取整 | `floor(int\|float $num): float` | [2.4.1](../chapter-04-types/section-01-scalar-types.md) |
| `ceil()` | 向上取整 | `ceil(int\|float $num): float` | [2.4.1](../chapter-04-types/section-01-scalar-types.md) |
| `max()` | 返回最大值 | `max(mixed ...$values): mixed` | [2.6.1](../chapter-06-expressions/section-01-arithmetic-operators.md) |
| `min()` | 返回最小值 | `min(mixed ...$values): mixed` | [2.6.1](../chapter-06-expressions/section-01-arithmetic-operators.md) |
| `pow()` | 幂运算（函数形式） | `pow(int\|float $base, int\|float $exp): int\|float` | [2.6.1](../chapter-06-expressions/section-01-arithmetic-operators.md) |
| `sqrt()` | 平方根 | `sqrt(float $num): float` | [2.6.1](../chapter-06-expressions/section-01-arithmetic-operators.md) |

### 比较运算符

| 运算符 | 描述 | 语法 | 链接 |
| :----- | :--- | :--- | :--- |
| `==` | 相等（宽松比较） | `$a == $b` | [2.5.4](../chapter-05-type-casting/section-04-comparison.md) |
| `===` | 全等（严格比较） | `$a === $b` | [2.5.4](../chapter-05-type-casting/section-04-comparison.md) |
| `!=` | 不等（宽松比较） | `$a != $b` | [2.5.4](../chapter-05-type-casting/section-04-comparison.md) |
| `!==` | 不全等（严格比较） | `$a !== $b` | [2.5.4](../chapter-05-type-casting/section-04-comparison.md) |
| `<` | 小于 | `$a < $b` | [2.5.4](../chapter-05-type-casting/section-04-comparison.md) |
| `>` | 大于 | `$a > $b` | [2.5.4](../chapter-05-type-casting/section-04-comparison.md) |
| `<=` | 小于等于 | `$a <= $b` | [2.5.4](../chapter-05-type-casting/section-04-comparison.md) |
| `>=` | 大于等于 | `$a >= $b` | [2.5.4](../chapter-05-type-casting/section-04-comparison.md) |
| `<=>` | 太空船运算符（返回 -1、0、1） | `$a <=> $b` | [2.5.4](../chapter-05-type-casting/section-04-comparison.md) |

---

## 5. 字符串操作和常用方法

### 长度计算

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `strlen()` | 获取字符串字节长度 | `strlen(string $string): int` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |
| `mb_strlen()` | 获取字符串字符长度（多字节） | `mb_strlen(string $string, ?string $encoding = null): int` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |

### 查找与定位

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `strpos()` | 查找子串首次出现位置 | `strpos(string $haystack, string $needle, int $offset = 0): int\|false` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |
| `strrpos()` | 查找子串最后出现位置 | `strrpos(string $haystack, string $needle, int $offset = 0): int\|false` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |
| `mb_strpos()` | 多字节查找子串位置 | `mb_strpos(string $haystack, string $needle, int $offset = 0, ?string $encoding = null): int\|false` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |
| `strstr()` | 查找子串并返回剩余部分 | `strstr(string $haystack, string $needle, bool $before_needle = false): string\|false` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |

### 截取子串

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `substr()` | 截取子串（按字节） | `substr(string $string, int $offset, ?int $length = null): string` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |
| `mb_substr()` | 截取子串（按字符，多字节） | `mb_substr(string $string, int $start, ?int $length = null, ?string $encoding = null): string` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |

### 替换操作

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `str_replace()` | 字符串替换 | `str_replace(array\|string $search, array\|string $replace, string\|array $subject, int &$count = null): string\|array` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |
| `str_ireplace()` | 不区分大小写替换 | `str_ireplace(array\|string $search, array\|string $replace, string\|array $subject, int &$count = null): string\|array` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |

### 去除空白

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `trim()` | 去除首尾空白字符 | `trim(string $string, string $characters = " \n\r\t\v\x00"): string` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |
| `ltrim()` | 去除左侧空白字符 | `ltrim(string $string, string $characters = " \n\r\t\v\x00"): string` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |
| `rtrim()` | 去除右侧空白字符 | `rtrim(string $string, string $characters = " \n\r\t\v\x00"): string` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |

### 大小写转换

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `strtolower()` | 转换为小写 | `strtolower(string $string): string` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |
| `strtoupper()` | 转换为大写 | `strtoupper(string $string): string` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |
| `ucfirst()` | 首字母大写 | `ucfirst(string $string): string` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |
| `ucwords()` | 每个单词首字母大写 | `ucwords(string $string, string $separators = " \t\r\n\f\v"): string` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |

### 分割与拼接

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `explode()` | 按分隔符分割字符串 | `explode(string $separator, string $string, int $limit = PHP_INT_MAX): array` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |
| `implode()` | 将数组元素拼接为字符串 | `implode(string $separator, array $array): string` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |
| `join()` | `implode()` 的别名 | `join(string $separator, array $array): string` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |

### 格式化

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `sprintf()` | 格式化字符串 | `sprintf(string $format, mixed ...$values): string` | [2.2.1](../chapter-02-output/section-01-output-api.md) |
| `number_format()` | 格式化数字 | `number_format(float $num, int $decimals = 0, ?string $decimal_separator = ".", ?string $thousands_separator = ","): string` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |

### HTML 转义

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `htmlspecialchars()` | 转义 HTML 特殊字符 | `htmlspecialchars(string $string, int $flags = ENT_QUOTES, ?string $encoding = null, bool $double_encode = true): string` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |
| `htmlentities()` | 转义所有 HTML 实体 | `htmlentities(string $string, int $flags = ENT_QUOTES, ?string $encoding = null, bool $double_encode = true): string` | [2.7.2](../chapter-07-strings/section-02-string-functions.md) |

---

## 6. 数组操作和常用方法

### 数组创建与访问

| 操作/函数 | 描述 | 语法 | 链接 |
| :-------- | :--- | :--- | :--- |
| `[]` | 短数组语法（PHP 5.4+） | `$arr = [1, 2, 3]` | [2.8.1](../chapter-08-arrays/section-01-array-basics.md) |
| `array()` | 数组创建（传统语法） | `$arr = array(1, 2, 3)` | [2.8.1](../chapter-08-arrays/section-01-array-basics.md) |
| `$arr[key]` | 访问数组元素 | `$arr['key']` | [2.8.1](../chapter-08-arrays/section-01-array-basics.md) |
| `$arr[]` | 追加元素 | `$arr[] = $value` | [2.8.1](../chapter-08-arrays/section-01-array-basics.md) |

### 数组检测

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `array_key_exists()` | 检查键是否存在 | `array_key_exists(string\|int $key, array $array): bool` | [2.8.3](../chapter-08-arrays/section-03-array-functions.md) |
| `isset()` | 检查键是否存在且值不为 `null` | `isset(array $array, string\|int $key): bool` | [2.8.1](../chapter-08-arrays/section-01-array-basics.md) |
| `in_array()` | 检查值是否存在 | `in_array(mixed $needle, array $haystack, bool $strict = false): bool` | [2.8.3](../chapter-08-arrays/section-03-array-functions.md) |
| `array_search()` | 查找值并返回键 | `array_search(mixed $needle, array $haystack, bool $strict = false): int\|string\|false` | [2.8.3](../chapter-08-arrays/section-03-array-functions.md) |

### 数组信息获取

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `count()` | 获取数组元素数量 | `count(Countable\|array $value, int $mode = COUNT_NORMAL): int` | [2.8.1](../chapter-08-arrays/section-01-array-basics.md) |
| `array_keys()` | 获取所有键 | `array_keys(array $array, mixed $search_value = null, bool $strict = false): array` | [2.8.3](../chapter-08-arrays/section-03-array-functions.md) |
| `array_values()` | 获取所有值（重新索引） | `array_values(array $array): array` | [2.8.3](../chapter-08-arrays/section-03-array-functions.md) |
| `array_key_first()` | 获取第一个键（PHP 7.3+） | `array_key_first(array $array): int\|string\|null` | [2.8.3](../chapter-08-arrays/section-03-array-functions.md) |
| `array_key_last()` | 获取最后一个键（PHP 7.3+） | `array_key_last(array $array): int\|string\|null` | [2.8.3](../chapter-08-arrays/section-03-array-functions.md) |

### 数组修改

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `array_push()` | 在数组末尾添加元素 | `array_push(array &$array, mixed ...$values): int` | [2.8.1](../chapter-08-arrays/section-01-array-basics.md) |
| `array_pop()` | 移除并返回最后一个元素 | `array_pop(array &$array): mixed` | [2.8.1](../chapter-08-arrays/section-01-array-basics.md) |
| `array_unshift()` | 在数组开头添加元素 | `array_unshift(array &$array, mixed ...$values): int` | [2.8.1](../chapter-08-arrays/section-01-array-basics.md) |
| `array_shift()` | 移除并返回第一个元素 | `array_shift(array &$array): mixed` | [2.8.1](../chapter-08-arrays/section-01-array-basics.md) |
| `unset()` | 删除数组元素 | `unset(array $array, string\|int $key): void` | [2.8.1](../chapter-08-arrays/section-01-array-basics.md) |

### 数组合并与分割

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `array_merge()` | 合并数组 | `array_merge(array ...$arrays): array` | [2.8.3](../chapter-08-arrays/section-03-array-functions.md) |
| `array_merge_recursive()` | 递归合并数组 | `array_merge_recursive(array ...$arrays): array` | [2.8.3](../chapter-08-arrays/section-03-array-functions.md) |
| `array_slice()` | 截取数组片段 | `array_slice(array $array, int $offset, ?int $length = null, bool $preserve_keys = false): array` | [2.8.3](../chapter-08-arrays/section-03-array-functions.md) |
| `array_splice()` | 删除并替换数组片段 | `array_splice(array &$array, int $offset, ?int $length = null, mixed $replacement = []): array` | [2.8.3](../chapter-08-arrays/section-03-array-functions.md) |

### 数组遍历

| 方法 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `foreach` | 遍历数组值 | `foreach ($array as $value)` | [2.8.2](../chapter-08-arrays/section-02-array-iteration.md) |
| `foreach` | 遍历数组键值对 | `foreach ($array as $key => $value)` | [2.8.2](../chapter-08-arrays/section-02-array-iteration.md) |
| `array_walk()` | 对每个元素应用回调函数 | `array_walk(array\|object &$array, callable $callback, mixed $arg = null): bool` | [2.8.2](../chapter-08-arrays/section-02-array-iteration.md) |

### 数组高阶函数

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `array_map()` | 对每个元素应用回调，返回新数组 | `array_map(?callable $callback, array $array, array ...$arrays): array` | [2.8.3](../chapter-08-arrays/section-03-array-functions.md) |
| `array_filter()` | 过滤数组元素 | `array_filter(array $array, ?callable $callback = null, int $mode = 0): array` | [2.8.3](../chapter-08-arrays/section-03-array-functions.md) |
| `array_reduce()` | 归约数组为单个值 | `array_reduce(array $array, callable $callback, mixed $initial = null): mixed` | [2.8.3](../chapter-08-arrays/section-03-array-functions.md) |

### 数组排序

| 函数 | 描述 | 语法 | 链接 |
| :--- | :--- | :--- | :--- |
| `sort()` | 按值升序排序（重新索引） | `sort(array &$array, int $flags = SORT_REGULAR): bool` | [2.8.4](../chapter-08-arrays/section-04-array-sorting.md) |
| `rsort()` | 按值降序排序（重新索引） | `rsort(array &$array, int $flags = SORT_REGULAR): bool` | [2.8.4](../chapter-08-arrays/section-04-array-sorting.md) |
| `asort()` | 按值升序排序（保持键） | `asort(array &$array, int $flags = SORT_REGULAR): bool` | [2.8.4](../chapter-08-arrays/section-04-array-sorting.md) |
| `arsort()` | 按值降序排序（保持键） | `arsort(array &$array, int $flags = SORT_REGULAR): bool` | [2.8.4](../chapter-08-arrays/section-04-array-sorting.md) |
| `ksort()` | 按键升序排序 | `ksort(array &$array, int $flags = SORT_REGULAR): bool` | [2.8.4](../chapter-08-arrays/section-04-array-sorting.md) |
| `krsort()` | 按键降序排序 | `krsort(array &$array, int $flags = SORT_REGULAR): bool` | [2.8.4](../chapter-08-arrays/section-04-array-sorting.md) |
| `usort()` | 使用自定义比较函数排序 | `usort(array &$array, callable $callback): bool` | [2.8.4](../chapter-08-arrays/section-04-array-sorting.md) |
| `uasort()` | 使用自定义比较函数排序（保持键） | `uasort(array &$array, callable $callback): bool` | [2.8.4](../chapter-08-arrays/section-04-array-sorting.md) |
| `uksort()` | 使用自定义比较函数按键排序 | `uksort(array &$array, callable $callback): bool` | [2.8.4](../chapter-08-arrays/section-04-array-sorting.md) |

### 数组解构

| 语法 | 描述 | 链接 |
| :--- | :--- | :--- |
| `[$a, $b] = $array` | 解构数组元素（PHP 7.1+） | [2.8.5](../chapter-08-arrays/section-05-array-destructuring.md) |
| `[$a, , $c] = $array` | 跳过某些元素 | [2.8.5](../chapter-08-arrays/section-05-array-destructuring.md) |
| `['key' => $value] = $array` | 解构关联数组 | [2.8.5](../chapter-08-arrays/section-05-array-destructuring.md) |

---

## 使用说明

本速查表提供阶段二（PHP 基础语法）中常用函数和操作的快速参考。每个条目包含：

- **函数/操作名称**：函数名或操作符
- **语法**：简要的语法格式和参数说明
- **描述**：简要的功能描述
- **链接**：指向详细文档的章节链接

### 查找方法

1. **按主题查找**：根据需要的功能类型，在对应的主题章节中查找
2. **按函数名查找**：使用编辑器的搜索功能（Ctrl+F / Cmd+F）搜索函数名
3. **查看详细文档**：点击链接查看完整的语法说明、参数列表、使用示例和最佳实践

### 注意事项

- 本速查表仅提供简要参考，详细用法请查看对应章节的完整文档
- 部分函数可能有多个重载形式，请查看详细文档了解所有用法
- 建议在学习阶段二时，结合详细文档和实际练习来掌握这些函数和方法

---

**相关章节**：

- [阶段二总览](readme.md)：完整的阶段二学习指南
- [2.21 本章小结与练习](chapter-21-summary/readme.md)：阶段总结与练习指引
