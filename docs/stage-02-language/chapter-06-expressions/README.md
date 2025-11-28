# 2.6 表达式与运算符

## 目标

- 掌握算术、比较、逻辑、赋值、位运算等常见运算符。
- 理解运算符优先级、结合性与短路行为。
- 熟练运用三元、空合并、管道式写法提升可读性。

## 运算符分类

| 类型     | 示例                           |
| :------- | :----------------------------- |
| 算术     | `+ - * / % **`                 |
| 赋值     | `=`、`+=`、`-=`、`**=`         |
| 比较     | `==`、`===`、`!=`、`<>`、`<=>` |
| 逻辑     | `&&`、`||`、`!`、`and`、`or`   |
| 字符串   | `.`（连接）、`.=`              |
| 位运算   | `&`、`\|`、`^`、`<<`、`>>`    |
| 其他     | `??`、`?:`、`match`           |

## 算术运算

- `**`：指数运算，PHP 5.6+。
- `/`：浮点除法；`intdiv($a, $b)` 提供整数除法。
- `%`：取模，符号与被除数一致。

示例：

```php
$result = (4 + 5) * 2 ** 3 / 4; // 18
```

## 赋值运算

- `+=`、`-=`、`.=` 等复合赋值可减少重复。
- 自增/自减：`$i++`（后增）、`++$i`（先增）。

```php
$name = 'PHP';
$name .= ' Guide'; // PHP Guide
```

## 比较运算

- `==` 宽松比较；`===` 严格比较（类型和值均一致）。
- `<=>` 太空船运算符：返回 -1/0/1，可用于排序。

```php
usort($versions, fn ($a, $b) => version_compare($a, $b));
```

## 逻辑运算

- `&&`、`||`：短路逻辑。
- `and`、`or` 结合性不同（优先级低），仅在控制流语句中偶尔使用。
- `??`：空合并运算符。

```php
$cache = $request['cache'] ?? 'redis';
```

## 三元运算符

- 语法：`条件 ? 真值 : 假值`
- PHP 7 引入“省略中间值”的写法：`$value ?: 'default';`

```php
$status = $user->active ? 'Active' : 'Inactive';
$email = $input['email'] ?? fn () => throw new InvalidArgumentException('Email required');
```

## 管道操作符（PHP 8.5+）

- **语法**：`表达式 |> 函数(...)`
- 将表达式的结果作为参数传递给下一个函数，简化函数调用链。

```php
<?php
declare(strict_types=1);

// PHP 8.5+ 管道操作符
$result = "Hello World"
    |> htmlentities(...)
    |> str_split(...)
    |> fn($x) => array_map(strtoupper(...), $x)
    |> fn($x) => array_filter($x, fn($v) => $v !== 'O');

// 等价于传统写法
$result = array_filter(
    array_map(
        strtoupper(...),
        str_split(htmlentities("Hello World"))
    ),
    fn($v) => $v !== 'O'
);
```

### 管道操作符的优势

```php
<?php
declare(strict_types=1);

// 传统写法（嵌套深，难以阅读）
$result = array_map(
    fn($x) => $x * 2,
    array_filter(
        array_map('strlen', explode(' ', 'hello world php')),
        fn($x) => $x > 4
    )
);

// PHP 8.5+ 管道写法（清晰易读）
$result = 'hello world php'
    |> explode(' ', ...)
    |> fn($x) => array_map('strlen', $x)
    |> fn($x) => array_filter($x, fn($v) => $v > 4)
    |> fn($x) => array_map(fn($v) => $v * 2, $x);
```

## `match` 表达式（PHP 8）

- 更安全的 `switch` 替代者，比较使用严格模式。
- 支持返回值，且必须覆盖全部分支。

```php
$label = match ($httpStatus) {
    200, 201 => 'success',
    400 => 'client_error',
    500 => 'server_error',
    default => 'unknown',
};
```

## 位运算

- 常用于权限标记、二进制协议。
- 示例：定义权限位 `READ = 1`, `WRITE = 2`, `EXECUTE = 4`。

```php
$permission = READ | WRITE;
if ($permission & WRITE) {
    // 可写
}
```

## 运算符优先级

| 优先级（高→低）      | 运算符示例                 |
| :------------------- | :------------------------- |
| 括号                 | `( )`                      |
| 指数                 | `**`                       |
| 取正/取负、逻辑非    | `+ - !`                    |
| 乘除模               | `* / %`                    |
| 加减、字符串连接     | `+ - .`                    |
| 移位                 | `<< >>`                    |
| 比较                 | `< <= > >= <>`             |
| 等于、不等           | `== != === !== <=>`        |
| 位与/或/异或         | `& ^ \|`                   |
| 逻辑与/或            | `&& \|\|`                  |
| 空合并               | `??`                       |
| 三元                 | `?:`                       |
| 赋值                 | `=`、`+=` 等               |

> `??` 的优先级低于 `?:`，因此组合使用时需加括号。

## 短路与副作用

- `&&`、`||` 在左侧条件已确定结果时不会执行右侧表达式。
- 慎在条件中写有副作用的表达式，例如：

```php
if ($user && $user->isAdmin()) {
    // 如果 $user 为 null，短路后不会调用 isAdmin
}
```

## 管道式写法

使用链式调用模拟“数据流”：

```php
$result = collect($items)
    ->filter(fn ($item) => $item->active)
    ->map(fn ($item) => strtoupper($item->name))
    ->toArray();
```

## 练习

1. 使用数组高阶函数与 `<=>` 运算符，实现一个支持多字段排序的函数。
2. 设计一组权限位常量，实现 `grantPermission($permission, $flag)` 与 `hasPermission($permission, $flag)`。
3. 将一段嵌套 `if/else` 的代码改写为 `match` 表达式，并添加默认分支。
