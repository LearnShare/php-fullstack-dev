# 2.4 数据类型

## 目标

- 熟悉 PHP 标量、复合、特殊类型的行为及常见陷阱。
- 能根据需求选择合适的数据结构并掌握类型检测方法。
- 理解多字节字符串处理与 JSON/数组互转。

## 类型总览

| 分类     | 类型                              | 说明                           |
| :------- | :-------------------------------- | :----------------------------- |
| 标量     | `bool`、`int`、`float`、`string`  | 基本值类型                     |
| 复合     | `array`、`object`、`callable`     | 可包含多个元素或方法           |
| 特殊     | `null`、`resource`                | 空值或外部资源句柄             |
| 伪类型   | `mixed`、`iterable`、`number` 等  | 文档或类型提示中使用           |

## 标量类型

### 布尔（`bool`）

- 可取 `true`/`false`。
- `is_bool($value)` 判断类型。
- 通过 `filter_var($input, FILTER_VALIDATE_BOOLEAN)` 从字符串解析布尔值。

### 整数（`int`）

- 32 位平台范围约 `-2^31 ~ 2^31-1`；64 位则 `-2^63 ~ 2^63-1`。
- 支持二进制（`0b1010`）、八进制（`012`）、十六进制（`0x1A`）。
- 函数：`is_int`、`intdiv($a, $b)`、`intval($value, $base)`。

### 浮点数（`float`）

- 采用 IEEE 754 双精度，存在精度误差。
- 使用 `round()`、`bcadd()`、`NumberFormatter` 处理货币。

### 字符串（`string`）

- 支持单引号、双引号、Heredoc/Nowdoc。
- 双引号中可使用转义与变量插值。
- UTF-8 多字节处理需使用 `mb_*` 系列函数。

## 常用字符串函数

| 函数       | 语法                                      | 说明                  |
| :--------- | :---------------------------------------- | :-------------------- |
| `strlen`   | `strlen(string $str): int`                | 返回字节长度          |
| `mb_strlen` | `mb_strlen(string $str, ?string $enc = null): int` | 返回字符数，需启用 `mbstring` |
| `substr`   | `substr(string $str, int $start, ?int $length = null): string` | 截取子串 |
| `strpos`   | `strpos(string $haystack, string $needle, int $offset = 0): int|false` | 查找子串位置 |
| `explode`  | `explode(string $delimiter, string $string, int $limit = PHP_INT_MAX): array` | 拆分字符串 |
| `implode`  | `implode(string $glue, array $pieces): string` | 拼接数组             |

示例：

```php
$name = "PHP 学习路线";
echo mb_strlen($name); // 6
echo substr($name, 0, 3); // PHP
```

## 数组（`array`）

- PHP 数组为有序映射，可同时包含数字键与字符串键。
- 创建方式：`[]` 或 `array()`。
- 表达常用结构：列表（索引数组）、字典（关联数组）、栈/队列。

### 高阶函数

- `array_map(callable $callback, array $array): array`
- `array_filter(array $array, ?callable $callback = null, int $mode = 0): array`
- `array_reduce(array $array, callable $callback, mixed $initial = null): mixed`

示例：

```php
$numbers = [1, 2, 3];
$double = array_map(fn ($n) => $n * 2, $numbers);
$sum = array_reduce($numbers, fn ($carry, $n) => $carry + $n, 0);
```

### 排序

- `sort()`、`rsort()`：按值排序并重新索引。
- `asort()`、`ksort()`：保留键名。
- `usort($array, $callback)`：自定义排序逻辑。

## 对象（`object`）

- 由 `class` 定义，实例化后可访问属性与方法。
- `clone` 复制对象，但不会复制 `static` 成员。
- 检测：`is_object($value)`、`$value instanceof ClassName`。

## 可调用（`callable`）

- 包括函数名字符串、`[$object, 'method']`、匿名函数、实现 `__invoke()` 的对象。
- 使用 `is_callable()` 检查。
- 典型用法：高阶函数、事件回调、依赖注入容器。

## `null` 与空值

- `null` 表示“无值”。
- 变量未赋值或显式 `null` 时 `isset` 返回 `false`。
- `??` 空合并运算符提供默认值：`$value = $input ?? 'default';`

## 资源（`resource`）

- 对外部资源（文件、数据库连接、cURL 等）的引用。
- 应通过对应的关闭函数释放：`fclose($stream)`、`mysqli_close($conn)`。
- PHP 8+ 引入 `\CurlHandle` 等对象化封装，逐步替代资源类型。

## 类型检测函数

- `gettype($value)`：返回字符串描述（不推荐做逻辑判断）。
- `is_*` 系列：`is_string`、`is_numeric`、`is_iterable` 等。
- `var_dump($value)`：调试时输出详细类型与值。

## 类型声明与联合类型

- 函数参数与返回值可声明类型：`function foo(int $x): string`.
- PHP 8 支持联合类型：`function handle(int|string $id)`.
- `mixed` 类型表示可以接受所有类型，慎用。

## JSON 与数组互转

- `json_encode($data, JSON_UNESCAPED_UNICODE)`：编码为 JSON。
- `json_decode($json, true, 512, JSON_THROW_ON_ERROR)`：解码为数组，同时抛出异常以便捕获错误。

示例：

```php
$payload = ['id' => 1, 'tags' => ['php', 'mysql']];
$json = json_encode($payload, JSON_UNESCAPED_UNICODE);
$decoded = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
```

## 练习

1. 写一个函数 `summarize(array $items): array`，返回数组元素数量、最大值、最小值与平均值。
2. 使用 `mb_*` 函数统计一段 UTF-8 文本的字数与单词频率。
3. 尝试实现一个 `invoke(callable $callback, array $args = [])` 辅助函数，支持传入闭包、字符串函数名与对象方法。
